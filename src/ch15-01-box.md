## Memakai `Box<T>` buat Menunjuk ke Data di Heap

_Smart pointer_ yang paling sederhana adalah *box* (kotak), yang tipenya 
ditulis `Box<T>`. _Boxes_ memungkinkan kita buat nyimpan data di _heap_ 
ketimbang di _stack_. Apa yang tersisa di _stack_ adalah si pointer yang 
menunjuk ke data di _heap_ tersebut. Silakan merujuk lagi ke Bab 4 buat 
me-review perbedaan antara _stack_ dan _heap_.

Boxes tidak punya _overhead_ performa, selain harus menyimpan data mereka 
di _heap_ alih-alih di _stack_. Tapi mereka juga tidak punya banyak 
kemampuan ekstra. Kita bakal paling sering memakainya di situasi-situasi 
berikut:

- Saat kita punya sebuah tipe yang ukurannya tidak bisa diketahui secara 
  pasti pas _compile time_ dan kita mau memakai sebuah nilai dari tipe itu 
  di konteks yang membutuhkan ukuran yang persis
- Saat kita punya jumlah data yang besar dan kita mau mentransfer 
  kepemilikannya (ownership) tapi tetap pengen memastikan kalau datanya 
  tidak bakal disalin (copied) saat kita melakukannya
- Saat kita pengen memiliki sebuah nilai dan kita cuma peduli kalau nilai 
  itu adalah tipe yang mengimplementasikan _trait_ tertentu, bukannya 
  merupakan suatu tipe spesifik

Kita bakal mendemonstrasikan situasi pertama di [“Memungkinkan Tipe Rekursif 
dengan Boxes”](#memungkinkan-tipe-rekursif-dengan-boxes). Di kasus kedua, 
mentransfer kepemilikan dari data yang sangat besar bisa memakan waktu 
lama karena datanya bakal disalin mondar-mandir di _stack_. Buat 
meningkatkan performa di situasi ini, kita bisa menyimpan jumlah data 
yang besar itu di _heap_ di dalam sebuah _box_. Dengan begitu, cuma data 
pointer yang kecil itu doang yang bakal disalin mondar-mandir di _stack_, 
sementara data yang ditunjuknya tetap diam di satu tempat di _heap_. 
Kasus ketiga dikenal sebagai _trait object_, dan [“Memakai Trait Objects yang 
Mengizinkan Nilai Dari Tipe yang Berbeda-beda”][trait-objects] di Bab 18 
memang dikhususkan untuk membahas topik tersebut. Jadi, apa yang kita 
pelajari di sini bakal kita terapkan lagi di bagian itu!

### Memakai `Box<T>` buat Menyimpan Data di Heap

Sebelum kita membahas kegunaan penyimpanan di _heap_ untuk `Box<T>`, kita 
bakal membahas sintaksnya dan gimana cara berinteraksi sama nilai yang 
disimpan di dalam `Box<T>`.

Listing 15-1 menunjukkan gimana cara memakai _box_ buat menyimpan nilai `i32` 
di _heap_.

<Listing number="15-1" file-name="src/main.rs" caption="Menyimpan nilai `i32` di _heap_ menggunakan sebuah box">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

Kita mendefinisikan variabel `b` buat punya nilai berupa sebuah `Box` yang 
menunjuk ke nilai `5`, yang mana nilainya itu dialokasikan di _heap_. Program 
ini bakal mencetak `b = 5`; di kasus ini, kita bisa mengakses data yang 
ada di dalam _box_ ini mirip kayak kalau datanya ada di _stack_. Sama 
kayak nilai (owned value) lainnya, pas sebuah _box_ keluar dari *scope*, 
sebagaimana yang terjadi pada `b` di akhir dari `main`, dia bakal di-_deallocate_ 
(dihapus dari memori). Proses dealokasi ini terjadi baik untuk _box_-nya 
(yang disimpan di _stack_) maupun untuk data yang ditunjuknya (yang 
disimpan di _heap_).

Menaruh satu nilai tunggal di _heap_ itu tidak terlalu berguna, jadi kita 
tidak akan terlalu sering memakai _boxes_ sendirian kayak gini. Membiarkan 
nilai-like satu `i32` tunggal ada di _stack_, di mana memang di situlah 
mereka disimpan secara default, itu lebih tepat buat mayoritas situasi. Mari 
kita lihat sebuah kasus di mana _boxes_ memungkinkan kita buat mendefinisikan 
tipe yang tidak bakal diizinkan untuk didefinisikan kalau kita tidak punya _boxes_.

### Memungkinkan Tipe Rekursif dengan Boxes

Nilai dari sebuah _tipe rekursif_ (recursive type) bisa punya nilai lain 
dari tipe yang sama sebagai bagian dari dirinya sendiri. Tipe rekursif ini 
menimbulkan masalah karena Rust perlu tahu saat _compile time_ seberapa banyak 
ruang (_space_) yang dipakai sama sebuah tipe. Namun, nilai yang bersarang 
(nesting) di tipe rekursif ini secara teoritis bisa terus berlanjut tanpa 
batas, jadi Rust tidak bisa tahu berapa banyak ruang yang dibutuhkan sama nilai 
tersebut. Karena _boxes_ punya ukuran yang sudah pasti diketahui, kita bisa 
memungkinkan tipe rekursif ini dengan menyelipkan (inserting) sebuah _box_ ke 
dalam definisi tipe rekursifnya.

Sebagai contoh tipe rekursif, mari kita eksplorasi _cons list_. Ini adalah 
tipe data yang sering sekali dijumpai di bahasa pemrograman fungsional. Tipe 
_cons list_ yang bakal kita definisikan itu mudah dipahami kecuali pada 
bagian rekursinya; maka dari itu, konsep-konsep di dalam contoh yang bakal 
kita kerjakan ini bakal berguna kapan pun kita masuk ke situasi yang lebih 
kompleks yang melibatkan tipe rekursif.

#### Info Lebih Lanjut soal Cons List

Sebuah _cons list_ adalah struktur data yang berasal dari bahasa pemrograman 
Lisp dan dialek-dialeknya, yang disusun dari pasangan (pairs) yang bersarang, 
dan ini adalah versi Lisp dari _linked list_. Namanya datang dari fungsi 
`cons` (kependekan dari fungsi _construct_) di Lisp yang mengonstruksi sebuah 
pasangan baru dari dua argumennya. Dengan memanggil `cons` pada sebuah pasangan 
yang terdiri dari sebuah nilai dan pasangan lain, kita bisa mengonstruksi 
_cons lists_ yang terbuat dari pasangan yang rekursif.

Misalnya, ini adalah representasi _pseudocode_ (kode semu) dari sebuah _cons list_ 
yang berisi list `1, 2, 3` dengan setiap pasangan ditaruh di dalam tanda kurung:

```text
(1, (2, (3, Nil)))
```

Tiap item di dalam _cons list_ terdiri dari dua elemen: nilai dari item saat ini 
dan item selanjutnya. Item terakhir di dalam list hanya terdiri dari sebuah 
nilai bernama `Nil` tanpa ada item berikutnya. Sebuah _cons list_ diproduksi dengan 
memanggil fungsi `cons` secara rekursif. Nama standar (canonical name) untuk 
menyebut kasus dasar (_base case_) dari rekursi ini adalah `Nil`. Perhatikan 
bahwa ini tidak sama dengan konsep “null” atau “nil” yang dibahas di Bab 6, yang 
mana itu artinya nilai yang tidak valid atau absen.

_Cons list_ bukanlah struktur data yang sering dipakai di Rust. Kebanyakan 
waktunya saat kita punya sebuah list yang berisi item-item di Rust, `Vec<T>` 
adalah pilihan yang lebih baik buat dipakai. Namun, tipe data rekursif lainnya 
yang lebih kompleks _memang_ berguna di berbagai situasi, tapi dengan memulai 
pakai _cons list_ di bab ini, kita bisa mengeksplorasi gimana _boxes_ memungkinkan 
kita buat mendefinisikan tipe data rekursif tanpa banyak gangguan.

Listing 15-2 mengandung sebuah definisi enum untuk sebuah _cons list_. 
Perhatikan kalau kode ini belum bisa di-compile karena tipe `List` tidak punya 
ukuran yang pasti, yang mana bakal kita demonstrasikan.

<Listing number="15-2" file-name="src/main.rs" caption="Usaha pertama buat mendefinisikan sebuah enum buat merepresentasikan struktur data _cons list_ dari nilai `i32`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> Catatan: Kita mengimplementasikan sebuah _cons list_ yang menampung cuma nilai 
> `i32` aja demi tujuan contoh ini. Kita bisa saja mengimplementasikannya memakai 
> generik (generics), seperti yang sudah kita bahas di Bab 10, buat mendefinisikan 
> tipe _cons list_ yang bisa menyimpan nilai dari tipe apa pun.

Memakai tipe `List` buat menyimpan list `1, 2, 3` bakal kelihatan seperti kode 
di Listing 15-3.

<Listing number="15-3" file-name="src/main.rs" caption="Memakai enum `List` buat menyimpan list `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

Nilai `Cons` pertama menampung `1` dan nilai `List` lain. Nilai `List` ini 
adalah nilai `Cons` lain yang menampung `2` dan sebuah nilai `List` lainnya. 
Nilai `List` ini adalah satu lagi nilai `Cons` yang menampung `3` dan sebuah 
nilai `List`, yang mana pada akhirnya adalah `Nil`, yaitu varian non-rekursif yang 
menandakan akhir dari list tersebut.

Kalau kita nyoba men-compile kode di Listing 15-3, kita dapat error yang 
ditunjukkan di Listing 15-4.

<Listing number="15-4" caption="Error yang kita dapat saat mencoba mendefinisikan sebuah enum yang rekursif">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

Error-nya bilang kalau tipe ini “punya ukuran tak terbatas” (has infinite size). 
Alasannya adalah karena kita mendefinisikan `List` dengan sebuah varian yang 
rekursif: ia memegang nilai lain dari dirinya sendiri secara langsung. Sebagai 
hasilnya, Rust tidak bisa menghitung seberapa banyak ruang yang ia butuhkan buat 
menyimpan sebuah nilai `List`. Mari kita pecahkan masalah kenapa kita dapat 
error ini. Pertama kita bakal melihat gimana cara Rust memutuskan berapa banyak 
ruang yang ia butuhkan buat menyimpan nilai dari tipe non-rekursif.

#### Menghitung Ukuran dari Tipe Non-Rekursif

Ingat kembali enum `Message` yang kita definisikan di Listing 6-2 saat kita 
membahas definisi enum di Bab 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Buat menentukan seberapa banyak ruang yang harus dialokasikan untuk nilai 
`Message`, Rust menelusuri setiap varian yang ada buat melihat varian mana yang 
butuh ruang paling banyak. Rust melihat kalau `Message::Quit` tidak butuh ruang 
sama sekali, `Message::Move` butuh ruang yang cukup buat menyimpan dua nilai `i32`, 
dan seterusnya. Karena cuma satu varian saja yang bakal dipakai dalam satu waktu, 
ruang maksimal yang bakal dibutuhkan oleh sebuah nilai `Message` adalah ruang yang 
dibutuhkan buat menyimpan varian terbesarnya.

Bandingkan ini sama apa yang terjadi saat Rust mencoba menentukan seberapa banyak 
ruang yang dibutuhkan oleh sebuah tipe rekursif kayak enum `List` di Listing 15-2. 
_Compiler_ mulai dengan melihat varian `Cons`, yang mana memegang sebuah nilai 
tipe `i32` dan sebuah nilai tipe `List`. Maka dari itu, `Cons` butuh jumlah ruang 
yang setara dengan ukuran dari `i32` ditambah sama ukuran dari `List`. Buat 
menghitung seberapa besar memori yang dibutuhkan oleh tipe `List`, _compiler_ 
melihat variannya, yang dimulai dengan varian `Cons`. Varian `Cons` memegang nilai 
bertipe `i32` dan sebuah nilai bertipe `List`, dan proses ini terus berlanjut tanpa 
batas, seperti yang ditunjukkan di Gambar 15-1.

<img alt="Sebuah Cons list yang tak terhingga: sebuah persegi panjang dengan label 'Cons' dibelah jadi dua persegi panjang yang lebih kecil. Persegi panjang kecil pertama berisi label 'i32', dan persegi panjang kecil kedua berisi label 'Cons' dan sebuah versi kecil dari persegi panjang 'Cons' yang ada di luarnya. Persegi panjang 'Cons' ini bakal terus memegang versi diri mereka sendiri yang makin mengecil sampai akhirnya sebuah persegi panjang yang berukuran wajar memegang sebuah simbol tak terhingga (infinity), yang menandakan kalau perulangan ini terjadi selamanya" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Gambar 15-1: Sebuah `List` yang tidak terhingga yang terdiri 
dari varian `Cons` yang tidak terhingga juga</span>

#### Memakai `Box<T>` buat Mendapatkan Tipe Rekursif dengan Ukuran Pasti

Karena Rust tidak bisa mencari tahu seberapa banyak ruang yang harus dialokasikan 
buat tipe-tipe yang definisinya rekursif, _compiler_ mengeluarkan error dengan 
saran yang ngebantu ini:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Di saran ini, _indirection_ berarti bahwa ketimbang menyimpan nilainya secara 
langsung, kita sebaiknya mengubah struktur datanya buat menyimpan nilai itu 
secara tidak langsung dengan menyimpan *pointer* yang merujuk ke nilai tersebut.

Karena sebuah `Box<T>` itu adalah sebuah *pointer*, Rust bakal selalu tahu 
seberapa banyak ruang yang dibutuhkan oleh sebuah `Box<T>`: ukuran dari sebuah 
*pointer* tidak bakal berubah tidak peduli seberapa banyak data yang dia tunjuk. 
Ini artinya kita bisa menaruh sebuah `Box<T>` di dalam varian `Cons` ketimbang 
menaruh nilai `List` lain secara langsung. `Box<T>` tersebut bakal menunjuk ke 
nilai `List` berikutnya yang mana bakal berada di _heap_ dan bukannya berada di 
dalam varian `Cons`. Secara konsep, kita masih punya sebuah list, yang dibikin 
dari list yang memegang list lainnya, tapi implementasi yang ini sekarang lebih 
mirip dengan menaruh item-item tersebut bersebelahan satu sama lain ketimbang 
di dalam satu sama lain.

Kita bisa mengubah definisi dari enum `List` di Listing 15-2 dan pemakaian dari 
`List` di Listing 15-3 menjadi kode di Listing 15-5, yang mana sekarang bakal 
bisa di-compile.

<Listing number="15-5" file-name="src/main.rs" caption="Definisi `List` yang memakai `Box<T>` biar bisa punya ukuran yang pasti diketahui">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

Varian `Cons` butuh ukuran sebesar sebuah `i32` ditambah sama ruang buat 
menyimpan data _pointer_ dari _box_ tersebut. Varian `Nil` tidak menyimpan nilai 
apa pun, jadi ia butuh lebih sedikit ruang di _stack_ dibandingkan varian `Cons`. 
Kita sekarang tahu kalau nilai `List` apa pun cuma bakal memakan ruang sebesar `i32` 
ditambah dengan ukuran dari data _pointer_ di _box_-nya. Dengan memakai sebuah _box_, 
kita sudah memutuskan (broken) rantai yang tidak terhingga dan rekursif tadi, 
jadi _compiler_ sekarang bisa menghitung ukuran yang dia butuhkan buat menyimpan 
nilai `List`. Gambar 15-2 menunjukkan seperti apa rupa dari varian `Cons` itu 
sekarang.

<img alt="Sebuah persegi panjang berlabel 'Cons' dibelah jadi dua persegi panjang yang lebih kecil. Persegi panjang kecil pertama memegang label 'i32', dan persegi panjang kecil kedua memegang label 'Box' dengan satu persegi panjang di dalamnya yang mengandung label 'usize', merepresentasikan ukuran terbatas dari pointer _box_ tersebut" src="img/trpl15-02.svg" class="center" />

<span class="caption">Gambar 15-2: Sebuah `List` yang ukurannya tidak lagi tidak 
terhingga karena `Cons` sekarang memegang sebuah `Box`</span>

_Boxes_ hanya menyediakan proses penyimpanan tidak langsung (indirection) 
beserta alokasi di _heap_; mereka tidak punya kapabilitas spesial lainnya, seperti 
yang bakal kita lihat di tipe-tipe _smart pointer_ lainnya. Mereka juga tidak 
punya _overhead_ performa dari kapabilitas spesial tersebut, jadi mereka bakal 
berguna di kasus-cases seperti *cons list* di mana _indirection_ itu adalah 
satu-satunya fitur yang kita butuhin. Kita bakal melihat lebih banyak contoh 
pemakaian buat _boxes_ di Bab 18.

Tipe `Box<T>` adalah sebuah _smart pointer_ karena ia mengimplementasikan trait 
`Deref`, yang memungkinkan nilai `Box<T>` buat diperlakukan seperti layaknya 
sebuah referensi biasa. Pas sebuah nilai `Box<T>` keluar dari *scope*, data 
di _heap_ yang ditunjuk sama _box_ tersebut juga bakal dibersihkan karena adanya 
implementasi trait `Drop`. Dua trait ini bakal jadi lebih penting lagi dalam 
memahami fungsionalitas yang disediakan oleh tipe-tipe _smart pointer_ lain 
yang bakal kita bahas di sisa bab ini. Mari kita eksplorasi dua trait ini secara 
lebih detail.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
