## Memvalidasi Referensi dengan Lifetimes

_Lifetimes_ (waktu hidup) adalah jenis generik lain yang sebenarnya sudah 
kita pakai. Bukannya memastikan kalau sebuah tipe punya perilaku yang kita mau, 
_lifetimes_ memastikan kalau referensi bakal tetap valid selama kita masih 
butuh.

Satu detail yang tidak kita bahas di bagian [“Referensi dan Borrowing”][references-and-borrowing] 
di Bab 4 adalah setiap referensi di Rust punya sebuah _lifetime_, yaitu _scope_ 
di mana referensi itu valid. Sebagian besar waktu, _lifetimes_ itu bersifat implisit 
dan ditebak (_inferred_), sama halnya kayak sebagian besar waktu tipe juga 
ditebak. Kita baru diwajibkan buat menganotasi tipe kalau ada beberapa kemungkinan 
tipe yang bisa dipakai. Mirip dengan itu, kita harus menganotasi _lifetimes_ kalau 
_lifetimes_ dari referensi-referensi yang ada bisa berhubungan dengan beberapa 
cara yang berbeda. Rust mewajibkan kita menganotasi hubungan ini memakai 
parameter _lifetime_ generik untuk memastikan referensi sebenarnya yang dipakai 
pas _runtime_ bakal pasti valid.

Menganotasi _lifetimes_ bahkan bukan konsep yang dimiliki kebanyakan bahasa 
pemrograman lain, jadi ini mungkin bakal terasa asing. Meskipun kita tidak 
bakal membahas _lifetimes_ secara menyeluruh di bab ini, kita bakal membahas 
cara-cara umum yang mungkin bakal kita temui terkait sintaks _lifetime_ supaya 
kita bisa nyaman dengan konsepnya.

### Mencegah Dangling References dengan Lifetimes

Tujuan utama dari _lifetimes_ adalah buat mencegah _dangling references_ 
(referensi menggantung), yang bikin program merujuk ke data yang salah alih-
alih data yang sebenarnya dituju. Coba perhatikan program di Listing 10-16, yang 
punya _scope_ luar dan _scope_ dalam.

<Listing number="10-16" caption="Usaha untuk memakai referensi yang nilainya sudah keluar dari _scope_">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Catatan: Contoh di Listing 10-16, 10-17, dan 10-23 mendeklarasikan variabel 
> tanpa memberi mereka nilai awal, jadi nama variabelnya eksis di _scope_ luar. 
> Sekilas, ini mungkin kelihatannya bertentangan sama aturan Rust yang tidak 
> mengizinkan nilai null. Tapi, kalau kita mencoba memakai sebuah variabel 
> sebelum memberinya nilai, kita bakal dapat error _compile-time_, yang 
> menunjukkan kalau Rust memang tidak mengizinkan nilai null.

_Scope_ luar mendeklarasikan variabel bernama `r` tanpa nilai awal, dan 
_scope_ dalam mendeklarasikan variabel bernama `x` dengan nilai awal `5`. Di 
dalam _scope_ dalam, kita mencoba nge-set nilai `r` jadi referensi ke `x`. 
Kemudian _scope_ dalamnya berakhir, dan kita mencoba mencetak nilai di `r`. 
Kode ini tidak bakal bisa di-compile karena nilai yang dirujuk sama `r` sudah 
keluar dari _scope_ sebelum kita mencoba memakainya. Ini pesan error-nya:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Pesan error-nya bilang kalau variabel `x` “tidak hidup cukup lama” (does not live long enough). 
Alasannya adalah `x` bakal keluar dari _scope_ saat _scope_ dalam berakhir di 
baris 7. Tapi `r` masih valid untuk _scope_ luar; karena _scope_-nya lebih 
besar, kita bilang kalau dia “hidup lebih lama.” Kalau Rust membiarkan kode ini 
jalan, `r` bakal merujuk ke memori yang sudah di-dealokasi saat `x` keluar dari 
_scope_, dan apa pun yang kita coba lakukan dengan `r` tidak bakal jalan dengan 
benar. Terus gimana caranya Rust bisa nentuin kalau kode ini tidak valid? Rust 
memakai sebuah _borrow checker_.

### Borrow Checker

_Compiler_ Rust punya sebuah _borrow checker_ yang membandingkan _scopes_ buat 
menentukan apakah semua referensi yang dipinjam (_borrows_) itu valid. Listing 
10-17 menunjukkan kode yang sama seperti Listing 10-16 tapi dengan anotasi 
yang menunjukkan _lifetimes_ dari variabel-variabelnya.

<Listing number="10-17" caption="Anotasi _lifetimes_ dari `r` dan `x`, dinamakan masing-masing `'a` dan `'b`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Di sini, kita sudah menganotasi _lifetime_ dari `r` dengan `'a` dan _lifetime_ 
dari `x` dengan `'b`. Seperti yang bisa dilihat, blok `'b` di dalam itu jauh 
lebih kecil daripada blok _lifetime_ `'a` di luar. Pada saat _compile time_, Rust 
membandingkan ukuran dari dua _lifetimes_ ini dan melihat kalau `r` punya 
_lifetime_ `'a` tapi dia merujuk ke memori dengan _lifetime_ `'b`. Programnya 
ditolak karena `'b` lebih pendek dari `'a`: subjek yang dirujuk tidak hidup 
selama referensinya.

Listing 10-18 memperbaiki kodenya biar dia tidak punya _dangling reference_ dan 
bisa di-compile tanpa error sama sekali.

<Listing number="10-18" caption="Referensi yang valid karena datanya punya _lifetime_ yang lebih panjang dari referensinya">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Di sini, `x` punya _lifetime_ `'b`, yang mana di kasus ini lebih besar dari `'a`. 
Ini berarti `r` bisa merujuk ke `x` karena Rust tahu kalau referensi di `r` bakal 
selalu valid selama `x` masih valid.

Sekarang setelah kita tahu di mana _lifetimes_ dari referensi berada dan gimana 
Rust menganalisis _lifetimes_ buat memastikan referensi bakal selalu valid, 
mari kita eksplor _lifetimes_ generik buat parameter dan nilai kembalian di 
dalam konteks fungsi.

### Generic Lifetimes di Fungsi

Kita bakal nulis fungsi yang mengembalikan _string slice_ yang lebih panjang 
di antara dua _string slice_. Fungsi ini bakal menerima dua _string slice_ dan 
mengembalikan satu _string slice_. Setelah kita mengimplementasikan fungsi `longest`, 
kode di Listing 10-19 seharusnya mencetak `The longest string is abcd`.

<Listing number="10-19" file-name="src/main.rs" caption="Fungsi `main` yang memanggil fungsi `longest` buat mencari mana yang lebih panjang dari dua _string slices_">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Perhatikan bahwa kita mau fungsi ini menerima _string slices_, yang merupakan 
referensi, bukannya _strings_, karena kita tidak mau fungsi `longest` mengambil 
_ownership_ dari parameter-parameternya. Coba cek lagi bagian [“String Slices 
sebagai Parameter”][string-slices-as-parameters] di Bab 4 buat pembahasan lebih 
lanjut soal kenapa parameter yang kita pakai di Listing 10-19 adalah yang memang 
kita perlukan.

Kalau kita mencoba mengimplementasikan fungsi `longest` seperti yang ditunjukkan 
di Listing 10-20, kode ini tidak bakal bisa di-compile.

<Listing number="10-20" file-name="src/main.rs" caption="Implementasi fungsi `longest` yang mengembalikan yang lebih panjang dari dua _string slices_ tapi belum bisa di-compile">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Alih-alih jalan, kita dapat error berikut yang membahas soal _lifetimes_:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Teks bantuannya ngasih tau kalau tipe kembaliannya butuh parameter _lifetime_ 
generik di situ karena Rust tidak bisa menebak apakah referensi yang bakal 
dikembalikan itu merujuk ke `x` atau ke `y`. Nyatanya, kita juga tidak tahu, 
karena blok `if` di dalam body fungsi ini mengembalikan referensi ke `x` dan 
blok `else` mengembalikan referensi ke `y`!

Saat mendefinisikan fungsi ini, kita tidak tahu nilai konkret apa yang bakal 
dimasukkan ke fungsi ini, jadi kita tidak tahu apakah blok `if` atau blok `else` 
yang bakal dijalankan. Kita juga tidak tahu _lifetimes_ konkret dari referensi 
yang bakal dimasukkan, jadi kita tidak bisa melihat _scopes_ seperti yang kita 
lakukan di Listing 10-17 dan 10-18 untuk menentukan apakah referensi yang kita 
kembalikan itu bakal selalu valid. _Borrow checker_ juga tidak bisa menentukannya, 
karena dia tidak tahu gimana _lifetimes_ dari `x` dan `y` berhubungan sama 
_lifetime_ dari nilai kembaliannya. Buat membenarkan error ini, kita bakal 
menambahkan parameter _lifetime_ generik yang mendefinisikan hubungan antara 
referensi-referensi tersebut agar _borrow checker_ bisa melakukan analisisnya.

### Sintaks Anotasi Lifetime

Anotasi _lifetime_ tidak mengubah seberapa lama suatu referensi hidup. Mereka 
justru menggambarkan hubungan dari _lifetimes_ antara banyak referensi satu sama 
lain tanpa memengaruhi _lifetimes_ itu sendiri. Sama seperti fungsi yang bisa 
menerima tipe apa pun saat _signature_-nya menentukan parameter tipe generik, 
fungsi juga bisa menerima referensi dengan _lifetime_ apa pun dengan menentukan 
parameter _lifetime_ generik.

Anotasi _lifetime_ punya sintaks yang agak tidak biasa: nama parameter _lifetime_ 
harus dimulai dengan apostrof (tanda kutip tunggal, `'`) dan biasanya semuanya 
huruf kecil dan sangat pendek, sama seperti tipe generik. Kebanyakan orang 
memakai nama `'a` untuk anotasi _lifetime_ yang pertama. Kita menaruh anotasi 
parameter _lifetime_ setelah tanda `&` dari sebuah referensi, dengan memakai 
spasi untuk memisahkan anotasinya dari tipe referensinya.

Ini beberapa contohnya: sebuah referensi ke `i32` tanpa parameter _lifetime_, 
sebuah referensi ke `i32` yang punya parameter _lifetime_ bernama `'a`, dan 
sebuah referensi _mutable_ ke `i32` yang juga punya _lifetime_ `'a`.

```rust,ignore
&i32        // sebuah referensi
&'a i32     // sebuah referensi dengan _lifetime_ eksplisit
&'a mut i32 // sebuah referensi _mutable_ dengan _lifetime_ eksplisit
```

Satu anotasi _lifetime_ yang berdiri sendiri tidak punya banyak arti karena 
anotasi itu ditujukan untuk memberi tahu Rust gimana parameter _lifetime_ generik 
dari berbagai referensi saling berhubungan. Mari kita teliti gimana anotasi 
_lifetime_ berhubungan satu sama lain di dalam konteks fungsi `longest`.

### Anotasi Lifetime di Signature Fungsi

Buat memakai anotasi _lifetime_ di _signature_ fungsi, kita harus 
mendeklarasikan parameter _lifetime_ generik di dalam kurung sudut di antara 
nama fungsi dan daftar parameter, sama persis seperti yang kita lakukan sama 
parameter _tipe_ generik.

Kita mau _signature_ ini mengekspresikan batasan ini: referensi yang 
dikembalikan bakal valid setidaknya selama kedua parameter itu juga valid. Ini 
adalah hubungan antara _lifetimes_ dari parameter dan nilai kembaliannya. 
Kita bakal menamakan _lifetime_ itu `'a` lalu menambahkannya ke setiap 
referensi, seperti yang ditunjukkan di Listing 10-21.

<Listing number="10-21" file-name="src/main.rs" caption="Definisi fungsi `longest` yang menentukan kalau semua referensi di _signature_ tersebut harus punya _lifetime_ `'a` yang sama">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Kode ini seharusnya bisa di-compile dan menghasilkan hasil yang kita inginkan 
saat kita memakainya bersama fungsi `main` di Listing 10-19.

_Signature_ fungsinya sekarang memberi tahu Rust bahwa untuk suatu _lifetime_ 
`'a`, fungsi ini menerima dua parameter, di mana keduanya adalah _string slices_ 
yang hidup setidaknya sepanjang _lifetime_ `'a`. _Signature_ fungsinya juga 
memberi tahu Rust bahwa _string slice_ yang dikembalikan dari fungsi itu bakal 
hidup setidaknya sepanjang _lifetime_ `'a`. Di praktiknya, ini berarti _lifetime_ 
dari referensi yang dikembalikan oleh fungsi `longest` itu sama dengan 
_lifetime_ yang paling kecil dari antara nilai-nilai yang dirujuk oleh 
argumen-argumen fungsi tersebut. Hubungan-hubungan inilah yang kita mau Rust 
pakai saat menganalisis kode ini.

Ingat, saat kita menentukan parameter _lifetime_ di _signature_ fungsi ini, kita 
tidak sedang mengubah _lifetimes_ dari nilai apa pun yang masuk atau keluar. 
Tapi, kita sedang menentukan kalau _borrow checker_ harus menolak nilai apa pun 
yang tidak mematuhi batasan-batasan ini. Perhatikan bahwa fungsi `longest` 
tidak perlu tahu persis berapa lama `x` dan `y` bakal hidup, dia cuma perlu 
tahu kalau ada suatu _scope_ yang bisa disubstitusi untuk `'a` yang bakal memenuhi 
_signature_ ini.

Saat menganotasi _lifetimes_ di fungsi, anotasinya ditaruh di _signature_ fungsi, 
bukan di body fungsi. Anotasi _lifetime_ menjadi bagian dari kontrak fungsi itu, 
mirip dengan tipe-tipe di _signature_-nya. Memiliki _signature_ fungsi yang 
mengandung kontrak _lifetime_ berarti analisis yang dilakukan _compiler_ Rust bisa 
jadi lebih sederhana. Kalau ada masalah dengan cara sebuah fungsi dianotasi atau 
cara dia dipanggil, pesan error _compiler_ bisa menunjuk ke bagian kode kita 
serta batasannya dengan lebih tepat. Kalau sebaliknya _compiler_ Rust menebak-nebak 
lebih banyak soal apa yang kita maksud terkait hubungan antar _lifetimes_, 
_compiler_ mungkin cuma bakal bisa nunjukin pemakaian kode kita yang berjarak 
beberapa langkah dari sumber masalah aslinya.

Saat kita memasukkan referensi konkret ke `longest`, _lifetime_ konkret yang 
disubstitusikan untuk `'a` adalah bagian dari _scope_ `x` yang tumpang tindih 
(_overlap_) dengan _scope_ `y`. Dengan kata lain, _lifetime_ generik `'a` 
bakal mendapatkan _lifetime_ konkret yang setara dengan yang lebih kecil 
di antara _lifetimes_ `x` dan `y`. Karena kita sudah menganotasi referensi yang 
dikembalikan dengan parameter _lifetime_ `'a` yang sama, referensi kembalian 
itu juga bakal valid sepanjang yang lebih kecil di antara _lifetimes_ `x` dan `y`.

Mari kita lihat gimana anotasi _lifetime_ membatasi fungsi `longest` dengan 
memasukkan referensi yang punya _lifetimes_ konkret yang berbeda. Listing 10-22 
adalah contoh yang simpel.

<Listing number="10-22" file-name="src/main.rs" caption="Memakai fungsi `longest` dengan referensi ke nilai `String` yang punya _lifetimes_ konkret yang berbeda">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Di contoh ini, `string1` itu valid sampai akhir dari _scope_ luar, `string2` itu 
valid sampai akhir _scope_ dalam, dan `result` merujuk ke sesuatu yang valid 
sampai akhir _scope_ dalam. Jalankan kode ini dan kita bakal lihat kalau _borrow 
checker_ menyetujuinya; kodenya bakal di-compile dan mencetak `The longest string
is long string is long`.

Berikutnya, mari kita coba contoh yang menunjukkan kalau _lifetime_ dari 
referensi di `result` harus merupakan _lifetime_ yang lebih kecil dari dua 
argumennya. Kita bakal memindahkan deklarasi variabel `result` ke luar _scope_ 
dalam tapi membiarkan proses assignment nilai ke variabel `result` di dalam 
_scope_ bareng `string2`. Lalu kita bakal pindahin `println!` yang memakai 
`result` ke luar _scope_ dalam, setelah _scope_ dalam tersebut berakhir. 
Kode di Listing 10-23 tidak bakal bisa di-compile.

<Listing number="10-23" file-name="src/main.rs" caption="Mencoba memakai `result` setelah `string2` keluar dari _scope_">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Pas kita nyoba compile kode ini, kita bakal dapet error ini:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Error ini menunjukkan kalau biar `result` valid buat _statement_ `println!`, 
`string2` harusnya valid sampai akhir dari _scope_ luar. Rust tahu ini karena 
kita sudah menganotasi _lifetimes_ dari parameter fungsi dan nilai kembalian 
dengan memakai parameter _lifetime_ `'a` yang sama.

Sebagai manusia, kita bisa melihat kode ini dan langsung tahu kalau `string1` 
lebih panjang dari `string2`, dan karenanya, `result` bakal berisi referensi ke 
`string1`. Karena `string1` belum keluar dari _scope_, referensi ke `string1` 
seharusnya masih valid buat _statement_ `println!`. Tapi, _compiler_ tidak 
bisa melihat kalau referensinya valid di kasus ini. Kita sudah memberi tahu Rust 
kalau _lifetime_ referensi yang dikembalikan oleh fungsi `longest` itu sama dengan 
yang lebih kecil di antara _lifetimes_ dari referensi-referensi yang dimasukkan. 
Maka dari itu, _borrow checker_ menolak kode di Listing 10-23 karena 
kemungkinannya punya referensi yang tidak valid.

Coba desain eksperimen lain yang memvariasikan nilai dan _lifetimes_ dari 
referensi yang di-_pass_ ke fungsi `longest` serta gimana referensi kembaliannya 
dipakai. Bikin hipotesis tentang apakah eksperimen kita bakal lolos _borrow 
checker_ sebelum men-compile; lalu cek apakah kita benar!

### Berpikir dalam Konteks Lifetimes

Gimana cara kita menentukan parameter _lifetime_ itu bergantung pada apa yang 
sedang dilakukan sama fungsi kita. Misalnya, kalau kita mengubah implementasi 
fungsi `longest` biar selalu mengembalikan parameter pertama bukannya 
_string slice_ yang paling panjang, kita tidak perlu menentukan _lifetime_ 
pada parameter `y`. Kode berikut ini bakal bisa di-compile:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

Kita sudah menentukan parameter _lifetime_ `'a` buat parameter `x` dan tipe 
kembaliannya, tapi tidak buat parameter `y`, karena _lifetime_ `y` tidak punya 
hubungan apa pun dengan _lifetime_ `x` atau nilai kembaliannya.

Saat mengembalikan sebuah referensi dari sebuah fungsi, parameter _lifetime_ 
buat tipe kembaliannya harus cocok dengan parameter _lifetime_ buat salah satu 
dari parameternya. Kalau referensi yang dikembalikan _tidak_ merujuk ke salah 
satu parameter, maka referensi itu pasti merujuk ke suatu nilai yang dibuat di 
dalam fungsi itu sendiri. Namun, ini bakal jadi _dangling reference_ karena 
nilai itu bakal keluar dari _scope_ di akhir dari fungsinya. Coba perhatikan 
usaha implementasi fungsi `longest` yang tidak bakal bisa di-compile ini:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Di sini, meskipun kita sudah menentukan parameter _lifetime_ `'a` buat tipe 
kembaliannya, implementasi ini bakal gagal di-compile karena _lifetime_ nilai 
kembaliannya sama sekali tidak berhubungan dengan _lifetime_ parameter-parameternya. 
Ini pesan error yang bakal kita dapat:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Masalahnya adalah `result` keluar dari _scope_ dan dibersihkan di akhir dari 
fungsi `longest`. Kita juga mencoba mengembalikan referensi ke `result` dari 
fungsinya. Tidak ada cara buat kita menentukan parameter _lifetime_ yang bisa 
mengubah _dangling reference_ tersebut, dan Rust tidak bakal ngebiarin kita bikin 
_dangling reference_. Di kasus ini, perbaikan terbaiknya adalah dengan 
mengembalikan tipe data yang _owned_ (dimiliki) bukannya sebuah referensi 
sehingga fungsi yang memanggilnya nanti bertanggung jawab buat membersihkan 
nilai tersebut.

Pada akhirnya, sintaks _lifetime_ adalah soal menghubungkan _lifetimes_ dari 
berbagai parameter dan nilai kembalian dari suatu fungsi. Begitu mereka 
terhubung, Rust punya informasi yang cukup buat mengizinkan operasi yang aman 
buat memori (memory-safe operations) dan menolak operasi yang bakal membuat 
_dangling pointers_ atau melanggar keamanan memori.

### Anotasi Lifetime di Definisi Struct

Sejauh ini, _struct_ yang kita definisikan semuanya menampung tipe-tipe yang 
_owned_. Kita bisa mendefinisikan _struct_ buat menampung referensi, tapi di 
kasus itu kita perlu menambahkan anotasi _lifetime_ pada setiap referensi di 
dalam definisi _struct_ tersebut. Listing 10-24 punya _struct_ bernama 
`ImportantExcerpt` yang menampung sebuah _string slice_.

<Listing number="10-24" file-name="src/main.rs" caption="Sebuah _struct_ yang menampung referensi, yang mana butuh anotasi _lifetime_">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

_Struct_ ini punya field tunggal `part` yang menampung _string slice_, yang 
merupakan sebuah referensi. Sama kayak tipe data generik, kita mendeklarasikan 
nama parameter _lifetime_ generik di dalam kurung sudut setelah nama _struct_ 
sehingga kita bisa memakai parameter _lifetime_ itu di dalam definisi _struct_-nya. 
Anotasi ini berarti sebuah instance dari `ImportantExcerpt` tidak bisa hidup 
lebih lama dari referensi yang ditampungnya di field `part`.

Fungsi `main` di sini bikin instance dari _struct_ `ImportantExcerpt` yang 
menampung referensi ke kalimat pertama dari `String` yang dimiliki sama variabel 
`novel`. Data di `novel` sudah ada sebelum instance `ImportantExcerpt` itu 
dibikin. Selain itu, `novel` belum keluar dari _scope_ sampai setelah 
`ImportantExcerpt` keluar dari _scope_, jadi referensi di dalam instance 
`ImportantExcerpt` itu dipastikan valid.

### Lifetime Elision (Penghilangan Lifetime)

Kita udah belajar kalau setiap referensi punya _lifetime_ dan kita harus 
menentukan parameter _lifetime_ untuk fungsi atau _struct_ yang memakai referensi. 
Namun, kita tadi punya fungsi di Listing 4-9, yang ditampilkan lagi di Listing 
10-25, yang berhasil di-compile tanpa anotasi _lifetime_.

<Listing number="10-25" file-name="src/lib.rs" caption="Sebuah fungsi yang kita definisikan di Listing 4-9 yang berhasil di-compile tanpa anotasi _lifetime_, biarpun parameter dan tipe kembaliannya berupa referensi">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

Alasan kenapa fungsi ini bisa di-compile tanpa anotasi _lifetime_ murni karena 
sejarah: di versi awal (sebelum 1.0) dari Rust, kode ini tidak bakal bisa 
di-compile karena setiap referensi butuh _lifetime_ yang eksplisit. Waktu itu, 
_signature_ fungsi ini bakal ditulis kayak gini:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Setelah menulis banyak kode Rust, tim Rust menemukan kalau programmer Rust 
memasukkan anotasi _lifetime_ yang sama berulang kali di situasi-situasi 
tertentu. Situasi-situasi ini bisa diprediksi dan mengikuti beberapa pola yang 
deterministik. Para pengembang memprogram pola-pola ini ke dalam kode _compiler_ 
supaya _borrow checker_ bisa menebak (infer) _lifetimes_ di situasi-situasi ini 
dan tidak membutuhkan anotasi yang eksplisit lagi.

Sejarah Rust ini cukup relevan karena mungkin saja ke depannya bakal ada pola 
deterministik lain yang muncul dan ditambahkan ke dalam _compiler_. Di masa depan, 
mungkin bakal lebih sedikit lagi anotasi _lifetime_ yang diwajibkan.

Pola-pola yang diprogram ke dalam analisis referensi Rust disebut 
_lifetime elision rules_ (aturan penghilangan lifetime). Ini bukan aturan 
buat dipatuhi sama programmer; mereka ini adalah serangkaian kasus tertentu yang 
bakal dipertimbangkan oleh _compiler_, dan kalau kode kita masuk ke kasus-kasus 
ini, kita tidak perlu nulis _lifetimes_-nya secara eksplisit.

Aturan elision ini tidak memberikan tebakan (inference) yang komplit. Kalau masih 
ada kebingungan atau ketidakpastian (ambiguity) soal apa _lifetimes_ dari 
referensi tersebut setelah Rust menerapkan aturan-aturannya, _compiler_ tidak 
bakal menebak-nebak apa seharusnya _lifetime_ untuk referensi yang tersisa. 
Alih-alih menebak, _compiler_ bakal ngasih kita error yang bisa diselesaikan 
dengan menambahkan anotasi _lifetime_ secara manual.

_Lifetimes_ pada parameter fungsi atau _method_ disebut _input lifetimes_, dan 
_lifetimes_ pada nilai kembalian disebut _output lifetimes_.

_Compiler_ memakai tiga aturan buat mencari tahu _lifetimes_ dari referensi saat 
tidak ada anotasi yang eksplisit. Aturan pertama berlaku buat _input lifetimes_, 
sedangkan aturan kedua dan ketiga berlaku buat _output lifetimes_. Kalau 
_compiler_ sudah sampai ke akhir dari tiga aturan ini dan masih ada referensi 
yang tidak diketahui _lifetimes_-nya, _compiler_ bakal berhenti dengan sebuah 
error. Aturan-aturan ini berlaku buat definisi `fn` maupun blok `impl`.

Aturan pertama adalah _compiler_ meng-assign parameter _lifetime_ ke setiap 
parameter yang berupa referensi. Dengan kata lain, fungsi dengan satu parameter 
dapat satu parameter _lifetime_: `fn foo<'a>(x: &'a i32)`; fungsi dengan dua 
parameter dapat dua parameter _lifetime_ terpisah: `fn foo<'a, 'b>(x: &'a i32,
y: &'b i32)`; dan seterusnya.

Aturan kedua adalah, kalau ada tepat satu parameter _input lifetime_, 
_lifetime_ itu di-assign ke semua parameter _output lifetime_: `fn foo<'a>(x: &'a i32)
-> &'a i32`.

Aturan ketiga adalah, kalau ada beberapa parameter _input lifetime_, tapi salah 
satunya adalah `&self` atau `&mut self` karena ini adalah sebuah _method_, maka 
_lifetime_ dari `self` itu bakal di-assign ke semua parameter _output lifetime_. 
Aturan ketiga ini bikin _methods_ jauh lebih enak buat dibaca dan ditulis karena 
kita butuh lebih sedikit simbol.

Mari pura-pura kita adalah _compiler_. Kita bakal menerapkan aturan-aturan ini 
buat mencari tahu _lifetimes_ dari referensi di dalam _signature_ fungsi 
`first_word` di Listing 10-25. _Signature_-nya mulai tanpa ada _lifetimes_ apa 
pun yang berkaitan dengan referensi-referensinya:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Kemudian _compiler_ menerapkan aturan pertama, yang menentukan bahwa tiap 
parameter dapat _lifetime_-nya masing-masing. Kita bakal menyebutnya `'a` 
seperti biasa, jadi sekarang _signature_-nya seperti ini:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Aturan kedua bisa diterapkan karena ada tepat satu _input lifetime_. Aturan kedua 
menentukan bahwa _lifetime_ dari satu parameter input itu di-assign ke _output 
lifetime_, jadi _signature_-nya sekarang jadi kayak gini:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Sekarang semua referensi di _signature_ fungsi ini sudah punya _lifetimes_, dan 
_compiler_ bisa melanjutkan analisisnya tanpa perlu _programmer_ buat menganotasi 
_lifetimes_ di _signature_ fungsi ini.

Mari kita lihat contoh lain, kali ini memakai fungsi `longest` yang tidak punya 
parameter _lifetime_ pas kita mulai ngerjain di Listing 10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Mari terapkan aturan pertama: tiap parameter dapat _lifetime_-nya sendiri. Kali 
ini kita punya dua parameter bukannya satu, jadi kita punya dua _lifetimes_:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Kita bisa lihat kalau aturan kedua tidak bisa diterapkan karena ada lebih dari 
satu _input lifetime_. Aturan ketiga juga tidak bisa diterapkan, karena `longest` 
adalah sebuah fungsi bukannya _method_, jadi tidak ada parameter yang berupa 
`self`. Setelah melewati ketiga aturan ini, kita masih belum tahu apa _lifetime_ 
dari tipe kembaliannya. Inilah alasan kenapa kita dapat error pas nyoba 
men-compile kode di Listing 10-20: _compiler_ sudah melewati aturan-aturan 
_lifetime elision_ tapi masih belum bisa mencari tahu semua _lifetimes_ dari 
referensi yang ada di _signature_ tersebut.

Karena aturan ketiga sebenarnya cuma berlaku buat _method signatures_, kita 
bakal membahas _lifetimes_ di konteks tersebut selanjutnya buat melihat kenapa 
aturan ketiga ini bikin kita tidak perlu menganotasi _lifetimes_ di _method 
signatures_ terlalu sering.

### Anotasi Lifetime di Definisi Method

Saat kita mengimplementasikan _methods_ pada _struct_ yang punya _lifetimes_, kita 
memakai sintaks yang sama persis seperti parameter tipe generik, yang ditunjukkan 
di Listing 10-11. Di mana kita mendeklarasikan dan memakai parameter _lifetimes_ 
itu bergantung pada apakah mereka berhubungan dengan field _struct_-nya atau 
parameter dan nilai kembalian _method_-nya.

Nama _lifetime_ buat field _struct_ selalu harus dideklarasikan setelah keyword 
`impl` dan kemudian dipakai setelah nama _struct_-nya karena _lifetimes_ itu 
adalah bagian dari tipe _struct_-nya.

Di dalam _method signatures_ di dalam blok `impl`, referensi mungkin terikat ke 
_lifetime_ dari referensi di dalam field _struct_, atau mungkin mereka independen. 
Selain itu, aturan _lifetime elision_ sering kali bikin anotasi _lifetime_ tidak 
diperlukan lagi di _method signatures_. Mari kita lihat beberapa contoh yang 
memakai _struct_ bernama `ImportantExcerpt` yang kita definisikan di Listing 
10-24.

Pertama kita bakal memakai sebuah _method_ bernama `level` yang satu-satunya 
parameternya adalah referensi ke `self` dan nilai kembaliannya adalah sebuah `i32`, 
yang mana bukan referensi ke apa pun:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Deklarasi parameter _lifetime_ setelah `impl` dan pemakaiannya setelah nama tipe 
itu wajib, tapi kita tidak diwajibkan buat menganotasi _lifetime_ dari referensi 
ke `self` berkat aturan _elision_ yang pertama.

Ini adalah contoh di mana aturan _lifetime elision_ yang ketiga bisa diterapkan:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Ada dua _input lifetimes_, jadi Rust menerapkan aturan _lifetime elision_ pertama 
dan memberi baik `&self` maupun `announcement` _lifetimes_ mereka masing-masing. 
Lalu, karena salah satu parameternya adalah `&self`, tipe kembaliannya bakal dapat 
_lifetime_ dari `&self`, dan semua _lifetimes_ sudah lengkap terjelaskan.

### Lifetime Static

Ada satu _lifetime_ spesial yang perlu kita bahas yaitu `'static`, yang 
menandakan kalau referensi yang bersangkutan *bisa* hidup selama keseluruhan durasi 
dari program. Semua _string literals_ punya _lifetime_ `'static`, yang bisa 
kita anotasi seperti berikut:

```rust
let s: &'static str = "Saya punya lifetime static.";
```

Teks dari _string_ ini disimpan langsung di dalam _binary_ program kita, yang mana 
bakal selalu tersedia. Maka dari itu, _lifetime_ dari semua _string literals_ 
adalah `'static`.

Kita mungkin bakal melihat saran di pesan error untuk memakai _lifetime_ `'static`. 
Tapi sebelum menentukan `'static` sebagai _lifetime_ buat sebuah referensi, 
pikirkan dulu apakah referensi yang kita punya itu sebenarnya hidup selama 
keseluruhan _lifetime_ program kita atau tidak, dan apakah kita emang maunya 
begitu. Sebagian besar waktu, pesan error yang menyarankan _lifetime_ `'static` 
itu terjadi gara-gara kita mencoba membuat _dangling reference_ atau ada 
ketidakcocokan (mismatch) antara _lifetimes_ yang tersedia. Di kasus seperti 
itu, solusinya adalah dengan memperbaiki masalah utamanya, bukannya asal 
menentukan _lifetime_ `'static`.

## Parameter Tipe Generik, Trait Bounds, dan Lifetimes Secara Bersamaan

Mari kita lihat secara singkat sintaks buat menentukan parameter tipe generik, 
_trait bounds_, dan _lifetimes_ semuanya di satu fungsi!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Ini adalah fungsi `longest` dari Listing 10-21 yang mengembalikan _string slice_ 
yang lebih panjang dari antara dua _string slices_. Tapi sekarang fungsi ini 
punya parameter ekstra bernama `ann` dari tipe generik `T`, yang mana bisa 
diisi oleh tipe apa pun yang mengimplementasikan _trait_ `Display` seperti yang 
ditentukan sama klausa `where`. Parameter ekstra ini bakal dicetak memakai `{}`, 
itulah kenapa kita butuh _trait bound_ `Display`. Karena _lifetimes_ itu adalah 
salah satu tipe dari generik, deklarasi parameter _lifetime_ `'a` dan parameter 
tipe generik `T` berada di dalam satu daftar yang sama di dalam kurung sudut 
setelah nama fungsinya.

## Ringkasan

Kita sudah ngebahas banyak hal di bab ini! Sekarang setelah kita paham tentang 
parameter tipe generik, _traits_ dan _trait bounds_, serta parameter _lifetime_ 
generik, kita udah siap buat nulis kode tanpa pengulangan yang bisa jalan 
di berbagai situasi yang beda-beda. Parameter tipe generik ngasih kita 
kemampuan buat menerapkan kode ke tipe yang berbeda. _Traits_ dan _trait bounds_ 
memastikan kalau walaupun tipe-tipenya generik, mereka tetap bakal punya 
perilaku yang dibutuhin sama kode kita. Kita udah belajar gimana cara memakai 
anotasi _lifetime_ buat memastikan kalau kode fleksibel ini tidak bakal punya 
_dangling references_ (referensi yang menggantung). Dan semua analisis ini 
terjadi saat _compile time_, yang sama sekali tidak memengaruhi performa saat 
_runtime_!

Percaya atau tidak, masih banyak lagi yang bisa dipelajari soal topik-topik 
yang kita bahas di bab ini: Bab 18 bakal ngebahas _trait objects_, yang merupakan 
cara lain buat memakai _traits_. Ada juga skenario-skenario yang lebih rumit 
yang melibatkan anotasi _lifetime_ yang cuma bakal kita perlukan di situasi 
yang sangat tingkat lanjut (advanced); buat itu, kita bisa membaca 
[Rust Reference][reference]. Tapi buat langkah selanjutnya, kita bakal belajar 
gimana cara menulis _tests_ di Rust supaya kita bisa memastikan kalau kode kita 
berjalan persis seperti yang seharusnya.

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/index.html
