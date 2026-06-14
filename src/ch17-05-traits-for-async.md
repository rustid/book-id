<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## Ngelihat Lebih Dekat pada Traits buat Async

Di sepanjang bab ini, kita sudah memakai trait `Future`, `Stream`, dan 
`StreamExt` dengan berbagai cara. Sejauh ini, kita memang sengaja menghindari 
membahas terlalu dalam soal gimana detail cara kerja mereka atau gimana mereka 
saling berhubungan, yang mana sebenarnya sah-sah saja buat pekerjaan Rust kita 
sehari-hari. Tapi kadang-kadang, kita bakal menjumpai situasi di mana kita butuh 
memahami lebih banyak detail soal trait-trait ini, bersamaan dengan tipe `Pin` 
dan trait `Unpin`. Di bagian ini, kita bakal menggalinya secukupnya saja buat 
membantu kita di skenario-skenario tersebut, sembari tetap membiarkan pembahasan 
yang _bener-bener_ mendalam buat dokumentasi lainnya.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### Trait `Future`

Mari kita mulai dengan melihat lebih dekat gimana cara kerja trait `Future`. 
Beginilah cara Rust mendefinisikannya:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Definisi trait tersebut menyertakan sejumlah tipe baru dan juga beberapa 
sintaks yang belum pernah kita lihat sebelumnya, jadi mari kita bedah 
definisinya satu per satu.

Pertama, _associated type_ `Output` milik `Future` menyatakan hasil akhir yang 
dihasilkan sama _future_ tersebut. Ini serupa dengan _associated type_ `Item` 
pada trait `Iterator`. Kedua, `Future` punya method `poll`, yang menerima sebuah 
referensi `Pin` spesial buat parameter `self`-nya dan sebuah referensi *mutable* 
ke tipe `Context`, serta mengembalikan sebuah `Poll<Self::Output>`. Kita bakal 
ngomongin soal `Pin` dan `Context` sebentar lagi. Buat sekarang, mari fokus sama 
apa yang dikembalikan sama method-nya, yaitu tipe `Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

Tipe `Poll` ini mirip sama `Option`. Ia punya satu varian yang punya nilai, 
`Ready(T)`, dan satu lagi yang nggak punya, `Pending`. Tapi makna `Poll` itu 
jauh beda lho sama `Option`! Varian `Pending` mengindikasikan kalau _future_-nya 
masih punya pekerjaan buat dilakukan, jadi si pemanggil perlu mengeceknya lagi 
nanti. Varian `Ready` mengindikasikan kalau `Future`-nya sudah selesai 
mengerjakan tugasnya dan nilai `T` sudah tersedia.

> Catatan: Jarang sekali ada kebutuhan buat memanggil `poll` secara langsung, 
> tapi kalau kita memang butuh melakukannya, ingatlah kalau pada kebanyakan 
> _futures_, si pemanggil tidak seharusnya memanggil `poll` lagi setelah 
> _future_-nya mengembalikan `Ready`. Banyak _futures_ yang bakal _panic_ kalau 
> di-_poll_ lagi setelah mereka jadi siap. _Futures_ yang aman buat di-_poll_ 
> lagi bakal menyatakannya secara eksplisit di dalam dokumentasinya. Perilaku 
> ini mirip sama gimana `Iterator::next` bekerja.

Pas kita melihat kode yang memakai `await`, Rust sebenarnya mengompilasi kode 
tersebut di balik layar menjadi kode yang memanggil `poll`. Kalau kita melihat 
balik ke Listing 17-4, di mana kita mencetak judul halaman buat satu URL begitu 
dia selesai, Rust mengompilasinya jadi sesuatu yang kira-kira (biarpun nggak 
persis sama) kayak gini:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    },
    Pending => {
        // Terus apa yang ditaruh di sini?
    }
}
```

Apa yang harus kita lakukan pas _future_-nya masih `Pending`? Kita butuh suatu 
cara buat mencoba lagi, lagi, dan lagi, sampai _future_-nya akhirnya siap. 
Dengan kata lain, kita butuh sebuah _loop_:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // lanjut
        }
    }
}
```

Tapi kalau seandainya Rust benar-benar mengompilasi kode yang persis kayak gitu, 
tiap `await` jadinya bakal memblokir—persis kebalikan dari apa yang mau kita 
capai! Sebaliknya, Rust memastikan kalau perulangannya bisa menyerahkan kontrol 
ke sesuatu yang bisa me-_pause_ pekerjaan pada _future_ ini buat mengerjakan 
_futures_ lainnya dan baru kemudian mengecek yang satu ini lagi nanti. Kayak 
yang sudah kita lihat, "sesuatu" itu adalah sebuah *async runtime*, dan 
pekerjaan penjadwalan dan koordinasi ini adalah salah satu tugas utamanya.

Di bagian [“Mengirim Data di Antara Dua Task Memakai Message Passing”][message-passing]<!-- 
ignore -->, kita mendeskripsikan proses menunggu di `rx.recv`. Pemanggilan 
`recv` mengembalikan sebuah _future_, dan me-_await_ _future_ tersebut bakal me-
_poll_-nya. Kita sudah mencatat kalau sebuah _runtime_ bakal me-_pause_ 
_future_-nya sampai dia siap dengan entah `Some(message)` atau `None` pas 
*channel*-nya tutup. Dengan pemahaman kita yang lebih dalam soal trait 
`Future`, dan secara spesifik `Future::poll`, kita bisa melihat gimana cara 
kerjanya. _Runtime_ tahu kalau _future_-nya belum siap pas dia mengembalikan 
`Poll::Pending`. Kebalikannya, _runtime_ tahu kalau _future_-nya *sudah* siap 
dan melanjutkannya pas `poll` mengembalikan `Poll::Ready(Some(message))` atau 
`Poll::Ready(None)`.

Detail persis soal gimana cara _runtime_ melakukan hal tersebut ada di luar 
cakupan buku ini, tapi kuncinya adalah melihat mekanisme dasar dari _futures_: 
sebuah _runtime_ me-_poll_ tiap _future_ yang jadi tanggung jawabnya, lalu 
menidurkan kembali _future_ tersebut pas dia belum siap.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### Tipe `Pin` dan Trait `Unpin`

Dulu di Listing 17-13, kita memakai macro `trpl::join!` buat menunggu tiga 
buah _futures_. Tapi, sudah jadi hal yang umum kalau kita punya sebuah koleksi 
seperti vector yang berisi sejumlah _futures_ yang jumlahnya baru bakal 
diketahui pas _runtime_. Mari kita ubah Listing 17-13 jadi kode di Listing 17-
23 yang menaruh ketiga _futures_ tersebut ke dalam sebuah vector lalu memanggil 
fungsi `trpl::join_all` sebagai gantinya, yang mana kode ini belum bisa di-
compile.

<Listing number="17-23" caption="Menunggu futures di dalam sebuah koleksi"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

Kita menaruh tiap _future_ di dalam sebuah `Box` buat menjadikannya sebagai 
_trait objects_, persis kayak yang kita lakukan di bagian “Mengembalikan Error 
dari `run`” di Bab 12. (Kita bakal membahas _trait objects_ secara detail di 
Bab 18.) Memakai _trait objects_ membiarkan kita memperlakukan tiap _futures_ 
anonim yang dihasilkan sama tipe-tipe ini sebagai tipe yang sama, karena mereka 
semua mengimplementasikan trait `Future`.

Ini mungkin mengejutkan. Lagian, tidak ada satu pun dari blok asinkron tersebut 
yang mengembalikan apa-apa, jadi masing-masing memproduksi sebuah 
`Future<Output = ()>`. Tapi ingat kalau `Future` itu adalah sebuah trait, dan 
_compiler_ membikin sebuah enum yang unik buat tiap blok asinkron, biarpun tipe 
output mereka identik. Sama halnya kayak kita tidak bisa menaruh dua buah 
struct tulisan tangan yang berbeda ke dalam sebuah `Vec`, kita juga tidak bisa 
mencampur aduk enum-enum buatan _compiler_ tersebut.

Terus kita mengoper koleksi _futures_ tersebut ke fungsi `trpl::join_all` lalu 
me-_await_ hasilnya. Tapi, kode ini tidak bisa di-compile; ini dia bagian yang 
relevan dari pesan errornya.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

Catatan di pesan error ini ngasih tahu kita kalau kita seharusnya memakai macro 
`pin!` buat me-_pin_ (mematok) nilai-nilainya, yang artinya menaruh mereka di 
dalam tipe `Pin` yang menjamin kalau nilai-nilainya tidak bakal dipindah-pindah 
di dalam memori. Pesan error-nya bilang kalau proses *pinning* itu diwajibkan 
karena `dyn Future<Output = ()>` perlu mengimplementasikan trait `Unpin` dan 
untuk sekarang dia belum melakukannya.

Fungsi `trpl::join_all` mengembalikan sebuah struct bernama `JoinAll`. Struct 
tersebut sifatnya generik terhadap tipe `F`, yang dibatasi (constrained) buat 
mengimplementasikan trait `Future`. Menunggu sebuah _future_ secara langsung 
dengan `await` bakal me-_pin_ _future_-nya secara implisit. Itulah alasan 
kenapa kita tidak perlu memakai `pin!` di semua tempat di mana kita mau me-
_await_ _futures_.

Tapi, kita ini sedang tidak menunggu sebuah _future_ secara langsung di sini. 
Sebaliknya, kita sedang membangun sebuah _future_ baru, `JoinAll`, dengan cara 
meneruskan sekumpulan _futures_ ke fungsi `join_all`. _Signature_ buat 
`join_all` mewajibkan tipe-tipe dari item di dalam koleksinya buat semuanya 
mengimplementasikan trait `Future`, dan `Box<T>` mengimplementasikan `Future` 
cuma kalau `T` yang ia bungkus itu adalah sebuah _future_ yang 
mengimplementasikan trait `Unpin`.

Duh, sangat banyak yang harus diserap ya! Biar benar-benar paham, mari kita gali 
sedikit lebih jauh soal gimana cara trait `Future` sebenarnya bekerja, terutama 
seputar urusan *pinning*. Mari kita lihat lagi definisi dari trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Method yang wajib ada
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Parameter `cx` dan tipe `Context`-nya adalah kunci gimana sebuah _runtime_ bisa 
benar-benar tahu kapan harus mengecek sembarang _future_ yang diberikan sambil 
tetap bersifat malas. Sekali lagi, detail gimana itu bekerja ada di luar 
cakupan bab ini, dan kita umumnya cuma perlu memikirkan hal ini pas lagi 
menulis implementasi `Future` kustom. Kita bakal fokus saja ke tipe buat 
`self`, karena ini adalah pertama kalinya kita melihat sebuah method di mana 
`self` punya anotasi tipe. Sebuah anotasi tipe buat `self` bekerja kayak 
anotasi tipe buat parameter fungsi lainnya tapi dengan dua perbedaan kunci:

- Ia memberi tahu Rust tipe `self` apa yang harus dimiliki supaya method-nya 
  bisa dipanggil.
- Ia nggak boleh sembarang tipe. Ia dibatasi cuma buat tipe tempat method-nya 
  diimplementasikan, sebuah referensi atau _smart pointer_ ke tipe tersebut, 
  atau sebuah `Pin` yang membungkus referensi ke tipe tersebut.

Kita bakal melihat lebih banyak lagi soal sintaks ini di [Bab 18][ch-18]<!-- 
ignore -->. Buat sekarang, cukup tahu saja kalau kita mau me-_poll_ sebuah 
_future_ buat mengecek apakah dia itu `Pending` atau `Ready(Output)`, kita 
butuh referensi *mutable* yang dibungkus `Pin` ke tipe tersebut.

`Pin` adalah sebuah pembungkus (wrapper) buat tipe-tipe yang mirip pointer kayak 
`&`, `&mut`, `Box`, dan `Rc`. (Secara teknis, `Pin` bekerja dengan tipe-tipe 
yang mengimplementasikan trait `Deref` atau `DerefMut`, tapi ini secara efektif 
ekuivalen dengan bekerja cuma bareng referensi dan _smart pointers_.) `Pin` 
bukanlah sebuah pointer itu sendiri dan ia tidak punya perilaku miliknya 
sendiri kayak `Rc` dan `Arc` yang punya fitur *reference counting*; ia murni 
hanyalah alat yang bisa dipakai _compiler_ buat menegakkan batasan (constraints) 
pada penggunaan pointer.

Mengingat kembali kalau `await` itu diimplementasikan lewat pemanggilan-
pemanggilan ke `poll` mulai bisa menjelaskan pesan error yang kita lihat tadi, 
tapi kan tadi itu dalam konteks `Unpin`, bukan `Pin`. Jadi apa sebenarnya 
hubungan `Pin` dengan `Unpin`, dan kenapa `Future` butuh `self` buat berada di 
dalam tipe `Pin` supaya bisa memanggil `poll`?

Ingat dari bagian awal bab ini kalau serangkaian titik _await_ di dalam sebuah 
_future_ bakal dikompilasi jadi sebuah _state machine_, dan _compiler_ 
mastiin kalau _state machine_ tersebut mengikuti semua aturan normal Rust 
seputar keamanan, termasuk _borrowing_ dan _ownership_. Biar itu bisa jalan, 
Rust melihat data apa saja yang dibutuhkan di antara satu titik _await_ dengan 
titik _await_ berikutnya atau sampai akhir dari blok asinkronnya. Terus dia 
membikin varian yang korespondensi di dalam _state machine_ hasil kompilasinya. 
Tiap varian dapet akses yang dibutuhkannya ke data yang bakal dipakai di bagian 
kode sumber tersebut, entah dengan mengambil kepemilikan dari data tersebut 
atau dengan mendapatkan referensi *mutable* atau *immutable* kepadanya.

Sejauh ini aman: kalau kita bikin kesalahan soal _ownership_ atau referensi di 
sebuah blok asinkron tertentu, si _borrow checker_ bakal kasih tahu kita. Tapi 
pas kita mau memindah-mindahkan _future_ yang berkaitan sama blok tersebut—kayak 
memindahkannya ke dalam sebuah `Vec` buat dioper ke `join_all`—urusannya jadi 
makin rumit.

Pas kita memindahkan sebuah _future_—entah itu dengan memasukkannya ke struktur 
data buat dipakai sebagai iterator bersama `join_all` atau dengan 
mengembalikannya dari sebuah fungsi—itu sebenarnya bermakna memindahkan _state 
machine_ yang Rust bikin buat kita. Dan beda sama mayoritas tipe lain di Rust, 
_futures_ yang Rust bikin buat blok asinkron bisa berakhir punya referensi ke 
dirinya sendiri di dalam field dari varian yang manapun, kayak yang ditunjukkan 
di ilustrasi yang disederhanakan di Gambar 17-4.

<figure>

<img alt="Sebuah tabel dengan satu kolom dan tiga baris yang merepresentasikan sebuah future, fut1, yang punya nilai data 0 dan 1 di dua baris pertamanya dan sebuah panah yang menunjuk dari baris ketiga balik ke baris kedua, melambangkan sebuah referensi internal di dalam future tersebut." src="img/trpl17-04.svg" class="center" />

<figcaption>Gambar 17-4: Sebuah tipe data yang punya referensi ke dirinya sendiri (self-referential)</figcaption>

</figure>

Tapi secara bawaan, objek apa saja yang punya referensi ke dirinya sendiri itu 
sifatnya tidak aman buat dipindahkan, karena referensi itu selalu menunjuk ke 
alamat memori asli dari apa pun yang dirujuknya (lihat Gambar 17-5). Kalau kita 
memindahkan struktur datanya itu sendiri, referensi internal tadi bakal 
ditinggalkan dalam posisi menunjuk ke lokasi yang lama. Padahal, lokasi memori 
tersebut sekarang sudah tidak valid. Di satu sisi, nilainya tidak bakal di-
update pas kita melakukan perubahan ke struktur datanya. Di sisi lain—yang 
lebih penting—komputernya sekarang bebas buat memakai ulang memori tersebut buat 
keperluan lain! Kita bisa berakhir membaca data yang sama sekali tidak ada 
hubungannya nanti.

<figure>

<img alt="Dua tabel, menggambarkan dua futures, fut1 dan fut2, yang masing-masing punya satu kolom dan tiga baris, merepresentasikan hasil dari memindahkan sebuah future keluar dari fut1 ke fut2. Yang pertama, fut1, digelapkan warnanya, dengan tanda tanya di tiap indeksnya, melambangkan memori yang tidak diketahui. Yang kedua, fut2, punya 0 dan 1 di baris pertama dan kedua dan sebuah panah yang menunjuk dari baris ketiganya balik ke baris kedua milik fut1, melambangkan sebuah pointer yang sedang merujuk ke lokasi lama di memori milik future tersebut sebelum ia dipindahkan." src="img/trpl17-05.svg" class="center" />

<figcaption>Gambar 17-5: Hasil yang tidak aman dari memindahkan tipe data yang bersifat self-referential</figcaption>

</figure>

Secara teori, _compiler_ Rust bisa saja mencoba buat meng-update tiap referensi 
ke suatu objek setiap kali objek tersebut dipindahkan, tapi itu bisa menambah 
banyak beban performa, apalagi kalau ada jaring-jaring referensi utuh yang 
perlu di-update. Kalau kita sebaliknya bisa memastikan struktur data yang 
dimaksud itu *tidak pindah-pindah di memori*, kita tidak perlu repot-repot meng-
update referensi apa pun. Nah, itulah gunanya _borrow checker_ milik Rust: di 
dalam kode yang aman, ia mencegah kita dari memindahkan item apa saja yang 
punya referensi aktif yang menunjuk kepadanya.

`Pin` dibangun di atas hal tersebut buat memberikan jaminan persis yang kita 
butuhkan. Pas kita me-_pin_ sebuah nilai dengan membungkus sebuah pointer ke 
nilai tersebut di dalam `Pin`, dia sudah tidak bisa pindah lagi. Jadi, kalau 
kita punya `Pin<Box<SuatuTipe>>`, kita sebenarnya sedang me-_pin_ nilai 
`SuatuTipe` tersebut, *bukan* pointer `Box`-nya. Gambar 17-6 mengilustrasikan 
proses ini.

<figure>

<img alt="Tiga kotak diletakkan berdampingan. Yang pertama dilabeli “Pin”, yang kedua “b1”, dan yang ketiga “pinned”. Di dalam “pinned” ada tabel dilabeli “fut”, dengan satu kolom; ia merepresentasikan sebuah future dengan sel-sel buat tiap bagian dari struktur datanya. Sel pertamanya punya nilai “0”, sel keduanya punya panah yang keluar darinya dan menunjuk ke sel keempat dan terakhir, yang punya nilai “1” di dalamnya, dan sel ketiga punya garis putus-putus dan elipsis buat menandakan mungkin ada bagian lain dari struktur datanya. Secara keseluruhan, tabel “fut” merepresentasikan sebuah future yang bersifat self-referential. Sebuah panah keluar dari kotak berlabel “Pin”, melewati kotak berlabel “b1” dan berakhir di dalam kotak “pinned” di tabel “fut”." src="img/trpl17-06.svg" class="center" />

<figcaption>Gambar 17-6: Me-pin sebuah `Box` yang menunjuk ke tipe future yang bersifat self-referential</figcaption>

</figure>

Kenyataannya, pointer `Box`-nya tetap bisa pindah-pindah dengan bebas. Ingat: 
kita peduli buat memastikan kalau data yang ujung-ujungnya sedang direferensikan 
itu tetap diam di tempat. Kalau sebuah pointer pindah-pindah, *tapi data yang 
ditunjuknya* tetap ada di tempat yang sama, kayak di Gambar 17-7, nggak bakal 
ada masalah potensial. (Sebagai latihan mandiri, coba lihat dokumentasi buat 
tipe-tipe tersebut sekaligus modul `std::pin` dan coba cari tahu gimana cara 
kita melakukan ini pakai `Pin` yang membungkus sebuah `Box`.) Kuncinya adalah 
tipe *self-referential*-nya itu sendiri nggak bisa pindah, karena ia tetap di-
_pin_.

<figure>

<img alt="Empat kotak diletakkan di tiga kolom kasar, identik dengan diagram sebelumnya dengan perubahan di kolom kedua. Sekarang ada dua kotak di kolom kedua, berlabel “b1” dan “b2”, “b1” warnanya digelapkan, dan panah dari “Pin” melewati “b2” bukannya “b1”, menandakan kalau pointernya sudah pindah dari “b1” ke “b2”, tapi data di dalam “pinned” belum pindah." src="img/trpl17-07.svg" class="center" />

<figcaption>Gambar 17-7: Memindahkan sebuah `Box` yang menunjuk ke tipe future yang bersifat self-referential</figcaption>

</figure>

Meskipun begitu, mayoritas tipe itu benar-benar aman buat dipindah-pindahkan, 
biarpun mereka kebetulan ada di balik sebuah pointer `Pin`. Kita cuma perlu 
mikirin soal *pinning* pas item-itemnya punya referensi internal. Nilai-nilai 
primitif kayak angka dan Boolean itu aman karena mereka jelas nggak punya 
referensi internal apa-apa. Begitu juga sama mayoritas tipe yang biasa kita 
pakai di Rust. Kita bisa memindah-mindahkan sebuah `Vec`, misalnya, tanpa perlu 
khawatir. Mengingat apa yang sudah kita lihat sejauh ini, kalau kita punya 
`Pin<Vec<String>>`, kita bakal dipaksa melakukan segalanya lewat API milik `Pin` 
yang aman tapi membatasi, padahal sebuah `Vec<String>` itu selalu aman buat 
dipindahkan kalau tidak ada referensi lain kepadanya. Kita butuh sebuah cara 
buat memberi tahu _compiler_ kalau tidak apa-apa buat memindah-mindahkan item di 
kasus-kasus kayak gini—dan di situlah `Unpin` beraksi.

`Unpin` adalah sebuah _marker trait_, mirip sama trait `Send` dan `Sync` yang 
kita lihat di Bab 16, dan oleh karenanya tidak punya fungsionalitas miliknya 
sendiri. _Marker traits_ eksis cuma buat memberi tahu _compiler_ kalau tipe yang 
mengimplementasikan trait tersebut aman buat dipakai di suatu konteks tertentu. 
`Unpin` menginformasikan ke _compiler_ kalau suatu tipe tertentu *tidak* perlu 
menjunjung tinggi jaminan apa pun soal apakah nilai yang dimaksud bisa 
dipindahkan dengan aman atau tidak.

Sama kayak `Send` dan `Sync`, _compiler_ mengimplementasikan `Unpin` secara 
otomatis buat semua tipe di mana dia bisa membuktikan keamanannya. Kasus 
spesialnya, sekali lagi mirip kayak `Send` dan `Sync`, adalah pas `Unpin` 
*tidak* diimplementasikan buat suatu tipe. Notasi buat hal ini adalah 
`impl !Unpin for SuatuTipe`, di mana `SuatuTipe` adalah nama dari tipe yang 
*memang* perlu menjunjung tinggi jaminan-jaminan tersebut supaya aman kapan pun 
sebuah pointer ke tipe tersebut dipakai di dalam sebuah `Pin`.

Dengan kata lain, ada dua hal yang harus diingat soal hubungan antara `Pin` dan 
`Unpin`. Pertama, `Unpin` adalah kasus yang "normal", dan `!Unpin` adalah kasus 
yang spesial. Kedua, apakah suatu tipe mengimplementasikan `Unpin` atau 
`!Unpin` itu *hanya* berpengaruh pas kita lagi memakai pointer yang di-_pin_ ke 
tipe tersebut kayak `Pin<&mut SuatuTipe>`.

Buat menjadikannya konkret, coba pikirkan soal sebuah `String`: ia punya sebuah 
panjang (length) dan karakter-karakter Unicode yang menyusunnya. Kita bisa 
membungkus sebuah `String` di dalam `Pin`, kayak yang terlihat di Gambar 17-8. 
Tapi, `String` secara otomatis mengimplementasikan `Unpin`, sama halnya kayak 
mayoritas tipe lain di Rust.

<figure>

<img alt="Sebuah kotak berlabel “Pin” di sebelah kiri dengan sebuah panah yang mengarah darinya ke sebuah kotak berlabel “String” di sebelah kanan. Kotak “String” berisi data 5usize, melambangkan panjang dari string-nya, dan huruf-huruf “h”, “e”, “l”, “l”, dan “o” melambangkan karakter dari string “hello” yang disimpan di dalam instance String ini. Sebuah persegi panjang putus-putus mengelilingi kotak “String” dan labelnya, tapi tidak dengan kotak “Pin”." src="img/trpl17-08.svg" class="center" />

<figcaption>Gambar 17-8: Me-pin sebuah `String`; garis putus-putusnya menandakan kalau `String` tersebut mengimplementasikan trait `Unpin` dan oleh karena itu tidak benar-benar ter-pin</figcaption>

</figure>

Hasilnya, kita bisa melakukan hal-hal yang tadinya ilegal kalau seandainya 
`String` mengimplementasikan `!Unpin`, kayak misalnya mengganti satu string 
dengan yang lain di lokasi memori yang sama persis kayak di Gambar 17-9. Ini 
tidak melanggar kontrak dari `Pin`, karena `String` nggak punya referensi 
internal yang bikin dia jadi nggak aman buat dipindah-pindahkan. Itulah 
persisnya kenapa ia mengimplementasikan `Unpin` bukannya `!Unpin`.

<figure>

<img alt="Data string “hello” yang sama dari contoh sebelumnya, sekarang dilabeli “s1” dan warnanya digelapkan. Kotak “Pin” dari contoh sebelumnya sekarang menunjuk ke instance String yang berbeda, yang dilabeli “s2”, bersifat valid, punya panjang 7usize, dan berisi karakter-karakter dari string “goodbye”. s2 juga dikelilingi sama persegi panjang putus-putus karena ia juga mengimplementasikan trait Unpin." src="img/trpl17-09.svg" class="center" />

<figcaption>Gambar 17-9: Mengganti si `String` dengan `String` yang benar-benar berbeda di memori</figcaption>

</figure>

Nah sekarang kita sudah tahu cukup banyak buat memahami error-error yang 
dilaporkan buat pemanggilan `join_all` tadi balik di Listing 17-23. Kita 
awalnya mencoba memindahkan _futures_ hasil produksi blok asinkron ke dalam 
sebuah `Vec<Box<dyn Future<Output = ()>>>`, tapi kayak yang sudah kita lihat, 
_futures_ tersebut mungkin saja punya referensi internal, jadi mereka tidak 
secara otomatis mengimplementasikan `Unpin`. Begitu kita me-_pin_ mereka, kita 
bisa meneruskan tipe `Pin` hasilnya ke dalam `Vec`, dengan rasa percaya diri 
kalau data yang mendasari _futures_ tersebut *tidak* bakal dipindahkan. 
Listing 17-24 menunjukkan cara membetulkan kodenya dengan memanggil macro 
`pin!` di tiap tempat di mana ketiga _futures_ tadi didefinisikan dan 
menyesuaikan tipe dari _trait object_-nya.

<Listing number="17-24" caption="Me-pin futures buat memungkinkan mereka dipindahkan ke dalam vector">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Contoh ini sekarang sudah bisa di-compile dan dijalankan, dan kita bisa 
menambah atau menghapus _futures_ dari vector-nya pas _runtime_ lalu 
menggabungkan mereka semua.

`Pin` dan `Unpin` itu paling banyak kepake pas lagi membangun *lower-level 
libraries*, atau pas kita lagi membangun sebuah _runtime_ itu sendiri, bukannya 
buat kode Rust sehari-hari. Tapi pas kita melihat trait-trait ini muncul di 
dalam pesan error, setidaknya sekarang kita punya gambaran yang lebih oke soal 
gimana cara membetulkan kode kita!

> Catatan: Kombinasi antara `Pin` dan `Unpin` ini memungkinkan implementasi 
> aman dari keseluruhan kelas tipe-tipe kompleks di Rust yang tadinya bakal 
> terbukti menantang gara-gara mereka bersifat *self-referential*. Tipe-tipe 
> yang mewajibkan `Pin` paling sering muncul di Rust asinkron saat ini, tapi 
> sekali-sekali, kita mungkin juga bakal menjumpai mereka di konteks lain.
>
> Detail spesifik soal gimana `Pin` dan `Unpin` bekerja, dan aturan apa saja 
> yang wajib mereka junjung tinggi, sudah dibahas secara ekstensif di dalam 
> dokumentasi API buat `std::pin`, jadi kalau kita tertarik buat belajar lebih 
> lanjut, itu adalah tempat yang bagus buat memulai.
>
> Kalau kita mau memahami gimana cara kerja di balik layarnya secara lebih 
> detail lagi, silakan baca Bab [2][under-the-hood]<!-- ignore --> dan 
> [4][pinning]<!-- ignore --> dari buku 
> [_Asynchronous Programming in Rust_][async-book].

### Trait `Stream`

Sekarang setelah kita punya pemahaman yang lebih dalam soal trait `Future`, 
`Pin`, dan `Unpin`, kita bisa mengalihkan perhatian kita ke trait `Stream`. 
Kayak yang sudah kita pelajari sebelumnya di bab ini, _streams_ itu mirip kayak 
iterator asinkron. Tapi beda sama `Iterator` dan `Future`, `Stream` itu tidak 
punya definisi di dalam *standard library* pada saat tulisan ini dibuat, tapi 
emang *ada* definisi yang sangat umum dari _crate_ `futures` yang dipakai di 
seluruh ekosistem.

Mari kita ulas balik definisi dari trait `Iterator` dan `Future` sebelum 
melihat gimana sebuah trait `Stream` mungkin bakal menggabungkan keduanya. Dari 
`Iterator`, kita dapet ide soal urutan (sequence): method `next`-nya 
menyediakan sebuah `Option<Self::Item>`. Dari `Future`, kita dapet ide soal 
kesiapan seiring berjalannya waktu: method `poll`-nya menyediakan sebuah 
`Poll<Self::Output>`. Buat merepresentasikan serangkaian item yang mulai siap 
seiring waktu, kita mendefinisikan sebuah trait `Stream` yang menggabungkan 
fitur-fitur tersebut:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

Trait `Stream` mendefinisikan sebuah _associated type_ bernama `Item` buat tipe 
dari item-item yang diproduksi sama _stream_-nya. Ini mirip sama `Iterator`, di 
mana itemnya bisa berjumlah nol sampai banyak, dan beda sama `Future`, yang 
mana output-nya selalu satu, biarpun itu cuma tipe unit `()`.

`Stream` juga mendefinisikan sebuah method buat mendapatkan item-item tersebut. 
Kita menamainya `poll_next`, buat memperjelas kalau dia me-_poll_ pakai cara 
yang sama kayak yang dilakukan `Future::poll` dan memproduksi serangkaian item 
pakai cara yang sama kayak yang dilakukan `Iterator::next`. Tipe kembaliannya 
menggabungkan `Poll` dengan `Option`. Tipe luarnya adalah `Poll`, karena dia 
wajib dicek kesiapannya, persis kayak sebuah _future_. Tipe dalamnya adalah 
`Option`, karena dia butuh memberi tanda apakah masih ada pesan lagi, persis 
kayak sebuah iterator.

Sesuatu yang sangat mirip dengan definisi ini kemungkinan besar bakal berakhir 
menjadi bagian dari _standard library_ milik Rust. Sembari menunggu, ia adalah 
bagian dari peralatan milik mayoritas _runtimes_, jadi kita bisa 
mengandalkannya, dan segala hal yang kita bahas selanjutnya secara umum bakal 
tetap berlaku!

Tapi di contoh-contoh yang kita lihat di bagian [“Streams: Futures dalam 
Urutan”][streams]<!-- ignore --> tadi, kita tidak memakai `poll_next` maupun 
`Stream`, melainkan malah memakai `next` dan `StreamExt`. Tentu saja kita *bisa* 
bekerja secara langsung mengikuti API `poll_next` dengan cara menulis tangan 
_state machine_ `Stream` kita sendiri, persis kayak kita juga *bisa* bekerja 
bareng _futures_ secara langsung lewat method `poll`-nya. Tapi memakai `await` 
itu jauh lebih enak, dan trait `StreamExt` menyuplai method `next` supaya kita 
bisa melakukan hal itu:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> Catatan: Definisi asli yang kita pakai sebelumnya di bab ini kelihatan sedikit 
> berbeda dari ini, karena ia mendukung versi Rust yang dulu belum mendukung 
> penggunaan fungsi asinkron di dalam traits. Hasilnya, ia kelihatannya kayak 
> gini:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Tipe `Next` tersebut adalah sebuah `struct` yang mengimplementasikan `Future` 
> dan mengizinkan kita buat menamai _lifetime_ dari referensi ke `self` dengan 
> `Next<'_, Self>`, supaya `await` bisa bekerja bareng method ini.

Trait `StreamExt` juga merupakan rumah dari semua method menarik yang tersedia 
buat dipakai bareng _streams_. `StreamExt` secara otomatis diimplementasikan 
buat tiap tipe yang mengimplementasikan `Stream`, tapi trait-trait ini 
didefinisikan secara terpisah supaya komunitas bisa beriterasi pada API-API 
kemudahan tanpa mengganggu trait dasarnya.

Di versi `StreamExt` yang dipakai di _crate_ `trpl`, trait-nya tidak cuma 
mendefinisikan method `next` tapi juga menyuplai implementasi default dari 
`next` yang secara benar menangani detail-detail pemanggilan 
`Stream::poll_next`. Ini artinya bahkan pas kita perlu menulis tipe data *streaming* 
kita sendiri, kita *cuma* harus mengimplementasikan `Stream`, dan nantinya 
siapa saja yang memakai tipe data kita bisa memakai `StreamExt` beserta method-
methodnya secara otomatis.

Nah, segitu saja pembahasan kita soal detail-detail tingkat rendah pada trait-
trait ini. Sebagai penutup, mari kita pertimbangkan gimana _futures_ (termasuk 
_streams_), _tasks_, dan _threads_ semuanya saling melengkapi!

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-antara-dua-task-memakai-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#program-asinkron-pertama-kita
[any-number-futures]: ch17-03-more-futures.html#bekerja-dengan-sembarang-jumlah-futures
[streams]: ch17-04-streams.html
