## Error Unrecoverable pake `panic!`

Kadang hal-hal buruk terjadi di kode kita, dan nggak ada yang bisa kita lakuin 
buat ngatasinnya. Di kasus kayak gini, Rust punya macro `panic!`. Ada dua cara 
buat micu sebuah _panic_ di praktiknya: dengan ngambil aksi yang bikin kode 
kita _panic_ (kayak akses array ngelewatin akhirnya) atau dengan secara 
eksplisit manggil macro `panic!`. Di dua kasus itu, kita nyebabin sebuah _panic_ 
di program kita. Secara default, _panics_ ini bakal nyetak pesan kegagalan, 
_unwind_ (nggulung balik), ngebersihin _stack_, terus _quit_ (keluar). Lewat 
_environment variable_ (variabel lingkungan), kita juga bisa nyuruh Rust buat 
nampilin _call stack_ pas sebuah _panic_ terjadi biar lebih gampang buat ngelacak 
sumber dari _panic_ itu.

> ### Unwinding the Stack atau Aborting sebagai Respon ke Panic
>
> Secara default, pas sebuah _panic_ terjadi, program mulai proses _unwinding_, 
> yang artinya Rust jalan mundur ke atas _stack_ terus ngebersihin data dari 
> tiap fungsi yang dia temuin. Tapi, jalan mundur terus ngebersihin data itu 
> butuh banyak kerjaan. Makanya, Rust ngebolehin kita milih alternatif yaitu 
> langsung _aborting_ (ngebatalin), yang ngeakhirin program tanpa ngebersihin 
> apa-apa.
>
> Memori yang tadinya dipake program terus bakal perlu dibersihin sama sistem 
> operasi (OS). Kalau di project kita kita butuh ngebikin file _binary_ hasil 
> akhirnya sekecil mungkin, kita bisa pindah dari _unwinding_ jadi _aborting_ 
> pas terjadi _panic_ dengan nambahin `panic = 'abort'` ke bagian `[profile]` 
> yang sesuai di file _Cargo.toml_ kita. Misalnya, kalau kita mau _abort_ pas 
> _panic_ di _release mode_, tambahin ini:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Yuk kita coba manggil `panic!` di program yang simpel:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Pas kita jalanin programnya, kita bakal liat yang kayak gini:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Pemanggilan `panic!` nyebabin pesan error yang ada di dua baris terakhir. Baris 
pertama nunjukin pesan _panic_ kita dan lokasi di _source code_ kita di mana 
_panic_-nya terjadi: _src/main.rs:2:5_ nunjukin kalau itu ada di baris kedua, 
karakter kelima di file _src/main.rs_ kita.

Di kasus ini, baris yang ditunjuk adalah bagian dari kode kita, dan kalau kita 
buka baris itu, kita bakal liat pemanggilan macro `panic!`. Di kasus lain, 
pemanggilan `panic!` mungkin ada di kode yang dipanggil sama kode kita, dan 
nama file serta nomor baris yang dilaporin sama pesan error-nya bakal nunjukin 
kode punya orang lain di mana macro `panic!` itu dipanggil, bukan baris kode 
kita yang akhirnya nyebabin pemanggilan `panic!` itu.

<!-- Old heading. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Kita bisa pake _backtrace_ dari fungsi-fungsi tempat `panic!` dipanggil buat 
nyari tau bagian mana dari kode kita yang nyebabin masalahnya. Buat mahamin 
gimana cara pake _backtrace_ `panic!`, yuk kita liat contoh lain dan liat 
gimana rasanya pas pemanggilan `panic!` dateng dari sebuah _library_ gara-gara 
ada _bug_ di kode kita, bukannya dari kode kita yang manggil macro-nya secara 
langsung. Listing 9-1 punya kode yang nyoba akses indeks di vector yang ngelewatin 
rentang (range) indeks yang valid.

<Listing number="9-1" file-name="src/main.rs" caption="Nyoba akses elemen yang ngelewatin akhir sebuah vector, yang bakal nyebabin pemanggilan `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Di sini, kita lagi nyoba akses elemen ke-100 dari vector kita (yang ada di indeks 
99 karena indexing mulai dari nol), tapi vector-nya cuma punya tiga elemen. Di 
situasi ini, Rust bakal _panic_. Pake `[]` seharusnya balikin sebuah elemen, 
tapi kalau kita ngasih indeks yang nggak valid, nggak ada elemen yang bisa 
dibalikin Rust di sini yang bener.

Di C, nyoba baca data ngelewatin akhir dari struktur data itu dianggap sebagai 
_undefined behavior_ (perilaku yang nggak terdefinisi). Kita mungkin bakal dapet 
apa pun yang ada di lokasi memori yang harusnya sesuai sama elemen itu di 
struktur datanya, walaupun memori itu bukan milik struktur data tersebut. Ini 
disebut _buffer overread_ dan bisa memicu celah keamanan (_security vulnerabilities_) 
kalau seorang _attacker_ (penyerang) bisa memanipulasi indeks sedemikian rupa 
biar bisa baca data yang nggak boleh mereka baca yang disimpan setelah struktur 
data itu.

Buat ngelindungin program kita dari celah semacam ini, kalau kita nyoba baca 
elemen di indeks yang nggak ada, Rust bakal ngehentiin eksekusi dan nolak buat 
lanjut. Yuk kita coba dan liat hasilnya:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Error ini nunjuk ke baris 4 dari _main.rs_ kita di mana kita nyoba akses indeks 
`99` dari vector di `v`.

Baris `note:` ngasih tau kita kalau kita bisa nge-set _environment variable_ 
`RUST_BACKTRACE` buat dapetin _backtrace_ dari apa persisnya yang terjadi yang 
nyebabin error itu. Sebuah _backtrace_ adalah daftar dari semua fungsi yang 
udah dipanggil sampe bisa nyampe ke titik ini. _Backtraces_ di Rust cara 
kerjanya sama kayak di bahasa lain: kunci buat baca _backtrace_ adalah mulai 
dari atas terus baca sampe kita liat file yang kita tulis sendiri. Itu adalah 
titik di mana masalahnya bermula. Baris-baris di atas titik itu adalah kode yang 
dipanggil sama kode kita; baris-baris di bawahnya adalah kode yang manggil kode 
kita. Baris-baris sebelum dan sesudah ini mungkin nyakup kode inti Rust, kode 
_standard library_, atau crates yang lagi kita pake. Yuk kita coba dapetin 
_backtrace_ dengan nge-set _environment variable_ `RUST_BACKTRACE` ke nilai apa 
pun kecuali `0`. Listing 9-2 nunjukin output yang mirip sama apa yang bakal kita 
liat.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="_Backtrace_ yang dihasilin oleh pemanggilan `panic!` yang ditampilin pas _environment variable_ `RUST_BACKTRACE` di-set">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Outputnya lumayan banyak tuh! Output pasti yang bakal kita liat mungkin beda-beda 
tergantung dari sistem operasi sama versi Rust yang dipake. Biar dapet 
_backtraces_ dengan informasi sebanyak ini, _debug symbols_ harus dinyalain. 
_Debug symbols_ dinyalain secara default pas kita pake `cargo build` atau 
`cargo run` tanpa _flag_ `--release`, kayak yang kita lakuin di sini.

Di output di Listing 9-2, baris 6 dari _backtrace_-nya nunjuk ke baris di 
project kita yang nyebabin masalahnya: baris 4 dari _src/main.rs_. Kalau kita 
nggak mau program kita _panic_, kita harus mulai nyari masalahnya di lokasi 
yang ditunjuk sama baris pertama yang nyebutin file yang kita tulis sendiri. Di 
Listing 9-1, di mana kita sengaja nulis kode yang bakal _panic_, cara buat 
benerin _panic_-nya adalah dengan nggak minta elemen yang ada di luar rentang 
indeks vector-nya. Pas kode kita _panic_ di masa depan nanti, kita harus nyari 
tau aksi apa yang lagi dilakuin sama kode itu pake nilai apa sampe bisa 
nyebabin _panic_ dan apa yang seharusnya dilakuin sama kode itu sebagai gantinya.

Kita bakal balik lagi bahas `panic!` dan kapan kita harus dan nggak harus pake 
`panic!` buat nanganin kondisi error di bagian [“To `panic!` or Not to 
`panic!`”][to-panic-or-not-to-panic] nanti di bab ini. Selanjutnya, kita bakal 
liat gimana caranya buat pulih dari sebuah error pake `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
