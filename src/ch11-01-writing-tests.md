## Cara Menulis Pengujian (Tests)

Pengujian (tests) adalah fungsi Rust yang memverifikasi kalau kode selain kode 
pengujian berjalan sesuai yang diharapkan. _Body_ dari fungsi pengujian biasanya 
melakukan tiga aksi ini:

- Menyiapkan data atau _state_ (keadaan) yang dibutuhkan.
- Menjalankan kode yang mau diuji.
- Menegaskan (assert) kalau hasilnya sesuai dengan yang diharapkan.

Mari kita lihat fitur-fitur yang disediakan Rust khusus buat menulis pengujian 
yang mengambil aksi-aksi ini, termasuk atribut `test`, beberapa macro, dan 
atribut `should_panic`.

### Anatomi Fungsi Pengujian

Paling sederhananya, pengujian di Rust adalah sebuah fungsi yang dianotasi 
dengan atribut `test`. Atribut adalah _metadata_ tentang kode Rust; salah satu 
contohnya adalah atribut `derive` yang kita pakai buat _structs_ di Bab 5. 
Untuk mengubah sebuah fungsi menjadi fungsi pengujian, tambahkan `#[test]` di 
baris sebelum `fn`. Saat kita menjalankan pengujian menggunakan perintah 
`cargo test`, Rust bakal mem-build sebuah _test runner binary_ yang menjalankan 
fungsi-fungsi yang dianotasi ini dan melaporkan apakah setiap fungsi pengujian 
tersebut sukses atau gagal.

Kapan pun kita membuat project _library_ baru dengan Cargo, sebuah modul 
pengujian (test module) dengan satu fungsi pengujian di dalamnya akan otomatis 
dibuatkan buat kita. Modul ini ngasih kita _template_ buat nulis pengujian jadi 
kita tidak perlu repot-repot nyari struktur dan sintaks persisnya setiap kali 
mulai project baru. Kita bisa menambahkan sebanyak apa pun fungsi pengujian 
tambahan dan modul pengujian yang kita mau!

Kita bakal eksplorasi beberapa aspek soal gimana pengujian itu bekerja dengan 
ber-eksperimen menggunakan pengujian _template_ ini sebelum kita benar-benar 
menguji kode apa pun. Lalu kita bakal nulis beberapa pengujian dunia nyata yang 
memanggil kode yang sudah kita tulis dan menegaskan kalau perilakunya sudah benar.

Mari kita buat project _library_ baru bernama `adder` yang bakal menjumlahkan 
dua angka:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

Isi dari file _src/lib.rs_ di _library_ `adder` kita seharusnya kelihatan seperti 
Listing 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="Kode yang di-generate secara otomatis oleh `cargo new`">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

File ini dimulai dengan sebuah contoh fungsi `add`, supaya kita punya sesuatu 
buat diuji.

Buat sekarang, mari fokus pada fungsi `it_works` aja. Perhatikan anotasi 
`#[test]`: atribut ini menunjukkan kalau ini adalah fungsi pengujian, jadi 
_test runner_ tahu buat memperlakukan fungsi ini sebagai pengujian. Kita juga 
bisa punya fungsi biasa di dalam modul `tests` untuk membantu menyiapkan 
skenario umum atau menjalankan operasi umum, jadi kita harus selalu menunjukkan 
fungsi mana yang merupakan fungsi pengujian.

Contoh _body_ fungsi ini memakai macro `assert_eq!` untuk menegaskan kalau 
`result`, yang menampung hasil pemanggilan `add` dengan angka 2 dan 2, itu 
sama dengan 4. Penegasan ini berfungsi sebagai contoh format buat pengujian yang 
umum. Mari kita jalankan untuk melihat kalau pengujian ini sukses (_passes_).

Perintah `cargo test` menjalankan semua pengujian di dalam project kita, seperti 
yang ditunjukkan di Listing 11-2.

<Listing number="11-2" caption="Output dari menjalankan pengujian yang di-generate otomatis">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo men-compile dan menjalankan pengujiannya. Kita melihat baris 
`running 1 test`. Baris berikutnya menunjukkan nama fungsi pengujian yang 
di-generate, bernama `tests::it_works`, dan hasil dari pengujian itu adalah `ok`. 
Ringkasan keseluruhan `test result: ok.` berarti semua pengujian sukses, dan 
bagian yang tertulis `1 passed; 0 failed` menjumlahkan banyaknya pengujian yang 
sukses atau gagal.

Kita bisa menandai sebuah pengujian untuk diabaikan (ignored) supaya tidak 
dijalankan pada waktu tertentu; kita bakal membahas ini di bagian [“Mengabaikan 
Beberapa Pengujian Kecuali Diminta Secara Spesifik”][ignoring] nanti di bab ini. 
Karena kita belum melakukannya di sini, ringkasannya menunjukkan `0 ignored`. Kita 
juga bisa mengirimkan argumen ke perintah `cargo test` untuk menjalankan hanya 
pengujian yang namanya cocok dengan _string_ tertentu; ini disebut _filtering_ 
(penyaringan) dan kita bakal membahasnya di bagian [“Menjalankan Sebagian 
Pengujian Berdasarkan Nama”][subset]. Di sini kita tidak menyaring pengujiannya, 
jadi akhir dari ringkasan menunjukkan `0 filtered out`.

Statistik `0 measured` adalah untuk pengujian _benchmark_ yang mengukur performa. 
Pengujian _benchmark_, saat tulisan ini dibuat, baru tersedia di versi Rust 
_nightly_. Cek [dokumentasi soal pengujian _benchmark_][bench] buat info lebih 
lanjut.

Bagian selanjutnya dari output pengujian yang dimulai dengan `Doc-tests adder` 
adalah hasil dari _documentation tests_ (pengujian dokumentasi). Kita belum 
punya _documentation tests_ apa pun, tapi Rust bisa men-compile contoh kode apa 
pun yang ada di dokumentasi API kita. Fitur ini membantu menjaga supaya 
dokumentasi dan kode kita tetap sinkron! Kita bakal membahas cara menulis 
_documentation tests_ di bagian [“Komentar Dokumentasi Sebagai 
Pengujian”][doc-comments] di Bab 14. Buat sekarang, kita abaikan saja output 
`Doc-tests` ini.

Mari kita mulai mengubah pengujian ini sesuai kebutuhan kita sendiri. Pertama, 
ubah nama fungsi `it_works` jadi nama yang beda, misalnya `exploration`, kayak 
gini:

<span class="filename">Nama file: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Kemudian jalankan `cargo test` lagi. Outputnya sekarang bakal menunjukkan 
`exploration` bukannya `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Sekarang kita bakal nambahin pengujian satu lagi, tapi kali ini kita bakal bikin 
pengujian yang gagal! Pengujian bakal gagal kalau ada sesuatu di dalam fungsi 
pengujian tersebut yang menyebabkan _panic_. Tiap pengujian dijalankan di dalam 
_thread_ baru, dan saat _main thread_ melihat ada _test thread_ yang mati, 
pengujian itu bakal ditandai sebagai gagal. Di Bab 9, kita membahas gimana cara 
paling sederhana untuk membuat _panic_ adalah dengan memanggil macro `panic!`. 
Masukkan pengujian baru ini sebagai fungsi bernama `another`, sehingga file 
_src/lib.rs_ kita kelihatan seperti Listing 11-3.

<Listing number="11-3" file-name="src/lib.rs" caption="Menambahkan pengujian kedua yang bakal gagal karena kita memanggil macro `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

Jalankan pengujiannya lagi memakai `cargo test`. Output-nya seharusnya kelihatan 
kayak Listing 11-4, yang menunjukkan kalau pengujian `exploration` kita sukses 
tapi pengujian `another` kita gagal.

<Listing number="11-4" caption="Hasil pengujian saat satu pengujian sukses dan yang lainnya gagal">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```
</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

Bukannya `ok`, baris `test tests::another` menunjukkan `FAILED`. Ada dua bagian 
baru yang muncul di antara hasil masing-masing pengujian dan ringkasan akhir: 
bagian pertama menampilkan detail alasan setiap pengujian yang gagal. Di kasus 
ini, kita mendapat detail bahwa `tests::another` gagal karena terjadi _panic_ 
dengan pesan `Make this test fail` di baris 17 pada file _src/lib.rs_. Bagian 
berikutnya mendaftarkan cuma nama-nama dari semua pengujian yang gagal, yang 
mana sangat berguna saat ada banyak pengujian dan sangat banyak output dari 
pengujian yang gagal. Kita bisa memakai nama pengujian yang gagal itu untuk 
menjalankan hanya pengujian tersebut agar lebih gampang men-_debug_-nya; kita 
bakal membahas lebih lanjut soal cara-cara menjalankan pengujian di bagian 
[“Mengontrol Bagaimana Pengujian Dijalankan”][controlling-how-tests-are-run].

Baris ringkasan ditampilkan di bagian paling akhir: secara keseluruhan, hasil 
pengujian kita adalah `FAILED`. Kita punya satu pengujian yang sukses dan satu 
yang gagal.

Sekarang karena kita sudah melihat seperti apa hasil pengujian di berbagai 
skenario, mari kita lihat beberapa macro selain `panic!` yang berguna buat 
pengujian.

### Memeriksa Hasil dengan Macro `assert!`

Macro `assert!`, yang disediakan oleh _standard library_, berguna saat kita 
mau memastikan bahwa suatu kondisi di pengujian dievaluasi menjadi `true`. Kita 
memberikan sebuah argumen ke macro `assert!` yang bakal dievaluasi jadi sebuah 
Boolean. Kalau nilainya `true`, tidak ada yang terjadi dan pengujiannya bakal 
sukses. Kalau nilainya `false`, macro `assert!` memanggil `panic!` sehingga 
pengujiannya gagal. Memakai macro `assert!` sangat membantu buat mengecek apakah 
kode kita berfungsi seperti yang kita mau.

Di Bab 5, Listing 5-15, kita memakai _struct_ `Rectangle` dan _method_ `can_hold`, 
yang mana diulangi lagi di sini di Listing 11-5. Mari kita masukkan kode ini 
ke dalam file _src/lib.rs_, lalu kita tulis beberapa pengujian buat kode tersebut 
menggunakan macro `assert!`.

<Listing number="11-5" file-name="src/lib.rs" caption="_Struct_ `Rectangle` dan _method_ `can_hold`-nya dari Bab 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

_Method_ `can_hold` mengembalikan Boolean, yang artinya ini adalah _use case_ 
yang sangat pas buat macro `assert!`. Di Listing 11-6, kita nulis pengujian 
yang mencoba _method_ `can_hold` dengan bikin instance `Rectangle` yang punya 
lebar 8 dan tinggi 7, lalu menegaskan kalau ia bisa menampung instance `Rectangle` 
lain yang punya lebar 5 dan tinggi 1.

<Listing number="11-6" file-name="src/lib.rs" caption="Pengujian buat `can_hold` yang mengecek apakah persegi panjang yang lebih besar benar-benar bisa menampung persegi panjang yang lebih kecil">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

Perhatikan baris `use super::*;` di dalam modul `tests`. Modul `tests` adalah 
modul biasa yang mengikuti aturan visibilitas standar yang sudah kita bahas 
di Bab 7 di bagian [“Paths untuk Merujuk ke sebuah Item di Pohon Modul”][paths-for-referring-to-an-item-in-the-module-tree]. 
Karena modul `tests` adalah _inner module_ (modul di dalam modul lain), kita 
perlu membawa kode yang mau diuji di modul luar ke dalam _scope_ modul 
`tests` ini. Kita memakai _glob_ (`*`) di sini, jadi semua hal yang kita 
definisikan di modul luar bakal tersedia di modul `tests` ini.

Kita menamakan pengujian kita `larger_can_hold_smaller`, dan kita membuat dua 
instance `Rectangle` yang kita butuhkan. Kemudian kita memanggil macro `assert!` 
dan memasukkan hasil pemanggilan `larger.can_hold(&smaller)` kepadanya. 
Ekspresi ini seharusnya mengembalikan `true`, jadi pengujian kita seharusnya 
sukses. Mari kita buktikan!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Sukses kan! Mari tambahkan pengujian satu lagi, kali ini menegaskan bahwa persegi 
panjang yang lebih kecil tidak bisa menampung persegi panjang yang lebih besar:

<span class="filename">Nama file: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Karena hasil yang benar dari fungsi `can_hold` di kasus ini adalah `false`, 
kita perlu men-negasikan hasil tersebut sebelum memasukkannya ke macro `assert!`. 
Sebagai hasilnya, pengujian kita bakal sukses kalau `can_hold` mengembalikan 
`false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Dua pengujian yang sukses! Sekarang mari kita lihat apa yang terjadi pada hasil 
pengujian kalau kita memunculkan _bug_ di kode kita. Kita bakal mengubah 
implementasi dari _method_ `can_hold` dengan mengganti tanda lebih-dari menjadi 
tanda kurang-dari saat dia membandingkan lebar (`widths`):

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Menjalankan pengujiannya sekarang bakal menghasilkan output ini:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Pengujian kita berhasil menangkap _bug_ tersebut! Karena `larger.width` adalah 
`8` dan `smaller.width` adalah `5`, perbandingan lebar-lebar ini di `can_hold` 
sekarang mengembalikan `false`: 8 itu tidak kurang dari 5.

### Menguji Kesamaan dengan Macro `assert_eq!` dan `assert_ne!`

Cara yang sangat umum buat memverifikasi fungsionalitas adalah dengan menguji 
kesamaan antara hasil dari kode yang diuji dengan nilai yang kita harapkan bakal 
dikembalikan oleh kode tersebut. Kita bisa saja melakukannya memakai macro 
`assert!` dengan memasukkan ekspresi yang memakai operator `==`. Tapi, karena 
ini adalah bentuk pengujian yang sering sekali dipakai, _standard library_ 
menyediakan dua macro—`assert_eq!` dan `assert_ne!`—buat melakukan pengujian 
ini dengan lebih praktis. Kedua macro ini membandingkan dua argumen buat melihat 
apakah mereka sama (_equality_) atau tidak sama (_inequality_). Mereka juga bakal 
mencetak kedua nilai tersebut kalau penegasannya gagal, yang mana bikin kita lebih 
gampang melihat _kenapa_ pengujian itu gagal; sebaliknya, macro `assert!` cuma 
menunjukkan kalau dia dapat nilai `false` dari ekspresi `==`, tanpa mencetak 
nilai-nilai apa saja yang membuat hasil tersebut jadi `false`.

Di Listing 11-7, kita menulis sebuah fungsi bernama `add_two` yang menambahkan 
`2` ke parameternya, lalu kita menguji fungsi ini menggunakan macro `assert_eq!`.

<Listing number="11-7" file-name="src/lib.rs" caption="Menguji fungsi `add_two` memakai macro `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Mari cek apakah pengujiannya sukses!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Kita membuat sebuah variabel bernama `result` yang menampung hasil pemanggilan 
`add_two(2)`. Kemudian kita memasukkan `result` dan `4` sebagai argumen buat 
macro `assert_eq!`. Baris output buat pengujian ini adalah `test tests::it_adds_two
... ok`, dan teks `ok` menunjukkan bahwa pengujian kita sukses!

Mari kita masukkan sebuah _bug_ ke dalam kode kita buat melihat seperti apa 
`assert_eq!` kalau dia gagal. Ubah implementasi fungsi `add_two` biar dia malah 
menambahkan `3`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Jalankan pengujiannya lagi:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Pengujian kita berhasil menangkap _bug_ tersebut! Pengujian `tests::it_adds_two` 
gagal, dan pesannya memberi tahu kita kalau penegasan yang gagal adalah 
`left == right` beserta apa nilai `left` dan `right` yang didapatkan. Pesan ini 
ngebantu kita buat mulai melakukan _debugging_: argumen `left`, di mana kita 
menaruh hasil pemanggilan `add_two(2)`, itu ternyata `5`, padahal argumen 
`right` adalah `4`. Kita bisa bayangin kalau informasi ini bakal kepake sekali 
terutama kalau ada banyak pengujian yang lagi jalan.

Perhatikan bahwa di beberapa bahasa dan _test frameworks_ (kerangka pengujian), 
parameter untuk fungsi penegasan kesamaan biasanya dinamakan `expected` (harapan) 
dan `actual` (asli), dan urutan kita memasukkan argumen-argumennya itu penting. 
Tapi di Rust, mereka disebut `left` (kiri) dan `right` (kanan), dan urutan 
kita meletakkan nilai ekspektasi kita sama nilai hasil dari kodenya itu sama 
sekali tidak masalah. Kita bisa saja menulis penegasan di pengujian ini sebagai 
`assert_eq!(4, result)`, dan ini bakal menghasilkan pesan gagal yang sama yang 
menampilkan `` assertion `left == right` failed``.

Macro `assert_ne!` bakal sukses (pass) kalau dua nilai yang kita masukkan 
tidak sama, dan bakal gagal kalau mereka sama. Macro ini paling berguna buat 
kasus di mana kita tidak yakin nilai tersebut _bakal_ jadi apa, tapi kita tahu 
pasti nilai tersebut _tidak boleh_ jadi apa. Misalnya, kalau kita menguji fungsi 
yang dijamin mengubah inputnya dengan cara tertentu, tapi perubahan itu 
tergantung sama hari dalam seminggu saat kita menjalankan pengujiannya, maka hal 
terbaik yang bisa ditegaskan (assert) mungkin adalah bahwa output fungsinya 
tidak sama dengan inputnya.

Di balik layar, macro `assert_eq!` dan `assert_ne!` masing-masing memakai operator 
`==` dan `!=`. Saat penegasan gagal, macro ini mencetak argumen mereka memakai 
_debug formatting_, yang berarti nilai-nilai yang dibandingkan harus 
mengimplementasikan _trait_ `PartialEq` dan `Debug`. Semua tipe primitif dan 
sebagian besar tipe di _standard library_ sudah mengimplementasikan _traits_ 
ini. Buat _structs_ dan _enums_ yang kita definisikan sendiri, kita harus 
mengimplementasikan `PartialEq` buat menegaskan kesamaan tipe tersebut. Kita juga 
harus mengimplementasikan `Debug` buat mencetak nilainya saat penegasan gagal. 
Karena kedua _traits_ ini adalah _derivable traits_, seperti yang disebutkan di 
Listing 5-12 di Bab 5, hal ini biasanya semudah menambahkan anotasi 
`#[derive(PartialEq, Debug)]` ke dalam definisi _struct_ atau _enum_ kita. 
Lihat Lampiran C, [“Derivable Traits,”][derivable-traits] buat detail lebih 
lanjut soal ini dan _derivable traits_ lainnya.

### Menambahkan Pesan Kegagalan Kustom

Kita juga bisa menambahkan pesan kustom buat dicetak bersama pesan kegagalan 
sebagai argumen opsional di macro `assert!`, `assert_eq!`, dan `assert_ne!`. 
Argumen apa pun yang ditaruh setelah argumen wajib bakal diteruskan langsung ke 
macro `format!` (dibahas di [“Penggabungan dengan Operator `+` atau 
Macro `format!`”][concatenation-with-the--operator-or-the-format-macro] di Bab 8), 
jadi kita bisa memasukkan _format string_ yang mengandung *placeholder* `{}` 
serta nilai-nilai buat mengisi *placeholder* tersebut. Pesan kustom ini berguna 
buat mendokumentasikan apa maksud dari sebuah penegasan; saat sebuah pengujian 
gagal, kita jadi punya gambaran yang lebih baik soal masalah apa yang sedang 
terjadi dengan kodenya.

Misalnya, katakanlah kita punya fungsi yang menyapa orang dengan namanya dan 
kita mau menguji apakah nama yang kita berikan ke fungsi tersebut muncul di 
dalam outputnya:

<span class="filename">Nama file: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Persyaratan buat program ini belum sepenuhnya disepakati, dan kita cukup yakin 
kalau teks `Hello` di awal sapaan bakal berubah. Kita udah mutusin kalau kita 
tidak mau terus-terusan meng-update pengujiannya tiap kali persyaratan ini 
berubah, jadi alih-alih mengecek kesamaan yang sama persis dengan nilai kembalian 
dari fungsi `greeting`, kita hanya menegaskan kalau outputnya mengandung teks 
dari parameter inputnya.

Sekarang mari kita masukkan sebuah _bug_ ke dalam kode ini dengan mengubah 
`greeting` sehingga dia tidak menyertakan `name` dan kita bisa lihat seperti apa 
kegagalan pengujian standarnya:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Menjalankan pengujian ini bakal menghasilkan output berikut:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Hasil ini cuma ngasih tahu kalau penegasannya gagal dan di baris mana penegasan 
itu berada. Pesan kegagalan yang lebih membantu seharusnya mencetak nilai dari 
fungsi `greeting`. Mari tambahkan pesan kegagalan kustom yang disusun dari 
_format string_ dan *placeholder* yang diisi dengan nilai yang kita dapat 
dari fungsi `greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Sekarang ketika kita menjalankan pengujiannya, kita bakal dapat pesan error yang 
lebih informatif:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Kita bisa melihat nilai asli yang kita dapat di dalam output pengujiannya, yang 
mana sangat membantu kita buat _debug_ apa yang sedang terjadi bukannya 
cuma apa yang kita harapkan terjadi.

### Mengecek Panics dengan `should_panic`

Selain mengecek nilai kembalian, penting juga buat memastikan kalau kode kita 
menangani kondisi error sesuai yang diharapkan. Contohnya, mari perhatikan 
tipe `Guess` yang kita bikin di Bab 9, Listing 9-13. Kode lain yang memakai 
`Guess` bergantung pada jaminan bahwa instance `Guess` hanya bakal berisi 
angka antara 1 dan 100. Kita bisa nulis pengujian buat memastikan kalau 
mencoba membuat instance `Guess` dengan nilai di luar batas itu bakal 
mengakibatkan _panic_.

Kita melakukan ini dengan menambahkan atribut `should_panic` ke dalam fungsi 
pengujian kita. Pengujian ini bakal sukses jika kode di dalam fungsi tersebut 
mengalami _panic_; dan pengujian ini gagal jika kode di dalam fungsinya tidak 
_panic_.

Listing 11-8 menunjukkan pengujian yang mengecek bahwa kondisi error dari 
`Guess::new` memang terjadi pada waktu yang diharapkan.

<Listing number="11-8" file-name="src/lib.rs" caption="Menguji apakah sebuah kondisi bakal mengakibatkan `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Kita menaruh atribut `#[should_panic]` setelah atribut `#[test]` dan sebelum 
fungsi pengujian yang ia kenai. Mari lihat hasilnya ketika pengujian ini sukses:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Terlihat bagus! Sekarang mari masukkan sebuah _bug_ di kode kita dengan menghapus 
kondisi yang membuat fungsi `new` _panic_ kalau nilainya lebih besar dari 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Saat kita menjalankan pengujian di Listing 11-8, dia bakal gagal:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Kita tidak dapat pesan yang cukup membantu di kasus ini, tapi pas kita melihat 
fungsi pengujiannya, kita ingat kalau ia dianotasi dengan `#[should_panic]`. 
Kegagalan yang kita dapat berarti kode di dalam fungsi pengujian ini tidak 
menyebabkan _panic_.

Pengujian yang memakai `should_panic` bisa kurang presisi. Sebuah pengujian 
`should_panic` bakal tetap sukses meskipun terjadi _panic_ dengan alasan yang 
berbeda dari apa yang kita harapkan. Biar pengujian `should_panic` jadi lebih 
presisi, kita bisa menambahkan parameter opsional `expected` ke dalam atribut 
`should_panic`. _Test harness_ bakal memastikan kalau pesan kegagalan tersebut 
mengandung teks yang diberikan. Sebagai contoh, pertimbangkan kode yang 
dimodifikasi untuk `Guess` di Listing 11-9 di mana fungsi `new` bakal _panic_ 
dengan pesan yang berbeda-beda tergantung apakah nilainya terlalu kecil atau 
terlalu besar.

<Listing number="11-9" file-name="src/lib.rs" caption="Menguji sebuah `panic!` di mana pesan _panic_-nya harus mengandung potongan teks tertentu (substring)">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Pengujian ini bakal sukses karena teks yang kita tulis di parameter `expected` 
dari atribut `should_panic` adalah substring (bagian string) dari pesan _panic_ 
yang muncul di fungsi `Guess::new`. Kita bisa juga memberikan pesan _panic_ 
utuh yang kita harapkan, yang mana di kasus ini adalah `Guess value must be less 
than or equal to 100, got 200`. Apa yang kita pilih buat ditulis tergantung dari 
seberapa unik atau seberapa dinamis pesan _panic_-nya dan seberapa presisi 
kita mau pengujian ini. Di kasus ini, satu potongan teks saja dari pesan _panic_-nya 
sudah cukup buat memastikan kalau fungsi pengujian ini menjalankan kasus blok 
`else if value > 100`.

Untuk melihat apa yang terjadi saat sebuah pengujian `should_panic` dengan pesan 
`expected` itu gagal, mari kembali masukkan _bug_ ke dalam kode kita dengan menukar 
body dari blok `if value < 1` dan blok `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Kali ini pas kita jalanin pengujian `should_panic` ini, dia bakal gagal:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Pesan kegagalannya menunjukkan kalau pengujian ini memang _panic_ seperti yang 
kita harapkan, tapi pesan _panic_-nya tidak mengandung string yang diharapkan 
yaitu `less than or equal to 100`. Pesan _panic_ yang justru kita dapatkan di 
kasus ini adalah `Guess value must be greater than or equal to 1, got 200.` 
Nah, dengan info ini, kita bisa mulai menelusuri di mana letak _bug_ kita!

### Menggunakan `Result<T, E>` di Pengujian

Semua pengujian yang kita bikin sejauh ini bakal _panic_ saat gagal. Kita juga 
bisa menulis pengujian yang menggunakan `Result<T, E>`! Berikut adalah 
pengujian dari Listing 11-1, tapi ditulis ulang untuk memakai `Result<T, E>` 
dan mengembalikan `Err` daripada _panic_:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

Fungsi `it_works` sekarang punya tipe kembalian `Result<(), String>`. Di dalam 
body fungsi, alih-alih memanggil macro `assert_eq!`, kita mengembalikan `Ok(())` 
ketika pengujiannya sukses dan mengembalikan `Err` yang menampung `String` ketika 
pengujiannya gagal.

Menulis pengujian biar mereka mengembalikan `Result<T, E>` memungkinkan kita 
menggunakan _question mark operator_ (`?`) di dalam body pengujiannya, yang bisa 
jadi cara sangat nyaman untuk menulis pengujian yang seharusnya gagal jika ada 
operasi di dalamnya yang mengembalikan varian `Err`.

Kita tidak bisa memakai anotasi `#[should_panic]` di pengujian yang memakai 
`Result<T, E>`. Buat menegaskan kalau suatu operasi mengembalikan varian `Err`, 
_jangan_ pakai _question mark operator_ pada nilai `Result<T, E>` itu. Sebaliknya, 
pakai `assert!(value.is_err())`.

Sekarang setelah kita paham beberapa cara untuk menulis pengujian, mari kita 
lihat apa yang terjadi di balik layar saat kita menjalankan pengujian dan mulai 
mengeksplorasi berbagai opsi yang bisa kita pakai dengan `cargo test`.

[concatenation-with-the--operator-or-the-format-macro]: ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
