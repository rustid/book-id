# Bikin Game Tebak Angka

Yuk langsung gas belajar Rust lewat proyek langsung praktek!. Di bab ini kamu
bakal kenalan sama konsep-konsep Rust yang sering dipakai, tapi bukan cuma teori
doang, langsung dipraktekin di program beneran. Kamu bakal ketemu `let`,
`match`, method, associated function, external crate, dan temen-temennya. Di bab
selanjutnya baru kita kupas lebih dalam. Sekarang fokus dulu ke basic-nya sambil
praktik.

Kita bakal ngerjain problem klasik buat pemula: **game tebak angka**, Kurang
lebih gini cara mainnya: Program bakal bikin angka random dari 1 sampai 100 Kamu
diminta masukin tebakan Abis itu program bakal ngasih tau tebakan kamu
**kekecilan** atau **kegedean** kalo pas… **boom!** Tebakan kamu bener, program
bakal ngucapin selamat dan selesai

## Setup Project Baru

Biar bisa mulai, masuk dulu ke folder _projects_ yang kamu bikin di Chapter 1,
terus bikin project baru pakai Cargo kayak gini:

```console
cargo new guessing_game
cd guessing_game
```

Perintah pertama, `cargo new`, pake nama project (`guessing_game`) sebagai
argumen pertamanya. Perintah kedua buat pindah ke folder project yang baru
dibuat.

Sekarang cek dulu file _Cargo.toml_ yang otomatis dibuat

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Seperti yang sudah kamu lihat di Chapter 1, `cargo new` otomatis bikin program
**“Hello, world!”** buat kamu. Sekarang cek file `src/main.rs`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Sekarang yuk kita **compile** program “Hello, world!” ini dan langsung jalanin
sekaligus pakai perintah:

```console
cargo run
```

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Perintah `cargo run` ini bakal kepake banget kalau kamu lagi butuh **iterasi
cepet** pas ngembangin project kayak yang bakal kita lakuin di game ini. Jadi
kamu bisa ngetes perubahan tiap langkah tanpa ribet.

Sekarang buka lagi file _src/main.rs_. Semua kode bakal kamu tulis di file ini.

## Memproses Tebakan

Bagian pertama dari program tebak angka ini bakal: minta input dari user, ngolah
inputnya, terus ngecek apakah inputnya udah sesuai format yang diharapkan.
Pertama-tama, kita bakal ngasih pemain kesempatan buat masukin tebakan. Masukin
kode di **Listing 2-1** ke dalam file `src/main.rs`.

<Listing number="2-1" file-name="src/main.rs" caption="Kode yang ngambil tebakan dari user terus nampilin (nge-print) tebakannya">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Kode ini lumayan banyak isinya, jadi mari kita bahas baris demi baris. Biar bisa
nerima input dari user terus nampilin hasilnya, kita perlu masukin library
input/output `io` ke dalam scope. Library `io` ini berasal dari standard library
Rust, yaitu `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Secara default, Rust sudah punya sekumpulan item dari standard library yang
otomatis dimasukin ke setiap program. Kumpulan ini disebut _**prelude**_, dan
kamu bisa lihat isinya lengkap di dokumentasi standard library.

Kalau tipe yang pengin kamu pakai nggak ada di prelude, kamu harus masukin
sendiri ke scope pakai `use`. Dengan `std::io`, kamu dapet banyak fitur berguna,
termasuk kemampuan buat nerima input dari user.

Seperti yang udah kamu lihat di Chapter 1, fungsi `main` adalah titik masuk
utama program:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Sintaks `fn` dipakai buat deklarasi fungsi baru, tanda kurung `()` berarti nggak
ada parameter dan kurung kurawal `{` menandakan awal dari isi fungsi.

Seperti yang juga kamu pelajari di Chapter 1, `println!` adalah macro yang
nge-print string ke output.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Kode ini menampilkan prompt yang ngasih tahu game apaan ini dan minta input dari
user.

### Menyimpan Nilai dengan Variabel

Selanjutnya, kita bakal bikin **variabel** untuk nyimpen input user, kayak gini:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Sekarang programnya mulai makin menarik! Ada cukup banyak hal yang terjadi di
satu baris kecil ini. Kita pakai pernyataan `let` untuk membuat variabel. Ini
contoh lainnya:

```rust,ignore
let apples = 5;
```

Baris ini membuat variabel baru bernama `apples` dan mengikatnya ke nilai `5`.
Di Rust, variabel itu **immutable secara default**, artinya setelah kita kasih
nilai ke variabel, nilainya nggak bisa diubah lagi. Kita bakal bahas ini lebih
detail di bagian **“Variables and Mutability”** di Chapter 3. Untuk membuat
variabel bisa diubah (mutable), kita tambahkan `mut` sebelum nama variabel:

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> Catatan: Sintak `//` memulai komentar yang berlanjut sampai akhir baris. Rust
> mengabaikan semua yang ada di dalam komentar. Kita akan membahas komentar
> lebih detail di [Chapter3][comments]<!-- ignore -->.

Kembali ke program tebak angka, sekarang kamu tahu bahwa `let mut guess` akan
memperkenalkan variabel mutable bernama `guess`. Tanda sama dengan (`=`) memberi
tahu Rust bahwa kita ingin langsung mengikat sesuatu ke variabel tersebut. Di
sebelah kanan tanda sama dengan adalah nilai yang diikat ke `guess`, yaitu hasil
pemanggilan `String::new`, sebuah fungsi yang mengembalikan instance baru dari
`String`. [`String`][string]<!-- ignore --> adalah tipe string yang disediakan
oleh standard library yang bisa bertambah ukurannya dan berisi teks UTF-8.

Sintaks `::` pada baris `::new` menunjukkan bahwa `new` adalah **associated
function** dari tipe `String`. **Associated function** adalah fungsi yang
diimplementasikan pada sebuah tipe, dalam kasus ini `String`. Fungsi `new` ini
membuat string baru yang kosong. Kamu akan menemukan fungsi `new` pada banyak
tipe karena ini adalah nama umum untuk fungsi yang membuat nilai baru.

Secara keseluruhan, baris `let mut guess = String::new();` telah membuat
variabel mutable yang saat ini terikat pada instance baru dan kosong dari
`String`. Whew!

### Menerima Input dari User

Ingat, tadi kita sudah masukin fitur input/output dari standard library dengan
`use std::io;` di baris pertama program. Sekarang kita bakal manggil fungsi
`stdin` dari module `io`, yang bakal kita pakai buat ngurus input dari user:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Kalau kita **nggak** meng-import module `io` dengan `use std::io;` di awal
program, kita tetap bisa pakai fungsinya dengan nulis pemanggilannya sebagai
`std::io::stdin`. Fungsi `stdin` ini bakal balikin sebuah instance dari
[`std::io::Stdin`][iostdin]<!-- ignore -->, yaitu tipe yang mewakili _handle_ ke
input standar terminal kamu.

Selanjutnya, baris `.read_line(&mut guess)` akan memanggil method
[`read_line`][read_line]<!-- ignore --> pada _input handle standard_ untuk
mengambil input dari user. Kita juga mengoper `&mut guess` sebagai argumen ke
`read_line` untuk kasih tahu string mana yang harus dipakai buat nyimpen input
user. Tugas penuh `read_line` adalah mengambil apapun yang user ketik di
standard input dan **menambahkan** itu ke dalam sebuah string (tanpa menghapus
isi sebelumnya), jadi kita kasih string itu sebagai argumen. String tersebut
harus mutable supaya method-nya bisa mengubah isinya.

Tanda `&` menunjukkan bahwa argumen ini adalah sebuah **reference**, yaitu cara
buat beberapa bagian kode mengakses data yang sama tanpa harus menyalin datanya
berkali-kali ke memori. Reference itu topik yang lumayan kompleks, dan salah
satu keunggulan besar Rust adalah reference-nya aman dan relatif mudah dipakai.
Kamu gak perlu paham detailnya untuk menyelesaikan program ini. Untuk sekarang,
semua yang perlu kamu ketahui bahwa, sama kayak variabel, reference itu
**immutable secara default**. Makanya kita perlu nulis `&mut guess` bukan
`&guess` supaya bisa diubah. (Chapter 4 bakal jelasin reference ini lebih
lengkap.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Menangani Kemungkinan Error dengan `Result`

Kita masih ngebahas baris kode yang sama. Sekarang kita masuk ke baris teks
ketiga, tapi perlu diingat ini sebenarnya masih bagian dari **satu baris logika
kode yang sama**. Bagian berikutnya adalah method ini:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Kita sebenarnya juga bisa nulis kodenya seperti ini:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Namun, satu baris yang terlalu panjang bakal susah dibaca, jadi lebih baik
dibagi beberapa baris. Biasanya lebih baik untuk menambahkan newline dan
whitespace lain untuk “memecah” baris panjang saat kamu memanggil method dengan
sintaks `.method_name()`. Sekarang mari kita bahas apa yang sebenarnya dilakukan
baris ini.

Seperti yang sudah disebutkan sebelumnya, `read_line` akan memasukkan apa pun
yang diketik user ke dalam string yang kita berikan, tapi fungsi ini juga
mengembalikan nilai bertipe `Result`. [`Result`][result]<!-- ignore --> adalah
sebuah [_enumeration_][enums]<!-- ignore --> (sering disebut _enum_), yaitu tipe
yang bisa berada dalam beberapa kemungkinan keadaan. Setiap kemungkinan keadaan
itu disebut _variant_.

[Chapter 6][enums]<!-- ignore --> akan membahas enum lebih dalam. Tujuan dari
tipe `Result` ini adalah untuk membawa informasi terkait penanganan error.

Variant dari `Result` adalah `Ok` dan `Err`.`Ok` berarti operasi berhasil, dan
dia menyimpan nilai hasil keberhasilan tersebut. `Err` berarti operasi gagal,
dan dia menyimpan informasi tentang bagaimana atau kenapa kegagalan itu terjadi.

Nilai bertipe `Result`, sama seperti tipe lain, juga punya method. Sebuah
instance `Result` punya method [`expect`][expect]<!-- ignore --> yang bisa kamu
panggil. Jika instance `Result` tersebut adalah `Err`, `expect` akan membuat
program crash dan menampilkan pesan yang kamu berikan ke `expect`. Jika
`read_line` mengembalikan `Err`, biasanya itu karena ada masalah dari sistem
operasi. Kalau instance `Result` tersebut adalah `Ok`, `expect` akan mengambil
nilai yang ada di dalam `Ok` dan mengembalikannya ke kamu supaya bisa dipakai.
Dalam kasus ini, nilainya adalah jumlah byte dari input user.

Kalau kamu tidak memanggil `expect`, program tetap akan berhasil dikompilasi,
tapi kamu akan mendapatkan peringatan:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust ngasih peringatan karena kamu nggak pakai nilai `Result` yang dikembalikan
dari `read_line`, yang artinya program belum menangani kemungkinan terjadinya
error.

Cara yang “benar” buat ngilangin peringatan itu adalah dengan benar-benar nulis
kode penanganan error. Tapi, dalam kasus kita sekarang, kita cuma pengin
programnya langsung crash kalau ada masalah, jadi kita bisa pakai `expect`. Kamu
bakal belajar cara “pulih” dari error di [Bab 9][recover]<!-- ignore -->.

### Nampilin Nilai dengan Placeholder `println!`

Selain kurung kurawal penutup, masih ada satu baris lagi yang perlu dibahas:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Baris ini bakal nge-print string yang sekarang udah berisi input dari user.
Tanda kurung kurawal `{}` itu jadi semacam “tempat dudukan” nilai, anggap aja
`{}` kayak capit kepiting kecil yang megang nilai biar bisa ditampilin. Kalau
mau nge-print nilai dari sebuah variabel, tinggal taruh nama variabelnya di
dalam `{}`. Kalau mau nge-print hasil dari sebuah ekspresi, cukup pakai `{}`
yang kosong di format string, lalu setelah stringnya kasih daftar ekspresi yang
dipisahkan koma sesuai urutan placeholder `{}` tadi. Nge-print variabel dan
hasil ekspresi sekaligus dalam satu `println!` bakal kelihatan seperti ini:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Kode itu bakal nge-print: `x = 5 and y + 2 = 12`

### Ngetes Bagian Pertama

Sekarang kita tes dulu bagian pertama dari game tebak angkanya. Jalanin
programnya pakai: `cargo run`

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Pada tahap ini, bagian pertama dari game-nya sudah kelar kita berhasil ngambil
input dari keyboard terus kita tampilin lagi.

## Membuat Angka Rahasia

Selanjutnya, kita perlu bikin angka rahasia yang nanti harus ditebak sama user.
Angka ini harus beda setiap kali game dijalankan biar tetap seru dimainkan
berkali-kali. Kita bakal pakai angka random antara 1 sampai 100 supaya gamenya
nggak terlalu susah.

Rust sendiri belum punya fitur angka random di standard library-nya. Tapi tim
Rust sudah nyediain crate bernama [`rand`][randcrate] yang punya fitur itu.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-crate-to-get-more-functionality"></a>

### Nambah Fungsionalitas dengan Crate

Ingat, **crate** itu kumpulan file kode sumber Rust. Project yang lagi kita
bikin sekarang adalah **binary crate**, yaitu program yang bisa dieksekusi.
Sementara itu, crate `rand` adalah **library crate**, yang isinya kode buat
dipakai di program lain dan nggak bisa dijalankan sendiri.

Bagian ngatur crate eksternal ini adalah salah satu hal yang bikin Cargo
keliatan keren banget. Sebelum kita bisa nulis kode yang pakai `rand`, kita
harus ngubah file _Cargo.toml_ dulu buat nambahin `rand` sebagai dependency.

Buka file itu sekarang, lalu tambahin baris berikut di bagian paling bawah,
tepat di bawah header `[dependencies]` yang sebelumnya udah dibuat Cargo buat
kamu. Pastikan kamu tulis `rand` persis kayak ini, termasuk versi nomornya,
karena kalau beda bisa bikin contoh kode di tutorial ini nggak jalan:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Di file _Cargo.toml_, semua yang ada setelah sebuah header akan jadi bagian dari
section itu sampai ada section baru dimulai. Di bagian `[dependencies]`, kamu
kasih tahu Cargo crate eksternal apa aja yang dibutuhin project kamu dan versi
berapa yang ingin dipakai.

Di kasus ini, kita nentuin crate `rand` dengan semantic version `0.8.5`. Cargo
paham soal [Semantic Versioning][semver]<!-- ignore --> (sering disebut
_SemVer_), yaitu standar penulisan versi. Penulisan `0.8.5` ini sebenarnya
singkatan dari `^0.8.5`, yang artinya versi berapapun yang minimal 0.8.5 dan
masih di bawah 0.9.0.

Cargo nganggep versi-versi dalam rentang itu punya API yang masih kompatibel
dengan 0.8.5. Dengan begitu, kamu bakal selalu dapet patch terbaru tapi tetap
aman dan bisa dikompilasi sesuai contoh di bab ini. Versi 0.9.0 ke atas nggak
dijamin punya API yang sama lagi.

Sekarang, tanpa mengubah kode apa pun, coba build project-nya dulu seperti yang
ditunjukkan di Listing 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption=" Output yang muncul saat menjalankan `cargo build` setelah menambahkan crate `rand` sebagai dependency. ">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

Kamu mungkin akan melihat nomor versi yang berbeda (tapi semuanya tetap
kompatibel kok berkat SemVer!) dan mungkin juga baris output yang berbeda
tergantung sistem operasi yang kamu pakai, bahkan urutannya juga bisa nggak
sama.

Saat kita nambahin dependency eksternal, Cargo akan ngambil versi terbaru dari
semua dependency yang dibutuhin lewat _registry_, yaitu salinan data dari
[Crates.io][cratesio]. Crates.io itu tempat orang-orang di ekosistem Rust
nge-publish proyek Rust open source mereka supaya bisa dipakai orang lain.

Setelah registry diperbarui, Cargo ngecek bagian `[dependencies]`, lalu
nge-download crate yang belum ada di lokal. Di kasus ini, meskipun kita cuma
nulis `rand` sebagai dependency, Cargo juga ikut ngambil crate lain yang
dibutuhin `rand` supaya bisa jalan. Setelah semua crate kelar di-download, Rust
akan nge-compile crate-crate itu, baru kemudian nge-compile project kamu dengan
dependency yang sudah siap dipakai.

Kalau kamu langsung menjalankan `cargo build` lagi tanpa ada perubahan apa pun,
kamu nggak bakal lihat output apa pun selain baris `Finished`. Cargo tahu kalau
dependency-nya sudah didownload dan dikompilasi, dan kamu juga nggak ngubah apa
pun di _Cargo.toml_. Cargo juga sadar kode kamu nggak berubah, jadi dia nggak
perlu recompile. Karena nggak ada kerjaan, dia langsung selesai.

Kalau kamu buka file _src/main.rs_, ubah sesuatu yang sepele aja, simpan, lalu
build lagi, kamu cuma bakal lihat dua baris output:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Baris-baris tadi nunjukin kalau Cargo cuma nge-update build berdasarkan
perubahan kecil yang kamu lakukan di file _src/main.rs_. Karena dependency kamu
nggak berubah, Cargo tahu dia bisa langsung pakai ulang hasil download dan
compile sebelumnya tanpa perlu ngulang dari awal.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### Memastikan Build Bisa Diulang dengan Hasil yang Sama

Cargo punya mekanisme supaya kamu bisa build proyek dengan hasil yang sama
setiap kali kamu atau orang lain nge-build kode kamu. Caranya: Cargo cuma bakal
pakai versi dependency yang sudah kamu tentuin, sampai kamu bilang mau ganti.

Misalnya, minggu depan keluar `rand` versi 0.8.6. Versi itu punya bug fix
penting, tapi ternyata ada regresi yang bikin kode kamu rusak. Untuk ngatasi
hal-hal kayak gitu, Rust bikin file _**Cargo.lock**_ saat pertama kali kamu
jalanin `cargo build`, jadi sekarang file itu sudah ada di folder
_guessing_game_.

Waktu kamu build project pertama kali, Cargo bakal nyari versi dependency yang
cocok sama aturan yang kamu tulis, lalu nyimpen hasilnya ke file _Cargo.lock_.
Build berikutnya, Cargo bakal lihat kalau _Cargo.lock_ sudah ada dan langsung
pakai versi yang tertulis di situ, tanpa repot-repot ngecek lagi. Ini bikin
build jadi **reproducible** alias hasilnya konsisten. Singkatnya: proyekmu bakal
tetap pakai versi 0.8.5 sampai kamu sengaja upgrade, berkat _Cargo.lock_. Karena
penting buat jaga konsistensi build, file ini biasanya ikut di-commit bareng
source code ke version control.

#### Update Crate ke Versi Baru

Kalau kamu **memang pengin update** crate, Cargo nyediain perintah `update`.
Perintah ini bakal ngabaikan _Cargo.lock_ dan nyari versi terbaru yang masih
cocok sama aturan di _Cargo.toml_. Setelah itu, Cargo bakal nulis versi barunya
ke _Cargo.lock_. Secara default, Cargo cuma bakal nyari versi yang lebih besar
dari 0.8.5 tapi masih di bawah 0.9.0.

Kalau misalnya crate `rand` udah keluar versi 0.8.6 dan 0.999.0, dan kamu
jalanin:

```
cargo update
```

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

Cargo mengabaikan rilis 0.999.0. Pada titik ini, kamu juga bakal lihat ada
perubahan di file _Cargo.lock_ yang nunjukin kalau sekarang versi `rand` yang
dipakai adalah 0.8.6. Kalau kamu mau pakai `rand` versi 0.999.0 atau versi lain
di seri 0.999._x_, kamu harus ngubah file _Cargo.toml_ jadi seperti ini (tapi
jangan beneran diubah, karena contoh-contoh selanjutnya diasumsikan masih pakai
`rand` 0.8):

```toml
[dependencies]
rand = "0.999.0"
```

Saat kamu menjalankan `cargo build` lagi, Cargo bakal memperbarui daftar crate
yang tersedia lalu mengevaluasi ulang kebutuhan `rand` kamu sesuai versi baru
yang sudah kamu tentukan.

Sebenernya masih banyak hal lain soal [Cargo][doccargo]<!-- ignore --> dan
[ekosistemnya][doccratesio]<!-- ignore --> yang bakal dibahas lebih dalam di
Bab 14. Tapi untuk sekarang, itu aja yang perlu kamu tahu dulu. Cargo bikin
reuse library jadi super gampang, sehingga para Rustacean bisa nulis project
yang lebih kecil dan nyusunnya dari banyak package yang sudah ada.

### Membuat Angka Acak

Sekarang kita mulai pakai `rand` buat ngehasilin angka yang bakal ditebak.
Langkah selanjutnya adalah update file _src/main.rs_, seperti yang ditunjukkan
di Listing 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Menambahkan kode untuk menghasilkan angka acak.">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Pertama, kita nambahin baris `use rand::Rng;`. Trait `Rng` ini berisi
method-method yang dipakai oleh random number generator, dan trait ini harus ada
di scope supaya kita bisa pakai method-nya. Detail soal trait bakal dibahas
lebih dalam di Chapter 10.

Selanjutnya, kita nambah dua baris di bagian tengah kode. Di baris pertama, kita
manggil fungsi `rand::thread_rng` untuk ngambil random number generator yang
bakal kita pakai. Generator ini terikat pada thread yang lagi jalan sekarang dan
“dibekali” (seeded) dari operating system.

Lalu, kita manggil method `gen_range` dari random number generator tadi. Method
ini didefinisikan oleh trait `Rng` yang tadi sudah kita masukin ke scope lewat
`use rand::Rng;`. Method `gen_range` menerima sebuah ekspresi range sebagai
argumen dan bakal ngasilin angka random di dalam range tersebut. Range yang kita
pakai bentuknya `start..=end`, yaitu range yang inklusif di batas bawah _dan_
batas atas. Jadi kita tulis `1..=100` supaya dapet angka acak antara 1
sampai 100.

> Catatan: Kamu nggak mungkin hafal begitu aja trait apa yang harus dipakai,
> method apa yang harus dipanggil, atau fungsi apa yang tersedia dari sebuah
> crate. Makanya setiap crate punya dokumentasi yang ngejelasin cara pakainya.
> Fitur keren Cargo lainnya: kalau kamu jalanin perintah `cargo doc --open`
> Cargo bakal bikin dokumentasi lokal untuk semua dependency kamu dan langsung
> buka di browser. Misalnya kamu pengin lihat fitur lain dari crate `rand`,
> tinggal jalankan `cargo doc --open` lalu klik `rand` di sidebar kiri.

Baris baru kedua cuma buat nge-print angka rahasianya. Ini berguna selama masa
pengembangan biar gampang ngetes, tapi nanti bakal kita hapus di versi final.
Soalnya nggak seru dong kalau game langsung kasih jawabannya

Sekarang coba jalankan programnya beberapa kali:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Kamu harusnya dapat angka acak yang berbeda-beda tiap kali jalanin program, dan
semuanya bakal berada di antara 1 sampai 100. Mantap! 🎉

## Membandingkan Tebakan dengan Angka Rahasia

Sekarang kita sudah punya input dari user dan angka acak. Berarti sekarang
saatnya kita bandingin keduanya. Langkah ini ditunjukkan di Listing 2-4.
Catatan: kode ini belum bisa dikompilasi dulu ya — nanti bakal dijelasin
kenapanya.

<Listing number="2-4" file-name="src/main.rs" caption="Menangani kemungkinan nilai hasil yang muncul saat membandingkan dua angka.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Pertama, kita nambah satu `use` lagi buat masukin tipe `std::cmp::Ordering` ke
dalam scope dari standard library. Tipe `Ordering` ini adalah enum lain yang
punya tiga kemungkinan nilai: `Less`, `Greater`, dan `Equal`. Ini adalah tiga
hasil yang mungkin terjadi saat kita membandingkan dua nilai.

Lalu, kita nambah lima baris kode baru di bagian bawah yang pakai `Ordering`.
Method `cmp` dipakai buat membandingkan dua nilai, dan bisa dipanggil di tipe
apa pun yang bisa dibandingkan. Method ini butuh referensi ke nilai yang mau
dibandingin: di sini, kita bandingin `guess` dengan `secret_number`. Setelah
itu, `cmp` bakal ngembalikan salah satu varian dari enum `Ordering` yang tadi
sudah kita masukin ke scope lewat `use`. Nah, kita pakai ekspresi
[`match`][match]<!-- ignore --> buat nentuin langkah selanjutnya berdasarkan
varian `Ordering` yang dikembalikan dari `cmp` saat bandingin `guess` dan
`secret_number`.

Ekspresi `match` tersusun dari beberapa _arm_. Satu arm terdiri dari _pattern_
yang mau dicocokkan, plus kode yang bakal dijalankan kalau nilai yang dikasih ke
`match` cocok sama pattern tersebut. Rust bakal ngambil nilai yang dikasih ke
`match`, lalu ngecek tiap pattern dari atas ke bawah. Pattern matching dan
`match` ini fitur yang kuat banget di Rust: mereka bikin kamu bisa menangani
banyak kemungkinan kondisi program dengan rapi, dan memastikan semuanya
tertangani. Fitur ini bakal dibahas lebih dalam di Chapter 6 dan Chapter 19
nanti.

Sekarang kita lihat contohnya pakai `match` yang ada di sini. Misalnya user
nebak angka 50, dan angka rahasianya ternyata 38.

Pas kode ngebandingin 50 dengan 38, method `cmp` bakal balikin
`Ordering::Greater` karena 50 lebih besar dari 38. Nilai `Ordering::Greater` ini
masuk ke `match`, lalu `match` mulai ngecek tiap arm. Pertama dia lihat pattern
`Ordering::Less`, tapi karena nggak cocok, arm itu dilewatin. Lanjut ke arm
berikutnya yang pattern-nya `Ordering::Greater`, dan ini cocok! Jadi kode di arm
itu dijalankan, dan program bakal nampilin `Too big!`. Setelah dapat kecocokan
pertama, `match` langsung selesai dan nggak ngecek arm terakhir.

Tapi… kode di Listing 2-4 ini belum bisa dikompilasi. Yuk kita coba jalanin
dulu:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Inti dari error-nya bilang kalau terjadi **mismatched types** alias tipe datanya
nggak cocok. Rust punya sistem tipe yang kuat dan statis, tapi juga punya fitur
_type inference_. Waktu kita nulis `let mut guess = String::new()`, Rust bisa
nebak sendiri kalau `guess` itu bertipe `String`, jadi kita nggak perlu nulis
tipenya secara eksplisit.

Di sisi lain, `secret_number` adalah tipe angka. Beberapa tipe angka di Rust
bisa punya nilai antara 1 sampai 100, misalnya `i32` (32-bit signed), `u32`
(32-bit unsigned), `i64` (64-bit signed), dan lainnya. Kalau nggak ditentukan
lain, Rust bakal default ke `i32`, jadi `secret_number` dianggap bertipe `i32`
kecuali ada informasi lain yang bikin Rust menebak tipe berbeda.

Masalahnya adalah: Rust **nggak bisa membandingkan String dengan angka**. Itu
yang bikin error.

Pada akhirnya, kita perlu mengubah `String` yang dibaca dari input user menjadi
tipe angka supaya bisa dibandingkan secara numerik dengan angka rahasia. Kita
lakukan itu dengan menambahkan baris ini ke dalam body fungsi `main`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

The line is:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Kita bikin variabel baru bernama `guess`. Eh, tapi kan kita udah punya variabel
`guess` sebelumnya? Betul. Tapi enaknya Rust, kita boleh “menimpa” nilai lama
`guess` dengan nilai baru. Fitur ini disebut **shadowing**. Dengan shadowing,
kita bisa pakai lagi nama variabel `guess` tanpa harus bikin dua nama berbeda,
misalnya `guess_str` dan `guess`. Topik ini bakal dibahas lebih detail di
[Chapter 3][shadowing]<!-- ignore -->, tapi untuk sekarang cukup tahu kalau
fitur ini sering dipakai saat kita ingin mengubah suatu nilai dari satu tipe ke
tipe lain.

Variabel baru ini kita isi dengan ekspresi `guess.trim().parse()`. `guess` yang
ada di ekspresi ini masih merujuk ke `guess` yang lama, yang isinya input user
dalam bentuk string. Method `trim` pada `String` bakal ngapus whitespace di awal
dan akhir teks, dan ini wajib dilakukan sebelum kita ubah string tersebut ke
`u32`, karena `u32` cuma boleh berisi angka doang.

User harus tekan <kbd>enter</kbd> buat ngirim input ke `read_line`, dan itu
bakal nambahin karakter newline ke string. Misalnya user ngetik <kbd>5</kbd>
lalu tekan <kbd>enter</kbd>, isi `guess` bakal jadi kayak gini: `5\n`. Tanda
`\n` itu newline. (Di Windows malah jadi `\r\n`.) Nah, `trim` bakal ngilangin
`\n` atau `\r\n`, jadi tinggal `5`.

Method [`parse` pada string][parse]<!-- ignore --> dipakai buat ngubah string
jadi tipe lain. Di sini kita pakai buat ngubah string jadi angka. Kita harus
ngasih tahu Rust angka tipe apa yang kita mau, makanya kita tulis
`let guess: u32`. Titik dua (`:`) setelah `guess` artinya kita lagi nentuin tipe
variabelnya. `u32` sendiri adalah unsigned 32-bit integer, pilihan yang pas buat
angka kecil dan positif. Tipe angka lain bakal dibahas di
[Chapter 3][integers]<!-- ignore -->.

Selain itu, karena di contoh ini kita pakai `u32` untuk `guess` dan kita
bandingin sama `secret_number`, Rust jadi nge-infer kalau `secret_number` juga
harus `u32`. Jadi sekarang kedua nilai punya tipe yang sama — mantap, bisa
dibandingkan!

Method `parse` cuma bakal berhasil kalau stringnya bener-bener bisa diubah ke
angka. Jadi gampang banget terjadi error kalau isinya aneh, misalnya `A👍%`,
jelas nggak bisa jadi angka. Karena bisa gagal, `parse` juga ngembalikan
`Result`, sama kayak `read_line` yang udah kita bahas di bagian
[“Handling Potential Failure with `Result`”](#handling-potential-failure-with-result)<!-- ignore -->.

Kita tangani `Result` ini dengan cara yang sama: pakai `expect`. Kalau `parse`
gagal dan balikin `Err`, `expect` bakal bikin game crash dan nampilin pesan yang
kita kasih. Tapi kalau `parse` berhasil dan balikin `Ok`, `expect` bakal ngasih
kita angkanya.

Sekarang, ayo jalanin programnya lagi:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Keren! 🎉 Meskipun ada spasi di depan tebakan, program tetap bisa ngerti kalau
user nebak angka 76. Coba jalanin programnya beberapa kali untuk ngetes berbagai
kondisi: tebak angka yang tepat, tebak angka yang terlalu besar, dan tebak angka
yang terlalu kecil.

Sekarang sebagian besar game-nya sudah jalan, tapi user baru bisa nebak **sekali
doang**. Kita perlu ubah itu dengan nambahin loop!

## Mengizinkan Banyak Tebakan dengan Loop

Keyword `loop` bakal bikin loop tak terbatas (infinite loop). Kita bakal
nambahin loop supaya user bisa punya banyak kesempatan buat nebak angkanya:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Seperti yang kamu lihat, sekarang kita mindahin semua bagian mulai dari prompt
input tebakan ke dalam sebuah loop. Pastikan baris-baris di dalam loop
di-_indent_ empat spasi lagi, lalu jalankan programnya. Sekarang program bakal
terus minta tebakan baru tanpa henti… tapi itu bikin masalah baru: kelihatannya
user jadi nggak bisa keluar dari game!

Sebenernya user masih bisa keluar pakai shortcut keyboard <kbd>Ctrl</kbd> +
<kbd>C</kbd>. Tapi ada cara lain buat kabur dari “monster tak pernah puas” ini,
seperti yang udah disinggung waktu bahas `parse` di bagian
[“Comparing the Guess to the Secret Number”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->.

Kalau user masukin input yang **bukan angka**, program bakal crash. Nah, kita
bisa manfaatin hal ini buat bikin cara keluar dari game, seperti di contoh
berikut:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Mengetik `quit` bakal langsung keluar dari game, tapi seperti yang kamu lihat,
input apa pun yang **bukan angka** juga bikin game berhenti. Jelas ini kurang
ideal, kita maunya game juga berhenti kalau user berhasil nebak angka yang
benar.

### Keluar Setelah Tebakan Benar

Sekarang kita bikin game berhenti saat user menang dengan nambahin statement
`break`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Menambahkan baris `break` setelah `You win!` bikin program keluar dari loop saat
user berhasil nebak angka dengan benar. Karena loop itu bagian terakhir dari
`main`, keluar dari loop berarti programnya juga langsung selesai.

### Menangani Input yang Tidak Valid

Biar game-nya makin rapi, daripada langsung crash kalau user masukin input yang
bukan angka, lebih bagus kalau game cuma **ngabaikan** input tersebut dan lanjut
main seperti biasa. Jadi user masih bisa terus nebak.

Kita bisa lakukan itu dengan mengubah baris yang mengonversi `guess` dari
`String` ke `u32`, seperti yang ditunjukkan di Listing 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Mengabaikan tebakan yang bukan angka dan minta tebakan baru lagi, bukannya bikin program crash.">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Kita mengganti pemanggilan `expect` dengan ekspresi `match` supaya bukan lagi
**crash saat error**, tapi **menangani error dengan benar**. Ingat, `parse`
mengembalikan tipe `Result`, dan `Result` adalah enum yang punya dua varian:
`Ok` dan `Err`. Di sini kita pakai `match`, sama seperti saat kita menangani
hasil `Ordering` dari method `cmp`.

Kalau `parse` berhasil mengubah string menjadi angka, dia akan mengembalikan
`Ok` yang berisi angka hasil konversi tersebut. Nilai `Ok` itu bakal cocok
dengan pattern di arm pertama, dan `match` akan mengembalikan nilai `num` di
dalamnya. Angka itulah yang kemudian disimpan ke variabel `guess` baru yang kita
buat.

Kalau `parse` **tidak** bisa mengubah string menjadi angka, dia akan
mengembalikan `Err` yang berisi informasi tentang errornya. Nilai `Err` ini
tidak cocok dengan pattern `Ok(num)` di arm pertama, tapi cocok dengan pattern
`Err(_)` di arm kedua. Tanda underscore `_` adalah penangkap umum (catch-all);
di sini artinya kita ingin menangkap semua `Err`, apa pun isi detail errornya.
Jadi program akan menjalankan kode di arm kedua, yaitu `continue`, yang menyuruh
program lanjut ke iterasi berikutnya dari `loop` dan meminta tebakan baru lagi.

Artinya, program sekarang **mengabaikan semua error yang mungkin terjadi saat
`parse`**, dan tetap berjalan dengan mulus.

Sekarang seharusnya seluruh program sudah bekerja sesuai harapan. Yuk kita coba
menjalankannya!

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Keren! Dengan satu sentuhan kecil terakhir, kita bakal selesai bikin game tebak
angka ini. Ingat, program kita masih nge-print angka rahasianya. Itu memang
berguna buat testing tadi, tapi jelas bikin gamenya nggak seru. Jadi sekarang
hapus aja `println!` yang nampilin angka rahasia itu. Listing 2-6 nunjukin kode
finalnya.

<Listing number="2-6" file-name="src/main.rs" caption="Kode lengkap game tebak angka">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Pada tahap ini, kamu sudah berhasil menyelesaikan pembuatan game tebak angka.
Selamat!

## Ringkasan

Project ini adalah cara belajar _hands-on_ buat ngenalin kamu ke banyak konsep
baru di Rust: `let`, `match`, fungsi, penggunaan external crate, dan lain-lain.
Di bab-bab selanjutnya, kamu bakal pelajari konsep-konsep ini lebih dalam lagi.
Chapter 3 bakal bahas konsep umum yang biasanya ada di hampir semua bahasa
pemrograman, seperti variabel, tipe data, dan fungsi, serta gimana cara pakainya
di Rust. Chapter 4 bakal ngebahas ownership, fitur khas Rust yang bikin dia beda
dari bahasa lain. Chapter 5 ngebahas `struct` dan sintaks method. Chapter 6
bakal ngejelasin cara kerja `enum`.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
