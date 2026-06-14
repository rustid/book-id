# Fitur Pemrograman Berorientasi Objek

<!-- Old headings. Do not remove or links may break. -->

<a id="object-oriented-programming-features-of-rust"></a>

Pemrograman berorientasi objek (_object-oriented programming_, disingkat OOP) 
adalah sebuah cara buat memodelkan program. Objek (objects) sebagai sebuah 
konsep terprogram diperkenalkan pada bahasa pemrograman Simula di tahun 1960-an. 
Objek-objek tersebut memengaruhi arsitektur pemrograman buatan Alan Kay di 
mana objek-objek bisa saling melempar pesan satu sama lain. Buat 
mendeskripsikan arsitektur ini, dia mencetuskan istilah 
_object-oriented programming_ di tahun 1967. Ada banyak definisi yang saling 
bersaing mengenai apa itu OOP, dan berdasarkan beberapa dari definisi-definisi 
tersebut Rust bisa dibilang berorientasi objek, tapi berdasarkan definisi 
lainnya dia bukan. Di bab ini, kita bakal mengeksplorasi ciri-ciri tertentu 
yang umumnya dianggap berorientasi objek dan gimana ciri-ciri itu diterjemahkan 
(translated) ke Rust yang idiomatik. Kita lalu bakal menunjukkan ke kita gimana 
cara mengimplementasikan sebuah desain pola (_design pattern_) berorientasi objek 
di Rust dan membahas _trade-offs_ (kelebihan dan kekurangan) dari melakukannya 
dibandingkan dengan mengimplementasikan solusi yang memanfaatkan kekuatan-
kekuatan dari Rust itu sendiri.
