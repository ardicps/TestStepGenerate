# QA Test Case Completion — Automation-Ready Prompt Workflow

Dokumen ini berisi cara menggunakan **Claude** untuk melengkapi dokumen test case (`.xlsx`) — mengubah deskripsi skenario satu kalimat menjadi **Test Scenario** step-by-step, **Expected Result** per step, dan **Pre-requisite** siap-otomasi, lengkap dengan highlight perubahan. Workflow ini disusun dari hasil praktik nyata melengkapi test case modul Penjualan (SI/SQ/SO), Pindah Produk, dan Pelanggan pada aplikasi accounting mobile.

Cocok dipakai oleh **QA Automation Engineer** yang punya file test case Excel setengah jadi dan ingin melengkapinya secara konsisten, cepat, dan bisa direview sebelum di-generate ulang.

---

## 1. Struktur File yang Didukung

Workflow ini mengasumsikan file Excel dengan struktur berikut (2 kolom):

| Kolom A: `Testcase` | Kolom B: `Expected Result` |
|---|---|
| Baris 1: header | Baris 1: header |
| Baris 3: **master data** (daftar akun/pelanggan/produk/gudang/role yang jadi rujukan semua skenario) | (kosong) |
| Baris 4+: satu skenario per baris | Hasil yang diharapkan, sejajar dengan skenario |

Kalau skenario di kolom Testcase sudah punya langkah bernomor (`1. ... 2. ...`), Expected Result **harus** punya nomor yang sama persis (1:1), bukan ringkasan satu paragraf.

---

## 2. Prompt Awal (Setup Sesi)

Gunakan prompt ini di awal, sambil melampirkan file `.xlsx`:

```
Bertindaklah sebagai Senior QA Automation Engineer. Saya lampirkan dokumen
test case bernama <nama_file>.xlsx.

Tugas kamu: lengkapi bagian Test Scenario, Expected Result per step, dan
Pre-requisite untuk persiapan otomasi (pra-otomasi).

Sebelum generate file baru, ikuti urutan ini:
1. Analisis pola penulisan yang sudah ada di dokumen (gaya bahasa, format
   penomoran, cara menulis Expected Result).
2. Pecah deskripsi skenario jadi step-by-step yang runtut dan logis,
   masukkan ke kolom Test Scenario.
3. Lengkapi Expected Result 1:1 mengikuti nomor step di Test Scenario.
4. Tambahkan Pre-requisite otomasi HANYA untuk kasus khusus (misal:
   preference/approval harus diaktifkan dulu, kombinasi data master yang
   belum ada) — tulis sebagai teks di dalam sel Testcase, BUKAN kolom baru.
5. Kalau ada gap data (pelanggan/produk/gudang yang disebut skenario tapi
   belum ada di master data), tambahkan ke master data — jangan mengarang
   diam-diam.
6. Kalau ada istilah/singkatan yang ambigu, TANYAKAN dulu, jangan
   diasumsikan sendiri.
7. Tandai setiap sel yang kamu sentuh dengan highlight warna:
   - Biru = konten yang benar-benar baru ditambahkan (dulunya kosong /
     belum ada step sama sekali)
   - Hijau = konten existing yang diperbaiki/dilengkapi (misalnya
     Expected Result yang tadinya cuma ringkasan, jadi step per step)

Tampilkan dulu ringkasan analisis + preview 2-3 contoh baris di chat ini
untuk saya review, sebelum kamu proses seluruh file Excel-nya.
```

**Kenapa harus preview dulu?** Supaya kesalahan asumsi (nama gudang, istilah, gaya penulisan) ketahuan dari 2-3 contoh, bukan setelah 60+ baris selesai digenerate.

---

## 3. Alur Percakapan yang Optimal

Berdasarkan pengalaman di chat ini, urutan berikut menghasilkan output paling akurat:

1. **Claude membaca seluruh isi file** dan mengelompokkan baris menjadi:
   - Sudah lengkap (skip, jangan disentuh)
   - Ada deskripsi tapi belum ada step sama sekali
   - Sudah ada step di Testcase, tapi Expected Result masih ringkasan
2. **Claude melaporkan gap** — istilah yang tidak jelas, data master yang direferensikan tapi tidak terdaftar, kombinasi kondisi yang belum ada.
3. **User mengonfirmasi/mengoreksi** istilah dan keputusan desain (mis. "PD itu penyesuaian diskon", "kalau ada gap data, tambahin ke master data").
4. **Claude memberi preview** (2-3 baris contoh) sebelum full generate.
5. **User approve** → Claude proses seluruh baris sekaligus, lalu apply highlight, lalu kirim file.
6. **User cross-check** hasil sewaktu-waktu ("case yang ini kemana ya?") → Claude tunjukkan lagi dalam bentuk tabel di chat tanpa perlu re-generate file, kecuali user minta revisi.

> Tips: minta Claude urutkan pengerjaan per baris kalau file besar (>30 baris), dan simpan definisi konten di variable/dict per baris (bukan langsung tulis ke Excel) supaya mudah di-review & diperbaiki sebelum file final ditulis.

---

## 4. Konvensi yang Dipakai

### 4.1 Penomoran Step ↔ Expected Result
Step nomor `N` di Testcase **selalu** berpasangan dengan hasil nomor `N` di Expected Result. Kalau originalnya ada nomor yang ter-skip (misal 16 → 18), perbaiki jadi berurutan (dan tandai **hijau** karena termasuk perbaikan konten existing).

### 4.2 Pre-requisite Otomasi
Ditulis sebagai blok teks di **dalam sel Testcase** (bukan kolom baru), format:
```
Pre-requisite:
1. <kondisi data/setting yang harus disiapkan sebelum script jalan>
2. ...

1. <step normal dimulai dari sini>
2. ...
```
Hanya ditulis untuk kasus **khusus** (approval harus aktif, kombinasi data master yang jarang, preference tertentu) — bukan untuk setiap baris.

### 4.3 Master Data (Baris 3)
- Semua entitas yang disebut di skenario (pelanggan, produk, gudang, role/akun) harus terdaftar di sini dengan detail lengkap (harga, diskon, hak akses, dll).
- Kalau skenario butuh kombinasi data yang belum ada (misal "pelanggan tanpa diskon tapi ada PHJ"), **tambahkan entitas baru** ke master data — jangan menaruh detail itu tersebar di tiap baris testcase.
- Setiap penambahan ke master data ditandai **biru**.

### 4.4 Warna Highlight
| Warna | Kode Hex (fill) | Arti |
|---|---|---|
| 🔵 Biru | `FFCCE5FF` | Konten benar-benar baru (dulu kosong / tidak ada step) |
| 🟢 Hijau | `FFC6EFCE` | Konten existing yang diperbaiki/dilengkapi |

---

## 5. Cara Menjalankan (untuk pengguna lain)

1. Siapkan file `.xlsx` dengan struktur di atas (kolom `Testcase` & `Expected Result`, master data di baris 3).
2. Upload file ke Claude (claude.ai, Claude Code, atau Claude Cowork).
3. Kirim **Prompt Awal** di atas.
4. Review ringkasan analisis + preview yang diberikan Claude. Jawab pertanyaan klarifikasi Claude soal istilah/gap data — ini penting, jangan di-skip.
5. Setelah setuju dengan preview, minta Claude lanjut proses seluruh file: *"Oke, lanjutkan proses ke semua baris dan generate file finalnya."*
6. Unduh file `.xlsx` hasilnya dari Claude. Cek warna highlight untuk fokus review ke bagian yang diubah/ditambah.
7. Kalau ada baris yang keliru, cukup sebutkan nomor barisnya dan minta revisi spesifik — tidak perlu regenerate seluruh file.

### Contoh Follow-up Prompt yang Berguna
```
- "Coba lengkapi juga baris X-Y, sepertinya belum ada step-nya."
- "Buat [istilah] itu artinya harus [aturan tertentu], tolong disesuaikan."
- "Tunjukkan lagi test case yang [deskripsi], sekarang ada di baris mana?"
- "Kalau ada gap data, tambahin ke master data, jangan diasumsikan diam-diam."
```

---

## 6. Yang Perlu Dihindari

- ❌ Langsung generate seluruh file tanpa preview → risiko salah asumsi di puluhan baris sekaligus.
- ❌ Menaruh Pre-requisite di kolom terpisah kalau strukturnya tidak dirancang untuk itu — ikuti gaya existing dokumen.
- ❌ Membiarkan Claude menebak istilah domain-specific (singkatan seperti PHJ/PD/KTS) tanpa konfirmasi — selalu klarifikasi dulu.
- ❌ Menambahkan detail data (harga, diskon, hak akses) langsung di baris testcase individual — taruh di master data supaya konsisten dan mudah dirujuk ulang.
- ❌ Lupa highlight — tanpa warna, reviewer tidak bisa cepat fokus ke bagian yang berubah.

---

## 7. Lisensi & Kontribusi

Silakan modifikasi workflow/prompt ini sesuai kebutuhan modul aplikasi kamu. Kontribusi dan perbaikan template prompt dipersilakan lewat pull request.
