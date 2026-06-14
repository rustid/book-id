## Advanced Traits (Traits Tingkat Lanjut)

Kita pertama kali ngebahas soal traits di [“Traits: Mendefinisikan Perilaku 
Bersama”][traits-defining-shared-behavior] di Bab 10, tapi kita tidak ngebahas 
detail-detail yang lebih mahirnya. Sekarang setelah kita tahu lebih banyak soal 
Rust, kita bisa masuk ke seluk beluk (nitty-gritty) dari traits ini.

<!-- Old link, do not remove -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>

### Associated Types

_Associated types_ (tipe terkait) menghubungkan sebuah _placeholder_ (tempat 
pengganti) tipe dengan sebuah trait sedemikian rupa sehingga definisi method 
dari trait tersebut bisa memakai tipe _placeholder_ ini di dalam _signatures_-nya. 
Si peng-implementasi (implementor) dari trait tersebut bakal menentukan tipe 
konkret yang bakal dipakai buat menggantikan tipe _placeholder_ itu buat 
implementasi khususnya. Dengan begitu, kita bisa mendefinisikan sebuah trait 
yang memakai tipe-tipe tertentu tanpa perlu tahu persis apa tipe-tipe tersebut 
sampai trait-nya benar-benar diimplementasikan.

Kita udah mendeskripsikan sebagian besar fitur-fitur tingkat lanjut di bab ini 
sebagai hal-hal yang jarang dibutuhkan. _Associated types_ ini letaknya ada di 
tengah-tengah: mereka dipakai lebih jarang ketimbang fitur-fitur yang dijelaskan 
di bagian lain buku ini tapi lebih sering dipakai ketimbang banyak fitur lain yang 
dibahas di bab ini.

Salah satu contoh dari trait yang punya _associated type_ adalah trait `Iterator` 
yang disediakan oleh _standard library_. _Associated type_-nya dinamakan `Item` 
dan dia bertindak sebagai pengganti buat tipe dari nilai-nilai yang lagi 
diiterasi sama tipe yang mengimplementasikan trait `Iterator` tersebut. 
Definisi dari trait `Iterator` ini ditunjukkan di Listing 20-13.

<Listing number="20-13" caption="Definisi dari trait `Iterator` yang punya sebuah _associated type_ `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

Tipe `Item` itu adalah sebuah _placeholder_, dan definisi method `next` nunjukin 
kalau dia bakal mengembalikan nilai-nilai bertipe `Option<Self::Item>`. 
Peng-implementasi dari trait `Iterator` bakal menentukan tipe konkret buat `Item`, 
dan method `next` bakal mengembalikan sebuah `Option` yang berisi sebuah nilai 
dari tipe konkret tersebut.

_Associated types_ mungkin kelihatannya mirip kayak konsep generik (generics), 
di mana generik itu memungkinkan kita buat mendefinisikan sebuah fungsi tanpa 
menentukan tipe-tipe apa yang bisa ditanganinya. Buat memeriksa perbedaan 
antara kedua konsep ini, kita bakal melihat sebuah implementasi dari trait 
`Iterator` pada sebuah tipe bernama `Counter` yang menentukan kalau tipe `Item`-nya 
adalah `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Sintaks ini kelihatannya bisa disamain sama sintaksnya generik. Terus kenapa 
tidak sekalian aja mendefinisikan trait `Iterator` pakai generik, seperti yang 
ditunjukkan di Listing 20-14?

<Listing number="20-14" caption="Sebuah definisi hipotetis (andaian) dari trait `Iterator` yang memakai generik">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

Perbedaannya adalah saat memakai generik, seperti di Listing 20-14, kita 
wajib menganotasi tipe-tipenya di setiap implementasinya; karena kita juga 
bisa mengimplementasikan `Iterator<String> for Counter` atau tipe apa pun 
lainnya, kita jadinya bisa punya banyak implementasi dari `Iterator` buat 
`Counter`. Dengan kata lain, saat sebuah trait punya parameter generik, 
dia bisa diimplementasikan buat satu tipe berkali-kali, asalkan tipe konkret 
dari parameter tipe generiknya selalu berbeda setiap kalinya. Saat kita memakai 
method `next` pada `Counter`, kita harus ngasih anotasi tipe buat nunjukin 
implementasi dari `Iterator` yang mana yang mau kita pakai.

Dengan _associated types_, kita tidak perlu menganotasi tipe-tipe karena kita 
tidak bisa mengimplementasikan sebuah trait pada satu tipe berkali-kali. Di 
Listing 20-13 yang mana definisinya memakai _associated types_, kita cuma bisa 
memilih tipe apa yang bakal jadi `Item` itu satu kali aja karena cuma boleh 
ada satu `impl Iterator for Counter`. Kita tidak perlu menyebutkan kalau kita 
mau sebuah iterator dari nilai-nilai `u32` di mana-mana di kode pas kita 
manggil `next` pada `Counter`.

_Associated types_ juga menjadi bagian dari kontrak si trait tersebut: para 
peng-implementasi dari trait tersebut wajib menyediakan sebuah tipe buat menggantikan 
_placeholder_ _associated type_-nya. _Associated types_ sering kali punya nama yang 
mendeskripsikan gimana tipe tersebut bakal dipakai, dan mendokumentasikan 
_associated type_ di dalam dokumentasi API adalah sebuah praktik yang baik.

### Parameter Tipe Generik Default (Bawaan) dan Operator Overloading

Saat kita memakai parameter tipe generik, kita bisa menentukan tipe konkret default 
(bawaan) buat tipe generik tersebut. Ini ngehapus kebutuhan bagi para peng-implementasi 
dari trait tersebut buat menentukan tipe konkret kalau tipe default-nya emang udah pas. 
Kita menentukan sebuah tipe default saat mendeklarasikan sebuah tipe generik dengan 
sintaks `<PlaceholderType=ConcreteType>`.

Satu contoh keren dari situasi di mana teknik ini sangat berguna adalah pada 
_operator overloading_ (penumpukan fungsi operator), di mana kita mengkustomisasi 
perilaku dari sebuah operator (seperti `+`) di situasi-situasi tertentu.

Rust tidak mengizinkan kita buat membikin operator kita sendiri atau melakukan 
_overload_ pada sembarang operator. Tapi kita bisa melakukan _overload_ pada operasi-
operasi dan trait-trait korespondennya yang terdaftar di `std::ops` dengan 
mengimplementasikan trait-trait yang berkaitan sama operator tersebut. Misalnya, 
di Listing 20-15 kita melakukan _overload_ pada operator `+` buat menjumlahkan 
dua instance `Point` bersama-sama. Kita ngelakuin ini dengan mengimplementasikan 
trait `Add` pada struct `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="Mengimplementasikan trait `Add` buat nge-_overload_ operator `+` untuk instance-instance `Point`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

Method `add` ngejumlahin nilai `x` dari dua instance `Point` dan nilai `y` dari dua 
instance `Point` buat membikin sebuah `Point` baru. Trait `Add` punya sebuah 
_associated type_ bernama `Output` yang menentukan tipe yang dikembalikan dari 
method `add`.

Tipe generik default di kode ini ada di dalam trait `Add`. Ini adalah definisinya:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Kode ini harusnya kelihatan familier pada umumnya: sebuah trait dengan satu method 
dan satu _associated type_. Bagian yang baru adalah `Rhs=Self`: sintaks ini disebut 
_default type parameters_ (parameter tipe default). Parameter tipe generik `Rhs` 
(singkatan dari “right-hand side” atau sisi kanan) mendefinisikan tipe dari 
parameter `rhs` di dalam method `add`. Kalau kita tidak menentukan sebuah tipe konkret 
buat `Rhs` saat kita mengimplementasikan trait `Add`, tipe dari `Rhs` bakal secara default 
menjadi `Self`, yang mana merupakan tipe di mana kita lagi mengimplementasikan trait 
`Add` tersebut.

Saat kita mengimplementasikan `Add` buat `Point`, kita memakai nilai default buat 
`Rhs` karena kita mau menjumlahkan dua instance `Point`. Mari kita lihat sebuah 
contoh pengimplementasian trait `Add` di mana kita mau mengkustomisasi tipe `Rhs` 
ketimbang memakai nilai default-nya.

Kita punya dua struct, `Millimeters` dan `Meters`, yang menampung nilai-nilai dalam 
satuan (units) yang berbeda. Pembungkusan tipis (_thin wrapping_) dari sebuah tipe 
yang udah ada ke dalam struct lain ini dikenal sebagai _newtype pattern_, yang mana bakal 
kita jelasin lebih detail di bagian [“Memakai Newtype Pattern Buat Mengimplementasikan 
External Traits”][newtype]. Kita pengen bisa menjumlahkan nilai-nilai dalam millimeter 
dengan nilai-nilai dalam meter lalu punya implementasi dari `Add` yang melakukan 
konversinya dengan benar. Kita bisa mengimplementasikan `Add` buat `Millimeters` 
dengan `Meters` sebagai si `Rhs`, seperti yang ditunjukkan di Listing 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="Mengimplementasikan trait `Add` pada `Millimeters` buat menjumlahkan `Millimeters` dan `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

Buat menjumlahkan `Millimeters` dan `Meters`, kita menentukan `impl Add<Meters>` 
buat menge-set nilai dari parameter tipe `Rhs` ketimbang memakai nilai default `Self`.

Kita bakal memakai parameter tipe default dalam dua cara utama:

1. Buat memperluas sebuah tipe tanpa merusak kode yang udah ada (existing code)
2. Buat memungkinkan adanya kustomisasi di kasus-kasus spesifik yang mana mayoritas 
   _user_ tidak bakal membutuhkannya

Trait `Add` di _standard library_ adalah contoh dari tujuan yang kedua: biasanya, 
kita bakal menjumlahkan dua tipe yang sama, tapi trait `Add` menyediakan kemampuan 
buat melakukan kustomisasi lebih dari itu. Memakai sebuah parameter tipe default di 
dalam definisi trait `Add` berarti kita tidak perlu menyebutkan parameter tambahan itu 
di sebagian besar waktunya. Dengan kata lain, sedikit _boilerplate code_ (kode 
berulang-ulang) tidak lagi diperlukan, ngebikin penggunaan trait-nya jadi lebih 
gampang.

Tujuan pertama itu mirip sama tujuan kedua tapi kebalikannya: kalau kita mau 
nambahin sebuah parameter tipe ke sebuah trait yang udah ada, kita bisa ngasih dia 
sebuah nilai default biar ekstensi fungsionalitas dari trait tersebut tidak 
merusak kode implementasi yang udah ada.

<!-- Old link, do not remove -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>

### Menghilangkan Ambiguitas (Disambiguating) di Antara Method-method yang Punya Nama yang Sama

Tidak ada aturan di Rust yang mencegah sebuah trait dari punya method dengan nama 
yang sama dengan method dari trait lain, dan Rust juga tidak mencegah kita buat 
mengimplementasikan kedua trait tersebut pada satu tipe. Sangat mungkin juga buat 
mengimplementasikan sebuah method secara langsung pada tipe tersebut dengan nama 
yang sama kayak nama-nama method dari trait-trait tadi.

Pas kita memanggil method-method yang punya nama yang sama ini, kita harus ngasih 
tahu Rust mana yang mau kita pakai. Coba perhatikan kode di Listing 20-17 di mana 
kita udah mendefinisikan dua trait, `Pilot` dan `Wizard`, yang mana dua-duanya punya 
sebuah method bernama `fly`. Kita lalu mengimplementasikan kedua trait tersebut pada 
sebuah tipe `Human` yang ternyata juga udah punya sebuah method bernama `fly` yang 
diimplementasikan langsung padanya. Masing-masing method `fly` ini ngelakuin hal yang 
berbeda.

<Listing number="20-17" file-name="src/main.rs" caption="Dua trait didefinisikan punya method `fly` dan diimplementasikan pada tipe `Human`, dan sebuah method `fly` juga diimplementasikan langsung pada `Human`.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Saat kita memanggil `fly` pada sebuah instance dari `Human`, _compiler_ secara default 
bakal memanggil method yang diimplementasikan secara langsung pada tipe tersebut, 
seperti yang ditunjukkan di Listing 20-18.

<Listing number="20-18" file-name="src/main.rs" caption="Memanggil `fly` pada sebuah instance dari `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Menjalankan kode ini bakal mencetak `*waving arms furiously*` (*melambaikan tangan 
dengan geram*), nunjukin kalau Rust memanggil method `fly` yang diimplementasikan pada 
`Human` secara langsung.

Buat memanggil method `fly` dari trait `Pilot` atau trait `Wizard`, kita harus 
memakai sintaks yang lebih eksplisit buat menyebutkan method `fly` yang mana yang kita 
maksud. Listing 20-19 mendemonstrasikan sintaks ini.

<Listing number="20-19" file-name="src/main.rs" caption="Menyebutkan method `fly` dari trait mana yang mau kita panggil">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Menyebutkan nama trait sebelum nama method-nya memperjelas bagi Rust implementasi 
dari `fly` yang mana yang mau kita panggil. Kita juga bisa menuliskan `Human::fly(&person)`, 
yang mana ini ekuivalen (sama) dengan `person.fly()` yang kita pakai di Listing 
20-19, tapi cara ini sedikit lebih panjang buat ditulis padahal kita tidak perlu 
menghilangkan ambiguitas apa pun di sana.

Menjalankan kode ini bakal mencetak yang berikut ini:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Karena method `fly` punya parameter `self`, kalau kita punya dua _tipe_ yang dua-
duanya mengimplementasikan satu _trait_, Rust bisa nyari tahu implementasi dari trait 
mana yang harus dipakai berdasarkan tipe dari `self`.

Namun, _associated functions_ (fungsi terkait) yang bukan methods tidak punya parameter 
`self`. Saat ada beberapa tipe atau trait yang mendefinisikan fungsi-fungsi non-method 
dengan nama fungsi yang sama, Rust tidak selalu tahu tipe mana yang kita maksud 
kecuali kalau kita memakai _fully qualified syntax_ (sintaks yang dikualifikasikan secara 
penuh). Misalnya, di Listing 20-20 kita membikin sebuah trait buat penampungan hewan 
(animal shelter) yang mau menamai semua anjing bayi (baby dogs) dengan nama Spot. 
Kita membikin sebuah trait `Animal` dengan sebuah fungsi _associated_ non-method 
bernama `baby_name`. Trait `Animal` ini diimplementasikan buat struct `Dog`, yang 
mana padanya kita juga menyediakan sebuah fungsi _associated_ non-method `baby_name` 
secara langsung.

<Listing number="20-20" file-name="src/main.rs" caption="Sebuah trait dengan fungsi _associated_ dan sebuah tipe dengan fungsi _associated_ yang namanya sama yang juga mengimplementasikan trait tersebut">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Kita mengimplementasikan kode buat menamai semua anak anjing dengan Spot di dalam 
fungsi _associated_ `baby_name` yang didefinisikan pada `Dog`. Tipe `Dog` juga 
mengimplementasikan trait `Animal`, yang mendeskripsikan karakteristik yang 
dimiliki semua hewan. Anjing bayi dipanggil _puppies_ (anak anjing), dan itu 
diekspresikan di dalam implementasi trait `Animal` pada `Dog` di dalam fungsi 
`baby_name` yang diasosiasikan dengan trait `Animal`.

Di `main`, kita memanggil fungsi `Dog::baby_name`, yang mana memanggil fungsi 
_associated_ yang didefinisikan pada `Dog` secara langsung. Kode ini mencetak 
yang berikut ini:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Output ini bukanlah apa yang kita inginkan. Kita pengen memanggil fungsi `baby_name` 
yang merupakan bagian dari trait `Animal` yang kita implementasikan pada `Dog` 
supaya kodenya mencetak `A baby dog is called a puppy`. Teknik menyebutkan nama 
trait yang kita pakai di Listing 20-19 tidak bisa membantu di sini; kalau kita ngubah 
`main` menjadi kode yang ada di Listing 20-21, kita bakal dapat error kompilasi.

<Listing number="20-21" file-name="src/main.rs" caption="Mencoba memanggil fungsi `baby_name` dari trait `Animal`, tapi Rust tidak tahu implementasi mana yang harus dipakai">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Karena `Animal::baby_name` tidak punya parameter `self`, dan bisa aja ada 
tipe-tipe lain yang mengimplementasikan trait `Animal`, Rust tidak bisa nyari tahu 
implementasi dari `Animal::baby_name` yang mana yang kita pengen. Kita bakal dapat 
error _compiler_ ini:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Buat menghilangkan ambiguitas ini dan ngasih tahu Rust kalau kita mau memakai 
implementasi `Animal` buat `Dog` ketimbang implementasi `Animal` buat tipe 
lain, kita perlu memakai _fully qualified syntax_. Listing 20-22 mendemonstrasikan 
gimana cara memakai _fully qualified syntax_.

<Listing number="20-22" file-name="src/main.rs" caption="Memakai _fully qualified syntax_ buat menyebutkan secara spesifik kalau kita mau manggil fungsi `baby_name` dari trait `Animal` seperti yang diimplementasikan pada `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Kita menyediakan Rust dengan sebuah anotasi tipe di dalam kurung sudut, yang 
mana mengindikasikan kalau kita mau memanggil method `baby_name` dari trait 
`Animal` seperti yang diimplementasikan pada `Dog` dengan mengatakan bahwa 
kita mau memperlakukan tipe `Dog` sebagai sebuah `Animal` buat pemanggilan fungsi 
ini. Kode ini sekarang bakal mencetak apa yang kita mau:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Secara umum, _fully qualified syntax_ didefinisikan kayak gini:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Buat _associated functions_ yang bukan method, tidak bakal ada yang namanya 
`receiver` (penerima): yang ada cuma daftar dari argumen-argumen lainnya aja. 
Kita bisa aja memakai _fully qualified syntax_ di mana-mana setiap kali kita manggil 
fungsi atau method. Namun, kita dibolehin buat ngilangin (_omit_) bagian apa pun dari 
sintaks ini yang mana Rust bisa cari tahu sendiri dari informasi lain di programnya. 
Kita cuma perlu memakai sintaks yang lebih panjang (_verbose_) ini di kasus-kasus di 
mana ada banyak implementasi yang memakai nama yang sama dan Rust butuh bantuan buat 
mengidentifikasi implementasi mana yang mau kita panggil.

<!-- Old link, do not remove -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Memakai Supertraits

Terkadang kita mungkin menulis sebuah definisi trait yang bergantung sama trait 
lain: supaya sebuah tipe bisa mengimplementasikan trait yang pertama, kita mau 
mewajibkan agar tipe tersebut juga mengimplementasikan trait yang kedua. Kita bakal 
melakukan ini supaya definisi trait kita bisa memanfaatkan item-item _associated_ 
(terkait) dari trait yang kedua tersebut. Trait yang diandalkan (relied on) oleh 
definisi trait kita itu disebut sebagai sebuah _supertrait_ dari trait kita.

Misalnya, katakanlah kita mau membikin sebuah trait `OutlinePrint` dengan sebuah 
method `outline_print` yang bakal mencetak sebuah nilai yang udah diformat sehingga 
dia dibingkai pakai tanda bintang (asterisks). Yakni, misalkan ada sebuah struct 
`Point` yang mengimplementasikan trait `Display` dari _standard library_ sehingga 
hasilnya `(x, y)`, maka saat kita memanggil `outline_print` pada instance `Point` 
yang punya nilai `1` buat `x` dan `3` buat `y`, dia seharusnya mencetak yang 
berikut ini:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Di dalam implementasi dari method `outline_print`, kita pengen memakai fungsionalitas 
dari trait `Display`. Oleh karena itu, kita perlu menentukan kalau trait 
`OutlinePrint` ini cuma bakal bekerja buat tipe-tipe yang juga mengimplementasikan 
`Display` dan menyediakan fungsionalitas yang dibutuhin sama `OutlinePrint`. Kita bisa 
melakukan itu di definisi trait-nya dengan menentukan `OutlinePrint: Display`. 
Teknik ini mirip sama menambahkan sebuah _trait bound_ ke dalam sebuah trait. 
Listing 20-23 menunjukkan sebuah implementasi dari trait `OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="Mengimplementasikan trait `OutlinePrint` yang mewajibkan fungsionalitas dari `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Karena kita udah menentukan kalau `OutlinePrint` mewajibkan adanya trait `Display`, 
kita jadi bisa memakai fungsi `to_string` yang mana otomatis diimplementasikan 
buat tipe apa pun yang mengimplementasikan `Display`. Kalau kita mencoba memakai 
`to_string` tanpa menambahkan titik dua dan menentukan trait `Display` setelah 
nama trait-nya, kita bakal dapat error yang bilang kalau tidak ada method bernama 
`to_string` yang ditemukan buat tipe `&Self` di dalam _scope_ saat ini.

Mari kita lihat apa yang terjadi saat kita mencoba mengimplementasikan `OutlinePrint` 
pada sebuah tipe yang tidak mengimplementasikan `Display`, kayak struct `Point` ini 
misalnya:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Kita dapat error yang bilang kalau `Display` itu diwajibkan tapi tidak 
diimplementasikan:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Buat memperbaiki ini, kita mengimplementasikan `Display` pada `Point` buat memenuhi 
(satisfy) batasan yang diwajibkan sama `OutlinePrint`, kayak gini:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Lalu setelahnya, mengimplementasikan trait `OutlinePrint` pada `Point` bakal berhasil 
di-compile dengan sukses, dan kita bisa memanggil `outline_print` pada sebuah instance 
`Point` buat nampilin dia di dalam sebuah bingkai yang isinya tanda bintang.

<!-- Old link, do not remove -->
<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>

### Memakai Newtype Pattern Buat Mengimplementasikan External Traits

Di [“Mengimplementasikan Trait pada Sebuah Tipe”][implementing-a-trait-on-a-type] 
di Bab 10, kita sempat nyebut soal *orphan rule* (aturan yatim piatu) yang 
menyatakan kalau kita cuma dibolehin buat mengimplementasikan sebuah trait pada 
sebuah tipe kalau entah trait tersebut atau tipe tersebut, atau bahkan keduanya, itu 
berada di (_local to_) _crate_ kita sendiri. Kita mungkin aja ngakalin (get around) 
batasan ini memakai _newtype pattern_, yang melibatkan pembuatan sebuah tipe baru di 
dalam sebuah _tuple struct_. (Kita udah ngebahas _tuple structs_ di [“Memakai Tuple Structs 
Tanpa Field Bernama buat Bikin Tipe yang Beda”][tuple-structs] di Bab 5.) _Tuple 
struct_ ini bakal punya satu field dan bertindak sebagai sebuah pembungkus tipis (_thin 
wrapper_) di sekitar tipe yang mana mau kita implementasikan trait padanya. Kemudian 
tipe pembungkus (wrapper type) itu jadinya sifatnya lokal buat _crate_ kita, dan kita 
bisa mengimplementasikan trait tersebut pada si pembungkus ini. _Newtype_ adalah 
sebuah istilah yang asalnya dari bahasa pemrograman Haskell. Tidak ada pinalti 
performa _runtime_ akibat memakai pola ini, dan tipe pembungkus ini bakal dihilangkan 
(elided) saat _compile time_.

Sebagai contoh, katakanlah kita mau mengimplementasikan `Display` pada `Vec<T>`, 
yang mana dilarang secara langsung sama si *orphan rule* karena baik trait `Display` 
maupun tipe `Vec<T>` itu didefinisikan di luar _crate_ kita. Kita bisa membikin sebuah 
struct `Wrapper` yang menampung sebuah instance dari `Vec<T>`; terus kita bisa 
mengimplementasikan `Display` pada `Wrapper` dan lalu memakai nilai `Vec<T>` 
tersebut, seperti yang ditunjukkan di Listing 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="Membikin sebuah tipe `Wrapper` di sekitar `Vec<String>` buat mengimplementasikan `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

Implementasi dari `Display` memakai `self.0` buat ngakses nilai `Vec<T>` yang 
ada di dalamnya karena `Wrapper` adalah sebuah _tuple struct_ dan `Vec<T>` adalah 
item yang ada di indeks 0 di dalam _tuple_ tersebut. Terus kita bisa deh memakai 
fungsionalitas dari trait `Display` ini pada `Wrapper`.

Kelemahan dari memakai teknik ini adalah bahwa `Wrapper` adalah sebuah tipe baru, 
jadi dia tidak punya method-method dari nilai yang dia tampung di dalamnya. 
Kita harus mengimplementasikan semua method-method dari `Vec<T>` secara langsung 
pada `Wrapper` sedemikian rupa sehingga method-method itu mendelegasikan panggilannya 
ke `self.0`, yang mana bakal memungkinkan kita buat memperlakukan `Wrapper` persis 
kayak sebuah `Vec<T>`. Kalau kita pengen tipe baru ini buat punya setiap method yang 
dipunyai sama tipe internalnya, mengimplementasikan trait `Deref` pada si `Wrapper` 
buat mengembalikan tipe internalnya bisa jadi sebuah solusi (kita udah ngebahas 
pengimplementasian trait `Deref` di [“Memperlakukan Smart Pointers seperti Referensi 
Biasa dengan `Deref`”][smart-pointer-deref] di Bab 15). Kalau kita tidak pengen 
tipe `Wrapper` ini buat punya semua method dari tipe internalnya—misalnya, buat 
ngebatesin perilaku dari si tipe `Wrapper` tersebut—maka kita harus mengimplementasikan 
hanya method-method yang emang kita mau aja secara manual.

_Newtype pattern_ ini juga berguna bahkan ketika tidak ada trait yang terlibat. Mari 
kita alihkan fokus kita lalu ngelihat beberapa cara tingkat lanjut (_advanced ways_) 
buat berinteraksi dengan sistem tipe Rust.

[newtype]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]: ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
