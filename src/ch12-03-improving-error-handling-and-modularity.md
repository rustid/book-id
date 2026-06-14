## Refactoring untuk Meningkatkan Modularitas dan Penanganan Error

Untuk meningkatkan program kita, kita bakal memperbaiki empat masalah yang berkaitan dengan struktur program dan gimana program menangani potensi error. Pertama, fungsi `main` kita sekarang melakukan dua tugas: mengurai (parsing) argumen dan membaca file. Seiring berkembangnya program kita, jumlah tugas terpisah yang ditangani oleh fungsi `main` juga bakal meningkat. Saat sebuah fungsi mendapat lebih banyak tanggung jawab, fungsi itu jadi lebih sulit buat dipahami, lebih susah buat diuji, dan lebih susah buat diubah tanpa merusak salah satu bagiannya. Hal terbaik adalah memisahkan fungsionalitas sehingga tiap fungsi bertanggung jawab atas satu tugas saja.

Isu ini juga terikat ke masalah kedua: walaupun `query` dan `file_path` adalah variabel konfigurasi untuk program kita, variabel seperti `contents` dipakai buat menjalankan logika programnya. Semakin panjang `main`, semakin banyak variabel yang harus kita bawa ke dalam *scope*; semakin banyak variabel yang ada di *scope*, semakin susah untuk mengingat tujuan dari masing-masing variabel tersebut. Praktik terbaiknya adalah mengelompokkan variabel-variabel konfigurasi ke dalam satu struktur buat memperjelas tujuan mereka.

Masalah ketiga adalah kita memakai `expect` buat mencetak pesan error saat membaca file gagal, tapi pesan error-nya cuma mencetak `Should have been able to read the file`. Membaca file bisa gagal karena berbagai alasan: contohnya, file tersebut bisa saja tidak ada, atau kita mungkin tidak punya izin buat membukanya. Saat ini, apa pun situasinya, kita bakal mencetak pesan error yang sama persis buat semuanya, yang mana tidak ngasih informasi apa pun ke *user*!

Keempat, kita memakai `expect` buat menangani error secara berulang kali, dan kalau *user* menjalankan program kita tanpa memberikan argumen yang cukup, mereka bakal dapat error `index out of bounds` dari Rust yang tidak menjelaskan masalahnya dengan jelas. Bakal lebih baik kalau semua kode penanganan error (error-handling code) ada di satu tempat sehingga para *maintainer* di masa depan cuma punya satu tempat buat dicek kalau logika penanganan error-nya perlu diubah. Mengumpulkan semua kode penanganan error di satu tempat juga bakal memastikan kalau kita mencetak pesan yang bermakna bagi *end users* (pengguna akhir) kita.

Mari kita atasi empat masalah ini dengan me-*refactor* project kita.

### Separation of Concerns (Pemisahan Kepentingan) untuk Binary Projects

Masalah organisasi dari mengalokasikan tanggung jawab untuk beberapa tugas ke dalam fungsi `main` itu umum terjadi di banyak *binary projects*. Sebagai hasilnya, banyak programmer Rust merasa berguna untuk memisahkan berbagai kepentingan dari sebuah program *binary* saat fungsi `main` mulai menjadi besar. Proses ini punya langkah-langkah berikut:

- Pisahkan program kita jadi file _main.rs_ dan file _lib.rs_, lalu pindahkan logika program kita ke _lib.rs_.
- Selama logika penguraian (parsing) *command line* masih kecil, ia bisa tetap berada di dalam fungsi `main`.
- Ketika logika penguraian *command line* mulai menjadi rumit, ekstrak logika itu dari fungsi `main` ke dalam fungsi atau tipe lain.

Tanggung jawab yang tersisa di fungsi `main` setelah proses ini seharusnya dibatasi hanya pada hal-hal berikut:

- Memanggil logika penguraian *command line* beserta nilai-nilai argumennya
- Menyiapkan konfigurasi apa pun lainnya
- Memanggil fungsi `run` yang ada di _lib.rs_
- Menangani error kalau fungsi `run` mengembalikan error

Pola ini adalah soal pemisahan kepentingan (*separating concerns*): _main.rs_ menangani jalannya program dan _lib.rs_ menangani semua logika dari tugas yang ada. Karena kita tidak bisa menguji fungsi `main` secara langsung, struktur ini memungkinkan kita untuk menguji semua logika program dengan memindahkannya ke luar dari fungsi `main`. Kode yang tersisa di fungsi `main` bakal cukup kecil untuk bisa diverifikasi kebenarannya hanya dengan membacanya. Mari kita rombak program kita dengan mengikuti proses ini.

#### Mengekstrak Parser Argumen

Kita bakal mengekstrak fungsionalitas untuk mengurai argumen ke dalam fungsi yang bakal dipanggil oleh `main`. Listing 12-5 menunjukkan permulaan baru dari fungsi `main` yang memanggil fungsi baru `parse_config`, yang bakal kita definisikan di _src/main.rs_.

<Listing number="12-5" file-name="src/main.rs" caption="Mengekstrak fungsi `parse_config` dari `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Kita masih mengumpulkan argumen *command line* ke dalam sebuah vector, tapi alih-alih me-*assign* nilai argumen di indeks 1 ke variabel `query` dan nilai argumen di indeks 2 ke variabel `file_path` di dalam fungsi `main`, kita memberikan keseluruhan vector-nya ke fungsi `parse_config`. Fungsi `parse_config` ini kemudian menampung logika yang menentukan argumen mana yang masuk ke variabel mana dan mengembalikan nilai-nilainya kembali ke `main`. Kita tetap membuat variabel `query` dan `file_path` di `main`, tapi `main` tidak lagi punya tanggung jawab buat menentukan gimana korelasi antara argumen *command line* dan variabel-variabel tersebut.

Perombakan ini mungkin kelihatan agak berlebihan buat program kita yang masih kecil, tapi kita melakukan *refactoring* dalam langkah-langkah kecil yang bertahap. Setelah membuat perubahan ini, jalankan programnya lagi buat memverifikasi kalau penguraian argumennya masih berfungsi. Mengecek progres kita secara rutin itu hal yang baik, untuk membantu mengidentifikasi penyebab masalah ketika masalah itu muncul.

#### Mengelompokkan Nilai-nilai Konfigurasi

Kita bisa mengambil langkah kecil lainnya buat meningkatkan fungsi `parse_config` lebih jauh lagi. Saat ini, kita mengembalikan sebuah tuple, tapi kemudian kita langsung memecah tuple itu menjadi bagian-bagian individual lagi. Ini adalah tanda kalau kita mungkin belum punya abstraksi yang tepat.

Indikator lain yang menunjukkan ada ruang buat peningkatan adalah bagian `config` dari nama `parse_config`, yang menyiratkan kalau dua nilai yang kita kembalikan itu saling berhubungan dan keduanya adalah bagian dari satu nilai konfigurasi. Saat ini kita belum menyampaikan makna tersebut di dalam struktur datanya selain dengan mengelompokkan kedua nilai itu ke dalam tuple; kita bakal lebih baik menaruh kedua nilai itu ke dalam satu *struct* dan memberikan nama yang bermakna buat tiap field dari *struct* tersebut. Melakukan hal ini bakal mempermudah para *maintainer* kode ini di masa depan buat paham gimana berbagai nilai tersebut saling berhubungan dan apa tujuannya.

Listing 12-6 menunjukkan peningkatan buat fungsi `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption="Me-*refactor* `parse_config` agar mengembalikan sebuah instance dari _struct_ `Config`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Kita sudah menambahkan sebuah *struct* bernama `Config` yang didefinisikan punya field bernama `query` dan `file_path`. *Signature* dari `parse_config` sekarang menunjukkan kalau dia mengembalikan nilai `Config`. Di dalam body `parse_config`, di mana kita tadinya mengembalikan _string slices_ yang merujuk pada nilai `String` di `args`, kita sekarang mendefinisikan `Config` agar menampung nilai `String` yang *owned* (dimiliki). Variabel `args` di `main` adalah pemilik dari nilai-nilai argumen tersebut dan dia hanya membiarkan fungsi `parse_config` meminjamnya (borrow), yang berarti kita bakal melanggar aturan _borrowing_ Rust kalau `Config` mencoba mengambil _ownership_ (kepemilikan) dari nilai-nilai yang ada di `args`.

Ada banyak cara yang bisa kita pakai buat mengelola data `String` ini; yang paling gampang, meskipun agak kurang efisien, adalah dengan memanggil method `clone` pada nilai-nilai tersebut. Ini bakal membuat salinan penuh dari datanya supaya instance `Config` tersebut bisa memilikinya, yang mana makan lebih banyak waktu dan memori dibandingkan cuma menyimpan referensi ke data string-nya. Tapi, meng-*clone* datanya juga bikin kode kita jadi sangat simpel karena kita tidak perlu pusing mengelola _lifetimes_ dari referensi-referensinya; di situasi ini, mengorbankan sedikit performa buat mendapatkan kesederhanaan adalah sebuah *trade-off* (pertukaran) yang sepadan.

> ### Pertukaran (Trade-Offs) dari Pemakaian `clone`
>
> Ada kecenderungan di antara banyak programmer Rust buat menghindari pemakaian `clone` demi memperbaiki masalah _ownership_ karena biaya _runtime_-nya. Di [Bab 13][ch13], kita bakal belajar gimana cara memakai *methods* yang lebih efisien di tipe situasi seperti ini. Tapi buat sekarang, tidak masalah menyalin beberapa string demi bisa terus maju karena kita cuma bakal membuat salinan ini sekali saja dan _path_ file serta string pencarian kita itu sangat kecil. Jauh lebih baik punya program yang bekerja walau sedikit tidak efisien daripada mencoba melakukan *hyperoptimize* (optimasi berlebihan) di kode pada percobaan pertama kita. Seiring kita jadi makin berpengalaman dengan Rust, bakal lebih gampang buat mulai dari solusi yang paling efisien, tapi buat sekarang, memanggil `clone` itu masih sangat bisa diterima.

Kita sudah meng-update `main` agar dia menaruh instance dari `Config` yang dikembalikan oleh `parse_config` ke dalam variabel bernama `config`, dan kita juga meng-update kode yang sebelumnya memakai variabel `query` dan `file_path` secara terpisah sehingga kini ia memakai field yang ada di struct `Config`.

Sekarang kode kita dengan lebih jelas menyampaikan kalau `query` dan `file_path` itu berhubungan dan tujuannya adalah buat mengkonfigurasi gimana program kita bakal berjalan. Kode apa pun yang memakai nilai-nilai ini tahu untuk mencari mereka di instance `config` di dalam field yang dinamai sesuai tujuannya.

#### Membuat Constructor buat `Config`

Sejauh ini, kita sudah mengekstrak logika yang bertanggung jawab buat mengurai argumen *command line* dari `main` dan menaruhnya di fungsi `parse_config`. Melakukan hal ini membantu kita melihat kalau nilai `query` dan `file_path` itu berhubungan, dan hubungan itu harus disampaikan di kode kita. Kita kemudian menambahkan struct `Config` untuk menamai tujuan terkait dari `query` dan `file_path` serta untuk bisa mengembalikan nama-nama dari nilai tersebut sebagai nama field struct dari fungsi `parse_config`.

Jadi sekarang karena tujuan dari fungsi `parse_config` adalah untuk membuat sebuah instance `Config`, kita bisa mengubah `parse_config` dari fungsi biasa menjadi sebuah fungsi bernama `new` yang dikaitkan (associated) dengan struct `Config`. Membuat perubahan ini bakal bikin kodenya jadi lebih idiomatik. Kita bisa membuat instance dari berbagai tipe di *standard library*, seperti `String`, dengan memanggil `String::new`. Mirip dengan itu, dengan mengubah `parse_config` menjadi sebuah fungsi `new` yang dikaitkan dengan `Config`, kita bakal bisa membuat instance dari `Config` dengan memanggil `Config::new`. Listing 12-7 menunjukkan perubahan yang perlu kita buat.

<Listing number="12-7" file-name="src/main.rs" caption="Mengubah `parse_config` menjadi `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Kita sudah meng-update `main` di tempat kita memanggil `parse_config` untuk memanggil `Config::new` sebagai gantinya. Kita sudah mengubah nama `parse_config` jadi `new` dan memindahkannya ke dalam blok `impl`, yang mengaitkan fungsi `new` ini dengan `Config`. Coba _compile_ kode ini lagi buat memastikan kalau ini berfungsi.

### Memperbaiki Penanganan Error

Sekarang kita bakal bekerja buat memperbaiki penanganan error (error handling) kita. Ingat kembali kalau mencoba mengakses nilai di dalam vector `args` di indeks 1 atau indeks 2 bakal bikin program mengalami _panic_ kalau vector-nya berisi kurang dari tiga item. Coba jalankan programnya tanpa argumen apa pun; outputnya bakal kayak gini:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Baris `index out of bounds: the len is 1 but the index is 1` adalah pesan error yang ditujukan buat para programmer. Ini tidak bakal ngebantu *end users* (pengguna akhir) kita buat paham apa yang seharusnya mereka lakuin. Mari kita perbaiki itu sekarang.

#### Memperbaiki Pesan Error

Di Listing 12-8, kita menambahkan pengecekan di fungsi `new` yang bakal memverifikasi apakah panjang _slice_-nya cukup sebelum mengakses indeks 1 dan indeks 2. Kalau _slice_-nya tidak cukup panjang, programnya bakal mengalami _panic_ dan menampilkan pesan error yang lebih bagus.

<Listing number="12-8" file-name="src/main.rs" caption="Menambahkan pengecekan jumlah argumen">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Kode ini mirip sama [fungsi `Guess::new` yang kita tulis di Listing 9-13][ch9-custom-types], di mana kita memanggil `panic!` saat argumen `value` berada di luar rentang nilai yang valid. Alih-alih mengecek sebuah rentang nilai di sini, kita mengecek apakah panjang dari `args` minimal adalah `3` dan sisa dari fungsi ini bisa beroperasi di bawah asumsi kalau kondisi ini sudah terpenuhi. Kalau `args` punya kurang dari tiga item, kondisi ini bakal bernilai `true`, dan kita memanggil macro `panic!` buat mengakhiri program seketika.

Dengan sedikit baris tambahan ini di `new`, mari kita jalankan programnya tanpa argumen lagi buat melihat seperti apa pesan error-nya sekarang:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Output ini lebih mendingan: kita sekarang punya pesan error yang masuk akal. Namun, kita juga punya informasi ekstra yang tidak mau kita kasih ke *user* kita. Mungkin teknik yang kita pakai di Listing 9-13 bukanlah teknik yang terbaik buat dipakai di sini: sebuah pemanggilan `panic!` lebih cocok buat mengatasi masalah pemrograman dibanding masalah pemakaian, [seperti yang dibahas di Bab 9][ch9-error-guidelines]. Sebagai gantinya, kita bakal memakai teknik lain yang sudah kita pelajari di Bab 9—[mengembalikan sebuah `Result`][ch9-result] yang menandakan sukses atau error.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Mengembalikan `Result` Alih-Alih Memanggil `panic!`

Kita bisa mengembalikan sebuah nilai `Result` yang bakal mengandung sebuah instance `Config` kalau sukses dan bakal mendeskripsikan masalahnya kalau terjadi error. Kita juga bakal mengubah nama fungsinya dari `new` jadi `build` karena banyak programmer berharap kalau fungsi bernama `new` itu tidak bakal pernah gagal. Saat `Config::build` sedang berkomunikasi dengan `main`, kita bisa memakai tipe `Result` buat ngasih tahu kalau ada masalah. Kemudian kita bisa mengubah `main` agar dia mengubah varian `Err` jadi error yang lebih praktis buat *user* kita, tanpa teks-teks tambahan soal `thread 'main'` dan `RUST_BACKTRACE` yang disebabkan oleh pemanggilan `panic!`.

Listing 12-9 menunjukkan perubahan yang perlu kita buat pada nilai kembalian dari fungsi yang sekarang kita namakan `Config::build` ini beserta isi (body) fungsinya yang dibutuhkan buat mengembalikan `Result`. Perhatikan bahwa kode ini tidak bakal bisa di-compile sampai kita meng-update `main` juga, yang mana bakal kita lakukan di listing berikutnya.

<Listing number="12-9" file-name="src/main.rs" caption="Mengembalikan sebuah `Result` dari `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Fungsi `build` kita mengembalikan sebuah `Result` dengan instance `Config` di kasus yang sukses dan string literal di kasus yang gagal. Nilai error kita bakal selalu berupa string literal yang punya _lifetime_ `'static`.

Kita sudah membuat dua perubahan di body fungsi tersebut: alih-alih memanggil `panic!` saat *user* tidak memberikan argumen yang cukup, kita sekarang mengembalikan sebuah nilai `Err`, dan kita sudah membungkus nilai kembalian `Config` di dalam sebuah `Ok`. Perubahan ini membuat fungsinya sesuai dengan *signature* tipe barunya.

Mengembalikan nilai `Err` dari `Config::build` memungkinkan fungsi `main` untuk menangani nilai `Result` yang dikembalikan dari fungsi `build` tersebut dan keluar dari proses dengan lebih rapi saat terjadi error.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Memanggil `Config::build` dan Menangani Error

Buat menangani kasus yang gagal (error) dan mencetak pesan yang *user-friendly*, kita perlu meng-update `main` untuk menangani `Result` yang dikembalikan oleh `Config::build`, seperti yang ditunjukkan di Listing 12-10. Kita juga bakal mengambil tanggung jawab untuk keluar dari alat *command line* dengan kode error bukan nol (*nonzero error code*) menjauh dari `panic!` dan malah mengimplementasikannya secara manual. Status keluar (*exit status*) bukan nol adalah sebuah konvensi untuk memberi tanda kepada proses yang memanggil program kita bahwa program kita telah keluar dengan *state* error.

<Listing number="12-10" file-name="src/main.rs" caption="Keluar dengan sebuah kode error kalau proses _build_ `Config` gagal">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Di listing ini, kita sudah memakai method yang belum pernah kita bahas secara mendetail: `unwrap_or_else`, yang didefinisikan pada `Result<T, E>` oleh *standard library*. Pemakaian `unwrap_or_else` memungkinkan kita buat mendefinisikan penanganan error kustom yang tidak menggunakan `panic!`. Kalau `Result`-nya adalah nilai `Ok`, method ini bakal berperilaku mirip seperti `unwrap`: ia mengembalikan nilai di dalam yang sedang dibungkus oleh `Ok`. Namun, kalau nilainya adalah sebuah nilai `Err`, method ini memanggil kode yang ada di dalam sebuah *closure*, yakni sebuah fungsi anonim yang kita definisikan dan kita kirimkan sebagai argumen ke `unwrap_or_else`. Kita bakal membahas *closures* lebih dalam di [Bab 13][ch13]. Buat sekarang, kita cuma perlu tahu kalau `unwrap_or_else` bakal meneruskan nilai yang ada di dalam `Err`, yang mana di kasus ini adalah string statis `"not enough arguments"` yang kita tambahkan di Listing 12-9, ke *closure* kita di dalam argumen `err` yang muncul di antara tanda garis vertikal (`|`). Kode di dalam *closure* tersebut kemudian bisa memakai nilai `err` itu saat dia berjalan.

Kita sudah menambahkan satu baris `use` baru buat membawa `process` dari *standard library* ke dalam *scope*. Kode di dalam *closure* yang bakal berjalan di kasus error ini cuma terdiri dari dua baris: kita mencetak nilai `err` lalu memanggil `process::exit`. Fungsi `process::exit` bakal menghentikan programnya secara instan dan mengembalikan angka yang diteruskan sebagai kode *exit status*. Hal ini mirip seperti penanganan berbasis `panic!` yang kita pakai di Listing 12-8, tapi kita tidak lagi dapat semua output ekstra. Mari kita coba:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Bagus! Output ini jauh lebih *friendly* (ramah) buat para *user* kita.

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-main"></a>

### Mengekstrak Logika dari Fungsi `main`

Sekarang setelah kita selesai me-*refactor* penguraian (parsing) konfigurasi, mari kita beralih ke logika programnya. Seperti yang kita nyatakan di [“Separation of Concerns (Pemisahan Kepentingan) untuk Binary Projects”](#separation-of-concerns-pemisahan-kepentingan-untuk-binary-projects), kita bakal mengekstrak sebuah fungsi bernama `run` yang bakal menampung semua logika yang saat ini ada di fungsi `main` yang tidak berkaitan dengan menyiapkan konfigurasi atau menangani error. Setelah kita selesai, fungsi `main` bakal ringkas dan gampang diverifikasi hanya dengan melihatnya, dan kita bakal bisa menulis pengujian untuk semua logika lainnya.

Listing 12-11 menunjukkan peningkatan kecil dan bertahap berupa mengekstrak sebuah fungsi `run`.

<Listing number="12-11" file-name="src/main.rs" caption="Mengekstrak fungsi `run` yang mengandung sisa dari logika program">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Fungsi `run` sekarang mengandung semua sisa logika dari `main`, mulai dari bagian membaca file. Fungsi `run` menerima instance `Config` sebagai argumennya.

#### Mengembalikan Error dari Fungsi `run`

Dengan sisa logika program yang sudah dipisahkan ke fungsi `run`, kita bisa meningkatkan penanganan error-nya, sama seperti yang kita lakukan dengan `Config::build` di Listing 12-9. Bukannya ngebiarin program kita mengalami _panic_ dengan memanggil `expect`, fungsi `run` bakal mengembalikan `Result<T, E>` pas terjadi sesuatu yang salah. Hal ini bakal membiarkan kita buat mengonsolidasi (menyatukan) logika seputar penanganan error ke dalam `main` lebih jauh lagi dengan cara yang ramah buat _user_. Listing 12-12 menunjukkan perubahan yang perlu kita buat pada *signature* dan body dari `run`.

<Listing number="12-12" file-name="src/main.rs" caption="Mengubah fungsi `run` agar mengembalikan `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Kita sudah membuat tiga perubahan yang signifikan di sini. Pertama, kita mengubah tipe kembalian (return type) dari fungsi `run` jadi `Result<(), Box<dyn Error>>`. Fungsi ini sebelumnya mengembalikan tipe unit, `()`, dan kita mempertahankan itu sebagai nilai yang dikembalikan pada kasus yang sukses (`Ok`).

Buat tipe error-nya, kita memakai _trait object_ `Box<dyn Error>` (dan kita juga sudah membawa `std::error::Error` ke dalam *scope* dengan *statement* `use` di paling atas). Kita bakal membahas _trait objects_ di [Bab 18][ch18]. Buat sekarang, cukup ketahui kalau `Box<dyn Error>` berarti fungsinya bakal mengembalikan sebuah tipe yang mengimplementasikan trait `Error`, tapi kita tidak usah menentukan dengan spesifik tipe apa nilai kembaliannya. Ini ngasih kita fleksibilitas buat mengembalikan nilai-nilai error yang mungkin bertipe beda-beda di kasus error yang berbeda-beda. Keyword `dyn` itu singkatan dari *dynamic* (dinamis).

Kedua, kita telah menghapus pemanggilan `expect` dan menggantinya dengan operator `?`, seperti yang kita bicarakan di [Bab 9][ch9-question-mark]. Bukannya melakukan `panic!` pas ada error, `?` bakal mengembalikan nilai error dari fungsi saat ini agar si pemanggil fungsi yang menangani error-nya.

Ketiga, fungsi `run` kini mengembalikan nilai `Ok` pada kasus yang sukses. Kita sudah mendeklarasikan kalau tipe sukses dari fungsi `run` adalah `()` pada *signature*-nya, yang artinya kita perlu membungkus nilai tipe unit tersebut di dalam sebuah nilai `Ok`. Sintaks `Ok(())` ini mungkin kelihatannya agak aneh pas awal-awal, tapi memakai `()` kayak gini adalah cara yang idiomatik buat menunjukkan kalau kita memanggil `run` murni hanya karena *side effects* (efek samping)-nya aja; ia tidak mengembalikan nilai yang kita perlukan.

Saat kita menjalankan kode ini, kode ini bakal berhasil di-compile tapi bakal menampilkan sebuah *warning* (peringatan):

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust ngasih tahu kita kalau kode kita mengabaikan nilai `Result`, padahal nilai `Result` itu mungkin menunjukkan kalau terjadi sebuah error. Tapi kita tidak mengecek apakah memang terjadi error atau tidak, dan *compiler* mengingatkan kita kalau kita mungkin lupa menaruh kode penanganan error di sini! Mari kita bereskan masalah itu sekarang.

#### Menangani Error yang Dikembalikan oleh `run` di `main`

Kita bakal mengecek error dan menangani mereka memakai teknik yang mirip dengan yang kita pakai bersama `Config::build` di Listing 12-10, tapi dengan sedikit perbedaan:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Kita memakai `if let` bukannya `unwrap_or_else` buat mengecek apakah `run` mengembalikan nilai `Err` dan memanggil `process::exit(1)` jika iya. Fungsi `run` tidak mengembalikan sebuah nilai yang mau kita `unwrap` seperti `Config::build` yang mengembalikan instance `Config`. Karena `run` mengembalikan `()` pada kasus yang sukses, kita cuma peduli pada mendeteksi terjadinya error, jadi kita tidak perlu `unwrap_or_else` untuk mengembalikan nilai yang tidak terbungkus, yang mana hasilnya cuma bakal `()`.

Body dari `if let` dan fungsi di `unwrap_or_else` itu sama persis di kedua kasus: kita mencetak error-nya lalu kita keluar dari program.

### Memisahkan Kode ke dalam sebuah Library Crate

Project `minigrep` kita sudah kelihatan bagus sejauh ini! Sekarang kita bakal memisahkan file _src/main.rs_ dan menaruh sebagian kode ke dalam file _src/lib.rs_. Dengan begitu, kita bisa menguji kodenya dan bisa punya file _src/main.rs_ dengan lebih sedikit tanggung jawab.

Mari kita definisikan kode yang bertanggung jawab untuk pencarian teks di dalam _src/lib.rs_ ketimbang di _src/main.rs_, yang mana hal ini bakal membiarkan kita (atau siapa pun yang memakai *library* `minigrep` kita) untuk memanggil fungsi pencarian dari lebih banyak konteks dibanding hanya lewat *binary* `minigrep` kita.

Pertama, mari definisikan *signature* fungsi `search` di _src/lib.rs_ seperti yang ditunjukkan di Listing 12-13, dengan isi (body) fungsi yang memanggil macro `unimplemented!`. Kita bakal menjelaskan *signature*-nya lebih detail pas kita mengisi implementasinya.

<Listing number="12-13" file-name="src/lib.rs" caption="Mendefinisikan fungsi `search` di *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

Kita sudah memakai keyword `pub` di definisi fungsinya untuk menunjuk `search` sebagai bagian dari API *public* dari *library crate* kita. Sekarang kita sudah punya sebuah *library crate* yang bisa kita pakai dari *binary crate* kita dan yang bisa kita tes!

Sekarang kita perlu membawa kode yang didefinisikan di _src/lib.rs_ ke dalam *scope* dari *binary crate* di _src/main.rs_ lalu memanggilnya, seperti yang ditunjukkan di Listing 12-14.

<Listing number="12-14" file-name="src/main.rs" caption="Memakai fungsi `search` dari *library crate* `minigrep` di dalam *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Kita menambahkan baris `use minigrep::search` untuk membawa fungsi `search` dari *library crate* ke dalam *scope* dari *binary crate*. Lalu, di dalam fungsi `run`, alih-alih mencetak konten dari file tersebut, kita memanggil fungsi `search` dan meneruskan nilai `config.query` dan `contents` sebagai argumennya. Setelah itu, `run` bakal memakai *for loop* buat mencetak setiap baris yang dikembalikan oleh `search` yang cocok dengan kueri (query) pencariannya. Ini juga waktu yang tepat buat menghapus pemanggilan `println!` di dalam fungsi `main` yang tadi menampilkan kueri dan *path* file sehingga program kita cuma mencetak hasil pencariannya aja (kalau tidak ada error yang terjadi).

Perhatikan bahwa fungsi pencarian bakal mengumpulkan semua hasil ke dalam sebuah vector yang dikembalikannya sebelum terjadi pencetakan ke layar. Implementasi ini bisa jadi agak lambat untuk menampilkan hasil ketika mencari di file yang berukuran sangat besar karena hasilnya tidak langsung dicetak saat ditemukan; kita bakal mendiskusikan kemungkinan untuk memperbaiki ini dengan memakai _iterators_ di Bab 13.

Huft! Kerjaan yang lumayan banyak ya, tapi kita udah nyiapin diri buat kesuksesan di masa depan. Sekarang jauh lebih gampang buat menangani error, dan kita sudah membuat kodenya jadi lebih modular. Mulai dari sini, hampir semua pekerjaan kita bakal dilakukan di _src/lib.rs_.

Mari kita manfaatkan modularitas yang baru kita dapatkan ini dengan melakukan sesuatu yang bakal susah sekali dilakuin sama kode kita yang lama tapi sangat gampang dengan kode yang baru: kita bakal nulis beberapa *tests* (pengujian)!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
