## Futures dan Sintaks Async

Elemen kunci dari pemrograman asinkron di Rust adalah _futures_ dan keyword 
`async` dan `await` milik Rust.

Sebuah _future_ adalah nilai yang mungkin belum siap sekarang tapi bakal jadi 
siap di suatu waktu di masa depan. (Konsep yang sama ini juga muncul di banyak 
bahasa lain, kadang-kadang pakai nama lain kayak _task_ atau _promise_.) Rust 
menyediakan trait `Future` sebagai blok penyusun supaya berbagai operasi asinkron 
bisa diimplementasikan pakai struktur data yang berbeda-beda tapi dengan 
antarmuka (interface) yang sama. Di Rust, _futures_ adalah tipe-tipe yang 
mengimplementasikan trait `Future`. Tiap _future_ menyimpan informasinya sendiri 
soal sejauh mana kemajuan yang sudah dibuat dan apa makna dari "siap" ("ready").

Kita bisa menerapkan keyword `async` ke blok dan fungsi buat menentukan kalau 
mereka bisa diinterupsi dan dilanjutkan lagi. Di dalam sebuah blok asinkron atau 
fungsi asinkron, kita bisa memakai keyword `await` buat _menunggu sebuah future_ 
(yaitu, nunggu dia sampai jadi siap). Titik mana pun di mana kita me-_await_ 
sebuah _future_ di dalam sebuah blok atau fungsi asinkron adalah titik potensial 
buat blok atau fungsi asinkron itu buat berhenti sejenak (pause) dan dilanjutkan 
lagi (resume). Proses pengecekan ke sebuah _future_ buat melihat apakah nilainya 
sudah tersedia atau belum ini disebut _polling_.

Beberapa bahasa pemrograman lain, kayak C# dan JavaScript, juga memakai keyword 
`async` dan `await` buat pemrograman asinkron. Kalau kita sudah familier sama 
bahasa-bahasa itu, kita mungkin bakal sadar ada beberapa perbedaan signifikan 
dari cara Rust melakukan hal tersebut, termasuk cara nanganin sintaksnya. Itu ada 
alasan bagusnya lho, kayak yang bakal kita lihat nanti!

Pas lagi nulis Rust asinkron, kita bakal memakai keyword `async` dan `await` 
sebagian besar waktunya. Rust mengompilasi mereka jadi kode yang ekuivalen 
menggunakan trait `Future`, mirip kayak gimana dia mengompilasi `for` _loops_ jadi 
kode yang ekuivalen memakai trait `Iterator`. Tapi karena Rust menyediakan trait 
`Future`, kita juga bisa mengimplementasikannya buat tipe data kita sendiri 
kalau kita butuh. Banyak fungsi yang bakal kita lihat di sepanjang bab ini 
mengembalikan tipe yang punya implementasi `Future`-nya masing-masing. Kita bakal 
balik lagi ke definisi trait-nya di akhir bab ini dan gali lebih dalam soal 
gimana cara kerjanya, tapi detail segini sudah cukup buat kita lanjut jalan dulu.

Ini semua mungkin terasa agak abstrak, jadi mari kita tulis program asinkron 
pertama kita: sebuah _web scraper_ mini. Kita bakal memasukkan dua URL dari 
_command line_, mengambil (_fetch_) kedua URL itu secara konkuren, terus 
mengembalikan hasil dari URL mana pun yang selesai duluan. Contoh ini bakal 
punya lumayan banyak sintaks baru, tapi jangan khawatir—kita bakal jelasin 
semua yang perlu kita tahu seiring kita berjalan.

## Program Asinkron Pertama Kita

Biar fokus bab ini tetap pada belajar asinkron ketimbang sibuk ngerjain bagian-
bagian ekosistem, kita sudah ngebikin _crate_ `trpl` (`trpl` itu singkatan dari 
"The Rust Programming Language"). _Crate_ ini mengekspor ulang (re-exports) semua 
tipe, trait, dan fungsi yang bakal kita butuhkan, utamanya dari _crate_ 
[`futures`][futures-crate]<!-- ignore --> dan [`tokio`][tokio]<!-- ignore -->. 
_Crate_ `futures` adalah tempat resmi buat bereksperimen dengan kode asinkron di 
Rust, dan sebenarnya di sanalah trait `Future` asal-muasalnya didesain. Tokio 
adalah _async runtime_ yang paling banyak dipakai di Rust saat ini, apalagi buat 
aplikasi web. Ada banyak _runtimes_ keren lainnya di luar sana, dan mereka 
mungkin lebih cocok buat kebutuhan kita. Kita memakai _crate_ `tokio` di balik 
layarnya `trpl` karena dia sudah teruji dengan baik dan banyak dipakai.

Di beberapa kasus, `trpl` juga mengganti nama atau membungkus (wraps) API aslinya 
biar kita tetap fokus sama detail-detail yang relevan buat bab ini. Kalau kita 
mau paham apa yang dilakukan sama _crate_ ini, kita menyarankan kita buat cek 
[_source code_-nya][crate-source]. Kita bakal bisa lihat dari _crate_ mana asal 
dari tiap fitur yang di-_re-export_, dan kita sudah ninggalin komentar yang 
ekstensif yang menjelaskan apa aja yang dilakukan sama _crate_ tersebut.

Bikin sebuah project *binary* baru bernama `hello-async` dan tambahkan _crate_ 
`trpl` sebagai dependensi:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Sekarang kita bisa pakai berbagai potongan yang disediakan sama `trpl` buat 
nulis program asinkron pertama kita. Kita bakal bikin alat _command line_ mini 
yang mengambil dua halaman web, menarik elemen `<title>` dari masing-masing 
halaman, terus mencetak judul dari halaman mana pun yang menyelesaikan seluruh 
proses tersebut paling duluan.

### Mendefinisikan Fungsi page_title

Mari mulai dengan nulis fungsi yang menerima satu URL halaman sebagai 
parameternya, melakukan _request_ ke sana, dan mengembalikan teks dari elemen 
judulnya (lihat Listing 17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Mendefinisikan fungsi asinkron buat dapat elemen judul dari halaman HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Pertama, kita mendefinisikan sebuah fungsi bernama `page_title` dan menandainya 
pakai keyword `async`. Terus kita pakai fungsi `trpl::get` buat mengambil 
URL apa pun yang dimasukkan dan menambahkan keyword `await` buat menunggu 
responsnya. Buat dapat teks dari respons tersebut, kita memanggil method `text`-
nya, dan sekali lagi menunggunya dengan keyword `await`. Kedua langkah ini 
sifatnya asinkron. Buat fungsi `get`, kita harus nunggu server buat mengirim 
balik bagian pertama dari responsnya, yang mana bakal berisi HTTP _headers_, 
_cookies_, dan lain-lain, dan bisa saja dikirim secara terpisah dari _body_ 
(isi utama) responsnya. Apalagi kalau _body_-nya itu besar sekali, itu bisa 
memakan waktu yang lumayan lama buat semuanya sampai. Karena kita harus nunggu 
buat _keseluruhan_ responsnya sampai, method `text` itu juga asinkron.

Kita harus secara eksplisit menunggu kedua _futures_ ini, karena _futures_ di 
Rust itu _lazy_ (malas): mereka tidak bakal melakukan apa-apa sampai kita 
menyuruh mereka dengan keyword `await`. (Faktanya, Rust bakal mengeluarkan 
peringatan _compiler_ kalau kita tidak menunggu sebuah _future_.) Ini mungkin 
mengingatkan kita soal pembahasan iterator di Bab 13 di bagian [“Memproses 
Serangkaian Item dengan Iterator”][iterators-lazy]<!-- ignore -->. Iterator 
tidak bakal melakukan apa-apa kecuali kalau kita memanggil method `next`-nya—baik 
itu secara langsung atau dengan memakai `for` _loops_ atau method-method kayak 
`map` yang memakai `next` di balik layarnya. Demikian juga, _futures_ tidak 
bakal melakukan apa-apa kecuali kita menyuruh mereka secara eksplisit. Sifat 
malas ini membolehkan Rust menghindari menjalankan kode asinkron sampai dia 
bener-bener dibutuhkan.

> Catatan: Ini berbeda dari perilaku yang kita lihat di Bab 16 saat memakai 
> `thread::spawn` di [“Membikin Thread Baru dengan spawn”][thread-spawn]<!-- 
> ignore -->, di mana _closure_ yang kita kasih ke _thread_ lain itu langsung 
> mulai berjalan. Ini juga beda dari cara banyak bahasa lain melakukan 
> pendekatan ke asinkron. Tapi penting buat Rust buat bisa ngasih jaminan 
> performanya, sama halnya dengan iterator.

Begitu kita dapet `response_text`, kita bisa menguraikan (_parse_) nilainya jadi 
sebuah instance dari tipe `Html` menggunakan `Html::parse`. Daripada pakai 
string mentah, sekarang kita punya sebuah tipe data yang bisa kita pakai buat 
mengolah HTML tersebut sebagai struktur data yang lebih kaya. Secara spesifik, 
kita bisa pakai method `select_first` buat menemukan instance pertama dari CSS 
_selector_ yang kita kasih. Dengan memasukkan string `"title"`, kita bakal dapet 
elemen `<title>` pertama di dokumen tersebut, kalau memang ada. Karena mungkin 
saja nggak ada elemen yang cocok, `select_first` mengembalikan `Option<ElementRef>`. 
Terakhir, kita memakai method `Option::map`, yang membolehkan kita beroperasi 
pada item di dalam `Option` kalau itemnya ada, dan tidak melakukan apa-apa 
kalau itemnya nggak ada. (Kita juga bisa saja pakai ekspresi `match` di sini, 
tapi `map` itu lebih idiomatik.) Di dalam isi fungsi yang kita berikan ke `map`, 
kita memanggil `inner_html` pada `title` buat mendapatkan kontennya, yang mana 
adalah sebuah `String`. Pada akhirnya, kita punya sebuah `Option<String>`.

Perhatikan bahwa keyword `await` di Rust ditaruh _setelah_ ekspresi yang lagi 
kita tungguin, bukan di sebelumnya. Yakni, ia adalah sebuah keyword _postfix_ 
(akhiran). Ini mungkin beda dari apa yang biasa kita jumpai kalau kita pernah 
memakai `async` di bahasa lain, tapi di Rust ini bikin _chaining_ (rentetan) 
pemanggilan method jadi jauh lebih enak buat dibuat. Hasilnya, kita bisa 
mengubah isi dari `page_title` buat menyambung (_chain_) pemanggilan fungsi 
`trpl::get` dan `text` sekaligus pakai `await` di antara mereka, kayak yang 
ditunjukkan di Listing 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Chaining (menyambung) dengan keyword `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Selesai deh, kita sudah berhasil menulis fungsi asinkron pertama kita! Sebelum 
kita nambahin beberapa kode di `main` buat manggil dia, mari kita ngomongin 
lebih lanjut soal apa yang sudah kita tulis ini dan apa maknanya.

Pas Rust melihat ada blok yang ditandai pakai keyword `async`, dia 
mengompilasinya jadi tipe data anonim unik yang mengimplementasikan trait 
`Future`. Pas Rust melihat fungsi ditandai pakai `async`, dia mengompilasinya 
jadi fungsi non-asinkron yang isinya merupakan sebuah blok asinkron. Tipe 
kembalian fungsi asinkron adalah tipe dari tipe data anonim yang dibuat sama 
_compiler_ buat blok asinkron tersebut.

Jadi, menulis `async fn` itu ekuivalen (sama aja) kayak menulis fungsi yang 
mengembalikan sebuah _future_ dari tipe kembaliannya. Bagi _compiler_, definisi 
fungsi kayak `async fn page_title` di Listing 17-1 itu kira-kira ekuivalen 
sama fungsi non-asinkron yang didefinisikan kayak gini:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Mari kita telusuri bagian demi bagian dari versi yang sudah diubah ini:

- Dia memakai sintaks `impl Trait` yang sudah kita bahas dulu di Bab 10 di 
  bagian [“Traits sebagai Parameter”][impl-trait]<!-- ignore -->.
- Nilai yang dikembalikan mengimplementasikan trait `Future` dengan 
  _associated type_ `Output`. Perhatikan bahwa tipe `Output`-nya adalah 
  `Option<String>`, yang mana sama dengan tipe kembalian asli dari versi 
  `async fn` si `page_title`.
- Semua kode yang dipanggil di dalam isi dari fungsi aslinya dibungkus di dalam 
  sebuah blok `async move`. Ingat kembali kalau blok itu adalah ekspresi. 
  Keseluruhan blok ini adalah ekspresi yang dikembalikan dari fungsinya.
- Blok asinkron ini menghasilkan nilai bertipe `Option<String>`, seperti yang 
  baru saja dijelaskan. Nilai tersebut cocok sama tipe `Output` di tipe 
  kembaliannya. Ini sama persis kayak blok-blok lain yang pernah kita lihat.
- Isi fungsi baru tersebut adalah sebuah blok `async move` gara-gara gimana 
  dia memakai parameter `url`. (Kita bakal ngebahas lebih banyak lagi soal 
  `async` versus `async move` nanti di bab ini.)

Sekarang kita bisa memanggil `page_title` di `main`.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### Mengeksekusi Fungsi Asinkron dengan sebuah Runtime

Sebagai awalan, kita cuma bakal mengambil judul buat satu halaman saja, yang 
ditunjukkan di Listing 17-3. Sayangnya, kode ini belum bisa di-compile.

<Listing number="17-3" file-name="src/main.rs" caption="Memanggil fungsi `page_title` dari `main` memakai argumen yang dikasih sama user">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Kita mengikuti pola yang sama yang kita pakai buat dapat argumen _command line_ 
di Bab 12 di bagian [“Menerima Argumen Command Line”][cli-args]<!-- ignore -->. 
Terus kita mengoper URL pertamanya ke `page_title` dan menunggu hasilnya. 
Karena nilai yang dihasilkan oleh _future_ tersebut adalah sebuah 
`Option<String>`, kita memakai ekspresi `match` buat mencetak pesan yang beda-
beda dengan memperhitungkan apakah halamannya punya `<title>` atau tidak.

Satu-satunya tempat di mana kita bisa memakai keyword `await` adalah di dalam 
fungsi atau blok asinkron, dan Rust tidak bakal membolehkan kita menandai 
fungsi spesial `main` sebagai `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

Alasan `main` tidak bisa ditandai `async` adalah karena kode asinkron itu 
butuh sebuah _runtime_: sebuah _crate_ Rust yang mengelola detail eksekusi kode 
asinkron. Fungsi `main` di sebuah program bisa _menginisialisasi_ (initialize) 
sebuah _runtime_, tapi fungsi `main` itu bukanlah sebuah _runtime_ itu sendiri. 
(Kita bakal lihat lebih lanjut soal kenapa ini terjadi sebentar lagi.) Setiap 
program Rust yang mengeksekusi kode asinkron punya minimal satu tempat di mana 
dia menyiapkan sebuah _runtime_ yang mengeksekusi _futures_-nya.

Kebanyakan bahasa yang mendukung asinkron sudah membundel sebuah _runtime_ 
bawaan, tapi Rust tidak begitu. Sebaliknya, ada banyak _async runtimes_ berbeda 
yang tersedia, di mana masing-masing membikin _tradeoffs_ (pertukaran) yang 
cocok buat kasus penggunaan yang jadi targetnya. Misalnya, _web server_ yang 
_high-throughput_ (kemampuan transmisi besar) dengan banyak _core_ CPU dan RAM 
dalam jumlah besar punya kebutuhan yang sangat berbeda dari mikrokontroler 
dengan _single core_, jumlah RAM yang kecil, dan tidak punya kemampuan alokasi 
_heap_ sama sekali. Crate yang menyediakan _runtimes_ ini juga sering kali 
memberikan versi asinkron dari fungsionalitas umum kayak I/O file atau jaringan.

Di sini, dan di sepanjang sisa bab ini, kita bakal memakai fungsi `block_on` 
dari _crate_ `trpl`, yang mana menerima sebuah _future_ sebagai argumen dan 
memblokir _thread_ saat ini sampai _future_ tersebut berjalan hingga selesai. 
Di balik layar, memanggil `block_on` menyiapkan sebuah _runtime_ menggunakan 
_crate_ `tokio` yang dipakai buat menjalankan _future_ yang diberikan (perilaku 
`block_on` dari _crate_ `trpl` ini mirip dengan fungsi `block_on` milik _crate_ 
_runtime_ lainnya). Setelah _future_ tersebut selesai, `block_on` bakal 
mengembalikan nilai apa pun yang dihasilkan oleh _future_ itu.

Kita bisa saja meneruskan _future_ yang dikembalikan sama `page_title` langsung 
ke `block_on` dan, begitu selesai, kita bisa melakukan `match` pada 
`Option<String>` hasilnya, kayak yang sudah kita coba lakukan di Listing 17-3. 
Tapi, buat mayoritas contoh di bab ini (dan mayoritas kode asinkron di dunia 
nyata), kita bakal melakukan lebih dari sekadar satu pemanggilan fungsi 
asinkron saja, jadi alih-alih begitu kita bakal meneruskan sebuah blok `async` 
lalu secara eksplisit menunggu hasil dari panggilan `page_title`, seperti di 
Listing 17-4.

<Listing number="17-4" caption="Menunggu sebuah blok asinkron dengan `trpl::block_on`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Pas kita jalankan kode ini, kita dapet perilaku kayak yang kita harapkan di awal:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Fiuh—akhirnya kita punya kode asinkron yang jalan! Tapi sebelum kita nambahin 
kode buat mengadu (_race_) kedua situs web itu satu sama lain, mari kita alihkan 
sejenak perhatian kita kembali ke gimana _futures_ itu bekerja.

Setiap _await point_ (titik penantian)—yakni, setiap tempat di mana kodenya 
memakai keyword `await`—merepresentasikan sebuah tempat di mana kontrol 
dikembalikan ke _runtime_. Biar itu bisa terjadi, Rust perlu melacak (_keep 
track of_) _state_ (keadaan) yang terlibat di dalam blok asinkron tersebut 
sehingga _runtime_ bisa memulai beberapa pekerjaan lain lalu balik lagi nanti 
kalau dia sudah siap buat mencoba melanjutkan pekerjaan pertama tadi. Ini 
adalah sebuah _state machine_ (mesin keadaan) kasatmata, seolah-olah kita 
menulis sebuah enum kayak gini buat menyimpan _state_ saat ini di tiap titik 
_await_:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Menulis kode buat bertransisi di antara setiap _state_ ini secara manual bakal 
melelahkan dan gampang rawan error, apalagi kalau nanti kita harus menambahkan 
fungsionalitas dan lebih banyak _states_ lagi ke kode tersebut. Untungnya, 
_compiler_ Rust otomatis membikin dan mengelola struktur data _state machine_ 
buat kode asinkron. Aturan-aturan _borrowing_ dan _ownership_ normal seputar 
struktur data itu tetap berlaku semua, dan syukurnya, _compiler_ juga menangani 
pengecekan itu buat kita dan menyediakan pesan error yang berguna. Kita bakal 
membedah beberapa kasus kayak gitu nanti di bab ini.

Pada akhirnya, sesuatu harus mengeksekusi _state machine_ ini, dan "sesuatu" 
itu adalah sebuah _runtime_. (Inilah kenapa kita mungkin pernah ketemu istilah 
_executors_ (pengeksekusi) pas lagi nyari tahu soal _runtimes_: sebuah 
_executor_ adalah bagian dari sebuah _runtime_ yang bertugas mengeksekusi kode 
asinkron tersebut.)

Sekarang kita bisa tahu kenapa _compiler_ melarang kita membikin `main` itu 
sendiri jadi fungsi asinkron balik di Listing 17-3 tadi. Kalau `main` itu fungsi 
asinkron, sesuatu yang lain bakal harus mengelola _state machine_ buat _future_ 
apa pun yang dikembalikan sama `main`, tapi padahal `main` adalah titik awal 
buat programnya! Sebaliknya, kita memanggil fungsi `trpl::block_on` di `main` 
buat menyiapkan sebuah _runtime_ dan menjalankan _future_ yang dikembalikan 
sama blok `async` sampai dia selesai.

> Catatan: Beberapa _runtimes_ menyediakan macros sehingga kita _bisa_ menulis 
> fungsi `main` yang asinkron. Macros itu menulis ulang `async fn main() { ... }` 
> jadi `fn main` normal, yang melakukan persis hal yang sama kayak yang kita 
> lakukan secara manual di Listing 17-4: memanggil fungsi yang mengeksekusi 
> sebuah _future_ sampai selesai kayak yang dilakukan sama `trpl::block_on`.

Sekarang mari kita gabungkan bagian-bagian ini dan lihat gimana kita bisa 
menulis kode konkuren.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### Menandingkan (Racing) Dua URL Secara Konkuren

Di Listing 17-5, kita memanggil `page_title` dengan dua URL berbeda yang 
dimasukkan dari _command line_ lalu mengadu (_race_) mereka berdua dengan 
memilih _future_ mana pun yang selesai duluan.

<Listing number="17-5" caption="Memanggil `page_title` buat dua URL untuk melihat mana yang kembali duluan" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Kita mulai dengan memanggil `page_title` buat tiap URL yang dikasih sama user. 
Kita simpan _futures_ hasilnya sebagai `title_fut_1` dan `title_fut_2`. Ingat, 
mereka ini belum melakukan apa-apa, karena _futures_ itu sifatnya malas dan 
kita belum menunggunya. Terus kita mengoper _futures_ tersebut ke 
`trpl::select`, yang mengembalikan sebuah nilai buat mengindikasikan _future_ 
mana yang selesai duluan di antara yang dioper kepadanya.

> Catatan: Di balik layar, `trpl::select` dibangun di atas fungsi `select` yang 
> lebih umum yang didefinisikan di _crate_ `futures`. Fungsi `select` milik 
> _crate_ `futures` bisa melakukan banyak hal yang fungsi `trpl::select` tidak 
> bisa, tapi dia juga punya kerumitan ekstra yang bisa kita lewati dulu buat 
> sekarang.

Masing-masing _future_ bisa saja "menang," jadi tidak masuk akal kalau kita 
mengembalikan `Result`. Alih-alih begitu, `trpl::select` mengembalikan sebuah 
tipe yang belum pernah kita lihat sebelumnya, yaitu `trpl::Either`. Tipe 
`Either` itu agak mirip sama `Result` dalam hal dia punya dua kasus. Bedanya 
sama `Result`, tidak ada konsep "sukses" atau "gagal" yang tertanam di dalam 
`Either`. Alih-alih begitu, dia memakai `Left` (kiri) dan `Right` (kanan) buat 
mengindikasikan "yang satu atau yang lainnya":

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

Fungsi `select` mengembalikan `Left` yang berisi output dari _future_ tersebut 
kalau argumen pertama menang, dan `Right` yang berisi output _future_ kedua 
kalau _yang itu_ yang menang. Ini cocok dengan urutan munculnya argumen-argumen 
tersebut saat memanggil fungsinya: argumen pertama ada di kiri argumen kedua.

Kita juga memperbarui `page_title` buat mengembalikan URL yang sama dengan yang 
dimasukkan. Dengan begitu, kalau halaman yang kembali duluan tidak punya 
`<title>` yang bisa kita uraikan, kita masih bisa mencetak pesan yang bermakna. 
Dengan informasi yang sudah tersedia itu, kita selesaikan ini semua dengan 
mengubah output `println!` kita buat mengindikasikan baik URL mana yang selesai 
duluan, dan apa, kalau memang ada, `<title>` buat halaman web di URL tersebut.

Kita sudah ngebikin _web scraper_ mini yang bisa jalan sekarang! Silakan pilih 
beberapa URL lalu jalankan alat _command line_ kita. Kita mungkin mendapati 
kalau beberapa situs secara konsisten memang lebih kencang dibanding yang lain, 
sementara di kasus lain situs yang kencang itu berubah-ubah di tiap jalan. Yang 
lebih penting, kita sudah belajar dasar-dasar dari bekerja dengan _futures_, 
jadi sekarang kita bisa gali lebih dalam soal apa yang bisa kita lakukan 
dengan asinkron.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
