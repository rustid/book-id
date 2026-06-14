## Komentar

Semua programmer pasti pengen kodenya gampang dipahamin, tapi kadang emang butuh 
penjelasan tambahan. Di kasus kayak gini, programmer naruh _komentar_ di 
source code mereka yang bakal dicuekin sama _compiler_ tapi bakal berguna buat 
orang yang baca kodenya.

Ini contoh komentar simpel:

```rust
// hello, world
```

Di Rust, gaya komentar yang idiomatik itu dimulai pake dua garis miring (`//`), 
dan komentarnya lanjut sampe akhir baris. Buat komentar yang panjangnya lebih 
dari satu baris, kita perlu masukin `//` di tiap barisnya, kayak gini:

```rust
// Jadi kita lagi ngerjain sesuatu yang ribet di sini, cukup panjang sampe 
// kita butuh beberapa baris komentar buat jelasinnya! Fiuh! Semoga komentar 
// ini bisa jelasin apa yang sebenernya lagi terjadi.
```

Komentar juga bisa ditaruh di akhir baris yang isinya kode:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Tapi kita bakal lebih sering liat komentar dipake dengan format kayak gini, 
di mana komentarnya ada di baris terpisah di atas kode yang lagi dianotasi:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust juga punya jenis komentar lain, yaitu _documentation comments_, yang bakal 
kita bahas di bagian [“Publishing a Crate to Crates.io”][publishing] di Bab 14.

[publishing]: ch14-02-publishing-to-crates-io.html
