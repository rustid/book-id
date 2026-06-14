## Lampiran D - Alat Bantu (Tools) Pengembangan yang Berguna

Di lampiran ini, kita bakal ngomongin soal beberapa alat bantu pengembangan yang 
berguna yang mana emang disediain sama project Rust ini. Kita bakal ngelihat 
alat pemformatan otomatis, cara-cara cepet buat nerapin perbaikan dari pesan 
peringatan, sebuah _linter_ (pemeriksa kode), dan juga seputar cara integrasi dengan IDEs.

### Pemformatan Otomatis dengan `rustfmt`

Alat `rustfmt` merombak dan memformat ulang (reformats) kode kita sedemikian 
rupa sehingga mengikuti gaya kode standar dari komunitas. Banyak proyek 
kolaboratif memakai `rustfmt` untuk membantu mencegah terjadinya perdebatan 
seputar gaya penulisan kode mana yang seharusnya dipakai saat menulis kode Rust: 
intinya, semua orang memformat kode mereka secara seragam memakai alat ini.

Instalasi Rust secara bawaan (_default_) sudah menyertakan `rustfmt`, jadi kita 
seharusnya saat ini sudah punya program `rustfmt` dan `cargo-fmt` di sistem kita. 
Dua perintah ini pada dasarnya serupa dengan `rustc` dan `cargo` di mana `rustfmt` 
memberikan opsi pengaturan yang lebih mendetail dan `cargo-fmt` memahami konvensi 
dari sebuah proyek yang memakai Cargo. Untuk memformat proyek Cargo apa pun, 
silakan ketik perintah berikut:

```console
$ cargo fmt
```

Menjalankan perintah ini otomatis bakal memformat ulang seantero kode Rust yang ada 
di dalam _crate_ saat ini. Perintah ini murni cuma bakal mengubah gaya kodenya saja, 
bukan mengubah semantik dari kodenya. Buat dapat info lebih lanjut soal `rustfmt`, 
silakan baca [dokumentasinya di sini][rustfmt].

### Perbaiki Kode kita dengan `rustfix`

Alat bantu `rustfix` sudah disertakan bareng instalasi Rust dan bisa secara otomatis membetulkan pesan peringatan _compiler_ yang punya rute perbaikan yang jelas untuk masalah tersebut, yang kemungkinan besar merupakan apa yang kita inginkan. Kita kemungkinan besar sudah pernah melihat pesan peringatan dari _compiler_ sebelumnya. Sebagai contoh, perhatikan kode berikut ini:

<span class="filename">Nama File: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

Di sini, kita mendefinisikan variabel `x` sebagai sesuatu yang bisa diubah (_mutable_), 
tapi kita tidak pernah mengubahnya. Rust bakal otomatis memberi tahu kita pesan peringatan:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

Pesan peringatan ini menyarankan supaya kita menyingkirkan keyword `mut` tersebut. 
Kita bisa secara otomatis menerapkan saran tersebut menggunakan bantuan alat `rustfix` 
dengan menjalankan perintah `cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Saat kita melihat isi file _src/main.rs_ lagi, kita bakal melihat kalau `cargo fix` 
telah membetulkan kodenya:

<span class="filename">Nama File: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

Sekarang variabel `x` sudah tidak bisa diubah (_immutable_), dan pesan peringatan 
itu sudah tidak muncul lagi.

Kita juga bisa memanfaatkan perintah `cargo fix` buat ngebantu mengurus masa transisi 
kode kita di antara edisi-edisi (*editions*) Rust yang berbeda-beda. Perkara seputar 
*editions* ini diulas di [Lampiran E][editions]<!--
ignore -->.

### Lints (Peringatan Tambahan) yang Lebih Beragam Memakai Clippy

Alat Clippy adalah sebuah koleksi _lints_ (aturan pemeriksa kode) yang bertugas 
menganalisis kode kita sehingga kita bisa menangkap kesalahan umum dan meningkatkan 
kualitas kode Rust kita. Clippy sudah disertakan di dalam standar instalasi Rust.

Buat menjalankan _lints_ Clippy pada sembarang project Cargo, silakan ketik perintah 
berikut:

```console
$ cargo clippy
```

Misalnya, katakanlah kita sedang menulis sebuah program yang memakai estimasi nilai 
aproksimasi buat sebuah konstanta matematika, seperti pi, seperti yang dilakukan 
program ini:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Menjalankan `cargo clippy` pada project ini bakal menghasilkan pesan error berikut:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Pesan error ini memberi tahu kita kalau Rust sudah punya konstanta `PI` yang jauh 
lebih presisi, dan program kita bakal jadi lebih benar kalau kita memakai konstanta 
tersebut. Jadinya kita bisa mengubah kode tersebut supaya memakai konstanta `PI`.

Kode berikut ini tidak bakal memicu munculnya error atau peringatan apa pun dari Clippy:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Untuk informasi lebih rinci tentang Clippy, silakan tuju [halaman dokumentasinya][clippy].

### Integrasi IDE Memakai `rust-analyzer`

Untuk membantu masalah integrasi IDE, komunitas Rust merekomendasikan penggunaan 
[`rust-analyzer`][rust-analyzer]. Alat ini adalah sekumpulan utilitas berbasis 
_compiler_ yang memakai protokol _Language Server Protocol_ ([LSP][lsp]), yang mana 
merupakan sebuah spesifikasi bagi IDE dan bahasa pemrograman supaya mereka bisa 
berkomunikasi satu sama lain. Berbagai _clients_ bisa memakai `rust-analyzer`, termasuk 
di antaranya [plug-in Rust analyzer untuk Visual Studio Code][vscode].

Silakan kunjungi [*home page* dari project `rust-analyzer`][rust-analyzer] untuk 
mendapatkan petunjuk instalasinya, lalu pasang dukungan *language server* tersebut 
ke dalam IDE spesifik kita. IDE kita bakal mendapatkan tambahan kapabilitas istimewa 
seperti _autocompletion_ (penyelesaian otomatis), _jump to definition_ (loncat ke definisi), 
dan _inline errors_ (pesan error yang muncul sebaris dengan kode).

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
