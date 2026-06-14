## Hello, Cargo!

Cargo adalah sistem build dan manajer paket Rust. Sebagian besar Rustacean
menggunakan alat ini untuk mengelola proyek Rust mereka karena Cargo menangani
banyak tugas untuk kita, seperti membangun kode, mengunduh pustaka yang dibutuhkan
kode kita, dan membangun pustaka tersebut. (Kita menyebut pustaka yang dibutuhkan
oleh kode kita sebagai _dependensi_.)

Program Rust paling sederhana, seperti yang sudah kita tulis sejauh ini,
tidak memiliki dependensi apa pun. Jika kita membangun proyek “Hello, world!”
dengan Cargo, Cargo hanya akan menggunakan bagian yang menangani proses build
kode kita. Seiring kita menulis program Rust yang lebih kompleks, kita akan
menambahkan dependensi, dan jika kita memulai proyek dengan Cargo, menambahkan
dependensi akan jauh lebih mudah dilakukan.

Karena sebagian besar proyek Rust menggunakan Cargo, sisa buku ini juga
mengasumsikan kita menggunakan Cargo. Cargo sudah terpasang bersama Rust
jika kita menggunakan installer resmi yang dibahas pada bagian
[“Instalasi”][installation]. Jika kita memasang Rust dengan cara lain, 
periksa apakah Cargo sudah terpasang dengan memasukkan perintah berikut di terminal:


```console
$ cargo --version
```

Jika kita melihat nomor versi, berarti Cargo sudah ada! Jika yang muncul adalah error,
seperti `command not found`, lihat dokumentasi dari metode instalasi yang kita gunakan
untuk mengetahui cara memasang Cargo secara terpisah.

### Membuat Proyek dengan Cargo

Mari kita buat proyek baru menggunakan Cargo dan melihat bagaimana perbedaannya
dengan proyek “Hello, world!” yang asli. Arahkan kembali ke direktori _projects_
(atau lokasi lain tempat kita menyimpan kode). Lalu, pada sistem operasi apa pun,
jalankan perintah berikut:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Perintah pertama membuat direktori dan proyek baru bernama _hello_cargo_.
Kita memberi nama proyek ini _hello_cargo_, dan Cargo membuat berkas-berkasnya
di dalam direktori dengan nama yang sama.

Masuklah ke direktori _hello_cargo_ dan lihat daftar berkasnya. Kita akan melihat
bahwa Cargo telah menghasilkan dua berkas dan satu direktori untuk kita: sebuah
berkas _Cargo.toml_ dan direktori _src_ dengan sebuah berkas _main.rs_ di dalamnya.

Cargo juga telah menginisialisasi sebuah repositori Git baru beserta berkas _.gitignore_.
Berkas Git tidak akan dibuat jika kita menjalankan `cargo new` di dalam repositori Git
yang sudah ada; kita bisa menimpa perilaku ini dengan menggunakan `cargo new --vcs=git`.

> Catatan: Git adalah sistem kontrol versi yang umum digunakan. Kita bisa mengubah  
> `cargo new` untuk menggunakan sistem kontrol versi lain atau tanpa sistem kontrol versi  
> dengan menambahkan flag `--vcs`. Jalankan `cargo new --help` untuk melihat opsi yang tersedia.

Buka _Cargo.toml_ di editor teks pilihan kita. Isinya akan terlihat mirip dengan kode  
pada Listing 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Isi *Cargo.toml* yang dihasilkan oleh `cargo new`">  

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Berkas ini menggunakan format [_TOML_][toml] (_Tom’s Obvious, Minimal Language_),
yang merupakan format konfigurasi untuk Cargo.

Baris pertama, `[package]`, adalah judul bagian yang menunjukkan bahwa pernyataan 
berikutnya berfungsi untuk mengonfigurasi sebuah paket. Saat kita menambahkan lebih
banyak informasi ke berkas ini, kita akan menambahkan bagian lain.

Tiga baris berikutnya mengatur informasi konfigurasi yang dibutuhkan Cargo untuk
mengompilasi program kita: nama, versi, dan edition Rust yang digunakan. Kita akan
membahas kunci `edition` di [Lampiran E][appendix-e].

Baris terakhir, `[dependencies]`, adalah awal dari bagian tempat kita mencantumkan
dependensi proyek. Dalam Rust, paket kode disebut _crate_. Untuk proyek ini kita
tidak membutuhkan crate tambahan, tetapi pada proyek pertama di Bab 2 kita akan
membutuhkannya, jadi kita akan menggunakan bagian dependensi ini nanti.

Sekarang buka _src/main.rs_ dan perhatikan isinya:

<span class="filename">Nama berkas: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo telah membuatkan program “Hello, world!” untuk kita, persis seperti yang
kita tulis di Listing 1-1! Sejauh ini, perbedaan antara proyek kita dan proyek
yang dihasilkan Cargo adalah bahwa Cargo menempatkan kode di dalam direktori _src_
dan kita memiliki berkas konfigurasi _Cargo.toml_ di direktori paling atas.

Cargo mengharapkan berkas sumber kita berada di dalam direktori _src_. 
Direktori proyek paling atas hanya untuk berkas README, informasi lisensi, 
berkas konfigurasi, dan hal lain yang tidak langsung berkaitan dengan kode. 
Menggunakan Cargo membantu kita mengatur proyek dengan rapi. Ada tempat untuk 
semuanya, dan semuanya berada di tempatnya.

Jika kita memulai sebuah proyek tanpa Cargo, seperti proyek “Hello, world!”,
kita bisa mengonversinya menjadi proyek yang menggunakan Cargo. Pindahkan kode
proyek ke dalam direktori _src_ dan buat berkas _Cargo.toml_ yang sesuai.
Salah satu cara mudah untuk mendapatkan berkas _Cargo.toml_ adalah dengan
menjalankan `cargo init`, yang akan membuatkannya secara otomatis.

### Membangun dan Menjalankan Proyek Cargo

Sekarang mari kita lihat perbedaan ketika kita membangun dan menjalankan program
“Hello, world!” dengan Cargo! Dari direktori _hello_cargo_, bangun proyek kita
dengan memasukkan perintah berikut:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Perintah ini membuat berkas executable di _target/debug/hello_cargo_  
(atau _target\debug\hello_cargo.exe_ di Windows) alih-alih di direktori kita saat ini.  
Karena build bawaan adalah build debug, Cargo menaruh biner di dalam direktori bernama _debug_.  
Kita bisa menjalankan berkas executable tersebut dengan perintah ini:

```console
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

Jika semuanya berjalan dengan baik, `Hello, world!` akan tercetak di terminal.  
Menjalankan `cargo build` untuk pertama kalinya juga membuat Cargo menghasilkan  
berkas baru di tingkat paling atas: _Cargo.lock_. Berkas ini menyimpan catatan  
versi pasti dari dependensi dalam proyek kita. Proyek ini tidak memiliki dependensi,  
jadi isi berkasnya masih cukup kosong. Kita tidak pernah perlu mengubah berkas ini secara manual;  
Cargo akan mengelola isinya untuk kita.

Kita baru saja membangun proyek dengan `cargo build` dan menjalankannya dengan  
`./target/debug/hello_cargo`, tetapi kita juga bisa menggunakan `cargo run`  
untuk mengompilasi kode dan kemudian menjalankan executable hasilnya hanya dengan satu perintah:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Menggunakan `cargo run` lebih praktis daripada harus mengingat untuk menjalankan  
`cargo build` lalu menggunakan path lengkap ke biner, jadi kebanyakan pengembang  
lebih memilih `cargo run`.

Perhatikan bahwa kali ini kita tidak melihat output yang menunjukkan bahwa Cargo  
sedang mengompilasi `hello_cargo`. Cargo mengetahui bahwa berkas-berkas tidak berubah,  
jadi ia tidak melakukan build ulang dan langsung menjalankan binernya. Jika kita  
mengubah kode sumber, Cargo akan melakukan build ulang proyek sebelum menjalankannya,  
dan kita akan melihat output seperti ini:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo juga menyediakan sebuah perintah yang dipanggil `cargo check`. perintah ini 
secara cepat memeriksa kode kita untuk memastikan bahwa kode tersebut bisa di 
kompilasi tetapi perintah ini tidak menghasilkan file eksekusi/executable:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Kenapa kita mungkin tidak ingin menghasilkan executable? Sering kali,
`cargo check` jauh lebih cepat dibandingkan `cargo build` karena ia melewati
langkah untuk membuat executable. Jika kita terus-menerus memeriksa pekerjaan
selagi menulis kode, menggunakan `cargo check` akan mempercepat proses untuk
memberi tahu apakah proyek kita masih bisa dikompilasi! Karena itu, banyak
Rustacean menjalankan `cargo check` secara berkala saat menulis program untuk
memastikan kodenya tetap bisa dikompilasi. Lalu, mereka menjalankan
`cargo build` ketika sudah siap menggunakan executable.

Mari kita rekap apa yang sudah kita pelajari tentang Cargo sejauh ini:

- Kita bisa membuat proyek dengan `cargo new`.
- Kita bisa membangun proyek dengan `cargo build`.
- Kita bisa membangun sekaligus menjalankan proyek dalam satu langkah dengan `cargo run`.
- Kita bisa membangun proyek tanpa menghasilkan biner untuk mengecek error dengan `cargo check`.
- Alih-alih menyimpan hasil build di direktori yang sama dengan kode kita, Cargo menyimpannya di direktori _target/debug_.

Keuntungan tambahan dari menggunakan Cargo adalah perintah-perintahnya sama
tidak peduli sistem operasi apa yang kita gunakan. Jadi, mulai dari titik ini,
kita tidak lagi memberikan instruksi khusus untuk Linux dan macOS dibandingkan Windows.

### Membangun untuk Rilis

Ketika proyek kita akhirnya siap dirilis, kita bisa menggunakan
`cargo build --release` untuk mengompilasi dengan optimisasi. Perintah ini akan
membuat executable di _target/release_ alih-alih di _target/debug_.
Optimisasi membuat kode Rust kita berjalan lebih cepat, tetapi mengaktifkannya
akan memperpanjang waktu kompilasi program. Inilah alasan mengapa ada dua profil
yang berbeda: satu untuk pengembangan, ketika kita ingin build cepat dan sering,
dan satu lagi untuk membangun program final yang akan kita berikan kepada pengguna,
yang tidak akan dibangun ulang berulang kali dan harus berjalan secepat mungkin.
Jika kita melakukan benchmarking waktu eksekusi kode, pastikan untuk menjalankan
`cargo build --release` dan melakukan benchmark dengan executable di _target/release_.

### Cargo sebagai Konvensi

Untuk proyek sederhana, Cargo mungkin tidak terlihat jauh lebih berguna daripada
sekadar menggunakan `rustc`, tetapi nilainya akan terasa ketika program kita
menjadi lebih rumit. Begitu program berkembang menjadi banyak berkas atau
membutuhkan dependensi, jauh lebih mudah membiarkan Cargo yang mengoordinasikan build.

Meskipun proyek `hello_cargo` sederhana, ia sudah menggunakan banyak tooling nyata
yang akan kita gunakan sepanjang perjalanan kita dengan Rust. Bahkan, untuk
bekerja pada proyek yang sudah ada, kita bisa menggunakan perintah berikut untuk
mengambil kode dengan Git, berpindah ke direktori proyek tersebut, dan melakukan build:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Untuk informasi lebih lanjut tentang Cargo, lihat [dokumentasinya][cargo].

## Ringkasan

Kita sudah memulai perjalanan Rust dengan langkah yang hebat! Dalam bab ini,
kita telah mempelajari cara:

- Memasang versi stabil terbaru Rust menggunakan `rustup`
- Memperbarui Rust ke versi yang lebih baru
- Membuka dokumentasi yang terpasang secara lokal
- Menulis dan menjalankan program “Hello, world!” langsung dengan `rustc`
- Membuat dan menjalankan proyek baru menggunakan konvensi Cargo

Ini adalah waktu yang tepat untuk membangun program yang lebih substansial
agar terbiasa membaca dan menulis kode Rust. Jadi, pada Bab 2, kita akan
membangun program permainan tebak angka. Jika kita lebih suka memulai dengan
mempelajari bagaimana konsep pemrograman umum bekerja di Rust, lihat Bab 3
lalu kembali ke Bab 2.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
