## Mengembangkan Fungsionalitas Library dengan Test-Driven Development

Sekarang setelah logika pencarian kita terpisah di _src/lib.rs_ dari fungsi 
`main`, jauh lebih mudah untuk menulis pengujian buat fungsionalitas inti dari 
kode kita. Kita bisa memanggil fungsi secara langsung dengan berbagai argumen 
dan mengecek nilai kembaliannya tanpa perlu menjalankan *binary* kita dari 
*command line*.

Di bagian ini, kita bakal menambahkan logika pencarian ke program `minigrep` 
memakai proses *test-driven development* (TDD) dengan langkah-langkah berikut:

1. Tulis sebuah pengujian yang gagal lalu jalankan buat memastikan kalau pengujian 
   itu gagal dengan alasan yang kita harapkan.
2. Tulis atau ubah kode secukupnya saja buat bikin pengujian baru itu sukses.
3. *Refactor* (rombak) kode yang baru saja ditambahkan atau diubah dan pastikan 
   pengujiannya tetap sukses.
4. Ulangi lagi dari langkah 1!

Meskipun ini cuma salah satu dari sekian banyak cara buat nulis *software*, 
TDD bisa membantu mengarahkan desain kode. Menulis pengujian sebelum kita 
menulis kode yang bakal membuat pengujian itu sukses membantu mempertahankan 
*test coverage* (cakupan pengujian) yang tinggi di sepanjang proses.

Kita bakal menguji-kembangkan (test-drive) implementasi fungsionalitas yang 
nantinya bakal benar-benar melakukan pencarian _string_ kueri di dalam isi file 
lalu menghasilkan daftar baris yang cocok dengan kueri tersebut. Kita bakal 
menambahkan fungsionalitas ini ke dalam sebuah fungsi bernama `search`.

### Menulis Pengujian yang Gagal

Di _src/lib.rs_, kita bakal menambahkan modul `tests` dengan fungsi pengujiannya, 
seperti yang kita lakukan di [Bab 11][ch11-anatomy]. Fungsi pengujian ini 
menentukan perilaku yang kita inginkan dari fungsi `search`: fungsi ini bakal 
menerima sebuah kueri dan teks tempat kita mencari, lalu ia cuma bakal 
mengembalikan baris-baris dari teks tersebut yang mengandung kuerinya. Listing 
12-15 menunjukkan pengujian ini.

<Listing number="12-15" file-name="src/lib.rs" caption="Membuat pengujian yang gagal buat fungsi `search` untuk fungsionalitas yang kita harapkan ada">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Pengujian ini mencari string `"duct"`. Teks yang lagi kita cari ada tiga baris, 
dan cuma satu yang mengandung `"duct"` (perhatikan bahwa tanda *backslash* 
setelah tanda kutip ganda pembuka memberi tahu Rust buat tidak menambahkan 
karakter *newline* alias baris baru di awal konten string literal ini). Kita 
menegaskan bahwa nilai yang dikembalikan oleh fungsi `search` cuma berisi 
baris yang kita harapkan.

Kalau kita menjalankan pengujian ini sekarang, ia bakal gagal karena macro 
`unimplemented!` mengalami _panic_ dengan pesan “not implemented”. Mengikuti 
prinsip-prinsip TDD, kita bakal mengambil langkah kecil dengan menambahkan kode 
secukupnya agar pengujian ini tidak _panic_ saat memanggil fungsi tersebut; kita 
lakukan dengan mendefinisikan fungsi `search` agar selalu mengembalikan vector 
kosong, seperti yang ditunjukkan di Listing 12-16. Kemudian pengujian ini 
seharusnya berhasil di-compile dan lalu gagal karena vector kosong tidak cocok 
dengan vector yang isinya baris `"safe, fast, productive."`

<Listing number="12-16" file-name="src/lib.rs" caption="Mendefinisikan fungsi `search` secukupnya saja sehingga ia tidak _panic_ saat dipanggil">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Sekarang mari kita bahas kenapa kita perlu mendefinisikan _lifetime_ `'a` secara 
eksplisit di *signature* fungsi `search` dan memakai _lifetime_ itu bersama 
argumen `contents` dan nilai kembaliannya. Ingat kembali di [Bab 10][ch10-lifetimes] 
bahwa parameter _lifetime_ menentukan *lifetime* argumen mana yang terhubung 
dengan *lifetime* nilai kembaliannya. Di kasus ini, kita mengindikasikan kalau 
vector yang dikembalikan seharusnya berisi _string slices_ yang merujuk pada 
_slices_ dari argumen `contents` (bukannya dari argumen `query`).

Dengan kata lain, kita memberi tahu Rust kalau data yang dikembalikan oleh fungsi 
`search` bakal hidup (live) selama data yang diteruskan ke dalam fungsi `search` 
lewat argumen `contents`. Ini penting! Data yang dirujuk _oleh_ sebuah _slice_ 
harus tetap valid agar referensinya juga ikut valid; kalau _compiler_ 
mengasumsikan kalau kita lagi bikin _string slices_ dari `query` bukannya 
`contents`, pengecekan keamanannya bakal dilakukan dengan tidak tepat.

Kalau kita kelupaan menganotasi _lifetime_ dan mencoba men-compile fungsi ini, 
kita bakal dapat error ini:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust tidak bisa tahu pasti dari kedua parameter ini mana yang kita butuhkan buat 
outputnya, jadi kita harus memberitahukannya secara eksplisit. Perhatikan bahwa 
teks bantuannya menyarankan buat menentukan parameter _lifetime_ yang sama buat 
semua parameter beserta tipe outputnya, yang mana itu salah! Karena `contents` 
adalah parameter yang berisi semua teks kita dan kita mau mengembalikan 
bagian-bagian dari teks itu yang cocok, kita jadi tahu kalau cuma parameter 
`contents` yang seharusnya dihubungkan dengan nilai kembaliannya menggunakan 
sintaks _lifetime_.

Bahasa pemrograman lain biasanya tidak mengharuskan kita buat menghubungkan 
argumen dengan nilai kembalian di *signature*-nya, tapi praktik ini bakal 
terasa lebih mudah seiring berjalannya waktu. Kita mungkin mau membandingkan 
contoh ini dengan contoh-contoh di bagian [“Memvalidasi Referensi dengan 
Lifetimes”][validating-references-with-lifetimes] di Bab 10.

### Menulis Kode Agar Pengujian Sukses

Saat ini, pengujian kita gagal karena kita selalu mengembalikan vector kosong. 
Buat memperbaikinya dan mengimplementasikan `search`, program kita harus mengikuti 
langkah-langkah berikut:

1. Iterasi melewati setiap baris dari konten.
2. Mengecek apakah baris tersebut mengandung string kueri kita.
3. Jika iya, tambahkan baris itu ke daftar nilai yang bakal kita kembalikan.
4. Jika tidak, jangan lakukan apa-apa.
5. Kembalikan daftar hasil yang cocok.

Mari kerjakan satu per satu setiap langkahnya, dimulai dari iterasi melewati 
baris-baris.

#### Iterasi Melewati Baris dengan Method `lines`

Rust punya method yang membantu sekali buat menangani iterasi baris per baris 
dari strings, dinamai dengan nama yang nyaman yaitu `lines`, yang bekerja seperti 
yang ditunjukkan di Listing 12-17. Perhatikan kalau kode ini belum bisa di-compile.

<Listing number="12-17" file-name="src/lib.rs" caption="Iterasi melewati tiap baris di `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

Method `lines` mengembalikan sebuah iterator. Kita bakal membahas iterator lebih 
mendalam di [Bab 13][ch13-iterators], tapi ingat kembali bahwa kita pernah 
melihat cara memakai iterator seperti ini di [Listing 3-5][ch3-iter], di mana 
kita memakai *for loop* dengan sebuah iterator buat menjalankan beberapa kode 
pada setiap *item* di dalam sebuah _collection_ (koleksi).

#### Mencari Kueri di Setiap Baris

Berikutnya, kita bakal mengecek apakah baris saat ini mengandung string kueri 
kita. Untungnya, tipe string punya method pembantu (helper) bernama `contains` 
yang ngelakuin persis apa yang kita butuhkan! Tambahkan pemanggilan ke method 
`contains` di dalam fungsi `search`, seperti yang ditunjukkan di Listing 12-18. 
Perhatikan kalau kode ini masih belum bisa di-compile.

<Listing number="12-18" file-name="src/lib.rs" caption="Menambahkan fungsionalitas buat ngecek apakah barisnya mengandung string di `query`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Saat ini, kita baru membangun fungsionalitasnya. Supaya kodenya bisa di-compile, 
kita perlu mengembalikan sebuah nilai dari body fungsi sesuai indikasi yang 
kita berikan di *signature* fungsi.

#### Menyimpan Baris yang Cocok

Buat menyelesaikan fungsi ini, kita butuh cara buat menyimpan baris-baris yang 
cocok yang mau kita kembalikan. Buat melakukan itu, kita bisa membuat vector 
*mutable* sebelum *for loop* lalu memanggil method `push` buat menyimpan `line` 
ke dalam vector tersebut. Setelah *for loop* selesai, kita mengembalikan 
vector-nya, seperti yang ditunjukkan di Listing 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Menyimpan baris-baris yang cocok supaya kita bisa mengembalikannya">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Sekarang fungsi `search` seharusnya hanya mengembalikan baris-baris yang 
mengandung `query`, dan pengujian kita seharusnya sukses. Mari kita jalankan 
pengujiannya:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Pengujian kita sukses, jadi kita tahu kalau ini berfungsi!

Di titik ini, kita bisa mempertimbangkan peluang buat me-*refactor* implementasi 
dari fungsi pencarian sambil menjaga agar pengujiannya tetap sukses demi 
mempertahankan fungsionalitas yang sama. Kode di fungsi pencariannya tidak 
terlalu jelek sih, tapi dia tidak memanfaatkan beberapa fitur iterator yang 
sangat berguna. Kita bakal kembali lagi ke contoh ini di [Bab 13][ch13-iterators], 
di mana kita bakal mengeksplorasi iterator lebih dalam, dan melihat gimana cara 
meningkatkannya.

Sekarang keseluruhan programnya seharusnya sudah bisa jalan! Mari kita coba, 
pertama dengan kata yang seharusnya cuma mengembalikan satu baris persis dari 
puisi Emily Dickinson: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Keren! Sekarang mari kita coba pakai kata yang bakal cocok dengan beberapa baris 
sekaligus, misalnya _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Dan terakhir, mari kita pastikan kalau kita tidak dapat baris apa pun ketika 
kita mencari kata yang sama sekali tidak ada di dalam puisinya, misalnya 
_monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Luar biasa! Kita sudah membangun versi mini kita sendiri dari alat klasik dan 
belajar banyak hal soal cara menata aplikasi. Kita juga belajar sedikit tentang 
input dan output file, _lifetimes_, pengujian, dan penguraian *command line*.

Buat melengkapi project ini, kita bakal mendemonstrasikan secara singkat gimana 
cara berurusan dengan _environment variables_ (variabel lingkungan) dan gimana 
cara mencetak pesan ke *standard error*, yang mana keduanya sangat berguna pas 
kita lagi nulis program *command line*.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
