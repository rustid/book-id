# The Rust Programming Language

![Build Status](https://github.com/rust-lang/book/workflows/CI/badge.svg)

Repositori ini berisi kode sumber dari buku "The Rust Programming Language".

[Buku ini tersedia dalam bentuk fisik dari No Starch Press][nostarch].

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

kita juga bisa membaca buku ini secara gratis online. Silakan lihat buku yang disertakan dengan rilis Rust [stable], [beta], atau [nightly] terbaru. Perlu diketahui bahwa masalah pada versi-versi tersebut mungkin sudah diperbaiki di repositori ini, karena rilis-rilis tersebut jarang diperbarui.

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

Lihat bagian [releases] untuk mengunduh kode dari semua daftar kode yang muncul di buku ini.

[releases]: https://github.com/rust-lang/book/releases

## Persyaratan

Membangun buku ini membutuhkan [mdBook], idealnya versi yang sama dengan yang digunakan rust-lang/rust di [file ini][rust-mdbook]. Cara mendapatkannya:

[mdBook]: https://github.com/rust-lang/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/HEAD/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --locked --version <version_num>
```

Buku ini juga menggunakan dua plugin mdbook yang merupakan bagian dari repositori ini. Jika kita tidak menginstalnya, kita akan melihat peringatan saat membangun dan outputnya tidak akan terlihat benar, tetapi kita tetap bisa membangun bukunya. Untuk menggunakan plugin tersebut, jalankan:

```bash
$ cargo install --locked --path packages/mdbook-trpl --force
```

## Membangun

Untuk membangun buku, ketik:

```bash
$ mdbook build
```

Outputnya akan berada di subdirektori `book`. Untuk melihatnya, buka di browser web kita.

_Firefox:_

```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome:_

```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

Untuk menjalankan tes:

```bash
$ cd packages/trpl
$ mdbook test --library-path packages/trpl/target/debug/deps
```

## Berkontribusi

Kami sangat senang menerima bantuan kita! Silakan lihat [CONTRIBUTING.md][contrib] untuk mempelajari jenis kontribusi yang kami cari.

[contrib]: https://github.com/rust-lang/book/blob/main/CONTRIBUTING.md

Karena buku ini [dicetak][nostarch], dan karena kami ingin menjaga versi online tetap dekat dengan versi cetak jika memungkinkan, proses penanganan masalah atau pull request mungkin memakan waktu lebih lama dari biasanya.

Sejauh ini, kami melakukan revisi besar bersamaan dengan [Edisi Rust](https://doc.rust-lang.org/edition-guide/). Di antara revisi besar tersebut, kami hanya akan memperbaiki error. Jika masalah atau pull request kita tidak secara khusus memperbaiki error, mungkin baru akan ditangani saat revisi besar berikutnya: harap bersabar karena bisa memakan waktu berbulan-bulan atau bertahun-tahun. Terima kasih atas kesabaran kita!

### Terjemahan

Kami sangat senang jika ada yang membantu menerjemahkan buku ini! Lihat label [Translations] untuk bergabung dalam upaya yang sedang berjalan. Buka issue baru untuk mulai mengerjakan bahasa baru! Kami sedang menunggu [dukungan mdbook] untuk banyak bahasa sebelum menggabungkannya, tapi silakan mulai saja!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5

## Pengecekan Ejaan

Untuk memindai file sumber dari kesalahan ejaan, kita bisa memakai skrip `spellcheck.sh` di direktori `ci`. Skrip ini butuh kamus kata valid di `ci/dictionary.txt`. Jika skripnya salah mendeteksi kata yang benar sebagai salah (false positive), tambahkan kata tersebut ke `ci/dictionary.txt` (tetap jaga urutan abjad untuk konsistensi).
