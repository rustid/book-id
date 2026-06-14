## Nyimpen Teks Berkode UTF-8 pake Strings

Kita udah pernah ngebahas _strings_ di Bab 4, tapi sekarang kita bakal bahas 
lebih mendalam. _Rustaceans_ (programmer Rust) baru biasanya sering mentok di 
_strings_ karena kombinasi tiga alasan: kecenderungan Rust buat nge-ekspos 
kemungkinan error, _strings_ yang ternyata adalah struktur data yang lebih ribet 
daripada yang dikira banyak programmer, dan UTF-8. Faktor-faktor ini kegabung 
dengan cara yang mungkin kerasa susah kalau kita asalnya dari bahasa 
pemrograman lain.

Kita ngebahas _strings_ di dalem konteks koleksi (_collections_) karena _strings_ 
diimplementasikan sebagai koleksi dari byte-byte, ditambah beberapa method buat 
nyediain fungsionalitas yang berguna pas byte-byte itu diterjemahin (interpreted) 
sebagai teks. Di bagian ini, kita bakal bahas operasi-operasi pada `String` 
yang dipunyai sama setiap tipe koleksi, kayak bikin (creating), ngubah (updating), 
sama ngebaca (reading). Kita juga bakal bahas gimana `String` itu beda dari 
koleksi lainnya, yaitu gimana proses _indexing_ ke dalem `String` itu dibikin 
ribet karena perbedaan antara gimana manusia sama komputer nerjemahin data 
`String`.

### Apa Itu String?

Pertama-tama kita bakal nentuin apa yang kita maksud dengan istilah _string_. 
Rust cuma punya satu tipe _string_ di dalem bahasa intinya (core language), 
yaitu _string slice_ `str` yang biasanya keliatan dalam bentuk referensi `&str`. 
Di Bab 4, kita udah ngebahas soal _string slices_, yang merupakan referensi ke 
sejumlah data _string_ berkode UTF-8 yang disimpan di tempat lain. Literal 
_string_, misalnya, disimpan di dalem _binary_ program kita dan makanya mereka 
itu adalah _string slices_.

Tipe `String`, yang disediain sama _standard library_ Rust bukannya dikodein 
langsung ke bahasa intinya, adalah tipe _string_ berkode UTF-8 yang bisa nambah 
ukurannya (growable), _mutable_, dan dimiliki (_owned_). Pas _Rustaceans_ nyebut 
“strings” di Rust, mereka mungkin maksudnya tipe `String` atau tipe _string 
slice_ `&str`, bukan cuma salah satunya doang. Walaupun bagian ini sebagian 
besar bahas soal `String`, kedua tipe ini sering sekali dipake di _standard 
library_ Rust, dan baik `String` maupun _string slices_ itu sama-sama berkode 
UTF-8.

### Bikin String Baru

Banyak operasi yang sama yang tersedia buat `Vec<T>` itu tersedia buat `String` 
juga karena `String` sebenernya diimplementasikan sebagai bungkus (wrapper) dari 
sebuah vector berisi byte-byte dengan beberapa jaminan (guarantees), batasan, 
dan kemampuan tambahan. Salah satu contoh fungsi yang cara kerjanya sama buat 
`Vec<T>` sama `String` adalah fungsi `new` buat bikin instance baru, kayak yang 
ditunjukin di Listing 8-11.

<Listing number="8-11" caption="Bikin `String` baru yang kosong">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Baris ini bikin _string_ baru yang kosong namanya `s`, yang nantinya bisa kita 
isiin data. Biasanya, kita punya data awal yang mau kita pake buat mulai _string_-nya. 
Buat kasus itu, kita pake method `to_string`, yang tersedia di tipe apa pun yang 
mengimplementasikan trait `Display`, kayak literal _string_. Listing 8-12 nunjukin 
dua contohnya.

<Listing number="8-12" caption="Pake method `to_string` buat bikin `String` dari literal _string_">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Kode ini bikin sebuah _string_ yang isinya teks `initial contents`.

Kita juga bisa pake fungsi `String::from` buat bikin `String` dari literal 
_string_. Kode di Listing 8-13 itu ekuivalen (sama) sama kode di Listing 8-12 
yang pake `to_string`.

<Listing number="8-13" caption="Pake fungsi `String::from` buat bikin `String` dari literal _string_">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Karena _strings_ dipake buat macem-macem hal, kita bisa pake banyak API generik 
yang beda-beda buat _strings_, ngasih kita sangat banyak opsi. Beberapa mungkin 
keliatannya berlebihan (redundant), tapi semuanya punya tempatnya masing-masing! 
Di kasus ini, `String::from` sama `to_string` ngelakuin hal yang persis sama, 
jadi milih yang mana itu cuma masalah gaya (style) dan _readability_ (keterbacaan) 
aja.

Inget ya kalau _strings_ itu berkode UTF-8, jadi kita bisa masukin data apa pun 
yang di-_encode_ dengan bener ke dalemnya, kayak yang ditunjukin di Listing 8-14.

<Listing number="8-14" caption="Nyimpen sapaan dalam berbagai bahasa di dalem _strings_">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Semua ini adalah nilai `String` yang valid.

### Ngubah String

Sebuah `String` bisa nambah ukurannya dan isinya bisa berubah, sama kayak isi 
dari `Vec<T>`, kalau kita nge-_push_ (masukin) lebih banyak data ke dalemnya. 
Selain itu, kita bisa pake operator `+` atau macro `format!` buat ngegabungin 
(concatenate) nilai-nilai `String` dengan gampang.

#### Nambahin Teks ke String pake `push_str` sama `push`

Kita bisa nambah ukuran `String` dengan pake method `push_str` buat nambahin 
_string slice_ di akhirnya, kayak yang ditunjukin di Listing 8-15.

<Listing number="8-15" caption="Nambahin _string slice_ ke dalem `String` pake method `push_str`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

Setelah dua baris ini, `s` bakal isinya `foobar`. Method `push_str` nerima 
_string slice_ karena kita nggak selamanya mau ngambil _ownership_ dari 
parameternya. Misalnya, di kode di Listing 8-16, kita mau tetep bisa pake `s2` 
setelah nambahin isinya ke `s1`.

<Listing number="8-16" caption="Pake _string slice_ setelah nambahin isinya ke dalem `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Kalau method `push_str` ngambil _ownership_ dari `s2`, kita nggak bakal bisa 
nyetak nilainya di baris terakhir. Tapi, kode ini jalan sesuai yang kita mau kok!

Method `push` nerima satu karakter sebagai parameter dan nambahin itu ke `String`. 
Listing 8-17 nambahin huruf _l_ ke dalem `String` pake method `push`.

<Listing number="8-17" caption="Nambahin satu karakter ke nilai `String` pake `push`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Hasilnya, `s` bakal isinya `lol`.

#### Penggabungan (Concatenation) pake Operator `+` atau Macro `format!`

Sering kali, kita mau ngegabungin dua _string_ yang udah ada. Salah satu 
caranya adalah pake operator `+`, kayak yang ditunjukin di Listing 8-18.

<Listing number="8-18" caption="Pake operator `+` buat ngegabungin dua nilai `String` jadi nilai `String` baru">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

_String_ `s3` bakal isinya `Hello, world!`. Alasan kenapa `s1` udah nggak 
valid lagi setelah penjumlahannya, dan alasan kenapa kita pake referensi ke `s2`, 
ada hubungannya sama _signature_ dari method yang dipanggil pas kita pake 
operator `+`. Operator `+` pake method `add`, yang _signature_-nya kira-kira 
kayak gini:

```rust,ignore
fn add(self, s: &str) -> String {
```

Di _standard library_, kita bakal liat `add` didefinisikan pake generik 
(generics) sama _associated types_. Di sini, kita udah ngegantiinnya pake tipe 
konkret, yang merupakan apa yang terjadi pas kita manggil method ini pake nilai 
`String`. Kita bakal bahas generik di Bab 10. _Signature_ ini ngasih kita 
petunjuk yang kita butuhin buat mahamin bagian-bagian _tricky_ dari operator `+`.

Pertama, `s2` punya `&`, yang artinya kita nambahin _referensi_ dari _string_ 
kedua ke _string_ pertama. Ini gara-gara parameter `s` di fungsi `add`: kita 
cuma bisa nambahin `&str` ke dalem `String`; kita nggak bisa nambahin dua nilai 
`String` bareng-bareng. Tapi tunggu—tipe dari `&s2` itu `&String`, bukan `&str`, 
kayak yang ditentuin di parameter kedua dari `add`. Terus kenapa Listing 8-18 
bisa di-compile?

Alasan kenapa kita bisa pake `&s2` di pemanggilan `add` adalah karena _compiler_ 
bisa nge-_coerce_ (maksa/ngubah) argumen `&String` jadi `&str`. Pas kita manggil 
method `add`, Rust pake yang namanya _deref coercion_, yang di sini ngerubah 
`&s2` jadi `&s2[..]`. Kita bakal bahas _deref coercion_ lebih dalem di Bab 15. 
Karena `add` nggak ngambil _ownership_ dari parameter `s`, `s2` bakal tetep jadi 
`String` yang valid setelah operasi ini.

Kedua, kita bisa liat di _signature_-nya kalau `add` ngambil _ownership_ dari 
`self` karena `self` _nggak_ punya `&`. Ini artinya `s1` di Listing 8-18 bakal 
di-_move_ ke dalem pemanggilan `add` dan nggak bakal valid lagi setelahnya. 
Jadi, walaupun `let s3 = s1 + &s2;` keliatannya kayak bakal ngopi kedua 
_string_ dan bikin yang baru, statement ini sebenernya ngambil _ownership_ dari 
`s1`, nambahin (append) salinan isi dari `s2` ke dalemnya, terus balikin 
_ownership_ dari hasilnya. Dengan kata lain, keliatannya dia bikin banyak 
salinan, tapi sebenernya nggak; implementasinya jauh lebih efisien daripada ngopi.

Kalau kita butuh ngegabungin banyak _strings_, perilaku dari operator `+` bakal 
jadi ribet sekali:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Di titik ini, `s` bakal jadi `tic-tac-toe`. Dengan semua karakter `+` sama `"`, 
susah buat liat apa yang sebenernya lagi terjadi. Buat ngegabungin _strings_ 
dengan cara yang lebih kompleks, kita bisa pake macro `format!` sebagai 
gantinya:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Kode ini juga nge-set `s` jadi `tic-tac-toe`. Macro `format!` cara kerjanya 
mirip `println!`, tapi bukannya nyetak output ke layar, dia balikin `String` 
yang isinya teks hasil formatnya. Versi kode yang pake `format!` itu jauh lebih 
gampang dibaca, dan kode yang dihasilin sama macro `format!` pake referensi jadi 
pemanggilan ini nggak bakal ngambil _ownership_ dari parameter mana pun.

### Indexing ke dalem Strings

Di banyak bahasa pemrograman lain, akses tiap karakter individu di dalem _string_ 
dengan ngerujuk ke indeks mereka itu adalah operasi yang valid dan umum sekali. 
Tapi, kalau kita nyoba akses bagian dari `String` pake sintaks _indexing_ di 
Rust, kita bakal dapet error. Coba liat kode yang nggak valid di Listing 8-19.

<Listing number="8-19" caption="Nyoba pake sintaks _indexing_ ke sebuah `String`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Kode ini bakal ngasilin error berikut:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Error dan catatannya nyeritain ceritanya: _strings_ di Rust nggak support 
_indexing_. Tapi kenapa nggak? Buat ngejawab pertanyaan itu, kita harus bahas 
gimana Rust nyimpen _strings_ di memori.

#### Representasi Internal

Sebuah `String` adalah bungkus (wrapper) buat `Vec<u8>`. Yuk kita liat beberapa 
contoh _strings_ berkode UTF-8 kita dari Listing 8-14. Pertama, yang ini:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Di kasus ini, `len` bakal bernilai `4`, yang artinya vector yang nyimpen _string_ 
`"Hola"` itu panjangnya 4 byte. Tiap huruf ini butuh satu byte pas di-_encode_ 
dalam UTF-8. Tapi, baris berikut ini mungkin bikin kita kaget (perhatiin ya 
kalau _string_ ini dimulai pake huruf kapital Cyrillic _Ze_, bukan angka 3):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Kalau kita ditanya seberapa panjang _string_ ini, kita mungkin bakal jawab 12. 
Nyatanya, jawaban Rust adalah 24: itu adalah jumlah byte yang dibutuhin buat nge-
_encode_ “Здравствуйте” dalam UTF-8, karena tiap nilai _scalar_ Unicode di 
dalem _string_ itu butuh memori sebesar 2 byte. Karena itu, sebuah indeks ke 
dalem byte-byte dari _string_ nggak bakal selalu sejalan sama nilai _scalar_ 
Unicode yang valid. Buat ngedemonstrasiin ini, coba liat kode Rust yang nggak 
valid ini:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Kita udah tau kalau `answer` nggak bakal isinya `З`, yaitu huruf pertamanya. 
Pas di-_encode_ di UTF-8, byte pertama dari `З` itu `208` dan yang kedua itu 
`151`, jadi kayaknya `answer` harusnya sebenernya `208`, tapi `208` itu bukan 
karakter yang valid kalau sendirian. Balikin nilai `208` kemungkinannya bukan 
apa yang dipengenin _user_ pas mereka minta huruf pertama dari _string_ ini; 
tapi, cuma itu data yang dipunyai Rust di indeks byte 0. _User_ biasanya nggak 
mau nilai byte-nya yang dibalikin, walaupun _string_-nya cuma isinya huruf Latin 
doang: kalau `&"hi"[0]` adalah kode valid yang balikin nilai byte-nya, dia bakal 
balikin `104`, bukan `h`.

Jadi jawabannya adalah buat ngehindarin balikin nilai yang nggak disangka-sangka 
dan nyebabin _bug_ yang mungkin nggak langsung ketahuan, Rust milih buat sama 
sekali nggak nge-compile kode ini dan nyegah kesalahpahaman dari awal di proses 
_development_ (pengembangan).

#### Bytes dan Nilai Scalar dan Grapheme Clusters! Waduh!

Poin lainnya soal UTF-8 adalah sebenernya ada tiga cara yang relevan buat 
ngeliat _strings_ dari sudut pandang Rust: sebagai _bytes_ (byte-byte), nilai 
_scalar_ (scalar values), sama _grapheme clusters_ (hal yang paling mendekati 
sama apa yang bakal kita sebut _letters_ atau huruf).

Kalau kita liat kata Hindi “नमस्ते” yang ditulis dalam aksara Devanagari, dia 
disimpan sebagai vector dari nilai `u8` yang keliatannya kayak gini:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Itu ada 18 byte dan ini adalah gimana komputer akhirnya nyimpen data ini. Kalau 
kita liat mereka sebagai nilai _scalar_ Unicode, yang mana itu adalah representasi 
tipe `char` di Rust, byte-byte itu bakal keliatan kayak gini:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Ada enam nilai `char` di sini, tapi yang keempat sama keenam itu bukan huruf: 
mereka itu _diacritics_ (tanda baca tambahan) yang nggak ada artinya kalau 
berdiri sendiri. Terakhir, kalau kita liat mereka sebagai _grapheme clusters_, 
kita bakal dapet apa yang bakal disebut orang sebagai empat huruf yang ngebentuk 
kata Hindi tersebut:

```text
["न", "म", "स्", "ते"]
```

Rust nyediain cara beda-beda buat nerjemahin (interpreting) data _string_ mentah 
yang disimpan komputer biar tiap program bisa milih terjemahan yang dia butuhin, 
nggak peduli apa bahasa manusia dari data tersebut.

Alasan terakhir kenapa Rust nggak ngebolehin kita nge-_index_ ke dalem `String` 
buat dapet sebuah karakter adalah karena operasi _indexing_ diharapkan bakal 
selalu butuh waktu konstan (O(1)). Tapi nggak mungkin buat ngejamin performa itu 
kalo pake `String`, karena Rust harus jalanin (walk through) isinya mulai dari 
awal sampe indeks tersebut buat nentuin berapa banyak karakter valid yang ada di 
sana.

### Slicing Strings

_Indexing_ ke dalem _string_ itu sering kali adalah ide yang jelek karena 
nggak jelas tipe _return_ apa yang seharusnya dihasilin dari operasi _indexing_ 
_string_ itu: apakah nilai byte, karakter, _grapheme cluster_, atau sebuah 
_string slice_. Jadi, kalau kita bener-bener butuh pake indeks buat bikin 
_string slice_, Rust minta kita buat lebih spesifik.

Bukannya _indexing_ pake `[]` sama satu angka doang, kita bisa pake `[]` 
bareng range (rentang) buat bikin _string slice_ yang isinya byte-byte tertentu:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Di sini, `s` bakal jadi `&str` yang isinya empat byte pertama dari _string_ 
tersebut. Tadi, kita udah bilang kalau tiap karakter ini ukurannya dua byte, 
yang artinya `s` bakal jadi `Зд`.

Kalau kita nyoba nge-slice cuma sebagian dari byte-byte punya satu karakter, 
misalnya pake `&hello[0..1]`, Rust bakal _panic_ pas _runtime_ sama kayak 
kalau ada indeks yang nggak valid yang diakses di vector:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Kita harus hati-hati pas bikin _string slices_ pake range, karena ngelakuin 
itu bisa bikin program kita _crash_.

### Methods buat Iterasi Lewat Strings

Cara terbaik buat ngoperasiin potongan-potongan dari _strings_ adalah dengan 
jelas (_explicit_) nentuin apakah kita mau karakter atau byte-nya. Buat dapet 
nilai _scalar_ Unicode individu, pake method `chars`. Manggil `chars` di “Зд” 
bakal misahin dan balikin dua nilai bertipe `char`, dan kita bisa iterasi 
hasilnya buat akses tiap elemennya:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Kode ini bakal nyetak output berikut:

```text
З
д
```

Alternatifnya, method `bytes` balikin tiap byte mentahnya (raw byte), yang 
mungkin cocok buat domain (ranah) kita:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Kode ini bakal nyetak empat byte yang ngebentuk _string_ ini:

```text
208
151
208
180
```

Tapi pastiin buat inget kalau nilai _scalar_ Unicode yang valid itu mungkin 
disusun dari lebih dari satu byte.

Dapetin _grapheme clusters_ dari _strings_, kayak pas pake aksara Devanagari 
tadi, itu kompleks, jadi fungsionalitas ini nggak disediain sama _standard 
library_. Ada crates yang tersedia di [crates.io](https://crates.io/) kalau ini 
fungsionalitas yang kita butuhin.

### Strings Nggak Sesimpel Itu

Sebagai ringkasan, _strings_ itu ribet (complicated). Bahasa pemrograman yang 
beda milih cara yang beda-beda juga soal gimana nyajiin keribetan ini ke 
programmer. Rust milih buat ngebikin cara nanganin data `String` dengan bener 
sebagai perilaku default buat semua program Rust, yang artinya programmer 
harus mikir lebih dalem buat nanganin data UTF-8 di awal. _Trade-off_ (pertukaran) 
ini nge-ekspos lebih banyak keribetan _strings_ daripada yang keliatan di bahasa 
pemrograman lain, tapi ini nyegah kita dari harus nanganin error yang 
ngelibatin karakter non-ASCII di masa depan pas siklus pengembangan (development 
life cycle).

Kabar baiknya adalah _standard library_ nawarin sangat banyak fungsionalitas 
yang dibangun di atas tipe `String` sama `&str` buat ngebantu kita nanganin 
situasi kompleks ini dengan bener. Pastiin buat cek dokumentasi buat method-method 
berguna kayak `contains` buat nyari sesuatu di dalem _string_ dan `replace` buat 
nggantiin bagian dari _string_ pake _string_ lainnya.

Yuk kita beralih ke sesuatu yang sedikit kurang ribet: hash maps!
