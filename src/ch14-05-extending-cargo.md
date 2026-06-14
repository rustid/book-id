## Memperluas Kemampuan Cargo dengan Custom Commands (Perintah Kustom)

Cargo didesain biar kita bisa memperluas kemampuannya dengan *subcommands* baru 
tanpa harus memodifikasi Cargo itu sendiri. Kalau sebuah _binary_ di dalam 
`$PATH` kita bernama `cargo-sesuatu`, kita bisa menjalankannya seolah-olah itu 
adalah *subcommand* Cargo dengan menjalankan `cargo sesuatu`. *Custom commands* 
kayak gini juga terdaftar saat kita menjalankan `cargo --list`. Kemampuan buat 
memakai `cargo install` untuk menginstal ekstensi-ekstensi lalu menjalankannya 
persis seperti *tools* bawaan Cargo adalah salah satu keuntungan super-nyaman 
dari desain Cargo!

## Ringkasan

Menge-share kode pakai Cargo dan [crates.io](https://crates.io/) adalah bagian 
dari hal yang bikin ekosistem Rust jadi berguna buat berbagai macam tugas. 
_Standard library_ Rust itu kecil dan stabil, tapi _crates_ itu gampang sekali 
buat di-_share_, dipakai, dan ditingkatkan di _timeline_ (garis waktu) yang beda 
dari _timeline_ bahasa Rust itu sendiri. Jangan malu-malu buat nge-share kode 
yang berguna buat kita di [crates.io](https://crates.io/); kemungkinan besar 
kode itu bakal berguna buat orang lain juga!
