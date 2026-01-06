## Shutdown Santai & Cleanup yang Rapi

Kode di Listing 21-20 sekarang sudah bener-bener nge-handle request secara async
pakai thread pool seperti yang kita pengen. Tapi kita masih dapet warning soal
field `workers`, `id`, dan `thread` yang gak dipakai langsung—ini jadi reminder
kalau kita sebenernya belum beresin proses cleanup. Kalau kita ngehentikan main
thread pakai cara barbar <kbd>ctrl</kbd>-<kbd>C</kbd>, semua thread lain
langsung ikut mati juga, bahkan kalau mereka lagi ngerjain request.

Selanjutnya, kita bakal implement `Drop` trait buat manggil `join` di tiap
thread di pool, supaya mereka bisa selesain request yang lagi dikerjain sebelum
ditutup. Habis itu, kita bakal bikin mekanisme buat ngasih sinyal ke thread
kalau mereka harus berhenti nerima request baru dan shutdown. Biar keliatan
efeknya, nanti kita ubah server kita supaya cuma nerima dua request sebelum
shutdown dengan cara yang “baik-baik”.

Perlu diperhatiin juga: semua yang kita lakukan di sini **gak ngubah bagian kode
yang ngejalanin closure**. Jadi kalaupun thread pool ini dipakai buat async
runtime, bagian ini bakal tetap sama aja.

### Implementasi `Drop` Trait di `ThreadPool`

Kita mulai dengan implement `Drop` buat thread pool. Waktu pool di-drop, semua
threadnya harus di-`join` biar bisa beresin kerjaannya dulu. Listing 21-22
nunjukin percobaan pertama implementasi `Drop`. Tapi kode ini belum bakal jalan
sempurna.

<Listing number="21-22" file-name="src/lib.rs" caption="Nge-join tiap thread waktu thread pool keluar scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Pertama, kita loop ke semua `workers` di thread pool. Kita pakai `&mut` karena
`self` itu reference mutable, dan kita juga bakal perlu mutate tiap `worker`.
Buat tiap `worker`, kita print pesan kalau `Worker` itu lagi shutdown, terus
kita panggil `join` di thread punya dia. Kalau `join` gagal, kita `unwrap` biar
langsung panic aja dan shutdown gak elegan.

Ini error yang muncul pas kita compile:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Error-nya bilang kita gak bisa manggil `join` karena kita cuma punya mutable
borrow ke tiap `worker`, sedangkan `join` butuh ownership. Jadi kita harus
mindahin thread keluar dari `Worker`, supaya bisa dikonsumsi sama `join`.

Satu cara adalah kayak yang kita lakuin di Listing 18-15. Kalau `Worker` nyimpen
`Option<thread::JoinHandle<()>>`, kita bisa `take` nilainya keluar dari `Some`
dan ninggalin `None`. Jadi selama `Worker` masih jalan, dia punya `Some`, dan
waktu dibersihin, kita ubah jadi `None` supaya dia udah gak punya thread lagi.

Tapi masalahnya, ini cuma kepake pas drop aja. Di luar itu, kita jadi ribet
harus selalu nge-handle `Option` tiap kali akses `worker.thread`. Rust emang
sering pakai `Option`, tapi kalau cuma buat “ngakalin” kasus kayak gini, lebih
baik cari solusi lain biar kode tetap clean dan minim error.

Untungnya, ada alternatif yang lebih cakep: method `Vec::drain`. Method ini
nerima range, hapus item di range itu, dan balikin iterator item yang dihapus.
Kalau kita pakai `..`, berarti hapus semua.

Jadi kita update implementasi `drop` di `ThreadPool` jadi kayak gini:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Ini nge-fix error compiler tanpa perlu ubah kode lain. Tapi note penting: karena
`drop` bisa kepanggil pas program lagi panic, `unwrap` di dalamnya juga bisa
panic dan bikin “double panic”, yang bikin program langsung crash total dan
berhenti cleanup. Buat contoh buku ini masih oke lah, tapi di production jelas
gak direkomendasiin.

### Ngasih Sinyal ke Thread Biar Berhenti Dengerin Job

Dengan semua perubahan tadi, kode kita sekarang bisa compile tanpa warning.
Masalahnya, masih belum sesuai yang kita mau. Intinya ada di logic closure yang
jalan di thread `Worker`: saat ini, walaupun kita panggil `join`, threadnya gak
bakal berhenti karena mereka `loop` selamanya nunggu job. Kalau kita drop
`ThreadPool` sekarang, main thread bakal nge-freeze selamanya nunggu thread
pertama selesai.

Buat benerin, kita harus ubah implementasi `drop` di `ThreadPool`, lalu ubah
loop di `Worker`.

Pertama, kita ubah implementasi `drop` biar `sender` di-drop dulu sebelum nunggu
thread selesai. Listing 21-23 nunjukin perubahan buat nge-drop `sender`. Beda
sama thread tadi, di sini kita **butuh** `Option` biar bisa mindahin `sender`
keluar dari `ThreadPool` pakai `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Nge-drop `sender` dulu sebelum nge-join thread `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Waktu `sender` di-drop, channel otomatis ketutup, yang artinya gak bakal ada
pesan baru dikirim. Setelah itu, semua `recv` di loop `Worker` bakal balikin
error. Di Listing 21-24, kita ubah loop `Worker` biar keluar dengan santai kalau
`recv` error. Dengan begitu, thread selesai, dan `drop` bisa manggil `join`
tanpa ke-lock selamanya.

<Listing number="21-24" file-name="src/lib.rs" caption="Keluar dari loop kalau `recv` balikin error">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Biar kelihatan jelas efeknya, kita ubah `main` supaya server cuma nerima dua
request sebelum shutdown santai, kayak di Listing 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Shutdown server setelah serve dua request dengan keluar dari loop">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Jelas di dunia nyata server gak mungkin cuma nerima dua request doang 😂 Ini
cuma buat ngebuktiin kalau graceful shutdown kita udah berfungsi.

Method `take` adalah bagian dari `Iterator`, dan dia ngebatesin iterasi ke dua
item pertama aja. Begitu `main` selesai, `ThreadPool` keluar scope dan `drop`
jalan.

Coba jalanin:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Urutan ID `Worker` mungkin beda di kamu, tapi alurnya sama: `Worker` 0 dan 3
dapet dua request pertama. Server berhenti nerima koneksi setelah request kedua,
dan `Drop` mulai jalan bahkan sebelum `Worker 3` mulai kerja. Waktu `sender`
di-drop, semua `Worker` disconnect dan shutdown. Mereka print pesan disconnect,
terus thread pool `join` mereka satu-satu.

Menariknya di eksekusi ini: ThreadPool nge-drop `sender`, lalu sebelum `Worker`
dapet error dari `recv`, kita udah coba `join Worker 0`. Karena dia belum error,
main thread nunggu. Sementara itu `Worker 3` lanjut kerja dan thread lain dapet
error dan keluar. Begitu `Worker 0` selesai, sisanya udah pada kelar duluan.

Mantap! 🎉 Sekarang kita udah selesai projectnya:

- web server basic
- support thread pool
- async-style handling
- bisa graceful shutdown
- semua thread ke-cleanup dengan baik

Ini full kode finalnya:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Kalau mau lanjut improve, ini ide-idenya:

- Tambahin dokumentasi ke `ThreadPool` dan method-methodnya.
- Tambahin testing.
- Ganti semua `unwrap` jadi error handling yang lebih proper.
- Pakai `ThreadPool` buat tugas selain web server.
- Coba bandingin dengan thread pool crate dari [crates.io](https://crates.io/),
  bikin web server yang sama, lalu bandingin API dan robustness-nya.

## Ringkasan

Keren banget! Kamu udah sampai akhir buku 🎉 Makasih udah nemenin perjalanan
Rust ini. Sekarang kamu udah siap bikin project Rust sendiri atau contribute ke
project orang lain. Jangan lupa, komunitas Rust itu ramah banget dan selalu siap
bantu kalau kamu nemu masalah di perjalanan Rust kamu 🤘
