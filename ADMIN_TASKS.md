# Tugas Administratif

Dokumen ini ditujukan untuk siapa pun yang mengelola repo ini, agar
mudah mengingat cara melakukan tugas pemeliharaan yang sesekali perlu dilakukan.

## Update versi `rustc`

- Hapus direktori `target`, karena semuanya akan dikompilasi ulang
- Ganti nomor versi di `.github/workflows/main.yml`
- Ganti nomor versi di `rust-toolchain`, ini juga akan mengubah versi
  Rust lokal kamu lewat `rustup`
- Ganti nomor versi di `src/title-page.md`
- Jalankan `./tools/update-rustc.sh` (lihat komentar dalam script
  untuk detail apa saja yang dikerjakan)
- Periksa perubahan yang terjadi (lihat file yang berubah di git) dan
  efeknya (lihat isi `tmp/book-before` dan `tmp/book-after`). Commit
  kalau semuanya terlihat baik.
- Cari `manual-regeneration` lalu ikuti instruksinya untuk memperbarui
  output yang tidak bisa dihasilkan otomatis oleh script

## Update `edition` di semua listing

Untuk memperbarui metadata `edition = "[year]"` di semua `Cargo.toml`
listing, jalankan script `./tools/update-editions.sh`. Cek diff‑nya
untuk memastikan hasilnya masuk akal, dan cek apakah perubahan tersebut
mengharuskan penyesuaian pada teks. Setelah itu commit.

## Update `edition` di konfigurasi mdBook

Buka `book.toml` dan `nostarch/book.toml`, lalu set nilai `edition` di
tabel `[rust]` ke edisi terbaru.

## Rilis versi baru listings

Sekarang kami membuat file `.tar` yang berisi project lengkap untuk semua listing,
dan merilisnya melalui [GitHub Releases](https://github.com/rust-lang/book/releases). Untuk
membuat _release artifact_ baru — misalnya kalau ada perubahan kode
karena edit atau update Rust dan `rustfmt` — lakukan langkah berikut:

- Buat **git tag** untuk rilis lalu push ke GitHub, atau buat tag baru
  lewat GitHub UI dengan cara membuka halaman [drafting a new release](https://github.com/rust-lang/book/releases/new),
  lalu masukkan tag baru (bukan memilih tag yang sudah ada).
- Jalankan: `cargo run --bin release_listings` Perintah ini akan menghasilkan file
  `tmp/listings.tar.gz`
- Upload `tmp/listings.tar.gz` melalui GitHub UI pada draft rilis
- Terbitkan (publish) rilisnya

## Menambahkan listing baru

Untuk mempermudah script yang: menjalankan `rustfmt` pada semua listing, memperbarui
output saat compiler diperbarui, dan menghasilkan _release artifact_ yang berisi
project lengkap untuk setiap listing, maka setiap listing yang lebih dari sekadar contoh sepele
sebaiknya dipisahkan ke dalam file sendiri. Untuk melakukannya:

- Temukan di mana listing baru harus ditempatkan di direktori `listings`.
  - Ada satu subdirektori untuk setiap bab
  - Listing bernomor harus menggunakan `listing-[nomor bab]-[nomor listing]`
    sebagai nama direktori.
  - Listing tanpa nomor harus dimulai dengan `no-listing-` diikuti angka
    yang menunjukkan posisinya di dalam bab relatif terhadap listing lain
    yang juga tanpa nomor, lalu deskripsi singkat yang bisa dibaca
    seseorang untuk menemukan kode yang mereka cari.
  - Listing yang hanya digunakan untuk menampilkan output kode (misalnya saat kita
    mengatakan “jika kita menulis x bukan y, kita akan mendapat error
    compiler berikut:” tetapi kita tidak benar-benar menampilkan kode x)
    harus diberi nama dengan `output-only-` diikuti angka yang menunjukkan posisinya
    di dalam bab relatif terhadap listing lain yang hanya untuk output,
    lalu deskripsi singkat yang bisa dibaca penulis atau kontributor
    untuk menemukan kode yang mereka cari.
  - **Ingat untuk menyesuaikan penomoran listing di sekitarnya sesuai kebutuhan!**
- Buat proyek Cargo lengkap di direktori tersebut, bisa dengan menggunakan `cargo new`
  atau menyalin listing lain sebagai titik awal.
- Tambahkan kode dan kode pendukung lain yang diperlukan untuk membuat contoh yang benar-benar dapat berjalan.
- Jika kamu hanya ingin menampilkan sebagian kode di file,
  gunakan komentar anchor (`// ANCHOR: some_tag` dan `// ANCHOR_END: some_tag`)
  untuk menandai bagian file yang ingin ditampilkan.
- Untuk kode Rust, gunakan direktif `{{#rustdoc_include [filename:some_tag]}}`
  di dalam blok kode pada teks. Direktif `rustdoc_include` memberikan kode
  yang tidak ditampilkan kepada `rustdoc` untuk keperluan `mdbook test`.
- Untuk hal lainnya, gunakan direktif `{{#include [filename:some_tag]}}`.
- Jika kamu juga ingin menampilkan output dari sebuah perintah di dalam teks, buat file
  `output.txt` di direktori listing dengan cara berikut:
  - Jalankan perintahnya, seperti `cargo run` atau `cargo test`, lalu salin seluruh
    outputnya.
  - Buat file `output.txt` baru dengan baris pertama
    `$ [perintah yang kamu jalankan]`.
  - Tempel output yang tadi kamu salin.
  - Jalankan `./tools/update-rustc.sh`, yang akan melakukan normalisasi pada
    output compiler.
  - Sertakan output tersebut di teks menggunakan direktif `{{#include [filename]}}`.
  - Tambahkan dan commit `output.txt`.
- Jika kamu ingin menampilkan output tetapi karena suatu alasan output tersebut
  tidak bisa dihasilkan oleh script (misalnya membutuhkan input pengguna atau
  bergantung pada kejadian eksternal seperti permintaan web), biarkan output tetap inline
  tetapi tambahkan komentar yang berisi `manual-regeneration` dan instruksi untuk memperbarui
  output inline tersebut secara manual.
- Jika kamu bahkan tidak ingin contoh ini dicoba diformat oleh `rustfmt`
  (misalnya karena contoh tersebut memang sengaja tidak bisa diparse), tambahkan
  file `rustfmt-ignore` di direktori listing dan tuliskan alasan mengapa contoh tersebut
  tidak diformat di dalam file tersebut (jika suatu saat itu adalah bug `rustfmt`
  yang bisa diperbaiki).

## Lihat efek suatu perubahan pada buku yang dirender

Untuk memeriksa, misalnya saat memperbarui `mdbook` atau mengubah cara file dimasukkan:

- Hasilkan build buku sebelum perubahan yang ingin diuji dengan menjalankan
  `mdbook build -d tmp/book-before`
- Terapkan perubahan yang ingin diuji lalu jalankan `mdbook build -d tmp/book-after`
- Jalankan `./tools/megadiff.sh`
- File yang tersisa di `tmp/book-before` dan `tmp/book-after` memiliki perbedaan
  yang bisa kamu periksa secara manual menggunakan alat pembanding (diff) favoritmu

## Hasilkan file markdown baru untuk No Starch

- Jalankan `./tools/nostarch.sh`
- Periksa secara sekilas file yang dibuat script tersebut di direktori `nostarch`
- Masukkan ke git jika kamu akan memulai sesi pengeditan

## Hasilkan markdown dari docx untuk keperluan diff

- Simpan file docx ke `tmp/chapterXX.docx`.
- Di Word, buka tab review, pilih “Accept all changes and stop tracking”
- Simpan kembali docx lalu tutup Word
- Jalankan `./tools/doc-to-md.sh`
- Ini akan menulis ke `nostarch/chapterXX.md`. Sesuaikan XSL di
  `tools/doc-to-md.xsl` dan jalankan `./tools/doc-to-md.sh` lagi jika diperlukan.

## Generate Graphviz dot

Kami menggunakan [Graphviz](http://graphviz.org/) untuk beberapa diagram di
buku ini. Sumber file tersebut ada di direktori `dot`. Untuk mengubah file `dot`,
misalnya `dot/trpl04-01.dot`, menjadi `svg`, jalankan:

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

Pada SVG yang dihasilkan, hapus atribut width dan height dari elemen `svg`
dan set atribut `viewBox` ke `0.00 0.00 1000.00 1000.00` atau nilai lain
yang tidak memotong gambar.

## Publikasikan pratinjau ke GitHub Pages

Terkadang kami mempublikasikan pratinjau yang sedang dikerjakan ke GitHub Pages.
Alur yang direkomendasikan adalah:

- Instal tool `ghp-import` dengan menjalankan `pip install ghp-import` (atau `pipx install ghp-import` menggunakan [pipx][pipx]).
- Di root, jalankan `tools/generate-preview.sh`

[pipx]: https://pipx.pypa.io/stable/#install-pipx
