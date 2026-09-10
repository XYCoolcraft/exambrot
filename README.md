# Xayz Exam Web

Web ujian online sederhana: fullscreen enforcement, timer, short URL berbatas waktu, scan QR,
dashboard live (jam, baterai, jaringan, FPS, dll), login Google opsional, dan pencatatan pelanggaran.

Dibangun dengan **Node.js v20 + Express (CommonJS)**, storage file JSON, tanpa dependensi berat.

---

## ⚠️ BACA DULU — Apa yang benar-benar bisa & TIDAK bisa dilakukan

Supaya tidak ada ekspektasi keliru saat dipakai untuk ujian sungguhan:

| Fitur | Status | Catatan |
|---|---|---|
| Fullscreen wajib + deteksi keluar fullscreen | ✅ Nyata | Pakai Fullscreen API standar, bekerja di semua browser modern |
| Timer & auto-submit saat waktu habis | ✅ Nyata | |
| Short URL dengan batas waktu (durasi/rentang tanggal) | ✅ Nyata | |
| Scan QR via webcam, auto redirect internal | ✅ Nyata | Pakai library jsQR |
| Login Google, redirect selalu balik ke domain sendiri | ✅ Nyata (opsional, perlu setup sendiri) | |
| Dashboard jam otomatis WIB/WITA/WIT | ✅ Nyata | |
| Battery indicator | ⚠️ Terbatas | Battery Status API sudah dihapus di banyak browser modern (Firefox, Safari, Chrome desktop terbaru). Tampil "N/A" jika tidak didukung. |
| CPU/Memory/Disk/GPU indicator | ⚠️ Terbatas & perkiraan | Browser TIDAK memberi akses ke persentase pemakaian CPU/RAM/disk sistem asli. Yang ditampilkan: jumlah core logis, heap JS (Chrome saja), nama render GPU, kuota storage browser (bukan disk fisik). |
| FPS real-time | ✅ Nyata | |
| Indikator sinyal/kecepatan internet | ⚠️ Terbatas | Network Information API hanya didukung sebagian browser (utamanya Chrome Android). Browser lain hanya tampil status online/offline. |
| Blokir klik kanan / copy / drag / select | ⚠️ Deterrent saja | Ini level tampilan (CSS/JS), bisa dilewati pengguna yang tahu caranya. Bukan proteksi mutlak. |
| **Anti screenshot / anti screen recording** | ❌ **Tidak mungkin** | Tidak ada website manapun di dunia yang bisa benar-benar mencegah ini — screenshot/recording terjadi di level sistem operasi, di luar kendali halaman web. |
| **Blokir DevTools sepenuhnya** | ❌ **Tidak mungkin, hanya heuristik** | Ada heuristik deteksi (ukuran window), tapi bisa dilewati (devtools di layar lain, mode tertentu, dll). |
| **Deteksi/blokir ekstensi browser luar (AI dll)** | ❌ **Tidak tersedia** | Browser tidak menyediakan API untuk website mendeteksi ekstensi pihak ketiga. |
| **Rotasi IP/User-Agent/Proxy otomatis milik pengguna** | ❌ **Sengaja tidak dibuat** | Ini pola teknik bot-evasion/anti-detect, bukan fitur ujian yang wajar, dan tidak diimplementasikan di proyek ini. |

---

## Instalasi & Menjalankan (VPS / Hosting / Lokal)

```bash
npm install
cp .env.example .env
# edit .env: isi ADMIN_PASSWORD, PUBLIC_BASE_URL, dst
npm start
```

Server berjalan di `http://localhost:3000` (atau `PORT` yang diset di `.env`).

## Deploy ke Vercel (Free Tier)

1. Push project ini ke repository GitHub Anda.
2. Buka [vercel.com](https://vercel.com) → **Import Project** → pilih repo ini.
3. Vercel otomatis mendeteksi `vercel.json` (sudah dikonfigurasi untuk free tier: serverless function + static assets).
4. Set Environment Variables di dashboard Vercel (ADMIN_PASSWORD, PUBLIC_BASE_URL, dll — isi manual, sesuai `.env.example`).
5. Deploy.

**Penting soal Vercel:** filesystem serverless hanya bisa menulis ke `/tmp`, dan `/tmp` bisa
hilang kapan saja saat cold start. Artinya data ujian/hasil **bisa hilang** di Vercel. Cocok untuk
demo atau ujian jangka pendek. Untuk produksi jangka panjang, gunakan VPS/Hosting biasa (`npm start`),
atau sambungkan storage eksternal (Vercel KV/Postgres — perlu modifikasi kode `lib/storage.js`).

> Catatan: kami **tidak** mengimplementasikan penyimpanan otomatis token GitHub (`ghp_...`) ke file
> apa pun. Menyimpan token pribadi secara otomatis setiap kali membuat URL adalah risiko keamanan
> besar (kebocoran kredensial). Jika Anda ingin otomasi deploy dari kode, gunakan GitHub Actions
> dengan **GitHub Secrets** resmi (bukan file biasa) dan Vercel CLI/GitHub integration standar.

## Deploy ke GitHub Pages

GitHub Pages **hanya statis** — tidak ada server backend, jadi fitur dinamis (buat ujian, short URL,
simpan hasil) **tidak bisa berjalan** di sana. Workflow `.github/workflows/deploy-pages.yml` hanya
men-deploy halaman info statis di folder `docs/`. Untuk ujian sungguhan, gunakan VPS/Hosting atau Vercel.

## Login Google (Opsional)

1. Buat OAuth Client ID di [Google Cloud Console](https://console.cloud.google.com/apis/credentials).
2. Redirect URI yang didaftarkan: `<PUBLIC_BASE_URL>/auth/google/callback`
3. Isi `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET` di `.env`.
4. Setelah login, peserta **selalu** diarahkan balik ke halaman ujian di domain ini — tidak pernah ke domain lain.

---

## Tutorial Penggunaan

### Sebagai Admin
1. Buka `/admin.html`, login dengan `ADMIN_PASSWORD`.
2. Buat ujian baru: isi judul, durasi, batas pelanggaran, dan soal (format JSON, lihat contoh di form).
3. Generate short URL: pilih ujian, tentukan batas waktu (durasi dari sekarang, atau rentang tanggal spesifik).
4. Bagikan short URL atau tampilkan sebagai QR code ke peserta.
5. Lihat hasil peserta di bagian "Hasil Peserta".

### Sebagai Peserta
1. Buka halaman utama, masukkan URL/kode ujian atau scan QR.
2. Klik "Masuk Fullscreen & Mulai" — wajib fullscreen untuk memulai.
3. Jawab soal; jawaban tersimpan otomatis setiap kali memilih opsi.
4. Jika keluar dari fullscreen atau pindah tab, sistem mencatat sebagai pelanggaran. Terlalu banyak pelanggaran → otomatis didiskualifikasi.
5. Submit manual, atau otomatis ter-submit saat waktu habis.

---

## Struktur Proyek

```
xayz-exam-web/
├── server.js              # Entry point utama (npm start)
├── api/index.js           # Wrapper untuk Vercel serverless
├── vercel.json            # Config Vercel (free tier)
├── lib/                    # Logic: exam, session, shorturl, storage, googleAuth
├── public/                 # Frontend: HTML/CSS/JS statis
├── data/                   # Storage JSON (dibuat otomatis, jangan commit isinya)
├── docs/                   # Halaman statis untuk GitHub Pages
└── .github/workflows/       # CI untuk deploy GitHub Pages
```

## Lisensi

MIT — silakan dimodifikasi sesuai kebutuhan Anda.
