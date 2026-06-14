## Sintaks Pattern

Di bagian ini, kita mengumpulkan semua sintaks yang valid buat dipakai di 
dalam _patterns_ dan ngebahas kenapa dan kapan kita mungkin mau memakai 
masing-masing dari sintaks tersebut.

### Mencocokkan Literals (Nilai Harfiah)

Seperti yang udah kita lihat di Bab 6, kita bisa mencocokkan _patterns_ secara 
langsung dengan _literals_. Kode berikut ini ngasih beberapa contoh:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Kode ini mencetak `one` karena nilai di dalam `x` adalah `1`. Sintaks ini berguna 
pas kita pengen kode kita mengambil suatu tindakan tertentu kalau dia dapat sebuah 
nilai konkret yang spesifik.

### Mencocokkan Variabel Bernama (Named Variables)

Variabel bernama adalah _patterns_ _irrefutable_ yang bakal cocok dengan nilai apa 
pun, dan kita udah memakainya berkali-kali di buku ini. Namun, ada sedikit kerumitan 
pas kita memakai variabel bernama di dalam ekspresi `match`, `if let`, atau 
`while let`. Karena setiap jenis ekspresi ini memulai sebuah _scope_ (ruang lingkup) 
baru, variabel yang dideklarasikan sebagai bagian dari sebuah _pattern_ di dalam 
ekspresi ini bakal menimpa (shadow) variabel dengan nama yang sama yang ada di 
luar konstruk tersebut, sama halnya kayak yang terjadi pada semua variabel di Rust. 
Di Listing 19-11, kita mendeklarasikan sebuah variabel bernama `x` dengan nilai 
`Some(5)` dan sebuah variabel `y` dengan nilai `10`. Terus kita membikin ekspresi 
`match` pada nilai `x`. Coba perhatikan _patterns_ di dalam _match arms_-nya dan 
`println!` di akhir, dan cobalah buat menebak apa yang bakal dicetak oleh kode ini 
sebelum menjalankannya atau membaca lebih lanjut.

<Listing number="19-11" file-name="src/main.rs" caption="Sebuah ekspresi `match` dengan sebuah _arm_ yang memperkenalkan variabel baru yang menimpa variabel `y` yang sudah ada">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Mari kita telusuri apa yang terjadi pas ekspresi `match` ini dijalankan. _Pattern_ 
di arm pertama tidak cocok dengan nilai yang udah didefinisikan buat `x`, jadi kodenya 
lanjut.

_Pattern_ di arm kedua memperkenalkan sebuah variabel baru bernama `y` yang bakal cocok 
dengan nilai apa pun yang ada di dalam sebuah nilai `Some`. Karena kita sekarang ada di 
dalam _scope_ baru di dalam ekspresi `match` ini, ini adalah variabel `y` yang baru, 
bukannya `y` yang kita deklarasikan di awal tadi yang nilainya `10`. _Binding_ `y` 
yang baru ini bakal cocok dengan nilai apa pun di dalam sebuah `Some`, yang mana 
itulah yang kita punya di dalam `x`. Oleh karena itu, si `y` baru ini mengikat 
(binds) dirinya ke nilai internal yang ada di dalam `Some` yang dimiliki `x`. Nilai 
itu adalah `5`, jadi ekspresi buat arm tersebut dieksekusi dan mencetak 
`Matched, y = 5`.

Seandainya `x` tadi adalah sebuah nilai `None` bukannya `Some(5)`, _patterns_ di dua 
arm pertama tidak bakal ada yang cocok, jadi nilainya bakal cocok sama si garis 
bawah (underscore). Kita tidak memperkenalkan variabel `x` di dalam _pattern_ buat 
arm garis bawah ini, jadi si `x` di ekspresi tersebut tetap merujuk pada `x` yang 
ada di luar yang belum tertimpa (unshadowed). Di kasus hipotetis (seandainya) ini, 
si `match` bakal mencetak `Default case, x = None`.

Begitu ekspresi `match` ini selesai, _scope_-nya berakhir, dan _scope_ buat si 
`y` internal tadi juga berakhir. `println!` terakhir menghasilkan 
`at the end: x = Some(5), y = 10`.

Buat membikin ekspresi `match` yang bisa membandingkan nilai-nilai dari `x` dan `y` 
yang ada di luar, ketimbang memperkenalkan variabel baru yang malah menimpa variabel 
`y` yang sudah ada, kita wajib memakai kondisional _match guard_ sebagai gantinya. 
Kita bakal ngebahas soal _match guards_ nanti di [“Kondisional Tambahan Memakai 
Match Guards”](#kondisional-tambahan-memakai-match-guards).

### Multiple Patterns (Banyak Pola Sekaligus)

Di dalam ekspresi `match`, kita bisa mencocokkan banyak _patterns_ sekaligus dengan 
memakai sintaks `|`, yang mana merupakan operator _or_ (atau) buat _pattern_. 
Misalnya, di kode berikut ini kita mencocokkan nilai `x` terhadap _match arms_-nya, 
yang mana arm pertamanya punya sebuah opsi _or_, yang berarti kalau nilai dari `x` 
itu cocok sama salah satu dari nilai yang ada di arm itu, kode milik arm itu bakal 
dijalankan:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Kode ini mencetak `one or two`.

### Mencocokkan Rentang (Ranges) Nilai Memakai `..=`

Sintaks `..=` memungkinkan kita buat mencocokkan dengan sebuah rentang nilai yang 
inklusif (inclusive range). Di kode berikut, saat sebuah _pattern_ cocok sama 
nilai apa pun yang ada di dalam rentang yang ditentukan, arm tersebut bakal dieksekusi:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Kalau `x` itu adalah `1`, `2`, `3`, `4`, atau `5`, arm pertama bakal cocok. Sintaks 
ini jauh lebih nyaman dipakai pas kita punya banyak nilai buat dicocokkan ketimbang 
memakai operator `|` buat mengekspresikan ide yang sama; kalau kita memakai `|`, 
kita harus menentukan `1 | 2 | 3 | 4 | 5`. Menentukan sebuah rentang (range) itu jauh 
lebih singkat, apalagi kalau kita mau mencocokkan, katakanlah, angka apa aja antara 1 
dan 1.000!

_Compiler_ mengecek kalau rentang tersebut tidak kosong pas _compile time_, dan 
karena tipe-tipe yang mana Rust bisa tahu apakah sebuah rentang itu kosong atau 
tidak itu cuma `char` dan nilai-nilai numerik aja, maka dari itu rentang (ranges) 
cuma diizinkan buat nilai numerik atau `char`.

Berikut ini adalah contoh yang memakai rentang buat nilai `char`:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust bisa tahu kalau `'c'` berada di dalam rentang _pattern_ pertama dan lalu mencetak 
`early ASCII letter`.

### Destructuring (Membongkar) buat Memecah Nilai

Kita juga bisa memakai _patterns_ buat men-_destructure_ (membongkar/memecah belah) 
structs, enums, dan tuples supaya kita bisa memakai berbagai bagian-bagian yang berbeda 
dari nilai-nilai tersebut. Mari kita telusuri masing-masing dari mereka.

#### Destructuring Structs

Listing 19-12 menunjukkan sebuah struct `Point` yang punya dua _fields_ (bidang), 
`x` dan `y`, yang bisa kita pecah memakai sebuah _pattern_ dengan statement `let`.

<Listing number="19-12" file-name="src/main.rs" caption="Membongkar field-field sebuah struct ke dalam variabel-variabel yang terpisah">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Kode ini membikin variabel `a` dan `b` yang masing-masing cocok dengan nilai dari 
field `x` dan `y` pada struct `p`. Contoh ini nunjukin kalau nama variabel-variabel 
di dalam _pattern_-nya tidak harus sama dengan nama-nama field di dalam struct-nya. 
Namun, sudah menjadi hal yang umum buat menyamakan nama variabelnya dengan nama 
field-nya supaya lebih gampang buat diingat variabel mana yang berasal dari field mana. 
Karena penggunaan umum ini, dan karena nulis `let Point { x: x, y: y } = p;` itu berisi 
sangat banyak duplikasi, Rust punya bentuk singkat (shorthand) buat _patterns_ yang 
mencocokkan field struct: kita cuma perlu menyebutkan nama field struct-nya aja, dan 
variabel yang dibikin dari _pattern_ tersebut bakal punya nama yang sama. Listing 
19-13 berperilaku persis sama kayak kode di Listing 19-12, tapi variabel yang dibikin 
di _pattern_ `let`-nya itu bernama `x` dan `y` bukannya `a` dan `b`.

<Listing number="19-13" file-name="src/main.rs" caption="Membongkar field struct memakai _struct field shorthand_ (sintaks pendek field struct)">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Kode ini membikin variabel `x` dan `y` yang cocok dengan field `x` dan `y` dari 
variabel `p`. Hasilnya adalah variabel `x` dan `y` itu sekarang berisi nilai-nilai yang 
ada dari struct `p` tersebut.

Kita juga bisa melakukan _destructure_ memakai nilai literal (literal values) sebagai 
bagian dari _pattern_ struct tersebut ketimbang membikin variabel buat semua 
field-nya. Melakukan hal ini memungkinkan kita buat ngetes (test) beberapa dari field 
tersebut terhadap suatu nilai tertentu sekaligus membikin variabel buat 
men-_destructure_ field-field yang lain.

Di Listing 19-14, kita punya sebuah ekspresi `match` yang memisahkan nilai-nilai 
`Point` ke dalam tiga kasus: titik yang terletak persis di sumbu `x` (yang mana 
itu benar kalau `y = 0`), di sumbu `y` (`x = 0`), atau tidak di sumbu mana pun.

<Listing number="19-14" file-name="src/main.rs" caption="Membongkar dan mencocokkan nilai literal di dalam satu pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Arm pertama bakal cocok sama titik apa pun yang terletak di sumbu `x` dengan menentukan 
bahwa field `y` itu bakal dianggap cocok kalau nilainya cocok dengan literal `0`. 
_Pattern_-nya tetap membikin variabel `x` yang mana bisa kita pakai di kode buat 
arm ini.

Serupa dengan itu, arm kedua mencocokkan titik apa pun yang ada di sumbu `y` dengan 
menentukan bahwa field `x` itu bakal dianggap cocok kalau nilainya `0` dan 
membikin sebuah variabel `y` buat nilai dari field `y` tersebut. Arm ketiga 
sama sekali tidak menentukan literal apa pun, jadi ia cocok buat `Point` yang mana aja 
dan membikin variabel buat kedua field `x` dan `y`.

Di contoh ini, nilai `p` itu cocok dengan arm kedua berkat fakta kalau `x` berisi 
nilai `0`, jadi kode ini bakal mencetak `On the y axis at 7`.

Ingat bahwa sebuah ekspresi `match` bakal berhenti ngecek lengan-lengan (arms) lain 
begitu dia nemuin _pattern_ pertama yang cocok, jadi biarpun `Point { x: 0, y: 0}` 
itu ada di sumbu `x` sekaligus ada di sumbu `y`, kode ini cuma bakal mencetak 
`On the x axis at 0`.

#### Destructuring Enums

Kita udah sering membongkar (destructured) enums di buku ini (misalnya, di Listing 
6-5 di Bab 6), tapi kita belum pernah secara eksplisit ngebahas kalau _pattern_ yang 
dipakai buat membongkar sebuah enum itu berhubungan erat dengan gimana cara data 
yang tersimpan di dalam enum tersebut didefinisikan. Sebagai contoh, di Listing 
19-15 kita memakai enum `Message` dari Listing 6-2 lalu kita menulis sebuah `match` 
yang punya _patterns_ buat membongkar setiap nilai internalnya.

<Listing number="19-15" file-name="src/main.rs" caption="Membongkar varian enum yang memegang berbagai macam nilai yang berbeda">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Kode ini bakal mencetak `Change color to red 0, green 160, and blue 255`. Cobalah 
buat mengubah nilai dari `msg` buat melihat kode dari lengan-lengan yang lain itu 
jalan.

Buat varian enum yang tidak punya data apa pun, kayak `Message::Quit`, kita tidak bisa 
membongkar nilainya lebih jauh lagi. Kita cuma bisa mencocokkan terhadap literal nilai 
`Message::Quit` secara langsung, dan tidak ada variabel sama sekali di dalam 
_pattern_ tersebut.

Buat varian enum yang bentuknya kayak struct (struct-like enum variants), seperti 
`Message::Move`, kita bisa memakai sebuah _pattern_ yang mirip sama _pattern_ yang kita 
tentukan buat mencocokkan structs. Setelah nama variannya, kita taruh kurung kurawal 
lalu kita sebutkan field-fieldnya beserta variabelnya supaya kita bisa memecah-mecah 
bagian-bagian itu buat dipakai di dalam kode untuk arm ini. Di sini kita memakai 
bentuk singkat (_shorthand_) sama seperti yang kita lakuin di Listing 19-13.

Buat varian enum yang bentuknya kayak tuple (tuple-like enum variants), kayak 
`Message::Write` yang menampung sebuah tuple dengan satu elemen dan 
`Message::ChangeColor` yang menampung sebuah tuple dengan tiga elemen, _pattern_-nya 
itu mirip sama _pattern_ yang kita tentukan buat mencocokkan tuples. Jumlah variabel 
di dalam _pattern_-nya itu harus cocok dengan jumlah elemen di dalam varian yang 
lagi kita cocokkan tersebut.

#### Destructuring Structs dan Enums yang Bersarang (Nested)

Sejauh ini, contoh-contoh kita semuanya cuma mencocokkan structs atau enums dengan 
kedalaman satu level aja (one level deep), tapi pencocokan itu bisa juga lho dipakai 
buat item-item yang bersarang (nested items)! Misalnya, kita bisa merombak (refactor) 
kode yang ada di Listing 19-15 buat ngedukung warna RGB maupun HSV di dalam pesan 
`ChangeColor`, kayak yang ditunjukin di Listing 19-16.

<Listing number="19-16" caption="Pencocokan pada enums yang bersarang (nested enums)">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

_Pattern_ dari arm pertama di ekspresi `match` itu cocok dengan varian enum 
`Message::ChangeColor` yang mengandung varian `Color::Rgb`; dan kemudian _pattern_ 
itu bakal mengikat dirinya (binds) ke ketiga nilai `i32` internal tersebut. 
_Pattern_ di arm kedua juga cocok dengan varian enum `Message::ChangeColor`, tapi 
enum internalnya itu cocok dengan `Color::Hsv` sebagai gantinya. Kita bisa menentukan 
kondisi-kondisi kompleks (complex conditions) ini semuanya di dalam satu ekspresi 
`match` tunggal, meskipun ada dua enum yang dilibatkan.

#### Destructuring Structs dan Tuples Bersamaan

Kita bisa nyampur (mix), mencocokkan (match), dan menyarangkan (nest) _destructuring 
patterns_ dengan cara yang bahkan lebih kompleks lagi. Contoh berikut nunjukin proses 
_destructure_ yang lumayan rumit di mana kita menyarangkan structs dan tuples di dalam 
sebuah tuple lalu kita mengekstrak (_destructure_) semua nilai primitifnya sampai 
keluar (out):

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Kode ini membiarkan kita memecah tipe-tipe yang kompleks menjadi bagian-bagian 
komponen dasarnya (component parts) supaya kita bisa memakai nilai-nilai yang kita 
butuhkan secara terpisah.

_Destructuring_ memakai _patterns_ ini adalah cara yang nyaman sekali buat memakai 
potongan-potongan dari sebuah nilai, kayak misalnya nilai yang ada dari masing-
masing field di dalam sebuah struct, secara terpisah dari satu sama lain.

### Mengabaikan Nilai di dalam sebuah Pattern

kita udah melihat kalau kadang-kadang itu sangat berguna buat mengabaikan nilai-
nilai di dalam sebuah _pattern_, kayak misalnya di arm terakhir dari sebuah `match`, 
buat ngedapetin _catch-all_ yang tidak benar-benar ngelakuin apa-apa tapi tetep 
mempertimbangkan (account for) semua kemungkinan nilai yang tersisa. Ada beberapa 
cara buat mengabaikan seluruh nilai atau sebagian aja dari suatu nilai di dalam 
sebuah _pattern_: dengan memakai _pattern_ `_` (yang udah kita lihat), dengan memakai 
_pattern_ `_` di dalam _pattern_ lainnya, dengan memakai nama yang diawali dengan 
garis bawah (underscore), atau dengan memakai `..` buat mengabaikan semua sisa bagian 
dari sebuah nilai. Mari kita eksplorasi gimana dan kapan alasan yang tepat buat 
memakai masing-masing _patterns_ ini.

<!-- Old link, do not remove -->

<a id="ignoring-an-entire-value-with-_"></a>

#### Mengabaikan Seluruh Nilai Memakai `_`

Kita udah memakai garis bawah (underscore) sebagai _wildcard pattern_ (pola yang bisa 
jadi apa saja) yang bakal cocok dengan nilai apa pun tapi tidak bakal mengikat 
dirinya (bind) ke nilai tersebut. Ini sangat berguna dipakai sebagai arm terakhir di 
dalam ekspresi `match`, tapi kita juga bisa memakainya di _pattern_ mana aja lho, 
termasuk di parameter fungsi, kayak yang ditunjukin di Listing 19-17.

<Listing number="19-17" file-name="src/main.rs" caption="Memakai `_` di dalam _signature_ sebuah fungsi">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Kode ini bakal bener-bener mengabaikan nilai `3` yang dioper sebagai argumen 
pertama, dan dia bakal mencetak `This code only uses the y parameter: 4`.

Di mayoritas kasus saat kita udah tidak butuh suatu parameter fungsi tertentu lagi, 
seharusnya kita sekalian aja ngubah _signature_-nya supaya dia tidak nyertain 
parameter yang tidak terpakai itu. Mengabaikan sebuah parameter fungsi bisa sangat 
sangat berguna di kasus-kasus di mana, misalnya, kita lagi mengimplementasikan 
sebuah trait saat kita diwajibkan buat punya _type signature_ yang spesifik tapi 
isi fungsinya (function body) di implementasi kita sama sekali tidak butuh salah satu 
parameternya. Dengan melakukan itu kita bisa terhindar dari dapat peringatan (warning) 
_compiler_ tentang parameter fungsi yang tidak terpakai (unused function parameters), 
yang mana peringatan itu bakal muncul kalau seandainya kita malah memakai sebuah nama.

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### Mengabaikan Bagian-bagian dari Sebuah Nilai Memakai `_` yang Bersarang (Nested)

Kita juga bisa memakai `_` di dalam _pattern_ lain buat cuma ngabaiin sebagian 
dari sebuah nilai, misalnya, pas kita cuma pengen ngetes sebagian dari sebuah 
nilai tapi tidak punya alasan apa pun buat memakai bagian-bagian lainnya di dalam 
kode korespondensinya yang mau kita jalanin. Listing 19-18 nunjukin kode yang 
bertanggung jawab buat mengelola nilai dari suatu pengaturan (setting). Persyaratan 
bisnisnya adalah _user_ tidak boleh ngubah (overwrite) penyesuaian (customization) 
yang udah ada pada suatu _setting_, tapi _user_ boleh aja menghapus (_unset_) 
_setting_ tersebut lalu ngasih dia nilai baru kalau _setting_ tersebut saat itu 
lagi kosong (unset).

<Listing number="19-18" caption="Memakai garis bawah di dalam _patterns_ yang nyocokin varian `Some` pas kita tidak perlu buat memakai nilai di dalam `Some` tersebut">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Kode ini bakal mencetak `Can't overwrite an existing customized value` dan terus 
`setting is Some(5)`. Di match arm yang pertama, kita tidak perlu mencocokkan atau 
memakai nilai-nilai yang ada di dalam masing-masing varian `Some`, tapi kita benar-benar 
perlu buat ngetes kasus di mana `setting_value` dan `new_setting_value` dua-duanya 
adalah varian `Some`. Di kasus itu, kita mencetak alasan kenapa kita tidak 
mengubah `setting_value`, dan nilainya emang tidak jadi diubah.

Di semua kasus lainnya (kalau entah `setting_value` atau `new_setting_value` itu `None`) 
yang diekspresikan sama _pattern_ `_` di arm yang kedua, kita pengen ngebolehin 
`new_setting_value` buat ngegantiin (become) `setting_value`.

Kita juga bisa memakai garis bawah di banyak tempat di dalam satu _pattern_ tunggal 
buat ngabaiin beberapa nilai tertentu. Listing 19-19 nunjukin sebuah contoh gimana 
caranya ngabaiin nilai kedua dan keempat di dalam sebuah tuple yang isinya ada lima item.

<Listing number="19-19" caption="Mengabaikan beberapa bagian dari sebuah tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Kode ini bakal mencetak `Some numbers: 2, 8, 32`, dan nilai `4` dan `16` bakal 
diabaikan.

<!-- Old link, do not remove -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### Mengabaikan Variabel yang Tidak Terpakai dengan Mengawali Namanya Memakai `_`

Kalau kita membikin sebuah variabel tapi tidak pernah memakannya di mana pun, Rust 
biasanya bakal mengeluarkan peringatan (warning) karena variabel yang tidak 
terpakai itu bisa aja merupakan sebuah _bug_. Namun, kadang-kadang itu kepake 
sekali buat bisa bikin sebuah variabel yang belum mau kita pakai sekarang, kayak 
pas kita lagi nge-prototipe (_prototyping_) atau pas lagi memulai sebuah project. 
Di situasi ini, kita bisa ngasih tahu Rust supaya tidak ngasih peringatan ke 
kita soal variabel yang tidak terpakai dengan ngawalin nama variabelnya pakai 
sebuah garis bawah (underscore). Di Listing 19-20, kita bikin dua variabel 
yang tidak dipakai, tapi pas kita mengompilasi kode ini, kita harusnya cuma bakal 
dapat satu peringatan aja tentang salah satunya.

<Listing number="19-20" file-name="src/main.rs" caption="Mengawali nama variabel dengan garis bawah buat ngehindarin peringatan variabel yang tidak terpakai">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Di sini, kita dapat satu peringatan karena tidak memakai variabel `y`, tapi kita 
tidak dapat peringatan apa pun soal tidak memakai `_x`.

Perhatikan bahwa ada sedikit perbedaan yang halus (subtle difference) antara memakai 
`_` sendirian dan memakai nama yang diawali dengan sebuah garis bawah. Sintaks 
`_x` itu tetap mengikat (binds) nilainya ke variabel tersebut, sementara `_` 
tidak mengikat nilai itu sama sekali. Buat nunjukin di mana letak kasus yang mana 
perbedaan ini jadi penting, Listing 19-21 bakal ngasih kita sebuah error.

<Listing number="19-21" caption="Sebuah variabel yang tidak terpakai dan diawali dengan sebuah garis bawah itu tetap mengikat nilainya, yang mana bisa aja ngambil kepemilikan (ownership) dari nilai tersebut.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Kita bakal nerima sebuah error karena nilai `s` tersebut bakal tetep aja di-_move_ 
(dipindahkan) ke dalam `_s`, yang mana mencegah kita dari memakai `s` lagi 
setelah itu. Namun, memakai garis bawah itu sendiri aja tidak pernah mengikat (bind) 
ke nilainya. Listing 19-22 bakal berhasil di-compile tanpa error apa pun karena `s` 
tidak dipindahkan (moved) ke dalam `_`.

<Listing number="19-22" caption="Memakai sebuah garis bawah aja tidak bakal mengikat nilainya.">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Kode ini bisa jalan dengan lancar karena kita bener-bener tidak pernah mengikat 
`s` ke mana pun; dia tidak dipindahkan.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### Mengabaikan Sisa Bagian dari Sebuah Nilai Memakai `..`

Buat nilai-nilai yang punya banyak bagian, kita bisa memakai sintaks `..` buat 
cuma memakai bagian-bagian yang spesifik aja lalu ngabaiin sisanya, sehingga 
kita bisa menghindari nulisin garis bawah buat setiap nilai yang mau kita 
abaikan. _Pattern_ `..` mengabaikan bagian mana pun dari sebuah nilai yang belum 
kita cocokkan (matched) secara eksplisit di sisa _pattern_ tersebut. Di Listing 19-23, 
kita punya struct `Point` yang memegang titik koordinat di ruang tiga dimensi 
(three-dimensional space). Di dalam ekspresi `match`, kita cuma mau beroperasi pada 
koordinat `x` doang dan ngabaiin nilai-nilai yang ada di field `y` dan `z`.

<Listing number="19-23" caption="Mengabaikan semua field dari `Point` kecuali buat `x` dengan memakai `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Kita nyebutin nilai `x` lalu setelah itu kita cuma masukin _pattern_ `..` aja. 
Ini jauh lebih cepet daripada kita harus nulisin `y: _` dan `z: _`, terutama 
pas kita lagi ngerjain structs yang punya sangat banyak field di mana cuma ada satu 
atau dua field doang yang relevan buat kita.

Sintaks `..` ini bakal melebar (expand) ke seberapa banyak pun nilai yang dibutuhin. 
Listing 19-24 nunjukin gimana cara memakai `..` bareng sebuah tuple.

<Listing number="19-24" file-name="src/main.rs" caption="Cuma mencocokkan nilai yang pertama dan yang terakhir dari sebuah tuple dan mengabaikan semua nilai-nilai lainnya">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Di kode ini, nilai yang pertama dan yang terakhir bakal dicocokkan dengan `first` 
dan `last`. `..` itu bakal nyocokin dan lalu ngabaiin semua hal yang ada di 
tengah-tengah mereka berdua.

Namun, pemakaian `..` itu harus tidak ambigu (unambiguous). Kalau sampai tidak 
jelas nilai-nilai yang mana aja yang ditujukan buat dicocokin dan nilai yang 
mana yang seharusnya diabaikan, Rust bakal ngasih kita sebuah error. Listing 
19-25 nunjukin sebuah contoh dari pemakaian `..` secara ambigu, jadi dia tidak 
bakal bisa di-compile.

<Listing number="19-25" file-name="src/main.rs" caption="Sebuah percobaan buat memakai `..` dengan cara yang ambigu">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Pas kita men-compile contoh ini, kita dapet error ini:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Mustahil bagi Rust buat nentuin seberapa banyak nilai di dalam tuple tersebut 
yang harus diabaikan sebelum dia mencocokkan sebuah nilai dengan `second` lalu 
berapa banyak lagi nilai selanjutnya yang harus diabaikan setelahnya. Kode ini bisa 
aja berarti kalau kita mau ngabaiin `2`, mengikat `second` ke `4`, dan terus 
ngabaiin `8`, `16`, dan `32`; atau bisa juga berarti kalau kita mau ngabaiin `2` dan 
`4`, mengikat `second` ke `8`, dan lalu ngabaiin `16` dan `32`; dan seterusnya 
(and so forth). Nama variabel `second` itu tidak punya arti khusus apa-apa buat Rust, 
jadi kita dapet error _compiler_ karena memakai `..` di dua tempat kayak gini itu 
ambigu.

### Kondisional Tambahan Memakai Match Guards

Sebuah _match guard_ adalah tambahan kondisi `if`, yang ditentukan setelah _pattern_ 
di dalam sebuah lengan (arm) `match`, yang wajib dicocokkan juga supaya arm tersebut 
bisa dipilih. Match guards ini berguna buat mengekspresikan ide-ide yang lebih 
kompleks daripada apa yang bisa diizinkan sama sebuah _pattern_ sendirian. Namun, 
perhatikan bahwa mereka itu cuma tersedia di dalam ekspresi `match`, bukan di 
ekspresi `if let` atau `while let`.

Kondisi tersebut bisa memakai variabel yang dibikin di dalam _pattern_-nya. Listing 
19-26 nunjukin sebuah `match` di mana arm pertamanya punya _pattern_ `Some(x)` 
dan juga punya sebuah match guard `if x % 2 == 0` (yang bakal jadi `true` kalau 
angkanya itu bilangan genap).

<Listing number="19-26" caption="Nambahin sebuah match guard ke dalam sebuah pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Contoh ini bakal mencetak `The number 4 is even`. Pas `num` dibandingkan dengan 
_pattern_ di arm yang pertama, dia cocok karena `Some(4)` cocok dengan `Some(x)`. 
Terus si match guard mengecek apakah sisa dari ngebagi `x` dengan 2 itu sama 
dengan 0, dan karena emang bener, arm yang pertama itu pun dipilih.

Kalau seandainya `num` tadi adalah `Some(5)`, si match guard di arm pertama 
bakal jadi `false` karena sisa pembagian 5 dengan 2 itu adalah 1, yang mana tidak 
sama dengan 0. Rust terus bakal lanjut ke arm yang kedua, yang mana bakal cocok 
karena arm yang kedua itu tidak punya match guard dan makanya dia cocok buat varian 
`Some` yang mana aja.

Tidak ada cara buat mengekspresikan kondisi `if x % 2 == 0` di dalam sebuah 
_pattern_ secara langsung, jadi match guard ngasih kita kemampuan buat bisa 
mengekspresikan logika ini. Kekurangan dari penambahan fungsionalitas ekspresi 
(expressiveness) ekstra ini adalah kalau _compiler_ tidak bakal mencoba buat 
mengecek kelengkapannya (exhaustiveness) saat ada ekspresi-ekspresi match guard yang 
dilibatkan.

Di Listing 19-11, kita sempat menyebutkan kalau kita bisa memakai match guards 
buat nyelesein masalah tertimpanya variabel (pattern-shadowing problem) yang 
kita alami. Ingat kembali kalau waktu itu kita membikin sebuah variabel baru di 
dalam _pattern_ di ekspresi `match`-nya ketimbang memakai variabel yang udah ada di 
luar `match`. Variabel yang baru itu berarti kalau kita tidak bisa lagi melakukan 
pengetesan terhadap (test against) nilai dari variabel yang ada di luar itu. 
Listing 19-27 nunjukin gimana caranya kita bisa memakai sebuah match guard buat 
memperbaiki masalah ini.

<Listing number="19-27" file-name="src/main.rs" caption="Memakai match guard buat ngetes kesamaan (equality) dengan sebuah variabel di luar">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Kode ini sekarang bakal mencetak `Default case, x = Some(5)`. _Pattern_ di arm 
match yang kedua tidak memperkenalkan sebuah variabel baru `y` yang mana bakal 
menimpa `y` yang ada di luar, yang berarti kita bisa memakai `y` luar tersebut di 
dalam match guard-nya. Ketimbang menentukan _pattern_-nya sebagai `Some(y)`, yang mana 
bakal menimpa `y` luar, kita menentukannya sebagai `Some(n)`. Ini membikin 
sebuah variabel baru `n` yang tidak menimpa apa pun karena tidak ada variabel 
`n` di luar `match`.

Match guard `if n == y` itu bukanlah sebuah _pattern_ dan makanya dia tidak 
memperkenalkan variabel baru apa pun. `y` ini _adalah_ si `y` luar ketimbang 
sebuah `y` baru yang menimpanya, dan kita bisa nyari-nyari sebuah nilai yang punya 
nilai yang sama kayak si `y` luar tersebut dengan membandingkan `n` ke `y`.

Kita juga bisa memakai operator _or_ `|` di dalam sebuah match guard buat 
menentukan lebih dari satu _patterns_; di mana kondisi dari match guard tersebut 
bakal berlaku buat kesemua _patterns_-nya. Listing 19-28 nunjukin hierarki (precedence) 
saat ngegabungin sebuah _pattern_ yang memakai `|` dengan sebuah match guard. Bagian 
yang penting dari contoh ini adalah bahwa si match guard `if y` ini berlaku buat 
`4`, `5`, _dan_ `6`, biarpun dia mungkin kelihatannya kayak si `if y` ini cuma 
berlaku buat `6` doang.

<Listing number="19-28" caption="Menggabungkan banyak patterns sekaligus dengan sebuah match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Kondisi _match_ itu menyatakan kalau arm tersebut cuma cocok kalau nilainya `x` 
itu sama dengan `4`, `5`, atau `6` _dan_ kalau `y` itu `true`. Pas kode ini jalan, 
_pattern_ dari arm pertama cocok karena `x` adalah `4`, tapi si match guard `if y` 
itu `false`, jadi arm pertama tersebut tidak dipilih. Kodenya lanjut ke arm yang 
kedua, yang mana emang cocok, dan program ini mencetak `no`. Alasannya adalah 
karena kondisi `if` tersebut berlaku buat keseluruhan _pattern_ `4 | 5 | 6`, bukan 
cuma buat nilai terakhir `6` aja. Dengan kata lain, hierarki dari sebuah match 
guard sehubungan dengan (in relation to) sebuah _pattern_ itu berperilaku kayak gini:

```text
(4 | 5 | 6) if y => ...
```

bukannya kayak gini:

```text
4 | 5 | (6 if y) => ...
```

Setelah menjalankan kodenya, perilaku hierarki ini terlihat dengan jelas: kalau 
match guard tersebut cuma diterapin ke nilai terakhir di daftar nilai yang ditentukan 
memakai operator `|`, arm tersebut pasti udah cocok dan programnya pasti bakal 
mencetak `yes`.

### Binding Memakai `@`

Operator _at_ `@` membiarkan kita membikin sebuah variabel yang nampung sebuah nilai 
di waktu yang bersamaan dengan kita melakukan tes apakah nilai tersebut cocok sama 
suatu _pattern_ (pattern match) atau tidak. Di Listing 19-29, kita mau ngetes apakah 
field `id` di sebuah `Message::Hello` itu ada di dalam rentang `3..=7`. Kita juga mau 
mengikat (bind) nilai tersebut ke variabel `id` supaya kita bisa memakainya di 
dalam kode yang diasosiasikan (associated) sama arm tersebut.

<Listing number="19-29" caption="Memakai `@` buat nge-bind ke suatu nilai di dalam sebuah pattern sembari juga ngetes (testing) nilai tersebut">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Contoh ini bakal mencetak `Found an id in range: 5`. Dengan menspesifikasikan `id @` 
sebelum rentang `3..=7`, kita lagi menangkap nilai apa pun yang sekiranya cocok sama 
rentang itu lalu menaruhnya ke dalam sebuah variabel bernama `id`, dan sembari ngelakuin 
itu kita juga ngetes kalau nilai tersebut cocok sama _pattern_ rentang (range 
pattern) tersebut.

Di arm yang kedua, di mana kita cuma menentukan sebuah rentang doang di dalam 
_pattern_-nya, kode yang terkait sama arm tersebut tidak punya variabel yang berisi 
nilai asli dari field `id` itu. Nilai dari field `id` itu bisa jadi adalah 10, 11, 
atau 12, tapi kode yang ada di arm _pattern_ tersebut sama sekali tidak tahu nilainya yang 
mana. Kode di _pattern_ tersebut tidak mampu buat memakai nilai dari field `id`, karena 
kita belum nyimpan nilai `id` tersebut di sebuah variabel.

Di arm terakhir, di mana kita udah menyebutkan sebuah variabel tanpa adanya rentang, 
kita bener-bener punya akses ke nilai yang bisa kita pakai di dalam kode untuk arm 
tersebut di sebuah variabel bernama `id`. Alasannya adalah karena kita udah memakai 
sintaks _shorthand_ field struct. Tapi kita belum menerapkan tes (test) apa-apa ke 
dalam nilai di field `id` ini buat arm yang ini, seperti yang udah kita lakuin di dua 
arm pertama: nilai apa pun bakal cocok dengan _pattern_ ini.

Memakai `@` ngasih kita kemampuan buat ngetes sebuah nilai dan kemudian menyimpannya 
ke dalam sebuah variabel dalam satu _pattern_ tunggal.

## Ringkasan

_Patterns_ di Rust itu sangat sangat berguna dalam membedakan (distinguishing) berbagai 
macam jenis data yang berbeda. Pas mereka dipakai di dalam ekspresi `match`, Rust 
memastikan kalau _patterns_ kita udah mencakup semua kemungkinan nilai yang ada, 
karena kalau tidak program kita tidak bakal bisa di-compile. _Patterns_ yang ada di 
dalam statement `let` dan parameter fungsi ngebikin konstruk-konstruk tersebut jadi 
jauh lebih berguna, memungkinkan proses memecah (_destructuring_) sebuah nilai jadi 
bagian-bagian yang lebih kecil dan meng-assign (ngasih nilai ke) bagian-bagian itu 
ke dalam berbagai variabel. Kita bisa bikin _patterns_ yang simpel maupun yang 
kompleks buat menyesuaikan dengan apa yang kita butuhkan.

Berikutnya, untuk bab kedua dari akhir buku ini, kita bakal melihat 
beberapa aspek tingkat mahir (advanced aspects) dari berbagai macam fitur 
yang ada di Rust.
