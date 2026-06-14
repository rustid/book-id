## Meningkatkan Project I/O Kita

Dengan pengetahuan baru tentang iterators ini, kita bisa meningkatkan project I/O 
di Bab 12 dengan memakai iterators buat bikin beberapa bagian kodenya jadi lebih 
jelas dan lebih ringkas. Mari kita lihat gimana iterators bisa meningkatkan 
implementasi dari fungsi `Config::build` dan fungsi `search` kita.

### Menghilangkan `clone` Menggunakan Iterator

Di Listing 12-6, kita menambahkan kode yang mengambil _slice_ berisi nilai 
`String` lalu membuat instance dari struct `Config` dengan mengindeks ke dalam 
_slice_ tersebut dan meng-_clone_ (menyalin) nilai-nilainya, yang memungkinkan 
struct `Config` buat memiliki (own) nilai-nilai itu. Di Listing 13-17, kita 
menampilkan ulang implementasi dari fungsi `Config::build` persis seperti yang 
ada di Listing 12-23.

<Listing number="13-17" file-name="src/main.rs" caption="Menampilkan ulang fungsi `Config::build` dari Listing 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

Waktu itu, kita bilang buat tidak mengkhawatirkan panggilan `clone` yang kurang 
efisien karena kita bakal menghapusnya di masa depan. Nah, sekaranglah saatnya!

Kita membutuhkan `clone` di sini karena kita punya sebuah _slice_ berisi 
elemen-elemen `String` di parameter `args`, tapi fungsi `build` tidak mengambil 
kepemilikan (ownership) atas `args`. Buat mengembalikan kepemilikan dari instance 
`Config`, kita harus meng-_clone_ nilai-nilai dari field `query` dan `file_path` 
milik `Config` supaya instance `Config` tersebut bisa memiliki nilai-nilainya.

Dengan pengetahuan baru kita tentang iterators, kita bisa mengubah fungsi `build` 
agar mengambil kepemilikan dari sebuah iterator sebagai argumennya, ketimbang 
meminjam (borrow) sebuah _slice_. Kita bakal memakai fungsionalitas iterator 
alih-alih kode yang mengecek panjang dari _slice_ dan mengindeks ke lokasi 
spesifik. Ini bakal memperjelas apa yang sedang dilakukan oleh fungsi 
`Config::build` karena iterator-lah yang bakal mengakses nilai-nilainya.

Begitu `Config::build` mengambil kepemilikan atas iterator dan berhenti memakai 
operasi _indexing_ yang sifatnya meminjam, kita bisa memindahkan (move) 
nilai-nilai `String` dari iterator tersebut ke dalam `Config` alih-alih memanggil 
`clone` dan membuat alokasi baru.

#### Memakai Iterator yang Dikembalikan secara Langsung

Buka file _src/main.rs_ dari project I/O kita, yang seharusnya kelihatan seperti ini:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Pertama-tama kita bakal mengubah bagian awal dari fungsi `main` yang kita punya 
di Listing 12-24 jadi kode yang ada di Listing 13-18, yang mana kali ini memakai 
iterator. Ini belum bisa di-compile sampai kita meng-update `Config::build` juga.

<Listing number="13-18" file-name="src/main.rs" caption="Meneruskan nilai yang dikembalikan oleh `env::args` ke `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

Fungsi `env::args` mengembalikan sebuah iterator! Alih-alih mengumpulkan nilai-nilai 
iterator itu ke dalam sebuah vector terus meneruskan sebuah _slice_ ke 
`Config::build`, sekarang kita meneruskan kepemilikan dari iterator yang 
dikembalikan dari `env::args` ke `Config::build` secara langsung.

Berikutnya, kita harus meng-update definisi dari `Config::build`. Mari kita ubah 
_signature_ (tanda tangan) dari `Config::build` agar kelihatan seperti Listing 
13-19. Ini masih belum bisa di-compile, karena kita harus meng-update isi (body) 
fungsinya.

<Listing number="13-19" file-name="src/main.rs" caption="Meng-update _signature_ dari `Config::build` buat mengharapkan sebuah iterator">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

Dokumentasi _standard library_ untuk fungsi `env::args` menunjukkan bahwa tipe 
dari iterator yang dikembalikannya adalah `std::env::Args`, dan tipe tersebut 
mengimplementasikan trait `Iterator` serta mengembalikan nilai `String`.

Kita sudah meng-update _signature_ dari fungsi `Config::build` jadi parameter 
`args` punya tipe generik dengan _trait bounds_ `impl Iterator<Item = String>` 
bukannya `&[String]`. Penggunaan sintaks `impl Trait` yang kita bahas di 
bagian [“Traits sebagai Parameter”][impl-trait] di Bab 10 ini berarti `args` 
bisa berupa tipe apa pun yang mengimplementasikan trait `Iterator` dan 
mengembalikan item berupa `String`.

Karena kita mengambil kepemilikan atas `args` dan kita bakal memutasi (mengubah) 
`args` saat kita beriterasi melewatinya, kita bisa menambahkan keyword `mut` 
ke dalam spesifikasi parameter `args` buat membikinnya jadi _mutable_.

#### Memakai Method Trait `Iterator` Alih-Alih Indexing

Berikutnya, kita bakal memperbaiki isi dari `Config::build`. Karena `args` 
mengimplementasikan trait `Iterator`, kita tahu kalau kita bisa memanggil method 
`next` padanya! Listing 13-20 meng-update kode dari Listing 12-23 buat memakai 
method `next`.

<Listing number="13-20" file-name="src/main.rs" caption="Mengubah isi dari `Config::build` buat memakai method iterator">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

Ingat kembali bahwa nilai pertama di dalam nilai yang dikembalikan oleh 
`env::args` adalah nama dari programnya. Kita mau mengabaikan itu dan lanjut ke 
nilai berikutnya, jadi pertama kita memanggil `next` dan tidak ngelakuin apa-apa 
dengan nilai yang dikembalikannya. Terus kita panggil `next` lagi buat dapet 
nilai yang mau kita masukin ke field `query` dari `Config`. Kalau `next` 
mengembalikan `Some`, kita pakai `match` buat mengekstrak nilainya. Kalau dia 
mengembalikan `None`, itu berarti argumen yang diberikan tidak cukup dan kita 
bisa keluar lebih awal dengan nilai `Err`. Kita ngelakuin hal yang sama buat 
nilai `file_path`.

### Membikin Kode Lebih Jelas dengan Iterator Adapters

Kita juga bisa memanfaatkan iterators di fungsi `search` dari project I/O kita, 
yang ditampilkan ulang di Listing 13-21 persis seperti yang ada di Listing 12-19.

<Listing number="13-21" file-name="src/lib.rs" caption="Implementasi dari fungsi `search` dari Listing 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Kita bisa nulis kode ini dengan cara yang lebih ringkas memakai method _iterator 
adapter_. Dengan begitu, kita juga terhindar dari kewajiban punya vector 
`results` menengah (_intermediate_) yang _mutable_. Gaya pemrograman fungsional 
lebih suka meminimalisir _mutable state_ (keadaan yang bisa berubah) demi 
bikin kodenya jadi lebih jelas. Membuang _mutable state_ ini mungkin bisa 
memungkinkan adanya peningkatan di masa depan yang bakal bikin pencarian terjadi 
secara paralel karena kita tidak perlu pusing mengelola akses konruen 
(_concurrent access_) ke vector `results` tersebut. Listing 13-22 menunjukkan 
perubahan ini.

<Listing number="13-22" file-name="src/lib.rs" caption="Memakai method _iterator adapter_ di implementasi fungsi `search`">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Ingat kembali bahwa tujuan dari fungsi `search` adalah mengembalikan semua baris 
di dalam `contents` yang mengandung `query`. Mirip seperti contoh `filter` di 
Listing 13-16, kode ini memakai *adapter* `filter` buat menyimpan cuma 
baris-baris di mana `line.contains(query)` mengembalikan `true`. Kita kemudian 
mengumpulkan baris-baris yang cocok itu ke dalam sebuah vector lain menggunakan 
`collect`. Jauh lebih simpel kan! Jangan ragu buat ngelakuin perubahan yang sama 
buat memakai method iterator di fungsi `search_case_insensitive` juga.

Sebagai peningkatan lanjutan, coba kembalikan sebuah iterator dari fungsi 
`search` dengan menghapus panggilan ke `collect` dan mengubah tipe kembaliannya 
jadi `impl Iterator<Item = &'a str>` supaya fungsi ini menjadi sebuah _iterator 
adapter_. Perhatikan bahwa kita juga bakal harus meng-update pengujiannya! Coba 
cari di dalam sebuah file berukuran besar menggunakan alat `minigrep` kita sebelum 
dan sesudah membikin perubahan ini untuk melihat perbedaan perilakunya. Sebelum 
perubahan ini, program tidak bakal mencetak hasil apa pun sampai ia selesai 
mengumpulkan semua hasilnya, tapi setelah perubahan itu, hasil bakal dicetak 
satu per satu setiap kali ada baris yang cocok karena _for loop_ di fungsi `run` 
bisa memanfaatkan sifat _lazy_ (malas) dari iterator-nya.

<!-- Old heading. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### Memilih antara Loops dan Iterators

Pertanyaan masuk akal berikutnya adalah gaya mana yang sebaiknya kita pilih di 
kode kita sendiri dan kenapa: implementasi awal di Listing 13-21 atau versi 
yang memakai iterators di Listing 13-22 (dengan asumsi kita mengumpulkan semua 
hasilnya sebelum mengembalikannya bukannya mengembalikan iteratornya). Sebagian 
besar programmer Rust lebih suka memakai gaya iterator. Mungkin agak sedikit 
susah buat memahaminya di awal, tapi begitu kita sudah mulai merasakan (_get a 
feel for_) berbagai _iterator adapters_ dan apa yang mereka lakukan, iterators 
bisa jadi lebih gampang buat dipahami. Ketimbang ribet ngurusin detail soal 
_looping_ (perulangan) dan membikin vector baru, kode kita jadi bisa lebih fokus 
ke tujuan tingkat tinggi (high-level objective) dari _loop_ tersebut. Ini 
mengabstraksi kode-kode umum (commonplace code) sehingga konsep-konsep yang 
unik buat kode ini jadi lebih mudah dilihat, seperti contohnya kondisi penyaringan 
(filtering condition) yang harus dilewati sama setiap elemen di dalam iterator.

Tapi apakah kedua implementasi ini benar-benar ekuivalen (sama)? Asumsi yang 
mungkin muncul secara intuitif adalah _loop_ tingkat rendah (lower-level loop) 
bakal lebih cepat. Mari kita bahas soal performa.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
