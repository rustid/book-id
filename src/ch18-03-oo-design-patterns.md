## Mengimplementasikan Desain Pola Object-Oriented

_State pattern_ (pola status) adalah sebuah desain pola _object-oriented_ 
(berorientasi objek). Inti dari pola ini adalah kita mendefinisikan serangkaian 
_states_ (status/keadaan) yang bisa dimiliki oleh suatu nilai di dalamnya 
(internally). _States_ ini direpresentasikan oleh sekumpulan _state objects_, 
dan perilaku dari nilai tersebut berubah berdasarkan _state_ yang dia miliki 
saat itu. Kita bakal mengerjakan contoh berupa struct postingan blog yang punya 
field buat menampung _state_-nya, yang mana bakal berupa _state object_ dari 
serangkaian pilihan: “draft” (draf), “review” (tinjauan), atau “published” 
(dipublikasikan).

Objek-objek _state_ ini saling berbagi fungsionalitas: di Rust, tentu saja, 
kita memakai struct dan traits bukannya objek dan pewarisan (inheritance). 
Setiap _state object_ bertanggung jawab buat perilakunya sendiri dan mengatur 
kapan dia harus berubah jadi _state_ lain. Nilai yang menampung _state object_ 
tersebut sama sekali tidak tahu tentang perilaku yang berbeda dari setiap 
_state_ atau kapan waktu yang tepat buat bertransisi (transition) antar _states_.

Keuntungan dari memakai _state pattern_ ini adalah, saat ada perubahan 
persyaratan bisnis (business requirements) di program kita, kita tidak perlu 
mengubah kode dari nilai yang menampung _state_-nya atau kode yang memakai nilai 
tersebut. Kita cuma perlu meng-update kode di dalam salah satu _state objects_ 
buat ngubah aturan-aturannya atau mungkin nambahin _state objects_ baru.

Pertama-tama kita bakal mengimplementasikan _state pattern_ pakai cara yang lebih 
tradisional ala _object-oriented_, terus kita bakal memakai pendekatan yang 
lebih natural di Rust. Mari kita gali pelan-pelan buat mengimplementasikan 
_workflow_ (alur kerja) postingan blog pakai _state pattern_.

Fungsionalitas akhirnya bakal kelihatan kayak gini:

1. Postingan blog bermula dari _draft_ kosong.
1. Saat _draft_-nya beres, sebuah _review_ dari postingan itu diminta (requested).
1. Saat postingannya disetujui (approved), dia bakal di-_publish_ (dipublikasikan).
1. Cuma postingan blog yang udah di-_publish_ yang mengembalikan konten buat 
   dicetak, jadi postingan yang belum disetujui tidak bakal bisa tidak sengaja 
   ke-_publish_.

Perubahan lain yang dicoba dilakukan pada postingan tersebut seharusnya tidak 
bisa mengubah apa pun. Misalnya, kalau kita mencoba men-_approve_ sebuah 
postingan _draft_ sebelum kita me-_request_ _review_, postingan itu seharusnya 
tetap berupa _draft_ yang belum di-_publish_.

### Percobaan Object-Oriented yang Tradisional

Ada sangat banyak cara buat menata struktur kode buat menyelesaikan masalah yang 
sama, masing-masing dengan *trade-offs* (kekurangan/kelebihan) yang beda. 
Implementasi di bagian ini memakai gaya _object-oriented_ yang lebih tradisional, 
yang mana bisa ditulis di Rust, tapi tidak memanfaatkan beberapa dari kekuatan 
unggulan yang dimiliki Rust. Nanti, kita bakal mendemonstrasikan solusi lain 
yang tetap memakai desain pola _object-oriented_ tapi disusun sedemikian rupa 
hingga mungkin kelihatan kurang familier buat programmer yang punya pengalaman 
_object-oriented_. Kita bakal membandingkan kedua solusi ini buat ngalamin 
langsung _trade-offs_ dari mendesain kode di Rust dengan cara yang beda 
daripada nulis kode di bahasa lain.

Listing 18-11 menunjukkan _workflow_ ini dalam bentuk kode: ini adalah contoh 
pemakaian dari API yang bakal kita implementasikan di sebuah _library crate_ 
bernama `blog`. Ini masih belum bisa di-compile karena kita belum 
mengimplementasikan _crate_ `blog`-nya.

<Listing number="18-11" file-name="src/main.rs" caption="Kode yang mendemonstrasikan perilaku yang kita pengen ada di _crate_ `blog` kita">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Kita mau ngebolehin _user_ bikin postingan blog _draft_ baru pakai `Post::new`. Kita 
mau ngebolehin teks buat ditambahin ke postingan blog itu. Kalau kita nyoba ngambil 
konten (content) dari postingan itu langsung, sebelum adanya *approval*, kita tidak 
seharusnya dapat teks apa pun karena postingannya masih berupa _draft_. Kita udah 
nambahin `assert_eq!` di kode ini buat tujuan demonstrasi aja. Pengujian *unit test* 
yang cakep sekali buat ini adalah dengan menegaskan (assert) kalau postingan blog 
_draft_ mengembalikan string kosong dari method `content`, tapi kita tidak bakal 
nulis _tests_ buat contoh ini.

Berikutnya, kita mau memungkinkan adanya permintaan (request) buat me-_review_ 
postingan tersebut, dan kita pengen method `content` tetap mengembalikan string 
kosong selama masih nungguin _review_. Pas postingannya udah dapat *approval*, 
dia harusnya langsung ke-_publish_, yang berarti teks dari postingan tersebut bakal 
dikembalikan saat method `content` dipanggil.

Perhatikan bahwa satu-satunya tipe yang kita pakai buat berinteraksi dari _crate_ ini 
adalah tipe `Post`. Tipe ini bakal memakai _state pattern_ dan bakal menampung 
sebuah nilai yang merupakan salah satu dari tiga _state objects_ yang mewakili 
berbagai macam _state_ yang mungkin ada pada suatu postingan—_draft_, _review_, 
atau _published_. Perubahan dari satu _state_ ke _state_ lainnya bakal dikelola 
secara internal di dalam tipe `Post`. _States_-nya berubah sebagai respons 
terhadap method-method yang dipanggil sama para pengguna _library_ kita pada 
instance `Post` tersebut, tapi mereka sendiri tidak perlu ngurusin (manage) 
perubahan _state_-nya secara langsung. Selain itu, _user_ juga tidak bisa ngelakuin 
kesalahan pada _states_-nya, kayak nge-_publish_ sebuah postingan sebelum dia 
di-_review_.

#### Mendefinisikan `Post` dan Membikin Instance Baru di State Draft

Mari kita mulai ngimplementasiin _library_-nya! Kita tahu kita butuh sebuah 
struct _public_ `Post` yang menampung beberapa konten, jadi kita bakal mulai dengan 
definisi dari struct tersebut dan fungsi _associated public_ bernama `new` buat 
bikin sebuah instance dari `Post`, seperti yang ditunjukkan di Listing 18-12. Kita 
juga bakal bikin trait _private_ bernama `State` yang bakal mendefinisikan 
perilaku yang wajib dimiliki sama semua _state objects_ buat `Post`.

Terus, `Post` bakal menampung sebuah _trait object_ berupa `Box<dyn State>` di dalam 
sebuah `Option<T>` di sebuah field _private_ bernama `state` buat naruh si 
_state object_. Kita bakal ngelihat kenapa `Option<T>` ini dibutuhkan sebentar lagi.

<Listing number="18-12" file-name="src/lib.rs" caption="Definisi dari struct `Post` dan fungsi `new` yang ngebikin instance `Post` baru, trait `State`, dan struct `Draft`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

Trait `State` mendefinisikan perilaku yang di-share (dibagi) oleh _states_ dari 
postingan yang beda-beda. _State objects_-nya adalah `Draft`, `PendingReview`, 
dan `Published`, dan mereka semua bakal mengimplementasikan trait `State`. Buat 
sekarang, trait-nya belum punya method apa-apa, dan kita bakal mulai dengan cuma 
mendefinisikan _state_ `Draft` aja karena itu adalah _state_ awal (start) yang kita 
pengen ada di sebuah postingan.

Pas kita membikin `Post` baru, kita nge-set field `state`-nya jadi sebuah nilai 
`Some` yang nampung sebuah `Box`. `Box` ini menunjuk ke sebuah instance baru dari 
struct `Draft`. Hal ini memastikan bahwa kapan pun kita membikin instance baru 
dari `Post`, dia bakal selalu mulai sebagai _draft_. Karena field `state` pada 
`Post` itu _private_, tidak ada cara buat membikin sebuah `Post` di _state_ 
selain _draft_! Di fungsi `Post::new`, kita menge-set field `content` jadi `String` 
baru yang masih kosong.

#### Menyimpan Teks dari Konten Postingan

Kita udah lihat di Listing 18-11 kalau kita pengen bisa memanggil method bernama 
`add_text` lalu ngasih dia sebuah `&str` yang kemudian ditambahkan sebagai teks 
konten dari postingan blog tersebut. Kita mengimplementasikan ini sebagai sebuah 
method, bukannya ngekspos field `content` sebagai `pub`, supaya nanti kita bisa 
mengimplementasikan sebuah method yang bakal ngontrol gimana data di field `content` 
ini bisa dibaca. Method `add_text` ini lumayan *straightforward* (sederhana), jadi 
mari kita tambahin implementasinya di Listing 18-13 ke dalam blok `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="Mengimplementasikan method `add_text` buat nambahin teks ke `content` di sebuah postingan">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

Method `add_text` menerima referensi _mutable_ ke `self` karena kita mau mengubah 
instance `Post` tempat kita manggil `add_text`. Kemudian kita memanggil `push_str` 
pada `String` di `content` dan masukin argumen `text` buat ditambahin ke `content` 
yang udah tersimpan. Perilaku ini tidak bergantung pada _state_ yang lagi 
dimiliki postingan, jadi ini bukanlah bagian dari _state pattern_. Method 
`add_text` tidak berinteraksi dengan field `state` sama sekali, tapi dia adalah 
bagian dari perilaku yang mau kita dukung (support).

#### Memastikan Konten dari Postingan Draft Itu Kosong

Bahkan setelah kita manggil `add_text` dan nambahin beberapa konten ke postingan 
kita, kita tetap pengen method `content` buat mengembalikan string _slice_ kosong 
karena postingannya masih ada di _state_ _draft_, kayak yang ditunjukin di baris 7 
di Listing 18-11. Buat sekarang, mari kita implementasikan method `content` dengan 
cara paling simpel yang bisa menuhi persyaratan ini: selalu mengembalikan string 
_slice_ kosong. Kita bakal mengubah ini nanti pas kita udah ngimplementasiin 
kemampuan buat mengubah _state_ postingan jadi bisa di-_publish_. Sejauh ini, 
postingan cuma bisa ada di _state_ _draft_, jadi konten postingan harus selalu 
kosong. Listing 18-14 menunjukkan implementasi _placeholder_ (sementara) ini.

<Listing number="18-14" file-name="src/lib.rs" caption="Nambahin implementasi _placeholder_ buat method `content` pada `Post` yang selalu ngembaliin string _slice_ kosong">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Dengan method `content` yang ditambahkan ini, semua yang ada di Listing 18-11 
sampai baris 7 bakal jalan sesuai rencana.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>

#### Meminta Review Bakal Mengubah State dari Postingan

Berikutnya, kita perlu nambahin fungsionalitas buat meminta sebuah _review_ dari 
sebuah postingan, yang mana bakal mengubah _state_-nya dari `Draft` jadi 
`PendingReview`. Listing 18-15 nunjukin kodenya.

<Listing number="18-15" file-name="src/lib.rs" caption="Mengimplementasikan method `request_review` pada `Post` dan trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Kita ngasih `Post` sebuah method _public_ bernama `request_review` yang bakal 
menerima referensi _mutable_ ke `self`. Terus kita memanggil method internal 
`request_review` pada _state_ saat ini dari `Post`, dan method `request_review` 
yang kedua ini bakal mengonsumsi _state_ saat ini dan mengembalikan _state_ yang 
baru.

Kita nambahin method `request_review` ke trait `State`; semua tipe yang 
mengimplementasikan trait tersebut sekarang juga harus mengimplementasikan method 
`request_review`. Perhatikan bahwa ketimbang pakai `self`, `&self`, atau `&mut self` 
sebagai parameter pertama di method ini, kita malah pakai `self: Box<Self>`. 
Sintaks ini berarti method tersebut cuma valid pas dipanggil pada sebuah `Box` 
yang nampung tipe tersebut. Sintaks ini ngambil kepemilikan (ownership) atas 
`Box<Self>`, yang bakal membikin _state_ yang lama jadi tidak valid sehingga 
nilai _state_ dari `Post` bisa berubah jadi _state_ yang baru.

Buat mengonsumsi _state_ yang lama, method `request_review` butuh mengambil 
kepemilikan dari nilai _state_-nya. Di sinilah `Option` yang ada di field `state` 
milik `Post` kepake: kita memanggil method `take` buat mengambil (take out) 
nilai `Some` dari field `state` dan ninggalin nilai `None` di tempatnya karena 
Rust tidak ngebolehin kita punya field yang tidak berisi apa-apa (unpopulated 
fields) di dalam struct. Ini memungkinkan kita mindahin nilai `state` keluar dari 
`Post` ketimbang meminjamnya (borrowing). Terus kita bakal menge-set nilai 
`state` dari postingannya ke hasil dari operasi ini.

Kita perlu nge-set `state` ke `None` untuk sementara waktu ketimbang menge-setnya 
secara langsung pakai kode seperti `self.state = self.state.request_review();` 
supaya kita bisa dapat kepemilikan atas nilai `state`-nya. Ini memastikan `Post` 
tidak bisa memakai nilai `state` yang lama setelah kita udah ngubah dia jadi 
_state_ yang baru.

Method `request_review` pada `Draft` mengembalikan sebuah instance baru yang 
dibungkus `Box` dari sebuah struct baru bernama `PendingReview`, yang mana 
merepresentasikan _state_ saat sebuah postingan lagi nunggu _review_. Struct 
`PendingReview` juga mengimplementasikan method `request_review` tapi dia tidak 
melakukan perubahan (transformations) apa pun. Sebaliknya, dia ngembaliin dirinya 
sendiri (returns itself) karena kalau kita meminta _review_ pada postingan yang 
emang udah ada di _state_ `PendingReview`, dia seharusnya tetap berada di _state_ 
`PendingReview`.

Sekarang kita bisa mulai ngelihat keuntungan dari _state pattern_: method 
`request_review` pada `Post` itu sama persis terlepas dari apa nilai `state`-nya. 
Setiap _state_ bertanggung jawab atas aturannya sendiri.

Kita bakal biarin method `content` pada `Post` apa adanya, yang mana bakal 
ngembaliin string _slice_ kosong. Kita sekarang bisa punya `Post` di _state_ 
`PendingReview` maupun di _state_ `Draft`, tapi kita pengen perilaku yang sama di 
_state_ `PendingReview`. Listing 18-11 sekarang udah bisa jalan sampai baris ke-10!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

#### Menambahkan `approve` buat Mengubah Perilaku dari `content`

Method `approve` bakal mirip sama method `request_review`: dia bakal menge-set 
`state` ke nilai yang dibilang sama _state_ saat ini sebagai nilai yang harusnya 
dia miliki saat _state_ itu di-_approve_, seperti yang ditunjukin di Listing 18-16.

<Listing number="18-16" file-name="src/lib.rs" caption="Mengimplementasikan method `approve` pada `Post` dan trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Kita nambahin method `approve` ke trait `State` dan nambahin struct baru yang 
mengimplementasikan `State`, yaitu _state_ `Published`.

Mirip sama cara kerja `request_review` di `PendingReview`, kalau kita memanggil 
method `approve` pada sebuah `Draft`, hal itu tidak bakal punya efek apa-apa karena 
`approve` bakal mengembalikan `self`. Saat kita memanggil `approve` pada 
`PendingReview`, dia mengembalikan instance baru yang dibungkus `Box` dari struct 
`Published`. Struct `Published` mengimplementasikan trait `State`, dan baik untuk 
method `request_review` maupun method `approve`, dia mengembalikan dirinya sendiri 
karena postingan seharusnya tetap berada di _state_ `Published` dalam kasus-kasus 
tersebut.

Sekarang kita perlu meng-update method `content` pada `Post`. Kita mau supaya nilai 
yang dikembalikan dari `content` itu bergantung sama _state_ saat ini dari si `Post`, 
jadi kita bakal membikin si `Post` mendelegasikan (delegate) panggilan ini ke method 
`content` yang didefinisikan pada `state`-nya, seperti yang ditunjukin di Listing 
18-17.

<Listing number="18-17" file-name="src/lib.rs" caption="Meng-update method `content` pada `Post` buat mendelegasikan panggilan ke method `content` pada `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Karena tujuan utamanya adalah buat menyimpan semua aturan ini di dalam struct-struct 
yang mengimplementasikan `State`, kita memanggil sebuah method `content` pada nilai 
yang ada di dalam `state` dan memasukkan instance dari postingan tersebut (yaitu, 
`self`) sebagai argumen. Terus kita mengembalikan nilai yang dikembalikan dari 
pemakaian method `content` pada nilai `state` tadi.

Kita memanggil method `as_ref` pada `Option` tersebut karena kita cuma pengen referensi 
ke nilai yang ada di dalam `Option`, bukannya ngambil kepemilikan dari nilai itu 
sendiri. Karena `state` adalah `Option<Box<dyn State>>`, pas kita manggil `as_ref`, 
yang dikembalikan adalah `Option<&Box<dyn State>>`. Kalau kita tidak memanggil 
`as_ref`, kita bakal dapat error karena kita tidak bisa mindahin `state` ke luar 
dari referensi pinjaman (borrowed reference) `&self` dari parameter fungsinya.

Terus kita memanggil method `unwrap`, yang mana kita tahu pasti tidak bakal pernah 
menyebabkan _panic_ karena kita tahu method-method pada `Post` memastikan kalau `state` 
bakal selalu berisi nilai `Some` saat method-method itu selesai dijalankan. Ini 
adalah salah satu kasus yang kita obrolin di [“Kasus Di Mana kita Punya Lebih 
Banyak Informasi Daripada Compiler”][more-info-than-rustc] di Bab 9, di mana kita 
tahu pasti kalau nilai `None` itu mustahil, meskipun _compiler_ tidak mampu buat 
memahami hal itu.

Pada titik ini, pas kita memanggil `content` pada `&Box<dyn State>`, fitur _deref 
coercion_ (paksaan dereferensi) bakal bekerja pada tanda `&` dan `Box` tersebut, 
sehingga pada akhirnya method `content` bakal dipanggil pada tipe yang 
mengimplementasikan trait `State`. Itu artinya kita perlu nambahin `content` ke 
definisi trait `State`, dan di sanalah kita bakal menaruh logika soal konten mana 
yang harus dikembalikan tergantung di _state_ mana kita berada sekarang, kayak 
yang ditunjukin di Listing 18-18.

<Listing number="18-18" file-name="src/lib.rs" caption="Menambahkan method `content` ke trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Kita nambahin sebuah implementasi default (bawaan) buat method `content` yang 
mengembalikan string _slice_ kosong. Itu artinya kita tidak perlu repot-repot 
mengimplementasikan `content` di struct `Draft` dan `PendingReview`. Nah, buat struct 
`Published`, kita bakal menimpa (override) method `content` ini lalu mengembalikan 
nilai yang ada di `post.content`. Walaupun praktis, membiarkan method `content` di 
`State` yang menentukan apa isi dari `content` milik `Post` itu agak mengaburkan garis 
batas antara apa yang jadi tanggung jawab `State` dan apa yang jadi tanggung 
jawab `Post`.

Perhatikan bahwa kita juga butuh anotasi _lifetime_ (waktu hidup) di method ini, 
seperti yang kita bahas di Bab 10. Kita menerima referensi ke sebuah `post` sebagai 
argumen dan mengembalikan sebuah referensi ke bagian dari `post` itu, jadi _lifetime_ 
dari referensi yang dikembalikan itu berkaitan erat sama _lifetime_ dari argumen 
`post` tersebut.

Dan kita udah selesai—semua kode di Listing 18-11 sekarang berjalan sesuai rencana! 
Kita sudah berhasil mengimplementasikan _state pattern_ dengan aturan-aturan dari 
_workflow_ postingan blog. Logika yang berkaitan sama aturan-aturannya kini hidup di 
dalam _state objects_ ketimbang bertebaran (scattered) ke mana-mana di dalam `Post`.

> ### Kenapa Tidak Pake Enum Aja?
>
> kita mungkin bingung dan nanya-nanya kenapa kita tidak pakai sebuah `enum` aja buat 
> berbagai macam kemungkinan _state_ postingan tersebut sebagai varian-variannya. 
> Itu emang salah satu solusi yang mungkin; cobain aja terus bandingin hasil akhirnya 
> buat ngelihat mana yang lebih kita suka! Satu kekurangan dari memakai enum adalah 
> di setiap tempat yang ngecek nilai dari enum itu, kita bakal butuh ekspresi `match` 
> atau sejenisnya buat menangani setiap kemungkinan varian yang ada. Ini bisa jadi 
> jauh lebih berulang-ulang (repetitive) ketimbang solusi yang memakai _trait 
> object_ ini.

#### Trade-offs dari State Pattern

Kita udah nunjukin kalau Rust itu mampu buat mengimplementasikan _state pattern_ 
ala _object-oriented_ buat mengenkapsulasi (encapsulate) berbagai jenis perilaku yang 
seharusnya dimiliki sama sebuah postingan di tiap _state_-nya. Method-method pada `Post` 
sama sekali tidak tahu soal perilaku yang bermacam-macam itu. Dengan cara kita menata 
kode ini, kita cuma perlu ngecek di satu tempat buat tahu bermacam-macam cara 
gimana sebuah postingan yang di-_publish_ bisa berperilaku: yaitu di implementasi 
trait `State` pada struct `Published`.

Seandainya kita membikin implementasi alternatif yang tidak pakai _state pattern_, 
kita mungkin bakal milih buat pakai ekspresi `match` di dalam method-method pada 
`Post` atau bahkan di dalam kode `main` yang ngecek _state_ dari postingannya dan 
mengubah perilaku di tempat-tempat itu. Itu artinya kita harus ngecek di beberapa tempat 
buat bisa paham semua implikasi dari sebuah postingan saat ia berada di _state_ 
_published_.

Dengan _state pattern_, method-method di `Post` dan tempat-tempat di mana kita memakai 
`Post` tidak butuh ekspresi `match`, dan buat nambahin sebuah _state_ baru, kita cuma 
perlu nambahin satu struct baru lalu mengimplementasikan _trait methods_ pada struct 
baru itu di satu tempat aja.

Implementasi yang memakai _state pattern_ ini gampang sekali buat diperluas buat 
nambahin lebih banyak fungsionalitas. Buat ngelihat sendiri seberapa simpelnya 
memelihara (maintaining) kode yang pakai _state pattern_, coba deh beberapa 
saran ini:

- Tambahin method `reject` (tolak) yang ngubah _state_ postingan dari `PendingReview` 
  balik lagi ke `Draft`.
- Wajibkan (require) dua panggilan ke `approve` sebelum _state_-nya bisa berubah 
  jadi `Published`.
- Izinkan (allow) *user* buat nambahin teks konten cuma pas postingan lagi ada 
  di _state_ `Draft`.
  Petunjuk: biarin _state object_ yang bertanggung jawab soal apa yang mungkin 
  berubah terkait konten, tapi jangan biarin dia bertanggung jawab buat memodifikasi 
  `Post` secara langsung.

Satu kelemahan dari _state pattern_ ini adalah karena _states_-nya sendirilah yang 
mengimplementasikan proses transisi (perpindahan) ke _state_ lain, beberapa _states_ 
jadi terikat (coupled) satu sama lain. Kalau kita nambahin _state_ lain di antara 
`PendingReview` dan `Published`, seperti `Scheduled` (dijadwalkan), kita harus mengubah 
kode di `PendingReview` buat bertransisi ke `Scheduled` sebagai gantinya. Bakal lebih 
sedikit kerjaannya kalau seandainya `PendingReview` tidak perlu diubah pas kita 
nambahin _state_ baru, tapi itu berarti kita harus beralih ke desain pola yang beda 
(another design pattern).

Kelemahan lainnya adalah kita jadi menduplikasi beberapa logika. Buat menghilangkan 
sebagian dari duplikasi ini, kita mungkin nyoba buat bikin implementasi _default_ buat 
method `request_review` dan `approve` di trait `State` yang selalu mengembalikan 
`self`. Namun, ini tidak bakal jalan: pas kita pakai `State` sebagai _trait object_, 
trait itu tidak tahu apa tipe konkret yang bakal jadi `self`-nya nanti, jadi 
tipe kembalian (return type) itu tidak bisa diketahui saat _compile time_. (Ini 
adalah salah satu dari aturan `dyn` compatibility yang udah disebutin sebelumnya.)

Duplikasi lainnya termasuk implementasi method `request_review` dan `approve` 
yang mirip-mirip di `Post`. Kedua method itu memakai `Option::take` dengan field 
`state` milik `Post`, dan kalau `state` itu isinya `Some`, mereka mendelegasikannya 
ke implementasi nilai yang dibungkus tersebut buat method yang sama lalu menge-set 
nilai baru dari field `state` dengan hasil panggilannya. Kalau kita punya 
sangat banyak method di `Post` yang ngikutin pola ini, kita mungkin bakal 
pertimbangin buat mendefinisikan sebuah _macro_ buat ngebuang pengulangan ini 
(lihat [“Macros”][macros] di Bab 20).

Dengan mengimplementasikan _state pattern_ persis kayak yang didefinisikan buat bahasa 
pemrograman _object-oriented_, kita nyatanya tidak memanfaatkan kekuatan unggulan Rust 
semaksimal mungkin. Mari kita lihat beberapa perubahan yang bisa kita lakukan pada 
_crate_ `blog` yang bisa ngebikin _invalid states_ (state yang tidak valid) dan 
transisi yang salah menjadi error saat _compile-time_ (compile-time errors).

### Menge-encode States dan Perilaku (Behavior) ke dalam Types (Tipe)

Kita bakal nunjukin gimana cara memikirkan ulang (rethink) _state pattern_ buat 
dapat kumpulan _trade-offs_ yang berbeda. Ketimbang mengenkapsulasi _states_ 
dan transisi secara keseluruhan supaya kode luar tidak tahu apa-apa soal mereka, 
kita bakal menge-_encode_ (menyandikan) _states_ ke dalam tipe-tipe yang berbeda-beda. 
Konsekuensinya, sistem pengecekan tipe Rust (_type checking system_) bakal mencegah 
usaha (attempts) buat memakai postingan _draft_ di tempat-tempat yang mana cuma 
postingan yang udah _published_ (dipublikasikan) yang boleh dipakai, dengan 
ngeluarin error _compiler_.

Mari kita perhatikan bagian awal dari `main` di Listing 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Kita tetep mau ngasih kemampuan buat bikin postingan baru di _state_ _draft_ memakai 
`Post::new` dan kemampuan buat nambahin teks ke dalam konten postingannya. Tapi 
ketimbang punya sebuah method `content` pada postingan _draft_ yang cuma ngembaliin 
string kosong, kita malah bakal membikin supaya postingan _draft_ itu *sama sekali 
tidak punya* method `content`. Dengan begitu, kalau kita nyoba ngambil konten 
dari postingan _draft_, kita bakal dapat error _compiler_ yang ngasih tahu kita 
kalau method itu tidak eksis. Sebagai hasilnya, bakal jadi mustahil bagi kita 
buat secara tidak sengaja menampilkan konten dari postingan _draft_ pas udah masuk 
_production_ (produksi), karena kodenya aja tidak bakal bisa di-compile. 
Listing 18-19 nunjukin definisi struct `Post` dan struct `DraftPost`, beserta method 
yang ada pada keduanya.

<Listing number="18-19" file-name="src/lib.rs" caption="Sebuah `Post` yang punya method `content` dan `DraftPost` yang tidak punya method `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Baik struct `Post` maupun `DraftPost` punya field _private_ bernama `content` yang 
nyimpen teks dari postingan blog tersebut. Struct-struct ini tidak lagi punya 
field `state` karena kita udah mindahin _encoding_ dari _state_-nya ke dalam tipe 
dari struct-struct itu. Struct `Post` bakal merepresentasikan sebuah postingan yang 
udah di-_publish_, dan dia punya method `content` yang bakal ngembaliin nilai dari 
`content`.

Kita tetap punya fungsi `Post::new`, tapi ketimbang mengembalikan sebuah instance dari 
`Post`, dia sekarang mengembalikan sebuah instance dari `DraftPost`. Karena `content` 
itu sifatnya _private_ dan tidak ada fungsi lain yang ngembaliin `Post`, maka mustahil 
buat membikin sebuah instance dari `Post` saat ini.

Struct `DraftPost` punya sebuah method `add_text`, jadi kita bisa nambahin teks ke 
dalam `content` sama kayak sebelumnya, tapi perhatikan kalau `DraftPost` *tidak 
punya* method `content` yang didefinisikan! Jadi sekarang programnya memastikan 
kalau semua postingan selalu bermula sebagai postingan _draft_, dan postingan 
_draft_ ini belum punya konten yang bisa ditampilkan. Usaha apa pun buat ngakalin 
atau ngelewatin batasan-batasan ini bakal ngasilin error _compiler_.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

Lalu gimana dong caranya kita dapet postingan yang udah ke-_publish_? Kita mau 
menegakkan aturan bahwa sebuah postingan _draft_ itu wajib di-_review_ dan 
di-_approve_ (disetujui) sebelum bisa di-_publish_. Postingan yang lagi ada di 
_state_ _pending review_ (nunggu review) juga seharusnya masih belum bisa nampilin 
konten apa pun. Mari kita implementasikan batasan-batasan ini dengan menambahkan struct 
baru, `PendingReviewPost`, lalu mendefinisikan method `request_review` pada 
`DraftPost` yang bakal mengembalikan `PendingReviewPost` dan mendefinisikan method 
`approve` pada `PendingReviewPost` yang bakal mengembalikan `Post`, seperti yang 
ditunjukin di Listing 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="Sebuah `PendingReviewPost` yang dibikin lewat manggil `request_review` pada `DraftPost` dan method `approve` yang mengubah `PendingReviewPost` jadi sebuah `Post` yang ke-publish">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Method `request_review` dan `approve` mengambil kepemilikan (ownership) dari `self`, 
dengan begitu mengonsumsi instance `DraftPost` dan `PendingReviewPost` lalu mengubah 
(_transforming_) mereka secara berurutan menjadi `PendingReviewPost` dan `Post` yang 
ke-_publish_. Dengan cara ini, kita tidak bakal punya sisa-sisa (lingering) instance 
dari `DraftPost` setelah kita memanggil `request_review` pada mereka, dan seterusnya. 
Struct `PendingReviewPost` tidak punya method `content` padanya, jadi usaha buat 
ngebaca kontennya bakal menghasilkan error _compiler_, persis kayak `DraftPost`. Karena 
satu-satunya cara buat dapetin instance `Post` yang udah ke-_publish_ (yang mana emang 
punya method `content` padanya) adalah dengan manggil method `approve` pada sebuah 
`PendingReviewPost`, dan satu-satunya cara buat dapetin `PendingReviewPost` adalah 
dengan manggil method `request_review` pada sebuah `DraftPost`, kita sekarang 
telah berhasil menge-_encode_ _workflow_ dari postingan blog ini ke dalam sistem tipe 
(type system) di Rust.

Tapi kita juga harus membikin beberapa perubahan kecil di `main`. Method 
`request_review` dan `approve` mengembalikan instance baru alih-alih memodifikasi 
struct tempat mereka dipanggil, jadi kita perlu nambahin *assignment* _shadowing_ 
`let post =` lagi buat nyimpen instance-instance baru yang dikembalikan itu. Kita 
juga tidak bisa lagi punya penegasan (assertions) soal postingan _draft_ maupun 
postingan yang nunggu _review_ punya konten string kosong, tapi kita juga tidak butuh 
penegasan itu lagi kok: kita emang udah tidak bisa lagi nge-compile kode yang mencoba 
buat memakai konten dari postingan yang ada di _states_ tersebut. Kode yang udah 
di-update di `main` ini ditunjukin di Listing 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="Modifikasi ke `main` buat memakai implementasi yang baru dari _workflow_ postingan blog kita">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

Perubahan-perubahan yang perlu kita lakuin di `main` buat me-*reassign* nilai ke 
`post` berarti kalau implementasi ini udah tidak bener-bener ngikutin _state pattern_ 
ala _object-oriented_ lagi: transisi-transisi antara _states_ tersebut udah tidak lagi 
dienkapsulasi sepenuhnya di dalam implementasi `Post`. Namun, keuntungan yang kita 
dapat adalah bahwa _invalid states_ sekarang jadi mustahil terjadi berkat sistem 
tipe dan _type checking_ (pengecekan tipe) yang terjadi saat _compile time_! Ini 
memastikan kalau beberapa jenis _bugs_ tertentu, seperti nge-display (nampilin) konten 
dari postingan yang belum di-_publish_, bakal ketahuan jauh-jauh sebelum kodenya 
berhasil masuk ke _production_.

Cobalah beberapa tugas yang disarankan di awal bagian ini pada _crate_ `blog` 
setelah memakai desain yang ada di Listing 18-21 buat melihat apa pendapat kita 
soal desain dari versi kode yang ini. Perhatikan kalau beberapa dari tugas 
tersebut mungkin emang udah terselesaikan secara natural di desain yang ini.

Kita udah melihat kalau walaupun Rust itu mampu mengimplementasikan desain pola 
_object-oriented_, pola-pola lain, kayak nge-_encode_ _state_ ke dalam sistem tipe, 
juga tersedia dan bisa diimplementasikan di Rust. Pola-pola ini punya kumpulan 
_trade-offs_ yang beda-beda. Walaupun kita mungkin udah familier sekali sama 
pola-pola _object-oriented_, memikirkan kembali (rethinking) masalahnya buat 
memanfaatkan fitur-fitur dari Rust bisa ngasih banyak keuntungan, seperti mencegah 
munculnya _bugs_ tertentu saat _compile time_. Pola-pola _object-oriented_ tidak bakal 
selalu jadi solusi yang terbaik di Rust karena adanya fitur-fitur tertentu, seperti 
_ownership_, yang mana memang tidak dipunyai oleh bahasa-bahasa pemrograman 
_object-oriented_ lainnya.

## Ringkasan

Terlepas dari apakah kita mikir kalau Rust itu adalah sebuah bahasa yang 
_object-oriented_ atau bukan setelah baca bab ini, kita sekarang udah tahu kalau 
kita bisa memakai _trait objects_ buat dapetin beberapa fitur ala _object-oriented_ 
di Rust. _Dynamic dispatch_ bisa ngasih kode kita sedikit keluwesan (flexibility) 
yang harus dibayar dengan sedikit pinalti di performa _runtime_. Kita bisa memakai 
keluwesan ini buat mengimplementasikan pola-pola _object-oriented_ yang bisa 
ngebantu kode kita supaya lebih gampang dipelihara (maintainability). Rust juga punya 
fitur lain, kayak _ownership_, yang tidak dipunyai sama bahasa-bahasa 
_object-oriented_ pada umumnya. Sebuah pola _object-oriented_ tidak bakal selalu 
jadi cara yang paling oke buat memanfaatkan kekuatan dari Rust, tapi dia jelas 
adalah salah satu pilihan yang tersedia.

Berikutnya, kita bakal melihat *patterns* (pola), yang mana merupakan fitur Rust 
lainnya yang juga ngasih keluwesan tingkat tinggi. Kita udah ngelihat sedikit 
soal pola-pola ini di sepanjang buku, tapi kita belum ngelihat potensi penuh dari 
kemampuan mereka. Ayo gas!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
