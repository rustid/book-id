## Tipe Data

Tiap nilai di Rust itu punya _data type_ (tipe data) tertentu, yang ngasih tau 
Rust jenis data apa yang kita maksud biar dia tau gimana cara nanganin data itu. 
Kita bakal liat dua subset tipe data: _scalar_ sama _compound_.

Inget ya kalau Rust itu bahasa yang _statically typed_, artinya dia harus tau 
tipe dari semua variabel pas _compile time_. _Compiler_ biasanya bisa tau (infer) 
tipe apa yang mau kita pake berdasarkan nilainya dan gimana kita pakenya. Di 
kasus di mana ada banyak kemungkinan tipe, kayak pas kita convert sebuah 
`String` jadi tipe numerik pake `parse` di bagian [“Membandingkan Tebakan dengan Secret Number”][comparing-the-guess-to-the-secret-number] 
di Bab 2, kita harus nambahin annotasi tipe, kayak gini:

```rust
let guess: u32 = "42".parse().expect("Bukan angka!");
```

Kalau kita nggak nambahin annotasi tipe `: u32` kayak di atas, Rust bakal 
nampilin error berikut, yang artinya _compiler_ butuh info lebih lanjut dari 
kita biar tau tipe mana yang mau kita pake:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Kita bakal liat annotasi tipe yang beda buat tipe data lainnya.

### Tipe Scalar

Tipe _scalar_ merepresentasikan sebuah nilai tunggal. Rust punya empat tipe 
scalar utama: integer, floating-point numbers, Boolean, dan karakter. Kita 
mungkin udah kenal ini dari bahasa pemrograman lain. Yuk kita liat gimana cara 
kerjanya di Rust.

#### Tipe Integer

_Integer_ itu angka tanpa komponen pecahan. Kita udah pake satu tipe integer di 
Bab 2, yaitu tipe `u32`. Deklarasi tipe ini nunjukin kalau nilai yang terkait 
harusnya sebuah _unsigned integer_ (tipe _signed integer_ diawali sama `i` 
bukannya `u`) yang makan tempat 32 bits. Tabel 3-1 nunjukin tipe-tipe integer 
bawaan di Rust. Kita bisa pake varian mana pun dari ini buat mendeklarasikan 
tipe dari sebuah nilai integer.

<span class="caption">Tabel 3-1: Tipe Integer di Rust</span>

| Panjang | Signed  | Unsigned |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| architecture dependent | `isize` | `usize`  |

Tiap varian bisa jadi _signed_ atau _unsigned_ dan punya ukuran yang eksplisit. 
_Signed_ sama _unsigned_ ngerujuk ke apakah mungkin buat angkanya bernilai 
negatif—dengan kata lain, apakah angkanya perlu punya tanda (sign) barengannya 
(_signed_) atau apakah dia cuma bakal selalu positif dan makanya bisa 
direpresentasikan tanpa tanda (_unsigned_). Ini kayak nulis angka di kertas: 
pas tandanya penting, angka ditunjukin pake tanda plus atau minus; tapi pas 
aman buat diasumsikan kalau angkanya positif, dia ditunjukin tanpa tanda. Angka 
_signed_ disimpan pake representasi [two’s complement][twos-complement].

Tiap varian _signed_ bisa nyimpen angka dari −(2<sup>n − 1</sup>) sampe 
2<sup>n − 1</sup> − 1 inklusif, di mana _n_ itu jumlah bit yang dipake varian 
tersebut. Jadi sebuah `i8` bisa nyimpen angka dari −(2<sup>7</sup>) sampe 
2<sup>7</sup> − 1, yang sama dengan −128 sampe 127. Varian _unsigned_ bisa 
nyimpen angka dari 0 sampe 2<sup>n</sup> − 1, jadi sebuah `u8` bisa nyimpen 
angka dari 0 sampe 2<sup>8</sup> − 1, yang sama dengan 0 sampe 255.

Terus, tipe `isize` sama `usize` itu tergantung dari arsitektur komputer tempat 
program kita jalan: 64 bits kalau kita di arsitektur 64-bit dan 32 bits kalau 
kita di arsitektur 32-bit.

Kita bisa nulis literal integer dalam bentuk apa pun yang ditunjukin di Tabel 3-2. 
Inget ya kalau literal angka yang bisa punya banyak tipe numerik ngebolehin ada 
akhiran (suffix) tipe, kayak `57u8`, buat nentuin tipenya. Literal angka juga 
bisa pake `_` sebagai pemisah visual biar angkanya lebih gampang dibaca, kayak 
`1_000`, yang bakal punya nilai yang sama kayak kalau kita tulis `1000`.

<span class="caption">Tabel 3-2: Literal Integer di Rust</span>

| Literal Angka    | Contoh        |
| ---------------- | ------------- |
| Desimal          | `98_222`      |
| Hex              | `0xff`        |
| Oktal            | `0o77`        |
| Biner            | `0b1111_0000` |
| Byte (`u8` doang)| `b'A'`        |

Terus gimana kita tau tipe integer mana yang harus dipake? Kalau bingung, 
default-nya Rust biasanya udah oke sekali: tipe integer default ke `i32`. 
Situasi utama di mana kita bakal pake `isize` atau `usize` itu pas lagi 
ngindeks sekumpulan koleksi (collection).

> ##### Integer Overflow
>
> Katakanlah kita punya variabel tipe `u8` yang bisa nampung nilai antara 0 
> sampe 255. Kalau kita nyoba ngerubah variabel itu jadi nilai di luar range 
> itu, kayak 256, bakal terjadi _integer overflow_, yang bisa ngasilin salah 
> satu dari dua perilaku. Pas kita compile di mode debug, Rust masukin 
> pengecekan buat _integer overflow_ yang bikin program kita _panic_ pas 
> _runtime_ kalau perilaku ini kejadian. Rust pake istilah _panicking_ pas 
> sebuah program exit karena error; kita bakal bahas _panic_ lebih dalem di 
> bagian [“Error yang Tidak Bisa Dipulihkan dengan `panic!`”][unrecoverable-errors-with-panic] 
> di Bab 9.
>
> Pas kita compile di mode release pake flag `--release`, Rust _nggak_ 
> masukin pengecekan buat _integer overflow_ yang bikin _panic_. Sebaliknya, 
> kalau _overflow_ kejadian, Rust ngelakuin _two’s complement wrapping_. 
> Singkatnya, nilai yang lebih gede dari nilai maksimal yang bisa ditampung 
> tipenya bakal "bungkus muter" (wrap around) ke nilai minimal yang bisa 
> ditampung tipenya. Di kasus `u8`, nilai 256 jadi 0, nilai 257 jadi 1, dan 
> seterusnya. Programnya nggak bakal _panic_, tapi variabelnya bakal punya 
> nilai yang mungkin nggak sesuai ekspektasi kita. Ngandelin perilaku _wrapping_ 
> dari _integer overflow_ itu dianggap sebagai error.
>
> Buat handle kemungkinan _overflow_ secara eksplisit, kita bisa pake keluarga 
> method yang dikasih standard library buat tipe numerik primitif:
>
> - Wrap di semua mode pake method `wrapping_*`, kayak `wrapping_add`.
> - Balikin nilai `None` kalau ada _overflow_ pake method `checked_*`.
> - Balikin nilainya sama Boolean yang nunjukin apakah ada _overflow_ pake 
>   method `overflowing_*`.
> - _Saturate_ di nilai minimal atau maksimal tipenya pake method 
>   `saturating_*`.

#### Tipe Floating-Point

Rust juga punya dua tipe primitif buat _floating-point numbers_, yaitu angka 
dengan titik desimal. Tipe _floating-point_ di Rust adalah `f32` dan `f64`, 
yang masing-masing ukurannya 32 bits dan 64 bits. Tipe default-nya adalah `f64` 
karena di CPU modern, kecepatannya hampir sama kayak `f32` tapi bisa lebih 
presisi. Semua tipe _floating-point_ itu _signed_.

Ini contoh tipe _floating-point_ beraksi:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Angka _floating-point_ direpresentasikan sesuai standar IEEE-754.

#### Operasi Numerik

Rust support operasi matematika dasar yang kita harapin buat semua tipe angka: 
penambahan, pengurangan, perkalian, pembagian, dan sisa bagi (remainder). 
Pembagian integer bakal dipotong (truncate) ke arah nol ke integer terdekat. 
Kode berikut nunjukin gimana cara pake tiap operasi numerik di statement `let`:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Tiap ekspresi di statement ini pake operator matematika terus dievaluasi jadi 
satu nilai tunggal, yang terus di-bind ke sebuah variabel. [Lampiran B][appendix_b] 
isinya daftar semua operator yang disediain Rust.

#### Tipe Boolean

Kayak di kebanyakan bahasa pemrograman lain, tipe Boolean di Rust punya dua 
kemungkinan nilai: `true` sama `false`. Boolean ukurannya satu byte. Tipe 
Boolean di Rust ditentuin pake `bool`. Contohnya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Cara utama buat pake nilai Boolean itu lewat kondisional, kayak ekspresi `if`. 
Kita bakal bahas gimana cara kerja ekspresi `if` di Rust di bagian [“Control Flow”][control-flow].

#### Tipe Karakter (Character)

Tipe `char` di Rust adalah tipe alfabetik paling primitif di bahasanya. Ini 
beberapa contoh deklarasi nilai `char`:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Inget ya kalau kita nentuin literal `char` pake kutip tunggal, beda sama literal 
string yang pake kutip ganda. Tipe `char` di Rust ukurannya empat byte dan 
merepresentasikan Unicode Scalar Value, yang artinya dia bisa merepresentasikan 
jauh lebih banyak dari cuma ASCII doang. Huruf beraksen; karakter Cina, Jepang, 
dan Korea; emoji; sama _zero-width spaces_ itu semua adalah nilai `char` yang 
valid di Rust. Unicode Scalar Values range-nya dari `U+0000` sampe `U+D7FF` dan 
`U+E000` sampe `U+10FFFF` inklusif. Tapi, sebuah "karakter" itu sebenernya 
bukan konsep di Unicode, jadi intuisi manusia kita soal apa itu "karakter" 
mungkin nggak pas sama apa itu `char` di Rust. Kita bakal bahas topik ini 
detail di [“Menyimpan Teks Berkode UTF-8 dengan Strings”][strings] di Bab 8.

### Tipe Compound

Tipe _compound_ (campuran) bisa ngelempokin banyak nilai jadi satu tipe. Rust 
punya dua tipe compound primitif: tuple sama array.

#### Tipe Tuple

Sebuah _tuple_ adalah cara umum buat ngelempokin sejumlah nilai dengan berbagai 
macam tipe jadi satu tipe compound. Tuple punya panjang yang tetap: sekali 
dideklarasikan, ukurannya nggak bisa nambah atau berkurang.

Kita bikin tuple dengan nulis daftar nilai yang dipisahin koma di dalem tanda 
kurung. Tiap posisi di tuple punya tipe, dan tipe-tipe dari nilai yang beda di 
tuple itu nggak harus sama. Kita udah nambahin annotasi tipe opsional di contoh 
ini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Variabel `tup` nge-bind ke seluruh tuple karena sebuah tuple dianggap sebagai 
elemen compound tunggal. Buat dapet nilai individunya dari sebuah tuple, kita 
bisa pake pattern matching buat _destructure_ sebuah nilai tuple, kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Program ini pertama-tama bikin tuple terus nge-bind ke variabel `tup`. Terus 
dia pake pattern bareng `let` buat ngambil `tup` terus diubah jadi tiga variabel 
terpisah, `x`, `y`, dan `z`. Ini namanya _destructuring_ karena dia mecah satu 
tuple jadi tiga bagian. Akhirnya, programnya nyetak nilai `y`, yaitu `6.4`.

Kita juga bisa akses elemen tuple secara langsung pake tanda titik (`.`) diikuti 
sama indeks nilai yang mau kita akses. Contohnya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Program ini bikin tuple `x` terus akses tiap elemen tuple pake indeksnya 
masing-masing. Kayak di kebanyakan bahasa pemrograman, indeks pertama di tuple 
itu 0.

Tuple tanpa nilai apa pun punya nama khusus, yaitu _unit_. Nilai ini sama tipe 
terkaitnya sama-sama ditulis `()` dan merepresentasikan nilai kosong atau tipe 
return kosong. Ekspresi secara implisit balikin nilai unit kalau mereka nggak 
balikin nilai lainnya.

#### Tipe Array

Cara lain buat punya sekumpulan banyak nilai itu pake _array_. Beda sama tuple, 
tiap elemen array harus punya tipe yang sama. Beda sama array di beberapa 
bahasa lain, array di Rust punya panjang yang tetap.

Kita nulis nilai-nilai di array sebagai daftar yang dipisahin koma di dalem 
kurung siku:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Array itu berguna pas kita mau data kita dialokasikan di _stack_, sama kayak 
tipe-tipe lain yang udah kita liat sejauh ini, bukannya di _heap_ (kita bakal 
bahas stack sama heap lebih lanjut di [Bab 4][stack-and-heap]) atau pas kita 
mau mastiin kalau kita selalu punya jumlah elemen yang tetap. Tapi array itu 
nggak sefleksibel tipe _vector_. _Vector_ adalah tipe koleksi serupa yang 
disediain standard library yang _boleh_ nambah atau berkurang ukurannya karena 
isinya ada di heap. Kalau bingung mau pake array atau vector, kemungkinan 
besar mending pake vector. [Bab 8][vectors] bahas vector lebih detail.

Tapi, array lebih berguna pas kita tau jumlah elemennya nggak perlu berubah. 
Misalnya, kalau kita lagi pake nama-nama bulan di sebuah program, kita mungkin 
bakal pake array bukannya vector karena kita tau isinya bakal selalu 12 elemen:

```rust
let months = ["Januari", "Februari", "Maret", "April", "Mei", "Juni", "Juli",
              "Agustus", "September", "Oktober", "November", "Desember"];
```

Kita nulis tipe array pake kurung siku yang isinya tipe tiap elemen, titik koma, 
terus jumlah elemen di array-nya, kayak gini:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Di sini, `i32` itu tipe tiap elemen. Setelah titik koma, angka `5` nunjukin 
kalau array-nya isinya lima elemen.

Kita juga bisa menginisialisasi array biar isinya nilai yang sama buat tiap 
elemen dengan nentuin nilai awalnya, diikuti titik koma, terus panjang 
array-nya di dalem kurung siku, kayak yang ditunjukin di sini:

```rust
let a = [3; 5];
```

Array namanya `a` bakal isinya `5` elemen yang semuanya bakal di-set ke nilai 
`3` pas awal. Ini sama aja kayak nulis `let a = [3, 3, 3, 3, 3];` tapi dengan 
cara yang lebih singkat.

##### Akses Elemen Array

Array itu satu potongan memori tunggal dengan ukuran yang udah tau dan tetap 
yang bisa dialokasikan di stack. Kita bisa akses elemen array pake _indexing_, 
kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Di contoh ini, variabel namanya `first` bakal dapet nilai `1` karena itu nilai 
di indeks `[0]` di array-nya. Variabel namanya `second` bakal dapet nilai `2` 
dari indeks `[1]` di array-nya.

##### Akses Elemen Array Nggak Valid

Yuk kita liat apa yang terjadi kalau kita nyoba akses elemen array yang 
ngelewatin akhir dari array-nya. Katakanlah kita jalanin kode ini, mirip kayak 
game tebak angka di Bab 2, buat dapet indeks array dari user:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Kode ini berhasil di-compile. Kalau kita jalanin kode ini pake `cargo run` terus 
masukin `0`, `1`, `2`, `3`, atau `4`, programnya bakal nyetak nilai yang 
terkait di indeks itu di array-nya. Kalau kita malah masukin angka yang 
ngelewatin akhir array, kayak `10`, kita bakal liat output kayak gini:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Programnya ngasilin _runtime error_ pas lagi pake nilai nggak valid di operasi 
indexing-nya. Programnya exit dengan pesan error dan nggak ngejalankan 
statement `println!` yang terakhir. Pas kita nyoba akses sebuah elemen pake 
indexing, Rust bakal nge-cek kalau indeks yang kita tentuin itu kurang dari 
panjang array-nya. Kalau indeksnya lebih gede dari atau sama dengan panjangnya, 
Rust bakal _panic_. Pengecekan ini harus kejadian pas runtime, apalagi di kasus 
ini, karena _compiler_ nggak mungkin tau nilai apa yang bakal dimasukin user 
pas mereka jalanin kodenya nanti.

Ini contoh dari prinsip _memory safety_ Rust yang lagi beraksi. Di banyak bahasa 
tingkat rendah (low-level), pengecekan kayak gini nggak dilakuin, dan pas kita 
ngasih indeks yang salah, memori yang nggak valid bisa diakses. Rust ngelindungin 
kita dari jenis error kayak gini dengan langsung exit bukannya ngebolehin akses 
memori itu terus lanjut. Bab 9 bakal bahas lebih banyak soal penanganan error 
di Rust dan gimana kita bisa nulis kode yang enak dibaca dan aman yang nggak 
_panic_ maupun ngebolehin akses memori nggak valid.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
