## Error Recoverable pake `Result`

Sebagian besar error nggak terlalu serius sampe harus ngehentiin program 
sepenuhnya. Kadang pas sebuah fungsi gagal itu karena alasan yang bisa kita 
interpretasi dan respon dengan gampang. Misalnya, kalau kita nyoba buka file 
dan operasi itu gagal gara-gara filenya nggak ada, kita mungkin mau bikin file 
itu bukannya malah nge-_terminate_ (menghentikan) prosesnya.

Inget dari [“Menangani Potensi Kegagalan dengan `Result`”][handle_failure] di 
Bab 2 kalau enum `Result` didefinisikan punya dua varian, `Ok` sama `Err`, 
kayak gini:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` sama `E` itu _generic type parameters_ (parameter tipe generik): kita bakal 
bahas generik lebih detail di Bab 10. Yang perlu kita tau sekarang adalah `T` 
merepresentasikan tipe dari nilai yang bakal dibalikin di kasus sukses di dalem 
varian `Ok`, dan `E` merepresentasikan tipe dari error yang bakal dibalikin di 
kasus gagal di dalem varian `Err`. Karena `Result` punya _generic type 
parameters_ ini, kita bisa pake tipe `Result` dan fungsi-fungsi yang 
didefinisikan padanya di banyak situasi yang beda di mana nilai sukses dan nilai 
error yang mau kita balikin mungkin beda-beda.

Yuk kita manggil fungsi yang balikin nilai `Result` karena fungsinya bisa aja 
gagal. Di Listing 9-3 kita nyoba ngebuka sebuah file.

<Listing number="9-3" file-name="src/main.rs" caption="Membuka sebuah file">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

Tipe kembalian (return type) dari `File::open` adalah `Result<T, E>`. Parameter 
generik `T` udah diisi sama implementasi dari `File::open` dengan tipe dari 
nilai suksesnya, yaitu `std::fs::File`, yang merupakan sebuah _file handle_. 
Tipe dari `E` yang dipake di nilai error adalah `std::io::Error`. Tipe kembalian 
ini artinya pemanggilan ke `File::open` bisa aja sukses dan balikin _file 
handle_ yang bisa kita baca atau tulis. Pemanggilan fungsi ini juga bisa aja 
gagal: misalnya, filenya mungkin nggak ada, atau kita mungkin nggak punya izin 
(permission) buat akses file itu. Fungsi `File::open` butuh cara buat ngasih 
tau kita apakah dia sukses atau gagal dan di saat yang sama ngasih kita antara 
_file handle_ atau informasi error. Informasi ini persis apa yang disampein 
sama enum `Result`.

Di kasus di mana `File::open` sukses, nilai di variabel `greeting_file_result` 
bakal jadi instance dari `Ok` yang nampung _file handle_. Di kasus di mana dia 
gagal, nilai di `greeting_file_result` bakal jadi instance dari `Err` yang 
nampung lebih banyak info soal jenis error yang terjadi.

Kita perlu nambahin kode di Listing 9-3 buat ngambil tindakan yang beda 
tergantung dari nilai yang dibalikin sama `File::open`. Listing 9-4 nunjukin 
salah satu cara buat nanganin `Result` pake _tool_ dasar, yaitu ekspresi `match` 
yang udah kita bahas di Bab 6.

<Listing number="9-4" file-name="src/main.rs" caption="Pake ekspresi `match` buat nanganin varian `Result` yang mungkin dibalikin">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Perhatiin ya kalau sama kayak enum `Option`, enum `Result` sama varian-variannya 
udah dibawa ke dalem scope lewat _prelude_, jadi kita nggak perlu nentuin 
`Result::` sebelum varian `Ok` sama `Err` di _arms_ dari `match`-nya.

Pas hasilnya `Ok`, kode ini bakal balikin nilai `file` di dalem varian `Ok`-nya, 
dan terus kita nge-assign nilai _file handle_ itu ke variabel `greeting_file`. 
Setelah `match`, kita bisa pake _file handle_-nya buat baca atau nulis.

Arm lain dari `match`-nya nanganin kasus di mana kita dapet nilai `Err` dari 
`File::open`. Di contoh ini, kita milih buat manggil macro `panic!`. Kalau 
nggak ada file namanya _hello.txt_ di direktori kita saat ini dan kita jalanin 
kode ini, kita bakal liat output berikut dari macro `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Kayak biasa, output ini ngasih tau kita persis apa yang salah.

### Nge-match di Error yang Beda-beda

Kode di Listing 9-4 bakal `panic!` nggak peduli apa alasan `File::open` gagal. 
Padahal, kita mau ngambil tindakan yang beda buat alasan kegagalan yang beda 
juga. Kalau `File::open` gagal gara-gara filenya nggak ada, kita mau bikin 
file itu terus balikin _handle_ ke file baru itu. Kalau `File::open` gagal buat 
alasan apa pun lainnya—misalnya, karena kita nggak punya izin buat buka 
filenya—kita tetep mau kodenya buat `panic!` dengan cara yang sama kayak di 
Listing 9-4. Buat ini, kita nambahin ekspresi `match` bersarang (inner match), 
yang ditunjukin di Listing 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Nanganin jenis error yang beda dengan cara yang beda juga">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

Tipe dari nilai yang dibalikin sama `File::open` di dalem varian `Err` adalah 
`io::Error`, yang merupakan struct yang disediain sama _standard library_. 
Struct ini punya method `kind` yang bisa kita panggil buat dapet nilai 
`io::ErrorKind`. Enum `io::ErrorKind` disediain sama _standard library_ dan 
punya varian-varian yang merepresentasikan berbagai jenis error yang mungkin 
terjadi dari sebuah operasi `io`. Varian yang mau kita pake adalah 
`ErrorKind::NotFound`, yang ngindikasikan kalau file yang lagi kita coba buka 
itu belum ada. Jadi kita nge-match `greeting_file_result`, tapi kita juga punya 
_inner match_ di `error.kind()`.

Kondisi yang mau kita cek di _inner match_ adalah apakah nilai yang dibalikin 
sama `error.kind()` itu adalah varian `NotFound` dari enum `ErrorKind`. Kalau 
iya, kita nyoba bikin filenya pake `File::create`. Tapi, karena `File::create` 
juga bisa aja gagal, kita butuh _arm_ kedua di ekspresi _inner match_ kita. Pas 
filenya nggak bisa dibuat, pesan error yang beda bakal dicetak. _Arm_ kedua dari 
`match` bagian luar (outer match) tetep sama, jadi programnya bakal _panic_ 
buat error apa pun selain error file nggak ditemuin.

> #### Alternatif Buat Penggunaan `match` dengan `Result<T, E>`
>
> Banyak sekali ya `match`-nya! Ekspresi `match` itu sangat berguna tapi dia 
> juga lumayan primitif. Di Bab 13, kita bakal belajar soal _closures_, yang 
> dipake bareng banyak method yang didefinisikan pada `Result<T, E>`. Method-
> method ini bisa lebih ringkas daripada pake `match` pas nanganin nilai 
> `Result<T, E>` di kode kita.
>
> Misalnya, ini cara lain buat nulis logika yang sama kayak yang ditunjukin di 
> Listing 9-5, kali ini pake _closures_ dan method `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Masalah pas bikin file: {error:?}");
>             })
>         } else {
>             panic!("Masalah pas buka file: {error:?}");
>         }
>     });
> }
> ```
>
> Walaupun kode ini punya perilaku yang sama kayak Listing 9-5, dia nggak punya 
> ekspresi `match` apa pun dan lebih bersih buat dibaca. Balik lagi ke contoh 
> ini setelah kita kelar baca Bab 13, terus cek method `unwrap_or_else` di 
> dokumentasi _standard library_. Masih banyak lagi method kayak gini yang bisa 
> ngerapihin ekspresi `match` bersarang yang sangat besar pas kita lagi berurusan 
> sama error.

#### Jalan Pintas buat Panic Kalo Error: `unwrap` sama `expect`

Pake `match` emang jalan dengan baik sih, tapi bisa agak kepanjangan (_verbose_) 
dan nggak selalu ngomunikasikan maksud kita dengan baik. Tipe `Result<T, E>` 
punya banyak method pembantu (helper methods) yang didefinisikan padanya buat 
ngelakuin berbagai tugas yang lebih spesifik. Method `unwrap` itu adalah method 
*shortcut* (jalan pintas) yang diimplementasikan persis kayak ekspresi `match` 
yang kita tulis di Listing 9-4. Kalau nilai `Result`-nya adalah varian `Ok`, 
`unwrap` bakal balikin nilai di dalem `Ok`-nya. Kalau `Result`-nya adalah 
varian `Err`, `unwrap` bakal manggil macro `panic!` buat kita. Ini contoh 
penggunaan `unwrap`:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Kalau kita jalanin kode ini tanpa file _hello.txt_, kita bakal liat pesan error 
dari pemanggilan `panic!` yang dilakuin sama method `unwrap`:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Sama halnya, method `expect` juga ngebolehin kita buat milih pesan error `panic!`-nya 
sendiri. Pake `expect` bukannya `unwrap` dan ngasih pesan error yang bagus 
bisa nyampein maksud kita dan bikin ngelacak sumber _panic_ jadi lebih gampang. 
Sintaks dari `expect` keliatan kayak gini:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Kita pake `expect` dengan cara yang sama kayak `unwrap`: buat balikin _file 
handle_-nya atau manggil macro `panic!`. Pesan error yang dipake sama `expect` 
di pemanggilan `panic!`-nya bakal jadi parameter yang kita masukin ke `expect`, 
bukannya pesan `panic!` default yang dipake sama `unwrap`. Ini contoh outputnya:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt harusnya udah disertain di project ini: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Di _production-quality code_ (kode level produksi), kebanyakan Rustacean milih 
`expect` daripada `unwrap` dan ngasih konteks lebih banyak soal kenapa operasi 
itu diharapkan bakal selalu berhasil. Dengan gitu, kalau asumsi kita terbukti 
salah, kita punya lebih banyak informasi yang bisa dipake pas _debugging_.

### Ngelepar Balik Error (Propagating Errors)

Pas implementasi sebuah fungsi manggil sesuatu yang mungkin gagal, bukannya 
nanganin error-nya di dalem fungsi itu sendiri, kita bisa milih buat balikin 
error-nya ke kode yang manggil fungsi itu biar mereka yang mutusin mau ngapain. 
Ini dikenal sebagai _propagating_ the error (ngelepar balik error) dan ngasih 
kontrol lebih banyak ke kode pemanggil, di mana mungkin ada lebih banyak 
informasi atau logika yang nentuin gimana error itu harus di-handle daripada 
apa yang tersedia di konteks fungsi kita.

Misalnya, Listing 9-6 nunjukin fungsi yang ngebaca username dari sebuah file. 
Kalau filenya nggak ada atau nggak bisa dibaca, fungsi ini bakal balikin error-
error itu ke kode yang manggil fungsi ini.

<Listing number="9-6" file-name="src/main.rs" caption="Fungsi yang balikin error ke kode pemanggil pake `match`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Fungsi ini sebenernya bisa ditulis pake cara yang jauh lebih pendek, tapi kita 
bakal mulai dengan ngelakuinnya secara manual biar bisa eksplor cara nanganin 
error; nanti di akhir, kita bakal tunjukin cara yang lebih singkat. Yuk kita 
liat tipe kembalian (return type) dari fungsinya dulu: 
`Result<String, io::Error>`. Ini artinya fungsinya balikin nilai dari tipe 
`Result<T, E>`, di mana parameter generik `T` udah diisi pake tipe konkret 
`String` dan tipe generik `E` udah diisi pake tipe konkret `io::Error`.

Kalau fungsinya berhasil tanpa masalah apa pun, kode yang manggil fungsi ini 
bakal nerima nilai `Ok` yang nampung sebuah `String`—yaitu `username` yang 
dibaca sama fungsi ini dari file. Kalau fungsi ini nemu masalah apa pun, kode 
pemanggil bakal nerima nilai `Err` yang nampung instance dari `io::Error` yang 
isinya informasi lebih lanjut soal masalahnya. Kita milih `io::Error` sebagai 
tipe kembalian dari fungsi ini karena kebetulan itu adalah tipe dari nilai error 
yang dibalikin dari kedua operasi yang lagi kita panggil di dalem body fungsi ini 
yang mungkin gagal: yaitu fungsi `File::open` sama method `read_to_string`.

Body dari fungsi ini dimulai dengan manggil fungsi `File::open`. Terus kita 
nanganin nilai `Result`-nya pake `match` yang mirip kayak `match` di Listing 9-4. 
Kalau `File::open` berhasil, _file handle_ di variabel pattern `file` bakal jadi 
nilai di variabel _mutable_ `username_file` dan fungsinya lanjut. Di kasus `Err`, 
bukannya manggil `panic!`, kita pake keyword `return` buat balik (return) 
lebih awal dari fungsi sepenuhnya dan ngelepar nilai error dari `File::open`, 
yang sekarang ada di variabel pattern `e`, balik ke kode pemanggil sebagai 
nilai error dari fungsi ini.

Jadi, kalau kita punya _file handle_ di `username_file`, fungsinya terus bikin 
`String` baru di variabel `username` terus manggil method `read_to_string` pada 
_file handle_ di `username_file` buat baca isi filenya ke dalem `username`. 
Method `read_to_string` juga balikin sebuah `Result` karena dia bisa aja gagal, 
walaupun `File::open` tadi udah berhasil. Jadi kita butuh `match` satu lagi 
buat nanganin `Result` itu: kalau `read_to_string` berhasil, berarti fungsi 
kita udah berhasil, dan kita balikin username dari filenya yang sekarang udah 
ada di variabel `username` yang dibungkus di dalem sebuah `Ok`. Kalau 
`read_to_string` gagal, kita balikin nilai error-nya pake cara yang sama kayak 
kita balikin nilai error di dalem `match` yang nanganin nilai kembalian dari 
`File::open`. Tapi, kita nggak perlu secara eksplisit bilang `return`, karena ini 
adalah ekspresi terakhir di fungsinya.

Kode yang manggil fungsi ini nanti bakal dapet antara nilai `Ok` yang isinya 
sebuah username atau nilai `Err` yang isinya sebuah `io::Error`. Terserah kode 
pemanggilnya mau ngapain sama nilai-nilai itu. Kalau kode pemanggilnya dapet 
nilai `Err`, dia bisa aja manggil `panic!` terus nge-_crash_-in programnya, 
pake username default, atau nyari username dari tempat lain selain dari file, 
misalnya. Kita nggak punya informasi yang cukup soal apa yang sebenernya lagi 
dicoba lakuin sama kode pemanggil, jadi kita nge-lempar balik semua informasi 
sukses atau error ke atas biar di-handle sama dia dengan bener.

Pola ngelepar balik error ini saking umumnya di Rust sampe Rust nyediain 
operator tanda tanya `?` buat bikin proses ini lebih gampang.

#### Jalan Pintas Buat Ngelepar Error: Operator `?`

Listing 9-7 nunjukin implementasi dari `read_username_from_file` yang punya 
fungsionalitas yang sama kayak di Listing 9-6, tapi implementasi ini pake 
operator `?`.

<Listing number="9-7" file-name="src/main.rs" caption="Sebuah fungsi yang balikin error ke kode pemanggil pake operator `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

Tanda `?` yang ditaruh setelah sebuah nilai `Result` itu didefinisikan buat 
kerja dengan cara yang hampir persis sama kayak ekspresi `match` yang kita 
definisikan buat nanganin nilai `Result` di Listing 9-6. Kalau nilai `Result`-nya 
adalah `Ok`, nilai di dalem `Ok`-nya bakal dibalikin dari ekspresi ini, dan 
programnya bakal lanjut. Kalau nilainya adalah `Err`, `Err` itu bakal dibalikin 
dari keseluruhan fungsi seolah-olah kita udah pake keyword `return`, jadi nilai 
error-nya bakal dilempar balik ke kode pemanggil.

Ada sedikit perbedaan antara apa yang dilakuin ekspresi `match` di Listing 9-6 
sama apa yang dilakuin operator `?`: nilai error yang dikenain operator `?` 
bakal ngelewatin fungsi `from`, yang didefinisikan di trait `From` di _standard 
library_, yang dipake buat nge-convert nilai dari satu tipe ke tipe lainnya. 
Pas operator `?` manggil fungsi `from`, tipe error yang diterima bakal di-convert 
jadi tipe error yang didefinisikan di tipe kembalian (return type) dari fungsi 
saat ini. Ini sangat berguna pas sebuah fungsi balikin satu tipe error kustom 
buat merepresentasikan semua kemungkinan cara fungsi itu bisa gagal, walaupun 
bagian-bagian di dalemnya mungkin gagal karena banyak alasan yang beda.

Misalnya, kita bisa ngubah fungsi `read_username_from_file` di Listing 9-7 buat 
balikin tipe error kustom namanya `OurError` yang kita definisikan sendiri. Kalau 
kita juga mendefinisikan `impl From<io::Error> for OurError` buat ngonstruksi 
sebuah instance dari `OurError` dari sebuah `io::Error`, maka pemanggilan 
operator `?` di dalem body `read_username_from_file` bakal manggil `from` dan 
nge-convert tipe error-nya tanpa perlu nambahin kode lain lagi ke fungsinya.

Di konteks Listing 9-7, tanda `?` di akhir pemanggilan `File::open` bakal 
balikin nilai di dalem `Ok` ke variabel `username_file`. Kalau ada error yang 
terjadi, operator `?` bakal langsung keluar (return early) dari keseluruhan 
fungsi dan ngasih nilai `Err` apa pun ke kode pemanggil. Hal yang sama juga 
berlaku buat tanda `?` di akhir pemanggilan `read_to_string`.

Operator `?` ngilangin sangat banyak _boilerplate code_ dan bikin implementasi 
fungsi ini jadi lebih simpel. Kita bahkan bisa nyederhanain kode ini lebih 
lanjut dengan nge-chaining (nyambungin) pemanggilan method langsung setelah `?`, 
kayak yang ditunjukin di Listing 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Nge-chaining pemanggilan method setelah operator `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Kita udah mindahin pembuatan `String` baru di variabel `username` ke awal dari 
fungsinya; bagian itu nggak berubah. Bukannya bikin variabel `username_file`, 
kita nge-chaining pemanggilan `read_to_string` secara langsung ke hasil dari 
`File::open("hello.txt")?`. Kita tetep punya tanda `?` di akhir pemanggilan 
`read_to_string`, dan kita tetep balikin nilai `Ok` yang nampung `username` 
pas baik `File::open` maupun `read_to_string` berhasil, bukannya balikin 
error-nya. Fungsionalitasnya tetep sama persis kayak di Listing 9-6 sama 
Listing 9-7; ini cuma cara yang beda dan lebih ergonomis buat nulis kodenya.

Listing 9-9 nunjukin cara buat ngebikin ini jadi lebih singkat lagi pake 
`fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Pake `fs::read_to_string` bukannya buka terus baca filenya secara manual">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Baca sebuah file jadi string itu operasi yang lumayan umum, jadi _standard 
library_ nyediain fungsi praktis `fs::read_to_string` yang buka filenya, bikin 
`String` baru, baca isinya dari file, masukin isinya ke dalem `String` itu, 
terus balikin nilai `String`-nya. Tentunya, kalau langsung pake 
`fs::read_to_string` dari awal kita jadi nggak dapet kesempatan buat ngejelasin 
semua penanganan error tadi, makanya kita bahas cara panjangnya dulu.

#### Di Mana Aja Operator `?` Bisa Dipake

Operator `?` cuma bisa dipake di fungsi-fungsi yang tipe kembaliannya (return 
type) kompatibel (cocok) sama nilai di mana tanda `?` itu dipake. Ini karena 
operator `?` didefinisikan buat ngelakuin _early return_ (kembali lebih awal) 
sebuah nilai keluar dari fungsi, sama kayak ekspresi `match` yang kita definisikan 
di Listing 9-6. Di Listing 9-6, `match`-nya pake nilai `Result`, dan arm _early 
return_-nya balikin nilai `Err(e)`. Tipe kembalian dari fungsinya juga harus 
berupa sebuah `Result` biar cocok sama `return` ini.

Di Listing 9-10, yuk kita liat error apa yang bakal kita dapet kalau kita nyoba 
pake operator `?` di dalem fungsi `main` yang punya tipe kembalian yang nggak 
kompatibel sama tipe dari nilai di mana kita pake tanda `?`.

<Listing number="9-10" file-name="src/main.rs" caption="Nyoba pake tanda `?` di fungsi `main` yang balikin `()` nggak bakal bisa di-compile.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Kode ini buka sebuah file, yang bisa aja gagal. Operator `?` ngikutin nilai 
`Result` yang dibalikin sama `File::open`, tapi fungsi `main` ini punya tipe 
kembalian `()`, bukan `Result`. Pas kita nge-compile kode ini, kita bakal dapet 
pesan error berikut:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Error ini nunjukin kalau kita cuma dibolehin pake operator `?` di fungsi yang 
balikin `Result`, `Option`, atau tipe lain yang mengimplementasikan `FromResidual`.

Buat benerin error-nya, kita punya dua pilihan. Pilihan pertama adalah ngubah 
tipe kembalian dari fungsinya biar kompatibel sama nilai yang lagi kita 
kenain operator `?` selama kita nggak punya batasan yang ngelarang hal itu. 
Pilihan lainnya adalah pake sebuah `match` atau salah satu dari method di 
`Result<T, E>` buat nanganin nilai `Result<T, E>`-nya dengan cara apa pun yang 
paling pas.

Pesan error-nya juga nyebutin kalau `?` bisa dipake bareng nilai `Option<T>` 
juga. Sama kayak pas pake `?` di dalem `Result`, kita cuma bisa pake `?` di 
dalem `Option` di sebuah fungsi yang juga balikin sebuah `Option`. Perilaku 
dari operator `?` pas dipanggil di dalem sebuah `Option<T>` itu mirip sama 
perilakunya pas dipanggil di dalem sebuah `Result<T, E>`: kalau nilainya adalah 
`None`, nilai `None` bakal dibalikin lebih awal dari fungsi di titik itu. Kalau 
nilainya adalah `Some`, nilai di dalem `Some`-nya bakal jadi nilai hasil dari 
ekspresi tersebut, dan fungsinya lanjut jalan. Listing 9-11 punya contoh 
sebuah fungsi yang nyari karakter terakhir dari baris pertama di dalem sebuah 
teks.

<Listing number="9-11" caption="Pake operator `?` di nilai `Option<T>`">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Fungsi ini balikin `Option<char>` karena mungkin aja ada karakter di sana, tapi 
mungkin juga nggak ada. Kode ini ngambil argumen string slice `text` terus 
manggil method `lines` pada string tersebut, yang bakal balikin sebuah iterator 
yang ngelewatin baris-baris di _string_-nya. Karena fungsi ini pengen nge-cek 
baris pertamanya, dia manggil `next` pada iterator-nya buat dapet nilai pertama 
dari iterator tersebut. Kalau `text` isinya _string_ kosong, pemanggilan 
`next` ini bakal balikin `None`, dan di kasus itu kita pake `?` buat berhenti 
terus balikin `None` dari `last_char_of_first_line`. Kalau `text` bukan _string_ 
kosong, `next` bakal balikin nilai `Some` yang nampung _string slice_ dari 
baris pertama di dalem `text`.

Tanda `?` ngekstrak _string slice_ itu, dan kita bisa manggil `chars` pada 
_string slice_ tersebut buat dapet iterator dari karakter-karakternya. Kita 
tertarik sama karakter terakhir di baris pertama ini, jadi kita manggil `last` 
buat balikin item terakhir di iterator-nya. Ini adalah sebuah `Option` karena 
mungkin aja baris pertamanya itu _string_ kosong; misalnya, kalau `text` dimulai 
pake baris kosong tapi punya karakter di baris lainnya, kayak di `"\nhi"`. Tapi, 
kalau emang ada karakter terakhir di baris pertama, dia bakal dibalikin di dalem 
varian `Some`. Operator `?` di tengah-tengah itu ngasih kita cara yang ringkas 
buat mengekspresikan logika ini, ngebolehin kita mengimplementasikan fungsi ini 
di dalem satu baris aja. Kalau kita nggak bisa pake operator `?` di dalem `Option`, 
kita harus mengimplementasikan logika ini pake lebih banyak pemanggilan method 
atau pake ekspresi `match`.

Perhatiin ya kalau kita bisa pake operator `?` di dalem `Result` di sebuah fungsi 
yang balikin `Result`, dan kita bisa pake operator `?` di dalem `Option` di 
sebuah fungsi yang balikin `Option`, tapi kita nggak bisa nyampur aduk. Operator 
`?` nggak bakal otomatis nge-convert sebuah `Result` jadi sebuah `Option` atau 
sebaliknya; di kasus kayak gitu, kita bisa pake method kayak `ok` pada `Result` 
atau method `ok_or` pada `Option` buat ngelakuin proses konversi-nya secara 
eksplisit.

Sejauh ini, semua fungsi `main` yang udah kita pake itu balikin `()`. Fungsi 
`main` itu spesial karena dia adalah _entry point_ dan _exit point_ dari program 
_executable_, dan ada batasan soal tipe kembaliannya apa aja yang dibolehin 
biar programnya bisa jalan sesuai ekspektasi.

Untungnya, `main` juga bisa balikin `Result<(), E>`. Listing 9-12 punya kode dari 
Listing 9-10, tapi kita udah ubah tipe kembalian dari `main` jadi 
`Result<(), Box<dyn Error>>` dan nambahin nilai kembalian `Ok(())` di akhirnya. 
Kode ini sekarang bakal bisa di-compile.

<Listing number="9-12" file-name="src/main.rs" caption="Ngubah `main` buat balikin `Result<(), E>` ngebolehin penggunaan operator `?` pada nilai `Result`.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

Tipe `Box<dyn Error>` adalah sebuah _trait object_, yang bakal kita bahas di 
[“Menggunakan Trait Objects Yang Mengizinkan Nilai Dari Tipe Yang Beda-beda”][trait-objects] 
di Bab 18. Buat sekarang, kita bisa anggep `Box<dyn Error>` artinya “jenis error 
apa pun.” Pake `?` di dalem nilai `Result` di fungsi `main` yang punya tipe 
error `Box<dyn Error>` itu dibolehin karena ini ngizinin nilai `Err` apa pun 
buat dibalikin lebih awal. Walaupun body dari fungsi `main` ini cuma bakal pernah 
balikin error bertipe `std::io::Error`, dengan nentuin `Box<dyn Error>`, 
_signature_ ini bakal tetep bener biarpun nanti ada lebih banyak kode yang 
balikin error tipe lain yang ditambahin ke dalem body `main`.

Pas fungsi `main` balikin `Result<(), E>`, executable-nya bakal exit (keluar) 
dengan nilai `0` kalau `main` balikin `Ok(())` dan bakal exit dengan nilai selain 
nol kalau `main` balikin nilai `Err`. Program executable yang ditulis dalam C 
bakal balikin integer pas mereka exit: program yang berhasil bakal balikin 
integer `0`, dan program yang error bakal balikin integer selain `0`. Rust juga 
balikin integer dari program executable buat tetep kompatibel sama konvensi ini.

Fungsi `main` bisa balikin tipe apa pun yang mengimplementasikan trait 
[`std::process::Termination`][termination], yang punya fungsi `report` yang 
balikin sebuah `ExitCode`. Cek dokumentasi _standard library_ buat info lebih 
lanjut soal gimana cara mengimplementasikan trait `Termination` buat tipe kustom 
kita sendiri.

Sekarang setelah kita bahas detail soal manggil `panic!` atau balikin `Result`, 
yuk kita balik ke topik soal gimana cara mutusin mana yang pas buat dipake di 
kasus yang mana.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html
