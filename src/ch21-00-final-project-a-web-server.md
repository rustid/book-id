# Project Akhir: Membikin Sebuah Multithreaded Web Server

Udah jadi perjalanan panjang ya, tapi akhirnya kita nyampe juga di akhir 
buku ini. Di bab ini, kita bakal ngebikin satu project lagi bareng-bareng 
buat mendemonstrasikan beberapa konsep yang udah kita bahas di bab-bab akhir, 
sekaligus juga ngeringkas (recap) pelajaran-pelajaran yang ada sebelumnya.

Buat project akhir kita, kita bakal membikin sebuah web server yang ngomong 
“hello” dan kelihatannya kayak Gambar 21-1 di web browser.

Berikut ini adalah rencana kita buat ngebangun web server-nya:

1. Belajar sedikit soal TCP dan HTTP.
2. Mendengarkan (listen) koneksi-koneksi TCP di dalam sebuah *socket*.
3. Mem-_parse_ sejumlah kecil _requests_ (permintaan) HTTP.
4. Membikin sebuah _response_ (respons/balasan) HTTP yang layak.
5. Ningkatin *throughput* (kemampuan ngelayanin banyak permintaan) dari 
   server kita dengan sebuah _thread pool_.

![hello dari rust](img/trpl21-01.png)

<span class="caption">Gambar 21-1: Project akhir yang kita buat bareng-bareng</span>

Sebelum kita mulai, kita harus nyebutin dua hal detail. Pertama, metode yang 
bakal kita pakai ini bukanlah cara terbaik buat ngebangun sebuah web server 
pakai Rust. Anggota komunitas udah memublikasikan beberapa _crates_ yang siap 
buat dipakai di _production_ (production-ready) yang tersedia di 
[crates.io](https://crates.io/) yang mana menyediakan implementasi web server 
dan _thread pool_ yang jauh lebih lengkap ketimbang apa yang bakal kita bikin 
ini. Namun, niat kita di bab ini adalah ngebantu kita buat belajar, bukannya 
ngambil jalan yang gampang. Karena Rust itu adalah bahasa pemrograman sistem 
(systems programming language), kita bisa milih tingkat abstraksi yang pengen 
kita kerjain dan kita bisa turun ke tingkat (level) yang lebih rendah ketimbang 
apa yang mungkin atau praktis buat dilakukan di bahasa pemrograman lainnya.

Kedua, kita tidak bakal memakai _async_ dan _await_ di sini. Ngebangun sebuah 
_thread pool_ aja itu udah tantangan yang lumayan gede sendirian, jadi kita tidak 
perlu nambah-nambahin keruwetan dengan ngebangun sebuah _runtime_ _async_ 
sekalian! Walaupun begitu, kita bakal ngasih catetan gimana sih _async_ dan _await_ 
mungkin bisa diterapin ke beberapa permasalahan yang bakal kita temui di bab ini. 
Pada akhirnya, seperti yang udah kita sebutin balik di Bab 17, banyak _runtimes_ 
_async_ yang juga memakai _thread pools_ buat mengelola kerjaan mereka kok.

Oleh karena itu, kita bakal menulis sebuah HTTP server dasar dan _thread pool_ 
secara manual supaya kita bisa belajar ide-ide umum dan teknik-teknik di balik 
_crates_ yang mana mungkin bakal kita pakai di masa depan.
