## Instalasi

Langkah pertama yang kita lakukan adalah instalasi bahasa Rust. Kita akan men-
download Rust melalui `rustup`, tool _command line_ ini digunakan untuk mengatur 
versi dari Rust dan beberapa _tool - tool_ yang ada didalamnya. untuk instalasi ini 
kita membutuhkan koneksi internet untuk mendownloadnya.

> Catatan: Jika Kita lebih memilih untuk tidak menggunakan `rustup` untuk beberapa
> alasan, Kita bisa merujuk pada [Halaman Instalasi Rust Lainnya][otherinstall]
> untuk opsi lainnya

Langkah selanjutnya memasang versi stable dari compiler Rust. Jaminan
stabilitas Rust memastikan bahwa semua contoh di buku ini yg bisa di compile
akan tetap bisa di compile dengan versi Rust yg lebih baru. Hasil Output
mungkin berbeda sedikit antarversi karena Rust sering kali memperbaiki pesan
error dan peringatan. Dengan kata lain, semua versi stable Rust yg lebih baru,
yg kita install menggunakan metode berikut, seharusnya bekerja sesuai harapan
dengan isi buku ini.

> ### Notasi Baris Perintah
>
> Pada bab ini dan keseluruhan buku, kita akan ditunjukkan beberapa perintah yg
> digunakan pada terminal. Baris yg seharusnya kita masukkan di terminal semua
> diawali dengan `$`. kita tidak perlu mengetikkan karakter `$`; Itu adalah penanda
> baris perintah yg ditampilkan untuk menunjukkan awal dari tiap perintah.
> Baris yg tidak dimulai dengan `$` biasanya menampilkan output dari perintah
> sebelumnya. Sebagai tambahan, contoh khusus PowerShell akan menggunakan `>`
> daripada `$`.

### Pemasangan `rustup` pada Linux atau macOS

Jika kita menggunakan Linux atau macOS, buka sebuah terminal dan masukkan
perintah berikut:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Perintah tersebut akan mengunduh sebuah script dan memulai pemasangan aplikasi
`rustup`, dan memasang versi stable terakhir Rust. Kita mungkin akan diminta
untuk memasukkan kata kunci. Jika pemasangan sukses, baris berikut akan muncul:

```text
Rust is installed now. Great!
```

Kita juga akan memerlukan sebuah _linker_, yaitu sebuah program yang digunakan 
Rust untuk menggabungkan hasil kompilasi menjadi satu berkas. Kemungkinan besar 
kita sudah memilikinya. Jika kita mengalami error terkait linker, kita sebaiknya 
menginstal compiler C, yang biasanya sudah menyertakan linker. Compiler C juga 
berguna karena beberapa paket Rust yang umum bergantung pada kode C dan akan 
membutuhkan compiler C.

Di macOS, kita memasang compiler C dengan menjalankan:

```console
$ xcode-select --install
```

Pengguna Linux biasanya memasang GCC atau Clang, sesuai dengan dokumentasi
masing-masing distribusi. Sebagai contoh, jika kita menggunakan Ubuntu, kita
bisa memasang paket `build-essential`.

### Memasang `rustup` pada Windows

Di Windows, buka [https://www.rust-lang.org/tools/install][install] dan ikuti 
instruksi untuk memasang Rust. Pada suatu tahap dalam proses instalasi, kita akan 
diminta untuk menginstal Visual Studio. Ini akan menyediakan sebuah linker dan 
pustaka native yang dibutuhkan untuk mengompilasi program. Jika kita membutuhkan 
bantuan lebih lanjut pada langkah ini, lihat [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]

Sisa buku ini menggunakan perintah yang bisa dijalankan baik di _cmd.exe_ maupun  
PowerShell. Jika ada perbedaan khusus, kita akan menjelaskan mana yang harus digunakan.

### Pemecahan Masalah

Untuk mengecek apakah kita memiliki Rust yg terpasang dengan baik, buka terminal
dan masukkan:

```console
$ rustc --version
```

Seharusnya keluar nomor versi, hash commit, dan tanggal commit untuk versi
stable terakhir yg telah diterbitkan, dalam format:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Jika kita mendapatkan informasi tersebut, berarti kita telah berhasil memasang
Rust! Jika kita tidak bertemu dengan informasi tersebut, silakan cek apakah Rust
berada di variabel sistem `%PATH%` sebagai berikut:

Di Windows CMD, gunakan:

```console
> echo %PATH%
```

Di PowerShell, gunakan:

```powershell
> echo $env:Path
```

Di Linux dan macOS, gunakan:

```console
$ echo $PATH
```

Jika semuanya sudah benar tetapi Rust masih belum bekerja, ada beberapa tempat
dimana kita bisa mencari bantuan. Cari tahu bagaimana berhubungan dengan Rustacean
(sebutan bagi kita) lainnya di [halaman komunitas][community].

### Memperbarui dan Menghapus

Begitu Rust terpasang melalui `rustup`, memperbarui ke versi terbaru menjadi
lebih mudah. Dari terminal kita, jalankan perintah berikut:

```console
$ rustup update
```

Untuk menghapus Rust dan `rustup`, jalankan perintah berikut dari terminal
kita:

```console
$ rustup self uninstall
```

### Dokumentasi Lokal

Pemasangan Rust juga menyertakan salinan dokumentasi lokal jadi kita dapat
membacanya secara luring. Jalankan `rustup doc` untuk membuka dokumentasi lokal
di peramban kita.

Setiap kali ada sebuah tipe atau fungsi yg tersedia di library std dan kita mungkin
tidak paham mengenai apa dan bagaimana cara menggunakannya, gunakan dokumentasi 
API (Application Programming Interface) berikut untuk mencari tahu!

### Editor Teks dan Lingkungan Pengembangan Terpadu

Buku ini tidak mengasumsikan alat apa yang kita gunakan untuk menulis kode Rust. 
Hampir semua editor teks bisa menyelesaikan pekerjaan! Namun, banyak editor teks 
dan Lingkungan Pengembangan Terpadu atau sering disebut Integrated Development 
Environments(IDE) yang memiliki dukungan bawaan untuk Rust. Kita selalu bisa 
menemukan daftar terkini dari berbagai editor dan IDE di [halaman tools][tools] pada 
situs web Rust.

### Bekerja Secara Offline dengan Buku Ini

Dalam beberapa contoh, kita akan menggunakan paket Rust di luar _library_ standar. 
Untuk mengikuti contoh-contoh tersebut, kita perlu memiliki koneksi internet 
atau sudah mengunduh dependensi tersebut sebelumnya. Untuk mengunduh dependensi 
lebih dulu, kita bisa menjalankan perintah berikut. (Nantinya kita akan menjelaskan 
apa itu `cargo` dan apa fungsi dari setiap perintah ini secara lebih rinci.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Perintah ini akan menyimpan hasil unduhan paket-paket tersebut di cache sehingga 
kita tidak perlu mengunduhnya lagi nanti. Setelah menjalankan perintah ini, kita 
tidak perlu menyimpan folder `get-dependencies`. Jika kita sudah menjalankan 
perintah ini, kita bisa menggunakan flag `--offline` pada semua perintah `cargo` 
di sisa buku ini untuk memakai versi yang sudah tersimpan di cache alih-alih 
mencoba menggunakan jaringan.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
