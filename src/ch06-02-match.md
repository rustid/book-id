<!-- Old heading. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## Konstruk Control Flow `match`

Rust punya konstruk _control flow_ yang sangat kuat (powerful) namanya `match` 
yang ngebolehin kita bandingin sebuah nilai sama serangkaian pattern (pola) 
terus ngejalanin kode berdasarkan pattern mana yang cocok. Pattern bisa dibuat 
dari nilai literal, nama variabel, _wildcards_, dan banyak hal lainnya; 
[Bab 19][ch19-00-patterns] ngebahas semua jenis pattern yang beda dan apa fungsinya. 
Kekuatan dari `match` dateng dari ekspresifitas pattern-nya dan fakta kalau 
_compiler_ mastiin semua kemungkinan kasus udah di-handle.

Bayangin ekspresi `match` itu kayak mesin penyortir koin: koin meluncur turun di 
jalur yang punya lubang dengan berbagai ukuran, dan tiap koin bakal jatuh ke 
lubang pertama yang pas buat dia. Dengan cara yang sama, nilai bakal ngelewatin 
tiap pattern di sebuah `match`, dan di pattern pertama di mana nilainya “pas,” 
nilai itu bakal masuk ke blok kode terkait buat dipake pas eksekusi.

Ngomong-ngomong soal koin, yuk kita pake mereka sebagai contoh buat `match`! 
Kita bisa nulis fungsi yang nerima koin US yang nggak tau jenisnya apa dan, 
dengan cara yang mirip kayak mesin penghitung, nentuin koin apa itu terus 
balikin nilainya dalam satuan sen (_cents_), kayak yang ditunjukin di Listing 6-3.

<Listing number="6-3" caption="Sebuah enum dan ekspresi `match` yang punya varian enum sebagai pattern-nya">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

Yuk kita bedah ekspresi `match` di fungsi `value_in_cents`. Pertama kita nulis 
keyword `match` diikuti sama sebuah ekspresi, yang di kasus ini adalah nilai 
`coin`. Ini keliatannya mirip sekali sama ekspresi kondisional yang dipake bareng 
`if`, tapi ada perbedaan gede: kalau pake `if`, kondisinya harus dievaluasi jadi 
nilai Boolean, tapi di sini dia bisa jadi tipe apa pun. Tipe dari `coin` di 
contoh ini adalah enum `Coin` yang kita definisikan di baris pertama.

Selanjutnya adalah _arms_ (lengan) dari `match`. Sebuah arm punya dua bagian: 
sebuah pattern dan sejumlah kode. Arm pertama di sini punya pattern yaitu nilai 
`Coin::Penny` dan terus operator `=>` yang misahin pattern sama kode yang bakal 
dijalanin. Kode di kasus ini cuma nilai `1`. Tiap arm dipisahin dari arm 
berikutnya pake tanda koma.

Pas ekspresi `match` jalan, dia bandingin nilai hasilnya sama pattern dari tiap 
arm, secara berurutan. Kalau ada pattern yang cocok sama nilainya, kode yang 
terkait sama pattern itu bakal dijalanin. Kalau pattern itu nggak cocok sama 
nilainya, eksekusi bakal lanjut ke arm berikutnya, sama persis kayak di mesin 
penyortir koin. Kita bisa punya sebanyak apa pun arm yang kita butuhin: di 
Listing 6-3, `match` kita punya empat arm.

Kode yang terkait sama tiap arm itu adalah sebuah ekspresi, dan nilai hasil dari 
ekspresi di arm yang cocok adalah nilai yang bakal dibalikin buat seluruh 
ekspresi `match`-nya.

Kita biasanya nggak pake kurung kurawal kalau kode match arm-nya pendek, kayak 
di Listing 6-3 di mana tiap arm cuma balikin sebuah nilai. Kalau kita mau 
ngejalanin beberapa baris kode di sebuah match arm, kita wajib pake kurung 
kurawal, dan koma setelah arm-nya itu jadi opsional. Misalnya, kode berikut 
nyetak “Lucky penny!” tiap kali method-nya dipanggil pake `Coin::Penny`, tapi 
tetep balikin nilai terakhir dari blok-nya, yaitu `1`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Pattern yang Nge-bind ke Nilai

Fitur berguna lainnya dari match arms adalah mereka bisa nge-bind ke bagian-
bagian nilai yang cocok sama pattern-nya. Ini cara kita ngekstrak nilai keluar 
dari varian enum.

Sebagai contoh, yuk kita ubah salah satu varian enum kita biar nyimpen data di 
dalemnya. Dari tahun 1999 sampe 2008, Amerika Serikat nyetak koin _quarter_ 
(25 sen) dengan desain beda-beda buat tiap 50 negara bagian di satu sisinya. 
Nggak ada koin lain yang dapet desain negara bagian, jadi cuma _quarter_ yang 
punya nilai ekstra ini. Kita bisa nambahin informasi ini ke `enum` kita dengan 
ngerubah varian `Quarter` biar masukin nilai `UsState` yang disimpan di dalemnya, 
kayak yang kita lakuin di Listing 6-4.

<Listing number="6-4" caption="Sebuah enum `Coin` di mana varian `Quarter` juga nyimpen nilai `UsState`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Bayangin ada temen kita yang lagi nyoba ngumpulin semua 50 _state quarters_. 
Pas kita lagi nyortir uang receh berdasarkan jenis koinnya, kita juga bakal 
nyebutin nama negara bagian yang terkait sama tiap _quarter_ biar kalau temen 
kita belum punya yang itu, dia bisa nambahin ke koleksinya.

Di ekspresi `match` buat kode ini, kita nambahin variabel namanya `state` ke 
pattern yang nyocokin nilai varian `Coin::Quarter`. Pas sebuah `Coin::Quarter` 
cocok, variabel `state` bakal di-bind ke nilai negara bagian dari _quarter_ itu. 
Terus kita bisa pake `state` di kode buat arm itu, kayak gini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Kalau kita manggil `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` bakal 
jadi `Coin::Quarter(UsState::Alaska)`. Pas kita bandingin nilai itu sama tiap 
match arms, nggak ada satu pun yang cocok sampe kita nyampe `Coin::Quarter(state)`. 
Di titik itu, _binding_ buat `state` bakal jadi nilai `UsState::Alaska`. Kita 
terus bisa pake _binding_ itu di ekspresi `println!`, dan dengan gitu kita dapet 
nilai _state_ dalemnya keluar dari varian enum `Coin` buat `Quarter`.

### Matching dengan `Option<T>`

Di bagian sebelumnya, kita mau dapet nilai `T` di dalem dari kasus `Some` pas 
lagi pake `Option<T>`; kita juga bisa nanganin `Option<T>` pake `match`, sama 
kayak yang kita lakuin sama enum `Coin`! Bukannya ngebandingin koin, kita bakal 
ngebandingin varian `Option<T>`, tapi cara kerja ekspresi `match`-nya tetep sama.

Katakanlah kita mau nulis fungsi yang nerima sebuah `Option<i32>` dan, kalau ada 
nilai di dalemnya, nambahin 1 ke nilai itu. Kalau nggak ada nilainya, fungsi itu 
harus balikin nilai `None` dan nggak nyoba ngelakuin operasi apa pun.

Fungsi ini gampang sekali ditulis, berkat `match`, dan bakal keliatan kayak 
Listing 6-5.

<Listing number="6-5" caption="Fungsi yang pake ekspresi `match` pada sebuah `Option<i32>`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

Yuk kita teliti eksekusi pertama dari `plus_one` lebih dalem. Pas kita manggil 
`plus_one(five)`, variabel `x` di body `plus_one` bakal punya nilai `Some(5)`. 
Kita terus bandingin itu sama tiap match arm:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Nilai `Some(5)` nggak cocok sama pattern `None`, jadi kita lanjut ke arm 
berikutnya:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

Apakah `Some(5)` cocok sama `Some(i)`? Cocok dong! Kita dapet varian yang sama. 
`i` di-bind ke nilai yang ditampung di `Some`, jadi `i` ngambil nilai `5`. Kode 
di match arm itu kemudian dijalanin, jadi kita nambahin 1 ke nilai `i` dan 
bikin nilai `Some` baru dengan total `6` kita di dalemnya.

Sekarang yuk kita pertimbangkan pemanggilan kedua dari `plus_one` di Listing 6-5, 
di mana `x` itu `None`. Kita masuk ke `match` dan bandingin sama arm pertama:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Cocok! Nggak ada nilai buat ditambahin, jadi programnya berhenti dan balikin 
nilai `None` di sisi kanan `=>`. Karena arm pertama cocok, nggak ada arm lain 
yang dibandingin.

Gabungin `match` sama enum itu berguna di banyak situasi. Kita bakal sering 
liat pola ini di kode Rust: nge-`match` terhadap sebuah enum, nge-bind variabel 
ke data di dalemnya, terus ngejalanin kode berdasarkan data itu. Agak ribet di 
awal emang, tapi sekali kita udah terbiasa, kita bakal ngarep ini ada di semua 
bahasa. Ini terus-terusan jadi favorit user.

### Matches Itu Exhaustive (Menyeluruh)

Ada satu aspek lain dari `match` yang perlu kita bahas: pattern-pattern di 
arm-nya harus mencakup semua kemungkinan. Coba liat versi dari fungsi `plus_one` 
kita ini, yang punya _bug_ dan nggak bakal bisa di-compile:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Kita nggak nge-handle kasus `None`, jadi kode ini bakal nyebabin _bug_. Untungnya, 
ini adalah _bug_ yang Rust tau cara nangkepnya. Kalau kita nyoba compile kode 
ini, kita bakal dapet error kayak gini:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust tau kalau kita nggak mencakup semua kasus yang mungkin, dan bahkan tau 
pattern mana yang kita kelupaan! Matches di Rust itu _exhaustive_: kita harus 
ngabisin setiap kemungkinan yang ada biar kodenya jadi valid. Terutama buat kasus 
`Option<T>`, pas Rust nyegah kita dari lupa buat secara eksplisit nanganin kasus 
`None`, dia ngelindungin kita dari ngasumsikan kalau kita punya nilai padahal 
mungkin kita punya null, makanya ini ngebikin kesalahan satu miliar dolar yang 
dibahas sebelumnya jadi mustahil terjadi.

### Catch-All Patterns dan Placeholder `_`

Pake enum, kita juga bisa ngambil aksi khusus buat beberapa nilai tertentu, tapi 
buat semua nilai lainnya kita mau ngambil satu aksi default. Bayangin kita lagi 
bikin game di mana kalau kita ngelempar dadu dan dapet 3, player kita nggak gerak, 
tapi malah dapet topi fancy baru. Kalau dapet 7, player kita kehilangan topi 
fancy-nya. Buat semua nilai lainnya, player kita maju sejumlah langkah sesuai 
angka itu di papan game. Ini ekspresi `match` yang mengimplementasikan logika itu, 
dengan hasil lemparan dadu yang di-_hardcoded_ bukannya nilai random, dan semua 
logika lainnya direpresentasikan sama fungsi tanpa body karena implementasi 
aslinya di luar cakupan contoh ini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Buat dua arm pertama, pattern-nya adalah nilai literal `3` dan `7`. Buat arm 
terakhir yang nyakup semua kemungkinan nilai lainnya, pattern-nya adalah 
variabel yang kita pilih buat dikasih nama `other`. Kode yang jalan buat arm 
`other` pake variabel itu dengan masukin dia ke fungsi `move_player`.

Kode ini berhasil di-compile, walaupun kita belum nyebutin semua nilai yang 
mungkin dipunyai `u8`, karena pattern terakhir bakal nyocokin semua nilai yang 
nggak disebutin secara spesifik. _Catch-all pattern_ ini menuhin syarat kalau 
`match` itu harus _exhaustive_. Perhatiin ya kalau kita harus naruh arm 
catch-all di paling akhir karena pattern-nya dievaluasi berurutan. Kalau kita 
naruh arm catch-all lebih awal, arm lainnya nggak bakal pernah jalan, jadi 
Rust bakal ngasih tau kita kalau kita nambahin arm setelah catch-all!

Rust juga punya pattern yang bisa kita pake pas kita butuh catch-all tapi nggak 
mau _pake_ nilainya di pattern catch-all itu: `_` adalah pattern khusus yang 
nyocokin nilai apa pun dan nggak nge-bind ke nilai itu. Ini ngasih tau Rust 
kalau kita nggak bakal pake nilainya, jadi Rust nggak bakal ngasih warning soal 
variabel yang nggak kepake.

Yuk kita ubah aturan gamenya: sekarang, kalau kita ngelempar dadu selain angka 
3 atau 7, kita harus ngelempar lagi. Kita udah nggak perlu pake nilai catch-all, 
jadi kita bisa ubah kode kita pake `_` bukannya variabel namanya `other`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Contoh ini juga menuhin syarat _exhaustiveness_ karena secara eksplisit kita 
ngabaikan semua nilai lain di arm terakhir; kita nggak ada yang kelupaan.

Terakhir, kita bakal ubah aturan gamenya sekali lagi biar nggak ada yang terjadi 
pas giliran kita kalau kita ngelempar selain angka 3 atau 7. Kita bisa ekspresikan 
itu pake nilai unit (tipe tuple kosong yang pernah kita sebut di bagian [“Tipe 
Tuple”][tuples]) sebagai kode yang nyertain arm `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Di sini, kita ngasih tau Rust secara eksplisit kalau kita nggak bakal pake nilai 
lain apa pun yang nggak cocok sama pattern di arm sebelumnya, dan kita nggak 
mau ngejalanin kode apa pun di kasus ini.

Masih banyak hal lain soal pattern dan matching yang bakal kita bahas di [Bab 19][ch19-00-patterns]. 
Buat sekarang, kita bakal lanjut ke sintaks `if let`, yang bisa kepake di 
situasi-situasi di mana ekspresi `match` dirasa agak kepanjangan (wordy).

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
