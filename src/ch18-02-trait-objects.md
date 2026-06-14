## Memakai Trait Objects Buat Mengabstraksi Perilaku Bersama

<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

Di Bab 8, kita sempat nyebut kalau salah satu batasan dari *vectors* adalah 
bahwa mereka cuma bisa nyimpan elemen dari satu tipe aja. Kita membikin sebuah 
solusi buat ngakalin hal ini (workaround) di Listing 8-9 di mana kita 
mendefinisikan enum `SpreadsheetCell` yang punya varian-varian buat menampung 
*integers*, *floats*, dan teks. Ini berarti kita bisa nyimpan tipe data yang 
berbeda-beda di setiap sel tapi tetap punya sebuah vector yang merepresentasikan 
satu baris sel. Ini adalah solusi yang sangat bagus kalau item-item yang mau kita 
tukar-tukar (interchangeable) itu merupakan serangkaian tipe tetap yang udah 
kita tahu pas kode kita di-compile.

Namun, kadang-kadang kita pengen supaya _user_ _library_ kita bisa memperluas 
serangkaian tipe yang valid di suatu situasi tertentu. Buat nunjukin gimana 
kita bisa mencapai hal ini, kita bakal membikin contoh alat *graphical user 
interface* (GUI) yang beriterasi ngelewatin sebuah daftar (list) item, 
lalu memanggil method `draw` pada masing-masing item tersebut buat menggambarnya 
ke layar—sebuah teknik yang umum buat alat-alat GUI. Kita bakal membikin sebuah 
_library crate_ bernama `gui` yang mengandung struktur dari _library_ GUI 
tersebut. _Crate_ ini mungkin bakal nyertakan beberapa tipe buat dipakai 
orang-orang, kayak `Button` atau `TextField`. Selain itu, para pengguna `gui` 
bakal pengen bikin tipe-tipe mereka sendiri yang bisa digambar: contohnya, 
satu programmer mungkin bakal nambahin `Image` dan yang lain mungkin bakal nambahin 
`SelectBox`.

Pas lagi nulis _library_-nya, kita tidak bisa tahu dan tidak bisa mendefinisikan 
semua tipe yang mungkin pengen dibikin sama programmer lain nantinya. Tapi kita 
tahu kalau `gui` itu perlu melacak (keep track of) banyak nilai dari 
tipe yang berbeda-beda, dan dia perlu manggil method `draw` pada setiap 
nilai yang bertipe beda-beda ini. Dia tidak perlu tahu persis apa yang bakal 
terjadi pas kita manggil method `draw` tersebut, dia cuma butuh tahu kalau nilai 
itu punya method tersebut dan tersedia buat kita panggil.

Buat ngelakuin ini di bahasa pemrograman yang punya _inheritance_ (pewarisan), 
kita mungkin bakal mendefinisikan sebuah *class* bernama `Component` yang 
punya sebuah method bernama `draw` padanya. Kelas-kelas lainnya, kayak `Button`, 
`Image`, dan `SelectBox`, bakal mewarisi (inherit) dari `Component` dan dengan 
gitu bakal mewarisi juga method `draw`-nya. Mereka masing-masing bisa menimpa 
(override) method `draw` tersebut buat mendefinisikan perilaku kustom mereka 
sendiri, tapi *framework*-nya bisa memperlakukan semua tipe-tipe tersebut 
seolah-olah mereka adalah instance dari `Component` dan memanggil `draw` pada 
mereka. Tapi karena Rust tidak punya _inheritance_, kita butuh cara lain buat 
menstrukturkan _library_ `gui` tersebut supaya _user_ bisa bikin tipe-tipe 
baru yang kompatibel sama _library_ kita.

### Mendefinisikan sebuah Trait untuk Perilaku Umum (Common Behavior)

Buat mengimplementasikan perilaku yang kita pengen ada di `gui`, kita bakal 
mendefinisikan sebuah trait bernama `Draw` yang bakal punya satu method 
bernama `draw`. Terus kita bisa mendefinisikan sebuah vector yang menerima 
sebuah _trait object_ (objek trait). Sebuah _trait object_ menunjuk ke sebuah 
instance dari suatu tipe yang mengimplementasikan trait yang sudah kita 
tentukan, dan juga menunjuk ke sebuah tabel yang dipakai buat nyari _trait methods_ 
pada tipe tersebut saat _runtime_. Kita membikin _trait object_ dengan 
menentukan semacam _pointer_, kayak referensi `&` atau _smart pointer_ 
`Box<T>`, lalu keyword `dyn`, dan lalu menentukan trait yang relevan. (Kita 
bakal ngomongin alasan kenapa _trait objects_ harus memakai _pointer_ di 
[“Tipe-tipe yang Berukuran Dinamis dan Trait `Sized`”][dynamically-sized] 
di Bab 20.) Kita bisa memakai _trait objects_ buat menggantikan tipe generik 
atau tipe konkret. Di mana pun kita memakai sebuah _trait object_, sistem tipe 
Rust bakal memastikan saat _compile time_ bahwa nilai apa pun yang dipakai di 
konteks tersebut bakal mengimplementasikan trait dari _trait object_ itu. 
Akibatnya, kita tidak perlu tahu semua tipe yang mungkin ada saat _compile time_.

Kita udah sempat nyebut kalau di Rust, kita menahan diri buat tidak nyebut 
_structs_ dan _enums_ sebagai "objek" buat ngebedain mereka dari objek di bahasa 
pemrograman lain. Di sebuah _struct_ atau _enum_, data di _fields_ struct-nya 
dan perilakunya di blok `impl` itu dipisahkan, sedangkan di bahasa pemrograman lain, 
data dan perilaku yang digabungkan jadi satu konsep itu yang sering kali 
dilabeli sebagai sebuah objek. _Trait objects_ berbeda dari objek di bahasa 
lain dalam hal kita tidak bisa menambahkan data ke sebuah _trait object_. _Trait 
objects_ tidak berguna secara umum (generally useful) layaknya objek di bahasa 
lain: tujuan spesifik mereka adalah memungkinkan abstraksi melewati berbagai 
perilaku umum (common behavior).

Listing 18-3 menunjukkan gimana cara mendefinisikan sebuah trait bernama `Draw` 
dengan satu method bernama `draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="Definisi dari trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Sintaks ini seharusnya udah kerasa familier dari pembahasan kita soal gimana 
cara mendefinisikan traits di Bab 10. Berikutnya datang beberapa sintaks baru: 
Listing 18-4 mendefinisikan sebuah struct bernama `Screen` yang memegang sebuah 
vector bernama `components`. Vector ini bertipe `Box<dyn Draw>`, yang mana adalah 
sebuah _trait object_; dia jadi pengganti (_stand-in_) buat tipe apa pun di dalam 
sebuah `Box` yang mengimplementasikan trait `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Definisi dari struct `Screen` dengan sebuah field `components` yang memegang sebuah vector berisi _trait objects_ yang mengimplementasikan trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

Pada struct `Screen`, kita bakal mendefinisikan sebuah method bernama `run` 
yang bakal memanggil method `draw` pada masing-masing komponen (`components`)-nya, 
seperti yang ditunjukkan di Listing 18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="Sebuah method `run` pada `Screen` yang memanggil method `draw` pada tiap komponen">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Ini bekerja dengan cara yang beda dari pas kita mendefinisikan sebuah struct 
yang memakai sebuah parameter tipe generik (generic type parameter) dengan 
_trait bounds_. Sebuah parameter tipe generik cuma bisa digantikan (_substituted_) 
oleh satu tipe konkret aja pada satu waktu, sedangkan _trait objects_ memungkinkan 
banyak tipe konkret buat ngisi posisi _trait object_ tersebut saat _runtime_. 
Misalnya, kita bisa saja mendefinisikan struct `Screen` memakai tipe generik 
dan sebuah _trait bound_, seperti di Listing 18-6.

<Listing number="18-6" file-name="src/lib.rs" caption="Sebuah implementasi alternatif buat struct `Screen` dan method `run`-nya memakai generik dan _trait bounds_">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Cara ini membatasi kita pada satu instance `Screen` yang punya daftar komponen yang 
semuanya bertipe `Button` atau semuanya bertipe `TextField`. Kalau kita emang 
cuma bakal punya koleksi yang homogen (sama semua jenisnya), memakai generik 
dan _trait bounds_ lebih disarankan karena definisi-definisinya bakal 
di-_monomorphize_ (diubah ke tipe spesifik) saat _compile time_ buat memakai 
tipe-tipe konkret tersebut.

Di sisi lain, dengan memakai method yang menggunakan _trait objects_, satu 
instance `Screen` bisa memegang sebuah `Vec<T>` yang berisi sebuah `Box<Button>` 
sekaligus sebuah `Box<TextField>`. Mari kita lihat gimana cara kerja ini, dan 
nanti kita bakal ngebahas soal implikasi performa _runtime_-nya.

### Mengimplementasikan Trait-nya

Sekarang kita bakal menambahkan beberapa tipe yang mengimplementasikan trait 
`Draw`. Kita bakal menyediakan tipe `Button`. Sekali lagi, sebenarnya 
mengimplementasikan _library_ GUI itu ada di luar cakupan buku ini, jadi method 
`draw` di sini tidak bakal punya implementasi yang berguna di isi fungsinya. Buat 
ngebayangin seperti apa rupa dari implementasinya, sebuah struct `Button` 
mungkin punya _fields_ buat `width`, `height`, dan `label`, seperti yang 
ditunjukkan di Listing 18-7.

<Listing number="18-7" file-name="src/lib.rs" caption="Sebuah struct `Button` yang mengimplementasikan trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

_Fields_ `width`, `height`, dan `label` pada `Button` bakal beda dari _fields_ 
pada komponen-komponen lain; misalnya, tipe `TextField` mungkin punya _fields_ 
yang sama ditambah sebuah field `placeholder`. Masing-masing dari tipe yang mau 
kita gambar di layar bakal mengimplementasikan trait `Draw` tapi mereka bakal 
memakai kode yang berbeda di dalam method `draw` buat mendefinisikan gimana cara 
menggambar tipe khusus tersebut, seperti yang dimiliki `Button` di sini 
(tanpa kode GUI aslinya, seperti yang disebutkan sebelumnya). Tipe `Button`, 
misalnya, mungkin punya blok `impl` tambahan yang mengandung method-method yang 
berkaitan dengan apa yang terjadi saat seorang _user_ mengklik tombol (button) itu. 
Method-method semacam ini tidak bakal berlaku buat tipe kayak `TextField`.

Kalau seseorang yang memakai _library_ kita memutuskan buat mengimplementasikan 
sebuah struct `SelectBox` yang punya _fields_ `width`, `height`, dan `options`, 
mereka bakal mengimplementasikan trait `Draw` pada tipe `SelectBox` tersebut juga, 
seperti yang ditunjukkan di Listing 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="Sebuah _crate_ lain yang memakai `gui` dan mengimplementasikan trait `Draw` pada sebuah struct `SelectBox`">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

_User_ dari _library_ kita sekarang bisa menulis fungsi `main` mereka buat bikin 
sebuah instance `Screen`. Ke instance `Screen` ini, mereka bisa menambahkan 
sebuah `SelectBox` dan sebuah `Button` dengan menaruh masing-masing tipe 
tersebut di dalam sebuah `Box<T>` buat menjadikannya _trait object_. Terus 
mereka bisa manggil method `run` pada instance `Screen` tersebut, yang mana 
bakal manggil `draw` pada masing-masing komponen. Listing 18-9 menunjukkan 
implementasi ini.

<Listing number="18-9" file-name="src/main.rs" caption="Memakai _trait objects_ buat nyimpan nilai-nilai dari tipe yang berbeda yang mengimplementasikan trait yang sama">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Pas kita nulis _library_-nya, kita tidak tahu kalau seseorang mungkin bakal 
nambahin tipe `SelectBox`, tapi implementasi `Screen` kita nyatanya bisa 
beroperasi pada tipe yang baru tersebut dan menggambarnya karena `SelectBox` 
mengimplementasikan trait `Draw`, yang artinya dia mengimplementasikan method 
`draw`.

Konsep ini—yaitu peduli hanya pada pesan apa yang direspon sama sebuah nilai 
ketimbang peduli sama tipe konkret dari nilai tersebut—itu mirip sama konsep 
_duck typing_ (tipe bebek) di bahasa pemrograman dengan tipe dinamis (dynamically 
typed languages): kalau dia jalan kayak bebek dan kwek kayak bebek, maka dia pasti 
bebek! Di dalam implementasi dari `run` pada `Screen` di Listing 18-5, `run` tidak 
perlu tahu apa tipe konkret dari masing-masing komponen tersebut. Dia tidak 
ngecek apakah sebuah komponen itu adalah instance dari sebuah `Button` atau sebuah 
`SelectBox`, dia cuma manggil method `draw` pada komponen tersebut. Dengan 
menentukan `Box<dyn Draw>` sebagai tipe dari nilai-nilai di vector `components`, 
kita udah mendefinisikan `Screen` buat butuh nilai-nilai yang bisa kita 
panggil method `draw`-nya.

Keuntungan dari memakai _trait objects_ dan sistem tipe Rust buat nulis kode yang 
mirip sama kode yang memakai _duck typing_ adalah bahwa kita tidak perlu repot 
ngecek apakah sebuah nilai mengimplementasikan suatu method tertentu atau enggak 
saat _runtime_, atau khawatir dapat error kalau sebuah nilai ternyata tidak 
mengimplementasikan sebuah method tapi kita tetep memanggilnya. Rust tidak 
bakal mengompilasi kode kita kalau nilai-nilainya tidak mengimplementasikan traits 
yang dibutuhkan sama _trait objects_ tersebut.

Misalnya, Listing 18-10 menunjukkan apa yang terjadi kalau kita mencoba bikin 
sebuah `Screen` dengan sebuah `String` sebagai komponennya.

<Listing number="18-10" file-name="src/main.rs" caption="Mencoba memakai tipe yang tidak mengimplementasikan trait dari _trait object_">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Kita bakal dapat error ini karena `String` tidak mengimplementasikan trait 
`Draw`:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Error ini ngasih tahu kita kalau entah kita itu memberikan sesuatu ke `Screen` yang 
sebenarnya tidak kita maksudkan untuk kita berikan dan oleh karena itu kita harusnya 
memberikan tipe yang berbeda, atau kita harusnya mengimplementasikan `Draw` pada 
`String` supaya `Screen` bisa memanggil `draw` pada `String` tersebut.

### Trait Objects Melakukan Dynamic Dispatch

Ingat kembali di [“Performa Kode yang Memakai Generik”][performance-of-code-using-generics] 
di Bab 10 pembahasan kita soal proses _monomorphization_ (monomorfisasi) yang 
dilakukan pada tipe generik oleh _compiler_: _compiler_ membikin (generates) 
implementasi fungsi dan method yang non-generik (spesifik) buat setiap tipe konkret 
yang kita pakai buat gantiin parameter tipe generik tersebut. Kode hasil dari 
_monomorphization_ ini melakukan _static dispatch_ (pengiriman statis), yaitu 
ketika _compiler_ sudah tahu method apa yang kita panggil saat _compile time_. 
Ini berlawanan dengan _dynamic dispatch_ (pengiriman dinamis), yaitu ketika 
_compiler_ tidak bisa tahu pasti pas _compile time_ method mana yang kita panggil. 
Di kasus _dynamic dispatch_, _compiler_ menghasilkan (emits) kode yang saat _runtime_ 
baru bakal mencari tahu method mana yang harus dipanggil.

Saat kita memakai _trait objects_, Rust pasti memakai _dynamic dispatch_. 
_Compiler_ tidak tahu semua tipe yang mungkin dipakai bersama kode yang lagi memakai 
_trait objects_ tersebut, jadi dia tidak tahu method yang mana (yang diimplementasikan 
pada tipe yang mana) yang harus dipanggil. Alih-alih begitu, saat _runtime_, Rust 
memakai pointer yang ada di dalam _trait object_ buat mencari tahu method mana yang 
harus dipanggil. Pencarian (_lookup_) ini menimbulkan beban _runtime_ (runtime 
cost) yang tidak terjadi pada _static dispatch_. _Dynamic dispatch_ juga mencegah 
_compiler_ dari memilih buat meng-_inline_ (menyisipkan kode secara langsung) 
kode dari sebuah method, yang mana berakibat mencegah (prevents) dilakukannya 
beberapa optimasi lain. Rust juga punya beberapa aturan soal di mana kita boleh 
dan tidak boleh memakai _dynamic dispatch_, yang disebut _dyn compatibility_ 
(kompatibilitas dinamis). Aturan-aturan itu ada di luar cakupan pembahasan ini, 
tapi kita bisa baca lebih lanjut soal itu [di referensinya][dyn-compatibility]. 
Namun, kita dapat kebebasan ekstra (extra flexibility) di dalam kode yang kita tulis 
di Listing 18-5 dan berhasil mendukung tipe baru di Listing 18-9, jadi ini adalah 
_trade-off_ (pertukaran) yang patut dipertimbangkan.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
