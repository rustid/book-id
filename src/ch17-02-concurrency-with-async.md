## Menerapkan Konkurensi dengan Async

Di bagian ini, kita bakal menerapkan asinkron ke beberapa tantangan konkurensi 
yang sama dengan yang sudah kita tangani memakai _threads_ di Bab 16. Karena 
kita sudah banyak ngomongin ide-ide kuncinya di sana, di bagian ini kita bakal 
fokus sama apa aja yang berbeda antara _threads_ dan _futures_.

Di banyak kasus, API buat bekerja bareng konkurensi memakai asinkron itu mirip 
sekali sama yang dipakai buat _threads_. Di kasus lain, mereka jadinya lumayan 
berbeda. Bahkan kalau API-nya _kelihatan_ mirip antara _threads_ dan asinkron, 
mereka sering kali punya perilaku yang berbeda—dan mereka hampir selalu punya 
karakteristik performa yang berbeda.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Membikin Task (Tugas) Baru dengan `spawn_task`

Operasi pertama yang kita tangani di bagian [“Membikin Thread Baru dengan 
`spawn`”][thread-spawn]<!-- ignore --> di Bab 16 adalah menghitung angka di dua 
_threads_ terpisah. Mari kita lakukan hal yang sama memakai asinkron. _Crate_ 
`trpl` menyediakan fungsi `spawn_task` yang kelihatannya mirip sekali sama API 
`thread::spawn`, dan fungsi `sleep` yang merupakan versi asinkron dari API 
`thread::sleep`. Kita bisa memakai keduanya bersama-sama buat 
mengimplementasikan contoh penghitungan angka tersebut, kayak yang ditunjukkan 
di Listing 17-6.

<Listing number="17-6" caption="Membikin task baru buat mencetak satu hal sementara task utama mencetak hal lainnya" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Sebagai titik awal, kita menyiapkan fungsi `main` kita pakai `trpl::block_on` 
supaya fungsi tingkat teratas kita bisa bersifat asinkron.

> Catatan: Mulai dari titik ini di bab ini, setiap contoh bakal menyertakan kode 
> pembungkus yang sama persis pakai `trpl::block_on` di `main`, jadi kita bakal 
> sering mengabaikannya (skip) sama seperti kita mengabaikan `main`. Jangan lupa 
> buat menyertakannya di dalam kode kita ya!

Terus kita menulis dua _loops_ di dalam blok tersebut, masing-masing mengandung 
pemanggilan `trpl::sleep`, yang bakal nunggu selama setengah detik (500 
milidetik) sebelum mengirim pesan berikutnya. Kita menaruh satu _loop_ di dalam 
isi dari `trpl::spawn_task` dan satu lagi di dalam _loop_ `for` tingkat teratas. 
Kita juga nambahin `await` setelah pemanggilan `sleep`.

Kode ini berperilaku mirip sama implementasi berbasis _thread_—termasuk fakta 
bahwa kita mungkin bakal melihat pesan-pesannya muncul dalam urutan yang berbeda 
di terminal kita pas kita menjalankannya:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Versi ini bakal berhenti begitu _loop_ `for` di dalam isi blok asinkron utamanya 
selesai, karena _task_ yang ditelurkan sama `spawn_task` bakal dimatikan pas 
fungsi `main` berakhir. Kalau kita pengen _task_ tersebut jalan sampai kelar, 
kita perlu memakai _join handle_ buat nungguin _task_ pertamanya selesai. Pas 
pakai _threads_, kita memakai method `join` buat "memblokir" sampai _thread_-nya 
selesai jalan. Di Listing 17-7, kita bisa memakai `await` buat ngelakuin hal yang 
sama, karena _task handle_ itu sendiri adalah sebuah _future_. Tipe `Output`-nya 
adalah sebuah `Result`, jadi kita juga meng-_unwrap_-nya setelah me-_await_-nya.

<Listing number="17-7" caption="Memakai `await` bersama join handle buat menjalankan task sampai selesai" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Versi yang sudah di-update ini bakal jalan sampai _kedua_ _loops_ selesai:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Sejauh ini, kelihatannya asinkron dan _threads_ ngasih kita hasil yang mirip, 
cuma beda di sintaksnya aja: memakai `await` ketimbang memanggil `join` pada 
_join handle_, dan me-_await_ pemanggilan `sleep`.

Perbedaan besarnya adalah kita tidak perlu menelurkan (spawn) _thread_ sistem 
operasi lainnya buat melakukan hal ini. Faktanya, kita bahkan tidak perlu 
menelurkan sebuah _task_ di sini. Karena blok asinkron dikompilasi jadi 
_futures_ anonim, kita bisa menaruh tiap _loop_ di dalam blok asinkron lalu 
menyuruh _runtime_ menjalankan keduanya sampai selesai memakai fungsi 
`trpl::join`.

Di bagian [“Menunggu Semua Thread sampai Selesai”][join-handles]<!-- ignore --> 
di Bab 16, kita menunjukkan gimana cara memakai method `join` pada tipe 
`JoinHandle` yang dikembalikan pas kita memanggil `std::thread::spawn`. Fungsi 
`trpl::join` itu mirip, tapi buat _futures_. Pas kita ngasih dia dua _futures_, 
dia bakal memproduksi satu _future_ baru yang output-nya adalah sebuah _tuple_ 
berisi output dari tiap _future_ yang kita masukkan tadi begitu _keduanya_ 
selesai. Jadi, di Listing 17-8, kita memakai `trpl::join` buat nungguin baik 
`fut1` maupun `fut2` sampai selesai. Kita *tidak* me-_await_ `fut1` dan `fut2` 
melainkan me-_await_ _future_ baru yang dihasilkan sama `trpl::join`. Kita 
ngabaiin output-nya, karena dia cuma sebuah _tuple_ berisi dua nilai unit.

<Listing number="17-8" caption="Memakai `trpl::join` buat menunggu dua futures anonim" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Pas kita jalankan ini, kita melihat kedua _futures_ jalan sampai selesai:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Nah, kita bakal melihat urutan yang sama persis di tiap jalannya, yang mana 
berbeda sekali sama apa yang kita lihat di _threads_ dan di `trpl::spawn_task` 
di Listing 17-7. Itu gara-gara fungsi `trpl::join` ini sifatnya _adil_ (fair), 
yang artinya dia mengecek tiap _future_ dengan frekuensi yang sama, ganti-
gantian di antara mereka, dan tidak pernah membiarkan satu pun balapan (race) 
mendahului yang lain kalau yang lainnya juga sudah siap. Kalau pakai _threads_, 
sistem operasilah yang menentukan _thread_ mana yang mau dicek dan seberapa 
lama dia dibiarkan jalan. Kalau di Rust asinkron, _runtime_-lah yang menentukan 
_task_ mana yang mau dicek. (Di praktiknya, detail-detailnya jadi rumit karena 
sebuah _async runtime_ mungkin memakai _threads_ sistem operasi di balik layar 
sebagai bagian dari caranya mengelola konkurensi, jadi menjamin keadilan bisa 
jadi kerjaan ekstra buat sebuah _runtime_—tapi tetap mungkin kok!) _Runtimes_ 
tidak diwajibkan buat menjamin keadilan buat sembarang operasi, dan mereka 
sering kali menawarkan API yang berbeda-beda biar kita bisa milih mau yang adil 
atau nggak.

Cobain deh beberapa variasi me-_await_ _futures_ ini dan lihat apa yang mereka 
lakukan:

- Hapus blok asinkron dari salah satu atau kedua _loops_ tersebut.
- _Await_ tiap blok asinkron seketika setelah mendefinisikannya.
- Bungkus cuma _loop_ pertama saja di dalam blok asinkron, dan _await_ _future_ 
  hasilnya setelah isi dari _loop_ kedua.

Buat tantangan ekstra, coba deh tebak bakal kayak apa output-nya di tiap kasus 
_sebelum_ kita menjalankan kodenya!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### Mengirim Data di Antara Dua Task Memakai Message Passing

Berbagi data antar _futures_ juga bakal terasa familier: kita bakal memakai 
_message passing_ lagi, tapi kali ini pakai versi asinkron dari tipe-tipe dan 
fungsi-fungsinya. Kita bakal ngambil jalan yang agak beda dibanding waktu di 
bagian [“Transfer Data antar Threads Memakai Message Passing”][message-passing-threads]<!-- 
ignore --> di Bab 16 buat mengilustrasikan beberapa perbedaan kunci antara 
konkurensi berbasis _thread_ dan berbasis _futures_. Di Listing 17-9, kita bakal 
mulai dengan cuma satu blok asinkron saja—*tidak* menelurkan sebuah _task_ 
terpisah sebagaimana dulu kita menelurkan sebuah _thread_ terpisah.

<Listing number="17-9" caption="Membikin sebuah async channel dan memberikan kedua bagiannya ke `tx` dan `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Di sini, kita memakai `trpl::channel`, sebuah versi asinkron dari API *multiple-
producer, single-consumer channel* yang kita pakai bareng _threads_ dulu di Bab 
16. Versi asinkron dari API-nya cuma beda sedikit dari versi berbasis _thread_: 
ia memakai _receiver_ `rx` yang bersifat *mutable* bukannya *immutable*, dan 
method `recv`-nya menghasilkan sebuah _future_ yang perlu kita _await_ bukannya 
menghasilkan nilainya secara langsung. Sekarang kita bisa mengirim pesan dari 
sisi pengirim ke sisi penerima. Perhatikan bahwa kita tidak harus menelurkan 
sebuah _thread_ terpisah atau bahkan sebuah _task_; kita cuma butuh me-_await_ 
pemanggilan `rx.recv`.

Method `Receiver::recv` yang sinkron di `std::mpsc::channel` memblokir sampai 
ia menerima pesan. Method `trpl::Receiver::recv` tidak memblokir, karena ia 
sifatnya asinkron. Alih-alih memblokir, ia menyerahkan kontrol kembali ke 
_runtime_ sampai entah ada pesan yang diterima atau sisi pengirim (send side) 
dari *channel* tersebut ditutup. Sebaliknya, kita tidak me-_await_ pemanggilan 
`send`, karena ia tidak memblokir. Dia tidak perlu memblokir karena *channel* 
yang kita pakai buat mengirim ini sifatnya *unbounded* (tidak dibatasi).

> Catatan: Karena semua kode asinkron ini jalan di dalam blok asinkron di dalam 
> pemanggilan `trpl::block_on`, semua hal di dalamnya bisa menghindari memblokir. 
> Namun, kode di *luar* blok tersebut bakal memblokir pada saat fungsi 
> `block_on` dikembalikan. Itulah poin utama dari fungsi `trpl::block_on`: ia 
> membiarkan kita *memilih* di mana harus memblokir pada sekumpulan kode 
inkron, dan dengan begitu bisa bertransisi antara kode sinkron dan asinkron.

Perhatikan dua hal soal contoh ini. Pertama, pesannya bakal tiba seketika itu 
juga. Kedua, meskipun kita memakai sebuah _future_ di sini, belum ada 
konkurensi sama sekali. Semua hal di dalam listing tersebut terjadi secara 
berurutan (sequence), persis kayak yang bakal terjadi kalau seandainya tidak ada 
_futures_ yang dilibatkan.

Mari kita beresin bagian pertamanya dengan mengirim serangkaian pesan dan tidur 
di antara mereka, kayak yang ditunjukkan di Listing 17-10.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Mengirim dan menerima banyak pesan melewati async channel dan tidur dengan sebuah `await` di antara tiap pesan" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Selain mengirim pesannya, kita juga perlu menerimanya. Di kasus ini, karena kita 
tahu ada berapa banyak pesan yang masuk, kita bisa melakukan itu secara manual 
dengan memanggil `rx.recv().await` sebanyak empat kali. Tapi di dunia nyata, 
kita umumnya bakal nungguin pesan yang jumlahnya *tidak diketahui*, jadi kita 
butuh terus menunggu sampai kita yakin sudah tidak ada pesan lagi.

Di Listing 16-10, kita memakai _loop_ `for` buat memproses semua item yang 
diterima dari sebuah _channel_ yang sinkron. Namun, Rust belum punya cara buat 
memakai _loop_ `for` dengan serangkaian item yang *diproduksi secara asinkron*, 
jadi kita perlu memakai jenis _loop_ yang belum pernah kita lihat sebelumnya: 
yaitu perulangan bersyarat `while let`. Ini adalah versi _loop_ dari konstruk 
`if let` yang sudah kita lihat di Bab 6 di bagian [“Control Flow Singkat Memakai 
`if let` dan `let...else`”][if-let]<!-- ignore -->. _Loop_ ini bakal terus 
dijalankan selama _pattern_ yang ditentukannya terus cocok sama nilainya.

Pemanggilan `rx.recv` menghasilkan sebuah _future_, yang kemudian kita _await_. 
_Runtime_ bakal me-_pause_ _future_ tersebut sampai dia siap. Begitu sebuah 
pesan tiba, _future_ tersebut bakal selesai (resolve) jadi `Some(message)` 
sebanyak jumlah pesan yang tiba. Saat *channel*-nya ditutup, terlepas dari 
apakah ada pesan yang tiba atau nggak, _future_ tersebut bakal selesai jadi 
`None` buat mengindikasikan kalau sudah tidak ada lagi nilainya dan oleh karena 
itu kita harus berhenti melakukan _polling_—yakni, berhenti me-_await_.

_Loop_ `while let` menggabungkan ini semua. Kalau hasil dari pemanggilan 
`rx.recv().await` adalah `Some(message)`, kita dapet akses ke pesannya dan kita 
bisa memakainya di dalam isi _loop_, persis kayak pas pakai `if let`. Kalau 
hasilnya `None`, _loop_-nya berakhir. Setiap kali _loop_-nya selesai, dia kena 
_await point_ lagi, jadi _runtime_ me-_pause_-nya lagi sampai ada pesan lain 
yang tiba.

Kodenya sekarang berhasil mengirim dan menerima semua pesan tersebut. 
Sayangnya, masih ada beberapa masalah. Salah satunya, pesan-pesannya tidak tiba 
dalam interval setengah detik. Mereka tiba semuanya sekaligus, 2 detik (2.000 
milidetik) setelah kita menyalakan programnya. Terus, program ini juga tidak 
pernah berakhir! Sebaliknya, ia menunggu selamanya buat pesan baru. Kita bakal 
perlu mematikannya memakai <kbd>ctrl</kbd>-<kbd>C</kbd>.

#### Kode di Dalam Satu Blok Asinkron Dieksekusi secara Linear

Mari kita mulai dengan menyelidiki kenapa pesan-pesannya datang sekaligus 
setelah penundaan penuh (full delay), bukannya datang dengan jeda di tiap 
pesannya. Di dalam sebuah blok asinkron tertentu, urutan munculnya keyword 
`await` di dalam kode juga merupakan urutan saat mereka dieksekusi pas programnya 
jalan.

Cuma ada satu blok asinkron di Listing 17-10, jadi semua hal di dalamnya jalan 
secara linear. Masih belum ada konkurensi. Semua pemanggilan `tx.send` terjadi, 
diselingi sama semua pemanggilan `trpl::sleep` dan _await points_ terkaitnya. 
Baru setelah itu _loop_ `while let` dapat giliran buat melewati titik _await_ 
apa pun pada pemanggilan `recv`.

Buat mendapatkan perilaku yang kita mau, di mana jeda tidurnya (sleep delay) 
terjadi di antara tiap pesan, kita perlu menaruh operasi `tx` dan `rx` di dalam 
blok asinkronnya masing-masing, kayak yang ditunjukkan di Listing 17-11. Terus 
si _runtime_ bisa mengeksekusi masing-masing blok secara terpisah memakai 
`trpl::join`, persis kayak di Listing 17-8. Sekali lagi, kita me-_await_ hasil 
pemanggilan `trpl::join`, bukannya me-_await_ _futures_ individunya. Kalau kita 
me-_await_ _futures_ individunya secara berurutan, kita ujung-ujungnya cuma 
bakal balik lagi ke alur sekuensial—persis kayak apa yang mau kita hindari.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="Memisahkan `send` dan `recv` ke dalam blok asinkronnya masing-masing dan menunggu futures dari blok-blok tersebut" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Dengan kode yang sudah di-update di Listing 17-11, pesan-pesannya dicetak dalam 
interval 500 milidetik, bukannya buru-buru sekaligus setelah 2 detik.

#### Memindahkan Kepemilikan (Ownership) ke dalam Blok Asinkron

Meskipun begitu, programnya tetap tidak pernah berakhir, karena cara _loop_ 
`while let` berinteraksi sama `trpl::join`:

- _Future_ yang dikembalikan dari `trpl::join` cuma selesai begitu *kedua* 
  _futures_ yang diberikan ke dia sudah selesai.
- _Future_ `tx_fut` selesai begitu dia kelar tidur setelah mengirim pesan 
  terakhir di dalam `vals`.
- _Future_ `rx_fut` tidak bakal selesai sampai _loop_ `while let` berakhir.
- _Loop_ `while let` tidak bakal berakhir sampai me-_await_ `rx.recv` 
  menghasilkan `None`.
- Me-_await_ `rx.recv` bakal mengembalikan `None` cuma kalau sisi lain dari 
  *channel*-nya sudah ditutup.
- *Channel*-nya bakal tutup cuma kalau kita memanggil `rx.close` atau pas sisi 
  pengirimnya, `tx`, di-_drop_.
- Kita tidak memanggil `rx.close` di mana pun, dan `tx` tidak bakal di-_drop_ 
  sampai blok asinkron terluar yang diberikan ke `trpl::block_on` berakhir.
- Blok tersebut tidak bisa berakhir karena dia terhambat (blocked) menunggu 
  `trpl::join` selesai, yang mana membawa kita balik lagi ke urutan paling atas 
  dari daftar ini.

Saat ini, blok asinkron tempat kita mengirim pesannya cuma *meminjam* `tx` 
karena mengirim pesan tidak mewajibkan adanya kepemilikan, tapi kalau seandainya 
kita bisa *memindahkan* (`move`) `tx` ke dalam blok asinkron tersebut, dia bakal 
di-_drop_ begitu blok itu berakhir. Di Bab 13 di bagian [“Menangkap Referensi 
atau Memindahkan Kepemilikan”][capture-or-move]<!-- ignore -->, kita sudah 
mempelajari cara memakai keyword `move` bersama _closures_, dan, seperti yang 
dibahas di bagian [“Memakai `move` Closures bersama Threads”][move-threads]<!-- 
ignore --> di Bab 16, kita sering kali perlu memindahkan data ke dalam 
_closures_ saat bekerja dengan _threads_. Dinamika dasar yang sama ini juga 
berlaku buat blok asinkron, jadi keyword `move` juga bekerja di blok asinkron 
sama seperti di _closures_.

Di Listing 17-12, kita mengubah blok yang dipakai buat mengirim pesan dari 
`async` jadi `async move`.

<Listing number="17-12" caption="Revisi dari kode di Listing 17-11 yang mematikan program secara benar begitu selesai" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Pas kita menjalankan versi kode *yang ini*, dia bakal berhenti secara anggun 
begitu pesan terakhir sudah dikirim dan diterima. Berikutnya, mari kita lihat 
apa yang butuh diubah buat mengirim data dari lebih dari satu _future_.

#### Menggabungkan (Joining) Sejumlah Futures dengan Macro `join!`

*Async channel* ini juga merupakan *multiple-producer channel*, jadi kita bisa 
memanggil `clone` pada `tx` kalau kita mau mengirim pesan dari banyak _futures_, 
kayak yang ditunjukkan di Listing 17-13.

<Listing number="17-13" caption="Memakai multiple producers bersama blok asinkron" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Pertama, kita meng-_clone_ `tx`, membikin `tx1` di luar blok asinkron pertama. 
Kita memindahkan `tx1` ke dalam blok tersebut sama kayak yang kita lakukan 
sebelumnya sama `tx`. Terus, belakangan, kita memindahkan `tx` asli ke dalam 
blok asinkron *baru*, di mana kita mengirim lebih banyak pesan dengan jeda yang 
sedikit lebih lambat. Kebetulan kita menaruh blok asinkron baru ini setelah blok 
asinkron buat menerima pesan, tapi kita juga bisa kok menaruhnya sebelumnya. 
Kuncinya adalah urutan pas _futures_-nya di-_await_, bukan urutan pas mereka 
dibikin.

Kedua blok asinkron buat mengirim pesannya wajib berupa blok `async move` supaya 
baik `tx` maupun `tx1` di-_drop_ pas blok-blok tersebut selesai. Kalau tidak, 
kita bakal balik lagi ke perulangan tiada henti yang sama kayak di awal tadi.

Terakhir, kita beralih dari `trpl::join` ke `trpl::join!` buat menangani 
_future_ tambahannya: macro `join!` me-_await_ jumlah _futures_ yang sembarang 
asalkan kita sudah tahu jumlah _futures_-nya pas masa kompilasi. Kita bakal 
ngebahas soal menunggu sekumpulan _futures_ yang jumlahnya tidak diketahui nanti 
di bab ini.

Sekarang kita bisa melihat semua pesan dari kedua _futures_ pengirimnya, dan 
karena _futures_ pengirimnya memakai jeda yang sedikit berbeda setelah 
mengirim, pesan-pesannya juga diterima dalam interval yang berbeda tersebut:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Kita sudah mengeksplorasi cara memakai _message passing_ buat mengirim data 
antar _futures_, gimana kode di dalam blok asinkron jalan secara sekuensial, 
gimana cara memindahkan kepemilikan ke dalam blok asinkron, dan gimana cara 
menggabungkan banyak _futures_. Berikutnya, mari kita bahas gimana dan kenapa 
harus kasih tahu _runtime_ kalau dia bisa beralih ke tugas lain.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
