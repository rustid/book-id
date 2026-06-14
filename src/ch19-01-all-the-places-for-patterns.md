## Semua Tempat di Mana Patterns Bisa Dipakai

_Patterns_ muncul di berbagai tempat di Rust, dan kita sebenarnya udah 
sering sekali memakai mereka tanpa menyadarinya! Bagian ini membahas semua 
tempat di mana _patterns_ itu valid untuk dipakai.

### Lengan (Arms) dari `match`

Seperti yang sudah dibahas di Bab 6, kita memakai _patterns_ di _arms_ 
(lengan) dari ekspresi `match`. Secara formal, ekspresi `match` didefinisikan 
sebagai keyword `match`, sebuah nilai yang mau dicocokkan, dan satu atau 
lebih _match arms_ yang terdiri dari sebuah _pattern_ dan sebuah ekspresi 
buat dijalankan kalau nilainya cocok dengan _pattern_ di arm tersebut, 
kayak gini:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>NILAI</em> {
    <em>PATTERN</em> => <em>EKSPRESI</em>,
    <em>PATTERN</em> => <em>EKSPRESI</em>,
    <em>PATTERN</em> => <em>EKSPRESI</em>,
}</code></pre>

Misalnya, ini adalah ekspresi `match` dari Listing 6-5 yang mencocokkan sebuah 
nilai `Option<i32>` di dalam variabel `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

_Patterns_ di ekspresi `match` ini adalah `None` dan `Some(i)` yang ada di 
sebelah kiri dari setiap tanda panah.

Satu persyaratan untuk ekspresi `match` adalah mereka harus bersifat 
_exhaustive_ (menyeluruh/tuntas) yang berarti semua kemungkinan nilai buat 
ekspresi `match` tersebut harus ditangani (accounted for). Salah satu cara 
buat memastikan kita udah mencakup semua kemungkinannya adalah dengan memakai 
_catch-all pattern_ (pola penangkap-semua) buat arm terakhirnya: misalnya, 
memakai nama variabel yang bakal cocok dengan nilai apa pun itu tidak bakal 
pernah gagal dan karena itu mencakup semua kasus yang tersisa.

_Pattern_ spesifik `_` bakal cocok dengan apa pun, tapi ia tidak pernah 
mengikat (bind) nilainya ke dalam sebuah variabel, jadi ia sering kali 
dipakai di match arm yang paling akhir. _Pattern_ `_` ini bisa berguna pas 
kita mau mengabaikan nilai apa pun yang tidak ditentukan (unspecified), 
sebagai contoh. Kita bakal membahas _pattern_ `_` lebih detail di [“Mengabaikan 
Nilai di dalam sebuah Pattern”][ignoring-values-in-a-pattern] nanti di bab ini.

### Statement `let`

Sebelum bab ini, kita cuma secara eksplisit ngebahas pemakaian _patterns_ bersama 
`match` dan `if let`, tapi pada kenyataannya, kita udah memakai _patterns_ di 
tempat lain juga, termasuk di dalam statement `let`. Misalnya, coba lihat 
pemberian nilai (variable assignment) langsung pakai `let` ini:

```rust
let x = 5;
```

Setiap kali kita memakai statement `let` kayak gini, kita sebenernya udah 
memakai _patterns_, biarpun kita mungkin tidak menyadarinya! Lebih 
formalnya, sebuah statement `let` itu kelihatannya kayak gini:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EKSPRESI</em>;</code>
</pre>

Di statement seperti `let x = 5;` di mana nama variabel ada di posisi PATTERN, 
nama variabel itu hanyalah bentuk yang paling sederhana dari sebuah _pattern_. 
Rust membandingkan ekspresi tersebut dengan _pattern_-nya lalu memberikan 
nilai (assigns) ke nama-nama yang ia temukan. Jadi, di contoh `let x = 5;`, `x` 
adalah sebuah _pattern_ yang artinya “ikat (bind) apa pun yang cocok di 
sini ke variabel `x`.” Karena nama `x` adalah _keseluruhan pattern_-nya, _pattern_ 
ini pada praktiknya berarti “ikat semuanya ke variabel `x`, apa pun nilainya.”

Buat ngelihat aspek _pattern-matching_ (pencocokan pola) dari `let` dengan 
lebih jelas, coba lihat Listing 19-1, yang memakai sebuah _pattern_ bareng 
`let` buat men-_destructure_ (memecah) sebuah _tuple_.

<Listing number="19-1" caption="Memakai pattern buat men-destructure sebuah tuple dan membikin tiga variabel sekaligus">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Di sini, kita mencocokkan sebuah _tuple_ terhadap sebuah _pattern_. Rust 
membandingkan nilai `(1, 2, 3)` dengan _pattern_ `(x, y, z)` dan melihat kalau 
nilainya cocok dengan _pattern_ tersebut, dalam arti dia melihat kalau jumlah 
elemennya sama di kedua sisinya, jadi Rust mengikat `1` ke `x`, `2` ke `y`, 
dan `3` ke `z`. Kita bisa membayangkan _tuple pattern_ ini seolah-olah menyarangkan 
(nesting) tiga _variable patterns_ individu di dalamnya.

Kalau jumlah elemen di dalam _pattern_-nya tidak cocok dengan jumlah elemen 
di dalam _tuple_-nya, tipe keseluruhannya tidak bakal cocok dan kita bakal 
dapat error _compiler_. Misalnya, Listing 19-2 menunjukkan sebuah percobaan buat 
men-_destructure_ sebuah _tuple_ yang berisi tiga elemen ke dalam dua variabel, 
yang mana tidak bakal jalan.

<Listing number="19-2" caption="Membikin sebuah pattern yang salah di mana jumlah variabelnya tidak cocok dengan jumlah elemen di dalam tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Mencoba men-compile kode ini bakal menghasilkan _type error_ (error tipe) ini:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Buat memperbaiki error-nya, kita bisa mengabaikan satu atau lebih nilai di 
dalam _tuple_ tersebut memakai `_` atau `..`, kayak yang bakal kita lihat 
di bagian [“Mengabaikan Nilai di dalam sebuah Pattern”][ignoring-values-in-a-pattern]. 
Kalau masalahnya adalah kita punya terlalu banyak variabel di dalam _pattern_-nya, 
solusinya adalah mencocokkan (match) tipe-tipenya dengan membuang variabel-variabel 
tersebut sehingga jumlah variabelnya sama dengan jumlah elemen di _tuple_-nya.

### Ekspresi Bersyarat (Conditional) `if let`

Di Bab 6, kita ngebahas gimana cara memakai ekspresi `if let` yang mana 
utamanya dipakai sebagai cara yang lebih singkat buat menulis bentuk ekuivalen 
(setara) dari sebuah `match` yang cuma mencocokkan satu kasus aja. Secara 
opsional, `if let` bisa dipasangkan dengan `else` yang berisi kode buat 
dijalankan kalau _pattern_ di dalam `if let` tersebut tidak cocok.

Listing 19-3 menunjukkan kalau kita juga bisa mencampur dan mencocokkan 
(mix and match) ekspresi `if let`, `else if`, dan `else if let`. Ngelakuin hal 
ini ngasih kita fleksibilitas yang lebih besar ketimbang ekspresi `match` 
di mana kita cuma bisa mengekspresikan satu nilai aja buat dibandingkan dengan 
semua _patterns_-nya. Selain itu, Rust tidak mewajibkan agar kondisi-kondisi 
di dalam rangkaian lengan (arms) `if let`, `else if`, dan `else if let` itu 
saling berhubungan satu sama lain.

Kode di Listing 19-3 menentukan warna apa yang bakal dijadikan warna _background_ 
(latar belakang) berdasarkan serangkaian pengecekan pada beberapa kondisi. 
Untuk contoh ini, kita sudah membikin variabel-variabel dengan nilai yang di-_hardcode_ 
(ditulis langsung) yang mana di program sungguhan mungkin aja nilai-nilai ini 
didapat dari input _user_.

<Listing number="19-3" file-name="src/main.rs" caption="Mencampur `if let`, `else if`, `else if let`, dan `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

Kalau _user_ menentukan sebuah warna favorit, warna itu bakal dipakai sebagai 
_background_. Kalau tidak ada warna favorit yang ditentukan dan hari ini adalah 
hari Selasa (Tuesday), warna _background_-nya adalah hijau (green). Selain itu, 
kalau _user_ menentukan umurnya sebagai sebuah string dan kita bisa 
mem-_parse_-nya jadi sebuah angka dengan sukses, warnanya bakal jadi ungu (purple) 
atau oranye (orange) tergantung dari nilai angkanya. Kalau tidak ada satu pun 
dari kondisi ini yang berlaku, warna _background_-nya adalah biru (blue).

Struktur kondisional (bersyarat) ini membiarkan kita mendukung persyaratan yang 
kompleks. Dengan nilai-nilai _hardcoded_ yang kita punya di sini, contoh ini bakal 
mencetak `Using purple as the background color`.

Kita bisa melihat kalau `if let` juga bisa memperkenalkan variabel baru yang 
menimpa (shadow) variabel yang sudah ada dengan cara yang sama kayak yang 
dilakukan sama `match` arms: baris `if let Ok(age) = age` memperkenalkan 
sebuah variabel `age` baru yang berisi nilai di dalam varian `Ok`-nya, menimpa 
variabel `age` yang sudah ada sebelumnya. Ini artinya kita harus menaruh kondisi 
`if age > 30` di dalam blok tersebut: kita tidak bisa menggabungkan kedua 
kondisi ini jadi `if let Ok(age) = age && age > 30`. Nilai `age` baru yang mau 
kita bandingkan dengan 30 belum valid sampai _scope_ (ruang lingkup) barunya 
dimulai bersamaan dengan tanda kurung kurawal pembuka.

Kelemahan dari memakai ekspresi `if let` adalah kalau _compiler_ tidak bakal 
mengecek kelengkapannya (exhaustiveness), sedangkan dengan ekspresi `match` 
_compiler_ bakal mengeceknya. Kalau kita kelupaan menaruh blok `else` terakhir dan 
oleh karenanya kelewatan (missed) buat menangani beberapa kasus, _compiler_ 
tidak bakal memperingatkan kita soal kemungkinan adanya _logic bug_ (kutu logika) 
tersebut.

### Perulangan Bersyarat `while let`

Mirip dengan konstruksi `if let`, perulangan (loop) bersyarat `while let` 
memungkinkan sebuah _loop_ `while` buat terus berjalan selama sebuah _pattern_ 
masih terus cocok. Di Listing 19-4 kita menunjukkan sebuah _loop_ `while let` 
yang menunggu pesan-pesan yang dikirim antar _threads_, tapi di kasus ini 
dia mengecek sebuah `Result` ketimbang sebuah `Option`.

<Listing number="19-4" caption="Memakai _loop_ `while let` buat mencetak nilai-nilai selama `rx.recv()` mengembalikan `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Contoh ini mencetak `1`, `2`, dan kemudian `3`. Method `recv` mengambil 
pesan pertama dari sisi penerima (receiver side) saluran (channel) tersebut dan 
mengembalikan sebuah `Ok(value)`. Saat pertama kali kita melihat `recv` 
di Bab 16, kita langsung meng-_unwrap_ error-nya, atau berinteraksi dengannya 
layaknya sebuah iterator memakai sebuah _loop_ `for`. Namun, seperti yang 
ditunjukkan Listing 19-4, kita juga bisa memakai `while let`, karena method 
`recv` mengembalikan sebuah `Ok` setiap kali ada pesan yang datang, selama si 
pengirimnya (sender) masih eksis, lalu mengembalikan sebuah `Err` begitu 
sisi pengirimnya memutuskan koneksi (disconnects).

### Perulangan `for`

Di dalam _loop_ `for`, nilai yang langsung mengikuti keyword `for` itu adalah 
sebuah _pattern_. Misalnya, di dalam `for x in y`, `x` itu adalah _pattern_-nya. 
Listing 19-5 mendemonstrasikan gimana cara memakai sebuah _pattern_ di dalam _loop_ 
`for` buat men-_destructure_ (memecah belah) sebuah _tuple_ sebagai bagian dari 
_loop_ `for` tersebut.

<Listing number="19-5" caption="Memakai sebuah _pattern_ di dalam _loop_ `for` buat men-_destructure_ sebuah _tuple_">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Kode di Listing 19-5 bakal mencetak yang berikut ini:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Kita mengadaptasi sebuah iterator memakai method `enumerate` sehingga ia 
menghasilkan sebuah nilai beserta indeks buat nilai tersebut, yang ditaruh 
di dalam sebuah _tuple_. Nilai pertama yang dihasilkan adalah _tuple_ 
`(0, 'a')`. Saat nilai ini dicocokkan dengan _pattern_ `(index, value)`, 
`index` bakal jadi `0` dan `value` bakal jadi `'a'`, lalu mencetak 
baris pertama dari outputnya.

### Parameter Fungsi

Parameter dari sebuah fungsi juga bisa berupa _patterns_. Kode di Listing 19-6, 
yang mendeklarasikan fungsi bernama `foo` yang menerima satu parameter bernama 
`x` bertipe `i32`, harusnya sekarang udah terasa familier.

<Listing number="19-6" caption="Sebuah _signature_ fungsi yang memakai _patterns_ di dalam parameternya">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

Bagian `x` itu adalah sebuah _pattern_ lho! Sama kayak yang kita lakuin dengan 
`let`, kita bisa mencocokkan sebuah _tuple_ di dalam argumen sebuah fungsi 
terhadap suatu _pattern_. Listing 19-7 memecah nilai-nilai di dalam sebuah 
_tuple_ saat kita meneruskannya ke sebuah fungsi.

<Listing number="19-7" file-name="src/main.rs" caption="Sebuah fungsi dengan parameter yang men-_destructure_ sebuah _tuple_">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Kode ini mencetak `Current location: (3, 5)`. Nilai-nilai `&(3, 5)` itu cocok 
dengan _pattern_ `&(x, y)`, jadi `x` adalah nilai `3` dan `y` adalah nilai `5`.

Kita juga bisa memakai _patterns_ di dalam daftar parameter _closure_ dengan cara 
yang sama seperti di dalam daftar parameter fungsi, karena _closures_ itu mirip 
dengan fungsi, kayak yang udah dibahas di Bab 13.

Pada titik ini, kita udah melihat beberapa cara buat memakai _patterns_, tapi 
_patterns_ tidak bekerja dengan cara yang sama persis di setiap tempat di mana 
kita bisa memakai mereka. Di beberapa tempat, _patterns_ itu wajib bersifat 
_irrefutable_ (tidak bisa dibantah/pasti sukses); di situasi lain, mereka 
bisa bersifat _refutable_ (bisa dibantah/bisa gagal). Kita bakal ngebahas 
kedua konsep ini selanjutnya.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
