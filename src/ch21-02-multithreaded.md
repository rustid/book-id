<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## Dari Server Single-Threaded ke Multithreaded

Sekarang server kita masih ngeproses request **satu per satu**. Artinya, server
gak bakal proses koneksi kedua sebelum koneksi pertama selesai. Kalau server
dapet makin banyak request, eksekusi model _serial_ kayak gini makin gak ideal.
Kalau ada satu request yang prosesnya lama, request-request berikutnya harus
nunggu sampai yang lama selesai, bahkan kalau request barunya sebenernya bisa
diproses cepat. Kita bakal benerin ini, tapi sebelumnya kita lihat dulu
masalahnya secara langsung.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Simulasi Request yang Lambat

Kita bakal lihat gimana satu request lambat bisa ngeganggu request lain di
server kita sekarang. Listing 21-10 nambahin handler buat request ke _/sleep_
yang bakal nyimulasikan respons lambat dengan **delay 5 detik** sebelum
ngerespon.

<Listing number="21-10" file-name="src/main.rs" caption="Ngesimulasikan request lambat dengan `sleep` selama lima detik">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Sekarang kita ganti dari `if` ke `match` karena udah ada tiga kondisi. Kita
harus match langsung ke _slice_ dari `request_line` buat bandingin dengan
literal string; `match` gak otomatis melakukan referencing/dereferencing kayak
pengecekan `==`.

Arm pertama sama kayak blok `if` di Listing 21-9. Arm kedua buat request ke
_/sleep_. Kalau dapet request ini, server bakal tidur 5 detik dulu baru kirim
HTML sukses. Arm ketiga sama kayak blok `else` di Listing 21-9.

Di sini keliatan banget server kita masih sangat sederhana—library beneran tentu
punya cara lebih rapi buat handle banyak route 😆

Sekarang jalanin server pakai `cargo run`. Buka dua tab browser:

- _[http://127.0.0.1:7878](http://127.0.0.1:7878)_
- _[http://127.0.0.1:7878/sleep](http://127.0.0.1:7878/sleep)_

Kalau buka `/` beberapa kali, responsnya bakal cepat. Tapi kalau buka `/sleep`,
lalu coba buka `/`, kamu bakal lihat kalau `/` **ikut nunggu** 5 detik sampai
`/sleep` selesai dulu.

Ada banyak cara buat ngatasi antrean request kayak gini, termasuk pakai async
kayak di Chapter 17, tapi di sini kita bakal pakai **thread pool**.

### Ningkatin Throughput dengan Thread Pool

**Thread pool** itu kumpulan thread yang udah dibuat duluan dan siap kerja.
Ketika ada task masuk, salah satu thread di pool bakal ditugasin ngerjain.
Thread lain tetap standby buat task berikutnya. Kalau tugas selesai, thread
balik lagi ke pool.

Dengan ini server bisa proses beberapa koneksi sekaligus, jadi throughput
meningkat.

Kita bakal batasi jumlah thread biar aman dari DoS attack. Kalau tiap request
bikin thread baru tanpa batas, orang iseng tinggal spam 10 juta request dan
server kamu tamat 💀

Jadi:

- kita punya jumlah thread tetap
- request masuk dimasukin ke queue
- tiap thread ambil job dari queue
- kerjain
- balik ngantri lagi

Dengan ini kita bisa handle sampai **N request paralel** (di mana N = jumlah
thread). Kalau semuanya lagi sibuk ngerjain task lama, request baru tetap
ngantri, tapi pool kita jauh lebih kuat dibanding satu thread doang.

Ini cuma satu dari banyak teknik buat naikin performa server. Kamu juga bisa
explore:

- fork/join
- async I/O single-threaded
- async I/O multithreaded

Karena Rust itu low-level friendly, semua itu memungkinkan.

Sebelum implementasi thread pool, kita pikirin dulu **seharusnya API-nya seperti
apa**. Kayak biasa, kita tulis dulu “cara pakainya”, baru bangun
implementasinya.

Kali ini kita pakai gaya **compiler-driven development**: tulis dulu kode yang
kita pengen, biarin compiler marah, terus kita perbaiki berdasarkan errornya 😎

Tapi sebelum itu, kita lihat dulu pendekatan naive sebagai titik awal.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Bikin Thread Baru untuk Tiap Request

Pertama bayangin kalau tiap koneksi kita bikin thread baru. Ini bukan solusi
akhir karena jelas bakal bahaya, tapi ini titik awal buat bikin server
multithreaded.

Listing 21-11 nunjukin perubahan di `main`.

<Listing number="21-11" file-name="src/main.rs" caption="Bikin thread baru buat tiap stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Kayak yang kamu pelajari di Chapter 16, `thread::spawn` bakal bikin thread baru
dan ngejalanin closure di dalamnya.

Kalau kamu coba jalanin ini, buka `/sleep`, terus buka `/` di tab lain, kamu
bakal lihat sekarang `/` **gak ikut nunggu lagi**.

Tapi ya balik lagi: kalau request jutaan, thread jutaan juga… system kamu bisa
langsung KO.

Dan ya, ini juga kasus pas banget di mana **async/await** bersinar ✨ Tapi
sekarang kita fokus ke thread pool dulu.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Bikin Jumlah Thread yang Terbatas

Kita pengen API thread pool tetap mirip biar gampang migrasinya.

Listing 21-12 nunjukin API idealnya:

<Listing number="21-12" file-name="src/main.rs" caption="Interface `ThreadPool` yang kita pengen">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Kita bikin:

```rust
let pool = ThreadPool::new(4);
```

Lalu pakai:

```rust
pool.execute(|| {
    handle_connection(stream);
});
```

Mirip `thread::spawn`, tapi jumlah thread fix.

Kodenya belum bisa compile—dan emang sengaja. Kita biarin compiler jadi mentor
kita 😎

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Ngebangun `ThreadPool` dengan Compiler-Driven Development

Masukin kode Listing 21-12, jalanin `cargo check`.

Error pertama:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Compiler bilang: “Bro, `ThreadPool` gak ada tuh”

Oke, kita bikin library dulu.

Bikin _src/lib.rs_:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

Import ke main:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

Cek lagi, error berikutnya bilang: “`new` gak ada”

Kita tambahin:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Kenapa `usize`? Karena jumlah thread gak mungkin negatif, dan dipakai buat
ukuran koleksi.

Cek lagi, sekarang error: “`execute` gak ada”

Kita definisikan dulu skeleton-nya:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Kita pakai `FnOnce + Send + 'static` biar closure aman dipindahin antar thread.

Dan yea, akhirnya compile 🎉 Walau… servernya masih gak ngapa-ngapain 🤣

> Jadi, “Kalau Rust compile berarti pasti jalan” itu gak selalu benar ya 😆

---

#### Validasi Jumlah Thread di `new`

Pool dengan 0 thread itu konyol, jadi kita proteksi:

<Listing number="21-13" file-name="src/lib.rs" caption="`ThreadPool::new` panic kalau size = 0">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

---

#### Nyediain Tempat Buat Nyimpen Thread

Kita butuh nyimpen thread. `thread::spawn` balikin `JoinHandle`, jadi kita
simpen itu.

Listing 21-14:

<Listing number="21-14" file-name="src/lib.rs" caption="Bikin vector buat nyimpen thread">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Sekarang compile lagi. Masih aman.

---

<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Ngirim Kode dari `ThreadPool` ke Thread

Masalah: `thread::spawn` butuh closure langsung. Tapi kita pengen thread standby
dulu, nunggu job dikirim nantinya.

Solusinya: kita bikin struct **Worker**.

Worker itu kayak pegawai restoran: nunggu order, kalau ada → kerjain.

Plan:

1. bikin `Worker`
2. `ThreadPool` sekarang nyimpen `Vec<Worker>`
3. tiap worker punya thread sendiri
4. kita kasih id buat debugging

Listing 21-15:

<Listing number="21-15" file-name="src/lib.rs" caption="`ThreadPool` sekarang nyimpen `Worker` bukan langsung thread">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Sekarang worker udah ada, tapi belum ngejalanin job.

---

#### Ngirim Job ke Thread Lewat Channel

Di sini kita pakai **channel** (Chapter 16).

Flow-nya:

1. pool punya sender
2. worker punya receiver
3. `execute()` kirim job
4. worker terima job
5. worker jalanin job

Listing 21-16:

<Listing number="21-16" file-name="src/lib.rs" caption="ThreadPool nyimpen sender channel `Job`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

Kita coba oper receiver ke tiap worker (Listing 21-17), tapi error karena
channel cuma boleh 1 consumer.

Jadi: pakai `Arc<Mutex<T>>`

Listing 21-18:

<Listing number="21-18" file-name="src/lib.rs" caption="Share receiver pakai Arc + Mutex">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

Compile ✔️

---

#### Implementasi `execute`

Sekarang bikin Job sebagai trait object:

Listing 21-19:

<Listing number="21-19" file-name="src/lib.rs" caption="Bikin alias `Job` dan kirim lewat channel">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Sekarang worker harus jalanin job:

Listing 21-20:

<Listing number="21-20" file-name="src/lib.rs" caption="Worker nerima dan ngejalanin job">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Dan… Thread pool kita resmi jalan 🎉🎉🎉

Coba `cargo run` terus spam request.

Outputnya bakal keliatan worker ngerjain job bergantian.

---

Catatan penting: Kalau kamu penasaran versi `while let` (21-21), itu kelihatan
keren tapi salah perilaku. Karena Mutex lock-nya kepegang terlalu lama.

---

Sekarang:

- thread pool jalan
- maksimal cuma 4 thread
- request lambat gak ganggu request lain

Server kita udah jauh lebih “dewasa” 🤘
