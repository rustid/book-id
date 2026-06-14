# Berkontribusi (Contributing)

Kami bakal sangat senang dapat bantuan kita! Makasih ya udah peduli sama buku ini.

## Di Mana Harus Mengedit

Semua editan wajib dilakukan di dalam direktori `src`.

Direktori `nostarch` mengandung *snapshots* yang gunanya buat dikirim ke pihak penerbit 
untuk keperluan versi cetak buku (print version). File-file snapshot ini mencerminkan 
apa aja yang udah dikirim dan belum dikirim, makanya file ini cuma bakal di-*update* saat 
kami ngirim hasil editan (edits) ke pihak No Starch. **Tolong jangan submit *pull requests* 
yang ngerubah file-file di dalem direktori `nostarch` ya, PR semacam itu bakal langsung 
kami tutup (closed).**

Kita menggunakan [`rustfmt`][rustfmt] untuk menerapkan gaya pemformatan standar (standard formatting) pada kode Rust di dalam repo ini, dan kita juga menggunakan [`dprint`][dprint] untuk menerapkan pemformatan serupa pada source Markdown sekaligus kode non-Rust lainnya di proyek ini.

[rustfmt]: https://github.com/rust-lang/rustfmt
[dprint]: https://dprint.dev

kita pada umumnya otomatis punya `rustfmt` yang sudah terpasang (installed) andaikata 
kita udah memasang komponen _toolchain_ Rust; tapi kalau misal kita ternyata belum 
punya salinan aplikasi `rustfmt`, kita bisa menambahkan perkakas ini dengan menjalankan 
perintah berikut:

```sh
rustup component add rustfmt
```

Buat menginstal `dprint`, kita bisa menjalankan perintah ini:

```sh
cargo install dprint
```

Atau silakan ikuti [instruksi-instruksi][install-dprint] yang terpajang di situs 
web `dprint`.

[install-dprint]: https://dprint.dev/install/

Buat memformat kode Rust, kita bisa menjalankan perintah `rustfmt <path to file>` 
(jalur ke file), dan untuk memformat file-file lainnya, kita bisa mengoper 
perintah `dprint fmt <path to file>`. Mayoritas teks _editors_ pada umumnya juga udah 
punya dukungan bawaan (*native*) maupun fitur ekstensi tambahan untuk `rustfmt` dan 
sekaligus `dprint`.

## Mengecek Ketersediaan Perbaikan (Checking for Fixes)

Sirkulasi buku ini ngebuntut di atas gerbong jadwal _release trains_ (kereta perilisan) Rust. 
Oleh karena itu, kalau kita kebetulan menjumpai ada sebuah masalah mampang di link 
https://doc.rust-lang.org/stable/book, besar kemungkinan masalah tersebut mungkin emang 
udah lebih dulu diberesin (fixed) dan nangkring dengan status beres di *branch* `main` dari repo 
ini, tapi cuma aja status perbaikannya ini emang belum resmi mbrojol ngelewatin silsilah rute 
nightly -> beta -> stable doang. Jadi mohon dicek terlebih dahulu ya isi dahan *branch* 
`main` di repo ini sebelum keburu ngebuka lemparan _issue_ (laporan).

Ngintip catatan _history_ (riwayat) buat suatu _file_ tertentu secara spesifik juga bisa 
ngasih suguhan ekstra info (more information) seputar ngasih kejelasan entah apakah dan 
gimana rute status sebuah *issue* tersebut udah diberesin atau belum kalau umpamanya 
kita emang lagi penasaran nyari tahu soal itu (trying to figure that out).

Dimohon juga untuk rajin-rajin nyempatin diri buat nyari menelusuri (*search*) tumpukan 
*issues* sama PR (Pull Requests) yang statusnya masih kebuka (_open_) maupun yang udah ketutup 
(_closed_) sebelum keburu nge-_report_ ngelaporin sebuah *issue* baru atau ngebuka PR baru 
(_opening a new PR_).

## Lisensi (Licensing)

Repositori ini bernaung di bawah payung bendera lisensi yang sama persis kayak si Rust itu 
sendiri, yaitu MIT/Apache2. Kita bisa nemuin salinan lengkap draf utuh dari teks kedua lisensi 
tersebut di balik berkas file-file `LICENSE-*` yang terpajang di repo ini.

## Pedoman Perilaku (Code of Conduct)

Project Rust emang punya pakem [sebuah pedoman perilaku](http://rust-lang.org/policies/code-of-conduct) 
(_code of conduct_) yang tegas ngatur perilaku di semua sub-proyek (sub-projects), di mana hal 
tersebut pastinya juga ikutan mencakup repo yang satu ini. Tolong amat dihargai dan direspek ya!

## Ekspektasi (Expectations)

Mengingat bahwa versi draf buku ini itu [dicetak][nostarch] (printed), dan 
mengingat bahwa kami ini punya niat sungguh-sungguh buat sebisa mungkin ngejaga isi konten 
versi _online_ buku ini supaya tetep sinkron sejajar sedekat mungkin bareng isi konten dari versi 
cetaknya andaikata memungkinkan (when possible), proses dan penanganan (_address_) terhadap 
urutan laporan *issue* maupun tumpukan *pull request* yang kita lempar barangkali bisa 
saja memakan tempo kelang waktu pengerjaan yang rasanya agak sedikit kelewat jauh lebih lama (_longer_) 
dari yang sewajarnya biasa kita tungguin.

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

Sejauh ini yang udah-udah, kami biasanya milih menggelar agenda sirkulasi perombakan 
skala raksasa gede-gedean (larger revision) supaya sengaja pas selaras nge-pas bareng 
_coincide_ beririsan seiringan nyambut dengan peluncuran momentum pergantian musim wujud [Rust Editions](https://doc.rust-lang.org/edition-guide/). 
Nah, pada jeda senggang sela waktu yang menyempil di antara musim-musim revisi raksasa tersebut (between those larger revisions), 
kami murni cuma bakal ngelakuin aksi pembenahan ralat error-error (correcting errors) sepele. 
Jadinya misal andaikata emang tumpukan *issue* atau draf *pull request* ajuan kita ini isinya emang 
bukan murni strictly (*strictly*) perihal ngebenerin status murni _error_ (fixing an error), laporannya 
ini besar peluangnya emang mungkin cuma dibiarin aja bakal ngendon diantrikan di pojok bangku penantian 
sampai masa datangnya waktu kelak (next time) pas giliran sirkulasi momentum jadwal kami balik lagi terjun berjibaku ngerjain edisi (_larger revision_) revisi raksasa yang berikutnya: kasaran perhitungannya sih ya (_expect_) 
bisa makan tempo waktu (on the order of) sampe kisaran sekian bilangan deretan bulan atau malah mungkin memakan hitungan deretan angka sekelas tahunan (years). Kami amat sanjung haturin apresiasi banyak-banyak (_Thank you_) 
terima kasih atas asupan ekstra porsi sabarnya kita (your patience)!

## Butuh Bantuan Tambahan (Help wanted)

Seandainya kalau kebetulan (If) kita ini emang lagi pada iseng nyari celah pintu-pintu (*ways to help*) 
buat bisa ikut nimbrung mampir bantu andil (help) tapi tanpa ngewajibin tanpa ada andil pengerahan campur urusan repot ngabisin ngetik apalagi buat (_don't involve_) 
kudu ngolah melahap ngebaca plus nulis bahan bacaan asupan _reading or writing_ dalam takaran kargo porsi raksasa (large amounts), silakan coba tengok kunjungi mampir nengok (_check out_) 
daftar kepingan [*open issues* (laporan bermasalah) yang nangkring dibelay disematin label nama tempelan mampang berjuluk E-help-wanted][help-wanted]. 
Isi borok kerjaannya ini sih aslinya bisa-bisa aja sekadar porsi benerin naskah teks (small fixes to the text), bongkar-bongkar dikit asupan kode Rust, 
ngoprek porsi rupa *frontend code*, ataupun bongkar benahin kode gubahan *shell scripts* yang 
pada gilirannya emang amat bakal ngebantu asupan kelancaran kita buat makin ngebikin rute proses kami kelak bisa (_help us_) ikutan jadi kerasa makin gesit efisien, 
atau senggaknya minimal ngasih bumbu tambahan pencerahan polesan (_enhance the book_) buat makin memoles mantap muka penampilan performa buku ini sedikit 
banyak di salah satu ragam arah dimensinya (in some way)!

[help-wanted]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3AE-help-wanted

## Terjemahan-terjemahan (Translations)

Kami bakal sangat suka dan seneng amat menyambut (love) bantuan asupan tenaga lu dalam ikutan menggarap proyek nerjemahin (translating) buku ini! 
Silakan baca ulasan detail di [Translations] (label terjemahan) andai kata pengen ikutan nimbrung ambil andil nyemplung mutusin nimbrung (_to join in_) bareng barisan gubahan rute pergerakan pengerahan karya pengerjaan usaha tatanan upaya (efforts) tsb 
yang mana sampai waktu ini sekarang ini (currently) emang nyatanya lagi tengah pada gencar asyik berproses berjalan diproses digarap (_in progress_). 
Silakan buat (Open) laporan _issue_ yang baru kalau sudi kepengen (to start) ngewujud mangkas awal proses penggarapan pengerjaan ke edisi draf bahasa yang murni (working on a new language) bener-bener emang baru! 
Kami emang saat ini masih (_waiting on_) pada posisi tengah dalam status tahap setia nyimak sambil diem narik nungguin perihal rupa sokongan [_mdbook support_] resmi 
demi nampung mengakomodasi urusan seputar perlakuan beragam _multiple languages_ (ratusan aneka rupa variasi rupa bahasa sekaligus) ini 
sebelum nantinya kami nekat baru berani bergegas main asal nggabung-ngegabungin campur rute proses peleburan (_merge any in_), 
tapi sungguh pun silakan amat merasa leluasa bebas gembira bebas mardeka bae buat memulainya kok (_feel free to start_)!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5
