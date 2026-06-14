## Memperlakukan Smart Pointers seperti Referensi Biasa dengan `Deref`

Mengimplementasikan trait `Deref` memungkinkan kita buat mengkustomisasi 
perilaku dari _dereference operator_ (operator dereferensi) `*` (jangan 
tertukar sama operator perkalian atau operator glob). Dengan 
mengimplementasikan `Deref` sedemikian rupa sehingga sebuah _smart pointer_ 
bisa diperlakukan seperti referensi biasa, kita bisa menulis kode yang 
beroperasi pada referensi dan memakai kode tersebut buat _smart pointers_ 
juga.

Mari kita lihat dulu gimana cara kerja operator dereferensi pada referensi 
biasa. Kemudian kita bakal mencoba mendefinisikan sebuah tipe kustom yang 
berperilaku mirip `Box<T>`, dan melihat kenapa operator dereferensi tidak 
bekerja seperti referensi pada tipe yang baru kita definisikan itu. Kita 
bakal mengeksplorasi gimana mengimplementasikan trait `Deref` membikin 
_smart pointers_ mungkin buat bekerja dengan cara yang mirip seperti 
referensi. Kemudian kita bakal melihat fitur _deref coercion_ di Rust dan 
gimana ia membiarkan kita bekerja entah itu pakai referensi maupun pakai 
_smart pointers_.

### Mengikuti Referensi ke Nilainya

Sebuah referensi biasa adalah salah satu jenis _pointer_, dan salah satu 
cara buat membayangkan sebuah _pointer_ adalah sebagai tanda panah yang 
menunjuk ke sebuah nilai yang disimpan di tempat lain. Di Listing 15-6, 
kita membuat sebuah referensi ke nilai `i32` lalu memakai operator 
dereferensi buat mengikuti referensi tersebut ke nilainya.

<Listing number="15-6" file-name="src/main.rs" caption="Memakai operator dereferensi buat mengikuti sebuah referensi ke nilai `i32`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

Variabel `x` memegang sebuah nilai `i32` yaitu `5`. Kita menge-set `y` agar 
sama dengan sebuah referensi ke `x`. Kita bisa menegaskan (_assert_) kalau 
`x` itu sama dengan `5`. Namun, kalau kita mau bikin penegasan soal nilai 
di dalam `y`, kita harus memakai `*y` buat mengikuti referensi tersebut ke 
nilai yang lagi dia tunjuk (karenanya disebut _dereference_) supaya 
_compiler_ bisa membandingkan nilai aslinya. Begitu kita men-dereferensi `y`, 
kita punya akses ke nilai integer yang ditunjuk sama `y` yang bisa kita 
bandingkan dengan `5`.

Kalau kita malah mencoba nulis `assert_eq!(5, y);`, kita bakal dapat error 
kompilasi ini:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Membandingkan sebuah angka dan sebuah referensi ke angka itu tidak 
diperbolehkan karena mereka adalah tipe yang berbeda. Kita harus memakai 
operator dereferensi buat mengikuti referensi tersebut ke nilai yang 
ditunjuknya.

### Memakai `Box<T>` Layaknya Sebuah Referensi

Kita bisa menulis ulang kode di Listing 15-6 buat memakai sebuah `Box<T>` 
bukannya referensi; operator dereferensi yang dipakai pada `Box<T>` di 
Listing 15-7 berfungsi dengan cara yang sama kayak operator dereferensi 
yang dipakai pada referensi di Listing 15-6.

<Listing number="15-7" file-name="src/main.rs" caption="Memakai operator dereferensi pada sebuah `Box<i32>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Perbedaan utama antara Listing 15-7 dan Listing 15-6 adalah di sini kita 
menge-set `y` agar menjadi sebuah instance dari sebuah _box_ yang menunjuk 
ke salinan nilai dari `x` ketimbang sebuah referensi yang menunjuk ke 
nilai dari `x`. Di penegasan terakhir, kita bisa memakai operator 
dereferensi buat mengikuti pointer _box_ tersebut dengan cara yang sama 
seperti saat `y` tadinya adalah sebuah referensi. Selanjutnya, kita bakal 
mengeksplorasi apa yang spesial dari `Box<T>` yang memungkinkan kita buat 
memakai operator dereferensi dengan mendefinisikan tipe _box_ kita sendiri.

### Mendefinisikan Smart Pointer Kita Sendiri

Mari kita bikin sebuah tipe pembungkus (_wrapper type_) yang mirip sama tipe 
`Box<T>` yang disediakan sama _standard library_ buat merasakan gimana 
tipe _smart pointer_ berperilaku secara berbeda dari referensi secara 
default. Kemudian kita bakal melihat gimana cara menambahkan kemampuan 
buat memakai operator dereferensi.

> Catatan: Ada satu perbedaan besar antara tipe `MyBox<T>` yang mau kita 
> bikin ini dengan `Box<T>` yang asli: versi kita ini tidak bakal 
> menyimpan datanya di _heap_. Kita memfokuskan contoh ini pada `Deref`, 
> jadi di mana datanya sebenarnya disimpan itu kurang penting dibanding 
> perilaku yang mirip _pointer_-nya.

Tipe `Box<T>` pada akhirnya didefinisikan sebagai sebuah _tuple struct_ dengan 
satu elemen, jadi Listing 15-8 mendefinisikan sebuah tipe `MyBox<T>` dengan 
cara yang sama. Kita juga bakal mendefinisikan sebuah fungsi `new` agar 
cocok sama fungsi `new` yang didefinisikan pada `Box<T>`.

<Listing number="15-8" file-name="src/main.rs" caption="Mendefinisikan sebuah tipe `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

Kita mendefinisikan sebuah struct bernama `MyBox` dan mendeklarasikan sebuah 
parameter generik `T` karena kita mau tipe kita bisa memegang nilai dari 
tipe apa pun. Tipe `MyBox` adalah sebuah _tuple struct_ dengan satu elemen 
bertipe `T`. Fungsi `MyBox::new` menerima satu parameter bertipe `T` dan 
mengembalikan sebuah instance `MyBox` yang memegang nilai yang dimasukkan.

Mari coba tambahkan fungsi `main` di Listing 15-7 ke Listing 15-8 lalu 
ubah kodenya biar memakai tipe `MyBox<T>` yang sudah kita definisikan 
bukannya `Box<T>`. Kode di Listing 15-9 tidak bakal bisa di-compile karena 
Rust tidak tahu gimana cara men-dereferensi `MyBox`.

<Listing number="15-9" file-name="src/main.rs" caption="Mencoba memakai `MyBox<T>` dengan cara yang sama seperti kita memakai referensi dan `Box<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

Berikut adalah error kompilasi yang dihasilkan:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Tipe `MyBox<T>` kita tidak bisa di-dereferensi karena kita belum 
mengimplementasikan kemampuan tersebut pada tipe kita. Untuk memungkinkan 
proses dereferensi dengan operator `*`, kita harus mengimplementasikan 
trait `Deref`.

### Mengimplementasikan Trait `Deref`

Seperti yang sudah dibahas di [“Mengimplementasikan sebuah Trait pada suatu 
Tipe”][impl-trait] di Bab 10, untuk mengimplementasikan sebuah trait kita 
perlu menyediakan implementasi buat method-method yang diwajibkan oleh 
trait tersebut. Trait `Deref`, yang disediakan oleh _standard library_, 
mewajibkan kita buat mengimplementasikan satu method bernama `deref` yang 
meminjam `self` lalu mengembalikan sebuah referensi ke data internalnya. 
Listing 15-10 mengandung implementasi `Deref` buat ditambahkan ke definisi 
`MyBox<T>`.

<Listing number="15-10" file-name="src/main.rs" caption="Mengimplementasikan `Deref` pada `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

Sintaks `type Target = T;` mendefinisikan sebuah _associated type_ buat 
dipakai oleh trait `Deref`. _Associated types_ adalah cara yang sedikit 
berbeda dalam mendeklarasikan sebuah parameter generik, tapi kita tidak usah 
pusing dulu soal itu buat sekarang; kita bakal membahasnya lebih detail 
di Bab 20.

Kita mengisi body dari method `deref` dengan `&self.0` supaya `deref` 
mengembalikan referensi ke nilai yang mau kita akses dengan operator `*`; 
ingat kembali dari [“Memakai Tuple Structs tanpa Field Bernama buat Bikin 
Tipe yang Beda”][tuple-structs] di Bab 5 kalau `.0` mengakses nilai pertama 
di dalam sebuah _tuple struct_. Fungsi `main` di Listing 15-9 yang 
memanggil `*` pada nilai `MyBox<T>` sekarang sudah bisa di-compile, dan 
penegasannya sukses!

Tanpa trait `Deref`, _compiler_ cuma bisa men-dereferensi referensi `&`. 
Method `deref` memberi _compiler_ kemampuan buat mengambil sebuah nilai dari 
tipe apa pun yang mengimplementasikan `Deref` lalu memanggil method `deref` 
buat mendapatkan sebuah referensi `&` yang dia tahu gimana cara 
men-dereferensinya.

Saat kita memasukkan `*y` di Listing 15-9, di balik layar Rust sebenarnya 
menjalankan kode ini:

```rust,ignore
*(y.deref())
```

Rust mengganti operator `*` dengan pemanggilan ke method `deref` dan lalu 
sebuah dereferensi biasa sehingga kita tidak perlu repot mikirin apakah 
kita butuh memanggil method `deref` atau tidak. Fitur Rust ini membiarkan 
kita menulis kode yang fungsinya identik baik saat kita punya referensi 
biasa maupun tipe yang mengimplementasikan `Deref`.

Alasan kenapa method `deref` mengembalikan sebuah referensi ke sebuah nilai, 
dan kenapa dereferensi biasa di luar tanda kurung di `*(y.deref())` itu 
tetap diperlukan, ada hubungannya sama sistem _ownership_. Kalau method 
`deref` mengembalikan nilainya secara langsung bukannya referensi ke 
nilainya, nilainya bakal di-_move_ keluar dari `self`. Kita tidak mau 
mengambil kepemilikan (ownership) dari nilai internal di dalam `MyBox<T>` 
di kasus ini maupun di kebanyakan kasus di mana kita memakai operator 
dereferensi.

Perhatikan bahwa operator `*` digantikan dengan pemanggilan ke method 
`deref` dan kemudian pemanggilan ke operator `*` cuma satu kali saja, setiap 
kali kita memakai tanda `*` di kode kita. Karena penggantian operator `*` 
tersebut tidak berulang secara rekursif tanpa henti, kita akhirnya mendapatkan 
data bertipe `i32`, yang cocok dengan angka `5` di `assert_eq!` di 
Listing 15-9.

### Deref Coercion Implisit dengan Fungsi dan Method

_Deref coercion_ mengubah sebuah referensi ke sebuah tipe yang 
mengimplementasikan trait `Deref` menjadi sebuah referensi ke tipe lainnya. 
Misalnya, _deref coercion_ bisa mengubah `&String` menjadi `&str` karena 
`String` mengimplementasikan trait `Deref` sedemikian rupa sehingga ia 
mengembalikan `&str`. _Deref coercion_ adalah sebuah kemudahan yang 
dilakukan Rust pada argumen buat fungsi dan method, dan cuma bekerja pada 
tipe yang mengimplementasikan trait `Deref`. Hal ini terjadi secara 
otomatis saat kita meneruskan sebuah referensi ke nilai dari tipe tertentu 
sebagai argumen ke sebuah fungsi atau method yang tipenya tidak cocok dengan 
tipe parameter di definisi fungsi atau method tersebut. Serangkaian 
pemanggilan ke method `deref` mengubah tipe yang kita berikan menjadi tipe 
yang dibutuhkan sama parameternya.

_Deref coercion_ ditambahkan ke Rust supaya para programmer yang menulis 
pemanggilan fungsi dan method tidak perlu menambahkan terlalu banyak 
referensi dan dereferensi eksplisit dengan `&` dan `*`. Fitur _deref coercion_ 
juga membiarkan kita menulis lebih banyak kode yang bisa bekerja baik 
buat referensi maupun buat _smart pointers_.

Buat melihat _deref coercion_ beraksi, mari kita gunakan tipe `MyBox<T>` yang 
kita definisikan di Listing 15-8 beserta implementasi `Deref` yang kita 
tambahkan di Listing 15-10. Listing 15-11 menunjukkan definisi dari sebuah 
fungsi yang punya parameter berupa _string slice_.

<Listing number="15-11" file-name="src/main.rs" caption="Sebuah fungsi `hello` yang punya parameter `name` bertipe `&str`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

Kita bisa memanggil fungsi `hello` dengan sebuah _string slice_ sebagai 
argumennya, misalnya `hello("Rust");`. _Deref coercion_ memungkinkan kita 
buat memanggil `hello` dengan sebuah referensi ke sebuah nilai bertipe 
`MyBox<String>`, seperti yang ditunjukkan di Listing 15-12.

<Listing number="15-12" file-name="src/main.rs" caption="Memanggil `hello` dengan referensi ke nilai `MyBox<String>`, yang mana berhasil berkat _deref coercion_">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

Di sini kita memanggil fungsi `hello` dengan argumen `&m`, yang mana adalah 
sebuah referensi ke sebuah nilai `MyBox<String>`. Karena kita 
mengimplementasikan trait `Deref` pada `MyBox<T>` di Listing 15-10, Rust 
bisa mengubah `&MyBox<String>` menjadi `&String` dengan memanggil `deref`. 
_Standard library_ menyediakan sebuah implementasi `Deref` pada `String` 
yang mengembalikan sebuah _string slice_, dan ini ada di dokumentasi API 
buat `Deref`. Rust memanggil `deref` sekali lagi buat mengubah `&String` 
menjadi `&str`, yang mana cocok sama definisi fungsi `hello`.

Kalau Rust tidak mengimplementasikan _deref coercion_, kita harus menulis 
kode di Listing 15-13 bukannya kode di Listing 15-12 buat memanggil `hello` 
dengan sebuah nilai bertipe `&MyBox<String>`.

<Listing number="15-13" file-name="src/main.rs" caption="Kode yang harus kita tulis kalau seandainya Rust tidak punya _deref coercion_">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

Tanda `(*m)` men-dereferensi `MyBox<String>` menjadi sebuah `String`. 
Kemudian tanda `&` dan `[..]` mengambil sebuah _string slice_ dari `String` 
tersebut yang setara dengan keseluruhan string-nya agar cocok sama _signature_ 
dari `hello`. Kode tanpa _deref coercions_ ini lebih susah buat dibaca, ditulis, 
dan dipahami dengan semua simbol yang terlibat ini. _Deref coercion_ membiarkan 
Rust menangani konversi-konversi ini buat kita secara otomatis.

Saat trait `Deref` didefinisikan buat tipe-tipe yang terlibat, Rust bakal 
menganalisis tipe-tipenya dan memakai `Deref::deref` sebanyak yang 
dibutuhkan buat mendapatkan sebuah referensi yang cocok sama tipe 
parameternya. Berapa kali `Deref::deref` perlu disisipkan itu sudah 
diselesaikan (resolved) pas _compile time_, jadi tidak ada hukuman performa 
saat _runtime_ karena memanfaatkan _deref coercion_!

### Gimana Deref Coercion Berinteraksi sama Mutabilitas

Sama seperti gimana kita memakai trait `Deref` buat menimpa operator `*` pada 
referensi _immutable_, kita juga bisa memakai trait `DerefMut` buat menimpa 
operator `*` pada referensi _mutable_.

Rust melakukan _deref coercion_ saat ia menemukan tipe-tipe dan implementasi 
trait di tiga kasus ini:

1. Dari `&T` ke `&U` saat `T: Deref<Target=U>`
2. Dari `&mut T` ke `&mut U` saat `T: DerefMut<Target=U>`
3. Dari `&mut T` ke `&U` saat `T: Deref<Target=U>`

Dua kasus pertama itu sama saja kecuali kalau yang kedua mengimplementasikan 
mutabilitas. Kasus pertama menyatakan kalau kita punya sebuah `&T`, dan `T` 
mengimplementasikan `Deref` ke suatu tipe `U`, kita bisa mendapatkan sebuah 
`&U` secara transparan. Kasus kedua menyatakan kalau _deref coercion_ yang 
sama terjadi buat referensi _mutable_.

Kasus ketiga itu sedikit lebih _tricky_: Rust juga bakal me-*coerce* sebuah 
referensi _mutable_ menjadi referensi _immutable_. Tapi kebalikannya itu 
_tidak_ mungkin: referensi _immutable_ tidak bakal pernah bisa di-*coerce* 
menjadi referensi _mutable_. Karena aturan _borrowing_, kalau kita punya sebuah 
referensi _mutable_, referensi _mutable_ tersebut haruslah menjadi satu-satunya 
referensi ke data tersebut (kalau tidak, programnya tidak bakal bisa 
di-compile). Mengonversi satu referensi _mutable_ menjadi satu referensi 
_immutable_ tidak bakal pernah melanggar aturan _borrowing_. Mengonversi sebuah 
referensi _immutable_ menjadi referensi _mutable_ bakal mewajibkan kalau 
referensi _immutable_ awalnya adalah satu-satunya referensi _immutable_ ke 
data tersebut, tapi aturan _borrowing_ tidak menjamin hal itu. Oleh karena 
itu, Rust tidak bisa membuat asumsi kalau mengonversi sebuah referensi 
_immutable_ menjadi referensi _mutable_ itu mungkin dilakukan.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
