# Fitur Bahasa Fungsional: Iterators dan Closures

Desain Rust mengambil inspirasi dari banyak bahasa dan teknik pemrograman yang 
sudah ada, dan salah satu pengaruh yang signifikan adalah _functional 
programming_ (pemrograman fungsional). Pemrograman dengan gaya fungsional sering 
kali mencakup penggunaan fungsi sebagai nilai, yaitu dengan memasukkan fungsi 
sebagai argumen, mengembalikannya dari fungsi lain, menaruhnya ke variabel untuk 
dieksekusi nanti, dan seterusnya.

Di bab ini, kita tidak akan berdebat soal apa itu pemrograman fungsional atau 
apa yang bukan, tapi kita bakal membahas beberapa fitur Rust yang mirip dengan 
fitur-fitur di banyak bahasa pemrograman yang sering disebut sebagai bahasa 
fungsional.

Secara lebih spesifik, kita bakal membahas:

- _Closures_, sebuah struktur mirip fungsi yang bisa kita simpan di dalam sebuah 
  variabel
- _Iterators_, sebuah cara buat memproses serangkaian elemen
- Gimana cara memakai _closures_ dan _iterators_ buat meningkatkan project I/O 
  yang kita buat di Bab 12
- Performa dari _closures_ dan _iterators_ (bocoran: performa mereka lebih cepat 
  dari yang mungkin kita bayangkan!)

Kita sebenarnya sudah membahas beberapa fitur Rust lainnya, seperti _pattern 
matching_ dan _enums_, yang juga terpengaruh oleh gaya fungsional. Karena 
menguasai _closures_ dan _iterators_ adalah bagian penting dari menulis kode Rust 
yang idiomatik dan kencang, kita bakal mendedikasikan seluruh bab ini buat 
membahas mereka.
