## Referensi dan Borrowing

Masalah dari kode tuple di Listing 4-5 adalah kita harus balikin `String`-nya ke 
fungsi pemanggil biar kita tetep bisa pake `String`-nya setelah manggil 
`calculate_length`, soalnya `String`-nya udah di-_move_ ke dalem 
`calculate_length`. Sebagai gantinya, kita bisa ngasih sebuah referensi ke nilai 
`String` itu. Sebuah _reference_ (referensi) itu kayak _pointer_ karena dia 
adalah alamat yang bisa kita ikutin buat akses data yang disimpan di alamat itu; 
data itu dimiliki sama variabel lain. Beda sama _pointer_, sebuah referensi 
dijamin bakal nunjuk ke sebuah nilai yang valid dari tipe tertentu selama masa 
hidup referensi itu.

Ini cara kita mendefinisikan dan pake fungsi `calculate_length` yang punya 
referensi ke sebuah objek sebagai parameter bukannya ngambil _ownership_ dari 
nilainya:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

Pertama, perhatiin kalau semua kode tuple di deklarasi variabel sama nilai 
return fungsi udah nggak ada. Kedua, perhatiin kalau kita masukin `&s1` ke 
`calculate_length` dan, di definisinya, kita nerima `&String` bukannya `String`. 
Tanda ampersand ini merepresentasikan _references_, dan mereka ngebolehin kita 
buat ngerujuk ke suatu nilai tanpa ngambil _ownership_-nya. Gambar 4-6 
ngeliatin konsep ini.

<img alt="Tiga tabel: tabel buat s isinya cuma pointer ke tabel buat s1. Tabel 
buat s1 isinya data stack buat s1 dan nunjuk ke data string di heap." 
src="img/trpl04-06.svg" class="center" />

<span class="caption">Gambar 4-6: Diagram `&String s` yang nunjuk ke `String s1`</span>

> Catatan: Kebalikan dari bikin referensi pake `&` adalah _dereferencing_, yang 
> dilakuin pake operator dereference, `*`. Kita bakal liat beberapa penggunaan 
> operator dereference di Bab 8 dan bahas detail soal dereferencing di Bab 15.

Yuk kita liat lebih deket pemanggilan fungsinya di sini:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

Sintaks `&s1` ngebolehin kita bikin sebuah referensi yang _ngerujuk_ ke nilai 
dari `s1` tapi nggak memilikinya. Karena referensinya nggak memiliki nilainya, 
nilai yang dia tunjuk nggak bakal di-_drop_ pas referensinya udah nggak dipake 
lagi.

Begitu juga sama signature fungsinya yang pake `&` buat nunjukin kalau tipe 
parameternya `s` itu adalah sebuah referensi. Yuk kita tambahin beberapa 
anotasi penjelasan:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

Scope di mana variabel `s` itu valid sama kayak scope parameter fungsi mana pun, 
tapi nilai yang ditunjuk sama referensinya nggak bakal di-_drop_ pas `s` udah 
nggak dipake lagi, karena `s` nggak punya _ownership_. Pas fungsi punya 
referensi sebagai parameter bukannya nilai aslinya, kita nggak perlu balikin 
nilai-nilainya buat ngasih balik _ownership_, karena emang kita nggak pernah 
punya _ownership_-nya dari awal.

Kita sebut aksi bikin referensi ini sebagai _borrowing_ (meminjam). Kayak di 
dunia nyata, kalau seseorang punya sesuatu, kita bisa pinjem dari mereka. Pas 
udah selese, kita harus balikin. Kita nggak memilikinya.

Jadi, apa yang terjadi kalau kita nyoba ngerubah sesuatu yang kita pinjem? Coba 
kode di Listing 4-6. Spoiler: kodenya nggak bakal jalan!

<Listing number="4-6" file-name="src/main.rs" caption="Nyoba ngerubah nilai yang dipinjem">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

Ini error-nya:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Sama kayak variabel yang _immutable_ secara default, referensi juga gitu. Kita 
nggak dibolehin ngerubah sesuatu yang kita punya referensinya.

### Mutable References

Kita bisa benerin kode dari Listing 4-6 biar kita dibolehin ngerubah nilai yang 
dipinjem dengan cuma beberapa perubahan kecil yang pake _mutable reference_ 
sebagai gantinya:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

Pertama kita ubah `s` jadi `mut`. Terus kita bikin sebuah mutable reference 
pake `&mut s` pas kita manggil fungsi `change`, terus update signature fungsinya 
biar nerima sebuah mutable reference pake `some_string: &mut String`. Ini 
bikin keliatan jelas sekali kalau fungsi `change` bakal ngerubah (_mutate_) 
nilai yang dia pinjem.

Mutable references punya satu larangan gede: kalau kita punya sebuah mutable 
reference ke sebuah nilai, kita nggak boleh punya referensi lain ke nilai itu. 
Kode ini yang nyoba bikin dua mutable reference ke `s` bakal gagal:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

Ini error-nya:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Error ini bilang kalau kodenya nggak valid karena kita nggak bisa minjem `s` 
sebagai mutable lebih dari sekali dalam satu waktu. _Mutable borrow_ yang 
pertama ada di `r1` dan harus bertahan sampe dia dipake di `println!`, tapi di 
antara pembuatan mutable reference itu sampe penggunaannya, kita nyoba bikin 
mutable reference lain di `r2` yang minjem data yang sama kayak `r1`.

Larangan yang nyegah banyak mutable reference ke data yang sama di waktu yang 
bersamaan ini ngebolehin adanya mutasi tapi dengan cara yang sangat terkontrol. 
Ini hal yang biasanya bikin Rustacean baru rada pusing karena kebanyakan bahasa 
ngebolehin kita ngerubah nilai kapan pun kita mau. Keuntungan punya larangan 
ini adalah Rust bisa nyegah _data races_ pas _compile time_. Sebuah _data race_ 
itu mirip kayak _race condition_ dan terjadi pas tiga perilaku ini muncul:

- Dua atau lebih _pointer_ akses data yang sama di waktu yang sama.
- Minimal salah satu dari _pointer_-nya dipake buat nulis ke datanya.
- Nggak ada mekanisme yang dipake buat sinkronisasi akses ke datanya.

_Data races_ bikin perilaku yang nggak terdefinisi (undefined behavior) dan bisa 
susah buat didiagnosa dan diperbaiki pas kita nyoba nyari tau pas _runtime_; 
Rust nyegah masalah ini dengan nolak buat nge-compile kode yang punya _data 
races_!

Kayak biasa, kita bisa pake kurung kurawal buat bikin scope baru, yang 
ngebolehin adanya banyak mutable reference, cuma bukan yang _bersamaan_:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust juga nerapin aturan yang mirip buat ngelempokin mutable sama immutable 
references. Kode ini ngasilin error:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Ini error-nya:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Fiuh! Kita _juga_ nggak bisa punya sebuah mutable reference pas kita lagi punya 
sebuah immutable reference ke nilai yang sama.

User dari sebuah immutable reference nggak bakal nyangka kalau nilainya tiba-
tiba berubah gitu aja! Tapi, banyak immutable references diperbolehkan karena 
nggak ada orang yang cuma baca datanya punya kemampuan buat ngaruhin bacaan 
data orang lain.

Perhatiin ya kalau scope sebuah referensi dimulai dari tempat dia dikenalin 
sampe terakhir kali referensi itu dipake. Misalnya, kode ini bakal ke-compile 
karena penggunaan terakhir dari immutable references ada di `println!`, sebelum 
mutable reference-nya dikenalin:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Scope dari immutable references `r1` sama `r2` abis setelah `println!` di mana 
mereka terakhir dipake, yang mana itu sebelum mutable reference `r3` dibuat. 
Scope-scope ini nggak tumpang tindih, jadi kode ini diperbolehkan: _compiler_ 
bisa tau kalau referensinya udah nggak dipake lagi di titik sebelum akhir dari 
scope-nya.

Walaupun error _borrowing_ kadang bikin kesel, inget ya kalau itu adalah 
_compiler_ Rust yang lagi nunjukin potensi _bug_ dari awal (pas _compile time_ 
bukannya pas _runtime_) dan nunjukin tepat di mana letak masalahnya. Jadi kita 
nggak perlu repot-repot nyari tau kenapa data kita nggak sesuai sama apa yang 
kita pikirkan.

### Dangling References

Di bahasa yang punya _pointer_, gampang sekali buat nggak sengaja bikin sebuah 
_dangling pointer_—sebuah _pointer_ yang ngerujuk ke sebuah lokasi di memori 
yang mungkin udah dikasih ke orang lain—dengan cara ngebebasin sejumlah memori 
tapi tetep nyimpen _pointer_ ke memori itu. Di Rust, sebaliknya, _compiler_ 
ngejamin kalau referensi nggak bakal pernah jadi _dangling references_: kalau 
kita punya sebuah referensi ke suatu data, _compiler_ bakal mastiin kalau 
datanya nggak bakal keluar dari scope sebelum referensi ke datanya keluar duluan.

Yuk kita coba bikin sebuah _dangling reference_ buat liat gimana Rust nyegah 
mereka pake _compile-time error_:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

Ini error-nya:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Pesan error ini ngerujuk ke fitur yang belum kita bahas: _lifetimes_. Kita bakal 
bahas _lifetimes_ secara detail di Bab 10. Tapi, kalau kita cuekin bagian soal 
_lifetimes_-nya, pesannya emang isinya kunci kenapa kode ini bermasalah:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

Yuk kita liat lebih deket apa sebenernya yang terjadi di tiap tahap kode 
`dangle` kita:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Karena `s` dibuat di dalem `dangle`, pas kode `dangle` selesai, `s` bakal 
di-dealokasi. Tapi kita nyoba buat balikin sebuah referensi kepadanya. Itu 
artinya referensi ini bakal nunjuk ke sebuah `String` yang nggak valid. Itu 
nggak oke sekali! Rust nggak bakal ngebolehin kita ngelakuin ini.

Solusinya di sini adalah dengan balikin `String`-nya secara langsung:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Ini jalan tanpa masalah apa pun. _Ownership_ di-_move_ keluar, dan nggak ada 
apa pun yang di-dealokasi.

### Aturan Referensi

Yuk kita ringkas apa yang udah kita bahas soal referensi:

- Dalam satu waktu, kita bisa punya _antara_ satu mutable reference _atau_ 
  sejumlah berapa pun immutable references.
- Referensi harus selalu valid.

Selanjutnya, kita bakal liat jenis referensi yang beda: _slices_.
