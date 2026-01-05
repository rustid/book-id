## Refactoring untuk Meningkatkan Modularitas dan Penanganan Error

Untuk meningkatkan program kita, kita akan memperbaiki empat masalah yang
berkaitan dengan struktur program dan cara program menangani kemungkinan error.
Pertama, fungsi `main` kita sekarang melakukan dua tugas: mem-parsing argumen
dan membaca file. Seiring program berkembang, jumlah tugas terpisah yang
ditangani `main` akan bertambah. Semakin banyak tanggung jawab sebuah fungsi,
semakin sulit untuk dipahami, diuji, dan diubah tanpa merusak bagian lainnya.
Akan lebih baik jika kita memisahkan fungsionalitas sehingga setiap fungsi hanya
bertanggung jawab pada satu tugas.

Masalah ini juga berkaitan dengan masalah kedua: `query` dan `file_path` adalah
variabel konfigurasi untuk program kita, sedangkan variabel seperti `contents`
digunakan untuk menjalankan logika program. Semakin panjang fungsi `main`,
semakin banyak variabel yang harus dimasukkan ke dalam scope; semakin banyak
variabel di dalam scope, semakin sulit melacak tujuan masing-masing. Lebih baik
mengelompokkan variabel konfigurasi ke dalam satu struktur agar tujuannya lebih
jelas.

Masalah ketiga adalah kita menggunakan `expect` untuk mencetak pesan error
ketika pembacaan file gagal, tetapi pesan error-nya hanya menampilkan
`Should have been able to read the file`. Membaca file bisa gagal karena
berbagai alasan: misalnya file tidak ada, atau kita tidak punya izin untuk
membukanya. Saat ini, apa pun penyebabnya, kita tetap mencetak pesan error yang
sama, yang tidak memberikan informasi berarti bagi pengguna!

Keempat, kita menggunakan `expect` untuk menangani error, dan jika pengguna
menjalankan program tanpa memberikan argumen yang cukup, mereka akan mendapatkan
error `index out of bounds` dari Rust yang tidak menjelaskan masalah sebenarnya.
Akan lebih baik jika semua kode penanganan error berada di satu tempat sehingga
pengembang di masa depan hanya perlu melihat satu bagian jika logika penanganan
error perlu diubah. Mengumpulkan seluruh penanganan error di satu tempat juga
memastikan bahwa kita mencetak pesan yang benar-benar berguna bagi pengguna
akhir.

Mari kita tangani keempat masalah ini dengan melakukan refactoring pada project
kita.

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### Memisahkan Tanggung Jawab dalam Proyek Biner

Masalah organisasi berupa pembagian banyak tugas ke dalam fungsi `main` adalah
hal yang umum pada banyak proyek biner. Karena itu, banyak programmer Rust
merasa berguna untuk memisahkan berbagai tanggung jawab program biner ketika
fungsi `main` mulai membesar. Proses ini memiliki langkah-langkah berikut:

- Pisahkan program Anda menjadi file _main.rs_ dan _lib.rs_, lalu pindahkan
  logika program ke _lib.rs_.
- Selama logika parsing command line masih kecil, ia bisa tetap berada di dalam
  fungsi `main`.
- Ketika logika parsing command line mulai menjadi rumit, ekstrak logika
  tersebut dari fungsi `main` ke fungsi atau tipe lain.

Tanggung jawab yang tersisa dalam fungsi `main` setelah proses ini sebaiknya
terbatas pada hal-hal berikut:

- Memanggil logika parsing command line dengan nilai argumen
- Menyiapkan konfigurasi lain yang diperlukan
- Memanggil fungsi `run` di _lib.rs_
- Menangani error jika `run` mengembalikan error

Pola ini bertujuan untuk memisahkan tanggung jawab: _main.rs_ menangani eksekusi
program, dan _lib.rs_ menangani seluruh logika tugas utama. Karena Anda tidak
bisa menguji fungsi `main` secara langsung, struktur ini memungkinkan Anda
menguji seluruh logika program dengan memindahkannya keluar dari fungsi `main`.
Kode yang tersisa di dalam `main` akan cukup kecil sehingga bisa diverifikasi
kebenarannya hanya dengan membaca. Mari kita perbarui program kita dengan
mengikuti proses ini.

#### Mengekstrak Argument Parser

Kita akan mengekstrak fungsionalitas untuk parsing argumen ke dalam sebuah
fungsi yang akan dipanggil oleh `main`. Listing 12-5 memperlihatkan awal baru
dari fungsi `main` yang memanggil fungsi baru `parse_config`, yang akan kita
definisikan di _src/main.rs_.

<Listing number="12-5" file-name="src/main.rs" caption=" Mengekstrak fungsi `parse_config` dari `main` ">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Kita masih mengumpulkan argumen baris perintah ke dalam sebuah vektor, tetapi
alih-alih langsung mengambil nilai argumen pada indeks 1 dan menyimpannya ke
variabel `query` serta nilai pada indeks 2 ke variabel `file_path` di dalam
fungsi `main`, sekarang kita meneruskan seluruh vektor tersebut ke fungsi
`parse_config`. Fungsi `parse_config` kemudian memuat logika yang menentukan
argumen mana yang masuk ke variabel mana dan mengembalikan nilai-nilainya
kembali ke `main`. Kita masih membuat variabel `query` dan `file_path` di
`main`, tetapi `main` tidak lagi bertanggung jawab untuk memetakan argumen baris
perintah ke variabel yang sesuai.

Perubahan ini mungkin terlihat berlebihan untuk program kecil seperti ini,
tetapi kita sedang melakukan refaktor secara bertahap dan terukur. Setelah
membuat perubahan tersebut, jalankan kembali programnya untuk memastikan parsing
argumen masih bekerja dengan benar. Penting untuk sering memeriksa progres,
supaya jika ada masalah, kita bisa lebih mudah mengetahui penyebabnya.

#### Mengelompokkan Nilai Konfigurasi

Kita bisa mengambil satu langkah kecil lagi untuk memperbaiki fungsi
`parse_config`. Saat ini kita mengembalikan sebuah tuple, namun kemudian
langsung memecah tuple tersebut menjadi bagian-bagian terpisah kembali. Ini
merupakan tanda bahwa mungkin abstraksi kita belum tepat.

Indikator lain adalah kata `config` pada `parse_config`, yang memberi kesan
bahwa kedua nilai yang kita kembalikan itu saling berhubungan dan merupakan
bagian dari satu nilai konfigurasi. Saat ini kita belum menyampaikan makna
tersebut dalam struktur datanya, selain hanya dengan mengelompokkannya dalam
tuple. Jadi, kita akan memasukkan kedua nilai itu ke dalam sebuah struct dan
memberikan nama field yang bermakna. Dengan begitu, akan lebih mudah bagi
pengembang lain di masa depan untuk memahami bagaimana nilai-nilai tersebut
saling terkait dan apa fungsi masing-masing.

Listing 12-6 menunjukkan perbaikan pada fungsi `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption=" Refactoring `parse_config` untuk mengembalikan sebuah instance dari struct `Config` ">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Kita telah menambahkan sebuah struct bernama `Config` yang didefinisikan
memiliki field bernama `query` dan `file_path`. Tanda tangan fungsi
`parse_config` sekarang menunjukkan bahwa fungsi ini mengembalikan sebuah nilai
`Config`. Di dalam tubuh fungsi `parse_config`, di mana sebelumnya kita
mengembalikan potongan string (string slices) yang mereferensikan nilai `String`
di `args`, sekarang kita mendefinisikan `Config` untuk memiliki nilai `String`
yang memiliki kepemilikan sendiri (owned).

Variabel `args` di `main` adalah pemilik dari nilai argumen tersebut dan hanya
meminjamkan nilainya ke fungsi `parse_config`, yang berarti kita akan melanggar
aturan peminjaman Rust jika `Config` mencoba mengambil alih kepemilikan
nilai-nilai di `args`.

Ada beberapa cara untuk menangani data `String`; cara yang paling mudah,
walaupun agak tidak efisien, adalah memanggil metode `clone` pada nilai-nilai
tersebut. Ini akan membuat salinan penuh dari data tersebut agar instance
`Config` dapat memilikinya, yang membutuhkan lebih banyak waktu dan memori
dibandingkan hanya menyimpan referensi ke data string. Namun, melakukan clone
juga membuat kode kita jauh lebih sederhana karena kita tidak perlu mengatur
lifetime dari referensi; dalam kasus ini, mengorbankan sedikit performa demi
kesederhanaan adalah pertukaran yang sepadan.

> ### Pertukaran Saat Menggunakan `clone`
>
> Ada kecenderungan di kalangan Rustacean untuk menghindari penggunaan `clone`
> untuk memperbaiki masalah kepemilikan karena biaya runtime-nya. Di
> [Bab 13][ch13], kamu akan mempelajari cara yang lebih efisien dalam situasi
> seperti ini. Namun untuk sekarang, tidak apa-apa menyalin beberapa string agar
> tetap bisa lanjut, karena kamu hanya akan membuat salinan ini sekali dan path
> file serta query string-mu sangat kecil. Lebih baik memiliki program yang
> bekerja meskipun sedikit tidak efisien dibandingkan mencoba terlalu
> mengoptimasi kode pada percobaan pertama. Seiring bertambahnya pengalamanmu
> dengan Rust, akan lebih mudah memulai dengan solusi paling efisien, tetapi
> untuk saat ini, menggunakan `clone` adalah hal yang sepenuhnya dapat diterima.

Kita telah memperbarui `main` sehingga instance `Config` yang dikembalikan oleh
`parse_config` disimpan ke dalam variabel bernama `config`, dan kita memperbarui
kode yang sebelumnya menggunakan variabel terpisah `query` dan `file_path` agar
sekarang menggunakan field pada struct `Config`.

Sekarang kode kita lebih jelas menunjukkan bahwa `query` dan `file_path` saling
berhubungan dan bahwa tujuan mereka adalah untuk mengonfigurasi bagaimana
program akan berjalan. Kode apa pun yang menggunakan nilai-nilai ini tahu bahwa
mereka dapat ditemukan di instance `config` pada field yang dinamai sesuai
fungsinya.

#### Membuat Constructor untuk `Config`

Sejauh ini, kita telah memisahkan logika yang bertanggung jawab untuk
mem-parsing argumen command line dari `main` dan memindahkannya ke fungsi
`parse_config`. Hal ini membantu kita melihat bahwa nilai `query` dan
`file_path` saling berhubungan, dan hubungan tersebut sebaiknya terlihat jelas
dalam kode kita. Kita kemudian menambahkan struct `Config` untuk memberi nama
tujuan yang saling terkait dari `query` dan `file_path`, serta agar kita bisa
mengembalikan nilai-nilai tersebut sebagai nama field struct dari fungsi
`parse_config`.

Jadi sekarang, karena tujuan fungsi `parse_config` adalah membuat sebuah
instance `Config`, kita dapat mengubah `parse_config` dari fungsi biasa menjadi
fungsi bernama `new` yang terkait dengan struct `Config`. Melakukan perubahan
ini akan membuat kode lebih idiomatis. Kita bisa membuat instance dari tipe-tipe
di standard library, seperti `String`, dengan memanggil `String::new`. Dengan
cara yang sama, dengan mengubah `parse_config` menjadi fungsi `new` yang terkait
dengan `Config`, kita akan dapat membuat instance `Config` dengan memanggil
`Config::new`. Listing 12-7 menunjukkan perubahan yang perlu kita lakukan.

<Listing number="12-7" file-name="src/main.rs" caption="Changing `parse_config` into `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Kita sudah memperbarui `main` yang sebelumnya memanggil `parse_config`, agar
sekarang memanggil `Config::new`. Kita juga mengganti nama `parse_config`
menjadi `new` dan memindahkannya ke dalam blok `impl`, yang mengasosiasikan
fungsi `new` dengan `Config`. Cobalah kompilasi kode ini lagi untuk memastikan
semuanya berfungsi.

### Memperbaiki Penanganan Error

Sekarang kita akan memperbaiki penanganan error. Ingat bahwa mencoba mengakses
nilai dalam vektor `args` pada indeks 1 atau indeks 2 akan menyebabkan program
panic jika vektor tersebut berisi kurang dari tiga item. Cobalah menjalankan
program tanpa argumen apa pun; hasilnya akan tampak seperti ini:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Baris `index out of bounds: the len is 1 but the index is 1` adalah pesan error
yang ditujukan untuk programmer. Pesan ini tidak akan membantu pengguna akhir
memahami apa yang seharusnya mereka lakukan. Mari kita perbaiki sekarang.

#### Meningkatkan Pesan Error

Pada Listing 12-8, kita menambahkan pengecekan di fungsi `new` untuk memastikan
slice cukup panjang sebelum mengakses indeks 1 dan indeks 2. Jika slice tidak
cukup panjang, program akan panic dan menampilkan pesan error yang lebih jelas.

<Listing number="12-8" file-name="src/main.rs" caption="Adding a check for the number of arguments">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Kode ini mirip dengan fungsi `Guess::new` yang kita tulis pada Listing 9-13, di
mana kita memanggil `panic!` ketika argumen `value` berada di luar rentang nilai
yang valid. Alih-alih memeriksa rentang nilai di sini, kita memeriksa apakah
panjang `args` setidaknya `3`, sehingga sisa fungsi bisa berasumsi bahwa kondisi
tersebut telah terpenuhi. Jika `args` memiliki kurang dari tiga item, kondisi
ini akan bernilai `true`, dan kita memanggil makro `panic!` untuk segera
menghentikan program.

Dengan beberapa baris tambahan di fungsi `new` ini, mari kita jalankan ulang
program tanpa argumen apa pun untuk melihat seperti apa error-nya sekarang.

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Keluaran ini sudah lebih baik: sekarang kita punya pesan error yang masuk akal.
Namun, masih ada informasi tambahan yang tidak perlu kita berikan ke pengguna.
Mungkin teknik yang kita gunakan pada Listing 9-13 bukan yang terbaik untuk
kasus ini: pemanggilan `panic!` lebih cocok untuk masalah pemrograman daripada
masalah penggunaan program, seperti dibahas di Bab 9. Sebagai gantinya, kita
akan menggunakan teknik lain yang sudah kamu pelajari di Bab 9—mengembalikan
`Result` yang menunjukkan apakah operasi berhasil atau terjadi error.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Mengembalikan `Result` Alih-alih Memanggil `panic!`

Sebagai gantinya, kita bisa mengembalikan nilai `Result` yang akan berisi
instance `Config` jika berhasil, dan akan menjelaskan masalahnya jika terjadi
error. Kita juga akan mengganti nama fungsi dari `new` menjadi `build` karena
banyak programmer mengharapkan fungsi `new` **tidak pernah gagal**. Saat
`Config::build` berkomunikasi dengan `main`, kita bisa menggunakan tipe `Result`
untuk memberi sinyal bahwa ada masalah. Lalu, kita bisa mengubah `main` agar
mengonversi varian `Err` menjadi pesan error yang lebih ramah bagi pengguna,
tanpa teks tambahan seperti `thread 'main'` dan `RUST_BACKTRACE` yang muncul
ketika kita memanggil `panic!`.

Listing 12-9 menunjukkan perubahan yang perlu kita lakukan pada nilai kembalian
fungsi yang sekarang kita sebut `Config::build`, serta perubahan pada isi
fungsinya agar bisa mengembalikan `Result`. Perlu dicatat bahwa kode ini belum
bisa dikompilasi sampai kita memperbarui `main`, yang akan kita lakukan pada
listing berikutnya.

<Listing number="12-9" file-name="src/main.rs" caption=" Mengembalikan `Result` dari `Config::build` ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Fungsi `build` kita sekarang mengembalikan `Result` dengan `Config` pada kasus
sukses dan literal string pada kasus error. Nilai error kita selalu berupa
string literal yang punya lifetime `'static`.

Kita membuat dua perubahan di dalam fungsi: Alih-alih memanggil `panic!` saat
user tidak memberikan argumen yang cukup, sekarang kita mengembalikan `Err`, dan
kita membungkus nilai `Config` dalam `Ok`. Perubahan ini membuat fungsi sesuai
dengan tipe barunya.

Dengan mengembalikan `Err` dari `Config::build`, fungsi `main` bisa menangani
nilai `Result` tersebut dan menghentikan program dengan lebih rapi saat terjadi
error.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Memanggil `Config::build` dan Menangani Error

Untuk menangani kasus error dan menampilkan pesan yang ramah bagi pengguna, kita
perlu memperbarui `main` agar menangani `Result` yang dikembalikan oleh
`Config::build`, seperti yang ditunjukkan pada Listing 12-10. Kita juga akan
mengambil alih tanggung jawab keluar dari program dengan kode error non-nol dari
`panic!` dan menerapkannya secara manual. Status keluar non-nol adalah konvensi
untuk memberi sinyal ke proses yang memanggil program bahwa program berhenti
dalam keadaan error.

<Listing number="12-10" file-name="src/main.rs" caption=" Keluar dengan kode error jika pembuatan `Config` gagal ">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Dalam listing ini, kita menggunakan sebuah metode yang belum kita bahas secara
rinci: `unwrap_or_else`, yang didefinisikan pada `Result<T, E>` di pustaka
standar. Menggunakan `unwrap_or_else` memungkinkan kita mendefinisikan
penanganan error kustom tanpa `panic!`. Jika `Result` bernilai `Ok`, perilaku
metode ini mirip dengan `unwrap`: ia mengembalikan nilai di dalam `Ok`. Namun,
jika bernilai `Err`, metode ini akan memanggil kode di dalam _closure_, yaitu
fungsi anonim yang kita definisikan dan berikan sebagai argumen ke
`unwrap_or_else`.

Kita akan membahas _closure_ lebih dalam di [Bab 13], tetapi untuk saat ini
cukup pahami bahwa `unwrap_or_else` akan mengoper nilai di dalam `Err`—dalam
kasus ini string statis `"not enough arguments"` yang kita tambahkan pada
Listing 12-9—ke _closure_ melalui parameter `err` (yang berada di antara tanda
garis vertikal `|err|`). Kode di dalam _closure_ kemudian dapat menggunakan
nilai `err` tersebut saat dijalankan.

Kita juga menambahkan baris `use` baru untuk membawa `process` dari pustaka
standar ke dalam scope. Kode dalam _closure_ yang dijalankan ketika terjadi
error hanya terdiri dari dua baris: kita mencetak nilai `err`, lalu memanggil
`process::exit`. Fungsi `process::exit` akan langsung menghentikan program dan
mengembalikan angka yang diberikan sebagai _exit status code_. Ini mirip dengan
penanganan berbasis `panic!` pada Listing 12-8, tetapi sekarang kita tidak lagi
mendapatkan keluaran tambahan yang berlebihan. Mari kita coba.

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Keren! Keluaran ini jauh lebih ramah bagi para pengguna kita.

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### Mengekstrak Logika dari `main`

Sekarang setelah kita selesai melakukan refactor pada parsing konfigurasi, mari
kita beralih ke logika program. Seperti yang sudah kita bahas di bagian
[“Memisahkan Tanggung Jawab pada Proyek Binary”](#separation-of-concerns-for-binary-projects)<!-- ignore -->,
kita akan mengekstrak sebuah fungsi bernama `run` yang akan menampung semua
logika yang saat ini ada di fungsi `main` dan tidak berhubungan dengan
pengaturan konfigurasi atau penanganan error. Setelah selesai, fungsi `main`
akan menjadi ringkas dan mudah dicek hanya dengan melihatnya, dan kita juga bisa
menuliskan test untuk seluruh logika lainnya.

Listing 12-11 menunjukkan peningkatan kecil dan bertahap dengan mengekstrak
fungsi `run`.

<Listing number="12-11" file-name="src/main.rs" caption=" Mengekstrak fungsi `run` yang berisi sisa logika program ">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Fungsi `run` sekarang berisi semua logika yang tersisa dari `main`, dimulai dari
proses membaca file. Fungsi `run` menerima instance `Config` sebagai argumennya.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

### Mengembalikan Error dari `run`

Dengan sisa logika program sudah dipisahkan ke dalam fungsi `run`, sekarang kita
bisa memperbaiki penanganan error seperti yang kita lakukan pada `Config::build`
di Listing 12-9. Alih-alih membiarkan program _panic_ dengan memanggil `expect`,
fungsi `run` akan mengembalikan `Result<T, E>` ketika terjadi masalah. Dengan
begitu, kita bisa memusatkan logika penanganan error di `main` agar lebih ramah
bagi pengguna.

Listing 12-12 menunjukkan perubahan yang perlu kita lakukan pada **signature**
dan isi fungsi `run`.

<Listing number="12-12" file-name="src/main.rs" caption=" Mengubah fungsi `run` agar mengembalikan `Result` ">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Kita telah membuat tiga perubahan penting di sini. Pertama, kita mengubah tipe
nilai balik fungsi `run` menjadi `Result<(), Box<dyn Error>>`. Sebelumnya fungsi
ini mengembalikan tipe unit `()`, dan kita tetap mempertahankannya sebagai nilai
yang dikembalikan pada kondisi `Ok`.

Untuk tipe error-nya, kita menggunakan trait object `Box<dyn Error>` (dan kita
memasukkan `std::error::Error` ke dalam scope dengan `use`). Kita akan membahas
trait object di **Bab 18** nanti. Untuk sekarang, cukup pahami bahwa
`Box<dyn Error>` berarti fungsi ini akan mengembalikan tipe apa pun yang
mengimplementasikan trait `Error`, tanpa harus menentukan secara spesifik tipe
error-nya. Ini memberi fleksibilitas untuk mengembalikan berbagai jenis error
pada situasi yang berbeda. Kata kunci `dyn` sendiri merupakan singkatan dari
_dynamic_.

Kedua, kita menghapus pemanggilan `expect` dan menggantinya dengan operator `?`,
seperti yang telah dibahas di **Bab 9**. Alih-alih memanggil `panic!` saat
terjadi error, `?` akan mengembalikan nilai error dari fungsi saat ini untuk
ditangani oleh pemanggilnya.

Ketiga, fungsi `run` sekarang mengembalikan nilai `Ok` pada kondisi sukses. Kita
telah mendeklarasikan tipe sukses `run` sebagai `()` di tanda tangan fungsi,
jadi kita perlu membungkus nilai unit tersebut dalam `Ok`. Sintaks `Ok(())`
mungkin terlihat agak aneh pada awalnya, tetapi penggunaan `()` seperti ini
adalah cara idiomatis Rust untuk menunjukkan bahwa kita memanggil `run` hanya
untuk efek sampingnya; fungsi ini tidak mengembalikan nilai yang perlu kita
gunakan.

Saat Anda menjalankan kode ini, program akan berhasil dikompilasi, tetapi akan
muncul sebuah peringatan:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust memberi tahu kita bahwa kode kita mengabaikan nilai `Result`, padahal nilai
`Result` tersebut mungkin menunjukkan bahwa terjadi error. Namun kita tidak
memeriksa apakah terjadi error atau tidak, dan compiler mengingatkan kita bahwa
kemungkinan besar kita memang perlu menambahkan kode penanganan error di sini!
Sekarang mari kita perbaiki masalah itu.

#### Menangani Error yang Dikembalikan dari `run` di `main`

Kita akan memeriksa error dan menanganinya menggunakan teknik yang mirip dengan
yang kita gunakan pada `Config::build` di Listing 12-10, tetapi dengan sedikit
perbedaan:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Kita menggunakan `if let` alih-alih `unwrap_or_else` untuk memeriksa apakah
`run` mengembalikan nilai `Err` dan memanggil `process::exit(1)` jika iya.
Fungsi `run` tidak mengembalikan nilai yang perlu kita `unwrap` seperti
`Config::build` yang mengembalikan instance `Config`. Karena `run` hanya
mengembalikan `()` pada kasus sukses, kita hanya peduli mendeteksi error saja,
jadi kita tidak membutuhkan `unwrap_or_else` untuk mengembalikan nilai yang
di-unwrap, yang sebenarnya hanya `()`.

Bagian isi dari `if let` dan `unwrap_or_else` pada kedua kasus tersebut sama:
kita mencetak pesan error dan keluar dari program.

### Memisahkan Kode ke dalam Library Crate

Proyek `minigrep` kita sejauh ini sudah terlihat bagus! Sekarang kita akan
membagi file _src/main.rs_ dan memindahkan sebagian kode ke dalam _src/lib.rs_.
Dengan begitu, kita bisa menguji kode tersebut dan membuat _src/main.rs_
memiliki tanggung jawab yang lebih sedikit.

Mari kita pindahkan kode yang bertanggung jawab untuk melakukan pencarian teks
ke dalam _src/lib.rs_, bukan di _src/main.rs_, sehingga kita (atau siapa pun
yang menggunakan library `minigrep` kita) bisa memanggil fungsi pencarian
tersebut dari konteks lain, tidak hanya dari binary `minigrep`.

Pertama, mari kita definisikan signature fungsi `search` di _src/lib.rs_ seperti
pada Listing 12-13, dengan body yang memanggil macro `unimplemented!`. Kita akan
menjelaskan signature-nya lebih detail setelah kita mengisi implementasinya.

<Listing number="12-13" file-name="src/lib.rs" caption=" Mendefinisikan fungsi `search` di *src/lib.rs* ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

Kita menggunakan keyword `pub` pada definisi fungsi untuk menjadikan `search`
bagian dari **public API** library crate kita. Sekarang kita punya library crate
yang bisa dipakai oleh binary crate kita, dan juga bisa kita uji!

Selanjutnya, kita perlu membawa kode yang didefinisikan di _src/lib.rs_ ke dalam
scope binary crate di _src/main.rs_ dan memanggilnya, seperti yang ditunjukkan
pada Listing 12-14.

<Listing number="12-14" file-name="src/main.rs" caption=" Menggunakan fungsi `search` dari library crate `minigrep` di *src/main.rs* ">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Kita menambahkan baris `use minigrep::search` untuk membawa fungsi `search` dari
library crate ke dalam cakupan (scope) binary crate. Lalu, di fungsi `run`,
alih-alih mencetak seluruh isi file, kita memanggil fungsi `search` dan mengoper
nilai `config.query` serta `contents` sebagai argumen. Setelah itu, `run` akan
menggunakan loop `for` untuk mencetak setiap baris hasil yang dikembalikan
`search` yang cocok dengan query. Ini juga saat yang tepat untuk menghapus
pemanggilan `println!` di fungsi `main` yang sebelumnya menampilkan query dan
path file, sehingga sekarang program hanya mencetak hasil pencarian (jika tidak
ada error).

Perhatikan bahwa fungsi `search` akan mengumpulkan semua hasil ke dalam sebuah
vector dan mengembalikannya sebelum proses pencetakan dilakukan. Implementasi
ini bisa terasa lambat saat mencari pada file yang sangat besar karena hasil
tidak langsung dicetak ketika ditemukan; kita akan membahas kemungkinan solusi
dengan _iterators_ di Chapter 13.

Whew! Cukup banyak kerja sejauh ini, tapi sekarang kita sudah menyiapkan
struktur program yang jauh lebih baik untuk pengembangan ke depannya. Kini
penanganan error menjadi lebih mudah dan kode jauh lebih modular. Mulai
sekarang, hampir semua pekerjaan akan dilakukan di _src/lib.rs_.

Sekarang mari kita manfaatkan modularitas baru ini untuk melakukan sesuatu yang
sebelumnya akan sulit dilakukan, tapi sekarang jadi mudah: kita akan menulis
beberapa _test_!

[ch13]: ch13-00-functional-features.html
