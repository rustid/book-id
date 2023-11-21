## Instalasi

Langkah pertama yang perlu Anda lakukan adalah menginstal Rust. Kita akan mengunduh
Rust menggunakan `rustup`, sebuah perkakas _command line_ yang digunakan untuk
mengatur versi Rust dan alat-alat yang berkaitan dengannya. Anda akan membutuhkan koneksi
internet untuk mengunduh `rustup`.

> Catatan: Jika Anda kurang suka menggunakan `rustup` karena suatu alasan,
> silakan lihat [halaman Metode Instalasi Rust Lainnya][otherinstall] untuk
> melihat opsi lain yang bisa Anda gunakan.

Langkah-langkah berikut digunakan untuk menginstal versi stabil kompilator Rust yang terbaru.
Jaminan kestabilan yang diberikan Rust memastikan bahwa semua contoh yang berhasil
dikompilasi di buku ini akan seterusnya berhasil dikompilasi menggunakan versi-versi Rust yang lebih baru.
Luarannya mungkin sedikit berbeda antara satu versi dengan versi lain dikarenakan
seringkali terdapat perbaikan pesan galat dan peringatan pada Rust. Dengan kata lain,
tidak peduli versi Rust mana yang Anda instal menggunakan langkah-langkah
berikut, selama versi itu merupakan versi stabil dan lebih baru, seharusnya masih
sesuai dengan isi dari buku ini.

> ### Notasi _Command Line_
>
> Pada bab ini dan seterusnya dalam buku ini, kami akan menampilkan perintah-perintah terminal.
> Baris perintah yang Anda perlu ketikkan ke terminal semuanya diawali dengan tanda `$`.
> Anda tidak perlu mengetikkan tanda `$`-nya; itu hanya penanda untuk menunjukkan
> bahwa itu merupakan awal dari perintah tersebut. Baris yang tidak diawali dengan
> tanda `$` biasanya merupakan luaran dari perintah sebelumnya. Perlu dicatat bahwa
> contoh-contoh yang spesifik ditujukan untuk PowerShell akan menggunakan tanda `>`
> alih-alih `$` untuk mengindikasikan awal perintah.

### Menginstal `rustup` di Linux atau macOS

Jika Anda menggunakan Linux atau macOS, silakan buka terminal dan ketikkan perintah berikut:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Perintah di atas akan mengunduh sebuah skrip dan otomatis memulai instalasi `rustup`,
yang juga akan menginstal versi stabil terbaru dari Rust. Anda mungkin akan ditanyai
kata sandi Anda ketika menjalankan perintah di atas. Jika instalasi berhasil, akan
muncul baris berikut:

```text
Rust is installed now. Great!
```

Anda juga akan memerlukan sebuah _linker_, yaitu sebuah program yang Rust gunakan
untuk menggabungkan luaran-luaran hasil kompilasinya menjadi satu berkas.
Kemungkinan besar sistem Anda sudah memiliki _linker_. Tetapi jika Anda mendapatkan
galat _linker_, Anda harus menginstal kompilator C yang biasanya membawa sebuah _linker_.
Kompilator C juga berguna karena beberapa _package_ Rust populer bergantung kepada
kode C dan akan membutuhkan kompilator C.

Di macOS, Anda bisa mengunduh kompilator C dengan menjalankan perintah berikut:

```console
$ xcode-select --install
```

Pengguna Linux biasanya lebih disarankan untuk menginstal GCC atau Clang, tergantung
dokumentasi distribusi Linux yang digunakan. Sebagai contoh, jika Anda menggunakan Ubuntu,
Anda dapat menginstal paket `build-essential`.

### Menginstal `rustup` di Windows

Untuk Windows, silakan menuju [https://www.rust-lang.org/tools/install][install] dan ikuti
instruksinya untuk menginstal Rust. Pada suatu titik dalam proses instalasi, Anda akan
mendapatkan pesan yang menjelaskan bahwa Anda juga membutuhkan _build tool_ MSVC untuk
Visual Studio 2013 atau yang lebih baru.

Untuk mendapatkan _build tool_ tersebut, Anda harus menginstal [Visual Studio 2022][visualstudio].
Dan ketika Anda ditanya _workload_ jenis apa yang hendak diinstal, silakan pilih:

- “Desktop Development with C++”
- The Windows 10 or 11 SDK
- The English language pack component, along with any other language pack of
  your choosing

Keseluruhan buku ini akan menggunakan perintah-perintah yang dapat bekerja di
_cmd.exe_ maupun di PowerShell. Jika ada perbedaan-perbedaan khusus, kami akan menambahkan
penjelasan terkait mana yang harus digunakan untuk menjalankan perintah tersebut.

### Penyelesaian Masalah

Untuk mengecek apakah Anda telah berhasil menginstal Rust dengan benar, buka terminal Anda
dan ketikkan perintah berikut:

```console
$ rustc --version
```

Anda seharusnya akan melihat nomor versi, hash commit, serta tanggal commit untuk versi stabil
terbaru yang telah dirilis, dalam format berikut:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Jika Anda melihat informasi tersebut, itu berarti Anda telah berhasil menginstal Rust dengan benar.
Jika tidak, periksalah apakah Rust ada di variabel sistem `%PATH%` di sistem Anda dengan
mengetikkan perintah berikut ke terminal:

Untuk CMD di Windows, gunakan:

```console
> echo %PATH%
```

Untuk PowerShell, gunakan:

```powershell
> echo $env:Path
```

Untuk Linux dan macOS, gunakan:

```console
$ echo $PATH
```

Jika hasil dari perintah-perintah di atas sudah sesuai ekspektasi tetapi Anda masih
belum melihat versi Rust-nya, Anda dapat meminta bantuan ke sesama Rustaceans (julukan
konyol yang kami berikan kepada kami sendiri) di [halaman komunitas][community].

### Pembaruan dan Menghapus Instalasi

Jika Rust sudah berhasil terinstal menggunakan `rustup`, pembaruan dapat dilakukan dengan mudah.
Melalui terminal Anda, jalankan perintah berikut:

```console
$ rustup update
```

Untuk menghapus instalasi Rust dan `rustup`, jalankan perintah berikut:

```console
$ rustup self uninstall
```

### Dokumentasi Lokal

Instalasi Rust juga membawa salinan dokumentasi yang dapat Anda baca secara luring.
Jalankan `rustup doc` untuk membuka dokumentasi tersebut di peramban internet Anda.

Kapanpun ada sebuah tipe data atau fungsi yang disediakan pustaka standar yang Anda
tidak yakin kegunaannya dan bagaimana menggunakannya, silakan manfaatkan dokumentasi API
untuk mencari tahu!

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[visualstudio]: https://visualstudio.microsoft.com/downloads/
[community]: https://www.rust-lang.org/community
