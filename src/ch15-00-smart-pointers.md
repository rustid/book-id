# Smart Pointers

_Pointer_ (penunjuk) adalah sebuah konsep umum buat sebuah variabel yang 
mengandung sebuah alamat di memori. Alamat ini merujuk ke, atau "menunjuk 
ke," data lain. Jenis _pointer_ yang paling umum di Rust adalah sebuah 
referensi, yang udah kita pelajari di Bab 4. Referensi ditandai oleh 
simbol `&` dan meminjam (_borrow_) nilai yang mereka tunjuk. Mereka tidak 
punya kemampuan spesial apa-apa selain cuma merujuk ke data, dan mereka 
tidak punya biaya overhead.

_Smart pointers_ (penunjuk pintar), di sisi lain, adalah struktur data yang 
bertindak seperti sebuah _pointer_ tapi juga punya _metadata_ dan 
kemampuan tambahan. Konsep _smart pointers_ ini bukan cuma ada di Rust: 
_smart pointers_ asalnya dari C++ dan ada di bahasa pemrograman lain juga. 
Rust punya berbagai macam _smart pointers_ yang didefinisikan di _standard 
library_ yang menyediakan fungsionalitas lebih dari sekadar apa yang 
disediakan oleh referensi biasa. Buat mengeksplorasi konsep umumnya, kita 
bakal melihat beberapa contoh _smart pointers_ yang berbeda, termasuk 
tipe _smart pointer_ _reference counting_ (penghitungan referensi). 
_Pointer_ ini memungkinkan kita buat membiarkan sebuah data punya banyak 
pemilik (_owners_) dengan melacak (_keeping track of_) jumlah pemilik yang 
ada dan, saat tidak ada pemilik yang tersisa, membersihkan (cleaning up) 
datanya.

Rust, dengan konsep _ownership_ dan _borrowing_-nya, punya perbedaan tambahan 
antara referensi dan _smart pointers_: walaupun referensi cuma meminjam 
data, di banyak kasus _smart pointers_ _memiliki_ (own) data yang mereka tunjuk.

_Smart pointers_ biasanya diimplementasikan memakai _structs_. Tidak seperti 
_struct_ biasa, _smart pointers_ mengimplementasikan trait `Deref` dan `Drop`. 
Trait `Deref` memungkinkan sebuah instance dari struct _smart pointer_ 
berperilaku seperti sebuah referensi sehingga kita bisa menulis kode yang 
bisa bekerja buat referensi maupun buat _smart pointers_. Trait `Drop` 
memungkinkan kita buat mengkustomisasi kode yang bakal dijalankan saat 
sebuah instance dari _smart pointer_ keluar dari *scope*. Di bab ini, kita 
bakal membahas kedua trait tersebut dan mendemonstrasikan kenapa mereka 
itu penting buat _smart pointers_.

Mengingat kalau _smart pointer_ itu adalah sebuah desain pola yang umum 
dan sering sekali dipakai di Rust, bab ini tidak akan membahas setiap 
_smart pointer_ yang pernah ada. Banyak _libraries_ punya _smart pointers_ 
mereka sendiri, dan kita bahkan bisa bikin _smart pointer_ kita sendiri. 
Kita bakal membahas _smart pointers_ yang paling umum di _standard library_:

- `Box<T>`, buat mengalokasikan nilai di *heap*
- `Rc<T>`, sebuah tipe _reference counting_ yang memungkinkan kepemilikan 
  ganda (multiple ownership)
- `Ref<T>` dan `RefMut<T>`, yang diakses melalui `RefCell<T>`, sebuah tipe yang 
  menerapkan (enforces) aturan _borrowing_ saat *runtime* ketimbang pas 
  *compile time*

Selain itu, kita bakal ngebahas desain pola _interior mutability_ (mutabilitas 
interior) di mana sebuah tipe yang _immutable_ (tidak bisa diubah) mengekspos 
sebuah API buat memutasi nilai internalnya. Kita juga bakal ngebahas 
siklus referensi (reference cycles): gimana mereka bisa membocorkan memori 
(leak memory) dan gimana cara mencegahnya.

Mari kita selami!
