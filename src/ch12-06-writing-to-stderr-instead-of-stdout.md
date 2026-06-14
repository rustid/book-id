## Menulis Pesan Error ke Standard Error Bukannya Standard Output

Saat ini, kita menulis semua output ke terminal memakai macro `println!`. Di 
sebagian besar terminal, ada dua jenis output: _standard output_ (`stdout`) 
buat informasi umum dan _standard error_ (`stderr`) buat pesan-pesan error. 
Pemisahan ini memungkinkan para *user* untuk memilih buat mengarahkan 
(redirect) output yang sukses dari suatu program ke sebuah file, tapi tetap 
bisa mencetak pesan-pesan error ke layar.

Macro `println!` cuma bisa mencetak ke *standard output*, jadi kita harus 
pakai sesuatu yang lain buat bisa mencetak ke *standard error*.

### Mengecek Ke Mana Error Ditulis

Pertama-tama mari kita amati gimana konten yang dicetak oleh `minigrep` saat 
ini ditulis ke *standard output*, termasuk pesan-pesan error apa pun yang 
sebenarnya mau kita tulis ke *standard error*. Kita bakal ngelakuin itu dengan 
mengarahkan *stream standard output* ke sebuah file sementara kita sengaja 
membikin sebuah error. Kita tidak akan mengarahkan *stream standard error*, 
jadi konten apa pun yang dikirim ke *standard error* bakal tetap ditampilkan 
di layar.

Program-program *command line* umumnya diharapkan bakal mengirim pesan error 
ke *stream standard error* supaya kita masih bisa melihat pesan-pesan error 
di layar bahkan kalau kita mengarahkan *stream standard output* ke sebuah file. 
Program kita saat ini belum berperilaku dengan baik: kita bakal segera melihat 
kalau dia malah menyimpan output pesan error-nya ke sebuah file!

Buat mendemonstrasikan perilaku ini, kita bakal menjalankan programnya dengan 
`>` dan _path_ file, _output.txt_, ke mana kita mau mengarahkan *stream standard 
output*. Kita tidak akan memasukkan argumen apa pun, yang mana seharusnya bakal 
menyebabkan error:

```console
$ cargo run > output.txt
```

Sintaks `>` memberi tahu *shell* (terminal) buat menulis konten dari *standard 
output* ke _output.txt_ ketimbang menampilkannya ke layar. Kita ternyata tidak 
melihat pesan error yang kita harapkan tercetak di layar, jadi itu artinya pesan 
tersebut pasti berujung di dalam file itu. Inilah apa yang terkandung di dalam 
_output.txt_:

```text
Problem parsing arguments: not enough arguments
```

Yup, pesan error kita benar-benar dicetak ke *standard output*. Bakal jauh 
lebih berguna kalau pesan-pesan error semacam ini dicetak ke *standard error* 
sehingga hanya data dari proses yang berjalan dengan sukses saja yang bakal 
berakhir di dalam file tersebut. Kita bakal mengubah hal itu.

### Mencetak Error ke Standard Error

Kita bakal memakai kode di Listing 12-24 buat mengubah gimana pesan-pesan error 
dicetak. Karena perombakan (refactoring) yang kita lakuin sebelumnya di bab ini, 
semua kode yang mencetak pesan error ada di satu fungsi aja, yaitu `main`. 
*Standard library* menyediakan macro `eprintln!` yang bisa mencetak ke *stream 
standard error*, jadi mari ubah dua tempat di mana kita memakai `println!` buat 
mencetak error agar memakai `eprintln!` sebagai gantinya.

<Listing number="12-24" file-name="src/main.rs" caption="Menulis pesan error ke *standard error* bukannya *standard output* menggunakan `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Sekarang mari kita coba jalankan lagi programnya pakai cara yang sama, tanpa 
argumen apa pun dan mengarahkan *standard output* pakai `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Sekarang kita bisa melihat pesan error-nya di layar dan _output.txt_ isinya 
kosong, yang mana ini persis seperti perilaku yang kita harapkan dari 
program-program *command line*.

Mari kita jalankan programnya sekali lagi, kali ini dengan argumen-argumen 
yang tidak bakal menyebabkan error tapi kita tetap mengarahkan *standard 
output* ke sebuah file, kayak gini:

```console
$ cargo run -- to poem.txt > output.txt
```

Kita tidak akan melihat output apa pun di terminal, dan _output.txt_ bakal 
berisi hasil-hasil pencarian kita:

<span class="filename">Nama file: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Ini mendemonstrasikan kalau sekarang kita sudah memakai *standard output* 
buat output yang sukses dan memakai *standard error* buat output yang berupa 
error sebagaimana mestinya.

## Ringkasan

Bab ini merangkum (recap) beberapa konsep besar yang udah kita pelajarin sejauh 
ini dan membahas soal gimana caranya ngejalanin operasi I/O (Input/Output) 
umum di Rust. Dengan memakai argumen *command line*, file, *environment variables*, 
dan macro `eprintln!` buat mencetak error, kita sekarang sudah siap buat nulis 
berbagai aplikasi *command line*. Digabungkan dengan konsep-konsep dari bab-bab 
sebelumnya, kode kita bakal terorganisir dengan rapi, menyimpan data secara efektif 
di struktur data yang tepat, menangani error dengan apik, dan telah teruji dengan 
baik.

Berikutnya, kita bakal mengeksplorasi beberapa fitur di Rust yang dipengaruhi 
oleh bahasa pemrograman fungsional: _closures_ dan _iterators_.
