# Menulis Pengujian Otomatis (Automated Tests)

Di esainya pada tahun 1972 yang berjudul “The Humble Programmer,” Edsger W. 
Dijkstra bilang kalau “pengujian program (program testing) bisa jadi cara 
yang sangat efektif buat menunjukkan adanya _bugs_, tapi sangat tidak 
memadai buat menunjukkan kalau _bugs_ itu tidak ada.” Itu bukan berarti 
kita tidak boleh mencoba melakukan pengujian sebanyak mungkin!

Kebenaran (correctness) dalam program kita adalah sejauh mana kode kita 
melakukan apa yang kita inginkan. Rust didesain dengan tingkat kepedulian 
yang tinggi soal kebenaran dari program, tapi kebenaran itu kompleks dan 
tidak mudah dibuktikan. Sistem tipe (type system) Rust menanggung sebagian 
besar beban ini, tapi sistem tipe tidak bisa menangkap semuanya. Oleh 
karena itu, Rust menyertakan dukungan buat menulis pengujian perangkat lunak 
otomatis (_automated software tests_).

Katakanlah kita menulis sebuah fungsi `add_two` yang menambahkan 2 ke angka 
apa pun yang dimasukkan ke dalamnya. _Signature_ dari fungsi ini menerima 
sebuah integer sebagai parameter dan mengembalikan sebuah integer sebagai 
hasil. Saat kita mengimplementasikan dan men-compile fungsi tersebut, Rust 
melakukan semua pengecekan tipe dan _borrow checking_ yang sudah kita 
pelajari sejauh ini untuk memastikan bahwa, misalnya, kita tidak memberikan 
nilai `String` atau referensi yang tidak valid ke fungsi ini. Tapi Rust 
_tidak bisa_ mengecek apakah fungsi ini bakal melakukan persis apa yang kita 
inginkan, yaitu mengembalikan parameternya ditambah 2, bukannya malah 
parameternya ditambah 10 atau dikurang 50! Di sinilah pengujian (_tests_) 
berperan.

Kita bisa menulis pengujian yang menegaskan (assert), misalnya, bahwa saat 
kita memasukkan angka `3` ke fungsi `add_two`, nilai yang dikembalikan 
adalah `5`. Kita bisa menjalankan pengujian-pengujian ini kapan pun kita 
membuat perubahan pada kode kita buat memastikan setiap perilaku benar yang 
sudah ada itu tidak berubah.

Pengujian adalah keterampilan yang kompleks: walaupun kita tidak bisa 
membahas setiap detail soal gimana cara nulis pengujian yang bagus di 
dalam satu bab, di bab ini kita bakal membahas mekanisme dari fasilitas 
pengujian Rust. Kita bakal membahas anotasi dan _macros_ yang tersedia pas 
kita menulis pengujian kita, perilaku _default_ dan opsi-opsi yang 
disediakan buat menjalankan pengujian kita, dan gimana cara mengatur 
pengujian jadi _unit tests_ (pengujian unit) dan _integration tests_ 
(pengujian integrasi).
