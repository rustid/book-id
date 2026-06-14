## Menyatukan Semuanya: Futures, Tasks, dan Threads

Seperti yang sudah kita lihat di [Bab 16][ch16]<!-- ignore -->, _threads_ 
menyediakan salah satu pendekatan buat konkurensi. Kita juga sudah melihat 
pendekatan lainnya di bab ini: memakai asinkron dengan _futures_ dan _streams_. 
Kalau kita bertanya-tanya kapan harus memilih salah satu metode di atas metode 
lainnya, jawabannya adalah: tergantung! Dan di banyak kasus, pilihannya bukan 
antara _threads_ *atau* asinkron, melainkan _threads_ *dan* asinkron.

Banyak sistem operasi sudah menyuplai model konkurensi berbasis _threading_ 
selama puluhan tahun, dan banyak bahasa pemrograman yang jadi menyokongnya 
sebagai hasilnya. Namun, model-model ini bukannya tanpa pertukaran (*tradeoffs*). 
Di banyak sistem operasi, tiap _thread_ memakan memori dalam jumlah yang cukup 
besar. _Threads_ juga cuma jadi opsi pas sistem operasi dan perangkat keras kita 
memang menyokongnya. Beda sama komputer desktop dan ponsel arus utama, beberapa 
sistem _embedded_ bahkan tidak punya sistem operasi sama sekali, jadinya mereka 
juga tidak punya _threads_.

Model asinkron menyediakan kumpulan *tradeoffs* yang berbeda—dan pada akhirnya 
saling melengkapi. Di dalam model asinkron, operasi konkuren tidak mewajibkan 
adanya _thread_ masing-masing. Sebagai gantinya, mereka bisa berjalan di atas 
_tasks_ (tugas), kayak pas kita memakai `trpl::spawn_task` buat memulai 
pekerjaan dari sebuah fungsi sinkron di bagian _streams_. Sebuah _task_ itu 
mirip sama _thread_, tapi bukannya dikelola oleh sistem operasi, ia dikelola 
sama kode di tingkat _library_: yaitu _runtime_.

Ada alasannya kenapa API buat menelurkan _threads_ dan menelurkan _tasks_ itu 
sangat mirip. _Threads_ bertindak sebagai batas (boundary) buat sekumpulan 
operasi sinkron; konkurensi dimungkinkan *di antara* _threads_. _Tasks_ 
bertindak sebagai batas buat sekumpulan operasi *asinkron*; konkurensi 
dimungkinkan baik *di antara* maupun *di dalam* _tasks_, karena sebuah _task_ 
bisa berganti-ganti di antara _futures_ di dalam isinya. Terakhir, _futures_ 
adalah unit konkurensi Rust yang paling spesifik (granular), dan tiap _future_ 
bisa merepresentasikan sebuah pohon berisi _futures_ lainnya. _Runtime_—lebih 
spesifiknya, *executor*-nya—mengelola _tasks_, dan _tasks_ mengelola _futures_. 
Dalam hal tersebut, _tasks_ itu mirip kayak _threads_ yang ringan dan dikelola 
sama _runtime_ dengan tambahan kapabilitas yang datang dari fakta bahwa ia 
dikelola sama _runtime_ bukannya sama sistem operasi.

Ini tidak berarti kalau _async tasks_ itu selalu lebih baik dibanding _threads_ 
(atau sebaliknya). Konkurensi memakai _threads_ dalam beberapa hal merupakan 
model pemrograman yang lebih simpel dibanding konkurensi memakai `async`. Itu 
bisa jadi kekuatan atau kelemahan. _Threads_ itu semacam "nyalakan dan lupakan" 
(_fire and forget_); mereka nggak punya padanan asli terhadap _future_, jadi 
mereka sekadar jalan sampai selesai tanpa bisa diinterupsi kecuali oleh sistem 
operasinya sendiri.

Dan ternyata _threads_ dan _tasks_ itu sering kali bekerja barengan dengan 
sangat baik, karena _tasks_ (setidaknya di beberapa _runtimes_) bisa dipindah-
pindahkan antar _threads_. Bahkan, di balik layar, _runtime_ yang sudah kita 
pakai—termasuk fungsi `spawn_blocking` dan `spawn_task`—sifatnya adalah *multi-
threaded* secara bawaan! Banyak _runtimes_ memakai pendekatan bernama *work 
stealing* buat memindah-mindahkan _tasks_ antar _threads_ secara transparan, 
berdasarkan gimana tiap _thread_ tersebut sedang dimanfaatkan saat itu, demi 
meningkatkan performa keseluruhan sistem. Pendekatan tersebut sebenarnya 
mewajibkan adanya _threads_ *dan* _tasks_, dan oleh karenanya juga _futures_.

Pas memikirkan metode mana yang mau dipakai kapan, coba pertimbangkan aturan 
praktis (_rules of thumb_) berikut:

- Kalau pekerjaannya *sangat bisa diparalelkan* (yaitu, *CPU-bound*), kayak 
  memproses sekumpulan data di mana tiap bagiannya bisa diproses secara 
  terpisah, _threads_ adalah pilihan yang lebih oke.
- Kalau pekerjaannya *sangat konkuren* (yaitu, *I/O-bound*), kayak menangani 
  pesan-pesan dari sekumpulan sumber berbeda yang mungkin datang di interval 
  atau kecepatan yang berbeda-beda, asinkron adalah pilihan yang lebih oke.

Dan kalau kita butuh baik paralelisme maupun konkurensi, kita tidak harus 
memilih antara _threads_ dan asinkron. Kita bisa memakai keduanya bersamaan 
dengan bebas, membiarkan masing-masing memainkan peran yang paling dikuasainya. 
Misalnya, Listing 17-25 menunjukkan contoh yang lumayan umum dari campuran kayak 
gini di kode Rust dunia nyata.

<Listing number="17-25" caption="Mengirim pesan pakai kode memblokir (blocking) di dalam sebuah thread dan menunggu pesan-pesan tersebut di dalam blok asinkron" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:all}}
```

</Listing>

Kita mulai dengan membikin asinkron _channel_, terus menelurkan sebuah _thread_ 
yang mengambil kepemilikan dari sisi pengirim _channel_ tersebut memakai keyword 
`move`. Di dalam _thread_ tersebut, kita mengirim angka 1 sampai 10, sambil 
tidur selama satu detik di tiap angkanya. Terakhir, kita menjalankan sebuah 
_future_ yang dibuat pakai blok asinkron yang dioper ke `trpl::block_on` persis 
kayak apa yang sudah kita lakukan di sepanjang bab ini. Di dalam _future_ 
tersebut, kita me-_await_ pesan-pesan tadi, persis kayak di contoh-contoh 
_message-passing_ lainnya yang sudah kita lihat.

Buat balik lagi ke skenario yang kita buka di awal bab, bayangkan menjalankan 
sekumpulan tugas *video encoding* memakai _thread_ khusus (karena *video 
encoding* itu sifatnya *compute-bound*) tapi memberikan notifikasi ke UI kalau 
operasi tersebut sudah selesai memakai asinkron _channel_. Ada banyak sekali 
contoh dari jenis kombinasi kayak gini di kasus penggunaan dunia nyata.

## Ringkasan

Ini bukan kali terakhir kita melihat soal konkurensi di buku ini. Proyek di [Bab 
21][ch21]<!-- ignore --> bakal menerapkan konsep-konsep ini di situasi yang 
lebih realistis dibanding contoh-contoh simpel yang dibahas di sini dan 
membandingkan penyelesaian masalah memakai _threading_ versus _tasks_ dan 
_futures_ secara lebih langsung.

Metode mana pun yang kita pilih, Rust memberi kita alat-alat yang kita butuhkan 
buat menulis kode konkuren yang aman dan kencang—baik itu buat *web server* yang 
*high-throughput* maupun buat sistem operasi *embedded*.

Berikutnya, kita bakal ngomongin soal cara yang idiomatik buat memodelkan masalah 
dan menstrukturkan solusi seiring dengan semakin besarnya program Rust kita. 
Selain itu, kita bakal membahas gimana kaitan antara idiom milik Rust dengan 
idiom yang mungkin kita sudah familier di pemrograman berorientasi objek 
(_object-oriented programming_).

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
