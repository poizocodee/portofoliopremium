# Personal Portfolio / Creative Studio

Portfolio React + TypeScript yang terinspirasi struktur visual Alfari, dengan sistem animasi GSAP / ScrollTrigger dan Lenis. Identitas Vano, foto stock, dan empat concept project adalah konten contoh yang perlu dipersonalisasi sebelum publikasi.

DEMO LIVE : https://vanoportofolio.freedev.app/?i=1 


<img width="2554" height="1149" alt="image" src="https://github.com/user-attachments/assets/6fa8e059-3c3b-4c50-85f9-752fdd6112c1" />


## Jalankan

```sh
npm install
npm run dev
```

Buka `http://localhost:5173`. Website, API, upload, dan admin berjalan pada origin yang sama. Jangan menjalankan Vite secara terpisah karena API disediakan oleh `server/index.mjs`.

## Studio Admin
<img width="2556" height="1182" alt="image" src="https://github.com/user-attachments/assets/fe82e4c0-8f3f-4e48-be44-acec4d6fc60e" />


Buka `/admin`. Login awal memakai akun bawaan (lihat `INITIAL_ADMIN_PASSWORD` di server). Ganti password melalui menu **Keamanan** sebelum publikasi. Password di-hash, sesi disimpan di server, dan perubahan konten tetap dilindungi CSRF. Password yang sudah pernah diubah tidak direset oleh pembaruan aplikasi.

- Profil, foto, identitas, biografi, availability, dan informasi kontak.
- Tambah, edit, hapus project; upload cover dan galeri; custom slug dan tautan website.
- Tambah, hapus, edit, dan urutkan skills / services.
- Edit copy hero, heading section, deskripsi, dan footer.
- Inbox untuk pesan form kontak; tandai dibaca, balas melalui aplikasi email, atau hapus.
- Ganti password dan akhiri sesi pada perangkat lain.

Tidak ada kredensial maupun konten global yang disimpan di localStorage. Form kontak menyimpan pesan di inbox server. Pengiriman email otomatis memerlukan integrasi SMTP/provider email tambahan dan belum diaktifkan.

## Login Dasar (Studio Admin)

Login bawaan saat database pertama kali dibuat (mode Node/lokal):

| Field    | Nilai                                                        |
| -------- | ------------------------------------------------------------ |
| URL      | `http://localhost:5173/admin`                                |
| Username | `admin`                                                      |
| Password | `admin` (atau nilai `INITIAL_ADMIN_PASSWORD` jika diatur)    |

Segera ganti password lewat menu **Keamanan** sebelum publikasi. Untuk password awal yang aman, jalankan dengan `INITIAL_ADMIN_PASSWORD="password-kuat-anda" npm run dev` **sebelum** database pertama kali dibuat.

> Catatan: di InfinityFree (hosting statis) halaman `/admin` tidak bisa login, karena tidak ada server Node/API. Lihat bagian deploy di bawah.

## Deploy ke InfinityFree (pakai folder `dist`)

InfinityFree hanya mendukung PHP/statis, **tidak** menjalankan Node.js. Karena itu yang di-upload hanyalah hasil build frontend (`dist`). Website tampil memakai konten dari `shared/content.json`.

1. Di komputer kamu, jalankan:

   ```sh
   npm install
   npm run build
   ```

   Hasilnya ada di folder `dist/` (berisi `index.html`, `images/`, `favicon.svg`, dan `.htaccess`).

2. Edit konten dulu di `shared/content.json` (nama, bio, kontak, project) **sebelum** build, karena admin tidak aktif di hosting statis.

3. Login ke panel InfinityFree, buka **File Manager** (atau FTP, mis. FileZilla) lalu masuk ke folder `htdocs`.

4. Hapus file bawaan `index2.html` jika ada, lalu upload **isi** folder `dist` (bukan folder `dist` itu sendiri) ke `htdocs`. Pastikan file `.htaccess` ikut ter-upload (aktifkan tampilan file tersembunyi).

5. Buka domain kamu. Route seperti `/projects/...` dialihkan ke `index.html` oleh `.htaccess`.

Yang **tidak** berfungsi di InfinityFree: `/admin`, upload gambar, dan inbox form kontak (butuh `/api`). Untuk fitur penuh, deploy ke hosting Node dengan persistent disk (lihat bagian Production).

## Production

```sh
npm run typecheck
npm test
npm run test:browser
npm run build
NODE_ENV=production PORT=3000 DATA_DIR=/path/to/persistent-data npm start
```

Gunakan Node.js 20+, hosting Node yang mendukung persistent volume, dan HTTPS melalui reverse proxy. `DATA_DIR` harus berupa direktori yang dapat ditulis dan tetap tersimpan saat deploy atau restart. Default-nya `./data`.

`INITIAL_ADMIN_PASSWORD` dapat mengubah password awal saat database **pertama kali dibuat**, agar akun publik tidak pernah memakai password bawaan. Password dapat diganti melalui menu Keamanan. Atur ini melalui environment sebelum deployment publik. Mengubah environment setelah database terbentuk tidak mereset password.

`SESSION_SECRET` opsional, minimum 32 karakter acak. Jika tidak disediakan, secret acak dibuat dan disimpan dengan izin `0600` pada `DATA_DIR/.session-secret`. Jangan menghapus secret ini tanpa bermaksud mengakhiri semua sesi.

SQLite menyimpan konten, akun admin, sesi, dan inbox di `DATA_DIR/portfolio.sqlite`. Gambar tersimpan di `DATA_DIR/uploads`. Lakukan backup seluruh direktori data; hentikan server sebelum menyalin database, atau gunakan SQLite Online Backup API. Database dan upload tidak boleh dimasukkan ke Git.

**Batas deployment:** SQLite lokal cocok untuk satu instance Node dengan persistent disk. Static-only hosting, ephemeral serverless, atau beberapa instance dengan disk terpisah tidak menyediakan persistensi global. Untuk lingkungan tersebut migrasikan ke PostgreSQL dan object storage bersama. Build frontend saja bukan deployment fullstack.

## Motion Dan Aksesibilitas

GSAP mengatur entrance bertahap, clip-path reveal, transform berbasis scroll, magnetic buttons, tilt, cursor project, footer reveal, dan transisi route. Lenis terhubung ke ticker GSAP yang sama. Native touch scrolling dipertahankan. `prefers-reduced-motion` menonaktifkan smoothing, pinning, reveal, dan motion dekoratif. Tidak memakai Three.js atau Rive karena efek ini tidak membutuhkan WebGL.

Dialog memakai elemen native `dialog`, keyboard focus trap, Escape, dan scroll isolation. Navbar mobile, filter project, form, dan admin mendukung keyboard. Gambar memakai dimensi tetap, WebP, dan lazy loading di bawah fold.

## Struktur

```text
src/App.tsx                  Routing, content synchronization, transitions
src/pages/Portfolio.tsx      Home, About, Skills, Projects, Contact
src/pages/ProjectDetail.tsx  Project stories and image gallery
src/pages/Admin.tsx          Studio authentication and CMS
src/components/Motion.tsx   Shared GSAP / Lenis motion system
src/components/InteractiveLanyard.tsx  Editable, interactive identity lanyard
src/lib/api.ts              Same-origin API and CSRF transport
server/index.mjs            Express API, validation, uploads, Vite integration
server/database.mjs         SQLite schema, sessions, scrypt credentials
shared/content.json         Initial editable demo content
tests/api.test.mjs          Isolated API and restart-persistence tests
```

Content tersimpan di server dan langsung tersedia melalui `/api/content`. Website mengambil pembaruan saat dibuka, saat tab mendapat fokus, dan setiap 45 detik. Admin memperbarui tampilan lokal setelah menyimpan. Seed JSON hanya digunakan ketika database pertama kali dibuat.

## Aset

Foto stock dari Pexels: portrait oleh Lil K (18165007), interior oleh Thoi Nam Cao (15867752), skincare oleh Los Muertos Crew (7615819). Foto portrait adalah model stock, bukan foto identitas pemilik portfolio.

Empat screenshot concept project merupakan komposisi UI original yang dibuat untuk konten contoh. `node scripts/generate-previews.mjs` merender ulang cover; memerlukan Playwright Chromium dan dependensi browser (`npx playwright install --with-deps chromium`). Jangan menyatakan concept project sebagai pekerjaan klien nyata.

Font: Manrope, Space Grotesk, dan Barlow Condensed, self-hosted melalui Fontsource. 

## Pembaruan Interaksi Dan CMS

- Kartu memakai tali SVG yang dihitung dari posisi pengait aktual. Drag mendukung mouse dan touch, auto-scroll di tepi layar, spring return, tombol reset, serta tombol panah dan Home untuk keyboard. Tidak memakai Three.js.
- Foto biasa dan Black Suit memiliki pose yang terdaftar sama pada aset contoh. Efek reveal mengikuti pointer; tombol pada foto menyediakan interaksi untuk touch dan keyboard. Ini pergantian dua gambar, **bukan AI real-time**. Untuk foto baru, unggah pasangan alternatif dengan pose dan crop identik.
- Tanda tangan beranimasi di sudut foto dapat berupa teks handwriting atau upload PNG transparan.
- Editor Kartu Identitas mengatur teks cetak, tali, foto biasa/alternatif, logo, tanda tangan, warna kertas, dan warna aksen.
- Riwayat sekolah mendukung tambah, edit, hapus, pengurutan, dan upload logo. Tiga sekolah awal adalah **data contoh**, bukan riwayat asli pemilik.
- Frameworks mendukung tambah, edit, hapus, pengurutan, ikon bawaan, upload logo, dan deskripsi interaktif.
- Konten Halaman juga menyediakan editor labels & microcopy untuk navigasi, CTA, filter, form kontak, dan footer.
- Migrasi database menambahkan field baru tanpa menimpa profil, project, atau password yang sudah disimpan.

`tests/enhancements-browser.mjs` memeriksa keterhubungan tali secara geometris, long drag dan auto-scroll, Black Suit, tanda tangan, editor sekolah/framework/kartu, upload, dan visibilitas konten untuk sesi pengunjung terpisah.

Aset Black Suit contoh adalah ilustrasi overlay yang dibuat sesuai koordinat foto stock. Regenerasi melalui `node scripts/generate-symbiote.mjs`. Gambar normal dan alternatif dapat diganti dari studio. Font handwriting Caveat hanya memuat subset Latin WOFF2.

## Bahasa (150 bahasa)

Di Studio Admin buka tab **Bahasa**: pilih bahasa dasar website dari 150 bahasa (`shared/languages.json`). Terjemahan memakai Google Website Translator (mesin, perlu internet; bahasa `id` = konten asli tanpa terjemahan). Pengunjung hanya melihat tombol bahasa di header jika admin mengaktifkan **Mode bahasa**; jika nonaktif, semua pengunjung memakai bahasa dasar. Halaman admin tidak diterjemahkan.
