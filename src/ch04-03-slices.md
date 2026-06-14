## Tipe Slice

_Slices_ ngebolehin kita buat ngerujuk ke serangkaian elemen yang berurutan di 
sebuah [koleksi](ch08-00-common-collections.md). Slice itu sejenis referensi, 
jadi dia nggak punya _ownership_.

Ini ada masalah pemrograman kecil: bikin sebuah fungsi yang nerima sebuah 
string kata-kata yang dipisahin spasi terus balikin kata pertama yang dia nemu 
di string itu. Kalau fungsinya nggak nemu spasi di string-nya, berarti seluruh 
string-nya adalah satu kata, jadi seluruh string-nya harus dibalikin.

> Catatan: Buat tujuan ngenalin string slices, kita asumsikan cuma pake ASCII 
> aja di bagian ini; pembahasan lebih mendalam soal penanganan UTF-8 ada di 
> bagian [“Menyimpan Teks Berkode UTF-8 dengan Strings”][strings] di Bab 8.

Yuk kita pelajari gimana cara kita nulis signature fungsi ini tanpa pake slices, 
biar paham masalah apa yang bakal diselesain sama slices:

```rust,ignore
fn first_word(s: &String) -> ?
```

Fungsi `first_word` punya parameter tipe `&String`. Kita nggak butuh 
_ownership_, jadi ini oke-oke aja. (Di Rust yang idiomatik, fungsi nggak ngambil 
_ownership_ dari argumennya kecuali emang butuh, dan alasannya bakal makin 
jelas seiring kita lanjut.) Tapi apa yang harus kita balikin? Kita sebenernya 
nggak punya cara buat nyebut *sebagian* dari sebuah string. Tapi, kita bisa 
balikin indeks dari akhir katanya, yang ditandain sama spasi. Yuk kita cobain, 
kayak yang ditunjukin di Listing 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="Fungsi `first_word` yang balikin nilai indeks byte ke dalam parameter `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Karena kita perlu nelusurin `String`-nya elemen demi elemen terus cek apakah 
sebuah nilai itu spasi, kita bakal convert `String` kita jadi sebuah array dari 
byte pake method `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Selanjutnya, kita bikin sebuah _iterator_ lewat array byte tadi pake method `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Kita bakal bahas iterator lebih detail di [Bab 13][ch13]. Buat sekarang, tau aja 
kalau `iter` itu method yang balikin tiap elemen di sebuah koleksi dan 
`enumerate` ngebungkus hasil dari `iter` terus balikin tiap elemen sebagai 
bagian dari sebuah tuple. Elemen pertama dari tuple yang dibalikin sama 
`enumerate` itu adalah indeksnya, dan elemen keduanya adalah referensi ke 
elemennya. Ini lumayan lebih nyaman daripada ngitung indeksnya sendiri.

Karena method `enumerate` balikin tuple, kita bisa pake pattern buat 
_destructure_ tuple itu. Kita bakal bahas pattern lebih banyak di [Bab 6][ch6]. 
Di dalem `for` loop, kita nentuin pattern yang punya `i` buat indeks di tuple 
dan `&item` buat satu byte di tuple. Karena kita dapet referensi ke elemennya 
dari `.iter().enumerate()`, kita pake `&` di pattern-nya.

Di dalem `for` loop, kita cari byte yang merepresentasikan spasi pake sintaks 
literal byte. Kalau kita nemu spasi, kita balikin posisinya. Kalau nggak, kita 
balikin panjang string-nya pake `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Kita sekarang punya cara buat nemuin indeks akhir dari kata pertama di string, 
tapi ada masalah. Kita balikin `usize` sendirian, tapi itu cuma angka yang 
bermakna di konteks `&String`. Dengan kata lain, karena itu nilai yang terpisah 
dari `String`, nggak ada jaminan kalau dia tetep bakal valid di masa depan. 
Coba liat program di Listing 4-8 yang pake fungsi `first_word` dari Listing 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="Nyimpen hasil dari manggil fungsi `first_word` terus ngerubah isi `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Program ini berhasil di-compile tanpa error apa pun dan bakal tetep gitu kalau 
kita pake `word` setelah manggil `s.clear()`. Karena `word` sama sekali nggak 
hubung sama state dari `s`, `word` tetep isinya nilai `5`. Kita bisa pake nilai 
`5` itu bareng variabel `s` buat nyoba ngambil kata pertamanya, tapi ini bakal 
jadi sebuah _bug_ karena isi dari `s` udah berubah sejak kita nyimpen `5` di 
`word`.

Harus pusing mikirin indeks di `word` yang bisa nggak sinkron sama data di `s` 
itu ribet dan gampang bikin error! Ngelola indeks-indeks ini bakal makin rapuh 
lagi kalau kita nulis fungsi `second_word`. Signature-nya harusnya kayak gini:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Sekarang kita mantau indeks awal _dan_ akhir, dan kita punya makin banyak nilai 
yang dihitung dari data di state tertentu tapi nggak terikat sama state itu 
sama sekali. Kita punya tiga variabel nggak berhubungan yang melayang-layang 
yang perlu dijaga biar tetep sinkron.

Untungnya, Rust punya solusi buat masalah ini: _string slices_.

### String Slices

Sebuah _string slice_ adalah referensi ke serangkaian elemen yang berurutan 
dari sebuah `String`, dan bentuknya kayak gini:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Bukannya referensi ke seluruh `String`, `hello` adalah referensi ke sebagian 
dari `String`, yang ditentuin di bagian tambahan `[0..5]`. Kita bikin slices 
pake range di dalem kurung siku dengan nentuin `[indeks_awal..indeks_akhir]`, 
di mana _`indeks_awal`_ itu posisi pertama di slice-nya dan _`indeks_akhir`_ itu 
satu lebih banyak dari posisi terakhir di slice-nya. Secara internal, struktur 
data slice nyimpen posisi awal sama panjang slice-nya, yang sesuai sama 
_`indeks_akhir`_ dikurang _`indeks_awal`_. Jadi, di kasus `let world = &s[6..11];`, 
`world` bakal jadi slice yang isinya sebuah pointer ke byte di indeks 6 dari `s` 
dengan nilai panjang `5`.

Gambar 4-7 nunjukin ini di sebuah diagram.

<img alt="Tiga tabel: tabel yang merepresentasikan data stack dari s, yang 
nunjuk ke byte di indeks 0 di tabel data string &quot;hello world&quot; di 
heap. Tabel ketiga merepresentasikan data stack dari slice world, yang punya 
nilai panjang 5 dan nunjuk ke byte 6 dari tabel data heap." 
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Gambar 4-7: String slice yang ngerujuk ke bagian dari sebuah `String`</span>

Dengan sintaks range `..` di Rust, kalau kita mau mulai di indeks 0, kita bisa 
ngilangin nilai sebelum dua titik itu. Dengan kata lain, ini sama aja:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Sama juga halnya, kalau slice kita masukin byte terakhir dari `String`, kita 
bisa ngilangin angka belakangnya. Itu artinya ini sama aja:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Kita juga bisa ngilangin kedua nilainya buat ngambil slice dari seluruh string. 
Jadi ini juga sama aja:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Catatan: Indeks range string slice harus ada di batas karakter UTF-8 yang 
> valid. Kalau kita nyoba bikin string slice di tengah-tengah karakter 
> multi-byte, program kita bakal exit dengan error.

Dengan semua info ini, yuk kita tulis ulang `first_word` biar balikin sebuah 
slice. Tipe yang nandain “string slice” ditulisnya `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Kita dapet indeks buat akhir katanya dengan cara yang sama kayak di Listing 4-7, 
dengan nyari spasi pertama. Pas kita nemu spasi, kita balikin sebuah string 
slice pake awal string-nya sama indeks spasi tadi sebagai indeks awal dan 
akhirnya.

Sekarang pas kita manggil `first_word`, kita dapet balik satu nilai tunggal 
yang terikat ke data aslinya. Nilainya disusun dari referensi ke titik awal 
slice-nya sama jumlah elemen di slice-nya.

Balikin sebuah slice juga bakal jalan buat fungsi `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Kita sekarang punya API yang jelas yang jauh lebih susah buat salah pakenya 
karena _compiler_ bakal mastiin kalau referensi ke dalem `String`-nya tetep 
valid. Inget kan _bug_ di program di Listing 4-8, pas kita dapet indeks buat 
akhir kata pertama tapi terus nge-clear string-nya sampe indeks kita jadi nggak 
valid? Kode itu secara logis salah tapi nggak nunjukin error langsung. 
Masalahnya baru muncul nanti kalau kita terus nyoba pake indeks kata pertama 
sama string yang udah kosong. Slices bikin _bug_ ini jadi mustahil dan ngasih 
tau kita kalau ada masalah sama kode kita jauh lebih awal. Pake versi slice dari 
`first_word` bakal ngelepar _compile-time error_:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Ini error _compiler_-nya:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Inget kan dari aturan _borrowing_ kalau kita punya immutable reference ke 
sesuatu, kita nggak boleh juga ngambil mutable reference. Karena `clear` perlu 
memotong `String`-nya, dia perlu dapet mutable reference. `println!` setelah 
manggil `clear` pake referensi di `word`, jadi immutable reference-nya harus 
tetep aktif di titik itu. Rust ngelarang mutable reference di `clear` sama 
immutable reference di `word` buat ada di waktu yang sama, makanya kompilasinya 
gagal. Nggak cuma Rust bikin API kita lebih gampang dipake, tapi dia juga 
ngilangin seluruh kelas error pas _compile time_!

<!-- Old heading. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### Literal String sebagai Slices

Inget kan kita pernah bahas soal literal string yang disimpan di dalem biner. 
Sekarang setelah kita tau soal slices, kita bisa paham literal string dengan 
bener:

```rust
let s = "Hello, world!";
```

Tipe `s` di sini adalah `&str`: dia adalah sebuah slice yang nunjuk ke titik 
spesifik di binernya. Ini juga kenapa literal string itu _immutable_; `&str` 
adalah sebuah immutable reference.

#### String Slices sebagai Parameter

Tau kalau kita bisa ngambil slices dari literal sama nilai `String` bawa kita 
ke satu lagi peningkatan buat `first_word`, yaitu di signature-nya:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Seorang _Rustacean_ yang lebih berpengalaman bakal nulis signature yang 
ditunjukin di Listing 4-9 sebagai gantinya karena dia ngebolehin kita pake 
fungsi yang sama buat baik nilai `&String` maupun nilai `&str`.

<Listing number="4-9" caption="Ningkatin fungsi `first_word` dengan pake string slice buat tipe parameter `s`-nya">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Kalau kita punya sebuah string slice, kita bisa masukin itu langsung. Kalau 
kita punya sebuah `String`, kita bisa masukin slice dari `String`-nya atau 
referensi ke `String`-nya. Fleksibilitas ini manfaatin _deref coercions_, sebuah 
fitur yang bakal kita bahas di bagian [“Implicit Deref Coercions dengan Fungsi 
dan Method”][deref-coercions] di Bab 15.

Mendefinisikan fungsi buat nerima string slice bukannya referensi ke sebuah 
`String` bikin API kita jadi lebih umum dan berguna tanpa ngurangin fungsionalitas 
apa pun:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Slices Lainnya

String slices, kayak yang bisa kita bayangin, itu spesifik buat string. Tapi ada 
tipe slice yang lebih umum juga. Coba liat array ini:

```rust
let a = [1, 2, 3, 4, 5];
```

Sama kayak kita mungkin mau ngerujuk ke bagian dari sebuah string, kita mungkin 
juga mau ngerujuk ke bagian dari sebuah array. Kita lakuin kayak gini:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Slice ini punya tipe `&[i32]`. Dia cara kerjanya sama kayak string slices, 
dengan nyimpen referensi ke elemen pertama sama sebuah panjangnya. Kita bakal 
pake jenis slice ini buat macem-macem koleksi lainnya. Kita bakal bahas koleksi 
ini secara detail pas kita bahas vector di Bab 8.

## Ringkasan

Konsep _ownership_, _borrowing_, sama _slices_ ngejamin keamanan memori di 
program Rust pas _compile time_. Bahasa Rust ngasih kita kontrol atas penggunaan 
memori dengan cara yang sama kayak bahasa pemrograman sistem lainnya, tapi 
dengan adanya pemilik data yang otomatis ngebersihin data itu pas pemiliknya 
keluar dari scope, artinya kita nggak perlu nulis dan debug kode tambahan buat 
dapet kontrol ini.

_Ownership_ ngaruh ke banyak bagian Rust lainnya, jadi kita bakal bahas konsep-
konsep ini lebih lanjut di sepanjang sisa bukunya. Yuk kita lanjut ke Bab 5 
buat liat gimana ngelempokin potongan-potongan data jadi satu di sebuah 
`struct`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods
