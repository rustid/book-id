# Project I/O: Bikin Program Command Line

Bab ini adalah rangkuman dari banyak *skill* yang udah kita pelajari sejauh ini 
dan sebuah eksplorasi ke beberapa fitur *standard library* lainnya. Kita bakal 
bikin alat (tool) *command line* yang berinteraksi sama file dan input/output 
dari *command line* buat melatih beberapa konsep Rust yang sekarang udah kita 
kuasai.

Kecepatan, keamanan, output *binary* tunggal, dan dukungan lintas platform 
bikin Rust jadi bahasa yang ideal buat bikin alat *command line*, jadi buat 
project kita ini, kita bakal bikin versi kita sendiri dari alat pencarian 
*command line* klasik `grep` (**g**lobally search a **r**egular **e**xpression 
and **p**rint). Di skenario penggunaan paling sederhana, `grep` mencari _string_ 
tertentu di dalam file yang ditentukan. Buat ngelakuin itu, `grep` menerima 
_path_ (jalur) file dan sebuah _string_ sebagai argumennya. Lalu dia ngebaca 
file tersebut, nyari baris-baris di file itu yang mengandung argumen _string_ 
tadi, terus mencetak baris-baris itu.

Di sepanjang jalan, kita bakal nunjukin gimana cara bikin alat *command line* 
kita memakai fitur terminal yang dipakai sama banyak alat *command line* 
lainnya. Kita bakal ngebaca nilai dari _environment variable_ buat ngebolehin 
_user_ mengonfigurasi perilaku alat kita. Kita juga bakal mencetak pesan error 
ke *stream* konsol *standard error* (`stderr`) bukannya *standard output* 
(`stdout`) sehingga, misalnya, _user_ bisa me-*redirect* (mengalihkan) output 
yang sukses ke sebuah file tapi tetap bisa melihat pesan error di layar.

Salah satu anggota komunitas Rust, Andrew Gallant, udah bikin versi `grep` yang 
berfitur lengkap dan kenceng sekali, namanya `ripgrep`. Sebagai perbandingan, 
versi kita bakal lumayan sederhana, tapi bab ini bakal ngasih kita beberapa 
pengetahuan dasar yang kita butuhin buat paham project dunia nyata kayak 
`ripgrep`.

Project `grep` kita bakal nggabungin sejumlah konsep yang udah kita pelajari 
sejauh ini:

- Mengorganisasi kode ([Bab 7][ch7])
- Memakai vectors dan strings ([Bab 8][ch8])
- Menangani errors ([Bab 9][ch9])
- Memakai traits dan lifetimes di tempat yang tepat ([Bab 10][ch10])
- Nulis pengujian (tests) ([Bab 11][ch11])

Kita juga bakal ngenalin secara singkat _closures_, _iterators_, dan _trait 
objects_, yang bakal dibahas lebih detail di [Bab 13][ch13] dan [Bab 18][ch18].

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
