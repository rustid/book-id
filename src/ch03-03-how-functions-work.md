## Fungsi

Fungsi itu ada di mana-mana di kode Rust. Kita udah liat salah satu fungsi 
paling penting di bahasanya: fungsi `main`, yang jadi _entry point_ buat banyak 
program. Kita juga udah liat keyword `fn`, yang ngebolehin kita mendeklarasikan 
fungsi baru.

Kode Rust pake _snake case_ sebagai gaya konvensional buat nama fungsi sama 
variabel, di mana semua hurufnya kecil (lowercase) dan pake garis bawah 
(underscore) buat misahin kata. Ini program yang isinya contoh definisi fungsi:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Kita mendefinisikan fungsi di Rust dengan nulis `fn` diikuti sama nama fungsi 
dan tanda kurung. Kurung kurawal ngasih tau _compiler_ di mana body fungsinya 
mulai sama selesai.

Kita bisa manggil fungsi apa pun yang udah kita definisikan dengan nulis namanya 
diikuti tanda kurung. Karena `another_function` didefinisikan di programnya, 
dia bisa dipanggil dari dalem fungsi `main`. Inget ya kalau kita mendefinisikan 
`another_function` _setelah_ fungsi `main` di source code-nya; kita bisa aja 
mendefinisikannya sebelum `main` juga kok. Rust nggak peduli di mana kita 
mendefinisikan fungsi kita, yang penting mereka didefinisikan di suatu tempat 
di scope yang bisa diliat sama pemanggilnya.

Yuk kita bikin project biner baru namanya _functions_ buat eksplor fungsi lebih 
lanjut. Taruh contoh `another_function` tadi di _src/main.rs_ terus jalanin. 
Kita bakal liat output kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Baris-baris kodenya jalan sesuai urutan kemunculannya di fungsi `main`. Pertama 
pesan “Hello, world!” dicetak, terus `another_function` dipanggil dan pesannya 
dicetak.

### Parameter

Kita bisa mendefinisikan fungsi biar punya _parameter_, yaitu variabel khusus 
yang jadi bagian dari _signature_ sebuah fungsi. Pas sebuah fungsi punya 
parameter, kita bisa ngasih nilai konkret buat parameter itu. Secara teknis, 
nilai konkret itu namanya _argument_, tapi pas lagi ngobrol santai, orang-orang 
cenderung pake kata _parameter_ sama _argument_ secara bergantian buat nyebut 
variabel di definisi fungsi maupun nilai konkret yang dimasukin pas manggil 
fungsinya.

Di versi `another_function` ini kita nambahin satu parameter:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Coba jalanin program ini; kita bakal dapet output kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

Deklarasi `another_function` punya satu parameter namanya `x`. Tipe dari `x` 
ditentuin sebagai `i32`. Pas kita masukin `5` ke `another_function`, macro 
`println!` naruh `5` di tempat pasangan kurung kurawal yang isinya `x` di 
format string-nya.

Di signature fungsi, kita _harus_ mendeklarasikan tipe dari tiap parameter. Ini 
keputusan yang disengaja di desainnya Rust: nuntut annotasi tipe di definisi 
fungsi artinya _compiler_ hampir nggak pernah butuh kita buat nulis tipenya di 
tempat lain di kode buat cari tau tipe mana yang kita maksud. _Compiler_ juga 
bisa ngasih pesan error yang lebih ngebantu kalau dia tau tipe apa yang 
diharapin sama fungsinya.

Pas mendefinisikan banyak parameter, pisahin deklarasi parameternya pake koma, 
kayak gini:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Contoh ini bikin fungsi namanya `print_labeled_measurement` dengan dua 
parameter. Parameter pertama namanya `value` dan tipenya `i32`. Yang kedua 
namanya `unit_label` dan tipenya `char`. Fungsinya terus nyetak teks yang isinya 
baik `value` maupun `unit_label`.

Yuk coba jalanin kode ini. Ganti program yang ada di project _functions_ kita 
di file _src/main.rs_ sama contoh di atas terus jalanin pake `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Karena kita manggil fungsinya dengan `5` sebagai nilai buat `value` dan `'h'` 
sebagai nilai buat `unit_label`, output programnya isinya nilai-nilai itu.

### Statement dan Ekspresi (Statements and Expressions)

Body fungsi itu disusun dari serangkaian statement yang opsional bisa diakhiri 
sama sebuah ekspresi. Sejauh ini, fungsi-fungsi yang kita bahas belum ada 
ekspresi akhirnya, tapi kita udah liat ekspresi sebagai bagian dari sebuah 
statement. Karena Rust itu bahasa yang berbasis ekspresi (_expression-based 
language_), ini perbedaan penting yang harus dipahamin. Bahasa lain nggak punya 
perbedaan yang sama, jadi yuk kita liat apa itu statement sama ekspresi dan 
gimana perbedaannya ngaruh ke body fungsi.

- **Statement** adalah instruksi yang ngelakuin suatu aksi dan nggak balikin 
  nilai.
- **Ekspresi** dievaluasi jadi sebuah nilai hasil.

Yuk kita liat beberapa contoh.

Sebenernya kita udah pake statement sama ekspresi. Bikin variabel terus ngasih 
nilai ke variabel itu pake keyword `let` itu adalah sebuah statement. Di 
Listing 3-1, `let y = 6;` adalah sebuah statement.

<Listing number="3-1" file-name="src/main.rs" caption="Deklarasi fungsi `main` yang isinya satu statement">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Definisi fungsi juga termasuk statement; seluruh contoh di atas itu sebenernya 
sebuah statement. (Tapi kayak yang bakal kita liat di bawah, _manggil_ fungsi 
itu bukan statement.)

Statement nggak balikin nilai. Makanya, kita nggak bisa nge-assign sebuah 
statement `let` ke variabel lain, kayak yang dicoba sama kode berikut; kita 
bakal dapet error:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Pas kita jalanin program ini, error yang kita dapet bakal keliatan kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

Statement `let y = 6` nggak balikin nilai, jadi nggak ada apa-apa buat di-bind 
ke `x`. Ini beda sama apa yang terjadi di bahasa lain, kayak C sama Ruby, di 
mana assignment balikin nilai dari assignment-nya. Di bahasa-bahasa itu, kita 
bisa nulis `x = y = 6` terus bikin baik `x` maupun `y` punya nilai `6`; hal itu 
nggak berlaku di Rust.

Ekspresi dievaluasi jadi sebuah nilai dan nyusun sebagian besar sisa kode yang 
bakal kita tulis di Rust. Coba pikirin operasi matematika, kayak `5 + 6`, yang 
merupakan ekspresi yang dievaluasi jadi nilai `11`. Ekspresi bisa jadi bagian 
dari statement: di Listing 3-1, angka `6` di statement `let y = 6;` adalah 
sebuah ekspresi yang dievaluasi jadi nilai `6`. Manggil fungsi itu adalah sebuah 
ekspresi. Manggil macro itu adalah sebuah ekspresi. Sebuah blok scope baru yang 
dibuat pake kurung kurawal juga ekspresi, contohnya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Ekspresi ini:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

adalah sebuah blok yang, dalam kasus ini, dievaluasi jadi `4`. Nilai itu terus 
di-bind ke `y` sebagai bagian dari statement `let`. Inget ya kalau baris 
`x + 1` nggak punya titik koma di akhirnya, beda sama kebanyakan baris yang 
udah kita liat sejauh ini. Ekspresi nggak pake titik koma di akhir. Kalau kita 
nambahin titik koma di akhir ekspresi, kita ngerubahnya jadi statement, dan dia 
nggak bakal balikin nilai. Terus inget ini pas kita eksplor nilai return fungsi 
sama ekspresi selanjutnya.

### Fungsi dengan Nilai Return

Fungsi bisa balikin nilai ke kode yang manggil mereka. Kita nggak ngasih nama 
buat nilai return, tapi kita harus mendeklarasikan tipenya setelah tanda panah 
(`->`). Di Rust, nilai return dari sebuah fungsi itu sinonim sama nilai dari 
ekspresi terakhir di blok body fungsinya. Kita bisa return lebih awal dari 
sebuah fungsi pake keyword `return` terus nentuin nilainya, tapi kebanyakan 
fungsi balikin ekspresi terakhir secara implisit. Ini contoh fungsi yang 
balikin nilai:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Nggak ada pemanggilan fungsi, macro, atau bahkan statement `let` di fungsi 
`five`—cuma ada angka `5` sendirian. Itu fungsi yang sangat valid di Rust. 
Inget ya kalau tipe return fungsinya ditentuin juga, yaitu `-> i32`. Coba 
jalanin kode ini; output-nya bakal kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

Angka `5` di `five` adalah nilai return fungsinya, makanya tipe return-nya 
`i32`. Yuk kita pelajari ini lebih detail. Ada dua bagian penting: pertama, 
baris `let x = five();` nunjukin kalau kita pake nilai return fungsi buat 
menginisialisasi variabel. Karena fungsi `five` balikin `5`, baris itu sama 
aja kayak gini:

```rust
let x = 5;
```

Kedua, fungsi `five` nggak punya parameter dan mendefinisikan tipe nilai 
return-nya, tapi body fungsinya cuma angka `5` kesepian tanpa titik koma karena 
itu adalah ekspresi yang nilainya mau kita balikin.

Yuk liat contoh lainnya:

<span class="filename">Nama file: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Jalanin kode ini bakal nyetak `The value of x is: 6`. Tapi kalau kita naruh 
titik koma di akhir baris yang isinya `x + 1`, ngerubahnya dari ekspresi jadi 
statement, kita bakal dapet error:

<span class="filename">Nama file: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Compile kode ini ngasilin error kayak gini:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Pesan error utamanya, `mismatched types`, ngungkapin inti masalah kodenya. 
Definisi fungsi `plus_one` bilang kalau dia bakal balikin `i32`, tapi statement 
nggak dievaluasi jadi sebuah nilai, yang direpresentasikan sama `()`, yaitu tipe 
unit. Makanya, nggak ada apa pun yang dibalikin, yang bertentangan sama 
definisi fungsi dan ngasilin error. Di output ini, Rust ngasih pesan yang 
mungkin bisa ngebantu benerin masalah ini: dia nyaranin buat ngapus titik 
komanya, yang bakal benerin error-nya.
