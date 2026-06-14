## Mendefinisikan sama Menginisialisasi Structs

Struct itu mirip sama tuple yang udah kita bahas di bagian [“Tipe Tuple”][tuples], 
karena keduanya sama-sama nampung banyak nilai yang terkait. Kayak tuple, 
bagian-bagian dari sebuah struct bisa punya tipe yang beda-beda. Bedanya sama 
tuple, di dalem struct kita ngasih nama ke tiap potongan datanya biar jelas 
apa makna dari nilai-nilai itu. Dengan adanya nama-nama ini, struct jadi lebih 
fleksibel daripada tuple: kita nggak perlu ngandelin urutan datanya buat 
nentuin atau akses nilai dari sebuah instance.

Buat mendefinisikan sebuah struct, kita tulis keyword `struct` terus kasih nama 
ke seluruh struct-nya. Nama struct harusnya ngejelasin seberapa penting potongan 
data yang lagi dikelompokin itu. Terus, di dalem kurung kurawal, kita 
mendefinisikan nama sama tipe dari potongan datanya, yang kita sebut sebagai 
_fields_. Contohnya, Listing 5-1 nunjukin sebuah struct yang nyimpen info soal 
akun user.

<Listing number="5-1" file-name="src/main.rs" caption="Definisi struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Buat pake sebuah struct setelah kita definisikan, kita bikin sebuah _instance_ 
dari struct itu dengan nentuin nilai konkret buat tiap field-nya. Kita bikin 
sebuah instance dengan nulis nama struct-nya terus nambahin kurung kurawal yang 
isinya pasangan _`key: value`_, di mana _key_-nya adalah nama field-nya dan 
_value_-nya adalah data yang mau kita simpen di field itu. Kita nggak harus 
nentuin field-nya sesuai urutan pas kita mendeklarasikannya di struct-nya. 
Dengan kata lain, definisi struct itu kayak template umum buat tipenya, dan 
instance ngisi template itu dengan data tertentu buat bikin nilai dari tipe 
tersebut. Contohnya, kita bisa mendeklarasikan seorang user tertentu kayak yang 
ditunjukin di Listing 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Bikin instance dari struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Buat dapet nilai spesifik dari sebuah struct, kita pake notasi titik (dot 
notation). Misalnya, buat akses alamat email user ini, kita pake `user1.email`. 
Kalau instance-nya mutable, kita bisa ngerubah nilainya pake notasi titik terus 
di-assign ke field tertentu. Listing 5-3 nunjukin gimana cara ngerubah nilai di 
field `email` dari sebuah instance `User` yang mutable.

<Listing number="5-3" file-name="src/main.rs" caption="Ngerubah nilai di field `email` dari sebuah instance `User` yang mutable">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Perhatiin ya kalau seluruh instance-nya harus mutable; Rust nggak ngebolehin 
kita nandain cuma field tertentu doang sebagai mutable. Sama kayak ekspresi 
mana pun, kita bisa ngekonstruksi instance baru dari struct-nya sebagai 
ekspresi terakhir di body fungsi buat secara implisit balikin instance baru itu.

Listing 5-4 nunjukin sebuah fungsi `build_user` yang balikin instance `User` 
dengan email sama username yang dikasih. Field `active` dapet nilai `true`, 
dan `sign_in_count` dapet nilai `1`.

<Listing number="5-4" file-name="src/main.rs" caption="Fungsi `build_user` yang nerima email sama username terus balikin instance `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Masuk akal sekali kan buat ngasih nama parameter fungsi sama kayak nama field 
struct-nya, tapi harus ngulang-ngulang nama field `email` sama `username` dan 
variabelnya itu agak ribet. Kalau struct-nya punya lebih banyak field lagi, 
ngulangin tiap namanya bakal makin nyebelin. Untungnya, ada cara singkat yang 
nyaman!

<!-- Old heading. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Pake Shorthand Inisialisasi Field (Field Init Shorthand)

Karena nama parameternya sama nama field struct-nya persis sama di Listing 5-4, 
kita bisa pake sintaks _field init shorthand_ buat nulis ulang `build_user` 
biar perilakunya persis sama tapi nggak ada pengulangan `username` sama `email`, 
kayak yang ditunjukin di Listing 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="Fungsi `build_user` yang pake field init shorthand karena parameter `username` sama `email` punya nama yang sama kayak field struct-nya">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Di sini, kita lagi bikin instance baru dari struct `User`, yang punya field 
namanya `email`. Kita mau set nilai field `email` ke nilai di parameter `email` 
dari fungsi `build_user`. Karena field `email` sama parameter `email` punya 
nama yang sama, kita cuma perlu nulis `email` doang bukannya `email: email`.

### Bikin Instance dari Instance Lain pake Sintaks Update Struct (Struct Update Syntax)

Sering kali berguna buat bikin instance baru dari sebuah struct yang isinya 
kebanyakan nilai dari instance lain dengan tipe yang sama, tapi ngerubah 
beberapa nilainya. Kita bisa lakuin ini pake _struct update syntax_.

Pertama, di Listing 5-6 kita liat cara bikin instance `User` baru di `user2` 
secara biasa, tanpa pake update syntax. Kita set nilai baru buat `email` tapi 
selain itu pake nilai-nilai yang sama dari `user1` yang udah kita bikin di 
Listing 5-2.

<Listing number="5-6" file-name="src/main.rs" caption="Bikin instance `User` baru pake hampir semua nilai dari `user1` kecuali satu">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Pake _struct update syntax_, kita bisa dapet efek yang sama dengan kode yang 
lebih dikit, kayak yang ditunjukin di Listing 5-7. Sintaks `..` nentuin kalau 
sisa field yang nggak di-set secara eksplisit harusnya punya nilai yang sama 
kayak field di instance yang dikasih.

<Listing number="5-7" file-name="src/main.rs" caption="Pake struct update syntax buat set nilai `email` baru buat instance `User` tapi pake sisa nilai dari `user1` lainnya">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Kode di Listing 5-7 juga bikin instance di `user2` yang punya nilai beda buat 
`email` tapi punya nilai yang sama buat field `username`, `active`, dan 
`sign_in_count` dari `user1`. Tanda `..user1` harus ditaruh di paling akhir 
buat nentuin kalau sisa field apa pun harus dapet nilainya dari field yang 
terkait di `user1`, tapi kita bebas nentuin nilai buat sebanyak apa pun field 
yang kita mau dengan urutan apa pun, nggak peduli urutan field-nya di definisi 
struct-nya.

Perhatiin ya kalau _struct update syntax_ pake `=` kayak sebuah assignment; 
ini karena dia nge-_move_ datanya, sama kayak yang kita liat di bagian 
[“Interaksi Variabel dan Data dengan Move”][move]. Di contoh ini, kita udah 
nggak bisa lagi pake `user1` setelah bikin `user2` karena `String` di field 
`username` dari `user1` udah di-_move_ ke dalem `user2`. Kalau kita ngasih 
`user2` nilai `String` baru buat baik `email` maupun `username`, dan makanya 
cuma pake nilai `active` sama `sign_in_count` dari `user1`, berarti `user1` 
bakal tetep valid setelah bikin `user2`. Baik `active` maupun `sign_in_count` 
adalah tipe yang mengimplementasikan trait `Copy`, jadi perilaku yang kita 
bahas di bagian [“Data Khusus Stack: Copy”][copy] bakal berlaku. Kita juga 
tetep bisa pake `user1.email` di contoh ini, karena nilainya nggak di-_move_ 
keluar dari `user1`.

### Pake Tuple Structs tanpa Field Bernama buat Bikin Tipe yang Beda

Rust juga support struct yang tampilannya mirip sama tuple, namanya _tuple 
structs_. Tuple structs punya makna tambahan yang dikasih sama nama struct-nya 
tapi nggak punya nama yang terkait sama field-field-nya; sebaliknya, mereka 
cuma punya tipe dari field-field-nya aja. Tuple structs berguna pas kita mau 
ngasih nama ke seluruh tuple-nya dan bikin tuple itu jadi tipe yang beda dari 
tuple lainnya, dan pas ngasih nama ke tiap field kayak di struct biasa bakal 
terasa terlalu panjang (verbose) atau redundan.

Buat mendefinisikan sebuah tuple struct, mulai pake keyword `struct` dan nama 
struct-nya diikuti sama tipe-tipe di dalem tuple-nya. Misalnya, di sini kita 
mendefinisikan dan pake dua tuple structs namanya `Color` dan `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Perhatiin ya kalau nilai `black` sama `origin` itu tipe yang beda karena mereka 
adalah instance dari tuple structs yang beda. Tiap struct yang kita definisikan 
itu adalah tipenya sendiri, walaupun field-field di dalem struct-nya mungkin 
punya tipe yang sama. Misalnya, sebuah fungsi yang nerima parameter tipe `Color` 
nggak bisa nerima sebuah `Point` sebagai argumen, walaupun kedua tipenya sama-
sama disusun dari tiga nilai `i32`. Selain itu, instance tuple struct mirip 
sama tuple karena kita bisa _destructure_ mereka jadi bagian-bagian individunya, 
dan kita bisa pake tanda `.` diikuti indeks buat akses nilai individunya. Beda 
sama tuple, tuple struct nuntut kita buat nulis nama tipe struct-nya pas kita 
_destructure_ mereka. Misalnya, kita bakal nulis `let Point(x, y, z) = origin;` 
buat _destructure_ nilai di titik `origin` jadi variabel namanya `x`, `y`, dan `z`.

### Struct Mirip-Unit (Unit-Like Structs) tanpa Field Apa pun

Kita juga bisa mendefinisikan struct yang nggak punya field apa pun! Ini namanya 
_unit-like structs_ karena perilakunya mirip sama `()`, yaitu tipe unit yang 
pernah kita sebutin di bagian [“Tipe Tuple”][tuples]. Unit-like structs bisa 
berguna pas kita perlu mengimplementasikan sebuah trait pada suatu tipe tapi 
nggak punya data apa pun yang mau kita simpen di tipe itu sendiri. Kita bakal 
bahas traits di Bab 10. Ini contoh deklarasi sama inisialisasi struct unit 
namanya `AlwaysEqual`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

Buat mendefinisikan `AlwaysEqual`, kita pake keyword `struct`, nama yang kita 
mau, terus tanda titik koma. Nggak perlu kurung kurawal atau tanda kurung! 
Terus kita bisa dapet instance dari `AlwaysEqual` di variabel `subject` dengan 
cara yang mirip: pake nama yang udah kita definisikan, tanpa kurung kurawal 
atau tanda kurung apa pun. Bayangin kalau nanti kita bakal mengimplementasikan 
perilaku buat tipe ini biar tiap instance dari `AlwaysEqual` itu selalu sama 
dengan tiap instance dari tipe lainnya, mungkin buat punya hasil yang udah tau 
buat tujuan testing. Kita nggak bakal butuh data apa pun buat 
mengimplementasikan perilaku itu! Kita bakal liat di Bab 10 gimana cara 
mendefinisikan traits dan mengimplementasikannya pada tipe apa pun, termasuk 
unit-like structs.

> ### Ownership dari Data Struct
>
> Di definisi struct `User` di Listing 5-1, kita pake tipe `String` yang 
> dimiliki (_owned_) bukannya tipe string slice `&str`. Ini pilihan yang 
> disengaja karena kita mau tiap instance dari struct ini punya semua datanya 
> sendiri dan biar datanya valid selama seluruh struct-nya juga valid.
>
> Mungkin juga buat struct nyimpen referensi ke data yang dimiliki sama hal 
> lain, tapi buat lakuin itu butuh penggunaan _lifetimes_, sebuah fitur Rust 
> yang bakal kita bahas di Bab 10. _Lifetimes_ mastiin kalau data yang dirujuk 
> sama sebuah struct itu valid selama struct-nya masih ada. Katakanlah kita 
> nyoba nyimpen sebuah referensi di sebuah struct tanpa nentuin _lifetimes_, 
> kayak berikut; ini nggak bakal jalan:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> _Compiler_ bakal protes kalau dia butuh penentu _lifetime_ (lifetime 
> specifiers):
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> Di Bab 10, kita bakal bahas gimana cara benerin error-error ini biar kita bisa 
> nyimpen referensi di struct, tapi buat sekarang, kita bakal benerin error 
> kayak gini pake tipe _owned_ kayak `String` bukannya referensi kayak `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
