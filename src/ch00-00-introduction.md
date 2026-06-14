# Pengenalan

> Catatan: Edisi buku ini sama dengan [The Rust Programming Language][nsprust] 
> yang tersedia dalam format cetak dan ebook dari [No Starch Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-3rd-edition
[nsp]: https://nostarch.com/

Selamat datang di _The Rust Programming Language_, sebuah buku pengantar tentang 
Rust. Bahasa pemrograman Rust membantu kita menulis perangkat lunak yang lebih 
cepat dan andal. Ergonomi tingkat tinggi dan kontrol tingkat rendah sering kali 
dianggap berlawanan dalam desain bahasa pemrograman; Rust menantang konflik 
tersebut. Dengan menyeimbangkan kapasitas teknis yang kuat dan pengalaman 
pengembang yang menyenangkan, Rust memberi kita opsi untuk mengendalikan detail 
tingkat rendah (seperti penggunaan memori) tanpa semua kerumitan yang biasanya 
terkait dengan kontrol semacam itu.

## Rust untuk Siapa

Rust sangat ideal untuk banyak orang karena berbagai alasan. Mari kita lihat 
beberapa kelompok yang paling penting.

### Tim Pengembang

Rust terbukti menjadi alat yang produktif untuk berkolaborasi di antara tim 
pengembang yang besar dengan berbagai tingkat pemahaman tentang pemrograman 
sistem. Kode tingkat rendah rentan terhadap berbagai bug halus, yang di 
kebanyakan bahasa lain hanya bisa dideteksi setelah melalui pengujian ekstensif 
dan audit kode yang teliti oleh pengembang berpengalaman. Di Rust, _compiler_ 
berperan sebagai penjaga gerbang dengan menolak memproses kode yang mengandung 
bug-bug sulit ini, termasuk bug konkurensi. Dengan bekerja berdampingan dengan 
_compiler_, tim bisa fokus menggunakan waktunya untuk logika program daripada 
sibuk mencari bug.

Rust juga membawa peralatan pengembang kontemporer ke dunia pemrograman sistem:

- Cargo, manajer dependensi dan alat build bawaan, membuat proses menambah, 
  mengompilasi, dan mengelola dependensi menjadi mudah dan konsisten di seluruh 
  ekosistem Rust.
- Alat pemformatan `rustfmt` memastikan gaya penulisan kode yang konsisten di 
  antara para pengembang.
- Rust Language Server mendukung integrasi Integrated Development Environment 
  (IDE) untuk melengkapi kode (code completion) dan menampilkan pesan error 
  secara langsung.

Dengan menggunakan ini dan alat lainnya di ekosistem Rust, pengembang dapat 
menjadi lebih produktif ketika menulis kode tingkat sistem.

### Pelajar

Rust cocok untuk pelajar dan mereka yang tertarik dalam mempelajari konsep 
sistem. Melalui Rust, banyak orang yang belajar mengenai topik seperti 
pengembangan sistem operasi. Komunitas Rust sangat ramah dan senang menjawab 
pertanyaan dari para pelajar. Melalui upaya-upaya seperti buku ini, tim Rust 
ingin membuat konsep sistem dapat diakses oleh lebih banyak orang, terutama 
mereka yang masih baru dalam pemrograman.

### Perusahaan

Ratusan perusahaan, besar maupun kecil, menggunakan Rust di lingkungan produksi 
untuk berbagai macam keperluan, termasuk alat baris perintah, layanan web, 
peralatan DevOps, perangkat *embedded*, analisis dan transkoding audio serta 
video, mata uang kripto, bioinformatika, mesin pencari, aplikasi Internet of 
Things, *machine learning*, dan bahkan bagian-bagian utama dari peramban web 
Firefox.

### Pengembang Sumber Terbuka (Open Source)

Rust ditujukan bagi orang-orang yang ingin membangun bahasa pemrograman Rust, 
komunitas, peralatan pengembang, dan pustaka (_libraries_). Kami menunggu 
kontribusi kita pada bahasa Rust.

### Mereka yang Mengutamakan Kecepatan dan Stabilitas

Rust ditujukan untuk orang-orang yang mendambakan kecepatan dan stabilitas dalam 
sebuah bahasa. Dengan kecepatan, maksudnya adalah seberapa cepat kode Rust bisa 
dijalankan dan seberapa cepat Rust memungkinkan kita menulis program. 
Pemeriksaan yang dilakukan oleh _compiler_ Rust memastikan stabilitas bahkan 
ketika ada penambahan fitur atau refactoring. Ini berbeda dengan kode warisan 
(legacy code) yang rapuh pada bahasa tanpa pemeriksaan seperti ini, yang sering 
membuat pengembang takut untuk memodifikasinya. Dengan berupaya mencapai 
*zero-cost abstractions*—fitur tingkat tinggi yang dikompilasi menjadi kode 
tingkat rendah secepat kode yang ditulis manual—Rust berusaha agar kode yang 
aman juga bisa menjadi kode yang cepat.

Bahasa Rust juga berharap bisa mendukung banyak pengguna lainnya; yang 
disebutkan di sini hanyalah beberapa pemangku kepentingan terbesar. Secara 
keseluruhan, ambisi terbesar Rust adalah menghapus kompromi yang telah diterima 
para programmer selama puluhan tahun dengan menyediakan keamanan _dan_ 
produktivitas, kecepatan _dan_ ergonomi. Cobalah Rust dan lihat apakah pilihan-
pilihannya cocok untuk kita.

## Buku ini Ditujukan untuk Siapa

Buku ini mengasumsikan bahwa kita sudah pernah menulis kode dalam bahasa 
pemrograman lain, tetapi tidak mengasumsikan bahasa mana yang kita gunakan. 
Kami berusaha membuat materi ini dapat diakses secara luas oleh siapa pun dengan 
berbagai latar belakang pemrograman. Kami tidak banyak membahas tentang apa itu 
pemrograman atau bagaimana cara berpikir tentangnya. Jika kita benar-benar baru 
dalam pemrograman, akan lebih baik membaca buku yang secara khusus memberikan 
pengantar tentang pemrograman.

## Bagaimana Cara Menggunakan Buku ini

Secara umum, buku ini mengasumsikan kita membacanya secara berurutan dari depan 
ke belakang. Bab selanjutnya mendasarkan konsepnya pada bab sebelumnya, dan bab-
bab awal mungkin tidak akan menjelaskan secara detail mengenai topik tertentu 
tetapi akan mengulasnya kembali pada bab selanjutnya.

Kita akan menemukan dua jenis bab dalam buku ini: bab konsep dan bab proyek. 
Dalam bab konsep, kita akan mempelajari satu aspek dari Rust. Dalam bab proyek, 
kita akan membangun program kecil bersama-sama, menerapkan apa yang sudah kita 
pelajari sejauh ini. Bab 2, Bab 12, dan Bab 21 adalah bab proyek; sisanya adalah 
bab konsep.

**Bab 1** menjelaskan bagaimana memasang Rust, bagaimana membuat program 
"Hello, world!", dan bagaimana cara menggunakan Cargo, manajer paket dan alat 
build Rust. **Bab 2** adalah pengenalan langsung dalam menulis program di Rust, 
di mana kita akan membuat permainan tebak-tebakan angka. Di sini, kita membahas 
konsep pada tingkat yang relatif tinggi, dan bab selanjutnya akan memberikan 
detail tambahan. Jika kita ingin segera mencoba mempraktikkan koding, Bab 2 
adalah tempat yang paling cocok untuk itu. Jika kita tipe pelajar yang sangat 
teliti yang lebih memilih mempelajari setiap detailnya sebelum berpindah ke 
topik selanjutnya, mungkin lebih baik kita lewati Bab 2 dan langsung ke **Bab 3**, 
yang membahas fitur-fitur Rust yang mirip dengan bahasa pemrograman lainnya; 
baru kemudian kita kembali ke Bab 2 jika ingin mengerjakan suatu proyek untuk 
menerapkan detail-detail yang sudah dipelajari.

Di **Bab 4**, kita akan mempelajari sistem *ownership* Rust. **Bab 5** membahas 
struct dan method. **Bab 6** membahas enum, ekspresi `match`, serta konstruk 
kontrol alur `if let` dan `let...else`. Kita akan menggunakan struct dan enum 
untuk membuat tipe kustom.

Di **Bab 7**, kita akan mempelajari sistem modul Rust dan aturan privasi untuk 
mengelola kode kita dan API publiknya yang terkait. **Bab 8** membahas beberapa 
struktur data koleksi umum yang tersedia pada pustaka standar, seperti vector, 
string, dan hash map. **Bab 9** menjelajahi filosofi dan teknik penanganan error 
di Rust.

**Bab 10** menggali lebih dalam mengenai generik, trait, dan lifetime, yang akan 
memberikan kita kemampuan dalam mendefinisikan kode yang diterapkan pada 
beberapa tipe. **Bab 11** adalah semua hal tentang pengujian, di mana bahkan 
dengan jaminan keamanan dari Rust, pengujian tetap dibutuhkan untuk memastikan 
logika program kita benar. Di **Bab 12**, kita akan membangun implementasi 
sendiri dari sebagian fungsionalitas aplikasi baris perintah `grep` yang 
melakukan pencarian teks di dalam berkas. Untuk keperluan ini, kita akan 
menggunakan banyak konsep yang telah dibahas pada bab-bab sebelumnya.

**Bab 13** mengeksplorasi *closures* dan *iterators*: fitur-fitur Rust yang 
berasal dari bahasa pemrograman fungsional. Di **Bab 14**, kita akan memeriksa 
Cargo lebih dalam dan berbicara mengenai praktik terbaik untuk membagikan 
pustaka kita ke orang lain. **Bab 15** membahas *smart pointers* yang disediakan 
pustaka standar dan trait yang mendukung fungsionalitasnya.

Di **Bab 16**, kita akan membahas berbagai model pemrograman konkuren dan 
mempelajari bagaimana Rust membantu kita menulis program di banyak *threads* 
tanpa rasa takut. Di **Bab 17**, kita membangun di atas hal tersebut dengan 
mengeksplorasi sintaks async dan await di Rust, beserta task, future, dan 
stream, serta model konkurensi ringan yang mereka mungkinkan.

**Bab 18** melihat bagaimana idiom Rust dibandingkan dengan prinsip pemrograman 
berorientasi objek yang mungkin sudah kita kenal. **Bab 19** adalah referensi 
tentang pola dan *pattern matching*, yang merupakan cara kuat untuk 
mengekspresikan ide di seluruh program Rust. **Bab 20** berisi beragam topik 
lanjutan yang menarik, termasuk *unsafe Rust*, macros, serta pembahasan lebih 
lanjut tentang lifetime, trait, tipe, fungsi, dan closure.

Di **Bab 21**, kita akan menyelesaikan sebuah proyek dengan mengimplementasikan 
web server *multithreaded* tingkat rendah!

Akhirnya, beberapa lampiran berisi informasi berguna tentang bahasa Rust dalam 
format yang lebih mirip referensi. **Lampiran A** membahas kata kunci Rust, 
**Lampiran B** membahas operator dan simbol Rust, **Lampiran C** membahas trait 
turunan (*derivable traits*) yang disediakan pustaka standar, **Lampiran D** 
membahas beberapa alat pengembangan yang berguna, dan **Lampiran E** menjelaskan 
edisi Rust. Pada **Lampiran F**, kita bisa menemukan terjemahan dari buku ini, 
dan di **Lampiran G** kita akan membahas bagaimana Rust dibuat serta apa itu 
Rust nightly.

Tidak ada cara salah dalam membaca buku ini: jika kita ingin melompat ke bab 
depan, lakukan saja! Kita mungkin harus melompat kembali ke bab-bab awal jika 
mengalami kebingungan. Tapi lakukanlah apa pun yang cocok untuk kita.

<span id="ferris"></span>

Bagian penting dari proses pembelajaran Rust adalah mempelajari bagaimana 
membaca pesan error yang ditampilkan oleh _compiler_: hal-hal ini akan menuntun 
kita menuju kode yang berfungsi. Karena itu, kami akan menyediakan banyak contoh 
yang tidak bisa di-compile beserta pesan error dari _compiler_ pada tiap situasi 
tersebut. Pahami bahwa jika kita memasukkan dan menjalankan contoh secara acak, 
ada kemungkinan ia tidak bisa di-compile! Pastikan kita membaca teks di 
sekitarnya untuk melihat apakah contoh tersebut memang ditujukan untuk error. 
Dalam kebanyakan situasi, kami akan mengarahkan kita ke versi yang benar dari 
kode yang tidak bisa di-compile tersebut. Ferris juga akan membantu kita 
membedakan kode yang memang tidak dimaksudkan agar berfungsi:

| Ferris                                                                                                           | Makna                                          |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris dengan tanda tanya"/>            | Kode ini tidak bisa di-compile!                |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris mengangkat tangan"/>                   | Kode ini menghasilkan panic!                   |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris dengan satu capit di atas, mengedikkan bahu"/> | Kode ini tidak menghasilkan perilaku yang diinginkan. |

Dalam kebanyakan situasi, kami akan mengarahkan kita ke versi yang benar dari 
kode apa pun yang tidak bisa di-compile.

## Kode Sumber (Source Code)

Berkas sumber dari mana buku ini dibuat dapat ditemukan di [GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
