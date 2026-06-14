## Konkurensi yang Bisa Diperluas (Extensible) dengan Trait `Send` dan `Sync`

<!-- Old link, do not remove -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>

Yang menarik, hampir semua fitur konkurensi yang udah kita bahas sejauh ini di 
bab ini adalah bagian dari _standard library_, bukan bagian dari bahasa Rust itu 
sendiri. Pilihan kita buat menangani konkurensi tidak cuma terbatas pada apa 
yang disediakan oleh bahasa atau _standard library_; kita bisa nulis fitur 
konkurensi kita sendiri atau memakai yang udah ditulis sama orang lain.

Namun, di antara konsep-konsep konkurensi utama yang tertanam (embedded) di 
dalam bahasanya ketimbang di _standard library_ adalah _marker traits_ (trait 
penanda) `std::marker` yaitu `Send` dan `Sync`.

### Mengizinkan Transfer Kepemilikan Antar Threads dengan `Send`

_Marker trait_ `Send` mengindikasikan bahwa kepemilikan (ownership) dari nilai 
dengan tipe yang mengimplementasikan `Send` itu bisa ditransfer antar _threads_. 
Hampir setiap tipe di Rust mengimplementasikan `Send`, tapi ada beberapa 
pengecualian, termasuk `Rc<T>`: tipe ini tidak bisa mengimplementasikan `Send` 
karena kalau kita meng-_clone_ sebuah nilai `Rc<T>` lalu mencoba buat mentransfer 
kepemilikan dari *clone* tersebut ke _thread_ lain, kedua _threads_ bisa aja 
meng-update _reference count_ di waktu yang bersamaan. Atas alasan inilah, 
`Rc<T>` diimplementasikan buat dipakai di situasi *single-threaded* di mana kita 
tidak mau membayar penalti performa (performance penalty) demi keamanan _thread_.

Oleh karena itu, sistem tipe dan *trait bounds* Rust memastikan kalau kita tidak 
bakal bisa secara tidak sengaja mengirim nilai `Rc<T>` melintasi _threads_ 
dengan cara yang tidak aman. Pas kita mencoba melakukan ini di Listing 16-14, 
kita dapat error `` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. 
Pas kita ganti jadi pakai `Arc<T>`, yang mana emang mengimplementasikan `Send`, 
kodenya berhasil di-compile.

Tipe apa pun yang secara utuh disusun (composed entirely) dari tipe-tipe yang 
mengimplementasikan `Send` bakal secara otomatis ditandai sebagai `Send` juga. 
Hampir semua tipe primitif itu `Send`, kecuali untuk _raw pointers_ (pointer 
mentah), yang bakal kita bahas di Bab 20.

### Mengizinkan Akses dari Banyak Threads dengan `Sync`

_Marker trait_ `Sync` mengindikasikan kalau aman buat sebuah tipe yang 
mengimplementasikan `Sync` untuk dirujuk (referenced) dari banyak _threads_. 
Dengan kata lain, tipe `T` apa pun mengimplementasikan `Sync` kalau `&T` 
(referensi _immutable_ ke `T`) mengimplementasikan `Send`, yang berarti 
referensi tersebut bisa dikirim dengan aman ke _thread_ lain. Sama kayak `Send`, 
semua tipe primitif mengimplementasikan `Sync`, dan tipe-tipe yang secara utuh 
disusun dari tipe-tipe yang mengimplementasikan `Sync` juga bakal mengimplementasikan 
`Sync`.

_Smart pointer_ `Rc<T>` juga tidak mengimplementasikan `Sync` dengan alasan yang 
sama kayak kenapa dia tidak mengimplementasikan `Send`. Tipe `RefCell<T>` 
(yang kita bahas di Bab 15) dan keluarga dari tipe `Cell<T>` yang terkait juga 
tidak mengimplementasikan `Sync`. Implementasi dari _borrow checking_ yang 
dilakukan `RefCell<T>` saat _runtime_ itu tidak _thread-safe_ (tidak aman di 
lingkungan banyak utas). _Smart pointer_ `Mutex<T>` mengimplementasikan `Sync` 
dan bisa dipakai buat berbagi akses dengan banyak _threads_, seperti yang kita 
lihat di [“Berbagi `Mutex<T>` di Antara Beberapa Threads”][sharing-a-mutext-between-multiple-threads].

### Mengimplementasikan `Send` dan `Sync` secara Manual Itu Unsafe (Tidak Aman)

Karena tipe yang secara utuh disusun dari tipe-tipe lain yang mengimplementasikan 
trait `Send` dan `Sync` itu juga otomatis mengimplementasikan `Send` dan `Sync`, 
kita tidak perlu mengimplementasikan trait-trait tersebut secara manual. Sebagai 
_marker traits_, mereka bahkan tidak punya method apa pun buat diimplementasikan. 
Mereka cuma berguna buat memaksakan (enforcing) aturan baku (invariants) yang 
berkaitan dengan konkurensi.

Mengimplementasikan trait-trait ini secara manual melibatkan penulisan kode Rust 
yang `unsafe`. Kita bakal ngomongin soal memakai kode Rust yang `unsafe` di 
Bab 20; buat sekarang, informasi pentingnya adalah bahwa membangun tipe konkuren 
baru yang tidak disusun dari bagian-bagian yang `Send` dan `Sync` membutuhkan 
pemikiran yang ekstra hati-hati buat mempertahankan jaminan keamanannya (safety 
guarantees). [“The Rustonomicon”][nomicon] punya lebih banyak informasi soal 
jaminan-jaminan ini dan gimana cara mempertahankannya.

## Ringkasan

Ini bukan terakhir kalinya kita bakal melihat konkurensi di buku ini: bab 
berikutnya berfokus pada pemrograman *async*, dan project di Bab 21 bakal memakai 
konsep-konsep di bab ini di dalam situasi yang lebih realistis ketimbang 
contoh-contoh kecil yang dibahas di sini.

Seperti yang disebutkan sebelumnya, karena cuma sebagian kecil dari cara Rust 
menangani konkurensi itu yang menjadi bagian dari bahasanya, banyak solusi 
konkurensi diimplementasikan dalam bentuk _crates_. Crate-crate ini berevolusi 
lebih cepat daripada _standard library_, jadi pastikan kita mencari secara online 
buat _crates_ yang paling mutakhir (state-of-the-art) buat dipakai di situasi-
situasi *multithreaded*.

_Standard library_ Rust menyediakan _channels_ buat pengiriman pesan (_message 
passing_) dan tipe-tipe _smart pointer_, seperti `Mutex<T>` dan `Arc<T>`, yang 
aman buat dipakai di konteks konkuren. Sistem tipe dan _borrow checker_ memastikan 
kalau kode yang memakai solusi-solusi ini tidak bakal berujung pada *data races* 
atau referensi yang tidak valid. Begitu kita berhasil membikin kode kita bisa 
di-compile, kita bisa bernapas lega karena dia bakal jalan dengan bahagia di atas 
banyak _threads_ tanpa jenis _bugs_ yang susah dilacak kayak yang biasa terjadi 
di bahasa pemrograman lain. Pemrograman konkuren bukan lagi konsep yang perlu 
ditakutkan: maju terus dan bikin program kita konkuren tanpa rasa takut!

[sharing-a-mutext-between-multiple-threads]: ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
