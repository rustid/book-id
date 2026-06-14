## Membikin Web Server yang Single-Threaded

Kita bakal mulai dengan ngebikin supaya sebuah web server *single-threaded* 
(satu utas/thread) bisa jalan. Sebelum kita mulai, mari kita lihat ikhtisar 
(overview) kilat soal protokol-protokol yang dilibatin di dalam pembuatan 
web servers. Detail-detail dari protokol ini emang ada di luar dari cakupan 
buku ini, tapi ikhtisar singkat ini bakal ngasih kita informasi yang 
kita butuhkan.

Dua protokol utama yang dilibatin di dalam web servers adalah _Hypertext 
Transfer Protocol_ _(HTTP)_ dan _Transmission Control Protocol_ _(TCP)_. Kedua 
protokol ini adalah protokol _request-response_ (minta dan balas), yang artinya 
sebuah _client_ (klien) ngirim _requests_ dan sebuah _server_ ngedengerin 
(listens to) _requests_ tersebut lalu ngasih sebuah _response_ (respons/balasan) 
ke si _client_ tadi. Konten (isi) dari _requests_ dan _responses_ ini 
didefinisikan sama protokol-protokol tersebut.

TCP itu adalah protokol tingkat lebih rendah (lower-level protocol) yang 
mendeskripsikan detail-detail soal gimana informasi itu nyampe dari satu server 
ke server lainnya tapi dia tidak menentukan secara spesifik apa sebenarnya bentuk 
informasi itu. HTTP ngebangun di atas (builds on top of) TCP dengan cara 
mendefinisikan konten dari _requests_ dan _responses_ tersebut. Secara teknis 
itu mungkin aja buat memakai HTTP pakai protokol selain TCP, tapi di mayoritas 
kasus yang ada, HTTP ngirim datanya lewat TCP. Kita bakal kerja dengan barisan 
_bytes_ mentah (raw bytes) dari _requests_ dan _responses_ TCP dan HTTP ini.

### Mendengarkan Koneksi TCP

Web server kita perlu mendengarkan (listen to) sebuah koneksi TCP, jadi 
itulah bagian pertama yang bakal kita kerjain. _Standard library_ menawarkan 
sebuah modul `std::net` yang membiarkan kita buat ngelakuin ini. Mari kita 
bikin _project_ baru pakai cara yang biasa:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Sekarang masukin kode yang ada di Listing 21-1 ke dalam _src/main.rs_ buat 
memulai. Kode ini bakal dengerin di alamat lokal `127.0.0.1:7878` nyari _streams_ 
(aliran data) TCP yang lagi mau masuk (incoming). Pas dia dapat sebuah _stream_ 
yang masuk, dia bakal mencetak `Connection established!`.

<Listing number="21-1" file-name="src/main.rs" caption="Mendengarkan streams yang masuk dan mencetak sebuah pesan pas kita nerima sebuah stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Memakai `TcpListener`, kita bisa ngedengerin nyari koneksi-koneksi TCP di 
alamat `127.0.0.1:7878`. Di alamat tersebut, bagian sebelum titik dua itu 
adalah alamat IP yang merepresentasikan komputer kita (ini sama aja di setiap 
komputer dan tidak merepresentasikan spesifik komputernya si penulis ya), 
dan `7878` itu adalah *port*-nya. Kita udah milih port ini karena dua 
alasan: HTTP itu umumnya tidak diterima di port ini, jadi server kita ini 
punya kemungkinan kecil buat berkonflik sama web server lain yang mungkin 
lagi jalan di mesin komputer kita, dan 7878 itu adalah kata _rust_ yang diketik 
di telepon jadul.

Fungsi `bind` (ikat) di dalam skenario ini bekerja kayak fungsi `new` di mana 
dia bakal mengembalikan (return) sebuah instance `TcpListener` yang baru. Fungsi 
ini dikasih nama `bind` karena, di dunia jaringan komputer (networking), nyambung 
ke sebuah port buat mulai mendengarkan ke sana itu dikenal dengan istilah 
“binding to a port” (ngikat ke sebuah port).

Fungsi `bind` ini mengembalikan sebuah `Result<T, E>`, yang mana mengindikasikan 
kalau proses _binding_ ini mungkin aja gagal (fail). Misalnya, kalau kita 
ngejalanin dua instance dari program kita sehingga ada dua program yang dengerin 
di port yang sama persis. Karena kita ini lagi nulis sebuah server super dasar 
(basic) cuma buat tujuan pembelajaran aja, kita tidak bakal ambil pusing buat 
menangani (_handling_) error-error semacam ini; sebaliknya, kita bakal memakai 
`unwrap` buat ngehentiin programnya kalau error-error ini emang kejadian.

Method `incoming` pada `TcpListener` mengembalikan sebuah _iterator_ yang ngasih 
kita serangkaian _streams_ (lebih spesifiknya, _streams_ dari tipe `TcpStream`). 
Sebuah _stream_ tunggal itu merepresentasikan sebuah koneksi terbuka 
(open connection) antara si _client_ sama si _server_. Sebuah _connection_ 
(koneksi) itu adalah nama buat proses pemanggilan _request_ dan _response_ secara 
utuh di mana si _client_ nyambung ke _server_-nya, si _server_ ngehasilin sebuah 
_response_, lalu si _server_ menutup koneksi tersebut. Makanya, kita bakal 
membaca (read) dari `TcpStream` ini buat tahu apa yang dikirim sama si 
_client_ dan lalu menulis (_write_) _response_ kita ke _stream_ tersebut buat ngirim 
datanya kembali ke si _client_. Secara umum, _loop_ (perulangan) `for` ini bakal 
memproses setiap koneksi secara bergantian dan menghasilkan serangkaian _streams_ 
buat kita tanganin.

Buat sekarang, cara penanganan kita terhadap _stream_ ini adalah dengan memanggil 
`unwrap` buat menghentikan (terminate) program kita kalau ternyata _stream_ 
tersebut punya error apa pun; kalau tidak ada error sama sekali, programnya bakal 
mencetak sebuah pesan. Kita bakal nambahin fungsionalitas yang lebih buat kasus di 
mana program sukses (success case) di listing berikutnya. Alasan kenapa kita mungkin 
nerima error dari method `incoming` pas seorang _client_ nyambung ke server adalah 
karena kita itu sebenarnya bukan beriterasi melewati _koneksi-koneksi_ (connections). 
Sebaliknya, kita itu lagi beriterasi ngelewatin _percobaan-percobaan koneksi_ (connection 
attempts). Koneksinya bisa aja tidak sukses karena banyak alasan, yang mana kebanyakan 
dari alasan itu spesifik sama sistem operasi (operating system specific) masing-masing. 
Misalnya, banyak sistem operasi yang punya batas seberapa banyak jumlah koneksi 
terbuka simultan (berbarengan) yang bisa mereka dukung; percobaan koneksi baru yang 
ngelebihi jumlah tersebut bakal ngehasilin error sampai ada beberapa koneksi yang 
udah kebuka tadi itu pada ditutupin dulu.

Mari kita cobain buat ngejalanin kode ini! Panggil `cargo run` di terminal dan lalu 
buka (_load_) _127.0.0.1:7878_ di sebuah web browser. Web browser-nya seharusnya 
nampilin pesan error kayak “Connection reset” karena emang si server-nya saat ini 
belum ngirim balik data apa pun. Tapi pas kita ngelihat ke terminal kita, kita 
harusnya bisa ngelihat beberapa pesan yang tadi dicetak pas si browser ini nyambung 
ke servernya!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Kadang-kadang kita bakal ngelihat banyak pesan yang dicetak cuma buat satu _request_ 
dari browser; alasannya mungkin adalah karena si browser itu lagi ngebikin _request_ 
buat halaman utamanya sekaligus ngebikin _request_ juga buat _resources_ (sumber daya) 
lainnya, kayak misalnya *icon* _favicon.ico_ yang suka muncul di _tab_ browser itu.

Bisa juga karena si browser ini lagi mencoba buat nyambung ke server berkali-kali 
karena si server tidak ngasih respons data apa-apa. Saat `stream` keluar dari 
_scope_ dan di-_drop_ (dibuang) di akhir perulangannya, koneksinya secara otomatis 
ditutup (closed) sebagai bagian dari implementasi dari method `drop` tersebut. Browser 
kadang-kadang nanganin koneksi yang ditutup ini dengan cara mencoba ulang (retrying), 
karena ya mungkin masalahnya itu cuma sementara.

Browser juga kadang-kadang ngebuka koneksi yang banyak ke sebuah server tanpa 
ngirim permintaan apa-apa, jadi kalau nanti mereka *memang* ngirim _request_, 
_request_-nya itu bisa kejadian lebih cepet. Pas ini kejadian, server kita 
bakal bisa ngelihat koneksi tersebut, terlepas dari apakah ada _request_ apa 
enggak yang dikirim liwat koneksi itu. Versi-versi dari browser berbasis Chrome 
misalnya banyak yang ngelakuin ini; kita bisa menonaktifkan optimasi ini dengan cara 
memakai mode _private browsing_ (samaran) atau dengan memakai web browser yang beda.

Faktor yang penting adalah kita udah berhasil dapetin sebuah pegangan (_handle_) ke 
sebuah koneksi TCP!

Inget ya buat ngestop (stop) programnya dengan cara neken <kbd>ctrl</kbd>-<kbd>C</kbd> 
pas kita udah selesai ngejalanin suatu versi kode tertentu. Terus nyalain ulang programnya 
dengan cara manggil perintah `cargo run` setiap kali habis ngebikin rangkaian perubahan 
kode (code changes) buat mastiin kalau kita emang ngejalanin kodenya yang paling baru.

### Membaca Request (Permintaan)

Mari kita implementasikan fungsionalitas buat membaca _request_ yang asalnya dari browser! 
Buat misahin urusan (concerns) dari yang awalnya dapet koneksi terlebih dahulu lalu setelah itu 
baru ngambil beberapa tindakan tertentu sama koneksi tersebut, kita bakal bikin sebuah 
fungsi baru yang khusus buat memproses koneksi (processing connections). Di dalam fungsi 
`handle_connection` yang baru ini, kita bakal membaca data yang asalnya dari TCP _stream_ 
tersebut dan lalu mencetaknya supaya kita bisa ngelihat data apa yang lagi dikirim 
sama si browser. Ubah kodenya supaya kelihatan kayak yang ada di Listing 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Membaca dari `TcpStream` dan mencetak data tersebut">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Kita ngebawa (bring) `std::io::prelude` dan `std::io::BufReader` ke dalam _scope_ 
buat dapetin akses ke trait-trait dan tipe-tipe yang membiarkan kita buat membaca 
dari dan nulis ke _stream_ tersebut. Di dalam _loop_ `for` yang ada di fungsi 
`main`, ketimbang kita sekadar nyetak pesan yang bilang kalau kita udah dapat koneksi, 
sekarang kita memanggil fungsi `handle_connection` yang baru lalu mengoper 
`stream` tersebut ke dalamnya.

Di dalam fungsi `handle_connection`, kita ngebikin sebuah instance `BufReader` 
yang membungkus referensi ke si `stream` tersebut. `BufReader` ini nambahin fitur 
_buffering_ (penyangga) dengan cara mengatur (managing) pemanggilan-pemanggilan ke 
method-method dari trait `std::io::Read` secara otomatis buat kita.

Kita ngebikin sebuah variabel bernama `http_request` buat ngumpulin (collect) baris-baris 
dari _request_ yang dikirim sama si browser ke server kita. Kita mengindikasikan kalau kita 
mau ngumpulin baris-baris tersebut ke dalam sebuah _vector_ dengan cara nambahin anotasi tipe 
`Vec<_>`.

`BufReader` mengimplementasikan trait `std::io::BufRead`, yang mana menyediakan method 
`lines`. Method `lines` ini mengembalikan sebuah iterator dari tipe 
`Result<String, std::io::Error>` dengan cara ngebelah-belah (splitting) aliran datanya 
(stream of data) setiap kali dia ngelihat sebuah byte `newline` (baris baru). Buat 
bisa dapat tiap `String`-nya, kita memakai `map` dan `unwrap` pada masing-masing 
`Result`. Tipe `Result` ini mungkin aja berisi sebuah error kalau datanya ternyata 
bukan UTF-8 yang valid atau kalau sekiranya ada masalah pas membaca dari _stream_ tersebut. 
Sekali lagi, di program level _production_ kita seharusnya nanganin error-error kayak 
gini dengan jauh lebih cakep (_gracefully_), tapi kita lebih milih buat ngestop aja 
programnya di kasus error ini demi menyederhanakan contoh.

Si browser nandain akhir (end) dari sebuah _request_ HTTP dengan cara ngirimin dua 
karakter baris baru (_newline_) secara berurutan (in a row), jadi supaya kita bisa dapet 
satu _request_ dari si _stream_, kita ngambil barisnya terus-terusan sampai kita 
dapet baris yang mana itu adalah string yang kosong. Setelah kita ngumpulin semua barisnya ke 
dalam vector, kita nyetak mereka pakai *pretty debug formatting* (format _debug_ 
cantik yang gampang dibaca) supaya kita bisa lihat sendiri instruksi-instruksi apa aja yang lagi 
dikirim sama si web browser ke server kita.

Mari kita cobain kode ini! Jalanin programnya dan coba lakuin _request_ (ngunjungin alamat) 
pakai web browser lagi. Perhatikan kalau kita bakal tetep dapet halaman error di web 
browsernya ya, tapi sekarang _output_ dari program kita yang ada di dalam terminal 
bakal kelihatan kira-kira kayak gini:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Tergantung dari browser apa yang kita pake, kita mungkin dapat *output* yang agak 
sedikit beda. Sekarang setelah kita udah nyetak isi dari _request_ datanya, kita 
bisa paham kan alasan kenapa kita dapat koneksi berkali-kali dari satu _request_ web 
browser kalau kita ngelihat _path_ (jalur) yang ada setelah kata `GET` di baris paling 
pertama dari _request_ tersebut. Kalau koneksi-koneksi yang berulang (repeated connections) 
itu semuanya lagi nge-_request_ _/_, kita jadi tahu kalau browser-nya itu lagi nyoba buat ngambil 
(fetch) _/_ berkali-kali karena dia tidak dapat respons apa-apa dari program kita.

Mari kita bedah dan perinci (break down) data _request_ ini supaya kita benar-benar ngerti 
apa yang lagi diminta (asking of) sama si browser dari program kita ini.

### Ngelihat Lebih Dekat pada Request HTTP

HTTP itu adalah protokol yang berbasis teks (text-based protocol), dan sebuah _request_ 
(permintaan) itu ngebentuk format kayak gini:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Baris yang paling pertama itu disebut dengan _request line_ (baris permintaan) yang 
menampung informasi tentang apa yang lagi di-_request_ sama si _client_. Bagian pertama 
dari si _request line_ ini mengindikasikan _method_ (metode) apa yang lagi dipakai, kayak 
misalnya `GET` atau `POST`, yang mana mendeskripsikan gimana caranya si _client_ ini 
melakukan _request_ tersebut. _Client_ kita (yakni si web browser tadi) itu memakai sebuah _request_ `GET`, yang 
berarti dia itu lagi minta dikasihin suatu informasi.

Bagian yang selanjutnya di _request line_ tersebut adalah _/_, yang mana mengindikasikan 
_uniform resource identifier_ _(URI)_ yang lagi di-_request_ sama _client_ tersebut: 
sebuah URI itu tuh hampir sekali, tapi tidak sepenuhnya sama persis, dengan sebuah 
_uniform resource locator_ _(URL)_. Perbedaan antara URI dan URL ini tidaklah penting buat 
tujuan pembelajaran kita di bab ini, tapi spesifikasi HTTP (HTTP spec) memakai istilah 
_URI_, jadi kita bisa dalam hati aja men-substitusikan (menggantikan) _URL_ jadi _URI_ di sini.

Bagian terakhirnya adalah versi HTTP yang lagi dipakai sama si _client_, dan kemudian si 
_request line_ tersebut diakhiri pakai urutan CRLF (CRLF sequence). (CRLF singkatan dari 
_carriage return_ dan _line feed_, yang mana ini adalah istilah yang asalnya dari jaman 
mesin tik lho!) Urutan CRLF ini juga bisa ditulis sebagai `\r\n`, di mana `\r` itu adalah 
si _carriage return_ dan `\n` itu adalah si _line feed_ (baris baru). _Urutan CRLF_ ini 
memisahkan bagian _request line_ dari sisa (rest) data _request_ yang lainnya. 
Perhatikan kalau pas CRLF ini dicetak, kita ngelihatnya kayak dimulainya baris baru kan ketimbang 
tulisan `\r\n`.

Ngelihat ke data _request line_ yang kita dapet dari hasil ngejalanin program kita sejauh 
ini, kita ngelihat kalau `GET` itu adalah method-nya, _/_ itu adalah _request URI_-nya, 
dan `HTTP/1.1` itu adalah versinya.

Setelah baris pertama (_request line_) tadi, baris-baris tersisa yang diawali dengan kata 
`Host:` dan seterusnya itu semuanya adalah bagian _headers_. _Requests_ tipe `GET` itu 
sama sekali tidak punya _body_ (badan/isi pesan).

Coba deh bikin sebuah _request_ (permintaan) dari browser yang beda atau coba minta sebuah 
alamat yang berbeda, kayak misalnya _127.0.0.1:7878/test_, dan perhatikan aja gimana isi 
dari data _request_-nya itu berubah.

Nah, sekarang karena kita udah paham apa yang sebenarnya lagi diminta sama si browser, 
mari kita coba buat ngirim balik beberapa data!

### Nulis Sebuah Response (Respons)

Kita bakal mengimplementasikan cara mengirim data sebagai sebuah _response_ 
(respons/balasan) terhadap _request_ yang dibikin _client_ (client request). 
_Responses_ itu punya bentuk format kayak gini:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Baris pertama itu disebut _status line_ (baris status) yang mengandung informasi 
versi HTTP yang dipakai di dalam _response_ ini, sebuah kode status berupa angka (numeric 
status code) yang nge-ringkas (summarizes) apa hasil akhir dari _request_-nya, dan 
juga _reason phrase_ (frasa alasan) yang menyediakan deskripsi teks dari kode status 
tersebut. Setelah urutan CRLF pertama adalah _headers_ (kalau ada), dan diikuti oleh 
satu urutan CRLF lagi, dan barulah kemudian _body_ (isi badan) dari si _response_ 
tersebut.

Berikut ini adalah sebuah contoh _response_ yang memakai versi HTTP 1.1, punya kode 
status 200, beserta _reason phrase_ `OK`, tidak punya _headers_, dan tidak punya 
_body_:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Kode status 200 itu adalah respons standar buat bilang sukses (success response). 
Teks barusan adalah sebuah _response_ HTTP sukses yang ukurannya sekecil mungkin. 
Mari kita tulis ini ke dalam _stream_ (aliran data) kita sebagai _response_ 
ke _request_ yang sukses! Dari dalam fungsi `handle_connection` tadi, silakan 
hapus kode `println!` yang fungsinya buat nyetak data _request_ tadi dan 
terus ganti pakai kode yang ada di Listing 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Menulis sebuah response HTTP sukses yang imut (tiny) ke dalam stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Baris baru yang pertama mendefinisikan variabel `response` yang bakal menyimpan 
data pesan sukses kita. Terus kita panggil method `as_bytes` pada variabel 
`response` kita ini buat mengkonversi data string tadi jadi kumpulan `bytes`. 
Method `write_all` pada variabel `stream` itu menerima nilai tipe `&[u8]` (array 
slice of bytes) dan dia bakal ngirim _bytes_ tersebut secara langsung nyusurin 
(down) koneksi tersebut. Karena operasi `write_all` ini berpotensi gagal, kita 
memakai `unwrap` pada segala _result_ error kayak sebelumnya. Sekali lagi ya, 
di aplikasi yang rill (real application) kita seharusnya nambahin error 
_handling_ (penanganan error) di sini.

Dengan adanya perubahan-perubahan ini, mari kita jalanin kode kita lalu kita 
bikin sebuah _request_ lewat browser. Kita udah tidak lagi mencetak data 
apa pun ke terminal ya, jadi kita tidak bakal ngelihat *output* apa-apa selain 
*output* dari Cargo. Pas kita memuat (_load_) alamat _127.0.0.1:7878_ di 
sebuah web browser, kita seharusnya ngedapetin halaman putih (blank page) 
ketimbang halaman error. Kita baru aja melakukan *hardcode* (kode manual) 
buat menerima _request_ HTTP lalu mengirimkan sebuah _response_ secara utuh!

### Mengembalikan HTML yang Asli (Real HTML)

Mari kita implementasikan fungsionalitas buat ngembaliin lebih dari sekadar 
halaman kosong (blank page). Silakan bikin sebuah file baru bernama _hello.html_ 
di *directory* utama (_root_) dari _project_ kita, inget ya **bukan** di dalem 
folder _src_. Kita bisa naruh (_input_) kode HTML apa aja yang kita mau kok; Listing 
21-4 nunjukin salah satu kemungkinan isinya.

<Listing number="21-4" file-name="hello.html" caption="Sebuah contoh file HTML sampel buat di-return (dikembalikan) di dalam sebuah response">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Ini adalah sebuah dokumen HTML5 yang sangat minimal yang cuma ada *heading* (judul) 
dan sedikit teks doang. Buat ngembaliin kode ini dari server saat ada _request_ yang 
diterima, kita bakal memodifikasi `handle_connection` seperti yang ditunjukin 
di Listing 21-5 supaya dia ngebaca file HTML tersebut, menambahkannya ke dalam 
_response_ kita sebagai isi dari _body_, lalu mengirimkannya.

<Listing number="21-5" file-name="src/main.rs" caption="Mengirim isi konten (contents) dari _hello.html_ sebagai body (isi pesan) dari response tersebut">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Kita udah menambahkan `fs` ke dalam statement `use` buat membawa _filesystem 
module_ (modul file sistem) kepunyaan _standard library_ masuk ke dalam _scope_. 
Kode buat membaca isi dari sebuah file ke dalam sebuah string seharusnya 
udah kelihatan familier; kita sempat memakainya pas kita lagi membaca isi konten dari 
sebuah file buat project I/O kita balik pas di Listing 12-4.

Selanjutnya, kita memakai `format!` buat menambahkan konten file tersebut 
sebagai _body_ dari _response_ sukses kita tadi. Supaya pasti kalau ini 
adalah _response_ HTTP yang benar-benar valid, kita juga menambahkan *header* `Content-Length` 
yang mana diatur isinya supaya ngepas sama ukuran (size) dari _body_ dari 
_response_ kita, di kasus ini berarti ukurannya sama dengan ukuran file `hello.html`.

Coba jalanin kode ini pakai `cargo run` terus muat (load) _127.0.0.1:7878_ 
di browser kita; kita harusnya bisa ngelihat HTML kita dimuat (rendered)!

Saat ini, kita emang lagi mengabaikan data _request_ yang ada di `http_request` 
dan kita cuma tanpa syarat (unconditionally) mengirimkan kembali isi (contents) 
dari file HTML tersebut. Itu artinya kalau kita mencoba nge-_request_ halaman 
_127.0.0.1:7878/something-else_ (apa-aja-lainnya) di browser kita, kita 
bakal tetep dapet balasan _response_ HTML yang ini-ini juga. Saat ini, server 
kita ini sifatnya sangat terbatas (_very limited_) dan masih belum berbuat apa yang 
mayoritas web server benar-benar lakuin. Kita mau mengkustomisasi _responses_ kita 
supaya bergantung pada si _request_ tersebut lalu cuma mengirimkan balik file HTML 
itu untuk _request_ _/_ yang formasinya bener (_well-formed request_).

### Memvalidasi Request dan Merespons Secara Selektif

Sekarang ini, web server kita ini bakal selalu nge-return (ngembaliin) file HTML 
kita ini tidak peduli apa pun yang diminta sama _client_-nya. Mari kita tambahin 
fungsionalitas buat mengecek apakah browser ini benar-benar lagi nge-_request_ rute _/_ 
sebelum nge-return si file HTML, dan terus dia bakal mengembalikan pesan error 
kalau browser tersebut mencoba minta apa pun yang lainnya. Buat ngelakuin ini, 
kita perlu memodifikasi fungsi `handle_connection`, kayak yang ditunjukin 
di Listing 21-6. Kode yang baru ini bakal mengecek konten dari _request_ yang 
baru diterima tersebut dan membandingkannya (against) terhadap rupa dari 
_request_ `GET` buat *path* (rute) _/_ yang kita ketahui (know), lalu nambahin 
blok `if` dan `else` buat memperlakukan (treat) _requests_ itu dengan cara 
yang berbeda-beda.

<Listing number="21-6" file-name="src/main.rs" caption="Menangani requests buat rute _/_ secara berbeda dari requests yang lain">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Kita emang cuma bakal ngelihat baris pertama (_first line_) doang dari _request_ HTTP-nya, 
jadi ketimbang baca keseluruhan _request_-nya dan dimasukin ke dalam sebuah vector, 
kita cuma manggil method `next` buat dapat item paling pertama (first item) dari sang iterator. 
Method `unwrap` yang pertama ngurusin nilai `Option`-nya dan langsung ngestop program 
kalau si iterator tidak punya item apa-apa. Terus `unwrap` yang kedua menangani 
nilai `Result`-nya yang mana efeknya persis sama kayak `unwrap` yang sempat 
ada di dalam method `map` pas di Listing 21-2.

Berikutnya, kita ngecek nilai `request_line` buat ngebuktiin apakah dia itu sama 
dengan *request line* milik sebuah _request_ `GET` buat *path* _/_. Kalau emang 
sama (it does), blok `if` tersebut bakal mengembalikan konten dari file HTML kita.

Kalau nilai `request_line` itu _tidak sama_ (_not_ equal) dengan `GET` request 
ke *path* _/_, itu artinya kita udah nerima _request_ untuk hal lain. Kita bakal 
nambahin kode ke dalam blok `else` sebentar lagi buat membalas segala macam 
_requests_ lain yang masuk.

Silakan jalankan kode ini sekarang dan terus coba _request_ alamat _127.0.0.1:7878_; 
kita seharusnya dapetin si HTML di dalam `hello.html` tersebut. Kalau kita bikin 
_request_ apa pun yang lainnya, kayak misalnya _127.0.0.1:7878/something-else_, 
kita bakal dapet error koneksi kayak yang kita temui pas ngejalanin kode di Listing 
21-1 dan Listing 21-2.

Sekarang mari kita tambahin kode yang ada di Listing 21-7 ke dalam blok `else` 
tersebut buat mengembalikan sebuah _response_ (balasan) yang mana punya status 
kode 404, yang mengindikasikan (signals) kalau konten buat _request_ tersebut 
tidak ditemukan (not found). Kita juga bakal mengembalikan (return) sebuah 
file HTML buat halaman (page) yang bakal ditampilin (render) ke dalam browser yang 
mana bakal mengindikasikan kepada sang _end user_ (pengguna akhir) apa sebenarnya 
_response_-nya.

<Listing number="21-7" file-name="src/main.rs" caption="Membalas dengan status kode 404 dan sebuah halaman error kalau apa pun selain rute _/_ yang di-request">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Di sini, _response_ kita punya _status line_ (baris status) dengan status kode 
404 dan _reason phrase_ `NOT FOUND`. Isi (_body_) dari _response_ ini bakal 
menjadi konten HTML dari sebuah file bernama _404.html_. Kita perlu ngebikin file 
_404.html_ ini di sebelah (next to) file _hello.html_ kita tadi buat halaman 
error ini; sekali lagi ya, silakan pake HTML apa aja yang kita mau secara 
bebas (feel free to use), atau kita juga bisa pakai HTML contoh (example HTML) yang ada 
di Listing 21-8.

<Listing number="21-8" file-name="404.html" caption="Isi sampel konten buat halaman yang bakal dikirim balik buat semua response 404">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Dengan perubahan-perubahan ini, jalanin server kita lagi. Nge-_request_ _127.0.0.1:7878_ 
seharusnya mengembalikan isi konten dari _hello.html_, dan nyoba _request_ apa pun 
yang lainnya, kayak _127.0.0.1:7878/foo_, seharusnya nge-return halaman HTML error 
dari _404.html_.

### Sentuhan Kecil Refactoring (Merombak Kode)

Saat ini, di blok `if` dan `else` ada sangat banyak kode yang ngulang 
(repetition): mereka berdua sama-sama lagi ngebaca file (_reading files_) lalu 
sama-sama nulis (writing) isi konten file tersebut ke dalam _stream_. Satu-
satunya perbedaan ada di bagian _status line_ dan nama filenya (filename). 
Mari kita bikin kode ini jadi jauh lebih ringkas (concise) dengan menarik keluar 
(pulling out) perbedaannya ke dalem baris-baris `if` dan `else` yang dipisah 
yang mana bakal nge-*assign* (ngasih) nilai-nilai dari status line dan nama filenya 
ke dalam variabel; kita lalu bisa memakai variabel-variabel tersebut secara mutlak 
(unconditionally) di dalam kodenya buat membaca file dan nulis (write) balasannya. 
Listing 21-9 nunjukin kode hasil (resultant code) sesudah kita nggantiin (replacing) 
blok `if` dan `else` yang sangat besar (large) tadi.

<Listing number="21-9" file-name="src/main.rs" caption="Merombak ulang (refactoring) blok `if` dan `else` supaya cuma berisi kode yang emang beda di antara kedua kasus tersebut">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Sekarang blok `if` dan `else` tersebut cuma bakal nge-return nilai yang 
paling pas buat _status line_ dan nama file (filename) di dalam sebuah _tuple_; 
terus kita memakai *destructuring* (pemecahan) buat nge-*assign* kedua nilai 
ini ke variabel `status_line` dan `filename` memakai _pattern_ (pola) di 
dalam statement `let`, seperti yang udah di bahas di Bab 19.

Kode yang tadinya menduplikasi (duplicated) sekarang udah ditaruh di luar 
(outside) blok `if` dan `else` lalu dia memakai variabel `status_line` dan 
`filename` tadi. Hal ini ngebikin perbedaan di antara kedua kasus ini (two cases) 
jadi jauh lebih gampang buat dilihat, dan ini juga berarti kita cuma perlu 
ngubah kode (update the code) di satu tempat aja kalau kita pengen ngubah cara kerja 
gimana si proses baca file (file reading) dan proses tulis balasan (response writing) ini 
berjalan. Perilaku kode (behavior) di Listing 21-9 ini bakal tetap persis 
sama kayak yang ada di Listing 21-7.

Keren sekali! (Awesome!) Nah sekarang kita udah punya sebuah web server sederhana cuma dalam 
waktu lebih kurang (approximately) 40 baris kode Rust yang mana merespons sebuah _request_ 
tertentu dengan halaman berisi konten (page of content) dan membalas (responds to) semua _requests_ lainnya 
pakai _response_ 404.

Saat ini, server kita masih jalan di dalem _single thread_ (satu utas/utas tunggal), yang artinya dia 
cuma bisa melayani (serve) satu _request_ dalam satu waktu tertentu (at a time). 
Mari kita telusuri dan menguji (examine) gimana cara kerja kayak gini bisa mendatangkan 
sebuah masalah dengan menyimulasikan beberapa _requests_ yang jalannya pelan (slow requests). 
Terus setelah itu kita bakal benerin itu supaya server kita bisa nanganin (handle) banyak 
_requests_ secara sekaligus (at once).
