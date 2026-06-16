# GB-02 — Matrix Testing
## Sistem: SaPoPoe FINANCE (Midnight Finance)
## Teknik: Gray Box Testing — Matrix Testing

---

> **Definisi Teknik:**
> Matrix Testing adalah teknik **pengujian perangkat lunak yang sistematis dan terstruktur untuk menguji berbagai kombinasi input dan kondisi dalam suatu aplikasi**. Teknik ini membantu mengidentifikasi bug yang disebabkan oleh interaksi antara parameter atau faktor yang berbeda dalam aplikasi.
>
> — Materi Pertemuan, Software Quality, T Informatika UKRI

---

## Langkah Skenario Matrix Testing

1. **Definisikan Parameter dan Kondisi**
2. **Membuat Tabel Matriks**
3. **Menjalankan Test Case**
4. **Analisis Hasil**

---

## Modul 1 — Autentikasi (Form Login)

### 1. Definisikan Parameter dan Kondisi

- **Parameter:**
  - Email: Terdaftar & format valid / Tidak terdaftar atau kosong
  - Password: Benar (sesuai akun) / Salah atau kosong
  - Format Request: Lengkap (semua field diisi) / Tidak lengkap (ada field kosong)

- **Kondisi:**
  - Kombinasi parameter (misalnya, email terdaftar + password benar + form lengkap)

### 2. Membuat Tabel Matriks

| Email | Password | Format Request |
|---|---|---|
| Terdaftar | Benar | Lengkap |
| Terdaftar | Benar | Tidak Lengkap |
| Terdaftar | Salah | Lengkap |
| Terdaftar | Salah | Tidak Lengkap |
| Tidak Terdaftar | Benar | Lengkap |
| Tidak Terdaftar | Benar | Tidak Lengkap |
| Tidak Terdaftar | Salah | Lengkap |
| Tidak Terdaftar | Salah | Tidak Lengkap |

### 3. Menjalankan Test Case

Untuk setiap kombinasi parameter dan kondisi dalam tabel matriks, jalankan test case yang sesuai untuk menguji fungsionalitas login. Catat hasil test case (lulus/gagal) di kolom "Status" pada tabel matriks.

| TC | Email | Password | Format Request | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | Terdaftar (`sultan@test.com`) | Benar (`password123`) | Lengkap | Login berhasil → Dashboard | Dashboard berhasil ditampilkan | ✅ Passed |
| 2 | Terdaftar (`sultan@test.com`) | Benar (`password123`) | Tidak Lengkap (tanpa header) | Login berhasil → Dashboard | Dashboard berhasil ditampilkan | ✅ Passed |
| 3 | Terdaftar (`sultan@test.com`) | Salah (`salahpassword`) | Lengkap | Ditolak — password salah | "Kata sandi yang Anda masukkan salah." | ✅ Passed |
| 4 | Terdaftar (`sultan@test.com`) | Salah (`salahpassword`) | Tidak Lengkap | Ditolak — password salah | "Kata sandi yang Anda masukkan salah." | ✅ Passed |
| 5 | Tidak Terdaftar (`tidakada@test.com`) | Benar (`password123`) | Lengkap | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |
| 6 | Tidak Terdaftar (`tidakada@test.com`) | Benar (`password123`) | Tidak Lengkap | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |
| 7 | Tidak Terdaftar (`tidakada@test.com`) | Salah (`salahpassword`) | Lengkap | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |
| 8 | Tidak Terdaftar (`tidakada@test.com`) | Salah (`salahpassword`) | Tidak Lengkap | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |

### Screenshot Bukti — TC1 (Login Berhasil)

<img width="1920" height="1080" alt="matrix-auth-tc1" src="https://github.com/user-attachments/assets/34e6ddd5-837e-4ad7-a938-a2a76c635e76" />


### 4. Analisis Hasil Test

- **Tidak ada kombinasi parameter yang menyebabkan test case gagal.**
- Sistem memprioritaskan validasi email terlebih dahulu — jika email tidak ditemukan, password tidak diperiksa lebih lanjut.
- Format request (lengkap/tidak lengkap header) tidak mempengaruhi hasil karena Laravel menangani request tanpa `Content-Type` secara graceful.
- **Perbarui tabel matriks:** seluruh 8 kombinasi Passed — tidak ada bug baru ditemukan.

---

## Modul 2 — Transfer (Pindah Dana)

### 1. Definisikan Parameter dan Kondisi

- **Parameter:**
  - Amount (Nominal): Valid — `50.000` (di atas minimum) / Invalid — `0` (batas bawah)
  - Saldo Brankas Sumber: Mencukupi (Mandiri) / Tidak Mencukupi (BCA saldo minus)
  - Brankas Tujuan: Dipilih (BSI) / Tidak Dipilih

- **Kondisi:**
  - Kombinasi parameter (misalnya, amount valid + saldo mencukupi + tujuan dipilih)

### 2. Membuat Tabel Matriks

| Amount | Saldo Sumber | Brankas Tujuan |
|---|---|---|
| Valid | Mencukupi | Dipilih |
| Valid | Mencukupi | Tidak Dipilih |
| Valid | Tidak Mencukupi | Dipilih |
| Valid | Tidak Mencukupi | Tidak Dipilih |
| Invalid | Mencukupi | Dipilih |
| Invalid | Mencukupi | Tidak Dipilih |
| Invalid | Tidak Mencukupi | Dipilih |
| Invalid | Tidak Mencukupi | Tidak Dipilih |

### 3. Menjalankan Test Case

Untuk setiap kombinasi parameter dan kondisi dalam tabel matriks, jalankan test case yang sesuai untuk menguji fungsionalitas transfer. Catat hasil test case (lulus/gagal) di kolom "Status" pada tabel matriks.

| TC | Amount | Saldo Sumber | Tujuan | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | Valid — `50.000` | Mencukupi (Mandiri) | Dipilih (BSI) | Transfer berhasil | "BERHASIL Dana berhasil dipindahkan!" | ✅ Passed |
| 2 | Valid — `50.000` | Mencukupi (Mandiri) | Tidak Dipilih | Ditolak — tujuan wajib dipilih | "Harap isi bidang ini." | ✅ Passed |
| 3 | Valid — `50.000` | Tidak Mencukupi (BCA) | Dipilih (BSI) | Ditolak — saldo tidak cukup | "GAGAL Saldo dompet asal tidak mencukupi..." | ✅ Passed |
| 4 | Valid — `50.000` | Tidak Mencukupi (BCA) | Tidak Dipilih | Ditolak — tujuan wajib dipilih | "Harap isi bidang ini." | ✅ Passed |
| 5 | Invalid — `0` | Mencukupi (Mandiri) | Dipilih (BSI) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 6 | Invalid — `0` | Mencukupi (Mandiri) | Tidak Dipilih | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 7 | Invalid — `0` | Tidak Mencukupi (BCA) | Dipilih (BSI) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 8 | Invalid — `0` | Tidak Mencukupi (BCA) | Tidak Dipilih | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |

### Screenshot Bukti — TC1 (Transfer Berhasil)

<img width="1920" height="1080" alt="matrix-transfer-tc1" src="https://github.com/user-attachments/assets/18fbed88-bc07-42ff-b468-9fe25de136f5" />


### 4. Analisis Hasil Test

- **Tidak ada kombinasi parameter yang menyebabkan test case gagal.**
- Validasi amount berjalan di layer frontend sebelum request dikirim — jika amount = 0, semua kombinasi lainnya tidak relevan karena form tidak dikirim.
- Validasi saldo berjalan di layer backend setelah form valid — jika saldo tidak cukup, sistem menolak dengan pesan informatif.
- **Catatan:** GET riwayat transfer (`GET /api/transfers`) masih mengembalikan HTTP 500 — bug di `TransferController@index()` yang tidak tertangkap di skenario POST ini.
- **Perbarui tabel matriks:** seluruh 8 kombinasi Passed untuk operasi POST transfer.

---

## Modul 3 — Transaksi (Catat Aliran Dana)

### 1. Definisikan Parameter dan Kondisi

- **Parameter:**
  - Amount (Nominal): Valid — `50.000` / Invalid — `0` atau kosong
  - Tipe Transaksi: Income (MASUK) / Expense (KELUAR)
  - Portofolio: Dipilih (BSI) / Tidak Dipilih

- **Kondisi:**
  - Kombinasi parameter (misalnya, amount valid + type expense + portofolio dipilih)

### 2. Membuat Tabel Matriks

| Amount | Tipe Transaksi | Portofolio |
|---|---|---|
| Valid | Income | Dipilih |
| Valid | Income | Tidak Dipilih |
| Valid | Expense | Dipilih |
| Valid | Expense | Tidak Dipilih |
| Invalid | Income | Dipilih |
| Invalid | Income | Tidak Dipilih |
| Invalid | Expense | Dipilih |
| Invalid | Expense | Tidak Dipilih |

### 3. Menjalankan Test Case

Untuk setiap kombinasi parameter dan kondisi dalam tabel matriks, jalankan test case yang sesuai untuk menguji fungsionalitas catat transaksi. Catat hasil test case (lulus/gagal) di kolom "Status" pada tabel matriks.

| TC | Amount | Tipe | Portofolio | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | Valid — `50.000` | Income | Dipilih (BSI) | Transaksi berhasil, saldo BSI +50.000 | Saldo BSI bertambah Rp 50.000 | ✅ Passed |
| 2 | Valid — `50.000` | Income | Tidak Dipilih | Ditolak — portofolio wajib | "Pilih item pada daftar." | ✅ Passed |
| 3 | Valid — `50.000` | Expense | Dipilih (BSI) | Ditolak — saldo tidak mencukupi | ⚠️ HTTP 201 — **saldo BSI berkurang menjadi negatif** | 🔴 **Failed** |
| 4 | Valid — `50.000` | Expense | Tidak Dipilih | Ditolak — portofolio wajib | "Pilih item pada daftar." | ✅ Passed |
| 5 | Invalid — `0` | Income | Dipilih (BSI) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 6 | Invalid — `0` | Income | Tidak Dipilih | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 7 | Invalid — `0` | Expense | Dipilih (BSI) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 8 | Invalid — `0` | Expense | Tidak Dipilih | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |

### Screenshot Bukti — TC1 (Transaksi Income Berhasil)

<img width="1920" height="1080" alt="matrix-transaksi-tc1" src="https://github.com/user-attachments/assets/430df2b8-18ac-4977-98e2-bb021aca2511" />


### 4. Analisis Hasil Test

- **Ditemukan 1 kombinasi yang menyebabkan test case gagal: TC3 (Valid × Expense × Dipilih).**
- Investigasi lebih lanjut: akar penyebab kegagalan adalah **`TransactionController::store()` tidak memiliki pengecekan saldo sebelum mencatat expense** — backend langsung mengurangi saldo tanpa validasi kecukupan dana.
- Kombinasi `amount valid + type expense + portofolio dipilih` adalah kombinasi yang paling umum digunakan pengguna nyata untuk mencatat pengeluaran, sehingga ini adalah **defect kritis yang berdampak langsung pada pengguna**.
- **Perbarui tabel matriks:** 7/8 Passed, 1 Failed (TC3). Rekomendasi perbaikan:
```php
if ($request->type === 'expense' && $wallet->balance < $request->amount) {
    return response()->json(['error' => 'Saldo tidak mencukupi'], 400);
}
```

---

## Modul 4 — Tabungan (Target Impian)

### 1. Definisikan Parameter dan Kondisi

- **Parameter:**
  - Nama Tabungan: Valid — `TabunganMT` (1–255 karakter) / Invalid — kosong (`""`) atau panjang (>255 karakter)
  - Target Amount: Valid — `500.000` (lebih dari 0) / Invalid — `0` (batas bawah)
  - Brankas Sumber: Dipilih (BSI) / Tidak Dipilih

- **Kondisi:**
  - Kombinasi parameter (misalnya, nama valid + target valid + brankas dipilih)

### 2. Membuat Tabel Matriks

| Nama Tabungan | Target Amount | Brankas Sumber |
|---|---|---|
| Valid | Valid | Dipilih |
| Valid | Valid | Tidak Dipilih |
| Valid | Invalid | Dipilih |
| Valid | Invalid | Tidak Dipilih |
| Invalid | Valid | Dipilih |
| Invalid | Valid | Tidak Dipilih |
| Invalid | Invalid | Dipilih |
| Invalid | Invalid | Tidak Dipilih |

### 3. Menjalankan Test Case

Untuk setiap kombinasi parameter dan kondisi dalam tabel matriks, jalankan test case yang sesuai untuk menguji fungsionalitas pembuatan target tabungan. Catat hasil test case (lulus/gagal) di kolom "Status" pada tabel matriks.

| TC | Nama | Target | Brankas | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | Valid — `TabunganMT` | Valid — `500.000` | Dipilih (BSI) | Tabungan berhasil dibuat | Muncul di daftar Target Impian | ✅ Passed |
| 2 | Valid — `TabunganMT` | Valid — `500.000` | Tidak Dipilih | Ditolak — brankas wajib dipilih | "Harap isi bidang ini." | ✅ Passed |
| 3 | Valid — `TabunganMT` | Invalid — `0` | Dipilih (BSI) | Ditolak — target minimal 1 | HTTP 422 "target amount minimal 1" | ✅ Passed |
| 4 | Valid — `TabunganMT` | Invalid — `0` | Tidak Dipilih | Ditolak — brankas & target tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 5 | Invalid — `""` kosong | Valid — `500.000` | Dipilih (BSI) | Ditolak — nama wajib diisi | "Harap isi bidang ini." | ✅ Passed |
| 6 | Invalid — `""` kosong | Valid — `500.000` | Tidak Dipilih | Ditolak — nama & brankas kosong | "Harap isi bidang ini." | ✅ Passed |
| 7 | Invalid — 300 karakter | Invalid — `0` | Dipilih (BSI) | Ditolak — nama terlalu panjang | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |
| 8 | Invalid — 300 karakter | Invalid — `0` | Tidak Dipilih | Ditolak — nama terlalu panjang | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |

### Screenshot Bukti — TC1 (Tabungan Berhasil Dibuat)

<img width="1920" height="1080" alt="matrix-tabungan-tc1" src="https://github.com/user-attachments/assets/67de3643-13f1-4fa4-bfda-f12c35646548" />


### 4. Analisis Hasil Test

- **Tidak ada kombinasi parameter yang menyebabkan test case gagal.**
- Validasi nama (required, max:255) dan target amount (min:1) bekerja dengan benar di semua kombinasi.
- Backend mengembalikan error nama terlebih dahulu jika nama > 255 karakter, meskipun target juga invalid (TC7 & TC8) — prioritas validasi backend sudah benar.
- **Perbarui tabel matriks:** seluruh 8 kombinasi Passed — tidak ada bug baru ditemukan.

---

## Ringkasan Hasil Matrix Testing — Seluruh Sistem

| Modul | Parameter | Kombinasi (TC) | Passed | Failed | Temuan |
|---|---|---|---|---|---|
| Autentikasi | Email × Password × Format | 8 | 8 | 0 | Semua kombinasi ditangani benar |
| Transfer | Amount × Saldo × Tujuan | 8 | 8 | 0 | Validasi POST konsisten |
| Transaksi | Amount × Type × Portofolio | 8 | 7 | **1** | 🔴 TC3: Expense valid → saldo negatif |
| Tabungan | Nama × Target × Brankas | 8 | 8 | 0 | Validasi nama & target konsisten |
| **TOTAL** | | **32** | **31** | **1** | |

> **Kesimpulan Matrix Testing:** Dari 32 kombinasi matriks yang diuji, ditemukan **1 kasus gagal** pada modul Transaksi — kombinasi `Amount Valid × Type Expense × Portofolio Dipilih` (TC3) menghasilkan saldo negatif karena tidak ada validasi saldo di backend. Matrix Testing berhasil mengisolasi bahwa bug hanya terjadi pada interaksi spesifik antara parameter **type=expense** dan **portofolio yang dipilih**, bukan pada semua kombinasi expense. Ketiga modul lainnya menunjukkan semua kombinasi parameter ditangani dengan benar.
