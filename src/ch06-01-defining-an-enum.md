## Mendefinisikan sebuah Enum

Kalau struct ngasih kita cara buat ngelempokin field sama data yang terkait 
bareng-bareng, kayak sebuah `Rectangle` (persegi panjang) dengan `width` 
(lebar) sama `height` (tinggi)-nya, enum ngasih kita cara buat bilang kalau 
sebuah nilai itu adalah salah satu dari sekumpulan nilai yang mungkin. Misalnya, 
kita mungkin mau bilang kalau `Rectangle` itu salah satu dari sekumpulan bentuk 
yang mungkin yang juga termasuk `Circle` (lingkaran) sama `Triangle` (segitiga). 
Buat lakuin ini, Rust ngebolehin kita buat nyimpen (encode) kemungkinan-kemungkinan 
ini sebagai sebuah enum.

Yuk kita liat situasi yang mungkin mau kita ekspresikan di kode dan liat kenapa 
enum itu berguna dan lebih cocok daripada struct di kasus ini. Katakanlah kita 
perlu ngurusin _IP addresses_ (alamat IP). Saat ini, ada dua standar utama yang 
dipake buat alamat IP: versi empat (v4) dan versi enam (v6). Karena cuma ini 
kemungkinan alamat IP yang bakal ditemuin sama program kita, kita bisa nge-
_enumerate_ (menjabarkan) semua varian yang mungkin, dari sinilah _enumeration_ 
dapet namanya.

Alamat IP mana pun bisa jadi alamat versi empat atau versi enam, tapi nggak 
bisa dua-duanya sekaligus. Sifat alamat IP itu bikin struktur data enum cocok 
karena sebuah nilai enum cuma bisa jadi salah satu dari varian-variannya. Baik 
alamat versi empat maupun versi enam itu tetep secara fundamental adalah alamat 
IP, jadi mereka harus diperlakukan sebagai tipe yang sama pas kode lagi nanganin 
situasi yang berlaku buat jenis alamat IP apa pun.

Kita bisa ekspresikan konsep ini di kode dengan mendefinisikan sebuah _enumeration_ 
`IpAddrKind` terus nge-list jenis-jenis alamat IP yang mungkin, yaitu `V4` dan 
`V6`. Ini adalah varian-varian dari enum-nya:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` sekarang adalah tipe data kustom yang bisa kita pake di tempat lain 
di kode kita.

### Nilai Enum

Kita bisa bikin instance dari masing-masing dari dua varian `IpAddrKind` kayak 
gini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Perhatiin ya kalau varian dari enum-nya punya _namespace_ di bawah nama 
enum-nya (identifier), dan kita pake titik dua ganda buat misahin keduanya. Ini 
berguna karena sekarang kedua nilai `IpAddrKind::V4` sama `IpAddrKind::V6` itu 
punya tipe yang sama: `IpAddrKind`. Kita terus bisa, misalnya, mendefinisikan 
sebuah fungsi yang nerima `IpAddrKind` mana pun:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Dan kita bisa manggil fungsi ini pake varian yang mana aja:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Pake enum punya lebih banyak keuntungan lagi. Kalau dipikir-pikir lagi soal 
tipe alamat IP kita, saat ini kita nggak punya cara buat nyimpen _data_ alamat 
IP aslinya; kita cuma tau apa _jenis_-nya doang. Berhubung kita baru aja belajar 
soal struct di Bab 5, kita mungkin tergoda buat nyelesein masalah ini pake 
struct kayak yang ditunjukin di Listing 6-1.

<Listing number="6-1" caption="Nyimpen data dan varian `IpAddrKind` dari sebuah alamat IP pake `struct`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Di sini, kita mendefinisikan sebuah struct `IpAddr` yang punya dua field: sebuah 
field `kind` yang tipenya `IpAddrKind` (enum yang kita definisikan sebelumnya) 
dan sebuah field `address` yang tipenya `String`. Kita punya dua instance dari 
struct ini. Yang pertama itu `home`, dan dia punya nilai `IpAddrKind::V4` 
sebagai `kind`-nya sama data alamat terkait `127.0.0.1`. Instance kedua adalah 
`loopback`. Dia punya varian lain dari `IpAddrKind` sebagai nilai `kind`-nya, 
yaitu `V6`, dan punya alamat `::1` yang terkait dengannya. Kita pake struct 
buat ngebungkus nilai `kind` sama `address` barengan, jadi sekarang variannya 
terkait sama nilainya.

Tapi, merepresentasikan konsep yang sama pake enum doang itu lebih singkat: 
bukannya naruh enum di dalem struct, kita bisa naruh datanya langsung ke dalem 
tiap varian enum. Definisi baru dari enum `IpAddr` ini bilang kalau baik varian 
`V4` maupun `V6` bakal punya nilai `String` yang terkait dengannya:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Kita nempelin data ke tiap varian dari enum secara langsung, jadi nggak perlu 
lagi struct tambahan. Di sini, juga lebih gampang buat liat detail lain soal 
gimana cara kerja enum: nama dari tiap varian enum yang kita definisikan juga 
jadi sebuah fungsi yang ngonstruksi sebuah instance dari enum itu. Yaitu, 
`IpAddr::V4()` adalah pemanggilan fungsi yang nerima argumen `String` terus 
balikin sebuah instance dari tipe `IpAddr`. Kita otomatis dapet fungsi 
_constructor_ ini sebagai hasil dari mendefinisikan enum-nya.

Ada lagi keuntungan pake enum bukannya struct: tiap varian bisa punya tipe dan 
jumlah data terkait yang beda-beda. Alamat IP versi empat bakal selalu punya 
empat komponen numerik yang nilainya antara 0 sampe 255. Kalau kita mau nyimpen 
alamat `V4` sebagai empat nilai `u8` tapi tetep mengekspresikan alamat `V6` 
sebagai satu nilai `String`, kita nggak bakal bisa lakuin itu pake struct. Enum 
nanganin kasus ini dengan gampang:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Kita udah nunjukin beberapa cara beda buat mendefinisikan struktur data buat 
nyimpen alamat IP versi empat sama versi enam. Tapi nyatanya, pengen nyimpen 
alamat IP dan nyimpen info soal jenis alamat apa mereka itu hal yang sangat 
umum sampe-sampe [standard library punya definisi yang bisa kita pake!][IpAddr] 
Yuk kita liat gimana standard library mendefinisikan `IpAddr`: dia punya enum 
dan varian yang persis sama kayak yang udah kita definisikan dan pake, tapi dia 
nempelin data alamat di dalem variannya dalam bentuk dua struct yang beda, yang 
didefinisikan secara beda buat tiap varian:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Kode ini ngegambarin kalau kita bisa masukin data jenis apa pun ke dalem varian 
enum: strings, tipe numerik, atau structs, misalnya. Kita bahkan bisa masukin 
enum lain! Selain itu, tipe-tipe standard library sering kali nggak jauh lebih 
ribet dari apa yang mungkin kita bikin sendiri.

Perhatiin ya walaupun standard library punya definisi buat `IpAddr`, kita tetep 
bisa bikin dan pake definisi kita sendiri tanpa bentrok karena kita belum bawa 
definisi dari standard library itu ke scope kita. Kita bakal bahas lebih lanjut 
soal bawa tipe ke scope di Bab 7.

Yuk kita liat contoh enum lain di Listing 6-2: yang ini punya macem-macem tipe 
yang disematkan (embedded) di variannya.

<Listing number="6-2" caption="Sebuah enum `Message` yang tiap variannya nyimpen jumlah dan tipe nilai yang beda">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Enum ini punya empat varian dengan tipe yang beda-beda:

- `Quit`: Nggak punya data yang terkait dengannya sama sekali.
- `Move`: Punya field bernama, kayak sebuah struct.
- `Write`: Termasuk sebuah `String` tunggal.
- `ChangeColor`: Termasuk tiga nilai `i32`.

Mendefinisikan sebuah enum dengan varian kayak yang ada di Listing 6-2 itu mirip 
sama mendefinisikan berbagai macam definisi struct, bedanya enum nggak pake 
keyword `struct` dan semua variannya dikelompokin di bawah satu tipe `Message`. 
Struct-struct berikut bisa nampung data yang sama kayak varian enum di atas:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Tapi kalau kita pake struct yang beda-beda, yang mana masing-masing punya 
tipenya sendiri, kita nggak bakal segampang itu mendefinisikan fungsi buat 
nerima semua jenis pesan ini kayak yang bisa kita lakuin sama enum `Message` 
yang didefinisikan di Listing 6-2, yang merupakan sebuah tipe tunggal.

Ada satu lagi kemiripan antara enum sama struct: sama kayak kita bisa 
mendefinisikan methods pada structs pake `impl`, kita juga bisa mendefinisikan 
methods pada enums. Ini sebuah method namanya `call` yang bisa kita definisikan 
pada enum `Message` kita:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Body dari method ini bakal pake `self` buat dapet nilai di mana kita manggil 
method itu. Di contoh ini, kita bikin variabel `m` yang punya nilai 
`Message::Write(String::from("hello"))`, dan itulah yang bakal jadi `self` di 
dalem body method `call` pas `m.call()` jalan.

Yuk kita liat enum lain di standard library yang sangat umum dan kepake sekali: 
`Option`.

### Enum `Option` dan Keuntungannya Dibandingin Nilai Null

Bagian ini ngeksplor studi kasus `Option`, yang merupakan enum lain yang 
didefinisikan sama standard library. Tipe `Option` nyimpen skenario yang sangat 
umum di mana sebuah nilai bisa ada isinya (something) atau bisa aja kosong 
nggak ada isinya sama sekali (nothing).

Misalnya, kalau kita minta item pertama dari list yang nggak kosong, kita bakal 
dapet sebuah nilai. Kalau kita minta item pertama dari list yang kosong, kita 
nggak dapet apa-apa. Mengekspresikan konsep ini dalam sistem tipe artinya 
_compiler_ bisa nge-cek apakah kita udah handle semua kasus yang seharusnya kita 
handle; fungsionalitas ini bisa nyegah _bug_ yang bener-bener umum di bahasa 
pemrograman lainnya.

Desain bahasa pemrograman sering kali dipikirin dari segi fitur apa aja yang 
dimasukin, tapi fitur apa aja yang nggak dimasukin (di-exclude) itu juga penting. 
Rust nggak punya fitur _null_ kayak yang dipunyai banyak bahasa lain. _Null_ 
adalah sebuah nilai yang artinya nggak ada nilai di sana. Di bahasa yang pake 
null, variabel itu selalu ada di salah satu dari dua state: null atau tidak-null.

Di presentasinya tahun 2009 yang judulnya “Null References: The Billion Dollar 
Mistake,” Tony Hoare, penemu null, bilang gini:

> Saya sebut ini kesalahan satu miliar dolar saya. Waktu itu, saya lagi desain 
> sistem tipe komprehensif pertama buat referensi di bahasa berbasis objek. 
> Tujuan saya adalah buat mastiin kalau semua penggunaan referensi harus bener-bener 
> aman, dengan pengecekan yang dilakuin otomatis sama _compiler_. Tapi saya 
> nggak bisa nahan godaan buat masukin referensi null, cuma karena itu gampang 
> sekali buat diimplementasikan. Ini udah memicu error, kerentanan, dan 
> kerusakan sistem yang nggak kehitung jumlahnya, yang mungkin udah nyebabin 
> penderitaan dan kerugian satu miliar dolar di empat puluh tahun terakhir.

Masalah dari nilai null adalah kalau kita nyoba pake nilai null seolah-olah itu 
nilai yang bukan-null, kita bakal dapet semacam error. Karena properti null 
atau tidak-null ini ada di mana-mana (pervasive), gampang sekali buat bikin 
error kayak gini.

Tapi, konsep yang dicoba diekspresikan sama null itu tetep berguna: sebuah null 
adalah nilai yang saat ini nggak valid atau absen karena suatu alasan.

Masalahnya sebenernya bukan di konsepnya tapi di implementasinya yang spesifik. 
Maka dari itu, Rust nggak punya null, tapi dia punya sebuah enum yang bisa 
mengekspresikan konsep kalau sebuah nilai itu ada atau absen. Enum ini adalah 
`Option<T>`, dan dia [didefinisikan sama standard library][option] kayak gini:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

Enum `Option<T>` ini saking bergunanya sampe dia dimasukkan ke dalem _prelude_; 
kita nggak perlu bawa dia ke scope secara eksplisit. Varian-variannya juga 
dimasukkan ke _prelude_: kita bisa pake `Some` sama `None` secara langsung 
tanpa prefix `Option::`. Enum `Option<T>` ini tetep cuma enum biasa, dan `Some(T)` 
serta `None` itu tetep varian dari tipe `Option<T>`.

Sintaks `<T>` adalah fitur di Rust yang belum kita bahas. Itu adalah _generic 
type parameter_ (parameter tipe generik), dan kita bakal bahas generik lebih 
detail di Bab 10. Buat sekarang, yang perlu kita tau adalah `<T>` artinya 
varian `Some` dari enum `Option` bisa nampung satu potong data dari tipe apa 
pun, dan tiap tipe konkret yang dipake gantiin `T` bakal bikin tipe `Option<T>` 
secara keseluruhan jadi tipe yang beda. Ini beberapa contoh pake nilai `Option` 
buat nampung tipe angka sama tipe char:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

Tipe dari `some_number` adalah `Option<i32>`. Tipe dari `some_char` adalah 
`Option<char>`, yang merupakan tipe yang beda. Rust bisa nebak (infer) tipe-tipe 
ini karena kita udah nentuin nilai di dalem varian `Some`. Buat `absent_number`, 
Rust nuntut kita buat nganotasi tipe `Option` secara keseluruhan: _compiler_ nggak 
bisa nebak tipe yang bakal ditampung sama varian `Some` pasangannya kalau cuma 
liat dari nilai `None` doang. Di sini, kita ngasih tau Rust kalau maksud kita 
adalah `absent_number` itu tipenya `Option<i32>`.

Pas kita punya nilai `Some`, kita tau kalau nilainya ada dan nilainya ditampung 
di dalem `Some`-nya. Pas kita punya nilai `None`, dalam arti tertentu maknanya 
sama kayak null: kita nggak punya nilai yang valid. Terus kenapa punya 
`Option<T>` itu lebih baik daripada punya null?

Singkatnya, karena `Option<T>` sama `T` (di mana `T` bisa tipe apa pun) adalah 
tipe yang beda, _compiler_ nggak bakal ngebolehin kita pake nilai `Option<T>` 
seolah-olah itu pasti nilai yang valid. Misalnya, kode ini nggak bakal bisa 
di-compile, karena dia nyoba nambahin `i8` ke dalem `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Kalau kita jalanin kode ini, kita dapet pesan error kayak gini:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Sadis ya! Intinya, pesan error ini artinya Rust nggak paham gimana cara nambahin 
`i8` sama `Option<i8>`, karena mereka berdua tipe yang beda. Pas kita punya 
nilai dari suatu tipe kayak `i8` di Rust, _compiler_ bakal mastiin kalau kita 
selalu punya nilai yang valid. Kita bisa lanjut dengan pede tanpa harus nge-cek 
null dulu sebelum pake nilai itu. Cuma pas kita punya `Option<i8>` (atau tipe 
nilai apa pun yang lagi kita kerjain) barulah kita harus khawatir soal kemungkinan 
nggak punya nilai, dan _compiler_ bakal mastiin kita handle kasus itu sebelum 
pake nilainya.

Dengan kata lain, kita harus convert `Option<T>` jadi `T` dulu sebelum kita bisa 
ngelakuin operasi `T` pake itu. Umumnya, ini ngebantu nangkap salah satu masalah 
paling umum sama null: ngasumsikan kalau sesuatu itu nggak null padahal 
sebenernya iya.

Ngilangin risiko salah ngasumsikan nilai nggak-null ngebantu kita biar lebih pede 
sama kode kita. Buat punya nilai yang mungkin bisa null, kita harus secara 
eksplisit milih (_opt in_) dengan bikin tipe dari nilai itu jadi `Option<T>`. 
Terus, pas kita pake nilai itu, kita diwajibkan buat secara eksplisit nanganin 
kasus pas nilainya itu null. Di mana pun ada nilai yang tipenya bukan `Option<T>`, 
kita _bisa_ dengan aman ngasumsikan kalau nilai itu bukan null. Ini adalah 
keputusan desain yang disengaja buat Rust buat ngebatesin null yang ada di 
mana-mana dan ningkatin keamanan kode Rust.

Terus gimana cara ngeluarin nilai `T` dari sebuah varian `Some` pas kita punya 
nilai bertipe `Option<T>` biar kita bisa pake nilainya? Enum `Option<T>` punya 
sangat banyak method yang kepake di berbagai situasi; kita bisa cek mereka di 
[dokumentasinya][docs]. Biasain diri sama method-method di `Option<T>` bakal 
sangat berguna di perjalanan kita bareng Rust.

Umumnya, buat pake sebuah nilai `Option<T>`, kita mau punya kode yang bakal 
nanganin tiap variannya. Kita mau ada kode yang bakal jalan cuma pas kita punya 
nilai `Some(T)`, dan kode ini dibolehin buat pake `T` di dalemnya. Kita mau ada 
kode lain yang jalan cuma kalau kita punya nilai `None`, dan kode itu nggak 
punya nilai `T` yang bisa dipake. Ekspresi `match` adalah konstruk _control flow_ 
yang ngelakuin hal ini pas dipake bareng enum: dia bakal ngejalanin kode yang 
beda-beda tergantung varian enum mana yang dia punya, dan kode itu bisa pake 
data yang ada di dalem nilai yang cocok.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
