# Mengelola Project yang Makin Gede pake Packages, Crates, sama Modules

Seiring kita nulis program yang makin gede, ngatur organisasi kode kita bakal 
makin penting. Dengan ngelempokin fungsionalitas yang terkait dan misahin kode 
dengan fitur yang beda, kita bakal lebih gampang nemuin di mana kode yang 
mengimplementasikan fitur tertentu dan ke mana harus pergi buat ngerubah cara 
kerja sebuah fitur.

Program-program yang udah kita tulis sejauh ini semuanya ada di satu modul di 
dalem satu file. Seiring berkembangnya project, kita harus ngatur kodenya dengan 
mecahnya jadi banyak modul dan terus jadi banyak file. Sebuah package bisa 
isinya banyak binary crates dan opsionalnya satu library crate. Pas sebuah 
package makin gede, kita bisa ngekstrak bagian-bagiannya jadi crates terpisah 
yang bakal jadi dependensi eksternal. Bab ini ngebahas semua teknik ini. Buat 
project yang bener-bener gede yang disusun dari sekumpulan packages yang saling 
berhubungan dan berkembang bareng, Cargo nyediain _workspaces_, yang bakal kita 
bahas di [“Cargo Workspaces”][workspaces] di Bab 14.

Kita juga bakal bahas gimana nyembunyiin (encapsulating) detail implementasi, 
yang ngebolehin kita buat pake ulang (_reuse_) kode di tingkat yang lebih 
tinggi: sekali kita udah mengimplementasikan sebuah operasi, kode lain bisa 
manggil kode kita lewat antarmuka _public_-nya tanpa harus tau gimana detail 
implementasinya jalan. Cara kita nulis kode bakal nentuin bagian mana yang 
_public_ buat dipake kode lain dan bagian mana yang merupakan detail implementasi 
_private_ yang kita punya hak buat ngubahnya kapan aja. Ini cara lain buat 
ngebatesin jumlah detail yang harus kita inget-inget di kepala kita.

Konsep yang terkait adalah _scope_ (ruang lingkup): konteks bersarang (nested) di mana 
kode itu ditulis punya sekumpulan nama yang didefinisikan sebagai "di dalem scope." 
Pas baca, nulis, dan nge-compile kode, programmer dan _compiler_ perlu tau 
apakah nama tertentu di tempat tertentu itu ngerujuk ke variabel, fungsi, struct, 
enum, modul, konstanta, atau item lainnya dan apa makna dari item itu. Kita 
bisa bikin _scopes_ dan ngerubah nama apa aja yang masuk atau keluar dari scope. 
Kita nggak bisa punya dua item dengan nama yang sama di scope yang sama; ada 
_tools_ yang tersedia buat nyelesein konflik nama.

Rust punya sejumlah fitur yang ngebolehin kita ngatur organisasi kode kita, 
termasuk detail apa yang diekspos, detail apa yang _private_, dan nama apa aja 
yang ada di tiap scope di program kita. Fitur-fitur ini, yang kadang secara 
kolektif disebut _module system_ (sistem modul), meliputi:

* **Packages**: Fitur Cargo yang ngebolehin kita buat build, test, dan nge-share crates.
* **Crates**: Struktur pohon modul yang ngasilin library atau file _executable_.
* **Modules dan use**: Ngebolehin kita buat ngontrol organisasi, scope, dan privasi dari _paths_.
* **Paths**: Cara buat namain sebuah item, kayak struct, fungsi, atau modul.

Di bab ini, kita bakal ngebahas semua fitur ini, liat gimana mereka berinteraksi, 
dan ngejelasin gimana cara pakenya buat ngelola scope. Pas selesai nanti, kita 
bakal punya pemahaman yang solid soal sistem modul dan bisa main-main sama 
scope layaknya pro!

[workspaces]: ch14-03-cargo-workspaces.html
