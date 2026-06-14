## Contoh Program pake Structs

Buat mahamin kapan kita mungkin mau pake struct, yuk kita tulis program yang 
ngitung luas (_area_) dari sebuah persegi panjang. Kita bakal mulai dengan pake 
variabel satu-satu, terus kita _refactor_ programnya sampe kita pake struct 
sebagai gantinya.

Yuk kita bikin project biner baru pake Cargo namanya _rectangles_ yang bakal 
nerima lebar (_width_) sama tinggi (_height_) dari persegi panjang dalam pixel 
terus ngitung luasnya. Listing 5-8 nunjukin program pendek dengan satu cara 
buat ngelakuin itu di file _src/main.rs_ project kita.

<Listing number="5-8" file-name="src/main.rs" caption="Ngitung luas persegi panjang yang ditentuin pake variabel lebar sama tinggi yang terpisah">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Sekarang, jalanin program ini pake `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Kode ini berhasil nyari tau luas persegi panjangnya dengan manggil fungsi 
`area` pake tiap dimensinya, tapi kita bisa lakuin lebih banyak lagi buat bikin 
kode ini lebih jelas dan enak dibaca.

Masalah dari kode ini keliatan sekali di signature fungsi `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Fungsi `area` harusnya ngitung luas dari satu persegi panjang, tapi fungsi yang 
kita tulis punya dua parameter, dan nggak jelas di mana pun di program kita 
kalau parameter-parameter itu sebenernya berhubungan. Bakal lebih enak dibaca 
dan lebih gampang dikelola kalau kita ngelempokin lebar sama tinggi jadi satu. 
Kita udah bahas salah satu cara buat lakuin itu di bagian [“Tipe Tuple”][the-tuple-type] 
di Bab 3: yaitu pake tuple.

### Refactoring pake Tuples

Listing 5-9 nunjukin versi lain dari program kita yang pake tuple.

<Listing number="5-9" file-name="src/main.rs" caption="Nentuin lebar sama tinggi persegi panjang pake sebuah tuple">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Dalam satu sisi, program ini lebih baik. Tuple ngebolehin kita nambahin sedikit 
struktur, dan kita sekarang cuma masukin satu argumen doang. Tapi di sisi lain, 
versi ini kurang jelas: tuple nggak ngasih nama ke elemen-elemennya, jadi kita 
harus ngindeks ke bagian-bagian tuple-nya, yang bikin kalkulasi kita jadi 
kurang gamblang.

Ketuker antara lebar sama tinggi nggak bakal ngaruh buat kalkulasi luas, tapi 
kalau kita mau gambar persegi panjangnya di layar, itu baru ngaruh sekali! 
Kita harus terus inget kalau `width` itu indeks tuple `0` dan `height` itu 
indeks tuple `1`. Ini bakal makin susah buat orang lain buat cari tau dan 
diinget-inget kalau mereka mau pake kode kita. Karena kita nggak nyampein 
makna dari data kita di kode, sekarang jadi lebih gampang buat masukin error.

### Refactoring pake Structs: Nambahin Lebih Banyak Makna

Kita pake struct buat nambahin makna dengan ngasih label ke datanya. Kita bisa 
ngubah tuple yang kita pake jadi sebuah struct dengan nama buat keseluruhannya 
sama nama buat tiap bagiannya, kayak yang ditunjukin di Listing 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Mendefinisikan struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Di sini, kita udah mendefinisikan sebuah struct terus dikasih nama `Rectangle`. 
Di dalem kurung kurawal, kita mendefinisikan field-field-nya sebagai `width` 
sama `height`, yang keduanya punya tipe `u32`. Terus, di `main`, kita bikin 
instance tertentu dari `Rectangle` yang punya lebar `30` sama tinggi `50`.

Fungsi `area` kita sekarang didefinisikan dengan satu parameter, yang kita 
kasih nama `rectangle`, yang tipenya adalah _immutable borrow_ dari sebuah 
instance struct `Rectangle`. Kayak yang udah disebutin di Bab 4, kita mau minjem 
(_borrow_) struct-nya bukannya ngambil _ownership_-nya. Dengan cara ini, `main` 
tetep megang _ownership_-nya dan bisa lanjut pake `rect1`, yang merupakan 
alasan kenapa kita pake `&` di signature fungsi sama pas kita manggil fungsinya.

Fungsi `area` akses field `width` sama `height` dari instance `Rectangle` 
(perhatiin ya kalau akses field dari instance struct yang dipinjem nggak bakal 
nge-_move_ nilai field-nya, makanya kita sering liat peminjaman struct). 
Signature fungsi kita buat `area` sekarang bilang persis apa yang kita maksud: 
itung luas dari `Rectangle`, pake field `width` sama `height`-nya. Ini 
nyampein kalau lebar sama tinggi itu berhubungan satu sama lain, dan ngasih 
nama deskriptif ke nilai-nilainya bukannya pake nilai indeks tuple `0` sama `1`. 
Ini kemenangan buat kejelasan kodenya.

### Nambahin Fungsionalitas Berguna pake Derived Traits

Bakal sangat berguna kalau kita bisa nyetak sebuah instance dari `Rectangle` 
pas lagi debugging program kita dan liat nilai buat semua field-nya. Listing 5-11 
nyoba pake [macro `println!`][println] kayak yang udah kita pake di bab-bab 
sebelumnya. Tapi, ini nggak bakal jalan.

<Listing number="5-11" file-name="src/main.rs" caption="Nyoba nyetak instance `Rectangle`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Pas kita compile kode ini, kita dapet error dengan pesan inti kayak gini:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Macro `println!` bisa ngelakuin banyak jenis format, dan secara default, kurung 
kurawal ngasih tau `println!` buat pake format yang dikenal sebagai `Display`: 
output yang tujuannya buat dikonsumsi langsung sama end user. Tipe-tipe 
primitif yang udah kita liat sejauh ini mengimplementasikan `Display` secara 
default karena cuma ada satu cara kita mau nunjukin angka `1` atau tipe 
primitif lainnya ke user. Tapi sama struct, gimana cara `println!` harus 
format output-nya itu kurang jelas karena ada banyak kemungkinan tampilan: Mau 
pake koma apa nggak? Mau nyetak kurung kurawal-nya juga? Apakah semua field 
harus ditunjukin? Karena ambiguitas ini, Rust nggak nyoba buat nebak apa yang 
kita mau, dan struct nggak dikasih implementasi bawaan dari `Display` buat 
dipake bareng `println!` sama placeholder `{}`.

Kalau kita lanjut baca error-nya, kita bakal nemu catatan berguna ini:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Yuk kita coba! Pemanggilan macro `println!` sekarang bakal keliatan kayak 
`println!("rect1 is {rect1:?}");`. Naruh penentu `:?` di dalem kurung kurawal 
ngasih tau `println!` kalau kita mau pake format output namanya `Debug`. Trait 
`Debug` ngebolehin kita nyetak struct kita dengan cara yang berguna buat 
developer biar kita bisa liat nilainya pas lagi debugging kode kita.

Compile kodenya dengan perubahan ini. Yah! Masih dapet error:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Tapi lagi-lagi, _compiler_-nya ngasih catatan yang ngebantu:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _emang_ masukin fungsionalitas buat nyetak info debugging, tapi kita harus 
secara eksplisit milih (_opt in_) buat bikin fungsionalitas itu tersedia buat 
struct kita. Caranya, kita tambahin atribut luar `#[derive(Debug)]` tepat 
sebelum definisi struct-nya, kayak yang ditunjukin di Listing 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Nambahin atribut buat derive trait `Debug` terus nyetak instance `Rectangle` pake debug formatting">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Sekarang pas kita jalanin programnya, kita nggak bakal dapet error apa-apa, dan 
kita bakal liat output berikut:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Mantap! Emang bukan output yang paling cakep sih, tapi dia nunjukin nilai dari 
semua field buat instance ini, yang pasti bakal ngebantu sekali pas lagi 
debugging. Pas kita punya struct yang lebih gede, bakal berguna kalau punya 
output yang sedikit lebih gampang dibaca; di kasus kayak gitu, kita bisa pake 
`{:#?}` bukannya `{:?}` di string `println!`. Di contoh ini, pake gaya `{:#?}` 
bakal ngeluarin output kayak gini:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Cara lain buat nyetak sebuah nilai pake format `Debug` itu pake [macro `dbg!`][dbg], 
yang ngambil _ownership_ dari sebuah ekspresi (beda sama `println!`, yang 
ngambil referensi), nyetak nama file sama nomor baris di mana pemanggilan macro 
`dbg!` itu ada di kode kita barengan sama nilai hasil dari ekspresi itu, terus 
balikin _ownership_ nilainya.

> Catatan: Manggil macro `dbg!` itu nyetaknya ke stream konsol standard error 
> (`stderr`), beda sama `println!`, yang nyetaknya ke stream konsol standard 
> output (`stdout`). Kita bakal bahas lebih banyak soal `stderr` sama `stdout` 
> di bagian [“Menulis Pesan Error ke Standard Error Bukannya Standard Output” 
> di Bab 12][err].

Ini contoh di mana kita tertarik sama nilai yang di-assign ke field `width`, 
sekaligus nilai dari seluruh struct di `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Kita bisa naruh `dbg!` di sekitar ekspresi `30 * scale` dan, karena `dbg!` 
balikin _ownership_ dari nilai ekspresinya, field `width` bakal dapet nilai yang 
sama kayak kalau kita nggak ada pemanggilan `dbg!` di situ. Kita nggak mau 
`dbg!` ngambil _ownership_ dari `rect1`, jadi kita pake sebuah referensi ke 
`rect1` di pemanggilan selanjutnya. Ini penampakan output dari contoh ini:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Kita bisa liat bagian output pertama dateng dari _src/main.rs_ baris 10 di mana 
kita lagi debugging ekspresi `30 * scale`, dan nilai hasilnya adalah `60` 
(formatting `Debug` yang diimplementasikan buat integer itu nyetak nilainya 
doang). Pemanggilan `dbg!` di baris 14 dari _src/main.rs_ ngeluarin nilai dari 
`&rect1`, yaitu struct `Rectangle`. Output ini pake formatting `Debug` yang 
rapi dari tipe `Rectangle`. Macro `dbg!` ini bisa ngebantu sekali pas kita 
lagi nyoba cari tau apa yang sebenernya lagi dilakuin kode kita!

Selain trait `Debug`, Rust juga nyediain sejumlah traits buat kita pake bareng 
atribut `derive` yang bisa nambahin perilaku berguna ke tipe data kustom kita. 
Traits itu sama perilakunya ada di daftar di [Lampiran C][app-c]. Kita bakal 
bahas gimana cara mengimplementasikan traits ini dengan perilaku kustom 
sekaligus gimana cara bikin traits kita sendiri di Bab 10. Ada juga banyak 
atribut lain selain `derive`; buat info lebih lanjut, liat [bagian “Attributes” 
di Rust Reference][attributes].

Fungsi `area` kita itu sangat spesifik: dia cuma ngitung luas persegi panjang. 
Bakal ngebantu kalau kita ngiket perilaku ini lebih deket ke struct `Rectangle` 
kita karena dia nggak bakal jalan sama tipe lainnya. Yuk kita liat gimana kita 
bisa lanjut _refactor_ kode ini dengan ngerubah fungsi `area` jadi sebuah 
_method_ `area` yang didefinisikan pada tipe `Rectangle` kita.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
