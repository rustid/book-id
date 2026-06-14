## Organisasi Pengujian

Seperti yang disebutkan di awal bab, pengujian adalah disiplin yang kompleks, 
dan orang-orang yang berbeda memakai istilah dan organisasi yang beda-beda juga. 
Komunitas Rust memikirkan pengujian dalam dua kategori utama: _unit tests_ (pengujian unit) 
dan _integration tests_ (pengujian integrasi). _Unit tests_ itu kecil dan lebih fokus, 
menguji satu modul secara terisolasi pada satu waktu, dan bisa menguji 
antarmuka (interface) _private_. _Integration tests_ itu sepenuhnya eksternal terhadap _library_ kita 
dan memakai kode kita dengan cara yang sama persis seperti kode eksternal lainnya, 
hanya memakai antarmuka _public_ dan berpotensi melibatkan beberapa modul per pengujian.

Menulis kedua jenis pengujian ini penting buat memastikan kalau bagian-bagian 
dari _library_ kita melakukan apa yang kita harapkan, baik secara terpisah 
maupun bersama-sama.

### Unit Tests

Tujuan dari _unit tests_ adalah buat menguji setiap unit kode secara terisolasi 
dari sisa kode lainnya buat dengan cepat mencari tahu di mana kode kita berjalan 
sesuai harapan atau tidak. Kita bakal menaruh _unit tests_ di dalam direktori 
_src_ di setiap file bersama dengan kode yang lagi mereka uji. Konvensinya adalah 
membuat modul bernama `tests` di setiap file buat menampung fungsi-fungsi 
pengujian dan menganotasi modul itu dengan `cfg(test)`.

#### Modul Tests dan `#[cfg(test)]`

Anotasi `#[cfg(test)]` pada modul `tests` memberi tahu Rust buat men-compile dan 
menjalankan kode pengujian hanya ketika kita menjalankan `cargo test`, dan tidak 
saat kita menjalankan `cargo build`. Ini menghemat waktu kompilasi (compile time) 
saat kita cuma mau mem-build _library_-nya dan menghemat ruang di dalam artefak 
hasil kompilasi (_compiled artifact_) karena pengujian-pengujiannya tidak 
dimasukkan. Nanti kita bakal lihat kalau _integration tests_ itu berada di 
direktori yang berbeda, jadi mereka tidak butuh anotasi `#[cfg(test)]`. Tapi, 
karena _unit tests_ berada di file yang sama dengan kodenya, kita bakal pakai 
`#[cfg(test)]` buat menentukan kalau mereka seharusnya tidak dimasukkan di 
hasil kompilasinya.

Ingat kembali waktu kita men-generate project `adder` baru di bagian pertama 
bab ini, Cargo men-generate kode ini buat kita:

<span class="filename">Nama file: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Pada modul `tests` yang di-generate otomatis, atribut `cfg` singkatan dari 
_configuration_ (konfigurasi) dan ngasih tahu Rust kalau _item_ berikutnya hanya 
boleh dimasukkan jika ada opsi konfigurasi tertentu. Di kasus ini, opsi 
konfigurasinya adalah `test`, yang disediakan oleh Rust buat men-compile dan 
menjalankan pengujian. Dengan memakai atribut `cfg`, Cargo men-compile kode 
pengujian kita cuma kalau kita secara aktif menjalankan pengujiannya dengan 
`cargo test`. Ini termasuk fungsi bantuan (_helper functions_) apa pun yang 
mungkin ada di dalam modul ini, selain fungsi-fungsi yang dianotasi dengan 
`#[test]`.

#### Menguji Fungsi Private

Ada perdebatan di komunitas pengujian tentang apakah fungsi _private_ (privat) 
sebaiknya diuji secara langsung atau tidak, dan bahasa pemrograman lain membuat 
pengujian fungsi _private_ jadi hal yang susah atau mustahil. Terlepas dari 
ideologi pengujian mana yang kita anut, aturan privasi Rust memang 
memperbolehkan kita untuk menguji fungsi _private_. Coba perhatikan kode di 
Listing 11-12 yang punya fungsi _private_ `internal_adder`.

<Listing number="11-12" file-name="src/lib.rs" caption="Menguji fungsi _private_">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Perhatikan bahwa fungsi `internal_adder` tidak ditandai sebagai `pub`. 
Pengujian itu pada dasarnya cuma kode Rust biasa, dan modul `tests` itu cuma 
modul biasa juga. Seperti yang kita bahas di [“Paths untuk Merujuk ke sebuah 
Item di Pohon Modul”][paths], _items_ di dalam modul anak (child modules) 
bisa memakai _items_ di dalam modul leluhurnya (ancestor modules). Di pengujian 
ini, kita membawa semua _items_ dari induk modul `tests` ke dalam _scope_ 
dengan `use super::*`, lalu pengujian ini bisa memanggil `internal_adder`. 
Kalau kita ngerasa fungsi _private_ itu tidak perlu diuji, tidak ada hal di 
Rust yang bakal memaksa kita buat ngelakuin itu.

### Integration Tests

Di Rust, _integration tests_ (pengujian integrasi) itu sepenuhnya eksternal 
terhadap _library_ kita. Mereka memakai _library_ kita dengan cara yang persis 
sama seperti kode eksternal lainnya, yang artinya mereka cuma bisa memanggil 
fungsi-fungsi yang merupakan bagian dari API _public_ _library_ kita. Tujuannya 
adalah buat menguji apakah banyak bagian dari _library_ kita bekerja sama dengan 
benar. Unit-unit kode yang berjalan dengan benar saat sendirian bisa saja 
punya masalah pas digabungkan (integrated), jadi cakupan pengujian (_test coverage_) 
pada kode yang tergabung itu juga penting. Buat bikin _integration tests_, kita 
pertama-tama harus punya direktori _tests_.

#### Direktori _tests_

Kita membuat direktori _tests_ di tingkat teratas (top level) direktori project 
kita, bersebelahan dengan _src_. Cargo tahu dia harus mencari file-file 
_integration test_ di direktori ini. Kita kemudian bisa membuat sebanyak 
apa pun file pengujian yang kita mau, dan Cargo bakal men-compile tiap file 
tersebut sebagai sebuah _crate_ individu.

Mari kita bikin sebuah _integration test_. Dengan kode di Listing 11-12 yang 
masih ada di file _src/lib.rs_, buat sebuah direktori _tests_, dan bikin 
file baru bernama _tests/integration_test.rs_. Struktur direktori kita 
seharusnya jadi seperti ini:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Masukkan kode di Listing 11-13 ke dalam file _tests/integration_test.rs_.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="Sebuah _integration test_ dari sebuah fungsi di dalam _crate_ `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

Tiap file di dalam direktori _tests_ itu adalah sebuah _crate_ terpisah, 
jadi kita perlu membawa _library_ kita ke dalam _scope_ masing-masing _test crate_. 
Oleh karena itu kita menambahkan `use adder::add_two;` di paling atas kode 
kita, yang mana tidak kita perlukan di _unit tests_.

Kita tidak perlu menganotasi kode apa pun di _tests/integration_test.rs_ dengan 
`#[cfg(test)]`. Cargo memperlakukan direktori _tests_ secara spesial dan 
men-compile file-file di direktori ini cuma kalau kita menjalankan `cargo test`. 
Jalankan `cargo test` sekarang:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Tiga bagian dari output ini mencakup _unit tests_, _integration test_, dan 
_doc tests_. Perhatikan bahwa kalau ada pengujian di sebuah bagian yang gagal, 
bagian-bagian selanjutnya tidak bakal dijalankan. Misalnya, kalau ada _unit test_ 
yang gagal, tidak bakal ada output buat _integration tests_ atau _doc tests_ 
karena pengujian-pengujian itu baru bakal jalan kalau semua _unit tests_ sukses.

Bagian pertama untuk _unit tests_ itu sama dengan yang selama ini kita lihat: 
satu baris buat setiap _unit test_ (satu bernama `internal` yang kita tambahkan 
di Listing 11-12) dan kemudian satu baris ringkasan buat _unit tests_ tersebut.

Bagian _integration tests_ dimulai dengan baris `Running
tests/integration_test.rs`. Setelah itu, ada satu baris buat setiap fungsi 
pengujian di dalam _integration test_ itu dan satu baris ringkasan buat 
hasil dari _integration test_ tepat sebelum bagian `Doc-tests adder` dimulai.

Tiap file _integration test_ punya bagiannya masing-masing, jadi kalau kita 
nambahin lebih banyak file di direktori _tests_, bakal ada lebih banyak 
bagian _integration test_.

Kita masih bisa menjalankan fungsi _integration test_ tertentu dengan 
menentukan nama fungsi pengujian itu sebagai argumen di `cargo test`. Buat 
menjalankan semua pengujian di dalam file _integration test_ tertentu, kita 
bisa memakai argumen `--test` di `cargo test` diikuti dengan nama filenya:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Perintah ini hanya menjalankan pengujian-pengujian yang ada di file 
_tests/integration_test.rs_.

#### Submodul dalam Integration Tests

Saat kita menambahkan lebih banyak _integration tests_, kita mungkin mau membuat 
lebih banyak file di dalam direktori _tests_ untuk membantu mengaturnya; 
misalnya, kita bisa mengelompokkan fungsi-fungsi pengujian berdasarkan 
fungsionalitas yang lagi mereka uji. Seperti yang disebutkan sebelumnya, setiap file 
di direktori _tests_ di-compile sebagai _crate_-nya sendiri-sendiri secara terpisah, 
yang mana sangat berguna buat membuat _scopes_ yang terpisah demi meniru dengan 
lebih dekat gimana para _end users_ bakal memakai _crate_ kita. Namun, ini artinya 
file-file di direktori _tests_ tidak berbagi perilaku yang sama dengan 
file-file di direktori _src_, seperti yang kita pelajari di Bab 7 soal gimana 
memisahkan kode jadi berbagai modul dan file.

Perbedaan perilaku dari file-file di direktori _tests_ ini paling terlihat saat 
kita punya sekumpulan fungsi bantuan (helper functions) buat dipakai di berbagai 
file _integration test_ lalu kita mencoba mengikuti langkah-langkah di bagian 
[“Memisahkan Modul ke dalam Berbagai File”][separating-modules-into-files] 
di Bab 7 buat mengekstrak fungsi-fungsi itu ke modul bersama (common module). 
Misalnya, kalau kita membuat _tests/common.rs_ dan menaruh fungsi bernama 
`setup` di dalamnya, kita bisa menambahkan beberapa kode ke `setup` yang mau 
kita panggil dari beberapa fungsi pengujian yang ada di file-file pengujian 
yang berbeda:

<span class="filename">Nama file: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Saat kita menjalankan pengujiannya lagi, kita bakal melihat bagian baru di 
output pengujiannya buat file _common.rs_, meskipun file ini sama sekali tidak 
mengandung fungsi pengujian maupun kita memanggil fungsi `setup` dari mana 
pun:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Munculnya `common` di hasil pengujian dengan pesan `running 0 tests` yang 
ditampilkan buat modul itu bukanlah apa yang kita mau. Kita cuma mau membagikan 
sedikit kode dengan file-file _integration test_ yang lain. Buat mencegah 
`common` muncul di output pengujian, alih-alih membuat _tests/common.rs_, kita 
bakal membuat _tests/common/mod.rs_. Direktori project-nya sekarang bakal 
kelihatan seperti ini:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Ini adalah konvensi penamaan versi lama yang juga dipahami oleh Rust yang 
sudah kita sebutkan di bagian [“Alternate File Paths”][alt-paths] di Bab 7. 
Menamai file dengan cara ini memberi tahu Rust untuk tidak memperlakukan modul 
`common` sebagai file _integration test_. Saat kita memindahkan kode fungsi 
`setup` ke dalam _tests/common/mod.rs_ dan menghapus file _tests/common.rs_, 
bagian khusus buat modul ini di output pengujian tidak akan muncul lagi. File-file 
yang ada di dalam subdirektori dari direktori _tests_ tidak akan di-compile 
sebagai _crate_ yang terpisah maupun mendapatkan bagian khususnya sendiri di 
output pengujian.

Setelah kita membuat _tests/common/mod.rs_, kita bisa memakainya dari file 
_integration test_ mana pun layaknya sebuah modul. Berikut ini contoh memanggil 
fungsi `setup` dari pengujian `it_adds_two` yang ada di 
_tests/integration_test.rs_:

<span class="filename">Nama file: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Perhatikan bahwa deklarasi `mod common;` itu sama dengan deklarasi modul yang 
sudah kita demonstrasikan di Listing 7-21. Kemudian, di dalam fungsi 
pengujiannya, kita bisa memanggil fungsi `common::setup()`.

#### Integration Tests buat Binary Crates

Kalau project kita adalah sebuah _binary crate_ yang hanya mengandung satu file 
_src/main.rs_ dan tidak punya file _src/lib.rs_, kita tidak bisa bikin 
_integration tests_ di direktori _tests_ yang mencoba membawa fungsi-fungsi 
yang didefinisikan di _src/main.rs_ ke dalam _scope_ dengan memakai *statement* 
`use`. Cuma _library crates_ yang mengekspos fungsi-fungsi buat bisa dipakai 
oleh _crates_ lain; _binary crates_ itu dimaksudkan untuk berjalan sendiri.

Inilah salah satu alasan kenapa project-project Rust yang menyediakan sebuah 
_binary_ biasanya punya file _src/main.rs_ yang lumayan simpel dan langsung 
memanggil logika yang berada di dalam file _src/lib.rs_. Dengan struktur itu, 
_integration tests_ _bisa_ menguji _library crate_ dengan perintah `use` buat 
membuat fungsionalitas utamanya jadi tersedia buat dites. Kalau fungsionalitas 
utamanya bekerja dengan baik, sebagian kecil kode yang ada di file _src/main.rs_ 
itu bakal ikut bekerja dengan baik juga, dan bagian kecil kode itu tidak perlu 
diuji lagi secara terpisah.

## Ringkasan

Fitur pengujian Rust ngasih kita cara buat menentukan dengan spesifik gimana 
kode kita seharusnya berfungsi untuk memastikan kode itu terus berjalan 
sesuai yang kita harapkan, bahkan ketika kita membuat berbagai perubahan nanti. 
_Unit tests_ mencoba bagian-bagian yang berbeda dari sebuah _library_ 
secara terpisah dan bisa menguji detail implementasi _private_. _Integration 
tests_ mengecek apakah banyak bagian dari _library_ itu bekerja sama dengan benar, 
dan mereka memakai API _public_ dari _library_ itu buat menguji kodenya dengan 
cara yang persis sama dengan bagaimana kode eksternal bakal memakainya. 
Walaupun sistem tipe (type system) Rust dan aturan _ownership_ ngebantu mencegah 
beberapa jenis _bugs_, pengujian tetaplah penting buat ngurangin _logic bugs_ 
(kutu logika) yang berkaitan dengan bagaimana kode kita seharusnya berperilaku.

Mari kita gabungin ilmu yang udah kita pelajarin di bab ini dan di bab-bab 
sebelumnya buat ngerjain sebuah project bareng!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths