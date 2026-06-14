## Mengontrol Bagaimana Pengujian Dijalankan

Sama halnya dengan `cargo run` yang men-compile kode kita lalu menjalankan *binary* hasilnya, `cargo test` men-compile kode kita dalam mode pengujian (*test mode*) lalu menjalankan *test binary* hasilnya. Perilaku default dari *binary* yang dihasilkan oleh `cargo test` adalah menjalankan semua pengujian secara paralel dan menangkap (capture) output yang dihasilkan selama pengujian berjalan, yang mana mencegah output tersebut ditampilkan sehingga kita lebih gampang membaca output yang berkaitan dengan hasil pengujian. Akan tetapi, kita bisa menentukan opsi baris perintah (*command line options*) untuk mengubah perilaku default ini.

Beberapa opsi baris perintah diteruskan ke `cargo test`, dan beberapa diteruskan ke *test binary* yang dihasilkan. Untuk memisahkan dua jenis argumen ini, kita mendaftarkan argumen yang diteruskan ke `cargo test`, diikuti dengan pemisah `--` (dua tanda hubung), lalu argumen yang diteruskan ke *test binary*. Menjalankan `cargo test --help` menampilkan opsi-opsi yang bisa kita pakai bareng `cargo test`, dan menjalankan `cargo test -- --help` menampilkan opsi-opsi yang bisa kita pakai setelah pemisah. Opsi-opsi tersebut juga didokumentasikan di [bagian “Tests”][tests] di [buku rustc][rustc].

[tests]: https://doc.rust-lang.org/rustc/tests/index.html
[rustc]: https://doc.rust-lang.org/rustc/index.html

### Menjalankan Pengujian secara Paralel atau Berurutan

Saat kita menjalankan banyak pengujian, secara default mereka berjalan secara paralel menggunakan *threads* (utas), yang berarti mereka selesai dijalankan lebih cepat dan kita bisa dapat *feedback* lebih cepat. Karena pengujian-pengujian ini berjalan di saat yang bersamaan, kita harus memastikan kalau pengujian kita tidak saling bergantung satu sama lain atau bergantung pada *shared state* (state/keadaan yang dibagikan) apa pun, termasuk lingkungan (*environment*) yang dibagikan, seperti direktori kerja saat ini (current working directory) atau variabel lingkungan (*environment variables*).

Misalnya, katakanlah setiap pengujian kita menjalankan beberapa kode yang membuat file di diska bernama _test-output.txt_ lalu menulis beberapa data ke file tersebut. Kemudian tiap pengujian membaca data di file itu dan menegaskan (*asserts*) kalau file tersebut mengandung nilai tertentu, yang mana nilainya beda-beda di tiap pengujian. Karena pengujian berjalan di saat yang bersamaan, satu pengujian mungkin bakal menimpa (*overwrite*) file itu di antara waktu pengujian lain sedang menulis dan membaca file tersebut. Pengujian kedua bakal gagal, bukan karena kodenya salah tapi karena pengujian-pengujian itu saling mengintervensi satu sama lain saat dijalankan secara paralel. Salah satu solusinya adalah memastikan tiap pengujian menulis ke file yang berbeda-beda; solusi lainnya adalah menjalankan pengujian satu per satu.

Kalau kita tidak mau menjalankan pengujian secara paralel atau kalau kita mau kontrol yang lebih mendetail terhadap jumlah *threads* yang dipakai, kita bisa mengirim *flag* `--test-threads` beserta jumlah *threads* yang mau kita pakai ke *test binary*. Coba lihat contoh berikut:

```console
$ cargo test -- --test-threads=1
```

Kita nge-set jumlah *test threads* jadi `1`, memberi tahu program untuk tidak menggunakan paralelisme apa pun. Menjalankan pengujian pakai satu *thread* bakal makan waktu lebih lama ketimbang menjalankannya secara paralel, tapi pengujiannya tidak bakal saling mengintervensi kalau mereka membagikan *state*.

### Menampilkan Output Fungsi

Secara default, kalau sebuah pengujian sukses, *library* pengujian Rust menangkap (*captures*) apa pun yang dicetak ke *standard output*. Misalnya, kalau kita memanggil `println!` di sebuah pengujian dan pengujian itu sukses, kita tidak bakal melihat output `println!` di terminal; kita cuma bakal melihat baris yang menandakan kalau pengujiannya sukses. Kalau pengujiannya gagal, barulah kita bakal melihat apa pun yang tadi dicetak ke *standard output* beserta sisa pesan kegagalannya.

Sebagai contoh, Listing 11-10 punya fungsi konyol yang mencetak nilai dari parameternya lalu mengembalikan nilai 10, serta punya sebuah pengujian yang sukses dan satu lagi pengujian yang gagal.

<Listing number="11-10" file-name="src/lib.rs" caption="Pengujian untuk fungsi yang memanggil `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Saat kita menjalankan pengujian ini pakai `cargo test`, kita bakal melihat output berikut:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Perhatikan bahwa di dalam output ini kita tidak melihat `I got the value 4`, yang dicetak saat pengujian yang sukses itu berjalan. Output tersebut sudah ditangkap (*captured*). Output dari pengujian yang gagal, `I got the value 8`, muncul di bagian ringkasan output pengujian, yang juga menunjukkan penyebab dari kegagalan pengujiannya.

Kalau kita mau melihat nilai yang dicetak dari pengujian yang sukses juga, kita bisa memberi tahu Rust buat juga menampilkan output dari pengujian yang sukses dengan `--show-output`:

```console
$ cargo test -- --show-output
```

Saat kita menjalankan pengujian di Listing 11-10 lagi pakai *flag* `--show-output`, kita melihat output berikut:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Menjalankan Sebagian Pengujian Berdasarkan Nama

Kadang-kadang, menjalankan *test suite* yang lengkap itu bisa makan waktu lama. Kalau kita lagi mengerjakan kode di area tertentu, kita mungkin mau menjalankan hanya pengujian yang berkaitan dengan kode tersebut aja. Kita bisa memilih pengujian mana yang mau dijalankan dengan mengirimkan nama atau nama-nama dari pengujian yang mau dijalankan sebagai argumen ke `cargo test`.

Buat mendemonstrasikan gimana cara menjalankan sebagian pengujian, pertama-tama kita bakal bikin tiga pengujian buat fungsi `add_two` kita, seperti yang ditunjukkan di Listing 11-11, lalu kita pilih mana yang mau dijalankan.

<Listing number="11-11" file-name="src/lib.rs" caption="Tiga pengujian dengan tiga nama yang berbeda">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Kalau kita menjalankan pengujian tanpa mengirim argumen apa pun, seperti yang kita lihat sebelumnya, semua pengujian bakal berjalan secara paralel:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Menjalankan Pengujian Tunggal

Kita bisa mengirimkan nama dari fungsi pengujian mana pun ke `cargo test` untuk menjalankan cuma pengujian itu saja:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Hanya pengujian dengan nama `one_hundred` yang jalan; dua pengujian lainnya tidak cocok sama nama itu. Output pengujian memberi tahu kita kalau kita punya lebih banyak pengujian yang tidak dijalankan dengan menampilkan `2 filtered out` di akhir.

Kita tidak bisa menentukan nama dari beberapa pengujian pakai cara ini; cuma nilai pertama yang diberikan ke `cargo test` aja yang bakal dipakai. Tapi ada cara buat menjalankan beberapa pengujian.

#### Menyaring untuk Menjalankan Beberapa Pengujian

Kita bisa menentukan sebagian dari nama pengujian, dan pengujian apa pun yang namanya cocok sama nilai tersebut bakal dijalankan. Misalnya, karena dua dari nama pengujian kita mengandung kata `add`, kita bisa menjalankan kedua pengujian itu dengan menjalankan `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Perintah ini menjalankan semua pengujian yang punya `add` di namanya dan menyaring (*filtered out*) pengujian bernama `one_hundred`. Perhatikan juga bahwa modul tempat pengujian itu berada menjadi bagian dari nama pengujian tersebut, jadi kita bisa menjalankan semua pengujian di dalam sebuah modul dengan menyaringnya berdasarkan nama modul.

### Mengabaikan Beberapa Pengujian Kecuali Diminta Secara Spesifik

Kadang-kadang ada beberapa pengujian spesifik yang sangat memakan waktu untuk dieksekusi, jadi kita mungkin mau mengecualikan pengujian-pengujian itu selama sebagian besar sesi `cargo test`. Ketimbang mendaftarkan semua pengujian yang *ingin* kita jalankan sebagai argumen, kita bisa menganotasi pengujian yang makan waktu tersebut dengan atribut `ignore` buat mengecualikannya, seperti yang ditunjukkan di sini:

<span class="filename">Nama file: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

Setelah `#[test]`, kita menambahkan baris `#[ignore]` ke pengujian yang mau kita kecualikan. Sekarang pas kita menjalankan pengujian kita, `it_works` bakal jalan, tapi `expensive_test` tidak:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Fungsi `expensive_test` didaftarkan sebagai `ignored` (diabaikan). Kalau kita mau menjalankan hanya pengujian yang diabaikan, kita bisa memakai `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Dengan mengontrol pengujian mana aja yang dijalankan, kita bisa memastikan hasil `cargo test` kita bakal keluar dengan cepat. Saat kita berada di titik di mana rasanya masuk akal untuk mengecek hasil dari pengujian yang `ignored` dan kita punya waktu buat menunggu hasilnya, kita bisa menjalankan `cargo test -- --ignored` alih-alih perintah biasa. Kalau kita mau menjalankan semua pengujian baik itu diabaikan atau tidak, kita bisa menjalankan `cargo test -- --include-ignored`.