## Paths (Jalur) buat Ngerujuk Item di Pohon Modul

Buat ngasih tau Rust di mana harus nyari sebuah item di pohon modul, kita pake 
_path_ (jalur) dengan cara yang sama kayak kita pake path pas navigasi sistem 
file. Buat manggil sebuah fungsi, kita harus tau _path_-nya.

Sebuah _path_ bisa punya dua bentuk:

- _Absolute path_ (path absolut) adalah path lengkap mulai dari _crate root_; 
  buat kode dari crate eksternal, absolute path dimulai pake nama crate-nya, 
  dan buat kode dari crate saat ini, dia dimulai pake literal `crate`.
- _Relative path_ (path relatif) dimulai dari modul saat ini terus pake `self`, 
  `super`, atau identifier (nama) di modul saat ini.

Baik absolute maupun relative path diikuti sama satu atau lebih identifier yang 
dipisahin pake titik dua ganda (`::`).

Balik lagi ke Listing 7-1, katakanlah kita mau manggil fungsi `add_to_waitlist`. 
Ini sama aja kayak nanya: apa sih path dari fungsi `add_to_waitlist`? 
Listing 7-3 isinya Listing 7-1 tapi beberapa modul sama fungsinya dihapus biar 
fokus.

Kita bakal nunjukin dua cara buat manggil fungsi `add_to_waitlist` dari fungsi 
baru, `eat_at_restaurant`, yang didefinisikan di _crate root_. Path-path ini 
udah bener, tapi ada satu masalah lagi yang bakal nyegah contoh ini buat bisa 
di-compile gitu aja. Kita bakal jelasin alasannya bentar lagi.

Fungsi `eat_at_restaurant` itu bagian dari API _public_ dari _library crate_ 
kita, jadi kita nandain dia pake keyword `pub`. Di bagian [“Mengekspos Paths 
dengan Keyword `pub`”][pub], kita bakal bahas `pub` lebih detail.

<Listing number="7-3" file-name="src/lib.rs" caption="Manggil fungsi `add_to_waitlist` pake absolute dan relative paths">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

Pertama kali kita manggil fungsi `add_to_waitlist` di `eat_at_restaurant`, 
kita pake absolute path. Fungsi `add_to_waitlist` didefinisikan di crate yang 
sama kayak `eat_at_restaurant`, yang artinya kita bisa pake keyword `crate` 
buat mulai absolute path-nya. Terus kita masukin tiap modul secara berurutan 
sampe kita nyampe ke `add_to_waitlist`. Bayangin aja sistem file dengan 
struktur yang sama: kita bakal nentuin path 
`/front_of_house/hosting/add_to_waitlist` buat jalanin program `add_to_waitlist`; 
pake nama `crate` buat mulai dari _crate root_ itu kayak pake `/` buat mulai 
dari _root_ sistem file di terminal (shell) kita.

Kedua kalinya kita manggil `add_to_waitlist` di `eat_at_restaurant`, kita 
pake relative path. Path-nya dimulai dari `front_of_house`, nama modul yang 
didefinisikan di level yang sama di pohon modul dengan `eat_at_restaurant`. Di 
sini, kalau di sistem file, ini sama aja kayak pake path 
`front_of_house/hosting/add_to_waitlist`. Mulai pake nama modul artinya 
path-nya itu relatif.

Milih buat pake relative atau absolute path itu keputusan yang bakal kita ambil 
berdasarkan project kita, dan itu tergantung apakah kita lebih sering mindahin 
kode definisi item secara terpisah atau barengan sama kode yang pake item itu. 
Misalnya, kalau kita mindahin modul `front_of_house` sama fungsi 
`eat_at_restaurant` ke dalem modul namanya `customer_experience`, kita harus 
update absolute path ke `add_to_waitlist`, tapi relative path-nya tetep bakal 
valid. Sebaliknya, kalau kita mindahin fungsi `eat_at_restaurant` secara 
terpisah ke dalem modul namanya `dining`, absolute path buat manggil 
`add_to_waitlist` bakal tetep sama, tapi relative path-nya harus di-update. 
Preferensi kita secara umum adalah nentuin pake absolute path karena biasanya 
kita lebih sering mindahin definisi kode sama pemanggilan item secara 
independen satu sama lain.

Yuk kita coba compile Listing 7-3 dan cari tau kenapa ini belum bisa 
di-compile! Error yang kita dapet ditunjukin di Listing 7-4.

<Listing number="7-4" caption="Error _compiler_ pas nge-build kode di Listing 7-3">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

Pesan error-nya bilang kalau modul `hosting` itu _private_. Dengan kata lain, 
kita udah punya path yang bener buat modul `hosting` sama fungsi 
`add_to_waitlist`, tapi Rust nggak ngebolehin kita pake mereka karena Rust 
nggak punya akses ke bagian _private_-nya. Di Rust, semua item (fungsi, method, 
struct, enum, modul, sama konstanta) itu _private_ terhadap modul induknya 
secara default. Kalau kita mau bikin sebuah item kayak fungsi atau struct jadi 
_private_, kita taruh dia di dalem modul.

Item di modul induk nggak bisa pake item _private_ di dalem anak modulnya 
(child modules), tapi item di anak modul bisa pake item di modul leluhurnya 
(ancestor modules). Ini karena anak modul ngebungkus dan nyembunyiin detail 
implementasinya, tapi anak modul bisa liat konteks di mana mereka didefinisikan. 
Lanjutin analogi kita, bayangin aturan privasi ini kayak dapur restoran 
(_back office_): apa yang terjadi di sana itu _private_ buat pelanggan restoran, 
tapi manajer bisa liat dan ngelakuin apa aja di restoran yang mereka jalanin.

Rust milih buat bikin sistem modul jalan kayak gini biar nyembunyiin detail 
implementasi internal jadi default. Dengan gitu, kita tau bagian kode internal 
mana yang bisa kita ubah tanpa ngerusak kode eksternalnya. Tapi, Rust tetep 
ngasih kita opsi buat ngekspos bagian internal dari kode anak modul ke modul 
leluhurnya pake keyword `pub` buat bikin item jadi _public_.

### Mengekspos Paths dengan Keyword `pub`

Yuk balik lagi ke error di Listing 7-4 yang ngasih tau kita kalau modul 
`hosting` itu _private_. Kita mau fungsi `eat_at_restaurant` di modul induknya 
punya akses ke fungsi `add_to_waitlist` di anak modulnya, jadi kita nandain 
modul `hosting` pake keyword `pub`, kayak yang ditunjukin di Listing 7-5.

<Listing number="7-5" file-name="src/lib.rs" caption="Mendeklarasikan modul `hosting` sebagai `pub` biar bisa dipake dari `eat_at_restaurant`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

Sayangnya, kode di Listing 7-5 tetep ngasilin error _compiler_, kayak yang 
ditunjukin di Listing 7-6.

<Listing number="7-6" caption="Error _compiler_ pas nge-build kode di Listing 7-5">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

Apa yang terjadi? Nambahin keyword `pub` di depan `mod hosting` bikin modul 
itu jadi _public_. Dengan perubahan ini, kalau kita bisa akses `front_of_house`, 
kita bisa akses `hosting`. Tapi _isi_ dari `hosting` itu tetep _private_; 
bikin modul jadi _public_ nggak bikin isinya otomatis ikutan _public_. Keyword 
`pub` pada sebuah modul cuma ngebolehin kode di modul leluhurnya buat ngerujuk 
ke dia, bukan buat akses kode di dalemnya. Karena modul itu adalah wadah 
(container), nggak banyak yang bisa kita lakuin dengan cuma bikin modulnya jadi 
_public_; kita perlu melangkah lebih jauh terus milih buat bikin satu atau lebih 
item di dalem modulnya ikutan jadi _public_ juga.

Error di Listing 7-6 bilang kalau fungsi `add_to_waitlist` itu _private_. Aturan 
privasi berlaku buat struct, enum, fungsi, dan method, dan juga modul.

Yuk kita bikin fungsi `add_to_waitlist` jadi _public_ juga dengan nambahin 
keyword `pub` sebelum definisinya, kayak di Listing 7-7.

<Listing number="7-7" file-name="src/lib.rs" caption="Nambahin keyword `pub` ke `mod hosting` dan `fn add_to_waitlist` ngebolehin kita manggil fungsinya dari `eat_at_restaurant`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

Sekarang kodenya bisa di-compile! Buat liat kenapa nambahin keyword `pub` 
ngebolehin kita pake path-path ini di `eat_at_restaurant` sesuai sama aturan 
privasi, yuk kita bahas absolute sama relative path-nya.

Di absolute path, kita mulai pake `crate`, yaitu akar (_root_) dari pohon 
modul crate kita. Modul `front_of_house` didefinisikan di _crate root_. Walaupun 
`front_of_house` itu bukan _public_, tapi karena fungsi `eat_at_restaurant` 
didefinisikan di modul yang sama kayak `front_of_house` (artinya `eat_at_restaurant` 
sama `front_of_house` itu sodaraan), kita bisa ngerujuk ke `front_of_house` dari 
`eat_at_restaurant`. Terus lanjut ke modul `hosting` yang udah ditandain pake 
`pub`. Kita bisa akses modul induk dari `hosting`, jadi kita bisa akses 
`hosting`. Terakhir, fungsi `add_to_waitlist` ditandain pake `pub` dan kita 
bisa akses modul induknya, jadi pemanggilan fungsi ini berhasil!

Di relative path, logikanya sama persis kayak absolute path kecuali buat langkah 
pertama: bukannya mulai dari _crate root_, path-nya mulai dari `front_of_house`. 
Modul `front_of_house` didefinisikan di dalem modul yang sama kayak 
`eat_at_restaurant`, jadi relative path yang dimulai dari modul tempat 
`eat_at_restaurant` didefinisikan itu berhasil. Terus, karena `hosting` sama 
`add_to_waitlist` ditandain pake `pub`, sisa path-nya berhasil, dan pemanggilan 
fungsi ini jadi valid!

Kalau kita berencana buat nge-share _library crate_ kita biar project lain 
bisa pake kode kita, API _public_ kita adalah kontrak kita sama _user_ dari crate 
kita yang nentuin gimana mereka bisa berinteraksi sama kode kita. Ada banyak 
pertimbangan soal cara ngelola perubahan di API _public_ kita buat ngebikin orang 
lebih gampang bergantung sama crate kita. Pertimbangan-pertimbangan ini di luar 
cakupan buku ini; kalau kita tertarik sama topik ini, cek [The Rust API Guidelines][api-guidelines].

> #### Best Practices buat Packages yang Punya Binary sama Library
>
> Kita sempet nyebut kalau sebuah package bisa punya baik _crate root binary_ di 
> _src/main.rs_ maupun _crate root library_ di _src/lib.rs_, dan kedua crates ini 
> bakal punya nama yang sama secara default. Biasanya, packages dengan pola ini 
> yang punya baik _library_ maupun _binary crate_ bakal punya kode di _binary 
> crate_-nya secukupnya aja buat mulai _executable_ yang manggil kode yang 
> didefinisikan di _library crate_. Ini bikin project lain bisa dapet manfaat 
> dari sebagian besar fungsionalitas yang disediain package-nya karena kode di 
> _library crate_-nya bisa di-share.
>
> Pohon modul harusnya didefinisikan di _src/lib.rs_. Terus, item _public_ mana 
> pun bisa dipake di _binary crate_ dengan mulai path-nya pake nama package-nya. 
> _Binary crate_ itu jadi _user_ dari _library crate_-nya sama kayak _crate_ 
> eksternal lainnya yang bakal pake _library crate_ itu: dia cuma bisa pake 
> API _public_-nya. Ini ngebantu kita desain API yang bagus; kita nggak cuma jadi 
> _author_-nya, tapi kita juga jadi kliennya!
>
> Di [Bab 12][ch12], kita bakal nunjukin praktik organisasi ini pake program 
> _command line_ yang bakal isinya _binary crate_ sekaligus _library crate_.

### Memulai Relative Paths dengan `super`

Kita bisa ngebangun relative paths yang mulai dari modul induknya, bukannya 
modul saat ini atau _crate root_, pake `super` di awal path-nya. Ini kayak 
mulai path sistem file pake sintaks `..` yang artinya naik ke direktori 
induknya. Pake `super` ngebolehin kita ngerujuk item yang kita tau ada di 
modul induknya, yang bisa ngebikin penataan ulang pohon modul jadi lebih gampang 
kalau modul itu terkait erat sama induknya tapi si induk mungkin dipindah ke 
tempat lain di pohon modul suatu hari nanti.

Coba liat kode di Listing 7-8 yang mensimulasikan situasi di mana seorang koki 
benerin pesanan yang salah terus ngasih langsung ke pelanggannya. Fungsi 
`fix_incorrect_order` yang didefinisikan di modul `back_of_house` manggil 
fungsi `deliver_order` yang didefinisikan di modul induknya dengan nentuin 
path ke `deliver_order`, mulai pake `super`.

<Listing number="7-8" file-name="src/lib.rs" caption="Manggil fungsi pake relative path yang dimulai dengan `super`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

Fungsi `fix_incorrect_order` ada di modul `back_of_house`, jadi kita bisa pake 
`super` buat pindah ke modul induk dari `back_of_house`, yang di kasus ini 
adalah `crate`, yaitu _root_-nya. Dari situ, kita nyari `deliver_order` dan 
nemuin dia. Mantap! Kita rasa modul `back_of_house` sama fungsi `deliver_order` 
kemungkinan bakal tetep punya hubungan yang sama satu sama lain dan bakal 
dipindah barengan kalau kita mutusin buat ngerombak pohon modul crate kita. 
Makanya, kita pake `super` biar lebih dikit tempat yang harus di-update nanti 
kalau kode ini dipindah ke modul yang beda.

### Bikin Structs dan Enums Jadi Public

Kita juga bisa pake `pub` buat nandain structs sama enums jadi _public_, tapi 
ada beberapa detail tambahan buat penggunaan `pub` bareng structs sama enums. 
Kalau kita pake `pub` sebelum definisi struct, kita bikin struct-nya jadi 
_public_, tapi field-field di struct-nya bakal tetep _private_. Kita bisa 
bikin tiap field jadi _public_ atau nggak sesuai kasusnya masing-masing. Di 
Listing 7-9, kita mendefinisikan sebuah struct _public_ `back_of_house::Breakfast` 
dengan field `toast` yang _public_ tapi field `seasonal_fruit` yang _private_. 
Ini mensimulasikan kasus di restoran di mana pelanggan bisa milih roti yang 
dateng bareng makanannya, tapi koki yang mutusin buah apa yang nyertain makanannya 
berdasarkan musim sama stoknya. Ketersediaan buah berubah-ubah dengan cepet, 
jadi pelanggan nggak bisa milih buahnya atau bahkan tau buah apa yang bakal 
mereka dapet.

<Listing number="7-9" file-name="src/lib.rs" caption="Sebuah struct dengan beberapa field _public_ dan beberapa field _private_">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

Karena field `toast` di struct `back_of_house::Breakfast` itu _public_, di 
`eat_at_restaurant` kita bisa nulis dan baca field `toast` pake notasi titik. 
Perhatiin kalau kita nggak bisa pake field `seasonal_fruit` di `eat_at_restaurant`, 
karena `seasonal_fruit` itu _private_. Coba di-uncomment baris yang ngubah nilai 
field `seasonal_fruit` buat liat error apa yang bakal dapet!

Terus, perhatiin karena `back_of_house::Breakfast` punya field _private_, struct 
ini harus nyediain fungsi _associated_ yang _public_ yang ngebikin (mengkonstruksi) 
instance dari `Breakfast` (kita namain `summer` di sini). Kalau `Breakfast` nggak 
punya fungsi kayak gitu, kita nggak bakal bisa bikin instance dari `Breakfast` 
di `eat_at_restaurant` karena kita nggak bisa set nilai dari field 
`seasonal_fruit` yang _private_ di `eat_at_restaurant`.

Sebaliknya, kalau kita bikin sebuah enum jadi _public_, semua variannya ikutan 
jadi _public_. Kita cuma perlu naruh `pub` sebelum keyword `enum`, kayak yang 
ditunjukin di Listing 7-10.

<Listing number="7-10" file-name="src/lib.rs" caption="Nandain enum sebagai _public_ bikin semua variannya ikutan _public_.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

Karena kita bikin enum `Appetizer` jadi _public_, kita bisa pake varian `Soup` 
sama `Salad` di `eat_at_restaurant`.

Enums nggak terlalu berguna kalau variannya nggak _public_; bakal nyebelin 
sekali kalau harus nganotasi semua varian enum pake `pub` di setiap kasus, jadi 
default buat varian enum adalah _public_. Structs biasanya berguna walaupun 
field-nya nggak _public_, jadi field struct ngikutin aturan umum bahwa segala 
hal itu _private_ secara default kecuali dianotasi pake `pub`.

Ada satu lagi situasi yang ngelibatin `pub` yang belum kita bahas, dan itu 
adalah fitur sistem modul kita yang terakhir: keyword `use`. Kita bakal ngebahas 
`use` sendirian dulu, terus kita bakal nunjukin gimana cara gabungin `pub` sama 
`use`.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
