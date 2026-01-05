## Menerima Argumen dari Command Line

Sekarang kita bikin project baru seperti biasa pakai `cargo new`. Kita kasih
nama project-nya **`minigrep`** supaya beda dari tool `grep` yang mungkin sudah
ada di sistem kamu:

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Tugas pertama kita adalah bikin `minigrep` bisa menerima **dua argumen dari
command line**: path file dan string yang mau dicari. Jadi, kita pengin bisa
jalanin program kaya gini: kita pakai `cargo run`, lalu dua tanda minus (`--`)
buat nandain bahwa argumen setelah itu milik program kita (bukan milik Cargo),
lalu string yang mau dicari, dan path file tempat mencarinya. Contohnya:

```console
cargo run -- searchstring example-filename.txt
```

Saat ini, program yang dibuat oleh `cargo new` belum bisa memproses argumen yang
kita kasih. Sebenarnya sudah ada beberapa library di
[crates.io](https://crates.io/) yang bisa bantu bikin program yang menerima
argumen command line, tapi karena kamu lagi belajar konsepnya, kita bakal bikin
fitur ini sendiri dulu.

### Membaca Nilai Argumen

Supaya `minigrep` bisa membaca nilai argumen command line yang kita kirim, kita
butuh fungsi `std::env::args` dari standard library Rust. Fungsi ini
mengembalikan **iterator** yang berisi argumen-argumen command line yang
diberikan ke `minigrep`.

Kita bakal membahas iterator lebih lengkap di [Chapter 13][ch13]<!-- ignore -->.
Untuk sekarang, cukup tahu dua hal ini:

- Iterator menghasilkan urutan nilai.
- Kita bisa memanggil `collect` pada iterator untuk mengubahnya menjadi koleksi
  (misalnya `vector`) yang berisi semua elemen dari iterator tersebut.

Kode di Listing 12-1 memungkinkan program `minigrep` kamu membaca semua argumen
command line yang diberikan dan mengumpulkannya ke dalam sebuah vector.

<Listing number="12-1" file-name="src/main.rs" caption="Mengumpulkan argumen command line ke dalam sebuah vector dan menampilkannya. ">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Pertama, kita masukin module `std::env` ke dalam scope pakai `use` supaya kita
bisa pakai fungsi `args`. Perhatikan kalau `std::env::args` itu berada dua level
di dalam module. Seperti yang dibahas di
[Chapter 7][ch7-idiomatic-use]<!-- ignore -->, kalau fungsi yang kita butuh ada
jauh di dalam beberapa module, biasanya lebih enak masukin **parent module-nya**
aja, bukan fungsi langsung. Dengan begitu kita bisa gampang pakai fungsi lain
dari `std::env`. Selain itu, ini juga lebih jelas dibanding `use std::env::args`
lalu memanggil `args`, karena nanti `args` bisa disalahpahami seolah-olah fungsi
lokal.

> ### Fungsi `args` dan Unicode Tidak Valid
>
> Perlu dicatat, `std::env::args` bakal panic kalau ada argumen yang mengandung
> Unicode tidak valid. Kalau programmu harus menerima argumen yang mungkin
> berisi Unicode tidak valid, pakai `std::env::args_os` saja. Fungsi itu
> mengembalikan iterator yang menghasilkan `OsString`, bukan `String`. Di sini
> kita tetap pakai `std::env::args` biar simpel, karena `OsString` beda-beda
> tergantung platform dan lebih ribet dipakai dibanding `String`.

Di baris pertama `main`, kita memanggil `env::args`, lalu langsung memanggil
`collect` untuk mengubah iterator-nya jadi vector yang berisi semua argumen yang
diterima. Fungsi `collect` bisa bikin banyak jenis koleksi, jadi kita perlu
kasih tahu tipenya secara eksplisit, dalam hal ini kita mau **vector of
String**. Walaupun biasanya kita jarang banget perlu tulis tipe di Rust, khusus
untuk `collect` kita memang sering butuh anotasi karena Rust nggak bisa nebak
mau jadi koleksi apa.

Terakhir, kita print vector tersebut pakai debug macro. Sekarang coba jalankan
programnya—pertama tanpa argumen sama sekali, lalu dengan dua argumen.

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Perhatikan bahwa nilai pertama di dalam vector adalah `"target/debug/minigrep"`,
yaitu nama binary program kita. Ini sama seperti perilaku daftar argumen di C,
yang memungkinkan program mengetahui nama perintah yang dipakai saat program
dijalankan. Ini sering berguna kalau kamu mau menampilkan nama program di pesan
tertentu atau mengubah perilaku program berdasarkan alias command line yang
dipakai. Tapi untuk kebutuhan bab ini, kita bakal abaikan bagian itu dan fokus
hanya pada dua argumen yang kita perlukan.

### Menyimpan Nilai Argumen ke dalam Variabel

Sekarang program sudah bisa mengakses nilai yang dikasih lewat command line.
Langkah berikutnya, kita perlu menyimpan kedua nilai argumen itu ke dalam
variabel supaya bisa dipakai di bagian lain program. Kita lakukan itu seperti
yang ditunjukkan pada Listing 12-2.

<Listing number="12-2" file-name="src/main.rs" caption=" Membuat variabel untuk menyimpan argumen **query** dan argumen **path file**. ">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Seperti yang sudah kita lihat waktu nge-print vector tadi, nama program ada di
indeks pertama (`args[0]`), jadi kita mulai baca argumen dari indeks 1. Argumen
pertama yang diterima `minigrep` adalah string yang ingin kita cari, jadi kita
simpan referensi ke argumen pertama itu ke variabel `query`. Argumen kedua
adalah path file, jadi kita simpan referensi ke argumen kedua ke variabel
`file_path`.

Untuk sementara, kita print nilai kedua variabel ini dulu untuk memastikan kode
bekerja sesuai yang kita mau. Sekarang coba jalankan lagi programnya dengan
argumen `test` dan `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Mantap, programnya sudah jalan dengan benar! Nilai argumen yang kita butuhkan
sekarang sudah tersimpan di variabel yang tepat. Nanti kita akan tambahkan error
handling untuk menangani kemungkinan masalah, misalnya kalau user tidak
memberikan argumen sama sekali.

Untuk sekarang, kita abaikan dulu kasus itu dan lanjut fokus ke langkah
berikutnya: menambahkan kemampuan membaca file.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
