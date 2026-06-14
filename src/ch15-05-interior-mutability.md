## `RefCell<T>` dan Desain Pola Interior Mutability

_Interior mutability_ (mutabilitas interior) adalah sebuah desain pola di Rust 
yang membiarkan kita memutasi (mengubah) data bahkan saat ada referensi 
_immutable_ ke data tersebut; normalnya, tindakan ini tidak diizinkan oleh 
aturan _borrowing_. Buat memutasi data, pola ini memakai kode `unsafe` di dalam 
sebuah struktur data untuk membengkokkan aturan-aturan normal Rust yang mengatur 
mutasi dan _borrowing_. Kode `unsafe` mengindikasikan ke _compiler_ bahwa kita 
memeriksa aturan-aturan tersebut secara manual ketimbang mengandalkan 
_compiler_ buat mengeceknya untuk kita; kita bakal membahas kode `unsafe` 
lebih banyak di Bab 20.

Kita bisa memakai tipe yang menggunakan desain pola _interior mutability_ cuma 
pas kita bisa memastikan kalau aturan _borrowing_ bakal dipatuhi saat _runtime_, 
meskipun _compiler_ tidak bisa menjamin hal itu. Kode `unsafe` yang terlibat 
kemudian dibungkus di dalam sebuah API yang aman (safe API), dan tipe yang ada 
di luar bakal tetap _immutable_.

Mari kita eksplorasi konsep ini dengan melihat tipe `RefCell<T>` yang mengikuti 
desain pola _interior mutability_ ini.

### Menegakkan Aturan Borrowing saat Runtime dengan `RefCell<T>`

Tidak seperti `Rc<T>`, tipe `RefCell<T>` merepresentasikan kepemilikan tunggal 
(single ownership) atas data yang dipegangnya. Jadi apa yang membikin `RefCell<T>` 
berbeda dari tipe seperti `Box<T>`? Ingat kembali aturan _borrowing_ yang 
kita pelajari di Bab 4:

- Pada waktu kapan pun, kita cuma boleh punya *satu* referensi _mutable_ atau 
  sejumlah referensi _immutable_ (tapi tidak boleh dua-duanya sekaligus).
- Referensi harus selalu valid.

Dengan referensi biasa dan `Box<T>`, invarian (aturan mutlak) dari aturan 
_borrowing_ ini ditegakkan (enforced) saat _compile time_. Dengan `RefCell<T>`, 
invarian ini ditegakkan *saat runtime*. Dengan referensi, kalau kita 
melanggar aturan-aturan ini, kita bakal dapat error _compiler_. Dengan 
`RefCell<T>`, kalau kita melanggar aturan-aturan ini, program kita bakal 
mengalami _panic_ dan keluar (exit).

Keuntungan dari mengecek aturan _borrowing_ saat _compile time_ adalah 
error bakal ditangkap lebih awal di proses _development_ (pengembangan), dan 
tidak ada dampak pada performa _runtime_ karena semua analisisnya udah 
diselesaikan sebelumnya. Karena alasan-alasan tersebut, mengecek aturan 
_borrowing_ saat _compile time_ adalah pilihan terbaik buat mayoritas kasus, 
yang mana itulah alasannya kenapa ini jadi default (bawaan) di Rust.

Keuntungan dari mengecek aturan _borrowing_ saat _runtime_ sebagai gantinya 
adalah ada beberapa skenario aman-memori (memory-safe) yang kemudian jadi 
diizinkan, di mana mereka tadinya tidak bakal diizinkan sama pengecekan 
_compile time_. Analisis statis, seperti _compiler_ Rust, secara bawaan sifatnya 
konservatif. Beberapa properti (sifat) kode mustahil buat dideteksi dengan 
hanya menganalisis kodenya saja: contoh paling terkenal adalah _Halting 
Problem_, yang ada di luar cakupan buku ini tapi merupakan topik yang 
menarik buat diteliti.

Karena beberapa analisis itu mustahil, kalau _compiler_ Rust tidak bisa yakin 
kodenya mematuhi aturan _ownership_, ia mungkin bakal menolak sebuah program 
yang sebenarnya benar; di sinilah letak sifat konservatifnya. Kalau Rust 
menerima program yang salah, para _user_ tidak bakal bisa mempercayai jaminan 
yang diberikan Rust. Namun, kalau Rust menolak program yang benar, si 
programmer bakal direpotkan, tapi tidak ada bencana besar yang bakal 
terjadi. Tipe `RefCell<T>` ini berguna saat kita yakin kode kita mematuhi 
aturan _borrowing_ tapi _compiler_ tidak mampu memahami dan menjamin hal 
tersebut.

Sama halnya dengan `Rc<T>`, `RefCell<T>` cuma buat dipakai di skenario 
_single-threaded_ dan bakal ngasih kita error _compile-time_ kalau kita 
mencoba memakainya di konteks _multithreaded_. Kita bakal bahas gimana 
cara mendapatkan fungsionalitas dari `RefCell<T>` di dalam program 
_multithreaded_ di Bab 16.

Berikut adalah rangkuman (recap) dari alasan memilih `Box<T>`, `Rc<T>`, 
atau `RefCell<T>`:

- `Rc<T>` memungkinkan banyak pemilik (multiple owners) dari data yang sama; 
  `Box<T>` dan `RefCell<T>` cuma punya pemilik tunggal (single owners).
- `Box<T>` mengizinkan _borrows_ (peminjaman) _immutable_ atau _mutable_ 
  yang dicek saat _compile time_; `Rc<T>` cuma mengizinkan _borrows_ 
  _immutable_ yang dicek saat _compile time_; `RefCell<T>` mengizinkan 
  _borrows_ _immutable_ atau _mutable_ yang dicek saat _runtime_.
- Karena `RefCell<T>` mengizinkan _borrows_ _mutable_ yang dicek saat _runtime_, 
  kita bisa memutasi nilai di dalam `RefCell<T>` bahkan saat `RefCell<T>` 
  itu sendiri sifatnya _immutable_.

Memutasi nilai yang ada di dalam sebuah nilai yang _immutable_ itulah yang 
disebut desain pola _interior mutability_. Mari kita lihat sebuah situasi di 
mana _interior mutability_ itu berguna dan memeriksa gimana hal itu bisa 
dilakukan.

### Interior Mutability: Peminjaman Mutable ke sebuah Nilai yang Immutable

Sebuah konsekuensi dari aturan _borrowing_ adalah pas kita punya nilai yang 
_immutable_, kita tidak bisa meminjamnya secara _mutable_. Misalnya, kode ini 
tidak bakal bisa di-compile:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Kalau kita mencoba men-compile kode ini, kita bakal dapat error berikut:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Namun, ada beberapa situasi di mana bakal berguna buat sebuah nilai buat 
bisa memutasi dirinya sendiri di dalam method-methodnya tapi terlihat 
_immutable_ bagi kode lain. Kode di luar method dari nilai tersebut tidak 
bakal bisa memutasi nilai itu. Memakai `RefCell<T>` adalah salah satu 
cara buat bisa mendapatkan kemampuan buat punya _interior mutability_, tapi 
`RefCell<T>` tidak sepenuhnya menghindari aturan _borrowing_: _borrow checker_ 
di dalam _compiler_ memang mengizinkan _interior mutability_ ini, dan sebagai 
gantinya aturan _borrowing_ dicek saat _runtime_. Kalau kita melanggar 
aturan-aturannya, kita bakal dapat `panic!` bukannya error _compiler_.

Mari kita kerjakan sebuah contoh praktis di mana kita bisa memakai 
`RefCell<T>` buat memutasi nilai yang _immutable_ dan melihat kenapa hal 
itu berguna.

#### Use Case untuk Interior Mutability: Mock Objects

Kadang-kadang saat lagi _testing_ (pengujian), seorang programmer bakal 
memakai sebuah tipe sebagai pengganti dari tipe lain, untuk mengobservasi 
perilaku tertentu dan menegaskan kalau perilakunya diimplementasikan 
dengan benar. Tipe _placeholder_ ini disebut _test double_ (pengganti tes). 
Bayangkan saja seperti pemeran pengganti (_stunt double_) di pembuatan film, 
di mana seseorang masuk dan menggantikan sang aktor buat melakukan adegan yang 
sangat berbahaya (_tricky scene_). _Test doubles_ menggantikan tipe lain 
saat kita sedang menjalankan pengujian. _Mock objects_ (objek pura-pura) 
adalah jenis _test double_ spesifik yang merekam (records) apa yang terjadi 
selama pengujian berjalan sehingga kita bisa menegaskan kalau aksi yang benar 
telah terjadi.

Rust tidak punya _objects_ (objek) dalam artian yang sama kayak bahasa lain 
punya _objects_, dan Rust tidak punya fungsionalitas _mock object_ bawaan 
di _standard library_ seperti yang ada di beberapa bahasa lain. Namun, kita 
tetap bisa membikin sebuah struct yang bakal melayani tujuan yang sama dengan 
sebuah _mock object_.

Ini skenario yang bakal kita uji: kita bakal bikin sebuah _library_ yang melacak 
(_tracks_) sebuah nilai berdasarkan nilai maksimalnya dan mengirim pesan 
berdasarkan seberapa dekat nilai saat ini ke nilai maksimalnya. _Library_ ini 
bisa dipakai buat melacak kuota _user_ untuk jumlah pemanggilan API yang 
diizinkan buat mereka, contohnya.

_Library_ kita cuma bakal menyediakan fungsionalitas buat melacak seberapa 
dekat suatu nilai ke nilai maksimalnya dan apa pesan yang seharusnya muncul 
di waktu kapan aja. Aplikasi-aplikasi yang memakai _library_ kita bakal 
diharapkan buat menyediakan mekanisme pengiriman pesannya: aplikasi itu bisa 
aja menaruh pesannya di dalam aplikasinya, mengirim email, mengirim pesan 
teks (SMS), atau melakukan hal lain. _Library_-nya tidak perlu tahu detail 
tersebut. Yang ia butuhkan cuma sesuatu yang mengimplementasikan sebuah trait 
yang bakal kita sediakan bernama `Messenger`. Listing 15-20 menunjukkan kode 
dari _library_ tersebut.

<Listing number="15-20" file-name="src/lib.rs" caption="Sebuah _library_ buat melacak seberapa dekat suatu nilai ke nilai maksimalnya dan ngasih peringatan saat nilai tersebut mencapai tingkat tertentu">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Satu bagian penting dari kode ini adalah trait `Messenger` punya satu method 
bernama `send` yang menerima sebuah referensi _immutable_ ke `self` dan teks 
pesannya. Trait ini adalah _interface_ (antarmuka) yang perlu 
diimplementasikan oleh _mock object_ kita sehingga si _mock_ (objek 
pura-pura) ini bisa dipakai dengan cara yang sama kayak objek aslinya dipakai. 
Bagian penting lainnya adalah kita mau menguji perilaku dari method 
`set_value` di `LimitTracker`. Kita bisa mengubah nilai apa yang kita teruskan 
ke dalam parameter `value`, tapi `set_value` tidak mengembalikan apa pun yang 
bisa kita pakai buat membuat penegasan (assertions). Kita mau bisa bilang kalau 
saat kita bikin sebuah `LimitTracker` dengan sesuatu yang mengimplementasikan 
trait `Messenger` dan sebuah nilai spesifik buat `max`, pas kita meneruskan 
angka-angka yang beda buat `value` maka si *messenger* (pengirim pesan) ini bakal 
disuruh mengirim pesan-pesan yang sesuai.

Kita butuh sebuah _mock object_ yang, alih-alih mengirim email atau SMS 
saat kita memanggil `send`, dia cuma bakal mencatat pesan-pesan apa aja yang 
disuruh untuk dikirim. Kita bisa membikin sebuah instance baru dari _mock object_ 
ini, membikin sebuah `LimitTracker` yang memakai _mock object_ itu, 
memanggil method `set_value` di `LimitTracker`, dan kemudian mengecek kalau 
_mock object_ itu memang punya pesan-pesan yang kita harapkan. Listing 15-21 
menunjukkan sebuah usaha buat mengimplementasikan sebuah _mock object_ buat 
melakukan hal itu persis, tapi _borrow checker_ tidak bakal mengizinkannya.

<Listing number="15-21" file-name="src/lib.rs" caption="Sebuah usaha buat mengimplementasikan sebuah `MockMessenger` yang mana tidak diizinkan oleh _borrow checker_">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Kode pengujian ini mendefinisikan sebuah struct `MockMessenger` yang punya field 
`sent_messages` berisi `Vec` dari nilai-nilai `String` buat melacak 
pesan-pesan apa aja yang dia disuruh buat kirim. Kita juga mendefinisikan 
fungsi _associated_ (terkait) `new` buat bikin nyaman pas membikin nilai 
`MockMessenger` baru yang dimulai dengan _list_ pesan yang kosong. Kita 
kemudian mengimplementasikan trait `Messenger` untuk `MockMessenger` supaya 
kita bisa ngasih sebuah `MockMessenger` ke sebuah `LimitTracker`. Di dalam 
definisi dari method `send`, kita mengambil pesan yang diberikan sebagai 
parameter lalu menyimpannya di _list_ `sent_messages` milik `MockMessenger`.

Di dalam pengujiannya, kita menguji apa yang terjadi saat `LimitTracker` 
disuruh buat nge-set `value` ke sesuatu yang jumlahnya lebih dari 75 persen 
dari nilai `max`. Pertama kita bikin sebuah `MockMessenger` baru, yang mana 
bakal dimulai dengan _list_ pesan yang kosong. Terus kita bikin sebuah 
`LimitTracker` baru dan ngasih dia sebuah referensi ke `MockMessenger` yang 
baru itu dan nilai `max` sebesar `100`. Kita memanggil method `set_value` di 
`LimitTracker` dengan nilai `80`, yang mana itu lebih dari 75 persen dari 100. 
Lalu kita menegaskan kalau _list_ pesan yang dilacak sama `MockMessenger` itu 
seharusnya sekarang punya satu pesan di dalamnya.

Namun, ada satu masalah dengan pengujian ini, seperti yang ditunjukkan di sini:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Kita tidak bisa memodifikasi `MockMessenger` buat melacak pesan-pesannya 
karena method `send` mengambil sebuah referensi _immutable_ ke `self`. 
Kita juga tidak bisa mengikuti saran dari teks error-nya buat memakai 
`&mut self` di method `impl` dan juga di definisi trait-nya. Kita tidak 
mau mengubah trait `Messenger` cuma demi bisa melakukan pengujian. 
Sebaliknya, kita harus cari cara buat bikin kode pengujian kita bekerja 
dengan benar dengan desain kita yang udah ada.

Ini adalah sebuah situasi di mana _interior mutability_ bisa membantu! 
Kita bakal menyimpan `sent_messages` di dalam sebuah `RefCell<T>`, dan kemudian 
method `send` bakal bisa memodifikasi `sent_messages` buat menyimpan 
pesan-pesan yang udah kita lihat. Listing 15-22 menunjukkan seperti apa rupa 
dari perubahan tersebut.

<Listing number="15-22" file-name="src/lib.rs" caption="Memakai `RefCell<T>` buat memutasi nilai internal (inner value) sembari nilai luarnya (outer value) dianggap _immutable_">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

Field `sent_messages` sekarang bertipe `RefCell<Vec<String>>` bukannya 
`Vec<String>`. Di dalam fungsi `new`, kita membikin sebuah instance 
`RefCell<Vec<String>>` baru di sekeliling _vector_ kosong.

Untuk implementasi method `send`-nya, parameter pertamanya tetap berupa 
pinjaman (borrow) _immutable_ dari `self`, yang mana cocok sama definisi 
trait-nya. Kita memanggil `borrow_mut` pada `RefCell<Vec<String>>` di dalam 
`self.sent_messages` buat dapet sebuah referensi _mutable_ ke nilai yang 
ada di dalam `RefCell<Vec<String>>`, yaitu vector-nya. Terus kita bisa 
manggil `push` pada referensi _mutable_ ke vector tersebut buat melacak 
pesan-pesan yang dikirim selama pengujiannya.

Perubahan terakhir yang harus kita buat adalah di penegasannya (assertion): 
buat melihat ada berapa banyak item yang ada di vector internalnya, kita 
memanggil `borrow` pada `RefCell<Vec<String>>` buat dapet sebuah referensi 
_immutable_ ke vector-nya.

Sekarang karena kita sudah melihat gimana cara memakai `RefCell<T>`, mari kita 
gali lebih dalam soal gimana cara kerjanya!

#### Melacak Borrows saat Runtime dengan `RefCell<T>`

Saat membikin referensi _immutable_ dan _mutable_, kita masing-masing 
memakai sintaks `&` dan `&mut`. Dengan `RefCell<T>`, kita memakai method 
`borrow` dan `borrow_mut`, yang merupakan bagian dari API aman yang dimiliki 
sama `RefCell<T>`. Method `borrow` mengembalikan tipe _smart pointer_ 
`Ref<T>`, dan `borrow_mut` mengembalikan tipe _smart pointer_ `RefMut<T>`. 
Kedua tipe ini mengimplementasikan `Deref`, jadi kita bisa memperlakukan 
mereka kayak referensi biasa.

`RefCell<T>` melacak berapa banyak _smart pointers_ `Ref<T>` dan 
`RefMut<T>` yang saat ini lagi aktif. Setiap kali kita memanggil `borrow`, 
`RefCell<T>` menaikkan hitungan (count) jumlah _borrows_ _immutable_ yang 
lagi aktif. Saat sebuah nilai `Ref<T>` keluar dari *scope*, hitungan dari 
_borrows_ _immutable_ itu turun sebanyak 1. Persis sama kayak aturan 
_borrowing_ di _compile-time_, `RefCell<T>` membiarkan kita punya banyak 
_borrows_ _immutable_ atau satu _borrow_ _mutable_ pada suatu waktu 
kapan pun.

Kalau kita mencoba melanggar aturan ini, bukannya dapat error _compiler_ 
kayak yang bakal kita dapat pas pakai referensi, implementasi dari 
`RefCell<T>` malah bakal *panic* di saat _runtime_. Listing 15-23 menunjukkan 
sebuah modifikasi dari implementasi `send` di Listing 15-22. Kita sengaja 
mencoba membikin dua _borrows_ _mutable_ aktif di _scope_ yang sama buat 
mengilustrasikan kalau `RefCell<T>` mencegah kita dari melakukan hal ini saat 
_runtime_.

<Listing number="15-23" file-name="src/lib.rs" caption="Membikin dua referensi _mutable_ di dalam _scope_ yang sama buat melihat kalau `RefCell<T>` bakal mengalami _panic_">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Kita membikin sebuah variabel `one_borrow` buat _smart pointer_ `RefMut<T>` 
yang dikembalikan dari `borrow_mut`. Terus kita membikin _borrow_ _mutable_ 
lainnya dengan cara yang sama di dalam variabel `two_borrow`. Ini membikin dua 
referensi _mutable_ di dalam _scope_ yang sama, yang mana tidak diizinkan. 
Saat kita menjalankan pengujian buat _library_ kita, kode di Listing 15-23 
bakal berhasil di-compile tanpa error apa pun, tapi pengujiannya bakal gagal:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Perhatikan kalau kodenya mengalami *panic* dengan pesan `already borrowed:
BorrowMutError`. Ini adalah gimana `RefCell<T>` menangani pelanggaran dari 
aturan _borrowing_ saat _runtime_.

Memilih buat menangkap error _borrowing_ saat _runtime_ ketimbang saat _compile 
time_, seperti yang udah kita lakuin di sini, berarti kita punya kemungkinan 
buat nemuin kesalahan di kode kita jauh belakangan di proses pengembangan: 
bahkan mungkin tidak bakal ketemu sampai kode kita sudah di-deploy ke 
*production* (produksi). Selain itu, kode kita bakal kena sedikit penalti 
(hukuman) performa _runtime_ akibat harus melacak _borrows_-nya pas 
_runtime_ ketimbang pas _compile time_. Namun, memakai `RefCell<T>` memungkinkan 
kita buat menulis sebuah _mock object_ yang bisa memodifikasi dirinya sendiri buat 
melacak pesan-pesan yang udah dilihatnya sementara kita memakainya di dalam sebuah 
konteks di mana cuma nilai _immutable_ yang diizinkan. Kita bisa memakai 
`RefCell<T>` meskipun ada *trade-offs* (kekurangannya) buat dapat fungsionalitas 
lebih banyak ketimbang yang disediakan oleh referensi biasa.

<!-- Old link, do not remove -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>

### Mengizinkan Kepemilikan Ganda (Multiple Owners) pada Data yang Mutable dengan `Rc<T>` dan `RefCell<T>`

Cara yang umum buat memakai `RefCell<T>` adalah dengan menggabungkannya bersama 
`Rc<T>`. Ingat kembali kalau `Rc<T>` membiarkan kita punya banyak pemilik 
dari suatu data, tapi ia cuma ngasih akses _immutable_ ke data tersebut. Kalau 
kita punya sebuah `Rc<T>` yang memegang sebuah `RefCell<T>`, kita bisa 
dapat sebuah nilai yang bisa punya banyak pemilik *dan* juga bisa kita mutasi 
(ubah)!

Sebagai contoh, ingat kembali contoh _cons list_ di Listing 15-18 di mana kita 
memakai `Rc<T>` agar beberapa _lists_ bisa berbagi kepemilikan dari sebuah _list_ 
lain. Karena `Rc<T>` cuma menampung nilai _immutable_, kita tidak bisa mengubah 
satu pun nilai di dalam _list_-nya setelah kita membuatnya. Mari kita 
menambahkan `RefCell<T>` agar kita bisa mendapatkan kemampuan buat mengubah 
nilai-nilai di dalam _lists_ tersebut. Listing 15-24 menunjukkan kalau dengan 
memakai sebuah `RefCell<T>` di dalam definisi `Cons`, kita bisa memodifikasi 
nilai yang disimpan di semua _lists_.

<Listing number="15-24" file-name="src/main.rs" caption="Memakai `Rc<RefCell<i32>>` buat bikin sebuah `List` yang bisa kita mutasi">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Kita membikin sebuah nilai yang merupakan sebuah instance dari 
`Rc<RefCell<i32>>` lalu menyimpannya ke dalam sebuah variabel bernama `value` 
supaya kita bisa mengaksesnya secara langsung nanti. Terus kita membikin 
sebuah `List` di dalam `a` dengan sebuah varian `Cons` yang memegang `value`. 
Kita perlu meng-_clone_ `value` sehingga baik `a` dan `value` dua-duanya punya 
kepemilikan dari nilai internal `5` tersebut ketimbang memindahkan (transferring) 
kepemilikan dari `value` ke `a` atau bikin `a` meminjam dari `value`.

Kita membungkus _list_ `a` di dalam sebuah `Rc<T>` sehingga saat kita membikin 
_lists_ `b` dan `c`, mereka berdua bisa merujuk ke `a`, persis kayak yang kita 
lakuin di Listing 15-18.

Setelah kita membikin _lists_ di `a`, `b`, dan `c`, kita mau menambahkan 10 ke 
nilai di dalam `value`. Kita melakukan ini dengan memanggil `borrow_mut` pada 
`value`, yang mana memakai fitur _automatic dereferencing_ yang udah kita bahas 
di [“Ke Mana Perginya Operator `->`?”][wheres-the---operator] di Bab 5 buat 
men-dereferensi `Rc<T>` ke nilai `RefCell<T>` di dalamnya. Method `borrow_mut` 
mengembalikan sebuah _smart pointer_ `RefMut<T>`, lalu kita memakai operator 
dereferensi padanya dan mengubah nilai internalnya.

Saat kita mencetak `a`, `b`, dan `c`, kita bisa melihat kalau mereka semua 
sekarang punya nilai yang udah dimodifikasi menjadi `15` bukannya `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Teknik ini lumayan keren! Dengan memakai `RefCell<T>`, dari luar kita punya 
sebuah nilai `List` yang terlihat _immutable_. Tapi kita bisa memakai 
method-method di `RefCell<T>` yang menyediakan akses ke _interior mutability_-nya 
sehingga kita bisa memodifikasi data kita saat kita butuhkan. Pengecekan aturan 
_borrowing_ saat _runtime_ ngelindungin kita dari *data races* (balapan data), 
dan kadang-kadang sepadan rasanya buat menukar sedikit kecepatan (_speed_) demi 
fleksibilitas di dalam struktur data kita ini. Perhatikan bahwa `RefCell<T>` 
tidak bakal bekerja buat kode yang _multithreaded_! `Mutex<T>` adalah versi 
yang _thread-safe_ (aman dari utas) dari `RefCell<T>`, dan kita bakal membahas 
`Mutex<T>` di Bab 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
