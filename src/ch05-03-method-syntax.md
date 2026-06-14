## Sintaks Method

_Methods_ itu mirip sama fungsi: kita mendeklarasikan mereka pake keyword `fn` 
sama sebuah nama, mereka bisa punya parameter sama nilai return, dan mereka 
isinya sejumlah kode yang dijalanin pas method-nya dipanggil dari tempat lain. 
Beda sama fungsi, method didefinisikan di dalem konteks sebuah struct (atau enum 
atau trait object, yang bakal kita bahas masing-masing di [Bab 6][enums] sama 
[Bab 18][trait-objects]), dan parameter pertamanya selalu `self`, yang 
merepresentasikan instance dari struct tempat method itu dipanggil.

### Mendefinisikan Methods

Yuk kita ubah fungsi `area` yang punya instance `Rectangle` sebagai parameter 
terus dijadiin sebuah method `area` yang didefinisikan pada struct `Rectangle`, 
kayak yang ditunjukin di Listing 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Mendefinisikan method `area` pada struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Buat mendefinisikan fungsi di dalem konteks `Rectangle`, kita mulai blok 
`impl` (implementasi) buat `Rectangle`. Segala hal di dalem blok `impl` ini 
bakal terkait sama tipe `Rectangle`. Terus kita pindahin fungsi `area` ke dalem 
kurung kurawal `impl` dan ngubah parameter pertama (dan di kasus ini, satu-
satunya parameter) jadi `self` di signature sama di mana-mana di dalem body-nya. 
Di `main`, tempat kita manggil fungsi `area` terus masukin `rect1` sebagai 
argumen, kita sekarang bisa pake _sintaks method_ buat manggil method `area` 
pada instance `Rectangle` kita. Sintaks method ditaruh setelah sebuah instance: 
kita tambahin titik diikuti sama nama method, tanda kurung, sama argumen apa pun.

Di signature buat `area`, kita pake `&self` bukannya `rectangle: &Rectangle`. 
`&self` sebenernya singkatan dari `self: &Self`. Di dalem blok `impl`, tipe 
`Self` adalah alias buat tipe yang lagi diimplementasikan sama blok `impl` itu. 
Methods harus punya parameter namanya `self` bertipe `Self` buat parameter 
pertama mereka, jadi Rust ngebolehin kita nyingkat ini dengan cuma nama `self` 
di tempat parameter pertama. Perhatiin ya kalau kita tetep perlu pake `&` di 
depan singkatan `self` buat nunjukin kalau method ini minjem (_borrows_) instance 
`Self`, sama kayak pas kita nulis `rectangle: &Rectangle`. Methods bisa ngambil 
_ownership_ dari `self`, minjem `self` secara _immutable_, kayak yang kita 
lakuin di sini, atau minjem `self` secara _mutable_, sama kayak parameter 
lainnya.

Kita milih `&self` di sini dengan alasan yang sama kayak kenapa kita pake 
`&Rectangle` di versi fungsinya: kita nggak mau ngambil _ownership_, dan kita 
cuma mau baca data di struct-nya, bukan nulis ke sana. Kalau kita mau ngerubah 
instance yang kita panggil method-nya sebagai bagian dari apa yang dilakuin 
method-nya, kita bakal pake `&mut self` sebagai parameter pertamanya. Punya 
method yang ngambil _ownership_ dari instance dengan cuma pake `self` sebagai 
parameter pertama itu jarang; teknik ini biasanya dipake pas method-nya ngerubah 
(`transform`) `self` jadi sesuatu yang lain terus kita mau nyegah pemanggilnya 
buat pake instance aslinya setelah transformasi itu.

Alasan utama buat pake method bukannya fungsi, selain ngasih sintaks method 
dan nggak perlu ngulang-ngulang nulis tipe `self` di tiap signature method, 
adalah buat pengaturan kode (organization). Kita naruh semua hal yang bisa kita 
lakuin sama sebuah instance dari suatu tipe di dalem satu blok `impl` bukannya 
bikin orang yang nanti pake kode kita harus nyari-nyari kemampuan dari 
`Rectangle` di berbagai tempat di library yang kita kasih.

Perhatiin ya kalau kita bisa milih buat ngasih nama method sama kayak salah 
satu nama field struct-nya. Misalnya, kita bisa mendefinisikan method di 
`Rectangle` yang juga namanya `width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Di sini, kita milih buat bikin method `width` balikin `true` kalau nilai di 
field `width` dari instance-nya lebih gede dari `0` dan `false` kalau nilainya 
`0`: kita bisa pake field di dalem method dengan nama yang sama buat tujuan apa 
pun. Di `main`, pas kita ngikutin `rect1.width` pake tanda kurung, Rust tau 
kita maksudnya method `width`. Pas kita nggak pake tanda kurung, Rust tau 
maksudnya field `width`.

Sering kali, tapi nggak selalu, pas kita ngasih method nama yang sama kayak 
sebuah field, kita mau method itu cuma balikin nilai di field-nya dan nggak 
ngelakuin hal lain. Method kayak gini namanya _getters_, dan Rust nggak 
mengimplementasikan mereka secara otomatis buat field struct kayak yang 
dilakuin beberapa bahasa lain. Getters itu berguna karena kita bisa bikin 
field-nya jadi _private_ tapi method-nya _public_, dan dengan gitu ngasih akses 
_read-only_ ke field itu sebagai bagian dari API _public_ tipe tersebut. Kita 
bakal bahas apa itu _public_ dan _private_ dan gimana cara nandain field atau 
method sebagai _public_ atau _private_ di [Bab 7][public].

> ### Ke Mana Perginya Operator `->`?
>
> Di C sama C++, dua operator yang beda dipake buat manggil method: kita pake 
> `.` kalau kita manggil method di objeknya secara langsung dan `->` kalau 
> kita manggil method di sebuah _pointer_ ke objeknya dan perlu nge-_dereference_ 
> _pointer_-nya dulu. Dengan kata lain, kalau `object` itu sebuah _pointer_, 
> `object->something()` itu mirip sama `(*object).something()`.
>
> Rust nggak punya padanan buat operator `->`; sebaliknya, Rust punya fitur 
> namanya _automatic referencing and dereferencing_ (referencing dan 
> dereferencing otomatis). Manggil method adalah salah satu dari sedikit tempat 
> di Rust yang punya perilaku ini.
>
> Ini cara kerjanya: pas kita manggil sebuah method pake `object.something()`, 
> Rust secara otomatis nambahin `&`, `&mut`, atau `*` biar `object` cocok sama 
> signature dari method-nya. Dengan kata lain, dua baris berikut itu sama aja:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Yang pertama keliatan jauh lebih bersih. Perilaku _automatic referencing_ ini 
> bisa jalan karena method punya penerima (_receiver_) yang jelas—yaitu tipe dari 
> `self`. Berdasarkan penerima dan nama method-nya, Rust bisa tau secara 
> definitif apakah method itu lagi baca (`&self`), nge-mutasi (`&mut self`), 
> atau ngonsumsi (`self`). Fakta kalau Rust bikin _borrowing_ jadi implisit buat 
> penerima method adalah bagian gede dari kenapa _ownership_ terasa ergonomis 
> di praktiknya.

### Methods dengan Lebih Banyak Parameter

Yuk kita latihan pake method dengan mengimplementasikan method kedua di struct 
`Rectangle`. Kali ini kita mau sebuah instance `Rectangle` nerima instance 
`Rectangle` lainnya dan balikin `true` kalau `Rectangle` yang kedua bisa muat 
sepenuhnya di dalem `self` (`Rectangle` yang pertama); kalau nggak, dia harus 
balikin `false`. Jadi, setelah kita mendefinisikan method `can_hold`, kita mau 
bisa nulis program kayak yang ditunjukin di Listing 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Pake method `can_hold` yang belum ditulis">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Output yang diharepin bakal keliatan kayak gini karena kedua dimensi `rect2` 
itu lebih kecil dari dimensi `rect1`, tapi `rect3` lebih lebar dari `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Kita tau kita mau mendefinisikan sebuah method, jadi dia bakal ada di dalem blok 
`impl Rectangle`. Nama method-nya adalah `can_hold`, dan dia bakal nerima 
_immutable borrow_ dari `Rectangle` lainnya sebagai parameter. Kita bisa tau apa 
tipe parameternya dengan ngeliat kode yang manggil method-nya: 
`rect1.can_hold(&rect2)` masukin `&rect2`, yang merupakan _immutable borrow_ ke 
`rect2`, sebuah instance dari `Rectangle`. Ini masuk akal karena kita cuma 
perlu baca `rect2` (bukannya nulis, yang bakal berarti kita butuh _mutable 
borrow_), dan kita mau `main` tetep punya _ownership_ dari `rect2` biar kita 
bisa pake lagi setelah manggil method `can_hold`. Nilai return dari `can_hold` 
bakal berupa Boolean, dan implementasinya bakal nge-cek apakah lebar sama tinggi 
dari `self` lebih gede dari lebar sama tinggi dari `Rectangle` yang satunya. 
Yuk kita tambahin method `can_hold` baru ini ke blok `impl` dari Listing 5-13, 
yang ditunjukin di Listing 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Mengimplementasikan method `can_hold` di `Rectangle` yang nerima instance `Rectangle` lainnya sebagai parameter">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Pas kita jalanin kode ini sama fungsi `main` di Listing 5-14, kita bakal dapet 
output yang kita mau. Method bisa nerima banyak parameter yang kita tambahin di 
signature setelah parameter `self`, dan parameter-parameter itu cara kerjanya 
persis sama kayak parameter di fungsi biasa.

### Associated Functions

Semua fungsi yang didefinisikan di dalem blok `impl` disebut _associated functions_ 
(fungsi terkait) karena mereka terkait sama tipe yang dinamain setelah kata 
`impl`. Kita bisa mendefinisikan _associated functions_ yang nggak punya `self` 
sebagai parameter pertamanya (dan makanya bukan methods) karena mereka nggak 
butuh instance dari tipe itu buat jalan. Kita udah pake salah satu fungsi kayak 
gini: fungsi `String::from` yang didefinisikan pada tipe `String`.

_Associated functions_ yang bukan methods sering dipake buat _constructors_ yang 
bakal balikin instance baru dari struct-nya. Ini sering dikasih nama `new`, tapi 
`new` bukan nama khusus dan nggak bawaan dari bahasanya. Misalnya, kita bisa 
milih buat nyediain _associated function_ namanya `square` yang punya satu 
parameter dimensi dan pakenya buat lebar sama tingginya, jadi lebih gampang 
buat bikin `Rectangle` bentuk persegi bukannya harus nentuin nilai yang sama dua 
kali:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Keyword `Self` di tipe return sama di dalem body fungsinya itu alias buat tipe 
yang muncul setelah keyword `impl`, yang di kasus ini adalah `Rectangle`.

Buat manggil _associated function_ ini, kita pake sintaks `::` bareng nama 
struct-nya; `let sq = Rectangle::square(3);` adalah contohnya. Fungsi ini punya 
_namespace_ oleh struct-nya: sintaks `::` dipake buat baik _associated functions_ 
maupun _namespaces_ yang dibuat sama modul. Kita bakal bahas modul di [Bab 7][modules].

### Banyak Blok `impl`

Tiap struct dibolehin buat punya banyak blok `impl`. Contohnya, Listing 5-15 
itu ekuivalen sama kode yang ditunjukin di Listing 5-16, yang punya tiap 
method di blok `impl`-nya masing-masing.

<Listing number="5-16" caption="Nulis ulang Listing 5-15 pake banyak blok `impl`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Nggak ada alesan khusus buat misahin method-method ini ke dalem banyak blok 
`impl` di sini, tapi ini sintaks yang valid. Kita bakal liat kasus di mana 
banyak blok `impl` berguna di Bab 10, pas kita bahas soal _generic types_ dan 
_traits_.

## Ringkasan

Structs ngebolehin kita bikin tipe kustom yang bermakna buat domain kita. Dengan 
pake struct, kita bisa nyimpen potongan data yang terkait tetep nyambung satu 
sama lain dan ngasih nama ke tiap potongannya buat bikin kode kita jelas. Di 
dalem blok `impl`, kita bisa mendefinisikan fungsi-fungsi yang terkait sama 
tipe kita, dan methods adalah jenis _associated function_ yang ngebolehin kita 
nentuin perilaku yang dimiliki sama instance dari struct kita.

Tapi struct bukan satu-satunya cara kita bisa bikin tipe kustom: yuk kita 
beralih ke fitur enum di Rust buat nambahin _tool_ lain ke _toolbox_ kita.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
