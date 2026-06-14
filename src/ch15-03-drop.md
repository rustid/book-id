## Menjalankan Kode saat Proses Pembersihan (Cleanup) dengan Trait `Drop`

Trait kedua yang penting buat pola _smart pointer_ adalah `Drop`, yang 
membiarkan kita mengkustomisasi apa yang terjadi saat sebuah nilai bakal 
keluar dari _scope_. Kita bisa menyediakan sebuah implementasi buat trait 
`Drop` di tipe apa pun, dan kode tersebut bisa dipakai buat melepaskan 
(release) _resources_ kayak file atau koneksi jaringan.

Kita ngenalin `Drop` di konteks _smart pointers_ karena fungsionalitas dari 
trait `Drop` hampir selalu dipakai pas kita mengimplementasikan sebuah _smart 
pointer_. Misalnya, saat sebuah `Box<T>` di-_drop_, dia bakal men-_deallocate_ 
(melepaskan) ruang di _heap_ yang ditunjuk sama _box_ tersebut.

Di beberapa bahasa, buat tipe-tipe tertentu, si programmer harus memanggil kode 
buat membebaskan memori atau _resources_ setiap kali mereka kelar memakai 
sebuah instance dari tipe-tipe tersebut. Contohnya termasuk _file handles_ 
(pegangan file), *sockets*, sama *locks*. Kalau mereka lupa, sistem bisa jadi 
_overloaded_ dan _crash_. Di Rust, kita bisa menentukan bahwa sekumpulan 
kode tertentu bakal dijalankan setiap kali sebuah nilai keluar dari _scope_, 
dan _compiler_ bakal menyisipkan (insert) kode ini secara otomatis. Hasilnya, 
kita tidak perlu repot-repot menaruh kode _cleanup_ (pembersihan) di mana-mana 
di dalam program setiap kali sebuah instance dari suatu tipe sudah selesai 
dipakai—dan kita tetap tidak bakal membocorkan (leak) _resources_!

Kita menentukan kode yang bakal jalan saat sebuah nilai keluar dari _scope_ 
dengan mengimplementasikan trait `Drop`. Trait `Drop` mewajibkan kita buat 
mengimplementasikan satu method bernama `drop` yang menerima referensi 
_mutable_ ke `self`. Buat melihat kapan Rust memanggil `drop`, mari kita 
mengimplementasikan `drop` dengan *statements* `println!` dulu buat sekarang.

Listing 15-14 menunjukkan sebuah struct `CustomSmartPointer` yang satu-satunya 
fungsionalitas kustom yang dia punya adalah dia bakal mencetak `Dropping 
CustomSmartPointer!` saat instance-nya keluar dari _scope_, buat menunjukkan 
kapan Rust menjalankan method `drop`.

<Listing number="15-14" file-name="src/main.rs" caption="Sebuah struct `CustomSmartPointer` yang mengimplementasikan trait `Drop` di mana kita bakal menaruh kode _cleanup_ kita">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

Trait `Drop` sudah dimasukkan ke dalam _prelude_, jadi kita tidak perlu membawa 
trait itu ke dalam _scope_. Kita mengimplementasikan trait `Drop` pada 
`CustomSmartPointer` dan menyediakan sebuah implementasi buat method `drop` yang 
memanggil `println!`. _Body_ dari method `drop` adalah tempat di mana kita 
bakal menaruh logika apa pun yang mau kita jalankan pas sebuah instance dari 
tipe kita keluar dari _scope_. Kita mencetak sedikit teks di sini buat 
mendemonstrasikan secara visual kapan Rust bakal memanggil `drop`.

Di `main`, kita bikin dua instance dari `CustomSmartPointer` lalu mencetak 
`CustomSmartPointers created`. Di akhir `main`, instance-instance dari 
`CustomSmartPointer` kita bakal keluar dari _scope_, dan Rust bakal memanggil 
kode yang kita taruh di method `drop`, yang mana mencetak pesan terakhir kita. 
Perhatikan bahwa kita tidak perlu memanggil method `drop` secara eksplisit.

Pas kita menjalankan program ini, kita bakal melihat output berikut:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust secara otomatis memanggil `drop` buat kita pas instance-instance kita keluar 
dari _scope_, dan menjalankan kode yang sudah kita tentukan. Variabel-variabel 
di-_drop_ dalam urutan yang berlawanan dari pembuatannya, jadi `d` di-_drop_ 
sebelum `c`. Contoh ini tujuannya adalah buat ngasih kita panduan visual soal 
gimana method `drop` itu bekerja; biasanya kita bakal menentukan kode _cleanup_ 
yang dibutuhkan sama tipe kita, ketimbang cuma pesan *print* biasa.

<!-- Old link, do not remove -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

Sayangnya, menonaktifkan fungsionalitas `drop` otomatis ini tidak 
gampang. Menonaktifkan `drop` biasanya juga tidak diperlukan; keseluruhan 
poin dari trait `Drop` adalah supaya hal itu diurus secara otomatis. Tapi, 
terkadang, kita mungkin pengen membersihkan (clean up) sebuah nilai lebih 
awal. Salah satu contohnya adalah pas lagi memakai _smart pointers_ yang 
mengelola *locks*: kita mungkin pengen memaksa method `drop` yang bakal 
melepaskan *lock* itu agar kode lain di _scope_ yang sama bisa mendapatkan 
*lock* tersebut. Rust tidak membiarkan kita buat memanggil method `drop` 
dari trait `Drop` secara manual; sebaliknya, kita harus memanggil fungsi 
`std::mem::drop` yang disediakan sama _standard library_ kalau kita mau 
memaksa sebuah nilai buat di-_drop_ sebelum akhir dari _scope_-nya.

Kalau kita mencoba memanggil method `drop` dari trait `Drop` secara manual 
dengan memodifikasi fungsi `main` dari Listing 15-14, seperti yang ditunjukkan 
di Listing 15-15, kita bakal dapat error _compiler_.

<Listing number="15-15" file-name="src/main.rs" caption="Mencoba memanggil method `drop` dari trait `Drop` secara manual untuk _cleanup_ lebih awal">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

Pas kita nyoba men-compile kode ini, kita bakal dapat error ini:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Pesan error ini menyatakan kalau kita tidak diizinkan buat memanggil `drop` 
secara eksplisit. Pesan error ini memakai istilah *destructor* (destruktor), 
yang mana adalah istilah pemrograman umum buat sebuah fungsi yang membersihkan 
sebuah instance. Sebuah *destructor* analog (mirip) dengan sebuah *constructor* 
(konstruktor), yang membikin sebuah instance. Fungsi `drop` di Rust adalah 
salah satu destruktor tertentu.

Rust tidak membiarkan kita memanggil `drop` secara eksplisit karena Rust tetap 
bakal secara otomatis memanggil `drop` pada nilainya di akhir dari `main`. Hal 
ini bakal menyebabkan error *double free* karena Rust bakal mencoba membersihkan 
nilai yang sama dua kali.

Kita tidak bisa menonaktifkan penyisipan `drop` secara otomatis pas sebuah nilai 
keluar dari _scope_, dan kita tidak bisa memanggil method `drop` secara eksplisit. 
Jadi, kalau kita butuh memaksa sebuah nilai buat dibersihkan lebih awal, kita 
memakai fungsi `std::mem::drop`.

Fungsi `std::mem::drop` berbeda dari method `drop` yang ada di trait `Drop`. 
Kita memanggilnya dengan meneruskan nilai yang mau kita paksa untuk di-_drop_ 
sebagai argumennya. Fungsi ini ada di dalam _prelude_, jadi kita bisa memodifikasi 
`main` di Listing 15-15 buat memanggil fungsi `drop` tersebut, seperti yang 
ditunjukkan di Listing 15-16.

<Listing number="15-16" file-name="src/main.rs" caption="Memanggil `std::mem::drop` buat me-_drop_ secara eksplisit sebuah nilai sebelum dia keluar dari _scope_">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Menjalankan kode ini bakal mencetak yang berikut ini:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Teks ``Dropping CustomSmartPointer with data `some data`!`` dicetak di antara 
teks `CustomSmartPointer created.` dan `CustomSmartPointer dropped
before the end of main.`, menunjukkan kalau kode method `drop` dipanggil 
buat me-_drop_ `c` pada titik tersebut.

Kita bisa memakai kode yang ditentukan di dalam implementasi trait `Drop` 
pakai berbagai cara buat membikin _cleanup_ jadi nyaman dan aman: misalnya, 
kita bisa memakainya buat membikin *memory allocator* kita sendiri! Dengan trait 
`Drop` dan sistem _ownership_ di Rust, kita tidak perlu repot nginget-nginget 
buat _cleanup_ karena Rust ngelakuin itu secara otomatis.

Kita juga tidak perlu khawatir soal masalah-masalah yang timbul dari secara 
tidak sengaja membersihkan nilai yang masih dipakai: sistem _ownership_ yang 
memastikan kalau referensi bakal selalu valid juga memastikan kalau `drop` 
cuma bakal dipanggil satu kali pas nilainya memang sudah tidak dipakai lagi.

Sekarang setelah kita meneliti `Box<T>` dan beberapa karakteristik dari 
_smart pointers_, mari kita lihat beberapa _smart pointers_ lain yang didefinisikan 
di _standard library_.
