## Advanced Types (Tipe Tingkat Lanjut)

Sistem tipe Rust punya beberapa fitur yang sejauh ini cuma kita sebut aja 
tapi belum benar-benar kita bahas. Kita bakal mulai dengan membahas _newtypes_ 
secara umum sembari kita menyelidiki kenapa _newtypes_ itu berguna sebagai tipe. 
Terus kita bakal lanjut ke _type aliases_ (alias tipe), sebuah fitur yang mirip sama 
_newtypes_ tapi punya semantik yang agak beda. Kita juga bakal ngebahas 
tipe `!` dan _dynamically sized types_ (tipe-tipe yang berukuran dinamis).

### Memakai Newtype Pattern Buat Keamanan Tipe dan Abstraksi

Bagian ini berasumsi kalau kita udah ngebaca bagian sebelumnya [“Memakai 
Newtype Pattern Buat Mengimplementasikan External Traits”][using-the-newtype-pattern]. 
_Newtype pattern_ (pola tipe baru) ini juga berguna buat hal-hal di luar dari 
apa yang udah kita bahas sejauh ini, termasuk secara statis menegakkan 
aturan supaya nilai-nilai tidak pernah tertukar (confused) dan buat 
mengindikasikan satuan (units) dari sebuah nilai. Kita udah lihat contoh 
pemakaian _newtypes_ buat mengindikasikan satuan di Listing 20-16: ingat 
kembali kalau struct `Millimeters` dan `Meters` itu membungkus nilai `u32` 
di dalam sebuah _newtype_. Kalau kita nulis sebuah fungsi dengan parameter 
bertipe `Millimeters`, kita tidak bakal bisa men-compile program yang 
secara tidak sengaja mencoba memanggil fungsi tersebut dengan nilai 
bertipe `Meters` atau nilai `u32` biasa.

Kita juga bisa memakai _newtype pattern_ buat mengabstraksi beberapa detail 
implementasi dari sebuah tipe: si tipe baru tersebut bisa ngekspos API _public_ 
yang mana berbeda dari API milik tipe _private_ yang ada di dalamnya.

_Newtypes_ juga bisa menyembunyikan (hide) implementasi internal. Misalnya, kita 
bisa aja menyediakan tipe `People` buat ngebungkus sebuah `HashMap<i32, String>` 
yang nyimpan ID seseorang yang diasosiasikan dengan nama mereka. Kode yang 
memakai `People` cuma bakal berinteraksi sama API _public_ yang kita sediakan, 
kayak method buat nambahin string nama ke dalam koleksi `People`; kode tersebut 
tidak perlu tahu kalau kita secara internal menaruh nilai ID `i32` ke nama-nama 
tersebut. _Newtype pattern_ adalah cara yang ringan (lightweight) buat mendapatkan 
enkapsulasi (encapsulation) guna menyembunyikan detail implementasi, yang mana 
udah kita bahas di [“Encapsulation (Enkapsulasi) yang Menyembunyikan Detail 
Implementasi”][encapsulation-that-hides-implementation-details] di Bab 18.

### Membikin Sinonim Tipe dengan Type Aliases

Rust menyediakan kemampuan buat mendeklarasikan _type alias_ (alias tipe) buat 
ngasih nama lain ke tipe yang udah ada. Buat hal ini kita memakai keyword 
`type`. Misalnya, kita bisa membikin alias `Kilometers` buat `i32` kayak gini:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Sekarang si alias `Kilometers` adalah sebuah _sinonim_ buat `i32`; beda sama tipe 
`Millimeters` dan `Meters` yang kita bikin di Listing 20-16, `Kilometers` 
bukanlah sebuah tipe baru yang terpisah. Nilai-nilai yang punya tipe `Kilometers` 
bakal diperlakukan persis sama kayak nilai-nilai bertipe `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Karena `Kilometers` dan `i32` adalah tipe yang sama, kita bisa menjumlahkan nilai 
dari kedua tipe tersebut dan kita bisa ngoper nilai `Kilometers` ke fungsi-fungsi 
yang menerima parameter `i32`. Namun, dengan memakai cara ini, kita tidak 
dapetin keuntungan pengecekan tipe (type-checking benefits) yang kita dapat 
dari pemakaian _newtype pattern_ yang dibahas sebelumnya. Dengan kata lain, kalau kita 
nyampur aduk (mix up) nilai `Kilometers` dan `i32` di suatu tempat, _compiler_ 
tidak bakal ngasih kita error.

Kegunaan utama (_main use case_) dari sinonim tipe adalah buat ngurangin 
pengulangan (repetition). Misalnya, kita mungkin punya tipe yang panjang sekali 
kayak gini:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Nulisin tipe sepanjang ini di _signatures_ fungsi dan sebagai anotasi tipe di 
semua tempat di kode kita bisa jadi melelahkan dan rentan kena error (error 
prone). Bayangin aja kalau punya sebuah project yang penuh dengan kode kayak 
yang ada di Listing 20-25.

<Listing number="20-25" caption="Memakai tipe yang panjang di banyak tempat">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Sebuah _type alias_ ngebikin kode ini jadi lebih gampang dikelola dengan cara 
ngurangin pengulangan tersebut. Di Listing 20-26, kita memperkenalkan sebuah 
alias bernama `Thunk` buat tipe yang panjang (_verbose_) tadi dan bisa mengganti 
semua penggunaan dari tipe tersebut dengan si alias `Thunk` yang lebih pendek.

<Listing number="20-26" caption="Memperkenalkan sebuah _type alias_, `Thunk`, buat ngurangin pengulangan">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Kode ini jadinya jauh lebih gampang buat dibaca dan ditulis! Milih nama yang punya 
makna (_meaningful_) buat _type alias_ juga bisa ngebantu ngomunikasiin niat (intent) 
kita (_thunk_ adalah kata yang artinya kode yang bakal dievaluasi nanti, jadi ini 
adalah nama yang tepat buat sebuah _closure_ yang lagi disimpen).

_Type aliases_ juga umumnya dipakai bareng sama tipe `Result<T, E>` buat ngurangin 
pengulangan. Coba perhatikan modul `std::io` di _standard library_. Operasi 
I/O (input/output) itu sering sekali mengembalikan `Result<T, E>` buat menangani 
situasi-situasi pas operasinya gagal jalan. Library ini punya struct 
`std::io::Error` yang merepresentasikan semua kemungkinan error I/O. Banyak 
dari fungsi-fungsi di `std::io` bakal mengembalikan `Result<T, E>` di mana si 
`E` itu adalah `std::io::Error`, kayak misalnya fungsi-fungsi di dalam trait 
`Write` ini:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

Bagian `Result<..., Error>` itu diulang berkali-kali. Karena hal itu, `std::io` 
punya deklarasi _type alias_ berikut ini:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Karena deklarasi ini ada di dalam modul `std::io`, kita bisa memakai alias 
_fully qualified_ `std::io::Result<T>`; yakni, sebuah `Result<T, E>` dengan si 
`E` udah diisi sebagai `std::io::Error`. Alhasil, _signatures_ dari fungsi-fungsi di 
trait `Write` kelihatannya jadi kayak gini deh:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

_Type alias_ ini sangat ngebantu dalam dua hal: dia ngebikin kodenya jadi 
lebih gampang ditulis _dan_ dia ngasih kita sebuah antarmuka (_interface_) yang 
konsisten di seluruh `std::io`. Karena dia cuma sebuah alias, dia sebenernya 
cuma `Result<T, E>` biasa aja, yang berarti kita bisa memakai method apa pun yang 
berlaku buat `Result<T, E>` dengannya, sekaligus juga sintaks-sintaks spesial 
kayak operator `?`.

### Tipe Never (Tak Pernah) yang Tidak Pernah Mengembalikan Apa-apa

Rust punya tipe spesial bernama `!` yang mana dikenal di bahasa gaulnya 
teori tipe sebagai _empty type_ (tipe kosong) karena dia tidak punya nilai 
sama sekali. Kita lebih milih menyebutnya _never type_ (tipe tak pernah) 
karena dia berdiri menempati posisi dari tipe kembalian saat sebuah fungsi tidak 
bakal pernah mengembalikan (never return) apa-apa. Ini adalah contohnya:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Kode ini dibaca sebagai “fungsi `bar` mengembalikan never.” Fungsi-fungsi yang 
mengembalikan `never` disebut _diverging functions_ (fungsi divergen). Kita 
tidak bisa membikin nilai dari tipe `!`, jadi si `bar` itu emang tidak mungkin bisa 
mengembalikan apa-apa.

Tapi apa gunanya coba sebuah tipe yang kita tidak bisa bikin nilai buatnya sama sekali? 
Ingat kembali kode dari Listing 2-5, bagian dari game tebak angka; kita udah 
menaruh sedikit dari bagian kode itu di sini di Listing 20-27.

<Listing number="20-27" caption="Sebuah `match` dengan _arm_ yang berakhir dengan `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

Waktu itu, kita mengabaikan (skipped over) beberapa detail di kode ini. Di 
[“Konstruk Control Flow `match`”][the-match-control-flow-construct] di Bab 6, kita ngebahas 
kalau *match arms* itu semuanya wajib mengembalikan tipe yang sama. Jadi, misalnya, 
kode berikut ini tidak bakal jalan:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Tipe dari `guess` di kode ini wajib jadi integer _sekaligus_ string, padahal 
Rust mewajibkan `guess` buat cuma punya satu tipe aja. Terus si `continue` itu 
ngembaliin apa dong? Gimana ceritanya kita dibolehin buat mengembalikan nilai 
`u32` dari satu arm padahal punya arm lain yang berakhir dengan `continue` di 
Listing 20-27?

Seperti yang mungkin udah kita tebak, `continue` itu punya nilai `!`. Yaitu, saat 
Rust menghitung tipe dari `guess`, dia ngelihat ke kedua _match arms_ tersebut, 
yang pertama bernilai `u32` dan yang terakhir (latter) bernilai `!`. Karena `!` 
itu tidak bakal pernah bisa punya nilai, Rust memutuskan kalau tipe dari `guess` 
adalah `u32`.

Cara formal buat mendeskripsikan perilaku ini adalah bahwa ekspresi-ekspresi dari 
tipe `!` itu bisa dipaksa (_coerced_) menjadi tipe apa aja yang lain. Kita 
dibolehin buat mengakhiri _match arm_ ini dengan `continue` karena `continue` 
tidak mengembalikan nilai; sebagai gantinya, dia memindahkan kontrol kembali ke 
atas perulangannya (loop), jadi di kasus `Err`, kita tidak pernah memberikan 
sebuah nilai ke `guess`.

Tipe _never_ ini berguna bareng macro `panic!` juga. Ingat kembali fungsi 
`unwrap` yang kita panggil pada nilai-nilai `Option<T>` buat menghasilkan nilai atau 
jadi _panic_ dengan definisi seperti ini:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Di kode ini, hal yang sama juga terjadi kayak yang ada di `match` di Listing 
20-27: Rust ngelihat kalau `val` punya tipe `T` dan `panic!` punya tipe `!`, jadi 
hasil dari keseluruhan ekspresi `match` tersebut adalah `T`. Kode ini bisa jalan 
karena `panic!` tidak memproduksi sebuah nilai; dia sekadar memberhentikan programnya. 
Di kasus `None`, kita tidak bakal mengembalikan nilai dari `unwrap`, jadi kode ini 
itu valid.

Satu ekspresi terakhir yang punya tipe `!` adalah sebuah `loop`:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Di sini, _loop_ tersebut tidak pernah berakhir, jadi nilai dari ekspresinya 
adalah `!`. Namun, ini tidak bakal benar kalau seandainya kita memasukkan 
`break`, karena perulangan tersebut bakal dihentikan pas dia mencapai `break`.

### Tipe yang Berukuran Dinamis (Dynamically Sized Types) dan Trait `Sized`

Rust perlu tahu detail-detail tertentu tentang tipe-tipenya, seperti seberapa 
banyak ruang yang harus dialokasikan buat menyimpan sebuah nilai dari suatu tipe 
tertentu. Hal ini ngebikin satu sudut dari sistem tipenya jadi agak membingungkan 
pada awalnya: yakni konsep tentang _dynamically sized types_ (tipe-tipe yang 
berukuran dinamis). Terkadang disebut juga sebagai _DSTs_ atau _unsized types_ (tipe 
tanpa ukuran tetap), tipe-tipe ini membiarkan kita nulis kode yang memakai 
nilai-nilai yang mana ukurannya cuma bisa kita ketahui saat _runtime_.

Mari kita gali detail-detail dari sebuah tipe berukuran dinamis yang bernama 
`str`, yang mana udah sering kita pakai di sepanjang buku ini. Yap benar, bukan 
`&str`, melainkan si `str` itu sendiri sendirian, dia itu adalah sebuah DST. Di 
banyak kasus, kayak misalnya pas lagi nyimpan teks yang dimasukkan (entered) oleh 
_user_, kita tidak bisa tahu seberapa panjang string-nya tersebut sampai _runtime_ 
datang. Itu artinya kita tidak bisa membikin variabel dengan tipe `str`, dan 
kita juga tidak bisa memakai argumen bertipe `str`. Coba perhatikan kode berikut, 
yang mana tidak bisa jalan:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust butuh tahu seberapa besar memori yang harus dialokasikan buat nilai apa pun 
dari suatu tipe tertentu, dan semua nilai dari sebuah tipe itu diwajibkan buat 
memakai jumlah ruang memori yang sama. Kalau seandainya Rust ngebolehin kita buat 
nulis kode ini, kedua nilai `str` ini pasti dituntut buat menempati jumlah ruang 
yang sama besarnya. Tapi kenyataannya panjang mereka itu berbeda: `s1` butuh 
penyimpanan memori 12 *bytes* dan `s2` butuh 15 *bytes*. Inilah alasan kenapa mustahil 
buat ngebikin variabel yang menampung sebuah tipe berukuran dinamis secara langsung.

Terus apa yang harus kita lakuin? Di kasus ini, kita sebenarnya udah tahu jawabannya: 
kita harus membikin tipe dari `s1` dan `s2` menjadi `&str` ketimbang `str`. 
Ingat kembali dari [“String Slices”][string-slices] di Bab 4 kalau struktur data _slice_ 
itu cuma sekadar menyimpan posisi awal (_starting position_) dan panjang (_length_) 
dari _slice_ tersebut. Jadi, meskipun `&T` itu merupakan sebuah nilai tunggal 
yang menyimpan alamat memori di mana si `T` tersebut berada, sebuah `&str` itu 
terdiri dari *dua* nilai: alamat dari si `str` dan juga panjangnya. Alhasil, kita bisa 
tahu pasti ukuran dari nilai sebuah `&str` saat _compile time_: ukurannya adalah dua 
kali panjang dari sebuah `usize`. Yaitu, kita selalu tahu ukuran dari sebuah `&str`, 
tidak peduli sepanjang apa pun string yang ia tunjuk tersebut. Secara umum, beginilah 
cara gimana tipe berukuran dinamis itu dipakai di Rust: mereka punya ekstra sedikit 
metadata yang menyimpan besaran ukuran dari informasi yang dinamis tersebut. Aturan 
emas (_golden rule_) dari tipe yang berukuran dinamis adalah kita harus selalu menaruh 
nilai-nilai dari tipe berukuran dinamis tersebut di balik (_behind_) semacam _pointer_.

Kita bisa menggabungkan `str` dengan berbagai macam pointer lainnya: misalnya, 
`Box<str>` atau `Rc<str>`. Faktanya, kita udah ngelihat hal ini sebelumnya tapi dengan 
tipe berukuran dinamis yang berbeda: yakni, traits. Setiap trait itu adalah sebuah 
tipe berukuran dinamis yang bisa kita rujuk (refer to) dengan memakai nama dari 
trait tersebut. Di [“Memakai Trait Objects Buat Mengabstraksi Perilaku Bersama”][using-trait-objects-to-abstract-over-shared-behavior] 
di Bab 18, kita nyebutin kalau buat memakai traits sebagai _trait objects_, kita wajib 
menaruh mereka di balik sebuah _pointer_, seperti `&dyn Trait` atau `Box<dyn Trait>` 
(`Rc<dyn Trait>` juga bisa jalan kok).

Buat bekerja sama DSTs, Rust menyediakan trait `Sized` buat menentukan apakah 
ukuran dari suatu tipe itu bisa diketahui saat _compile time_ atau tidak. Trait ini 
secara otomatis diimplementasikan buat semua hal yang ukurannya bisa diketahui 
saat _compile time_. Selain itu, Rust juga secara implisit menambahkan batasan (bound) 
pada `Sized` ke semua fungsi generik (generic function). Yakni, definisi fungsi 
generik kayak gini:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

itu sebenernya bakal diperlakukan seolah-olah kita udah nulis kayak gini:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Secara bawaan (_by default_), fungsi-fungsi generik cuma bakal bekerja buat tipe-tipe 
yang ukurannya itu diketahui pas _compile time_. Namun, kita bisa memakai sintaks 
spesial berikut ini buat mengendurkan (_relax_) pembatasan tersebut:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Batasan _trait bound_ pada `?Sized` itu artinya “`T` mungkin `Sized` atau mungkin juga 
tidak `Sized`” dan notasi ini menimpa (overrides) sifat bawaan yang mewajibkan 
tipe generik buat harus punya ukuran yang udah diketahui pas _compile time_. 
Sintaks `?Trait` yang punya arti (meaning) kayak gini cuma tersedia buat trait 
`Sized` doang ya, tidak bisa dipakai buat traits yang lain.

Perhatikan juga kalau kita juga mengubah tipe dari parameter `t` dari asalnya `T` 
menjadi `&T`. Karena tipe tersebut bisa aja tidak `Sized`, kita wajib memakai dia di 
balik semacam _pointer_. Di kasus ini, kita milih buat memakai _reference_ (referensi).

Berikutnya, kita bakal membahas tentang fungsi dan _closures_!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[using-the-newtype-pattern]: ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits
