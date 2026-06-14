## Lampiran B: Operator dan Simbol

Lampiran ini berisi glosarium (kamus ringkas) tentang sintaks Rust, termasuk 
operator dan simbol-simbol lain yang muncul sendirian maupun di dalam konteks 
seperti _paths_, _generics_, _trait bounds_, _macros_, atribut, komentar, *tuples*, 
dan tanda kurung.

### Operator

Tabel B-1 berisi daftar operator di Rust, contoh gimana operator tersebut 
muncul di dalam konteksnya, penjelasan singkat, dan apakah operator 
tersebut bisa di-overload (overloadable) atau tidak. Kalau sebuah operator 
bisa di-overload, trait relevan yang dipakai buat nge-overload operator 
tersebut juga dicantumkan.

<span class="caption">Tabel B-1: Operator</span>

| Operator                  | Contoh                                                  | Penjelasan                                                            | Bisa Di-overload?  |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Ekspansi macro                                                       |                |
| `!`                       | `!expr`                                                 | Bitwise atau logical complement (komplemen logika)                                        | `Not`          |
| `!=`                      | `expr != expr`                                          | Perbandingan ketidaksamaan (nonequality)                                                | `PartialEq`    |
| `%`                       | `expr % expr`                                           | Sisa hasil bagi (remainder) aritmatika                                                  | `Rem`          |
| `%=`                      | `var %= expr`                                           | Sisa hasil bagi aritmatika dan assignment (penugasan)                                   | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | Meminjam (Borrow)                                                                |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Tipe pointer pinjaman (Borrowed pointer type)                                                 |                |
| `&`                       | `expr & expr`                                           | Bitwise AND                                                           | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | Bitwise AND dan assignment                                            | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | Logical AND (hubungan singkat/short-circuiting)                                          |                |
| `*`                       | `expr * expr`                                           | Perkalian aritmatika                                             | `Mul`          |
| `*=`                      | `var *= expr`                                           | Perkalian aritmatika dan assignment                              | `MulAssign`    |
| `*`                       | `*expr`                                                 | Dereference (Membuka rujukan)                                                           | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | Raw pointer                                                           |                |
| `+`                       | `trait + trait`, `'a + trait`                           | Batasan tipe gabungan (Compound type constraint)                                              |                |
| `+`                       | `expr + expr`                                           | Penjumlahan aritmatika                                                   | `Add`          |
| `+=`                      | `var += expr`                                           | Penjumlahan aritmatika dan assignment                                    | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Pemisah argumen dan elemen                                        |                |
| `-`                       | `- expr`                                                | Negasi aritmatika                                                   | `Neg`          |
| `-`                       | `expr - expr`                                           | Pengurangan aritmatika                                                | `Sub`          |
| `-=`                      | `var -= expr`                                           | Pengurangan aritmatika dan assignment                                 | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipe balasan (return type) fungsi dan closure                                      |                |
| `.`                       | `expr.ident`                                            | Akses ke field                                                          |                |
| `.`                       | `expr.ident(expr, ...| `+`                       | `trait + trait`, `'a + trait`                           | Batasan tipe gabungan (Compound type constraint)                                              |                |
| `+`                       | `expr + expr`                                           | Penjumlahan aritmatika                                                   | `Add`          |
| `+=`                      | `var += expr`                                           | Penjumlahan aritmatika dan assignment                                    | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Pemisah argumen dan elemen                                        |                |
| `-`                       | `- expr`                                                | Negasi aritmatika                                                   | `Neg`          |
| `-`                       | `expr - expr`                                           | Pengurangan aritmatika                                                | `Sub`          |
| `-=`                      | `var -= expr`                                           | Pengurangan aritmatika dan assignment                                 | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipe balasan (return type) fungsi dan closure                                      |                |
| `.`                       | `expr.ident`                                            | Akses ke field                                                          |                |
| `.`                       | `expr.ident(expr, ...
| `^`                       | `expr ^ expr`                                           | Bitwise exclusive OR                                                  | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | Bitwise exclusive OR dan assignment                                   | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Alternatif di _pattern_                                                  |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Bitwise OR                                                            | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Bitwise OR dan assignment                                             | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | Logical OR (hubungan singkat/short-circuiting)                                           |                |
| `?`                       | `expr?`                                                 | Propagasi error                                                     |                |

### Simbol Non-Operator

Tabel-tabel berikut ini berisi semua simbol yang tidak berfungsi sebagai operator; 
yang artinya, mereka tidak berperilaku seperti pemanggilan fungsi atau method.

Tabel B-2 menunjukkan simbol-simbol yang muncul sendirian dan valid dipakai di 
berbagai macam tempat.

<span class="caption">Tabel B-2: Sintaks Berdiri Sendiri (Stand-Alone Syntax)</span>

| Simbol                                                                 | Penjelasan                                                            |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'

Tabel B-3 nunjukin simbol-simbol yang muncul di dalam konteks penulisan 
jalur (path) melewati hierarki modul menuju sebuah item.

<span class="caption">Tabel B-3: Sintaks Terkait Path</span>



Tabel B-4 nunjukin simbol-simbol yang muncul di konteks pemakaian 
parameter tipe _generic_.

<span class="caption">Tabel B-4: Generics</span>



Tabel B-5 menunjukkan simbol-simbol yang muncul di dalam konteks untuk 
membatasi (constraining) parameter tipe generik dengan _trait bounds_ 
(batasan trait).

<span class="caption">Tabel B-5: Batasan Trait Bound</span>

| Simbol                        | Penjelasan                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `T: U`                        | Parameter generik `T` dibatasi pada tipe-tipe yang mengimplementasikan `U`                                                                              |
| `T: 'a`                       | Tipe generik `T` wajib berumur lebih panjang (outlive) dari _lifetime_ `'a` (artinya tipe tersebut tidak boleh secara transitif menampung referensi yang punya _lifetime_ lebih pendek dari `'a`) |
| `T: 'static`                  | Tipe generik `T` tidak punya referensi pinjaman (borrowed references) selain referensi yang bersifat `'static`                                                                 |
| `'b: 'a`                      | _Lifetime_ generik `'b` wajib berumur lebih panjang (outlive) dari _lifetime_ `'a`                                                                                           |
| `T: ?Sized`                   | Mengizinkan parameter tipe generik untuk bisa berupa tipe yang berukuran dinamis (dynamically sized type)                                                                                |
| `'a + trait`, `trait + trait` | Batasan tipe gabungan (Compound type constraint)                                                                                                                   |

Tabel B-6 menunjukkan simbol-simbol yang muncul di konteks untuk 
memanggil atau mendefinisikan _macros_ dan nentuin atribut buat 
sebuah item.

<span class="caption">Tabel B-6: Macros dan Atribut</span>

| Simbol                                      | Penjelasan        |
| ------------------------------------------- | ------------------ |
| `#[meta]`                                   | Atribut luar (Outer attribute)    |
| `#![meta]`                                  | Atribut dalam (Inner attribute)    |
| `$ident`                                    | Substitusi (penggantian) macro |
| `$ident:kind`                               | Metavariabel macro |
| `$(...)...`                                 | Pengulangan macro (Macro repetition)   |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Pemanggilan (invocation) macro   |

Tabel B-7 menunjukkan simbol-simbol yang berfungsi buat ngebikin 
komentar.

<span class="caption">Tabel B-7: Komentar</span>

| Simbol     | Penjelasan             |
| ---------- | ----------------------- |
| `//`       | Komentar baris            |
| `//!`      | Komentar dokumentasi baris bagian dalam (Inner line doc comment)  |
| `///`      | Komentar dokumentasi baris bagian luar (Outer line doc comment)  |
| `/*...*/`  | Komentar blok (Block comment)           |
| `/*!...*/` | Komentar dokumentasi blok bagian dalam (Inner block doc comment) |
| `/**...*/` | Komentar dokumentasi blok bagian luar (Outer block doc comment) |

Tabel B-8 nunjukin konteks di mana tanda kurung biasa (parentheses) 
dipakai.

<span class="caption">Tabel B-8: Tanda Kurung Biasa (Parentheses)</span>

| Simbol                   | Penjelasan                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()`                     | Tuple kosong (dikenal juga dengan sebutan unit), baik literal maupun tipenya                                               |
| `(expr)`                 | Ekspresi yang dibungkus tanda kurung                                                                    |
| `(expr,)`                | Ekspresi tuple dengan satu elemen tunggal                                                             |
| `(type,)`                | Tipe tuple dengan satu elemen tunggal                                                                   |
| `(expr, ...)`            | Ekspresi tuple                                                                            |
| `(type, ...)`            | Tipe tuple                                                                                  |
| `expr(expr, ...)`        | Ekspresi pemanggilan fungsi; juga dipakai buat menginisialisasi _tuple structs_ dan varian dari _tuple enum_ |

Tabel B-9 nunjukin konteks di mana kurung kurawal (curly brackets) dipakai.

<span class="caption">Tabel B-9: Kurung Kurawal (Curly Brackets)</span>

| Konteks      | Penjelasan      |
| ------------ | ---------------- |
| `{...}`      | Ekspresi blok |
| `Type {...}` | Literal struct   |

Tabel B-10 nunjukin konteks di mana kurung siku (square brackets) 
dipakai.

<span class="caption">Tabel B-10: Kurung Siku (Square Brackets)</span>

| Konteks                                            | Penjelasan                                                                                                                   |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |

