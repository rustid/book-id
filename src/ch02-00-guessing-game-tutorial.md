# Membuat Game Tebak Angka

Yuk, kita langsung terjun ke Rust dengan ngerjain project bareng-bareng! Di bab 
ini, kita bakal kenalan sama beberapa konsep umum di Rust lewat program benar-benar. 
Kita bakal belajar soal `let`, `match`, methods, associated functions, external 
crates, dan banyak lagi! Di bab-bab selanjutnya, kita bakal bahas konsep ini 
lebih dalem. Tapi buat sekarang, kita latihan yang basic-basic dulu ya.

Kita bakal bikin problem klasik buat pemula: game tebak angka. Cara mainnya 
simpel: program bakal nge-generate angka random antara 1 sampe 100. Terus, 
program bakal minta player buat masukin tebakannya. Setelah tebakan dimasukin, 
program bakal ngasih tau apakah tebakannya ketinggian atau kerendahan. Kalau 
bener, game-nya bakal ngasih ucapan selamat terus exit.

## Setup Project Baru

Buat nge-setup project baru, masuk ke direktori _projects_ yang udah kita bikin 
di Bab 1, terus bikin project baru pake Cargo kayak gini:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Perintah pertama, `cargo new`, ngambil nama project (`guessing_game`) sebagai 
argumen pertamanya. Perintah kedua buat pindah ke direktori project yang baru 
dibikin.

Coba liat file _Cargo.toml_ yang dihasilkan:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Nama file: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Kayak yang kita liat di Bab 1, `cargo new` nge-generate program “Hello, world!” 
buat kita. Cek file _src/main.rs_-nya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Sekarang kita compile program “Hello, world!” ini terus jalanin sekalian pake 
perintah `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Perintah `run` ini kepake sekali pas kita butuh iterasi cepet di sebuah project, 
kayak di game ini nanti, buat ngetes tiap perubahan sebelum lanjut ke langkah 
berikutnya.

Buka lagi file _src/main.rs_. Kita bakal nulis semua kodenya di file ini.

## Memproses Tebakan (Guess)

Bagian pertama dari program game tebak angka ini bakal minta input dari user, 
proses input-nya, terus nge-cek apakah input-nya udah sesuai format. Buat awal, 
kita bakal bolehin player buat masukin tebakan. Masukin kode di Listing 2-1 ke 
dalam _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Kode buat ngambil tebakan dari user dan mencetaknya">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Kode ini isinya banyak informasi, jadi yuk kita bahas baris demi baris. Buat 
dapet input user terus nge-print hasilnya sebagai output, kita perlu bawa 
library `io` (input/output) ke dalam scope. Library `io` ini dateng dari 
standard library, yang dikenal sebagai `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Secara default, Rust punya sekumpulan item yang udah didefinisiin di standard 
library yang otomatis dibawa ke scope tiap program. Kumpulan ini namanya 
_prelude_, dan kita bisa liat isinya [di dokumentasi standard library][prelude].

Kalau tipe yang mau kita pake nggak ada di prelude, kita harus bawa tipe itu ke 
scope secara eksplisit pake statement `use`. Pake library `std::io` ngasih kita 
beberapa fitur berguna, termasuk kemampuan buat nerima input user.

Kayak yang kita liat di Bab 1, fungsi `main` adalah entry point ke programnya:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Sintaks `fn` mendeklarasikan fungsi baru; tanda kurung `()` nunjukin kalau 
nggak ada parameter; dan kurung kurawal `{` buat mulai body fungsinya.

Kita juga udah belajar di Bab 1 kalau `println!` itu macro buat nyetak string ke 
layar:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Kode ini nyetak prompt yang ngasih tau gamenya apa dan minta input dari user.

### Menyimpan Nilai dengan Variabel

Selanjutnya, kita bakal bikin sebuah _variable_ buat nyimpen input user, kayak 
gini:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Nah, programnya mulai seru nih! Ada banyak hal yang terjadi di baris kecil ini. 
Kita pake statement `let` buat bikin variabel. Ini contoh lainnya:

```rust,ignore
let apel = 5;
```

Baris ini bikin variabel baru namanya `apel` dan nge-bind ke nilai 5. Di Rust, 
variabel itu _immutable_ (nggak bisa diubah) secara default, artinya sekali kita 
kasih nilai ke variabelnya, nilainya nggak bakal berubah. Kita bakal bahas 
konsep ini lebih detail di bagian [“Variabel dan Mutabilitas”][variables-and-mutability] 
di Bab 3. Buat bikin variabel jadi _mutable_ (bisa diubah), kita tambahin `mut` 
sebelum nama variabelnya:

```rust,ignore
let apel = 5; // immutable
let mut pisang = 5; // mutable
```

> Catatan: Sintaks `//` buat mulai komentar sampe akhir baris. Rust bakal cuekin 
> apa pun yang ada di dalam komentar. Kita bakal bahas komentar lebih detail di 
> [Bab 3][comments].

Balik lagi ke program game tebak angka, sekarang kita tau kalau `let mut guess` 
bakal ngenalin variabel mutable namanya `guess`. Tanda sama dengan (`=`) ngasih 
tau Rust kalau kita mau nge-bind sesuatu ke variabelnya sekarang. Di sebelah 
kanan tanda sama dengan adalah nilai yang di-bind ke `guess`, yaitu hasil dari 
manggil `String::new`, fungsi yang balikin instance baru dari sebuah `String`. 
[`String`][string] adalah tipe string yang dikasih standard library yang isinya 
teks UTF-8 yang bisa nambah terus ukurannya (growable).

Sintaks `::` di baris `::new` nunjukin kalau `new` itu _associated function_ dari 
tipe `String`. _Associated function_ adalah fungsi yang diimplementasikan pada 
sebuah tipe, dalam hal ini `String`. Fungsi `new` ini bikin string baru yang 
kosong. Kita bakal sering nemu fungsi `new` di banyak tipe karena itu nama umum 
buat fungsi yang bikin nilai baru dari jenis tertentu.

Secara lengkap, baris `let mut guess = String::new();` udah bikin variabel 
mutable yang sekarang di-bind ke instance baru dari `String` yang masih kosong. 
Fiuh!

### Menerima Input User

Inget kan kalau kita udah masukin fungsi input/output dari standard library pake 
`use std::io;` di baris pertama program. Sekarang kita bakal manggil fungsi 
`stdin` dari modul `io`, yang bakal ngebolehin kita handle input user:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Kalau kita nggak import modul `io` pake `use std::io;` di awal program, kita 
tetep bisa pake fungsinya dengan nulis pemanggilan fungsi ini jadi 
`std::io::stdin`. Fungsi `stdin` balikin instance dari [`std::io::Stdin`][iostdin], 
tipe yang merepresentasikan handle ke standard input buat terminal kita.

Selanjutnya, baris `.read_line(&mut guess)` manggil method [`read_line`][read_line] 
pada handle standard input buat dapet input dari user. Kita juga masukin 
`&mut guess` sebagai argumen ke `read_line` buat ngasih tau string mana yang 
bakal dipake buat nyimpen input user. Tugas utama `read_line` adalah ngambil 
apa pun yang diketik user ke standard input terus nambahin itu ke sebuah string 
(tanpa nimpa isinya), makanya kita masukin string itu sebagai argumen. Argumen 
string-nya harus mutable biar method-nya bisa ngerubah isi string-nya.

Tanda `&` nunjukin kalau argumen ini adalah sebuah _reference_ (referensi), 
yang ngasih cara biar beberapa bagian kode kita bisa akses satu data tanpa perlu 
copy data itu ke memori berkali-kali. Reference itu fitur yang lumayan kompleks, 
dan salah satu keunggulan utama Rust adalah seberapa aman dan gampangnya pake 
reference. Kita nggak perlu tau banyak detailnya buat nyelesein program ini. 
Buat sekarang, yang perlu kita tau adalah, kayak variabel, reference itu 
immutable secara default. Makanya, kita perlu nulis `&mut guess` bukannya 
`&guess` biar dia mutable. (Bab 4 bakal jelasin soal reference lebih mendalam.)

<!-- Old heading. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Menangani Potensi Gagal dengan `Result`

Kita masih bahas baris kode yang tadi ya. Sekarang kita lagi bahas bagian 
ketiganya, tapi inget kalau ini masih bagian dari satu baris kode yang logis. 
Bagian selanjutnya adalah method ini:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Kita bisa aja nulis kode ini kayak gini:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Gagal baca baris");
```

Tapi, satu baris panjang itu susah dibaca, jadi mendingan dibagi-bagi. Biasanya 
lebih bijak buat nambahin baris baru dan whitespace lainnya buat bantu mecah 
baris panjang pas kita manggil method pake sintaks `.nama_method()`. Sekarang 
mari kita bahas fungsi baris ini.

Kayak yang udah disebutin tadi, `read_line` naruh apa pun yang user masukin ke 
string yang kita kasih, tapi dia juga balikin nilai `Result`. [`Result`][result] 
adalah sebuah [_enumeration_][enums], yang sering disebut _enum_, yaitu tipe 
yang bisa ada di salah satu dari beberapa kemungkinan state. Kita sebut tiap 
state itu sebagai _variant_.

[Bab 6][enums] bakal bahas enum lebih detail. Tujuan dari tipe-tipe `Result` ini 
adalah buat nge-encode informasi penanganan error (error-handling).

Varian dari `Result` adalah `Ok` dan `Err`. Varian `Ok` nunjukin operasinya 
berhasil, dan di dalemnya ada nilai yang berhasil di-generate. Varian `Err` 
artinya operasinya gagal, dan isinya informasi soal gimana atau kenapa 
operasinya gagal.

Nilai dari tipe `Result`, kayak nilai dari tipe apa pun, punya methods yang 
didefinisiin di atasnya. Instance dari `Result` punya [`expect` method][expect] 
yang bisa kita panggil. Kalau instance `Result` ini adalah nilai `Err`, `expect` 
bakal bikin programnya _crash_ dan nampilin pesan yang kita kasih sebagai 
argumen ke `expect`. Kalau method `read_line` balikin `Err`, itu kemungkinan 
hasil dari error yang dateng dari sistem operasi di bawahnya. Kalau instance 
`Result` ini adalah nilai `Ok`, `expect` bakal ngambil nilai balik yang dipegang 
sama `Ok` terus balikin nilai itu aja biar bisa kita pake. Dalam kasus ini, 
nilai itu adalah jumlah byte dari input user.

Kalau kita nggak manggil `expect`, programnya tetep bakal ke-compile, tapi kita 
bakal dapet warning:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust ngasih tau kalau kita nggak pake nilai `Result` yang dibalikin sama 
`read_line`, yang nunjukin kalau programnya belum handle kemungkinan error.

Cara yang bener buat ilangin warning itu adalah dengan benar-benar nulis kode 
error-handling, tapi dalam kasus kita, kita cuma mau nge-_crash_-in program ini 
pas ada masalah, jadi kita bisa pake `expect`. Kita bakal belajar soal cara 
bangkit dari error di [Bab 9][recover].

### Mencetak Nilai dengan Placeholder `println!`

Selain kurung kurawal tutup, cuma ada satu baris lagi yang perlu dibahas di kode 
sejauh ini:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Baris ini nyetak string yang sekarang isinya input user. Tanda kurung kurawal 
`{}` itu adalah placeholder: bayangin aja `{}` kayak capit kepiting kecil yang 
megang sebuah nilai di tempatnya. Pas nyetak nilai variabel, nama variabelnya 
bisa masuk ke dalem kurung kurawal. Pas nyetak hasil evaluasi sebuah ekspresi, 
taruh kurung kurawal kosong di format string-nya, terus ikuti format string itu 
dengan daftar ekspresi yang dipisahin koma buat dicetak di tiap placeholder 
kurung kurawal kosong sesuai urutannya. Nyetak variabel dan hasil ekspresi dalam 
satu panggilan ke `println!` bakal keliatan kayak gini:

```rust
let x = 5;
let y = 10;

println!("x = {x} dan y + 2 = {}", y + 2);
```

Kode ini bakal nyetak `x = 5 dan y + 2 = 12`.

### Ngetes Bagian Pertama

Yuk kita tes bagian pertama dari game tebak angka ini. Jalanin pake `cargo run`:

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

Sampai titik ini, bagian pertama game-nya udah jadi: kita dapet input dari 
keyboard terus nyetak hasilnya.

## Menghasilkan Secret Number

Selanjutnya, kita perlu generate angka rahasia (secret number) yang bakal ditebak 
sama user. Angka rahasianya harus beda terus tiap kali biar gamenya seru buat 
dimainin berkali-kali. Kita bakal pake angka random antara 1 sampe 100 biar 
gamenya nggak terlalu susah. Rust belum masukin fungsionalitas angka random di 
standard library-nya. Tapi, tim Rust nyediain [`rand` crate][randcrate] dengan 
fungsionalitas tersebut.

### Pake Crate buat Dapet Fitur Lebih

Inget ya, crate itu adalah kumpulan file source code Rust. Project yang lagi 
kita bangun ini adalah sebuah _binary crate_, yaitu sebuah executable. `rand` 
crate adalah sebuah _library crate_, yang isinya kode yang tujuannya buat dipake 
di program lain dan nggak bisa dijalankan sendiri.

Koordinasi Cargo sama external crates adalah bagian di mana Cargo bener-bener 
bersinar. Sebelum kita bisa nulis kode yang pake `rand`, kita perlu modifikasi 
file _Cargo.toml_ buat masukin `rand` crate sebagai dependensi. Buka filenya 
sekarang terus tambahin baris berikut di paling bawah, di bawah header section 
`[dependencies]` yang udah dibikinin Cargo. Pastiin buat nulis `rand` persis 
kayak di sini, dengan nomor versi ini, biar contoh kode di tutorial ini bisa 
jalan:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Nama file: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Di file _Cargo.toml_, apa pun yang ngikutin sebuah header adalah bagian dari 
section itu sampe section lain mulai. Di `[dependencies]` kita ngasih tau Cargo 
external crates mana aja yang diperluin project kita dan versi mana yang kita 
butuhin. Dalam hal ini, kita nentuin `rand` crate dengan penentu versi semantik 
`0.8.5`. Cargo ngerti [Semantic Versioning][semver] (kadang disebut _SemVer_), 
yaitu standar buat nulis nomor versi. Penentu `0.8.5` sebenernya singkatan dari 
`^0.8.5`, yang artinya versi apa pun yang minimal 0.8.5 tapi di bawah 0.9.0.

Cargo nganggep versi-versi ini punya public API yang kompatibel sama versi 0.8.5, 
dan spesifikasi ini mastiin kita dapet patch release terbaru yang tetep bisa 
di-compile sama kode di bab ini. Versi 0.9.0 atau yang lebih baru nggak dijamin 
punya API yang sama kayak contoh-contoh yang kita pake.

Sekarang, tanpa ngerubah kode apa pun, yuk kita build project-nya, kayak yang 
ditunjukin di Listing 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="Output dari jalanin `cargo build` setelah nambahin rand crate sebagai dependensi">

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

Mungkin kita bakal liat nomor versi yang beda (tapi semuanya bakal kompatibel 
sama kodenya, makasih buat SemVer!) dan baris-baris yang beda (tergantung sistem 
operasi), dan urutannya mungkin beda juga.

Pas kita masukin external dependency, Cargo bakal ngambil versi terbaru dari 
segala hal yang dibutuhin dependency itu dari _registry_, yang merupakan copy 
data dari [Crates.io][cratesio]. Crates.io adalah tempat orang-orang di 
ekosistem Rust posting project open source Rust mereka biar bisa dipake orang lain.

Setelah update registry, Cargo cek section `[dependencies]` terus download 
crate apa pun yang terdaftar tapi belum ada di komputer kita. Dalam hal ini, 
walaupun kita cuma daftarin `rand` sebagai dependensi, Cargo juga ngambil crates 
lain yang dibutuhin `rand` biar bisa jalan. Setelah download crates-nya, Rust 
bakal compile crates itu terus compile project kita dengan dependensi yang udah 
siap.

Kalau kita langsung jalanin `cargo build` lagi tanpa ngerubah apa pun, kita 
nggak bakal dapet output apa-apa selain baris `Finished`. Cargo tau kalau dia 
udah download dan compile dependensi-nya, dan kita nggak ngerubah apa pun soal 
itu di file _Cargo.toml_. Cargo juga tau kalau kita nggak ngerubah kode kita, 
jadi dia nggak compile ulang juga. Karena nggak ada kerjaan, dia langsung exit.

Kalau kita buka file _src/main.rs_, bikin perubahan kecil, terus save dan build 
lagi, kita cuma bakal liat dua baris output:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Baris-baris ini nunjukin kalau Cargo cuma update build-nya sama perubahan kecil 
kita di file _src/main.rs_. Dependensi kita nggak berubah, jadi Cargo tau dia 
bisa pake lagi apa yang udah dia download dan compile sebelumnya.

#### Mastiin Build Bisa Direproduksi (Reproducible) dengan File _Cargo.lock_

Cargo punya mekanisme yang mastiin kita bisa rebuild artifact yang sama tiap 
kali kita atau orang lain build kode kita: Cargo cuma bakal pake versi 
dependensi yang kita tentuin sampe kita bilang sebaliknya. Misalnya, katakanlah 
minggu depan versi 0.8.6 dari `rand` crate keluar, dan versi itu isinya bug fix 
penting, tapi ada juga regresi yang bikin kode kita rusak. Buat handle ini, 
Rust bikin file _Cargo.lock_ pas pertama kali kita jalanin `cargo build`, jadi 
sekarang kita punya file ini di direktori _guessing_game_.

Pas kita build project buat pertama kali, Cargo cari tau semua versi dependensi 
yang cocok sama kriterianya terus nulis versi itu ke file _Cargo.lock_. Pas kita 
build project-nya nanti, Cargo bakal liat kalau file _Cargo.lock_ ada terus 
bakal pake versi yang ditentuin di situ bukannya cari-cari versi lagi. Ini bikin 
kita punya reproducible build secara otomatis. Dengan kata lain, project kita 
bakal tetep di versi 0.8.5 sampe kita eksplisit buat upgrade, berkat file 
_Cargo.lock_. Karena file _Cargo.lock_ itu penting buat reproducible builds, 
filenya sering kali dimasukan ke source control barengan sama kode project lainnya.

#### Update Crate buat Dapet Versi Baru

Pas kita _emang_ mau update sebuah crate, Cargo nyediain perintah `update`, 
yang bakal cuekin file _Cargo.lock_ terus cari tau semua versi terbaru yang 
cocok sama spesifikasi kita di _Cargo.toml_. Cargo terus bakal nulis versi itu 
ke file _Cargo.lock_. Dalam hal ini, Cargo cuma bakal nyari versi yang lebih 
gede dari 0.8.5 dan kurang dari 0.9.0. Kalau `rand` crate udah ngerilis dua 
versi baru 0.8.6 dan 0.9.0, kita bakal liat output kayak gini pas jalanin 
`cargo update`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.9.0)
```

Cargo cuekin rilis 0.9.0. Di titik ini, kita juga bakal nyadar ada perubahan 
di file _Cargo.lock_ yang nyatet kalau versi `rand` crate yang sekarang kita 
pake itu 0.8.6. Buat pake `rand` versi 0.9.0 atau versi mana pun di seri 
0.9._x_, kita harus update file _Cargo.toml_-nya jadi kayak gini:

```toml
[dependencies]
rand = "0.9.0"
```

Lain kali kita jalanin `cargo build`, Cargo bakal update registry crate yang 
tersedia terus evaluasi ulang kebutuhan `rand` kita sesuai versi baru yang 
kita tentuin.

Masih banyak lagi hal soal [Cargo][doccargo] dan [ekosistemnya][doccratesio] 
yang bakal kita bahas di Bab 14, tapi buat sekarang, itu aja yang perlu kita 
tau. Cargo bikin kita gampang sekali buat reuse library, jadi para Rustacean 
bisa nulis project kecil yang disusun dari banyak packages.

### Menghasilkan Angka Random

Yuk mulai pake `rand` buat nge-generate angka buat ditebak. Langkah selanjutnya 
adalah update _src/main.rs_, kayak yang ditunjukin di Listing 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Nambahin kode buat nge-generate angka random">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Pertama kita tambahin baris `use rand::Rng;`. Trait `Rng` nentuin methods yang 
diimplementasiin sama random number generators, dan trait ini harus ada di 
scope biar kita bisa pake methods itu. Bab 10 bakal bahas traits lebih detail.

Selanjutnya, kita nambahin dua baris di tengah. Di baris pertama, kita manggil 
fungsi `rand::thread_rng` yang ngasih kita random number generator tertentu 
yang bakal kita pake: yang lokal buat thread eksekusi saat ini dan dikasih 
seed sama sistem operasi. Terus kita manggil method `gen_range` pada random 
number generator-nya. Method ini didefinisiin sama trait `Rng` yang kita bawa 
ke scope lewat statement `use rand::Rng;`. Method `gen_range` ngambil ekspresi 
range sebagai argumen terus nge-generate angka random di dalem range itu. Jenis 
ekspresi range yang kita pake di sini bentuknya `start..=end` dan inklusif 
(termasuk) batas bawah sama batas atasnya, jadi kita perlu nulis `1..=100` 
buat minta angka antara 1 sampe 100.

> Catatan: Kita nggak bakal tau gitu aja trait mana yang harus dipake sama 
> method dan fungsi mana yang harus dipanggil dari sebuah crate, makanya tiap 
> crate punya dokumentasi yang isinya instruksi buat pake crate itu. Fitur keren 
> lainnya dari Cargo adalah jalanin perintah `cargo doc --open` bakal build 
> dokumentasi yang disediain sama semua dependensi kita secara lokal terus buka 
> di browser. Kalau kita penasaran sama fitur lainnya di `rand` crate, misalnya, 
> jalanin `cargo doc --open` terus klik `rand` di sidebar sebelah kiri.

Baris baru yang kedua nyetak secret number-nya. Ini berguna pas kita lagi 
ngembangin programnya biar bisa dites, tapi nanti kita hapus di versi final. 
Nggak seru dong gamenya kalau programnya langsung ngasih tau jawabannya pas baru 
mulai!

Coba jalanin programnya beberapa kali:

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

Kita harusnya dapet angka random yang beda-beda, dan semuanya harusnya angka 
antara 1 sampe 100. Mantap!

## Membandingkan Tebakan dengan Secret Number

Sekarang setelah kita dapet input user sama angka random, kita bisa bandingin 
keduanya. Langkah itu ditunjukin di Listing 2-4. Inget ya kalau kode ini belum 
bisa di-compile sekarang, nanti kita jelasin kenapa.

<Listing number="2-4" file-name="src/main.rs" caption="Menangani kemungkinan nilai balik dari membandingkan dua angka">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Pertama kita tambahin statement `use` lagi, bawa tipe namanya 
`std::cmp::Ordering` ke scope dari standard library. Tipe `Ordering` ini adalah 
enum lainnya dan punya varian `Less`, `Greater`, dan `Equal`. Ini adalah tiga 
hasil yang mungkin terjadi pas kita bandingin dua nilai.

Terus kita tambahin lima baris baru di bawah yang pake tipe `Ordering`. Method 
`cmp` bandingin dua nilai dan bisa dipanggil pada apa pun yang bisa dibandingin. 
Dia ngambil reference ke apa pun yang mau kita bandingin: di sini dia bandingin 
`guess` sama `secret_number`. Terus dia balikin varian dari enum `Ordering` yang 
kita bawa ke scope tadi. Kita pake ekspresi [`match`][match] buat nentuin apa 
yang harus dilakuin selanjutnya berdasarkan varian `Ordering` mana yang 
dibalikin dari pemanggilan `cmp` sama nilai di `guess` dan `secret_number`.

Ekspresi `match` itu disusun dari _arms_. Sebuah arm isinya sebuah _pattern_ 
(pola) buat dicocokin, sama kode yang harus jalan kalau nilai yang dikasih ke 
`match` cocok sama pattern di arm itu. Rust ngambil nilai yang dikasih ke 
`match` terus liat tiap pattern di tiap arm secara bergantian. Pattern sama 
konstruk `match` itu fitur Rust yang sangat kuat: mereka ngebolehin kita 
mengekspresikan berbagai situasi yang mungkin dihadapi kode kita dan mastiin 
kita handle semuanya. Fitur-fitur ini bakal dibahas detail di Bab 6 dan Bab 19.

Yuk kita telusuri contoh ekspresi `match` yang kita pake di sini. Katakanlah user 
nebak 50 dan secret number yang di-generate random kali ini adalah 38.

Pas kodenya bandingin 50 sama 38, method `cmp` bakal balikin 
`Ordering::Greater` karena 50 lebih besar dari 38. Ekspresi `match` dapet nilai 
`Ordering::Greater` terus mulai cek tiap pattern di tiap arm. Dia liat pattern 
di arm pertama, `Ordering::Less`, dan liat kalau nilai `Ordering::Greater` 
nggak cocok sama `Ordering::Less`, jadi dia cuekin kode di arm itu terus lanjut 
ke arm berikutnya. Pattern arm berikutnya itu `Ordering::Greater`, yang _emang_ 
cocok sama `Ordering::Greater`! Kode yang terkait di arm itu bakal jalan terus 
nyetak `Too big!` ke layar. Ekspresi `match` berakhir setelah match pertama yang 
berhasil, jadi dia nggak bakal liat arm terakhir di skenario ini.

Tapi, kode di Listing 2-4 belum bisa di-compile. Yuk kita coba:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Inti dari error-nya bilang kalau ada _mismatched types_ (tipe nggak cocok). Rust 
punya sistem tipe yang kuat dan statis. Tapi, dia juga punya _type inference_ 
(inferensi tipe). Pas kita nulis `let mut guess = String::new()`, Rust bisa tau 
kalau `guess` harusnya sebuah `String` dan nggak maksa kita buat nulis tipenya. 
Di sisi lain, `secret_number` itu tipe angka. Beberapa tipe angka di Rust bisa 
punya nilai antara 1 sampe 100: `i32`, angka 32-bit; `u32`, angka 32-bit tanpa 
tanda (unsigned); `i64`, angka 64-bit; dan lainnya. Kecuali ditentuin lain, 
Rust default-nya ke `i32`, yang merupakan tipe dari `secret_number` kecuali kita 
nambahin informasi tipe di tempat lain yang bakal bikin Rust tau tipe angka yang 
beda. Alasan kenapa ada error itu karena Rust nggak bisa bandingin tipe string 
sama tipe angka.

Akhirnya, kita mau convert `String` yang dibaca program sebagai input jadi tipe 
angka biar kita bisa bandingin secara numerik sama secret number-nya. Kita 
lakuin itu dengan nambahin baris ini ke body fungsi `main`:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Barisnya adalah:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Kita bikin variabel namanya `guess`. Tapi tunggu, bukannya programnya udah 
punya variabel namanya `guess`? Emang punya, tapi untungnya Rust ngebolehin kita 
buat _shadow_ nilai `guess` yang lama sama yang baru. _Shadowing_ ngebolehin kita 
pake lagi nama variabel `guess` bukannya maksa kita bikin dua variabel unik, 
kayak `guess_str` dan `guess`, misalnya. Kita bakal bahas ini lebih detail di 
[Bab 3][shadowing], tapi buat sekarang, tau aja kalau fitur ini sering dipake 
pas kita mau convert sebuah nilai dari satu tipe ke tipe lainnya.

Kita nge-bind variabel baru ini ke ekspresi `guess.trim().parse()`. `guess` yang 
ada di ekspresi itu ngerujuk ke variabel `guess` asli yang isinya input sebagai 
string. Method `trim` pada instance `String` bakal ngilangin whitespace apa pun 
di awal dan akhir, yang emang harus kita lakuin sebelum kita bisa convert 
string-nya jadi `u32`, yang cuma bisa berisi data numerik. User harus teken 
<kbd>enter</kbd> buat menuhi `read_line` dan masukin tebakan mereka, yang 
nambahin karakter baris baru (newline) ke string-nya. Misalnya, kalau user ketik 
<kbd>5</kbd> terus teken <kbd>enter</kbd>, `guess` bakal keliatan kayak gini: 
`5\n`. `\n` itu simbol buat “newline.” (Di Windows, nekan <kbd>enter</kbd> 
ngasilin carriage return sama newline, `\r\n`.) Method `trim` ngilangin `\n` 
atau `\r\n`, hasilnya cuma `5`.

Method [`parse` pada string][parse] convert sebuah string ke tipe lain. Di sini, 
kita pake itu buat convert dari string jadi angka. Kita perlu ngasih tau Rust 
tipe angka pasti yang kita mau dengan pake `let guess: u32`. Titik dua (`:`) 
setelah `guess` ngasih tau Rust kalau kita bakal annotasi tipe variabelnya. 
Rust punya beberapa tipe angka bawaan; `u32` yang diliat di sini adalah 32-bit 
unsigned integer. Ini pilihan default yang bagus buat angka positif kecil. Kita 
bakal belajar tipe angka lainnya di [Bab 3][integers].

Terus, annotasi `u32` di program contoh ini dan perbandingan sama 
`secret_number` bikin Rust tau kalau `secret_number` juga harusnya tipe `u32`. 
Jadi sekarang perbandingannya bakal terjadi antara dua nilai dengan tipe yang 
sama!

Method `parse` cuma bakal kerja pada karakter yang logisnya bisa di-convert jadi 
angka, jadi dia gampang sekali bikin error. Kalau misalnya string-nya isinya 
`A👍%`, nggak bakal ada cara buat convert itu jadi angka. Karena dia mungkin 
gagal, method `parse` balikin tipe `Result`, mirip kayak method `read_line` (yang 
udah dibahas tadi di [“Menangani Potensi Gagal dengan `Result`”](#handling-potential-failure-with-the-result-type)). 
Kita bakal perlakukan `Result` ini dengan cara yang sama pake method `expect` 
lagi. Kalau `parse` balikin varian `Err` dari `Result` karena dia nggak bisa 
bikin angka dari string-nya, panggilan `expect` bakal bikin gamenya crash terus 
nyetak pesan yang kita kasih. Kalau `parse` berhasil convert string-nya jadi 
angka, dia bakal balikin varian `Ok` dari `Result`, terus `expect` bakal 
balikin angka yang kita mau dari nilai `Ok` itu.

Yuk kita jalanin programnya sekarang:

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

Keren! Walaupun ada spasi yang ditambahin sebelum tebakannya, programnya tetep 
tau kalau user nebak 76. Jalanin programnya beberapa kali buat mastiin perilaku 
yang beda-beda dengan berbagai jenis input: tebak angkanya dengan bener, tebak 
angka yang ketinggian, sama tebak angka yang kerendahan.

Kita udah punya sebagian besar gamenya jalan sekarang, tapi user cuma bisa nebak 
sekali. Yuk kita ubah itu dengan nambahin loop!

## Ngebolehin Banyak Tebakan dengan Looping

Keyword `loop` bikin loop yang nggak bakal berhenti (infinite loop). Kita bakal 
tambahin loop biar user punya lebih banyak kesempatan buat nebak angkanya:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Kayak yang kita liat, kita udah mindahin segala hal dari prompt input tebakan 
dan seterusnya ke dalem sebuah loop. Pastiin buat indentasi baris-baris di dalem 
loop-nya nambah empat spasi lagi masing-masing terus jalanin programnya lagi. 
Programnya sekarang bakal minta tebakan lagi selamanya, yang sebenernya malah 
bikin masalah baru. Kayaknya user nggak bisa quit nih!

User sebenernya selalu bisa matiin programnya pake keyboard shortcut 
<kbd>ctrl</kbd>-<kbd>c</kbd>. Tapi ada cara lain buat kabur dari monster yang 
nggak pernah kenyang ini, kayak yang disebutin pas bahas `parse` di [“Membandingkan Tebakan dengan Secret Number”](#comparing-the-guess-to-the-secret-number): 
kalau user masukin jawaban yang bukan angka, programnya bakal crash. Kita bisa 
manfaatin itu biar user bisa quit, kayak yang ditunjukin di sini:

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

Ngetik `quit` bakal bikin gamenya berhenti, tapi kayak yang kita liat, masukin 
input apa pun yang bukan angka juga bakal bikin gitu. Ini kurang oke sih; kita 
mau gamenya juga berhenti pas angkanya udah berhasil ditebak dengan bener.

### Quit Setelah Tebakan Bener

Yuk program gamenya biar quit pas user menang dengan nambahin statement `break`:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Nambahin baris `break` setelah `You win!` bikin programnya keluar dari loop pas 
user nebak secret number-nya dengan bener. Keluar dari loop juga berarti keluar 
dari program, karena loop itu bagian terakhir dari `main`.

### Menangani Input Nggak Valid

Buat makin memperhalus perilaku gamenya, bukannya nge-crash-in program pas user 
input bukan angka, mendingan kita bikin gamenya cuekin input bukan angka itu 
biar user bisa lanjut nebak. Kita bisa lakuin itu dengan ngerubah baris di mana 
`guess` di-convert dari `String` jadi `u32`, kayak yang ditunjukin di Listing 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Cuekin tebakan bukan-angka terus minta tebakan lagi bukannya bikin program crash">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Kita ganti dari panggilan `expect` jadi ekspresi `match` biar bisa pindah dari 
nge-crash pas error jadi handle error-nya. Inget kalau `parse` balikin tipe 
`Result` dan `Result` itu enum yang punya varian `Ok` dan `Err`. Kita pake 
ekspresi `match` di sini, kayak yang kita lakuin sama hasil `Ordering` dari 
method `cmp`.

Kalau `parse` berhasil ngerubah string jadi angka, dia bakal balikin nilai `Ok` 
yang isinya angka hasilnya. Nilai `Ok` itu bakal cocok sama pattern arm pertama, 
terus ekspresi `match` bakal langsung balikin nilai `num` yang dihasilin 
`parse` dan ditaruh di dalem nilai `Ok` itu. Angka itu bakal berakhir tepat di 
tempat yang kita mau di variabel `guess` baru yang kita bikin.

Kalau `parse` _nggak_ bisa ngerubah string jadi angka, dia bakal balikin nilai 
`Err` yang isinya informasi lebih lanjut soal error-nya. Nilai `Err` itu nggak 
cocok sama pattern `Ok(num)` di arm `match` pertama, tapi dia cocok sama 
pattern `Err(_)` di arm kedua. Garis bawah, `_`, itu nilai _catch-all_; di 
contoh ini, kita bilang kita mau nyocokin semua nilai `Err`, nggak peduli 
informasi apa yang ada di dalemnya. Jadi programnya bakal jalanin kode arm 
kedua, `continue`, yang ngasih tau program buat lanjut ke iterasi `loop` 
berikutnya dan minta tebakan lagi. Jadi, secara efektif, programnya cuekin semua 
error yang mungkin ditemuin `parse`!

Sekarang segala hal di programnya harusnya jalan sesuai harapan. Yuk coba:

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

Mantul! Dengan satu perubahan kecil terakhir, kita bakal selesein game tebak 
angka ini. Inget kalau programnya masih nyetak secret number-nya. Itu emang enak 
buat ngetes, tapi ngerusak gamenya. Yuk kita hapus `println!` yang ngeluarin 
secret number itu. Listing 2-6 nunjukin kode final-nya.

<Listing number="2-6" file-name="src/main.rs" caption="Kode lengkap game tebak angka">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Sampai titik ini, kita udah berhasil bikin game tebak angka. Selamat ya!

## Ringkasan

Project ini adalah cara belajar langsung (hands-on) buat ngenalin kita ke banyak 
konsep baru di Rust: `let`, `match`, fungsi, penggunaan external crates, dan 
banyak lagi. Di beberapa bab ke depan, kita bakal belajar konsep-konsep ini 
lebih detail. Bab 3 bahas konsep yang ada di kebanyakan bahasa pemrograman, 
kayak variabel, tipe data, dan fungsi, terus nunjukin cara pakenya di Rust. 
Bab 4 bahas ownership, fitur yang bikin Rust beda dari bahasa lain. Bab 5 bahas 
structs sama sintaks method, terus Bab 6 jelasin gimana cara kerja enum.

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
