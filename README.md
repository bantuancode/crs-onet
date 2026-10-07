# crs-onet — texin, find your career

Sistem rekomendasi karier untuk siswa SMA berbasis data O*NET 31.0 (capstone project).
Siswa menjawab 60 pertanyaan minat (RIASEC, dari O*NET Interest Profiler Short Form), melihat
profil minatnya, lalu mendapat sepuluh karier yang paling cocok beserta persentase kecocokan,
alasannya, kompetensi dan pendidikan yang dibutuhkan, serta jaringan karier terkait.

Metode: Content-Based Filtering. Enam skor minat siswa dibandingkan dengan profil minat tiap
pekerjaan memakai Pearson Correlation, cara yang juga dipakai O*NET Interest Profiler.
"Kecocokan" di layar adalah korelasi itu dalam persen (korelasi negatif ditampilkan 0%).

## Menjalankan di komputer sendiri

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/uvicorn web.app:app --reload
```

Situs terbuka di `http://127.0.0.1:8000`. Tampilannya HTML + Tailwind CSS (`web/static/`), dilayani FastAPI
(`web/app.py`).

Kalau mengubah class Tailwind di `web/static/js/` atau `web/tailwind.css`, bangun ulang CSS-nya
(butuh Node.js):

```bash
npm install      # sekali saja
npm run css      # atau: npm run css:watch
```

Hasilnya, `web/static/tailwind.css`, ikut di-commit, jadi server hosting tidak butuh Node.js.

## Akun dan riwayat

- Siswa mendaftar dengan nama, email, dan password, lalu masuk dengan email dan password.
- Yang disimpan di basis data: nama, email, hash password (scrypt, bukan password aslinya), dan tiap
  hasil tes (60 jawaban dan ringkasan tiga rekomendasi teratas) atas nama akun itu.
- Sesi masuk memakai cookie `HttpOnly`; basis data hanya menyimpan hash token-nya. Tanpa "Ingat saya"
  sesi berakhir saat browser ditutup (paling lama 12 jam), dengan "Ingat saya" 30 hari.
- Sepuluh password salah berturut-turut untuk satu email memblokir percobaan masuk selama 15 menit.
- Semua yang dilihat siswa setelah masuk mengikuti akunnya ke perangkat mana pun: tes yang sedang
  dikerjakan (jawaban sejauh ini), hasil terakhir, riwayat, dan feedback yang sudah diberikan.
- Feedback (dua rating bintang dan komentar) disimpan bersama hasil tes yang dinilai.
- Belum ada: masuk dengan Google, lupa password, ubah password, dan hapus akun.

Browser juga menyimpan salinan tes yang sedang dikerjakan (localStorage) supaya halaman cepat dan
memuat ulang tidak menghapus kemajuan; salinan di server yang berlaku saat pindah perangkat.

Tanpa `DATABASE_URL`, semua disimpan di `data/crs.sqlite3` (tidak ikut di-commit).

Tes: `.venv/bin/python -m unittest discover tests`

## Memasang secara daring

Situs ini butuh hosting yang bisa menjalankan Python, misalnya Render atau Railway.

- Perintah pasang: `pip install -r requirements.txt`
- Perintah jalan: `uvicorn web.app:app --host 0.0.0.0 --port $PORT`
- Wajib: variabel lingkungan `DATABASE_URL` berisi alamat Postgres (misalnya Neon). Tanpa itu akun dan
  riwayat ditulis ke berkas di server hosting, yang hilang setiap aplikasi dinyalakan ulang. Tabel dibuat otomatis.
- Opsional: `ADMIN_TOKEN` untuk mengunduh feedback.
- Situs harus dibuka lewat HTTPS (bawaan di Render dan Railway), supaya cookie masuk hanya dikirim terenkripsi.

Kata sandi dan alamat basis data jangan pernah di-commit.

## Mengambil feedback

Dengan variabel lingkungan `ADMIN_TOKEN` terisi:

- `https://<alamat>/api/admin/feedback.csv?token=<ADMIN_TOKEN>`: feedback atas hasil tes (minat terkuat,
  rekomendasi teratas, dua rating, komentar), tanpa nama dan email

## Isi repo

| Path | Isi |
|---|---|
| `web/app.py` | Server FastAPI: konfigurasi, rekomendasi, detail, jaringan karier, feedback, ekspor |
| `web/static/` | Tampilan: `index.html`, `tailwind.css` (hasil bangun), gambar |
| `web/static/js/` | JavaScript dalam modul: `main.js` (rute dan event), `state.js`, `sync.js`, `router.js`, `api.js`, `components/` (bagian yang dipakai bersama), `views/` (satu berkas per halaman) |
| `web/tailwind.css`, `package.json` | Sumber CSS dan perintah bangunnya |
| `crs/questionnaire.py` | 60 butir minat dan penghitungan skor RIASEC |
| `crs/recommender.py` | Pearson Correlation antara profil siswa dan profil pekerjaan; kompetensi penting tiap pekerjaan |
| `crs/service.py` | Isi halaman hasil, alasan kecocokan, jaringan karier, detail pekerjaan, dan ekspor |
| `crs/accounts.py` | Hash password, token masuk, pembatas percobaan masuk |
| `crs/storage.py` | Penyimpanan akun, tes yang sedang dikerjakan, hasil, dan feedback (SQLite atau Postgres) |
| `crs/i18n/` | Terjemahan nama pekerjaan, deskripsi, dan kompetensi |
| `data/` | Tabel O*NET 31.0 (pekerjaan, rating, pekerjaan terkait, pendidikan, gaya kerja, perangkat lunak), diolah dari RDF resmi O*NET |
| `tests/` | Tes otomatis |

## Atribusi

Aplikasi ini memuat informasi dari O*NET 31.0 Database dan O*NET Career Exploration Tools oleh
U.S. Department of Labor, Employment and Training Administration (USDOL/ETA), digunakan di bawah
lisensi CC BY 4.0 dan O*NET Tools Developer License. O*NET® adalah merek dagang USDOL/ETA.
Pengembang aplikasi ini telah mengubah sebagian informasi tersebut. USDOL/ETA tidak menyetujui,
mendukung, atau menguji perubahan ini.
