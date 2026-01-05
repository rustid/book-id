<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## Ngeredirect Error ke Standard Error

Sekarang ini, kita masih nge-print semua output ke terminal pakai macro
`println!`. Di kebanyakan terminal, sebenarnya ada dua jenis output:
*standard output* (`stdout`) buat info biasa dan *standard error* (`stderr`)
buat pesan error. Pemisahan ini bikin user bisa ngirim hasil sukses program ke
file, tapi pesan errornya tetap muncul di layar.

Masalahnya, `println!` cuma bisa nge-print ke standard output, jadi kita perlu
cara lain buat nge-print ke standard error.

### Ngecek Error Sekarang Lagi Dicetak ke Mana

Pertama, kita cek dulu sebenernya konten yang diprint `minigrep` sekarang itu
keluar ke standard output semua, termasuk error yang sebenernya pengen kita
arahin ke standard error. Kita bakal ngetes ini dengan nge-redirect output
standard ke file sambil sengaja bikin error. Kita gak bakal nge-redirect
standard error, jadi kalau ada sesuatu dikirim ke standard error, itu tetap
bakal muncul di layar.

Program command line yang “sopan” itu harusnya ngirim error ke standard error
biar kita masih bisa lihat error di layar meskipun standard output lagi
di-redirect ke file. Tapi program kita sekarang belum sopan nih: kita bakal
lihat kalau dia malah nyimpen pesan error ke file!

Buat nunjukin ini, kita jalanin program pakai tanda `>` dan nama file,
*output.txt*, buat tujuan redirect standard output. Kita gak bakal masukin
argumen apa pun, jadi pasti muncul error:

```console
$ cargo run > output.txt
```

Sintaks `>` ngasih tau shell buat nulis isi standard output ke *output.txt*
bukannya ke layar. Kita gak lihat pesan error yang seharusnya muncul di layar,
berarti kemungkinan besar malah masuk ke file. Ini isi file *output.txt*:

```text
Problem parsing arguments: not enough arguments
```

Yap, error kita lagi-lagi dicetak ke standard output. Jauh lebih berguna kalau
pesan error kayak gini diprint ke standard error, jadi yang masuk file cuma
output sukses aja. Yuk kita benerin.

### Nge-print Error ke Standard Error

Kita bakal pakai kode di Listing 12-24 buat ngubah cara nge-print error.
Untungnya, karena refactoring yang udah kita lakukan sebelumnya, semua kode
yang nge-print error sekarang ada cuma di satu fungsi: `main`. Standard library
udah nyediain macro `eprintln!` yang nge-print ke standard error, jadi kita
tinggal ganti dua pemanggilan `println!` yang dipakai buat error jadi
`eprintln!`.

<Listing number="12-24" file-name="src/main.rs" caption="Nulis pesan error ke standard error bukan standard output pakai `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Sekarang kita jalanin program lagi dengan cara yang sama: tanpa argumen dan
tetap nge-redirect standard output pakai `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Sekarang errornya muncul di layar dan *output.txt* kosong. Ini behavior yang
memang diharapkan dari program command line.

Sekarang kita coba lagi, tapi kali ini pakai argumen yang valid dan tetap
redirect standard output ke file:

```console
$ cargo run -- to poem.txt > output.txt
```

Kita gak bakal lihat output apa pun di terminal, dan isi *output.txt* bakal
jadi:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Ini nunjukin kalau sekarang kita udah bener: standard output dipakai buat hasil
yang sukses, dan standard error khusus buat pesan error.

## Ringkasan

Di chapter ini kita nge-recap beberapa konsep besar yang udah kamu pelajari
sejauh ini dan ngebahas gimana caranya ngelakuin operasi I/O umum di Rust.
Dengan pakai argumen command line, file, environment variable, dan macro
`eprintln!` buat nge-print error, sekarang kamu udah siap bikin aplikasi
command line sendiri.

Digabung sama konsep-konsep di chapter sebelumnya, sekarang kode kamu:

* lebih rapi dan terorganisir,
* bisa nyimpen data dengan struktur yang tepat,
* punya error handling yang oke,
* dan tentu aja, udah bisa dites dengan baik.

Selanjutnya, kita bakal eksplor fitur Rust yang terinspirasi dari bahasa
functional: closures dan iterators.
