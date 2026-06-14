# Error Handling (Penanganan Error)

Error itu adalah kenyataan hidup di _software_, jadi Rust punya sejumlah fitur 
buat nanganin situasi di mana ada sesuatu yang salah. Di banyak kasus, Rust 
mewajibkan kita buat ngakuin kemungkinan adanya error dan ngambil suatu aksi 
sebelum kode kita bisa di-compile. Persyaratan ini bikin program kita jadi lebih 
kuat (robust) dengan mastiin kalau kita bakal nemuin error dan nanganin mereka 
dengan bener sebelum kita nge-deploy kode kita ke _production_!

Rust ngelempokin error jadi dua kategori besar: error _recoverable_ (yang bisa 
dipulihkan) dan _unrecoverable_ (yang nggak bisa dipulihkan). Buat error yang 
_recoverable_, kayak error _file not found_ (file nggak ditemuin), kemungkinan 
besar kita cuma mau ngelaporin masalahnya ke _user_ terus nyoba operasinya lagi. 
Error yang _unrecoverable_ selalu jadi gejala dari _bugs_, kayak nyoba akses 
lokasi yang ngelewatin akhir dari sebuah array, dan makanya kita mau langsung 
ngehentiin programnya.

Kebanyakan bahasa nggak ngebedain dua jenis error ini dan nanganin keduanya 
pake cara yang sama, pake mekanisme kayak _exceptions_ (pengecualian). Rust nggak 
punya _exceptions_. Sebaliknya, dia punya tipe `Result<T, E>` buat error yang 
_recoverable_ dan macro `panic!` yang ngehentiin eksekusi pas program nemu error 
yang _unrecoverable_. Bab ini bakal bahas soal manggil `panic!` dulu terus bahas 
soal balikin nilai `Result<T, E>`. Selain itu, kita bakal eksplor pertimbangan-
pertimbangan pas milih buat nyoba pulih (recover) dari error atau ngehentiin 
eksekusi.
