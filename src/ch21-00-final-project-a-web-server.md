# Proyek Final: Membangun Web Server Multithreaded

Perjalanan kita panjang banget, tapi akhirnya sampai juga di akhir buku ini. Di
chapter ini, kita bakal bikin satu project terakhir bareng-bareng buat nunjukin
beberapa konsep yang udah kita pelajari di chapter akhir, sekaligus ngerecap
materi sebelumnya juga.

Buat project penutup ini, kita bakal bikin web server yang bakal nampilin
“Hello!” dan tampilannya bakal mirip kayak Gambar 21-1 di browser.

Ini rencana kita buat bikin web servernya:

1. Belajar dikit tentang TCP dan HTTP.
2. Dengerin koneksi TCP di sebuah socket.
3. Parsing beberapa request HTTP sederhana.
4. Bikin response HTTP yang proper.
5. Ningkatin performa server pakai thread pool.

<img alt="Screenshot of a web browser visiting the address 127.0.0.1:8080 displaying a webpage with the text content “Hello! Hi from Rust”" src="img/trpl21-01.png" class="center" style="width: 50%;" />

<span class="caption">Gambar 21-1: Proyek terakhir kita bareng-bareng</span>

Sebelum mulai, ada dua hal penting yang perlu disebutin dulu. Pertama, cara yang
bakal kita pake ini **bukan** cara terbaik buat bikin web server di Rust.
Komunitas Rust udah bikin banyak crate siap pakai yang bisa kamu temuin di
[crates.io](https://crates.io/), dan crate- crate itu punya implementasi web
server dan thread pool yang jauh lebih matang dibanding apa yang bakal kita
bikin di sini. Tapi tujuan chapter ini bukan “cara tercepat bikin web server”,
melainkan biar kamu **belajar konsepnya**. Karena Rust itu bahasa sistem, kita
punya kebebasan memilih level abstraksi yang kita mau, bahkan bisa turun ke
level yang lebih “rendah” dibanding bahasa lain.

Kedua, di sini kita **gak bakal pakai async/await dulu**. Bikin thread pool aja
udah cukup menantang, apalagi kalau ditambah bikin async runtime sendiri! Tapi
nanti kita tetap bakal bahas dikit gimana async/await bisa relevan sama
masalah-masalah yang kita bahas di chapter ini. Pada akhirnya, kayak yang udah
disebut di Chapter 17, banyak async runtime juga sebenernya pakai thread pool
buat ngatur pekerjaannya.

Jadi kita bakal nulis HTTP server dasar dan thread pool-nya secara manual biar
kamu beneran paham konsep dan teknik dasarnya, yang nantinya bakal kepake banget
kalau kamu mau pake crate siap pakai di masa depan.
