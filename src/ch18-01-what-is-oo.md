## Ciri-ciri Bahasa Pemrograman Berorientasi Objek

Tidak ada konsensus (kesepakatan umum) di komunitas pemrograman mengenai fitur-
fitur apa aja yang wajib dimiliki oleh sebuah bahasa supaya bisa dianggap 
berorientasi objek. Rust dipengaruhi oleh banyak paradigma pemrograman, termasuk 
OOP; misalnya, kita sudah mengeksplorasi fitur-fitur yang datang dari pemrograman 
fungsional (functional programming) di Bab 13. Walaupun bisa diperdebatkan 
(arguably), bahasa pemrograman OOP biasanya berbagi ciri-ciri umum tertentu, 
yakni objek, _encapsulation_ (enkapsulasi), dan _inheritance_ (pewarisan). 
Mari kita lihat apa arti dari masing-masing ciri tersebut dan apakah Rust 
mendukungnya atau tidak.

### Objek Mengandung Data dan Perilaku (Behavior)

Buku _Design Patterns: Elements of Reusable Object-Oriented Software_ karangan 
Erich Gamma, Richard Helm, Ralph Johnson, dan John Vlissides (Addison-Wesley, 
1994), yang secara santai sering disebut sebagai buku _The Gang of Four_, 
adalah sebuah katalog berisi desain pola berorientasi objek. Buku itu 
mendefinisikan OOP dengan cara ini:

> Program berorientasi objek dibikin dari objek-objek. Sebuah **objek** 
> membungkus (packages) baik data maupun prosedur-prosedur yang beroperasi 
> pada data tersebut. Prosedur-prosedur tersebut biasanya disebut **methods** 
> atau **operations**.

Memakai definisi ini, Rust itu berorientasi objek: _structs_ dan _enums_ punya 
data, dan blok `impl` menyediakan _methods_ pada _structs_ dan _enums_ tersebut. 
Meskipun _structs_ dan _enums_ yang dilengkapi _methods_ tidak _disebut_ sebagai 
objek, mereka menyediakan fungsionalitas yang sama, menurut definisi objek dari 
the Gang of Four.

### Encapsulation yang Menyembunyikan Detail Implementasi

Aspek lain yang umumnya dikaitkan dengan OOP adalah ide tentang 
_encapsulation_ (enkapsulasi), yang berarti kalau detail implementasi dari sebuah 
objek itu tidak bisa diakses (accessible) oleh kode yang memakai objek tersebut. 
Oleh karena itu, satu-satunya cara buat berinteraksi dengan sebuah objek adalah 
melalui API _public_-nya; kode yang memakai objek itu tidak seharusnya bisa 
menjangkau bagian internal dari si objek lalu mengubah data atau perilakunya 
secara langsung. Hal ini memungkinkan programmer untuk mengubah dan me-*refactor* 
bagian internal dari sebuah objek tanpa perlu mengubah kode yang memakai objek 
tersebut.

Kita sudah membahas cara mengontrol _encapsulation_ di Bab 7: kita bisa memakai 
keyword `pub` buat menentukan modul, tipe, fungsi, dan _method_ mana saja di kode 
kita yang seharusnya bersifat _public_, sementara secara default segala hal 
lainnya bersifat _private_. Misalnya, kita bisa mendefinisikan sebuah struct 
`AveragedCollection` yang punya field yang berisi vector dari nilai `i32`. 
Struct tersebut juga bisa punya sebuah field yang berisi nilai rata-rata 
(average) dari nilai-nilai di vector tersebut, yang berarti nilai rata-ratanya 
tidak harus dihitung seketika (on demand) kapan pun seseorang membutuhkannya. 
Dengan kata lain, `AveragedCollection` bakal menge-_cache_ nilai rata-rata 
yang sudah dihitung tersebut buat kita. Listing 18-1 punya definisi dari struct 
`AveragedCollection`.

<Listing number="18-1" file-name="src/lib.rs" caption="Sebuah struct `AveragedCollection` yang memelihara daftar (list) dari angka _integer_ dan nilai rata-rata dari item-item di koleksi tersebut">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struct ini ditandai sebagai `pub` supaya kode lain bisa memakainya, tapi field-
field di dalam struct tersebut tetap bersifat _private_. Hal ini penting di 
kasus ini karena kita pengen memastikan bahwa kapan pun sebuah nilai 
ditambahkan atau dihapus dari *list*, nilai *average* juga harus di-update. 
Kita melakukan ini dengan mengimplementasikan method `add`, `remove`, dan 
`average` pada struct tersebut, seperti yang ditunjukkan di Listing 18-2.

<Listing number="18-2" file-name="src/lib.rs" caption="Implementasi dari _public methods_ `add`, `remove`, dan `average` pada `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

_Public methods_ `add`, `remove`, dan `average` adalah satu-satunya cara buat 
mengakses atau memodifikasi data di dalam sebuah instance `AveragedCollection`. 
Saat sebuah item ditambahkan ke `list` memakai method `add` atau dihapus 
memakai method `remove`, implementasi dari kedua method ini bakal memanggil 
method _private_ `update_average` yang juga menangani pembaruan pada field 
`average`.

Kita membiarkan field `list` dan `average` tetap _private_ sehingga tidak ada 
cara buat kode eksternal (external code) buat menambahkan atau menghapus item 
ke atau dari field `list` secara langsung; karena jika dibiarkan, field `average` 
bisa jadi tidak sinkron saat `list` berubah. Method `average` mengembalikan nilai 
yang ada di dalam field `average`, yang memungkinkan kode eksternal buat membaca 
nilai `average` tersebut tapi tidak bisa memodifikasinya.

Karena kita udah mengenkapsulasi (encapsulated) detail implementasi dari struct 
`AveragedCollection`, kita bisa gampang mengubah berbagai aspeknya, kayak 
struktur datanya, di masa depan. Misalnya, kita bisa aja memakai sebuah 
`HashSet<i32>` ketimbang `Vec<i32>` buat field `list`. Asalkan _signature_ 
dari _public methods_ `add`, `remove`, dan `average` itu tetap sama, kode 
yang memakai `AveragedCollection` tidak perlu berubah. Tapi, kalau seandainya 
kita membikin `list` jadi _public_, hal ini belum tentu benar: `HashSet<i32>` 
dan `Vec<i32>` punya method yang berbeda buat menambahkan dan menghapus item, 
jadi kode eksternalnya kemungkinan besar juga harus diubah kalau mereka 
awalnya memodifikasi `list` secara langsung.

Kalau _encapsulation_ adalah aspek yang wajib dipunyai sama sebuah bahasa agar 
bisa dianggap berorientasi objek, maka Rust memenuhi syarat tersebut. Pilihan 
buat memakai `pub` atau tidak pada berbagai bagian kode memungkinkan adanya 
_encapsulation_ terhadap detail-detail implementasi.

### Inheritance sebagai Sistem Tipe (Type System) dan Pembagian Kode (Code Sharing)

_Inheritance_ (pewarisan) adalah sebuah mekanisme di mana sebuah objek bisa 
mewarisi (inherit) elemen-elemen dari definisi objek lain, dengan begitu ia 
mendapatkan data dan perilaku dari objek induk (parent object) tanpa kita 
harus mendefinisikannya lagi.

Kalau sebuah bahasa _wajib_ punya _inheritance_ buat bisa disebut berorientasi 
objek, maka Rust tidak termasuk dalam bahasa tersebut. Tidak ada cara buat 
mendefinisikan sebuah struct yang mewarisi field-field dan implementasi _method_ 
dari struct induk tanpa memakai _macro_.

Namun, kalau kita udah biasa (used to) punya _inheritance_ di alat (toolbox) 
pemrograman kita, kita bisa memakai solusi lain di Rust, tergantung dari alasan 
kenapa kita mencari _inheritance_ tersebut sejak awal.

Kita biasanya bakal milih buat memakai _inheritance_ karena dua alasan utama. 
Alasan pertama adalah buat pemakaian ulang kode (reuse of code): kita bisa 
mengimplementasikan perilaku tertentu untuk satu tipe, dan _inheritance_ 
memungkinkan kita buat memakai ulang implementasi tersebut buat tipe yang berbeda. 
Kita bisa melakukan hal ini secara terbatas di kode Rust dengan memakai 
_default trait method implementations_ (implementasi bawaan pada trait method), 
yang udah kita lihat di Listing 10-14 pas kita menambahkan implementasi default 
dari method `summarize` pada trait `Summary`. Tipe apa pun yang 
mengimplementasikan trait `Summary` bakal langsung punya method `summarize` yang 
bisa dipakai padanya tanpa perlu nulis kode tambahan lagi. Ini mirip dengan 
sebuah *parent class* (kelas induk) yang punya implementasi dari suatu _method_ 
dan sebuah *inheriting child class* (kelas anak yang mewarisi) yang juga ikutan 
punya implementasi dari _method_ tersebut. Kita juga bisa menimpa (override) 
implementasi default dari method `summarize` pas kita mengimplementasikan 
trait `Summary`, yang mana mirip kayak saat sebuah *child class* menimpa 
implementasi _method_ yang diwarisinya dari sebuah *parent class*.

Alasan lain memakai _inheritance_ berkaitan dengan sistem tipenya: yakni 
memungkinkan sebuah tipe *child* (anak) buat dipakai di tempat-tempat yang 
sama kayak tipe *parent* (induk)-nya. Ini juga disebut dengan _polymorphism_ 
(polimorfisme), yang berarti kita bisa mensubstitusikan (menggantikan) berbagai 
objek untuk satu sama lain pas _runtime_ asalkan mereka berbagi karakteristik 
tertentu.

> ### Polimorfisme
>
> Buat banyak orang, polimorfisme itu sinonim dengan _inheritance_. Tapi dia 
> sebenarnya adalah konsep yang lebih umum yang mengacu ke kode yang bisa bekerja 
> bareng data dari berbagai macam tipe. Buat _inheritance_, tipe-tipe itu 
> umumnya adalah *subclasses* (subkelas).
>
> Sebaliknya, Rust memakai _generics_ (generik) buat mengabstraksi bermacam-
> macam kemungkinan tipe dan _trait bounds_ buat memaksakan batasan-batasan 
> (constraints) pada apa yang wajib disediakan sama tipe-tipe tersebut. Ini 
> kadang-kadang disebut sebagai _bounded parametric polymorphism_ (polimorfisme 
> parametrik terbatas).

Rust memilih serangkaian _tradeoffs_ yang berbeda dengan tidak menawarkan 
_inheritance_. _Inheritance_ sering kali punya risiko buat nge-share lebih banyak 
kode daripada yang diperlukan. *Subclasses* tidak seharusnya selalu berbagi semua 
karakteristik dari *parent class* mereka, tapi mereka bakal melakukan itu kalau 
pakai _inheritance_. Hal ini bisa bikin desain dari sebuah program jadi kurang 
fleksibel. Ini juga memunculkan kemungkinan memanggil _method-method_ pada 
*subclasses* yang sebenarnya tidak masuk akal atau menyebabkan error gara-gara 
_method-method_ itu nyatanya tidak berlaku (apply) buat *subclass* tersebut. Selain 
itu, beberapa bahasa cuma mengizinkan _single inheritance_ (berarti satu 
*subclass* cuma boleh mewarisi dari satu kelas saja), yang lebih lanjut membatasi 
fleksibilitas dari desain sebuah program.

Atas alasan-alasan ini, Rust mengambil pendekatan yang berbeda yaitu dengan 
memakai _trait objects_ (objek trait) sebagai ganti _inheritance_ buat 
memungkinkan adanya polimorfisme. Mari kita lihat gimana cara _trait objects_ 
bekerja.
