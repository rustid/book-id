## Cargo Workspaces

Di Bab 12, kita udah bikin sebuah _package_ yang isinya satu _binary crate_ 
sama satu _library crate_. Seiring berkembangnya project kita, kita mungkin 
menemukan bahwa _library crate_ kita terus jadi makin besar dan kita mau 
membagi _package_ kita lebih jauh lagi jadi beberapa _library crates_. 
Cargo menawarkan fitur bernama _workspaces_ (ruang kerja) yang bisa membantu 
mengelola beberapa _packages_ yang saling terkait yang dikembangkan secara 
beriringan (in tandem).

### Membuat Workspace

Sebuah _workspace_ adalah sekumpulan _packages_ yang berbagi _Cargo.lock_ dan 
direktori output yang sama. Mari kita bikin project yang memakai 
_workspace_—kita bakal pakai kode yang sepele biar kita bisa fokus ke 
struktur dari _workspace_ tersebut. Ada banyak cara buat menata struktur 
sebuah _workspace_, jadi kita cuma bakal nunjukin satu cara yang umum. 
Kita bakal punya sebuah _workspace_ yang berisi satu _binary_ dan dua 
_libraries_. Si _binary_, yang bakal menyediakan fungsionalitas utama, 
bakal bergantung pada (depend on) kedua _libraries_ itu. Satu _library_ 
bakal menyediakan fungsi `add_one` dan _library_ yang satunya lagi 
menyediakan fungsi `add_two`. Ketiga _crates_ ini bakal jadi bagian dari 
_workspace_ yang sama. Kita bakal memulainya dengan membuat direktori baru 
buat _workspace_ tersebut:

```console
$ mkdir add
$ cd add
```

Berikutnya, di dalam direktori _add_, kita bikin file _Cargo.toml_ yang 
bakal mengkonfigurasi seluruh _workspace_. File ini tidak bakal punya 
bagian `[package]`. Sebaliknya, file ini bakal diawali dengan bagian 
`[workspace]` yang bakal memungkinkan kita buat menambahkan anggota 
(members) ke dalam _workspace_. Kita juga sengaja menentukan buat memakai 
algoritma _resolver_ (pemecah) Cargo versi yang paling baru dan paling 
bagus di _workspace_ kita dengan menge-set nilai `resolver` ke `"3"`.

<span class="filename">Nama file: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Selanjutnya, kita bakal membikin _binary crate_ `adder` dengan menjalankan 
`cargo new` di dalam direktori _add_:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Menjalankan `cargo new` di dalam sebuah _workspace_ juga secara otomatis 
menambahkan _package_ yang baru dibuat itu ke dalam key `members` di 
definisi `[workspace]` yang ada di _Cargo.toml_ tingkat _workspace_, kayak gini:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Pada titik ini, kita bisa mem-build _workspace_ ini dengan menjalankan 
`cargo build`. File-file di direktori _add_ kita seharusnya kelihatan 
seperti ini:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

_Workspace_ ini cuma punya satu direktori _target_ di tingkat teratas 
(top level) tempat artefak hasil kompilasi bakal ditaruh; _package_ 
`adder` tidak punya direktori _target_-nya sendiri. Bahkan kalau pun kita 
menjalankan `cargo build` dari dalam direktori _adder_, artefak 
hasil kompilasinya bakal tetap berujung di _add/target_ bukannya di 
_add/adder/target_. Cargo menata struktur direktori _target_ di sebuah 
_workspace_ seperti ini karena _crates_ di dalam sebuah _workspace_ itu 
memang ditujukan buat bergantung satu sama lain. Kalau setiap _crate_ punya 
direktori _target_-nya sendiri, setiap _crate_ harus men-compile ulang 
setiap _crate_ lainnya di dalam _workspace_ itu buat menaruh artefaknya di 
direktori _target_-nya sendiri-sendiri. Dengan berbagi satu direktori 
_target_, _crates_ bisa menghindari kompilasi ulang yang tidak diperlukan.

### Membuat Package Kedua di dalam Workspace

Berikutnya, mari kita buat _member package_ (paket anggota) lain di dalam 
_workspace_ ini dan namakan dia `add_one`. _Generate_ sebuah _library crate_ 
baru bernama `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

File _Cargo.toml_ tingkat teratas sekarang bakal menyertakan _path_ _add_one_ 
ke dalam daftar `members`:

<span class="filename">Nama file: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Direktori _add_ kita seharusnya sekarang punya direktori dan file berikut ini:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Di dalam file _add_one/src/lib.rs_, mari kita tambahkan sebuah fungsi `add_one`:

<span class="filename">Nama file: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Sekarang kita bisa bikin _package_ `adder` dengan *binary* kita bergantung pada 
_package_ `add_one` yang punya _library_ kita. Pertama-tama, kita harus 
menambahkan *path dependency* pada `add_one` ke dalam _adder/Cargo.toml_.

<span class="filename">Nama file: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo tidak mengasumsikan kalau _crates_ di dalam sebuah _workspace_ bakal 
bergantung satu sama lain, jadi kita harus secara eksplisit mendefinisikan 
hubungan dependensi (ketergantungan) mereka.

Selanjutnya, mari kita pakai fungsi `add_one` (dari _crate_ `add_one`) di dalam 
_crate_ `adder`. Buka file _adder/src/main.rs_ dan ubah fungsi `main` buat 
memanggil fungsi `add_one`, seperti di Listing 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Memakai _library crate_ `add_one` dari dalam _crate_ `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Mari kita build _workspace_ ini dengan menjalankan `cargo build` di direktori 
tingkat teratas _add_!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Buat menjalankan _binary crate_ tersebut dari direktori _add_, kita bisa 
menentukan _package_ mana di dalam _workspace_ yang mau kita jalankan dengan 
memakai argumen `-p` beserta nama _package_-nya dengan `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Ini bakal menjalankan kode di _adder/src/main.rs_, yang mana bergantung pada 
_crate_ `add_one`.

#### Bergantung pada Package Eksternal di dalam Workspace

Perhatikan bahwa _workspace_ ini cuma punya satu file _Cargo.lock_ di tingkat 
teratas, bukannya punya file _Cargo.lock_ di setiap direktori _crate_. Ini 
memastikan kalau semua _crates_ memakai versi yang persis sama buat semua 
dependensinya. Kalau kita menambahkan _package_ `rand` ke dalam file 
_adder/Cargo.toml_ dan _add_one/Cargo.toml_, Cargo bakal me-resolve keduanya 
ke satu versi dari `rand` dan mencatat hal itu di dalam satu file _Cargo.lock_ 
tersebut. Membuat semua _crates_ di _workspace_ memakai dependensi yang sama 
berarti semua _crates_ tersebut bakal selalu kompatibel satu sama lain. 
Mari kita tambahkan _crate_ `rand` ke bagian `[dependencies]` di file 
_add_one/Cargo.toml_ supaya kita bisa memakai _crate_ `rand` di dalam _crate_ 
`add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Nama file: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Sekarang kita bisa menambahkan `use rand;` ke dalam file _add_one/src/lib.rs_, 
dan saat mem-build seluruh _workspace_ dengan menjalankan `cargo build` di 
direktori _add_ bakal ikut membawa dan men-compile _crate_ `rand`. Kita bakal 
dapat satu peringatan (warning) karena kita belum memakai `rand` yang sudah kita 
bawa ke dalam *scope* tersebut:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

File _Cargo.lock_ di tingkat teratas sekarang berisi informasi mengenai 
dependensi dari `add_one` terhadap `rand`. Namun, meskipun `rand` sudah dipakai 
di suatu tempat di dalam _workspace_, kita tidak bisa memakainya di _crates_ 
lainnya di _workspace_ ini kecuali kita menambahkan `rand` ke dalam file 
_Cargo.toml_ mereka juga. Misalnya, kalau kita menambahkan `use rand;` ke dalam 
file _adder/src/main.rs_ untuk _package_ `adder`, kita bakal dapat error:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Buat memperbaikinya, edit file _Cargo.toml_ untuk _package_ `adder` dan 
indikasikan kalau `rand` juga adalah sebuah dependensi buatnya. Mem-build 
_package_ `adder` bakal menambahkan `rand` ke dalam daftar dependensi untuk 
`adder` di dalam _Cargo.lock_, tapi tidak akan ada salinan tambahan dari 
`rand` yang bakal di-download. Cargo bakal memastikan kalau setiap _crate_ di 
setiap _package_ di dalam _workspace_ yang memakai _package_ `rand` bakal 
memakai versi yang persis sama selama mereka menentukan versi dari `rand` yang 
kompatibel, hal ini menghemat kapasitas penyimpanan kita dan memastikan kalau 
semua _crates_ di _workspace_ ini bakal kompatibel satu sama lain.

Kalau _crates_ di _workspace_ menentukan versi yang tidak kompatibel dari 
dependensi yang sama, Cargo bakal mencoba me-resolve masing-masing dari mereka, 
tapi tetap bakal berusaha me-resolve ke sesedikit mungkin versi.

#### Menambahkan Pengujian ke Workspace

Untuk peningkatan selanjutnya, mari kita tambahkan sebuah pengujian buat fungsi 
`add_one::add_one` di dalam _crate_ `add_one`:

<span class="filename">Nama file: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Sekarang jalankan `cargo test` di dalam direktori _add_ di tingkat teratas. 
Menjalankan `cargo test` di sebuah _workspace_ yang ditata seperti ini bakal 
menjalankan pengujian untuk semua _crates_ di dalam _workspace_ tersebut:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Bagian pertama dari outputnya menunjukkan kalau pengujian `it_works` di dalam 
_crate_ `add_one` itu sukses (passed). Bagian selanjutnya menunjukkan kalau ada 
nol pengujian yang ditemukan di dalam _crate_ `adder`, dan lalu bagian terakhir 
menunjukkan kalau ada nol pengujian dokumentasi yang ditemukan di dalam _crate_ 
`add_one`.

Kita juga bisa menjalankan pengujian buat satu _crate_ tertentu di dalam 
sebuah _workspace_ dari direktori tingkat teratas dengan memakai _flag_ `-p` 
dan menentukan nama dari _crate_ yang mau kita uji:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Output ini menunjukkan kalau `cargo test` cuma menjalankan pengujian untuk 
_crate_ `add_one` dan tidak menjalankan pengujian untuk _crate_ `adder`.

Kalau kita mempublikasikan _crates_ yang ada di dalam _workspace_ ke 
[crates.io](https://crates.io/), setiap _crate_ di dalam _workspace_ itu 
harus dipublikasikan secara terpisah. Sama seperti `cargo test`, kita bisa 
mempublikasikan satu _crate_ tertentu di dalam _workspace_ kita dengan memakai 
_flag_ `-p` dan menentukan nama dari _crate_ yang mau kita publikasikan.

Sebagai latihan tambahan, coba tambahkan _crate_ `add_two` ke dalam _workspace_ 
ini dengan cara yang sama seperti _crate_ `add_one`!

Seiring project kita bertambah besar, pertimbangkanlah buat memakai sebuah 
_workspace_: ini memungkinkan kita buat bekerja dengan komponen-komponen yang 
lebih kecil dan lebih gampang dipahami ketimbang bekerja dengan satu gumpalan 
kode (blob of code) yang super besar. Selain itu, menyimpan _crates_ di dalam 
sebuah _workspace_ bisa bikin koordinasi antar _crates_ jadi lebih mudah kalau 
mereka sering diubah secara bersamaan.
