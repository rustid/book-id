## Menerima Argumen Command Line

Mari kita bikin project baru dengan `cargo new`, seperti biasa. Kita bakal 
menamakan project kita `minigrep` buat ngebedain dari alat `grep` yang mungkin 
udah ada di sistem kita.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Tugas pertama adalah bikin `minigrep` menerima dua argumen *command line*-nya: 
_path_ file dan sebuah string yang mau dicari. Yakni, kita mau bisa menjalankan 
program kita pakai `cargo run`, dengan dua tanda hubung (hyphens) buat 
menandakan argumen berikutnya itu buat program kita bukannya buat `cargo`, sebuah 
string buat dicari, dan _path_ ke file tempat nyarinya, kayak gini:

```console
$ cargo run -- searchstring example-filename.txt
```

Saat ini, program yang di-generate sama `cargo new` belum bisa memproses 
argumen yang kita berikan. Beberapa _libraries_ yang udah ada di 
[crates.io](https://crates.io/) bisa bantu nulis program yang menerima argumen 
*command line*, tapi karena kita baru aja belajar konsep ini, mari kita 
mengimplementasikan kemampuan ini sendiri.

### Membaca Nilai Argumen

Biar `minigrep` bisa ngebaca nilai argumen *command line* yang kita masukkan 
ke dalamnya, kita butuh fungsi `std::env::args` yang disediakan di *standard 
library* Rust. Fungsi ini mengembalikan sebuah *iterator* dari argumen-argumen 
*command line* yang diberikan ke `minigrep`. Kita bakal membahas iterators secara 
lengkap di [Bab 13][ch13]. Buat sekarang, kita cuma perlu tahu dua detail soal 
iterators: iterators menghasilkan serangkaian nilai, dan kita bisa memanggil 
method `collect` pada sebuah iterator buat mengubahnya jadi sebuah _collection_ 
(koleksi), kayak vector misalnya, yang berisi semua elemen yang dihasilkan 
sama iterator tersebut.

Kode di Listing 12-1 memungkinkan program `minigrep` kita ngebaca argumen 
*command line* apa pun yang diberikan ke dia, terus mengumpulkan nilai-nilainya 
ke dalam sebuah vector.

<Listing number="12-1" file-name="src/main.rs" caption="Mengumpulkan argumen command line ke dalam sebuah vector dan mencetaknya">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Pertama kita bawa modul `std::env` ke dalam _scope_ pakai *statement* `use` 
biar kita bisa memakai fungsi `args`-nya. Perhatikan kalau fungsi 
`std::env::args` ini bersarang (_nested_) di dua level modul. Kayak yang udah 
kita bahas di [Bab 7][ch7-idiomatic-use], di kasus di mana fungsi yang mau kita 
pakai bersarang di lebih dari satu modul, kita memilih buat membawa induk 
modulnya ke dalam _scope_ ketimbang fungsinya langsung. Dengan melakukan hal itu, 
kita bisa dengan gampang memakai fungsi lain dari `std::env`. Hal ini juga 
mengurangi kebingungan (ambiguity) dibandingkan menambahkan `use std::env::args` 
lalu memanggil fungsinya dengan `args` doang, karena `args` gampang sekali 
disangka fungsi yang didefinisikan di modul saat ini.

> ### Fungsi `args` dan Unicode Tidak Valid
>
> Perhatikan bahwa `std::env::args` bakal *panic* kalau ada argumen yang 
> mengandung karakter Unicode yang tidak valid. Kalau program kita perlu menerima 
> argumen yang mengandung Unicode tidak valid, pakai `std::env::args_os` sebagai 
> gantinya. Fungsi tersebut mengembalikan iterator yang menghasilkan nilai 
> `OsString` bukannya nilai `String`. Kita memilih memakai `std::env::args` 
> di sini biar simpel karena nilai `OsString` itu beda-beda di setiap platform 
> dan lebih rumit buat dikerjain dibanding nilai `String`.

Di baris pertama dari `main`, kita memanggil `env::args`, lalu kita langsung 
memakai `collect` buat mengubah iterator itu jadi vector yang berisi semua nilai 
yang dihasilkan sama iterator-nya. Kita bisa memakai fungsi `collect` buat bikin 
berbagai jenis koleksi, jadi kita harus secara eksplisit menganotasi tipe dari 
`args` buat menentukan kalau kita mau sebuah vector berisi _strings_. Walaupun 
kita jarang sekali perlu menganotasi tipe di Rust, `collect` adalah salah satu 
fungsi yang sering sekali butuh anotasi karena Rust tidak bisa menebak jenis 
koleksi apa yang kita mau.

Terakhir, kita mencetak vector-nya pakai _debug macro_. Mari kita coba 
jalankan kodenya dulu tanpa argumen lalu dengan dua argumen:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Perhatikan kalau nilai pertama di vector itu adalah `"target/debug/minigrep"`, 
yaitu nama dari *binary* kita. Ini sesuai sama perilaku daftar argumen di 
bahasa C, membiarkan program memakai nama yang digunakan saat mereka dipanggil 
dalam eksekusinya. Sering kali praktis punya akses ke nama program kalau 
kita mau mencetaknya di dalam pesan atau ngubah perilaku program berdasarkan 
alias *command line* apa yang dipakai buat memanggil programnya. Tapi buat 
tujuan bab ini, kita abaikan saja itu dan cuma nyimpen dua argumen yang kita 
butuhin.

### Menyimpan Nilai Argumen di dalam Variabel

Program kita saat ini sudah bisa mengakses nilai yang ditentukan sebagai argumen 
*command line*. Sekarang kita perlu menyimpan nilai dari kedua argumen itu di 
dalam variabel supaya kita bisa memakai nilainya di seluruh bagian program. 
Kita lakuin itu di Listing 12-2.

<Listing number="12-2" file-name="src/main.rs" caption="Membuat variabel buat menampung argumen _query_ (pencarian) dan argumen _path_ (jalur) file">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Seperti yang kita lihat saat kita mencetak vector tadi, nama program menempati 
nilai pertama di vector pada `args[0]`, jadi kita mulai ngambil argumennya di 
indeks 1. Argumen pertama yang diterima `minigrep` adalah _string_ yang lagi 
kita cari, jadi kita menaruh referensi ke argumen pertama tersebut di variabel 
`query`. Argumen kedua bakal jadi _path_ file, jadi kita menaruh referensi ke 
argumen kedua tersebut di variabel `file_path`.

Kita sementara mencetak nilai dari variabel-variabel ini buat membuktikan 
kalau kodenya berjalan sesuai keinginan kita. Mari kita jalankan program ini lagi 
dengan argumen `test` dan `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Mantap, programnya jalan! Nilai-nilai argumen yang kita butuhkan sudah disimpan 
ke dalam variabel yang tepat. Nanti kita bakal menambahkan sedikit _error handling_ 
(penanganan error) buat menangani beberapa potensi situasi yang keliru, misalnya 
saat _user_ tidak memberikan argumen apa pun; buat sekarang, kita abaikan dulu 
situasi itu dan lanjut bekerja menambahkan kemampuan membaca file.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
