## Alur Kontrol Ringkas dengan `if let`

Dengan `if let` syntaks anda bisa mengkombinasikan `if` dan `let` menjadi lebih sederhana 
untuk menangani hanya satu pola nilai yang cocok dan mengabaikan nilai sisanya. 
Pertimbangkan program di Daftar 6-6 yang cocok dengan nilai `Option<u8>` di variabel
`config_max` tetapi hanya ingin mengeksekusi kode jika nilainya adalah variant `Some`.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

<span class="caption">Daftar 6-6: Sebuah `match` yang hanya mengeksekusi kode 
ketika nilai adalah `Some`</span>

Jika nilainya adalah `Some`, kita mencetak nilai dalam varian `Some` dengan mengikat
nilai ke variabel `max` yang ada pada pola. Kami tidak ingin melakukan apa pun
dengan nilai `None`. Untuk memenuhi ekspresi `match`, kita harus menambahkan `_ =>
()` setelah memproses hanya satu variant, ini adalah merupakan kode boilerplate 
yang cukup mengganggu.

Sebagai gantinya, kita bisa menulisnya dengan cara yang lebih singkat menggunakan
`if let`. Berikut kode yang berperilaku sama dengan `match` di Daftar 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

Sintaks `if let` mengambil pola dan ekspresi yang dipisahkan dengan tanda persamaan.
Cara kerjanya sama seperti `match`, dimana ekspresi diberikan kepada `match` dan 
polanya adalah lengan pertamanya. Dalam hal ini, polanya adalah `Some(max)`, dan `max`
mengikat nilai di dalam `Some`. Kita bisa menggunakan `max` di badan blok `if let`
dengan cara yang sama seperti kita menggunakan `max` di lengan `match` yang cocok. 
Kode di blok `if let` tidak dijalankan jika nilainya tidak sesuai dengan pola.

Menggunakan `if let` berarti mengurangi penulisan, mengurangi indentasi, dan 
mengurangi kode boilerplate. Namun, Anda kehilangan pemeriksaan menyeluruh 
yang diterapkan oleh `match`. Memilih antara `match` dan `if let` bergantung 
pada apa yang Anda lakukan di situasi khusus dan apakah mendapatkan keringkasan
merupakan trade-off yang tepat dari kehilangan pemeriksaan menyeluruh.

Dengan kata lain, Anda dapat menganggap `if let` sebagai sintaksis sugar untuk `match`
itu akan menjalankan kode ketika nilainya cocok dengan satu pola dan kemudian 
mengabaikan semua nilai lainnya.

Kita dapat memasukkan `else` dengan `if let`. Blok kode yang menyertai `else` sama
dengan blok kode yang disertakan dengan simbol `_` di dalam Ekspresi `match` 
yang setara dengan `if let` dan `else`. Ingat definisi enum `Coin` di Daftar 6-4, 
di mana variant `Quarter` memiliki nilai `UsState`. Jika kita ingin menghitung semua
koin non-seperempat yang kita lihat juga mengumumkan keadaan kuartal, kita bisa 
melakukannya dengan ekspresi `match`, seperti ini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Atau kita bisa menggunakan ekspresi `if let` dan `else`, seperti ini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

Jika Anda menghadapi situasi di mana program Anda memiliki logika yang terlalu 
bertele-tele saat menggunakan ekspresi `match`, ingatlah bahwa `if let` juga ada 
di kotak peralatan Rust Anda.

## Ringkasan

Kami sekarang telah membahas cara menggunakan enum untuk membuat tipe khusus yang
dapat menjadi salah satu dari sebuah kumpulan nilai yang disebutkan. Kami telah menunjukkan
bagaimana tipe `Option<T>` dari perpustakaan standar membantu Anda menggunakan type system
untuk mencegah kesalahan. Ketika nilai enum dimiliki data di dalamnya, Anda dapat menggunakan 
`match` atau `if let` untuk mengekstrak dan menggunakannya nilainya, tergantung pada berapa
banyak kasus yang perlu Anda tangani.

Program Rust Anda sekarang dapat mengekspresikan konsep di domain anda menggunakan
struct dan enum. Membuat tipe khusus untuk digunakan di API Anda memastikan keamanan
tipenya: Kompiler akan memastikan fungsi anda hanya mendapatkan nilai dari tipe yang 
masing-masing fungsi harapkan.

Untuk menyediakan API yang terorganisir dengan baik kepada pengguna anda dengan mudah
untuk menggunakannya dan hanya menampilkan apa yang dibutuhkan pengguna anda, sekarang
mari kita beralih ke Modul Rust.

