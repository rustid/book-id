# Proyek I/O: Membangun Program Command Line

Bab ini adalah rekap dari banyak skill yang sudah kamu pelajari sejauh ini
sekaligus eksplorasi beberapa fitur tambahan dari standard library. Kita akan
membangun sebuah **command line tool** yang berinteraksi dengan file dan
input/output terminal untuk mempraktikkan konsep Rust yang sekarang sudah kamu
kuasai.

Kecepatan Rust, keamanannya, hasil berupa satu binary, dan dukungan lintas
platform membuat Rust jadi bahasa yang ideal untuk bikin command line tool.
Jadi, di proyek kali ini kita akan bikin versi sederhana dari tool classic
bernama `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint).
Dalam penggunaan paling sederhana, `grep` akan mencari string tertentu di dalam
file tertentu. `grep` menerima dua argumen: path file dan string yang mau
dicari. Lalu tool ini membaca file tersebut, mencari baris yang mengandung
string itu, dan menampilkan baris-baris yang cocok.

Sepanjang prosesnya, kita juga bakal belajar gimana bikin command line tool kita
terasa “profesional”, seperti tool CLI lain pada umumnya. Kita akan:

- membaca nilai environment variable untuk mengonfigurasi perilaku tool,
- menampilkan error ke stream **stderr** (standard error), bukan `stdout`,
  supaya user bisa mengalihkan output normal ke file sambil tetap melihat pesan
  error di layar.

Di komunitas Rust sendiri, sudah ada anggota bernama Andrew Gallant yang bikin
versi `grep` yang jauh lebih lengkap dan super cepat bernama **ripgrep**.
Dibandingkan itu, versi kita bakal jauh lebih sederhana. Tapi bab ini bakal
memberi kamu fondasi untuk bisa memahami proyek dunia nyata seperti `ripgrep`.

Proyek `grep` kita bakal menggabungkan berbagai konsep yang sudah kamu pelajari:

- Mengorganisasi kode ([Chapter 7][ch7]<!-- ignore -->)
- Menggunakan vector dan string ([Chapter 8][ch8]<!-- ignore -->)
- Menangani error ([Chapter 9][ch9]<!-- ignore -->)
- Menggunakan traits dan lifetimes dengan tepat
  ([Chapter 10][ch10]<!-- ignore -->)
- Menulis test ([Chapter 11][ch11]<!-- ignore -->)

Selain itu, kita juga bakal sekilas mengenalkan closures, iterators, dan trait
objects, yang akan dibahas lebih dalam di:

- [Chapter 13][ch13]<!-- ignore -->
- [Chapter 18][ch18]<!-- ignore -->

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
