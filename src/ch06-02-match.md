<!-- Old heading. Do not remove or links may break. -->
<a id="the-match-control-flow-operator"></a>
## Aliran kontrol `match`

Rust mempunyai alur kontrol yang sangat kuat yaitu  `match` itu mengizinkan anda 
untuk membandingkan nilai dengan serangkaian pola dan kemudian mengeksekusi kode 
berdasaran pola yang cocok. Pola atau pattern dapat terdiri dari nilai literal, 
nama variabel, wildcards, dan yang lainnya; [Bab 18][ch18-00-patterns]<!-- ignore -->
mencakup semua jenis pola yang berbeda dan apa yang saja yang bisa mereka lakukan. 
Kekuatan `match` berasal dari ekspresi pola dan fakta bahwa kompiler mengkonfirmasi 
bahwa semua kasus yang mungkin terjadi harus ditangani. 

Anggaplah ekspresi `match` seperti mesin pengurut koin: koin meluncur menyusuri trek 
dengan berbagai ukuran lubang di sepanjang itu, dan setiap koin jatuh melaluinya lubang
pertama yang ditemuinya yang cocok dengannya. Demikian pula halnya dengan nilai-nilai
melalui setiap pola dalam `match`, dan pada pola pertama yang nilainya "cocok", nilainya
termasuk dalam blok kode terkait untuk digunakan selama eksekusi.

Berbicara soal koins, mari menggunakannya sebagai contoh untuk penggunaan `match`! 
Kita bisa menulis fungsi yang mengambil koin US tidak diketahui dan, di situasi
yang sama sebagai mesin penghitung, menentukan koin mana dan mengembalikan nilainya 
dalam sen, seperti yang ditunjukkan di Daftar 6-3.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

<span class="caption">Daftar 6-3: Enum dan ekspresi `match` yang memiliki
variant enum sebagai polanya.</span>

Mari kita uraikan `match` dalam fungsi `value_in_cents`. Pertama-tama kita lihat 
kata kunci `match` diikuti dengan ekspresi, yang dalam hal ini nilainya adalah `coin`. 
Ini nampaknya sangat mirip dengan ekspresi kondisional yang digunakan dengan `if`, 
tetapi ada perbedaan besar: dengan `if`, kondisinya perlu dievaluasi menjadi sebuah
nilai boolean, tapi ini bisa tipe apa saja. Jenis `coin` dalam contoh ini adalah enum
`Coin` yang kita tentukan pada baris pertama.

Berikutnya adalah lengan `match` atau match arms. Sebuah lengan memiliki dua bagian: 
pola atau pattern dan beberapa kode atau ekspresi. Lengan pertama di sini memiliki 
pola yaitu nilai `Coin::Penny` dan kemudian `=>`operator yang memisahkan pola dan
kode yang akan dijalankan. Kode dalam hal ini hanyalah nilai `1`. Setiap lengan 
dipisahkan dari lengan berikutnya dengan koma.

Ketika ekspresi `match` dijalankan, ia membandingkan nilai yang dihasilkan
pola masing-masing lengan, secara berurutan. Jika suatu pola cocok dengan nilainya, kodenya
terkait dengan pola itu dieksekusi. Jika pola itu tidak cocok dengan nilai, eksekusi
berlanjut ke kelompok berikutnya atau arm berikutnya, seperti halnya pada mesin pengurut koin.
Kita dapat memiliki senjata sebanyak yang kita butuhkan: pada Daftar 6-3, `match` kita
memiliki empat lengan.

Kode yang terkait dengan masing-masing lengan adalah ekspresi, dan nilai yang dihasilkan
ekspresi di lengan yang cocok adalah nilai yang dikembalikan untuk seluruh ekspresi `match`.

Kita biasanya tidak menggunakan tanda kurung kurawal jika kode lengan pencocokannya pendek
di Daftar 6-3 di mana masing-masing lengan hanya mengembalikan nilai. Jika Anda ingin 
menjalankan banyak baris kode di match arm, Anda harus menggunakan tanda kurung kurawal, dan koma
mengikuti lengan adalah opsional. Misalnya, kode berikut mencetak "Lucky sen!" setiap kali 
metode dipanggil dengan `Coin::Penny`, tapi tetap saja mengembalikan nilai terakhir blok, `1`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Pola yang mengikat nilai

Fitur berguna lainnya dari lengan match atau match arm adalah dapat mengikat bagian-bagian nilai 
yang sesuai dengan polanya. Inilah cara kita mengekstrak nilai dari enum variant.

Sebagai contoh, mari kita ubah salah satu variant enum untuk menyimpan data di dalamnya.
Pada tahun 1999 hingga 2008, Amerika Serikat mencetak uang kertas yang berbeda-beda desain
untuk masing-masing dari 50 negara bagian di satu sisi. Tidak ada koin lain yang mendapat status
desain, jadi hanya seperempat yang memiliki nilai ekstra ini. Kita dapat menambahkan informasi ini ke
`enum` kita, dengan mengubah variant `Quarter` untuk menyertakan nilai `UsState` yang disimpan 
di dalamnya, yang telah kita lakukan di Daftar 6-4.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

<span class="caption">Daftar 6-4: Sebuah enum `Coin` dimana variant `Quarter`
juga memiliki nilai `UsState`</span>

Mari kita bayangkan bahwa seorang teman sedang mencoba untuk mengumpulkan seluruh quarters 
pada 50 bagian negara. Ketika kita mengurutkan recehan berdasarkan jenis koin, kita juga 
akan menyebutkan nama negara bagian yang terkait dengan setiap kuartal sehingga jika itu
adalah salah satu yang tidak dimiliki teman kita, mereka dapat menambahkannya ke koleksi mereka.

Dalam ekspresi match untuk kode ini, kita menambahkan variabel bernama `state` ke pola 
yang cocok dengan variant `Coin::Quarter`. Ketika sebuah `Coin::Quarter` cocok, 
variabel `state` akan terikat ke nilai negara bagian seperempat. Lalu kita bisa menggunakan
`state` dalam kode untuk lengan itu, seperti berikut:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Jika kita memanggil `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` akan menjadi
`Coin::Quarter(UsState::Alaska)`. Ketika kita membandingkan nilai itu dengan masing-masing
dari lengan match atau match arms, tidak ada satupun yang cocok hingga kita mencapai 
`Coin::Quarter(state)`. Pada saat itu nilai dari ikatan `state` akan berupa `UsState::Alaska`. 
Selanjutnya kita bisa menggunakan ikatan nilai tersebut di ekspresi `println!`, sehingga 
mendapatkan nilai didalamnya dari nilai enum `Coin` untuk variant `Quarter`.

### Mencocokkan dengan `Option<T>`

Di bagian sebelumnya, kita ingin mendapatkan nilai `T` bagian dalam dari `Some`
di kasus saat ingin menggunakan `Option<T>`; kita juga dapat menangani `Option<T>` 
menggunakan `match`, seperti yang kita lakukan dengan enum `Coin`! Daripada
membandingkan koin, kita akan membandingkannya variant dari `Option<T>`, 
namun cara kerja ekspresi `match` tetap sama sama.

Katakanlah kita ingin menulis fungsi yang mengambil `Option<i32>` dan, jika
ada nilai di dalamnya, tambahkan 1 pada nilai itu. Jika tidak ada nilai di dalamnya,
fungsi tersebut harus mengembalikan nilai `None` dan tidak mencoba untuk 
melakukan operasi apa pun.

Fungsi ini sangat mudah untuk ditulis, berkat `match`, dan akan terlihat seperti 
di Daftar 6-5.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

<span class="caption">Listing 6-5: Sebuah fungsi yang menggunakan ekspresi `match` 
pada `Option<i32>`</span>

Mari kita periksa eksekusi pertama `plus_one` secara lebih rinci. Saat kita memanggil
`plus_one(five)`, variabel `x` di badan `plus_one` akan memiliki nilai `Some(5)`.
Kita kemudian membandingkannya dengan setiap lengan atau match arm:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```
Nilai `Some(5)` tidak cocok dengan pola `None`, jadi kita lanjutkan ke arm atau lengan selanjutnya.

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

Apakah `Some(5)` cocok dengan `Some(i)`? Benar! Kita memiliki variant yang sama. 'i'
mengikat nilai yang terdapat dalam `Some`, jadi `i` mengambil nilai `5`. Kode di
lengan yang cocok kemudian dieksekusi, jadi kita menambahkan 1 ke nilai `i` dan membuat 
sebuah nilai `Some` baru dengan total `6` di dalamnya.

Sekarang mari kita perhatikan pemanggilan kedua `plus_one` pada Daftar 6-5, di mana `x`
adalah `None`. Kita masuk ke `match` dan membandingkannya dengan lengan pertama:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Cocok! Tidak ada nilai yang dapat ditambahkan, sehingga program berhenti dan mengembalikan nilai tersebut
Nilai `None` di sisi kanan `=>`. Karena lengan pertama cocok, tidak ada lengan lain yang dibandingkan.

Menggabungkan `match` dan enum berguna dalam banyak situasi. Anda akan melihat banyak pola ini
dalam kode Rust: `match` dengan enum, ikat variabel ke data di dalamnya, lalu jalankan kode 
berdasarkan data tersebut. Ini agak rumit pada awalnya, tapi setelah Anda terbiasa, Anda 
akan berharap memilikinya dalam semua bahasa. Itu secara konsisten menjadi favorit pengguna.

### Kecocokan Sangat Lengkap

Ada satu aspek lain dari `match` yang perlu kita diskusikan: pola lengan harus
mencakup semua kemungkinan. Pertimbangkan versi fungsi `plus_one` kita ini,
yang memiliki bug dan tidak dapat dikompilasi:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Kita tidak menangani kasus `None`, jadi kode ini akan menyebabkan bug. Untungnya
bug itu diketahui oleh Rust. Jika kita mencoba mengkompilasi kode ini, kita akan
mendapatkan ini kesalahan:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust mengetahui bahwa kita tidak menangani setiap kasus yang mungkin terjadi,
dan bahkan mengetahui kasus pola mana yang kita lupa! Kecocokan di Rust *menyeluruh*: 
kita harus menangani seluruh kemungkinan agar kode tersebut valid. Terutama dalam kasus
`Option<T>`, ketika Rust mencegah kita lupa menanganinya secara eksplisit kasus `None`, 
Ini melindungi kita dari asumsi bahwa kita memiliki nilai padahal kita bisa memiliki null, 
sehingga membuat kesalahan miliaran dolar yang dibahas sebelumnya menjadi mustahil.

### Tangkap semua Pola dan Penampung `_` 

Dengan menggunakan enum, kita juga dapat mengambil tindakan khusus untuk beberapa nilai 
tertentu, namun untuk semua nilai lainnya, ambil satu tindakan default. Bayangkan kita
sedang mengimplementasikan sebuah game dimana jika anda melempar angka 3 pada pelemparan dadu,
pemain anda tidak bergerak, melainkan mendapat topi mewah baru. Jika Anda mendapatkan angka 7,
pemain Anda kehilangan topi mewahnya. Untuk semua nilai lainnya, pemain Anda memindahkan 
sejumlah ruang di papan permainan. Ini dia sebuah `match` yang mengimplementasikan logika
tersebut, dengan hasil pelemparan dadu dikodekan daripada nilai acak, dan semua logika 
lainnya diwakili oleh fungsi tanpa badan karena pelaksanaannya berada di luar jangkauan
contohnya seperti berikut:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Untuk dua kelompok pertama, polanya adalah nilai literal `3` dan `7`. Untuk
lengan terakhir yang mencakup setiap nilai lain, polanya adalah variabel 
yang kita pilih untuk diberi nama `other`. Kode yang dijalankan untuk 
lengan `other` menggunakan variabel dengan meneruskannya ke fungsi `move_player`.

Kode ini dikompilasi, meskipun kita belum mencantumkan semua kemungkinan nilai pada `u8`,
karena pola terakhir akan cocok dengan semua nilai tidak secara spesifik terdaftar.
Pola umum ini memenuhi persyaratan bahwa `match` harus menangani semuanya. Perhatikan
bahwa kita harus meletakkan lengan penampung semua di bagian terakhir karena pola 
dievaluasi secara berurutan. Jika kita meletakkan lengan penampung semua di awal, lengan 
yang lain tidak akan pernah dijalankan, jadi Rust akan memperingatkan kita jika kita 
menambahkan lengan setelah melakukan semuanya!

Rust juga memiliki pola yang dapat kita gunakan ketika kita menginginkan sesuatu yang 
mencakup semua hal tetapi tidak ingin *menggunakan* nilai dalam pola tersebut,
tangkap semua atau catch-all: `_` adalah pola khusus yang cocok dengan nilai apa pun
dan tidak mengikat  nilai itu. Ini memberi tahu Rust bahwa kita tidak akan menggunakan
nilainya, sehingga Rust tidak akan memperingatkan kita tentang variabel yang tidak digunakan.

Mari kita ubah aturan permainannya: sekarang, jika Anda mendapatkan angka selain 3 atau 7,
anda harus memutar lagi. Kita tidak perlu lagi menggunakan nilai umum, jadi kita
dapat mengubah kode kita untuk menggunakan `_` sebagai pengganti variabel bernama `other`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Contoh ini juga memenuhi persyaratan exhaustiveness atau penanganan pada semua kasus nya 
karena kita melakukannya secara eksplisit mengabaikan semua nilai lain di kelompok terakhir; 
kita tidak melupakan apa pun.

Terakhir, kita akan mengubah aturan mainnya sekali lagi sehingga tidak terjadi apapun 
pada giliran Anda jika anda mendapatkan angka selain 3 atau 7. Kita dapat menanganinya 
dengan menggunakan nilai unit (tuple kosong yang pernah kita bahas di ["The Tuple
Type"][tuples]<!-- ignore --> section) sebagai kode ekspresi di lengan `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Di sini, kita memberi tahu Rust secara eksplisit bahwa kita tidak akan menggunakan 
nilai lain apa pun yang tidak cocok dengan pola pada kelompok sebelumnya, dan kita
tidak ingin menjalankan kode apapun dalam hal ini.

Masih banyak lagi tentang pola dan pencocokan yang akan kita bahas di [Bab
18][ch18-00-patterns]<!-- ignore -->. Untuk saat ini, kita akan beralih ke
Sintaks `if let`, yang dapat berguna dalam situasi di mana ekspresi `match`
agak bertele-tele.

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch18-00-patterns]: ch18-00-patterns.html
