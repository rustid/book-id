## Packages dan Crates

Bagian pertama dari sistem modul yang bakal kita bahas adalah _packages_ (paket) 
dan _crates_.

Sebuah _crate_ adalah jumlah kode paling kecil yang dipertimbangkan sama _compiler_ 
Rust dalam satu waktu. Walaupun kita jalanin `rustc` bukannya `cargo` terus 
masukin satu file source code (kayak yang kita lakuin dulu sekali di “Menulis 
dan Menjalankan Program Rust” di Bab 1), _compiler_ nganggep file itu sebagai 
sebuah crate. Crates bisa isinya modul, dan modul itu mungkin didefinisikan di 
file lain yang bakal di-compile barengan sama crate-nya, kayak yang bakal kita 
liat di bagian-bagian selanjutnya.

Sebuah crate bisa dateng dalam salah satu dari dua bentuk: sebuah _binary crate_ 
atau sebuah _library crate_. _Binary crates_ adalah program yang bisa kita 
compile jadi _executable_ yang bisa dijalanin, kayak program _command line_ 
atau sebuah server. Masing-masing harus punya fungsi namanya `main` yang 
nentuin apa yang terjadi pas program _executable_ itu jalan. Semua crates yang 
udah kita bikin sejauh ini adalah _binary crates_.

_Library crates_ nggak punya fungsi `main`, dan mereka nggak di-compile jadi 
_executable_. Sebaliknya, mereka mendefinisikan fungsionalitas yang tujuannya 
buat di-share ke banyak project. Misalnya, `rand` crate yang kita pake di 
[Bab 2][rand] nyediain fungsionalitas buat nge-generate angka random. Kebanyakan 
waktu pas Rustacean bilang “crate,” maksudnya adalah _library crate_, dan mereka 
pake kata “crate” secara bergantian sama konsep pemrograman umum dari sebuah 
“library” (perpustakaan).

_Crate root_ adalah file sumber (source file) tempat _compiler_ Rust mulai 
dan ngebentuk modul _root_ (akar) dari crate kita (kita bakal jelasin modul 
secara mendalam di [“Mendefinisikan Modul untuk Mengontrol Scope dan 
Privasi”][modules]).

Sebuah _package_ adalah bundel dari satu atau lebih crates yang nyediain 
sekumpulan fungsionalitas. Sebuah package punya file _Cargo.toml_ yang ngejelasin 
gimana cara nge-build crates itu. Cargo sebenernya adalah sebuah package yang 
isinya _binary crate_ buat tool _command line_ yang selama ini kita pake buat 
nge-build kode kita. Package Cargo juga punya sebuah _library crate_ yang 
bergantung pada _binary crate_-nya. Project lain bisa bergantung pada _library 
crate_ Cargo buat pake logika yang sama kayak yang dipake sama tool _command line_ 
Cargo.

Sebuah package bisa isinya sebanyak apa pun _binary crates_ yang kita mau, 
tapi maksimal cuma boleh punya satu _library crate_. Sebuah package minimal 
harus punya satu crate, entah itu _library crate_ atau _binary crate_.

Yuk kita telusuri apa yang terjadi pas kita bikin sebuah package. Pertama kita 
jalanin perintah `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Setelah kita jalanin `cargo new my-project`, kita pake `ls` buat liat apa yang 
dibuat sama Cargo. Di direktori project-nya, ada file _Cargo.toml_, yang ngasih 
kita sebuah package. Ada juga direktori _src_ yang isinya _main.rs_. Buka file 
_Cargo.toml_ di text editor kita, dan perhatiin kalau nggak ada sebutan soal 
_src/main.rs_. Cargo ngikutin konvensi kalau _src/main.rs_ itu adalah _crate root_ 
dari sebuah _binary crate_ dengan nama yang sama kayak package-nya. Sama juga, 
Cargo tau kalau direktori package-nya punya _src/lib.rs_, berarti package itu 
punya _library crate_ dengan nama yang sama kayak package-nya, dan _src/lib.rs_ 
itu adalah _crate root_-nya. Cargo bakal ngasih file _crate root_ ini ke `rustc` 
buat nge-build _library_ atau _binary_-nya.

Di sini, kita punya package yang isinya cuma _src/main.rs_, artinya dia cuma 
punya sebuah _binary crate_ namanya `my-project`. Kalau sebuah package punya 
_src/main.rs_ sama _src/lib.rs_, dia punya dua crates: sebuah _binary_ dan 
sebuah _library_, yang keduanya punya nama yang sama kayak package-nya. Sebuah 
package bisa punya banyak _binary crates_ dengan naruh file-file-nya di direktori 
_src/bin_: tiap file bakal jadi _binary crate_ yang terpisah.

[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
