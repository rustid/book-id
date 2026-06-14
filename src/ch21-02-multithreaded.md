## Mengubah Server Single-Threaded Kita Menjadi Server Multithreaded

Saat ini, server kita memproses setiap _request_ secara bergantian (in turn), 
yang berarti dia tidak bakal memproses koneksi yang kedua sebelum proses untuk 
koneksi yang pertama selesai (finished processing). Kalau server ini nerima 
makin banyak _requests_, eksekusi secara berurutan (serial execution) kayak gini 
bakal jadi makin kurang optimal. Kalau server ini nerima sebuah _request_ yang 
makan waktu lama sekali buat diproses, _requests_ yang masuk berikutnya 
(subsequent requests) bakal terpaksa harus nungguin sampai _request_ lama itu 
selesai, biarpun _requests_ yang baru ini sebenernya bisa aja diproses dengan cepat. 
Kita perlu benerin ini nih, tapi pertama-tama mari kita ngelihat aksinya masalah 
ini secara langsung.

<!-- Old headings. Do not remove or links may break. -->
<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Menyimulasikan Sebuah Request yang Pelan (Slow Request)

Kita bakal ngelihat gimana sebuah _request_ yang pemrosesannya lambat bisa 
berdampak sama _requests_ lainnya yang dilakuin ke implementasi server kita 
saat ini. Listing 21-10 mengimplementasikan penanganan sebuah _request_ ke 
_path_ _/sleep_ dengan menyimulasikan (simulated) sebuah balasan yang pelan yang 
mana bakal ngebikin server kita ini _sleep_ (tidur/berhenti sejenak) selama lima 
detik sebelum dia ngasih balasan (responding).

<Listing number="21-10" file-name="src/main.rs" caption="Menyimulasikan sebuah request yang pelan dengan cara menidurkan server selama lima detik">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Kita beralih (_switched_) dari yang asalnya pakai `if` jadi `match` karena sekarang 
kita punya tiga kemungkinan kasus (cases). Kita perlu secara eksplisit memakai 
pencocokan (match) pada sebuah _slice_ dari `request_line` buat mencocokkan _pattern_ 
(pattern-match) dengan nilai-nilai _string literal_ tersebut; `match` itu tidak 
secara otomatis ngelakuin referencing dan dereferencing kayak yang dilakuin sama 
method *equality* (pemeriksaan kesamaan `==`).

Arm (lengan) pertama ini bunyinya sama kayak isi dari blok `if` yang ada di Listing 
21-9. Arm kedua cocok dengan sebuah _request_ ke _path_ _/sleep_. Pas _request_ ini 
diterima, server kita ini bakal tidur (sleep) selama lima detik sebelum dia nge-_render_ 
halaman HTML yang isinya sukses tersebut. Arm yang ketiga bunyinya sama persis kayak 
blok `else` dari Listing 21-9.

Kita bisa ngelihat sendiri betapa primitifnya (primitive) server kita ini: 
_libraries_ sungguhan (real libraries) itu biasanya nanganin proses rekognisi 
(pengenalan) banyak _requests_ dengan cara yang jauh tidak lebih _verbose_ 
(kepanjangan nulisnya) dari ini!

Jalanin servernya memakai `cargo run`. Terus buka dua jendela (windows) browser: 
satu buat _http://127.0.0.1:7878_ dan satu lagi buat _http://127.0.0.1:7878/sleep_. 
Kalau kita masukin URI _/_ beberapa kali kayak sebelumnya, kita bakal ngelihat kalau dia 
nge-responsnya cepet sekali. Tapi kalau kita masukin _/sleep_ dan kemudian muat (_load_) 
_/_ di tab lain, kita bakal ngelihat kalau _/_ ini terpaksa harus nungguin (waits) 
sampai si `sleep` tadi selesai tidur selama durasi penuh lima detiknya sebelum 
halamannya bisa dimuat.

Ada banyak teknik yang bisa kita pakai buat ngehindarin situasi di mana _requests_ 
ini pada numpuk (backing up) di belakang sebuah _request_ yang pelan, termasuk 
salah satunya dengan memakai `async` kayak yang udah kita lakuin di Bab 17; 
sementara yang bakal kita implementasikan di sini adalah sebuah _thread pool_ 
(kumpulan utas).

### Meningkatkan Throughput dengan Sebuah Thread Pool

Sebuah _thread pool_ itu adalah sekelompok _threads_ yang udah ditelurkan (spawned) 
yang lagi bersiap-siap dan standby nungguin buat nanganin sebuah tugas (task). 
Saat program tersebut nerima tugas yang baru, dia bakal ngasih salah satu _threads_ 
yang ada di dalam kolam (pool) ini buat ngerjain tugas tersebut, dan _thread_ itulah 
yang bakal memprosesnya. Sisa _threads_ lainnya yang ada di dalam _pool_ ini tetep 
tersedia (available) buat nanganin tugas-tugas lain yang masuk saat _thread_ yang 
pertama tadi lagi sibuk memproses. Pas _thread_ pertama udah beres ngerjain tugasnya, 
dia dikembaliin (_returned_) lagi ke dalam _pool_ yang isinya _threads_ nganggur (idle 
threads), terus dia siap (ready) buat nanganin tugas baru lagi. Sebuah _thread pool_ 
memungkinkan kita buat memproses banyak koneksi secara konkuren (bersamaan), ningkatin 
_throughput_ (kemampuan nangani permintaan) dari server kita.

Kita bakal membatasi (limit) jumlah _threads_ yang ada di dalam _pool_ ini menjadi 
angka yang kecil buat melindungi (protect) kita dari serangan DoS (Denial of Service); 
kalau kita ngebikin program kita buat netasin (_create_) _thread_ baru buat _setiap_ 
kali ada _request_ yang masuk, seseorang yang ngebikin 10 juta _requests_ ke server kita 
bisa-bisa bikin kekacauan parah dengan cara ngabisin semua sumber daya (resources) 
server kita lalu bikin semua pemrosesan _requests_ jadi mandek total (grinding to a halt).

Jadi ketimbang menelurkan _threads_ tanpa batas (unlimited threads), kita bakal 
punya jumlah _threads_ yang tetap (fixed number) yang pada _standby_ nungguin di 
dalam _pool_ tersebut. _Requests_ yang masuk bakal dikirimin ke dalam _pool_ ini 
buat diproses. _Pool_ ini bakal memelihara sebuah antrean (_queue_) yang isinya _requests_ 
yang baru masuk. Masing-masing dari _threads_ yang ada di dalam _pool_ ini bakal 
mengambil (_pop off_) satu _request_ dari antrean ini, menangani _request_ tersebut, 
dan lalu minta satu _request_ lagi ke antrean tersebut. Pakai desain kayak gini, 
kita bisa memproses maksimal _`N`_ _requests_ secara konkuren, di mana _`N`_ itu 
adalah jumlah _threads_ yang ada. Kalau setiap _thread_ itu lagi sibuk merespons ke 
_requests_ yang jalan lama sekali, _requests_ yang masuk berikutnya emang masih 
tetap bisa pada numpuk di dalem antreannya, tapi kita udah ningkatin seberapa banyak jumlah 
_requests_ yang jalannya lama sekali yang sanggup kita tangani sebelum kita nyampe ke titik 
jenuh tersebut.

Teknik ini itu hanyalah salah satu dari sekian banyak cara yang ada buat ningkatin 
_throughput_ dari sebuah web server. Opsi-opsi lain yang mungkin bisa kita eksplorasi 
adalah model _fork/join_, model _single-threaded async I/O_, sama model _multithreaded 
async I/O_. Kalau kita tertarik sama topik ini, kita bisa ngebaca lebih lanjut soal 
solusi-solusi lainnya dan nyobain mengimplementasikan mereka; dengan bahasa pemrograman 
tingkat rendah (low-level) kayak Rust ini, semua opsi ini sangat mungkin sekali buat 
dikerjain (possible).

Sebelum kita mulai mengimplementasikan sebuah _thread pool_, mari kita obrolin kayak gimana 
rupa dari memakai si _pool_ ini nantinya (what using the pool should look like). 
Pas kita lagi mencoba mendesain (_design_) sebuah kode, menulis *interface* 
_client_-nya terlebih dahulu bisa ngebantu memandu jalannya desain kita. Tulis API 
dari kodenya sehingga strukturnya itu udah sesuai dengan cara kita manggil dia nantinya; 
baru deh setelah itu implementasikan fungsionalitasnya di dalam struktur tersebut 
ketimbang mikirin fungsionalitasnya duluan baru mikirin desain API _public_-nya belakangan.

Mirip dengan gimana kita memakai _test-driven development_ (pengembangan berbasis pengujian) 
di dalam _project_ kita pas Bab 12 kemarin, kita bakal memakai *compiler-driven development* 
(pengembangan berbasis _compiler_) di sini. Kita bakal nulis kode yang manggil 
fungsi-fungsi yang pengen kita panggil, lalu baru deh kita ngelihat ke error-error 
yang dikasih sama _compiler_ buat nentuin apa yang harus kita ubah berikutnya supaya 
kodenya bisa benar-benar jalan. Tapi sebelum kita melakukan itu, kita bakal nyelidikin 
teknik yang _tidak_ bakal kita pakai dulu sebagai titik mulai kita.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Menelurkan (Spawning) Sebuah Thread Buat Setiap Request

Pertama-tama, mari kita eksplorasi kira-kira kayak gimana kelihatannya kode kita ini kalau 
seandainya dia *benar-benar* ngebikin _thread_ baru buat setiap koneksi yang masuk. Seperti yang 
udah kita sebutin sebelumnya, ini itu bukan rencana akhir kita gara-gara ada masalah yang 
mana kita berpotensi bakal netasin _threads_ dalam jumlah yang tidak terbatas, tapi cara ini 
adalah titik pijak (starting point) yang oke buat ngebikin supaya server _multithreaded_ 
kita ini bisa jalan dulu. Nanti barulah kita tambahin _thread pool_ sebagai 
sebuah perbaikan (_improvement_), dan jadinya membandingkan (contrasting) kedua buah 
solusi ini bakal jadi lebih gampang.

Listing 21-11 nunjukin beberapa perubahan yang perlu dibikin di dalam fungsi `main` 
supaya dia menelurkan (spawn) sebuah _thread_ baru buat nanganin masing-masing 
_stream_ yang ada di dalam _loop_ `for` tersebut.

<Listing number="21-11" file-name="src/main.rs" caption="Menelurkan sebuah thread baru buat masing-masing stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Kayak yang udah kita pelajarin di Bab 16, `thread::spawn` itu bakal ngebikin _thread_ 
baru lalu dia bakal ngejalanin kode yang ada di dalam _closure_ tersebut di dalem si _thread_ 
yang baru ini. Kalau kita jalanin kode ini dan memuat _/sleep_ di browser kita, lalu buka 
_/_ di dua tab browser yang lain, kita benar-benar bakal ngelihat kalau _requests_ ke _/_ itu 
tidak perlu lagi nungguin si _/sleep_ sampai selesai beres (finish). Namun, seperti yang 
tadi udah kita sebutin, cara ini pada akhirnya bakal bikin sistemnya kewalahan (overwhelm 
the system) karena kita bakal terus-terusan ngebikin _threads_ baru tanpa ada batas 
sama sekali.

Kita juga mungkin masih inget dari Bab 17 kalau ini itu adalah tipe-tipe situasi 
persis yang mana _async_ dan _await_ bakal benar-benar bersinar! Simpan pikiran itu di kepala 
kita ya selagi kita ngebangun _thread pool_ ini dan coba renungkan (think about) gimana 
situasinya bakal kelihatan berbeda atau malah sama aja kalau kita pakai _async_.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Ngebikin Sejumlah Threads dalam Jumlah Terbatas (Finite Number of Threads)

Kita mau supaya _thread pool_ kita ini bekerja dengan cara yang mirip-mirip dan kerasa 
familier supaya pindah (switching) dari _threads_ biasa ke _thread pool_ ini tidak butuh 
perubahan gede-gedean pada kode-kode yang memakai API kita. Listing 21-12 nunjukin 
antarmuka bayangan (hypothetical interface) buat sebuah struct `ThreadPool` yang pengen 
kita pakai ketimbang `thread::spawn`.

<Listing number="21-12" file-name="src/main.rs" caption="Antarmuka (interface) `ThreadPool` ideal milik kita">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Kita memakai `ThreadPool::new` buat ngebikin sebuah _thread pool_ baru dengan jumlah 
_threads_ yang bisa dikonfigurasi, yang di kasus ini yaitu empat biji. Terus, di dalam 
_loop_ `for`, si `pool.execute` ini punya antarmuka (interface) yang mirip sekali 
sama `thread::spawn` karena dia juga nerima sebuah _closure_ yang mana seharusnya bakal 
dijalanin sama si _pool_ tersebut buat setiap _stream_ yang masuk. Kita perlu mengimplementasikan 
`pool.execute` ini sedemikian rupa sehingga dia bakal ngambil _closure_ yang diterimanya 
itu terus ngasihin itu ke salah satu _thread_ yang ada di dalam _pool_ buat dijalanin. 
Kode ini jelas masih belum bisa di-compile, tapi kita bakal nyoba men-compile-nya 
supaya si _compiler_ bisa memandu (guide) kita soal gimana caranya ngeberesin ini.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Ngebangun `ThreadPool` Memakai Compiler-Driven Development (Pengembangan Berbasis Compiler)

Silakan bikin perubahan-perubahan yang ada di Listing 21-12 ke file _src/main.rs_ kita, 
dan lalu mari kita pakai pesan-pesan error _compiler_ yang asalnya dari `cargo check` 
buat mengarahkan jalan (_drive_) dari proses _development_ kita. Ini dia error 
pertama yang kita dapet:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Sip sekali! (Great!) Error ini ngasih tahu kita kalau kita ini butuh punya tipe atau 
modul `ThreadPool`, jadi kita bakal ngebangunnya sekarang juga. Implementasi `ThreadPool` 
kita ini sifatnya bakal independen (independent) dan tidak peduli apa jenis kerjaan 
yang lagi dilakuin sama web server kita ini. Jadi mari kita alihkan (switch) 
_crate_ `hello` kita ini dari yang tadinya sebuah _binary crate_ menjadi sebuah 
_library crate_ buat nampung kode implementasi `ThreadPool` kita ini. Setelah kita ngubah 
dia jadi _library crate_, kita juga jadi bisa lho memakai _library_ _thread pool_ yang 
udah terpisah ini buat sekiranya ada pekerjaan apa pun lainnya yang mau kita lakuin pakai 
sebuah _thread pool_, bukannya cuma khusus buat ngelayanin (serving) web _requests_ doang.

Bikin sebuah file _src/lib.rs_ yang isinya mengandung yang berikut ini, yang mana ini 
adalah definisi paling simpel yang bisa kita punya buat sebuah struct `ThreadPool` buat 
saat ini:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>


Terus edit file _main.rs_ kita buat ngebawa (bring) `ThreadPool` tersebut masuk ke 
dalam _scope_ yang asalnya dari _library crate_ dengan nambahin kode berikut ke bagian 
paling atas (top) dari _src/main.rs_:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Kode ini tentu masih belum bisa jalan ya, tapi mari kita cek (check) kodenya lagi 
buat dapetin pesan error selanjutnya yang perlu kita beresin (address):

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Error ini mengindikasikan kalau langkah selanjutnya yang perlu kita lakuin adalah 
ngebikin sebuah fungsi _associated_ bernama `new` untuk si `ThreadPool` ini. Kita 
juga tahu kalau `new` ini butuh satu parameter yang mana bisa menerima angka `4` 
sebagai argumen dan harus ngembaliin sebuah instance `ThreadPool`. 
Mari kita implementasikan fungsi `new` yang paling simpel yang punya karakteristik 
kayak gitu:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Kita milih `usize` sebagai tipe buat parameter `size` karena kita tahu kalau ngebikin 
_threads_ dengan jumlah minus (negative number) itu emang kedengerannya 
tidak masuk akal (doesn't make sense). Kita juga tahu kalau kita bakal makek angka 
`4` ini sebagai ukuran jumlah elemen di dalam sebuah koleksi (collection) yang 
isinya _threads_, yang mana itu adalah tujuan asli kenapa tipe `usize` dibikin, kayak yang udah 
kita obrolin di [“Tipe-tipe Angka Bulat (Integer Types)”][integer-types] di Bab 3.

Mari kita cek kodenya lagi:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Sekarang errornya kejadian gara-gara kita tidak punya method `execute` pada 
struct `ThreadPool` kita. Ingat kembali materi dari [“Ngebikin Sejumlah Threads dalam 
Jumlah Terbatas (Finite Number of Threads)”](#creating-a-finite-number-of-threads) tadi 
di mana kita memutuskan kalau _thread pool_ kita ini seharusnya punya _interface_ (antarmuka) 
yang mirip sama `thread::spawn`. Selain itu, kita bakal mengimplementasikan fungsi 
`execute` ini sedemikian rupa sehingga dia nerima _closure_ yang udah dikasih ke dia 
lalu mengopernya ke sebuah _thread_ yang lagi nganggur (idle thread) di dalam si _pool_ 
tersebut buat dijalanin.

Kita bakal mendefinisikan method `execute` pada `ThreadPool` ini supaya dia menerima 
sebuah _closure_ sebagai parameter. Ingat kembali dari [“Mengoper Nilai yang Ditangkap 
Keluar dari Closure dan Trait `Fn`”][fn-traits] di Bab 13 kalau kita bisa nerima _closures_ 
sebagai parameter yang memakai tiga jenis trait yang berbeda: yaitu `Fn`, `FnMut`, dan 
`FnOnce`. Kita harus memutuskan trait _closure_ mana yang mau dipakai di sini. Kita tahu 
kalau ujung-ujungnya kita ini bakal ngerjain hal yang mirip-mirip sama yang dilakuin 
oleh implementasi `thread::spawn` punya _standard library_, jadi kita bisa nyontek (_look at_) 
batasan-batasan (bounds) apa aja yang dipunyai sama _signature_ si `thread::spawn` ini pada parameternya. 
Dokumentasi di sana ngasih tahu kita hal berikut:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Parameter bertipe `F` inilah yang lagi jadi fokus (concerned with) kita di sini; 
sedangkan parameter bertipe `T` itu ada kaitannya sama nilai kembalian 
(return value) dari fungsi itu, yang mana itu tidak jadi masalah buat kita. Kita bisa 
ngelihat kalau si `spawn` ini memakai `FnOnce` sebagai _trait bound_ (batasan trait) 
buat `F`-nya. Ini juga merupakan trait yang kemungkinan besar kita inginkan, karena 
nantinya argumen _closure_ yang kita dapat di dalam `execute` ini juga pada akhirnya 
bakal kita operin (pass) ke dalam `spawn`. Kita bisa makin yakin kalau `FnOnce` itu 
adalah trait yang benar-benar pengen kita pake soalnya _thread_ yang dijalanin buat 
nanganin satu _request_ itu emang cuma bakal mengeksekusi _closure_ buat _request_ tersebut 
sebanyak satu kali doang, yang mana ya cocok persis (matches) sama embel-embel kata 
`Once` (sekali) di dalam trait `FnOnce`.

Parameter bertipe `F` itu juga punya _trait bound_ `Send` dan _lifetime bound_ 
(batasan rentang hidup) `'static`, yang mana emang sangat berguna buat situasi kita 
saat ini: kita butuh trait `Send` ini buat mindahin (transfer) si _closure_ 
ini dari satu _thread_ ke _thread_ yang lainnya dan `'static` ini gara-gara kita tidak 
tahu seberapa lama waktu yang dibutuhkan sama si _thread_ tersebut buat selesai 
melakukan eksekusi kodenya. Mari kita bikin method `execute` pada `ThreadPool` 
yang bakal menerima parameter generik (generic parameter) bertipe `F` dengan 
_bounds_ berikut:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Kita tetep pake tambahan `()` setelah trait `FnOnce` tersebut karena si `FnOnce` 
ini merepresentasikan sebuah _closure_ yang mana dia tidak menerima parameter 
apa-apa dan dia juga mengembalikan _unit type_ (tipe unit kosong) `()`. Sama kayak 
halnya definisi fungsi (function definitions), tipe _return_-nya bisa 
aja disingkirkan (omitted) dari _signature_-nya, tapi meskipun kita tidak 
nerima parameter apa-apa di dalam _closure_-nya, kita tetep wajib nulisin tanda kurung 
yang kosong tersebut (parentheses).

Sekali lagi, ini adalah sekadar implementasi paling simpel (simplest) dari method 
`execute`: dia sama sekali tidak ngelakuin apa-apa, tapi kan tujuan kita cuma 
mau ngebikin kode kita sukses di-compile doang buat saat ini. Mari kita cek (check) lagi deh:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Kompilasi sukses! (It compiles!) Tapi perlu dicatet nih kalau seandainya kita 
mencoba ngejalanin pake `cargo run` dan terus nyoba ngasih sebuah _request_ dari browser, 
kita bakal ngelihat lagi error-error di browser tadi yang sempat kita lihat di 
bagian awal bab ini. _Library_ kita ini masih belum secara harfiah (actually) memanggil 
_closure_ yang dioper ke dalem fungsi `execute` lho ya!

> Catatan: Pepatah yang mungkin sering kita dengar soal bahasa-bahasa pemrograman 
> yang _compiler_-nya rewel (strict compilers), kayak Haskell dan Rust, adalah 
> "kalau kodenya berhasil di-compile, berarti kodenya jalan." Tapi pepatah ini 
> itu tidak selalu bener kok. Project kita ini sukses di-compile kan, padahal 
> dia itu bener-bener tidak ngelakuin apa-apa sama sekali! Kalau seandainya kita ini 
> lagi ngebangun project sungguhan yang lengkap, ini adalah saat-saat yang paling 
> pas buat mulai nulisin unit _tests_ buat ngetes (_check_) apakah kodenya 
> sukses di-compile _sekaligus_ punya perilaku yang emang kita mau atau tidak.

Renungkan ini: kira-kira apa yang bakal berbeda di sini kalau seandainya kita ini 
lagi mau ngejalanin sebuah *future* ketimbang sebuah _closure_?

#### Memvalidasi Jumlah Threads yang Ada di `new`

Kita sama sekali tidak ngelakuin tindakan apa-apa lho sama parameter-parameter 
yang ada di fungsi `new` dan `execute` ini. Mari kita implementasikan *body* 
(isi) dari fungsi-fungsi ini supaya punya perilaku yang emang kita inginkan. Buat 
memulai, mari kita pikirin soal fungsi `new`. Sebelumnya kita udah milih (chose) tipe 
tanpa tanda (unsigned type) buat parameter `size` karena ngebikin sebuah _pool_ dengan jumlah _threads_ 
yang negatif itu emang tidak masuk akal (makes no sense). Tapi ya, 
sebuah _pool_ dengan angka nol _threads_ juga tidak kalah tidak masuk akalnya dong, 
padahal angka nol itu adalah angka yang sah-sah aja (perfectly valid) di tipe 
`usize`. Kita bakal tambahin barisan kode buat ngecek (check) kalau variabel `size` 
itu harus lebih gede dari angka nol sebelum kita mengembalikan sebuah instance 
`ThreadPool` lalu ngebikin programnya jadi _panic_ dengan memakai _macro_ 
`assert!` kalau ternyata kita dikasih angka nol, kayak yang kelihatan di 
Listing 21-13.

<Listing number="21-13" file-name="src/lib.rs" caption="Mengimplementasikan `ThreadPool::new` supaya dia panic kalau `size`-nya nol">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Kita juga udah nambahin secuil dokumentasi buat `ThreadPool` kita memakai komentar 
dok (doc comments). Perhatikan kalau kita udah mengikuti (followed) kaidah penulisan dokumentasi 
yang bagus (good documentation practices) dengan cara nambahin sebuah seksi yang mana 
membeberkan kasus-kasus (call out the situations) di mana fungsi kita ini bisa jadi 
_panic_, kayak yang dibahas di Bab 14. Silakan cobain jalanin `cargo doc --open` 
lalu klik struct `ThreadPool` tersebut buat ngelihat kayak apa rupa dari *docs* 
yang udah di-*generate* buat method `new` tersebut!

Ketimbang nambahin _macro_ `assert!` kayak yang baru aja kita lakuin di sini, 
kita sebenernya bisa aja kok ngerubah method `new` ini jadi `build` dan terus 
ngembaliin (return) sebuah tipe `Result` persis kayak apa yang udah kita lakuin 
sama fungsi `Config::build` di dalem *project* I/O kita di Listing 12-9. 
Tapi kita udah memutuskan kalau di kasus kali ini usaha buat ngebikin sebuah _thread pool_ 
tanpa punya satupun _threads_ di dalamnya (without any threads) itu seharusnya 
dijadikan sebuah error yang tidak bisa dipulihkan (unrecoverable error). Kalau 
kita lagi ngerasa ambisius hari ini, cobain deh buat nulis sebuah fungsi bernama 
`build` dengan _signature_ berikut buat ngebandingin hasilnya dengan fungsi 
`new` ini:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Menyiapkan Tempat Buat Menyimpan Para Threads

Sekarang karena kita udah punya cara yang valid buat mengetahui (know) jumlah _threads_ 
yang harus disimpan di dalam _pool_, kita akhirnya bisa ngebikin _threads_ tersebut lalu 
menyimpan mereka di dalam struct `ThreadPool` sebelum kita ngembaliin si struct itu. Tapi 
gimana ya caranya kita "menyimpan" sebuah _thread_? Mari kita ngelirik balik 
(_take another look_) ke _signature_ dari `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Fungsi `spawn` ini ngembaliin (returns) sebuah `JoinHandle<T>`, di mana `T` itu 
adalah tipe balasan (return type) dari _closure_ tersebut. Mari kita cobain pakai 
`JoinHandle` juga deh dan lihat apa yang bakal kejadian. Di kasus yang kita kerjain 
sekarang, _closures_ yang kita oper masuk ke dalem _thread pool_ ini emang tugasnya 
buat nanganin (handle) koneksi dan bukannya buat ngembaliin data apa pun juga, jadi si 
`T` ini nilainya bakal berupa _unit type_ `()`.

Kode yang ada di Listing 21-14 ini bakal berhasil di-compile tapi masih belum ngebikin 
satu pun _threads_. Kita udah ngubah (changed) definisi dari `ThreadPool` supaya dia 
menyimpan (hold) sebuah _vector_ yang isinya berupa _instances_ dari 
`thread::JoinHandle<()>`, menginisialisasi _vector_ tersebut supaya punya kapasitas memori 
sejumlah `size` (with a capacity of `size`), nyiapin (_set up_) _loop_ `for` yang 
nantinya bakal njalanin beberapa kode buat ngebikin _threads_ tersebut, 
dan baru deh membalikkan (_returned_) sebuah instance `ThreadPool` yang ngandung mereka semua.

<Listing number="21-14" file-name="src/lib.rs" caption="Membikin vector buat `ThreadPool` buat menampung para threads yang ada">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Kita udah ngebawa (_brought_) `std::thread` masuk ke dalam _scope_ di dalam _library 
crate_ kita gara-gara kita lagi mau makek `thread::JoinHandle` sebagai tipe buat 
item-item yang bakal masuk ke dalam _vector_ milik si `ThreadPool`.

Setelah angka `size` yang valid itu diterima (received), `ThreadPool` kita ini ngebikin 
sebuah _vector_ baru yang mana sanggup nampung sejumlah `size` item. Fungsi 
`with_capacity` ini ngelakuin tugas yang sama persis kayak fungsi `Vec::new` 
tapi dengan satu perbedaan krusial (important difference): dia udah mengalokasikan memori lebih dulu 
(_pre-allocates space_) ke dalam _vector_ tersebut. Karena kita tahu pasti (we know) 
kalau kita ini perlu menyimpan `size` elemen di dalam _vector_ ini, ngelakuin 
proses alokasi (_allocation_) memori dari awal kayak gini itu slightly (sedikit) 
lebih efisien dan cepet ketimbang makek fungsi `Vec::new` yang mana dia itu bakal 
mengubah ukurannya sendiri (_resizes itself_) setiap kali ada elemen baru yang 
dimasukin ke dalam situ.

Pas kita jalanin perintah `cargo check` lagi, dia harusnya berhasil (succeed).

<!-- Old headings. Do not remove or links may break. -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Ngirimin Kode dari Dalam `ThreadPool` ke Sebuah Thread

Kita ninggalin (left) sebuah komentar di dalam _loop_ `for` tadi yang ada di 
Listing 21-14 mengenai (regarding) urusan pembuatan _threads_. Di sini, kita bakal 
ngelihat gimana sebenarnya langkah yang kita tempuh buat ngebikin _threads_ tersebut. 
_Standard library_ menyediakan fungsi `thread::spawn` sebagai cara buat bikin 
_threads_ baru, dan si `thread::spawn` ini ngeharepin buat langsung dikasih beberapa 
kode yang harus dijalanin seketika (as soon as) pas _thread_ tersebut selesai 
dibikin. Padahal, di kasus yang kita punya, kita pengennya (_want to_) 
ngebikin _threads_ tersebut terus ngebikin mereka buat _menunggu_ (wait) kode-kode 
(tasks) yang bakal kita kirimin nanti (later). Implementasi (_implementation_) dari 
_threads_ bawaan (standard library) ini sama sekali tidak punya opsi (way) buat ngelakuin hal semacam 
itu; jadinya kita harus mengimplementasikannya secara manual (manually).

Kita bakal mengimplementasikan perilaku (_behavior_) ini dengan cara 
memperkenalkan sebuah struktur data baru (new data structure) di antara si 
`ThreadPool` tersebut dan _threads_ ini yang mana struktur data ini bakal bertugas 
mengelola (manage) tingkah laku yang baru ini. Kita bakal namain struktur data ini 
_Worker_ (Pekerja), yang mana merupakan sebuah istilah lazim (common term) yang 
suka dipakai di dalam berbagai implementasi model pemusatan (pooling). Sang `Worker` 
ini tugasnya ngambilin (picks up) kode-kode (tasks) yang harus dijalanin terus 
menjalanin kode-kode itu di dalam _thread_ miliknya.

Coba bayangin (_think of_) kayak orang-orang yang lagi kerja di dalem dapur 
sebuah restoran: para pekerja dapur (workers) ini pada nungguin sampai pesanan (orders) 
datang dari para pelanggan (customers), dan terus mereka jadinya bertanggung jawab 
(_responsible_) buat nerima (taking) pesanan-pesanan tersebut terus menuhin (filling) 
pesanan itu (masak makanannya).

Ketimbang nyimpen _vector_ yang isinya sekumpulan _instances_ `JoinHandle<()>` di 
dalam _thread pool_ ini, kita sebaliknya bakal nyimpen *instances* dari struct 
`Worker`. Masing-masing `Worker` ini bakal nge-*store* (menyimpan) sebuah instance 
`JoinHandle<()>` tunggal. Kemudian kita bakal mengimplementasikan sebuah method pada 
si `Worker` ini yang mana method itu nerima sebuah _closure_ berisi kode yang harus 
dijalanin terus method itu bakal ngirimin _closure_ itu ke _thread_ yang emang udah lagi pada nyala (already 
running) supaya bisa tereksekusi. Kita juga bakal ngasih setiap `Worker` ini 
sebuah `id` (identifikasi/identitas) supaya kita gampang mbedain di antara berbagai macam _instances_ `Worker` 
yang ada di dalam _pool_ tersebut saat kita lagi nyatet log (logging) atau *debugging*.

Berikut ini adalah gambaran proses baru yang bakal berlangsung (happen) pas 
kita ngebikin sebuah `ThreadPool`. Kita bakal mengimplementasikan kode yang tugasnya 
ngirimin si _closure_ tersebut ke _thread_ itu setelah kita selesai nge-*setup* si `Worker` 
ini memakai cara di bawah ini:

1. Definisikan sebuah struct `Worker` yang menampung (holds) sebuah `id` dan 
   sebuah `JoinHandle<()>`.
2. Ubah `ThreadPool` supaya dia itu sekarang malah nampung sebuah _vector_ berisi 
   _instances_ dari `Worker`.
3. Definisikan sebuah fungsi `Worker::new` yang nerima sebuah angka (number) buat 
   jadi `id` terus dia ngembaliin sebuah instance `Worker` yang nampung si `id` 
   itu beserta sebuah _thread_ baru yang ditetaskan (spawned) memakai sebuah 
   _closure_ yang kosong.
4. Di dalam `ThreadPool::new`, pakailah angka penghitung (_counter_) yang asalnya 
   dari _loop_ `for` itu buat di-*generate* (dijadiin) sebuah `id`, bikin sebuah 
   `Worker` baru pakai `id` tadi, terus masukin dan simpan si `Worker` baru itu ke 
   dalam _vector_-nya.

Kalau kita ngerasa pengen nyari tantangan, coba deh kerjain sendiri perubahan-perubahan 
ini (implementing these changes on your own) sebelum kita ngelihat ke kodenya di 
Listing 21-15.

Udah siap (Ready)? Ini dia Listing 21-15 yang berisi salah satu cara buat ngebikin (make) 
serangkaian modifikasi-modifikasi sebelumnya.

<Listing number="21-15" file-name="src/lib.rs" caption="Memodifikasi `ThreadPool` supaya dia menampung instances `Worker` ketimbang menampung threads secara langsung">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Kita udah mengganti nama field di `ThreadPool` dari asalnya `threads` menjadi 
`workers` karena emang sekarang dia jadinya malah nyimpen *instances* `Worker` 
ketimbang *instances* dari `JoinHandle<()>`. Kita pakai angka hitungan 
(counter) dari *loop* `for` tersebut sebagai argumen buat `Worker::new`, lalu 
menyimpan (store) tiap `Worker` yang baru dibikin ke dalam *vector* yang namanya `workers`.

Kode-kode luar (external code, seperti server kita yang ada di _src/main.rs_) tidak perlu 
tahu detail-detail spesifik implementasinya sehubungan sama pemakaian struct `Worker` 
di dalam sebuah `ThreadPool`, makanya kita membiarkan struct `Worker` dan fungsi 
`new`-nya itu bernilai (sifatnya) _private_. Fungsi `Worker::new` tersebut memakai 
`id` yang udah kita kasih terus menyimpan sebuah instance dari `JoinHandle<()>` yang mana 
instance ini dibikin dengan cara menelurkan sebuah _thread_ baru memakai _closure_ kosong.

> Catatan: Kalau sistem operasinya (OS) tidak mampu buat membikin sebuah _thread_ 
> gara-gara kekurangan sumber daya sistem (aren't enough system resources), fungsi 
> `thread::spawn` itu jadinya bakal meledak (_panic_). Hal ini bakal ngebikin seluruh server 
> kita jadi ikutan panik, sekalipun pembuatan dari beberapa _threads_ yang lain itu 
> sebenarnya berhasil dengan lancar (might succeed). Demi urusan kemudahan buat 
> dipelajari (simplicity's sake), membiarkan kelakuan ini terjadi itu sah-sah 
> saja kok (is fine), tapi kalau di kasus implementasi *thread pool* tipe tingkat produksi 
> (_production_), kita bakal jauh lebih direkomendasiin (likely want to) buat memakai 
> [`std::thread::Builder`][builder] barengan sama method 
> [`spawn`][builder-spawn]-nya karena method tersebut nge-return (ngembaliin) tipe `Result` 
> sebagai gantinya.

Kode kita ini bakal bisa di-compile dan bakal berhasil menyimpan jumlah dari 
instances `Worker` sebanyak yang udah kita spesifikasikan sebagai argumen 
waktu manggil `ThreadPool::new`. Tapi kita ini _masih_ juga belum memproses 
(processing) _closure_ yang kita dapetin dari pemanggilan `execute` ya. Mari kita ngelihat 
gimana caranya supaya kita bisa ngerjain langkah yang itu sekarang.

#### Mengirim Requests ke Dalem Threads Melalui Channels (Saluran Komunikasi)

Permasalahan (_problem_) berikutnya yang harus segera kita tangani adalah fakta kalau _closures_ yang dikasihin ke 
`thread::spawn` itu bener-bener nyatanya tidak berbuat apa-apa (do absolutely nothing). Saat ini, 
kita ngedapetin _closure_ yang mana pengen kita jalanin tersebut (execute) lewat method `execute`. 
Tapi kita ini perlu bisa ngasih si fungsi `thread::spawn` tadi sebuah _closure_ untuk dijalankan 
ketika (_when_) kita lagi repot-repotnya membikin setiap `Worker` tersebut saat fase-fase (during) 
pembentukan (creation) dari `ThreadPool` itu.

Kita pengennya supaya struct-struct `Worker` yang baru aja kita bikin ini bisa ngambilin 
(_fetch_) kode yang mau mereka jalanin tersebut yang dapetnya dari sebuah antrean (queue) yang 
ditampung (held) di dalam `ThreadPool` terus ngirimin kode (task) tersebut ke _thread_ 
miliknya buat dijalankan.

Saluran komunikasi (_channels_) yang sempat kita pelajarin di Bab 16—sebuah 
cara yang simpel buat berkomunikasi di antara dua buah _threads_—itu bakal jadi pilihan 
(candidate) yang luar biasa pas sekali (_perfect_) buat menangani skenario (use case) 
ini. Kita bakal memakai sebuah _channel_ supaya dia bisa bertindak (function) sebagai antrean 
pekerjaan (_queue of jobs_) tersebut, dan method `execute` bakal mengirimkan (_send_) 
sebuah pekerjaan (_job_) dari dalam `ThreadPool` menuju *instances* `Worker`, yang 
mana kemudian bakal ngirimin si _job_ tersebut menuju _thread_ miliknya. Ini 
dia rancangannya (_plan_):

1. `ThreadPool` bakal membikin (create) sebuah _channel_ (saluran) dan berpegang erat (hold on to) pada 
   bagian ujung pengirimnya (sender).
2. Masing-masing `Worker` bakal berpegangan (hold on to) pada bagian penerimanya (receiver).
3. Kita bakal membikin sebuah struct `Job` baru yang mana tugasnya buat 
   nampung (hold) _closures_ yang pengen kita kirimkan masuk menyusuri (_down_) si _channel_ tersebut.
4. Method `execute` bakal mengirim (send) si _job_ (pekerjaan) yang mau dia 
   eksekusi (execute) tersebut melalui si *sender* (pengirim) ini.
5. Di dalam *thread* masing-masing, si `Worker` ini bakal muter berulang-ulang (_loop over_) menanyai 
   _receiver_-nya (penerimanya) lalu mengeksekusi _closures_ yang asalnya dari semua 
   _jobs_ apa pun yang mana dia terima (receive).

Mari kita mulai (_start by_) dengan ngebikin sebuah _channel_ di dalam `ThreadPool::new` 
dan menyimpan (holding) si pengirim (sender) itu di dalam instance 
`ThreadPool` kita ini, persis kayak apa yang ditunjukin di Listing 21-16. Struct `Job` 
ini sendiri belum nampung benda apa pun sih buat saat ini tapi si `Job` inilah yang bakal jadi 
tipe dari *item* yang lagi mau kita kirim menyusuri (down) ke dalam *channel* (saluran) ini.

<Listing number="21-16" file-name="src/lib.rs" caption="Memodifikasi `ThreadPool` buat menyimpan sender dari sebuah channel yang mentransmisikan instances `Job`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

Di dalam `ThreadPool::new`, kita membikin _channel_ yang baru terus nyuruh si 
_pool_ tersebut buat menyimpen (hold) ujung si pengirimnya (_sender_). Kode ini 
bakal berhasil sukses di-compile dengan mulus.

Mari kita cobain buat ngoper masuk sebuah ujung penerima (receiver) dari si *channel* ini 
ke masing-masing `Worker` seiringan saat si *thread pool* lagi ngebikin si *channel* 
tersebut. Kita tahu (_know_) kan kalau kita itu pengen makek bagian _receiver_ (penerima) ini 
dari dalam _thread_ yang mana ditetaskan (spawn) oleh tiap *instances* si `Worker` tadi, jadi kita 
bakal memasukkan referensi (reference) parameter si `receiver` ini di dalam _closure_ yang 
lagi kita buat itu. Sayangnya (won't quite), kode yang ada di Listing 21-17 ini ternyata masih belum 
bisa di-compile dengan sukses nih buat saat ini.

<Listing number="21-17" file-name="src/lib.rs" caption="Mengoper receiver ke masing-masing `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Kita udah membuat sedikit perubahan kecil dan gampang (_straightforward_): kita oper si *receiver* 
masuk ke dalam `Worker::new`, terus kita memakai si *receiver* itu di dalam 
_closure_-nya.

Pas kita mencoba buat ngecek kode ini (check this code), kita langsung kejedot sama 
error ini nih:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

Kodenya ini lagi berusaha (trying to) ngoper nilai `receiver` yang cuma satu ini ke 
banyak *instances* `Worker` secara bebarengan. Ini jelas tidak bisa jalan (won't work), seperti 
yang pasti kita masih ingat (_recall_) dari memori kita pas lagi ngebaca Bab 16: bentuk implementasi 
_channel_ yang disediain sama Rust itu formatnya adalah sistem **banyak pengirim (multiple producer), 
tapi cuma satu penerima (single consumer)**. Ini bermakna kalau kita ini tidak bisa lho sekadar nge-_clone_ 
(kloning) si bagian _consumer_ (pengkonsumsi) dari saluran komunikasi ini buat mbetulin kode 
yang lagi eror ini. Lagian, kita emang sebenernya juga tidak mau kok (_don't want to_) buat 
ngirimin pesan yang itu-itu lagi berkali-kali nuju ke bermacam _consumers_ (penerima pesan); 
kita sebaliknya pengen ngirimin satu _list_ panjang isinya pesan ke banyak (_multiple_) *instances* 
`Worker` tapi dengan harapan bahwa masing-masing pesannya itu cuma berhak (gets processed) 
bakal diproses sekali doang secara giliran.

Sebagai tambahan (additionally), tindakan ngambil (taking) satu *job* pekerjaan ngelepasin 
dari *queue* (antrian) saluran tersebut itu emang pastinya bakal mengubah wujud (mutating) 
si `receiver`-nya ini, gara-gara hal ini makanya _threads_ yang ada itu sangat perlu punya sebuah 
mekanisme (way) yang dirasa aman (safe) supaya mereka bisa bagi-bagi (share) dan ngubah 
(modify) isi `receiver` ini secara bebarengan; kalau tidak, kita malah bisa-bisa ngejeblos dapet 
_race conditions_ (perlombaan data) yang bikin program rusak (seperti yang udah dibahas babak 
belur di Bab 16 kemaren).

Ingat balik soal tipe _smart pointers_ (_pointer_ cerdas) yang udah terjamin aman buat di 
dalam *thread* (_thread-safe_) yang barusan kita obrolin di Bab 16 kemaren: buat membagikan 
(_share_) hak kepemilikan (ownership) melintasi (_across_) banyak _threads_ yang berbeda dan secara bebarengan 
ngebolehin para _threads_ tersebut buat ngubah (mutate) nilai datanya bareng-bareng, kita 
sangat butuh pake `Arc<Mutex<T>>`. Tipe `Arc` ini yang bakal ngebolehin kalau banyak *instances* 
`Worker` buat bisa sama-sama ngantongin (own) si _receiver_, dan tipe `Mutex` ini yang bakal 
mastiin ke kita kalau di satu titik waktu tertentu (at a time) itu cuma ada bener-bener 
satu doang dari sekian banyak `Worker` yang berhak ngambil sebuah *job* (kerjaan) 
langsung dari si *receiver* (penerimanya) tersebut. Listing 21-18 di bawah ini mendemonstrasikan perubahan 
macam apa yang wajib kita lakuin ini.

<Listing number="21-18" file-name="src/lib.rs" caption="Membagikan (sharing) isi receiver ini di antara para instances `Worker` tersebut dengan cara makek `Arc` dan `Mutex`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

Di dalam fungsi `ThreadPool::new`, kita ngeletakkin (put) si `receiver` tersebut ke dalam balutan 
sebuah `Arc` dan sebuah `Mutex`. Buat masing-masing `Worker` yang baru dibikin (new), kita meng-_clone_ 
(bikin kloningan) si `Arc` ini tujuannya supaya dia bakal nge-bump (nambah) _reference count_-nya (hitungan referensinya) 
sedemikian rupa sehingga keseluruhan *instances* dari `Worker` ini pada akhirnya bisa 
saling ngebagi-bagi hak kepemilikannya bareng (share ownership) buat si penerima (_receiver_) tersebut.

Dengan berbekal semua perubahan ini, kodenya akhirnya bisa sukses di-compile! Kita udah 
hampir nyampe (getting there) ke tujuan akhir kita ini loh!

#### Mengimplementasikan Method `execute`

Mari kita benar-benar akhirnya mulai ngerjain dan mengimplementasikan method `execute` 
pada `ThreadPool` tersebut secara tuntas. Kita juga bakal ngubah tipe `Job` ini dari asalnya sebuah struct menjadi 
sebuah _type alias_ (alias buat sebuah tipe) aja buat nampung si _trait object_ (objek trait) yang mana 
benar-benar bakal nampung (hold) secara bener tipe dari _closure_ asli yang mana si `execute` ini tadi lagi nerima. 
Kayak yang barusan kelar dibahas di [“Membikin Sinonim Tipe dengan Type Aliases”][creating-type-synonyms-with-type-aliases] 
di dalem Bab 20 kemaren, fitur _type aliases_ ini ngebolehin kita buat mempersingkat (make shorter) 
nama dari tipe-tipe yang kelihatannya sumpek kepanjangan supaya nanti mereka itu jadi jauh lebih 
enak plus lebih gampang (ease) buat dipakek sehari-hari. Coba tengok dan perhatikan Listing 21-19 
ini deh.

<Listing number="21-19" file-name="src/lib.rs" caption="Ngebikin type alias buat si `Job` supaya nampung sebuah `Box` yang aslinya berisi masing-masing closure dan terus ngirimin si job ini menelusuri ke dalem channelnya">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Sehabis kita selesai kelar bikin (_creating_) *instance* dari sebuah `Job` yang baru yang 
berbekalkan `closure` yang kebetulan kita raup (get) dan kita panen dari parameter di fungsi `execute` ini, 
kita akhirnya ngirimin si *job* ini merosot lurus menuju saluran (*channel*) komunikasi nglewatin _sending end_ 
(ujung buat ngirim) si `sender` ini. Di situ kita itu emang manggil `unwrap` pas lagi manggil method `send` buat 
antisipasi seandainya pengirimannya itu entah kenapa jadi gagal (_fails_). Hal macam error ini 
sebenernya emang masih ada peluang kejadian (_might happen_), apalagi kalau misalnya contohnya, 
kita itu nyetop secara paksa (stop) biar semua _threads_ kita ini berhenti melakukan semua pekerjaan dan ngejalanin eksekusinya, 
yang mana artinya si ujung penerimanya (_receiving end_) tersebut udah pasti juga ikutan 
mandeg (stopped) alias udah tidak nerima pesen (_messages_) baru lagi. Sampai dengan menit saat 
ini sih, emang jujurnya kita ini masih belum nyiapin fitur apa pun buat bisa 
berhentiin (stop) segerombolan _threads_ ini dari ngejalanin eksekusi programnya: _threads_ 
yang udah jalan milik kita ini masih bakal terus melenggang jalan narik ngegas pol gas terus beroperasi 
(continue executing) sekuat lama umur (as long as) *pool* milik kita ini juga masih dibiarin buat tetep *exist*. 
Satu-satunya alasan kenapa kita dengan berani masang (use) fitur `unwrap` di situ adalah karena 
berbekal jaminan (_know_) dari diri kita sendiri kalau skenario kemungkinan gagalnya (_failure case_) ini sebenernya emang 
benar-benar secara harfiah tidak bakal pernah terwujud kejadian sama sekali, tapi ya sayangnya si 
_compiler_ ini mana ngerti kalau di kenyataannya kelakuan ini itu tidak bakal pernah berbuat salah macam itu (doesn't know that).

Tapi kita ini juga belum benar-benar beres juga nih kerjaannya! Di dalam bagian _Worker_ itu, si `closure` 
kepunyaan kita ini yang tadinya dioper ke dalem `thread::spawn` kan emang tugas utamanya dia itu _cuma_ 
lagi ngerujuk (meminjam referensi/*references*) doang kan ke si ujung penerima (_receiving end_) milik si saluran itu. 
Padahal yang bener-bener kita harapkan di sini adalah, kita ngebutuhin sekali supaya 
si `closure` ini benar-benar bisa berputar dan berkeliling di dalam *loop* buat selama-lamanya (forever), 
nanyain dan malakin (asking) si ujung penerima *channel* ini mulu nanyain apaan dia ini lagi dapet kerjaan _job_ 
sambil setelahnya dia benar-benar ngejalanin isi dari si pekerjaan (*running the job*) ini langsung abis dia kebetulan berhasil 
ngedapetin satu *job*. Makanya, mari kita lakuin rombakan perubahan barusan yang ada kelihatan nyempil di 
Listing 21-20 tersebut ke dalam dalem fungsi `Worker::new` ini.

<Listing number="21-20" file-name="src/lib.rs" caption="Menerima dan juga sekaligus ngejalanin berbagai rupa jobs di dalam isi dari thread instance `Worker` tersebut">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Di sini, hal pertama yang kita lakuin duluan itu adalah manggil `lock` dari si *receiver* 
tadi tujuannya buat mengakuisisi atau merebut dan narik status si *mutex* ini, dan 
baru abis narik status aman itu barulah kita berani manggil `unwrap` biar nantinya kodenya bisa otomatis njerit panik (panic) 
atas segala bentukan macam rupa kesalahan (_errors_) yang mungkin keluar. Merebut (Acquiring) status kunci 
(lock) itu pada aslinya sangat mungkin berpeluang buat jadi gagal berantakan kalau ternyata di balik bayangan layar, 
kondisi status (state) dari dalem the *mutex*-nya ini udah keburu kejebak berubah menjadi beracun keracunan (poisoned state). 
Kondisi beracun ini bisa terjadi kalau ternyata ada utas (*thread*) yang lainnya yang entah 
salah apa dari mana dia tiba-tiba malahan udah milih panik meledak berantakan (panicked) tapi saat itu dia ini masih ngantongin status 
kuncinya (holding the lock) ini alih-alih melepaskan kunci tersebut (releasing the lock) sebelum matinya. 
Kalau kita lagi apes kejebak situasi (situation) yang semacam begini, maka tindakan manggil method `unwrap` biar si utasan (_thread_) kita 
yang ini juga ikut-ikutan njerit panik ikutan hancur adalah langkah jalur yang emang diakui emang udah bener (_correct action_) 
buat dikerjain. Silakan santai aja luangin bebasin dan rombak sendiri ini semua ngubah (*change*) bentuk `unwrap` ini 
biar jadinya makek format dari `expect` yang disisipin makek pesan galat kesalahan (*error message*) 
yang mungkin agak lebih masuk kerasa ngena bunyinya gampang dimengerti sama kuping (meaningful to you) 
sendiri aja tidak masalah.

Kalau ternyata kita mulus-mulus aja sukses mulus kebagian ngunci gembok (*got the lock*) ke gembok *mutex* ini, 
maka baru di saat itu kita bisa ikutan berani nekat manggil si perintah `recv` biar bisa nangkep nerima (_receive_) satu balok _Job_ (Pekerjaan) dari si *channel* saluran komunikasi pipa tersebut. 
Sebuah sematan paku tempelan `unwrap` yang nangkring di bagian pucuk paling terakhir ini 
bener-bener membantu kita berjalan tegar dan melewati (_moves past_) nerjang segala bentuk halangan segala kelakuan aneh error macam apa aja yang 
kebetulan mungkin aja tetiba nongol dari sini, error model macem ginian bisa aja pada nongol mencuat (_occur_) umpamanya di mana si utas _thread_ 
yang mana lagi sibuk-sibuknya naruh (holding) megangin ujung *sender*-nya ini ternyata tetiba udah keburu modar matikan paksa nutup jalan (_shut down_) ngedahuluin si penerimanya. 
Yang ini sebenernya jalan nalar kerjanya emang nyerempet mirip-mirip (similar) nian loh percis dengan cara 
fungsi metode `send` yang mana bakal pasti berontak ngebalik (returns) pesan `Err` semisal si utas sang *receiver*-nya yang ini malah mati berantakan nutup mendadak dari depan.

Panggilan metode lurus menuju si baris `recv` ini sifatnya membikin laju proses berhentinya kodenya jadi terblokir nunggu (_blocks_), makanya ini 
menimbulkan efek di mana kalau andai kata aja ternyata eh emang belum nongol dateng kerjaan _job_ satu biji doang pun di depan matanya saat itu (_no job yet_), maka di ujung akhir-akhirnya ini sih utas *_thread_ yang lagi kerja saat ini (current thread) jadinya ya mau gimanapun bakal terus bengong nganggur nungguin nge-drop nunggu nunggu anteng (wait) sampe sebuah *job* bener-bener kelar udah nyampe hadir keluar di depannya. Struct si `Mutex<T>` pada intinya inilah si sosok kuncian yang menjamin mastiin pasti (ensures) ke semuanya ke kita kalau emang disisihkan pada sebuah jeda satu waktu yang spesifik 
tertentu (at a time) bener-bener bakal pasti cuma bakal disidang dan diberika akses ke cuma satu doang *_thread_* punya *Worker* (_only one Worker thread_) yang dikasih dan dibolehin ijin buat nyoba manggil-manggil mau *request* nanyain apaan ada *job* (pekerjaan) 
atau gaknya ke *channel* tadi.

Tadaa, selamat loh! *Thread pool* bikinan kita pada detik jaman tulisan ini dibikin sebenernya udah pada posisi sukses jalan ngacir lanjay ngebut dengan mulus (working state)! 
Majuin cobain aja suruh tes dengan nge-running `cargo run` sekalian lu buat dikit _requests_ ngetes jalannya juga ya:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Sukses mulus (Success)! Kita sekarang ngenalin kalau kita emang akhirnya sukses benar-benar punyain sebuah bentuk 
sejati dari sebuah _thread pool_ murni yang mengeksekusi koneksi-koneksi yang singgah dari depan ini 
ngejalanin proses eksekusinya berbarengan asinkron tidak usah gantian satu-satu lgi secara sinkron (_asynchronously_). 
Emang kenyataannya kagak bakal pernah (_never_) ada riwayat kejadian di mana lebih dari jumlah sekian empet (_four_) potong _threads_ ini 
di-lahirkan (created) pas lagi ngejalanin kode di depannya ini. Jaminan ini ngegaransi ke seisi segenap 
bangsa sistem (_our system_) kita ke semuanya tidak bakal pernah sekalipun ikutan panik mabuk jadi kepanasan (_overloaded_) 
bahkan bilamana semisal jikalau aja mesin server kita itu dikeroyok dikirim hujan dideras rentetan gelombang 
panah ngerima *request* super bertubi-tubi seabreg rupa yang pada berjejer dateng masuk (_receives a lot of requests_). 
Andaikata emang bener lho kita sampe sengaja jahil iseng-iseng kita nge-_request_ secara spesifik (make a request) 
ke alamat lintasan jalur (_path_) tujuan kita ini di _/sleep_, maka server kita yang satu ini emang bakal teteupan 
tetep tegak berdiri bisa santai terus dengan gampangnya bisa ngelayanin ngeresponi (serve) panggilan para rentetan requests yang lainnya 
gara-gara dia tinggal nyuruh _thread_ *Worker* lain yang emang dari kemaren emang nganggur buat sekalian ikutan terjun ngerjain bantu manggil fungsi *tasks* tersebut.

> Catatan: Andai kata lu maksa nekat ngebuka *path* rute link URL tujuan yang ditaruh di dalem rute khusus jebakan si _/sleep_ ini di dalem banyakan (multiple) jajaran tumpukan sekian puluhan (multiple) *browser windows* secara bareng-bareng langsung sekaligus jebred dalam waktu sekerdipan mata bebarengan (_simultaneously_), kita mungkina aja kelewatan bingung kepancing ngerasa panik dan nyangka kok kelihatannya dia ngeladenin ngememuat nge-load-nya ganti-gantian dengan selang pelan durasi santai (_time intervals_) per *five-second* (selang *lima deik*) sekali padahal kita udah makek *pool* multithread. Aslinya benar-benar sebagian (_some_) tipe dari program jenis peramban _web browsers_ jaman di luar sekarang (today) emang emang punya perilaku ngeksekusi kelakuan ganjil ngerapihin nge-barisin numpuk antrian deretan dari satu deretan *requests*-nya yang asalnya identik dan modelnya bener-bener punya *endpoint* rute kembar sama persis ganjil secara sengaja satu per satu berurut bergantian (sequentially), ya murni semata-mata itu gara-gara (reasons) sekadar buat tujuan urusan nge-*caching*. Intinya pembatasan kebodohan macem (_limitation_) begini ini mah samsek seutuhnya _tidak_ diakibatin bersumber murni disebabkan (_not caused by_) ulah dari kelakuan kinerja web server yang kita lagi pada bikin ini.

Saat-saat momen sekarang yang ini emang kelihatannya pas rasanya emang waktu jeda yang enak (good time) 
buat minggir mingser ambil napas ngaso sebentar ngecoba ngebayangin nyimak (pause and consider) ngebandingin 
gimanakah jeroannya raut muka model rancangan tatanan bentukan kode-kodean barisan yang pada nangkring mentereng 
sedari di dalem bingkai bingkisan kotak *Listings 21-18, 21-19*, dan ujungnya juga di selipan *21-20* ini 
seumpamanya bentuk mereka itu dibikin pada agak berbeda sedikit andaikata kta beralih ngeganti jalan pikir buat emang lebih 
memilih (using) tatanan asinkron bernama `futures` ketimbang bertahan kukuh cuma memakai fungsional model *closure* doang buat emang 
membedah ngerjain mengeksekusi semua rupa tugas (work) tersebut. Bagian tipe-tipe spesifik manakah (what types) di sana yang pada 
bakal ngalamin ubah ganti baju (_would change_)? Kayak manakah (how) emangnya *signatures* aslinya dari _method_ (_method signatures_)-nya ini 
kudu ikutan berubah drastis diganti-ganti mingser ke sana-kemari, seandainya emang emang perlu perombakan bener-bener (if at all)? Sisi pinggiran belah pecahan (_parts of the code_) manakah sih 
yang bakal tetep dibiarin aman bertahan (stay) anteng kagak perlu ikutan ngalamin diobok-obok perubahan (_the same_)?

Abis panjang lebar berpuas-puas dirimu beres nuntasin masa masa ngulik ngebaca bahan di pelajaran soal tata krama kelakuan dari jenis gubahan struktur kelakuan *loop* dari bentuk model _`while let`_ ini yang kita kulik balik pas ngerampungin bab-bab awal di antara Bab 17 dan sembari selip di tengah-tengahnya Bab 19 pula, otak di kepala dirimu barangkali mulai berbisik kepikiran lari membatin (you might be wondering) kenapa yak pantesan kok kita *nggak* langsung main sabet milih aja buat mutusin secara kilat nekat naruh masang serangkaian urutan kode eksekusi milik utas _thread_ khusus si _Worker_ kita (Worker thread code) yang seolah nampak rupa cantiknya kayak yang sempet ngintip kepajang di dalem pajangan galeri *Listing 21-21*.

<Listing number="21-21" file-name="src/lib.rs" caption="Alternatif rancangan rupa implementasi buat kode eksekusi dari si `Worker::new` yang coba-coba ngandelin `while let`">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Jujur emang barisan rombongan susunan set perabot alat tempur kode-kodean ini mah emang aslinya (this code compiles and runs) pantes sukses bisa di-*compile* lalu berhasil sukses diajak buat *running* nyala-nyala aja aslinya asalkan dipanggil mah tanpa cacat, akan tetapi (_but_) sayangnya kelakuannya dia tidak bakal sanggup membikin lalu menyetorkan membuahkan asilnya tingkah prilaku eksekusi pola ngantri tata perlintasan multi _threading_ yang aslinya sembenernya emang jujur jadi inceran idam-idaman (*desired threading behavior*) sasaran awal milik hajat (_desired_) diri kita: gara-gara ujung-ujungnya ya ntar sebuah *request* yang super lelet lambat (*a slow request*) jalannya bakal cuman ngehasilkan _requests_ yang berentet di antrean lain di belakangnya yang tetep ngadat mandek mandeg antri merana kudu terpaksa harus disuruh _wait_ menunggu dan antri ngurut memanjang barisan kebagian jatah jadwal biar mereka bisa dieksekusi dikelarin dibereskan dan diproses (processed). Perkara alasan dibalik _reason_ kejanggalan ginian ini tergolong ada sedari kelakuan halus _subtle_: bahwa sesungguhnya susunan wujud si struct _Mutex_ ini emang takditakdirkan emang tidak dikasih *public method* (*metode publik*) khusus sengaja dikasih nama panggilan buat fitur nge-*unlock* karena nyatanya status penguasa kunci miliknya kepemilikan jatah buat kuncian rahasia tersebut ini aslinya udah emang *based on* terikat mutlak dikunci ditentuin dari seberapa panjang seberapa bentar siklus riwayat hidup umur *lifetime* umurnya si benda keramat bertipe perlindungan yang menyandang identitas `MutexGuard<T>` yang mana sosok tersebut disisipkan rapi di perut selimut isi *LockResult<MutexGuard<T>>* yang emang udah jadi kodrat nasib jatah tugas buat metode dari fitur bernama *`lock`* buat dia setorkan ngeluarin dia (returns) pas selesai beres dia dipanggil beroperasi jalan. Menjelang masa kompilasi (*at compile time*), sosok mandor tukang sensor pinjeman `borrow checker` inilah yang selanjutnya ganti berjaga bakal dengan telitinya bisa memelototi lalu memberlakukan narapain menegakkan menembakkan _enforce the rule_ maklumat seputar tata kaidah yang ngegarisbawahi kalau setiap butir selongsong _resource_ (sumber daya) yang tadinya ketat diselimuti diborgol ketat (guarded) diawasi pelindungan di belakang benteng pelindung sebuah palang gembok berjenis _Mutex_ itu niscaya haram statusnya diharamkan (_cannot be accessed_) sama sekali dilarang terjamah alias tidak bakal mempan dibolehin buat bisa diutak-atik (diakses) oleh siapapun pun kalau seumpamanya status kita saat di kejadian eksekusi saat itu kondisinya lagi belom emang kita megang lalu punya si *lock* stempel kuncinya ini di tangan (unless we hold the lock). Masalahnya adalah, penerapan (_implementation_) kode yang bergaya ngasal semacam ini berisiko nimbul-nimbulin bahaya efek di mana wujud kuncian (`lock`) dari yang mestinya diserahin kembali ini ternyata masih dipaksa kudu bertahan kepeluk tertahan kepegang erat-erat tertahan di genggaman (_being held_) jauh memakan durasi yang kepanjangan sangat jauh di luar dari maksud jadwal normal target awal selesainya tugas aslinya seandainya (intended if we aren't mindful) andaikata otak pikiran kita belom ngerasa gih hati-hati nyadari seputaran *mindful* ngeh dan perduli buat memandang mikirin masalah masa waktu jeda _lifetime_ (lama waktu) dari _MutexGuard<T>_ tersebut.

Sepetak bentuk kodingan susunan naskah dari balok kode di balikan dalem ruang gubahan *Listing 21-20* yang menumpukan tumpuannya di dalam bentuk susunan gaya pemakaian dari format pemanggilan dari perkenalan lajur baris berupa panggilan *`let job = receiver.lock().unwrap().recv().unwrap();`* benar-benar jalan bekerja karena aslinya dengan membiasakan manggil (_with `let`_), rupa jenis semua ragam dari kumpulan serpihan serentetan aneka serabut serangkaian sosok rentetan sekian harga biji *temporary values* nilai-nilai angka dadakan instan bayangan yang kebetulan aja sementara disisipin (_temporary values_) dipakai sengaja dipakai di daleman seonggok kumpulan bongkahan _expression_ tersebut yang ada bertumpuk mojok mentereng ngendon mangkal terparkir mejeng di perbatasan tapal batas perlintasan di ruas sisi lajur paling _right-hand side_ pojokan sebelah barisan pinggiran sisi seberang belahan sisi arah sebelah kanannya dari batasan silang palang lintasan lambang sama dengan (the equal sign) bakal emang nasibnya dengan sadar seketika cepatnya dalam detik waktu itu juga (immediately dropped) dimusnahin diputus di-drop hilang tak bersisa ditiadakan pas waktu (when) persis saat barisan deretan panggilan milik sang _let statement_ ini nemuin garis ajalnya dan kelar nyampe berakhir tamat ngakhirin garis _ends_ nasib eksekusinya. Beda malang apes nasib halnya, si konstruksi kelakuan si panggilan buat `while let` (dan tak pelak kelakuan yang kembar sama juga melekat pada panggilan buat deretan panggilan di struktur perlintasan buat konstruksi barisan kelakuan *`if let`* dan juga konstruksi yang ngerujuk seputaran kelakuan _`match`_) sifat kodrat aslinya pantang dan sejatinya anti dan juga (does not) tidak pernah sekalipun membuang secara sepihak memusnahkan (*drop*) barang bawaannya beruba wujud barang rakitan nilai-nilai aneka rentetan sosok figur *temporary values* angka naskah serpihan bawaan serba sementaranya (_temporary values_) sisaan tersebut sebelum rentetan eksekusinya ini benar-benar merayap tuntas tiba berjalan sampai ngejejak sukses nutup berhasil (_until the end of_) merambat tiba menapaki penghujung purna tutup gawang palang blok kodenya (_the associated block_). Merujuk narasi skenario paparan kejadian malang perbandingan percontohan yang tergambar pas nongkrong mojok di sela daleman sketsa yang terbingkai rapi di sela *Listing 21-21* ini tadi, sosok batang kuncian yang ditugasin gembok gerbang (`lock`) tetep ajeg kepaksa bakal awet mangkal terkunci (_remains held_) nangkring mengunci selagi sepanjang (*for the duration of*) di sepanjang rentetan durasi berlangsungnya kelakuan waktu (*the call to*) lamanya rentang periode pas fungsi tugas sang  _`job()`_ tersebut lagi ngejalanin rutinitas gawang di dalam masa rutinitasnya buat menunaikan (_call to_) ngejalanin seisi jeroan badan pemanggilan tugas kerjanya tsb., niscaya memunculkan satu hal arti rupa artian yang maknanya adalah emang pada saat detik-detik nasib gembok lagi kehalang nutup rupa begini maka segenap sekompi pasukan komplotan deretan gerombolan *_Worker instances_* yang *other* (para punggawa lainnya) bakal pastinya pada terpojok tak kan sanggup alias buntu dilarang masuk narik _cannot receive jobs_ alias tidak ada satu pun mereka yang bakal sukses nerima narik sisa gilir antrian tugas sisa deretan job sisanya tsb.

[creating-type-synonyms-with-type-aliases]: ch20-03-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]: ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
