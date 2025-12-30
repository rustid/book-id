# Bahasa Pemrograman Rust

![Build Status](https://github.com/rust-lang/book/workflows/CI/badge.svg)

Repositori ini berisi kode sumber untuk buku "The Rust Programming Language".

[Buku ini tersedia dalam bentuk fisik (cetak) dari No Starch Press.][nostarch]

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

Kamu juga bisa baca buku ini gratis secara online. Silakan cek versi yang dirilis
bersamaan dengan Rust versi [stable], [beta], atau [nightly]. Perlu diingat,
masalah atau error yang ada di versi tersebut mungkin sudah diperbaiki di repositori ini,
karena versi rilis resminya tidak diperbarui sesering repositori ini.

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

Cek bagian [releases] untuk mendownload kode-kode contoh yang muncul di dalam buku.

[releases]: https://github.com/rust-lang/book/releases

## Persyaratan

Untuk nge-build buku ini, kamu butuh [mdBook], idealnya versi yang sama dengan
yang dipakai rust-lang/rust di [file ini][rust-mdbook]. Cara installnya:

[mdBook]: https://github.com/rust-lang/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/HEAD/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --locked --version <nomor_versi>
```

## Membangun

Untuk membangun bukunya, ketik:

```bash
$ mdbook build
```

Hasilnya bakal ada di subdirektori `book`. Buat ngecek hasilnya, buka saja
pakai browser.

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

## Kontribusi

Kami senang banget kalau kamu mau bantu! Silakan baca [CONTRIBUTING.md][contrib]
untuk mengetahui kontribusi apa saja yang kami cari.

[contrib]: https://github.com/dudinsdn/rust-book-id/blob/docs/id-meta/CONTRIBUTING.md

Karena buku ini [dicetak][nostarch], dan kami ingin
versi online-nya tetap mirip dengan versi cetak,
mungkin proses penanganan isu atau _pull request_ dari kamu bakal lebih lama
dari biasanya.

Sejauh ini, kami melakukan revisi besar-besaran barengan dengan rilis [Edisi Rust](https://doc.rust-lang.org/edition-guide/).
Di antara revisi besar itu, kami cuma akan memperbaiki error saja.
Kalau isu atau _pull request_ kamu bukan untuk benerin error yang penting,
kemungkinan bakal didiamkan dulu sampai jadwal revisi besar berikutnya:
kira-kira dalam hitungan bulan atau tahun. Terima kasih atas kesabarannya!

### Terjemahan

Kami sangat butuh bantuan buat nerjemahin buku ini! Cek label [Translations]
untuk bergabung dengan tim yang sudah jalan. Kalau mau mulai bahasa baru,
buka saja isu baru! Kami masih menunggu [mdbook support] untuk banyak bahasa sebelum menggabungkannya,
tapi kalau mau mulai duluan silakan saja!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5

## Spellchecking

Buat ngecek kesalahan ejaan di file sumber, kamu bisa pakai script `spellcheck.sh`
yang ada di folder `ci`. Script ini butuh kamus kata-kata benar yang ada
di `ci/dictionary.txt`. Kalau script-nya salah deteksi
(misalnya, kamu nulis kata `BTreeMap` tapi dianggap salah),
kamu tinggal tambahin kata itu ke `ci/dictionary.txt` (urutkan sesuai abjad ya biar rapi).
