# Fitur-fitur Tingkat Lanjut (Advanced Features)

Sekarang, kita udah mempelajari bagian-bagian yang paling sering dipakai di 
bahasa pemrograman Rust. Sebelum kita mengerjakan satu project lagi di Bab 21, 
kita bakal melihat beberapa aspek dari bahasa ini yang mungkin bakal kita temui 
sekali-sekali, biarpun kita mungkin tidak memakainya setiap hari. Kita bisa 
memakai bab ini sebagai referensi saat kita ketemu hal-hal yang belum kita 
ketahui. Fitur-fitur yang dibahas di sini sangat berguna buat situasi-situasi 
yang sangat spesifik. Walaupun kita mungkin jarang memakainya, kita pengen 
memastikan kita punya pemahaman soal semua fitur yang ditawarkan oleh Rust.

Di bab ini, kita bakal membahas:

- Unsafe Rust: gimana cara keluar (opt out) dari beberapa jaminan yang dikasih 
  Rust dan mengambil tanggung jawab buat menjunjung tinggi jaminan-jaminan itu 
  secara manual
- Advanced traits: _associated types_, _default type parameters_, _fully qualified 
  syntax_, _supertraits_, dan _newtype pattern_ (pola tipe baru) sehubungan 
  dengan traits
- Advanced types: lebih banyak lagi soal _newtype pattern_, _type aliases_ 
  (alias tipe), tipe `never` (tipe tak pernah), dan _dynamically sized types_ 
  (tipe-tipe yang berukuran dinamis)
- Advanced functions dan closures: _function pointers_ (pointer fungsi) dan 
  mengembalikan _closures_
- Macros: berbagai cara buat mendefinisikan kode yang membikin lebih banyak 
  kode lagi saat _compile time_

Ini adalah sekumpulan fitur Rust yang punya sesuatu buat semua orang! Mari 
kita selami!
