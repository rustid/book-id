## Mengkustomisasi Build dengan Release Profiles

Di Rust, _release profiles_ (profil rilis) adalah profil yang sudah didefinisikan 
sebelumnya (predefined) dan bisa dikustomisasi dengan berbagai konfigurasi yang 
memungkinkan programmer buat punya lebih banyak kontrol atas berbagai opsi saat 
men-compile kode. Setiap profil dikonfigurasi secara independen satu sama lain.

Cargo punya dua profil utama: profil `dev` yang dipakai Cargo saat kita 
menjalankan `cargo build`, dan profil `release` yang dipakai Cargo saat kita 
menjalankan `cargo build --release`. Profil `dev` didefinisikan dengan 
_default_ yang bagus buat fase _development_ (pengembangan), dan profil 
`release` punya _default_ yang bagus buat _release builds_ (build versi rilis).

Nama-nama profil ini mungkin terasa familier dari output _build_ yang pernah 
kita jalankan:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` dan `release` adalah profil-profil berbeda yang dipakai sama _compiler_.

Cargo punya pengaturan _default_ buat setiap profil yang berlaku kalau kita 
belum menambahkan bagian `[profile.*]` secara eksplisit di file _Cargo.toml_ 
milik project kita. Dengan menambahkan bagian `[profile.*]` buat profil mana 
pun yang mau kita kustomisasi, kita bisa menimpa (_override_) sebagian dari 
pengaturan _default_-nya. Misalnya, ini adalah nilai _default_ buat pengaturan 
`opt-level` di profil `dev` dan `release`:

<span class="filename">Nama file: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

Pengaturan `opt-level` mengontrol seberapa banyak optimasi yang bakal 
diterapkan Rust ke kode kita, dengan rentang nilai dari 0 sampai 3. Menerapkan 
lebih banyak optimasi bakal memperpanjang waktu kompilasi, jadi kalau kita 
sedang dalam fase pengembangan dan sering men-compile kode kita, kita bakal 
mau lebih sedikit optimasi agar kodenya bisa di-compile lebih cepat meskipun 
kode akhirnya nanti berjalan lebih lambat. Oleh karena itu, nilai `opt-level` 
_default_ buat `dev` adalah `0`. Saat kita sudah siap merilis kode kita, hal 
terbaik adalah menghabiskan lebih banyak waktu buat kompilasi. Kita cuma 
bakal men-compile dalam mode `release` sesekali saja, tapi kita bakal 
menjalankan program yang sudah di-compile itu berkali-kali, jadi mode 
`release` menukar waktu kompilasi yang lebih lama demi mendapatkan kode yang 
berjalan lebih cepat. Itulah kenapa nilai `opt-level` _default_ buat profil 
`release` adalah `3`.

Kita bisa menimpa pengaturan _default_ dengan menambahkan nilai yang berbeda 
untuknya di _Cargo.toml_. Misalnya, kalau kita mau memakai tingkat optimasi 1 
di profil _development_, kita bisa menambahkan dua baris ini ke file _Cargo.toml_ 
project kita:

<span class="filename">Nama file: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Kode ini menimpa pengaturan _default_ yaitu `0`. Sekarang pas kita menjalankan 
`cargo build`, Cargo bakal memakai nilai-nilai _default_ buat profil `dev` 
ditambah dengan kustomisasi kita pada `opt-level`. Karena kita menge-set 
`opt-level` jadi `1`, Cargo bakal menerapkan lebih banyak optimasi daripada 
nilai _default_-nya, tapi tidak sebanyak di versi _release build_.

Buat melihat daftar lengkap dari opsi konfigurasi dan nilai _default_ dari 
setiap profil, silakan cek 
[dokumentasi Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).
