## Memakai Threads buat Menjalankan Kode Secara Bersamaan

Di sebagian besar sistem operasi saat ini, kode dari sebuah program yang 
dieksekusi dijalankan di dalam sebuah _process_ (proses), dan sistem operasi 
bakal mengelola banyak proses sekaligus. Di dalam sebuah program, kita juga 
bisa punya bagian-bagian independen yang berjalan secara bersamaan 
(simultaneously). Fitur yang menjalankan bagian-bagian independen ini 
disebut _threads_ (utas). Misalnya, sebuah _web server_ bisa punya banyak 
_threads_ sehingga ia bisa merespons lebih dari satu _request_ (permintaan) 
di saat yang bersamaan.

Memecah komputasi di program kita menjadi banyak _threads_ buat menjalankan 
banyak tugas di saat yang bersamaan bisa meningkatkan performa, tapi ini juga 
menambahkan kerumitan (complexity). Karena _threads_ bisa berjalan secara 
bersamaan, tidak ada jaminan bawaan (inherent guarantee) tentang urutan bagian 
kode mana di _threads_ yang berbeda yang bakal jalan duluan. Ini bisa berujung 
pada masalah-masalah, seperti:

- _Race conditions_ (balapan kondisi), di mana _threads_ mengakses data atau 
  sumber daya (resources) dalam urutan yang tidak konsisten
- _Deadlocks_ (jalan buntu), di mana dua _threads_ saling menunggu satu sama 
  lain, mencegah kedua _threads_ tersebut buat bisa lanjut
- _Bugs_ yang cuma terjadi di situasi-situasi tertentu dan susah buat direka 
  ulang (reproduce) dan diperbaiki secara andal

Rust berusaha memitigasi efek-efek negatif dari memakai _threads_, tapi 
memprogram di dalam konteks _multithreaded_ tetap butuh pemikiran yang hati-hati 
dan membutuhkan struktur kode yang berbeda dari program yang berjalan di satu 
_thread_ saja (single thread).

Bahasa pemrograman mengimplementasikan _threads_ dengan beberapa cara yang 
berbeda-beda, dan banyak sistem operasi menyediakan sebuah API yang bisa dipanggil 
oleh bahasa pemrograman tersebut buat membikin _threads_ baru. _Standard library_ 
Rust memakai model implementasi _thread_ _1:1_, di mana sebuah program memakai 
satu _thread_ sistem operasi untuk satu _thread_ bahasa. Ada _crates_ yang 
mengimplementasikan model _threading_ lain yang membikin _trade-offs_ (pertukaran) 
yang beda dari model 1:1. (Sistem _async_ Rust, yang bakal kita lihat di bab 
selanjutnya, menyediakan pendekatan lain buat konkurensi.)

### Membikin Thread Baru dengan `spawn`

Buat membikin _thread_ baru, kita memanggil fungsi `thread::spawn` lalu 
memberikan sebuah _closure_ (kita sudah membahas _closures_ di Bab 13) yang 
mengandung kode yang mau kita jalankan di _thread_ baru tersebut. Contoh 
di Listing 16-1 mencetak sedikit teks dari _main thread_ (thread utama) dan 
teks lainnya dari _thread_ yang baru dibikin.

<Listing number="16-1" file-name="src/main.rs" caption="Membikin sebuah _thread_ baru buat mencetak sesuatu sementara _main thread_ mencetak hal lain">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Perhatikan bahwa saat _main thread_ dari program Rust selesai, semua _threads_ 
yang baru dibikin (spawned threads) bakal dimatikan, tidak peduli apakah mereka 
sudah selesai berjalan atau belum. Output dari program ini mungkin bakal 
sedikit berbeda setiap kalinya, tapi bakal kelihatan mirip seperti berikut:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Pemanggilan ke `thread::sleep` memaksa sebuah _thread_ buat menghentikan 
eksekusinya untuk durasi yang singkat, memungkinkan _thread_ lain buat berjalan. 
_Threads_ tersebut kemungkinan bakal berjalan bergantian, tapi itu tidak dijamin: 
itu bergantung sama gimana sistem operasi kita menjadwalkan (_schedules_) 
_threads_ tersebut. Di jalannya (run) kali ini, _main thread_ mencetak duluan, 
meskipun *statement print* dari _spawned thread_ muncul duluan di kodenya. Dan 
walaupun kita menyuruh _spawned thread_ buat mencetak sampai `i` itu `9`, ia cuma 
sampai ke `5` sebelum _main thread_ dimatikan.

Kalau kita menjalankan kode ini dan cuma melihat output dari _main thread_, atau 
tidak melihat tumpang tindih (overlap) apa pun, coba naikkan angka di rentangnya 
(ranges) buat membikin lebih banyak kesempatan buat sistem operasi beralih di 
antara _threads_ tersebut.

### Menunggu Semua Threads buat Selesai Memakai `join` Handles

Kode di Listing 16-1 tidak cuma menghentikan _spawned thread_ sebelum waktunya 
(prematurely) di sebagian besar waktu karena _main thread_ yang berakhir duluan, 
tapi karena tidak ada jaminan di urutan mana _threads_ itu berjalan, kita juga 
tidak bisa menjamin apakah _spawned thread_ itu bakal dapat kesempatan buat 
berjalan sama sekali!

Kita bisa membereskan masalah _spawned thread_ yang tidak berjalan atau 
berakhir sebelum waktunya dengan menyimpan nilai kembalian (return value) dari 
`thread::spawn` ke dalam sebuah variabel. Tipe kembalian dari `thread::spawn` 
adalah `JoinHandle<T>`. Sebuah `JoinHandle<T>` adalah nilai yang dimiliki (owned 
value) yang, saat kita memanggil method `join` padanya, bakal menunggu sampai 
_thread_-nya selesai. Listing 16-2 menunjukkan gimana cara memakai 
`JoinHandle<T>` dari _thread_ yang kita bikin di Listing 16-1 dan gimana cara 
memanggil `join` buat memastikan _spawned thread_ tersebut selesai sebelum `main` 
keluar (exits).

<Listing number="16-2" file-name="src/main.rs" caption="Menyimpan sebuah `JoinHandle<T>` dari `thread::spawn` buat menjamin _thread_-nya berjalan sampai selesai">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Memanggil `join` pada *handle* bakal memblokir (_blocks_) _thread_ yang saat ini 
lagi jalan sampai _thread_ yang diwakili oleh *handle* tersebut berhenti (terminates). 
_Memblokir_ sebuah _thread_ berarti _thread_ tersebut dicegah buat melakukan 
pekerjaan atau keluar. Karena kita menaruh pemanggilan `join` setelah _for loop_ 
milik _main thread_, menjalankan Listing 16-2 seharusnya menghasilkan output 
yang mirip kayak gini:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Dua _threads_ tersebut lanjut berjalan bergantian, tapi _main thread_ menunggu 
karena adanya pemanggilan `handle.join()` dan tidak berakhir sampai _spawned 
thread_-nya selesai.

Tapi mari kita lihat apa yang terjadi kalau kita malah memindahkan `handle.join()` 
sebelum _for loop_ di `main`, kayak gini:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

_Main thread_ bakal menunggu _spawned thread_ buat selesai baru kemudian dia 
menjalankan _for loop_-nya, jadi outputnya tidak bakal tumpang tindih lagi, 
seperti yang ditunjukkan di sini:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Detail-detail kecil, kayak di mana `join` itu dipanggil, bisa memengaruhi apakah 
_threads_ kita berjalan secara bersamaan atau tidak.

### Memakai Closures `move` bersama Threads

Kita bakal sering memakai keyword `move` bersama _closures_ yang diteruskan 
ke `thread::spawn` karena _closure_ tersebut kemudian bakal mengambil 
kepemilikan atas nilai-nilai yang dia pakai dari lingkungannya, sehingga 
mentransfer kepemilikan dari nilai-nilai tersebut dari satu _thread_ ke _thread_ 
lainnya. Di [“Menangkap Referensi atau Memindahkan Kepemilikan”][capture] di 
Bab 13, kita sudah membahas `move` di dalam konteks _closures_. Sekarang kita 
bakal lebih konsentrasi pada interaksi antara `move` dan `thread::spawn`.

Perhatikan di Listing 16-1 bahwa _closure_ yang kita teruskan ke 
`thread::spawn` tidak menerima argumen apa pun: kita tidak memakai data apa 
pun dari _main thread_ di dalam kode milik _spawned thread_. Buat memakai 
data dari _main thread_ di dalam _spawned thread_, _closure_ si _spawned thread_ 
harus menangkap (capture) nilai-nilai yang dia butuhkan. Listing 16-3 
menunjukkan usaha buat membikin sebuah vector di _main thread_ dan memakainya 
di dalam _spawned thread_. Namun, ini belum bisa jalan, seperti yang bakal kita 
lihat sebentar lagi.

<Listing number="16-3" file-name="src/main.rs" caption="Usaha buat memakai sebuah vector yang dibikin oleh _main thread_ di dalam _thread_ lainnya">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

_Closure_ ini memakai `v`, jadi dia bakal menangkap `v` dan menjadikannya bagian 
dari lingkungan _closure_ tersebut. Karena `thread::spawn` menjalankan _closure_ 
ini di sebuah _thread_ baru, kita seharusnya bisa mengakses `v` di dalam _thread_ 
baru tersebut. Tapi pas kita men-compile contoh ini, kita dapat error berikut:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust _menebak_ (infers) gimana cara menangkap `v`, dan karena `println!` cuma 
butuh sebuah referensi ke `v`, _closure_ tersebut mencoba meminjam (borrow) `v`. 
Namun, ada sebuah masalah: Rust tidak bisa memberi tahu berapa lama _spawned 
thread_ tersebut bakal berjalan, jadi dia tidak tahu apakah referensi ke `v` 
itu bakal selalu valid.

Listing 16-4 memberikan skenario yang punya kemungkinan lebih tinggi di mana 
referensi ke `v` tidak bakal valid.

<Listing number="16-4" file-name="src/main.rs" caption="Sebuah _thread_ dengan _closure_ yang mencoba menangkap sebuah referensi ke `v` dari sebuah _main thread_ yang men-_drop_ `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Kalau Rust mengizinkan kita menjalankan kode ini, ada kemungkinan kalau _spawned 
thread_ tersebut bakal langsung ditaruh di _background_ (latar belakang) tanpa 
sempat dijalankan sama sekali. _Spawned thread_ itu punya referensi ke `v` di 
dalamnya, tapi _main thread_ langsung men-_drop_ (membuang) `v`, memakai 
fungsi `drop` yang sudah kita bahas di Bab 15. Lalu, saat _spawned thread_ 
mulai dieksekusi, `v` sudah tidak valid lagi, jadi referensi ke dia juga ikutan 
tidak valid. Waduh!

Buat membenarkan error _compiler_ di Listing 16-3, kita bisa memakai saran dari 
pesan error-nya:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Dengan menambahkan keyword `move` sebelum _closure_, kita memaksa _closure_ 
buat mengambil kepemilikan dari nilai-nilai yang dia pakai, ketimbang membiarkan 
Rust menebak kalau dia seharusnya meminjam (borrow) nilai-nilai tersebut. 
Modifikasi buat Listing 16-3 yang ditunjukkan di Listing 16-5 bakal bisa 
di-compile dan berjalan sesuai keinginan kita.

<Listing number="16-5" file-name="src/main.rs" caption="Memakai keyword `move` buat memaksa sebuah _closure_ untuk mengambil kepemilikan dari nilai-nilai yang dia pakai">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Kita mungkin tergiur buat mencoba hal yang sama buat membenarkan kode di Listing 
16-4 di mana _main thread_ memanggil `drop`, dengan memakai _closure_ `move`. 
Namun, perbaikan ini tidak bakal bisa karena apa yang coba dilakukan oleh 
Listing 16-4 itu tidak diizinkan buat alasan yang berbeda. Kalau kita menambahkan 
`move` ke _closure_ tersebut, kita bakal memindahkan `v` ke dalam lingkungan 
_closure_-nya, dan kita tidak bisa lagi memanggil `drop` padanya di _main thread_. 
Kita malah bakal dapat error _compiler_ ini:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Aturan _ownership_ (kepemilikan) Rust sudah menyelamatkan kita lagi! Kita dapat 
error dari kode di Listing 16-3 karena Rust bertindak konservatif dan cuma 
meminjam `v` buat _thread_ tersebut, yang mana berarti _main thread_ secara 
teoritis bisa membikin referensi si _spawned thread_ jadi tidak valid. Dengan 
memberi tahu Rust buat memindahkan kepemilikan dari `v` ke _spawned thread_, 
kita menjamin ke Rust kalau _main thread_ tidak bakal memakai `v` lagi. Kalau 
kita mengubah Listing 16-4 dengan cara yang sama, kita malah melanggar aturan 
kepemilikan saat kita mencoba memakai `v` di _main thread_. Keyword `move` 
menimpa (overrides) aturan default konservatif Rust yang melakukan peminjaman 
(borrowing); ia tidak mengizinkan kita buat melanggar aturan kepemilikannya.

Sekarang karena kita sudah membahas apa itu _threads_ dan method-method yang 
disediakan oleh API _thread_, mari kita lihat beberapa situasi di mana kita 
bisa memakai _threads_.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
