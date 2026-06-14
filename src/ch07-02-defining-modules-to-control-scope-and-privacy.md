## Mendefinisikan Modul untuk Mengontrol Scope dan Privasi

Di bagian ini, kita bakal bahas modul dan bagian lain dari sistem modul, yaitu 
_paths_ (jalur), yang ngebolehin kita buat namain item; keyword `use` yang bawa 
sebuah _path_ ke dalem _scope_; dan keyword `pub` buat bikin item jadi _public_. 
Kita juga bakal bahas keyword `as`, _external packages_ (package eksternal), dan 
operator _glob_.

### Contekan (Cheat Sheet) Modul

Sebelum kita masuk ke detail soal modul dan _paths_, di sini kita nyediain 
referensi cepet soal gimana cara kerja modul, _paths_, keyword `use`, sama 
keyword `pub` di _compiler_, dan gimana kebanyakan _developer_ ngatur kode 
mereka. Kita bakal ngebahas contoh-contoh dari tiap aturan ini di sepanjang bab 
ini, tapi tempat ini cocok sekali buat dijadiin pengingat soal gimana modul itu 
jalan.

- **Mulai dari _crate root_**: Pas nge-compile sebuah crate, _compiler_ pertama 
  kali nyari di file _crate root_ (biasanya _src/lib.rs_ buat _library crate_ 
  atau _src/main.rs_ buat _binary crate_) buat nyari kode yang mau di-compile.
- **Mendeklarasikan modul**: Di file _crate root_, kita bisa mendeklarasikan 
  modul baru; katakanlah kita mendeklarasikan modul “garden” pake `mod garden;`. 
  _Compiler_ bakal nyari kode modul itu di tempat-tempat ini:
  - _Inline_, di dalem kurung kurawal yang nggantiin titik koma setelah 
    `mod garden`
  - Di file _src/garden.rs_
  - Di file _src/garden/mod.rs_
- **Mendeklarasikan submodul**: Di file mana pun selain _crate root_, kita bisa 
  mendeklarasikan submodul. Misalnya, kita mungkin mendeklarasikan 
  `mod vegetables;` di _src/garden.rs_. _Compiler_ bakal nyari kode submodul 
  itu di dalem direktori yang namanya sama kayak modul induk (parent)-nya di 
  tempat-tempat ini:
  - _Inline_, langsung setelah `mod vegetables`, di dalem kurung kurawal 
    bukannya titik koma
  - Di file _src/garden/vegetables.rs_
  - Di file _src/garden/vegetables/mod.rs_
- **Paths ke kode di modul**: Begitu sebuah modul jadi bagian dari crate kita, 
  kita bisa ngerujuk ke kode di modul itu dari mana pun di crate yang sama, 
  selama aturan privasinya ngebolehin, pake _path_ ke kodenya. Misalnya, tipe 
  `Asparagus` di modul vegetables dari garden bakal ditemuin di 
  `crate::garden::vegetables::Asparagus`.
- **Private vs. public**: Kode di dalem sebuah modul itu _private_ (privat) 
  dari modul induknya secara default. Buat bikin modul jadi _public_ (publik), 
  deklarasikan pake `pub mod` bukannya `mod`. Buat bikin item di dalem modul 
  _public_ ikutan jadi _public_ juga, pake `pub` sebelum deklarasinya.
- **Keyword `use`**: Di dalem sebuah _scope_, keyword `use` bikin *shortcut* 
  (jalan pintas) ke item buat ngurangin pengulangan _paths_ yang panjang. Di 
  _scope_ mana pun yang bisa ngerujuk ke 
  `crate::garden::vegetables::Asparagus`, kita bisa bikin *shortcut* pake 
  `use crate::garden::vegetables::Asparagus;` dan dari situ kita cuma perlu 
  nulis `Asparagus` buat pake tipe itu di _scope_ tersebut.

Di sini, kita bikin sebuah _binary crate_ namanya `backyard` yang nunjukin 
aturan-aturan ini. Direktori crate-nya, yang juga namanya `backyard`, punya file 
dan direktori ini:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

File _crate root_ di kasus ini adalah _src/main.rs_, dan isinya:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

Baris `pub mod garden;` ngasih tau _compiler_ buat masukin kode yang dia nemu 
di _src/garden.rs_, yaitu:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Di sini, `pub mod vegetables;` artinya kode di _src/garden/vegetables.rs_ juga 
dimasukin. Kode itu adalah:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Sekarang yuk kita masuk ke detail dari aturan-aturan ini dan demonstrasikan pas 
lagi dipake!

### Ngelempokin Kode yang Terkait di Modul

_Modul_ ngebolehin kita ngatur kode di dalem sebuah crate buat _readability_ 
(keterbacaan) dan biar gampang dipake ulang (_reuse_). Modul juga ngebolehin 
kita ngontrol privasi dari item karena kode di dalem sebuah modul itu _private_ 
secara default. Item _private_ adalah detail implementasi internal yang nggak 
tersedia buat dipake dari luar. Kita bisa milih buat bikin modul dan item di 
dalemnya jadi _public_, yang nge-ekspos mereka biar kode eksternal bisa pake dan 
bergantung pada mereka.

Sebagai contoh, yuk kita tulis sebuah _library crate_ yang nyediain fungsionalitas 
dari sebuah restoran. Kita bakal mendefinisikan signature dari fungsi-fungsinya 
tapi ngebiarin body-nya kosong buat fokus ke organisasi kodenya bukannya 
implementasi dari restorannya.

Di industri restoran, beberapa bagian dari restoran disebut _front of house_ 
(bagian depan) dan yang lainnya _back of house_ (bagian dapur). _Front of house_ 
itu tempat para pelanggan berada; ini nyakup tempat para _host_ ngarahin pelanggan 
ke tempat duduk, pelayan nerima pesanan dan pembayaran, dan bartender bikin 
minuman. _Back of house_ itu tempat para koki dan tukang masak kerja di dapur, 
pencuci piring bersih-bersih, dan manajer ngelakuin kerjaan administratif.

Buat menstruktur crate kita pake cara ini, kita bisa ngatur fungsi-fungsinya ke 
dalem modul yang bersarang. Bikin library baru namanya `restaurant` dengan 
jalanin `cargo new restaurant --lib`. Terus masukin kode di Listing 7-1 ke 
_src/lib.rs_ buat mendefinisikan beberapa modul dan signature fungsi; kode ini 
adalah bagian _front of house_.

<Listing number="7-1" file-name="src/lib.rs" caption="Sebuah modul `front_of_house` yang nyimpen modul lain yang terus nyimpen fungsi">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Kita mendefinisikan sebuah modul pake keyword `mod` diikuti sama nama modulnya 
(di kasus ini, `front_of_house`). Body dari modulnya ditaruh di dalem kurung 
kurawal. Di dalem modul, kita bisa naruh modul lain, kayak di kasus ini pake 
modul `hosting` sama `serving`. Modul juga bisa nampung definisi buat item lain, 
kayak struct, enum, konstanta, trait, dan kayak di Listing 7-1, fungsi.

Dengan pake modul, kita bisa ngelempokin definisi yang terkait dan ngasih nama 
kenapa mereka terkait. Programmer yang pake kode ini bisa navigasi kodenya 
berdasarkan kelompok-kelompoknya bukannya harus baca semua definisinya satu-satu, 
bikin lebih gampang buat nemuin definisi yang relevan buat mereka. Programmer 
yang nambahin fungsionalitas baru ke kode ini bakal tau ke mana harus naruh 
kodenya biar programnya tetep teratur.

Tadi, kita sempet nyebut kalau _src/main.rs_ sama _src/lib.rs_ itu disebut 
_crate roots_. Alasan dinamain gitu karena isi dari salah satu dari dua file 
ini ngebentuk modul namanya `crate` di _root_ dari struktur modul crate itu, 
yang dikenal sebagai _module tree_ (pohon modul).

Listing 7-2 nunjukin pohon modul buat struktur di Listing 7-1.

<Listing number="7-2" caption="Pohon modul buat kode di Listing 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Pohon ini nunjukin gimana beberapa modul bersarang di dalem modul lain; 
misalnya, `hosting` bersarang di dalem `front_of_house`. Pohonnya juga nunjukin 
kalau beberapa modul itu saling sodaraan (_siblings_), artinya mereka 
didefinisikan di modul yang sama; `hosting` sama `serving` itu sodaraan yang 
didefinisikan di dalem `front_of_house`. Kalau modul A ada di dalem modul B, 
kita bilang kalau modul A itu anaknya (_child_) modul B dan modul B itu induknya 
(_parent_) modul A. Perhatiin ya kalau seluruh pohon modul itu berakar di bawah 
modul implisit yang namanya `crate`.

Pohon modul mungkin ngingetin kita sama pohon direktori sistem file di komputer 
kita; ini perbandingan yang pas sekali! Kayak direktori di sistem file, kita 
pake modul buat ngatur kode kita. Dan kayak file di dalem direktori, kita perlu 
cara buat nemuin modul kita.
