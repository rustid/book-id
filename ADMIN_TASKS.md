# Tugas-tugas Administratif (Administrative Tasks)

Dokumentasi ini dibuat untuk siapa saja yang mengelola (managing) repo ini sebagai 
pengingat gimana cara melakukan tugas-tugas pemeliharaan (maintenance) yang 
sekali-sekali diperlukan.

## Memperbarui (Update) versi `rustc`

- Hapus *directory* `target` kita, toh kita juga bakal mengkompilasi ulang 
  (recompile) semuanya
- Ubah nomor versi di `.github/workflows/main.yml`
- Ubah nomor versi di `rust-toolchain`, yang mana seharusnya bakal ngubah 
  versi yang kita pakai secara lokal lewat `rustup`
- Ubah nomor versi di `src/title-page.md`
- Jalankan `./tools/update-rustc.sh` (silakan lihat kode yang dikomentari di 
  dalamnya buat detail seputar apa aja yang dilakukannya)
- Periksa perubahan-perubahannya (dengan cara ngelihat file-file yang berubah 
  menurut `git`) dan efek-efeknya (dengan melihat file-file di `tmp/book-before` 
  dan `tmp/book-after`) lalu _commit_ perubahannya kalau mereka kelihatan oke
- Pakai `grep` buat mencari `manual-regeneration` lalu ikuti instruksi-instruksi 
  di tempat tersebut buat meng-_update_ output yang tidak bisa dihasilkan oleh 
  sebuah _script_

## Memperbarui `edition` (edisi) di semua listings (daftar kode)

Buat meng-update metadata `edition = "[year]"` di dalam semua `Cargo.toml` milik 
_listings_, jalankan _script_ `./tools/update-editions.sh`. Cek perbedaan 
(_diff_)-nya buat memastikan kalau semuanya kelihatan wajar (reasonable), dan 
terkhusus periksa apakah _updates_ tersebut mewajibkan adanya perubahan ke 
dalam teksnya. Terus _commit_ perubahan tersebut.

## Memperbarui `edition` di dalam konfigurasi mdBook

Buka `book.toml` dan `nostarch/book.toml` lalu atur (set) nilai `edition` di dalam 
tabel `[rust]` menjadi edisi yang baru.

## Merilis versi baru dari listings

Sekarang ini kita ngebikin file `.tar` dari proyek komplit yang berisi semua 
_listing_ yang ada dan menyediakannya [sebagai GitHub Releases](https://github.com/rust-lang/book/releases). 
Buat ngebikin sebuah rilis artefak (release artifact) yang baru, contohnya kalau ada 
perubahan kode akibat dari hasil editan atau karena kita baru meng-_update_ Rust 
dan `rustfmt`, lakukan langkah-langkah berikut:

- Bikin sebuah *git tag* buat rilis tersebut lalu *push* tag-nya ke GitHub, atau 
  bikin tag baru dengan cara pergi ke UI GitHub, [membikin draf rilis baru](https://github.com/rust-lang/book/releases/new), dan memasukkan sebuah 
  tag baru ketimbang memilih tag yang sudah ada
- Jalankan `cargo run --bin release_listings`, yang mana bakal menghasilkan 
  `tmp/listings.tar.gz`
- Unggah (upload) `tmp/listings.tar.gz` di antarmuka GitHub untuk draf rilis 
  tersebut
- Terbitkan (publish) rilisnya

## Menambahkan listing baru

Untuk memfasilitasi _scripts_ yang bertugas buat menjalankan `rustfmt` di semua _listings_, 
meng-_update_ output saat _compiler_ diperbarui, dan juga buat memproduksi rilis 
artefak yang berisi *projects* penuh dari semua _listings_, listing apa pun yang 
dirasa lebih dari sekadar trivial (remeh) wajib diekstrak menjadi sebuah file. 
Buat melakukannya:

- Cari tahu di mana listing baru itu harusnya ditaruh di dalam direktori `listings`.
  - Ada satu subdirektori (subdirectory) khusus buat masing-masing bab
  - Listings yang dikasih nomor wajib memakai nama direktori `listing-[chapter num]-[listing num]`.
  - Listings tanpa nomor wajib diawali dengan `no-listing-` diikuti oleh sebuah 
    nomor yang mengindikasikan posisinya di dalam bab relatif terhadap listings tanpa 
    nomor lainnya di bab tersebut, kemudian ditutup dengan deskripsi singkat supaya 
    orang bisa ngebaca dan nemuin kode yang lagi mereka cari.
  - Listings yang dipakai cuma buat menampilkan _output_ dari sebuah kode (contohnya, 
    pas kita bilang "kalau seandainya kita nulis x ketimbang y, kita bakal dapat 
    error compiler ini:" tapi kita aslinya tidak nunjukin kode x tersebut) wajib dinamakan dengan awalan 
    `output-only-` lalu diikuti sama sebuah nomor yang ngindikasikan posisinya di 
    dalam bab tersebut relatif terhadap _listings_ penampil _output_ yang lain, lalu 
    ditutup dengan deskripsi pendek supaya para penulis atau _contributors_ bisa ngebaca 
    dan nyari kode yang lagi mereka cari.
  - **Ingat ya buat menyesuaikan (adjust) urutan nomor-nomor listing di sekitarnya 
    kalau dirasa perlu!**
- Bikin sebuah project Cargo utuh di dalam _directory_ tersebut, entah dengan memakai 
  `cargo new` atau men-copy listing lain buat dijadiin titik pijak (starting point).
- Masukkan kode dan juga kode sekitarnya yang dibutuhkan buat ngebikin sebuah contoh 
  program yang komplit dan bisa jalan.
- Kalau kita cuma mau menampilkan sebagian dari kode di dalam file tersebut, pakailah 
  _anchor comments_ (komentar jangkar, misal `// ANCHOR: some_tag` dan 
  `// ANCHOR_END: some_tag`) buat nandain bagian mana dari file tersebut yang pengen 
  kita tampilin.
- Untuk kode Rust, pakailah direktif `{{#rustdoc_include [filename:some_tag]}}` di 
  dalam blok kode di teksnya. Direktif `rustdoc_include` ini ngasih tahu kode mana 
  yang tidak perlu ditampilkan ke `rustdoc` untuk tujuan pengujian `mdbook test`.
- Buat hal-hal lainnya, gunakan direktif `{{#include [filename:some_tag]}}`.
- Kalau kita juga mau menampilkan _output_ dari sebuah perintah (command) di dalam 
  teksnya, bikin sebuah file `output.txt` di direktori listing tersebut pakai 
  cara ini:
  - Jalankan perintahnya, semisal `cargo run` atau `cargo test`, lalu copy semua 
    _output_ yang keluar.
  - Bikin sebuah file `output.txt` baru di mana baris pertamanya adalah `$ [perintah 
    yang tadi kita jalankan]`.
  - _Paste_ output yang barusan kita *copy*.
  - Jalankan `./tools/update-rustc.sh`, yang mana seharusnya bakal nerapin beberapa 
    tindakan normalisasi (_normalization_) pada output _compiler_ tersebut.
  - Cantumkan output tersebut ke dalam teks dengan memakai direktif `{{#include [filename]}}`.
  - Lakukan `add` dan `commit` untuk file output.txt.
- Kalau kita mau menampilin _output_ tapi entah gara-gara suatu hal _output_ 
  tersebut tidak bisa dihasilkan oleh sebuah _script_ (misalnya karena dia butuh 
  *input* dari _user_ atau ngelibatin aktivitas dari luar kayak *request* jaringan 
  web), biarkan outputnya tetap menyatu sejajar (inline) tapi bikin sebuah 
  komentar yang ngandung tulisan `manual-regeneration` lengkap beserta panduan 
  buat memperbarui _output_ *inline* tersebut secara manual.
- Kalau kita sama sekali tidak mau contoh kode ini dipaksa diformat sama `rustfmt` 
  (contohnya karena contoh kode tersebut emang sengaja dibikin supaya tidak bisa di-_parse_), 
  silakan tambahkan sebuah file `rustfmt-ignore` di direktori listing tersebut dan 
  tulisin alasan kenapa dia tidak diformat sebagai isi dari file tersebut (jaga-jaga 
  kalau ternyata itu adalah _bug_ di rustfmt yang mungkin bakal dibenerin suatu 
  hari nanti).

## Ngelihat efek dari sebuah perubahan ke dalem buku yang di-render

Buat sekadar ngecek, misal, _update_ `mdbook` atau ngubah cara-cara file 
disematkan (included):

- Hasilkan _build_ buku sebelum kita menerapkan perubahan yang mau dites dengan 
  menjalankan `mdbook build -d tmp/book-before`
- Terapkan perubahan-perubahan yang mau kita tes lalu jalankan `mdbook build -d tmp/book-after`
- Jalankan `./tools/megadiff.sh`
- File-file yang tersisa di dalam `tmp/book-before` dan `tmp/book-after` punya 
  perbedaan (differences) yang bisa kita inspeksi secara manual memakai alat 
  pelihat _diff_ andalan kita

## Memproduksi file markdown baru untuk No Starch (penerbit)

- Jalankan `./tools/nostarch.sh`
- Lakukan cek acak (spot check) pada file-file yang dihasilkan _script_ tersebut 
  di dalam direktori `nostarch`
- Masukkan mereka ke _git_ (check them in) kalau kita lagi mulai melakukan ronde perombakan (edits)

## Memproduksi markdown dari format docx untuk diffing (perbandingan)

- Simpan file docx menjadi `tmp/chapterXX.docx`.
- Di Microsoft Word, buka tab _review_ (tinjauan), pilih "Accept all changes and stop tracking"
- Simpan lagi file docx-nya terus tutup Word
- Jalankan `./tools/doc-to-md.sh`
- Ini seharusnya bakal menulis `nostarch/chapterXX.md`. Sesuaikan bagian XSL di dalam 
  `tools/doc-to-md.xsl` lalu jalankan kembali `./tools/doc-to-md.sh` andaikata 
  dirasa perlu.

## Menghasilkan (Generate) Graphviz dot

Kita memakai [Graphviz](http://graphviz.org/) buat beberapa diagram yang ada 
di buku ini. Kode sumber (source) dari file-file tersebut bersarang di direktori 
`dot`. Buat mengubah sebuah file `dot`, contohnya, `dot/trpl04-01.dot` menjadi 
sebuah file `svg`, jalankan:

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

Di dalam file SVG yang dihasilkan tersebut, hapus atribut _width_ (lebar) dan 
_height_ (tinggi) dari elemen `svg` dan terus atur atribut `viewBox` menjadi 
`0.00 0.00 1000.00 1000.00` atau nilai angka lain yang dipastiin tidak bakal ngepotong 
tampilan gambarnya.

## Menerbitkan (Publish) pratinjau (preview) ke GitHub Pages

Kadang-kadang kita mempublikasikan versi pratinjau (previews) buat kerjaan 
yang masih berjalan (in-progress) ke GitHub Pages. Alur yang direkomendasikan 
(recommended flow) buat proses _publishing_ ini adalah:

- Instal alat `ghp-import` dengan menjalankan `pip install ghp-import` (atau 
  `pipx install ghp-import`, dengan menggunakan [pipx][pipx]).
- Di direktori *root* (utama), jalankan `tools/generate-preview.sh`

[pipx]: https://pipx.pypa.io/stable/#install-pipx
