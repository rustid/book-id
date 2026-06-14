# Enum dan Pattern Matching

Di bab ini, kita bakal liat _enumerations_, yang juga sering disebut sebagai 
_enums_. Enum ngebolehin kita buat mendefinisikan sebuah tipe dengan menjabarkan 
kemungkinan _variants_-nya (varian). Pertama kita bakal mendefinisikan dan pake 
sebuah enum buat nunjukin gimana enum bisa nyimpen makna barengan sama data. 
Selanjutnya, kita bakal eksplor enum yang kepake sekali, namanya `Option`, yang 
mengekspresikan kalau sebuah nilai itu bisa ada isinya (something) atau nggak ada 
isinya sama sekali (nothing). Terus kita bakal liat gimana _pattern matching_ 
(pencocokan pola) di ekspresi `match` bikin gampang buat ngejalanin kode yang 
beda-beda buat nilai enum yang beda. Terakhir, kita bakal bahas gimana konstruk 
`if let` jadi idiom lain yang nyaman dan ringkas buat handle enum di kode kita.
