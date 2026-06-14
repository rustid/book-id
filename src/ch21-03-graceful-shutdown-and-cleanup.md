## Graceful Shutdown (Mati Secara Anggun) dan Cleanup (Bersih-bersih)

Kode di Listing 21-20 itu benar-benar udah merespons ke _requests_ secara asinkron (asynchronously) melalui 
penggunaan _thread pool_, sesuai sama apa yang kita rencanakan (intended). Tapi kita ngedapetin 
beberapa pesan peringatan (warnings) soal *fields* `workers`, `id`, dan `thread` 
yang nampaknya tidak kita pakai secara langsung, yang mana ngingetin (reminds) kita 
kalau kita ini belum ngelakuin acara bersih-bersih (cleaning up) apa pun. Pas kita 
memakai metode pencet tombol <kbd>ctrl</kbd>-<kbd>C</kbd> yang agak bar-bar (less elegant) 
buat ngehentiin (halt) eksekusi si _main thread_, semua _threads_ lain di dalamnya juga bakal 
dihentiin seketika itu juga (immediately), biarpun mereka posisinya saat itu lagi di tengah-tengah 
nugas (in the middle of) ngelayanin sebuah _request_.

Maka dari itu selanjutnya, kita bakal mengimplementasikan trait `Drop` buat manggil `join` 
pada masing-masing _threads_ di dalam _pool_ tersebut supaya mereka bisa nyelesaiin 
dulu (finish) _requests_ yang lagi mereka kerjain sebelum akhirnya bener-bener ditutup (closing). 
Baru deh setelah itu kita bakal mengimplementasikan sebuah cara buat ngasih tahu para _threads_ 
kalau mereka seharusnya berhenti nerima _requests_ baru lalu mematikan diri (shut down). 
Buat ngelihat aksi langsung dari kode ini, kita bakal memodifikasi server kita supaya dia 
cuma nerima maksimal dua buah _requests_ aja sebelum akhirnya secara anggun (gracefully) 
mematikan semua proses di _thread pool_-nya.

Satu hal yang perlu diperhatiin (to notice) seiringan kita jalan: tidak ada satu pun dari 
proses ini yang bakal berdampak (affects) sama bagian-bagian kode yang tugasnya 
mengeksekusi (executing) _closures_, jadi segala hal yang ada di sini bakal tetep 
sama persis andaikata kita ini lagi memakai sebuah _thread pool_ buat sebuah _runtime_ _async_.

### Mengimplementasikan Trait `Drop` pada `ThreadPool`

Mari kita mulai dengan mengimplementasikan `Drop` pada _thread pool_ kita. 
Pas si _pool_ tersebut di-_drop_, seharusnya semua _threads_ kita ini di-`join` (digabungin) 
buat ngebuktiin dan mastiin (make sure) kalau mereka semua udah beres (finish) ngerjain 
kerjaan mereka. Listing 21-22 nunjukin percobaan perdana (first attempt) buat bikin 
implementasi `Drop` ini; tapi kode ini masih belum bakal jalan lho ya.

<Listing number="21-22" file-name="src/lib.rs" caption="Manggil join ke masing-masing thread pas si thread pool itu keluar dari scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Pertama-tama kita melakukan perulangan (loop) ngelewatin masing-masing elemen di `workers` 
kepunyaan si _thread pool_. Kita memakai referensi `&mut` di sini karena `self` itu sendiri 
kan emang sebuah referensi *mutable* (bisa diubah), dan kita juga perlu bisa ngubah (mutate) 
si variabel `worker`. Buat masing-masing `worker`, kita mencetak sebuah pesan yang ngasih tahu 
kalau *instance* spesifik `Worker` ini tuh lagi bersiap dimatikan (shutting down), dan baru 
setelahnya kita manggil `join` pada nilai _thread_ yang dimiliki *instance* `Worker` 
tersebut. Kalau pemanggilan ke `join` ini ternyata gagal (fails), kita memakai `unwrap` 
buat maksa Rust jadi _panic_ dan masuk ke fase mati secara tidak anggun (ungraceful shutdown).

Ini dia error yang bakal kita dapat pas kita mencoba buat men-compile kode ini:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Pesan error ini ngasih tahu kita kalau kita itu tidak bisa manggil `join` gara-gara kita cuma 
punya sebuah pinjaman _mutable_ (mutable borrow) dari masing-masing `worker` sedangkan si 
`join` ini mengambil hak kepemilikan (ownership) dari argumennya. Buat memecahkan 
(solve) isu ini, kita perlu buat mindahin (_move_) si nilai _thread_ tersebut keluar dari 
*instance* `Worker` yang tadinya nge-hak miliki (_owns_) si `thread` itu supaya si 
`join` bisa narik memakan (consume) _thread_ tersebut. Salah satu cara buat melakukan ini 
adalah dengan ngambil pendekatan (_approach_) yang sama kayak yang pernah kita lakuin di 
Listing 18-15. Kalau si `Worker` ini tadinya nampung `Option<thread::JoinHandle<()>>`, 
kita bisa aja manggil method `take` pada nilai `Option` tersebut buat mindahin (move) nilai 
aslinya dari varian `Some` lalu ninggalin sebuah varian `None` buat menempati posisinya 
(in its place). Dengan kata lain, sebuah `Worker` yang posisinya lagi jalan (running) bakal 
punya sebuah varian `Some` di dalam `thread`, dan pas kita pengen ngebersihin (clean up) 
sebuah `Worker`, kita bakal ngeganti si `Some` jadi `None` sehingga si `Worker` ini 
tidak bakal punya _thread_ buat dijalanin.

Namun masalahnya, momen di mana proses ini itu dibutuhkan (come up) _satu-satunya_ cuma 
terjadi saat kita lagi nge-_drop_ si `Worker` aja kan. Konsekuensinya sebagai ganti rugi 
(in exchange), kita jadinya mesti terus-terusan berurusan (deal with) sama tipe 
`Option<thread::JoinHandle<()>>` di mana pun kita lagi nyoba ngakses `worker.thread`. 
Penulisan bahasa Rust yang idiomatik itu emang cukup sering sekali (quite a bit) memakai 
`Option`, tapi pas kita mulai nemuin diri kita lagi asik ngebungkusin (wrapping) sesuatu yang mana 
padahal kita udah tahu (_know_) nilai itu tuh _selalu_ ada dan eksis (present) ke dalam 
sebuah `Option` cuma murni dijadiin sekadar jalan pintas buat ngakalin (workaround) hal macem 
gini, maka ini merupakan tanda ide yang bagus buat mulai nyari pendekatan alternatif 
(alternative approaches) supaya bisa ngebikin kode kita lebih rapi (cleaner) dan tidak 
gampang rawan error (less error-prone).

Di kasus yang ini, sebenernya ada jalan alternatif yang lebih oke (better alternative): 
yaitu method `Vec::drain`. Method ini menerima sebuah parameter jarak (range parameter) buat 
menentukan _items_ mana aja yang mau dihilangkan dari dalam _vector_ tersebut lalu dia bakal 
mengembalikan (returns) sebuah iterator dari item-item yang ditarik itu. Dengan memberikan 
sintaks *range* `..` kita bakal mengosongkan dan menghilangkan *semua* (*every*) nilai dari _vector_ tersebut.

Jadi kita perlu meng-_update_ implementasi `drop` untuk `ThreadPool` supaya jadi kayak gini:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Kode ini ngeberesin dan nyelesein pesan error _compiler_ tadi tanpa perlu adanya (require) 
perubahan lain lagi ke dalem kode kita. Perhatikan kalau, karena `drop` bisa aja dipanggil pas 
lagi masa-masa terjadi kepanikan (panicking), si `unwrap` ini juga bisa aja tiba-tiba ikutan _panic_ 
yang mana akhirnya nyebabin kepanikan tumpuk ganda (double panic), yang mana bakal langsung 
membikin programnya _crash_ (rusak) dan secara terpaksa ngakhirin semua proses pembersihan 
(cleanup) yang lagi berjalan. Buat ukuran program contoh kayak gini, tindakan _panic_ ini sah-sah aja (fine), 
tapi hal kayak gini ya tidak disarankan (isn't recommended) lho buat kode di level produksi.

### Mengirim Sinyal ke Threads Supaya Berhenti Mendengarkan Jobs (Tugas) Baru

Dengan serangkaian perubahan yang udah kita bikin barusan, kode kita sekarang udah bisa 
sukses di-compile tanpa ngeluarin peringatan apa-apa. Namun, kabar buruknya adalah 
kalau kode ini itu masih belum (doesn't function) berjalan pakai cara yang kita harepin. Kunci permasalahannya ada di 
logika yang ada di dalem _closures_ yang lagi dijalanin (run by) sama _threads_ kepunyaan 
*instances* si `Worker`: saat ini, kita emang udah manggil `join`, tapi hal itu tidak 
bakal bisa nutup mematikan (shut down) si _threads_ ini lho, soalnya mereka itu pada asyik 
berputar (_loop_) nyari kerjaan (_looking for jobs_) buat selama-lamanya (forever). Kalau 
kita coba buat nge-_drop_ si `ThreadPool` kita ini pakai implementasi `drop` kita yang 
sekarang, _main thread_-nya bakal malah ikutan mandek keblokir (block forever), diam selamanya nungguin _thread_ 
yang pertama itu kelar (finish) yang mana tidak bakal bisa kelar.

Buat mbetulin (fix) masalah ini, kita perlu ngebikin satu ubahan (change) di dalam implementasi `drop` buat `ThreadPool` 
dan abis itu juga butuh satu ubahan di dalem perulangan (loop) si `Worker`.

Pertama-tama kita bakal ngubah implementasi `drop` pada `ThreadPool` supaya dia secara eksplisit nge-_drop_ 
si `sender` (pengirim) ini _sebelum_ dia nungguin para _threads_-nya pada kelar jalan. Listing 21-23 
nunjukin rupa ubahan-ubahan ke `ThreadPool` tersebut buat secara eksplisit nge-_drop_ `sender`. Tidak kayak 
yang terjadi di `thread`, di sini kita emang *benar-benar butuh* (_do_ need) memakai 
`Option` biar kita bisa memindahkan (_move_) variabel `sender` keluar dari 
`ThreadPool` dengan memanggil `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Secara eksplisit nge-drop `sender` sebelum nge-join para threads `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Tindakan men-_drop_ (dropping) `sender` ini otomatis bakal nutup (closes) *channel*-nya, yang mana mengindikasikan kalau 
tidak bakal ada lagi pesan (messages) baru yang bakal dikirimin. Saat momen itu benar-benar terjadi, 
semua pemanggilan (_calls_) ke `recv` yang lagi dikerjain sama *instances* si `Worker` di dalam _infinite 
loop_ (perulangan tiada henti)-nya bakal otomatis nge-return sebuah pesan error. Di Listing 21-24, 
kita ngubah bagian perulangan `Worker` supaya dia mau keluar (exit the loop) dengan anggun (gracefully) 
di dalam skenario kayak gitu (in that case), yang berarti _threads_-nya ini akhirnya benar-benar 
bisa kelar (finish) pas implementasi `drop` milik `ThreadPool` manggil `join` 
ke mereka-mereka semua.

<Listing number="21-24" file-name="src/lib.rs" caption="Secara eksplisit mecah kabur (breaking out) dari loop pas fungsi `recv` nge-return sebuah error">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Buat bisa ngelihat sendiri aksi kode (code in action) ini benar-benar berjalan, mari kita modifikasi (modify) `main` 
supaya dia ini cuma nerima batas (accept only) dua *requests* aja sebelum akhirnya dia nutup 
(shutting down) si server-nya dengan anggun (_gracefully_), kayak yang ditunjukin di Listing 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Menutup server setelah ngelayanin dua requests aja dengan cara ngelewatin/keluar dari loop-nya">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

kita pastinya sangat tidak mau ngarep (_wouldn't want_) kalau sebuah web server betulan di dunia 
nyata tiba-tiba mati nutup sendiri (shut down) sesudah cuma ngelayanin dua buah _requests_ doang. 
Tapi kan kode ini itu cuma murni (just demonstrates) dibikin buat pamer ngedemoin kalau kemampuan matikan proses 
secara anggun (graceful shutdown) dan fitur bersih-bersihnya (cleanup) emang benar-benar udah bisa kerja 
lancar beroperasi (in working order).

Method `take` ini aslinya udah didefinisikan secara bawaan di dalem trait `Iterator` 
yang mana dia berfungsi ngebatesin (limits) _iteration_ tersebut supaya paling banter (_at most_) dia 
itu cuma muter buat ngerjain dua item pertama (first two items) doang. Terus, si `ThreadPool` ini jadinya 
bakal terpaksa dihempaskan keluar dari _scope_ (go out of scope) pada momen titik paling 
ujung batas akhir dari fungsi `main`, dan otomatis implementasi fungsi `drop` miliknya bakal segera dioperasikan (will run).

Silakan nyalakan kembali (start) si server ini pakai instruksi `cargo run`, terus cobain bikin tiga (three) buah _requests_ 
masuk ke sana. Pemanggilan _request_ yang ketiga ini harusnya langsung ngebentur pesan error (should error), 
dan terus di dalem layar terminal kita itu kita kudu ngelihat tulisan keluaran (output) 
yang lumayan kelihatan persis mirip-mirip (similar) kayak ini:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

kita emang sangat mungkin ngelihat kalau rupa dari urutan-urutan (ordering) barisan teks _Worker_ ID 
dan tulisan pesannya yang nampil ke layar ini agak beda-beda tatanannya. Kita 
bisa perhatiin dengan jelas (can see) gimana dalemnya proses cara kerja kode ini dari bacaan pesannya 
(messages) tersebut: *Instances* si `Worker` urutan 0 dan 3 ternyata beruntung dapet 
nyaplok ngerjain (_got_) kedua belah *requests* rentetan awal. Terus si server ini mulai ngadat berhenti dan ogah ngerima 
(stopped accepting) adanya sambungan koneksi (_connections_) yang baru pas tepat habis selesai 
ngurus koneksi yang ke-dua tadi, dan si implementasi metode `Drop` yang nempel pada si `ThreadPool` pun 
langsung gas tancap jalan (_starts executing_) duluan padahal faktanya si `Worker` 3 aja malahan belom kelar nyalain (even starts) 
pekerjaannya. Kejadian di-`drop`-nya (dropping) `sender` ini langsung tanpa tedeng aling-aling melepaskan putus (_disconnects_) 
sambungan jembatan tali komunikasi ke semua penjuru *instances* si `Worker` 
lalu lantang nyuruh ngomando ngasih tahu (tells) mereka biar pada mingser bubar barisan nutup layanan (shut down). 
Tiap *instances* `Worker` masing-masing terus nyaut nge-print sebuah *message* saat tali 
mereka terputus dilepas (disconnect), dan abis beres itu semua, si _thread pool_ tadi gantian secara mandiri bergegas manggil 
method `join` lurus dengan satu-satunya niat mau diem duduk setia menunggu (wait) supaya 
setiap barisan satu per satu dari `Worker` _thread_ kelar (_finish_) ngerampungin pekerjaannya.

Coba teliti merhatikan (notice) adanya satu aspek detail perlakuan (_aspect_) yang lumayan kerasa asik unik 
(interesting) di balik serangkaian rentetan rupa spesifik proses eksekusi (execution) satu ini: di mana 
si `ThreadPool` udah keburu kelar nge-_drop_ si `sender`, dan bahkan sebelum sempet ada *instances* 
`Worker` mana aja yang nangkep ngerima kode error (received an error), kita malahan udah ngebikin status program kita ini buat maksa nyoba nggabung 
(join) ke `Worker` urutan 0. Waktu momen itu ya, si `Worker` 0 benar-benar belum ada dapetin sinyal kabar tangkepan error 
apa-apa (not yet gotten an error) yang mana berasal dari tarikan *recv*, jadinya ya benar-benar wajar aja sih kalau si otak pusat jalan *main thread*-nya ini terpaksa mandeg macet diem nge-blok di jalan 
(blocked), ngaso setia sembari nungguin si `Worker` 0 supaya ngerampungin pekerjaannya sampai tuntas (_finish_). 
Eh pas lagi asik nunggu itu (In the meantime), si `Worker` 3 yang udah ketiban ngerima jatah mandat narik dapet 
(received) satu butir kerjaan (_a job_) yang selanjutnya diikuti oleh riwayat sisa-sisa para _threads_ sekaliannya yang ikutan kebagian serentak seragam dapet nerima cipratan pesan eror (error). 
Terus nah pas begitu si `Worker` 0 kelar tuntas ngerjain jatah lapaknya (finished), si otak tengahnya (_main thread_) ini lantas lanjut ngecoba 
nungguin antrean jejeran gerbong serombongan sisa komplotan (the rest) punggawa *instances* `Worker` ini biar pada cepetan (_to finish_) juga ikutan kelar nutup lapaknya. Pada pas 
tibanya waktu masa (at that point) tersebut, eh ternyata semuanya tanpa basa-basi emang udah benar-benar pada 
tuntas cabut ngeluarin diri (exited) berhamburan dari daleman siklus lingkar putaran (loops) rute hidup mereka itu lalu udah berhenti total dari segala aktivitas hidup (_stopped_).

Selamat ya (Congrats)! Kita bener-bener udah sanggup nyampe di titik purna ngerampungin _project_ ini secara utuh (completed); 
kita sekarang ini udah sah punyain (_have_) sepenggal program _web server_ murni kelas cetek mendasar (_basic_) yang aslinya juga 
udah bisa jalan gagah memanfaatkan pengerahan tenaga tatanan sekelompok balok (_thread pool_) demi sanggup ngeresponi secara sigap lalu melayani sambutan balik secara asinkron tak serentak (_asynchronously_). Kita udah sangat terbukti mampu benar-benar 
melancarkan (_perform_) atraksi sebuah tarian purna pemutusan nafas nyawa _graceful shutdown_ terhadap si komponen server tersebut, 
yang mana benar-benar menuntaskan lalu ngebersihin angkat tuntas ngurus sisaan barang bongkaran kotoran riwayat sampah kelakuan (cleans up) riwayat para seantero rentetan _threads_ 
di ruang bilik _pool_ ini.

Nih ditaruh rupa segenap rincian barisan kode final seutuhnya (full code) secara utuh komplit di sini sebagai referensi pedoman (_reference_) ya:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Kita emang nyatanya masih sangat bisa sih buat ngerjain (_could do more_) lebih jauh dan memodifikasi lebih heboh hal-hal lainnya lagi di tempat ini lho! 
Kalau seumpamanya emang hasrat di hati emang kita kepengen (want to) buat sudi terus merutinkan niat maju ngelanjut (continue) nambahin memoles (_enhancing_) gubahan kerangka arsitektur project ini jadi makin keren lagi mantep ke depannya, ini di bawah disediain list segelintir barisan rupa corak deretan bayangan pencerahan curhatan ide (some ideas) racikan kreasi mantap:

- Lengkapin dan imbuhin rentetan sekelumit tambalan isi dari dokumentasi tambahan _(more documentation)_ buat struct `ThreadPool` beserta metode-metode rentetan _method_ `public` miliknya juga ya.
- Coba isengin rakit bikinin sisipin sederet rentetan program tes asinkron (tests) buat nguji kemantapan jalan kelakuan isi _fungsionalitas_ (functionality) jeroan library-nya ini.
- Ganti rombak dan obrak-abrik rentetan pemanggilan wujud rupa `unwrap` yang kelewat asal nabrak paksa njerit ini supaya berubah format menjadi bentukan rentetan taktik kelakuan pengurusan error (_error handling_) yang karakternya terasa sifat jauh lumayan lebih kokoh perkasa pantang rontok (_robust_).
- Manfaatin fitur keberadaan balok struktur `ThreadPool` ini untuk sengaja nyobain ngelaksanain rentetan panggilan _tasks_ rutinitas yang bentukannya ini murni lain di luar formatnya sekadar menugas melayani (serving) serapan _requests_ dari kancah rupa lalu-lalang aliran arus penjelajahan halaman dunia maya (web requests).
- Sisihkan cari lirik bongkar-bongkar iseng nyari (_Find_) bentukan tipe produk paket rak buku kreasi buatan anak lain dari komunitas (thread pool crate) bertebaran liar pating mencelat lepas terpajang bebas melenggang kangkung rilis tebar nongol tersedia gratis pampang mejeng tayang di galeri pustaka etalase repositori perpustakaan lapak kerdus bebas gratis wadah pasar kumpulan [crates.io](https://crates.io/) sono terus pasang dan terapkan ngrakit lalu *implement* rupa susunan formasi wujud web server yang gaya alur kelakuan pola lagaknya miripin sebelas-dua-belas percis plek kayak (_similar_) gubahan hasil rancangan ini dengan murni nekat sekadar sepenuhnya murni berpatokan menunggang makek library produk kreasi bikinan orang laen _the crate instead_ tersebut doang. Barulah (Then) setelah tuntasnya praktek pembuktian tsb. kelar disituasi tersebut silahkan monggo silakan (compare) coba banding-banding rupa rentetan kerangka rute API miliknya library itu sama wujud tingkat derajat kekuatan ketahanan banting kokoh (_robustness_) ketangguhan tahan uji banting tangguhnya punya dia dengan diukur diselidik secara adil membedakan dan aduin lurus dibandingin langsung dipajang sejajar teliti diseret ke hadapan dan disejajarkan ngebentur ke _thread pool_ rakikan tangan pribadi asli mandiri punya yang baru kemaren udah sempet kita *implement* bikin bangun praktekin dari kemaren itu.

## Ringkasan (Summary)

Mantap (Well done)! Kita udah berhasil nyampe tuntas tembus tamat nyentuh ujung akhir pucuk paling buntut sampul ujung (end) dari seri halaman buku ini! Kita semua dari 
diri kami di sini ini secara pribadi kepengen bener sekali pingin ngelempar sepatah kata berterima kasih sedalam-dalamnya buat ngaturin wujud hormat salam 
terima kasih tulus buat panjenengan-panjenengan semua yang udah ikhlas (thank you for) ngabisin waktu ngikut jalan jejer iring bersamai (joining) kita-kita nyusurin merambat rute panjang pelesiran perjalanan (tour) petualangan liburan berburu pesona Rust ini sedari titik garis pinggir batas mula dulu. Kini emang pastinya seutuhnya (now) kita ini udah sepenuhnya dirasa matang dan bener-bener dirasa dipastikan udah sangat siap sanggup sedia (_ready to_) langsung nge-gass terjang 
nerapin ngimplementasiin sendiri kreasi murni orisinil pribadi gagasan wujud gubahan deretan gagasan tatanan project rintisan (_projects_) program rill asli bikinan sendiri punya kita yang berbasis ngusung 
bendera teknologi sistem Rust lalu sekalian sudi dan rela rela buat ngeringanin naruh bantu campur turut menaruh campur tenaga ulur ngasih derma tangan ngebantu nambahin andil campur gotong berderma (_help with_) berkontribusi di gubahan wujud program garapan rintisan _projects_ sumbangsih kreasi racikan buatan anak punggawa temen sejawat *people's projects* (orang-orang) pahlawan tetangga sanak rupa saudara kita di sekitar sana. Harus dimasukin paksa (_Keep in mind_) diranap di simpen lekat ingatan tanam patri ke dalem nalar benak kepala ingat-ingat simpan resapin ya 
di ingatan (mind) dalem bawah sadar otak memori paten benak kepalamu ini andaikata (that) ini tuh sesungguhnya nyata terang 
bersinar nyata terang benderang niscaya terbentang terbentang mekar membentang subur emang masih menyisa (there is a) terhuni rupa sepetak jajaran luas 
gerombolan wadah kerumunan kelompok perhimpunan _welcoming community_ (komunitas yang sangat terbuka nyambut ramah dan sedia hangat menjamu rupa tangan senyum terbuka gembira lebar dada tulus nyambut seneng asri hangat peluk) persaudaraan ikatan erat rupa serumpun bangsa perkumpulan perserikatan komplotan jejaring _other Rustaceans_ (kalangan para pengabdi programmer pemuja aliran seiman setia pejuang pendekar kode penyuka aliran kepercayaan kasta Rust ksatria lain-lainnya sesama punggawa yang sealiran) lainnya di pelosok buana pelosok sana luar sekitar pojokan pelosok dunia sana ini yang tak ada bosannya tak kan bakal pernah nyerah sudi ikhlas niscaya aslinya emang _would love to_ (pada seneng rela hati bakal cinta mati gemar nyenengin cinta ngebet pengen gemar dan hobi pake cinta bahagia hepi benar-benar girang tulus girang rela asyik seru sumringah demen kepingin riang berhati tulus bakal doyan bakal suka bakal sayang dan sangat girang sekali teramat rindu sangat ingin mau rela) ikhlas turun nolong bantu menyambut nyuapin nyuapin kasih (_help you with_) membina mandu memayungi mendampingin mandorin nyodor tangan mbopong mbimbing nolong kita-kita pada ikhlas terjun dan andil nimbrung bantuin mberesin dan nyikat beresin mengurus sedia ngadepin dan ngeberesin (_any challenges_) seberapa parah gawatnya semua aneka segudang rupa kendala aral ragam himpitan halang aneka lika rupa jurang duri kesulitan ujian badai cobaan halangan ragam perkara problema jerat tikungan batu cadas tantangan rintang terjalan problem himpitan perkara ganjalan segenap segala seberat sedempet serepot rupa kendala seisi sebentang onak seisi segala segenap (any) ragam masalah kendala benturan apa jua sekalipun belaka apa pun bentuknya rupa aja yang di jalan nanti kebetulan emang rupa nyata riil fakta nyatanya _you encounter_ (kita tabrak kepentok dapati tabrak papasi bersua sandung hantam ketatap kebentur derita alami jumpa dapet tebas terjang lalui pergokin tempuh alami libas gilas temui tabrak lewati hantam rasakan langgar cicipin terjang tatap derita lintasi rasai cicip rasain hadapi cicip deritain hadapi hadapi) ke-tubruk kepentok bersua melintang terlintas nyandung hadap temui hadapi sandung kita di belantara sepanjang masa-masa pelayaran peruntungan jalan pendakian jalur petualang perjalanan lintas jalur rute jelajah ngeluyur (journey) pengembaraan karir kiprah rekam pelesiran perjalanan _Rust_ (kancah rute perjalanan karir Rust) asuhan panjenengan kita seiring maju langkah laju menapaki jejak berjejak kita maju panjang menjejak merentas jauh berjalan ini di waktu ke detik harinya nanti menjejak (your Rust journey).
