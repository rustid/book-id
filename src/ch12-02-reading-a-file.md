## Membaca File

Sekarang kita bakal menambahkan fungsionalitas untuk membaca file yang ditentukan di argumen `file_path`. Pertama kita butuh sebuah file contoh buat mengujinya: kita bakal memakai file dengan sedikit teks yang membentang di beberapa baris dan punya beberapa kata yang diulang. Listing 12-3 punya puisi Emily Dickinson yang bakal pas sekali! Buat sebuah file bernama _poem.txt_ di tingkat *root* project kita, lalu masukkan puisi “I’m Nobody! Who are you?”

<Listing number="12-3" file-name="poem.txt" caption="Sebuah puisi dari Emily Dickinson jadi *test case* yang bagus.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Dengan teks yang sudah siap, edit _src/main.rs_ dan tambahkan kode buat membaca file tersebut, seperti yang ditunjukkan di Listing 12-4.

<Listing number="12-4" file-name="src/main.rs" caption="Membaca isi dari file yang ditentukan oleh argumen kedua">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Pertama kita membawa bagian yang relevan dari *standard library* dengan *statement* `use`: kita butuh `std::fs` buat menangani file.

Di `main`, *statement* baru `fs::read_to_string` menerima `file_path`, membuka file tersebut, dan mengembalikan nilai bertipe `std::io::Result<String>` yang berisi konten filenya.

Setelah itu, kita kembali menambahkan *statement* `println!` sementara yang mencetak nilai dari `contents` setelah file dibaca, jadi kita bisa mengecek kalau programnya berfungsi dengan baik sejauh ini.

Mari kita jalankan kode ini dengan sembarang string sebagai argumen *command line* pertama (karena kita belum mengimplementasikan bagian pencariannya) dan file _poem.txt_ sebagai argumen kedua:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Mantap! Kodenya berhasil membaca dan kemudian mencetak isi dari filenya. Tapi kodenya punya beberapa kelemahan. Saat ini, fungsi `main` punya banyak tanggung jawab: umumnya, fungsi bakal lebih jelas dan gampang dipelihara (maintain) kalau tiap fungsi bertanggung jawab untuk satu ide saja. Masalah lainnya adalah kita belum menangani error sebaik yang kita bisa. Programnya masih kecil, jadi kelemahan-kelemahan ini belum jadi masalah besar, tapi seiring programnya makin besar, bakal lebih susah untuk memperbaikinya dengan rapi. Praktik yang bagus adalah mulai me-*refactor* sejak awal saat mengembangkan program karena bakal jauh lebih mudah untuk me-*refactor* jumlah kode yang lebih sedikit. Kita bakal melakukan itu selanjutnya.
