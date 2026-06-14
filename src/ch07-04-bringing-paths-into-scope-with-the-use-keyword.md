## Bawa Paths ke Dalem Scope pake Keyword `use`

Harus nulis paths lengkap-lengkap buat manggil fungsi tuh rasanya kurang nyaman 
dan ngulang-ngulang terus. Di Listing 7-7, entah kita milih absolute atau 
relative path buat manggil fungsi `add_to_waitlist`, tiap kali kita mau manggil 
`add_to_waitlist` kita harus nulis `front_of_house` sama `hosting` juga. 
Untungnya, ada cara buat nyederhanain proses ini: kita bisa bikin *shortcut* 
(jalan pintas) ke sebuah path pake keyword `use` sekali aja, dan terus pake 
nama yang lebih pendek itu di mana-mana di dalem scope-nya.

Di Listing 7-11, kita bawa modul `crate::front_of_house::hosting` ke dalem scope 
dari fungsi `eat_at_restaurant` biar kita cuma perlu nentuin 
`hosting::add_to_waitlist` buat manggil fungsi `add_to_waitlist` di 
`eat_at_restaurant`.

<Listing number="7-11" file-name="src/lib.rs" caption="Bawa sebuah modul ke dalem scope pake `use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Nambahin `use` sama sebuah path di dalem sebuah scope itu mirip kayak bikin 
_symbolic link_ di sistem file. Dengan nambahin 
`use crate::front_of_house::hosting` di _crate root_, `hosting` sekarang jadi 
nama yang valid di scope itu, seolah-olah modul `hosting` itu didefinisikan di 
_crate root_. Path yang dibawa ke scope pake `use` juga bakal nge-cek privasi, 
sama kayak path lainnya.

Perhatiin ya kalau `use` cuma bikin *shortcut* buat scope tertentu di mana `use` 
itu dipanggil. Listing 7-12 mindahin fungsi `eat_at_restaurant` ke dalem anak 
modul baru namanya `customer`, yang mana itu beda scope dari statement `use`-nya, 
jadi body fungsinya nggak bakal bisa di-compile.

<Listing number="7-12" file-name="src/lib.rs" caption="Statement `use` cuma berlaku di scope tempat dia ditaruh.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

Error _compiler_ nunjukin kalau *shortcut*-nya udah nggak berlaku lagi di dalem 
modul `customer`:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Perhatiin ada warning juga yang bilang kalau `use`-nya nggak dipake di scope-nya! 
Buat benerin masalah ini, pindahin `use`-nya ke dalem modul `customer` juga, 
atau rujuk *shortcut* yang ada di modul induk pake `super::hosting` dari dalem 
anak modul `customer`.

### Bikin Paths `use` yang Idiomatik

Di Listing 7-11, kita mungkin mikir kenapa kita nulis 
`use crate::front_of_house::hosting` terus manggil `hosting::add_to_waitlist` di 
`eat_at_restaurant`, bukannya nulis path `use` sampe ke fungsi `add_to_waitlist` 
buat dapet hasil yang sama, kayak di Listing 7-13.

<Listing number="7-13" file-name="src/lib.rs" caption="Bawa fungsi `add_to_waitlist` ke dalem scope pake `use`, yang nggak idiomatik">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Walaupun Listing 7-11 sama Listing 7-13 ngelakuin tugas yang sama, Listing 7-11 
adalah cara yang idiomatik buat bawa sebuah fungsi ke dalem scope pake `use`. 
Bawa modul induk dari sebuah fungsi ke dalem scope pake `use` artinya kita 
harus nyebutin modul induknya pas manggil fungsi itu. Nyebutin modul induk pas 
manggil fungsi bikin jelas kalau fungsi itu nggak didefinisikan secara lokal, 
tapi tetep minimalisir pengulangan nulis absolute path-nya. Kode di Listing 7-13 
bikin nggak jelas di mana `add_to_waitlist` itu sebenernya didefinisikan.

Sebaliknya, pas kita bawa structs, enums, dan item lain pake `use`, itu 
idiomatik buat nyebutin full path-nya. Listing 7-14 nunjukin cara idiomatik 
buat bawa struct `HashMap` dari standard library ke dalem scope dari sebuah 
_binary crate_.

<Listing number="7-14" file-name="src/main.rs" caption="Bawa `HashMap` ke dalem scope pake cara yang idiomatik">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Nggak ada alesan kuat sih di balik idiom ini: ini cuma konvensi yang udah muncul, 
dan orang-orang udah kebiasa baca sama nulis kode Rust dengan cara kayak gini.

Pengecualian buat idiom ini adalah kalau kita bawa dua item yang namanya sama ke 
dalem scope pake statement `use`, karena Rust nggak ngebolehin itu. Listing 7-15 
nunjukin gimana cara bawa dua tipe `Result` yang namanya sama tapi modul induknya 
beda ke dalem scope, dan gimana cara merujuk ke mereka.

<Listing number="7-15" file-name="src/lib.rs" caption="Bawa dua tipe dengan nama yang sama ke dalem scope yang sama nuntut kita buat pake modul induk mereka.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Kayak yang bisa kita liat, pake modul induk bisa ngebedain kedua tipe `Result` 
ini. Kalau kita malah nulis `use std::fmt::Result` sama `use std::io::Result`, 
kita bakal punya dua tipe `Result` di scope yang sama, dan Rust nggak bakal tau 
mana yang kita maksud pas kita pake nama `Result`.

### Ngasih Nama Baru pake Keyword `as`

Ada solusi lain buat masalah bawa dua tipe dengan nama yang sama ke dalem scope 
yang sama pake `use`: setelah path, kita bisa nambahin `as` sama nama lokal baru, 
atau _alias_, buat tipe itu. Listing 7-16 nunjukin cara lain buat nulis kode 
di Listing 7-15 dengan nge-rename salah satu dari tipe `Result` itu pake `as`.

<Listing number="7-16" file-name="src/lib.rs" caption="Nge-rename sebuah tipe pas dibawa ke dalem scope pake keyword `as`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Di statement `use` yang kedua, kita milih nama baru `IoResult` buat tipe 
`std::io::Result`, yang nggak bakal bentrok sama `Result` dari `std::fmt` yang 
juga udah kita bawa ke dalem scope. Listing 7-15 sama Listing 7-16 sama-sama 
dianggap idiomatik, jadi milih yang mana itu terserah kita!

### Re-exporting Nama pake `pub use`

Pas kita bawa sebuah nama ke dalem scope pake keyword `use`, nama itu _private_ 
buat scope tempat kita nge-import dia. Buat ngebolehin kode di luar scope itu 
buat ngerujuk ke nama itu seolah-olah nama itu didefinisikan di scope tersebut, 
kita bisa gabungin `pub` sama `use`. Teknik ini namanya _re-exporting_ karena 
kita bawa item ke dalem scope tapi juga nge-ekspos item itu biar orang lain 
bisa bawa item itu ke scope mereka.

Listing 7-17 nunjukin kode di Listing 7-11 dengan `use` di modul _root_ diganti 
jadi `pub use`.

<Listing number="7-17" file-name="src/lib.rs" caption="Bikin sebuah nama bisa dipake dari scope baru oleh kode lain pake `pub use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Sebelum perubahan ini, kode eksternal harus manggil fungsi `add_to_waitlist` 
pake path `restaurant::front_of_house::hosting::add_to_waitlist()`, yang juga 
bakal mewajibkan modul `front_of_house` buat ditandain sebagai `pub`. Sekarang, 
karena `pub use` ini udah nge-_re-export_ modul `hosting` dari modul _root_, 
kode eksternal bisa pake path `restaurant::hosting::add_to_waitlist()` sebagai 
gantinya.

_Re-exporting_ berguna pas struktur internal kode kita itu beda dari gimana 
programmer yang manggil kode kita bakal mikirin soal domain-nya. Misalnya, di 
analogi restoran ini, orang yang jalanin restorannya mikirnya “front of house” 
sama “back of house.” Tapi pelanggan yang dateng ke restoran mungkin nggak bakal 
mikirin bagian-bagian restoran pake istilah-istilah itu. Pake `pub use`, kita 
bisa nulis kode kita pake satu struktur tapi nge-ekspos struktur yang beda. 
Ngelakuin ini bikin library kita terorganisir dengan baik buat programmer yang 
ngerjain library-nya dan juga buat programmer yang manggil library-nya. Kita 
bakal liat contoh lain dari `pub use` dan gimana pengaruhnya ke dokumentasi crate 
kita di [“Ngekspor API Public yang Nyaman pake `pub use`”][ch14-pub-use] di Bab 14.

### Pake Package Eksternal

Di Bab 2, kita bikin project game tebak angka yang pake package eksternal 
namanya `rand` buat dapet angka random. Buat pake `rand` di project kita, kita 
nambahin baris ini ke _Cargo.toml_:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

Nambahin `rand` sebagai _dependency_ (dependensi) di _Cargo.toml_ ngasih tau 
Cargo buat download package `rand` sama dependensinya dari [crates.io](https://crates.io/) 
terus nyediain `rand` buat project kita.

Terus, buat bawa definisi `rand` ke dalem scope package kita, kita nambahin 
baris `use` yang dimulai dari nama crate-nya, `rand`, dan nge-list item-item 
yang mau kita bawa ke dalem scope. Inget kan di [“Menghasilkan Angka Random”][rand] 
di Bab 2, kita bawa trait `Rng` ke dalem scope terus manggil fungsi 
`rand::thread_rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Anggota komunitas Rust udah bikin sangat banyak package yang tersedia di 
[crates.io](https://crates.io/), dan masukin salah satu dari mereka ke package 
kita itu ngelibatin langkah-langkah yang sama: daftarin mereka di file 
_Cargo.toml_ package kita terus pake `use` buat bawa item dari crate mereka ke 
dalem scope.

Perhatiin ya kalau _standard library_ `std` itu juga sebuah crate yang eksternal 
buat package kita. Karena standard library udah dipaket bareng bahasa Rust, kita 
nggak perlu ngubah _Cargo.toml_ buat masukin `std`. Tapi kita tetep perlu 
ngerujuk ke dia pake `use` buat bawa item-item dari sana ke dalem scope package 
kita. Misalnya, buat `HashMap` kita bakal pake baris ini:

```rust
use std::collections::HashMap;
```

Ini adalah absolute path yang dimulai dari `std`, nama dari crate standard 
library.

### Pake Nested Paths Buat Ngerapihin Daftar `use` yang Panjang

Kalau kita pake banyak item yang didefinisikan di crate yang sama atau modul 
yang sama, nulisin tiap item di barisnya sendiri-sendiri bakal menuhin tempat 
secara vertikal di file kita. Misalnya, dua statement `use` ini yang kita pake 
di game tebak angka di Listing 2-4 bawa item-item dari `std` ke dalem scope:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Sebagai gantinya, kita bisa pake _nested paths_ (path bersarang) buat bawa item-
item yang sama ke dalem scope dalam satu baris. Kita lakuin ini dengan nulisin 
bagian yang sama dari path-nya, diikuti sama dua titik dua (`::`), terus kurung 
kurawal di sekitar list dari bagian-bagian path yang beda, kayak yang ditunjukin 
di Listing 7-18.

<Listing number="7-18" file-name="src/main.rs" caption="Nentuin nested path buat bawa beberapa item dengan awalan yang sama ke dalem scope">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Di program yang lebih gede, bawa banyak item ke dalem scope dari crate atau modul 
yang sama pake nested paths bisa ngurangin sekali jumlah statement `use` 
terpisah yang dibutuhin!

Kita bisa pake nested path di level mana pun di dalem sebuah path, yang berguna 
sekali pas ngegabungin dua statement `use` yang nge-share _subpath_. Misalnya, 
Listing 7-19 nunjukin dua statement `use`: satu yang bawa `std::io` ke dalem 
scope dan satu lagi yang bawa `std::io::Write` ke dalem scope.

<Listing number="7-19" file-name="src/lib.rs" caption="Dua statement `use` di mana salah satunya adalah subpath dari yang lain">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

Bagian yang sama dari kedua path ini adalah `std::io`, dan itu adalah path 
lengkap pertama. Buat ngegabungin dua path ini jadi satu statement `use`, kita 
bisa pake `self` di dalem nested path, kayak yang ditunjukin di Listing 7-20.

<Listing number="7-20" file-name="src/lib.rs" caption="Ngegabungin path di Listing 7-19 jadi satu statement `use`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Baris ini bawa `std::io` sama `std::io::Write` ke dalem scope.

### Operator Glob

Kalau kita mau bawa _semua_ item public yang didefinisikan di sebuah path ke 
dalem scope, kita bisa nulis path itu diikuti sama operator _glob_ `*`:

```rust
use std::collections::*;
```

Statement `use` ini bawa semua item public yang didefinisikan di 
`std::collections` ke dalem scope saat ini. Hati-hati ya pas pake operator 
glob! Glob bisa bikin kita lebih susah buat tau nama apa aja yang ada di dalem 
scope dan di mana nama yang dipake di program kita itu didefinisikan. Selain itu, 
kalau dependensinya ngerubah definisi mereka, apa yang kita import juga ikut 
berubah, yang bisa memicu error _compiler_ pas kita upgrade dependensi kalau 
dependensinya nambahin definisi dengan nama yang sama kayak definisi punya kita 
di scope yang sama, misalnya.

Operator glob sering sekali dipake pas lagi _testing_ buat bawa semua hal yang 
mau di-test ke dalem modul `tests`; kita bakal bahas itu di [“Gimana Cara Nulis 
Test”][writing-tests] di Bab 11. Operator glob juga kadang dipake sebagai bagian 
dari pola _prelude_: liat [dokumentasi standard library](../std/prelude/index.html#other-preludes) 
buat info lebih lanjut soal pola itu.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
