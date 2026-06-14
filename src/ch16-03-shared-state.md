## Konkurensi Shared-State (Keadaan Berbagi)

Pengiriman pesan (_message passing_) adalah cara yang bagus buat menangani 
konkurensi, tapi itu bukan satu-satunya cara. Metode lain adalah dengan 
membiarkan banyak _threads_ buat mengakses data bersama (_shared data_) yang 
sama. Ingat kembali slogan dari dokumentasi bahasa Go ini: “Jangan berkomunikasi 
dengan membagikan (sharing) memori.”

Kira-kira gimana sih bentuknya berkomunikasi dengan membagikan memori itu? 
Selain itu, kenapa para penggemar _message-passing_ mewanti-wanti buat tidak 
memakai berbagi memori (_memory sharing_)?

Di satu sisi, _channels_ (saluran) di bahasa pemrograman mana pun itu mirip 
dengan kepemilikan tunggal (_single ownership_) karena begitu kita mentransfer 
sebuah nilai lewat _channel_, kita tidak seharusnya memakai nilai itu lagi. 
Konkurensi memori bersama (_shared-memory concurrency_) itu kayak kepemilikan 
ganda (_multiple ownership_): banyak _threads_ bisa mengakses lokasi memori 
yang sama di saat yang bersamaan. Seperti yang udah kita lihat di Bab 15, di mana 
_smart pointers_ memungkinkan kepemilikan ganda, kepemilikan ganda bisa 
nambahin kerumitan (complexity) karena pemilik-pemilik yang berbeda ini butuh 
dikelola (managed). Sistem tipe (type system) dan aturan _ownership_ Rust 
ngebantu sekali buat bikin pengelolaan ini jadi benar. Sebagai contoh, mari 
kita lihat _mutexes_, salah satu dari struktur data konkurensi (_concurrency 
primitives_) yang paling umum dipakai buat memori bersama.

### Memakai Mutexes buat Mengizinkan Akses ke Data dari Satu Thread dalam Satu Waktu

_Mutex_ adalah singkatan dari _mutual exclusion_ (pengecualian timbal balik), 
yang artinya sebuah mutex cuma mengizinkan satu _thread_ aja buat mengakses 
beberapa data di satu waktu tertentu. Buat mengakses data di dalam sebuah mutex, 
sebuah _thread_ pertama-tama harus ngasih sinyal kalau dia mau akses dengan 
meminta buat ngambil (_acquire_) _lock_ (kunci) milik mutex tersebut. _Lock_ 
adalah struktur data yang jadi bagian dari mutex yang melacak siapa yang 
saat ini punya akses eksklusif ke datanya. Oleh karena itu, mutex digambarkan 
sebagai _penjaga_ (_guarding_) data yang dia pegang melalui sistem *locking* ini.

Mutexes terkenal susah dipakai karena kita harus ingat dua aturan ini:

1. Kita harus mencoba mengambil (_acquire_) _lock_-nya sebelum memakai datanya.
2. Pas kita kelar sama data yang dijaga sama mutex tersebut, kita harus 
   membuka kunci (_unlock_) datanya supaya _threads_ lain bisa mengambil _lock_ 
   itu.

Sebagai metafora dunia nyata buat sebuah mutex, bayangin sebuah panel diskusi 
di sebuah konferensi yang cuma punya satu mikrofon. Sebelum seorang panelis 
bisa ngomong, dia harus minta atau ngasih sinyal kalau dia mau memakai 
mikrofonnya. Saat dia dapat mikrofonnya, dia bisa ngomong selama yang dia mau 
lalu memberikan mikrofonnya ke panelis berikutnya yang meminta buat ngomong. 
Kalau ada panelis yang lupa ngasih mikrofonnya ke orang lain pas udah selesai, 
tidak bakal ada orang lain yang bisa ngomong. Kalau pengelolaan mikrofon bersama 
ini jadi berantakan, panelnya tidak bakal berjalan sesuai rencana!

Pengelolaan mutex bisa jadi susah sekali buat dibikin benar, itulah kenapa banyak 
orang jadi antusias sekali sama _channels_. Namun, berkat sistem tipe dan 
aturan _ownership_ di Rust, kita tidak mungkin salah mengunci dan membuka kunci.

#### API dari `Mutex<T>`

Sebagai contoh gimana cara memakai mutex, mari mulai dengan memakai sebuah 
mutex di dalam konteks satu _thread_ (single-threaded), seperti yang ditunjukkan 
di Listing 16-12.

<Listing number="16-12" file-name="src/main.rs" caption="Mengeksplorasi API dari `Mutex<T>` di dalam konteks *single-threaded* buat kesederhanaan">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Sama kayak banyak tipe lainnya, kita membikin sebuah `Mutex<T>` dengan memakai 
fungsi *associated* `new`. Buat mengakses data di dalam mutex-nya, kita memakai 
method `lock` buat ngambil *lock*-nya. Pemanggilan ini bakal memblokir (_block_) 
_thread_ yang saat ini lagi jalan sehingga ia tidak bisa melakukan kerjaan 
apa-apa sampai giliran kita dapat *lock*-nya.

Pemanggilan ke `lock` bakal gagal kalau _thread_ lain yang sedang memegang 
*lock* tersebut mengalami *panic*. Di kasus itu, tidak bakal ada yang bisa 
ngedapetin *lock*-nya lagi, jadi kita memilih buat `unwrap` dan membikin 
_thread_ ini *panic* kalau kita ada di situasi tersebut.

Setelah kita dapat *lock*-nya, kita bisa memperlakukan nilai kembaliannya, 
yang kita namakan `num` di sini, sebagai referensi _mutable_ ke data internalnya. 
Sistem tipe memastikan kalau kita dapat *lock*-nya dulu sebelum memakai nilai di 
dalam `m`. Tipe dari `m` adalah `Mutex<i32>`, bukannya `i32`, jadi kita _harus_ 
memanggil `lock` supaya bisa memakai nilai `i32` tersebut. Kita tidak bisa lupa; 
sistem tipenya tidak bakal ngebiarin kita mengakses `i32` internal itu kalau kita 
tidak melakukannya.

Pemanggilan ke `lock` mengembalikan sebuah tipe bernama `MutexGuard`, yang 
dibungkus di dalam sebuah `LockResult` yang tadi kita tangani dengan panggilan ke 
`unwrap`. Tipe `MutexGuard` mengimplementasikan `Deref` agar dia menunjuk ke data 
internal kita; tipe ini juga punya implementasi `Drop` yang melepaskan *lock*-nya 
secara otomatis saat sebuah `MutexGuard` keluar dari _scope_, yang mana terjadi di 
akhir dari *inner scope* (scope dalam). Sebagai hasilnya, kita tidak bakal ambil 
risiko lupa melepaskan *lock*-nya dan ngeblokir mutex tersebut dari dipakai sama 
_threads_ lain, karena pelepasan *lock* itu terjadi secara otomatis.

Setelah *lock*-nya di-_drop_, kita bisa mencetak nilai mutex tersebut dan melihat 
kalau kita berhasil ngubah `i32` internalnya jadi `6`.

#### Berbagi `Mutex<T>` di Antara Beberapa Threads

Sekarang mari kita coba membagikan sebuah nilai di antara beberapa _threads_ 
memakai `Mutex<T>`. Kita bakal membikin (_spin up_) 10 _threads_ dan menyuruh 
masing-masing dari mereka buat menaikkan nilai penghitung (counter) sebanyak 1, 
sehingga penghitungnya berjalan dari 0 sampai 10. Contoh di Listing 16-13 
bakal mengalami error _compiler_, dan kita bakal memakai error itu buat belajar 
lebih banyak soal memakai `Mutex<T>` dan gimana Rust ngebantu kita memakainya 
dengan benar.

<Listing number="16-13" file-name="src/main.rs" caption="Sepuluh _threads_, di mana masing-masing menaikkan penghitung yang dijaga sama sebuah `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Kita bikin sebuah variabel `counter` buat memegang sebuah `i32` di dalam 
sebuah `Mutex<T>`, kayak yang kita lakuin di Listing 16-12. Selanjutnya, kita 
bikin 10 _threads_ dengan iterasi melewati serangkaian angka. Kita memakai 
`thread::spawn` dan ngasih _closure_ yang sama ke semua _threads_ itu: sebuah 
_closure_ yang memindahkan (moves) penghitung tersebut ke dalam _thread_, 
mengambil *lock* pada `Mutex<T>` dengan memanggil method `lock`, lalu 
menambahkan 1 ke nilai yang ada di dalam mutex tersebut. Pas sebuah _thread_ 
kelar ngejalanin _closure_-nya, `num` bakal keluar dari _scope_ dan melepaskan 
*lock*-nya supaya _thread_ lain bisa mengambil *lock* itu.

Di _main thread_, kita mengumpulkan (collect) semua *join handles*. Terus, 
sama kayak di Listing 16-2, kita memanggil `join` pada setiap *handle* buat 
memastikan semua _threads_ selesai. Di titik itu, _main thread_ bakal ngambil 
*lock*-nya dan mencetak hasil dari program ini.

Kita tadi udah ngebocorin kalau contoh ini tidak bakal bisa di-compile. Sekarang 
mari kita cari tahu alasannya kenapa!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Pesan error-nya bilang kalau nilai `counter` itu udah dipindahkan (moved) di 
iterasi _loop_ sebelumnya. Rust ngasih tahu kita kalau kita tidak bisa 
memindahkan kepemilikan *lock* `counter` ke dalam beberapa _threads_. Mari 
kita perbaiki error _compiler_ ini dengan metode kepemilikan ganda (multiple-
ownership method) yang udah kita bahas di Bab 15.

#### Multiple Ownership dengan Multiple Threads

Di Bab 15, kita ngasih sebuah nilai ke beberapa pemilik (multiple owners) 
dengan memakai _smart pointer_ `Rc<T>` buat membikin nilai yang jumlah 
referensinya dilacak (*reference counted value*). Mari kita lakuin hal yang 
sama di sini dan lihat apa yang terjadi. Kita bakal ngebungkus `Mutex<T>` ke 
dalam `Rc<T>` di Listing 16-14 dan meng-_clone_ `Rc<T>`-nya sebelum 
memindahkan kepemilikannya ke dalam _thread_.

<Listing number="16-14" file-name="src/main.rs" caption="Mencoba memakai `Rc<T>` buat mengizinkan beberapa _threads_ buat memiliki `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Sekali lagi, kita compile dan... dapat error yang beda! _Compiler_-nya ngajarin 
kita banyak hal.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, pesan error-nya panjang sekali ya! Ini bagian penting yang perlu jadi 
fokus: `` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. _Compiler_-nya 
juga ngasih tahu kita apa alasannya: `` the trait `Send` is not implemented for
`Rc<Mutex<i32>>` ``. Kita bakal ngomongin soal `Send` di bagian selanjutnya: dia 
adalah salah satu trait yang memastikan tipe-tipe yang kita pakai bersama _threads_ 
emang ditujukan buat dipakai di situasi yang konkuren.

Sayangnya, `Rc<T>` tidak aman buat dibagikan antar _threads_. Saat `Rc<T>` 
mengelola _reference count_, dia nambahin jumlahnya buat setiap panggilan ke 
`clone` dan ngurangin jumlahnya saat setiap *clone* di-_drop_. Tapi dia tidak 
memakai *concurrency primitives* apa pun buat memastikan kalau perubahan pada 
jumlah itu tidak diinterupsi sama _thread_ lain. Hal ini bisa menyebabkan 
perhitungan (counts) yang salah—*bugs* halus (subtle bugs) yang pada akhirnya 
bisa menyebabkan _memory leaks_ (kebocoran memori) atau sebuah nilai di-_drop_ 
sebelum kita selesai memakainya. Apa yang kita butuhkan adalah sebuah tipe 
yang persis kayak `Rc<T>`, tapi yang ngubah jumlah referensinya pakai cara yang 
_thread-safe_ (aman di lingkungan utas ganda).

#### Atomic Reference Counting dengan `Arc<T>`

Untungnya, `Arc<T>` *adalah* tipe yang mirip `Rc<T>` yang aman buat dipakai di 
situasi yang konkuren. Huruf _A_-nya singkatan dari _atomic_, yang artinya 
dia adalah tipe *atomically reference-counted* (penghitungan referensi secara 
atomik). *Atomics* adalah jenis primitif konkurensi tambahan yang tidak bakal 
kita bahas secara mendetail di sini: cek dokumentasi _standard library_ buat 
[`std::sync::atomic`][atomic] buat detail lebih lanjut. Di titik ini, kita 
cuma perlu tahu kalau *atomics* bekerja kayak tipe primitif biasa tapi aman 
buat dibagikan antar _threads_.

Terus kita mungkin penasaran kenapa tidak semua tipe primitif dibikin jadi 
*atomic* dan kenapa tipe-tipe _standard library_ tidak diimplementasikan buat 
memakai `Arc<T>` secara default. Alasannya adalah keamanan _thread_ (thread 
safety) itu datang dengan penalti performa (performance penalty) yang mana 
kita cuma mau membayarnya saat kita bener-bener butuh. Kalau kita cuma ngelakuin 
operasi pada nilai di dalam satu _thread_, kode kita bisa berjalan lebih kencang 
kalau dia tidak perlu memaksakan jaminan yang disediakan sama *atomics*.

Mari kembali ke contoh kita: `Arc<T>` dan `Rc<T>` punya API yang sama, jadi 
kita memperbaiki program kita dengan mengubah baris `use`, panggilan ke `new`, 
dan panggilan ke `clone`. Kode di Listing 16-15 akhirnya bakal bisa di-compile 
dan jalan.

<Listing number="16-15" file-name="src/main.rs" caption="Memakai sebuah `Arc<T>` buat ngebungkus `Mutex<T>` supaya bisa membagikan kepemilikan di beberapa _threads_">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Kode ini bakal mencetak output berikut:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Kita berhasil! Kita menghitung dari 0 sampai 10, yang mungkin tidak kelihatan 
terlalu mengesankan, tapi itu banyak ngajarin kita soal `Mutex<T>` dan keamanan 
_thread_. Kita juga bisa memakai struktur program ini buat melakukan operasi 
yang lebih rumit ketimbang cuma menaikkan nilai penghitung. Memakai strategi ini, 
kita bisa membagi perhitungan jadi bagian-bagian independen, memecah bagian-
bagian itu ke berbagai _threads_, lalu memakai `Mutex<T>` agar masing-masing 
_thread_ bisa meng-update hasil akhirnya dengan bagian perhitungan mereka.

Perhatikan bahwa kalau kita ngelakuin operasi angka (numerical operations) yang 
simpel, ada tipe yang lebih sederhana ketimbang tipe `Mutex<T>` yang disediakan 
sama [modul `std::sync::atomic` di _standard library_][atomic]. Tipe-tipe ini 
menyediakan akses yang aman, konkuren, dan atomik ke tipe-tipe primitif. Kita 
memilih buat memakai `Mutex<T>` bersama sebuah tipe primitif buat contoh ini 
supaya kita bisa berkonsentrasi pada gimana `Mutex<T>` itu bekerja.

### Kesamaan Antara `RefCell<T>`/`Rc<T>` dan `Mutex<T>`/`Arc<T>`

kita mungkin nyadar kalau `counter` itu _immutable_ tapi kita bisa dapat sebuah 
referensi _mutable_ ke nilai di dalamnya; ini artinya `Mutex<T>` menyediakan 
_interior mutability_ (mutabilitas interior), sama kayak keluarga `Cell`. Dengan 
cara yang sama seperti kita memakai `RefCell<T>` di Bab 15 buat memungkinkan kita 
memutasi isi di dalam sebuah `Rc<T>`, kita memakai `Mutex<T>` buat memutasi isi 
di dalam sebuah `Arc<T>`.

Detail lain yang perlu diperhatikan adalah Rust tidak bisa melindungi kita dari 
semua jenis *logic errors* (kutu logika) pas kita memakai `Mutex<T>`. Ingat 
kembali dari Bab 15 kalau memakai `Rc<T>` punya risiko membikin _reference cycles_ 
(siklus referensi), di mana dua nilai `Rc<T>` merujuk ke satu sama lain, yang 
menyebabkan _memory leaks_. Serupa dengan hal itu, `Mutex<T>` punya risiko 
membikin _deadlocks_ (jalan buntu). Hal ini terjadi pas suatu operasi butuh 
mengambil dua *locks* dan dua _threads_ masing-masing udah ngambil salah satu dari 
*locks* tersebut, yang menyebabkan mereka saling menunggu satu sama lain 
selamanya. Kalau kita tertarik sama _deadlocks_, coba bikin program Rust yang 
punya sebuah _deadlock_; terus cari tahu soal strategi mitigasi _deadlock_ buat 
mutexes di bahasa pemrograman apa pun lalu cobalah mengimplementasikan strategi 
itu di Rust. Dokumentasi API _standard library_ buat `Mutex<T>` dan `MutexGuard` 
nawarin informasi yang berguna.

Kita bakal menuntaskan bab ini dengan ngomongin soal trait `Send` dan `Sync` 
dan gimana cara kita bisa memakainya bersama _custom types_ (tipe kustom).

[atomic]: ../std/sync/atomic/index.html
