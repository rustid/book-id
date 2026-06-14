## Membandingkan Performa: Loops vs. Iterators

Buat menentukan apakah kita sebaiknya memakai _loops_ atau _iterators_, kita 
perlu tahu implementasi mana yang lebih cepat: versi fungsi `search` yang 
memakai _for loop_ eksplisit atau versi yang memakai iterators.

Kita menjalankan sebuah _benchmark_ (pengujian performa) dengan memuat seluruh isi 
buku _The Adventures of Sherlock Holmes_ karya Sir Arthur Conan Doyle ke dalam 
sebuah `String` dan mencari kata _the_ di dalam isinya. Berikut adalah hasil dari 
_benchmark_ pada fungsi `search` versi _for loop_ dan versi iterators:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Kedua implementasi ini punya performa yang mirip! Kita tidak bakal menjelaskan 
kode _benchmark_-nya di sini karena tujuannya bukanlah buat membuktikan kalau 
kedua versi itu persis ekuivalen, tapi buat ngasih gambaran umum soal gimana 
perbandingan performa kedua implementasi ini.

Untuk _benchmark_ yang lebih komprehensif, kita sebaiknya menguji dengan memakai 
berbagai teks dari berbagai ukuran sebagai `contents`, kata yang berbeda-beda dan 
kata dengan panjang yang beda-beda sebagai `query`, dan berbagai jenis variasi 
lainnya. Poin utamanya adalah ini: iterators, meskipun merupakan sebuah 
abstraksi tingkat tinggi (high-level abstraction), bakal di-compile menjadi kode 
yang kurang lebih sama seperti kalau kita menulis sendiri kode tingkat rendah 
(lower-level code)-nya secara manual. Iterators adalah salah satu dari 
_zero-cost abstractions_ (abstraksi tanpa biaya) di Rust, yang maksudnya adalah 
penggunaan abstraksi tersebut tidak menambahkan beban (overhead) apa pun pas 
_runtime_. Ini analog dengan gimana Bjarne Stroustrup, perancang dan 
pengimplementasi asli dari C++, mendefinisikan _zero-overhead_ di 
“Foundations of C++” (2012):

> Secara umum, implementasi C++ mematuhi prinsip _zero-overhead_: Apa yang tidak 
> kita pakai, kita tidak perlu membayarnya. Dan lebih jauh lagi: Apa yang kita 
> pakai, kita tidak bakal bisa nulis kodenya secara manual dengan lebih baik 
> lagi.

Di banyak kasus, kode Rust yang memakai iterators di-compile menjadi kode 
_assembly_ (bahasa rakitan) yang sama persis kayak yang bakal kita tulis pakai 
tangan sendiri. Berbagai optimasi kayak _loop unrolling_ dan menghilangkan 
pengecekan batas (_bounds checking_) pada akses array bakal diterapkan dan membikin 
kode akhirnya jadi sangat efisien. Sekarang karena kita sudah tahu hal ini, 
kita bisa memakai iterators dan _closures_ tanpa rasa takut! Mereka bikin kode 
kelihatan seperti di tingkat yang lebih tinggi (higher level) tapi tidak 
mengenakan hukuman performa (performance penalty) pas _runtime_ karena ngelakuin 
hal tersebut.

## Ringkasan

_Closures_ dan _iterators_ adalah fitur-fitur Rust yang terinspirasi dari 
ide-ide bahasa pemrograman fungsional. Mereka berkontribusi pada kemampuan Rust 
buat mengekspresikan ide-ide tingkat tinggi (high-level ideas) dengan jelas 
sembari mempertahankan performa tingkat rendah (low-level performance). Implementasi 
dari _closures_ dan _iterators_ dirancang sedemikian rupa sehingga performa pas 
_runtime_ tidak terpengaruh. Ini adalah bagian dari tujuan Rust buat berjuang 
menyediakan _zero-cost abstractions_.

Sekarang setelah kita meningkatkan kemampuan ekspresi dari project I/O kita, 
mari kita lihat beberapa fitur `cargo` lainnya yang bakal membantu kita membagikan 
project kita dengan dunia.
