## Mendefinisikan Enum

structs memberi anda jalan untuk mengelompokkan beberapa bidang yang berhubungan dan data, 
seperti `Rectangle` dengan bidang `width` dan `height`, enums memberi anda jalan 
untuk bilang bahwa nilai adalah salah satu dari beberapa kemungkinan set nilai. Sebagai contoh, 
kita mungkin ingin untuk bilang bahwa `Rectangle` adalah salah satu set dari berbagai kemungkinan bentuk 
juga termasuk `Circle` dan `Triangle`. 
Untuk itu Rust mengizinkan kita untuk mengenkode kemungkinan ini dengan enum.

Mari lihat situasi dimana kita mungkin ingin untuk mengekspresikannya di kode 
dan lihat kenapa enums sangat berguna dan lebih sesuai dari pada structs untuk kasus ini.
Anggaplah kita butuh bekerja dengan alamat IP. Pada saat ini, dua standar utama yang digunakan untuk alamat IP: versi empat dan versi enam. 
Karena ini adalah satu-satunya kemungkinan untuk sebuah alamat IP yang kita temui di program kita, kita bisa *enumerate* atau menghitung semua kemungkinan variant nya, oleh sebab itu mengapa di namai enumeration.

Setiap alamat IP bisa berupa versi empat atau versi enam, tetapi tidak bisa keduanya di waktu yang bersamaan. 
Properti alamat IP itu membuat struktur data enum sesuai karena nilai enum hanya dapat menjadi salah satu variantnya.
Versi empat maupun versi enam masih berupa fundamental dari IP addresses, jadi seharusnya akan di perlakukan 
seperti type yang sama ketika kode di handle di situasi yang menggunakan type apapun dari IP address. 

Kita bisa mengekspresikan konsep ini di kode dengan mendefinisikan `IpAddrKind` enumeration dan
dan me-list beberapa kemungkinan IP address di antaranya, `V4` dan `V6`. Ini adalah variant dari enum:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` sekarang adalah custom type data yang bisa kita gunakan dimana saja di kode anda.

### Nilai Enum

Kita dapat membuat instance dari masing-masing dua variant `IpAddrKind` seperti ini:
```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```
Perhatikan bahwa variant enum diberi namespaced di bawah pengidentifikasinya, dan kita
gunakan titik dua untuk memisahkan keduanya. Ini berguna karena sekarang keduanya bernilai
`IpAddrKind::V4` dan `IpAddrKind::V6` memiliki jenis yang sama: `IpAddrKind`. Kita
kemudian dapat, misalnya, mendefinisikan fungsi yang menggunakan `IpAddrKind` apa pun:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Dan kita dapat memanggil fungsi ini dengan variant mana pun:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```
Menggunakan enum memiliki lebih banyak keuntungan. Memikirkan lebih banyak tentang jenis alamat IP kita,
saat ini kita tidak memiliki cara untuk menyimpan *data* alamat IP sebenarnya; Kita
hanya tahu *jenis* apa itu. Mengingat Anda baru saja mempelajari tentang struct di
Bab 5, Anda mungkin terpikir untuk mengatasi masalah ini dengan struct seperti yang ditunjukkan pada
Daftar 6-1.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

<span class="caption">Daftar 6-1: Menyimpan data dan `IpAddrKind` variant daru alamat IP menggunakan sebuah `struct`</span>

Disini kita mendefinisikan struct `IpAddr` yang mempunyai dua bidang yaitu: bidang 
`kind`yang bertipe `IpAddrKind` (enum yang kita definisikan sebelumnya) dan bidang
`address` yang bertipe `String`. Kita mempunyai dua instance dari struct ini. Yang pertama adalah `home`,
dan mempunyai nilai `IpAddrKind::V4` sebagai nilai `kind` nya, dengan asosiasi data dari alamat `127.0.0.1`. 
Yang ke dua adalah instance `loopback`. Mempunyai variant yang lain dari `IpAddrKind` dengan `kind` yaitu `V6`, 
dan mempunyai alamat `::1` yang terasosiasi dengannya. kita menggunakan struct untuk mem-bundle kedua nilai `kind` dan `address`, jadi sekarang variant sudah terasosiasi dengan nilai nya.

Namun, merepresentasikan konsep yang sama hanya dengan menggunakan enum akan lebih ringkas:
daripada enum di dalam struct, kita bisa memasukkan data langsung ke setiap variant pada enum nya.
Definisi baru dari enum `IpAddr` ini mengatakan bahwa variant `V4` dan `V6` 
akan memiliki nilai `String` yang terkait:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Kita melampirkan data ke setiap variant enum secara langsung, sehingga tidak diperlukan tambahan struct. 
Di sini, juga lebih mudah untuk melihat detail lain tentang cara kerja enum:
nama setiap variant enum yang kita definisikan juga menjadi fungsi itu
membangun sebuah instance dari enum. Artinya, `IpAddr::V4()` adalah pemanggilan fungsi
yang mengambil argumen `String` dan mengembalikan instance tipe `IpAddr`. Kita
secara otomatis mendefinisikan fungsi constructor ini sebagai hasil dari pendefinisian
enum.

Ada keuntungan lain menggunakan enum daripada struct: setiap variant
dapat memiliki jenis dan jumlah data terkait yang berbeda. Alamat IP versi empat akan 
selalu memiliki empat komponen numerik yang akan memiliki nilai antara 0 dan 255. 
Jika kita ingin menyimpan alamat `V4` sebagai empat nilai `u8` tetapi
masih menyatakan alamat `V6` sebagai satu nilai `String`, kita tidak akan bisa 
melakukan ini dengan sebuah struct. Enum menangani kasus ini dengan mudah:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```
kita telah menunjukkan beberapa cara berbeda untuk menentukan struktur data untuk menyimpan versi
empat dan versi enam alamat IP. Namun, Menyimpan alamat Ip dan meng-enkode kind nya adalah hal yang umum
maka [perpustakaan standar mempunyai definisi tersebut yang dapat kita gunakan secara langsung!][IpAddr]<!-- ignore --> 
Mari kita lihat caranya
perpustakaan standar mendefinisikan `IpAddr`: ia memiliki enum dan variant yang tepat seperti
yang kita telah definisikan dan gunakan, tetapi data alamat tertanam di dalam variant di
bentuk dua struct berbeda, yang didefinisikan berbeda untuk masing-masingnya
varian:

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

Kode ini mengilustrasikan bahwa Anda dapat memasukkan segala jenis data ke dalam variant enum:
string, tipe numerik, atau struct, misalnya. Anda bahkan dapat memasukkan enum yang lain!
Selain itu, tipe perpustakaan standar seringkali tidak lebih rumit dari apa yang mungkin Anda temukan.

Perhatikan bahwa meskipun perpustakaan standar berisi definisi untuk `IpAddr`,
kita tetap dapat membuat dan menggunakan definisi kita sendiri tanpa konflik karena kita
belum membawa definisi perpustakaan standar ke dalam scope kita atau meng-import nya. 
kita akan bicara lebih lanjut tentang import tipe ke dalam scope di Bab 7.

Mari kita lihat contoh enum yang lain di Daftar 6-2: yang ini memiliki cakupan yang luas
beragam tipe tersemat pada variantnya.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

<span class="caption">Daftar 6-2: Sebuah enum `Message` yang setiap variant nya 
menyimpan nilai dari tipe data yang berbeda</span>

Enum ini memliliki empat variant dengan tipe data yang berbeda-beda:

* `Quit` tidak mempunyai asosiasi tipe data sama sekali.
* `Move` mempunyai bidang yang bernama, seperti yang bisa dilakukan oleh struct.
* `Write` mempunyai satu buah `String`.
* `ChangeColor` mempunyai tiga buah nilai `i32`.

Mendefinisikan enum dengan variant seperti yang ada di Daftar 6-2 serupa dengan
mendefinisikan berbagai jenis definisi struct, kecuali enum tidak menggunakan
Kata kunci `struct` dan semua variantnya dikelompokkan bersama di bawah `Message`
jenis. Struct berikut dapat menyimpan data yang sama dengan enum sebelumnya 
sekaligus variant nya:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Tetapi jika kita menggunakan struct yang berbeda, yang masing-masing memiliki tipenya sendiri, kita
tidak dapat dengan mudah mendefinisikan suatu fungsi untuk menerima pesan-pesan seperti ini
kita bisa melakukannya dengan enum `Message` yang ditentukan dalam Daftar 6-2, yang merupakan tipe tunggal.

Ada satu lagi kesamaan antara enum dan struct: sama seperti yang bisa kita lakukan
mendefinisikan metode pada struct menggunakan `impl`, kita juga dapat mendefinisikan metode pada
enum. Contohnya adalah metode bernama `call` yang dapat kita definisikan pada enum `Message` kita:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Badan metode akan menggunakan `self` untuk mendapatkan nilai pada metode tersebut saat di panggil.
Dalam contoh ini, kita telah membuat variabel `m` yang memiliki nilai
`Message::Write(String::from("hello"))`, dan itulah yang akan menjadi `self` di
isi metode `call` saat `m.call()` dijalankan.

Mari kita lihat enum lain di perpustakaan standar yang sangat umum dan
berguna yaitu: `Option`.

### Enum `Option` dan Keunggulannya Dibandingkan Nilai Null

Bagian ini mengeksplorasi studi kasus `Option`, yang merupakan enum lain yang ditentukan
oleh perpustakaan standar. Tipe `Option` mengkodekan skenario yang sangat umum
dimana suatu nilai bisa berupa sesuatu atau bisa juga kosong atau tidak ada.

Misalnya, jika Anda meminta item pertama dalam daftar yang tidak kosong, Anda akan mendapatkannya
sebuah nilai. Jika Anda meminta item pertama dalam daftar kosong, Anda tidak akan mendapatkan apa pun.
Mengekspresikan konsep ini dalam bentuk type system berarti kompiler bisa
memeriksa apakah Anda sudah menangani semua kasus yang seharusnya Anda tangani; 
fungsionalitas ini dapat mencegah bug yang sangat umum terjadi pada bahasa pemrograman lain.

Desain bahasa pemrograman sering kali dianggap sesuai dengan apa saja fitur yang disertakan, 
namun fitur yang dikecualikan juga penting. Rust tidak memiliki fitur null
yang dimiliki banyak bahasa lain. *Null* adalah nilai yang artinya tidak ada nilainya di sana.
Dalam bahasa dengan null, variabel selalu bisa berada di salah satu dua keadaan: null atau not-null.

Dalam presentasinya tahun 2009, "Null References: The Billion Dollar Mistake," Tony
Hoare, penemu null, mengatakan ini:

> Pada saat itu, saya sedang merancang type system komprehensif pertama 
> untuk referensi dalam bahasa berorientasi objek. Tujuan saya adalah memastikan 
> bahwa semua penggunaan referensi harus benar-benar aman, dengan pemeriksaan
> dilakukan secara otomatis oleh kompiler. Namun saya tidak bisa menahan godaan
> untuk memasukkan referensi null, hanya karena sangat mudah diterapkan. Hal ini 
> telah menyebabkan banyak sekali kesalahan, kerentanan, dan kerusakan sistem,
> yang mungkin menyebabkan kerugian dan kerusakan senilai miliaran dolar 
> dalam empat puluh tahun terakhir.

Masalah dengan nilai null adalah jika Anda mencoba menggunakan nilai null sebagai
nilai yang not-null, Anda akan mendapatkan semacam kesalahan. Karena ini null 
atau not-null properti tersebar luas, sangat mudah untuk membuat kesalahan seperti ini.

Namun, konsep yang coba diungkapkan oleh null masih berguna: null adalah 
nilai yang saat ini tidak valid atau tidak ada karena alasan tertentu.

Masalahnya sebenarnya bukan pada konsepnya, melainkan pada penerapannya. 
Dengan demikian, Rust tidak memiliki null, tetapi memiliki enum yang dapat
menyandikan konsep suatu nilai ada atau tidak ada. Enum ini adalah `Option<T>`,
dan itu sudah [didefinisikan oleh perpustakaan standar][option]<!-- ignore -->
seperti berikut:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

Enum `Option<T>` sangat berguna bahkan disertakan dalam prelude; Anda
tidak perlu memasukkannya ke dalam scope secara eksplisit. Variantnya juga disertakan
dalam prelude: Anda dapat menggunakan `Some` dan `None` secara langsung tanpa awalan
`Option::`. Enum `Option<T>` masih berupa enum biasa, dan `Some(T)` dan
`None` masih merupakan variant dari tipe `Option<T>`.

Sintaks `<T>` adalah fitur Rust yang belum kita bicarakan. Ini adalah sebuah
parameter bertipe generik, dan generik nanti akan dibahas secara lebih rinci di Bab 10.
Untuk saat ini, yang perlu Anda ketahui adalah `<T>` berarti variant `Some` dari
enum `Option` dapat menampung satu bagian data jenis apa pun, dan masing-masingnya
tipe yang digunakan sebagai pengganti `T` menjadikan tipe `Option<T>` secara keseluruhan
tipe yang berbeda. Berikut beberapa contoh penggunaan nilai `Option` yang menggunakan
tipe angka dan tipe string:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

Jenis `some_number` adalah `Option<i32>`. Jenis `some_char` adalah
`Option<char>`, yang merupakan tipe berbeda. Rust dapat menyimpulkan jenis ini karena
kita telah menentukan nilai di dalam variant `Some`. Untuk `absent_number`, Rust
mengharuskan kita untuk membubuhi keterangan pada keseluruhan tipe `Option`: kompiler 
tidak dapat menyimpulkan ketika variant `Some` yang sesuai akan ditampung dengan hanya 
melihat sebuah nilai `None`. Di sini, kita memberi tahu Rust bahwa yang kita maksud adalah
`absent_number` bertipe `Option<i32>`.

Ketika kita memiliki nilai `Some`, kita mengetahui bahwa suatu nilai ada dan nilai tersebut ada
dan didalam `Some`. Ketika kita memiliki nilai `None`, dalam arti tertentu itu berarti
hal yang sama dengan null: kita tidak memiliki nilai yang valid. Jadi mengapa ada `Option<T>`
adakah yang lebih baik daripada memiliki null?

Singkatnya, karena `Option<T>` dan `T` (di mana `T` dapat berupa jenis apa pun) berbeda
jenis, kompiler tidak akan membiarkan kita menggunakan nilai `Option<T>` seolah-olah itu adalah
pasti nilai yang valid. Misalnya, kode ini tidak dapat dikompilasi, karena kita mencoba menambahkan
`i8` ke `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Jika kita jalankan kode tersebut, kita akan dapat pesan kesalahan seperti berikut:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Intens! Sebenarnya, pesan kesalahan ini berarti Rust tidak mengerti caranya
untuk menambahkan `i8` dan `Option<i8>`, karena keduanya berbeda tipe. ketika kita
memiliki nilai tipe seperti `i8` di Rust, kompiler akan memastikan bahwa kita
selalu mempunyai nilai yang valid. Kita dapat melanjutkan dengan percaya diri tanpa harus memeriksa
untuk null sebelum menggunakan nilai itu. Hanya ketika kita memiliki `Option<i8>` (atau
apapun jenis nilai yang kita kerjakan) yang mungkin perlu kita khawatirkan
tidak memiliki nilai atau null, dan kompiler akan memastikan kita menangani kasus
itu sebelum menggunakan nilai tersebut.

Dengan kata lain, Anda harus mengonversi `Option<T>` menjadi `T` sebelum Anda bisa
melakukan operasi dengan `T`. Umumnya, ini membantu meminimalisir masalah umum dengan null:
dengan asumsi bahwa sesuatu tidaklah null padahal sebenarnya null.

Menghilangkan risiko salah mengasumsikan nilai yang bukan null akan membantu Anda
lebih percaya diri dengan kode Anda. Agar mempunyai nilai yang mungkin
null, Anda harus serta secara eksplisit menjadikan jenis nilai tersebut `Option<T>`. Kemudian, 
saat Anda menggunakan nilai tersebut, Anda diharuskan menangani kasus tersebut secara eksplisit 
ketika nilainya null. Di mana pun suatu nilai memiliki tipe yang bukan merupakan `Option<T>`, 
Anda *dapat* dengan aman berasumsi bahwa nilainya bukan null. Ini adalah sebuah
keputusan desain yang disengaja untuk Rust untuk membatasi penyebaran dan peningkatan null
dikeamanan kode Rust.

Jadi bagaimana Anda mendapatkan nilai `T` dari variant `Some` ketika Anda memiliki sebuah nilai
bertipe `Option<T>` sehingga Anda dapat menggunakan nilai itu? Enum `Option<T>` memiliki 
sejumlah metode yang berguna dalam berbagai situasi; anda bisa
lihat di [dokumentasinya][docs]<!-- ignore -->. Memahami sebagian metode pada `Option<T>` akan 
sangat berguna dalam perjalanan belajar anda di Rust.

Secara umum, untuk menggunakan nilai `Option<T>`, Anda harus memiliki kode yang akan menangani 
setiap variant. Anda dapat memiliki beberapa kode yang hanya akan berjalan jika Anda memiliki
Nilai `Some(T)`, dan kode ini diperbolehkan menggunakan bagian dalam `T`. anda mungkin juga mau
kode lain untuk dijalankan hanya jika Anda memiliki nilai `None`, dan kode tersebut tidak memiliki
nilai `T`. Ekspresi `match` adalah konstruksi aliran kontrol yang melakukan hal ini ketika 
digunakan dengan enum: ia akan menjalankan kode yang berbeda tergantung pada variant enum mana
yang dimilikinya, dan kode tersebut dapat menggunakan data di dalam variant yang cocok.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
