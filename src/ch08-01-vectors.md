## Nyimpen Daftar Nilai pake Vectors

Tipe koleksi pertama yang bakal kita liat adalah `Vec<T>`, yang juga dikenal 
sebagai _vector_. Vectors ngebolehin kita nyimpen lebih dari satu nilai di dalem 
satu struktur data tunggal yang naruh semua nilai itu bersebelahan di memori. 
Vectors cuma bisa nyimpen nilai dengan tipe yang sama. Mereka berguna pas kita 
punya daftar (list) item, kayak baris-baris teks di sebuah file atau harga-harga 
barang di keranjang belanja.

### Bikin Vector Baru

Buat bikin vector baru yang kosong, kita manggil fungsi `Vec::new`, kayak yang 
ditunjukin di Listing 8-1.

<Listing number="8-1" caption="Bikin vector baru yang kosong buat nampung nilai tipe `i32`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Perhatiin ya kalau kita nambahin anotasi tipe di sini. Karena kita nggak masukin 
nilai apa pun ke vector ini, Rust nggak tau jenis elemen apa yang mau kita simpen. 
Ini poin penting. Vectors diimplementasikan pake generik (generics); kita bakal 
bahas cara pake generik bareng tipe kita sendiri di Bab 10. Buat sekarang, tau 
aja kalau tipe `Vec<T>` yang disediain sama standard library bisa nampung tipe 
apa pun. Pas kita bikin vector buat nampung tipe spesifik, kita bisa nentuin 
tipenya di dalem kurung siku. Di Listing 8-1, kita ngasih tau Rust kalau 
`Vec<T>` di variabel `v` bakal nampung elemen dengan tipe `i32`.

Biasanya, kita bakal bikin `Vec<T>` dengan nilai awal dan Rust bakal nebak (infer) 
tipe nilai yang mau kita simpen, jadi kita jarang sekali butuh anotasi tipe 
kayak gini. Rust nyediain macro yang praktis sekali, `vec!`, yang bakal bikin 
vector baru yang isinya nilai-nilai yang kita kasih. Listing 8-2 bikin 
`Vec<i32>` baru yang isinya nilai `1`, `2`, dan `3`. Tipe integer-nya adalah 
`i32` karena itu adalah tipe integer default, kayak yang kita bahas di bagian 
[“Tipe Data”][data-types] di Bab 3.

<Listing number="8-2" caption="Bikin vector baru yang isinya nilai-nilai">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

Karena kita udah ngasih nilai awal `i32`, Rust bisa nebak kalau tipe dari `v` 
adalah `Vec<i32>`, dan anotasi tipenya nggak dibutuhin. Selanjutnya, kita bakal 
liat cara ngubah sebuah vector.

### Ngubah Vector

Buat bikin vector terus nambahin elemen ke dalemnya, kita bisa pake method 
`push`, kayak yang ditunjukin di Listing 8-3.

<Listing number="8-3" caption="Pake method `push` buat nambahin nilai ke vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Sama kayak variabel mana pun, kalau kita mau bisa ngubah nilainya, kita harus 
bikin variabelnya _mutable_ pake keyword `mut`, kayak yang dibahas di Bab 3. 
Angka-angka yang kita taruh di dalemnya semuanya bertipe `i32`, dan Rust nebak 
ini dari datanya, jadi kita nggak perlu anotasi `Vec<i32>`.

### Ngebaca Elemen dari Vectors

Ada dua cara buat ngerujuk ke nilai yang disimpan di sebuah vector: lewat 
_indexing_ atau pake method `get`. Di contoh-contoh berikut, kita udah 
nganotasi tipe dari nilai yang dibalikin sama fungsi-fungsi ini biar lebih jelas.

Listing 8-4 nunjukin kedua cara buat akses nilai di dalem vector, pake sintaks 
indexing dan method `get`.

<Listing number="8-4" caption="Pake sintaks indexing dan pake method `get` buat akses item di vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Ada beberapa detail yang perlu diperhatiin di sini. Kita pake nilai indeks `2` 
buat dapet elemen ketiga karena vectors diindeks pake angka, mulai dari nol. 
Pake `&` sama `[]` ngasih kita sebuah referensi ke elemen di nilai indeks 
tersebut. Pas kita pake method `get` dengan indeks yang dimasukin sebagai argumen, 
kita dapet `Option<&T>` yang bisa kita pake bareng `match`.

Rust nyediain dua cara buat ngerujuk elemen biar kita bisa milih gimana program 
kita bereaksi pas kita nyoba pake nilai indeks yang di luar rentang (_range_) 
elemen yang ada. Sebagai contoh, yuk kita liat apa yang terjadi pas kita punya 
vector isinya lima elemen terus kita nyoba akses elemen di indeks 100 pake 
kedua teknik ini, kayak yang ditunjukin di Listing 8-5.

<Listing number="8-5" caption="Nyoba akses elemen di indeks 100 di vector yang isinya lima elemen">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Pas kita jalanin kode ini, metode pertama `[]` bakal bikin program _panic_ 
karena dia ngerujuk ke elemen yang nggak ada. Metode ini paling pas dipake 
kalau kita mau program kita nge-_crash_ kalau ada percobaan buat akses elemen 
ngelewatin akhir vector.

Pas method `get` dikasih indeks yang di luar vector, dia bakal balikin `None` 
tanpa bikin _panic_. Kita bakal pake metode ini kalau akses elemen di luar 
rentang vector mungkin sesekali kejadian di bawah kondisi normal. Kode kita 
terus bakal punya logika buat nanganin dapet `Some(&element)` atau `None`, 
kayak yang dibahas di Bab 6. Misalnya, indeksnya bisa jadi dateng dari orang 
yang masukin angka. Kalau mereka nggak sengaja masukin angka yang kegedean dan 
programnya dapet nilai `None`, kita bisa ngasih tau _user_ berapa banyak item 
yang ada di vector saat ini dan ngasih mereka kesempatan lagi buat masukin 
nilai yang valid. Itu bakal lebih _user-friendly_ daripada nge-_crash_-in 
program gara-gara _typo_!

Pas programnya punya referensi yang valid, _borrow checker_ bakal nerapin 
aturan _ownership_ dan _borrowing_ (yang dibahas di Bab 4) buat mastiin referensi 
ini dan referensi apa pun lainnya ke isi vector tetep valid. Inget aturan yang 
bilang kalau kita nggak bisa punya referensi _mutable_ sama _immutable_ di 
_scope_ yang sama. Aturan itu berlaku di Listing 8-6, di mana kita megang 
_immutable reference_ ke elemen pertama di vector terus nyoba nambahin elemen di 
akhir vector. Program ini nggak bakal jalan kalau kita juga nyoba ngerujuk ke 
elemen itu nanti di fungsinya.

<Listing number="8-6" caption="Nyoba nambahin elemen ke vector sambil megang referensi ke sebuah item">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

Compile kode ini bakal ngasilin error ini:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Kode di Listing 8-6 mungkin keliatannya harusnya jalan: kenapa referensi ke 
elemen pertama harus peduli sama perubahan di akhir vector? Error ini terjadi 
karena cara kerja vectors: karena vectors naruh nilai bersebelahan satu sama 
lain di memori, nambahin elemen baru di akhir vector mungkin butuh ngalokasiin 
memori baru dan ngopi elemen-elemen lama ke ruang yang baru, kalau ternyata nggak 
ada ruang yang cukup buat naruh semua elemen bersebelahan di tempat vector itu 
saat ini disimpan. Di kasus itu, referensi ke elemen pertama bakal nunjuk ke 
memori yang udah di-dealokasi (_deallocated memory_). Aturan _borrowing_ 
nyegah program berakhir di situasi kayak gitu.

> Catatan: Buat detail implementasi lebih lanjut dari tipe `Vec<T>`, liat [“The
> Rustonomicon”][nomicon].

### Iterasi Lewat Nilai-nilai di dalem Vector

Buat akses tiap elemen di vector secara bergiliran, kita bakal iterasi lewat 
semua elemennya bukannya pake indeks buat akses satu-satu. Listing 8-7 nunjukin 
cara pake `for` loop buat dapet _immutable references_ ke tiap elemen di vector 
nilai `i32` terus nyetak semuanya.

<Listing number="8-7" caption="Nyetak tiap elemen di vector dengan iterasi lewat elemen pake `for` loop">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Kita juga bisa iterasi lewat _mutable references_ ke tiap elemen di _mutable 
vector_ buat bikin perubahan ke semua elemen. `for` loop di Listing 8-8 bakal 
nambahin `50` ke tiap elemen.

<Listing number="8-8" caption="Iterasi lewat _mutable references_ ke elemen-elemen di vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Buat ngubah nilai yang ditunjuk sama _mutable reference_, kita harus pake 
operator _dereference_ `*` buat nyampe ke nilai di `i` sebelum kita bisa pake 
operator `+=`. Kita bakal bahas operator _dereference_ lebih dalem di bagian 
[“Ngikutin Pointer ke Nilai pake Operator Dereference”][deref] di Bab 15.

Iterasi lewat sebuah vector, entah itu secara _immutable_ atau _mutable_, 
selalu aman berkat aturan _borrow checker_. Kalau kita nyoba _insert_ atau 
_remove_ item di body `for` loop di Listing 8-7 sama Listing 8-8, kita bakal 
dapet error _compiler_ yang mirip kayak yang kita dapet dari kode di Listing 8-6. 
Referensi ke vector yang dipegang sama `for` loop nyegah modifikasi 
keseluruhan vector di waktu yang sama.

### Pake Enum Buat Nyimpen Banyak Tipe

Vectors cuma bisa nyimpen nilai yang tipenya sama. Ini bisa jadi kurang 
nyaman; pasti ada kasus di mana kita butuh nyimpen daftar item yang tipenya 
beda-beda. Untungnya, varian dari sebuah enum didefinisikan di bawah tipe enum 
yang sama, jadi pas kita butuh satu tipe buat ngewakilin elemen dari berbagai 
tipe, kita bisa bikin dan pake enum!

Misalnya, katakanlah kita mau dapet nilai dari sebuah baris di _spreadsheet_ di 
mana beberapa kolom di baris itu isinya integer, beberapa angka _floating-point_, 
dan beberapa lagi strings. Kita bisa mendefinisikan enum yang varian-variannya 
bakal nampung tipe nilai yang beda, dan semua varian enum itu bakal dianggap 
sebagai tipe yang sama: yaitu tipe dari enum tersebut. Terus kita bisa bikin 
vector buat nampung enum itu dan akhirnya bisa nampung tipe yang beda-beda. Kita 
udah demonstrasikan ini di Listing 8-9.

<Listing number="8-9" caption="Mendefinisikan sebuah `enum` buat nyimpen nilai dari tipe yang beda di dalem satu vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust perlu tau tipe apa aja yang bakal ada di vector pas _compile time_ biar dia 
tau persis berapa banyak memori di _heap_ yang bakal dibutuhin buat nyimpen 
tiap elemen. Kita juga harus eksplisit soal tipe apa aja yang dibolehin di 
vector ini. Kalau Rust ngebolehin sebuah vector buat nampung tipe apa aja, ada 
kemungkinan satu atau lebih dari tipe itu bakal nyebabin error sama operasi yang 
dijalanin pada elemen vector-nya. Pake enum ditambah ekspresi `match` artinya 
Rust bakal mastiin pas _compile time_ kalau setiap kasus yang mungkin terjadi 
itu di-handle, kayak yang dibahas di Bab 6.

Kalau kita nggak tau daftar lengkap dari tipe-tipe yang bakal didapet program pas 
_runtime_ buat disimpan di vector, teknik enum ini nggak bakal jalan. Sebagai 
gantinya, kita bisa pake _trait object_, yang bakal kita bahas di Bab 18.

Sekarang setelah kita bahas beberapa cara paling umum buat pake vectors, pastiin 
buat cek [dokumentasi API-nya][vec-api] buat semua method berguna yang 
didefinisikan pada `Vec<T>` sama standard library. Misalnya, selain `push`, ada 
method `pop` yang ngehapus dan balikin elemen terakhir.

### Nge-drop Vector Bakal Nge-drop Elemennya Juga

Kayak `struct` lainnya, sebuah vector bakal dibebasin (freed) pas dia keluar 
dari scope, kayak yang dianotasi di Listing 8-10.

<Listing number="8-10" caption="Nunjukin di mana vector dan elemennya di-_drop_">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Pas vector di-_drop_, semua isinya juga ikut di-_drop_, artinya integer-integer 
yang ada di dalemnya bakal dibersihin. _Borrow checker_ mastiin kalau referensi 
apa pun ke isi dari vector cuma dipake selama vector itu sendiri masih valid.

Yuk kita lanjut ke tipe koleksi berikutnya: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
