# Tipe Generik, Traits, dan Lifetimes

Setiap bahasa pemrograman punya _tools_ buat menangani duplikasi konsep 
secara efektif. Di Rust, salah satu _tool_ itu adalah _generics_ (generik): 
pengganti abstrak buat tipe konkret atau properti lainnya. Kita bisa 
mengekspresikan perilaku dari generik atau gimana mereka berhubungan dengan 
generik lainnya tanpa perlu tau apa yang bakal menempati posisi mereka saat 
kode di-compile dan dijalankan.

Fungsi bisa menerima parameter dari suatu tipe generik, bukannya tipe konkret 
seperti `i32` atau `String`, dengan cara yang sama seperti mereka menerima 
parameter dengan nilai yang belum diketahui untuk menjalankan kode yang sama di 
beberapa nilai konkret. Sebenarnya, kita sudah memakai generik di Bab 6 dengan 
`Option<T>`, di Bab 8 dengan `Vec<T>` dan `HashMap<K, V>`, serta di Bab 9 
dengan `Result<T, E>`. Di bab ini, kita bakal eksplor gimana cara 
mendefinisikan tipe, fungsi, dan _method_ kita sendiri pakai generik!

Pertama-tama kita bakal mengulang gimana cara mengekstrak sebuah fungsi buat 
mengurangi duplikasi kode. Kemudian kita bakal memakai teknik yang sama buat 
bikin fungsi generik dari dua fungsi yang hanya berbeda di tipe parameternya. 
Kita juga bakal menjelaskan gimana cara memakai tipe generik di dalam definisi 
_struct_ dan _enum_.

Setelah itu, kita bakal belajar gimana cara memakai _traits_ untuk 
mendefinisikan perilaku dengan cara yang generik. Kita bisa menggabungkan 
_traits_ dengan tipe generik untuk membatasi tipe generik agar cuma menerima 
tipe yang punya perilaku tertentu, dan tidak asal menerima tipe apa saja.

Terakhir, kita bakal membahas _lifetimes_: berbagai jenis generik yang memberi 
_compiler_ informasi soal gimana referensi saling berhubungan. _Lifetimes_ 
memungkinkan kita ngasih informasi yang cukup ke _compiler_ soal _borrowed 
values_ (nilai yang dipinjam) agar _compiler_ bisa memastikan referensinya 
bakal valid di lebih banyak situasi yang tidak akan dia sanggup lakukan tanpa 
bantuan kita.

## Menghapus Duplikasi dengan Mengekstrak Fungsi

Generik memungkinkan kita mengganti tipe spesifik pakai _placeholder_ (tempat 
pengganti) yang mewakili banyak tipe buat menghilangkan duplikasi kode. Sebelum 
masuk ke sintaks generik, mari kita lihat dulu gimana cara menghilangkan duplikasi 
pakai cara yang tidak melibatkan tipe generik, yaitu dengan mengekstrak sebuah 
fungsi yang menggantikan nilai spesifik pakai _placeholder_ yang mewakili banyak 
nilai. Habis itu, kita bakal menerapkan teknik yang sama buat mengekstrak fungsi 
generik! Dengan melihat gimana cara mengenali kode duplikat yang bisa diekstrak 
jadi sebuah fungsi, kita bakal mulai terbiasa mengenali kode duplikat yang bisa 
memakai generik.

Kita bakal mulai dari program pendek di Listing 10-1 yang mencari angka paling 
besar di dalam sebuah daftar (_list_).

<Listing number="10-1" file-name="src/main.rs" caption="Mencari angka paling besar di dalam daftar angka">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Kita menyimpan daftar integer di variabel `number_list` dan menaruh referensi 
ke angka pertama di daftar itu di variabel bernama `largest`. Kemudian kita 
iterasi melewati semua angka di daftar itu, dan kalau angka saat ini lebih 
besar dari angka yang disimpan di `largest`, kita mengganti referensi di 
variabel itu. Tapi, kalau angka saat ini lebih kecil atau sama dengan angka 
paling besar yang dilihat sejauh ini, variabelnya tidak berubah, dan kodenya 
lanjut ke angka berikutnya di daftar itu. Setelah memeriksa semua angka di 
daftar, `largest` seharusnya merujuk ke angka paling besar, yang di kasus ini 
adalah 100.

Sekarang kita dapat tugas buat mencari angka paling besar di dua daftar angka 
yang berbeda. Untuk melakukan itu, kita bisa memilih buat menduplikasi kode di 
Listing 10-1 lalu memakai logika yang sama di dua tempat yang berbeda di program, 
seperti yang ditunjukkan di Listing 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Kode buat mencari angka paling besar di *dua* daftar angka">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Walaupun kode ini jalan, menduplikasi kode itu merepotkan dan rawan error. Kita 
juga harus ingat buat meng-update kodenya di banyak tempat kalau kita mau 
mengubahnya.

Buat menghilangkan duplikasi ini, kita bakal bikin abstraksi dengan 
mendefinisikan sebuah fungsi yang beroperasi pada daftar integer apa pun yang 
dimasukkan sebagai parameternya. Solusi ini bikin kode kita lebih jelas dan 
memungkinkan kita mengekspresikan konsep pencarian angka paling besar di sebuah 
daftar secara abstrak.

Di Listing 10-3, kita mengekstrak kode yang mencari angka paling besar ke dalam 
fungsi bernama `largest`. Terus kita panggil fungsi itu buat mencari angka 
paling besar di dua daftar dari Listing 10-2. Kita juga bisa memakai fungsi itu 
di daftar nilai `i32` lain yang mungkin kita punya di masa depan.

<Listing number="10-3" file-name="src/main.rs" caption="Kode yang diabstraksi buat mencari angka paling besar di dua daftar">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

Fungsi `largest` punya parameter bernama `list`, yang mewakili _slice_ konkret 
apa pun dari nilai `i32` yang mungkin kita masukkan ke fungsi tersebut. Hasilnya, 
pas kita memanggil fungsinya, kodenya jalan di nilai-nilai spesifik yang kita 
masukkan.

Sebagai ringkasan, ini langkah-langkah yang kita ambil buat ngubah kode dari 
Listing 10-2 jadi Listing 10-3:

1. Kenali kode yang terduplikasi.
1. Ekstrak kode yang terduplikasi ke dalam body fungsi, lalu tentukan input 
   dan nilai kembalian dari kode itu di _signature_ fungsinya.
1. Update dua instance kode yang terduplikasi buat memanggil fungsinya sebagai gantinya.

Berikutnya, kita bakal pakai langkah-langkah yang sama ini dengan generik buat 
mengurangi duplikasi kode. Sama seperti body fungsi yang bisa beroperasi di `list` 
abstrak bukannya nilai spesifik, generik memungkinkan kode buat beroperasi di 
tipe abstrak.

Misalnya, katakanlah kita punya dua fungsi: satu yang mencari item paling besar 
di _slice_ nilai `i32` dan satu lagi yang mencari item paling besar di _slice_ 
nilai `char`. Gimana cara kita menghilangkan duplikasi itu? Mari kita cari tahu!
