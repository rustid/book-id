## Misahin Modul ke Dalem File yang Beda

Sejauh ini, semua contoh di bab ini mendefinisikan beberapa modul di dalem satu 
file. Pas modulnya jadi makin gede, kita mungkin mau mindahin definisi mereka ke 
file terpisah biar kodenya lebih gampang dinavigasi.

Sebagai contoh, yuk kita mulai dari kode di Listing 7-17 yang punya beberapa 
modul restoran. Kita bakal ngekstrak modul-modulnya ke dalem file bukannya 
punya semua modul didefinisikan di file _crate root_. Di kasus ini, file _crate 
root_-nya adalah _src/lib.rs_, tapi prosedur ini juga berlaku buat _binary crates_ 
yang mana file _crate root_-nya adalah _src/main.rs_.

Pertama kita bakal ngekstrak modul `front_of_house` ke dalem filenya sendiri. 
Hapus kode di dalem kurung kurawal buat modul `front_of_house`, nyisain cuma 
deklarasi `mod front_of_house;` aja, jadi _src/lib.rs_ isinya cuma kode yang 
ditunjukin di Listing 7-21. Perhatiin ya kalau ini nggak bakal bisa di-compile 
sampe kita bikin file _src/front_of_house.rs_ di Listing 7-22.

<Listing number="7-21" file-name="src/lib.rs" caption="Mendeklarasikan modul `front_of_house` yang body-nya bakal ada di *src/front_of_house.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Selanjutnya, taruh kode yang tadinya ada di dalem kurung kurawal ke file baru 
namanya _src/front_of_house.rs_, kayak yang ditunjukin di Listing 7-22. 
_Compiler_ tau buat nyari di file ini karena dia nemu deklarasi modul di 
_crate root_ dengan nama `front_of_house`.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="Definisi di dalem modul `front_of_house` di *src/front_of_house.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Perhatiin ya kalau kita cuma perlu _load_ (muat) file pake deklarasi `mod` 
*sekali* aja di pohon modul kita. Begitu _compiler_ tau filenya adalah bagian 
dari project (dan tau di mana letak kode itu di pohon modul gara-gara tempat 
kita naruh statement `mod`), file lain di project kita harus ngerujuk ke kode 
di file yang di-_load_ pake _path_ ke tempat di mana dia dideklarasikan, kayak 
yang udah dibahas di bagian [“Paths (Jalur) buat Ngerujuk Item di Pohon Modul”][paths]. 
Dengan kata lain, `mod` itu *bukan* operasi “include” kayak yang mungkin kita 
liat di bahasa pemrograman lainnya.

Selanjutnya, kita bakal ngekstrak modul `hosting` ke dalem filenya sendiri. 
Prosesnya agak beda karena `hosting` adalah anak modul dari `front_of_house`, 
bukan dari modul _root_. Kita bakal naruh file buat `hosting` di direktori 
baru yang namanya ngikutin leluhurnya di pohon modul, di kasus ini 
_src/front_of_house_.

Buat mulai mindahin `hosting`, kita ubah _src/front_of_house.rs_ biar isinya 
cuma deklarasi dari modul `hosting` aja:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Terus kita bikin direktori _src/front_of_house_ sama file _hosting.rs_ buat 
nampung definisi yang dibikin di modul `hosting`:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Kalau kita malah naruh _hosting.rs_ di direktori _src_, _compiler_ bakal ngarepin 
kode _hosting.rs_ ada di modul `hosting` yang dideklarasikan di _crate root_, 
dan bukan dideklarasikan sebagai anak dari modul `front_of_house`. Aturan 
_compiler_ soal file mana yang harus dicek buat kode modul mana bikin direktori 
dan file jadi lebih mirip (match) sama pohon modulnya.

> ### Alternate File Paths (Path File Alternatif)
>
> Sejauh ini kita udah ngebahas path file yang paling idiomatik yang dipake sama 
> _compiler_ Rust, tapi Rust juga support gaya path file yang lebih lama. Buat 
> modul namanya `front_of_house` yang dideklarasikan di _crate root_, _compiler_ 
> bakal nyari kode modulnya di:
>
> - _src/front_of_house.rs_ (yang baru aja kita bahas)
> - _src/front_of_house/mod.rs_ (gaya lama, tetep di-support path-nya)
>
> Buat modul namanya `hosting` yang merupakan submodul dari `front_of_house`, 
> _compiler_ bakal nyari kode modulnya di:
>
> - _src/front_of_house/hosting.rs_ (yang baru aja kita bahas)
> - _src/front_of_house/hosting/mod.rs_ (gaya lama, tetep di-support path-nya)
>
> Kalau kita pake kedua gaya ini buat modul yang sama, kita bakal dapet error 
> _compiler_. Pake campuran kedua gaya buat modul yang beda di project yang 
> sama itu dibolehin, tapi mungkin bakal ngebingungin orang yang lagi navigasi 
> project kita.
>
> Kekurangan utama dari gaya yang pake file namanya _mod.rs_ adalah project 
> kita bisa berakhir dengan banyak file yang namanya _mod.rs_, yang bisa 
> ngebingungin pas kita ngebuka mereka semua di _editor_ secara bersamaan.

Kita udah mindahin kode tiap modul ke file yang terpisah, dan pohon modulnya 
tetep sama. Pemanggilan fungsi di `eat_at_restaurant` bakal jalan tanpa 
modifikasi apa pun, walaupun definisinya sekarang tinggal di file yang beda. 
Teknik ini ngebolehin kita mindahin modul ke file baru seiring ukurannya makin 
gede.

Perhatiin ya kalau statement `pub use crate::front_of_house::hosting` di 
_src/lib.rs_ juga nggak berubah, dan `use` juga nggak punya pengaruh apa pun 
ke file mana yang di-compile sebagai bagian dari crate-nya. Keyword `mod` 
mendeklarasikan modul, dan Rust nyari di file dengan nama yang sama kayak 
modulnya buat nyari kode yang masuk ke modul itu.

## Ringkasan

Rust ngebolehin kita misahin sebuah package jadi beberapa crates dan sebuah 
crate jadi beberapa modul biar kita bisa ngerujuk ke item yang didefinisikan 
di satu modul dari modul lain. Kita bisa lakuin ini dengan nentuin absolute 
atau relative paths. Path-path ini bisa dibawa ke dalem scope pake statement 
`use` biar kita bisa pake path yang lebih pendek buat banyak pemakaian item itu 
di scope tersebut. Kode modul itu _private_ secara default, tapi kita bisa 
bikin definisinya jadi _public_ dengan nambahin keyword `pub`.

Di bab berikutnya, kita bakal liat beberapa struktur data koleksi (collection) 
di standard library yang bisa kita pake di dalem kode kita yang udah terorganisir 
dengan rapi.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
