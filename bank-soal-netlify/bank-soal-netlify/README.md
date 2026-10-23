# Bank Soal Auto-Search — Netlify + Python API + Excel

Proyek ini memisahkan frontend statis (Netlify) dan backend Python (Render). Netlify tidak menjalankan Flask/FastAPI sebagai server web biasa; backend disediakan sebagai layanan terpisah.

## Struktur
- `index.html` — antarmuka pencarian.
- `netlify.toml` — konfigurasi publish statis.
- `backend/app.py` — Flask API membaca Excel dan mencari kemiripan pertanyaan.
- `backend/requirements.txt` — dependensi Python.
- `backend/render.yaml` — konfigurasi deploy backend di Render.
- `backend/data/` — tempat file Excel bank soal lokal saat pengembangan.
- `backend/template_bank_soal.xlsx` — template kolom Soal/Jawaban.

## 1. Siapkan Excel
1. Buka `backend/template_bank_soal.xlsx` atau siapkan file Excel sendiri.
2. Baris pertama harus berupa judul kolom. Disarankan `Soal` dan `Jawaban`.
3. Header yang juga dikenali antara lain `Pertanyaan`, `Question`, `Kunci Jawaban`, dan `Answer`.
4. Simpan file `.xlsx` ke `backend/data/`. Anda boleh memasukkan beberapa file; backend akan membaca semua file `.xlsx` dan `.xlsm` di folder tersebut.
5. Jangan gunakan data yang tidak boleh dilihat pengguna situs. Endpoint pencarian mengembalikan teks soal dan jawaban.

## 2. Uji backend di komputer (opsional)
Pasang Python 3.11 atau lebih baru. Buka terminal pada folder proyek:

```bash
cd backend
python -m venv .venv
```

Windows:
```powershell
.venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

macOS/Linux:
```bash
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

API akan tersedia di `http://127.0.0.1:8000`. Buka `http://127.0.0.1:8000/health`; pastikan `records` lebih dari 0. Letakkan Excel di `backend/data/` sebelum menjalankan pengujian.

## 3. Deploy backend ke Render
**Penting:** file Excel yang ada di komputer tidak ikut tersedia otomatis pada server online. Cara paling sederhana untuk data statis adalah memasukkan file Excel yang memang boleh dipublikasikan ke folder `backend/data/` sebelum mengunggah proyek ke repositori Git.

1. Buat repositori GitHub baru dan unggah seluruh isi folder proyek ini.
2. Buka https://dashboard.render.com/ lalu pilih **New → Blueprint** dan hubungkan repositori tersebut. Render membaca `backend/render.yaml`.
3. Jika diminta, isi environment variable `ACCESS_CODE` dengan kode akses yang Anda tentukan. Jika dibiarkan kosong, endpoint tidak meminta kode akses. Jangan tulis kode rahasia di HTML.
4. Tunggu deploy selesai. Buka URL layanan Render dan tambahkan `/health`, misalnya `https://nama-layanan.onrender.com/health`.
5. Pastikan respons JSON menampilkan `"ok": true` dan `"records"` lebih dari nol. Jika nol, periksa apakah file Excel sudah ada di `backend/data/` pada repositori.

Render free services dapat tidur saat tidak aktif dan perlu waktu untuk bangun. File sistem lokal di layanan hosting juga bukan penyimpanan permanen untuk perubahan data. Untuk bank soal yang sering diperbarui, pertimbangkan penyimpanan persisten atau database.

## 4. Deploy frontend ke Netlify
1. Buka https://app.netlify.com/.
2. Pilih **Add new site → Deploy manually**.
3. Unggah folder proyek yang berisi `index.html` dan `netlify.toml` (jika antarmuka meminta folder hasil build, pilih folder root proyek ini).
4. Setelah situs aktif, buka URL `https://nama-situs.netlify.app`.
5. Pada kolom **URL Server**, masukkan URL backend Render tanpa `/health` atau `/api/cari`, misalnya `https://nama-layanan.onrender.com`, lalu tekan **Simpan**.
6. Jika Anda menetapkan `ACCESS_CODE`, masukkan kode yang sama pada kolom Kode akses dan tekan **Masuk**.
7. Tempel soal dan tekan **Cari Soal**.

## API
- `GET /health` — status API dan jumlah baris soal yang terbaca.
- `POST /api/cari` — body JSON `{"text":"teks soal"}`; header opsional `X-Kode`.
- Respons pencarian: `{"c":[{"q":"soal","a":"jawaban","sim":0.9,"sh":["file.xlsx"]}]}`.

## Catatan keamanan dan kualitas
- `ACCESS_CODE` adalah gerbang sederhana, bukan sistem akun penuh. Gunakan kode yang kuat dan jangan menyimpan rahasia di HTML.
- CORS disiapkan untuk memudahkan deploy awal. Untuk produksi, batasi origin ke domain Netlify Anda di `backend/app.py`.
- Skor kemiripan adalah perkiraan berbasis teks, bukan jaminan kebenaran. Hasil 100% hanya menunjukkan teks ternormalisasi sama persis.
- Pastikan semua nama kolom dan isi Excel benar. Backend hanya bisa menjawab berdasarkan data yang ada di Excel.
