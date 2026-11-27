# Nuvella Caffe

Situs pre-order & reservasi untuk Nuvella Caffe, Madiun. Tanpa framework dan tanpa build step: HTML + CSS + JavaScript (ES modules).

## Menjalankan

ES modules tidak bisa dibuka dengan klik dua kali pada `index.html`. Jalankan lewat server lokal, salah satunya:

- VS Code: pasang ekstensi **Live Server**, klik **Go Live**
- Terminal: `python3 -m http.server` lalu buka `http://localhost:8000`
- Terminal: `npx serve`

Untuk produksi, unggah seluruh folder ke hosting statis mana pun (Netlify, GitHub Pages, Vercel, dsb.).

## Struktur

```
nuvella-caffe/
├── index.html
├── assets/
│   ├── foto/            foto menu sendiri, nama file = id menu (contoh: pour-tiramisu.jpg)
│   └── audio/           sound.wav (suara tambah keranjang; kalau tidak ada, pakai suara bawaan)
├── css/
│   ├── main.css         daftar @import — URUTAN PENTING (lihat catatan di bawah)
│   ├── base.css         variabel warna/font, reset, tipografi
│   ├── components/      buttons, navigation, toast
│   ├── layout/          footer
│   ├── pages/           stylesheet per halaman (menu, checkout, faq, cake-studio, ...)
│   ├── themes/          lapisan tema (3D, beranda, coklat)
│   └── effects/         hujan bunga/kue
└── js/
    ├── main.js          titik masuk: menyalakan semua modul
    ├── config/          constants.js (nomor WhatsApp, lokasi aset)
    ├── data/            menu, jam operasional, FAQ, opsi Cake Studio
    ├── state/           store.js (state bersama), cart.js, cake.js
    ├── utils/           dom, format (rupiah, pad, ...), storage, color, icons, env
    ├── core/            router, events, toast, audio
    ├── effects/         animasi & visual (kedai 3D, latar, hujan kue, tilt, reveal)
    └── features/        menu, cart, checkout, order-status, reservation, faq, hours,
                         mascot, intro, cake-studio/ (svg, pricing, studio, order)
```

## Mengubah isi

| Yang ingin diubah | File |
|---|---|
| Menu, harga, kategori | `js/data/menu.js` |
| Jam operasional | `js/data/hours.js` |
| Pertanyaan FAQ | `js/data/faq.js` |
| Nomor WhatsApp (pesanan / Cake Studio) | `js/config/constants.js` |
| Akun admin & alamat database pesanan | `js/config/constants.js` |
| Jumlah menu per kategori (sekarang 3) | `PER_CATEGORY` di `js/features/menu.js` |
| Harga & pilihan Cake Studio | `js/data/cake-options.js` |
| Warna & font utama | `css/base.css` (variabel di `:root`) |

## Cara kerja singkat

- **Router** (`core/router.js`) membaca hash URL (`#/menu`, `#/keranjang`). Setiap halaman mendaftar di `main.js` lewat `registerPage(...)`, jadi router tidak perlu mengenal fitur satu per satu.
- **State bersama** ada di satu objek `state` (`state/store.js`), disimpan ke `localStorage` untuk keranjang, pesanan, dan desain kue.
- **Antar-modul**: modul yang akan saling mengimpor (misalnya checkout → keranjang) memakai pesan sederhana di `core/events.js` (`emit("cart:changed")`), sehingga tidak ada impor melingkar.
- Setiap fitur punya fungsi `initXxx()` yang dipanggil sekali dari `main.js`.

## Catatan

- **Urutan CSS penting.** File CSS asli berlapis: blok belakang sengaja menimpa blok depan (tema 3D, tema coklat, dst.). Pemecahan ini mempertahankan urutan itu persis. Menggabungkan atau mengurutkan ulang file di `css/main.css` bisa mengubah tampilan. Merapikan lapisan-lapisan itu jadi satu tema adalah pekerjaan terpisah.
- **Kode yang tidak aktif** dibiarkan apa adanya: intro maskot (`features/intro.js`) hanya jalan bila ada elemen `#splash` di HTML, dan `initHeroParallax` (`effects/pointer.js`) mencari elemen `.hero` yang sekarang tidak ada. Aman dihapus kalau memang tidak dipakai lagi.
- **Lokasi aset berubah**: foto menu sekarang di `assets/foto/` (dulu `foto/`) dan suara di `assets/audio/sound.wav` (dulu `sound.wav`). Pindahkan file lamamu ke lokasi itu, atau ubah `FOTO_DIR` / `SFX_FILE` di `js/config/constants.js`.

## Admin pesanan

Buka `#/admin` (contoh `http://localhost:8000/#/admin`), login dengan email & kata sandi admin. Admin bisa melihat pesanan, mengubah status (Diterima → Disiapkan → Siap diambil → Selesai), dan menghapus pesanan. Status itu yang tampil di halaman Keranjang pelanggan.

- Password tidak disimpan di kode, hanya hash SHA-256 dari `email:password` (`ADMIN_PASS_HASH`). Untuk ganti password, hitung ulang hash itu.
- Tanpa `ORDERS_DB_URL`, pesanan hanya tersimpan di browser yang sama (untuk uji coba). Agar pesanan dari HP pelanggan masuk ke admin, buat Firebase Realtime Database dan isi `ORDERS_DB_URL`.
- Login ini hanya pengaman di sisi tampilan; untuk keamanan penuh data pesanan perlu backend dengan autentikasi.
