# Pendahuluan

> Catatan: Edisi buku ini sama dengan [Pemrograman Rust
> Bahasa] [nsprust] tersedia dalam format cetak dan ebook dari [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition
[nsp]: https://nostarch.com/

Selamat datang di *Bahasa Pemrograman Rust*, sebuah buku pengantar tentang Rust.
Bahasa pemrograman Rust membantu Anda menulis perangkat lunak yang lebih cepat dan lebih andal.
Sangat ergonomis pada pemrograman tingkat tinggi dan kontrol yang mudah pada pemrograman tingkat rendah yang sering kali bertentangan dalam desain bahasa pemrograman;
Rust menantang konflik itu. Melalui keseimbangan yang kuat
kapasitas teknis dan pengalaman pengembang yang hebat, Rust memberi Anda pilihan
untuk mengontrol detail low-level (seperti penggunaan memori) tanpa semua kerumitan
yang secara tradisional dikaitkan dengan kontrol semacam itu.

## Untuk Siapa Rust

Rust sangat ideal untuk banyak orang karena berbagai alasan. Mari kita kita lihat siapa saja yang orang-orang yang ideal menggunakannya.

### Tim Pengembang

Rust terbukti menjadi alat yang produktif untuk berkolaborasi di antara tim pengembang besar dengan berbagai tingkat pengetahuan pemrograman sistem. Kode Low-level
yang rentan terhadap berbagai bug (kesalahan) tersembunyi, yang di sebagian besar bahasa lain dapat ditemukan
hanya melalui pengujian ekstensif dan tinjauan kode yang cermat oleh
developer yang berpengalaman. Dalam Rust, kompiler memainkan peran sebagai penjaga gerbang dengan menolak untuk
mengkompilasi kode dengan bug yang sulit dipahami ini, termasuk bug konkurensi. Dengan bekerja
bersama kompiler, tim dapat menghabiskan waktu mereka dengan fokus pada
logika program daripada sibuk dengan bug.


Rust juga membawa alat pengembang kontemporer ke dunia pemrograman sistem:

* Cargo, manajer paket (dependensi) dan alat pengembang (build tool ) yang disertakan, membuat penambahan,
  kompilasi, dan mengelola dependensi tanpa kesulitan dan konsisten di seluruh
  ekosistem Rust.
* Perangkat permformatan Rustfmt memastikan gaya penulisan kode yang konsisten di seluruh
  developer.
* Server Bahasa Rust mendukung Lingkungan Pengembangan Terintegrasi (IDE),
  integrasi untuk penyelesaian kode dan pesan error secara langsung.

Dengan menggunakan alat ini dan alat lainnya dalam ekosistem Rust, developer dapat
produktif saat menulis kode tingkat rendah (sistem).

### Pelajar

Rust diperuntukkan bagi pelajar dan mereka yang tertarik untuk belajar tentang konsep yang berhubungan dengan sistem. Dengan menggunakan Rust, banyak orang telah belajar tentang topik-topik seperti pengembangan sistem operasi. Anggota komunitas sangat ramah dan dengan senang hati menjawab
pertanyaan-pertanyaan dari para pelajar. Melalui upaya seperti buku ini, tim Rust ingin
membuat konsep yang berhubungan dengan sistem lebih mudah diakses oleh lebih banyak orang, terutama mereka pemula di bidang pemrograman.

### Perusahaan

Ratusan perusahaan, besar dan kecil, menggunakan Rust dalam produksi untuk berbagai macam
tugas, termasuk alat baris perintah, layanan web, alat devOps, perangkat yang disematkan (embedded devices), analisis dan transkode audio dan video, mata uang kripto,
bioinformatika, mesin pencari, aplikasi Internet of Things, machine
learning, dan bahkan bagian utama dari peramban web Firefox.

### Pengembang Sumber Terbuka (Open Source)

Rust diperuntukkan bagi orang-orang yang ingin membangun bahasa pemrograman Rust, komunitas,
alat bantu developer, dan berbagai pustaka. Kami ingin Anda berkontribusi pada bahasa pemrograman Rust.

### Orang yang Menghargai Kecepatan dan Stabilitas

Rust adalah untuk orang-orang yang mendambakan kecepatan dan stabilitas dalam sebuah bahasa. `cepat``, maksud kami
berarti seberapa cepat kode Rust dapat berjalan dan seberapa cepat Rust memampukan Anda
menulis program. Kompilator Rust memastikan stabilitas melalui fitur
Penambahan fitur dan refactoring. Hal ini berbeda dengan kode lama yang rentan dalam bahasa-bahasa yang tidak memiliki pemeriksaan ini, yang sering kali membuat para pengembang takut untuk memodifikasinya. Dengan
Mengupayakan abstraksi tanpa biaya(zero-cost abstractions), fitur tingkat tinggi yang dikompilasi ke
kode tingkat rendah (low-level) secepat kode yang ditulis secara manual, Rust berupaya untuk menjadikan kode yang aman menjadi kode yang cepat juga.

Bahasa Rust berharap untuk mendukung banyak pengguna lain juga; mereka yang disebutkan
yang disebutkan di sini hanyalah beberapa pemangku kepentingan terbesar. Secara keseluruhan, ambisi terbesar Rust adalah untuk menghilangkan trade-off yang telah diterima oleh para programmer selama beberapa dekade dengan memberikan keamanan *dan* produktivitas, kecepatan *dan* ergonomi. 
Berikan Rust kesempatan dan lihat apakah 
solusinya cocok untuk Anda.

## Untuk Siapa Buku Ini

Buku ini mengasumsikan bahwa Anda telah menulis kode dalam bahasa pemrograman lain tetapi
tidak membuat asumsi apa pun tentang bahasa pemrograman yang mana. Kami telah mencoba untuk membuat materi
dapat diakses secara luas oleh mereka yang berasal dari berbagai latar belakang pemrograman.
Kami tidak membuang banyak waktu untuk berbicara tentang apa itu pemrograman atau bahkan bagaimana cara berpikir
tentang hal itu. Jika Anda benar-benar baru dalam pemrograman, Anda akan lebih baik jika membaca yang secara khusus memberikan pendahuluan tentang pemrograman.

## Bagaimana Cara Menggunakan Buku Ini

Secara umum, buku ini mengasumsikan bahwa Anda membacanya secara berurutan dari depan ke
belakang. Bab-bab selanjutnya dibangun berdasarkan konsep-konsep di bab sebelumnya, dan bab-bab sebelumnya mungkin tidak membahas secara rinci tentang topik tertentu, tetapi akan meninjau kembali
topik tersebut di bab selanjutnya.

Anda akan menemukan dua jenis bab dalam buku ini: bab konsep dan bab proyek. 
Dalam bab konsep, Anda akan belajar tentang sebuah aspek dari Rust. Dalam bab proyek, kita akan membuat program kecil bersama-sama, menerapkan apa yang telah Anda pelajari sejauh ini. Bab 2, 12, dan 20 adalah bab proyek; sisanya adalah bab konsep.

Bab 1 menjelaskan cara menginstal Rust, cara menulis program "Halo, Dunia!",
dan cara menggunakan Cargo, manajer paket dan alat pengembangan Rust. Bab 2 adalah
pengantar langsung untuk menulis program di Rust, meminta Anda membuat
permainan menebak angka. Di sini kita akan membahas konsep-konsep pada tingkat tinggi, dan kemudian
bab-bab selanjutnya akan memberikan detail tambahan. Jika Anda ingin mendapatkan pengalaman langsung, Bab 2 adalah tempatnya. Bab 3 mencakup fitur-fitur Rust
yang mirip dengan bahasa pemrograman lainnya, dan di Bab 4
Anda akan belajar tentang sistem kepemilikan Rust. Jika Anda adalah seorang pelajar yang tekun (teliti)
yang lebih suka mempelajari setiap detail sebelum melanjutkan ke materi berikutnya, Anda mungkin ingin melewatkan Bab 2 dan langsung ke Bab 3, kembali ke Bab
2 ketika Anda ingin mengerjakan sebuah proyek dengan menerapkan detail yang telah Anda pelajari.

Bab 5 membahas tentang struktur dan metode, dan Bab 6 membahas tentang enum, `match`, dan
ekspresi, dan `if let` yang merupakan struktur aliran kontrol. Anda akan menggunakan struktur dan
enum untuk membuat tipe khusus di Rust.

Di Bab 7, Anda akan belajar tentang sistem modul Rust dan tentang aturan keamanan
untuk mengatur kode Anda dan Antarmuka Pemrograman Aplikasi (API) publiknya. 
Bab 8 membahas beberapa struktur data himpunan umum yang disediakan oleh 
pustaka standar, seperti vektor, string, dan hashmap. Bab 9
mengeksplorasi filosofi dan teknik penanganan kesalahan Rust.

Bab 10 mendalami generik, traits, dan lifetimes, yang memberi Anda kekuatan
untuk mendefinisikan kode yang berlaku untuk berbagai type. Bab 11 adalah tentang pengujian,
walaupun dengan jaminan keamanan Rust diperlukan untuk memastikan program Anda sudah benar. Pada Bab 12, kita akan membuat implementasi kita sendiri dari sebuah bagian
fungsionalitas dari alat baris perintah `grep` yang mencari
teks di dalam file. Untuk itu, kita akan menggunakan banyak konsep yang telah kita bahas pada
bab-bab sebelumnya

Bab 13 membahas closures dan iterator: fitur-fitur dari Rust yang berasal dari 
bahasa pemrograman fungsional. Pada Bab 14, kita akan membahas Cargo secara lebih
lebih dalam dan berbicara tentang praktik terbaik untuk berbagi pustaka Anda dengan orang lain.
Bab 15 membahas pointer pintar yang disediakan oleh pustaka standar dan
traits yang mengaktifkan fungsionalitasnya.

Di Bab 16, kita akan membahas berbagai model pemrograman konkuren
dan berbicara tentang bagaimana Rust membantu Anda memprogram dalam banyak thread tanpa rasa takut.
Bab 17 membahas bagaimana idiom-idiom Rust dibandingkan dengan pemrograman berorientasi objek
yang mungkin sudah Anda kenal.

Bab 18 adalah referensi tentang pattern dan pattern matching, yang merupakan cara
cara yang kuat untuk mengekspresikan ide di seluruh program Rust. Bab 19 berisi
banyak sekali topik-topik lanjutan yang menarik, termasuk tentang unsafe, makro, dan
lebih lanjut tentang lifetime, traits, type, fungsi, dan closure.

Di Bab 20, kita akan menyelesaikan sebuah proyek di mana kita akan mengimplementasikan sebuah low-level server web multithreaded!

Akhirnya, beberapa lampiran berisi informasi yang berguna tentang Rust dalam
format yang lebih mirip referensi. Lampiran A mencakup kata kunci Rust, Lampiran B
mencakup operator dan simbol-simbol Rust, Lampiran C mencakup trait yang dapat digunakan
yang disediakan oleh pustaka standar, Lampiran D mencakup beberapa alat pengembangan yang berguna
dan Lampiran E menjelaskan edisi-edisi Rust. Dan di Lampiran F, Anda dapat menemukan
terjemahan buku ini, dan dalam Lampiran G kita akan membahas bagaimana Rust dibuat dan
apa yang dimaksud dengan Rust nightly.

Tidak ada cara yang salah untuk membaca buku ini: jika Anda ingin melompati bab demi bab, lakukanlah!
Anda mungkin harus kembali ke bab-bab sebelumnya jika Anda mengalami
kebingungan. Namun, lakukan apa pun yang sesuai untuk Anda.

<span id="ferris"></span>

Bagian penting dari proses belajar Rust adalah mempelajari cara membaca pesan error
yang ditampilkan oleh kompiler: ini akan memandu Anda menuju kode yang berfungsi.
Oleh karena itu, kami akan memberikan banyak contoh kode yang gagal dikompilasi beserta pesan error
yang akan ditampilkan oleh kompiler pada berbagai situasi. Ketahuilah bahwa jika Anda memasukkan
dan menjalankan sebuah contoh acak, maka contoh tersebut mungkin tidak akan dikompilasi! Pastikan Anda membaca teks-teks di sekelilingnya untuk melihat apakah contoh yang Anda coba jalankan memang bertujuan untuk
kesalahan. Ferris juga akan membantu Anda membedakan kode yang memang tidak semestinya dijalankan:

| Ferris                                                                                                           | Meaning                                          |
|------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris dengan tanda tanya"/>            | Kode ini tidak dapat dikompilasi!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris mengangkat tangan"/>                   | Kode ini membuat panik!                                |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris dengan satu cakar ke atas, mengangkat bahu"/> | Kode ini tidak menghasilkan perilaku yang diinginkan. |

Dalam sebagian besar situasi, kami akan mengarahkan Anda ke versi yang benar dari kode apa pun yang
tidak dikompilasi.

## Source Code

File sumber yang digunakan untuk membuat buku ini dapat ditemukan di
[GitHub] [buku].

[buku]: https://github.com/rust-lang/book/tree/main/src
