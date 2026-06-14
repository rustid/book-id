## Unsafe Rust

Semua kode yang udah kita bahas sejauh ini punya jaminan keamanan memori (memory 
safety guarantees) yang ditegakkan oleh Rust saat _compile time_. Namun, Rust 
punya sebuah bahasa kedua yang tersembunyi di dalamnya yang tidak menegakkan 
jaminan keamanan memori ini: namanya _unsafe Rust_ dan dia bekerja persis kayak 
Rust biasa, tapi ngasih kita kekuatan super (superpowers) tambahan.

Unsafe Rust eksis karena, secara natur, analisis statis itu sifatnya konservatif. 
Saat _compiler_ mencoba buat menentukan apakah sebuah kode mematuhi jaminan 
keamanannya atau tidak, jauh lebih baik baginya buat menolak beberapa program 
yang sebenarnya valid ketimbang menerima beberapa program yang ternyata tidak 
valid. Walaupun kodenya _mungkin_ baik-baik aja, kalau _compiler_ Rust tidak 
punya informasi yang cukup buat merasa yakin, dia bakal menolak kode tersebut. 
Di kasus-kasus seperti ini, kita bisa memakai kode _unsafe_ buat ngasih tahu 
_compiler_, “Percaya deh, aku tahu apa yang lagi aku lakuin.” Namun, 
peringatan: kita memakai _unsafe_ Rust dengan risiko kita sendiri: kalau kita 
memakai kode _unsafe_ dengan tidak benar, masalah-masalah bisa bermunculan 
akibat tidak amannya memori, seperti proses _dereferencing_ pada _null pointer_.

Alasan lain kenapa Rust punya sebuah alter ego yang _unsafe_ adalah karena *hardware* 
komputer yang mendasarinya (underlying computer hardware) itu sifatnya memang 
tidak aman (_inherently unsafe_). Kalau Rust tidak membiarkan kita melakukan 
operasi-operasi yang tidak aman, kita tidak bakal bisa mengerjakan tugas-tugas 
tertentu. Rust perlu mengizinkan kita melakukan pemrograman sistem tingkat rendah 
(low-level systems programming), kayak berinteraksi secara langsung sama sistem 
operasi atau bahkan menulis sistem operasi kita sendiri. Bekerja dengan pemrograman 
sistem tingkat rendah adalah salah satu tujuan dari bahasa ini. Mari kita 
eksplorasi apa aja yang bisa kita lakukan dengan _unsafe_ Rust dan gimana 
cara melakukannya.

### Unsafe Superpowers

Buat beralih ke _unsafe_ Rust, gunakan keyword `unsafe` lalu mulai sebuah blok baru 
buat menampung kode _unsafe_ tersebut. Ada lima hal yang bisa kita lakukan di 
_unsafe_ Rust yang tidak bisa kita lakukan di Rust biasa (safe Rust), yang mana 
kita sebut sebagai *unsafe superpowers*. Kekuatan super tersebut meliputi kemampuan 
buat:

1. Men-dereferensi sebuah _raw pointer_ (pointer mentah)
1. Memanggil fungsi atau method _unsafe_
1. Mengakses atau memodifikasi variabel _static_ yang _mutable_
1. Mengimplementasikan trait _unsafe_
1. Mengakses field dari sebuah `union`

Penting sekali buat dipahami kalau `unsafe` tidak mematikan _borrow checker_ atau 
menonaktifkan (disable) pengecekan keamanan Rust lainnya: kalau kita memakai 
sebuah referensi di dalam kode _unsafe_, dia bakal tetap dicek. Keyword `unsafe` 
cuma ngasih kita akses ke lima fitur ini yang kemudian tidak bakal dicek sama 
_compiler_ buat keamanan memori. Kita bakal tetap dapat tingkat keamanan 
tertentu di dalam sebuah blok _unsafe_.

Selain itu, `unsafe` tidak berarti kalau kode di dalam blok tersebut pasti berbahaya 
atau bahwa dia pasti bakal punya masalah keamanan memori: maksud utamanya adalah 
sebagai programmer, Andalah yang bakal memastikan kalau kode di dalam blok 
`unsafe` tersebut bakal mengakses memori dengan cara yang valid.

Manusia itu bisa aja salah dan kesalahan (mistakes) emang bakal terjadi, tapi dengan 
mewajibkan kelima operasi _unsafe_ ini buat berada di dalam blok-blok yang dianotasi 
dengan `unsafe`, kita bakal tahu kalau error apa pun yang berkaitan dengan keamanan 
memori itu pasti ada di dalam sebuah blok `unsafe`. Usahakan supaya blok `unsafe` 
itu kecil; kita bakal bersyukur nanti pas kita lagi nyelidikin _bugs_ memori.

Buat mengisolasi kode _unsafe_ sebanyak mungkin, praktik terbaiknya adalah membungkus 
(enclose) kode semacam itu di dalam sebuah abstraksi yang aman (safe abstraction) 
lalu menyediakan API yang aman, yang mana bakal kita bahas nanti di bab ini pas 
kita meneliti fungsi dan method _unsafe_. Beberapa bagian dari _standard library_ 
diimplementasikan sebagai abstraksi yang aman di atas kode _unsafe_ yang udah diaudit. 
Membungkus kode _unsafe_ di dalam sebuah abstraksi yang aman bakal mencegah 
penggunaan `unsafe` agar tidak bocor (leaking out) ke semua tempat di mana kita 
atau *user* kita mungkin pengen memakai fungsionalitas yang diimplementasikan dengan 
kode `unsafe` tersebut, karena memakai sebuah abstraksi yang aman itu sifatnya aman.

Mari kita bahas masing-masing dari lima *unsafe superpowers* tersebut satu per satu. 
Kita juga bakal melihat beberapa abstraksi yang nyediain *interface* (antarmuka) 
yang aman buat mengakses kode _unsafe_.

### Men-dereferensi sebuah Raw Pointer

Di Bab 4, di bagian [“Dangling References”][dangling-references], kita menyebutkan 
kalau _compiler_ selalu memastikan kalau referensi itu selalu valid. _Unsafe_ Rust 
punya dua tipe baru bernama _raw pointers_ yang mirip sama referensi. Sama kayak 
referensi, _raw pointers_ bisa bersifat _immutable_ atau _mutable_ dan masing-masing 
ditulis sebagai `*const T` dan `*mut T`. Tanda bintang (`*`) di sini bukanlah operator 
dereferensi; dia adalah bagian dari nama tipenya. Di dalam konteks _raw pointers_, 
_immutable_ berarti kalau pointer tersebut tidak bisa secara langsung di-_assign_ (diisi 
nilai baru) setelah ia di-dereferensi.

Berbeda dari referensi dan _smart pointers_, _raw pointers_:

- Diizinkan buat ngabaikan aturan _borrowing_ dengan membiarkan kita punya baik 
  pointer _immutable_ maupun _mutable_, atau banyak pointer _mutable_ yang menunjuk 
  ke lokasi yang sama
- Tidak dijamin bakal menunjuk ke memori yang valid
- Diizinkan buat bernilai null
- Tidak mengimplementasikan pembersihan otomatis (automatic cleanup) apa pun

Dengan memilih keluar (opting out) dari ditegakkannya jaminan-jaminan ini oleh 
Rust, kita bisa mengorbankan (give up) keamanan yang dijamin demi mendapatkan 
performa yang lebih besar atau kemampuan buat berinteraksi sama bahasa pemrograman 
lain atau *hardware* yang mana jaminan Rust tidak berlaku di situ.

Listing 20-1 menunjukkan gimana cara membikin sebuah _raw pointer_ yang _immutable_ 
dan yang _mutable_.

<Listing number="20-1" caption="Membikin raw pointers memakai operator raw borrow">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Perhatikan bahwa kita tidak memasukkan keyword `unsafe` di kode ini. Kita bisa membikin 
_raw pointers_ di dalam _safe code_ (kode yang aman); kita cuma tidak bisa 
men-dereferensi _raw pointers_ di luar sebuah blok _unsafe_, kayak yang bakal kita 
lihat sebentar lagi.

Kita udah membikin _raw pointers_ dengan memakai operator *raw borrow*: `&raw const num` 
membikin sebuah _raw pointer_ _immutable_ `*const i32`, dan `&raw mut num` membikin 
sebuah _raw pointer_ _mutable_ `*mut i32`. Karena kita membikin mereka secara 
langsung dari sebuah variabel lokal, kita tahu pasti kalau _raw pointers_ spesifik ini 
itu valid, tapi kita tidak bisa bikin asumsi kayak gitu buat sembarang _raw pointer_ 
yang mana aja.

Buat mendemonstrasikan hal ini, selanjutnya kita bakal membikin sebuah _raw pointer_ yang 
mana validitasnya tidak bisa kita pastikan, dengan memakai keyword `as` buat meng-_cast_ 
(mengubah tipe) sebuah nilai ketimbang memakai operator *raw borrow*. Listing 20-2 
menunjukkan gimana cara membikin sebuah _raw pointer_ ke sebuah lokasi sembarang 
(arbitrary location) di memori. Mencoba buat memakai memori sembarang itu adalah 
perilaku yang tidak terdefinisi (undefined behavior): mungkin ada data di alamat 
tersebut atau mungkin tidak ada, _compiler_ mungkin bakal mengoptimasi kodenya 
sehingga tidak ada akses memori yang terjadi, atau programnya mungkin bakal berhenti 
(terminate) karena *segmentation fault*. Biasanya, tidak ada alasan yang bagus buat 
menulis kode kayak gini, apalagi di kasus-kasus di mana kita bisa memakai operator 
*raw borrow* sebagai gantinya, tapi hal ini tetap memungkinkan buat dilakukan.

<Listing number="20-2" caption="Membikin sebuah raw pointer ke alamat memori sembarang">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Ingat kembali kalau kita bisa membikin _raw pointers_ di _safe code_, tapi kita tidak 
bisa _men-dereferensi_ _raw pointers_ dan membaca data yang ditunjuknya. Di Listing 20-3, 
kita memakai operator dereferensi `*` pada sebuah _raw pointer_ yang mana mewajibkan 
sebuah blok `unsafe`.

<Listing number="20-3" caption="Men-dereferensi raw pointers di dalam sebuah blok `unsafe`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Membikin sebuah pointer itu tidak membahayakan apa-apa; cuma pas kita mencoba buat 
mengakses nilai yang ditunjuknya barulah kita berisiko berurusan dengan sebuah 
nilai yang tidak valid.

Perhatikan juga kalau di Listing 20-1 dan 20-3, kita membikin _raw pointers_ `*const i32` 
dan `*mut i32` yang dua-duanya menunjuk ke lokasi memori yang sama, tempat di mana 
`num` disimpan. Kalau kita nyoba buat membikin referensi _immutable_ dan _mutable_ 
ke `num` sebagai gantinya, kodenya tidak bakal bisa di-compile karena aturan _ownership_ 
Rust tidak mengizinkan adanya referensi _mutable_ di saat yang bersamaan dengan 
referensi _immutable_ apa pun. Dengan _raw pointers_, kita bisa membikin pointer 
_mutable_ dan pointer _immutable_ ke lokasi yang sama dan mengubah datanya lewat 
pointer _mutable_ tersebut, yang secara potensial bisa ngebikin sebuah *data race*. 
Hati-hati ya!

Dengan semua bahaya ini, kenapa kita mau repot-repot memakai _raw pointers_? 
Salah satu skenario penggunaan (use case) utamanya adalah saat berinteraksi 
sama kode C, kayak yang bakal kita lihat di bagian selanjutnya. Skenario 
lainnya adalah pas kita lagi ngebangun abstraksi yang aman yang mana si 
_borrow checker_ tidak bisa pahami. Kita bakal mengenalkan fungsi-fungsi 
_unsafe_ lalu melihat sebuah contoh abstraksi aman yang memakai kode _unsafe_.

### Memanggil Fungsi atau Method Unsafe

Jenis operasi kedua yang bisa kita lakukan di dalam blok _unsafe_ adalah memanggil 
fungsi-fungsi _unsafe_. Fungsi dan method _unsafe_ kelihatannya persis kayak 
fungsi dan method biasa, tapi mereka punya tambahan `unsafe` sebelum sisa 
definisinya. Keyword `unsafe` di konteks ini mengindikasikan bahwa fungsi 
tersebut punya persyaratan yang harus kita junjung tinggi pas kita manggil fungsi 
ini, karena Rust tidak bisa ngejamin kalau kita udah menuhi persyaratan 
tersebut. Dengan memanggil fungsi _unsafe_ di dalam blok `unsafe`, kita menyatakan 
kalau kita udah ngebaca dokumentasi fungsi ini dan kita mengambil tanggung jawab 
buat mematuhi kontrak dari fungsi tersebut.

Berikut ini adalah fungsi _unsafe_ bernama `dangerous` yang tidak melakukan apa-apa 
di dalam isinya (body):

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Kita wajib memanggil fungsi `dangerous` di dalam sebuah blok `unsafe` yang terpisah. 
Kalau kita mencoba buat memanggil `dangerous` tanpa blok `unsafe`, kita bakal dapat error:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Dengan blok `unsafe`, kita lagi menegaskan ke Rust kalau kita udah membaca dokumentasi 
dari fungsinya, kita ngerti gimana cara memakainya dengan benar, dan kita udah 
memverifikasi kalau kita mematuhi kontrak dari fungsi tersebut.

Buat melakukan operasi _unsafe_ di dalam isi dari sebuah fungsi `unsafe`, kita tetap 
butuh memakai sebuah blok `unsafe`, sama seperti di dalam fungsi biasa, dan _compiler_ 
bakal ngingetin (warn) kita kalau kita lupa. Ini ngebantu kita ngejaga supaya blok 
`unsafe` itu sekecil mungkin, karena operasi _unsafe_ mungkin tidak diperlukan 
di seluruh isi fungsi tersebut.

#### Membikin Abstraksi yang Aman di atas Kode Unsafe

Cuma karena sebuah fungsi mengandung kode _unsafe_ tidak berarti kita harus menandai 
keseluruhan fungsinya sebagai _unsafe_. Faktanya, membungkus kode _unsafe_ di dalam 
sebuah fungsi yang aman adalah sebuah abstraksi yang sangat umum. Sebagai contoh, mari 
kita pelajari fungsi `split_at_mut` dari _standard library_, yang mana mewajibkan 
beberapa kode _unsafe_. Kita bakal mengeksplorasi gimana kita mungkin 
mengimplementasikannya. Method aman ini didefinisikan pada slices _mutable_: dia 
mengambil satu _slice_ lalu membikinnya jadi dua dengan membelah _slice_ tersebut di 
indeks yang diberikan sebagai argumen. Listing 20-4 menunjukkan gimana cara 
memakai `split_at_mut`.

<Listing number="20-4" caption="Memakai fungsi aman `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Kita tidak bisa mengimplementasikan fungsi ini kalau cuma memakai Rust yang aman 
saja. Percobaannya mungkin bakal kelihatan kayak yang ada di Listing 20-5, 
yang mana tidak bakal bisa di-compile. Demi kesederhanaan, kita bakal 
mengimplementasikan `split_at_mut` sebagai sebuah fungsi bukannya _method_ 
dan cuma buat slices yang berisi nilai `i32` bukannya buat sebuah tipe generik `T`.

<Listing number="20-5" caption="Sebuah percobaan implementasi `split_at_mut` yang cuma memakai Rust yang aman">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Fungsi ini pertama-tama mengambil panjang (length) total dari _slice_ tersebut. 
Terus dia menegaskan (asserts) kalau indeks yang diberikan sebagai parameter itu 
ada di dalam batas-batas _slice_ dengan mengecek apakah indeks tersebut kurang dari 
atau sama dengan panjangnya. Penegasan ini berarti kalau kita ngasih indeks yang lebih 
besar dari panjang _slice_ tersebut buat ngebelah si _slice_, fungsinya bakal _panic_ 
sebelum dia mencoba buat memakai indeks tersebut.

Lalu kita mengembalikan dua slices _mutable_ di dalam sebuah _tuple_: yang satu 
dari awal (start) _slice_ aslinya sampai ke indeks `mid`, dan satu lagi dari `mid` 
sampai ke akhir dari _slice_ tersebut.

Pas kita mencoba buat men-compile kode di Listing 20-5, kita bakal dapat sebuah error:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

_Borrow checker_ Rust tidak bisa ngerti kalau kita lagi meminjam (borrowing) bagian 
yang berbeda dari _slice_ tersebut; dia cuma tahu kalau kita lagi meminjam dari _slice_ 
yang sama sebanyak dua kali. Meminjam bagian yang berbeda dari sebuah _slice_ itu 
secara fundamental baik-baik aja karena kedua slices tersebut tidak saling tumpang tindih 
(overlapping), tapi Rust tidak cukup pintar buat tahu hal ini. Pas kita tahu kalau 
kodenya itu aman, tapi Rust tidak tahu, inilah saatnya buat ngambil kode _unsafe_.

Listing 20-6 nunjukin gimana caranya memakai sebuah blok `unsafe`, sebuah _raw pointer_, 
dan beberapa pemanggilan ke fungsi _unsafe_ buat ngebikin implementasi dari 
`split_at_mut` ini jalan.

<Listing number="20-6" caption="Memakai kode unsafe di dalam implementasi fungsi `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Ingat kembali dari [“Tipe Slice”][the-slice-type] di Bab 4 kalau sebuah _slice_ itu 
adalah pointer ke suatu data beserta panjang dari _slice_ tersebut. Kita memakai 
method `len` buat dapat panjang dari sebuah _slice_ dan method `as_mut_ptr` buat ngakses 
_raw pointer_ dari sebuah _slice_. Di kasus ini, karena kita punya _slice_ _mutable_ ke 
nilai `i32`, `as_mut_ptr` mengembalikan sebuah _raw pointer_ dengan tipe `*mut i32`, 
yang udah kita simpan di dalam variabel `ptr`.

Kita tetap naruh penegasan kalau indeks `mid` itu ada di dalam _slice_. Terus kita 
masuk ke kode _unsafe_-nya: fungsi `slice::from_raw_parts_mut` menerima sebuah 
_raw pointer_ dan sebuah panjang, terus dia ngebikin sebuah _slice_. Kita 
memakai fungsi ini buat ngebikin sebuah _slice_ yang dimulai dari `ptr` dan 
panjangnya sebesar `mid` item. Terus kita panggil method `add` pada `ptr` dengan 
`mid` sebagai argumen buat dapetin _raw pointer_ yang mulai di posisi `mid`, 
dan kita ngebikin sebuah _slice_ memakai pointer itu dan sisa jumlah item setelah 
`mid` sebagai panjangnya.

Fungsi `slice::from_raw_parts_mut` itu sifatnya _unsafe_ karena dia menerima sebuah 
_raw pointer_ dan harus percaya kalau pointer ini benar-benar valid. Method `add` 
pada _raw pointers_ juga sifatnya _unsafe_ karena dia harus percaya kalau lokasi 
*offset*-nya itu juga merupakan pointer yang valid. Oleh karena itu, kita harus menaruh 
blok `unsafe` di sekeliling pemanggilan ke `slice::from_raw_parts_mut` dan `add` 
supaya kita bisa memanggil mereka. Dengan ngelihat kodenya dan dengan menambahkan 
penegasan kalau `mid` itu harus kurang dari atau sama dengan `len`, kita bisa 
tahu pasti kalau semua _raw pointers_ yang dipakai di dalam blok `unsafe` tersebut 
bakal jadi pointers yang valid yang menunjuk ke data di dalam _slice_ tersebut. Ini 
adalah penggunaan dari `unsafe` yang bisa diterima dan sangat tepat.

Perhatikan bahwa kita tidak perlu menandai fungsi `split_at_mut` hasilnya sebagai 
`unsafe`, dan kita bisa memanggil fungsi ini dari _safe code_. Kita udah ngebikin 
sebuah abstraksi yang aman ke dalam kode _unsafe_ tersebut dengan sebuah implementasi 
fungsi yang memakai kode `unsafe` dengan cara yang aman, karena dia cuma 
membikin pointers yang valid dari data yang emang bisa diakses sama fungsi ini.

Sebaliknya, pemakaian `slice::from_raw_parts_mut` di Listing 20-7 kemungkinan 
besar bakal _crash_ (rusak) pas _slice_-nya dipakai. Kode ini mengambil sebuah lokasi 
memori sembarang lalu membikin sebuah _slice_ yang panjangnya 10.000 item.

<Listing number="20-7" caption="Membikin sebuah _slice_ dari lokasi memori sembarang">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Kita tidak memiliki memori di lokasi sembarang ini, dan tidak ada jaminan kalau 
_slice_ yang dibikin sama kode ini benar-benar berisi nilai `i32` yang valid. Mencoba buat 
memakai `values` seolah-olah dia adalah _slice_ yang valid bakal menghasilkan perilaku 
yang tidak terdefinisi (undefined behavior).

#### Memakai Fungsi `extern` Buat Memanggil Kode Eksternal

Kadang-kadang kode Rust kita mungkin perlu berinteraksi sama kode yang ditulis di bahasa 
pemrograman lain. Buat keperluan ini, Rust punya keyword `extern` yang memfasilitasi 
pembuatan dan penggunaan _Foreign Function Interface (FFI)_, yang mana adalah sebuah 
cara bagi sebuah bahasa pemrograman buat mendefinisikan fungsi lalu memungkinkan 
bahasa pemrograman (asing) lain buat memanggil fungsi-fungsi tersebut.

Listing 20-8 mendemonstrasikan gimana caranya nge-*setup* sebuah integrasi dengan fungsi 
`abs` dari _standard library_ C. Fungsi-fungsi yang dideklarasikan di dalam 
blok `extern` itu umumnya tidak aman (unsafe) buat dipanggil dari kode Rust, jadi blok 
`extern` tersebut juga harus ditandai sebagai `unsafe`. Alasannya adalah karena bahasa-
bahasa lain tidak menerapkan (enforce) aturan-aturan dan jaminan-jaminan yang dipunyai Rust, 
dan Rust tidak bisa ngecek mereka, jadi tanggung jawabnya jatuh ke tangan si programmer buat 
memastikan keamanan kodenya.

<Listing number="20-8" file-name="src/main.rs" caption="Mendeklarasikan dan memanggil sebuah fungsi `extern` yang didefinisikan di bahasa pemrograman lain">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

Di dalam blok `unsafe extern "C"`, kita mendaftarkan (list) nama-nama dan _signatures_ 
dari fungsi-fungsi eksternal dari bahasa lain yang mau kita panggil. Bagian `"C"` itu 
mendefinisikan _application binary interface (ABI)_ mana yang dipakai sama fungsi eksternal 
tersebut: ABI ini mendefinisikan gimana caranya memanggil fungsinya pada tingkat bahasa rakitan 
(assembly level). ABI `"C"` adalah yang paling umum dipakai dan mengikuti standar 
ABI dari bahasa pemrograman C. Informasi tentang semua ABI yang didukung oleh Rust ada 
di [Rust Reference][ABI].

Semua item yang dideklarasikan di dalam blok `unsafe extern` itu secara implisit sifatnya 
_unsafe_. Namun, beberapa fungsi FFI *memang* aman buat dipanggil. Contohnya, fungsi 
`abs` dari _standard library_ C itu tidak punya pertimbangan apa pun soal keamanan memori dan 
kita tahu dia bisa dipanggil pakai sembarang `i32`. Di kasus-kasus kayak gini, kita bisa 
memakai keyword `safe` buat ngasih tahu kalau fungsi yang spesifik ini itu aman buat 
dipanggil biarpun dia ada di dalam sebuah blok `unsafe extern`. Begitu kita bikin perubahan 
tersebut, manggil fungsinya udah tidak perlu blok `unsafe` lagi, kayak yang ditunjukin 
di Listing 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="Secara eksplisit menandai sebuah fungsi sebagai `safe` di dalam sebuah blok `unsafe extern` dan memanggilnya dengan aman">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Menandai sebuah fungsi sebagai `safe` tidak secara otomatis (inherently) membikin fungsinya 
jadi aman ya! Sebaliknya, ini ibarat janji yang kita buat ke Rust kalau dia itu aman. 
Itu tetap jadi tanggung jawab kita buat mastiin kalau janji itu ditepati!

#### Memanggil Fungsi Rust dari Bahasa Lain

Kita juga bisa memakai `extern` buat membikin _interface_ yang memungkinkan bahasa 
pemrograman lain buat memanggil fungsi Rust. Ketimbang membikin satu blok `extern` utuh, kita 
menambahkan keyword `extern` dan menentukan ABI apa yang mau dipakai tepat sebelum 
keyword `fn` buat fungsi yang relevan. Kita juga perlu nambahin anotasi 
`#[unsafe(no_mangle)]` buat ngasih tahu _compiler_ Rust supaya tidak nge-_mangle_ nama dari 
fungsi ini. _Mangling_ adalah pas _compiler_ ngubah nama yang udah kita kasih ke sebuah 
fungsi jadi nama lain yang mengandung lebih banyak informasi biar bisa dikonsumsi sama 
bagian-bagian lain dari proses kompilasi tapi jadinya kurang enak dibaca sama manusia. 
Setiap _compiler_ bahasa pemrograman nge-_mangle_ nama dengan cara yang agak berbeda-beda, 
jadi supaya sebuah fungsi Rust bisa dipanggil namanya sama bahasa lain, kita harus 
mematikan fitur _name mangling_ dari _compiler_ Rust. Hal ini sifatnya _unsafe_ karena bisa 
aja terjadi bentrok nama (name collisions) antar _libraries_ kalau _mangling_ bawaan ini 
dimatiin, jadi ini adalah tanggung jawab kita buat mastiin kalau nama yang kita pilih 
itu aman buat diekspor tanpa di-_mangle_.

Di contoh berikut ini, kita membikin fungsi `call_from_c` supaya bisa diakses dari 
kode C, setelah dia di-compile jadi _shared library_ dan di-_link_ dari C:

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

Penggunaan `extern` ini cuma mewajibkan adanya `unsafe` di dalam atributnya aja, bukan 
di blok `extern`-nya.

### Mengakses atau Memodifikasi Variabel Static yang Mutable

Di buku ini, kita belum pernah ngebahas soal variabel global (_global variables_), yang mana 
memang didukung oleh Rust tapi bisa jadi bermasalah kalau digabungkan sama aturan _ownership_ 
Rust. Kalau ada dua _threads_ yang lagi mengakses variabel global yang _mutable_ yang sama, 
hal itu bisa nyebabin sebuah *data race*.

Di Rust, variabel global itu disebut sebagai variabel _static_ (statis). Listing 20-10 
menunjukkan contoh deklarasi dan penggunaan dari sebuah variabel statis dengan nilai 
berupa sebuah _string slice_.

<Listing number="20-10" file-name="src/main.rs" caption="Mendefinisikan dan memakai sebuah variabel statis yang immutable">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Variabel _static_ itu mirip sama konstanta (constants), yang mana udah kita bahas di 
[“Konstanta”][differences-between-variables-and-constants] di Bab 3. Nama-nama buat 
variabel statis itu ditulis pakai gaya `SCREAMING_SNAKE_CASE` secara konvensi. Variabel 
statis cuma bisa menyimpan referensi dengan _lifetime_ `'static`, yang berarti _compiler_ 
Rust udah tahu soal _lifetime_-nya dan kita tidak diwajibkan buat menganotasinya 
secara eksplisit. Mengakses sebuah variabel statis yang _immutable_ itu aman.

Satu perbedaan yang cukup halus (subtle) antara konstanta dan variabel statis _immutable_ 
adalah nilai-nilai yang ada di dalam sebuah variabel statis punya alamat yang tetap di 
memori. Memakai nilai tersebut bakal selalu mengakses data yang sama. Konstanta, di sisi lain, 
diizinkan buat menduplikasi data mereka kapan pun mereka dipakai. Perbedaan lainnya adalah 
variabel statis itu bisa bersifat _mutable_ (bisa diubah). Mengakses dan memodifikasi 
variabel statis _mutable_ itu sifatnya _unsafe_. Listing 20-11 menunjukkan gimana 
caranya mendeklarasikan, mengakses, dan memodifikasi sebuah variabel statis _mutable_ 
bernama `COUNTER`.

<Listing number="20-11" file-name="src/main.rs" caption="Ngebaca atau nulis ke sebuah variabel statis mutable itu unsafe.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Sama kayak variabel biasa, kita menentukan mutabilitasnya dengan memakai keyword `mut`. 
Kode apa pun yang membaca atau menulis dari `COUNTER` wajib berada di dalam sebuah 
blok `unsafe`. Kode di Listing 20-11 berhasil di-compile dan mencetak `COUNTER: 3` kayak 
yang kita harapkan karena ini adalah program *single-threaded* (utas tunggal). Kalau 
sampai ada banyak _threads_ yang mengakses `COUNTER`, itu kemungkinan besar bakal nyebabin 
*data races*, jadi itu dianggap sebagai perilaku yang tidak terdefinisi. Oleh karena itu, 
kita perlu menandai keseluruhan fungsinya sebagai `unsafe` dan mendokumentasikan batasan 
keamanannya, supaya siapa pun yang memanggil fungsi ini tahu apa aja yang boleh dan tidak 
boleh mereka lakuin dengan aman.

Kapan pun kita nulis sebuah fungsi yang _unsafe_, sangatlah idiomatik buat nulis sebuah 
komentar yang diawali dengan kata `SAFETY` dan ngejelasin apa aja yang perlu dilakuin 
sama si pemanggil fungsi buat memanggil fungsi tersebut dengan aman. Sama halnya, kapan pun 
kita ngelakuin operasi _unsafe_, juga sangat idiomatik buat nulis sebuah komentar yang 
diawali dengan `SAFETY` buat ngejelasin gimana aturan-aturan keamanan tersebut dijunjung 
tinggi.

Selain itu, _compiler_ secara default bakal melarang usaha apa pun buat ngebikin referensi ke 
sebuah variabel statis yang _mutable_ lewat mekanisme _lint_ (peringatan kode) _compiler_. 
Kita wajib secara eksplisit keluar (opt-out) dari perlindungan _lint_ tersebut dengan 
nambahin anotasi `#[allow(static_mut_refs)]` atau ngakses variabel statis _mutable_ tersebut 
lewat sebuah _raw pointer_ yang dibikin pakai salah satu dari operator *raw borrow*. Hal ini 
termasuk juga kasus-kasus di mana referensi tersebut dibikin secara kasatmata, kayak pas dia 
dipakai di `println!` di dalam kode ini. Mewajibkan supaya referensi ke variabel statis 
yang _mutable_ harus dibikin lewat _raw pointers_ ngebantu ngebikin persyaratan 
keamanannya jadi lebih terlihat jelas saat kita memakai mereka.

Dengan data _mutable_ yang bisa diakses secara global, itu susah sekali buat memastikan 
kalau tidak bakal ada *data races*, yang mana inilah alasan kenapa Rust menganggap variabel 
statis _mutable_ itu sebagai _unsafe_. Kalau memungkinkan, jauh lebih baik buat memakai 
teknik-teknik konkurensi dan _smart pointers_ yang *thread-safe* yang udah kita bahas 
di Bab 16 supaya _compiler_ bisa ngecek kalau akses data dari berbagai _threads_ yang 
berbeda dilakukan dengan aman.

### Mengimplementasikan Sebuah Unsafe Trait

Kita bisa memakai `unsafe` buat mengimplementasikan sebuah trait _unsafe_. Sebuah trait 
itu dianggap _unsafe_ saat minimal salah satu dari method-methodnya punya aturan mutlak 
(invariant) yang mana tidak bisa diverifikasi sama _compiler_. Kita mendeklarasikan kalau 
sebuah trait itu `unsafe` dengan menambahkan keyword `unsafe` sebelum keyword `trait` dan 
menandai implementasi dari trait tersebut sebagai `unsafe` juga, seperti yang ditunjukin 
di Listing 20-12.

<Listing number="20-12" caption="Mendefinisikan dan mengimplementasikan sebuah unsafe trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

Dengan memakai `unsafe impl`, kita lagi berjanji kalau kita bakal menjunjung tinggi 
aturan-aturan mutlak (invariants) yang tidak bisa diverifikasi sama _compiler_.

Sebagai contoh, ingat kembali _marker traits_ `Send` dan `Sync` yang kita bahas di 
[“Konkurensi yang Bisa Diperluas dengan Trait `Send` dan `Sync`”][extensible-concurrency-with-the-send-and-sync-traits] 
di Bab 16: _compiler_ mengimplementasikan trait-trait ini secara otomatis kalau tipe-tipe 
kita sepenuhnya disusun dari tipe-tipe lain yang udah mengimplementasikan `Send` dan 
`Sync`. Kalau kita ngimplementasiin sebuah tipe yang mengandung sebuah tipe yang tidak 
mengimplementasikan `Send` atau `Sync`, contohnya _raw pointers_, dan kita mau menandai 
tipe tersebut sebagai `Send` atau `Sync`, kita wajib memakai `unsafe`. Rust tidak bisa 
memverifikasi kalau tipe kita itu mematuhi jaminan bahwa dia bisa dikirim ke _thread_ 
lain dengan aman atau diakses dari banyak _threads_ dengan aman; makanya, kita perlu 
ngelakuin pengecekan itu sendiri secara manual dan ngasih tahu hal itu lewat keyword `unsafe`.

### Mengakses Field dari Union

Tindakan (action) terakhir yang cuma bisa bekerja dengan keyword `unsafe` adalah 
mengakses *field* (bidang) dari sebuah `union`. Sebuah *union* (gabungan) itu mirip sama 
sebuah `struct`, tapi cuma satu aja dari *field* yang dideklarasikan yang bisa dipakai 
di dalam sebuah instance pada satu waktu tertentu. *Unions* biasanya dipakai buat 
berinteraksi dengan *unions* yang ada di dalam kode C. Mengakses *field* dari *union* 
itu _unsafe_ karena Rust tidak bisa ngejamin tipe dari data yang saat itu lagi 
disimpan di dalam instance *union* tersebut. Kita bisa belajar lebih banyak soal *unions* 
di [Rust Reference][unions].

### Memakai Miri Buat Mengecek Kode Unsafe

Pas kita lagi nulis kode _unsafe_, kita mungkin pengen ngecek apakah kode yang udah 
kita tulis itu benar-benar aman dan udah betul (correct) atau tidak. Salah satu cara 
terbaik buat ngebuktiin itu adalah dengan memakai Miri, sebuah _tool_ resmi Rust buat 
mendeteksi perilaku yang tidak terdefinisi (undefined behavior). Kalau _borrow checker_ 
itu adalah sebuah _tool_ yang statis (_static_) yang bekerja pas _compile time_, Miri itu 
adalah _tool_ yang dinamis (_dynamic_) yang bekerja pas _runtime_. Miri mengecek kode 
kita dengan cara menjalankan program kita, atau seperangkat pengujiannya (_test suite_), 
lalu mendeteksi kapan kita melanggar aturan-aturan yang Miri pahami tentang gimana 
seharusnya Rust bekerja.

Memakai Miri mewajibkan kita punya instalasi Rust versi _nightly_ (yang mana kita obrolin 
lebih banyak di [Lampiran G: Gimana Rust Dibuat dan “Nightly Rust”][nightly]). Kita bisa 
menginstal baik Rust versi _nightly_ sekaligus _tool_ Miri dengan mengetikkan perintah 
`rustup +nightly component add miri`. Hal ini tidak bakal ngubah versi Rust apa yang 
lagi dipakai sama project kita; dia cuma nambahin _tool_ tersebut ke sistem kita biar 
kita bisa memakainya pas kita mau aja. Kita bisa menjalankan Miri di sebuah project dengan 
ngetik `cargo +nightly miri run` atau `cargo +nightly miri test`.

Sebagai contoh betapa bergunanya ini, bayangin apa yang terjadi pas kita menjalankan 
Miri buat nge-tes kode di Listing 20-7.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri ngingetin kita dengan tepat kalau kita lagi nge-_cast_ (mengubah tipe) sebuah integer 
jadi sebuah pointer, yang mana mungkin aja jadi masalah tapi Miri tidak bisa mendeteksi 
apakah emang ada masalah atau tidak karena Miri tidak tahu dari mana asal-usul si 
pointer tersebut. Terus, Miri ngembaliin sebuah error di mana Listing 20-7 punya perilaku 
yang tidak terdefinisi (undefined behavior) karena kita punya sebuah _dangling pointer_. 
Berkat Miri, sekarang kita jadi tahu kalau ada risiko terjadinya perilaku yang tidak 
terdefinisi, dan kita bisa mikirin gimana caranya ngebikin kodenya jadi aman. Di beberapa 
kasus, Miri bahkan bisa ngasih saran gimana cara buat ngeberesin error-error tersebut.

Miri tidak bisa menangkap semua hal yang mungkin keliru pas kita nulis kode _unsafe_. 
Miri itu adalah sebuah _tool_ analisis yang dinamis, jadi dia cuma bisa nangkap 
masalah pada kode yang emang benar-benar dijalankan. Itu artinya kita perlu memakainya 
barengan sama teknik-teknik pengujian (testing techniques) yang oke buat ningkatin rasa 
percaya diri kita terhadap kode _unsafe_ yang udah kita tulis. Miri juga tidak 
mencakup setiap kemungkinan yang ada yang bikin kode kita jadi cacat (unsound).

Dengan kata lain: Kalau Miri *emang* nangkap sebuah masalah, kita jadi tahu kalau ada 
sebuah _bug_, tapi cuma karena Miri *tidak* nangkap sebuah _bug_ itu tidak berarti kalau 
tidak ada masalah di situ. Walaupun begitu, dia bisa nangkap sangat banyak lho. Cobain 
deh ngejalanin Miri di contoh-contoh kode _unsafe_ lain di bab ini dan lihat apa aja 
yang dia bilang!

Kita bisa belajar lebih lanjut soal Miri di [repositori GitHub-nya][miri].

### Kapan Harus Memakai Kode Unsafe

Memakai `unsafe` buat melakukan salah satu dari lima _superpowers_ yang baru aja kita bahas 
itu bukanlah hal yang salah atau yang dilarang keras, tapi emang lebih _tricky_ (susah) buat 
bikin kode `unsafe` itu jadi benar (correct) karena _compiler_ tidak bisa ngebantu buat 
menjunjung tinggi keamanan memori. Pas kita punya alasan buat memakai kode `unsafe`, 
kita bebas aja buat melakukannya, dan dengan adanya anotasi `unsafe` yang eksplisit itu malah 
ngebikin kita lebih gampang buat ngelacak sumber dari suatu masalah kalau sampai 
masalah itu muncul nanti. Kapan pun kita nulis kode _unsafe_, kita bisa memakai Miri buat 
ngebantu kita jadi lebih yakin kalau kode yang udah kita tulis itu benar-benar mematuhi 
aturan-aturan Rust.

Buat penjelajahan yang jauh lebih dalam soal gimana cara kerja efektif dengan _unsafe_ Rust, 
silakan baca panduan resmi dari Rust soal subjek ini, yaitu [Rustonomicon][nomicon].

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[differences-between-variables-and-constants]: ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-send-and-sync-traits]: ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-send-and-sync-traits
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
