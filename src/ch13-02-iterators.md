## Memproses Serangkaian Item dengan Iterators

Pola (pattern) _iterator_ memungkinkan kita buat melakukan suatu tugas secara 
berurutan pada serangkaian item. Sebuah iterator bertanggung jawab atas logika 
untuk melakukan iterasi melewati setiap item dan menentukan kapan rangkaian 
tersebut sudah selesai. Saat kita memakai iterator, kita tidak perlu repot-repot 
mengimplementasikan ulang logika tersebut sendiri.

Di Rust, iterators itu _lazy_ (malas), yang artinya mereka tidak punya efek 
apa-apa sampai kita memanggil method-method yang bakal mengonsumsi (consume) 
iterator itu untuk memakainya sampai habis. Sebagai contoh, kode di Listing 
13-10 membuat sebuah iterator atas item-item di dalam vector `v1` dengan 
memanggil method `iter` yang didefinisikan pada `Vec<T>`. Kode ini kalau 
berdiri sendiri tidak melakukan hal berguna apa pun.

<Listing number="13-10" file-name="src/main.rs" caption="Membuat sebuah iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Iterator ini disimpan di dalam variabel `v1_iter`. Setelah kita membuat sebuah 
iterator, kita bisa memakainya dalam berbagai cara. Di Listing 3-5, kita 
beriterasi melewati sebuah _array_ memakai _for loop_ untuk mengeksekusi 
beberapa kode pada masing-masing itemnya. Di balik layar, hal ini secara 
implisit membuat lalu mengonsumsi sebuah iterator, tapi kita melewatkan detail 
tentang gimana sebenarnya itu bekerja sampai saat ini.

Di contoh pada Listing 13-11, kita memisahkan proses pembuatan iterator dari 
penggunaan iterator tersebut di dalam _for loop_. Saat _for loop_ ini dipanggil 
memakai iterator di `v1_iter`, setiap elemen di iterator tersebut dipakai 
dalam satu putaran (iteration) _loop_, yang mana mencetak setiap nilainya.

<Listing number="13-11" file-name="src/main.rs" caption="Memakai iterator di dalam sebuah _for loop_">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Di bahasa pemrograman yang tidak menyediakan iterator dari _standard library_-nya, 
kita kemungkinan bakal menulis fungsionalitas yang sama ini dengan memulai 
sebuah variabel di indeks 0, memakai variabel tersebut buat mengindeks ke dalam 
vector untuk mendapatkan nilai, lalu menambah nilai variabel itu di dalam _loop_ 
sampai jumlahnya mencapai total item yang ada di vector.

Iterators menangani semua logika tersebut buat kita, mengurangi kode yang 
berulang-ulang (repetitive code) yang mana bisa saja kita bikin salah. 
Iterators ngasih kita fleksibilitas lebih buat memakai logika yang sama dengan 
berbagai jenis urutan data, bukan cuma struktur data yang bisa kita indeks aja, 
kayak vector. Mari kita teliti gimana cara iterators melakukan itu.

### Trait `Iterator` dan Method `next`

Semua iterators mengimplementasikan sebuah trait bernama `Iterator` yang 
didefinisikan di _standard library_. Definisi dari trait ini kelihatan kayak gini:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods dengan implementasi default dihilangkan
}
```

Perhatikan bahwa definisi ini memakai beberapa sintaks baru: `type Item` dan 
`Self::Item`, yang mendefinisikan sebuah _associated type_ dengan trait ini. 
Kita bakal membahas _associated types_ lebih mendalam di Bab 20. Buat sekarang, 
yang perlu kita tahu adalah bahwa kode ini bilang kalau buat mengimplementasikan 
trait `Iterator`, kita juga harus mendefinisikan tipe `Item`, dan tipe `Item` 
ini dipakai di tipe kembalian (return type) dari method `next`. Dengan kata 
lain, tipe `Item` bakal jadi tipe yang dikembalikan dari iterator tersebut.

Trait `Iterator` cuma mewajibkan para peng-implementasi (implementors) buat 
mendefinisikan satu method saja: yaitu method `next`, yang mengembalikan satu 
item dari iterator pada satu waktu, dibungkus di dalam `Some`, dan, ketika 
iterasi selesai, mengembalikan `None`.

Kita bisa memanggil method `next` pada iterators secara langsung; Listing 13-12 
mendemonstrasikan nilai-nilai apa aja yang dikembalikan dari pemanggilan 
berulang ke `next` pada iterator yang dibikin dari vector.

<Listing number="13-12" file-name="src/lib.rs" caption="Memanggil method `next` pada sebuah iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Perhatikan bahwa kita harus membikin `v1_iter` jadi _mutable_: memanggil 
method `next` pada sebuah iterator bakal mengubah _state_ (keadaan) internal 
yang dipakai sama iterator tersebut untuk melacak (keep track of) ada di mana 
dia saat ini di urutan tersebut. Dengan kata lain, kode ini mengonsumsi 
(_consumes_), atau menghabiskan, iterator-nya. Setiap pemanggilan ke `next` 
bakal "memakan" satu item dari iterator-nya. Kita tidak perlu membikin 
`v1_iter` jadi _mutable_ saat kita memakai _for loop_ karena _loop_ tersebut 
mengambil kepemilikan (ownership) dari `v1_iter` dan membikinnya _mutable_ di 
balik layar.

Perhatikan juga bahwa nilai yang kita dapat dari pemanggilan `next` adalah 
referensi _immutable_ ke nilai-nilai yang ada di vector-nya. Method `iter` 
menghasilkan sebuah iterator yang berisi referensi _immutable_. Kalau kita 
mau membuat iterator yang mengambil kepemilikan dari `v1` lalu mengembalikan 
nilai yang _owned_ (dimiliki), kita bisa memanggil `into_iter` alih-alih `iter`. 
Sama juga halnya, kalau kita mau beriterasi melewati referensi _mutable_, kita 
bisa memanggil `iter_mut` alih-alih `iter`.

### Method-method yang Mengonsumsi Iterator

Trait `Iterator` punya sejumlah method berbeda dengan implementasi default 
yang disediakan oleh _standard library_; kita bisa tahu soal method-method 
ini dengan melihat dokumentasi API _standard library_ untuk trait `Iterator`. 
Beberapa dari method ini memanggil method `next` di dalam definisi mereka, 
dan inilah alasannya kenapa kita diwajibkan buat mengimplementasikan method 
`next` saat kita mau mengimplementasikan trait `Iterator`.

Method-method yang memanggil `next` ini disebut _consuming adapters_, karena 
memanggil mereka bakal menghabiskan iterator-nya. Salah satu contohnya adalah 
method `sum`, yang mengambil kepemilikan dari iterator lalu beriterasi melewati 
item-itemnya dengan memanggil `next` secara berulang, yang mana ini bakal 
mengonsumsi iterator tersebut. Selama ia beriterasi, method ini menambahkan 
tiap item ke dalam sebuah total berjalan (running total) lalu mengembalikan 
totalnya saat iterasi selesai. Listing 13-13 punya contoh tes yang 
mengilustrasikan pemakaian method `sum`.

<Listing number="13-13" file-name="src/lib.rs" caption="Memanggil method `sum` buat mendapatkan total dari semua item di dalam iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Kita tidak diizinkan buat memakai `v1_iter` lagi setelah memanggil `sum` karena 
`sum` mengambil kepemilikan dari iterator tempat dia dipanggil.

### Method-method yang Menghasilkan Iterator Lain

_Iterator adapters_ adalah method-method yang didefinisikan pada trait `Iterator` 
yang tidak mengonsumsi iterator-nya. Alih-alih mengonsumsi, mereka malah 
menghasilkan iterator-iterator yang berbeda dengan cara mengubah beberapa aspek 
dari iterator aslinya.

Listing 13-14 menunjukkan contoh dari pemanggilan method iterator adapter `map`, 
yang menerima sebuah _closure_ buat dipanggil pada setiap item saat item-item 
tersebut dilewati dalam proses iterasi. Method `map` mengembalikan sebuah 
iterator baru yang menghasilkan item-item yang sudah dimodifikasi tersebut. 
_Closure_ di sini membuat iterator baru di mana tiap item dari vector bakal 
ditambahkan 1.

<Listing number="13-14" file-name="src/main.rs" caption="Memanggil iterator adapter `map` buat membuat iterator baru">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Namun, kode ini menghasilkan sebuah peringatan (warning):

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Kode di Listing 13-14 tidak melakukan apa-apa; _closure_ yang sudah kita 
tentukan itu tidak akan pernah dipanggil. Peringatannya mengingatkan kita soal 
alasannya kenapa: _iterator adapters_ itu _lazy_ (malas), dan kita harus 
mengonsumsi iterator-nya di sini.

Untuk membereskan peringatan ini dan mengonsumsi iterator-nya, kita bakal 
memakai method `collect`, yang sudah kita pakai bersama `env::args` di Listing 
12-1. Method ini mengonsumsi iterator-nya lalu mengumpulkan (collects) nilai-
nilai yang dihasilkan ke dalam sebuah tipe data koleksi.

Di Listing 13-15, kita mengumpulkan hasil dari iterasi iterator yang dikembalikan 
oleh panggilan ke `map` menjadi sebuah vector. Vector ini nantinya bakal berisi 
setiap item dari vector aslinya, masing-masing ditambah 1.

<Listing number="13-15" file-name="src/main.rs" caption="Memanggil method `map` buat membuat iterator baru, lalu memanggil method `collect` buat mengonsumsi iterator baru itu dan membuat sebuah vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Karena `map` menerima sebuah _closure_, kita bisa menentukan operasi apa pun yang 
mau kita lakukan pada tiap itemnya. Ini adalah contoh yang bagus tentang gimana 
_closures_ memungkinkan kita buat mengkustomisasi beberapa perilaku tertentu 
sambil menggunakan kembali perilaku iterasi yang disediakan sama trait `Iterator`.

Kita bisa menyambung (chain) beberapa panggilan ke berbagai _iterator adapters_ 
buat melakukan tindakan yang rumit pakai cara yang tetap enak dibaca. Tapi karena 
semua iterator itu _lazy_, kita harus memanggil salah satu dari method _consuming 
adapter_ buat mendapatkan hasil dari rentetan panggilan ke _iterator adapters_ 
tersebut.

### Menggunakan Closures yang Menangkap Lingkungannya

Banyak _iterator adapters_ menerima _closures_ sebagai argumen, dan biasanya 
_closures_ yang bakal kita berikan sebagai argumen ke _iterator adapters_ itu 
adalah _closures_ yang bakal menangkap lingkungan (environment) mereka.

Untuk contoh ini, kita bakal memakai method `filter` yang menerima sebuah 
_closure_. _Closure_ ini mengambil item dari iterator lalu mengembalikan sebuah 
`bool`. Kalau _closure_-nya mengembalikan `true`, nilainya bakal diikutsertakan 
dalam iterasi yang dihasilkan oleh `filter`. Kalau _closure_-nya mengembalikan 
`false`, nilainya tidak bakal diikutsertakan.

Di Listing 13-16, kita memakai `filter` bersama sebuah _closure_ yang menangkap 
variabel `shoe_size` dari lingkungannya untuk beriterasi melewati sekumpulan 
instance struct `Shoe`. Ini cuma bakal mengembalikan sepatu-sepatu yang punya 
ukuran sama dengan yang ditentukan.

<Listing number="13-16" file-name="src/lib.rs" caption="Memakai method `filter` bersama sebuah _closure_ yang menangkap `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

Fungsi `shoes_in_size` mengambil kepemilikan dari sebuah vector sepatu (shoes) 
dan ukuran sepatu sebagai parameternya. Ia mengembalikan vector yang hanya 
berisi sepatu-sepatu dengan ukuran yang ditentukan tersebut.

Di dalam body dari `shoes_in_size`, kita memanggil `into_iter` buat membuat 
sebuah iterator yang mengambil kepemilikan dari vector-nya. Kemudian kita 
memanggil `filter` buat mengadaptasi iterator itu menjadi iterator baru yang 
hanya mengandung elemen-elemen yang bikin _closure_-nya mengembalikan `true`.

_Closure_-nya menangkap parameter `shoe_size` dari lingkungan di sekitarnya dan 
membandingkan nilai itu dengan setiap ukuran dari sepatunya, mempertahankan 
hanya sepatu-sepatu dengan ukuran yang sesuai. Terakhir, memanggil `collect` 
bakal mengumpulkan (gathers) nilai-nilai yang dikembalikan oleh iterator yang 
diadaptasi ini ke dalam sebuah vector yang kemudian dikembalikan oleh fungsi 
tersebut.

Pengujian ini menunjukkan bahwa saat kita memanggil `shoes_in_size`, kita cuma 
bakal dapat balik sepatu-sepatu yang punya ukuran yang sama dengan nilai yang 
kita tentukan.
