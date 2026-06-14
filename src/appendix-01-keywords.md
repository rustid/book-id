## Lampiran A: Kata Kunci (Keywords)

Daftar berikut ini berisi *keywords* (kata kunci) yang direservasi buat 
penggunaan saat ini maupun penggunaan di masa depan oleh bahasa Rust. Oleh 
karena itu, kata-kata ini tidak bisa dipakai sebagai _identifiers_ (pengidentifikasi) 
(terkecuali dipakai sebagai *raw identifiers*, seperti yang bakal kita obrolin di 
bagian [“Raw Identifiers”][raw-identifiers]<!-- ignore -->). _Identifiers_ adalah 
nama-nama fungsi, variabel, parameter, field struct, modul, _crates_, konstanta, 
_macros_, nilai statis, atribut, tipe, _traits_, atau _lifetimes_.

[raw-identifiers]: #raw-identifiers

### Keywords yang Saat Ini Dipakai

Berikut adalah daftar *keywords* yang saat ini sedang dipakai, beserta 
penjelasan fungsionalitasnya.

- **`as`**: melakukan _casting_ (pengubahan tipe) primitif, menghilangkan 
  ambiguitas trait spesifik yang menampung sebuah item, atau mengganti nama 
  item di dalam statement `use`
- **`async`**: mengembalikan sebuah `Future` ketimbang memblokir _thread_ saat ini
- **`await`**: menahan eksekusi sampai hasil dari sebuah `Future` sudah siap
- **`break`**: keluar dari sebuah perulangan (loop) secara langsung
- **`const`**: mendefinisikan item konstanta atau *raw pointers* konstanta
- **`continue`**: lanjut ke iterasi perulangan berikutnya
- **`crate`**: di dalam _module path_, ini merujuk ke akar _crate_ (crate root)
- **`dyn`**: penyaluran dinamis (dynamic dispatch) ke sebuah _trait object_
- **`else`**: jalan alternatif (_fallback_) untuk struktur *control flow* `if` 
  dan `if let`
- **`enum`**: mendefinisikan sebuah enumerasi
- **`extern`**: menautkan (link) sebuah fungsi atau variabel eksternal
- **`false`**: nilai literal salah (false) pada Boolean
- **`fn`**: mendefinisikan sebuah fungsi atau tipe dari _function pointer_
- **`for`**: perulangan (loop) melewati item-item dari sebuah iterator, 
  mengimplementasikan sebuah trait, atau menentukan _higher-ranked lifetime_
- **`if`**: percabangan berdasarkan hasil dari ekspresi kondisional
- **`impl`**: mengimplementasikan fungsionalitas bawaan (inherent) atau fungsionalitas trait
- **`in`**: bagian dari sintaks perulangan `for`
- **`let`**: mengikat (_bind_) sebuah variabel
- **`loop`**: perulangan (loop) tanpa syarat
- **`match`**: mencocokkan sebuah nilai terhadap _patterns_ (pola-pola)
- **`mod`**: mendefinisikan sebuah modul
- **`move`**: membuat _closure_ mengambil alih kepemilikan (ownership) atas semua 
  nilai yang ditangkapnya (captures)
- **`mut`**: menandakan mutabilitas pada referensi, *raw pointers*, atau _pattern bindings_
- **`pub`**: menandakan visibilitas publik pada field struct, blok `impl`, atau modul
- **`ref`**: mengikat berdasarkan referensi
- **`return`**: mengembalikan nilai dari fungsi
- **`Self`**: _type alias_ untuk tipe yang sedang kita definisikan atau implementasikan
- **`self`**: subjek dari _method_ atau modul saat ini
- **`static`**: variabel global atau _lifetime_ yang berlangsung selama keseluruhan 
  eksekusi program
- **`struct`**: mendefinisikan sebuah struktur
- **`super`**: modul induk (parent module) dari modul saat ini
- **`trait`**: mendefinisikan sebuah trait
- **`true`**: nilai literal benar (true) pada Boolean
- **`type`**: mendefinisikan _type alias_ atau _associated type_
- **`union`**: mendefinisikan [union][union]<!-- ignore -->; hanya menjadi keyword saat dipakai 
  di dalam deklarasi _union_
- **`unsafe`**: menandakan kode, fungsi, trait, atau implementasi yang tidak aman
- **`use`**: membawa *symbols* ke dalam _scope_; menentukan tangkapan pasti (precise captures) 
  untuk batasan _generic_ dan _lifetime_
- **`where`**: menandakan klausa yang membatasi (constrain) sebuah tipe
- **`while`**: perulangan bersyarat berdasarkan hasil dari sebuah ekspresi

[union]: ../reference/items/unions.html

### Keywords yang Direservasi buat Penggunaan di Masa Depan

Keywords berikut ini belum punya fungsionalitas apa pun, tapi sudah direservasi 
oleh Rust buat potensi pemakaian di masa depan:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### Raw Identifiers

_Raw identifiers_ adalah sintaks yang memungkinkan kita memakai keywords di tempat-tempat 
di mana mereka biasanya tidak diperbolehkan. Kita bisa memakai _raw identifier_ dengan 
menambahkan awalan `r#` pada sebuah keyword.

Misalnya, `match` itu adalah sebuah keyword. Kalau kita mencoba men-compile 
fungsi berikut yang memakai `match` sebagai namanya:

<span class="filename">Nama File: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

kita bakal mendapat error ini:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

Error tersebut menunjukkan kalau kita tidak bisa memakai keyword `match` sebagai 
identifikasi fungsi. Supaya bisa memakai `match` sebagai nama fungsi, kita perlu 
memakai sintaks _raw identifier_, kayak gini:

<span class="filename">Nama File: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Kode ini bakal berhasil di-compile tanpa error. Perhatikan awalan `r#` pada 
nama fungsi di bagian definisinya sekaligus di tempat fungsi tersebut dipanggil 
di dalam `main`.

_Raw identifiers_ membiarkan kita memakai kata apa pun sebagai identifier, bahkan 
jika kata tersebut kebetulan adalah keyword yang direservasi. Hal ini ngasih 
kita kebebasan lebih dalam memilih nama identifier, sekaligus memungkinkan kita 
berintegrasi dengan program yang ditulis di bahasa lain di mana kata-kata 
tersebut bukanlah keywords. Selain itu, _raw identifiers_ juga memungkinkan 
kita memakai *libraries* yang ditulis dalam edisi (_edition_) Rust yang berbeda 
dari yang dipakai oleh _crate_ kita. Misalnya, `try` bukanlah keyword di edisi 2015, 
tapi dia menjadi keyword di edisi 2018, 2021, dan 2024. Kalau kita bergantung 
pada sebuah *library* yang ditulis pakai edisi 2015 dan punya fungsi `try`, kita 
harus memakai sintaks _raw identifier_, yaitu `r#try` di kasus ini, buat memanggil 
fungsi tersebut dari kode kita di edisi-edisi yang lebih baru. Lihat [Lampiran E][appendix-e] 
untuk informasi lebih lanjut tentang *editions*.

[appendix-e]: appendix-05-editions.html
