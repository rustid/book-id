## Macros

Kita udah sering memakai macros (makro) kayak `println!` di sepanjang buku ini, 
tapi kita belum sepenuhnya mengeksplorasi apa itu _macro_ dan gimana cara 
kerjanya. Istilah _macro_ mengacu ke sekumpulan fitur di Rust: macros _declarative_ 
(deklaratif) yang memakai `macro_rules!` dan tiga macam macros _procedural_ 
(prosedural):

- Macros `#[derive]` kustom yang menentukan kode yang bakal ditambahin 
  pakai atribut `derive` yang dipakai pada structs dan enums
- Macros mirip atribut (attribute-like) yang mendefinisikan atribut kustom 
  yang bisa dipakai pada item apa pun
- Macros mirip fungsi (function-like) yang kelihatannya kayak pemanggilan 
  fungsi tapi beroperasi pada tokens yang ditentukan sebagai argumen mereka

Kita bakal ngebahas masing-masing dari ini secara bergantian, tapi pertama-
tama, mari kita bahas kenapa kita butuh macros padahal kita udah punya fungsi 
biasa.

### Perbedaan Antara Macros dan Fungsi

Secara fundamental, macros itu adalah sebuah cara buat nulis kode yang 
nulisin kode lain, yang mana dikenal dengan istilah _metaprogramming_ (metapemrograman). 
Di Lampiran C, kita ngebahas soal atribut `derive`, yang menghasilkan 
(generates) sebuah implementasi dari berbagai traits buat kita. Kita juga udah 
memakai macro `println!` dan `vec!` di sepanjang buku ini. Semua macros 
ini melebar (_expand_) buat menghasilkan lebih banyak kode ketimbang 
kode yang secara manual kita tulis.

_Metaprogramming_ itu sangat berguna buat ngurangin seberapa banyak kode 
yang harus kita tulis dan pelihara, yang mana juga merupakan salah satu dari 
peran fungsi biasa. Namun, macros punya beberapa kekuatan tambahan yang 
tidak dipunyai sama fungsi biasa.

Sebuah _signature_ fungsi wajib mendeklarasikan jumlah dan tipe dari 
parameter-parameter yang dipunyai fungsi tersebut. Macros, di sisi lain, 
bisa menerima jumlah parameter yang bervariasi (variable number of parameters): 
kita bisa memanggil `println!("hello")` dengan satu argumen atau 
`println!("hello {}", name)` dengan dua argumen. Selain itu, macros itu 
dijabarkan (expanded) sebelum _compiler_ menginterpretasi (menafsirkan) 
makna dari kode tersebut, jadi sebuah macro bisa, contohnya, mengimplementasikan 
sebuah trait pada suatu tipe tertentu. Sebuah fungsi tidak bisa ngelakuin ini, 
karena dia dipanggil pas _runtime_ dan sebuah trait wajib diimplementasikan pas 
_compile time_.

Kelemahan dari mengimplementasikan sebuah macro ketimbang sebuah fungsi adalah 
kalau definisi macro itu lebih kompleks daripada definisi fungsi karena kita lagi 
nulis kode Rust yang bertugas buat nulis kode Rust. Gara-gara ketidaklangsungan 
(indirection) ini, definisi macro itu umumnya lebih susah buat dibaca, dipahami, 
dan dipelihara ketimbang definisi fungsi.

Perbedaan penting lainnya antara macros dan fungsi adalah kita wajib mendefinisikan 
macros atau membawa mereka ke dalam _scope_ (ruang lingkup) _sebelum_ kita manggil 
mereka di dalam sebuah file, berlawanan dengan fungsi biasa yang bisa kita definisikan 
di mana aja dan panggil dari mana aja.

### Macros Declarative dengan `macro_rules!` buat Metaprogramming Umum

Bentuk macros yang paling sering dipakai di Rust adalah _declarative macro_ 
(makro deklaratif). Ini kadang-kadang juga disebut sebagai “macros by example” 
(makro lewat contoh), “macros `macro_rules!`”, atau sekadar “macros” doang. Pada 
intinya, macros deklaratif membiarkan kita nulis sesuatu yang mirip sama ekspresi 
`match` di Rust. Seperti yang udah dibahas di Bab 6, ekspresi `match` adalah 
struktur kontrol yang mengambil sebuah ekspresi, ngebandingin nilai hasil dari ekspresi 
tersebut dengan serangkaian _patterns_ (pola), lalu menjalankan kode yang 
berasosiasi sama _pattern_ yang cocok tersebut. Macros juga membandingkan 
sebuah nilai terhadap _patterns_ yang diasosiasikan dengan kode tertentu: di situasi 
ini, nilainya itu adalah kode sumber (source code) Rust literal yang dioper ke 
dalam macro tersebut; lalu _patterns_ tersebut dibandingkan dengan struktur 
dari kode sumber tadi; dan kode yang terkait dengan setiap _pattern_ itu, pas dia cocok, 
bakal menggantikan kode yang dioper ke dalam macro tersebut. Ini semua terjadi pas 
masa kompilasi.

Buat mendefinisikan sebuah macro, kita memakai konstruk `macro_rules!`. 
Mari kita telusuri gimana cara memakai `macro_rules!` dengan melihat gimana si 
macro `vec!` itu didefinisikan. Bab 8 mencakup gimana kita bisa memakai macro 
`vec!` buat ngebikin sebuah vector baru dengan nilai-nilai tertentu. Misalnya, 
macro berikut ini ngebikin sebuah vector baru yang berisi tiga buah integer:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Kita juga bisa memakai macro `vec!` buat ngebikin sebuah vector yang berisi dua 
integer atau sebuah vector yang berisi lima _string slices_. Kita tidak bakal bisa 
memakai fungsi biasa buat ngelakuin hal yang sama karena kita tidak bakal tahu 
jumlah atau tipe nilai-nilainya secara pasti dari awal (up front).

Listing 20-35 nunjukin definisi yang sedikit disederhanakan dari macro `vec!`.

<Listing number="20-35" file-name="src/lib.rs" caption="Sebuah versi yang disederhanakan dari definisi macro `vec!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Catatan: Definisi asli dari macro `vec!` yang ada di _standard library_ 
> juga mengandung kode buat mengalokasikan (pre-allocate) jumlah memori yang 
> tepat dari awal. Kode tersebut adalah sebuah optimasi (optimization) yang tidak 
> kita sertakan di sini supaya contohnya lebih simpel.

Anotasi `#[macro_export]` mengindikasikan kalau macro ini seharusnya dibikin 
tersedia di mana pun _crate_ di mana macro ini didefinisikan dibawa ke dalam 
_scope_. Tanpa anotasi ini, si macro tersebut tidak bisa dibawa masuk ke dalam 
_scope_.

Terus kita memulai definisi macro-nya dengan `macro_rules!` dan nama dari macro 
yang lagi kita definisikan ini _tanpa_ pake tanda seru. Nama itu, di kasus ini 
yakni `vec`, lalu diikuti dengan kurung kurawal yang menandakan isi (body) dari 
definisi macro tersebut.

Struktur yang ada di dalam isi dari `vec!` ini mirip sekali sama struktur dari 
ekspresi `match`. Di sini kita punya satu arm (lengan) dengan _pattern_ 
`( $( $x:expr ),* )`, lalu diikuti dengan `=>` dan blok kode yang terkait sama 
_pattern_ ini. Kalau _pattern_ ini cocok, blok kode yang terkait itu bakal dipancarkan 
(emitted). Mengingat bahwa ini adalah satu-satunya _pattern_ yang ada di dalam 
macro ini, berarti cuma ada satu cara valid buat mencocokkan nilainya; _pattern_ 
lain apa pun bakal nyebabin error. Macros yang lebih kompleks bakal punya lebih dari 
satu arm.

Sintaks _pattern_ yang valid di dalam definisi macro itu berbeda dengan sintaks 
_pattern_ yang udah kita bahas di Bab 19 karena _patterns_ pada macro itu dicocokkan 
terhadap struktur dari kode Rust ketimbang terhadap nilai. Mari kita telusuri 
apa arti dari potongan-potongan _pattern_ yang ada di Listing 20-29; buat ngelihat 
sintaks _pattern_ macro yang seutuhnya, silakan lihat [Rust Reference][ref].

Pertama-tama kita memakai sepasang tanda kurung biasa (parentheses) buat ngebungkus 
keseluruhan _pattern_ tersebut. Kita memakai tanda dolar (`$`) buat mendeklarasikan 
sebuah variabel di dalam sistem macro tersebut yang bakal menampung kode Rust 
yang cocok dengan _pattern_-nya. Tanda dolar ini ngejelasin (makes it clear) kalau 
ini adalah sebuah variabel macro bukannya variabel Rust biasa. Berikutnya ada 
sepasang tanda kurung lagi yang menangkap nilai-nilai yang cocok sama _pattern_ 
yang ada di dalam tanda kurung tersebut buat dipakai di dalam kode penggantinya. 
Di dalam `$()` ada `$x:expr`, yang mana bakal cocok sama sembarang ekspresi Rust 
dan lalu ngasih nama `$x` ke ekspresi tersebut.

Koma yang ada di belakang `$()` mengindikasikan kalau sebuah karakter pemisah (separator) 
berupa koma secara literal itu wajib muncul di antara setiap instance kode yang cocok 
sama kode yang ada di dalam `$()`. Tanda bintang `*` menentukan kalau _pattern_ 
tersebut cocok nol atau sekian kali (zero or more) dari apa pun yang ngeduluin 
(precedes) tanda `*` tersebut.

Saat kita memanggil macro ini pakai `vec![1, 2, 3];`, _pattern_ `$x` bakal cocok 
sebanyak tiga kali dengan tiga ekspresi `1`, `2`, dan `3`.

Sekarang mari kita lihat pada pola (pattern) yang ada di dalam isi blok kode yang 
terkait sama arm ini: `temp_vec.push()` yang ada di dalam `$()*` bakal dihasilkan 
buat setiap bagian yang cocok sama `$()` di dalam _pattern_ sebelumnya sebanyak 
nol atau lebih kali (tergantung dari seberapa banyak _pattern_ itu cocok). Si `$x` 
bakal diganti dengan masing-masing ekspresi yang cocok tadi. Saat kita manggil 
macro ini pakai `vec![1, 2, 3];`, kode yang dihasilkan yang bakal menggantikan 
pemanggilan macro ini bakal jadi kayak gini:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Kita udah mendefinisikan sebuah macro yang bisa menerima jumlah argumen sebanyak 
apa pun yang bertipe apa pun dan bisa menghasilkan kode buat ngebikin sebuah vector 
yang berisi elemen-elemen yang udah kita tentukan.

Buat belajar lebih jauh soal gimana cara nulis macros, silakan konsultasi ke 
dokumentasi online atau sumber referensi lainnya, kayak misalnya [“The Little 
Book of Rust Macros”][tlborm] yang dimulai sama Daniel Keep dan terus dilanjutin 
sama Lukas Wirth.

### Macros Procedural Buat Menghasilkan Kode dari Atribut

Bentuk kedua dari macros adalah _procedural macro_ (makro prosedural), yang mana 
bekerjanya lebih mirip kayak sebuah fungsi (dan emang merupakan sebuah tipe prosedur). 
_Procedural macros_ nerima beberapa kode sebagai input, lalu beroperasi pada kode 
tersebut, dan akhirnya memproduksi (menghasilkan) beberapa kode sebagai output, 
bukannya mencocokkan _patterns_ lalu nggantiin kode tersebut dengan kode lain 
kayak yang dilakuin sama macros deklaratif. Tiga macam macros prosedural ini adalah 
`derive` kustom, mirip atribut (attribute-like), dan mirip fungsi (function-like), 
dan mereka semua bekerja pakai cara yang mirip-mirip.

Pas lagi membikin macros prosedural, definisi-definisi ini wajib berada di dalam 
_crate_ mereka sendiri (tersendiri) dengan sebuah tipe _crate_ spesial. Ini terjadi 
karena alasan-alasan teknis yang rumit yang mana kita harap bisa kita hilangkan di 
masa depan. Di Listing 20-36, kita nunjukin gimana cara mendefinisikan sebuah 
macro prosedural, di mana `some_attribute` itu adalah _placeholder_ buat memakai 
salah satu variasi spesifik dari macro tersebut.

<Listing number="20-36" file-name="src/lib.rs" caption="Sebuah contoh definisi macro prosedural">

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Fungsi yang mendefinisikan sebuah macro prosedural menerima sebuah `TokenStream` 
sebagai input dan memproduksi sebuah `TokenStream` sebagai output. Tipe 
`TokenStream` ini didefinisikan sama _crate_ `proc_macro` yang mana emang disertakan 
bareng Rust dan merepresentasikan sekumpulan dari _tokens_. Inilah inti (core) dari 
macro tersebut: kode sumber (source code) yang lagi dioperasikan sama si macro itu 
ngebentuk input `TokenStream` tersebut, dan kode yang dihasilkan sama si macro 
itu adalah output `TokenStream` hasilnya. Fungsi ini juga punya atribut yang 
nempel ke dirinya yang menentukan jenis macro prosedural apa yang lagi kita bikin. 
Kita bisa punya berbagai jenis macros prosedural di dalam satu _crate_ yang sama.

Mari kita ngelihat jenis-jenis dari macros prosedural tersebut. Kita bakal mulai 
dengan macro `derive` kustom dan terus ngejelasin sedikit perbedaan kecil (small 
dissimilarities) yang ngebikin bentuk-bentuk lainnya jadi beda.

### Gimana Cara Menulis Sebuah Macro `derive` Kustom

Mari kita bikin sebuah _crate_ bernama `hello_macro` yang mendefinisikan 
sebuah trait bernama `HelloMacro` dengan satu fungsi _associated_ bernama `hello_macro`. 
Ketimbang harus maksa supaya *user* kita mengimplementasikan trait `HelloMacro` 
secara manual buat setiap tipe mereka, kita bakal nyediain sebuah macro prosedural 
sehingga para pengguna bisa menganotasi tipe mereka dengan `#[derive(HelloMacro)]` 
buat dapetin implementasi default (bawaan) dari fungsi `hello_macro` ini. Implementasi 
default ini bakal mencetak `Hello, Macro! My name is TypeName!` di mana `TypeName` 
itu adalah nama dari tipe di mana trait ini baru aja didefinisikan. Dengan 
kata lain, kita bakal nulis sebuah _crate_ yang memungkinkan programmer lain 
buat nulis kode kayak di Listing 20-37 dengan memakai _crate_ kita.

<Listing number="20-37" file-name="src/main.rs" caption="Kode yang bisa ditulis sama user _crate_ kita pas lagi memakai macro prosedural kita">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Kode ini bakal mencetak `Hello, Macro! My name is Pancakes!` pas kita udah selesai. 
Langkah pertamanya adalah membikin _library crate_ baru, kayak gini:

```console
$ cargo new hello_macro --lib
```

Berikutnya, di Listing 20-38, kita bakal mendefinisikan trait `HelloMacro` dan 
fungsi _associated_-nya.

<Listing file-name="src/lib.rs" number="20-38" caption="Sebuah trait sederhana yang bakal kita pakai bareng macro `derive`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Kita udah punya trait dan fungsinya. Pada titik ini, _user_ dari _crate_ kita udah 
bisa mengimplementasikan trait ini buat dapetin fungsionalitas yang mereka inginkan, 
kayak di Listing 20-39.

<Listing number="20-39" file-name="src/main.rs" caption="Gimana kelihatannya kalau users menulis implementasi manual dari trait `HelloMacro`">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

Namun, mereka masih butuh buat nulis blok implementasi ini buat setiap tipe yang 
pengen mereka pakai dengan `hello_macro`; kita mau nyelametin mereka dari keharusan 
(having to do) ngelakuin kerjaan ini.

Selain itu, kita belum bisa nyediain fungsi `hello_macro` dengan implementasi 
default yang mana bakal bisa nyetak nama dari tipe di mana trait itu diimplementasikan: 
Rust tidak punya kapabilitas _reflection_ (kemampuan buat meneliti tipe-tipe saat jalan), 
jadi dia tidak bisa nyari tahu nama dari sebuah tipe pas _runtime_ dateng. Kita 
butuh macro buat bisa menghasilkan (generate) kode tersebut saat _compile time_.

Langkah selanjutnya adalah mendefinisikan macro prosedural tersebut. Saat tulisan ini 
dibikin, macros prosedural wajib berada di dalam _crate_ mereka sendiri (terpisah). 
Suatu hari nanti, pembatasan ini mungkin bakal diangkat (lifted). Konvensi buat 
menata struktur (structuring) _crates_ dan _crates_ macro itu kayak gini: buat sebuah 
_crate_ yang namanya `foo`, _crate_ macro prosedural `derive` kustomnya itu dikasih nama 
`foo_derive`. Mari kita mulai _crate_ baru bernama `hello_macro_derive` di dalam 
_project_ `hello_macro` kita:

```console
$ cargo new hello_macro_derive --lib
```

Dua _crates_ kita ini itu saling berkaitan erat (tightly related), jadi kita ngebikin 
_crate_ macro prosedural ini di dalam *directory* dari _crate_ `hello_macro` kita. Kalau 
kita ngubah definisi trait yang ada di dalam `hello_macro`, kita juga bakal harus 
mengubah implementasi macro prosedural di dalam `hello_macro_derive`. Kedua _crates_ 
ini harus dipublikasikan (_published_) secara terpisah, dan programmer-programmer yang 
memakai _crates_ ini harus menambahkan dua-duanya sebagai dependensi lalu membawa 
keduanya masuk ke dalam _scope_. Sebagai gantinya, kita juga bisa sih bikin supaya 
_crate_ `hello_macro` itu memakai `hello_macro_derive` sebagai dependensi terus dia yang 
mengekspor ulang (_re-export_) kode macro prosedural tersebut. Namun, cara kita menata 
struktur _project_ kita ini membiarkan programmer-programmer buat memakai `hello_macro` 
bahkan kalau mereka sebenernya tidak butuh sama fungsionalitas `derive`-nya.

Kita perlu mendeklarasikan _crate_ `hello_macro_derive` ini sebagai sebuah _crate_ 
macro prosedural. Kita juga bakal butuh fungsionalitas dari _crates_ `syn` dan 
`quote`, kayak yang bakal kita lihat sebentar lagi, jadi kita perlu nambahin mereka 
sebagai dependensi. Tambahkan yang berikut ini ke dalam file _Cargo.toml_ untuk 
`hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Buat mulai mendefinisikan macro prosedural ini, tempatin kode dari Listing 20-40 
ke dalam file _src/lib.rs_ kita buat _crate_ `hello_macro_derive`. Perhatikan bahwa 
kode ini belum bakal bisa di-compile sampai kita udah nambahin definisi buat fungsi 
`impl_hello_macro`.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Kode yang mana sebagian besar crates macro prosedural butuhkan buat memproses kode Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Coba perhatikan kalau kita udah membelah (split) kodenya ke dalam fungsi 
`hello_macro_derive`, yang mana bertanggung jawab buat nge-_parse_ (mengurai/menguraikan) 
si `TokenStream`, dan fungsi `impl_hello_macro`, yang mana bertanggung jawab 
buat mengubah (_transforming_) struktur pohon sintaksnya (syntax tree): ini 
ngebikin penulisan sebuah macro prosedural jadi lebih nyaman (convenient). Kode di dalam 
fungsi yang luar (`hello_macro_derive` di kasus ini) itu bakal sama aja bunyinya buat 
hampir sebagian besar dari _crates_ macro prosedural yang pernah kita lihat atau bikin. 
Kode yang kita tentuin di dalam isi fungsi dalamnya (`impl_hello_macro` di kasus ini) 
itu bakal berbeda-beda tergantung dari apa tujuan macro prosedural kita itu sebenernya.

Kita udah memperkenalkan tiga _crates_ baru di sini: `proc_macro`, [`syn`][syn], 
dan [`quote`][quote]. _Crate_ `proc_macro` itu emang udah dibawa bareng sama Rust, 
jadi kita tidak perlu nambahin dia ke dalam dependensi di _Cargo.toml_ kita. _Crate_ 
`proc_macro` ini adalah API dari _compiler_ yang memungkinkan kita buat membaca dan 
memanipulasi kode Rust dari dalam kode kita.

_Crate_ `syn` itu mem-_parse_ kode Rust yang asalnya dari string menjadi sebuah struktur 
data yang mana bisa kita operasikan lebih lanjut. _Crate_ `quote` kemudian mengubah si 
struktur data `syn` tersebut kembali (_turns back_) menjadi kode Rust biasa. 
_Crates_ ini ngebikin gampang sekali buat mem-_parse_ segala macam kode Rust apa pun yang 
mungkin pengen kita tangani (handle): nulis _parser_ yang sempurna (full parser) 
buat bahasa pemrograman Rust bukanlah tugas yang gampang lho.

Fungsi `hello_macro_derive` bakal dipanggil pas ada _user_ _library_ kita yang 
mencantumkan `#[derive(HelloMacro)]` pada sebuah tipe. Hal ini dimungkinkan 
karena kita udah ngasih anotasi ke fungsi `hello_macro_derive` di sini pakai 
`proc_macro_derive` dan nentuin nama `HelloMacro`, yang mana nama ini emang cocok 
sama nama dari trait kita; ini adalah konvensi yang paling banyak diikuti sama macros 
prosedural.

Fungsi `hello_macro_derive` pertama-tama mengkonversi `input` yang asalnya dari sebuah 
`TokenStream` menjadi sebuah struktur data yang lalu bisa kita interpretasikan 
dan kita operasikan lebih lanjut. Di sinilah si `syn` ikut campur. Fungsi `parse` 
di dalam `syn` mengambil sebuah `TokenStream` lalu mengembalikan sebuah struct 
`DeriveInput` yang merepresentasikan kode Rust yang udah selesai di-_parse_. Listing 
20-41 nunjukin bagian-bagian yang relevan dari struct `DeriveInput` yang kita dapetin 
sebagai hasil dari menge-_parse_ string `struct Pancakes;`.

<Listing number="20-41" caption="Instance `DeriveInput` yang kita dapetin pas kita mem-parse kode yang punya atribut macro tersebut di Listing 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Bidang (_fields_) dari struct ini nunjukin kalau kode Rust yang udah kita _parse_ ini 
adalah sebuah _unit struct_ dengan *ident* (_identifier_/pengidentifikasi, yang artinya 
nama) `Pancakes`. Ada lebih banyak bidang lagi pada struct ini buat 
mendeskripsikan segala macam variasi kode Rust; silakan cek dokumentasi [`syn` 
untuk `DeriveInput`][syn-docs] buat informasi lebih lengkap.

Bentar lagi kita bakal mendefinisikan fungsi `impl_hello_macro`, yang mana ini 
bakal jadi tempat di mana kita ngebangun kode Rust baru yang pengen kita 
masukkan (include) ke dalem programnya. Tapi sebelum kita melakukan itu, perhatikan 
kalau output dari macro `derive` kita ini juga merupakan sebuah `TokenStream`. Si 
`TokenStream` kembalian (_returned_) ini bakal ditambahin ke kode yang dibikin sama _user_ 
_crate_ kita, jadi pas mereka mengompilasi (compile) _crate_ mereka, mereka bakal 
dapetin fungsi tambahan yang udah kita sediain di dalam `TokenStream` yang udah 
dimodifikasi tadi.

Kita mungkin sempat merhatiin kalau kita tadi memanggil `unwrap` yang mana bakal 
ngebikin fungsi `hello_macro_derive` jadi _panic_ kalau pemanggilan ke fungsi 
`syn::parse` tersebut ternyata gagal (fails) di sini. Sangat diwajibkan buat macro 
prosedural kita supaya jadi _panic_ pas ada error karena fungsi-fungsi `proc_macro_derive` 
itu wajib ngembaliin tipe `TokenStream` ketimbang `Result` supaya dia bisa patuh (conform) 
sama API macro prosedural tersebut. Kita udah menyederhanakan contoh ini dengan 
memakai `unwrap`; kalau di dalem kode produksi sungguhan (_production code_), kita 
harusnya menyediakan pesan error yang jauh lebih spesifik soal apa yang 
sebenernya salah dengan memakai `panic!` atau `expect`.

Nah, sekarang karena kita udah punya kode buat ngubah kode Rust yang udah dianotasikan 
(annotated Rust code) dari sebuah `TokenStream` menjadi sebuah instance `DeriveInput`, 
mari kita hasilkan kode (_generate the code_) yang mengimplementasikan trait 
`HelloMacro` pada tipe yang dianotasi tersebut, kayak yang ditunjukin di Listing 20-42.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Mengimplementasikan trait `HelloMacro` memakai kode Rust yang udah diparse">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Kita mendapatkan sebuah instance dari struct `Ident` yang mengandung nama (identifier) dari 
tipe yang dianotasi tadi memakai `ast.ident`. Struct yang ada di Listing 20-41 tadi 
nunjukin kalau pas kita ngejalanin fungsi `impl_hello_macro` pada kode yang ada di 
Listing 20-37, si `ident` yang bakal kita dapet ini bakal punya *field* `ident` 
dengan nilai `"Pancakes"`. Oleh karenanya variabel `name` di dalam Listing 20-42 
bakal berisi instance struct `Ident` yang mana, pas dicetak, dia bakal ngasih 
string `"Pancakes"`, nama dari struct di dalam Listing 20-37.

Macro `quote!` membiarkan kita mendefinisikan kode Rust yang mau kita kembalikan (return). 
_Compiler_ mengharapkan sesuatu yang agak beda dengan hasil eksekusi langsung dari 
macro `quote!`, jadi kita perlu buat ngubahnya (_convert it_) menjadi sebuah 
`TokenStream`. Kita melakukan hal ini dengan cara memanggil method `into`, yang mana 
memakan (consumes) si representasi menengah (intermediate representation) ini lalu 
mengembalikan sebuah nilai dengan tipe `TokenStream` yang diwajibkan tersebut.

Macro `quote!` ini juga nyediain mekanisme templat (templating) yang keren sekali lho: kita 
bisa memasukkan `#name`, dan `quote!` bakal nggantiin itu dengan nilai yang ada di 
dalam variabel `name`. Kita bahkan bisa ngelakuin pengulangan yang mirip sama cara 
kerja macros yang biasa. Silakan cek dokumentasi [crate `quote`][quote-docs] 
buat perkenalan yang komprehensif (thorough).

Kita pengen macro prosedural kita ini buat menghasilkan sebuah implementasi dari 
trait `HelloMacro` kita buat tipe yang dianotasi sama si *user*, yang mana nama tipenya 
itu bisa kita dapetin dengan memakai `#name`. Implementasi trait-nya ini punya 
satu fungsi doang yaitu `hello_macro`, di mana isinya (body) mengandung 
fungsionalitas yang pengen kita berikan: yaitu mencetak tulisan `Hello, Macro! 
My name is` lalu diikuti dengan nama dari tipe yang dianotasi tersebut.

Macro `stringify!` yang dipakai di sini emang udah tertanam (built into) di dalam Rust. 
Dia mengambil (takes) sebuah ekspresi Rust, kayak misalnya `1 + 2`, lalu pada 
saat _compile time_ dia bakal ngerubah ekspresi tersebut jadi string literal (string harfiah), 
kayak misalnya `"1 + 2"`. Ini beda sama `format!` atau `println!`, dua macro ini kan 
mengevaluasi ekspresinya dan baru kemudian ngubah hasilnya jadi sebuah `String`. 
Ada kemungkinan input `#name` ini bisa jadi adalah sebuah ekspresi yang mana harus 
dicetak apa adanya secara harfiah (literally), jadi makanya kita memakai `stringify!`. 
Memakai `stringify!` juga ngirit alokasi (saves an allocation) karena dia ngubah 
`#name` jadi string literal saat _compile time_ (saat kompilasi).

Pada titik ini, jalanin `cargo build` seharusnya udah bisa kelar tanpa masalah di 
dalam `hello_macro` sekaligus `hello_macro_derive`. Mari kita pasangkan (_hook up_) 
_crates_ ini dengan kode yang ada di dalam Listing 20-37 tadi buat ngelihat gimana 
macro prosedural kita ini beraksi! Bikin sebuah project *binary* baru 
di dalam *directory* _projects_ kita memakai `cargo new pancakes`. Kita perlu menambahkan 
`hello_macro` dan `hello_macro_derive` sebagai dependensi (dependencies) di 
dalam _Cargo.toml_ milik _crate_ `pancakes` ini. Kalau seandainya kita mempublikasikan 
versi dari `hello_macro` dan `hello_macro_derive` punya kita ke 
[crates.io](https://crates.io/), mereka bakal jadi kayak dependensi biasa; 
kalau belum dipublikasikan, kita bisa menspesifikasikan mereka sebagai dependensi bertipe `path` 
(jalur ke folder) kayak gini:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Masukin kode dari Listing 20-37 tadi ke dalam file _src/main.rs_, terus coba jalanin `cargo run`: 
dia seharusnya mencetak `Hello, Macro! My name is Pancakes!` Implementasi dari 
trait `HelloMacro` dari macro prosedural tersebut udah dimasukkan (included) 
tanpa _crate_ `pancakes` ini harus mengimplementasikannya sendiri; si atribut 
`#[derive(HelloMacro)]` itulah yang udah nambahin implementasi trait tersebut secara 
otomatis.

Selanjutnya, mari kita telusuri di mana letak perbedaan dari jenis-jenis macro 
prosedural lain kalau dibandingin dengan macros `derive` kustom.

### Macros Mirip Atribut (Attribute-Like Macros)

Macros yang mirip atribut itu serupa sama macros `derive` kustom, tapi 
ketimbang menghasilkan (generating) kode buat atribut `derive`, macros ini ngebolehin 
kita buat membikin atribut baru (new attributes). Mereka juga lebih fleksibel: `derive` 
itu kan cuma bisa kerja buat structs dan enums doang; sedangkan atribut itu bisa aja 
diterapkan ke item-item lain, kayak fungsi misalnya. Berikut ini adalah sebuah 
contoh penggunaan macro yang mirip atribut. Katakanlah kita punya sebuah atribut 
bernama `route` yang mana nganotasi fungsi-fungsi pas lagi makek _framework_ aplikasi 
web (web application framework):

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Atribut `#[route]` ini idealnya bakal didefinisikan sama _framework_ tersebut sebagai 
sebuah macro prosedural. _Signature_ dari fungsi pendefinisi macro-nya (macro 
definition function) mungkin bakal kelihatan kayak gini:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Di sini, kita punya dua parameter bertipe `TokenStream`. Yang pertama itu adalah 
buat menampung konten dari atributnya: yaitu di bagian `GET, "/"`. Terus yang 
kedua itu adalah buat isinya (body) si item di mana atribut itu ditempelkan (attached 
to): di contoh kasus ini, buat si `fn index() {}` beserta sisa isi dari fungsi 
tersebut.

Selain perbedaan tadi, macros mirip atribut ini bekerja dengan cara yang sama persis 
dengan macros `derive` kustom: kita membikin sebuah _crate_ dengan tipe _crate_ `proc-macro` 
lalu kita mengimplementasikan fungsi yang tugasnya buat ngehasilin kode (generates the code) 
yang pengen kita bikin!

### Macros Mirip Fungsi (Function-Like Macros)

Macros mirip fungsi itu mendefinisikan macros yang kelihatannya kayak pemanggilan fungsi. 
Sama kayak macros `macro_rules!`, macros ini itu lebih luwes (fleksibel) daripada 
fungsi biasa; misalnya, mereka bisa nerima jumlah argumen yang bervariasi 
(unknown number of arguments). Namun, macros `macro_rules!` itu cuma bisa didefinisikan 
memakai sintaks yang mirip `match` (match-like syntax) yang udah kita obrolin di 
[“Macros Declarative dengan `macro_rules!` buat Metaprogramming Umum”][decl] tadi 
sebelumnya. Macros yang mirip fungsi ini mengambil satu parameter `TokenStream` dan 
kemudian di definisinya dia memanipulasi `TokenStream` tersebut memakai kode 
Rust sama persis kayak apa yang dilakuin sama kedua tipe macros prosedural 
sebelumnya. Sebuah contoh dari macro mirip fungsi adalah macro `sql!` yang mana mungkin 
bakal dipanggil kayak gini:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Macro ini bakal mem-_parse_ pernyataan (statement) SQL yang ada di dalamnya dan ngecek 
apakah secara sintaks itu (syntactically) benar atau tidak, yang mana itu merupakan pemrosesan 
yang jauh lebih ribet (_complex processing_) ketimbang apa yang sanggup dilakukan 
sama macro tipe `macro_rules!`. Definisi macro `sql!` ini kelihatannya bakal kayak gini:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Definisi ini mirip sekali kan sama _signature_ dari macro `derive` kustom tadi: kita 
menerima tokens yang ada di dalam tanda kurung tersebut (parentheses) dan 
terus kita ngembaliin (return) kode yang emang pengen kita hasilin (generate).

## Ringkasan

Fiuh! (Whew!) Nah sekarang kita punya segelintir fitur Rust baru di sabuk perkakas 
(toolbox) kita yang mana kemungkinan besar kita tidak bakal sering memakainya, 
tapi seenggaknya kita bakal tahu kalau mereka itu tersedia di situasi-situasi tertentu 
yang amat spesifik. Kita udah memperkenalkan beberapa topik yang kompleks (complex 
topics) sehingga saat kita kebetulan menjumpainya di dalam saran-saran pesan 
error (error message suggestions) atau di dalam kode orang lain, kita bakal sanggup 
buat mengenali berbagai macam konsep dan sintaks ini. Gunakan bab ini 
sebagai sebuah pedoman referensi (reference guide) buat ngebimbing kita nemuin solusinya.

Berikutnya, kita bakal mempraktikkan (put into practice) segala macam hal yang udah 
kita bicarain di sepanjang buku ini dengan cara mengerjakan satu _project_ lagi!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
