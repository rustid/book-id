## Memakai Message Passing buat Mentransfer Data Antar Threads

Satu pendekatan yang makin populer buat memastikan konkurensi yang aman 
adalah _message passing_ (pengiriman pesan), di mana _threads_ atau _actors_ 
berkomunikasi dengan mengirimkan pesan berisi data ke satu sama lain. Ide 
ini digambarkan dalam sebuah slogan dari [dokumentasi bahasa Go](https://golang.org/doc/effective_go.html#concurrency):
“Jangan berkomunikasi dengan membagikan (sharing) memori; sebaliknya, 
bagikan memori dengan berkomunikasi.”

Buat mencapai konkurensi pengiriman pesan ini, _standard library_ Rust 
menyediakan sebuah implementasi dari saluran (channels). Sebuah _channel_ 
adalah konsep pemrograman umum di mana data dikirim dari satu _thread_ ke 
_thread_ lainnya.

Kita bisa membayangkan sebuah _channel_ di dalam pemrograman itu kayak 
saluran air yang mengalir ke satu arah, seperti sungai atau selokan. Kalau 
kita menaruh sesuatu kayak bebek karet ke dalam sungai, dia bakal mengalir 
ke hilir (downstream) sampai ke ujung saluran air tersebut.

Sebuah _channel_ punya dua paruh: sebuah _transmitter_ (pemancar/pengirim) 
dan sebuah _receiver_ (penerima). Paruh _transmitter_ adalah lokasi hulu 
(upstream) tempat kita menaruh bebek karetnya ke dalam sungai, dan paruh 
_receiver_ adalah hilir tempat si bebek karet pada akhirnya berlabuh. Satu 
bagian dari kode kita memanggil method-method di _transmitter_ dengan data 
yang mau kita kirim, dan bagian lain mengecek ujung penerima (receiving end) 
buat melihat pesan yang datang. Sebuah _channel_ dikatakan _closed_ (tertutup) 
kalau entah paruh _transmitter_ atau _receiver_-nya di-_drop_ (dibuang).

Di sini, kita bakal perlahan ngebangun sebuah program yang punya satu _thread_ 
buat nge-generate nilai dan mengirimkannya ke dalam sebuah _channel_, dan 
satu _thread_ lain yang bakal menerima nilai-nilai tersebut lalu mencetaknya 
ke layar. Kita bakal mengirim nilai-nilai sederhana antar _threads_ 
memakai sebuah _channel_ buat mengilustrasikan fitur ini. Begitu kita udah 
terbiasa sama tekniknya, kita bisa memakai _channels_ buat _threads_ mana 
aja yang butuh berkomunikasi satu sama lain, kayak sistem _chat_ atau sistem 
di mana banyak _threads_ melakukan bagian-bagian dari sebuah perhitungan 
lalu mengirim bagian-bagian tersebut ke satu _thread_ yang mengagregasikan 
(mengumpulkan) hasilnya.

Pertama, di Listing 16-6, kita bakal membikin sebuah _channel_ tapi tidak 
melakukan apa-apa dengannya. Perhatikan bahwa ini belum bisa di-compile 
karena Rust tidak tahu tipe nilai apa yang mau kita kirim lewat _channel_ ini.

<Listing number="16-6" file-name="src/main.rs" caption="Membikin sebuah _channel_ dan me-assign kedua paruhnya ke `tx` dan `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Kita membikin sebuah _channel_ baru memakai fungsi `mpsc::channel`; `mpsc` 
adalah singkatan dari _multiple producer, single consumer_ (banyak 
penghasil, satu konsumen). Singkatnya, cara _standard library_ Rust 
mengimplementasikan _channels_ berarti sebuah _channel_ bisa punya banyak 
ujung pengirim (_sending_ ends) yang memproduksi nilai tapi cuma bisa punya 
satu ujung penerima (_receiving_ end) yang mengonsumsi nilai-nilai tersebut. 
Bayangin ada banyak aliran sungai kecil yang ngalir bersatu jadi satu 
sungai besar: apa pun yang dikirim ke salah satu sungai kecil itu bakal 
berakhir di satu sungai besar tersebut di ujungnya. Kita bakal mulai dengan 
satu produsen (_producer_) aja buat sekarang, tapi kita bakal nambahin banyak 
produsen pas contoh ini udah bisa jalan.

Fungsi `mpsc::channel` mengembalikan sebuah _tuple_, yang elemen pertamanya 
adalah ujung pengirim (the sending end)—yaitu si _transmitter_—dan elemen 
keduanya adalah ujung penerima (the receiving end)—yaitu si _receiver_. 
Singkatan `tx` dan `rx` secara tradisional sering dipakai di banyak bidang 
untuk masing-masing _transmitter_ dan _receiver_, jadi kita menamai variabel 
kita dengan nama tersebut buat mengindikasikan setiap ujungnya. Kita 
memakai *statement* `let` dengan sebuah pola (pattern) yang men-_destructure_ 
_tuples_ tersebut; kita bakal membahas pemakaian pola di dalam *statements* 
`let` dan _destructuring_ di Bab 19. Buat sekarang, ketahui aja kalau 
memakai *statement* `let` dengan cara ini adalah pendekatan yang nyaman buat 
mengekstrak potongan-potongan dari _tuple_ yang dikembalikan sama 
`mpsc::channel`.

Mari kita pindahkan ujung pengirim (transmitting end) ke dalam _spawned thread_ 
lalu suruh dia mengirim satu string supaya _spawned thread_ tersebut berkomunikasi 
sama _main thread_, seperti yang ditunjukkan di Listing 16-7. Ini ibarat 
naruh bebek karet di bagian hulu sungai atau ngirim pesan _chat_ dari satu 
_thread_ ke _thread_ lainnya.

<Listing number="16-7" file-name="src/main.rs" caption='Memindahkan `tx` ke dalam sebuah _spawned thread_ dan mengirim `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Sekali lagi, kita memakai `thread::spawn` buat membikin _thread_ baru lalu 
memakai `move` buat memindahkan `tx` ke dalam _closure_ supaya _spawned 
thread_ tersebut memiliki (owns) `tx`. _Spawned thread_ perlu memiliki si 
_transmitter_ supaya bisa mengirim pesan lewat _channel_.

_Transmitter_ punya sebuah method `send` yang menerima nilai yang mau kita 
kirim. Method `send` mengembalikan sebuah tipe `Result<T, E>`, jadi kalau 
_receiver_-nya ternyata sudah di-_drop_ dan tidak ada tempat lagi buat ngirim 
nilai, operasi pengirimannya (send) bakal mengembalikan sebuah error. Di 
contoh ini, kita memanggil `unwrap` buat *panic* seandainya terjadi error. Tapi 
di aplikasi betulan (real application), kita bakal menanganinya dengan benar: 
silakan kembali ke Bab 9 buat me-review strategi-strategi buat penanganan 
error yang tepat.

Di Listing 16-8, kita bakal mengambil nilainya dari _receiver_ di dalam 
_main thread_. Ini ibarat memungut bebek karet dari air di ujung hilir sungai 
atau menerima sebuah pesan _chat_.

<Listing number="16-8" file-name="src/main.rs" caption='Menerima nilai `"hi"` di dalam _main thread_ dan mencetaknya'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

_Receiver_ punya dua method yang berguna: `recv` dan `try_recv`. Kita 
memakai `recv`, singkatan dari _receive_ (menerima), yang bakal memblokir 
eksekusi dari _main thread_ dan menunggu sampai sebuah nilai dikirim lewat 
_channel_ tersebut. Begitu ada nilai yang dikirim, `recv` bakal 
mengembalikannya di dalam sebuah `Result<T, E>`. Saat _transmitter_ ditutup, 
`recv` bakal mengembalikan sebuah error buat memberi sinyal bahwa tidak akan 
ada lagi nilai yang bakal datang.

Method `try_recv` tidak melakukan pemblokiran (doesn't block), tapi ia bakal 
langsung mengembalikan sebuah `Result<T, E>`: nilai `Ok` yang memegang sebuah 
pesan kalau pesannya lagi tersedia dan nilai `Err` kalau tidak ada pesan 
sama sekali saat ini. Memakai `try_recv` berguna kalau _thread_ ini punya 
kerjaan lain yang harus dilakuin sambil nunggu pesan: kita bisa nulis 
sebuah _loop_ yang memanggil `try_recv` sesekali, menangani pesannya kalau 
lagi tersedia, dan kalau enggak, ngerjain tugas lain dulu sebentar sampai 
waktunya ngecek lagi.

Kita memakai `recv` di contoh ini buat kesederhanaan; kita tidak punya 
kerjaan lain buat _main thread_ selain menunggu pesan, jadi memblokir 
_main thread_ adalah pilihan yang tepat.

Pas kita menjalankan kode di Listing 16-8, kita bakal melihat nilai yang 
dicetak dari _main thread_:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Sempurna!

### Channels dan Transfer Kepemilikan (Ownership Transference)

Aturan-aturan _ownership_ (kepemilikan) memainkan peran vital dalam 
pengiriman pesan karena mereka ngebantu kita nulis kode konkuren yang aman. 
Mencegah error di pemrograman konkuren adalah keuntungan (advantage) dari 
memikirkan tentang _ownership_ di sepanjang program Rust kita. Mari kita 
lakukan sebuah eksperimen buat nunjukin gimana _channels_ dan _ownership_ 
bekerja bersama-sama mencegah timbulnya masalah: kita bakal nyoba memakai 
sebuah nilai `val` di dalam _spawned thread_ _setelah_ kita ngirim nilai itu 
lewat _channel_. Coba compile kode di Listing 16-9 buat melihat kenapa kode 
ini tidak diizinkan.

<Listing number="16-9" file-name="src/main.rs" caption="Mencoba memakai `val` setelah kita mengirimnya lewat _channel_">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Di sini, kita mencoba buat mencetak `val` setelah kita mengirimnya lewat 
_channel_ via `tx.send`. Mengizinkan ini adalah ide yang buruk: begitu nilainya 
sudah dikirim ke _thread_ lain, _thread_ tersebut bisa aja memodifikasi atau 
men-_drop_-nya sebelum kita nyoba memakai nilai itu lagi. Secara potensial, 
modifikasi yang dilakukan sama _thread_ lain bisa menyebabkan error atau hasil 
yang tidak disangka-sangka akibat adanya data yang tidak konsisten atau sudah 
tidak eksis lagi. Namun, Rust ngasih kita error kalau kita mencoba men-compile 
kode di Listing 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Kesalahan konkurensi (concurrency mistake) kita ini sudah membikin sebuah error 
_compile-time_. Fungsi `send` mengambil kepemilikan atas parameternya, dan 
saat nilainya dipindahkan (moved), _receiver_ bakal mengambil kepemilikannya. 
Hal ini menghentikan kita dari memakai nilai itu secara tidak sengaja lagi 
setelah mengirimnya; sistem _ownership_ memastikan kalau semuanya aman terkendali.

### Mengirim Banyak Nilai dan Melihat Receiver Menunggu

Kode di Listing 16-8 berhasil di-compile dan jalan, tapi dia tidak secara jelas 
menunjukkan ke kita kalau ada dua _threads_ terpisah yang lagi ngobrol satu sama 
lain lewat _channel_.

Di Listing 16-10 kita sudah membuat beberapa modifikasi yang bakal membuktikan 
kalau kode di Listing 16-8 itu berjalan secara konkuren: _spawned thread_ sekarang 
bakal mengirim banyak pesan dan ngasih jeda sebentar (pause) satu detik di 
antara setiap pesannya.

<Listing number="16-10" file-name="src/main.rs" caption="Mengirim banyak pesan dan ngasih jeda (pause) di antara setiap pesannya">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Kali ini, _spawned thread_ punya sebuah _vector_ berisi string yang mau kita 
kirim ke _main thread_. Kita iterasi ngelewatin mereka, ngirim setiap string-nya 
satu-satu, dan ngasih jeda di antara setiap pengiriman dengan memanggil 
fungsi `thread::sleep` beserta sebuah nilai `Duration` satu detik.

Di dalam _main thread_, kita tidak memanggil fungsi `recv` secara eksplisit 
lagi: sebaliknya, kita memperlakukan `rx` sebagai sebuah iterator. Buat setiap 
nilai yang diterima, kita bakal mencetaknya. Saat _channel_-nya ditutup, 
iterasinya bakal berakhir.

Pas kita menjalankan kode di Listing 16-10, kita seharusnya melihat output 
berikut dengan jeda satu detik di antara setiap barisnya:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Karena kita tidak punya kode yang melakukan jeda atau penundaan (delays) di 
dalam _for loop_ di _main thread_, kita bisa tahu kalau _main thread_ tersebut 
lagi nunggu buat nerima nilai-nilai dari _spawned thread_.

### Membikin Banyak Producers dengan Meng-clone si Transmitter

Tadi kita sempat menyebutkan kalau `mpsc` adalah singkatan dari _multiple producer, 
single consumer_ (banyak penghasil, satu konsumen). Mari kita manfaatin 
`mpsc` ini lalu ekspansi kode di Listing 16-10 buat membikin beberapa _threads_ 
yang semuanya mengirim nilai ke satu _receiver_ yang sama. Kita bisa melakukan 
itu dengan meng-_clone_ _transmitter_-nya, seperti yang ditunjukkan di Listing 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="Mengirim banyak pesan dari banyak *producers*">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Kali ini, sebelum kita membikin _spawned thread_ yang pertama, kita memanggil 
`clone` pada _transmitter_-nya. Ini bakal ngasih kita sebuah _transmitter_ baru 
yang bisa kita teruskan ke _spawned thread_ yang pertama. Kita meneruskan 
_transmitter_ aslinya ke sebuah _spawned thread_ yang kedua. Hal ini ngasih 
kita dua _threads_, di mana masing-masing mengirim pesan yang berbeda ke 
satu _receiver_ yang sama.

Pas kita menjalankan kodenya, output kita harusnya bakal kelihatan kurang 
lebih kayak gini:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

kita mungkin bakal melihat nilai-nilainya dalam urutan yang berbeda, 
tergantung dari sistem yang kita pakai. Inilah yang membikin konkurensi 
jadi hal yang menarik sekaligus sulit. Kalau kita eksperimen sama 
`thread::sleep`, ngasih dia nilai yang beda-beda di berbagai _threads_ 
tersebut, masing-masing jalan (run) bakal jadi makin tidak deterministik 
dan menghasilkan output yang berbeda-beda setiap kalinya.

Sekarang setelah kita melihat gimana _channels_ itu bekerja, mari kita lihat 
metode konkurensi yang berbeda.
