<!-- Old link, do not remove -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Menginstal Binaries dengan `cargo install`

Perintah `cargo install` memungkinkan kita buat menginstal dan memakai _binary 
crates_ secara lokal. Ini tidak ditujukan buat menggantikan sistem _packages_; 
ini ditujukan sebagai cara yang praktis buat para _developer_ Rust untuk 
menginstal *tools* yang udah di-_share_ sama orang lain di 
[crates.io](https://crates.io/). Perhatikan bahwa kita cuma bisa menginstal 
_packages_ yang punya _binary targets_. Sebuah _binary target_ adalah program 
yang bisa dijalankan (_runnable_) yang dibikin kalau _crate_ tersebut punya 
file _src/main.rs_ atau file lain yang ditentukan sebagai _binary_, kebalikan 
dari _library target_ yang tidak bisa dijalankan secara mandiri melainkan cocok 
buat dimasukkan ke dalam program lain. Biasanya, _crates_ punya informasi di 
dalam file _README_ soal apakah sebuah _crate_ itu _library_, punya _binary 
target_, atau dua-duanya.

Semua _binaries_ yang diinstal pakai `cargo install` disimpan di dalam folder 
_bin_ di direktori _root_ instalasi. Kalau kita menginstal Rust pakai 
_rustup.rs_ dan tidak punya konfigurasi kustom apa pun, direktori ini bakal ada 
di _$HOME/.cargo/bin_. Pastikan direktori tersebut ada di dalam `$PATH` kita 
biar kita bisa menjalankan program-program yang udah kita instal pakai 
`cargo install`.

Misalnya, di Bab 12 kita sempat menyinggung kalau ada implementasi Rust dari 
_tool_ `grep` yang bernama `ripgrep` buat nyari-nyari file. Buat menginstal 
`ripgrep`, kita bisa menjalankan yang berikut ini:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

Dua baris terakhir dari output-nya menunjukkan lokasi dan nama dari _binary_ 
yang udah diinstal, yang mana di kasus `ripgrep` namanya adalah `rg`. Selama 
direktori instalasinya ada di dalam `$PATH` kita, seperti yang udah disebutkan 
sebelumnya, kita kemudian bisa menjalankan `rg --help` dan mulai memakai _tool_ 
yang lebih kencang dan lebih bergaya Rust buat nyari file!
