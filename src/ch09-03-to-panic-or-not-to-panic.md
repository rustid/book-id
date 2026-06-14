## Kapan Harus `panic!` dan Kapan Nggak

Terus gimana caranya kita mutusin kapan harus manggil `panic!` dan kapan harus 
balikin `Result`? Pas kode _panic_, nggak ada cara buat pulih (recover). Kita 
bisa aja manggil `panic!` buat situasi error apa pun, mau ada cara buat pulih 
atau nggak, tapi kalau gitu kita jadi ngambil keputusan atas nama kode pemanggil 
kalau situasinya emang nggak bisa dipulihin. Pas kita milih buat balikin nilai 
`Result`, kita ngasih opsi ke kode pemanggil. Kode pemanggil bisa milih buat 
nyoba pulih dengan cara yang pas buat situasinya, atau dia bisa mutusin kalau 
nilai `Err` di kasus ini emang nggak bisa dipulihin, jadi dia bisa manggil 
`panic!` dan ngerubah error _recoverable_ kita jadi error _unrecoverable_. 
Makanya, balikin `Result` itu pilihan default yang bagus pas kita lagi 
mendefinisikan fungsi yang mungkin aja gagal.

Di situasi-situasi kayak ngasih contoh, kode prototipe, sama nulis test, lebih 
pantes buat nulis kode yang bakal _panic_ daripada balikin `Result`. Yuk kita 
eksplor alasannya, terus bahas situasi-situasi di mana _compiler_ nggak bisa 
tau kalau kegagalan itu mustahil terjadi, tapi kita sebagai manusia tau. Bab ini 
bakal ditutup pake beberapa panduan umum (guidelines) soal gimana cara mutusin 
apakah harus _panic_ di kode _library_ atau nggak.

### Contoh-contoh, Kode Prototipe, sama Tests

Pas kita lagi nulis contoh buat ngejelasin suatu konsep, masukin kode penanganan 
error yang kuat (robust) malah bisa bikin contohnya jadi kurang jelas. Di dalam 
contoh-contoh, udah dimaklumin kalau pemanggilan ke method kayak `unwrap` yang 
bisa _panic_ itu dimaksudkan sebagai _placeholder_ (tempat pengganti) buat 
gimana kita maunya aplikasi kita nanganin error, yang mana bisa beda-beda 
tergantung dari apa yang lagi dilakuin sama sisa kode kita.

Sama halnya, method `unwrap` sama `expect` itu praktis sekali pas lagi 
_prototyping_ (bikin prototipe), sebelum kita siap mutusin gimana cara nanganin 
error. Mereka ninggalin tanda yang jelas di kode kita buat pas kita udah siap 
bikin program kita jadi lebih kuat (robust).

Kalau pemanggilan method gagal di dalem sebuah test, kita pasti mau seluruh 
test-nya ikutan gagal, biarpun method itu bukan fungsionalitas yang lagi dites. 
Karena `panic!` adalah cara sebuah test ditandain gagal, manggil `unwrap` atau 
`expect` adalah hal yang bener-bener seharusnya dilakuin.

### Kasus di mana Kita Punya Lebih Banyak Informasi daripada Compiler

Bakal pantes juga buat manggil `expect` pas kita punya logika lain yang mastiin 
kalau `Result`-nya bakal punya nilai `Ok`, tapi logikanya bukan sesuatu yang 
dipahamin sama _compiler_. Kita bakal tetep punya nilai `Result` yang harus 
ditanganin: operasi apa pun yang lagi kita panggil secara umum tetep punya 
kemungkinan buat gagal, biarpun itu mustahil terjadi secara logika di situasi 
spesifik kita. Kalau kita bisa mastiin dengan nge-cek kodenya secara manual 
kalau kita nggak bakal pernah dapet varian `Err`, itu sah-sah aja buat manggil 
`expect` dan dokumentasiin alesan kenapa kita yakin kita nggak bakal pernah dapet 
varian `Err` di dalem teks argumennya. Ini contohnya:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Kita bikin instance `IpAddr` dengan nge-_parse_ (mengurai) sebuah _hardcoded_ 
string. Kita bisa liat kalau `127.0.0.1` itu alamat IP yang valid, jadi sah-sah 
aja buat pake `expect` di sini. Tapi, punya _hardcoded string_ yang valid nggak 
ngubah tipe kembalian dari method `parse`: kita tetep dapet nilai `Result`, dan 
_compiler_ tetep bakal nyuruh kita nanganin `Result`-nya seolah-olah varian `Err` 
itu mungkin terjadi karena _compiler_ nggak cukup pinter buat ngeliat kalau 
string ini bakal selalu jadi alamat IP yang valid. Kalau string alamat IP-nya 
dateng dari _user_ bukannya di-_hardcode_ ke program dan makanya _emang_ punya 
kemungkinan gagal, kita pastinya mau nanganin `Result`-nya dengan cara yang 
lebih kuat (robust) sebagai gantinya. Nyebutin asumsi kalau alamat IP ini di-
_hardcode_ bakal ngingetin kita buat ngubah `expect` jadi kode penanganan error 
yang lebih baik kalau, di masa depan, kita harus ngedapetin alamat IP dari 
sumber lain.

### Panduan buat Error Handling

Sangat disaranin buat bikin kode kita _panic_ pas ada kemungkinan kode kita bisa 
berakhir di keadaan yang buruk (_bad state_). Di konteks ini, _bad state_ adalah 
pas ada asumsi, jaminan, kontrak, atau invarian (aturan yang harus selalu benar) 
yang dilanggar, misalnya pas nilai yang nggak valid, nilai yang saling bertentangan, 
atau nilai yang ilang dimasukin ke kode kita—ditambah satu atau lebih dari hal-
hal berikut:

- _Bad state_ itu adalah sesuatu yang nggak diduga-duga, beda sama sesuatu yang 
  kemungkinan bakal sesekali kejadian, kayak _user_ masukin data pake format 
  yang salah.
- Kode kita setelah titik ini harus ngandelin kalau dia nggak lagi di _bad state_ 
  itu, bukannya nge-cek masalah itu di tiap langkahnya.
- Nggak ada cara yang bagus buat nge-_encode_ (nyimpen) informasi ini ke tipe-
  tipe yang kita pake. Kita bakal bahas contoh dari apa yang kita maksud di 
  [“Meng-encode Keadaan dan Perilaku sebagai Tipe”][encoding] di Bab 18.

Kalau seseorang manggil kode kita terus ngasih nilai-nilai yang nggak masuk akal, 
paling bener sih balikin sebuah error kalau bisa biar _user_ dari _library_-nya 
bisa mutusin apa yang mau mereka lakuin di kasus itu. Tapi, di kasus di mana 
lanjut jalan bisa ngebahayain keamanan atau ngerusak, pilihan terbaik mungkin 
adalah manggil `panic!` terus ngingetin orang yang pake _library_ kita soal _bug_ 
di kode mereka biar mereka bisa benerin pas masa _development_ (pengembangan). 
Sama juga, `panic!` itu sering kali pas kalau kita lagi manggil kode eksternal 
yang ada di luar kendali kita terus dia balikin _invalid state_ yang nggak bisa 
kita benerin.

Tapi, pas kegagalan emang udah di-ekspektasi, lebih pantes buat balikin sebuah 
`Result` daripada manggil `panic!`. Contohnya kayak sebuah _parser_ yang dikasih 
data yang cacat formatnya (malformed) atau sebuah HTTP request yang balikin 
status yang ngindikasikan kalau kita udah kena _rate limit_ (batas batas frekuensi 
permintaan). Di kasus-kasus ini, balikin `Result` nunjukin kalau kegagalan 
adalah kemungkinan yang di-ekspektasi yang harus diputusin sama kode pemanggil 
gimana cara nanganinnya.

Pas kode kita ngejalanin operasi yang bisa ngebahayain _user_ kalau dipanggil 
pake nilai-nilai yang nggak valid, kode kita harusnya nge-verifikasi kalau nilai-
nilainya valid dulu terus _panic_ kalau ternyata nggak valid. Ini sebagian besar 
karena alasan keamanan (_safety_): nyoba beroperasi pada data yang nggak valid 
bisa nge-ekspos kode kita ke celah keamanan (vulnerabilities). Ini alasan utama 
kenapa _standard library_ bakal manggil `panic!` kalau kita nyoba akses memori 
di luar batas (out-of-bounds): nyoba akses memori yang bukan milik struktur 
data saat ini adalah masalah keamanan yang umum sekali. Fungsi-fungsi sering 
kali punya _contracts_ (kontrak): perilaku mereka cuma dijamin kalau inputnya 
menuhin syarat tertentu. _Panic_ pas kontrak dilanggar itu masuk akal karena 
pelanggaran kontrak selalu ngindikasikan ada _bug_ di pihak pemanggil (_caller-side_), 
dan ini bukan jenis error yang kita mau kode pemanggil harus tangani secara 
eksplisit. Malah, nggak ada cara yang masuk akal buat kode pemanggil buat bisa 
pulih; si _programmer_ yang bikin kode pemanggil harus benerin kodenya. Kontrak 
buat sebuah fungsi, terutama pas ada pelanggaran yang bakal nyebabin _panic_, 
harusnya dijelasin di dokumentasi API buat fungsi itu.

Tapi, punya sangat banyak pengecekan error di semua fungsi kita bakal panjang 
sekali (verbose) dan nyebelin. Untungnya, kita bisa pake sistem tipe (_type system_) 
Rust (dan karena itu dapet pengecekan tipe yang dilakuin sama _compiler_) buat 
ngelakuin banyak pengecekan buat kita. Kalau fungsi kita nerima tipe tertentu 
sebagai parameternya, kita bisa lanjut sama logika kode kita dengan tenang 
karena tau _compiler_ udah mastiin kalau kita punya nilai yang valid. Misalnya, 
kalau kita nerima sebuah tipe bukannya sebuah `Option`, program kita berharap 
dapet _sesuatu_ bukannya _nggak ada apa-apa_. Kode kita terus nggak perlu 
nanganin dua kasus buat varian `Some` sama `None`: dia cuma bakal punya satu 
kasus di mana dia pasti dapet sebuah nilai. Kode yang nyoba masukin _nggak ada 
apa-apa_ ke fungsi kita bahkan nggak bakal bisa di-compile, jadi fungsi kita 
nggak perlu nge-cek kasus itu pas _runtime_. Contoh lain adalah pake tipe integer 
_unsigned_ (nggak ada tanda minus) kayak `u32`, yang mastiin kalau parameternya 
nggak bakal pernah negatif.

### Bikin Tipe Kustom Buat Validasi

Yuk kita bawa ide pake sistem tipe Rust buat mastiin kita punya nilai yang valid 
satu langkah lebih jauh terus liat cara bikin tipe kustom buat validasi. Inget 
game tebak angka di Bab 2 di mana kode kita minta _user_ buat nebak angka antara 
1 sampe 100. Kita nggak pernah mevalidasi kalau tebakan _user_ bener-bener ada 
di antara angka-angka itu sebelum kita bandingin sama angka rahasia kita; kita 
cuma mevalidasi kalau tebakannya itu positif. Di kasus ini, konsekuensinya nggak 
terlalu parah sih: output “Ketinggian” atau “Kerendahan” kita bakal tetep bener. 
Tapi ini bakal jadi peningkatan yang berguna buat mandu _user_ ke arah tebakan 
yang valid dan punya perilaku yang beda pas _user_ nebak angka di luar rentang 
versus pas _user_ ngetik huruf, misalnya.

Salah satu cara buat ngelakuin ini adalah dengan nge-_parse_ (mengurai) tebakannya 
sebagai sebuah `i32` bukannya cuma `u32` buat ngebolehin angka yang potensial 
negatif, terus nambahin pengecekan apakah angkanya ada di dalem rentang, kayak 
gini:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

Ekspresi `if` nge-cek apakah nilai kita ada di luar rentang, ngasih tau _user_ 
soal masalahnya, terus manggil `continue` buat mulai iterasi `loop` berikutnya 
dan minta tebakan lain. Setelah ekspresi `if`, kita bisa lanjut sama perbandingan 
antara `guess` sama angka rahasianya dengan tenang karena tau `guess` pasti di 
antara 1 sampe 100.

Tapi, ini bukan solusi yang ideal: kalau bener-bener kritis sekali (absolutely 
critical) kalau programnya cuma beroperasi pada nilai di antara 1 sampe 100, dan 
program itu punya banyak fungsi dengan persyaratan ini, punya pengecekan kayak 
gini di tiap fungsi bakal ngebosenin dan repetitif sekali (dan mungkin ngaruh ke 
performa juga).

Sebagai gantinya, kita bisa bikin tipe baru di dalem modul yang didedikasikan 
khusus dan naruh validasinya di dalem sebuah fungsi buat bikin instance dari 
tipe itu bukannya ngulangin validasinya di mana-mana. Dengan gitu, bakal aman 
buat fungsi-fungsi buat pake tipe baru ini di _signature_ mereka dan pake nilai 
yang mereka terima dengan pede. Listing 9-13 nunjukin salah satu cara buat 
mendefinisikan tipe `Guess` yang cuma bakal bikin instance dari `Guess` kalau 
fungsi `new` nerima nilai di antara 1 sampe 100.

<Listing number="9-13" caption="Sebuah tipe `Guess` yang cuma bakal lanjut kalau nilainya di antara 1 sampe 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Perhatiin ya kalau kode ini di *src/guessing_game.rs* bergantung sama penambahan 
deklarasi modul `mod guessing_game;` di *src/lib.rs* yang belum kita tunjukin 
di sini. Di dalem file modul baru ini, kita mendefinisikan sebuah struct namanya 
`Guess` yang punya field namanya `value` yang nampung sebuah `i32`. Di sinilah 
angkanya bakal disimpan.

Terus kita mengimplementasikan sebuah fungsi _associated_ namanya `new` pada 
`Guess` yang bertugas bikin instance-instance dari nilai `Guess`. Fungsi `new` 
didefinisikan buat punya satu parameter namanya `value` dari tipe `i32` dan 
balikin sebuah `Guess`. Kode di body fungsi `new` ngetes `value` buat mastiin 
kalau dia ada di antara 1 sampe 100. Kalau `value` nggak lolos tes ini, kita 
manggil `panic!`, yang bakal ngingetin _programmer_ yang nulis kode pemanggil 
kalau mereka punya _bug_ yang harus dibenerin, karena bikin sebuah `Guess` 
dengan `value` di luar rentang ini bakal melanggar kontrak yang diandelin sama 
`Guess::new`. Kondisi-kondisi di mana `Guess::new` mungkin bakal _panic_ 
harusnya didiskusikan di dokumentasi API yang ngadep _public_ (public-facing API); 
kita bakal ngebahas konvensi dokumentasi buat nunjukin kemungkinan `panic!` di 
dokumentasi API yang kita bikin di Bab 14. Kalau `value` lolos tes, kita bikin 
`Guess` baru dengan field `value`-nya di-set ke parameter `value` terus balikin 
`Guess`-nya.

Selanjutnya, kita mengimplementasikan sebuah method namanya `value` yang minjem 
(`borrows`) `self`, nggak punya parameter lain apa pun, dan balikin sebuah `i32`. 
Tipe method kayak gini kadang disebut _getter_ karena tujuannya adalah buat dapet 
beberapa data dari field-nya terus balikin datanya. Method _public_ ini dibutuhin 
karena field `value` dari struct `Guess` itu _private_. Ini penting sekali biar 
field `value` tetep _private_ biar kode yang pake struct `Guess` nggak dibolehin 
nge-set `value` secara langsung: kode di luar modul `guessing_game` *harus* pake 
fungsi `Guess::new` buat bikin instance dari `Guess`, dan dengan gitu ngejamin 
nggak ada cara buat sebuah `Guess` buat punya `value` yang belum dicek sama 
kondisi di fungsi `Guess::new`.

Fungsi yang punya parameter atau balikin cuma angka di antara 1 sampe 100 
kemudian bisa mendeklarasikan di _signature_-nya kalau dia nerima atau balikin 
sebuah `Guess` bukannya sebuah `i32` dan nggak perlu ngelakuin pengecekan 
tambahan apa pun di body-nya.

## Ringkasan

Fitur _error-handling_ di Rust didesain buat ngebantu kita nulis kode yang 
lebih kuat (robust). Macro `panic!` nandain kalau program kita ada di keadaan 
yang dia nggak bisa tanganin dan ngasih kita cara buat nyuruh prosesnya buat 
berhenti bukannya nyoba lanjut pake nilai yang nggak valid atau salah. Enum 
`Result` pake sistem tipe Rust buat ngindikasikan kalau operasi bisa aja gagal 
dengan cara yang kode kita bisa pulih (recover) darinya. Kita bisa pake `Result` 
buat ngasih tau kode yang manggil kode kita kalau dia harus nanganin potensi 
sukses atau gagal juga. Pake `panic!` sama `Result` di situasi yang pas bakal 
bikin kode kita lebih bisa diandelin pas ngadepin masalah yang nggak bisa 
dihindarin.

Sekarang setelah kita liat cara yang kepake sekali di mana _standard library_ 
pake generik bareng enum `Option` sama `Result`, kita bakal bahas gimana cara 
kerja generik (generics) dan gimana kita bisa pakenya di kode kita.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
