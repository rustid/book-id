## Nyimpen Keys (Kunci) dengan Nilai Terkait di Hash Maps

Koleksi umum kita yang terakhir adalah _hash map_. Tipe `HashMap<K, V>` nyimpen 
pemetaan dari keys (kunci) bertipe `K` ke nilai bertipe `V` pake sebuah fungsi 
_hashing_, yang nentuin gimana cara naruh keys sama nilai-nilai ini ke dalem 
memori. Banyak bahasa pemrograman support struktur data jenis ini, tapi mereka 
sering pake nama yang beda-beda, kayak _hash_, _map_, _object_, _hash table_, 
_dictionary_, atau _associative array_, buat nyebut beberapa di antaranya.

Hash maps sangat berguna pas kita mau nyari data bukan pake indeks, kayak yang 
kita lakuin pake vectors, tapi pake _key_ yang tipenya bisa apa aja. Misalnya, 
di dalem game, kita bisa nyatet skor tiap tim di dalem hash map di mana tiap 
_key_-nya adalah nama timnya dan nilainya adalah skor tiap tim. Kalo kita punya 
nama timnya, kita bisa ngambil skornya.

Kita bakal ngebahas API dasar dari hash maps di bagian ini, tapi masih banyak 
lagi fungsionalitas keren yang ngumpet di fungsi-fungsi yang didefinisikan pada 
`HashMap<K, V>` sama _standard library_. Kayak biasa, cek dokumentasi _standard 
library_ buat info lebih lanjut.

### Bikin Hash Map Baru

Salah satu cara buat bikin hash map kosong adalah pake `new` terus nambahin 
elemennya pake `insert`. Di Listing 8-20, kita lagi nyatet skor dari dua tim 
yang namanya _Blue_ sama _Yellow_. Tim Blue mulai dengan 10 poin, dan tim Yellow 
mulai dengan 50 poin.

<Listing number="8-20" caption="Bikin hash map baru dan nge-_insert_ beberapa key sama nilai">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Perhatiin ya kalau kita pertama-tama harus bawa `HashMap` dari porsi collections 
di _standard library_ ke dalem scope pake `use`. Dari ketiga koleksi umum kita, 
yang satu ini paling jarang dipake, jadi dia nggak dimasukin ke fitur-fitur yang 
otomatis dibawa ke dalem scope lewat _prelude_. Hash maps juga dapet lebih 
dikit _support_ dari _standard library_; nggak ada macro bawaan buat 
ngonstruksi mereka, misalnya.

Sama kayak vectors, hash maps nyimpen datanya di _heap_. `HashMap` ini punya 
_keys_ tipe `String` sama nilai tipe `i32`. Sama juga kayak vectors, hash maps 
itu homogen: semua _keys_ harus punya tipe yang sama, dan semua nilai harus 
punya tipe yang sama.

### Akses Nilai di dalem Hash Map

Kita bisa ngambil nilai dari hash map dengan masukin _key_-nya ke method `get`, 
kayak yang ditunjukin di Listing 8-21.

<Listing number="8-21" caption="Akses skor buat tim Blue yang disimpan di hash map">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Di sini, `score` bakal punya nilai yang terkait sama tim Blue, dan hasilnya 
bakal `10`. Method `get` balikin sebuah `Option<&V>`; kalau nggak ada nilai 
buat _key_ itu di hash map-nya, `get` bakal balikin `None`. Program ini nanganin 
`Option`-nya pake manggil `copied` buat dapet `Option<i32>` bukannya 
`Option<&i32>`, terus pake `unwrap_or` buat nge-set `score` jadi nol kalau 
`scores` nggak punya entri buat _key_ tersebut.

Kita bisa iterasi lewat tiap pasangan key-value (kunci-nilai) di hash map pake 
cara yang mirip kayak di vectors, pake `for` loop:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Kode ini bakal nyetak tiap pasangan dengan urutan yang sembarangan (arbitrary):

```text
Yellow: 50
Blue: 10
```

### Hash Maps dan Ownership

Buat tipe-tipe yang mengimplementasikan trait `Copy`, kayak `i32`, nilainya 
di-copy ke dalem hash map. Buat nilai yang dimiliki (_owned values_) kayak 
`String`, nilainya bakal di-_move_ dan hash map bakal jadi pemilik (_owner_) 
dari nilai-nilai itu, kayak yang didemonstrasiin di Listing 8-22.

<Listing number="8-22" caption="Nunjukin kalau keys sama nilai itu jadi milik hash map begitu mereka di-_insert_">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Kita nggak bisa pake variabel `field_name` sama `field_value` setelah mereka di-
_move_ ke dalem hash map pake pemanggilan `insert`.

Kalau kita masukin referensi ke nilai ke dalem hash map, nilainya nggak bakal 
di-_move_ ke dalem hash map-nya. Nilai yang ditunjuk sama referensi itu harus 
tetep valid setidaknya selama hash map-nya valid. Kita bakal bahas masalah ini 
lebih banyak di [“Mervalidasi Referensi pake Lifetimes”][validating-references-with-lifetimes] 
di Bab 10.

### Ngubah Hash Map

Walaupun jumlah pasangan key sama value bisa nambah, tiap _unique key_ cuma bisa 
punya satu nilai yang terkait dengannya dalam satu waktu (tapi nggak berlaku 
sebaliknya: misalnya, baik tim Blue maupun tim Yellow bisa punya nilai `10` 
yang disimpan di hash map `scores`).

Pas kita mau ngubah data di dalem hash map, kita harus mutusin gimana cara 
nanganin kasus pas sebuah _key_ udah punya nilai yang di-assign ke dia. Kita bisa 
nggantiin nilai lama pake nilai baru, sama sekali nyuekin nilai yang lama. Kita 
bisa pertahanin nilai lama dan nyuekin nilai baru, dan cuma nambahin nilai baru 
kalau _key_ itu _belum_ punya nilai. Atau kita bisa ngegabungin nilai lama sama 
nilai baru. Yuk kita liat gimana cara ngelakuin semua hal ini!

#### Nindih Nilai (Overwriting a Value)

Kalau kita nge-_insert_ sebuah key sama nilai ke hash map terus nge-_insert_ 
key yang sama pake nilai yang beda, nilai yang terkait sama key itu bakal 
diganti. Walaupun kode di Listing 8-23 manggil `insert` dua kali, hash map-nya 
cuma bakal punya satu pasangan key-value karena kita nge-_insert_ nilai buat 
key tim Blue dua kali.

<Listing number="8-23" caption="Nggantiin nilai yang disimpan pake key tertentu">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Kode ini bakal nyetak `{"Blue": 25}`. Nilai asli `10` udah ditindih.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Nambahin Key dan Nilai Cuma Kalau Key-nya Belum Ada

Sering sekali kita mau nge-cek apakah sebuah key tertentu udah ada di hash map 
dengan suatu nilai terus ngambil tindakan berikut: kalau key itu emang udah ada 
di hash map, nilai yang udah ada biarin aja kayak gitu; kalau key-nya belum ada, 
_insert_ key itu bareng nilainya.

Hash maps punya API khusus buat ini namanya `entry` yang nerima key yang mau 
kita cek sebagai parameternya. Nilai return dari method `entry` ini adalah enum 
namanya `Entry` yang ngewakilin nilai yang mungkin udah ada atau mungkin belum. 
Katakanlah kita mau nge-cek apakah key buat tim Yellow punya nilai yang terkait 
dengannya. Kalau belum ada, kita mau nge-_insert_ nilai `50`, dan lakuin hal yang 
sama buat tim Blue. Pake API `entry`, kodenya keliatan kayak Listing 8-24.

<Listing number="8-24" caption="Pake method `entry` buat nge-_insert_ cuma kalau key-nya belum punya nilai">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

Method `or_insert` pada `Entry` didefinisikan buat balikin sebuah _mutable 
reference_ ke nilai buat key `Entry` yang terkait kalau key itu ada, dan kalau 
nggak ada, dia nge-_insert_ parameternya sebagai nilai baru buat key ini terus 
balikin _mutable reference_ ke nilai yang baru. Teknik ini jauh lebih bersih 
daripada nulis logikanya sendiri dan, selain itu, main lebih akur sama _borrow 
checker_.

Jalanin kode di Listing 8-24 bakal nyetak `{"Yellow": 50, "Blue": 10}`. 
Pemanggilan `entry` yang pertama bakal nge-_insert_ key buat tim Yellow dengan 
nilai `50` karena tim Yellow belum punya nilai. Pemanggilan `entry` yang kedua 
nggak bakal ngubah hash map-nya karena tim Blue udah punya nilai `10`.

#### Ngubah Nilai Berdasarkan Nilai Lamanya

Skenario umum lainnya buat hash maps adalah nyari nilai dari sebuah key terus 
ngubah nilainya berdasarkan nilai yang lama. Misalnya, Listing 8-25 nunjukin 
kode yang ngitung berapa kali tiap kata muncul di sebuah teks. Kita pake hash 
map dengan kata-katanya sebagai keys dan nambahin (increment) nilainya buat 
nyatet berapa kali kita ngeliat kata itu. Kalau ini pertama kalinya kita liat 
kata itu, kita bakal nge-_insert_ nilai `0` dulu.

<Listing number="8-25" caption="Ngitung kemunculan kata-kata pake hash map yang nyimpen kata dan jumlahnya">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Kode ini bakal nyetak `{"world": 2, "hello": 1, "wonderful": 1}`. Kita mungkin 
bakal ngeliat pasangan key-value yang sama dicetak pake urutan yang beda: inget 
kan di [“Akses Nilai di dalem Hash Map”][access] kalau iterasi lewat hash map 
itu terjadi dengan urutan yang sembarangan (arbitrary).

Method `split_whitespace` balikin iterator yang ngelewatin *subslices*, 
dipisahin sama spasi (whitespace), dari nilai di `text`. Method `or_insert` 
balikin sebuah _mutable reference_ (`&mut V`) ke nilai buat key yang spesifik 
itu. Di sini, kita nyimpen _mutable reference_ itu di variabel `count`, jadi 
buat nge-assign (ngasih nilai) ke nilai itu, kita pertama-tama harus pake _dereference_ 
pada `count` pake tanda bintang (`*`). _Mutable reference_-nya keluar dari 
scope di akhir dari `for` loop, jadi semua perubahan ini aman dan dibolehin sama 
aturan _borrowing_.

### Fungsi Hashing

Secara default, `HashMap` pake fungsi hashing namanya _SipHash_ yang bisa ngasih 
ketahanan dari serangan _denial-of-service (DoS)_ yang ngelibatin _hash 
tables_[^siphash]. Ini bukan algoritma hashing paling cepet yang ada, tapi 
_trade-off_ buat keamanan yang lebih baik dengan sedikit penurunan performa 
itu sepadan sekali. Kalau kita nge-_profile_ kode kita dan ngerasa kalau 
fungsi hash default-nya terlalu lambat buat tujuan kita, kita bisa ganti ke 
fungsi lain dengan nentuin _hasher_ yang beda. Sebuah _hasher_ adalah tipe yang 
mengimplementasikan trait `BuildHasher`. Kita bakal bahas traits dan cara 
mengimplementasikan mereka di [Bab 10][traits]. Kita nggak perlu harus 
mengimplementasikan hasher kita sendiri dari nol kok; 
[crates.io](https://crates.io/) punya library-library yang di-share sama _user_ 
Rust lainnya yang nyediain hashers yang mengimplementasikan banyak algoritma 
hashing umum.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Ringkasan

Vectors, strings, sama hash maps bakal nyediain banyak fungsionalitas yang 
dibutuhin di program kita pas kita perlu nyimpen, akses, dan ngubah data. Ini 
ada beberapa latihan yang sekarang harusnya udah bisa kita selesein:

1. Dikasih sekumpulan integer, pake sebuah vector dan balikin median (pas 
   diurutin, nilai di posisi tengah) sama modus (nilai yang paling sering 
   muncul; hash map bakal ngebantu sekali di sini) dari sekumpulan nilai itu.
1. Convert strings ke _pig latin_. Konsonan pertama dari tiap kata dipindah ke 
   akhir kata terus ditambahin _ay_, jadi _first_ jadi _irst-fay_. Kata-kata 
   yang diawali sama huruf vokal ditambahin _hay_ di akhirnya (_apple_ jadi 
   _apple-hay_). Inget detail soal _encoding_ UTF-8 ya!
1. Pake hash map sama vectors, bikin interface teks buat ngebolehin user 
   nambahin nama pegawai ke sebuah departemen di sebuah perusahaan; misalnya, 
   “Tambahin Sally ke Engineering” atau “Tambahin Amir ke Sales.” Terus 
   bolehin user buat narik (retrieve) daftar semua orang di suatu departemen 
   atau semua orang di perusahaan berdasarkan departemen, diurutin secara alfabet.

Dokumentasi API _standard library_ ngejelasin method-method yang dipunyai sama 
vectors, strings, sama hash maps yang bakal ngebantu sekali buat latihan-
latihan ini!

Kita lagi masuk ke program-program yang lebih kompleks di mana operasi bisa aja 
gagal, jadi ini waktu yang paling pas buat bahas penanganan error (error 
handling). Kita bakal ngelakuin itu berikutnya!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
