# Koleksi Umum (Common Collections)

Standard library Rust nyediain sejumlah struktur data yang sangat berguna yang 
disebut _collections_ (koleksi). Kebanyakan tipe data lain merepresentasikan 
satu nilai spesifik, tapi koleksi bisa nampung banyak nilai. Beda sama tipe 
array sama tuple bawaan, data yang ditunjuk sama koleksi ini disimpan di _heap_, 
yang artinya jumlah datanya nggak perlu diketahuin pas _compile time_ dan bisa 
nambah atau berkurang seiring programnya jalan. Tiap jenis koleksi punya 
kemampuan dan biaya (cost) yang beda-beda, dan milih yang paling pas buat 
situasi kita saat itu adalah _skill_ yang bakal kita kembangin seiring berjalannya 
waktu. Di bab ini, kita bakal ngebahas tiga koleksi yang sering sekali dipake di 
program Rust:

- Sebuah _vector_ ngebolehin kita nyimpen sejumlah nilai yang jumlahnya bisa 
  berubah-ubah dan posisinya bersebelahan satu sama lain.
- Sebuah _string_ adalah koleksi dari karakter-karakter. Kita udah sempet nyebut 
  tipe `String` sebelumnya, tapi di bab ini kita bakal bahas lebih mendalam.
- Sebuah _hash map_ ngebolehin kita buat ngaitin (associate) sebuah nilai sama 
  sebuah _key_ tertentu. Ini adalah implementasi spesifik dari struktur data 
  yang lebih umum yang disebut _map_.

Buat belajar soal jenis koleksi lain yang disediain sama standard library, cek 
[dokumentasinya][collections].

Kita bakal bahas gimana cara bikin dan ngubah vectors, strings, dan hash maps, 
serta apa yang bikin masing-masing dari mereka itu spesial.

[collections]: ../std/collections/index.html
