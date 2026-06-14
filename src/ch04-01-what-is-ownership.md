## Apa itu Ownership?

_Ownership_ (Kepemilikan) adalah sekumpulan aturan yang ngatur gimana program 
Rust ngelola memori. Semua program harus ngatur cara mereka pake memori 
komputer pas lagi jalan. Beberapa bahasa punya _garbage collection_ (GC) yang 
secara rutin nyari memori yang udah nggak kepake pas programnya jalan; di bahasa 
lain, programmer harus secara eksplisit ngalokasiin dan ngebebasin memorinya. 
Rust pake pendekatan ketiga: memori dikelola lewat sistem _ownership_ dengan 
sekumpulan aturan yang dicek sama _compiler_. Kalau ada aturan yang dilanggar, 
programnya nggak bakal ke-compile. Nggak ada satu pun fitur dari _ownership_ 
yang bakal bikin program kita jadi lemot pas lagi jalan.

Karena _ownership_ itu konsep baru buat banyak programmer, emang butuh waktu 
buat terbiasa. Kabar baiknya, makin kita berpengalaman sama Rust dan aturan 
sistem _ownership_-nya, kita bakal makin gampang buat nulis kode yang aman dan 
efisien secara alami. Semangat terus ya!

Pas kita paham _ownership_, kita bakal punya pondasi yang kuat buat mahamin 
fitur-fitur yang bikin Rust unik. Di bab ini, kita bakal belajar _ownership_ 
lewat beberapa contoh yang fokus ke struktur data yang sangat umum: _strings_.

> ### Stack dan Heap
>
> Banyak bahasa pemrograman nggak nuntut kita buat sering-sering mikirin soal 
> _stack_ sama _heap_. Tapi di bahasa pemrograman sistem kayak Rust, apakah 
> sebuah nilai ada di _stack_ atau _heap_ itu ngaruh ke gimana bahasanya 
> berperilaku dan kenapa kita harus ngambil keputusan tertentu. Bagian-bagian 
> dari _ownership_ bakal dijelasin hubungannya sama _stack_ dan _heap_ nanti di 
> bab ini, jadi ini penjelasan singkat buat persiapan.
>
> Baik _stack_ maupun _heap_ adalah bagian dari memori yang tersedia buat dipake 
> kode kita pas _runtime_, tapi mereka disusun dengan cara yang beda. _Stack_ 
> nyimpen nilai sesuai urutan yang dia dapet terus ngapus nilainya dengan urutan 
> kebalikannya. Ini disebut _last in, first out_ (LIFO). Bayangin tumpukan 
> piring: pas kita nambahin piring lagi, kita taruh di atas tumpukannya, dan pas 
> kita butuh piring, kita ambil satu dari paling atas. Nambahin atau ngambil 
> piring dari tengah atau bawah nggak bakal semudah itu! Nambahin data disebut 
> _pushing onto the stack_, dan ngambil data disebut _popping off the stack_. 
> Semua data yang disimpan di _stack_ harus punya ukuran yang udah tau dan 
> tetap. Data dengan ukuran yang nggak tau pas _compile time_ atau ukuran yang 
> mungkin berubah harus disimpan di _heap_.
>
> _Heap_ itu kurang teratur: pas kita naruh data di _heap_, kita minta sejumlah 
> tempat tertentu. _Memory allocator_ bakal nemuin tempat kosong di _heap_ yang 
> cukup gede, nandain tempat itu lagi dipake, terus balikin sebuah _pointer_, 
> yaitu alamat dari lokasi itu. Proses ini disebut _allocating on the heap_ dan 
> kadang disingkat jadi _allocating_ doang (naruh nilai ke _stack_ nggak 
> dianggap sebagai _allocating_). Karena _pointer_ ke _heap_ itu ukurannya udah 
> tau dan tetap, kita bisa nyimpen _pointer_-nya di _stack_, tapi pas kita mau 
> datanya benar-benar, kita harus ngikutin _pointer_-nya. Bayangin kayak duduk di 
> restoran. Pas masuk, kita bilang jumlah orang di grup kita, terus pelayannya 
> nemuin meja kosong yang pas buat semuanya terus nganterin kita ke sana. Kalau 
> ada temen kita yang telat dateng, mereka bisa nanya kita duduk di mana buat 
> nemuin kita.
>
> _Pushing to the stack_ itu lebih cepet daripada _allocating on the heap_ 
> karena _allocator_ nggak perlu cari-cari tempat buat nyimpen data baru; 
> lokasinya selalu di paling atas _stack_. Sebagai perbandingan, ngalokasiin 
> tempat di _heap_ butuh kerja ekstra karena _allocator_ harus nemuin dulu 
> tempat yang cukup gede buat nampung datanya terus ngelakuin pembukuan buat 
> persiapan alokasi selanjutnya.
>
> Akses data di _heap_ umumnya lebih lambat daripada akses data di _stack_ 
> karena kita harus ngikutin _pointer_ buat nyampe ke sana. Prosesor zaman 
> sekarang bakal lebih cepet kalau mereka nggak terlalu banyak lompat-lompat di 
> memori. Lanjutin analoginya, bayangin seorang pelayan di restoran yang ngambil 
> orderan dari banyak meja. Bakal paling efisien kalau dia ngambil semua 
> orderan di satu meja sebelum lanjut ke meja berikutnya. Ngambil orderan dari 
> meja A, terus meja B, terus meja A lagi, terus meja B lagi bakal jadi proses 
> yang jauh lebih lambat. Dengan cara yang sama, prosesor biasanya bisa 
> ngerjain tugasnya lebih baik kalau dia kerja sama data yang deket sama data 
> lainnya (kayak di _stack_) bukannya yang jauh (kayak yang mungkin terjadi di 
> _heap_).
>
> Pas kode kita manggil sebuah fungsi, nilai-nilai yang dimasukin ke fungsinya 
> (termasuk, mungkin, _pointer_ ke data di _heap_) sama variabel lokal fungsinya 
> bakal di-_push_ ke _stack_. Pas fungsinya kelar, nilai-nilai itu bakal di-_pop_ 
> keluar dari _stack_.
>
> Mantau bagian kode mana yang lagi pake data apa di _heap_, minimalisir jumlah 
> data duplikat di _heap_, dan ngebersihin data yang udah nggak kepake di _heap_ 
> biar nggak keabisan tempat adalah masalah-masalah yang diselesein sama 
> _ownership_. Sekali kita paham _ownership_, kita nggak bakal butuh sering-
> sering mikirin soal _stack_ sama _heap_, tapi tau kalau tujuan utama 
> _ownership_ adalah buat ngelola data _heap_ bisa bantu jelasin kenapa dia 
> cara kerjanya kayak gitu.

### Aturan Ownership

Pertama, yuk kita liat aturan-aturan _ownership_. Inget terus aturan ini pas 
kita ngerjain contoh-contoh yang bakal ngejelasin aturan ini:

- Tiap nilai di Rust punya seorang _owner_ (pemilik).
- Cuma boleh ada satu _owner_ dalam satu waktu.
- Pas _owner_-nya keluar dari scope, nilainya bakal di-_drop_ (dihapus).

### Scope Variabel

Sekarang setelah kita ngelewatin sintaks dasar Rust, kita nggak bakal masukin 
semua kode `fn main() {` di contoh-contohnya, jadi kalau kita lagi ngikutin, 
pastiin buat masukin contoh-contoh berikut ke dalem fungsi `main` secara manual. 
Hasilnya, contoh-contoh kita bakal lebih singkat, biar kita bisa fokus ke 
detail aslinya bukannya kode _boilerplate_.

Sebagai contoh pertama dari _ownership_, kita bakal liat _scope_ dari beberapa 
variabel. Sebuah scope adalah range di dalem program di mana sebuah item itu 
valid. Coba liat variabel ini:

```rust
let s = "hello";
```

Variabel `s` ngerujuk ke sebuah literal string, di mana nilai string-nya di-_hardcoded_ 
ke teks program kita. Variabelnya valid dari titik di mana dia dideklarasikan 
sampe akhir dari _scope_ saat ini. Listing 4-1 nunjukin program dengan komentar 
yang dianotasi di mana variabel `s` bakal valid.

<Listing number="4-1" caption="Sebuah variabel dan scope di mana dia valid">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

Dengan kata lain, ada dua titik waktu penting di sini:

- Pas `s` masuk _ke dalam_ scope, dia jadi valid.
- Dia tetep valid sampe dia keluar _dari_ scope.

Sampai titik ini, hubungan antara scope sama kapan variabel itu valid mirip 
sama bahasa pemrograman lainnya. Sekarang kita bakal kembangin pemahaman ini 
dengan ngenalin tipe `String`.

### Tipe `String`

Buat gambarin aturan _ownership_, kita butuh tipe data yang lebih kompleks dari 
yang udah kita bahas di bagian [“Tipe Data”][data-types] di Bab 3. Tipe-tipe 
yang udah dibahas sebelumnya ukurannya udah tau, bisa disimpan di _stack_ dan 
di-_pop_ keluar dari _stack_ pas scope-nya abis, dan bisa di-copy secara cepet 
dan gampang buat bikin instance baru yang independen kalau bagian kode lain 
perlu pake nilai yang sama di scope yang beda. Tapi kita mau liat data yang 
disimpan di _heap_ dan eksplor gimana Rust tau kapan harus ngebersihin data itu, 
dan tipe `String` adalah contoh yang oke sekali.

Kita bakal fokus ke bagian-bagian `String` yang terkait sama _ownership_. 
Aspek-aspek ini juga berlaku buat tipe data kompleks lainnya, baik yang 
disediain standard library maupun yang kita bikin sendiri. Kita bakal bahas 
`String` lebih dalem di [Bab 8][ch8].

Kita udah liat literal string, di mana nilai string-nya di-_hardcoded_ ke program 
kita. Literal string emang nyaman, tapi mereka nggak cocok buat semua situasi 
di mana kita mungkin mau pake teks. Salah satu alasannya karena mereka itu 
_immutable_. Alasan lainnya karena nggak semua nilai string bisa diketahuin pas 
kita nulis kode: misalnya, gimana kalau kita mau ngambil input user terus nyimpannya? 
Buat situasi kayak gini, Rust punya tipe string kedua, yaitu `String`. Tipe ini 
ngelola data yang dialokasikan di _heap_ dan makanya dia bisa nyimpen sejumlah 
teks yang ukurannya nggak kita ketahuin pas _compile time_. Kita bisa bikin 
sebuah `String` dari literal string pake fungsi `from`, kayak gini:

```rust
let s = String::from("hello");
```

Operator titik dua ganda `::` ngebolehin kita buat ngelempokin fungsi `from` 
ini di bawah tipe `String` bukannya pake nama kayak `string_from`. Kita bakal 
bahas sintaks ini lebih lanjut di bagian [“Sintaks Method”][method-syntax] di 
Bab 5, dan pas kita bahas soal _namespacing_ pake modul di [“Path buat Ngerujuk Item di Pohon Modul”][paths-module-tree] 
di Bab 7.

Jenis string ini _bisa_ diubah (mutated):

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Jadi, apa bedanya di sini? Kenapa `String` bisa diubah tapi literal nggak bisa? 
Bedanya ada di gimana kedua tipe ini nanganin memori.

### Memori dan Alokasi

Di kasus literal string, kita tau isinya pas _compile time_, jadi teksnya di-_hardcoded_ 
langsung ke file executable final-nya. Ini kenapa literal string itu cepet dan 
efisien. Tapi sifat-sifat ini cuma dateng dari sifat _immutability_ literal 
string-nya. Sayangnya, kita nggak bisa naruh sepotong memori ke dalem biner 
buat tiap teks yang ukurannya nggak tau pas _compile time_ dan ukurannya mungkin 
berubah pas lagi jalanin programnya.

Dengan tipe `String`, buat support sepotong teks yang _mutable_ dan bisa nambah 
ukurannya, kita perlu ngalokasiin sejumlah memori di _heap_, yang nggak tau pas 
_compile time_, buat nampung isinya. Ini artinya:

- Memorinya harus diminta dari _memory allocator_ pas _runtime_.
- Kita butuh cara buat balikin memori ini ke _allocator_ pas kita udah selese 
  pake `String` kita.

Bagian pertama itu kita yang ngerjain: pas kita manggil `String::from`, 
implementasinya minta memori yang dia butuhin. Ini hal yang cukup universal di 
bahasa pemrograman.

Tapi, bagian kedua itu beda. Di bahasa yang punya _garbage collector (GC)_, GC 
bakal terus mantau dan ngebersihin memori yang udah nggak dipake lagi, dan kita 
nggak perlu mikirin itu. Di kebanyakan bahasa tanpa GC, itu tanggung jawab kita 
buat ngenalin kapan memori udah nggak dipake lagi terus manggil kode buat secara 
eksplisit ngebebasinnya, sama kayak pas kita memintanya. Ngelakuin ini dengan 
bener secara historis udah jadi masalah pemrograman yang susah. Kalau kita lupa, 
kita bakal buang-buang memori. Kalau kita lakuin terlalu cepet, kita bakal punya 
variabel yang nggak valid. Kalau kita lakuin dua kali, itu juga sebuah _bug_. 
Kita perlu masangin tepat satu `allocate` sama tepat satu `free`.

Rust ngambil jalur yang beda: memorinya otomatis dibalikin begitu variabel yang 
punya (_owns_) memori itu keluar dari scope. Ini versi contoh scope kita dari 
Listing 4-1 pake `String` bukannya literal string:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

Ada titik waktu alami di mana kita bisa balikin memori yang dibutuhin `String` 
kita ke _allocator_: pas `s` keluar dari scope. Pas sebuah variabel keluar dari 
scope, Rust manggil fungsi khusus buat kita. Fungsi ini namanya [`drop`][drop], 
dan di situlah pembuat `String` bisa naruh kode buat balikin memorinya. Rust 
manggil `drop` secara otomatis di kurung kurawal tutup.

> Catatan: Di C++, pola nge-dealokasi resource di akhir masa hidup sebuah item 
> ini kadang disebut _Resource Acquisition Is Initialization (RAII)_. Fungsi 
> `drop` di Rust bakal terasa familiar kalau kita pernah pake pola-pola RAII.

Pola ini punya pengaruh yang sangat dalem ke gimana kode Rust ditulis. Mungkin 
keliatan simpel sekarang, tapi perilaku kodenya bisa jadi nggak terduga di 
situasi yang lebih ribet pas kita mau punya banyak variabel pake data yang 
udah kita alokasiin di _heap_. Yuk kita eksplor beberapa situasi itu sekarang.

<!-- Old heading. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-move"></a>

#### Interaksi Variabel dan Data dengan Move

Beberapa variabel bisa berinteraksi sama data yang sama dengan berbagai cara di 
Rust. Yuk kita liat contoh pake integer di Listing 4-2.

<Listing number="4-2" caption="Assign nilai integer variabel `x` ke `y`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

Kita mungkin bisa nebak apa yang dilakuin kode ini: “bind nilai `5` ke `x`; terus 
bikin copy dari nilai di `x` terus bind ke `y`.” Kita sekarang punya dua 
variabel, `x` sama `y`, dan keduanya sama dengan `5`. Ini emang bener yang 
terjadi, karena integer adalah nilai simpel dengan ukuran yang udah tau dan 
tetap, dan dua nilai `5` ini di-_push_ ke _stack_.

Sekarang yuk liat versi `String`-nya:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Ini keliatannya mirip sekali, jadi kita mungkin asumsikan kalau cara kerjanya 
bakal sama: yaitu, baris kedua bakal bikin copy dari nilai di `s1` terus bind 
ke `s2`. Tapi nggak gitu yang sebenernya terjadi.

Coba liat Gambar 4-1 buat liat apa yang terjadi di `String` di balik layar. 
Sebuah `String` disusun dari tiga bagian, yang ditunjukin di sebelah kiri: 
sebuah _pointer_ ke memori yang nampung isi string-nya, sebuah _length_ (panjang), 
dan sebuah _capacity_ (kapasitas). Grup data ini disimpan di _stack_. Di sebelah 
kanan adalah memori di _heap_ yang nampung isinya.

<img alt="Dua tabel: tabel pertama isinya representasi s1 di stack, terdiri 
dari length (5), capacity (5), dan sebuah pointer ke nilai pertama di tabel 
kedua. Tabel kedua isinya representasi data string di heap, byte demi byte." 
src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Gambar 4-1: Representasi di memori dari sebuah `String` 
yang nampung nilai `"hello"` yang di-bind ke `s1`</span>

_Length_ itu seberapa banyak memori, dalam byte, yang lagi dipake isinya 
`String` saat ini. _Capacity_ itu total jumlah memori, dalam byte, yang diterima 
`String` dari _allocator_. Perbedaan antara _length_ sama _capacity_ itu penting, 
tapi nggak di konteks ini, jadi buat sekarang, cuekin aja _capacity_-nya.

Pas kita nge-assign `s1` ke `s2`, data `String`-nya di-copy, artinya kita copy 
_pointer_, _length_, dan _capacity_ yang ada di _stack_. Kita nggak copy data 
yang ada di _heap_ yang dirujuk sama _pointer_-nya. Dengan kata lain, 
representasi data di memori keliatannya kayak Gambar 4-2.

<img alt="Tiga tabel: tabel s1 dan s2 merepresentasikan string itu di stack, 
masing-masing, dan keduanya nunjuk ke data string yang sama di heap." 
src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Gambar 4-2: Representasi di memori dari variabel `s2` 
yang punya copy dari pointer, length, dan capacity dari `s1`</span>

Representasinya _nggak_ keliatan kayak Gambar 4-3, yang merupakan penampakan 
memori kalau misalnya Rust malah ikut copy data _heap_-nya juga. Kalau Rust 
lakuin ini, operasi `s2 = s1` bisa jadi sangat mahal dalam hal performa 
Pas _runtime_ kalau datanya di _heap_ itu sangat besar.

<img alt="Empat tabel: dua tabel merepresentasikan data stack buat s1 dan s2, 
dan masing-masing nunjuk ke copy data string-nya sendiri di heap." 
src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Gambar 4-3: Kemungkinan lain soal apa yang mungkin 
dilakuin `s2 = s1` kalau Rust ikut copy data heap-nya juga</span>

Tadi kita bilang kalau pas sebuah variabel keluar dari scope, Rust otomatis 
manggil fungsi `drop` dan ngebersihin memori _heap_ buat variabel itu. Tapi 
Gambar 4-2 nunjukin kedua _pointer_ data nunjuk ke lokasi yang sama. Ini masalah: 
pas `s2` sama `s1` keluar dari scope, mereka berdua bakal nyoba buat ngebebasin 
memori yang sama. Ini dikenal sebagai _double free_ error dan merupakan salah 
satu _bug memory safety_ yang kita sebutin sebelumnya. Ngebebasin memori dua 
kali bisa bikin kerusakan memori (memory corruption), yang berpotensi memicu 
kerentanan keamanan.

Buat mastiin keamanan memori, setelah baris `let s2 = s1;`, Rust nganggep `s1` 
udah nggak valid lagi. Makanya, Rust nggak perlu ngebebasin apa pun pas `s1` 
keluar dari scope. Coba liat apa yang terjadi pas kita nyoba pake `s1` setelah 
`s2` dibuat; nggak bakal bisa:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Kita bakal dapet error kayak gini karena Rust ngelarang kita pake referensi yang 
udah nggak valid:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Kalau kita pernah denger istilah _shallow copy_ (copy dangkal) sama _deep copy_ 
(copy dalem) pas lagi belajar bahasa lain, konsep nyalin _pointer_, _length_, 
dan _capacity_ tanpa nyalin datanya mungkin kedengeran kayak lagi bikin _shallow 
copy_. Tapi karena Rust juga ngebatalin variabel pertamanya, bukannya disebut 
_shallow copy_, ini dikenal sebagai _move_ (pindah). Di contoh ini, kita bakal 
bilang kalau `s1` udah di-_move_ ke dalem `s2`. Jadi, apa yang benar-benar terjadi 
ditunjukin di Gambar 4-4.

<img alt="Tiga tabel: tabel s1 dan s2 merepresentasikan string itu di stack, 
masing-masing, dan keduanya nunjuk ke data string yang sama di heap. Tabel s1 
di-grayed out karena s1 udah nggak valid; cuma s2 yang bisa dipake buat akses 
data heap-nya." src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Gambar 4-4: Representasi di memori setelah `s1` udah 
dibatalkan</span>

Itu nyelesein masalah kita! Dengan cuma `s2` yang valid, pas dia keluar dari 
scope cuma dia sendiri yang bakal ngebebasin memorinya, dan beres deh.

Sebagai tambahan, ada pilihan desain yang tersirat dari sini: Rust nggak bakal 
pernah otomatis bikin "deep" copy dari data kita. Makanya, penyalinan _otomatis_ 
apa pun bisa diasumsikan nggak mahal dalam hal performa pas _runtime_.

#### Scope dan Assignment

Kebalikan dari ini juga bener buat hubungan antara _scoping_, _ownership_, dan 
memori yang dibebasin lewat fungsi `drop` juga. Pas kita ngasih nilai yang 
bener-bener baru ke variabel yang udah ada, Rust bakal manggil `drop` dan 
ngebebasin memori nilai aslinya langsung. Coba liat kode ini, contohnya:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

Kita awalnya mendeklarasikan variabel `s` terus di-bind ke sebuah `String` 
dengan nilai `"hello"`. Terus kita langsung bikin `String` baru dengan nilai 
`"ahoy"` terus di-assign ke `s`. Di titik ini, nggak ada apa pun yang ngerujuk 
ke nilai asli di _heap_ sama sekali.

<img alt="Satu tabel s merepresentasikan nilai string di stack, nunjuk ke 
potongan data string kedua (ahoy) di heap, dengan data string asli (hello) 
di-grayed out karena udah nggak bisa diakses lagi." 
src="img/trpl04-05.svg" 
class="center" 
style="width: 50%;" 
/>

<span class="caption">Gambar 4-5: Representasi di memori setelah nilai awal 
udah diganti seluruhnya.</span>

String aslinya makanya langsung keluar dari scope. Rust bakal jalanin fungsi 
`drop` padanya dan memorinya bakal langsung dibebasin. Pas kita nyetak nilainya 
di akhir, hasilnya bakal `"ahoy, world!"`.

<!-- Old heading. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-clone"></a>

#### Interaksi Variabel dan Data dengan Clone

Kalau kita _emang_ mau copy data _heap_ dari `String` secara dalem (deeply copy), 
nggak cuma data _stack_-nya aja, kita bisa pake method umum namanya `clone`. 
Kita bakal bahas sintaks method di Bab 5, tapi karena method adalah fitur umum 
di banyak bahasa pemrograman, kita mungkin udah pernah liat sebelumnya.

Ini contoh method `clone` beraksi:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Ini jalan dengan lancar dan secara eksplisit ngasilin perilaku yang ditunjukin 
di Gambar 4-3, di mana data _heap_-nya _emang_ ikut di-copy.

Pas kita liat pemanggilan ke `clone`, kita tau kalau ada sejumlah kode sembarang 
yang lagi dijalankan dan kode itu mungkin mahal harganya. Ini adalah indikator 
visual kalau ada sesuatu yang beda yang lagi terjadi.

#### Data Khusus Stack: Copy

Ada hal unik lain yang belum kita bahas. Kode yang pake integer ini—yang 
sebagiannya ditunjukin di Listing 4-2—jalan dan valid:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Tapi kode ini kayaknya bertentangan sama apa yang baru aja kita pelajari: kita 
nggak manggil `clone`, tapi `x` tetep valid dan nggak di-_move_ ke dalem `y`.

Alasannya karena tipe-tipe kayak integer yang ukurannya udah tau pas _compile 
time_ disimpan seluruhnya di _stack_, jadi nyalin nilai aslinya itu cepet buat 
dilakuin. Itu artinya nggak ada alasan kenapa kita mau ngelarang `x` buat tetep 
valid setelah kita bikin variabel `y`. Dengan kata lain, nggak ada bedanya 
antara _deep copy_ sama _shallow copy_ di sini, jadi manggil `clone` nggak bakal 
ngelakuin hal yang beda dari _shallow copy_ biasa, makanya kita bisa 
ngelewatinnya.

Rust punya anotasi khusus namanya trait `Copy` yang bisa kita taruh di tipe-tipe 
yang disimpan di _stack_, kayak integer (kita bakal bahas traits lebih banyak 
di [Bab 10][traits]). Kalau sebuah tipe mengimplementasikan trait `Copy`, 
variabel yang pakenya nggak bakal di-_move_, tapi lebih ke disalin secara 
sepele, bikin mereka tetep valid setelah di-assign ke variabel lain.

Rust nggak bakal ngebolehin kita ngasih anotasi `Copy` ke sebuah tipe kalau 
tipe itu, atau bagian apa pun darinya, udah mengimplementasikan trait `Drop`. 
Kalau tipenya butuh sesuatu yang khusus terjadi pas nilainya keluar dari scope 
terus kita nambahin anotasi `Copy` ke tipe itu, kita bakal dapet _compile-time 
error_. Buat belajar gimana cara nambahin anotasi `Copy` ke tipe kita buat 
mengimplementasikan trait-nya, liat [“Derivable Traits”][derivable-traits] di 
Lampiran C.

Jadi, tipe apa aja yang mengimplementasikan trait `Copy`? Kita bisa cek 
dokumentasi buat tipe tertentu buat mastiin, tapi sebagai aturan umum, 
kumpulan nilai scalar simpel apa pun bisa mengimplementasikan `Copy`, dan nggak 
ada satu pun yang butuh alokasi atau bentuk resource apa pun yang bisa 
mengimplementasikan `Copy`. Ini beberapa tipe yang mengimplementasikan `Copy`:

- Semua tipe integer, kayak `u32`.
- Tipe Boolean, `bool`, dengan nilai `true` sama `false`.
- Semua tipe _floating-point_, kayak `f64`.
- Tipe karakter, `char`.
- Tuple, kalau isinya cuma tipe-tipe yang juga mengimplementasikan `Copy`. 
  Contohnya, `(i32, i32)` mengimplementasikan `Copy`, tapi `(i32, String)` nggak.

### Ownership dan Fungsi

Mekanisme masukin nilai ke sebuah fungsi mirip sama pas kita ngasih nilai ke 
sebuah variabel. Masukin variabel ke fungsi bakal nge-_move_ atau copy, sama 
kayak assignment. Listing 4-3 punya contoh dengan beberapa anotasi yang nunjukin 
di mana variabel masuk dan keluar dari scope.

<Listing number="4-3" file-name="src/main.rs" caption="Fungsi dengan ownership dan scope yang dianotasi">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

Kalau kita nyoba pake `s` setelah manggil `takes_ownership`, Rust bakal ngelepar 
_compile-time error_. Pengecekan statis ini ngelindungin kita dari kesalahan. 
Coba tambahin kode ke `main` yang pake `s` sama `x` buat liat di mana kita bisa 
pake mereka dan di mana aturan _ownership_ ngelarang kita buat ngelakuin itu.

### Nilai Return dan Scope

Balikin nilai (returning values) juga bisa mentransfer _ownership_. Listing 4-4 
nunjukin contoh fungsi yang balikin sebuah nilai, dengan anotasi yang mirip 
kayak di Listing 4-3.

<Listing number="4-4" file-name="src/main.rs" caption="Transfer ownership dari nilai return">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

_Ownership_ sebuah variabel ngikutin pola yang sama tiap kalinya: ngasih nilai 
ke variabel lain bakal nge-_move_ nilainya. Pas sebuah variabel yang isinya 
data di _heap_ keluar dari scope, nilainya bakal dibersihin sama `drop` kecuali 
kalau _ownership_ datanya udah di-_move_ ke variabel lain.

Walaupun ini jalan, ngambil _ownership_ terus balikin lagi di tiap fungsi itu 
agak ribet. Gimana kalau kita mau ngebolehin sebuah fungsi pake sebuah nilai 
tapi nggak usah ngambil _ownership_-nya? Agak nyebelin kan kalau apa pun yang 
kita masukin juga harus dibalikin lagi kalau kita mau pake lagi, ditambah 
data apa pun hasil dari body fungsinya yang mungkin juga mau kita balikin.

Rust ngebolehin kita buat balikin banyak nilai pake tuple, kayak yang ditunjukin 
di Listing 4-5.

<Listing number="4-5" file-name="src/main.rs" caption="Balikin ownership dari parameter">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

Tapi ini terlalu banyak upacaranya (ceremony) dan kerjaan sekali buat konsep 
yang harusnya umum. Untungnya buat kita, Rust punya fitur buat pake sebuah nilai 
tanpa mentransfer _ownership_, namanya _references_ (referensi).

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[method-syntax]: ch05-03-method-syntax.html#method-syntax
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[drop]: ../std/ops/trait.Drop.html#tymethod.drop
