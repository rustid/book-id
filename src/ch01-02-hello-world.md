## Hello, World!

Sekarang setelah kita memasang Rust, saatnya menulis program Rust pertama kita. 
Sudah menjadi tradisi ketika belajar bahasa pemrograman baru untuk membuat program 
kecil yang mencetak teks `Hello, world!` ke layar, jadi mari kita lakukan hal yang 
sama di sini!

> Catatan: Buku ini mengasumsikan kita memiliki pemahaman dasar tentang command line.  
> Rust tidak memiliki tuntutan khusus mengenai editor, tools, atau di mana kode kita berada,  
> jadi jika lebih suka menggunakan integrated development environment (IDE)  
> daripada command line, silakan gunakan IDE favorit kita sendiri. Banyak IDE sekarang sudah  
> memiliki dukungan tertentu untuk Rust; periksa dokumentasi IDE untuk detailnya.  
> Tim Rust berfokus pada peningkatan dukungan IDE melalui `rust-analyzer`.  
> Lihat [Lampiran D][devtools] untuk detail lebih lanjut.

### Membuat Direktori Proyek

Kita akan mulai dengan membuat sebuah direktori untuk menyimpan kode Rust kita. 
Rust tidak peduli di mana kode kita berada, tetapi untuk latihan dan proyek dalam buku ini, 
kami menyarankan untuk membuat direktori _projects_ di direktori home kita dan menyimpan 
semua proyek di sana.

Buka terminal dan masukkan perintah berikut untuk membuat direktori _projects_ 
dan sebuah direktori untuk proyek “Hello, world!” di dalam direktori _projects_.

Untuk Linux, macOS, dan PowerShell di Windows, masukkan perintah ini:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

untuk Command Line Windows, masukkan perintah ini:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Menulis dan Menjalankan Program Rust

Selanjutnya, buat sebuah berkas sumber baru dan beri nama _main.rs_.
Berkas Rust selalu diakhiri dengan ekstensi _.rs_. Jika kita menggunakan lebih
dari satu kata dalam nama berkas, konvensinya adalah menggunakan garis bawah
(underscore) untuk memisahkan kata-kata tersebut. Misalnya, gunakan
_hello_world.rs_ daripada _helloworld.rs_.

Sekarang buka berkas _main.rs_ yang baru saja kita buat dan masukkan kode pada Listing 1-1.

<Listing number="1-1" file-name="main.rs" caption="Program yang mencetak `Hello, world!`">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

Simpan berkas tersebut lalu kembali ke jendela terminal kita di direktori  
_~/projects/hello_world_. Pada Linux atau macOS, masukkan perintah berikut  
untuk mengompilasi dan menjalankan berkas:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Pada Windows, masukan perintah `.\main` daripada `./main`: 

```powershell
> rustc main.rs
> .\main
Hello, world!
```

Terlepas dari sistem operasi yang kita gunakan, string `Hello, world!` seharusnya 
muncul di terminal. Jika kita tidak melihat output ini, lihat kembali bagian
[“Troubleshooting”][troubleshooting] pada bagian Instalasi untuk cara mendapatkan 
bantuan.

Jika `Hello, world!` berhasil tercetak, selamat! Kita secara resmi telah menulis
program Rust. Itu berarti kita sudah menjadi seorang pemrogram Rust—selamat datang!

### Anatomi Program Rust

Mari kita ulas program “Hello, world!” ini secara lebih rinci. Berikut adalah
potongan pertama dari teka-teki tersebut:


```rust
fn main() {

}
```

Baris-baris ini mendefinisikan sebuah fungsi bernama `main`. Fungsi `main` itu  
spesial: selalu menjadi kode pertama yang dijalankan dalam setiap program Rust  
yang bisa dieksekusi. Di sini, baris pertama mendeklarasikan fungsi bernama  
`main` yang tidak memiliki parameter dan tidak mengembalikan nilai apa pun.  
Jika ada parameter, maka mereka akan ditulis di dalam tanda kurung `()`.

Badan fungsi dibungkus dengan `{}`. Rust mewajibkan penggunaan kurung kurawal  
pada semua badan fungsi. Gaya penulisan yang baik adalah meletakkan kurung kurawal  
pembuka pada baris yang sama dengan deklarasi fungsi, dengan memberi satu spasi  
di antaranya.

> Catatan: Jika kita ingin tetap menggunakan gaya standar di seluruh proyek Rust,  
> kita bisa memakai alat pemformat otomatis bernama `rustfmt` untuk memformat kode  
> sesuai gaya tertentu (lebih lanjut tentang `rustfmt` ada di [Lampiran D][devtools]).  
> Tim Rust sudah menyertakan alat ini dalam distribusi standar Rust, sama seperti `rustc`,  
> jadi seharusnya sudah terpasang di komputer kita!

Badan fungsi `main` berisi kode berikut:

```rust
println!("Hello, world!");
```

Baris ini melakukan seluruh pekerjaan dalam program kecil ini: mencetak teks ke layar.
Ada tiga detail penting yang perlu kita perhatikan di sini.

Pertama, `println!` memanggil sebuah macro Rust. Jika yang dipanggil adalah fungsi biasa,
maka penulisannya akan `println` (tanpa `!`). Macro Rust adalah cara untuk menulis kode
yang menghasilkan kode lain untuk memperluas sintaks Rust, dan kita akan membahasnya lebih
lanjut di [Bab 20][ch20-macros]. Untuk saat ini, kita hanya perlu tahu bahwa penggunaan `!`
berarti kita sedang memanggil macro, bukan fungsi biasa, dan macro tidak selalu mengikuti
aturan yang sama dengan fungsi.

Kedua, kita melihat string `"Hello, world!"`. Kita meneruskan string ini sebagai argumen
ke `println!`, dan string tersebut akan dicetak ke layar.

Ketiga, kita mengakhiri baris dengan tanda titik koma (`;`), yang menunjukkan bahwa ekspresi
tersebut sudah selesai dan ekspresi berikutnya siap dimulai. Sebagian besar baris kode Rust
diakhiri dengan titik koma.

### Kompilasi dan Eksekusi adalah Langkah yang Terpisah

Kita baru saja menjalankan sebuah program baru, jadi mari kita periksa setiap langkah dalam prosesnya.

Sebelum menjalankan program Rust, kita harus mengompilasinya dengan compiler Rust
dengan memasukkan perintah `rustc` dan menambahkan nama berkas sumber kita, seperti ini:

```console
$ rustc main.rs
```

Jika kita memiliki latar belakang C atau C++, kita akan menyadari bahwa ini mirip
dengan `gcc` atau `clang`. Setelah kompilasi berhasil, Rust menghasilkan sebuah
biner yang bisa dieksekusi.

Di Linux, macOS, dan PowerShell di Windows, kita bisa melihat berkas executable
dengan memasukkan perintah `ls` di shell:


```console
$ ls
main  main.rs
```

Di Linux dan macOS, kita akan melihat dua berkas. Dengan PowerShell di Windows,
kita akan melihat tiga berkas yang sama seperti yang muncul saat menggunakan CMD.
Dengan CMD di Windows, kita akan memasukkan perintah berikut:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Ini menampilkan berkas kode sumber dengan ekstensi _.rs_, berkas executable
(_main.exe_ di Windows, tetapi _main_ di semua platform lain), dan, ketika
menggunakan Windows, sebuah berkas yang berisi informasi debugging dengan
ekstensi _.pdb_. Dari sini, kita bisa menjalankan berkas _main_ atau _main.exe_,
seperti ini:

```console
$ ./main # or .\main on Windows
```
Jika _main.rs_ kita adalah program “Hello, world!”, baris ini akan mencetak
`Hello, world!` ke terminal kita.

Jika kita lebih familiar dengan bahasa dinamis seperti Ruby, Python, atau
JavaScript, kita mungkin belum terbiasa dengan proses kompilasi dan eksekusi
sebagai langkah terpisah. Rust adalah bahasa _ahead-of-time compiled_, artinya
kita bisa mengompilasi program lalu memberikan file executable-nya kepada orang lain,
dan mereka dapat menjalankannya tanpa harus memasang Rust. Sebaliknya, jika kita
memberikan berkas _.rb_, _.py_, atau _.js_, orang tersebut perlu memasang Ruby,
Python, atau JavaScript (sesuai bahasa yang digunakan). Namun, pada bahasa-bahasa
tersebut, kita hanya perlu satu perintah untuk mengompilasi sekaligus menjalankan
program. Semua ini adalah bentuk kompromi dalam desain bahasa pemrograman.

Mengompilasi dengan `rustc` saja sudah cukup untuk program sederhana,
tetapi seiring pertumbuhan proyek, kita akan ingin mengatur semua opsi
dan mempermudah berbagi kode. Selanjutnya, kita akan berkenalan dengan alat Cargo,
yang akan membantu kita menulis program Rust untuk dunia nyata.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
