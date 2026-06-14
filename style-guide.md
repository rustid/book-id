# Panduan Gaya Penulisan (Style Guide)

## Prosa

- Gunakan *title case* untuk judul bab/seksi, misal: `## Generating a Secret Number` bukan `## Generating a secret number`.
- Gunakan huruf miring ketimbang petik tunggal untuk istilah, misal: `is an *associated function* of`.
- Jangan sertakan tanda kurung saat menyebut method di prosa, misal: `read_line` bukan `read_line()`.
- Lakukan *hard wrap* pada 80 karakter.
- Jangan mencampur kode dan teks biasa dalam satu kata, misal: ``Remember when we wrote `use std::io`?`` bukan ``Remember when we `use`d `std::io`?``.

## Kode

- Tambahkan nama file sebelum blok markdown jika relevan agar jelas file mana yang dibahas.
- Saat mengubah kode, buat bagian yang berubah terlihat jelas (sedang dikembangkan caranya).
- Bagi baris panjang agar tetap di bawah 80 karakter jika memungkinkan.
- Gunakan syntax highlighting `bash` untuk output command line.

## Link

- Jika link tidak untuk dicetak, tandai agar diabaikan.
- Buat link intra-book dan link dokumentasi API stdlib bersifat relatif agar bisa dibaca offline maupun online.
- Gunakan link markdown dan ingat bahwa di versi cetak akan diubah menjadi `teks di *url*`, jadi susunlah kalimatnya agar enak dibaca dalam format tersebut.
