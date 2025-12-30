# Kontribusi

Kami akan sangat senang kalau kamu mau membantu! Terima kasih sudah peduli dengan buku ini.

## Mengedit di Bagian Mana?

Semua perubahan harus dilakukan di direktori `src`.

Direktori `nostarch` berisi snapshot yang dikirim ke penerbit versi cetak. File snapshot
mencerminkan apa yang sudah dikirim atau belum, jadi hanya diperbarui
saat kami benar-benar mengirim perubahan ke No Starch. **Jangan kirim pull
request yang mengubah file di direktori `nostarch`, PR tersebut akan langsung ditutup.**

Kami menggunakan [`rustfmt`][rustfmt] untuk menerapkan format standar pada kode Rust,
dan [`dprint`][dprint] untuk memformat Markdown serta kode non-Rust lainnya.

[rustfmt]: https://github.com/rust-lang/rustfmt
[dprint]: https://dprint.dev

Biasanya `rustfmt` sudah terpasang jika kamu sudah punya toolchain Rust.
Kalau belum ada, kamu bisa memasangnya dengan:

```sh
rustup component add rustfmt
```

Untuk memasang `dprint`, jalankan:

```sh
cargo install dprint
```

Atau ikuti panduan di situs resminya:

[install-dprint]: https://dprint.dev/install/

Untuk memformat kode Rust, jalankan `rustfmt <path ke file>`,
Untuk file lain `dprint fmt <path ke file>`, Banyak editor teks juga sudah punya dukungan bawaan atau
ekstensi untuk `rustfmt` dan `dprint`.

## Mengecek Perbaikan

Buku ini mengikuti jadwal rilis Rust. Jadi, jika kamu melihat masalah di [https://doc.rust-lang.org/stable/book](https://doc.rust-lang.org/stable/book),
bisa jadi masalah tersebut sebenarnya sudah diperbaiki di branch `main`
pada repo ini, tetapi perbaikannya belum melewati tahap nightly -> beta -> stable.
Silakan cek branch `main` di repo ini sebelum melaporkan sebuah isu.

Melihat riwayat untuk file tertentu juga bisa memberikan informasi tambahan
tentang bagaimana atau apakah suatu isu sudah diperbaiki atau belum
jika kamu sedang mencoba memastikannya.

Tolong juga cari isu yang masih terbuka dan yang sudah ditutup, serta PR
yang masih terbuka dan yang sudah ditutup sebelum melaporkan isu baru atau membuka PR baru.

## Lisensi

Repository ini menggunakan lisensi yang sama dengan Rust itu sendiri, yaitu MIT/Apache2.
Kamu bisa menemukan teks lengkap masing-masing lisensi di file `LICENSE-*`
yang ada di repository ini.

## Code of Conduct

Proyek Rust memiliki [code of conduct](http://rust-lang.org/policies/code-of-conduct)
yang berlaku untuk semua sub-proyek, termasuk yang ini. Mohon dihormati dan dipatuhi!

## Ekspektasi

Karena buku ini juga [dicetak][nostarch], dan karena kami ingin
versi online tetap sama dengan versi cetaknya,
mungkin akan membutuhkan waktu lebih lama dari yang biasa kamu alami
bagi kami untuk menangani isu atau pull request milikmu.

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

Sejauh ini, revisi besar biasanya dilakukan bertepatan dengan rilis [Rust Editions](https://doc.rust-lang.org/edition-guide/).
Di antara revisi besar tersebut, kami hanya akan memperbaiki kesalahan.
Jika isu atau pull request-mu tidak benar-benar memperbaiki sebuah kesalahan,
mungkin akan menunggu sampai revisi besar berikutnya\
bisa dalam hitungan bulan hingga tahun. Terima kasih atas kesabarannya!

## Butuh Bantuan

Jika kamu mencari cara untuk membantu tanpa harus banyak membaca atau menulis,
lihatlah [open issue dengan label E-help-wanted][help-wanted].
Biasanya itu berupa perbaikan kecil pada teks, kode Rust, kode frontend,
atau shell script yang bisa membantu kami bekerja lebih efisien
atau meningkatkan kualitas buku ini!

[help-wanted]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3AE-help-wanted

## Terjemahan

Kami akan sangat senang jika kamu ingin membantu menerjemahkan buku ini! Lihat label [Translations] untuk bergabung dengan usaha penerjemahan
yang sedang berjalan. Jika ingin memulai bahasa baru, silakan buka issue baru!
Saat ini kami masih menunggu dukungan multi-bahasa dari [mdbook support]
sebelum bisa menggabungkan terjemahan ke repo utama, tapi jangan ragu untuk memulai terlebih dahulu!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5
