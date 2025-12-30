# Panduan Gaya

## Prosa

- Utamakan **Title Case** untuk judul bab/bagian, misalnya:
  `## Generating a Secret Number` daripada `## Generating a secret number`.
- Lebih utamakan _italic_ daripada tanda kutip tunggal saat menyoroti istilah, misalnya:
  `is an *associated function* of` daripada `is an ‘associated function’ of`.
- Saat membahas sebuah method dalam prosa, JANGAN sertakan tanda kurung, misalnya:
  `read_line` daripada `read_line()`.
- Hard wrap di 80 karakter.
- Usahakan tidak mencampur kode dan non-kode dalam satu kata, misalnya:
  ``Remember when we wrote `use std::io`?`` daripada ``Remember when we `use`d `std::io`?``

## Kode

- Tambahkan nama file sebelum blok markdown untuk memperjelas
  file mana yang dibahas, jika relevan.
- Saat membuat perubahan kode, buat jelas bagian mana yang berubah
  dan mana yang tetap sama… (belum yakin cara terbaiknya)
- Pecah baris yang terlalu panjang seperlunya agar tetap di bawah 80 karakter jika memungkinkan.
- Gunakan penyorotan sintaks `bash` untuk blok kode output command line.

## Tautan

Jika semua script sudah selesai:

- Jika sebuah tautan tidak seharusnya dicetak, tandai agar diabaikan
  - Ini termasuk semua tautan intra-buku “Chapter XX”, yang _harusnya_
    berupa tautan untuk versi HTML
- Buat tautan intra-buku dan tautan dokumentasi API stdlib menjadi relatif
  agar tetap berfungsi baik saat buku dibaca offline maupun di docs.rust-lang.org
- Gunakan tautan markdown dan ingat bahwa tautan tersebut akan diubah menjadi
  `teks di *url*` pada versi cetak, jadi susunlah kalimatnya agar tetap enak dibaca dalam format tersebut
