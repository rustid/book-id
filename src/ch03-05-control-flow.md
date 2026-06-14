## Control Flow

Kemampuan buat ngejalanin kode tergantung dari apakah sebuah kondisi itu `true` 
dan buat ngejalanin kode berulang kali pas sebuah kondisi itu `true` adalah blok 
dasar di kebanyakan bahasa pemrograman. Konstruk paling umum yang ngebolehin 
kita ngatur alur eksekusi kode Rust adalah ekspresi `if` sama loop.

### Ekspresi `if`

Ekspresi `if` ngebolehin kita buat nyabangin kode tergantung kondisinya. Kita 
ngasih sebuah kondisi terus bilang, “Kalau kondisi ini terpenuhi, jalanin blok 
kode ini. Kalau nggak terpenuhi, jangan jalanin blok kode ini.”

Bikin project baru namanya _branches_ di direktori _projects_ kita buat eksplor 
ekspresi `if`. Di file _src/main.rs_, masukin kode berikut:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Semua ekspresi `if` dimulai pake keyword `if`, diikuti sama sebuah kondisi. Di 
kasus ini, kondisinya nge-cek apakah variabel `number` punya nilai kurang dari 5. 
Kita taruh blok kode yang bakal jalan kalau kondisinya `true` tepat setelah 
kondisinya di dalem kurung kurawal. Blok kode yang terkait sama kondisi di 
ekspresi `if` kadang disebut _arms_ (lengan), sama kayak _arms_ di ekspresi 
`match` yang kita bahas di bagian [“Membandingkan Tebakan dengan Secret Number”][comparing-the-guess-to-the-secret-number] 
di Bab 2.

Opsionalnya, kita juga bisa masukin ekspresi `else`, kayak yang kita lakuin di 
sini, buat ngasih program blok kode alternatif buat jalan kalau kondisinya 
ternyata `false`. Kalau kita nggak ngasih ekspresi `else` dan kondisinya 
`false`, programnya bakal langsung lewatin blok `if` terus lanjut ke kode 
selanjutnya.

Coba jalanin kode ini; kita bakal dapet output kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Yuk coba ubah nilai `number` jadi nilai yang bikin kondisinya `false` buat liat 
apa yang terjadi:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Jalanin programnya lagi, terus liat output-nya:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Penting juga buat dicatet kalau kondisi di kode ini _harus_ sebuah `bool`. Kalau 
kondisinya bukan `bool`, kita bakal dapet error. Contohnya, coba jalanin kode 
berikut:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Kondisi `if` dievaluasi jadi nilai `3` kali ini, dan Rust ngelepar error:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Error-nya nunjukin kalau Rust ngarepin `bool` tapi dapetnya integer. Beda sama 
bahasa kayak Ruby sama JavaScript, Rust nggak bakal otomatis nyoba convert tipe 
non-Boolean jadi Boolean. Kita harus eksplisit dan selalu ngasih `if` sebuah 
Boolean sebagai kondisinya. Kalau kita mau blok kode `if` jalan cuma pas sebuah 
angka nggak sama dengan `0`, misalnya, kita bisa ubah ekspresi `if`-nya jadi 
kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Jalanin kode ini bakal nyetak `number was something other than zero`.

#### Handle Banyak Kondisi dengan `else if`

Kita bisa pake banyak kondisi dengan ngelempokin `if` sama `else` di sebuah 
ekspresi `else if`. Contohnya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Program ini punya empat kemungkinan jalur yang bisa diambil. Setelah jalanin 
programnya, kita bakal liat output kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Pas program ini jalan, dia cek tiap ekspresi `if` secara berurutan terus 
ngejalanin body pertama yang kondisinya dievaluasi jadi `true`. Inget ya 
walaupun 6 itu bisa dibagi 2, kita nggak liat output `number is divisible by 2`, 
dan kita juga nggak liat teks `number is not divisible by 4, 3, or 2` dari blok 
`else`. Itu karena Rust cuma ngejalanin blok buat kondisi `true` yang pertama, 
dan sekali dia nemu satu, dia bahkan nggak bakal cek sisanya.

Pake terlalu banyak ekspresi `else if` bisa bikin kode kita berantakan, jadi 
kalau kita punya lebih dari satu, mendingan di-refactor kodenya. Bab 6 jelasin 
konstruk percabangan Rust yang sangat kuat namanya `match` buat kasus-kasus 
kayak gini.

#### Pake `if` di Statement `let`

Karena `if` itu adalah sebuah ekspresi, kita bisa pakenya di sisi kanan 
statement `let` buat ngasih hasil kondisinya ke sebuah variabel, kayak di 
Listing 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Assign hasil dari ekspresi `if` ke sebuah variabel">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

Variabel `number` bakal di-bind ke sebuah nilai berdasarkan hasil dari ekspresi 
`if`. Jalanin kode ini buat liat apa yang terjadi:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Inget ya kalau blok kode dievaluasi jadi ekspresi terakhir di dalemnya, dan 
angka sendirian itu juga sebuah ekspresi. Di kasus ini, nilai dari seluruh 
ekspresi `if` tergantung dari blok kode mana yang jalan. Ini artinya nilai yang 
berpotensi jadi hasil dari tiap _arm_ di `if` harus punya tipe yang sama; di 
Listing 3-2, hasil dari baik _arm_ `if` maupun _arm_ `else` adalah integer 
`i32`. Kalau tipenya nggak cocok, kayak di contoh berikut, kita bakal dapet 
error:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Pas kita nyoba compile kode ini, kita bakal dapet error. _Arm_ `if` sama `else` 
punya tipe nilai yang nggak kompatibel, dan Rust nunjukin tepat di mana letak 
masalahnya di program kita:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

Ekspresi di blok `if` dievaluasi jadi integer, dan ekspresi di blok `else` 
dievaluasi jadi string. Ini nggak bakal bisa karena variabel harus punya tipe 
tunggal, dan Rust perlu tau pas _compile time_ tipe apa variabel `number` itu, 
secara definitif. Tau tipe dari `number` bikin _compiler_ bisa verifikasi kalau 
tipenya valid di mana pun kita pake `number`. Rust nggak bakal bisa ngelakuin 
itu kalau tipe `number` cuma bisa ditentuin pas _runtime_; _compiler_-nya bakal 
jadi lebih ribet dan bakal ngasih jaminan yang lebih dikit soal kodenya kalau 
dia harus jagain banyak tipe hipotetis buat variabel apa pun.

### Pengulangan dengan Loops

Sering kali berguna buat ngejalanin sebuah blok kode lebih dari sekali. Buat 
tugas ini, Rust nyediain beberapa jenis _loops_, yang bakal ngejalanin kode di 
dalem body loop sampe selesai terus langsung mulai lagi dari awal. Buat 
eksperimen sama loops, yuk kita bikin project baru namanya _loops_.

Rust punya tiga jenis loop: `loop`, `while`, dan `for`. Yuk kita coba satu-satu.

#### Ngulang Kode pake `loop`

Keyword `loop` ngasih tau Rust buat ngejalanin sebuah blok kode terus-menerus 
selamanya sampe kita eksplisit nyuruh dia berhenti.

Sebagai contoh, ubah file _src/main.rs_ di direktori _loops_ kita jadi kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Pas kita jalanin program ini, kita bakal liat `again!` dicetak terus-menerus 
tanpa henti sampe kita stop programnya secara manual. Kebanyakan terminal 
support keyboard shortcut <kbd>ctrl</kbd>-<kbd>c</kbd> buat interupsi program 
yang terjebak di loop terus-menerus. Cobain deh:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

Simbol `^C` merepresentasikan di mana kita teken <kbd>ctrl</kbd>-<kbd>c</kbd>.

Kita mungkin liat atau nggak liat kata `again!` dicetak setelah `^C`, tergantung 
di mana kodenya lagi ada di dalem loop pas dia nerima sinyal interupsi.

Untungnya, Rust juga nyediain cara buat keluar dari loop pake kode. Kita bisa 
naruh keyword `break` di dalem loop buat ngasih tau program kapan harus berhenti 
ngejalanin loop-nya. Inget kan kita udah lakuin ini di game tebak angka di bagian 
[“Quit Setelah Tebakan Bener”][quitting-after-a-correct-guess] di Bab 2 buat 
keluar dari program pas user menangin gamenya dengan nebak angka yang bener.

Kita juga pake `continue` di game tebak angka, yang di dalem loop ngasih tau 
program buat lewatin sisa kode di iterasi loop ini terus lanjut ke iterasi 
berikutnya.

#### Balikin Nilai dari Loops

Salah satu kegunaan `loop` itu buat nyoba lagi sebuah operasi yang kita tau 
mungkin gagal, kayak nge-cek apakah sebuah thread udah kelar tugasnya. Kita 
mungkin juga perlu masukin hasil dari operasi itu keluar dari loop ke sisa kode 
kita. Buat lakuin ini, kita bisa nambahin nilai yang mau dibalikin setelah 
ekspresi `break` yang kita pake buat stop loop-nya; nilai itu bakal dibalikin 
keluar dari loop biar bisa kita pake, kayak yang ditunjukin di sini:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Sebelum loop, kita mendeklarasikan variabel namanya `counter` terus 
diinisialisasi jadi `0`. Terus kita mendeklarasikan variabel namanya `result` 
buat nampung nilai yang dibalikin dari loop. Di tiap iterasi loop-nya, kita 
nambahin `1` ke variabel `counter`, terus cek apakah `counter` sama dengan 
`10`. Pas udah sama, kita pake keyword `break` bareng nilai `counter * 2`. 
Setelah loop, kita pake titik koma buat ngakhiri statement yang ngasih nilainya 
ke `result`. Akhirnya, kita nyetak nilai di `result`, yang di kasus ini 
hasilnya `20`.

Kita juga bisa `return` dari dalem loop. Kalau `break` cuma keluar dari loop 
saat ini, `return` bakal selalu keluar dari fungsi saat ini.

#### Loop Labels buat Bedain Banyak Loops

Kalau kita punya loop di dalem loop, `break` sama `continue` berlaku buat loop 
paling dalem di titik itu. Kita opsional bisa nentuin _loop label_ di sebuah 
loop yang terus bisa kita pake bareng `break` atau `continue` buat nentuin 
kalau keyword itu berlaku buat loop yang dikasih label bukannya loop paling 
dalem. Loop label harus dimulai pake kutip tunggal. Ini contoh dengan dua loop 
bersarang:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Loop luarnya punya label `'counting_up`, dan dia bakal ngitung dari 0 sampe 2. 
Loop dalemnya tanpa label ngitung mundur dari 10 sampe 9. `break` pertama yang 
nggak nentuin label cuma bakal keluar dari loop dalem aja. Statement 
`break 'counting_up;` bakal keluar dari loop luar. Kode ini nyetak:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### Conditional Loops dengan `while`

Sebuah program sering kali perlu nge-evaluasi sebuah kondisi di dalem loop. Pas 
kondisinya `true`, loop-nya jalan. Pas kondisinya udah nggak `true` lagi, 
programnya manggil `break`, yang stop loop-nya. Mungkin aja buat 
mengimplementasikan perilaku kayak gini pake kombinasi `loop`, `if`, `else`, 
sama `break`; kita bisa cobain itu sekarang di sebuah program kalau mau. Tapi, 
pola ini saking umumnya sampe Rust punya konstruk bahasa bawaan buat itu, 
namanya `while` loop. Di Listing 3-3, kita pake `while` buat ngulang programnya 
tiga kali, ngitung mundur tiap kalinya, terus setelah loop-nya kelar, nyetak 
pesan terus exit.

<Listing number="3-3" file-name="src/main.rs" caption="Pake `while` loop buat jalanin kode pas sebuah kondisi dievaluasi jadi `true` outdoor">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Konstruk ini ngilangin banyak _nesting_ (sarang) yang bakal diperluin kalau kita 
pake `loop`, `if`, `else`, sama `break`, dan dia lebih jelas. Selama sebuah 
kondisi dievaluasi jadi `true`, kodenya jalan; kalau nggak, dia keluar dari 
loop.

#### Looping Lewat Koleksi dengan `for`

Kita bisa milih buat pake konstruk `while` buat looping elemen-elemen dari 
sebuah koleksi, kayak array. Contohnya, loop di Listing 3-4 nyetak tiap elemen 
di array `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Looping lewat tiap elemen koleksi pake `while` loop">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Di sini, kodenya ngitung naik lewat elemen-elemen di array-nya. Dia mulai di 
indeks `0`, terus looping sampe nyampe indeks terakhir di array-nya (yaitu pas 
`index < 5` udah nggak `true` lagi). Jalanin kode ini bakal nyetak tiap elemen 
di array:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Semua lima nilai array muncul di terminal, sesuai ekspektasi. Walaupun `index` 
bakal nyampe nilai `5` di suatu titik, loop-nya berhenti jalan sebelum nyoba 
ngambil nilai keenam dari array-nya.

Tapi, pendekatan ini gampang bikin error; kita bisa bikin programnya _panic_ 
kalau nilai indeks atau kondisi tes-nya salah. Misalnya, kalau kita ngerubah 
definisi array `a` jadi punya empat elemen tapi lupa update kondisinya jadi 
`while index < 4`, kodenya bakal _panic_. Dia juga pelan, karena _compiler_ 
nambahin kode _runtime_ buat ngelakuin pengecekan kondisional apakah indeksnya 
masih di dalem batas array-nya di tiap iterasi loop-nya.

Sebagai alternatif yang lebih singkat, kita bisa pake `for` loop terus 
ngejalanin kode buat tiap item di koleksi. `for` loop keliatannya kayak kode di 
Listing 3-5.

<Listing number="3-5" file-name="src/main.rs" caption="Looping lewat tiap elemen koleksi pake `for` loop">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Pas kita jalanin kode ini, kita bakal liat output yang sama kayak di Listing 3-4. 
Yang lebih penting, sekarang kita udah ningkatin keamanan kodenya dan ngilangin 
kemungkinan _bug_ yang bisa hasil dari ngelewatin akhir array atau nggak cukup 
jauh dan ngelewatin beberapa item. Kode mesin yang dihasilin dari `for` loops 
juga bisa lebih efisien, karena indeksnya nggak perlu dibandingin sama panjang 
array-nya di tiap iterasi.

Pake `for` loop, kita nggak perlu repot-repot ngerubah kode lain kalau kita 
ngerubah jumlah nilai di array-nya, beda sama metode yang dipake di Listing 3-4.

Keamanan sama kesingkatan `for` loops bikin mereka jadi konstruk loop yang 
paling sering dipake di Rust. Bahkan di situasi di mana kita mau ngejalanin kode 
sejumlah kali tertentu, kayak di contoh hitung mundur yang pake `while` loop di 
Listing 3-3, kebanyakan _Rustacean_ bakal pake `for` loop. Caranya adalah pake 
`Range`, yang disediain sama standard library, yang nge-generate semua angka 
secara berurutan mulai dari satu angka dan berakhir sebelum angka lainnya.

Ini penampakan hitung mundur kalau pake `for` loop sama metode lain yang belum 
kita bahas, `rev`, buat nge-reverse (balikin) range-nya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Kode ini jauh lebih keren, kan?

## Ringkasan

Kita berhasil! Ini bab yang lumayan gede: kita udah belajar soal variabel, 
tipe data scalar sama compound, fungsi, komentar, ekspresi `if`, sama loop! 
Buat latihan konsep-konsep yang dibahas di bab ini, coba bikin program buat 
ngelakuin hal-hal berikut:

- Convert temperatur antara Fahrenheit sama Celsius.
- Generate angka Fibonacci ke-*n*.
- Nyetak lirik lagu Natal “The Twelve Days of Christmas,” manfaatin pengulangan 
  yang ada di lagunya.

Pas kita udah siap buat lanjut, kita bakal bahas konsep di Rust yang _nggak_ 
umum ada di bahasa pemrograman lain: ownership.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
