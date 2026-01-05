## Bekerja dengan Environment Variable

Kita bakal ningkatin binary `minigrep` dengan nambahin fitur baru: opsi
pencarian yang **gak peka huruf besar–kecil (case-insensitive)** yang bisa
diaktifin user lewat environment variable. Sebenernya kita bisa aja bikin fitur
ini jadi opsi command line, tapi itu bakal maksa user buat terus ngetik opsinya
setiap kali mau dipake. Dengan dijadiin environment variable, user cukup set
sekali aja dan semua pencarian di session terminal itu bakal otomatis jadi
case-insensitive.

<!-- Old headings. Do not remove or links may break. -->

<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### Nulis Test yang Gagal Buat Case-Insensitive Search

Pertama, kita bakal nambahin fungsi baru `search_case_insensitive` ke library
`minigrep` yang bakal dipanggil kalau environment variable punya nilai. Kita
tetap lanjut pake proses TDD, jadi langkah pertama masih sama: bikin test yang
gagal dulu. Kita bakal nambahin test baru buat fungsi `search_case_insensitive`,
dan ganti nama test lama dari `one_result` jadi `case_sensitive` biar perbedaan
dua test ini makin jelas, kayak di Listing 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Nambahin test gagal baru buat fungsi case-insensitive yang bakal kita bikin">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Perhatiin kalau kita juga ngedit `contents` di test lama. Kita nambah satu baris
`"Duct tape."` dengan huruf _D_ kapital yang **gak seharusnya ke-detect** pas
pencarian case-sensitive `"duct"`. Perubahan ini buat mastiin kita gak tanpa
sengaja ngerusak fitur case-sensitive yang udah jadi. Test lama ini harusnya
tetap lulus sekarang dan harus tetap lulus selama kita ngerjain fitur
case-insensitive.

Test baru buat pencarian case-_insensitive_ pake query `"rUsT"`. Di fungsi
`search_case_insensitive` yang bakal kita tambahin, query `"rUsT"` harusnya
cocok sama baris `"Rust:"` yang huruf depannya kapital, dan juga cocok sama
`"Trust me."` walaupun casing-nya beda. Ini test gagal kita, dan sekarang bakal
gagal compile karena fungsi `search_case_insensitive` belum ada. Kalau mau,
bikin aja skeleton function yang cuma return vector kosong dulu, sama kayak
waktu kita bikin `search` di Listing 12-16, biar testnya bisa compile dan gagal
normal.

### Implementasi Fungsi `search_case_insensitive`

Fungsi `search_case_insensitive`, kayak di Listing 12-21, bakal hampir sama
kayak `search`. Bedanya cuma satu: kita bakal **lowercase** `query` dan tiap
`line` biar perbandingannya gak peduli huruf besar kecil.

<Listing number="12-21" file-name="src/lib.rs" caption="Ngedefine fungsi `search_case_insensitive` dengan nge-lowercase query dan baris sebelum dibandingin">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Pertama, kita lowercase string `query` dan simpen ke variabel baru dengan nama
yang sama (shadowing variabel lama). `to_lowercase` dipanggil supaya mau
query-nya `"rust"`, `"RUST"`, `"Rust"`, atau `"rUsT"`, semuanya bakal
diperlakuin sama.

Walaupun `to_lowercase` udah ngedukung Unicode basic, tapi emang gak 100% akurat
sih. Kalau ini aplikasi beneran, mungkin butuh effort lebih. Tapi karena fokus
bagian ini environment variable, bukan Unicode, kita santai aja dulu.

Yang perlu dicatat: Sekarang `query` jadi `String`, bukan string slice lagi,
karena `to_lowercase` bikin data baru, bukan referensi data lama. Misal query
`"rUsT"`, string aslinya gak punya huruf `u` dan `t` lowercase, jadi kita harus
bikin `String` baru `"rust"`. Dan karena `contains` butuh string slice, kita
kasih `&query`.

Selanjutnya kita juga panggil `to_lowercase` ke tiap `line` biar semuanya
lowercase juga. Karena sekarang `line` dan `query` udah sama-sama lowercase,
pencarian jadi benar-benar case-insensitive.

Coba cek testnya:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Mantap, lulus semua 🎉

Sekarang kita panggil fungsi `search_case_insensitive` dari `run`. Pertama, kita
nambah opsi konfigurasi ke struct `Config` buat nge-switch antara mode
case-sensitive dan case-insensitive. Tapi nambah field ini bakal bikin compiler
error dulu karena belum diinisialisasi:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Kita nambah field `ignore_case` bertipe Boolean. Selanjutnya, fungsi `run` harus
ngecek nilai `ignore_case` dan mutusin apakah mau manggil `search` atau
`search_case_insensitive`, kayak di Listing 12-22. Ini juga masih belum compile.

<Listing number="12-22" file-name="src/main.rs" caption="Milih antara `search` atau `search_case_insensitive` tergantung nilai `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Terakhir, kita perlu ngecek environment variable. Fungsi buat kerja sama
environment variable ada di module `env` di standard library, dan dia udah
di-import di bagian atas _src/main.rs_. Kita bakal pake fungsi `var` buat ngecek
apakah environment variable `IGNORE_CASE` punya nilai, kayak di Listing 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="Ngecek environment variable `IGNORE_CASE` apakah punya nilai atau enggak">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Di sini kita bikin variabel baru `ignore_case`. Nilainya diambil dari
`env::var("IGNORE_CASE")`. Fungsi `env::var` balikannya `Result`:

- `Ok(value)` kalau environment variablenya ada dan punya nilai apa pun
- `Err` kalau environment variablenya gak ada

Kita pake `is_ok` buat ngecek apakah variabel environment diset. Kalau
`IGNORE_CASE` diset → pencarian case-insensitive. Kalau gak diset → tetap
case-sensitive.

Kita gak peduli nilai environment variablenya apa, yang penting ada atau enggak,
jadi `is_ok` udah cukup. Nilai `ignore_case` ini kita lempar ke `Config`, biar
`run` bisa baca dan mutusin mau pake fungsi yang mana.

Sekarang kita coba! Pertama jalankan program TANPA environment variable, pakai
query `to`, yang cuma harus match huruf kecil.

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Masih jalan ✔️

Sekarang kita jalanin lagi tapi dengan `IGNORE_CASE=1`

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Kalau kamu pake PowerShell, caranya beda dikit:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Ini bikin `IGNORE_CASE` tetap aktif sampai session shell selesai. Buat
ngehapusnya:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Sekarang hasilnya harusnya juga match huruf kapital:

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Gas, berhasil 🎉 Sekarang `minigrep` bisa pencarian case-insensitive yang
dikontrol environment variable. Jadi sekarang kamu tau gimana caranya ngatur
opsi lewat argumen command line maupun environment variable.

Beberapa program malah nge-dukung dua-duanya sekaligus, dan biasanya salah
satunya punya prioritas lebih. Kalau mau latihan tambahan, coba bikin:

- case sensitivity bisa dikontrol via command line argument
- atau environment variable
- lalu tentuin mana yang lebih prioritas kalau dua-duanya dipakai

Module `std::env` masih punya banyak fitur lain buat kerja dengan environment
variable. Silakan cek dokumentasinya kalau penasaran!
