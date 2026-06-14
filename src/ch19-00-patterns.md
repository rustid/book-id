# Pola (Patterns) dan Pencocokan (Matching)

_Patterns_ (pola) adalah sebuah sintaks spesial di Rust buat mencocokkan (matching) 
struktur dari berbagai tipe, baik yang kompleks maupun yang sederhana. Memakai 
_patterns_ bersamaan dengan ekspresi `match` dan konstruk-konstruk lainnya 
ngasih kita kontrol yang lebih banyak terhadap _control flow_ (alur kontrol) 
dari sebuah program. Sebuah _pattern_ terdiri dari beberapa kombinasi dari 
hal-hal berikut ini:

- *Literals* (nilai harfiah)
- Array, enum, struct, atau tuple yang di-_destructure_ (dipecah-pecah)
- Variabel
- _Wildcards_ (kartu liar)
- _Placeholders_ (tempat pengganti)

Beberapa contoh _patterns_ meliputi `x`, `(a, 3)`, dan `Some(Color::Red)`. Di 
dalam konteks di mana _patterns_ itu valid, komponen-komponen ini mendeskripsikan 
bentuk (shape) dari suatu data. Program kita kemudian mencocokkan nilai 
dengan _patterns_ tersebut buat menentukan apakah nilai tersebut punya bentuk 
data yang tepat buat melanjutkan eksekusi potongan kode tertentu.

Buat memakai sebuah _pattern_, kita membandingkannya dengan suatu nilai. 
Kalau _pattern_ tersebut cocok dengan nilainya, kita memakai bagian-bagian dari 
nilai itu di dalam kode kita. Ingat kembali ekspresi `match` di Bab 6 yang 
memakai _patterns_, kayak di contoh mesin penyortir koin. Kalau nilainya cocok 
sama bentuk dari _pattern_-nya, kita bisa memakai potongan-potongan yang udah 
dikasih nama. Kalau tidak cocok, kode yang terkait sama _pattern_ tersebut tidak 
bakal dijalankan.

Bab ini adalah sebuah referensi tentang semua hal yang berkaitan dengan _patterns_. 
Kita bakal membahas tempat-tempat valid di mana kita bisa memakai _patterns_, 
perbedaan antara _refutable_ (bisa dibantah/bisa gagal) dan _irrefutable_ (tidak 
bisa dibantah/pasti sukses) _patterns_, serta berbagai macam sintaks _pattern_ 
yang mungkin bakal kita temui. Di akhir bab ini, kita bakal tahu gimana cara 
memakai _patterns_ buat mengekspresikan banyak konsep dengan cara yang jelas.
