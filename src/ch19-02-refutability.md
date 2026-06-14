## Refutability (Keterbantahan): Apakah sebuah Pattern Bisa Gagal Cocok?

_Patterns_ datang dalam dua bentuk: _refutable_ (bisa dibantah/bisa gagal) dan 
_irrefutable_ (tidak bisa dibantah/pasti sukses). _Patterns_ yang bakal selalu 
cocok dengan kemungkinan nilai apa pun yang diberikan ke dia itu disebut 
_irrefutable_. Contohnya adalah `x` di dalam statement `let x = 5;` karena `x` 
cocok dengan apa aja dan oleh karena itu tidak mungkin gagal buat cocok. 
_Patterns_ yang bisa aja gagal buat cocok dengan beberapa kemungkinan nilai itu 
disebut _refutable_. Contohnya adalah `Some(x)` di dalam ekspresi 
`if let Some(x) = a_value` karena kalau nilai di variabel `a_value` ternyata 
adalah `None` bukannya `Some`, maka _pattern_ `Some(x)` itu tidak bakal cocok.

Parameter fungsi, statement `let`, dan _loop_ `for` cuma bisa nerima _patterns_ 
yang _irrefutable_ karena programnya tidak bisa ngelakuin hal berguna apa pun 
kalau nilainya ternyata tidak cocok. Ekspresi `if let` dan `while let` serta 
statement `let...else` bisa nerima _patterns_ yang _refutable_ maupun 
_irrefutable_, tapi _compiler_ bakal ngasih peringatan (warning) kalau kita 
memakai _patterns_ yang _irrefutable_ di sana. Hal ini karena, secara definisi, 
konstruk-konstruk tersebut memang ditujukan buat menangani kemungkinan gagal: 
fungsionalitas dari sebuah kondisional ada pada kemampuannya buat melakukan 
hal yang berbeda-beda tergantung pada apakah dia sukses atau gagal.

Secara umum, kita tidak perlu pusing mikirin perbedaan antara _patterns_ yang 
_refutable_ dan _irrefutable_; namun, kita tetap harus familier dengan konsep 
_refutability_ ini supaya kita tahu apa yang harus dilakukan pas kita ngelihat 
istilah ini di dalam sebuah pesan error. Di kasus-kasus seperti itu, kita perlu 
mengubah entah _pattern_-nya atau konstruk tempat kita memakai _pattern_ 
tersebut, tergantung dari perilaku yang kita inginkan buat kodenya.

Mari kita lihat sebuah contoh tentang apa yang terjadi pas kita nyoba memakai 
sebuah _pattern_ yang _refutable_ di tempat di mana Rust mewajibkan _pattern_ yang 
_irrefutable_, dan sebaliknya. Listing 19-8 menunjukkan sebuah statement `let`, 
tapi buat _pattern_-nya, kita menentukan `Some(x)`, yang mana adalah sebuah 
_pattern_ yang _refutable_. Seperti yang mungkin kita tebak, kode ini tidak bakal 
bisa di-compile.

<Listing number="19-8" caption="Mencoba memakai sebuah pattern refutable bersama `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Kalau `some_option_value` kebetulan bernilai `None`, maka dia bakal gagal buat 
cocok sama _pattern_ `Some(x)`, yang artinya _pattern_ tersebut adalah 
_refutable_. Namun, statement `let` cuma bisa nerima _pattern_ yang _irrefutable_ 
karena tidak ada aksi valid yang bisa dilakuin sama kodenya dengan sebuah nilai 
`None`. Saat _compile time_, Rust bakal komplain kalau kita udah nyoba memakai 
sebuah _pattern_ _refutable_ di tempat di mana sebuah _pattern_ _irrefutable_ 
diwajibkan:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Karena kita tidak mencakup (dan memang tidak bisa mencakup!) setiap kemungkinan 
nilai yang valid dengan _pattern_ `Some(x)`, Rust dengan tepat menghasilkan error 
_compiler_.

Kalau kita punya sebuah _pattern_ _refutable_ di tempat di mana _pattern_ 
_irrefutable_ dibutuhkan, kita bisa memperbaikinya dengan mengubah kode yang 
memakai _pattern_ tersebut: ketimbang memakai `let`, kita bisa memakai 
`let else`. Dengan begitu, kalau _pattern_-nya tidak cocok, kodenya bakal sekadar 
melewati (skip) kode yang ada di dalam kurung kurawal, ngasih jalan (way out) 
buat dia lanjut dengan cara yang valid. Listing 19-9 menunjukkan cara 
memperbaiki kode di Listing 19-8.

<Listing number="19-9" caption="Memakai `let...else` dan sebuah blok dengan patterns refutable bukannya `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Kita udah ngasih jalan keluar buat kodenya! Kode ini benar-benar valid sekarang, 
meskipun ini berarti kita tidak bisa memakai sebuah _pattern_ _irrefutable_ tanpa 
menerima sebuah peringatan. Kalau kita ngasih `let...else` sebuah _pattern_ yang 
bakal selalu cocok, kayak `x`, seperti yang ditunjukkan di Listing 19-10, 
_compiler_ bakal ngasih sebuah peringatan.

<Listing number="19-10" caption="Mencoba memakai sebuah pattern irrefutable bersama `let...else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust komplain kalau tidak masuk akal buat memakai `let...else` dengan sebuah 
_pattern_ yang _irrefutable_:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Atas alasan ini, match arms wajib memakai _patterns_ yang _refutable_, kecuali buat 
arm yang paling terakhir, yang seharusnya mencocokkan sisa nilai apa pun yang belum 
ditangani dengan sebuah _pattern_ yang _irrefutable_. Rust mengizinkan kita buat 
memakai sebuah _pattern_ _irrefutable_ di dalam sebuah `match` yang cuma punya satu 
arm, tapi sintaks ini tidak terlalu berguna dan bisa aja diganti pakai sebuah 
statement `let` yang lebih sederhana.

Sekarang karena kita udah tahu di mana aja bisa memakai _patterns_ dan perbedaan 
antara _patterns_ _refutable_ dan _irrefutable_, mari kita bahas semua sintaks yang 
bisa kita pakai buat bikin _patterns_.
