## Traits: Mendefinisikan Perilaku Bersama

Sebuah _trait_ mendefinisikan fungsionalitas yang dimiliki suatu tipe tertentu 
dan bisa dibagikan (di-_share_) dengan tipe lainnya. Kita bisa memakai _traits_ 
buat mendefinisikan perilaku bersama (_shared behavior_) secara abstrak. Kita 
bisa memakai _trait bounds_ buat menentukan kalau sebuah tipe generik bisa 
berupa tipe apa pun asalkan punya perilaku tertentu.

> Catatan: _Traits_ itu mirip sama fitur yang sering disebut _interfaces_ di 
> bahasa pemrograman lain, walaupun ada beberapa perbedaan.

### Mendefinisikan sebuah Trait

Perilaku dari sebuah tipe terdiri dari _methods_ yang bisa kita panggil pada tipe 
tersebut. Berbagai tipe bisa berbagi perilaku yang sama kalau kita bisa 
memanggil _methods_ yang sama pada semua tipe itu. Definisi _trait_ adalah 
cara buat mengelompokkan _method signatures_ (tanda tangan metode) bersama-sama 
untuk mendefinisikan sekumpulan perilaku yang dibutuhkan untuk mencapai suatu 
tujuan.

Misalnya, katakanlah kita punya beberapa _struct_ yang menampung berbagai 
macam dan jumlah teks: sebuah _struct_ `NewsArticle` yang menampung berita di 
lokasi tertentu dan sebuah `SocialPost` yang maksimal isinya 280 karakter 
beserta _metadata_ yang menunjukkan apakah itu postingan baru, di-_repost_, 
atau balasan buat postingan lain.

Kita mau bikin _library crate_ agregator media bernama `aggregator` yang bisa 
nampilin ringkasan data yang mungkin disimpan di dalam instance `NewsArticle` 
atau `SocialPost`. Untuk melakukan ini, kita butuh ringkasan dari tiap tipe, 
dan kita bakal minta ringkasan itu dengan memanggil _method_ `summarize` di 
tiap instance-nya. Listing 10-12 menunjukkan definisi _trait_ publik `Summary` 
yang mengekspresikan perilaku ini.

<Listing number="10-12" file-name="src/lib.rs" caption="Sebuah _trait_ `Summary` yang terdiri dari perilaku yang disediakan oleh _method_ `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Di sini, kita mendeklarasikan sebuah _trait_ memakai keyword `trait` lalu nama 
_trait_-nya, yang mana adalah `Summary` di kasus ini. Kita juga mendeklarasikan 
_trait_ ini sebagai `pub` supaya _crates_ yang bergantung pada _crate_ ini 
bisa memanfaatkan _trait_ ini juga, seperti yang bakal kita lihat di beberapa 
contoh nanti. Di dalam kurung kurawal, kita mendeklarasikan _method signatures_ 
yang menggambarkan perilaku tipe-tipe yang mengimplementasikan _trait_ ini, 
yang di kasus ini adalah `fn summarize(&self) -> String`.

Setelah _method signature_, bukannya ngasih implementasi di dalam kurung kurawal, 
kita memakai titik koma. Tiap tipe yang mengimplementasikan _trait_ ini harus 
menyediakan perilaku khususnya sendiri buat _body_ (isi) dari _method_ ini. 
_Compiler_ bakal memastikan kalau tipe apa pun yang punya _trait_ `Summary` 
bakal punya _method_ `summarize` yang didefinisikan dengan _signature_ yang 
persis sama kayak gini.

Sebuah _trait_ bisa punya banyak _method_ di dalamnya: _method signatures_ 
didaftarkan satu baris satu, dan tiap baris diakhiri dengan titik koma.

### Mengimplementasikan sebuah Trait pada suatu Tipe

Sekarang setelah kita mendefinisikan _signatures_ yang diinginkan dari _method_ 
_trait_ `Summary`, kita bisa mengimplementasikannya pada tipe-tipe di agregator 
media kita. Listing 10-13 menunjukkan implementasi _trait_ `Summary` pada 
_struct_ `NewsArticle` yang memakai judul (headline), penulis, dan lokasi buat 
bikin nilai kembalian dari `summarize`. Buat _struct_ `SocialPost`, kita 
mendefinisikan `summarize` sebagai username diikuti sama seluruh teks postingannya, 
dengan asumsi kalau konten postingan sudah dibatasi sampai 280 karakter.

<Listing number="10-13" file-name="src/lib.rs" caption="Mengimplementasikan _trait_ `Summary` pada tipe `NewsArticle` dan `SocialPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Mengimplementasikan _trait_ pada suatu tipe itu mirip dengan mengimplementasikan 
_method_ biasa. Bedanya adalah setelah `impl`, kita menaruh nama _trait_ yang 
mau kita implementasikan, lalu memakai keyword `for`, dan kemudian menentukan 
nama tipe di mana kita mau mengimplementasikan _trait_ tersebut. Di dalam blok 
`impl`, kita menaruh _method signatures_ yang sudah didefinisikan sama definisi 
_trait_-nya. Alih-alih menambahkan titik koma setelah setiap _signature_, kita 
memakai kurung kurawal dan mengisi isi _method_ dengan perilaku spesifik yang 
kita mau dari _method_ _trait_ tersebut untuk tipe khususnya.

Sekarang setelah _library_ ini mengimplementasikan _trait_ `Summary` pada 
`NewsArticle` dan `SocialPost`, pengguna dari _crate_ ini bisa memanggil 
_method_ dari _trait_ tersebut pada instance `NewsArticle` dan `SocialPost` 
dengan cara yang sama seperti memanggil _method_ biasa. Bedanya cuma si pengguna 
harus membawa _trait_ tersebut ke dalam _scope_ sekaligus membawa tipe-tipenya. 
Ini contoh gimana sebuah _binary crate_ bisa memakai _library crate_ 
`aggregator` kita:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Kode ini bakal mencetak `1 new post: horse_ebooks: of course, as you probably already
know, people`.

_Crates_ lain yang bergantung pada _crate_ `aggregator` juga bisa membawa _trait_ 
`Summary` ke dalam _scope_ untuk mengimplementasikan `Summary` di tipe mereka 
sendiri. Satu batasan yang perlu dicatat adalah kita cuma bisa mengimplementasikan 
sebuah _trait_ pada suatu tipe kalau setidaknya _trait_-nya atau tipenya, atau 
keduanya, berada di _crate_ kita sendiri (_local to our crate_). Misalnya, kita 
bisa mengimplementasikan _trait_ dari _standard library_ seperti `Display` 
pada tipe kustom seperti `SocialPost` sebagai bagian dari fungsionalitas 
_crate_ `aggregator` kita karena tipe `SocialPost` itu ada di _crate_ 
`aggregator` kita. Kita juga bisa mengimplementasikan `Summary` pada `Vec<T>` 
di _crate_ `aggregator` kita karena _trait_ `Summary` itu ada di _crate_ 
`aggregator` kita.

Tapi kita tidak bisa mengimplementasikan _traits_ eksternal pada tipe eksternal. 
Misalnya, kita tidak bisa mengimplementasikan _trait_ `Display` pada `Vec<T>` 
di dalam _crate_ `aggregator` kita karena `Display` dan `Vec<T>` dua-duanya 
didefinisikan di _standard library_ dan bukan bagian dari _crate_ `aggregator` 
kita. Batasan ini adalah bagian dari properti yang disebut _coherence_ 
(koherensi), dan lebih spesifik lagi disebut _orphan rule_ (aturan yatim piatu), 
dinamai begitu karena tipe induknya tidak ada. Aturan ini memastikan kalau kode 
milik orang lain tidak bisa merusak kode kita dan sebaliknya. Tanpa aturan ini, 
dua _crates_ bisa saja mengimplementasikan _trait_ yang sama untuk tipe yang sama, 
dan Rust tidak bakal tau implementasi mana yang harus dipakai.

### Implementasi Default

Kadang-kadang akan berguna kalau kita punya perilaku _default_ untuk beberapa 
atau semua _method_ di sebuah _trait_ daripada mewajibkan implementasi untuk 
semua _method_ di setiap tipe. Dengan begitu, saat kita mengimplementasikan 
_trait_ pada tipe tertentu, kita bisa tetap menyimpan atau menimpa (override) 
perilaku _default_ dari tiap _method_.

Di Listing 10-14, kita menentukan _string default_ buat _method_ `summarize` 
dari _trait_ `Summary` alih-alih cuma mendefinisikan _method signature_-nya, 
seperti yang kita lakukan di Listing 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="Mendefinisikan _trait_ `Summary` dengan implementasi _default_ buat _method_ `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

Untuk memakai implementasi _default_ buat meringkas instance dari `NewsArticle`, 
kita cukup menentukan blok `impl` yang kosong dengan 
`impl Summary for NewsArticle {}`.

Meskipun kita tidak lagi mendefinisikan _method_ `summarize` di `NewsArticle` 
secara langsung, kita sudah menyediakan implementasi _default_ dan menentukan 
kalau `NewsArticle` mengimplementasikan _trait_ `Summary`. Hasilnya, kita tetap 
bisa memanggil _method_ `summarize` pada instance `NewsArticle`, seperti ini:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Kode ini mencetak `New article available! (Read more...)`.

Membuat implementasi _default_ tidak mengharuskan kita untuk mengubah apa pun 
dari implementasi `Summary` pada `SocialPost` di Listing 10-13. Alasannya adalah 
sintaks buat menimpa implementasi _default_ itu persis sama kayak sintaks buat 
mengimplementasikan _method trait_ yang tidak punya implementasi _default_.

Implementasi _default_ bisa memanggil _method_ lain di _trait_ yang sama, 
bahkan kalau _method_ lain itu tidak punya implementasi _default_. Dengan cara 
ini, sebuah _trait_ bisa menyediakan banyak fungsionalitas berguna dan cuma 
mewajibkan si peng-implementasi buat menentukan sebagian kecil saja. Misalnya, 
kita bisa mendefinisikan _trait_ `Summary` agar punya _method_ `summarize_author` 
yang implementasinya wajib, lalu mendefinisikan _method_ `summarize` yang 
punya implementasi _default_ yang memanggil _method_ `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Untuk memakai versi `Summary` ini, kita cuma perlu mendefinisikan 
`summarize_author` pas kita mengimplementasikan _trait_-nya pada sebuah tipe:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Setelah kita mendefinisikan `summarize_author`, kita bisa memanggil `summarize` 
pada instance dari _struct_ `SocialPost`, dan implementasi _default_ dari 
`summarize` bakal memanggil definisi `summarize_author` yang sudah kita sediakan. 
Karena kita sudah mengimplementasikan `summarize_author`, _trait_ `Summary` 
sudah ngasih kita perilaku dari _method_ `summarize` tanpa mengharuskan kita 
menulis kode tambahan lagi. Berikut contoh pemakaiannya:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Kode ini mencetak `1 new post: (Read more from @horse_ebooks...)`.

Perhatikan kalau tidak mungkin untuk memanggil implementasi _default_ dari dalam 
implementasi yang lagi menimpa (_overriding_) _method_ yang sama.

### Traits sebagai Parameter

Sekarang setelah kita tahu cara mendefinisikan dan mengimplementasikan _traits_, 
kita bisa eksplor gimana cara memakai _traits_ buat mendefinisikan fungsi yang 
bisa menerima berbagai macam tipe. Kita bakal memakai _trait_ `Summary` yang 
sudah kita implementasikan di tipe `NewsArticle` dan `SocialPost` di Listing 
10-13 untuk mendefinisikan fungsi `notify` yang memanggil _method_ `summarize` 
pada parameter `item`-nya, yang bertipe apa pun selama tipe itu 
mengimplementasikan _trait_ `Summary`. Buat melakukannya, kita memakai sintaks 
`impl Trait`, seperti ini:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Alih-alih tipe konkret buat parameter `item`, kita memakai keyword `impl` 
bersama dengan nama _trait_-nya. Parameter ini bakal menerima tipe apa pun yang 
mengimplementasikan _trait_ yang ditentukan. Di dalam _body_ dari `notify`, 
kita bisa memanggil _method_ apa pun pada `item` yang asalnya dari _trait_ 
`Summary`, contohnya `summarize`. Kita bisa memanggil `notify` dan memberikan 
instance apa pun dari `NewsArticle` atau `SocialPost`. Kode yang memanggil 
fungsi tersebut dengan tipe lain, misalnya `String` atau `i32`, tidak bakal bisa 
di-compile karena tipe-tipe tersebut tidak mengimplementasikan `Summary`.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Sintaks Trait Bound

Sintaks `impl Trait` memang praktis buat kasus-kasus sederhana tapi sebenarnya 
itu cuma _syntax sugar_ (sintaks pemanis) dari bentuk yang lebih panjang yang 
dikenal sebagai _trait bound_; bentuknya kayak gini:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Bentuk yang lebih panjang ini ekuivalen (sama) dengan contoh di bagian 
sebelumnya, tapi lebih panjang (_verbose_). Kita menaruh _trait bounds_ 
bersamaan dengan deklarasi parameter tipe generik setelah tanda titik dua (`:`) 
dan di dalam kurung sudut.

Sintaks `impl Trait` itu nyaman dan bikin kode lebih ringkas buat kasus-kasus 
sederhana, sementara sintaks _trait bound_ yang lebih lengkap bisa 
mengekspresikan lebih banyak kerumitan buat kasus lain. Misalnya, kita bisa punya 
dua parameter yang dua-duanya mengimplementasikan `Summary`. Kalau pakai sintaks 
`impl Trait`, bentuknya bakal seperti ini:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Memakai `impl Trait` cocok kalau kita mau fungsi ini mengizinkan `item1` dan 
`item2` untuk punya tipe yang berbeda (asalkan dua-duanya mengimplementasikan 
`Summary`). Tapi, kalau kita mau memaksa kedua parameter tersebut buat punya 
tipe yang sama persis, kita harus memakai _trait bound_, seperti ini:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

Tipe generik `T` yang ditentukan sebagai tipe dari parameter `item1` dan `item2` 
membatasi fungsi ini sehingga tipe konkret dari nilai yang diberikan buat argumen 
`item1` dan `item2` itu harus sama.

#### Menentukan Beberapa Trait Bounds dengan Sintaks `+`

Kita juga bisa menentukan lebih dari satu _trait bound_. Katakanlah kita mau 
`notify` bisa memakai _display formatting_ di samping memanggil `summarize` 
pada `item`: kita tentukan di definisi `notify` kalau `item` harus 
mengimplementasikan `Display` sekaligus `Summary`. Kita bisa melakukannya 
menggunakan sintaks `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

Sintaks `+` ini juga valid buat dipakai sama _trait bounds_ pada tipe generik:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Dengan dua _trait bounds_ yang ditentukan, body dari `notify` bisa memanggil 
`summarize` dan juga memakai `{}` buat memformat `item`.

#### Trait Bounds yang Lebih Rapi pake Klausa `where`

Memakai terlalu banyak _trait bounds_ ada sisi negatifnya. Masing-masing generik 
punya _trait bounds_-nya sendiri, jadi fungsi dengan banyak parameter tipe generik 
bisa mengandung sangat banyak informasi _trait bound_ di antara nama fungsi dan 
daftar parameternya, yang mana bisa bikin _signature_ fungsinya jadi susah 
dibaca. Karena alasan ini, Rust punya sintaks alternatif buat menentukan 
_trait bounds_ di dalam sebuah klausa `where` setelah _signature_ fungsinya. 
Jadi, alih-alih nulis begini:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

kita bisa pakai klausa `where`, kayak gini:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

_Signature_ fungsinya jadi tidak terlalu penuh: nama fungsi, daftar parameter, 
dan tipe kembalian semuanya berdekatan, mirip seperti fungsi yang tidak punya 
banyak _trait bounds_.

### Mengembalikan Tipe yang Mengimplementasikan Traits

Kita juga bisa memakai sintaks `impl Trait` di posisi kembalian (return position) 
buat mengembalikan nilai dari suatu tipe yang mengimplementasikan sebuah _trait_, 
kayak gini:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Dengan memakai `impl Summary` buat tipe kembaliannya, kita menentukan kalau 
fungsi `returns_summarizable` bakal mengembalikan suatu tipe yang 
mengimplementasikan _trait_ `Summary` tanpa harus menyebut nama tipe konkretnya. 
Di kasus ini, `returns_summarizable` mengembalikan sebuah `SocialPost`, tapi 
kode yang memanggil fungsi ini tidak perlu tau soal itu.

Kemampuan buat menentukan tipe kembalian hanya berdasarkan _trait_ yang 
diimplementasikannya itu sangat berguna, apalagi di konteks _closures_ dan 
_iterators_, yang bakal kita bahas di Bab 13. _Closures_ dan _iterators_ bikin 
tipe-tipe yang cuma _compiler_ doang yang tau, atau tipe-tipe yang namanya 
kepanjangan buat ditulis. Sintaks `impl Trait` memudahkan kita menentukan secara 
ringkas kalau sebuah fungsi mengembalikan tipe tertentu yang mengimplementasikan 
_trait_ `Iterator` tanpa perlu nulis tipe yang kepanjangan.

Tapi, kita cuma bisa memakai `impl Trait` kalau kita mengembalikan satu tipe 
tunggal. Misalnya, kode ini, yang mengembalikan entah `NewsArticle` atau 
`SocialPost` dengan tipe kembalian `impl Summary`, tidak bakal bisa jalan:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Mengembalikan entah `NewsArticle` atau `SocialPost` itu tidak diperbolehkan 
karena adanya batasan dari gimana sintaks `impl Trait` diimplementasikan di 
dalam _compiler_. Kita bakal bahas gimana cara nulis fungsi dengan perilaku 
kayak gini di bagian [“Memakai Trait Objects yang Mengizinkan Nilai Dari 
Tipe yang Berbeda-beda”][using-trait-objects-that-allow-for-values-of-different-types] 
di Bab 18.

### Memakai Trait Bounds Buat Mengimplementasikan Method secara Bersyarat

Dengan memakai _trait bound_ bareng sebuah blok `impl` yang memakai parameter 
tipe generik, kita bisa mengimplementasikan _methods_ secara bersyarat 
(conditionally) buat tipe-tipe yang mengimplementasikan _traits_ yang ditentukan. 
Misalnya, tipe `Pair<T>` di Listing 10-15 selalu mengimplementasikan fungsi `new` 
buat mengembalikan instance baru dari `Pair<T>` (ingat dari bagian 
[“Mendefinisikan Methods”][methods] di Bab 5 bahwa `Self` adalah alias tipe buat 
tipe dari blok `impl`-nya, yang mana di kasus ini adalah `Pair<T>`). Tapi di 
blok `impl` berikutnya, `Pair<T>` cuma mengimplementasikan _method_ 
`cmp_display` kalau tipe di dalamnya `T` mengimplementasikan _trait_ 
`PartialOrd` (yang memungkinkan perbandingan) _dan_ _trait_ `Display` (yang 
memungkinkan untuk dicetak).

<Listing number="10-15" file-name="src/lib.rs" caption="Mengimplementasikan _method_ pada tipe generik secara bersyarat bergantung pada _trait bounds_">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Kita juga bisa mengimplementasikan secara bersyarat sebuah _trait_ buat tipe apa 
pun yang mengimplementasikan _trait_ lain. Implementasi sebuah _trait_ pada 
tipe apa pun yang memenuhi _trait bounds_-nya disebut sebagai _blanket 
implementations_ (implementasi selimut) dan ini sangat banyak dipakai di _standard 
library_ Rust. Misalnya, _standard library_ mengimplementasikan _trait_ `ToString` 
pada tipe apa pun yang mengimplementasikan _trait_ `Display`. Blok `impl` di 
_standard library_ keliatan mirip kayak kode ini:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Karena _standard library_ punya _blanket implementation_ ini, kita bisa 
memanggil _method_ `to_string` yang didefinisikan sama _trait_ `ToString` 
pada tipe apa pun yang mengimplementasikan _trait_ `Display`. Misalnya, kita bisa 
mengubah integer jadi nilai `String` miliknya seperti ini karena integer 
mengimplementasikan `Display`:

```rust
let s = 3.to_string();
```

_Blanket implementations_ ini biasanya muncul di dokumentasi buat suatu _trait_ di 
bagian “Implementors”.

_Traits_ dan _trait bounds_ memungkinkan kita nulis kode yang memakai parameter 
tipe generik untuk mengurangi duplikasi, sekaligus ngasih tau _compiler_ kalau 
kita maunya tipe generik itu punya perilaku tertentu. _Compiler_ kemudian bakal 
memakai informasi _trait bound_ tersebut buat mengecek apakah semua tipe 
konkret yang dipakai di kode kita sudah menyediakan perilaku yang benar. Di 
bahasa pemrograman yang _dynamically typed_ (tipe dinamis), kita bakal dapat error 
pas _runtime_ kalau kita memanggil _method_ di suatu tipe yang sebenarnya tidak 
punya definisi _method_ tersebut. Tapi Rust mindahin error-error ini ke fase 
_compile time_ jadi kita dipaksa buat membenarkan masalah ini sebelum kode kita 
bahkan bisa dijalankan. Sebagai bonus, kita tidak perlu nulis kode buat ngecek 
perilaku saat _runtime_ karena kita sudah mengeceknya pas _compile time_. Melakukan 
ini bakal meningkatkan performa tanpa harus mengorbankan fleksibilitas dari 
generik.

[using-trait-objects-that-allow-for-values-of-different-types]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
