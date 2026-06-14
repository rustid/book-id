## Variabel dan Mutabilitas

Kayak yang udah disebutin di bagian [“Menyimpan Nilai dengan Variabel”][storing-values-with-variables], 
secara default, variabel itu _immutable_ (nggak bisa diubah). Ini salah satu 
cara Rust "nyenggol" kita buat nulis kode yang manfaatin keamanan dan kemudahan 
_concurrency_ yang ditawarin Rust. Tapi, kita tetep punya opsi buat bikin 
variabel jadi _mutable_ (bisa diubah). Yuk kita eksplor gimana dan kenapa Rust 
nyaranin kita buat lebih milih _immutability_, dan kenapa kadang kita malah mau 
milih buat nggak pakenya.

Pas sebuah variabel itu _immutable_, sekali nilainya di-bind ke sebuah nama, 
kita nggak bisa ngerubah nilai itu. Buat gambarin ini, coba bikin project baru 
namanya _variables_ di direktori _projects_ kita pake `cargo new variables`.

Terus, di direktori _variables_ yang baru, buka _src/main.rs_ terus ganti kodenya 
jadi kayak gini, yang sebenernya belum bisa di-compile sekarang:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Simpan terus jalanin programnya pake `cargo run`. Kita bakal dapet pesan error 
soal _immutability error_, kayak yang ditunjukin di output ini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Contoh ini nunjukin gimana _compiler_ ngebantu kita nemuin error di program kita. 
Error dari _compiler_ emang kadang bikin kesel, tapi sebenernya itu cuma berarti 
program kita belum aman buat ngelakuin apa yang kita mau; itu _bukan_ berarti 
kita bukan programmer yang jago! Para _Rustacean_ yang udah pro pun tetep sering 
dapet error dari _compiler_.

Kita dapet pesan error `` cannot assign twice to immutable variable `x` `` 
karena kita nyoba buat ngasih nilai kedua ke variabel `x` yang _immutable_.

Penting sekali buat kita dapet _compile-time error_ pas kita nyoba ngerubah nilai 
yang udah ditentuin sebagai _immutable_ karena situasi ini bisa memicu _bug_. 
Kalau satu bagian kode kita jalan dengan asumsi kalau sebuah nilai nggak bakal 
berubah, terus bagian kode lain malah ngerubah nilai itu, ada kemungkinan bagian 
pertama tadi nggak bakal jalan sesuai desainnya. Penyebab _bug_ kayak gini bisa 
susah sekali dilacak setelah kejadian, apalagi kalau bagian kode kedua ngerubah 
nilainya cuma "kadang-kadang" doang. _Compiler_ Rust ngejamin kalau pas kita 
bilang sebuah nilai nggak bakal berubah, ya dia benar-benar nggak bakal berubah, 
jadi kita nggak perlu repot-repot jagain sendiri. Kode kita jadi lebih gampang 
buat dipahamin alurnya.

Tapi _mutability_ emang bisa sangat berguna, dan bisa bikin kode lebih nyaman 
buat ditulis. Walaupun variabel itu _immutable_ secara default, kita bisa bikin 
mereka jadi _mutable_ dengan nambahin `mut` di depan nama variabelnya kayak yang 
kita lakuin di [Bab 2][storing-values-with-variables]. Nambahin `mut` juga 
ngasih tau maksud (intent) kita ke orang yang baca kode kita nanti kalau bagian 
lain dari kode bakal ngerubah nilai variabel ini.

Contohnya, yuk kita ubah _src/main.rs_ jadi kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Pas kita jalanin programnya sekarang, hasilnya kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Kita diperbolehkan buat ngerubah nilai yang di-bind ke `x` dari `5` jadi `6` 
pas `mut` dipake. Akhirnya, keputusan buat pake _mutability_ atau nggak itu 
balik lagi ke kita dan tergantung apa yang menurut kita paling jelas di situasi 
tertentu itu.

### Konstanta (Constants)

Kayak variabel _immutable_, _konstanta_ adalah nilai yang di-bind ke sebuah nama 
dan nggak boleh berubah, tapi ada beberapa perbedaan antara konstanta sama 
variabel.

Pertama, kita nggak boleh pake `mut` sama konstanta. Konstanta nggak cuma 
_immutable_ secara default—mereka _selalu_ _immutable_. Kita mendeklarasikan 
konstanta pake keyword `const` bukannya `let`, dan tipe nilainya _harus_ 
diannotasi. Kita bakal bahas soal tipe dan annotasi tipe di bagian selanjutnya, 
[“Tipe Data”][data-types], jadi nggak usah pusing dulu soal detailnya sekarang. 
Pokoknya tau aja kalau kita harus selalu nulis tipenya.

Konstanta bisa dideklarasikan di scope mana pun, termasuk scope global, yang 
bikin mereka berguna buat nilai yang perlu diketahuin sama banyak bagian kode.

Perbedaan terakhir adalah konstanta cuma boleh di-set ke _constant expression_, 
bukan hasil dari nilai yang cuma bisa dihitung pas _runtime_.

Ini contoh deklarasi konstanta:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Nama konstantanya adalah `THREE_HOURS_IN_SECONDS` dan nilainya di-set ke hasil 
perkalian 60 (jumlah detik dalam satu menit) dikali 60 (jumlah menit dalam satu 
jam) dikali 3 (jumlah jam yang mau kita itung di program ini). Konvensi 
penamaan Rust buat konstanta adalah pake huruf kapital semua (uppercase) dengan 
garis bawah (underscore) di antara kata-katanya. _Compiler_ bisa nge-evaluasi 
sekumpulan operasi terbatas pas _compile time_, yang bikin kita bisa milih buat 
nulis nilai ini dengan cara yang lebih gampang dipahamin dan diverifikasi, 
bukannya langsung nulis nilai 10.800. Liat [bagian Rust Reference soal constant 
evaluation][const-eval] buat info lebih lanjut soal operasi apa aja yang bisa 
dipake pas deklarasi konstanta.

Konstanta itu valid selama program jalan, di dalem scope tempat mereka 
dideklarasikan. Sifat ini bikin konstanta berguna buat nilai di domain aplikasi 
kita yang mungkin perlu diketahuin sama banyak bagian program, kayak jumlah 
poin maksimal yang boleh didapet player sebuah game, atau kecepatan cahaya.

Ngambil nilai _hardcoded_ yang dipake di seluruh program terus dikasih nama 
sebagai konstanta itu sangat berguna buat nyampein makna nilai itu ke orang 
yang bakal maintain kodenya nanti. Ini juga ngebantu biar cuma ada satu tempat 
di kode kita yang perlu diubah kalau nilai _hardcoded_ itu perlu di-update di 
masa depan.

### Shadowing

Kayak yang kita liat di tutorial game tebak angka di [Bab 2][comparing-the-guess-to-the-secret-number], 
kita bisa mendeklarasikan variabel baru dengan nama yang sama kayak variabel 
sebelumnya. Para _Rustacean_ bilang kalau variabel pertama itu di-_shadow_ 
(dibayangi) sama variabel kedua, yang artinya variabel kedua lah yang bakal 
diliat sama _compiler_ pas kita pake nama variabel itu. Efektifnya, variabel 
kedua menutupi variabel pertama, ngambil semua penggunaan nama variabel itu buat 
dirinya sendiri sampe dia sendiri di-_shadow_ atau scope-nya abis. Kita bisa 
nge-_shadow_ sebuah variabel dengan pake nama variabel yang sama dan ngulangin 
penggunaan keyword `let` kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Program ini pertama-tama nge-bind `x` ke nilai `5`. Terus dia bikin variabel 
baru `x` dengan ngulangin `let x =`, ngambil nilai aslinya terus ditambahin `1` 
biar nilai `x` jadi `6`. Terus, di dalem scope dalem yang dibuat pake kurung 
kurawal, statement `let` yang ketiga juga nge-_shadow_ `x` dan bikin variabel 
baru, ngaliin nilai sebelumnya sama `2` biar `x` jadi `12`. Pas scope itu abis, 
_shadowing_ dalemnya kelar dan `x` balik lagi jadi `6`. Pas kita jalanin 
program ini, output-nya bakal kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

_Shadowing_ itu beda sama nandain variabel sebagai `mut` karena kita bakal dapet 
_compile-time error_ kalau kita nggak sengaja nyoba buat nge-_assign_ ulang ke 
variabel ini tanpa pake keyword `let`. Dengan pake `let`, kita bisa ngelakuin 
beberapa transformasi pada sebuah nilai tapi tetep bikin variabelnya jadi 
_immutable_ setelah transformasi itu selesai.

Perbedaan lain antara `mut` sama _shadowing_ adalah karena kita sebenernya bikin 
variabel baru pas kita pake keyword `let` lagi, kita bisa ngerubah tipe nilainya 
tapi tetep pake nama yang sama. Misalnya, katakanlah program kita minta user 
buat nunjukin berapa banyak spasi yang mereka mau di antara teks dengan masukin 
karakter spasi, terus kita mau nyimpen input itu sebagai angka:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Variabel `spaces` yang pertama itu tipe string dan variabel `spaces` yang kedua 
itu tipe angka. Jadi _shadowing_ bikin kita nggak perlu repot mikirin nama yang 
beda, kayak `spaces_str` dan `spaces_num`; mendingan kita pake lagi nama 
`spaces` yang lebih simpel. Tapi, kalau kita nyoba pake `mut` buat hal ini, 
kayak yang ditunjukin di sini, kita bakal dapet _compile-time error_:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Error-nya bilang kalau kita nggak diperbolehkan buat nge-mutasi tipe variabel:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Sekarang setelah kita eksplor gimana cara kerja variabel, yuk kita liat tipe 
data lainnya yang bisa mereka punya.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
