## Mempublikasikan Crate ke Crates.io

Kita sudah memakai _packages_ dari [crates.io](https://crates.io/) sebagai 
_dependencies_ buat project kita, tapi kita juga bisa nge-share kode kita 
sama orang lain dengan mempublikasikan _packages_ milik kita sendiri. _Registry_ 
_crate_ di [crates.io](https://crates.io/) mendistribusikan _source code_ dari 
_packages_ kita, jadi ia pada dasarnya meng-_host_ kode yang _open source_ 
(sumber terbuka).

Rust dan Cargo punya berbagai fitur yang bikin _package_ yang kita publikasikan 
jadi lebih gampang dicari dan dipakai orang. Kita bakal membahas beberapa fitur 
ini lalu menjelaskan gimana caranya mempublikasikan sebuah _package_.

### Menulis Komentar Dokumentasi yang Berguna

Mendokumentasikan _packages_ kita secara akurat bakal membantu para _user_ 
lain tahu gimana dan kapan mereka bisa memakainya, jadi sangat sepadan 
menghabiskan waktu buat nulis dokumentasi. Di Bab 3, kita sudah membahas gimana 
cara memberi komentar di kode Rust menggunakan dua garis miring, `//`. Rust 
juga punya jenis komentar khusus buat dokumentasi, yang lebih enak disebut 
_documentation comment_ (komentar dokumentasi), yang bakal men-_generate_ 
dokumentasi dalam format HTML. HTML ini menampilkan isi dari komentar dokumentasi 
buat item-item API _public_ yang ditujukan buat para programmer yang tertarik 
buat tahu gimana cara _memakai_ _crate_ kita, bukannya gimana _crate_ kita itu 
_diimplementasikan_.

Komentar dokumentasi memakai tiga garis miring, `///`, bukannya dua dan 
mendukung notasi Markdown buat memformat teksnya. Taruh komentar dokumentasi 
persis sebelum item yang lagi mereka dokumentasikan. Listing 14-1 menunjukkan 
komentar dokumentasi buat sebuah fungsi `add_one` di dalam sebuah _crate_ 
bernama `my_crate`.

<Listing number="14-1" file-name="src/lib.rs" caption="Sebuah komentar dokumentasi buat sebuah fungsi">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Di sini, kita memberikan deskripsi tentang apa yang dilakukan sama fungsi `add_one`, 
memulai sebuah bagian (_section_) dengan judul `Examples` (Contoh), dan kemudian 
menyediakan kode yang mendemonstrasikan gimana cara memakai fungsi `add_one`. Kita 
bisa men-_generate_ dokumentasi HTML dari komentar dokumentasi ini dengan 
menjalankan `cargo doc`. Perintah ini menjalankan _tool_ `rustdoc` yang 
didistribusikan bersama Rust dan menaruh dokumentasi HTML hasilnya ke dalam 
direktori _target/doc_.

Biar lebih praktis, menjalankan `cargo doc --open` bakal mem-build HTML buat 
dokumentasi dari _crate_ kita saat ini (beserta dokumentasi buat semua 
_dependencies_ _crate_ kita) lalu membuka hasilnya di web browser. Arahkan 
navigasi ke fungsi `add_one` dan kita bakal melihat gimana teks di komentar 
dokumentasi tersebut dirender, seperti yang ditunjukkan di Gambar 14-1.

<img alt="Dokumentasi HTML yang dirender untuk fungsi `add_one` dari `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Gambar 14-1: Dokumentasi HTML untuk fungsi `add_one`</span>

#### Bagian-bagian yang Sering Dipakai

Kita memakai _heading_ Markdown `# Examples` di Listing 14-1 buat membikin 
sebuah bagian di HTML-nya dengan judul “Examples.” Berikut ini beberapa bagian 
lain yang sering sekali dipakai oleh para pembuat _crate_ di dokumentasi mereka:

- **Panics**: Skenario-skenario di mana fungsi yang didokumentasikan ini bisa 
  mengalami _panic_. Kode pemanggil fungsi ini yang tidak mau programnya 
  mengalami _panic_ harus memastikan kalau mereka tidak memanggil fungsi ini 
  di situasi-situasi tersebut.
- **Errors**: Kalau fungsinya mengembalikan sebuah `Result`, mendeskripsikan 
  jenis-jenis error apa aja yang mungkin terjadi dan kondisi apa yang 
  mungkin menyebabkan error-error itu dikembalikan bisa sangat ngebantu 
  pemanggil agar mereka bisa nulis kode buat menangani berbagai jenis error 
  dengan cara yang berbeda-beda.
- **Safety**: Kalau fungsinya itu `unsafe` (tidak aman) buat dipanggil (kita 
  membahas soal _unsafety_ di Bab 20), harusnya ada bagian yang menjelaskan 
  kenapa fungsi itu tidak aman dan mencakup invariants (batasan) apa yang 
  diharapkan oleh fungsi tersebut buat dipatuhi sama pemanggilnya.

Kebanyakan komentar dokumentasi tidak perlu punya semua bagian ini, tapi ini 
adalah _checklist_ yang bagus buat ngingetin kita soal aspek-aspek kode kita 
yang bakal bikin _user_ tertarik buat tahu.

#### Komentar Dokumentasi Sebagai Tests (Pengujian)

Menambahkan blok-blok contoh kode di dalam komentar dokumentasi kita bisa 
membantu mendemonstrasikan gimana cara memakai _library_ kita, dan 
melakukannya punya keuntungan ekstra: menjalankan `cargo test` bakal 
menjalankan contoh kode di dokumentasi kita tersebut sebagai pengujian! 
Tidak ada yang lebih baik dari dokumentasi yang disertai contoh. Tapi tidak 
ada yang lebih parah dari contoh yang tidak jalan gara-gara kodenya udah 
diubah sejak dokumentasinya ditulis. Kalau kita menjalankan `cargo test` 
dengan dokumentasi buat fungsi `add_one` dari Listing 14-1, kita bakal melihat 
sebuah bagian di hasil pengujian yang kelihatan kayak gini:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Sekarang, kalau kita mengubah fungsinya atau contohnya sehingga `assert_eq!` 
di contoh tersebut mengalami _panic_, lalu menjalankan `cargo test` lagi, kita 
bakal melihat kalau _doc tests_ bakal menangkap bahwa contoh dan kodenya 
sudah tidak sinkron lagi!

#### Mengomentari Item Penampungnya (Contained Items)

Gaya komentar dokumentasi `//!` menambahkan dokumentasi ke item yang *menampung* 
komentar tersebut, bukannya ke item-item yang *mengikuti* komentarnya. Kita 
biasanya memakai jenis komentar dokumentasi ini di dalam file _crate root_ 
(_src/lib.rs_ secara konvensi) atau di dalam sebuah modul buat mendokumentasikan 
_crate_ atau modulnya secara keseluruhan.

Misalnya, buat menambahkan dokumentasi yang mendeskripsikan tujuan dari 
_crate_ `my_crate` yang mengandung fungsi `add_one`, kita bisa menambahkan 
komentar dokumentasi yang dimulai dengan `//!` ke bagian paling awal dari 
file _src/lib.rs_, seperti yang ditunjukkan di Listing 14-2.

<Listing number="14-2" file-name="src/lib.rs" caption="Dokumentasi untuk _crate_ `my_crate` secara keseluruhan">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Perhatikan bahwa tidak ada kode apa pun setelah baris terakhir yang dimulai 
dengan `//!`. Karena kita memulai komentarnya dengan `//!` bukannya `///`, 
kita mendokumentasikan item yang menampung komentar ini, bukannya item yang 
ada setelah komentar ini. Di kasus ini, item tersebut adalah file 
_src/lib.rs_, yang mana itu adalah _crate root_. Komentar-komentar ini 
mendeskripsikan _crate_-nya secara keseluruhan.

Pas kita menjalankan `cargo doc --open`, komentar-komentar ini bakal tampil 
di halaman depan dokumentasi untuk `my_crate` tepat di atas daftar item-item 
_public_ di _crate_ tersebut, seperti yang ditunjukkan di Gambar 14-2.

<img alt="Dokumentasi HTML yang dirender dengan komentar buat _crate_ secara keseluruhan" src="img/trpl14-02.png" class="center" />

<span class="caption">Gambar 14-2: Dokumentasi yang dirender buat `my_crate`, termasuk komentar yang mendeskripsikan _crate_ secara keseluruhan</span>

Komentar dokumentasi yang ditaruh di dalam item sangat berguna buat 
mendeskripsikan _crates_ dan juga _modules_ (modul). Pakailah mereka buat 
menjelaskan tujuan keseluruhan dari wadah (container)-nya demi membantu para 
_user_ kita memahami organisasi _crate_ tersebut.

### Mengekspor API Public yang Nyaman dengan `pub use`

Struktur dari API _public_ kita adalah pertimbangan besar saat mempublikasikan 
sebuah _crate_. Orang-orang yang memakai _crate_ kita itu kurang familier 
dengan strukturnya dibandingkan kita dan mungkin bakal kesusahan mencari 
bagian-bagian yang mau mereka pakai kalau _crate_ kita punya hierarki modul 
yang besar.

Di Bab 7, kita sudah membahas gimana cara membikin item jadi _public_ memakai 
keyword `pub`, dan gimana cara membawa item ke dalam _scope_ memakai keyword 
`use`. Namun, struktur yang menurut kita masuk akal saat kita lagi ngembangin 
sebuah _crate_ mungkin aja tidak terlalu nyaman buat para _user_ kita. Kita 
mungkin mau mengatur _structs_ kita di dalam sebuah hierarki yang terdiri dari 
beberapa tingkat, tapi nanti orang yang mau memakai tipe yang sudah kita 
definisikan jauh di kedalaman hierarki itu mungkin bakal kesulitan buat tahu kalau 
tipe tersebut ada. Mereka juga mungkin bakal jengkel karena harus mengetikkan 
`use my_crate::some_module::another_module::UsefulType;` bukannya sekadar 
`use my_crate::UsefulType;`.

Kabar baiknya adalah kalau struktur yang kita buat _tidak_ nyaman buat orang lain 
pakai dari _library_ mereka, kita tidak harus menata ulang organisasi 
internalnya: sebagai gantinya, kita bisa me-*re-export* (mengekspor ulang) 
item-item buat bikin sebuah struktur _public_ yang beda dari struktur _private_ 
kita dengan menggunakan `pub use`. *Re-exporting* mengambil sebuah item _public_ 
di satu lokasi lalu membikinnya jadi _public_ di lokasi lain, seolah-olah 
item tersebut didefinisikan di lokasi yang lain itu.

Misalnya, katakanlah kita bikin sebuah _library_ bernama `art` untuk memodelkan 
konsep-konsep kesenian. Di dalam _library_ ini ada dua modul: modul `kinds` yang 
berisi dua _enum_ bernama `PrimaryColor` dan `SecondaryColor`, serta modul 
`utils` yang berisi fungsi bernama `mix`, seperti yang ditunjukkan di Listing 14-3.

<Listing number="14-3" file-name="src/lib.rs" caption="Sebuah _library_ `art` yang punya item-item yang diatur ke dalam modul `kinds` dan `utils`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Gambar 14-3 menunjukkan seperti apa jadinya halaman depan dokumentasi untuk 
_crate_ ini yang di-generate oleh `cargo doc`.

<img alt="Dokumentasi yang dirender untuk _crate_ `art` yang mendaftarkan modul `kinds` dan `utils`" src="img/trpl14-03.png" class="center" />

<span class="caption">Gambar 14-3: Halaman depan dokumentasi untuk `art` yang mendaftarkan modul `kinds` dan `utils`</span>

Perhatikan bahwa tipe `PrimaryColor` dan `SecondaryColor` tidak terdaftar di 
halaman depan, fungsi `mix` juga tidak ada. Kita harus mengklik `kinds` dan 
`utils` buat melihat mereka.

_Crate_ lain yang bergantung pada _library_ ini bakal butuh *statements* 
`use` yang membawa item-item dari `art` ke dalam *scope*, dengan menentukan 
struktur modul yang saat ini didefinisikan. Listing 14-4 menunjukkan contoh 
sebuah _crate_ yang memakai item `PrimaryColor` dan `mix` dari _crate_ `art`.

<Listing number="14-4" file-name="src/main.rs" caption="Sebuah _crate_ yang memakai item dari _crate_ `art` dengan struktur internalnya yang terekspor">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Pembuat kode di Listing 14-4, yang memakai _crate_ `art`, harus mencari tahu 
kalau `PrimaryColor` ada di dalam modul `kinds` dan `mix` ada di dalam 
modul `utils`. Struktur modul dari _crate_ `art` ini lebih relevan buat para 
_developer_ yang mengerjakan _crate_ `art` tersebut dibanding buat orang yang 
memakainya. Struktur internalnya tidak memberikan informasi yang berguna buat 
seseorang yang lagi mencoba buat paham gimana cara memakai _crate_ `art` 
tersebut, melainkan malah bikin bingung karena _developer_ yang memakainya harus 
nyari tahu di mana harus mencari, dan harus mengetikkan nama-nama modulnya 
di *statement* `use`.

Buat menghilangkan organisasi internalnya dari API _public_, kita bisa 
memodifikasi kode _crate_ `art` di Listing 14-3 dengan menambahkan *statements* 
`pub use` buat me-*re-export* item-item itu di tingkat paling atas, seperti 
yang ditunjukkan di Listing 14-5.

<Listing number="14-5" file-name="src/lib.rs" caption="Menambahkan *statements* `pub use` buat me-*re-export* item">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

Dokumentasi API yang di-generate sama `cargo doc` buat _crate_ ini sekarang 
bakal mendaftarkan dan menaruh tautan ke *re-exports* tersebut di halaman depan, 
seperti yang ditunjukkan di Gambar 14-4, sehingga bikin tipe `PrimaryColor` dan 
`SecondaryColor` serta fungsi `mix` jadi lebih gampang dicari.

<img alt="Dokumentasi yang dirender untuk _crate_ `art` dengan hasil *re-exports* di halaman depan" src="img/trpl14-04.png" class="center" />

<span class="caption">Gambar 14-4: Halaman depan dokumentasi buat `art` yang mendaftarkan hasil *re-exports*</span>

Para pengguna _crate_ `art` masih bisa melihat dan memakai struktur internalnya 
dari Listing 14-3 seperti yang didemonstrasikan di Listing 14-4, atau mereka 
bisa memakai struktur yang lebih nyaman dari Listing 14-5, seperti yang 
ditunjukkan di Listing 14-6.

<Listing number="14-6" file-name="src/main.rs" caption="Sebuah program yang memakai item yang di-*re-export* dari _crate_ `art`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Di kasus di mana ada banyak modul yang bersarang (nested modules), me-*re-export* 
tipe di level teratas dengan `pub use` bisa bikin perbedaan yang sangat besar 
dalam pengalaman (_experience_) orang-orang yang memakai _crate_ tersebut. 
Kegunaan umum lain dari `pub use` adalah buat me-*re-export* definisi dari 
sebuah _dependency_ (dependensi) di _crate_ saat ini buat bikin definisi 
_crate_ itu jadi bagian dari API _public_ _crate_ kita.

Membikin struktur API _public_ yang berguna itu lebih ke arah seni ketimbang sains 
eksak, dan kita bisa melakukan iterasi buat nemuin API apa yang bekerja paling 
baik buat para _user_ kita. Memilih buat pakai `pub use` ngasih kita fleksibilitas 
dalam mengatur gimana struktur internal _crate_ kita dan melepaskan kaitan 
(decouples) antara struktur internal itu dari apa yang kita tampilkan ke para _user_ 
kita. Coba deh lihat beberapa kode dari _crates_ yang udah kita instal buat melihat 
apakah struktur internal mereka berbeda dari API _public_ mereka.

### Menyiapkan Akun Crates.io

Sebelum kita bisa mempublikasikan _crates_ apa pun, kita harus bikin akun dulu 
di [crates.io](https://crates.io/) dan dapetin API token. Buat melakukannya, 
kunjungi halaman depan di [crates.io](https://crates.io/) lalu _login_ pakai akun 
GitHub. (Akun GitHub saat ini adalah syarat wajibnya, tapi situsnya mungkin bakal 
mendukung cara lain buat bikin akun di masa depan.) Setelah kita _login_, kunjungi 
pengaturan akun kita di [https://crates.io/me/](https://crates.io/me/) lalu ambil 
kunci (key) API kita. Kemudian jalankan perintah `cargo login` dan *paste* 
kunci API kita pas diminta, kayak gini:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Perintah ini bakal ngasih tahu Cargo soal token API kita lalu menyimpannya secara 
lokal di _~/.cargo/credentials.toml_. Perhatikan bahwa token ini adalah 
sebuah *rahasia* (_secret_): jangan bagikan ke orang lain. Kalau kita 
membagikannya ke orang lain dengan alasan apa pun, kita harus menariknya (revoke) 
dan membuat token baru di [crates.io](https://crates.io/).

### Menambahkan Metadata ke Crate Baru

Katakanlah kita punya sebuah _crate_ yang mau kita publikasikan. Sebelum dipublish, 
kita harus menambahkan beberapa metadata di bagian `[package]` dari file _Cargo.toml_ 
milik _crate_ tersebut.

_Crate_ kita bakal butuh nama yang unik. Selama kita mengerjakan _crate_ secara 
lokal, kita bisa menamai _crate_ sesuka kita. Namun, nama _crate_ di 
[crates.io](https://crates.io/) itu dialokasikan berdasarkan siapa cepat dia 
dapat (first-come, first-served). Begitu sebuah nama _crate_ sudah diambil, 
tidak ada orang lain yang bisa mempublikasikan _crate_ dengan nama tersebut. 
Sebelum mencoba buat mempublikasikan sebuah _crate_, carilah nama yang pengen kita 
pakai. Kalau nama tersebut sudah terpakai, kita harus mencari nama lain lalu 
mengedit field `name` di dalam file _Cargo.toml_ di bawah bagian `[package]` buat 
memakai nama baru tersebut untuk publikasi, kayak gini:

<span class="filename">Nama file: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Meskipun kita sudah milih nama yang unik, saat kita menjalankan `cargo publish` 
buat mempublikasikan _crate_ di titik ini, kita bakal dapat peringatan dan lalu 
sebuah error:

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Ini menghasilkan error karena kita kelupaan beberapa informasi yang krusial: 
sebuah deskripsi (description) dan lisensi (license) diwajibkan supaya 
orang-orang bisa tahu apa yang dilakukan sama _crate_ kita dan di bawah 
ketentuan (terms) apa mereka bisa memakainya. Di _Cargo.toml_, tambahkan sebuah 
deskripsi yang hanya terdiri dari satu atau dua kalimat aja, karena itu bakal 
muncul bareng _crate_ kita di hasil pencarian. Buat field `license`, kita harus 
memberikan _nilai pengenal lisensi_ (license identifier value). 
[Software Package Data Exchange (SPDX) milik Linux Foundation][spdx] 
mendaftarkan pengenal yang bisa kita pakai buat nilai ini. Misalnya, buat 
menentukan kalau kita melisensikan _crate_ kita memakai Lisensi MIT, tambahkan 
pengenal `MIT`:

<span class="filename">Nama file: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Kalau kita mau memakai lisensi yang tidak muncul di SPDX, kita perlu menaruh teks 
dari lisensi tersebut di sebuah file, memasukkan file itu di project kita, lalu 
memakai `license-file` buat menentukan nama dari file tersebut sebagai ganti dari 
memakai key `license`.

Panduan soal lisensi mana yang pas buat project kita ada di luar cakupan buku ini. 
Banyak orang di komunitas Rust melisensikan project mereka pakai cara yang sama 
kayak Rust dengan memakai _dual license_ (lisensi ganda) `MIT OR Apache-2.0`. 
Praktik ini menunjukkan kalau kita juga bisa menentukan lebih dari satu 
pengenal lisensi yang dipisahkan oleh `OR` buat punya banyak lisensi di 
project kita.

Dengan nama yang unik, versi, deskripsi kita, dan lisensi yang sudah 
ditambahkan, file _Cargo.toml_ untuk project yang siap dipublish mungkin 
bakal kelihatan kayak gini:

<span class="filename">Nama file: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Dokumentasi Cargo](https://doc.rust-lang.org/cargo/) mendeskripsikan 
metadata lain yang bisa kita tentukan buat memastikan orang lain bisa lebih 
gampang menemukan dan memakai _crate_ kita.

### Mempublikasikan ke Crates.io

Sekarang karena kita udah bikin akun, nyimpan token API kita, memilih nama 
buat _crate_ kita, dan menentukan metadata yang dibutuhin, kita sudah siap 
buat melakukan publikasi! Mempublikasikan sebuah _crate_ mengunggah versi 
spesifiknya ke [crates.io](https://crates.io/) biar orang lain bisa pakai.

Hati-hati ya, karena mempublikasikan itu sifatnya _permanen_. Versi itu 
tidak akan pernah bisa ditimpa (overwritten), dan kodenya tidak bisa dihapus 
kecuali di situasi-situasi tertentu. Salah satu tujuan utama dari Crates.io 
adalah bertindak sebagai arsip kode yang permanen sehingga proses *build* 
dari semua project yang bergantung pada _crates_ dari [crates.io](https://crates.io/) 
bakal terus berfungsi. Mengizinkan penghapusan versi bakal bikin pemenuhan 
tujuan tersebut jadi mustahil. Namun, tidak ada batas buat seberapa banyak 
versi _crate_ yang bisa kita publikasikan.

Jalankan perintah `cargo publish` lagi. Kali ini harusnya udah berhasil:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Selamat! Kita sekarang sudah nge-share kode kita sama komunitas Rust, dan 
siapa pun bisa dengan gampang nambahin _crate_ kita sebagai *dependency* 
di project mereka.

### Mempublikasikan Versi Baru dari Crate yang Sudah Ada

Saat kita bikin perubahan di _crate_ kita dan udah siap buat ngerilis versi 
baru, kita cukup ganti nilai `version` yang ditentukan di file _Cargo.toml_ 
kita dan _publish_ ulang (republish). Pakai aturan [Semantic Versioning][semver] 
buat nentuin apa nomor versi selanjutnya yang paling pas, berdasarkan 
jenis perubahan yang sudah kita buat. Terus jalankan `cargo publish` buat 
mengunggah versi barunya.

<!-- Old link, do not remove -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>

### Melarang Penggunaan Versi Lama dari Crates.io dengan `cargo yank`

Meskipun kita tidak bisa menghapus versi lama dari sebuah _crate_, kita bisa 
mencegah project-project di masa depan buat menambahkan versi tersebut 
sebagai *dependency* baru. Hal ini berguna pas sebuah versi _crate_ ternyata 
rusak (broken) karena suatu alasan tertentu. Di situasi semacam itu, Cargo 
mendukung aksi "menyentak" (_yanking_) sebuah versi _crate_.

_Yanking_ sebuah versi mencegah project baru untuk bisa bergantung pada 
versi tersebut sekaligus tetap membiarkan semua project yang sudah ada 
yang bergantung pada versi itu buat terus berjalan. Pada dasarnya, aksi 
_yank_ berarti bahwa semua project yang sudah punya file _Cargo.lock_ tidak 
akan rusak, dan file _Cargo.lock_ apa pun yang di-generate di masa depan 
tidak bakal memakai versi yang di-_yank_ tersebut.

Buat menge-_yank_ sebuah versi _crate_, dari dalam direktori _crate_ yang 
tadinya sudah kita publikasikan, jalankan `cargo yank` dan tentukan versi mana 
yang mau kita _yank_. Misalnya, kalau kita sudah mempublikasikan sebuah _crate_ 
bernama `guessing_game` versi 1.0.1 dan kita mau menge-_yank_-nya, di dalam 
direktori project buat `guessing_game` kita bakal menjalankan:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Dengan menambahkan `--undo` ke dalam *command*-nya, kita juga bisa 
membatalkan aksi _yank_ (meng-_unyank_) lalu mengizinkan project-project buat 
mulai bergantung lagi pada sebuah versi:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Aksi _yank_ _sama sekali tidak_ menghapus kode apa pun. Ia tidak bisa, misalnya, 
menghapus data rahasia (*secrets*) yang tidak sengaja terunggah. Kalau hal 
seperti itu terjadi, kita harus langsung nge-reset rahasia tersebut.

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/