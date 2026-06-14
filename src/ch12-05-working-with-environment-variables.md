## Berurusan dengan Environment Variables

Kita bakal meningkatkan *binary* `minigrep` dengan menambahkan fitur tambahan: 
opsi pencarian *case-insensitive* (tidak membedakan huruf besar/kecil) yang 
bisa dinyalakan oleh *user* via *environment variable* (variabel lingkungan). 
Kita bisa saja bikin fitur ini jadi opsi di *command line* dan mewajibkan 
*user* buat memasukkannya setiap kali mereka mau fitur itu aktif, tapi dengan 
menjadikannya *environment variable*, kita membiarkan para *user* untuk menge-set 
*environment variable*-nya sekali saja dan semua pencarian mereka bakal jadi 
*case-insensitive* selama sesi terminal (terminal session) tersebut.

### Menulis Pengujian yang Gagal buat Fungsi `search` yang Case-Insensitive

Pertama-tama kita menambahkan fungsi `search_case_insensitive` baru ke _library_ 
`minigrep` yang bakal dipanggil pas *environment variable*-nya punya nilai. Kita 
bakal terus ngikutin proses TDD, jadi langkah pertamanya adalah kembali menulis 
pengujian yang gagal. Kita bakal menambahkan pengujian baru buat fungsi baru 
`search_case_insensitive` ini lalu me-*rename* pengujian lama kita dari 
`one_result` jadi `case_sensitive` buat memperjelas perbedaan di antara kedua 
pengujian ini, seperti yang ditunjukkan di Listing 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Menambahkan pengujian baru yang gagal untuk fungsi case-insensitive yang mau kita tambahkan">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Perhatikan bahwa kita juga sudah ngedit `contents` di pengujian lama. Kita 
menambahkan satu baris baru dengan teks `"Duct tape."` yang memakai huruf _D_ 
kapital yang tidak boleh cocok dengan kueri `"duct"` ketika kita lagi mencari 
dengan metode pencarian yang *case-sensitive*. Ngubah pengujian lama dengan 
cara ini ngebantu kita memastikan kalau kita tidak bakal tidak sengaja merusak 
fungsionalitas pencarian *case-sensitive* yang sudah kita implementasikan. 
Pengujian ini seharusnya sukses sekarang dan bakal terus sukses saat kita 
ngerjain fungsi pencarian yang *case-insensitive*.

Pengujian baru buat pencarian yang *case-insensitive* ini memakai `"rUsT"` 
sebagai kuerinya. Di dalam fungsi `search_case_insensitive` yang bakal kita 
tambahkan nanti, kueri `"rUsT"` ini seharusnya cocok sama baris yang 
mengandung `"Rust:"` dengan huruf _R_ kapital, dan juga cocok sama baris 
`"Trust me."` biarpun *casing* (huruf besar/kecil)-nya berbeda dari kueri aslinya. 
Ini adalah pengujian kita yang gagal, dan pengujian ini bakal gagal di-compile 
karena kita belum mendefinisikan fungsi `search_case_insensitive`. Kalau mau, 
silakan tambahkan implementasi kerangkanya yang selalu mengembalikan vector 
kosong, mirip kayak yang kita lakukan buat fungsi `search` di Listing 12-16 buat 
melihat pengujiannya berhasil di-compile lalu gagal.

### Mengimplementasikan Fungsi `search_case_insensitive`

Fungsi `search_case_insensitive`, yang ditunjukkan di Listing 12-21, bakal 
mirip sekali sama fungsi `search`. Satu-satunya perbedaan adalah kita bakal 
mengubah `query` dan setiap `line` jadi huruf kecil (lowercase) agar tidak peduli 
apa *casing* dari argumen inputnya, mereka bakal punya *casing* yang sama pas 
kita mengecek apakah baris tersebut mengandung kuerinya.

<Listing number="12-21" file-name="src/lib.rs" caption="Mendefinisikan fungsi `search_case_insensitive` agar mengubah kueri dan baris teks jadi huruf kecil sebelum membandingkan keduanya">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Pertama kita ngubah string `query` jadi huruf kecil lalu menyimpannya ke dalam 
variabel baru dengan nama yang sama, menimpa (shadowing) variabel `query` aslinya. 
Memanggil `to_lowercase` pada kueri ini diperlukan supaya tidak peduli apakah 
kueri dari *user* itu `"rust"`, `"RUST"`, `"Rust"`, atau `"rUsT"`, kita bakal 
memperlakukan kueri tersebut seolah-olah itu `"rust"` dan tidak mempedulikan 
*casing*-nya. Meskipun `to_lowercase` bakal menangani Unicode dasar, fungsi ini 
tidak 100 persen akurat. Kalau kita lagi bikin aplikasi sungguhan, kita bakal 
mau bekerja sedikit lebih jauh di sini, tapi bagian ini membahas tentang 
_environment variables_, bukan Unicode, jadi kita biarkan saja seperti ini buat 
sekarang.

Perhatikan bahwa `query` sekarang adalah tipe `String` bukannya _string slice_ 
karena pemanggilan `to_lowercase` itu membikin data baru alih-alih merujuk ke 
data yang sudah ada. Katakanlah kuerinya itu `"rUsT"`, sebagai contoh: *string 
slice* tersebut tidak mengandung karakter `u` atau `t` kecil yang bisa kita 
pakai, jadi kita harus mengalokasikan memori buat `String` baru yang mengandung 
`"rust"`. Saat kita meneruskan `query` sebagai argumen ke method `contains` 
sekarang, kita perlu menambahkan *ampersand* (`&`) karena *signature* dari 
`contains` didefinisikan buat nerima _string slice_.

Selanjutnya, kita nambahin panggilan ke `to_lowercase` di setiap `line` buat 
mengubah semua karakternya jadi huruf kecil. Sekarang karena kita udah mengonversi 
baik `line` maupun `query` jadi huruf kecil, kita bakal menemukan baris yang 
cocok tidak peduli apa *casing* kuerinya.

Mari kita lihat apakah implementasi ini berhasil melewati pengujian:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Mantap! Mereka sukses. Sekarang, mari kita panggil fungsi 
`search_case_insensitive` baru ini dari dalam fungsi `run`. Pertama kita bakal 
menambahkan opsi konfigurasi ke struct `Config` buat beralih antara metode 
pencarian *case-sensitive* dan *case-insensitive*. Nambahin field ini bakal 
menghasilkan error *compiler* karena kita belum menginisialisasi field ini di 
mana pun:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Kita nambahin field `ignore_case` yang menampung sebuah Boolean. Selanjutnya, 
kita butuh fungsi `run` buat mengecek nilai dari field `ignore_case` lalu 
memakai nilai itu buat menentukan apakah harus memanggil fungsi `search` atau 
fungsi `search_case_insensitive`, seperti yang ditunjukkan di Listing 12-22. 
Ini masih belum bisa di-compile.

<Listing number="12-22" file-name="src/main.rs" caption="Memanggil `search` atau `search_case_insensitive` berdasarkan nilai dari `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Terakhir, kita perlu mengecek apakah *environment variable*-nya diset. 
Fungsi-fungsi buat berinteraksi dengan *environment variables* berada di 
dalam modul `env` di *standard library*, yang mana sudah berada di dalam 
*scope* di bagian paling atas _src/main.rs_. Kita bakal memakai fungsi `var` 
dari modul `env` buat mengecek dan melihat apakah ada nilai yang sudah diset 
buat *environment variable* bernama `IGNORE_CASE`, seperti yang ditunjukkan 
di Listing 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="Mengecek jika ada nilai di *environment variable* yang bernama `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Di sini, kita membuat variabel baru, `ignore_case`. Buat menge-set nilainya, 
kita memanggil fungsi `env::var` dan meneruskan nama dari *environment 
variable* `IGNORE_CASE` kepadanya. Fungsi `env::var` mengembalikan sebuah 
`Result` yang bakal jadi varian sukses `Ok` yang berisi nilai dari 
*environment variable* tersebut jika ia telah diset dengan nilai apa pun. Ia 
bakal mengembalikan varian `Err` kalau *environment variable*-nya tidak diset.

Kita memakai method `is_ok` pada `Result` tersebut buat mengecek apakah 
*environment variable*-nya diset, yang berarti programnya seharusnya melakukan 
pencarian secara *case-insensitive*. Kalau *environment variable* `IGNORE_CASE` 
tidak diset sama sekali, `is_ok` bakal mengembalikan `false` dan programnya 
bakal melakukan pencarian *case-sensitive*. Kita tidak peduli dengan *nilai* 
dari *environment variable*-nya, yang penting dia diset atau tidak aja, 
makanya kita memakai `is_ok` dan tidak memakai `unwrap`, `expect`, atau 
method lain dari `Result` yang sudah kita lihat sebelumnya.

Kita meneruskan nilai di variabel `ignore_case` ke instance `Config` agar 
fungsi `run` bisa ngebaca nilai itu dan memutuskan apakah bakal memanggil 
`search_case_insensitive` atau `search`, seperti yang sudah kita implementasikan 
di Listing 12-22.

Mari kita coba! Pertama kita bakal jalankan program kita tanpa menge-set 
*environment variable* apa pun dan memakai kueri `to`, yang mana seharusnya 
cocok dengan baris mana pun yang mengandung kata _to_ dalam huruf kecil semua:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Sepertinya masih jalan dengan baik! Sekarang mari kita jalankan programnya 
dengan `IGNORE_CASE` diset ke `1` tapi dengan kueri _to_ yang sama:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Kalau kita pakai PowerShell, kita perlu menge-set *environment variable*-nya 
lalu menjalankan programnya sebagai dua *command* yang terpisah:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Ini bakal membikin `IGNORE_CASE` bertahan (persist) sepanjang sisa sesi *shell* 
kita. Ia bisa di-_unset_ pakai *cmdlet* `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Kita seharusnya mendapatkan baris-baris yang mengandung _to_ yang mungkin 
huruf-hurufnya kapital:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Hebat, kita juga dapat baris yang mengandung _To_! Program `minigrep` kita 
sekarang bisa melakukan pencarian *case-insensitive* yang dikendalikan oleh 
sebuah *environment variable*. Sekarang kita tahu gimana caranya mengatur opsi 
yang diset melalui argumen *command line* maupun *environment variables*.

Beberapa program mengizinkan pemakaian argumen *dan* *environment variables* 
buat tujuan konfigurasi yang sama. Di kasus seperti itu, program-program 
tersebut harus memutuskan mana yang harus didahulukan (precedence). Sebagai 
latihan, cobalah mengatur opsi *case sensitivity* (kepekaan huruf besar/kecil) 
melalui argumen *command line* atau *environment variable*. Putuskan apakah 
argumen *command line* atau *environment variable* yang seharusnya lebih 
didahulukan kalau ternyata programnya dijalankan dengan salah satu opsi 
dijadikan *case-sensitive* sementara opsi lainnya diset untuk mengabaikan 
*casing*.

Modul `std::env` berisi banyak lagi fitur yang berguna buat berurusan dengan 
*environment variables*: cek dokumentasinya buat melihat fitur apa saja yang 
tersedia.
