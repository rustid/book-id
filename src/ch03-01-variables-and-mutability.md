## Variabel dan Mutabilitas

Sebagaimana disinggung di bagian ["Menyimpan Nilai dengan 
Variabel"][storing-values-with-variables]<!-- ignore -->, secara baku, 
variabel itu immutable. Ini adalah satu dari banyak dorongan yang Rust berikan
kepada Anda untuk menulis kode dengan cara yang memanfaatkan keamanan dan
konkurensi mudah yang ditawarkan oleh Rust. Namun Anda masih punya pilihan
untuk membuat variabel Anda mutable. Mari kita eksplorasi bagaimana dan
mengapa Rust mendorong Anda agar lebih suka atas imutabilitas dan mengapa
kadang Anda perlu untuk tidak memilihnya.

Ketika suatu variabel immutable, sekali suatu nilai diikat ke sebuah nama,
Anda tidak bisa mengubah nilai tersebut. Untuk mengilustrasikan ini, buatlah
sebuah proyek baru bernama *variables* dalam direktori *projects* Anda dengan
memakai `cargo new variables`.

Lalu, dalam direktori *variables* baru Anda, buka *src/main.rs* dan ganti
kodenya dengan kode berikut, yang kini belum bisa dikompilasi:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Simpan dan jalankan program memakai `cargo run`. Anda mestinya memperoleh
pesan kesalahan terkait kesalahan imutabilitas, seperti ditunjukkan dalam
keluaran ini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Contoh ini menunjukkan bagaimana compiler membantu Anda menemukan kesalahan
dalam program Anda. Kesalahan compiler bisa membuat frustrasi, tetapi 
sebenarnya mereka hanya berarti bahwa program Anda belum secara aman
melakukan apa yang Anda ingin dia lakukan; mereka *bukan* berarti bahwa Anda 
bukanlah programmer yang baik! Rustacean yang berpengalaman masih mendapat
kesalahan compiler.

Anda menerima pesan kesalahan `` cannot assign twice to immutable variable `x`
`` karena Anda mencoba menugaskan suatu nilai kedua ke variabel `x` yang
immutable.

Penting bahwa kita mendapat kesalahan saat compile ketika kita mencoba
mengubah suatu nilai yang ditugaskan sebagai immutable karena situasi seperti
ini dapat mengarah ke bug. Bila satu bagian dari kode kita beroperasi pada
asumsi bahwa suatu nilai tidak akan pernah berubah dan bagian lain dari kode
kita mengubah nilai tersebut, mungkin bahwa bagian pertama kode tidak akan
melakukan apa yang itu dirancang untuk melakukannya. Penyebab bug jenis ini
bisa sulit dilacak setelah fakta, khususnya ketika penggalan kedua kode
hanya *kadang-kadang* mengubah nilai. Compiler Rust menjamin bahwa ketika
Anda menyatakan bahwa suatu nilai tidak akan berubah, itu benar-benar tidak
akan berubah, sehingga Anda tidak perlu melacaknya sendiri. Maka kode Anda
lebih mudah dipahami.

Tapi mutabilitas bisa sangat berguna, dan dapat membuat kode lebih nyaman
ditulis. Walaupun variabel secara default immutable, Anda dapat membuat mereka
mutable dengan menambahkan `mut` di depan nama variabel seperti yang telah
Anda lakukan dalam [Bab 2][storing-values-with-variables]<!-- ignore -->. 
Menambahkan `mut` juga menyampaikan maksud ke pembaca kode di masa depan
dengan mengindikasikan bahwa bagian lain dari kode akan mengubah nilai
variabel ini.

Sebagai contoh, mari kita ubah *src/main.rs* menjadi yang berikut:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Saat kita sekarang menjalankan program, kita memperoleh ini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Kita diizinkan mengubah nilai yang diikat ke `x` dari `5` ke `6` ketika 
`mut` dipakai. Pada akhirnya, menentukan apakah memakai mutabilitas atau 
tidak terserah padamu dan tergantung kepada apa yang Anda pikir paling 
jelas dalam situasi tersebut.

### Konstanta

Seperti variable yang immutable, *konstanta* adalah nilai-nilai yang terikat
ke suatu nama dan tidak diizinkan berubah, tapi ada beberapa perbedaan
antara konstanta dan variabel.

Pertama, Anda tidak diizinkan memakai `mut` dengan konstanta. Konstanta tidak
hanya sekadar immutable secara baku — mereka selalu immutable. Anda
mendeklarasikan konstanta memakai kata kunci `const` bukan kata kunci `let`,
dan tipe nilai *harus* dianotasikan. Kita akan membahas tipe dan anotasi tipe
dalam bagian selanjutnya, ["Tipe Data,"][data-types]<!-- ignore -->, jadi
jangan khawatir tentang detilnya sekarang. Ketahui saja bahwa Anda mesti selalu
menganotasi tipe.

Konstanta dapat dideklarasikan dalam sebarang skup, termasuk skup global, yang
membuat mereka berguna untuk nilai-nilai yang banyak bagian kode perlu tahu
tentangnya.

Perbedaan terakhir adalah konstanta dapat diatur hanya ke suatu ekspresi
konstan, bukan hasil dari suatu nilai yang hanya dapat dihitung saat runtime.

Ini adalah sebuah contoh dari deklarasi konstanta:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Nama konstantanya adalah `THREE_HOURS_IN_SECONDS` dan nilainya diatur ke
hasil perkalian 60 (banyaknya detik dalam satu menit) dengan 60 (banyaknya
menit dalam satu jam) dengan 3 (banyaknya jam yang ingin kita hitung dalam
program ini). Konvensi penamaan Rust bagi konstanta adalah memakai huruf besar
dengan garis bawah antara kata. Compiler dapat mengevaluasi set operasi
terbatas saat compile, yang memungkinkan kita memilih menulis nilai ini dalam
suatu cara yang lebih mudah dipahami dan diverifikasi, daripada menata
konstanta ini ke nilai 10.800. Lihat [Bagian Referensi Rust tentang evaluasi
konstanta][const-eval] untuk informasi lebih banyak tentang operasi apa yang
dapat dipakai saat mendeklarasikan konstanta.

Konstanta valid bagi seluruh waktu suatu program dijalankan, dalam skup
tempat mereka dideklarasikan. Properti ini membuat konstanta berguna untuk
nilai-nilai dalam domain aplikasi Anda yang berbagai bagian dari program
mungkin perlu tahu, seperti banyaknya maksimum nilai yang diizinkan diperoleh
seorang pemain, atau kecepatan cahaya.

Menamai nilai-nilai ter-*hardcode* yang dipakai dalam seluruh program Anda
sebagai konstanta berguna dalam menyampaikan makna nilai itu ke pemelihara
kode di masa mendatang. Itu juga membantu agar hanya satu tempat dalam kode
Anda yang perlu diubah bila nilai ter-*hardcode* perlu diperbarui nanti.

### Shadowing

Sebagaimana Anda lihat dalam tutorial permainan tebakan dalam
[Bab 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->, Anda dapat 
mendeklarasikan suatu variabel baru dengan nama sama seperti variabel
sebelumnya. Rustacean mengatakan bahwa variabel pertama *di-shadow* oleh yang
kedua, yang berarti bahwa variabel kedua adalah yang akan dilihat oleh
compiler ketika Anda memakai nama variabel tersebut. Efeknya, variabel
kedua menutupi yang pertama, mengambil sebarang penggunaan dari nama variabel
ke dirinya sendiri dengan memakai nama variabel yang sama dan mengulangi
penggunaan kata kunci `let` sebagai berikut:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Program ini pertama mengikat `x` ke suatu nilai `5`. Lalu itu membuat
sebuah variabel baru `x` dengan mengulangi `let x =`, mengambil nilai asli
dan menambahkan `1` sehingga nilai `x` lalu menjadi `6`. Kemudian, di dalam
skup dalam yang dibuat dengan kurung kurawal, pernyataan `let` ketiga juga
membayang `x` dan membuat sebuah variabel baru, mengalikan nilai sebelumnya
dengan `2` untuk menghasilkan suatu nilai `12`. Ketika skup itu berakhir, 
bayang dalam berakhir dan `x` kembali menjadi `6`. Ketika kita menjalankan
program ini, itu akan mengeluarkan yang berikut:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Shadowing berbeda dari menandai suatu variabel sebagai `mut` karena kita akan
memperoleh suatu galat saat compile bila kita secara tidak sengaja mencoba
menugaskan ulang ke variabel ini tanpa memakai kata kunci `let`. Dengan
memakai `let`, kita bisa melakukan beberapa transformasi pada suatu nilai
tapi memiliki variabel yang immutable setelah transformasi itu selesai.

Perbedaan lain antara `mut` dan shadowing adalah karena kita secara efektif
membuat sebuah variabel baru ketika kita memakai kata kunci `let` lagi, kita
dapat mengubah tipe nilai tapi memakai ulang nama yang sama. Misalnya, 
katakanlah program kita meminta pengguna untuk menunjukkan berapa banyak spasi
yang mereka inginkan antara beberapa teks dengan memasukkan karakter spasi,
lalu kita ingin menyimpan masukan itu sebagai suatu bilangan:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Variabel `spaces` pertama bertipe string dan variabel `spaces` kedua bertipe
angka. Maka *shadowing* membebaskan kita dari mencari nama lain, seperti
misalnya `spaces_str` dan `spaces_num`; sebagai gantinya, kita dapat memakai
ulang nama `spaces` yang lebih sederhana. Namun, bila kita mencoba memakai
`mut` untuk ini, seperti yang ditunjukkan di sini, kita akan memperoleh 
kesalahan waktu compile:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Kesalahannya mengatakan bahwa kita tidak diizinkan untuk memutasi suatu
tipe variabel:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Kini setelah kita mengeksplorasi bagaimana variabel bekerja, mari kita 
melihat lebih banyak tipe data yang dapat mereka miliki.

[comparing-the-guess-to-the-secret-number]:
ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
