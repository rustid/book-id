## Lampiran G - Bagaimana Rust Dibuat dan “Nightly Rust”

Lampiran ini membahas bagaimana Rust dibuat dan pengaruhnya bagi kita sebagai pengembang Rust.

### Kestabilan Tanpa Kemandekan

Sebagai sebuah bahasa, Rust sangat peduli pada kestabilan kode kita. Kami ingin Rust menjadi fondasi kokoh untuk membangun program. Jika segalanya terus berubah, hal itu mustahil dilakukan. Di sisi lain, tanpa eksperimen fitur baru, kita tidak akan menemukan kekurangan sebelum fitur dirilis dan tidak bisa diubah lagi.

Solusi kami adalah "kestabilan tanpa kemandekan". Prinsipnya: kita tidak perlu takut melakukan upgrade ke versi stabil Rust yang baru. Setiap upgrade harusnya mudah, membawa fitur baru, lebih sedikit bug, dan kompilasi lebih cepat.

### Saluran Rilis dan Model Kereta

Pengembangan Rust beroperasi dengan jadwal kereta. Semua pengembangan dilakukan di branch `master`. Rilis mengikuti model kereta rilis perangkat lunak. Ada tiga saluran rilis Rust:

- Nightly
- Beta
- Stable

Kebanyakan pengembang menggunakan saluran stable. Mereka yang ingin mencoba fitur eksperimental bisa menggunakan nightly atau beta.

Setiap enam minggu, rilis baru disiapkan. Branch `beta` memisahkan diri dari branch `master`. Jika ditemukan masalah di beta, perbaikan diterapkan di `master` (nightly) lalu di-backport ke `beta`. Setelah enam minggu, `beta` menjadi `stable` yang baru, dan prosesnya berulang untuk versi berikutnya.

Proses ini memastikan rilis baru selalu bisa diuji terlebih dahulu di saluran beta sebelum menjadi stabil.

### Waktu Pemeliharaan

Proyek Rust mendukung versi stabil terbaru. Saat versi baru dirilis, versi lama mencapai akhir masa pakainya (EOL). Setiap versi didukung selama enam minggu.

### Fitur Tidak Stabil

Rust menggunakan "feature flags" untuk fitur yang masih dikembangkan. Fitur ini ada di branch `master` dan `nightly` di balik flag tersebut. Kita bisa mencobanya dengan menggunakan Rust nightly dan mengaktifkan flag-nya di kode kita.

Fitur semacam ini tidak bisa digunakan di versi beta atau stable. Ini memungkinkan kami menguji fitur baru secara praktis sebelum dinyatakan stabil selamanya. Buku ini hanya membahas fitur stabil.

### Rustup dan Peran Rust Nightly

Rustup memudahkan kita berganti antar saluran rilis. Secara default kita menggunakan stable. Untuk menginstal nightly:

```console
$ rustup toolchain install nightly
```

kita bisa melihat daftar toolchain dengan `rustup toolchain list`. Kita juga bisa mengatur toolchain tertentu untuk proyek tertentu menggunakan `rustup override set nightly` di direktori proyek tersebut.

### Proses RFC dan Tim

Fitur baru diawali dengan proses *Request For Comments* (RFC). Siapa pun bisa menulis proposal RFC untuk meningkatkan Rust. Proposal tersebut akan diulas dan didiskusikan oleh tim Rust.

Jika disetujui, issue baru dibuka dan fitur tersebut diimplementasikan. Setelah implementasi siap, fitur tersebut masuk ke branch `master` di balik *feature gate*. Setelah diuji oleh pengguna nightly, tim akan memutuskan apakah fitur tersebut layak masuk ke versi stabil atau tidak. Jika ya, feature gate dihapus dan fitur tersebut menjadi stabil.
