# Portal Monitoring Pelanggan — Panduan Setup

Paket ini menambahkan 2 hal baru ke project `DIRECTORY-V8` Anda, **tanpa mengubah**
data atau tabel `orders`/`products`/`tagar_batches` yang sudah ada:

1. **Menu baru "👤 Akun Pelanggan"** di dashboard admin Anda (`index.html`) — untuk
   membuat & mengelola akun login pelanggan.
2. **Website terpisah di folder `/pelanggan`** — khusus pelanggan login dan
   memantau progress langganan harian mereka sendiri (read-only, tidak bisa
   edit apa pun).

## 1. Jalankan SQL (WAJIB, sekali saja)

Buka **Supabase Dashboard → SQL Editor** di project yang sama dengan `config.js`
Anda, lalu jalankan seluruh isi file:

```
sql/customer_portal.sql
```

Ini akan membuat tabel `customer_accounts`, `app_admin_settings`, dan semua
fungsi login/monitoring/reset password. Aman dijalankan di database yang
sudah berisi data `orders` lama Anda — tidak ada data yang terhapus.

## 2. PIN Admin

Menu "Akun Pelanggan" dikunci PIN (default: **123456**). Setelah masuk pertama
kali, langsung ganti lewat bagian "🔑 Ganti PIN Admin" di bagian bawah menu
tersebut. Jangan bagikan PIN atau URL admin ke pelanggan.

## 3. Cara Membuat Akun Pelanggan

1. Buka dashboard admin → menu **Akun Pelanggan** → masukkan PIN.
2. Di form "Buat Akun Pelanggan Baru", pilih nama pelanggan dari dropdown
   **"Pilih dari Pelanggan Aktif"** (otomatis diambil dari data Dashboard Aktif
   BEM/LEM/SEM yang belum punya akun), atau ketik manual di kolom
   "Nama / Kode Pelanggan".
   - Nama `"Pelanggan 88"` dan `"88"` dianggap **sama** secara otomatis.
3. Isi username, nomor WhatsApp (dipakai untuk fitur lupa password), dan
   password (atau klik 🎲 untuk generate otomatis).
4. Klik **Buat Akun**. Setelah berhasil, muncul pop-up berisi username +
   password yang bisa langsung disalin untuk dikirim ke pelanggan lewat WhatsApp.

Pelanggan tersebut, saat login di portal `/pelanggan`, **hanya akan melihat**
baris-baris pesanan miliknya sendiri (dicocokkan otomatis lewat nama pemesan),
dari dashboard BEM, LEM, maupun SEM sekaligus.

## 4. Deploy Portal Pelanggan

Folder `pelanggan/` adalah website statis terpisah (login, dashboard, lupa
password). Upload folder ini ke hosting Anda, misalnya sebagai:
- Subfolder di domain yang sama: `namadomainanda.com/pelanggan`
- Atau subdomain terpisah: `pelanggan.namadomainanda.com`

Bagikan link tersebut + username/password ke masing-masing pelanggan.

## 5. Fitur yang Tersedia untuk Pelanggan

- Login dengan username & password.
- Dashboard **read-only**: ringkasan jumlah langganan aktif, total hari
  terkirim vs total hari paket, dan detail progress per tagar/pesanan
  (termasuk status "sudah dicatat hari ini / belum").
- **Ganti password** sendiri (perlu password lama).
- **Lupa password**: reset mandiri dengan verifikasi nomor WhatsApp yang
  terdaftar di akun (tanpa perlu email).
- Sesi login otomatis berakhir setelah 7 hari (harus login ulang), dan akun
  otomatis terkunci 15 menit setelah 5x salah password berturut-turut.

## 6. Fitur Tambahan yang Bisa Dikembangkan Nanti (saran)

- Notifikasi WhatsApp otomatis (butuh integrasi WhatsApp Business API) saat
  langganan pelanggan mendekati/limit habis.
- Riwayat harian (log setiap kali admin ceklis pengiriman), bukan cuma
  angka total, supaya pelanggan bisa lihat kalender.
- Multi-bahasa / mode gelap pada portal pelanggan.
- Ekspor riwayat langganan pelanggan ke PDF.
- Autentikasi lebih kuat lewat Supabase Auth (email/OTP) jika ke depan
  volume pelanggan sudah besar dan butuh keamanan tingkat produksi.

## Catatan Keamanan

Seperti dashboard admin Anda yang sekarang (tabel `orders` bisa diakses penuh
lewat anon key tanpa login), arsitektur ini juga masih **client-side saja**
(tanpa server backend). Yang sudah diperbaiki di sini:
- Password pelanggan **di-hash** (bukan disimpan polos) dan **tidak pernah**
  bisa dibaca langsung lewat anon key — hanya lewat fungsi database khusus.
- Setiap pelanggan hanya bisa mengambil data miliknya sendiri lewat fungsi
  `get_my_orders`, yang memverifikasi sesi login (bukan sekadar filter di
  browser yang bisa dilewati).
- Menu admin dikunci PIN terpisah dari anon key.

Untuk keamanan tingkat produksi yang lebih tinggi (mis. jika akan dipakai
banyak pelanggan/data sensitif), pertimbangkan migrasi ke Supabase Auth +
Row Level Security berbasis JWT di masa depan.
