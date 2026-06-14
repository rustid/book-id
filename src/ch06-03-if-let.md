## Control Flow yang Ringkas pake `if let` sama `let else`

Sintaks `if let` ngebolehin kita ngegabungin `if` sama `let` jadi cara yang lebih 
ringkas buat nge-handle nilai yang cocok sama satu pattern sambil nyuekin sisanya. 
Coba liat program di Listing 6-6 yang nge-match nilai `Option<u8>` di variabel 
`config_max` tapi cuma mau ngejalanin kode kalau nilainya itu varian `Some`.

<Listing number="6-6" caption="Sebuah `match` yang cuma peduli buat ngejalanin kode pas nilainya `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Kalau nilainya `Some`, kita nyetak nilainya di varian `Some` dengan nge-bind 
nilai itu ke variabel `max` di dalem pattern-nya. Kita nggak mau ngelakuin apa-
apa sama nilai `None`. Buat menuhin syarat ekspresi `match`, kita harus nambahin 
`_ => ()` setelah memproses cuma satu varian, yang mana ini lumayan nyebelin 
karena jadi _boilerplate code_ yang harus ditambahin.

Sebagai gantinya, kita bisa nulis ini dengan cara yang lebih singkat pake `if let`. 
Kode berikut perilakunya sama persis kayak `match` di Listing 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

Sintaks `if let` nerima sebuah pattern sama sebuah ekspresi yang dipisahin sama 
tanda sama dengan. Dia cara kerjanya sama kayak sebuah `match`, di mana 
ekspresinya dikasih ke `match` dan pattern-nya itu adalah _arm_ pertamanya. 
Di kasus ini, pattern-nya adalah `Some(max)`, dan `max` di-bind ke nilai di dalem 
`Some`. Terus kita bisa pake `max` di dalem body blok `if let` dengan cara yang 
sama kayak kita pake `max` di arm `match` yang terkait. Kode di dalem blok `if let` 
cuma jalan kalau nilainya cocok sama pattern-nya.

Pake `if let` artinya lebih dikit ngetik, lebih dikit indentasi, dan lebih dikit 
_boilerplate code_. Tapi, kita kehilangan pengecekan _exhaustive_ (menyeluruh) 
yang diterapin sama `match` yang mastiin kalau kita nggak lupa nge-handle kasus 
apa pun. Milih antara `match` sama `if let` tergantung dari apa yang lagi kita 
lakuin di situasi kita saat itu dan apakah dapet keringkasan itu sebuah _trade-off_ 
yang pas buat ngorbanin pengecekan menyeluruh.

Dengan kata lain, kita bisa mikirin `if let` sebagai _syntax sugar_ buat sebuah 
`match` yang ngejalanin kode pas nilainya cocok sama satu pattern terus nyuekin 
semua nilai lainnya.

Kita bisa masukin sebuah `else` barengan sama `if let`. Blok kode yang ngikutin 
`else` itu sama kayak blok kode yang ngikutin kasus `_` di ekspresi `match` 
yang setara sama `if let` dan `else` itu. Inget definisi enum `Coin` di 
Listing 6-4, di mana varian `Quarter` juga nyimpen nilai `UsState`. Kalau kita 
mau ngitung semua koin yang bukan quarter sambil nyebutin negara bagian dari si 
quarter, kita bisa lakuin itu pake ekspresi `match`, kayak gini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Atau kita bisa pake ekspresi `if let` dan `else`, kayak gini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## Tetep di Jalur Aman (“Happy Path”) pake `let...else`

Pola yang umum adalah ngelakuin sebuah komputasi pas sebuah nilai ada isinya 
dan balikin nilai default kalau sebaliknya. Lanjut pake contoh kita soal koin 
dengan nilai `UsState`, kalau kita mau ngomong sesuatu yang lucu tergantung 
seberapa tua negara bagian di koin quarter itu, kita mungkin bakal nambahin 
sebuah method di `UsState` buat nge-cek umur sebuah negara bagian, kayak gini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Terus kita mungkin pake `if let` buat nge-match tipe koinnya, ngenalin variabel 
`state` di dalem body kondisinya, kayak di Listing 6-7.

<Listing number="6-7" caption="Nge-cek apakah sebuah negara bagian udah ada di tahun 1900 pake kondisional bersarang (nested) di dalem sebuah `if let`.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Emang beres sih kerjaannya, tapi ini ngegeser kerjaannya ke dalem body 
statement `if let`, dan kalau kerjaan yang harus dilakuin lebih ribet, bakal 
susah buat ngikutin persis gimana cabang-cabang _top-level_ (tingkat atas)-nya 
berhubungan. Kita juga bisa manfaatin fakta kalau ekspresi ngasilin nilai buat 
ngasilin `state` dari `if let` atau _return early_ (kembali lebih awal), kayak 
di Listing 6-8. (Kita juga bisa ngelakuin hal yang mirip pake `match`.)

<Listing number="6-8" caption="Pake `if let` buat ngasilin sebuah nilai atau return early.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Tapi ini agak nyebelin buat diikutin dengan caranya sendiri! Satu cabang dari 
`if let` ngasilin nilai, dan yang satunya _return_ dari fungsi sepenuhnya.

Buat bikin pola umum ini lebih enak buat diekspresikan, Rust punya `let...else`. 
Sintaks `let...else` nerima sebuah pattern di sisi kiri dan sebuah ekspresi di 
sisi kanan, mirip sekali sama `if let`, tapi dia nggak punya cabang `if`, cuma 
cabang `else`. Kalau pattern-nya cocok, dia bakal nge-bind nilai dari pattern 
ke _scope_ luar. Kalau pattern-nya _nggak_ cocok, programnya bakal ngalir ke 
dalem arm `else`, yang harus _return_ (kembali) dari fungsinya.

Di Listing 6-9, kita bisa liat gimana penampakan Listing 6-8 pas pake `let...else` 
sebagai ganti dari `if let`.

<Listing number="6-9" caption="Pake `let...else` buat ngejelasin alur (flow) lewat fungsinya.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Perhatiin ya kalau dia tetep “on the happy path” (di jalur aman yang diharapkan) 
di body utama fungsinya pake cara ini, tanpa harus punya alur kontrol yang 
bener-bener beda jauh buat dua cabang kayak yang dilakuin sama `if let`.

Kalau kita ada di situasi di mana program kita punya logika yang terlalu panjang 
(verbose) buat diekspresikan pake `match`, inget ya kalau `if let` sama 
`let...else` juga ada di dalem _toolbox_ Rust kita.

## Ringkasan

Kita sekarang udah ngebahas gimana cara pake enum buat bikin tipe kustom yang 
bisa jadi salah satu dari sekumpulan nilai yang di-_enumerate_. Kita udah 
nunjukin gimana tipe `Option<T>` bawaan standard library ngebantu kita pake 
sistem tipe buat nyegah error. Pas nilai enum punya data di dalemnya, kita bisa 
pake `match` atau `if let` buat ngekstrak dan pake nilai-nilai itu, tergantung 
dari seberapa banyak kasus yang perlu kita handle.

Program Rust kita sekarang bisa mengekspresikan konsep di domain kita pake 
struct dan enum. Bikin tipe kustom buat dipake di API kita mastiin keamanan 
tipe (type safety): _compiler_ bakal mastiin fungsi kita cuma dapet nilai dari 
tipe yang diharapkan sama tiap fungsinya.

Buat nyediain API yang terorganisir dengan baik ke _user_ kita yang gampang 
buat dipake dan cuma nge-ekspos tepat apa yang dibutuhin sama _user_ kita aja, 
yuk sekarang kita beralih ke _modules_ (modul) di Rust.
