## Tipe Data Generik

Kita memakai generik buat membuat definisi untuk item seperti _signature_ fungsi 
atau _struct_, yang nantinya bisa kita pakai dengan berbagai macam tipe data 
konkret. Mari kita lihat dulu gimana cara mendefinisikan fungsi, _struct_, _enum_, 
dan _method_ memakai generik. Setelah itu, kita bakal membahas gimana pengaruh 
generik terhadap performa kode.

### Di Definisi Fungsi

Saat mendefinisikan fungsi yang memakai generik, kita menaruh generik itu di 
_signature_ fungsi di tempat kita biasanya menentukan tipe data untuk parameter 
dan nilai kembalian. Melakukan hal ini bikin kode kita jadi lebih fleksibel dan 
memberikan lebih banyak fungsionalitas bagi pemanggil fungsi kita sekaligus 
mencegah duplikasi kode.

Melanjutkan fungsi `largest` kita, Listing 10-4 menunjukkan dua fungsi yang 
keduanya mencari nilai paling besar di dalam sebuah _slice_. Kita bakal 
menggabungkan kedua fungsi ini jadi satu fungsi tunggal yang memakai generik.

<Listing number="10-4" file-name="src/main.rs" caption="Dua fungsi yang cuma beda di nama dan tipe di _signature_-nya">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

Fungsi `largest_i32` adalah fungsi yang kita ekstrak di Listing 10-3 untuk 
mencari `i32` paling besar di dalam _slice_. Fungsi `largest_char` mencari 
`char` paling besar di dalam _slice_. Body fungsinya punya kode yang persis 
sama, jadi mari kita hilangkan duplikasi ini dengan memperkenalkan parameter 
tipe generik di satu fungsi tunggal.

Buat memparameterisasi tipe di fungsi tunggal yang baru, kita harus menamai 
parameter tipenya, sama seperti kita menamai parameter nilai buat sebuah fungsi. 
Kita bisa memakai identifier (nama) apa saja sebagai nama parameter tipe. Tapi 
kita bakal memakai `T` karena, secara konvensi, nama parameter tipe di Rust itu 
pendek, sering kali cuma satu huruf, dan konvensi penamaan tipe di Rust adalah 
_CamelCase_. Singkatan dari _type_ (tipe), `T` adalah pilihan default buat 
kebanyakan programmer Rust.

Pas kita memakai sebuah parameter di dalam body fungsi, kita harus 
mendeklarasikan nama parameter itu di _signature_ agar _compiler_ tau apa makna 
nama tersebut. Demikian juga, pas kita memakai nama parameter tipe di _signature_ 
fungsi, kita harus mendeklarasikan nama parameter tipe itu sebelum memakainya. 
Untuk mendefinisikan fungsi `largest` yang generik, kita menaruh deklarasi nama 
tipe di dalam kurung sudut, `<>`, di antara nama fungsinya dan daftar parameternya, 
seperti ini:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Kita ngebaca definisi ini sebagai: fungsi `largest` bersifat generik terhadap 
suatu tipe `T`. Fungsi ini punya satu parameter bernama `list`, yang merupakan 
sebuah _slice_ berisi nilai bertipe `T`. Fungsi `largest` bakal mengembalikan 
referensi ke nilai dengan tipe `T` yang sama.

Listing 10-5 menunjukkan definisi fungsi `largest` gabungan yang memakai tipe 
data generik di _signature_-nya. Listing ini juga menunjukkan gimana kita bisa 
memanggil fungsi tersebut baik dengan _slice_ nilai `i32` maupun nilai `char`. 
Perhatikan bahwa kode ini belum bisa di-compile.

<Listing number="10-5" file-name="src/main.rs" caption="Fungsi `largest` yang memakai parameter tipe generik; kode ini belum bisa di-compile">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Kalau kita men-compile kode ini sekarang, kita bakal dapat error ini:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Teks bantuannya menyebutkan `std::cmp::PartialOrd`, yang mana itu adalah sebuah 
_trait_, dan kita bakal membahas _traits_ di bagian selanjutnya. Buat sekarang, 
ketahuilah bahwa error ini menyatakan kalau body dari `largest` tidak bakal jalan 
untuk semua tipe yang mungkin bakal mengisi `T`. Karena kita mau membandingkan 
nilai-nilai bertipe `T` di dalam body-nya, kita cuma bisa memakai tipe-tipe 
yang nilainya bisa diurutkan. Buat memungkinkan perbandingan, _standard library_ 
punya _trait_ `std::cmp::PartialOrd` yang bisa kita implementasikan di tipe-tipe 
tertentu (lihat Lampiran C buat info lebih lanjut soal _trait_ ini). Buat 
memperbaiki Listing 10-5, kita bisa mengikuti saran di teks bantuannya dan 
membatasi tipe-tipe yang valid buat `T` hanya pada tipe-tipe yang 
mengimplementasikan `PartialOrd`. Listing ini kemudian bakal bisa di-compile, 
karena _standard library_ mengimplementasikan `PartialOrd` buat `i32` dan `char`.

### Di Definisi Struct

Kita juga bisa mendefinisikan _struct_ agar memakai parameter tipe generik di 
satu atau lebih field-nya menggunakan sintaks `<>`. Listing 10-6 mendefinisikan 
sebuah _struct_ `Point<T>` buat menampung nilai koordinat `x` dan `y` dari tipe 
apa pun.

<Listing number="10-6" file-name="src/main.rs" caption="Sebuah _struct_ `Point<T>` yang menampung nilai `x` dan `y` bertipe `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Sintaks buat memakai generik di definisi _struct_ itu mirip kayak yang dipakai 
di definisi fungsi. Pertama kita deklarasikan nama parameter tipenya di dalam 
kurung sudut persis setelah nama _struct_-nya. Kemudian kita pakai tipe generik 
itu di definisi _struct_-nya di tempat kita biasanya memasukkan tipe data konkret.

Perhatikan bahwa karena kita cuma pakai satu tipe generik buat mendefinisikan 
`Point<T>`, definisi ini berarti _struct_ `Point<T>` bersifat generik terhadap 
suatu tipe `T`, dan field `x` serta `y` itu _keduanya_ memiliki tipe yang sama 
tersebut, tidak peduli apa tipe aslinya. Kalau kita bikin instance dari 
`Point<T>` yang punya nilai dengan tipe yang berbeda, seperti di Listing 10-7, 
kode kita tidak bakal bisa di-compile.

<Listing number="10-7" file-name="src/main.rs" caption="Field `x` dan `y` harus punya tipe yang sama karena keduanya punya tipe data generik yang sama yaitu `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

Di contoh ini, saat kita nge-assign nilai integer `5` ke `x`, kita memberitahu 
_compiler_ kalau tipe generik `T` bakal jadi integer untuk instance `Point<T>` 
ini. Lalu saat kita memberikan `4.0` untuk `y`, yang mana sebelumnya kita 
definisikan punya tipe yang sama dengan `x`, kita bakal dapat error ketidakcocokan 
tipe (type mismatch) seperti ini:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Buat mendefinisikan _struct_ `Point` di mana `x` dan `y` keduanya adalah generik 
tapi bisa punya tipe yang berbeda, kita bisa memakai banyak parameter tipe generik. 
Misalnya, di Listing 10-8, kita mengubah definisi `Point` agar bersifat generik 
terhadap tipe `T` dan `U`, di mana `x` bertipe `T` dan `y` bertipe `U`.

<Listing number="10-8" file-name="src/main.rs" caption="Sebuah `Point<T, U>` yang generik terhadap dua tipe sehingga `x` dan `y` bisa menampung nilai dengan tipe yang berbeda">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Sekarang semua instance dari `Point` yang ditunjukkan itu diperbolehkan! Kita 
bisa memakai sebanyak apa pun parameter tipe generik di sebuah definisi, tapi 
memakai lebih dari beberapa bakal bikin kode kita jadi susah dibaca. Kalau kita 
merasa butuh banyak tipe generik di kode kita, itu bisa jadi pertanda kalau 
kode kita butuh direstrukturisasi jadi bagian-bagian yang lebih kecil.

### Di Definisi Enum

Sama seperti _struct_, kita bisa mendefinisikan _enum_ buat menampung tipe 
data generik di dalam variannya. Mari kita lihat lagi _enum_ `Option<T>` yang 
disediakan oleh _standard library_, yang kita pakai di Bab 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Definisi ini seharusnya sekarang jadi lebih masuk akal. Seperti yang bisa kita 
lihat, _enum_ `Option<T>` bersifat generik terhadap tipe `T` dan punya dua varian: 
`Some`, yang menampung satu nilai bertipe `T`, dan varian `None` yang tidak 
menampung nilai apa pun. Dengan memakai _enum_ `Option<T>`, kita bisa 
mengekspresikan konsep abstrak dari nilai yang opsional (bisa ada isinya atau 
tidak), dan karena `Option<T>` itu generik, kita bisa memakai abstraksi ini 
apa pun tipe dari nilai opsional tersebut.

_Enum_ juga bisa memakai banyak tipe generik. Definisi dari _enum_ `Result` 
yang kita pakai di Bab 9 adalah salah satu contohnya:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

_Enum_ `Result` bersifat generik terhadap dua tipe, `T` dan `E`, dan punya dua 
varian: `Ok`, yang menampung nilai bertipe `T`, dan `Err`, yang menampung nilai 
bertipe `E`. Definisi ini membuatnya sangat nyaman untuk memakai _enum_ `Result` 
di mana pun kita punya operasi yang mungkin berhasil (mengembalikan nilai bertipe 
`T`) atau gagal (mengembalikan error bertipe `E`). Kenyataannya, inilah yang kita 
pakai buat membuka file di Listing 9-3, di mana `T` diisi dengan tipe 
`std::fs::File` saat filenya berhasil dibuka dan `E` diisi dengan tipe 
`std::io::Error` pas ada masalah saat membuka file tersebut.

Kalau kita mengenali situasi di kode kita di mana banyak definisi _struct_ atau 
_enum_ yang cuma berbeda di tipe nilai yang mereka tampung, kita bisa 
menghindari duplikasi dengan memakai tipe generik.

### Di Definisi Method

Kita bisa mengimplementasikan _method_ di _struct_ dan _enum_ (seperti yang kita 
lakukan di Bab 5) dan memakai tipe generik di definisinya juga. Listing 10-9 
menunjukkan _struct_ `Point<T>` yang kita definisikan di Listing 10-6 dilengkapi 
sebuah _method_ bernama `x` yang diimplementasikan di atasnya.

<Listing number="10-9" file-name="src/main.rs" caption="Mengimplementasikan sebuah _method_ bernama `x` di _struct_ `Point<T>` yang bakal mengembalikan referensi ke field `x` bertipe `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Di sini, kita sudah mendefinisikan sebuah _method_ bernama `x` pada `Point<T>` 
yang mengembalikan referensi ke data di field `x`.

Perhatikan bahwa kita harus mendeklarasikan `T` persis setelah `impl` supaya kita 
bisa memakai `T` buat menentukan kalau kita lagi mengimplementasikan _method_ di 
tipe `Point<T>`. Dengan mendeklarasikan `T` sebagai tipe generik setelah `impl`, 
Rust bisa mengidentifikasi kalau tipe di dalam kurung sudut di `Point` itu adalah 
tipe generik, bukan tipe konkret. Kita bisa saja memilih nama yang berbeda buat 
parameter generik ini daripada parameter generik yang dideklarasikan di definisi 
_struct_, tapi memakai nama yang sama adalah konvensi yang umum. Kalau kita menulis 
_method_ di dalam `impl` yang mendeklarasikan tipe generik, _method_ itu bakal 
didefinisikan pada instance tipe apa pun, tidak peduli tipe konkret apa yang 
akhirnya menggantikan tipe generiknya.

Kita juga bisa ngasih batasan pada tipe generik saat mendefinisikan _method_ 
pada suatu tipe. Misalnya, kita bisa mengimplementasikan _method_ hanya pada 
instance `Point<f32>` saja bukannya pada instance `Point<T>` dengan tipe generik 
apa pun. Di Listing 10-10 kita memakai tipe konkret `f32`, yang artinya kita 
tidak mendeklarasikan tipe apa pun setelah `impl`.

<Listing number="10-10" file-name="src/main.rs" caption="Blok `impl` yang hanya berlaku buat _struct_ dengan tipe konkret tertentu buat parameter tipe generik `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

Kode ini berarti tipe `Point<f32>` bakal punya _method_ `distance_from_origin`; 
instance `Point<T>` lain di mana `T` bukan tipe `f32` tidak bakal punya _method_ 
ini. _Method_ ini mengukur seberapa jauh titik kita dari titik origin di 
koordinat (0.0, 0.0) dan memakai operasi matematika yang hanya tersedia buat tipe 
_floating-point_.

Parameter tipe generik di definisi _struct_ tidak selalu sama dengan parameter 
yang kita pakai di _signature_ _method_ pada _struct_ yang sama. Listing 10-11 
memakai tipe generik `X1` dan `Y1` buat _struct_ `Point` serta `X2` `Y2` buat 
_signature_ _method_ `mixup` buat bikin contohnya jadi lebih jelas. _Method_ ini 
bikin instance `Point` baru dengan nilai `x` dari `Point` yang mewakili `self` 
(bertipe `X1`) dan nilai `y` dari `Point` yang di-pass masuk (bertipe `Y2`).

<Listing number="10-11" file-name="src/main.rs" caption="Sebuah _method_ yang memakai tipe generik yang berbeda dari definisi _struct_-nya">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

Di `main`, kita sudah mendefinisikan sebuah `Point` yang punya `i32` buat `x` 
(dengan nilai `5`) dan `f64` buat `y` (dengan nilai `10.4`). Variabel `p2` 
adalah _struct_ `Point` yang punya _string slice_ buat `x` (dengan nilai 
`"Hello"`) dan `char` buat `y` (dengan nilai `c`). Memanggil `mixup` pada `p1` 
dengan argumen `p2` bakal menghasilkan `p3`, yang bakal punya `i32` buat `x` 
karena `x`-nya datang dari `p1`. Variabel `p3` bakal punya `char` buat `y` karena 
`y`-nya datang dari `p2`. Pemanggilan macro `println!` bakal mencetak 
`p3.x = 5, p3.y = c`.

Tujuan dari contoh ini adalah buat mendemonstrasikan situasi di mana beberapa 
parameter generik dideklarasikan bersama `impl` dan beberapa lainnya dideklarasikan 
bersama definisi _method_-nya. Di sini, parameter generik `X1` dan `Y1` 
dideklarasikan setelah `impl` karena mereka ditujukan buat definisi _struct_-nya. 
Parameter generik `X2` dan `Y2` dideklarasikan setelah `fn mixup` karena mereka 
cuma relevan buat _method_ itu aja.

### Performa Kode yang Memakai Generik

Kita mungkin bertanya-tanya apakah ada biaya performa saat _runtime_ kalau kita 
pakai parameter tipe generik. Kabar baiknya adalah memakai tipe generik tidak 
bakal bikin program kita berjalan lebih lambat dibanding kalau kita memakai tipe 
konkret.

Rust mencapai ini dengan melakukan _monomorphization_ pada kode yang memakai 
generik di saat _compile time_. _Monomorphization_ adalah proses mengubah kode 
generik jadi kode spesifik dengan mengisi tipe-tipe konkret yang dipakai pas 
di-compile. Dalam proses ini, _compiler_ melakukan hal kebalikan dari langkah-
langkah yang kita ambil buat bikin fungsi generik di Listing 10-5: _compiler_ 
melihat semua tempat di mana kode generik itu dipanggil dan menghasilkan kode 
buat tipe-tipe konkret tempat kode generik itu dipanggil.

Mari kita lihat gimana proses ini bekerja dengan menggunakan _enum_ generik 
`Option<T>` dari _standard library_:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Pas Rust men-compile kode ini, dia melakukan _monomorphization_. Selama proses 
itu, _compiler_ membaca nilai-nilai yang udah dipakai di instance `Option<T>` 
dan mengenali dua jenis `Option<T>`: satu buat `i32` dan satu lagi buat `f64`. 
Oleh karena itu, dia memperluas definisi generik dari `Option<T>` jadi dua 
definisi yang dikhususkan buat `i32` dan `f64`, sehingga mengganti definisi 
generik dengan yang spesifik.

Versi kode hasil _monomorphization_ terlihat mirip seperti berikut (_compiler_ 
sebenarnya memakai nama yang berbeda dari apa yang kita pakai di sini buat 
ilustrasi):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

Generik `Option<T>` digantikan dengan definisi spesifik yang dibikin sama 
_compiler_. Karena Rust men-compile kode generik jadi kode yang menentukan tipe 
asli di masing-masing instance, kita tidak harus membayar biaya apa pun saat 
_runtime_ akibat pemakaian generik. Pas kodenya jalan, performanya sama persis 
seperti kalau kita menduplikasi masing-masing definisi pakai tangan sendiri. 
Proses _monomorphization_ ini bikin generik di Rust jadi sangat efisien pas 
_runtime_.
