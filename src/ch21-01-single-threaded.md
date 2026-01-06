## Membangun Web Server Single-Threaded

Kita bakal mulai dulu dengan bikin web server yang jalan pake **single thread**.
Sebelum mulai, kita lihat dulu gambaran singkat tentang protokol yang terlibat
saat bikin web server. Detail lengkap soal protokol ini sebenernya di luar fokus
buku ini, tapi gambaran singkatnya udah cukup kok buat kebutuhan kita.

Dua protokol utama yang dipake web server adalah **Hypertext Transfer Protocol
(HTTP)** dan **Transmission Control Protocol (TCP)**. Dua-duanya adalah protokol
tipe _request-response_, artinya **client** bakal ngirim request dan **server**
bakal dengerin lalu ngasih response balik. Isi request dan response itu diatur
sama protokolnya.

TCP itu protokol level bawah yang ngatur gimana data dikirim dari satu server ke
server lain, tapi dia gak nentuin isi datanya itu apa. HTTP dibangun di atas TCP
buat ngatur isi request dan response-nya. Secara teknis HTTP bisa jalan di atas
protokol lain, tapi hampir semua kasus HTTP dikirim lewat TCP. Di sini kita
bakal kerja langsung sama raw bytes request dan response HTTP dan TCP.

### Ngedengerin Koneksi TCP

Web server kita harus bisa dengerin koneksi TCP, jadi ini hal pertama yang bakal
kita kerjain. Standard library Rust punya module `std::net` yang bisa bantu
kita. Bikin project baru kayak biasa:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Sekarang masukin kode Listing 21-1 ke _src/main.rs_. Kode ini bakal dengerin
alamat lokal `127.0.0.1:7878` buat nerima TCP stream baru. Kalau ada koneksi
masuk, dia bakal nge-print `Connection established!`.

<Listing number="21-1" file-name="src/main.rs" caption="Ngedengerin incoming stream dan nge-print pesan kalau ada stream masuk">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Dengan `TcpListener`, kita bisa dengerin koneksi TCP di alamat `127.0.0.1:7878`.
Bagian sebelum titik dua itu IP address komputer kamu (ini sama di semua
komputer, bukan khusus penulis ya), dan `7878` itu portnya. Kita pilih port ini
karena:

- HTTP biasanya gak jalan di port ini, jadi kecil kemungkinan bentrok dengan
  server lain
- dan “7878” kalau diketik di keypad telepon bacaannya mirip “rust”

Fungsi `bind` di sini mirip kayak `new`, yang balikannya instance `TcpListener`
baru. Disebut `bind` karena di dunia networking, proses ngiket ke port buat
dengerin koneksi itu disebut “binding to a port”.

`bind` balikin `Result<T, E>` karena bisa aja gagal, misalnya kalau kita jalanin
dua instance program ini sekaligus di port yang sama. Karena ini cuma server
buat belajar, kita gak ribet handle errornya — cukup `unwrap` aja biar program
stop kalau error.

Method `incoming` di `TcpListener` balikin iterator berisi stream TCP
(`TcpStream`). Satu _stream_ itu satu koneksi aktif antara client dan server.
“Connection” itu keseluruhan proses request–response: client connect, server
balas, lalu server tutup koneksi. Jadi kita bakal baca dari `TcpStream` buat
lihat apa yang dikirim client, lalu nulis response ke stream buat dibalikin ke
client. Loop `for` bakal proses tiap koneksi satu-satu.

Untuk sekarang, kita cuma `unwrap` stream biar program berhenti kalau error;
kalau aman, kita print pesan. Nanti bagian suksesnya bakal kita tambahin. Kenapa
bisa error? Karena yang kita iterasi sebenernya “usaha koneksi”, bukan selalu
koneksi yang berhasil. OS punya batas maksimal koneksi, kalau lewat batas bisa
gagal.

Ayo coba jalanin! Jalankan `cargo run`, terus buka _127.0.0.1:7878_ di browser.
Browser bakal nunjukin error kayak “Connection reset” karena server belum
ngirimin apa-apa. Tapi di terminal kamu harusnya kelihatan ini:

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Kadang satu request bisa bikin beberapa pesan muncul. Bisa karena browser juga
minta resource lain kayak _favicon.ico_. Bisa juga browser nyoba connect ulang
karena server gak ngasih response. Kalau `stream` drop (keluar scope), koneksi
ditutup, browser bisa nyoba lagi otomatis.

Beberapa browser juga sengaja buka banyak koneksi kosong dulu biar nanti kalau
ada request beneran bisa lebih cepat. Server kita bakal tetap lihat koneksinya,
walaupun belum ada requestnya.

Intinya, kita udah sukses dapet koneksi TCP

Jangan lupa stop pakai <kbd>ctrl</kbd>-<kbd>C</kbd> kalau mau berhenti, dan
setiap abis ngedit kode, jalankan ulang `cargo run`.

### Ngebaca Request

Sekarang kita siap baca request dari browser! Biar rapi, kita bikin fungsi baru
buat handle koneksi. Di fungsi baru `handle_connection`, kita bakal baca data
dari stream lalu print buat lihat apa yang dikirim browser. Ubah kodenya jadi
kayak Listing 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Membaca `TcpStream` dan nge-print datanya">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Kita import `BufReader` dan trait I/O supaya bisa baca tulis stream. Di loop
`main`, sekarang kita panggil `handle_connection` sambil passing `stream`.

Di `handle_connection`, kita bungkus stream pake `BufReader` biar bacanya lebih
efisien. Kita bikin variabel `http_request` buat nampung semua baris request.
`lines()` bakal balikin iterator `Result<String, Error>`, jadi kita `map` dan
`unwrap` biar dapet `String`.

Browser nandain request selesai dengan dua newline kosong berturut-turut, jadi
kita ambil line sampai ketemu string kosong. Setelah terkumpul, kita print pake
pretty debug supaya gampang dibaca.

Coba jalanin lagi, buka browser ke alamat tadi. Browser masih error, tapi di
terminal sekarang bakal kelihatan request HTTP lengkapnya.

Sekarang kita udah tau browser ngirim apa. Saatnya balas

### Ngeliat Lebih Deket HTTP Request

HTTP itu protokol berbasis teks, format requestnya:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Baris pertama adalah **request line**.

- bagian pertama: method (`GET`, `POST`, dll)
- bagian kedua: URI yang diminta (contohnya `/`)
- bagian terakhir: versi HTTP

CRLF (`\r\n`) adalah pemisah antar bagian.

Di output kita kelihatan:

- method → `GET`
- path → `/`
- versi → `HTTP/1.1`

Sisanya adalah header. `GET` biasanya gak punya body.

Coba request alamat lain misalnya _127.0.0.1:7878/test_ biar lihat perbedaan
requestnya.

Sekarang kita udah paham apa yang diminta browser, waktunya balas balik!

### Nulis Response

Response HTTP formatnya kayak gini:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Contoh response super simpel:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Ini berarti:

- sukses
- gak ada header
- gak ada body

Sekarang kita tulis response ini ke stream. Ganti `println!` sebelumnya dengan
kode di Listing 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Nulis HTTP response sederhana ke stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Kita simpan response di variabel `response`, ubah jadi bytes pakai `as_bytes`,
lalu kirim ke stream. Kalau gagal, `unwrap` bakal berhentiin program.

Sekarang coba jalanin dan buka browser lagi. Kamu bakal dapat halaman kosong —
tapi sekarang server kita udah bener-bener balas request

### Balikin HTML Beneran

Sekarang kita balikin halaman HTML beneran. Bikin file baru _hello.html_ di root
project (bukan di _src_). Isi bebas, contohnya kayak Listing 21-4.

<Listing number="21-4" file-name="hello.html" caption="Contoh HTML yang bakal dikirim sebagai response">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Sekarang ubah `handle_connection` jadi kayak Listing 21-5 supaya:

- baca file HTML
- masukin ke response body
- kirim ke client

<Listing number="21-5" file-name="src/main.rs" caption="Ngirim isi *hello.html* sebagai body response">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Sekarang jalankan `cargo run`, buka browser ke _127.0.0.1:7878_, dan halaman
HTML kamu bakal muncul

Saat ini kita masih ngabaikan isi request dan selalu balikin file yang sama,
walaupun user akses path lain. Jadi server kita masih “ polos banget”.

### Validasi Request & Respon Selektif

Sekarang kita bikin server cuma balikin HTML kalau request-nya ke `/`. Kalau
selain itu, kita balikin error. Edit `handle_connection` kayak Listing 21-6.

<Listing number="21-6" file-name="src/main.rs" caption="Ngebedain request ke */* sama selain itu">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Kita cuma baca baris pertama request, cek apakah sama dengan GET `/`. Kalau iya
→ balikin html. Kalau enggak → nanti kita isi else-nya.

Kalau sekarang kamu request:

- `/` → dapet HTML
- selain itu → error koneksi

Sekarang lanjut, kita tambahin response 404 kayak Listing 21-7.

<Listing number="21-7" file-name="src/main.rs" caption="Balikin 404 + halaman error kalau bukan */*">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Bikin file _404.html_ juga (contohnya Listing 21-8).

<Listing number="21-8" file-name="404.html" caption="Contoh halaman error buat response 404">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Sekarang:

- buka `/` → dapet hello.html
- buka `/apa-aja` → dapet halaman 404

### Sedikit Refactoring

Sekarang `if` dan `else` kita agak banyak duplikasi: dua-duanya masih baca file
dan kirim stream. Bedanya cuma status dan nama file.

Mari kita ringkas kayak Listing 21-9.

<Listing number="21-9" file-name="src/main.rs" caption="Ngerapihin kode biar lebih clean dan gak duplikatif">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Sekarang lebih clean, gampang dibaca, dan gampang di-maintain.

Keren! Sekarang kita punya web server sederhana, sekitar 40 baris Rust:

- bisa nerima koneksi
- baca request
- balikin halaman normal
- balikin halaman 404 kalau salah path

Tapi sekarang servernya masih **single-threaded**, artinya cuma bisa handle satu
request dalam satu waktu. Di bagian selanjutnya kita bakal lihat kenapa ini
masalah kalau ada request lambat, dan gimana cara bikin servernya bisa handle
banyak request sekaligus.
