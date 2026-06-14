# Dasar-dasar Pemrograman Asinkron: Async, Await, Futures, dan Streams

Banyak operasi yang kita suruh komputer buat lakukan bisa memakan waktu agak lama
buat selesai. Bakal menyenangkan sekali kalau kita bisa melakukan hal lain selagi
kita nungguin proses-proses yang panjang itu buat kelar. Komputer modern menawarkan
dua teknik buat mengerjakan lebih dari satu operasi pada satu waktu: paralelisme 
(parallelism) dan konkurensi (concurrency). Namun, logika program kita biasanya 
ditulis dengan gaya yang linear. Kita pengen bisa menentukan operasi apa aja yang 
harus dilakukan program dan titik-titik di mana sebuah fungsi bisa berhenti sejenak 
(pause) dan bagian lain dari program bisa berjalan sebagai gantinya, tanpa perlu 
menentukan di awal secara persis urutan dan cara tiap potongan kode itu berjalan. 
_Pemrograman asinkron_ (asynchronous programming) adalah sebuah abstraksi yang 
memungkinkan kita mengekspresikan kode kita dalam bentuk titik-titik jeda 
potensial dan hasil akhirnya, yang mana bakal menangani detail koordinasinya buat 
kita.

Bab ini dibangun di atas penggunaan _threads_ buat paralelisme dan konkurensi 
di Bab 16 dengan memperkenalkan pendekatan alternatif buat nulis kode: _futures_, 
_streams_, dan sintaks `async` dan `await` di Rust yang memungkinkan kita buat 
mengekspresikan gimana operasi-operasi tersebut bisa bersifat asinkron, serta 
_crates_ pihak ketiga yang mengimplementasikan _asynchronous runtimes_: kode yang 
mengelola dan mengoordinasi eksekusi dari operasi-operasi asinkron tersebut.

Mari kita pertimbangkan sebuah contoh. Katakanlah kita lagi ngekspor video perayaan 
keluarga yang sudah kita bikin, sebuah operasi yang bisa memakan waktu mulai 
dari beberapa menit sampai berjam-jam. Ekspor video bakal memakai sebanyak 
mungkin tenaga CPU dan GPU yang bisa dia dapatkan. Kalau kita cuma punya satu 
_core_ (inti) CPU dan sistem operasi kita tidak nge-_pause_ ekspor itu sampai ia 
selesai—yaitu, kalau ia mengeksekusi ekspornya secara _sinkron_ (synchronously)—kita 
tidak bakal bisa ngelakuin hal lain di komputer kita saat tugas itu berjalan. Itu 
bakal jadi pengalaman yang cukup bikin frustrasi. Untungnya, sistem operasi di 
komputer kita bisa, dan emang ngelakuin itu, menginterupsi proses ekspor secara 
kasatmata cukup sering biar kita bisa mengerjakan tugas lain di saat yang 
bersamaan.

Sekarang katakanlah kita lagi men-download video yang di-_share_ sama orang lain, 
yang juga bisa memakan waktu lumayan lama tapi tidak terlalu menyita banyak waktu 
CPU. Di kasus ini, CPU harus menunggu data dari jaringan buat tiba. Meskipun kita 
bisa mulai membaca data tersebut begitu dia mulai berdatangan, itu bisa memakan 
waktu beberapa saat sampai semuanya muncul. Bahkan setelah semua datanya ada, 
kalau videonya sangat besar, itu bisa memakan waktu setidaknya satu atau dua detik 
buat nge-_load_ semuanya. Itu mungkin tidak kedengeran terlalu lama, tapi itu 
adalah waktu yang sangat lama buat prosesor modern, yang mana bisa melakukan 
miliaran operasi setiap detik. Sekali lagi, sistem operasi kita bakal 
menginterupsi program kita secara kasatmata buat membiarkan CPU mengerjakan 
tugas lain sembari nunggu pemanggilan jaringan selesai.

Ekspor video adalah contoh dari operasi _CPU-bound_ atau _compute-bound_ 
(bergantung pada prosesor). Dia dibatasi oleh potensi kecepatan pemrosesan data 
komputer di dalam CPU atau GPU, dan seberapa besar dari kecepatan itu yang bisa 
didedikasikan buat operasi tersebut. Download video adalah contoh dari operasi 
_I/O-bound_ (bergantung pada input-output), karena dia dibatasi oleh kecepatan 
_input dan output_ komputer; dia cuma bisa berjalan secepat data yang bisa dikirim 
lewat jaringan.

Di dua contoh ini, interupsi kasatmata dari sistem operasi ngasih suatu bentuk 
konkurensi. Namun, konkurensi itu cuma terjadi di tingkat keseluruhan program: 
sistem operasi menginterupsi satu program buat membiarkan program lain 
mengerjakan tugasnya. Di banyak kasus, karena kita paham program kita di tingkat 
yang jauh lebih spesifik (granular) ketimbang sistem operasi, kita bisa menemukan 
peluang-peluang buat konkurensi yang tidak bisa dilihat sama sistem operasi.

Misalnya, kalau kita lagi bikin _tool_ buat mengelola download file, kita seharusnya 
bisa menulis program kita sedemikian rupa sehingga pas mulai nge-download satu 
file UI-nya tidak bakal macet (lock up), dan _user_ seharusnya bisa memulai banyak 
download di saat yang bersamaan. Namun, banyak API sistem operasi buat berinteraksi 
dengan jaringan itu sifatnya _blocking_ (memblokir); yakni, mereka memblokir 
progress program sampai data yang mereka proses itu benar-benar siap seutuhnya.

> Catatan: Ini adalah cara kerja _kebanyakan_ pemanggilan fungsi, kalau dipikir-pikir. 
> Namun, istilah _blocking_ biasanya dikhususkan buat pemanggilan fungsi yang 
> berinteraksi dengan file, jaringan, atau sumber daya lain di komputer, karena 
> itu adalah kasus-kasus di mana suatu program individu bakal dapat keuntungan 
> kalau operasi tersebut _non_-blocking.

Kita bisa menghindari memblokir _main thread_ kita dengan membikin sebuah _thread_ 
khusus buat men-download tiap file. Namun, beban (overhead) dari sumber daya sistem 
yang dipakai sama _threads_ itu pada akhirnya bakal jadi masalah. Bakal lebih oke 
kalau pemanggilan fungsinya dari awal emang tidak memblokir, dan sebaliknya kita 
bisa menentukan sejumlah tugas yang pengen diselesaikan program kita lalu 
membiarkan _runtime_ memilih urutan dan cara terbaik buat menjalankannya.

Nah, itulah persisnya yang dikasih sama abstraksi _asinkron_ (singkatan dari 
_async_) di Rust buat kita. Di bab ini, kita bakal belajar semua hal tentang 
asinkron selagi kita membahas topik-topik berikut:

- Gimana cara memakai sintaks `async` dan `await` di Rust dan mengeksekusi 
  fungsi-fungsi asinkron dengan sebuah _runtime_
- Gimana cara memakai model asinkron buat nyelesein beberapa tantangan yang sama 
  seperti yang udah kita lihat di Bab 16
- Gimana _multithreading_ dan asinkron menyediakan solusi yang saling melengkapi, 
  yang bisa kita gabungkan di banyak kasus

Tapi, sebelum kita lihat gimana cara kerja asinkron di praktiknya, kita perlu 
sedikit belok sebentar buat ngebahas perbedaan antara paralelisme dan konkurensi.

## Paralelisme dan Konkurensi

Sejauh ini kita memperlakukan paralelisme dan konkurensi seolah-olah maknanya bisa 
ditukar-tukar. Sekarang kita harus bisa membedakannya dengan lebih presisi, 
karena perbedaannya bakal kerasa pas kita udah mulai bekerja.

Bayangin aja cara-cara beda yang bisa dipakai sebuah tim buat ngebagi kerjaan 
di suatu proyek _software_. Kita bisa menugaskan satu orang beberapa tugas, 
menugaskan tiap anggota satu tugas, atau memakai campuran dari kedua pendekatan 
tersebut.

Saat satu orang individu mengerjakan beberapa tugas yang berbeda sebelum satupun 
dari tugas-tugas itu selesai, ini dinamakan _konkurensi_. Salah satu cara buat 
mengimplementasikan konkurensi adalah mirip kayak punya dua project berbeda yang 
lagi dibuka di komputer kita, dan pas kita bosan atau mentok di satu project, 
kita beralih ke project yang satu lagi. Kita cuma satu orang, jadi kita tidak 
bisa bikin progress di kedua tugas pada waktu yang sama persis, tapi kita bisa 
*multitask*, bikin progress pada satu tugas secara bergantian dengan beralih 
di antara mereka (lihat Gambar 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="Sebuah diagram dengan kotak-kotak bertumpuk berlabel Tugas A dan Tugas B, dengan belah ketupat di dalamnya yang melambangkan subtugas. Ada panah yang menunjuk dari A1 ke B1, B1 ke A2, A2 ke B2, B2 ke A3, A3 ke A4, dan A4 ke B3. Panah di antara subtugas-subtugas ini menyilang di antara kotak untuk Tugas A dan Tugas B." />

<figcaption>Gambar 17-1: Sebuah alur kerja konkuren, beralih di antara Tugas A dan Tugas B</figcaption>

</figure>

Saat sebuah tim membagi sekelompok tugas dengan nyuruh setiap anggota mengambil satu 
tugas lalu mengerjakannya sendirian, ini dinamakan _paralelisme_. Setiap orang di 
tim tersebut bisa membikin progress pada waktu yang sama persis (lihat Gambar 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="Sebuah diagram dengan kotak-kotak bertumpuk berlabel Tugas A dan Tugas B, dengan belah ketupat di dalamnya yang melambangkan subtugas. Ada panah yang menunjuk dari A1 ke A2, A2 ke A3, A3 ke A4, B1 ke B2, dan B2 ke B3. Tidak ada panah yang menyilang antara kotak untuk Tugas A dan Tugas B." />

<figcaption>Gambar 17-2: Sebuah alur kerja paralel, di mana pekerjaan terjadi pada Tugas A dan Tugas B secara independen</figcaption>

</figure>

Di kedua alur kerja (workflows) ini, kita mungkin harus mengoordinasikan antara 
tugas-tugas yang berbeda. Mungkin kita pikir tugas yang diberikan ke satu orang 
itu benar-benar independen dari pekerjaan orang lain, tapi ternyata dia butuh 
orang lain di tim itu buat nyelesein tugas mereka lebih dulu. Beberapa pekerjaan 
bisa dilakukan secara paralel, tapi sebagian darinya itu sebenarnya _serial_: 
dia cuma bisa terjadi dalam satu urutan, satu tugas setelah tugas lainnya, 
kayak di Gambar 17-3.

<figure>

<img src="img/trpl17-03.svg" class="center" alt="Sebuah diagram dengan kotak-kotak bertumpuk berlabel Tugas A dan Tugas B, dengan belah ketupat di dalamnya yang melambangkan subtugas. Di Tugas A, panah menunjuk dari A1 ke A2, dari A2 ke sepasang garis vertikal tebal seperti simbol “pause”, dan dari simbol itu ke A3. Di Tugas B, panah menunjuk dari B1 ke B2, B2 ke B3, B3 ke A3, dan B3 ke B4." />

<figcaption>Gambar 17-3: Sebuah alur kerja yang sebagiannya paralel, di mana pekerjaan terjadi pada Tugas A dan Tugas B secara independen sampai Tugas A3 terhambat (blocked) menunggu hasil dari Tugas B3.</figcaption>

</figure>

Sama halnya, kita mungkin sadar kalau salah satu tugas kita sendiri itu bergantung 
sama tugas kita yang lainnya. Sekarang pekerjaan konkuren kita juga udah jadi 
serial.

Paralelisme dan konkurensi juga bisa bersinggungan satu sama lain. Kalau kita 
tahu kalau rekan kerja kita lagi mentok sampai kita nyelesein salah satu tugas 
kita, kita mungkin bakal memusatkan seluruh usaha kita pada tugas itu buat 
"membuka jalan" (unblock) rekan kerja kita tersebut. Kita dan rekan kerja kita 
tidak lagi bisa bekerja secara paralel, dan kita juga tidak lagi bisa bekerja 
secara konkuren di tugas kita masing-masing.

Dinamika dasar yang sama mulai berlaku pada perangkat lunak dan perangkat keras. 
Di mesin yang punya satu _core_ CPU, CPU cuma bisa melakukan satu operasi pada 
satu waktu, tapi dia tetap bisa bekerja secara konkuren. Memakai alat seperti 
_threads_, proses, dan asinkron, komputer bisa mem-_pause_ satu aktivitas lalu 
beralih ke aktivitas lain sebelum akhirnya berputar kembali ke aktivitas pertama 
tadi. Di mesin dengan banyak _core_ CPU, dia juga bisa mengerjakan tugas secara 
paralel. Satu _core_ bisa ngerjain satu tugas sementara _core_ lain ngerjain 
tugas yang sama sekali tidak ada hubungannya, dan operasi-operasi itu benar-benar 
terjadi pada waktu yang bersamaan.

Menjalankan kode asinkron di Rust biasanya terjadi secara konkuren. Tergantung 
pada perangkat keras, sistem operasi, dan *async runtime* yang lagi kita pakai 
(*async runtimes* bakal dibahas sebentar lagi), konkurensi itu mungkin juga 
memakai paralelisme di balik layar.

Sekarang, mari kita selami gimana cara kerja pemrograman asinkron di Rust sebenarnya.
