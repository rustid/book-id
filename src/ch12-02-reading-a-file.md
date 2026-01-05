## Membaca File

Sekarang kita bakal nambahin fitur buat membaca file yang disebutkan di argumen
`file_path`. Pertama, kita butuh file contoh buat ngetesnya. Kita akan pakai
file berisi teks pendek dengan beberapa baris dan beberapa kata yang berulang.
Listing 12-3 berisi puisi Emily Dickinson yang pas banget buat ini!

Buat file bernama _**poem.txt**_ di root project kamu, lalu isi dengan puisi
**“I’m Nobody! Who are you?”**

<Listing number="12-3" file-name="poem.txt" caption="Puisi karya Emily Dickinson yang cocok sebagai bahan uji.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Sekarang setelah teksnya sudah siap, edit file _**src/main.rs**_ dan tambahkan
kode untuk membaca file, seperti yang ditunjukkan pada Listing 12-4.

<Listing number="12-4" file-name="src/main.rs" caption="Membaca isi file yang ditentukan oleh argumen kedua">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Pertama, kita masukin bagian yang relevan dari standard library pakai `use`:
kita butuh `std::fs` untuk menangani file.

Di dalam `main`, perintah baru `fs::read_to_string` menerima `file_path`,
membuka file tersebut, lalu mengembalikan nilai bertipe
`std::io::Result<String>` yang berisi isi file.

Setelah itu, kita tambahkan lagi `println!` sementara untuk nge-print nilai
`contents` setelah file dibaca, supaya kita bisa cek apakah programnya sudah
jalan sesuai harapan.

Sekarang coba jalankan programnya—isi argumen pertama dengan string apa saja
(karena fitur pencariannya belum kita implementasi), dan argumen kedua dengan
nama file **poem.txt**:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Baik! Kode tersebut berhasil membaca lalu mencetak isi file. Namun, kode ini
memiliki beberapa kekurangan. Saat ini, fungsi `main` memiliki banyak tanggung
jawab sekaligus: secara umum, fungsi akan lebih jelas dan lebih mudah dirawat
jika masing-masing hanya bertanggung jawab pada satu hal. Masalah lainnya adalah
kita belum menangani error sebaik mungkin. Program ini memang masih kecil, jadi
kekurangan tersebut belum menjadi masalah besar, tetapi ketika program
berkembang, akan lebih sulit untuk memperbaikinya dengan rapi. Merupakan praktik
yang baik untuk mulai melakukan refactoring sejak awal saat mengembangkan
program karena akan jauh lebih mudah melakukan refactor pada kode yang masih
sedikit. Kita akan melakukannya selanjutnya.
