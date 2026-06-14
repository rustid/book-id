## Advanced Functions (Fungsi Tingkat Lanjut) dan Closures

Bagian ini mengeksplorasi beberapa fitur tingkat lanjut yang berkaitan dengan 
fungsi dan _closures_, termasuk _function pointers_ (pointer fungsi) dan 
mengembalikan _closures_.

### Function Pointers

Kita udah ngebahas gimana caranya mengoper _closures_ ke fungsi-fungsi; kita juga 
bisa mengoper fungsi biasa ke fungsi-fungsi lho! Teknik ini sangat berguna pas 
kita mau ngoper fungsi yang emang udah kita definisikan sebelumnya ketimbang 
harus ngebikin _closure_ baru. Fungsi itu bisa dipaksa (_coerce_) menjadi 
tipe `fn` (dengan huruf _f_ kecil), yang mana jangan sampai tertukar sama trait 
_closure_ `Fn`. Tipe `fn` ini disebut sebagai _function pointer_. Mengoper fungsi 
memakai _function pointers_ bakal memungkinkan kita buat memakai fungsi sebagai 
argumen buat fungsi yang lainnya.

Sintaks buat menentukan kalau sebuah parameter itu adalah _function pointer_ itu 
mirip sama sintaks _closures_, kayak yang ditunjukin di Listing 20-28, di mana kita 
udah mendefinisikan sebuah fungsi `add_one` yang menjumlahkan 1 ke parameternya. 
Fungsi `do_twice` menerima dua parameter: sebuah _function pointer_ ke fungsi mana aja 
yang nerima parameter `i32` dan ngembaliin sebuah `i32`, dan satu nilai `i32`. 
Fungsi `do_twice` ini memanggil fungsi `f` sebanyak dua kali, mengoper nilai `arg` ke 
dalamnya, lalu menjumlahkan kedua hasil pemanggilan fungsi tersebut bersama-sama. 
Fungsi `main` lalu memanggil `do_twice` dengan argumen `add_one` dan `5`.

<Listing number="20-28" file-name="src/main.rs" caption="Memakai tipe `fn` buat nerima _function pointer_ sebagai argumen">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

Kode ini mencetak `The answer is: 12`. Kita menentukan kalau parameter `f` di 
dalam `do_twice` itu adalah sebuah `fn` yang nerima satu parameter bertipe `i32` 
dan mengembalikan sebuah `i32`. Terus kita bisa manggil `f` di dalam isi (_body_) 
dari `do_twice`. Di `main`, kita bisa mengoper nama fungsi `add_one` sebagai 
argumen pertama ke `do_twice`.

Beda sama _closures_, `fn` itu adalah sebuah tipe bukannya trait, jadi kita 
menentukan `fn` sebagai tipe parameternya secara langsung ketimbang harus 
mendeklarasikan sebuah parameter tipe generik yang mana trait _bounds_-nya 
memakai salah satu dari trait `Fn`.

_Function pointers_ mengimplementasikan ketiga trait _closure_ sekaligus 
(`Fn`, `FnMut`, dan `FnOnce`), yang berarti kita bakal selalu bisa mengoper sebuah 
_function pointer_ sebagai argumen buat fungsi yang membutuhkan sebuah _closure_. 
Praktik terbaiknya adalah nulis fungsi memakai tipe generik dan salah satu dari 
trait-trait _closure_ tersebut supaya fungsi kita bisa nerima fungsi biasa maupun 
_closures_.

Namun, ada satu contoh di mana kita mungkin cuma mau nerima tipe `fn` aja dan 
tidak mau nerima _closures_, yaitu pas kita lagi berinteraksi (interfacing) 
dengan kode eksternal yang emang tidak punya _closures_: Fungsi-fungsi C bisa 
menerima fungsi biasa sebagai argumen, tapi C tidak punya fitur _closures_.

Sebagai contoh buat situasi di mana kita bisa memakai sebuah _closure_ yang 
didefinisikan secara langsung (inline) atau sebuah fungsi bernama, mari kita 
lihat penggunaan method `map` yang disediain sama trait `Iterator` di _standard 
library_. Buat memakai method `map` buat mengubah sebuah _vector_ angka-angka 
jadi _vector_ string-string, kita bisa memakai _closure_, kayak di Listing 20-29.

<Listing number="20-29" caption="Memakai sebuah closure dengan method `map` buat mengubah angka jadi string">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

Atau kita juga bisa memakai fungsi bernama sebagai argumen buat method `map` 
ketimbang memakai sebuah _closure_. Listing 20-30 nunjukin bakal seperti apa 
kelihatannya kalau pakai cara ini.

<Listing number="20-30" caption="Memakai fungsi `String::to_string` bareng method `map` buat mengubah angka jadi string">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

Perhatikan bahwa kita wajib memakai _fully qualified syntax_ (sintaks terkualifikasi 
penuh) yang udah kita obrolin di [“Advanced Traits”][advanced-traits] karena 
ada banyak fungsi tersedia yang namanya sama-sama `to_string`.

Di sini, kita memakai fungsi `to_string` yang didefinisikan di dalam trait 
`ToString`, yang mana udah diimplementasikan secara otomatis sama _standard 
library_ buat tipe apa aja yang mengimplementasikan trait `Display`.

Ingat kembali dari [“Nilai Enum (Enum Values)”][enum-values] di Bab 6 kalau nama 
dari setiap varian enum yang kita definisikan itu juga bertindak sebagai sebuah 
fungsi inisialisasi (initializer function). Kita bisa memakai fungsi-fungsi inisialisasi 
ini sebagai _function pointers_ yang mengimplementasikan trait-trait _closure_, yang 
berarti kita bisa mengoper fungsi inisialisasi tersebut sebagai argumen buat method-method 
yang nerima _closures_, kayak yang kelihatan di Listing 20-31.

<Listing number="20-31" caption="Memakai fungsi inisialisasi enum bareng method `map` buat ngebikin instance `Status` dari angka-angka">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

Di sini, kita ngebikin instance-instance `Status::Value` memakai setiap nilai `u32` 
di dalam rentang yang mana dipanggil oleh method `map` dengan jalan memakai 
fungsi inisialisasi dari varian `Status::Value` tersebut. Beberapa orang lebih 
suka gaya penulisan kayak gini dan beberapa orang lainnya lebih milih buat pakai 
_closures_. Mereka di-compile jadi hasil kode yang sama kok, jadi pakai aja gaya mana 
yang lebih jelas dan enak dibaca buat kita.

### Mengembalikan Closures

_Closures_ direpresentasikan memakai traits, yang artinya kita tidak bisa ngembaliin 
_closures_ secara langsung. Di sebagian besar kasus di mana kita mungkin pengen 
mengembalikan sebuah trait, kita bisa memakai tipe konkret yang emang 
mengimplementasikan trait tersebut sebagai nilai kembalian fungsinya. Tapi, 
biasanya kita tidak bisa ngelakuin itu buat _closures_ karena mereka tidak punya 
tipe konkret yang bisa di-return (dikembalikan); contohnya, kita tidak diizinkan 
buat memakai _function pointer_ `fn` sebagai tipe kembalian kalau _closure_-nya itu 
menangkap (captures) nilai-nilai apa pun dari _scope_-nya.

Sebaliknya, kita biasanya bakal memakai sintaks `impl Trait` yang udah kita 
pelajarin di Bab 10. Kita bisa ngembaliin tipe fungsi apa aja, dengan memakai 
`Fn`, `FnOnce` dan `FnMut`. Contohnya, kode di Listing 20-32 bakal sukses 
di-compile dengan baik.

<Listing number="20-32" caption="Mengembalikan sebuah closure dari sebuah fungsi memakai sintaks `impl Trait`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

Namun, kayak yang udah kita sebutin di [“Inferensi Tipe dan Anotasi pada 
Closure”][closure-types] di Bab 13, masing-masing _closure_ itu juga merupakan tipe 
mereka sendiri yang berbeda-beda. Kalau kita perlu beroperasi sama fungsi-fungsi 
yang punya _signature_ yang sama tapi implementasi yang berbeda-beda, kita perlu 
memakai *trait object* buat mereka. Coba bayangin apa yang terjadi kalau kita nulis 
kode kayak yang ditunjukin di Listing 20-33.

<Listing file-name="src/main.rs" number="20-33" caption="Membikin sebuah `Vec<T>` berisi closures yang didefinisikan sama fungsi-fungsi yang mengembalikan tipe `impl Fn`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

Di sini kita punya dua fungsi, `returns_closure` dan `returns_initialized_closure`, 
yang mana dua-duanya mengembalikan `impl Fn(i32) -> i32`. Perhatikan kalau 
_closures_ yang mereka kembalikan itu berbeda isinya, biarpun mereka 
mengimplementasikan tipe yang sama. Kalau kita mencoba men-compile ini, Rust 
bakal ngasih tahu kita kalau ini tidak bisa jalan:

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

Pesan error ini ngasih tahu kita kalau kapan pun kita mengembalikan `impl Trait`, 
Rust ngebikin tipe _opaque_ (buram/tidak tembus pandang) yang unik, yaitu sebuah 
tipe di mana kita tidak bisa melihat ke dalam detail dari apa yang dibangun sama 
Rust buat kita, dan kita juga tidak bisa nebak-nebak tipe apa yang bakal dihasilkan 
(generated) sama Rust supaya kita bisa nulisin tipenya sendiri secara manual. Jadi 
meskipun fungsi-fungsi ini sama-sama ngembaliin _closures_ yang mengimplementasikan 
trait yang sama, yaitu `Fn(i32) -> i32`, tipe _opaque_ yang dihasilkan oleh Rust 
buat masing-masing fungsinya itu tetap berbeda. (Ini mirip dengan gimana cara 
Rust memproduksi tipe konkret yang berbeda-beda untuk blok _async_ yang berbeda 
bahkan pas mereka punya tipe output yang sama, kayak yang kita lihat di 
[“Bekerja dengan Jumlah Futures yang Sembarang”][any-number-of-futures] di Bab 17.) 
Kita udah pernah lihat solusi buat masalah ini beberapa kali sebelumnya: kita 
bisa memakai sebuah *trait object*, kayak di Listing 20-34.

<Listing number="20-34" caption="Membikin sebuah `Vec<T>` berisi closures yang didefinisikan sama fungsi-fungsi yang mengembalikan `Box<dyn Fn>` supaya mereka punya tipe yang sama">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

Kode ini bakal sukses di-compile dengan mulus. Buat tahu lebih lanjut soal 
_trait objects_, silakan mengacu ke bagian [“Memakai Trait Objects Buat 
Mengabstraksi Perilaku Bersama”][using-trait-objects-to-abstract-over-shared-behavior] 
di Bab 18.

Berikutnya, mari kita lihat tentang macro!

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[any-number-of-futures]: ch17-03-more-futures.html
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
