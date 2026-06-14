# Konkurensi Tanpa Rasa Takut (Fearless Concurrency)

Menangani pemrograman konkuren secara aman dan efisien adalah salah satu 
tujuan utama Rust lainnya. _Concurrent programming_ (pemrograman konkuren), 
di mana berbagai bagian dari sebuah program dieksekusi secara independen, dan 
_parallel programming_ (pemrograman paralel), di mana berbagai bagian dari sebuah 
program dieksekusi di saat yang bersamaan, menjadi makin penting seiring makin 
banyaknya komputer yang memanfaatkan prosesor multi-inti (multiple processors). 
Secara historis, pemrograman di dalam konteks ini selalu sulit dan rawan error. 
Rust berharap bisa mengubah hal tersebut.

Awalnya, tim Rust berpikir bahwa memastikan keamanan memori (memory safety) 
dan mencegah masalah konkurensi adalah dua tantangan terpisah yang harus 
diselesaikan dengan metode yang berbeda. Seiring berjalannya waktu, tim 
menemukan bahwa sistem kepemilikan (ownership) dan sistem tipe adalah 
serangkaian alat yang sangat kuat buat membantu mengelola keamanan memori 
_dan_ masalah konkurensi! Dengan memanfaatkan _ownership_ dan pengecekan tipe 
(type checking), banyak error konkurensi di Rust bakal menjadi error 
_compile-time_ (saat kompilasi) ketimbang error _runtime_. Oleh karena itu, 
ketimbang membiarkan kita menghabiskan banyak waktu mencoba mereka ulang 
kondisi persis di mana sebuah _bug_ konkurensi _runtime_ terjadi, kode yang 
salah bakal menolak untuk di-compile dan menyajikan error yang menjelaskan 
masalahnya. Sebagai hasilnya, kita bisa memperbaiki kode kita saat kita sedang 
mengerjakannya, bukannya nanti setelah kodenya dikirim ke tahap produksi. 
Kita menjuluki aspek dari Rust ini sebagai _fearless concurrency_ (konkurensi 
tanpa rasa takut). _Fearless concurrency_ memungkinkan kita buat menulis kode 
yang bebas dari _bugs_ yang tersembunyi (subtle bugs) dan mudah buat di-_refactor_ 
tanpa memunculkan _bugs_ baru.

> Catatan: Demi kesederhanaan, kita bakal menyebut banyak dari masalah-masalah 
> ini sebagai _konkuren_ ketimbang lebih presisi dengan bilang _konkuren dan/atau 
> paralel_. Buat bab ini, tolong substitusikan dalam hati _konkuren dan/atau 
> paralel_ kapan pun kita memakai kata _konkuren_. Di bab selanjutnya, di mana 
> perbedaannya lebih penting, kita bakal lebih spesifik.

Banyak bahasa pemrograman bersifat dogmatis soal solusi yang mereka tawarkan buat 
menangani masalah konkuren. Misalnya, Erlang punya fungsionalitas yang elegan buat 
konkurensi _message-passing_ (pengiriman pesan) tapi cuma punya cara yang samar-samar 
(obscure) buat membagikan _state_ (keadaan) di antara _threads_. Mendukung hanya 
sebagian dari solusi yang memungkinkan adalah strategi yang masuk akal buat 
bahasa tingkat tinggi (higher-level languages) karena bahasa tingkat tinggi 
menjanjikan manfaat dari mengorbankan sedikit kontrol buat mendapatkan abstraksi. 
Namun, bahasa tingkat rendah (lower-level languages) diharapkan bisa menyediakan 
solusi dengan performa terbaik di situasi apa pun dan punya lebih sedikit abstraksi 
di atas perangkat kerasnya. Oleh karena itu, Rust menawarkan berbagai alat buat 
memodelkan masalah dengan cara apa pun yang cocok buat situasi dan kebutuhan kita.

Berikut adalah topik-topik yang bakal kita bahas di bab ini:

- Cara membikin _threads_ (utas) buat menjalankan beberapa potong kode di saat 
  yang bersamaan
- Konkurensi _message-passing_, di mana saluran (channels) mengirim pesan antar 
  _threads_
- Konkurensi _shared-state_, di mana banyak _threads_ punya akses ke sekumpulan data
- Trait `Sync` dan `Send`, yang memperluas jaminan konkurensi Rust ke tipe-tipe 
  yang didefinisikan sama pengguna (user-defined types) serta tipe-tipe yang 
  disediakan oleh _standard library_
