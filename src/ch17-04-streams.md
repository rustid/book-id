<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Streams: Futures dalam Urutan

Ingat kembali gimana cara kita memakai _receiver_ buat asinkron _channel_ kita di 
awal bab ini di bagian [“Message Passing”][17-02-messages]<!-- ignore -->. 
Method asinkron `recv` memproduksi serangkaian item seiring berjalannya waktu. 
Ini adalah salah satu bentuk dari pola yang jauh lebih umum yang dikenal sebagai 
_stream_. Banyak konsep yang secara alami bisa direpresentasikan sebagai 
_streams_: item-item yang mulai tersedia di dalam antrean (queue), potongan data 
yang ditarik secara bertahap dari sistem file pas set datanya terlalu besar buat 
memori komputer, atau data yang tiba melalui jaringan seiring waktu. Karena 
_streams_ adalah _futures_, kita bisa memakainya bareng jenis _future_ lainnya 
dan menggabungkannya dengan cara-cara yang menarik. Misalnya, kita bisa 
mengumpulkan (_batch up_) kejadian-kejadian (events) buat menghindari terlalu 
banyak pemanggilan jaringan, mengatur batas waktu (_timeouts_) pada urutan 
operasi yang berjalan lama, atau membatasi (_throttle_) kejadian-kejadian UI 
buat menghindari melakukan pekerjaan yang sia-sia.

Kita sudah pernah melihat serangkaian item di Bab 13, pas kita melihat trait 
`Iterator` di bagian [“Trait `Iterator` dan Method `next`”][iterator-trait]<!-- 
ignore -->, tapi ada dua perbedaan antara iterator dan asinkron _channel 
receiver_. Perbedaan pertama adalah waktu: iterator itu sinkron, sedangkan 
_channel receiver_ itu asinkron. Perbedaan kedua adalah API-nya. Pas bekerja 
langsung dengan `Iterator`, kita memanggil method `next`-nya yang sinkron. 
Khusus buat _stream_ `trpl::Receiver`, kita memanggil method `recv` yang 
asinkron sebagai gantinya. Di luar itu, API-API ini terasa sangat mirip, dan 
kemiripan itu bukan kebetulan kok. Sebuah _stream_ itu kayak bentuk asinkron 
dari iterasi. Meskipun `trpl::Receiver` secara spesifik menunggu buat menerima 
pesan, tapi API _stream_ yang buat tujuan umum itu jauh lebih luas: dia 
menyediakan item berikutnya sama kayak yang dilakukan `Iterator`, tapi secara 
asinkron.

Kemiripan antara iterator dan _streams_ di Rust artinya kita sebenarnya bisa 
membikin sebuah _stream_ dari iterator apa saja. Sama kayak iterator, kita bisa 
bekerja bareng sebuah _stream_ dengan memanggil method `next`-nya lalu me-_await_ 
output-nya, kayak di Listing 17-21, yang mana kode ini belum bisa di-compile.

<Listing number="17-21" caption="Membikin sebuah stream dari sebuah iterator dan mencetak nilai-nilainya" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

Kita mulai dengan sebuah array angka, yang kemudian kita ubah jadi sebuah 
iterator lalu memanggil `map` buat melipatgandakan semua nilainya. Terus kita 
mengubah iterator tersebut jadi sebuah _stream_ menggunakan fungsi 
`trpl::stream_from_iter`. Berikutnya, kita melakukan _loop_ melewati item-item 
di dalam _stream_ tersebut pas mereka tiba memakai _loop_ `while let`.

Sayangnya, pas kita coba menjalankan kodenya, ia tidak bisa di-compile 
melainkan malah melaporkan kalau tidak ada method `next` yang tersedia:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Sesuai penjelasan output ini, alasan error _compiler_-nya adalah karena kita 
butuh trait yang tepat berada di dalam _scope_ supaya bisa memakai method 
`next`. Mengingat pembahasan kita sejauh ini, kita mungkin secara wajar 
berekspektasi kalau trait tersebut adalah `Stream`, tapi sebenarnya ia adalah 
`StreamExt`. Singkatan dari _extension_ (ekstensi), `Ext` adalah pola yang umum 
di komunitas Rust buat memperluas satu trait dengan trait lainnya.

Trait `Stream` mendefinisikan sebuah *low-level interface* yang secara efektif 
menggabungkan trait `Iterator` dan `Future`. `StreamExt` menyuplai sekumpulan 
API tingkat lebih tinggi di atas `Stream`, termasuk method `next` sekaligus 
method pembantu lainnya yang mirip sama yang disediakan oleh trait `Iterator`. 
`Stream` dan `StreamExt` belum menjadi bagian dari _standard library_ milik 
Rust, tapi mayoritas _crates_ di ekosistem memakai definisi yang serupa.

Solusi buat error _compiler_ tadi adalah dengan menambahkan statement `use` buat 
`trpl::StreamExt`, kayak yang ditunjukkan di Listing 17-22.

<Listing number="17-22" caption="Berhasil memakai sebuah iterator sebagai dasar buat sebuah stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

Dengan semua bagian itu disatukan, kode ini berjalan sesuai yang kita mau! 
Bahkan, sekarang karena kita punya `StreamExt` di dalam _scope_, kita bisa 
memakai semua method pembantunya, persis kayak iterator.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
