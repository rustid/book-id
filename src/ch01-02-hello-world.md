## Hello, World!

Setelah berhasil menginstal Rust, sekarang waktunya Anda menulis program Rust pertama Anda.
Sudah menjadi tradisi ketika pemrogram belajar bahasa pemrograman baru biasanya dimulai
dengan membuat sebuah program kecil yang dapat menampilkan teks `Hello, world!` ke layar,
dan itulah yang akan kita lakukan!

> Catatan: Kami mengasumsikan Anda sedikit familiar dengan _command line_. Rust tidak
> mengharuskan Anda menggunakan alat tertentu untuk mengedit kode Anda ataupun di mana
> Anda harus meletakkan kode Anda, jadi jika Anda lebih suka menggunakan _integrated development environment_ (IDE)
> daripada _command line_, silakan lakukan hal tersebut. Banyak IDE sekarang mendukung
> Rust; silakan periksa dokumentasi IDE yang hendak Anda gunakan. Tim Rust saat ini
> sedang fokus untuk memfasilitasi dukungan IDE yang lebih baik melalui `rust-analyzer`. Lihat
> [Lampiran D][devtools]<!-- ignore --> untuk detail lebih lanjut.

### Membuat Direktori Proyek

Anda akan memulai dengan membuat sebuah direktori untuk menyimpan kode Rust Anda.
Sebenarnya Rust tidak peduli di mana Anda meletakkan kode Anda, tetapi untuk latihan dan
proyek-proyek yang ada di buku ini, kami menyarankan Anda membuat sebuah direktori bernama
_projects_ di direktori _home_ Anda dan menyimpan semua kode Anda di sana.

Buka terminal dan masukkan perintah berikut untuk membuat direktori _projects_ dan
sebuah direktori untuk proyek "Hello, world!" di dalam direktori tersebut.

Untuk Linux, macOS, dan PowerShell di Windows, gunakan perintah berikut:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Untuk CMD di Windows, gunakan perintah berikut:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Menulis dan Menjalankan Program Rust

Selanjutnya, buat sebuah berkas baru dengan nama _main.rs_. Berkas Rust selalu
memiliki akhiran _.rs_. Jika nama berkas Anda terdiri atas lebih dari satu kata,
konvensinya adalah gunakan garis bawah untuk memisahkan tiap kata. Sebagai contoh,
gunakan _hello_world.rs_ alih-alih _helloworld.rs_.

Sekarang bukan berkas _main.rs_ yang telah Anda buat dan ketikkan kode di Daftar 1-1.

<span class="filename">Nama berkas: main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

<span class="caption">Daftar 1-1: Program yang menampilkan teks "Hello, world!"</span>

Simpan berkas tersebut dan kembali ke jendela terminal di direktori
_~/projects/hello_world_. Di Linux atau macOs, masukkan perintah berikut untuk
mengkompilasi dan menjalankan berkas tersebut:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Di Windows, gunakan perintah `.\main.exe` alih-alih `./main`:

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

Terlepas sistem operasi apa yang Anda gunakan, teks `Hello, world!` seharusnya tampil
di terminal Anda. Jika Anda tidak melihatnya, silakan membaca bagian ["Pemecahan Masalah"][troubleshooting]<!-- ignore -->
pada Bab Instalasi untuk melihat cara-cara mendapatkan bantuan.

Jika `Hello, world!` berhasil tampil, selamat! Anda telah berhasil menulis program
Rust. Anda sekarang adalah pemrogram Rust-selamat datang!

### Anatomi Program Rust

Mari meninjau program "Hello, world!" di atas dengan lebih detail. Berikut adalah
potongan teka-teki pertama:

```rust
fn main() {

}
```

Baris-baris di atas mendefinisikan sebuah fungsi bernama `main`. Fungsi `main` adalah
fungsi spesial: fungsi itu merupakan kode yang akan selalu dijalankan pertama kali
dalam program Rust. Di sini, baris pertama mendeklarasikan fungsi bernama `main` yang
tidak punya parameter dan tidak mengembalikan apapun. Jika ada parameter, paramter itu
akan berada di dalam kurung `()`.

Isi fungsi dibungkus dengan tanda `{}`. Rust mengharuskan penggunaan kurung kurawal
untuk membungkus isi fungsi. Selain itu, menempatkan kurung kurawal buka di baris yang
sama dengan deklarasi fungsi dengan memberikan jarak satu spasi adalah gaya kode yang bagus.

> Catatan: Jika Anda ingin konsisten menggunakan gaya kode yang sama di seluruh proyek Rust
> Anda, Anda dapat menggunakan alat format otomatis bernama `rustfmt` untuk memformat
> kode Anda ke gaya kode tertentu (lihat [Lampiran D][devtools]<!-- ignore --> untuk membaca
> lebih lanjut tentang `rustfmt`). Tim Rust telah memasukkan alat ini di distribusi
> Rust standar, seperti `rustc`, jadi ia seharusnya sudah terinstal di komputer Anda.

Fungsi `main` berisi kode berikut:

```rust
    println!("Hello, world!");
```

Baris ini adalah inti dari program kecil ini: ia bertugas menampilkan teks ke
layar. Terdapat 4 detail penting yang perlu dicatat di sini.

Yang pertama, gaya kode dalam hal indentasi adalah mengindentasi dengan 4 spasi,
bukannya tab.

Kedua, `println!` digunakan untuk memanggil macro dalam Rust. Jika hendak memanggil fungsi,
itu seharusnya ditulis dengan `println` (tanpa tanda `!`). Kita akan mendiskusikan macro
Rust dengan lebih detail di Bab 19. Untuk sekarang, Anda hanya perlu tahu bahwa penggunaan
tanda `!` berarti Anda memanggil macro dan bukannya fungsi.

Ketiga, Anda lihat teks `"Hello, world!"`. Kita mengoper teks tersebut sebagai
argumen ke `println!`, dan teks tersebut kemudian ditampilkan di layar.

Keempat, kita mengakhiri baris dengan tanda titik koma (`;`), yang mengindikasikan bahwa
ekspresi ini telah berakhir dan ekspersi berikutnya hendak dimulai. Sebagian besar
baris dalam kode Rust berakhir dengan tanda titik koma.

### Mengkompilasi dan Menjalankan Adalah Dua Langkah yang Berbeda

Anda baru saja menjalankan program yang baru Anda buat, mari kita teliti tiap langkahnya
dengan lebih detail.

Sebelum menajalankan program Rust, Anda harus mengkompilasinya menggunakan kompilator Rust
dengan memasukkan perintah `rustc` dan memberikannya nama berkas yang hendak dikompilasi,
seperti berikut:

```console
$ rustc main.rs
```

Jika Anda punya latar belakang C atau C++, Anda akan merasa bahwa ini mirip dengan `gcc`
atau `clang`. Setelah kompilasi berhasil, Rust akan mengeluarkan hasil berupa berkas
biner yang dapat dieksekusi.

Di Linux, macOS, dan PowerShell di Windows, Anda dapat melihat berkas tersebut dengan
memasukkan perintah `ls` ke terminal Anda:

```console
$ ls
main  main.rs
```

Di Linux dan macOS, Anda akan melihat dua berkas. Untuk PowerShell di Windows, Anda akan melihat
tiga berkas yang sama sebagaimana yang Anda lihat ketika menggunakan CMD. Untuk CMD di Windows,
Anda dapat memasukkan perintah berikut:

```cmd
> dir /B %= opsi /B digunakan untuk hanya menampilkan nama berkas saja =%
main.exe
main.pdb
main.rs
```

Perintah di atas menampilkan kode sumber berkas dengan akhiran _.rs_, berkas biner
yang dapat dieksekusi (_main.exe_ di Windows, _main_ di platform lain), dan jika
menggunakan Windows, sebuah berkas yang berisi informasi awakutu dengan akhiran _.pdb_.
Dari sini, Anda dapat menjalankan berkas _main_ atau _main.exe_, seperti berikut:

```console
$ ./main # atau .\main.exe di Windows
```

Jika berkas _main.rs_ adalah program "Hello, world!" Anda, perintah di atas seharusnya
akan menampilkan `Hello, world!` ke terminal Anda.

Jika Anda familiar dengan bahasa dinamis seperti Ruby, Python, atau JavaScript, Anda
mungkin tidak terbiasa dengan mengkompilasi dan menjalankan program sebagai dua langkah yang
terpisah. Rust adalah bahasa yang dikompilasi di awal, yang berarti Anda dapat mengkompilasi
programnya dan memberikan hasil kompilasinya ke orang lain dan mereka akan dapat
menjalankannya tanpa harus menginstal Rust terlebih dahulu. Hal ini berbeda dengan Ruby, Python,
atau JavaScript. Jika Anda memberikan berkas berakhiran _.rb_, _.py_, atau _.js_ kepada
orang lain, mereka harus menginstal Ruby, Python, atau JavaScript di komputer mereka. Tetapi
di bahasa-bahasa tersebut, Anda hanya memerlukan satu perintah untuk mengkompilasi sekaligus
menjalankan program Anda. Demikianlah, semuanya dalam desain bahasa pemrograman adalah
tentang pertukaran kelebihan dan kekurangan.

Mengkompilasi dengan `rustc` sebenarnya tidak masalah untuk program kecil, tetapi seiring
dengan tumbuhnya proyek Anda, Anda ingin dapat mengatur semua opsi dan memiliki cara yang
mudah untuk membagikan proyek Anda ke orang lain. Selanjutnya, kita akan berkenalan dengan
alat bernama Cargo, yang akan membantu Anda menulis program Rust yang benar-benar digunakan di dunia nyata.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
