## Reference Cycles Bisa Membocorkan Memori (Memory Leak)

Jaminan keamanan memori (memory safety guarantees) di Rust membikin hal itu 
jadi sulit, tapi bukannya mustahil, buat secara tidak sengaja membikin memori 
yang tidak akan pernah dibersihkan (dikenal sebagai _memory leak_ atau 
kebocoran memori). Mencegah _memory leaks_ secara total bukanlah salah satu 
jaminan yang diberikan Rust, yang berarti _memory leaks_ itu sifatnya aman-memori 
(_memory safe_) di Rust. Kita bisa melihat kalau Rust mengizinkan _memory leaks_ 
dengan memakai `Rc<T>` dan `RefCell<T>`: sangat mungkin buat membikin referensi 
di mana item-itemnya merujuk satu sama lain dalam sebuah _cycle_ (siklus). 
Ini membikin _memory leaks_ karena jumlah referensi (reference count) dari 
setiap item di dalam _cycle_ tidak akan pernah mencapai 0, dan nilainya 
tidak akan pernah di-_drop_.

### Membikin Sebuah Reference Cycle

Mari kita lihat gimana sebuah _reference cycle_ bisa terjadi dan gimana cara 
mencegahnya, dimulai dengan definisi dari enum `List` dan method `tail` 
di Listing 15-25.

<Listing number="15-25" file-name="src/main.rs" caption="Definisi sebuah cons list yang menampung sebuah `RefCell<T>` supaya kita bisa memodifikasi apa yang ditunjuk oleh varian `Cons`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs}}
```

</Listing>

Kita memakai variasi lain dari definisi `List` dari Listing 15-5. Elemen 
kedua di varian `Cons` sekarang adalah `RefCell<Rc<List>>`, yang berarti 
bahwa ketimbang cuma punya kemampuan buat memodifikasi nilai `i32` seperti 
yang kita lakuin di Listing 15-24, kita sekarang mau memodifikasi nilai `List` 
yang ditunjuk oleh sebuah varian `Cons`. Kita juga menambahkan method `tail` 
buat memudahkan kita mengakses item kedua kalau kita punya sebuah varian `Cons`.

Di Listing 15-26, kita menambahkan fungsi `main` yang memakai definisi di 
Listing 15-25. Kode ini membikin sebuah list di `a` dan sebuah list di `b` 
yang menunjuk ke list di `a`. Terus dia memodifikasi list di `a` buat menunjuk 
ke `b`, sehingga membikin sebuah _reference cycle_. Ada *statements* `println!` 
di sepanjang jalan buat menunjukkan berapa jumlah referensi (_reference counts_) 
di berbagai titik dalam proses ini.

<Listing number="15-26" file-name="src/main.rs" caption="Membikin sebuah _reference cycle_ dari dua nilai `List` yang saling menunjuk satu sama lain">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

Kita membikin instance `Rc<List>` yang memegang sebuah nilai `List` di variabel 
`a` dengan list awal berupa `5, Nil`. Terus kita membikin instance `Rc<List>` 
yang memegang nilai `List` lainnya di variabel `b` yang berisi nilai `10` 
dan menunjuk ke list di `a`.

Kita memodifikasi `a` agar dia menunjuk ke `b` ketimbang ke `Nil`, sehingga 
membikin sebuah _cycle_. Kita melakukan itu dengan memakai method `tail` buat 
mendapatkan sebuah referensi ke `RefCell<Rc<List>>` di `a`, yang kemudian kita 
taruh di variabel `link`. Lalu kita memakai method `borrow_mut` pada 
`RefCell<Rc<List>>` tersebut buat mengubah nilai internalnya dari sebuah 
`Rc<List>` yang memegang nilai `Nil` menjadi `Rc<List>` yang ada di `b`.

Saat kita menjalankan kode ini, dengan membiarkan `println!` terakhir 
tetap dikomentari (commented out) buat saat ini, kita bakal dapat output ini:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

Jumlah referensi (reference count) dari instance `Rc<List>` di `a` maupun `b` 
adalah 2 setelah kita ngubah list di `a` agar menunjuk ke `b`. Di akhir dari 
`main`, Rust men-_drop_ variabel `b`, yang mana menurunkan jumlah referensi 
dari instance `Rc<List>` si `b` dari 2 menjadi 1. Memori yang dimiliki `Rc<List>` 
di _heap_ tidak bakal di-_drop_ di titik ini karena jumlah referensinya itu 1, 
bukannya 0. Terus Rust men-_drop_ `a`, yang mana menurunkan jumlah referensi dari 
instance `Rc<List>` si `a` dari 2 menjadi 1 juga. Memori dari instance ini 
juga tidak bisa di-_drop_, karena instance `Rc<List>` yang satunya lagi masih 
merujuk ke dia. Memori yang dialokasikan ke list tersebut bakal terus tersisa 
dan tidak dibersihkan selamanya. Buat memvisualisasikan _reference cycle_ ini, 
kita sudah membikin diagram di Gambar 15-4.

<img alt="Sebuah persegi panjang berlabel 'a' yang menunjuk ke sebuah persegi panjang berisi integer 5. Sebuah persegi panjang berlabel 'b' yang menunjuk ke sebuah persegi panjang berisi integer 10. Persegi panjang yang berisi 5 menunjuk ke persegi panjang yang berisi 10, dan persegi panjang yang berisi 10 menunjuk balik ke persegi panjang yang berisi 5, menciptakan sebuah cycle (siklus)" src="img/trpl15-04.svg" class="center" />

<span class="caption">Gambar 15-4: Sebuah _reference cycle_ dari list `a` dan 
`b` yang saling menunjuk satu sama lain</span>

Kalau kita menghilangkan komentar (_uncomment_) pada `println!` yang terakhir lalu 
menjalankan programnya, Rust bakal mencoba mencetak _cycle_ ini dengan `a` 
menunjuk ke `b` menunjuk ke `a` dan seterusnya sampai dia mengalami _stack overflow_.

Dibandingkan dengan program di dunia nyata, konsekuensi dari membikin 
_reference cycle_ di contoh ini tidak terlalu fatal: sesaat setelah kita 
membikin _reference cycle_ tersebut, programnya berakhir. Namun, kalau sebuah 
program yang lebih kompleks mengalokasikan banyak memori di dalam sebuah _cycle_ 
lalu memegangnya untuk waktu yang lama, program tersebut bakal memakai lebih 
banyak memori daripada yang dia butuhkan dan bisa bikin sistem kewalahan, 
yang berujung pada kehabisan memori (_out of memory_).

Membikin _reference cycles_ itu memang tidak mudah dilakukan, tapi itu juga 
bukanlah hal yang mustahil. Kalau kita punya nilai `RefCell<T>` yang 
mengandung nilai `Rc<T>` atau kombinasi bersarang yang mirip dari tipe-tipe yang 
punya _interior mutability_ dan _reference counting_, kita harus memastikan 
kalau kita tidak membikin _cycles_; kita tidak bisa ngandelin Rust buat 
menangkap hal tersebut. Membikin sebuah _reference cycle_ adalah sebuah *logic 
bug* (kutu logika) di program kita yang harusnya diminimalisir dengan memakai 
_automated tests_ (pengujian otomatis), _code reviews_ (tinjauan kode), dan 
praktik pengembangan *software* lainnya.

Solusi lain buat menghindari _reference cycles_ adalah dengan mengatur ulang 
struktur data kita sedemikian rupa sehingga beberapa referensi mengekspresikan 
kepemilikan (ownership) dan referensi lainnya tidak. Sebagai hasilnya, kita 
bisa punya _cycles_ yang dibikin dari beberapa hubungan kepemilikan dan beberapa 
hubungan non-kepemilikan, dan cuma hubungan kepemilikan lah yang memengaruhi 
apakah suatu nilai bisa di-_drop_ atau tidak. Di Listing 15-25, kita selalu mau 
varian `Cons` buat memiliki list mereka, jadi mengatur ulang struktur datanya 
itu tidak memungkinkan. Mari kita lihat contoh yang memakai *graphs* (graf) yang 
dibikin dari *parent nodes* (simpul induk) dan *child nodes* (simpul anak) 
buat melihat kapan hubungan non-kepemilikan menjadi cara yang pas buat 
mencegah _reference cycles_.

<!-- Old link, do not remove -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### Mencegah Reference Cycles Memakai `Weak<T>`

Sejauh ini, kita sudah mendemonstrasikan kalau memanggil `Rc::clone` bakal 
menaikkan nilai `strong_count` dari sebuah instance `Rc<T>`, dan sebuah instance 
`Rc<T>` cuma bakal dibersihkan kalau nilai `strong_count`-nya 0. Kita juga bisa 
membikin sebuah _weak reference_ (referensi lemah) ke sebuah nilai yang ada 
di dalam instance `Rc<T>` dengan memanggil `Rc::downgrade` dan meneruskan sebuah 
referensi ke `Rc<T>` tersebut. _Strong references_ (referensi kuat) adalah cara 
gimana kita bisa berbagi kepemilikan dari sebuah instance `Rc<T>`. _Weak references_ 
tidak mengekspresikan hubungan kepemilikan, dan jumlah mereka (count) tidak 
memengaruhi kapan sebuah instance `Rc<T>` dibersihkan. Mereka tidak bakal bikin 
sebuah _reference cycle_ karena _cycle_ apa pun yang melibatkan beberapa _weak 
references_ bakal terputus begitu jumlah _strong reference_ dari nilai-nilai yang 
terlibat mencapai 0.

Pas kita memanggil `Rc::downgrade`, kita bakal dapat sebuah _smart pointer_ 
bertipe `Weak<T>`. Alih-alih menaikkan `strong_count` di instance `Rc<T>` sebanyak 
1, memanggil `Rc::downgrade` menaikkan `weak_count` sebanyak 1. Tipe `Rc<T>` memakai 
`weak_count` buat melacak berapa banyak referensi `Weak<T>` yang ada, mirip 
kayak `strong_count`. Bedanya adalah `weak_count` tidak harus 0 supaya instance 
`Rc<T>`-nya bisa dibersihkan.

Karena nilai yang ditunjuk sama `Weak<T>` mungkin sudah di-_drop_, untuk melakukan 
apa pun dengan nilai yang ditunjuk sama `Weak<T>` kita harus memastikan kalau 
nilainya masih eksis. Lakukan ini dengan memanggil method `upgrade` pada sebuah 
instance `Weak<T>`, yang mana bakal mengembalikan sebuah `Option<Rc<T>>`. Kita 
bakal dapat hasil `Some` kalau nilai `Rc<T>` tersebut belum di-_drop_ dan hasil 
`None` kalau nilai `Rc<T>` tersebut sudah di-_drop_. Karena `upgrade` mengembalikan 
sebuah `Option<Rc<T>>`, Rust bakal memastikan kalau kasus `Some` maupun kasus `None` 
udah ditangani, dan tidak bakal ada yang namanya pointer yang tidak valid.

Sebagai contoh, ketimbang memakai sebuah list yang item-itemnya cuma tahu soal 
item selanjutnya aja, kita bakal membikin sebuah struktur _tree_ (pohon) yang 
item-itemnya tahu soal *children items* (item anak) mereka *dan* *parent items* 
(item induk) mereka.

#### Membikin Struktur Data Tree: Sebuah `Node` dengan Child Nodes

Sebagai awalan, kita bakal ngebangun sebuah _tree_ yang terdiri dari _nodes_ (simpul) 
yang tahu soal *child nodes* mereka. Kita bakal membikin sebuah struct bernama `Node` 
yang memegang nilai `i32`-nya sendiri dan juga referensi ke nilai-nilai `Node` dari 
*children*-nya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Kita pengen supaya sebuah `Node` memiliki *children*-nya, dan kita mau berbagi 
kepemilikan tersebut dengan berbagai variabel sehingga kita bisa mengakses 
setiap `Node` di _tree_ tersebut secara langsung. Buat melakukannya, kita 
mendefinisikan item-item `Vec<T>` agar berupa nilai-nilai bertipe `Rc<Node>`. 
Kita juga mau mengubah *nodes* mana saja yang merupakan *children* dari _node_ lain, 
jadi kita membungkus `Vec<Rc<Node>>` tersebut di dalam sebuah `RefCell<T>` di 
field `children`.

Selanjutnya, kita bakal memakai definisi struct kita buat membikin satu instance 
`Node` bernama `leaf` (daun) dengan nilai `3` dan tanpa *children*, serta instance 
lain bernama `branch` (cabang) dengan nilai `5` dan `leaf` sebagai salah satu 
*children*-nya, seperti yang ditunjukkan di Listing 15-27.

<Listing number="15-27" file-name="src/main.rs" caption="Membikin sebuah *leaf* node tanpa *children* dan sebuah *branch* node yang menjadikan *leaf* sebagai salah satu *children*-nya">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

Kita meng-_clone_ `Rc<Node>` yang ada di `leaf` dan menyimpannya di dalam 
`branch`, yang artinya `Node` di `leaf` sekarang punya dua pemilik: `leaf` 
dan `branch`. Kita bisa pindah dari `branch` ke `leaf` melalui `branch.children`, 
tapi tidak ada cara buat pindah dari `leaf` ke `branch`. Alasannya adalah karena 
`leaf` tidak punya referensi ke `branch` dan tidak tahu kalau mereka itu 
berhubungan. Kita pengen supaya `leaf` tahu kalau `branch` itu adalah *parent*-nya 
(induknya). Kita bakal melakukan hal itu selanjutnya.

#### Menambahkan Referensi dari Child ke Parent-nya

Buat membikin si *child node* sadar akan *parent*-nya, kita perlu menambahkan 
sebuah field `parent` ke definisi struct `Node` kita. Masalahnya adalah 
menentukan apa seharusnya tipe dari `parent` ini. Kita tahu kalau dia tidak 
boleh berisi `Rc<T>`, karena itu bakal membikin sebuah _reference cycle_ dengan 
`leaf.parent` yang menunjuk ke `branch` dan `branch.children` yang menunjuk ke 
`leaf`, yang mana bakal membikin nilai `strong_count` mereka tidak akan 
pernah mencapai 0.

Membayangkan hubungan ini dengan cara lain, sebuah *parent node* seharusnya 
memiliki *children*-nya: kalau sebuah *parent node* di-_drop_, *child nodes*-nya 
seharusnya ikut di-_drop_ juga. Namun, sebuah *child* tidak seharusnya memiliki 
*parent*-nya: kalau kita men-_drop_ sebuah *child node*, sang *parent* seharusnya 
tetap eksis. Ini adalah kasus yang tepat sekali buat memakai _weak references_!

Jadi ketimbang memakai `Rc<T>`, kita bakal membikin tipe dari `parent` tersebut 
agar memakai `Weak<T>`, spesifiknya adalah `RefCell<Weak<Node>>`. Sekarang 
definisi struct `Node` kita kelihatan kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Sebuah _node_ sekarang bakal bisa merujuk ke *parent node*-nya tapi dia tidak 
memiliki *parent* tersebut. Di Listing 15-28, kita meng-update `main` buat memakai 
definisi baru ini sehingga *node* `leaf` bakal punya cara buat merujuk ke 
*parent*-nya, yaitu `branch`.

<Listing number="15-28" file-name="src/main.rs" caption="Sebuah *leaf* node dengan referensi lemah (_weak reference_) ke *parent node*-nya, `branch`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

Membikin *node* `leaf` ini kelihatan mirip sama yang di Listing 15-27 dengan 
pengecualian di field `parent`: `leaf` awalnya tidak punya *parent*, jadi kita 
membikin sebuah instance referensi `Weak<Node>` baru yang kosong.

Pada titik ini, saat kita mencoba buat mendapatkan referensi ke *parent* dari 
`leaf` dengan memakai method `upgrade`, kita bakal dapat sebuah nilai `None`. 
Kita bisa melihat ini di dalam output dari *statement* `println!` yang pertama:

```text
leaf parent = None
```

Saat kita membikin *node* `branch`, dia juga bakal punya sebuah referensi 
`Weak<Node>` baru di field `parent`-nya karena `branch` tidak punya *parent 
node*. Kita tetap punya `leaf` sebagai salah satu dari *children* si `branch`. 
Setelah kita mendapatkan instance `Node` di dalam `branch`, kita bisa 
memodifikasi `leaf` buat ngasih dia sebuah referensi `Weak<Node>` ke 
*parent*-nya. Kita memakai method `borrow_mut` pada `RefCell<Weak<Node>>` di 
dalam field `parent` si `leaf`, dan terus kita pakai fungsi `Rc::downgrade` buat 
membikin sebuah referensi `Weak<Node>` ke `branch` dari `Rc<Node>` yang ada 
di dalam `branch`.

Pas kita mencetak *parent* dari `leaf` lagi, kali ini kita bakal dapat 
sebuah varian `Some` yang memegang `branch`: sekarang `leaf` bisa mengakses 
*parent*-nya! Pas kita mencetak `leaf`, kita juga terhindar dari _cycle_ yang 
pada akhirnya berujung pada _stack overflow_ kayak yang terjadi di Listing 15-26; 
referensi `Weak<Node>` tersebut cuma dicetak sebagai `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Absennya (lack of) output yang tidak terhingga ini menandakan bahwa kode ini 
tidak membikin sebuah _reference cycle_. Kita juga bisa mengetahuinya dengan melihat 
ke nilai-nilai yang kita dapat dari memanggil `Rc::strong_count` dan 
`Rc::weak_count`.

#### Memvisualisasikan Perubahan pada `strong_count` dan `weak_count`

Mari kita lihat gimana nilai `strong_count` dan `weak_count` dari instance-instance 
`Rc<Node>` tersebut berubah dengan membikin sebuah _inner scope_ (scope dalam) baru 
lalu memindahkan pembuatan `branch` ke dalam _scope_ tersebut. Dengan melakukan ini, 
kita bisa melihat apa yang terjadi saat `branch` dibikin dan kemudian di-_drop_ 
pas dia keluar dari _scope_. Modifikasi ini ditunjukkan di Listing 15-29.

<Listing number="15-29" file-name="src/main.rs" caption="Membikin `branch` di sebuah _inner scope_ dan memeriksa jumlah (count) _strong_ dan _weak_ reference-nya">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

Setelah `leaf` dibikin, `Rc<Node>` miliknya punya *strong count* sebesar 1 dan 
*weak count* sebesar 0. Di dalam _inner scope_, kita membikin `branch` dan 
mengaitkannya (associate) dengan `leaf`, di mana pada titik ini kalau kita 
mencetak jumlahnya, `Rc<Node>` di dalam `branch` bakal punya *strong count* 1 
dan *weak count* 1 (karena `leaf.parent` menunjuk ke `branch` dengan 
sebuah `Weak<Node>`). Pas kita mencetak jumlah (counts) yang ada di `leaf`, 
kita bakal melihat kalau dia punya *strong count* sebesar 2 karena `branch` 
sekarang punya *clone* dari `Rc<Node>` milik `leaf` yang disimpan di 
`branch.children`, tapi dia tetap punya *weak count* sebesar 0.

Saat _inner scope_-nya berakhir, `branch` keluar dari _scope_ dan *strong count* 
dari `Rc<Node>`-nya turun menjadi 0, jadi `Node`-nya bakal di-_drop_. *Weak count* 
sebesar 1 dari `leaf.parent` sama sekali tidak memengaruhi apakah si `Node` itu 
bakal di-_drop_ atau tidak, jadi kita tidak dapat _memory leaks_!

Kalau kita nyoba mengakses *parent* dari `leaf` setelah akhir dari _scope_ itu, 
kita bakal dapat `None` lagi. Di akhir dari program, `Rc<Node>` di dalam `leaf` 
punya *strong count* sebesar 1 dan *weak count* sebesar 0 karena variabel `leaf` 
sekarang menjadi satu-satunya referensi ke `Rc<Node>` tersebut lagi.

Semua logika yang mengelola jumlah referensi (_counts_) dan pen-_drop_-an nilai ini 
dibangun langsung di dalam `Rc<T>` dan `Weak<T>` beserta implementasi mereka 
pada trait `Drop`. Dengan secara spesifik menentukan kalau hubungan dari seorang 
*child* ke *parent*-nya haruslah memakai referensi `Weak<T>` di dalam definisi 
dari `Node`, kita bisa membikin *parent nodes* menunjuk ke *child nodes* dan 
begitu juga sebaliknya tanpa membikin sebuah _reference cycle_ dan _memory leaks_.

## Ringkasan

Bab ini membahas gimana cara memakai _smart pointers_ buat membikin berbagai jaminan 
dan _trade-offs_ (pertukaran) yang berbeda dari apa yang Rust lakukan secara 
default dengan referensi biasa. Tipe `Box<T>` punya ukuran yang sudah pasti 
diketahui dan menunjuk ke data yang dialokasikan di _heap_. Tipe `Rc<T>` melacak 
jumlah referensi ke data yang ada di _heap_ supaya data tersebut bisa punya 
banyak pemilik. Tipe `RefCell<T>` dengan kemampuan _interior mutability_-nya 
ngasih kita tipe yang bisa kita pakai pas kita butuh tipe yang _immutable_ tapi 
juga butuh buat ngubah nilai di dalamnya; ia juga menegakkan aturan _borrowing_ 
saat _runtime_ bukannya saat _compile time_.

Kita juga udah ngebahas trait `Deref` dan `Drop`, yang memungkinkan berjalannya 
banyak dari fungsionalitas _smart pointers_ ini. Kita mengeksplorasi _reference 
cycles_ yang bisa menyebabkan _memory leaks_ dan gimana cara mencegah mereka 
memakai `Weak<T>`.

Kalau bab ini sudah membangkitkan rasa penasaran kita dan kita pengen 
mengimplementasikan _smart pointers_ kita sendiri, silakan cek 
[“The Rustonomicon”][nomicon] buat dapetin lebih banyak informasi yang berguna.

Berikutnya, kita bakal ngomongin soal konkurensi (concurrency) di Rust. Kita 
bahkan bakal mempelajari beberapa _smart pointers_ baru lagi lho!

[nomicon]: ../nomicon/index.html
