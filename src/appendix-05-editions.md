## Lampiran E - Edisi (Editions)

Di Bab 1, kita sudah melihat kalau perintah `cargo new` itu menambahkan sedikit 
metadata ke dalam file _Cargo.toml_ kita terkait sebuah _edition_ (edisi). Lampiran 
ini membahas tentang apa arti dari hal tersebut!

Bahasa pemrograman Rust dan *compiler*-nya punya siklus rilis yang bergulir 
tiap enam minggu, yang mana berarti para pengguna bakal mendapatkan serangkaian 
fitur baru secara konstan. Bahasa pemrograman lain biasanya merilis perubahan-
perubahan besar dengan frekuensi yang lebih jarang; sedangkan Rust merilis 
pembaruan-pembaruan kecil lebih sering. Setelah sekian lama, semua perubahan 
kecil ini pada akhirnya bakal menumpuk. Tapi kalau dilihat dari satu rilis ke rilis 
yang lain, kadang bisa jadi susah buat menengok ke belakang dan bilang, “Wow, di 
antara Rust 1.10 dan Rust 1.31, Rust ternyata udah berubah sangat banyak!”

Setiap tiga tahun sekali, tim Rust memproduksi sebuah _edition_ (edisi) baru buat 
Rust. Setiap edisi merangkul semua fitur yang sudah berhasil mendarat menjadi 
sebuah paket yang jelas beserta kelengkapan dokumentasi dan peralatan (_tooling_) 
yang sudah dimutakhirkan. Edisi-edisi baru ini dikirim sebagai bagian dari rute 
proses rilis reguler enam mingguan.

Edisi-edisi ini memiliki tujuan yang berbeda-beda bagi kelompok orang yang berbeda:

- Buat para pengguna aktif Rust, sebuah edisi baru menggabungkan perubahan-perubahan 
  tambahan tersebut menjadi sebuah paket yang mudah dipahami.
- Buat orang-orang yang belum memakai Rust, sebuah edisi baru menjadi sinyal 
  kalau ada beberapa kemajuan besar yang sudah terealisasi, yang mana barangkali 
  bakal bikin Rust jadi layak buat dilirik lagi.
- Buat para pengembang yang ikut mengerjakan bahasa Rust, sebuah edisi baru 
  menyediakan sebuah titik kumpul (_rallying point_) yang memacu semangat proyek ini 
  secara keseluruhan.

Saat buku ini sedang ditulis, ada empat edisi Rust yang tersedia: Rust 2015, 
Rust 2018, Rust 2021, dan Rust 2024. Buku ini sendiri sepenuhnya ditulis memakai 
pola dan gaya bahasa (_idioms_) dari edisi Rust 2024.

Kunci `edition` yang ada di dalam _Cargo.toml_ menandakan edisi mana yang seharusnya 
dipakai sama si _compiler_ untuk kode kita. Kalau kuncinya tidak ada, Rust otomatis 
bakal memakai `2015` sebagai nilai edisinya demi alasan kompabilitas ke belakang 
(_backward compatibility_).

Setiap project berhak memilih untuk masuk (opt in) ke sebuah edisi lain selain edisi 
bawaan 2015. Edisi bisa saja mengandung perubahan-perubahan yang tidak kompatibel 
(_incompatible changes_), seperti memasukkan kata kunci (_keyword_) baru yang mana 
mungkin berkonflik dengan nama-nama _identifiers_ yang ada di dalam kode. Namun, 
terkecuali kita secara sadar setuju masuk (opt in) ke perubahan-perubahan tersebut, 
kode kita bakal dijamin terus bisa di-compile dengan mulus sekalipun kita 
meng-_upgrade_ versi dari _compiler_ Rust yang kita pakai.

Semua versi _compiler_ Rust menyokong edisi apa pun yang memang udah eksis sebelum 
versi _compiler_ tersebut dirilis, dan mereka juga sanggup menautkan (_link_) 
_crates_ dari sembarang edisi yang disokong untuk jalan bersama-sama. Perubahan 
edisi itu cuma berdampak pada cara sang _compiler_ mengurai (parses) kode kita pada 
awalnya. Oleh karena itu, kalau kita lagi memakai Rust 2015 dan salah satu _dependency_ 
(dependensi) kita memakai Rust 2018, project kita dijamin bakal tetap bisa sukses 
di-compile dan bisa memakai dependensi tersebut. Dari sisi sebaliknya pun begitu.

Biar lebih jelas: sebagian besar fitur itu bakal tetap tersedia di semua edisi. 
Para pengembang yang memakai edisi Rust yang mana pun bakal terus bisa melihat 
adanya perbaikan seiringan dengan dibuatnya rilis-rilis versi stabil yang baru. 
Namun, di dalam beberapa kasus tertentu, utamanya pas lagi ada _keywords_ baru 
yang ditambahkan, beberapa fitur baru itu barangkali cuma bakal tersedia di edisi-
edisi yang dirilis belakangan. Kita harus berpindah (_switch_) edisi kalau kita pengen 
bisa memanfaatkan fitur-fitur semacam itu.

Untuk detail lebih lanjut, buku [_The Rust Edition Guide_](https://doc.rust-lang.org/stable/edition-guide/) 
adalah buku komplit seputar edisi yang memaparkan semua perbedaan di antara 
tiap-tiap edisi dan menjelaskan gimana caranya meng-_upgrade_ kode kita secara 
otomatis ke edisi baru melalui perintah `cargo fix`.
