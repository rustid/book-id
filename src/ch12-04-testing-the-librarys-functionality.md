<!-- Old headings. Do not remove or links may break. -->

<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## Nambahin Fungsionalitas pakai Test-Driven Development

Sekarang karena kita udah mindahin logika pencarian ke _src/lib.rs_ dan misahin
dari fungsi `main`, ngetes core logic program jadi jauh lebih gampang. Kita bisa
langsung manggil fungsi-fungsi dengan berbagai argumen dan ngecek nilai baliknya
tanpa harus jalanin binary lewat command line.

Di bagian ini, kita bakal nambahin logika pencarian ke program `minigrep` pakai
proses test-driven development (TDD) dengan langkah-langkah kayak gini:

1. Tulis test yang gagal dan jalankan buat pastiin gagal karena alasan yang kita
   ekspektasikan.
2. Tulis atau ubah kode secukupnya aja supaya test baru itu lulus.
3. Refactor kode yang baru kamu tambahin atau ubah, dan pastiin test masih tetap
   lulus.
4. Ulangi lagi dari langkah 1!

Walaupun ini cuma salah satu cara nulis software, TDD bisa bantu ngarahin desain
kode. Nulis test sebelum nulis kode yang bikin testnya lulus juga bantu ngejaga
coverage test tetap tinggi selama proses development.

Kita bakal bikin implementasi fungsi yang beneran ngelakuin pencarian query di
isi file dan ngasilin daftar baris yang cocok sama query. Kita bakal nambahin
fungsionalitas ini di sebuah fungsi bernama `search`.

### Nulis Test yang Gagal Dulu

Di _src/lib.rs_, kita bakal nambahin module `tests` dengan sebuah fungsi test,
sama kayak yang udah kita lakukan di [Chapter 11][ch11-anatomy]<!-- ignore -->.
Fungsi test ini bakal ngejelasin perilaku yang kita pengen dari fungsi `search`:
dia bakal nerima query dan teks yang mau dicariin, lalu balikannya cuma baris
yang mengandung query itu aja. Listing 12-15 nunjukin testnya.

<Listing number="12-15" file-name="src/lib.rs" caption="Bikin test gagal dulu buat fungsi `search` sesuai fitur yang kita pengen">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Test ini nyari string `"duct"`. Teks yang kita cari terdiri dari tiga baris, dan
cuma satu baris yang ngandung `"duct"` (perhatikan backslash setelah tanda kutip
buka buat ngasih tau Rust supaya gak nambah karakter newline di awal string
literal ini). Kita ngasih assert kalau nilai balik dari fungsi `search` harus
cuma berisi baris yang kita ekspektasikan.

Kalau sekarang kita jalanin test ini, jelas bakal gagal karena macro
`unimplemented!` bakal panic dengan pesan “not implemented”. Sesuai prinsip TDD,
kita bakal ambil langkah kecil dulu: tambahin kode secukupnya supaya fungsi ini
gak panic lagi saat dipanggil, misalnya dengan bikin `search` selalu nge-return
vector kosong dulu, kayak di Listing 12-16. Setelah itu testnya bakal bisa
dikompilasi tapi gagal, karena vector kosong jelas gak sama dengan vector yang
isinya baris `"safe, fast, productive."`.

<Listing number="12-16" file-name="src/lib.rs" caption="Ngedefine fungsi `search` secukupnya dulu supaya gak panic waktu dipanggil">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Sekarang kita bahas kenapa kita perlu nentuin lifetime eksplisit `'a` di
signature fungsi `search` dan pakai lifetime itu buat argumen `contents` dan
nilai baliknya. Ingat lagi di [Chapter 10][ch10-lifetimes]<!-- ignore --> kalau
parameter lifetime itu nunjukin argumen mana yang lifetimenya nyambung ke return
value. Di kasus ini, kita ngasih tau kalau vector yang dikembalikan bakal berisi
string slice yang mereferensikan slice dari argumen `contents` (bukan dari
`query`).

Dengan kata lain, kita ngasih tau Rust kalau data yang dikembalikan fungsi
`search` bakal hidup selama data yang dikirim lewat argumen `contents` masih
valid. Ini penting banget! Data yang direferensikan sama slice harus valid
supaya referensinya valid. Kalau compiler malah nganggep kita bikin slice dari
`query` bukan dari `contents`, pengecekan keamanannya jadi salah.

Kalau kita lupa nulis lifetime ini dan langsung coba kompilasi, kita bakal dapet
error kayak gini:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust gak tau parameter mana yang lifetimenya harus dipakai buat output, jadi
kita harus jelasin eksplisit. Perhatikan juga kalau pesan bantuannya nyaranin
pakai lifetime yang sama buat semua parameter dan output, tapi itu salah! Karena
`contents` adalah parameter yang nyimpen semua teks yang kita cari, dan yang mau
kita balikin itu bagian dari teks tersebut, berarti cuma `contents` yang harus
dikaitin sama return value lewat syntax lifetime.

Bahasa pemrograman lain mungkin gak maksa kamu ngaitin argumen dengan return
value di signature kayak gini, tapi lama-lama kamu bakal kebiasa. Kamu mungkin
mau bandingin contoh ini sama bagian
[“Validating References with Lifetimes”][validating-references-with-lifetimes]<!-- ignore -->
di Chapter 10.

### Nulis Kode Biar Testnya Lulus

Sekarang test kita gagal karena kita selalu balikkin vector kosong. Buat benerin
dan ngelakuin implementasi `search`, program kita perlu ngelakuin
langkah-langkah ini:

1. Iterasi setiap baris dari `contents`.
2. Cek apakah baris tersebut mengandung query string.
3. Kalau iya, masukin barisnya ke list hasil yang mau kita return.
4. Kalau enggak, ya di-skip aja.
5. Balikin daftar hasil yang cocok.

Yuk kita kerjain step-step ini satu-satu, mulai dari looping tiap baris dulu.

#### Iterasi Baris Pakai Method `lines`

Rust punya method handy buat nge-handle iterasi string per baris, namanya
`lines`, dan cara kerjanya kayak di Listing 12-17. Tapi ini masih belum bisa
dikompilasi ya.

<Listing number="12-17" file-name="src/lib.rs" caption="Ngelakuin iterasi ke tiap baris di `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

Method `lines` bakal balikin iterator. Kita bakal bahas iterators lebih dalam di
[Chapter 13][ch13-iterators]<!-- ignore -->. Tapi inget, kamu udah pernah lihat
cara pake iterator kayak gini di [Listing 3-5][ch3-iter]<!-- ignore -->, waktu
kita pake loop `for` bareng iterator buat ngejalanin kode ke setiap item di
suatu koleksi.

#### Nyari Query di Setiap Baris

Selanjutnya kita bakal ngecek apakah baris saat ini mengandung query string
kita. Untungnya, string punya method berguna bernama `contains` buat bantu kita!
Tambahin aja pemanggilan `contains` ke fungsi `search`, kayak di Listing 12-18.
Tapi ini juga masih belum bisa dikompilasi.

<Listing number="12-18" file-name="src/lib.rs" caption="Nambahin fungsi pengecekan apakah baris mengandung string dari `query`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Sekarang kita lagi dalam proses ngebangun fungsionalitasnya pelan-pelan. Supaya
bisa dikompilasi, kita perlu balikin nilai sesuai yang udah kita definisiin di
function signature.

#### Nyimpen Baris yang Cocok

Biar fungsi ini kelar, kita butuh cara buat nyimpen baris-baris yang cocok dan
bakal kita balikin nanti. Untuk itu, kita bisa bikin vector mutable sebelum
`for` loop dan manggil method `push` buat masukin `line` ke vector. Setelah
`for` loop selesai, tinggal return vectornya. Kayak di Listing 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Nyimpen baris yang cocok supaya bisa kita return">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Sekarang fungsi `search` harusnya cuma bakal balikin baris yang mengandung
`query`, dan test kita harusnya lulus. Yuk coba jalanin testnya:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Mantap! Testnya lulus, berarti udah jalan sesuai harapan.

Sampai di titik ini, kita bisa mulai kepikiran buat refactor implementasi
`search` sambil tetep pastiin testnya lulus supaya perilakunya tetap sama. Kode
di fungsi search sebenernya udah oke, tapi masih belum manfaatin fitur-fitur
keren dari iterator. Kita bakal balik lagi ke contoh ini di
[Chapter 13][ch13-iterators]<!-- ignore --> buat bahas iterator lebih dalam dan
ngeliat cara ningkatinnya.

Sekarang seluruh programnya harusnya udah bekerja! Yuk kita cobain, pertama
pakai kata yang harusnya cuma match satu baris dari puisi Emily Dickinson:
_frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Keren! Sekarang coba kata yang match ke beberapa baris, misalnya _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Dan terakhir, pastiin gak ada hasil yang keluar kalau kita cari kata yang gak
ada sama sekali di puisinya, misalnya _monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Cakep! Kita berhasil bikin versi mini dari tool klasik dan belajar banyak
tentang cara nge-structure aplikasi. Kita juga udah belajar sedikit soal input
output file, lifetimes, testing, dan parsing command line.

Buat nutupin project ini, kita bakal sekilas nunjukin cara kerja environment
variable dan cara nge-print ke standard error, dua hal yang berguna banget kalau
kamu bikin program command line.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
