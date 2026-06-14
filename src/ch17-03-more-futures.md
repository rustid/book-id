<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Menyerahkan Kontrol (Yielding) ke Runtime

Ingat kembali dari bagian [“Program Asinkron Pertama Kita”][async-program]<!-- 
ignore --> kalau di tiap titik _await_, Rust ngasih kesempatan ke sebuah 
_runtime_ buat me-_pause_ _task_ dan beralih ke _task_ lain kalau _future_ yang 
lagi di-_await_ ternyata belum siap. Kebalikannya juga benar: Rust *hanya* 
me-_pause_ blok asinkron dan menyerahkan kontrol kembali ke _runtime_ pada 
saat titik _await_. Semua hal di antara titik-titik _await_ itu sifatnya sinkron.

Itu artinya kalau kita melakukan banyak pekerjaan di dalam sebuah blok asinkron 
tanpa adanya titik _await_, _future_ tersebut bakal memblokir _futures_ lainnya 
supaya tidak bisa bikin _progress_. Kita mungkin kadang-kadang mendengar hal ini 
disebut sebagai satu _future_ yang me-_starving_ (membuat lapar/menghambat) 
_futures_ lainnya. Di beberapa kasus, itu mungkin bukan masalah besar. Tapi, 
kalau kita lagi melakukan semacam _setup_ yang berat atau pekerjaan yang makan 
waktu lama, atau kalau kita punya sebuah _future_ yang bakal terus-menerus 
melakukan suatu tugas tertentu selamanya, kita perlu memikirkan kapan dan di 
mana harus menyerahkan kontrol kembali ke _runtime_.

Mari kita simulasikan sebuah operasi yang memakan waktu lama buat 
mengilustrasikan masalah *starvation* ini, lalu mengeksplorasi gimana cara 
menyelesaikannya. Listing 17-14 memperkenalkan sebuah fungsi `slow`.

<Listing number="17-14" caption="Memakai `thread::sleep` buat menyimulasikan operasi yang lambat" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Kode ini memakai `std::thread::sleep` bukannya `trpl::sleep` supaya pemanggilan 
`slow` bakal memblokir _thread_ saat ini selama beberapa milidetik. Kita bisa 
memakai `slow` sebagai pengganti buat operasi di dunia nyata yang sifatnya makan 
waktu lama sekaligus memblokir.

Di Listing 17-15, kita memakai `slow` buat meniru pengerjaan tugas semacam 
_CPU-bound_ ini di dalam sepasang _futures_.

<Listing number="17-15" caption="Memanggil fungsi `slow` buat menyimulasikan operasi yang lambat" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

Tiap _future_ menyerahkan kontrol kembali ke _runtime_ cuma *setelah* 
melaksanakan serangkaian operasi lambat. Kalau kita menjalankan kode ini, kita 
bakal melihat output ini:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Sama kayak di Listing 17-5 di mana kita memakai `trpl::select` buat mengadu 
_futures_ yang mengambil dua URL, `select` tetap selesai begitu `a` beres. Tapi 
nggak ada proses selang-seling (interleaving) di antara pemanggilan ke `slow` di 
kedua _futures_ tersebut. _Future_ `a` melakukan semua pekerjaannya sampai 
pemanggilan `trpl::sleep` di-_await_, baru setelah itu _future_ `b` melakukan 
semua pekerjaannya sampai pemanggilan `trpl::sleep`-nya sendiri di-_await_, dan 
akhirnya _future_ `a` selesai. Buat membolehkan kedua _futures_ membikin 
_progress_ di sela-sela tugas lambat mereka, kita butuh titik _await_ supaya 
kita bisa menyerahkan kontrol kembali ke _runtime_. Itu artinya kita butuh 
sesuatu yang bisa kita _await_!

Kita sudah bisa melihat penyerahan kontrol semacam ini terjadi di Listing 17-15: 
kalau seandainya kita menghapus `trpl::sleep` di akhir _future_ `a`, dia bakal 
selesai tanpa _future_ `b` sempat berjalan *sama sekali*. Mari kita coba memakai 
fungsi `trpl::sleep` sebagai titik awal buat membiarkan operasi-operasinya 
bergantian membikin _progress_, kayak yang ditunjukkan di Listing 17-16.

<Listing number="17-16" caption="Memakai `trpl::sleep` buat membiarkan operasi-operasi bergantian membuat progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Kita sudah nambahin pemanggilan `trpl::sleep` dengan titik-titik _await_ di tiap 
sela pemanggilan ke `slow`. Sekarang pekerjaan kedua _futures_ tersebut sudah 
diselang-seling:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

_Future_ `a` tetap jalan sebentar sebelum menyerahkan kontrol ke `b`, karena 
dia memanggil `slow` sebelum pernah memanggil `trpl::sleep`, tapi setelah itu 
_futures_-nya bertukar peran bolak-balik setiap kali salah satu dari mereka 
mencapai titik _await_. Di kasus ini, kita sudah melakukan itu setelah tiap 
pemanggilan ke `slow`, tapi kita bisa membagi-bagi pekerjaannya pakai cara apa 
pun yang paling masuk akal buat kita.

Tapi sebenarnya kita nggak mau bener-bener "tidur" (_sleep_) di sini: kita mau 
bikin _progress_ secepat yang kita bisa. Kita cuma perlu menyerahkan kembali 
kontrol ke _runtime_. Kita bisa melakukan itu secara langsung, menggunakan 
fungsi `trpl::yield_now`. Di Listing 17-17, kita mengganti semua pemanggilan 
`trpl::sleep` tadi dengan `trpl::yield_now`.

<Listing number="17-17" caption="Memakai `yield_now` buat membiarkan operasi bergantian membuat progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

Kode ini terasa lebih jelas soal niat aslinya dan bisa jadi jauh lebih kencang 
ketimbang memakai `sleep`, karena *timers* kayak yang dipakai sama `sleep` 
sering kali punya batas seberapa spesifik (granular) mereka bisa bekerja. Versi 
`sleep` yang kita pakai, misalnya, bakal selalu tidur setidaknya selama satu 
milidetik, biarpun kita memberinya `Duration` sebesar satu nanodetik. Sekali 
lagi, komputer modern itu *cepat*: mereka bisa melakukan banyak hal dalam satu 
milidetik!

Ini artinya asinkron bisa berguna bahkan buat tugas-tugas *compute-bound*, 
tergantung dari apa lagi yang lagi dilakukan sama program kita, karena dia 
menyediakan alat yang berguna buat menstrukturkan hubungan antara berbagai 
bagian program yang berbeda (tapi dengan biaya beban dari _state machine_ 
asinkron tersebut). Ini adalah suatu bentuk _cooperative multitasking_, di mana 
tiap _future_ punya kuasa buat menentukan kapan dia menyerahkan kontrol lewat 
titik-titik _await_. Oleh karena itu, tiap _future_ juga punya tanggung jawab 
buat menghindari memblokir terlalu lama. Di beberapa sistem operasi _embedded_ 
berbasis Rust, ini adalah *satu-satunya* jenis _multitasking_ yang ada!

Di kode dunia nyata, tentu saja kita tidak bakal biasanya menyelingi pemanggilan 
fungsi dengan titik-titik _await_ di tiap baris kodenya. Meskipun menyerahkan 
kontrol pakai cara ini terhitung murah biayanya, tetap saja ia tidak gratis. Di 
banyak kasus, mencoba memecah-mecah tugas *compute-bound* bisa jadi malah 
bikin dia jauh lebih lambat, jadi kadang-kadang lebih baik buat performa 
_keseluruhan_ kalau kita membiarkan sebuah operasi memblokir sebentar. Selalu 
lakukan pengukuran (_measure_) buat melihat di mana sebenarnya letak *bottlenecks* 
performa kode kita. Dinamika dasarnya tetap penting buat diingat, terutama 
kalau kita melihat banyak pekerjaan yang terjadi secara serial padahal kita 
mengharapnya terjadi secara konkuren!

### Membikin Abstraksi Asinkron Kita Sendiri

Kita juga bisa menggabungkan (compose) _futures_ bersama-sama buat membikin pola 
baru. Misalnya, kita bisa membangun fungsi `timeout` dengan blok penyusun 
asinkron yang sudah kita punya. Pas kita sudah selesai, hasilnya bakal jadi 
blok penyusun lainnya yang bisa kita pakai buat membikin abstraksi asinkron 
yang lebih banyak lagi.

Listing 17-18 menunjukkan gimana ekspektasi kita soal cara kerja `timeout` ini 
bersama sebuah _future_ yang lambat.

<Listing number="17-18" caption="Memakai andalan `timeout` kita buat menjalankan operasi lambat dengan batas waktu" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Mari kita implementasikan ini! Sebagai permulaan, mari pikirkan soal API buat 
`timeout`:

- Ia sendiri harus berupa fungsi asinkron supaya kita bisa me-_await_-nya.
- Parameter pertamanya haruslah berupa sebuah _future_ buat dijalankan. Kita 
  bisa membikinnya generik biar dia bisa bekerja buat _future_ apa saja.
- Parameter keduanya adalah waktu maksimal buat menunggu. Kalau kita memakai 
  `Duration`, itu bakal gampang diteruskan ke `trpl::sleep`.
- Ia harus mengembalikan sebuah `Result`. Kalau _future_-nya sukses selesai, 
  `Result`-nya bakal berupa `Ok` dengan nilai yang dihasilkan sama _future_ 
  tersebut. Kalau batas waktunya keburu habis duluan, `Result`-nya bakal berupa 
  `Err` dengan durasi yang sudah dilewati sama si _timeout_.

Listing 17-19 menunjukkan deklarasi ini.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="Mendefinisikan signature dari `timeout`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

Itu sudah memenuhi tujuan kita buat tipe-tipenya. Sekarang mari kita pikirkan 
soal _perilaku_ yang kita butuhkan: kita mau mengadu (_race_) _future_ yang 
dimasukkan tadi melawan durasinya. Kita bisa memakai `trpl::sleep` buat membikin 
_timer future_ dari durasinya, lalu memakai `trpl::select` buat menjalankan 
_timer_ tersebut barengan sama _future_ yang diberikan sama si pemanggil.

Di Listing 17-20, kita mengimplementasikan `timeout` dengan melakukan _match_ 
pada hasil dari me-_await_ `trpl::select`.

<Listing number="17-20" caption="Mendefinisikan `timeout` dengan `select` dan `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

Implementasi dari `trpl::select` itu tidak adil (not fair): ia selalu melakukan 
_polling_ pada argumen-argumennya sesuai urutan saat mereka dioper (implementasi 
`select` lainnya biasanya bakal memilih argumen mana yang mau di-_poll_ duluan 
secara acak). Maka dari itu, kita mengoper `future_to_try` ke `select` duluan 
biar dia dapet kesempatan buat selesai biarpun `max_time` itu durasinya sangat 
singkat. Kalau `future_to_try` beres duluan, `select` bakal mengembalikan 
`Left` yang isinya output dari `future_to_try`. Kalau `timer` beres duluan, 
`select` bakal mengembalikan `Right` berisi output dari timernya yaitu `()`.

Kalau `future_to_try` sukses dan kita dapet `Left(output)`, kita mengembalikan 
`Ok(output)`. Kalau sebaliknya si _sleep timer_ yang habis waktunya dan kita 
dapet `Right(())`, kita mengabaikan `()` tersebut pakai `_` lalu mengembalikan 
`Err(max_time)` sebagai gantinya.

Selesai deh, kita sudah punya fungsi `timeout` yang bisa jalan yang dibangun 
dari dua buah pembantu asinkron lainnya. Kalau kita jalankan kodenya, dia bakal 
mencetak mode kegagalan setelah *timeout* terjadi:

```text
Failed after 2 seconds
```

Karena _futures_ bisa digabung-gabungin bareng _futures_ lain, kita bisa 
membangun alat-alat yang sangat kuat memakai blok penyusun asinkron yang kecil-
kecil. Misalnya, kita bisa memakai pendekatan yang sama ini buat menggabungkan 
*timeouts* dengan *retries*, dan sebaliknya memakai itu semua bersama operasi 
lain kayak pemanggilan jaringan (seperti contoh yang ada di Listing 17-5).

Di praktiknya, biasanya kita bakal bekerja secara langsung dengan `async` dan 
`await`, dan secara sekunder memakai fungsi-fungsi kayak `select` dan macros 
kayak macro `join!` buat mengontrol gimana _futures_ paling luarnya dieksekusi.

Kita sekarang sudah melihat sejumlah cara buat bekerja bareng banyak _futures_ 
di saat yang bersamaan. Selanjutnya, kita bakal melihat gimana cara kita bekerja 
dengan banyak _futures_ di dalam sebuah urutan seiring berjalannya waktu 
menggunakan _streams_.

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
