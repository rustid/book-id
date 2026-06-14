<!-- Old heading. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Closures: Fungsi Anonim yang Bisa Menangkap Lingkungannya

_Closures_ di Rust adalah fungsi anonim (tanpa nama) yang bisa kita simpan di 
dalam sebuah variabel atau diteruskan sebagai argumen ke fungsi lain. Kita bisa 
membuat sebuah _closure_ di satu tempat lalu memanggil _closure_ tersebut di 
tempat lain untuk dievaluasi dalam konteks yang berbeda. Tidak seperti fungsi 
biasa, _closures_ bisa menangkap (capture) nilai-nilai dari _scope_ tempat 
mereka didefinisikan. Kita bakal mendemonstrasikan gimana fitur-fitur _closure_ 
ini memungkinkan penggunaan ulang kode dan kustomisasi perilaku.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Menangkap Lingkungan Menggunakan Closures

Pertama-tama kita bakal meneliti gimana kita bisa memakai _closures_ buat 
menangkap nilai dari lingkungan tempat mereka didefinisikan untuk dipakai nanti. 
Berikut skenarionya: sesekali, perusahaan kaos kita membagikan kaos edisi 
terbatas eksklusif kepada seseorang di _mailing list_ kita sebagai bentuk promosi. 
Orang-orang di _mailing list_ bisa secara opsional menambahkan warna favorit 
mereka ke profilnya. Kalau orang yang terpilih untuk dapat kaos gratis itu sudah 
menge-set warna favoritnya, dia bakal dapat kaos dengan warna itu. Tapi kalau 
orang tersebut belum menentukan warna favorit, dia bakal dapat warna apa pun yang 
saat itu stoknya paling banyak di perusahaan.

Ada banyak cara buat mengimplementasikan ini. Buat contoh ini, kita bakal 
memakai sebuah _enum_ bernama `ShirtColor` yang punya varian `Red` (merah) dan 
`Blue` (biru) (kita membatasi jumlah warna yang ada biar simpel). Kita mewakili 
stok barang milik perusahaan dengan sebuah _struct_ `Inventory` yang punya field 
bernama `shirts` yang berisi sebuah `Vec<ShirtColor>` yang merepresentasikan 
warna-warna kaos yang saat ini ada di stok. Method `giveaway` yang didefinisikan 
pada `Inventory` menerima preferensi warna kaos opsional dari pemenang kaos 
gratis, lalu mengembalikan warna kaos yang bakal didapat oleh orang tersebut. 
Persiapan ini ditunjukkan di Listing 13-1.

<Listing number="13-1" file-name="src/main.rs" caption="Skenario pembagian hadiah dari perusahaan kaos">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

Toko (`store`) yang didefinisikan di fungsi `main` punya dua kaos biru dan satu 
kaos merah yang tersisa untuk dibagikan dalam promosi edisi terbatas ini. Kita 
memanggil method `giveaway` untuk seorang _user_ dengan preferensi kaos merah 
dan seorang _user_ tanpa preferensi sama sekali.

Sekali lagi, kode ini bisa saja diimplementasikan dengan banyak cara, dan di 
sini, untuk fokus ke _closures_, kita cuma memakai konsep-konsep yang sudah 
kita pelajari, kecuali buat _body_ dari method `giveaway` yang memakai sebuah 
_closure_. Di method `giveaway`, kita menerima preferensi _user_ sebagai sebuah 
parameter bertipe `Option<ShirtColor>` lalu memanggil method `unwrap_or_else` 
pada `user_preference`. [Method `unwrap_or_else` pada `Option<T>`][unwrap-or-else] 
didefinisikan oleh _standard library_. Method ini menerima satu argumen: sebuah 
_closure_ tanpa argumen apa pun yang mengembalikan sebuah nilai bertipe `T` 
(tipe yang sama dengan yang disimpan di varian `Some` dari `Option<T>`, yang 
mana di kasus ini adalah `ShirtColor`). Kalau `Option<T>` itu adalah varian 
`Some`, `unwrap_or_else` bakal mengembalikan nilai dari dalam `Some` tersebut. 
Tapi kalau `Option<T>` adalah varian `None`, `unwrap_or_else` bakal memanggil 
_closure_-nya dan mengembalikan nilai yang dikembalikan oleh _closure_ tersebut.

Kita menentukan ekspresi _closure_ `|| self.most_stocked()` sebagai argumen 
untuk `unwrap_or_else`. Ini adalah sebuah _closure_ yang tidak menerima parameter 
apa pun (kalau _closure_-nya punya parameter, parameternya bakal muncul di antara 
dua garis vertikal). _Body_ dari _closure_ ini memanggil `self.most_stocked()`. 
Kita mendefinisikan _closure_-nya di sini, dan implementasi dari `unwrap_or_else` 
nanti bakal mengevaluasi _closure_ ini kalau hasilnya memang dibutuhkan.

Menjalankan kode ini bakal mencetak output berikut:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Satu aspek yang menarik di sini adalah kita sudah meneruskan sebuah _closure_ 
yang memanggil `self.most_stocked()` pada instance `Inventory` saat ini. 
_Standard library_ tidak perlu tahu apa-apa tentang tipe `Inventory` atau 
`ShirtColor` yang kita definisikan, ataupun logika yang mau kita pakai di 
skenario ini. _Closure_ ini menangkap sebuah referensi _immutable_ ke instance 
`Inventory` `self` dan meneruskannya bersama kode yang kita tentukan ke method 
`unwrap_or_else`. Di sisi lain, fungsi biasa tidak bisa menangkap lingkungan 
mereka dengan cara seperti ini.

### Inference dan Anotasi Tipe untuk Closure

Ada lebih banyak perbedaan antara fungsi biasa dan _closures_. _Closures_ 
biasanya tidak mengharuskan kita untuk menganotasi tipe dari parameter atau nilai 
kembalian seperti yang diwajibkan oleh fungsi `fn`. Anotasi tipe diwajibkan 
pada fungsi karena tipe-tipe tersebut adalah bagian dari antarmuka eksplisit yang 
diekspos ke para pengguna fungsi tersebut. Mendefinisikan antarmuka ini secara 
kaku penting untuk memastikan bahwa semua orang sepakat mengenai tipe-tipe nilai 
yang dipakai dan dikembalikan oleh sebuah fungsi. Sebaliknya, _closures_ tidak 
dipakai dalam antarmuka yang terekspos seperti itu: mereka disimpan di dalam 
variabel dan dipakai tanpa menamai mereka atau mengeksposnya ke pengguna _library_ 
kita.

_Closures_ biasanya berukuran pendek dan hanya relevan di dalam konteks yang 
sempit, bukan di sembarang skenario acak. Di dalam batasan konteks ini, 
_compiler_ bisa menebak (infer) tipe-tipe parameternya dan tipe kembaliannya, 
mirip dengan bagaimana ia bisa menebak tipe dari sebagian besar variabel (walaupun 
ada kasus-kasus langka di mana _compiler_ juga membutuhkan anotasi tipe untuk 
_closure_).

Sama halnya dengan variabel, kita bisa menambahkan anotasi tipe kalau kita mau 
meningkatkan kejelasan secara eksplisit walau akibatnya kode kita bakal sedikit 
lebih bertele-tele (_verbose_) dari yang sebenarnya diperlukan. Menganotasi tipe 
buat sebuah _closure_ bakal kelihatan seperti definisi yang ditunjukkan di 
Listing 13-2. Di contoh ini, kita mendefinisikan sebuah _closure_ dan 
menyimpannya di dalam sebuah variabel, bukannya mendefinisikan _closure_ tersebut 
tepat di tempat kita meneruskannya sebagai argumen seperti yang kita lakukan di 
Listing 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="Menambahkan anotasi tipe opsional buat tipe parameter dan nilai kembalian di _closure_">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Dengan menambahkan anotasi tipe, sintaks _closures_ jadi kelihatan lebih mirip 
dengan sintaks fungsi biasa. Di sini, kita mendefinisikan sebuah fungsi yang 
menambahkan 1 ke parameternya dan sebuah _closure_ yang punya perilaku yang sama, 
sebagai perbandingan. Kita sudah menambahkan sedikit spasi supaya bagian-bagian 
yang relevan sejajar. Ini mengilustrasikan gimana sintaks _closure_ itu mirip 
dengan sintaks fungsi, kecuali di penggunaan garis vertikal (`|`) dan seberapa 
banyak sintaks yang sifatnya opsional:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Baris pertama menunjukkan sebuah definisi fungsi dan baris kedua menunjukkan 
definisi _closure_ yang dianotasi secara penuh. Di baris ketiga, kita membuang 
anotasi tipe dari definisi _closure_. Di baris keempat, kita membuang kurung 
kurawal, yang mana jadi opsional karena _body_ dari _closure_ ini hanya punya 
satu ekspresi. Ini semua adalah definisi yang valid dan bakal menghasilkan 
perilaku yang sama saat mereka dipanggil. Baris `add_one_v3` dan `add_one_v4` 
mewajibkan _closures_ tersebut untuk dievaluasi agar kodenya bisa di-compile, 
karena tipe-tipenya bakal ditebak berdasarkan gimana _closures_ tersebut dipakai. 
Ini mirip dengan bagaimana `let v = Vec::new();` membutuhkan entah anotasi tipe 
atau adanya nilai dengan tipe tertentu yang dimasukkan ke dalam `Vec` agar Rust 
bisa menebak tipenya.

Buat definisi _closure_, _compiler_ bakal menebak satu tipe konkret untuk masing-
masing parameternya dan juga untuk nilai kembaliannya. Misalnya, Listing 13-3 
menunjukkan definisi _closure_ singkat yang cuma mengembalikan nilai yang dia 
terima sebagai parameter. _Closure_ ini sebenarnya tidak terlalu berguna kecuali 
buat tujuan contoh ini. Perhatikan bahwa kita tidak menambahkan anotasi tipe apa 
pun di definisinya. Karena tidak ada anotasi tipe, kita bisa memanggil _closure_ 
ini dengan tipe apa pun, yang mana kita lakukan di sini dengan tipe `String` 
untuk panggilan pertama. Kalau kita kemudian mencoba memanggil `example_closure` 
dengan sebuah integer, kita bakal dapat error.

<Listing number="13-3" file-name="src/main.rs" caption="Mencoba memanggil sebuah _closure_ yang tipenya masih ditebak memakai dua tipe yang berbeda">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

_Compiler_ ngasih kita error ini:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Saat pertama kali kita memanggil `example_closure` memakai nilai `String`, 
_compiler_ menebak kalau tipe dari `x` dan tipe kembalian dari _closure_ itu 
adalah `String`. Tipe-tipe itu kemudian "terkunci" ke dalam _closure_ di 
`example_closure`, dan kita bakal dapat _type error_ (error tipe) saat kita 
mencoba memakai tipe yang berbeda dengan _closure_ yang sama.

### Menangkap Referensi atau Memindahkan Kepemilikan (Ownership)

_Closures_ bisa menangkap nilai dari lingkungannya memakai tiga cara, yang mana 
berkorelasi langsung dengan tiga cara sebuah fungsi bisa menerima parameter: 
meminjam secara _immutable_, meminjam secara _mutable_, dan mengambil 
kepemilikan (_taking ownership_). _Closure_ bakal memutuskan cara mana yang mau 
dipakai berdasarkan apa yang dilakukan oleh isi fungsinya terhadap nilai-nilai 
yang ditangkapnya.

Di Listing 13-4, kita mendefinisikan sebuah _closure_ yang menangkap referensi 
_immutable_ ke vector bernama `list` karena _closure_ itu cuma butuh referensi 
_immutable_ buat mencetak nilainya.

<Listing number="13-4" file-name="src/main.rs" caption="Mendefinisikan dan memanggil sebuah _closure_ yang menangkap sebuah referensi _immutable_">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Contoh ini juga mengilustrasikan kalau sebuah variabel bisa diikat ke definisi 
sebuah _closure_, dan nanti kita bisa memanggil _closure_ tersebut memakai nama 
variabel dan tanda kurung, seolah-olah nama variabel itu adalah nama sebuah fungsi.

Karena kita bisa punya banyak referensi _immutable_ ke `list` di waktu yang 
bersamaan, `list` masih bisa diakses dari kode sebelum definisi _closure_, di 
antara definisi _closure_ namun sebelum _closure_-nya dipanggil, dan setelah 
_closure_-nya dipanggil. Kode ini berhasil di-compile, jalan, dan mencetak:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Selanjutnya, di Listing 13-5, kita ngubah _body_ _closure_-nya biar dia nambahin 
sebuah elemen ke vector `list`. _Closure_ ini sekarang menangkap referensi 
_mutable_.

<Listing number="13-5" file-name="src/main.rs" caption="Mendefinisikan dan memanggil sebuah _closure_ yang menangkap sebuah referensi _mutable_">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Kode ini berhasil di-compile, jalan, dan mencetak:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Perhatikan bahwa sekarang tidak ada lagi `println!` di antara definisi dan 
pemanggilan _closure_ `borrows_mutably`: ketika `borrows_mutably` didefinisikan, 
ia menangkap referensi _mutable_ ke `list`. Kita tidak lagi menggunakan _closure_ 
ini setelah ia dipanggil, jadi peminjaman _mutable_ itu pun berakhir. Di antara 
definisi _closure_ dan pemanggilannya, kita tidak diizinkan buat melakukan 
peminjaman _immutable_ untuk mencetaknya karena peminjaman lain tidak 
diperbolehkan selama masih ada peminjaman _mutable_. Coba tambahkan `println!` 
di situ buat melihat pesan error apa yang bakal kita dapatkan!

Kalau kita mau memaksa _closure_ buat mengambil _ownership_ dari nilai yang dia 
pakai dari lingkungannya, biarpun isi _closure_-nya sebenarnya tidak 
mewajibkan _ownership_, kita bisa menambahkan keyword `move` sebelum daftar 
parameternya.

Teknik ini biasanya sangat berguna pas kita mau meneruskan sebuah _closure_ ke 
_thread_ baru untuk memindahkan datanya supaya ia dimiliki oleh _thread_ baru 
tersebut. Kita bakal bahas _threads_ dan kenapa kita mau menggunakannya secara 
mendetail di Bab 16 saat kita ngomongin soal konkurensi (concurrency), tapi buat 
sekarang, mari kita eksplor secara singkat cara bikin _thread_ baru yang memakai 
sebuah _closure_ yang membutuhkan keyword `move`. Listing 13-6 menampilkan kode 
dari Listing 13-4 yang diubah buat mencetak isi vector di sebuah _thread_ baru, 
bukannya di _thread_ utama (main thread).

<Listing number="13-6" file-name="src/main.rs" caption="Memakai `move` buat memaksa _closure_ di _thread_ baru untuk mengambil kepemilikan dari `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Kita membuat _thread_ baru, sambil memberikan sebuah _closure_ buat dijalankan 
sebagai argumen untuk _thread_ tersebut. Isi _closure_-nya mencetak daftar `list`. 
Di Listing 13-4, _closure_ tersebut cuma menangkap `list` dengan memakai 
referensi _immutable_ karena itulah hak akses minimal yang dibutuhkan ke `list` 
buat mencetaknya. Di contoh ini, meskipun isi _closure_-nya masih cuma butuh 
referensi _immutable_, kita harus menentukan secara spesifik bahwa `list` 
seharusnya dipindahkan (moved) ke dalam _closure_ dengan menaruh keyword `move` 
di awal definisi _closure_-nya. Kalau _thread_ utamanya melakukan lebih banyak 
operasi sebelum memanggil `join` pada _thread_ barunya, _thread_ baru tersebut 
bisa saja selesai sebelum _thread_ utamanya selesai, atau sebaliknya _thread_ 
utama bisa saja selesai lebih dulu. Kalau _thread_ utama masih mempertahankan 
_ownership_ dari `list` tapi dia berakhir lebih dulu sebelum _thread_ barunya 
dan men-_drop_ (membuang) `list` tersebut, referensi _immutable_ di _thread_ baru 
bakal jadi tidak valid. Maka dari itu, _compiler_ mewajibkan `list` untuk 
dipindahkan ke dalam _closure_ yang diberikan ke _thread_ baru supaya 
referensinya bakal dipastikan valid. Coba hapus keyword `move`-nya atau 
coba pakai variabel `list` di _thread_ utama setelah _closure_ itu didefinisikan 
buat melihat error _compiler_ apa yang bakal muncul!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Memindahkan Nilai yang Ditangkap ke Luar Closures dan Traits `Fn`

Begitu sebuah _closure_ sudah menangkap referensi atau mengambil kepemilikan dari 
sebuah nilai dari lingkungan tempat _closure_ itu didefinisikan (yang artinya, hal 
itu memengaruhi apa yang dipindahkan _ke dalam_ _closure_-nya), kode di dalam isi 
_closure_-nya bakal menentukan apa yang terjadi sama referensi atau nilai 
tersebut pas _closure_-nya dievaluasi nantinya (yang artinya, hal ini memengaruhi 
apa yang dipindahkan _ke luar_ dari _closure_-nya).

Isi dari sebuah _closure_ bisa ngelakuin mana aja dari hal-hal berikut: 
memindahkan nilai yang ditangkap ke luar dari _closure_, memutasi (mengubah) 
nilai yang ditangkap, tidak memindahkan atau memutasi nilainya, atau dari awal 
emang tidak menangkap apa pun dari lingkungannya.

Cara sebuah _closure_ menangkap dan menangani nilai dari lingkungannya 
memengaruhi trait mana yang diimplementasikan oleh _closure_ tersebut, dan 
traits adalah cara bagaimana fungsi dan struct bisa menentukan jenis _closures_ 
apa yang bisa mereka terima. _Closures_ bakal secara otomatis mengimplementasikan 
satu, dua, atau ketiga traits `Fn` ini secara aditif, tergantung dari gimana isi 
_closure_-nya menangani nilai-nilai tersebut:

* `FnOnce` berlaku untuk _closures_ yang bisa dipanggil sekali saja. Semua 
  _closures_ minimal mengimplementasikan trait ini karena semua _closures_ itu 
  bisa dipanggil. _Closure_ yang memindahkan nilai yang ditangkap keluar dari 
  isi kodenya cuma bakal mengimplementasikan `FnOnce` dan bukan trait `Fn` 
  lainnya karena ia cuma bisa dipanggil satu kali.
* `FnMut` berlaku untuk _closures_ yang tidak memindahkan nilai yang ditangkap 
  keluar dari isinya, tapi yang mungkin memutasi nilai-nilai tersebut. _Closures_ 
  jenis ini bisa dipanggil lebih dari sekali.
* `Fn` berlaku untuk _closures_ yang tidak memindahkan nilai yang ditangkap keluar 
  dari isinya dan tidak memutasi nilai-nilai tersebut, serta berlaku juga buat 
  _closures_ yang memang tidak menangkap apa pun dari lingkungannya. _Closures_ 
  seperti ini bisa dipanggil lebih dari sekali tanpa memutasi lingkungannya, 
  yang mana ini penting buat kasus-kasus seperti saat kita memanggil sebuah 
  _closure_ berkali-kali secara konruen (bersamaan).

Mari kita lihat definisi dari method `unwrap_or_else` pada `Option<T>` yang 
kita pakai di Listing 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Ingat kembali bahwa `T` adalah tipe generik yang merepresentasikan tipe dari 
nilai di dalam varian `Some` milik sebuah `Option`. Tipe `T` itu juga adalah 
tipe kembalian (return type) dari fungsi `unwrap_or_else`: kode yang memanggil 
`unwrap_or_else` pada sebuah `Option<String>`, misalnya, bakal mendapatkan sebuah 
`String`.

Selanjutnya, perhatikan bahwa fungsi `unwrap_or_else` juga punya parameter tipe 
generik tambahan `F`. Tipe `F` adalah tipe dari parameter bernama `f`, yang 
mana itu adalah _closure_ yang kita kasih saat memanggil `unwrap_or_else`.

Trait bound yang ditentukan pada tipe generik `F` adalah `FnOnce() -> T`, yang 
berarti `F` harus bisa dipanggil sekali, tidak menerima argumen apa pun, dan 
mengembalikan sebuah `T`. Memakai `FnOnce` di trait bound ini mengekspresikan 
batasan bahwa `unwrap_or_else` cuma bakal memanggil `f` maksimal satu kali saja. 
Di dalam isi `unwrap_or_else`, kita bisa melihat kalau `Option`-nya itu `Some`, 
`f` tidak bakal dipanggil. Kalau `Option`-nya itu `None`, `f` bakal dipanggil 
sekali. Karena semua _closures_ mengimplementasikan `FnOnce`, `unwrap_or_else` 
bisa menerima ketiga jenis _closures_ dan dia fleksibel sekali.

> Catatan: Kalau hal yang mau kita lakukan tidak mewajibkan kita menangkap 
> nilai dari lingkungannya, kita bisa aja memakai nama sebuah fungsi sebagai 
> ganti dari _closure_ di tempat kita butuh sesuatu yang mengimplementasikan 
> salah satu dari trait `Fn`. Contohnya, pada nilai `Option<Vec<T>>`, kita bisa 
> manggil `unwrap_or_else(Vec::new)` buat mendapatkan vector baru yang kosong 
> kalau nilainya adalah `None`. _Compiler_ bakal secara otomatis 
> mengimplementasikan trait `Fn` yang paling pas buat definisi fungsi tersebut.

Sekarang mari kita lihat method _standard library_ `sort_by_key`, yang 
didefinisikan pada slices, buat melihat bedanya dari `unwrap_or_else` dan 
kenapa `sort_by_key` memakai `FnMut` dan bukannya `FnOnce` buat trait bound-nya. 
_Closure_ ini menerima satu argumen berupa referensi ke item saat ini yang ada 
di slice yang lagi diperiksa, lalu mengembalikan nilai bertipe `K` yang bisa 
diurutkan (ordered). Fungsi ini berguna pas kita mau mengurutkan sebuah slice 
berdasarkan atribut spesifik dari setiap itemnya. Di Listing 13-7, kita punya 
daftar berisi instance `Rectangle` dan kita memakai `sort_by_key` buat mengurutkan 
mereka berdasarkan atribut `width` (lebar) mereka dari yang terkecil sampai 
terbesar.

<Listing number="13-7" file-name="src/main.rs" caption="Memakai `sort_by_key` buat mengurutkan persegi panjang berdasarkan lebarnya">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Kode ini mencetak:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

Alasan kenapa `sort_by_key` didefinisikan buat menerima _closure_ `FnMut` adalah 
karena method itu memanggil _closure_-nya berkali-kali: satu kali untuk setiap 
item di slice-nya. _Closure_ `|r| r.width` tidak menangkap, memutasi, atau 
memindahkan apa pun keluar dari lingkungannya, jadi ia memenuhi syarat trait 
bound-nya.

Sebaliknya, Listing 13-8 menunjukkan contoh _closure_ yang cuma mengimplementasikan 
trait `FnOnce`, karena dia memindahkan sebuah nilai ke luar dari lingkungannya. 
_Compiler_ tidak bakal ngebiarin kita pakai _closure_ ini dengan `sort_by_key`.

<Listing number="13-8" file-name="src/main.rs" caption="Mencoba memakai _closure_ `FnOnce` bersama `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Ini adalah cara aneh yang dibikin-bikin (dan tidak bisa jalan) buat mencoba 
menghitung berapa kali `sort_by_key` memanggil _closure_-nya saat ia mengurutkan 
`list`. Kode ini mencoba melakukan penghitungan ini dengan memasukkan (pushing) 
`value`—sebuah `String` dari lingkungan _closure_-nya—ke dalam vector 
`sort_operations`. _Closure_ ini menangkap `value` lalu memindahkan `value` 
tersebut keluar dari _closure_ dengan mentransfer kepemilikan (ownership) 
`value` ke vector `sort_operations`. _Closure_ ini bisa dipanggil satu kali; 
mencoba memanggilnya untuk kedua kalinya tidak bakal berhasil karena `value` 
sudah tidak ada lagi di lingkungan _closure_-nya buat dimasukkan ke dalam 
`sort_operations`! Maka dari itu, _closure_ ini cuma mengimplementasikan 
`FnOnce`. Pas kita mencoba men-compile kode ini, kita bakal dapat error yang 
bilang kalau `value` tidak bisa dipindahkan keluar dari _closure_ karena 
_closure_-nya harus mengimplementasikan `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Error ini menunjuk ke baris di dalam isi _closure_ yang memindahkan `value` 
keluar dari lingkungannya. Buat memperbaikinya, kita harus mengubah isi _closure_ 
agar ia tidak memindahkan nilai-nilai keluar dari lingkungannya. Menyimpan sebuah 
variabel _counter_ di lingkungan dan menambah nilainya dari dalam _closure_ adalah 
cara yang jauh lebih masuk akal buat menghitung berapa kali _closure_ tersebut 
dipanggil. _Closure_ di Listing 13-9 berhasil jalan dengan `sort_by_key` karena 
ia cuma menangkap referensi _mutable_ ke _counter_ `num_sort_operations` dan 
makanya bisa dipanggil lebih dari sekali:

<Listing number="13-9" file-name="src/main.rs" caption="Memakai _closure_ `FnMut` bersama `sort_by_key` diperbolehkan">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

Traits `Fn` penting pas kita mendefinisikan atau memakai fungsi atau tipe yang 
memakai _closures_. Di bagian selanjutnya, kita bakal membahas _iterators_. Banyak 
method iterator yang menerima argumen _closure_, jadi tetap ingat detail-detail 
tentang _closures_ ini ya saat kita lanjut belajar!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else