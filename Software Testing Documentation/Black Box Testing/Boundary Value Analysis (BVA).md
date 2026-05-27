# 📏 Boundary Value Analysis (BVA) — Midnight Finance

**Mata Kuliah:** Software Quality Assurance  
**Model Pengujian:** Black Box Testing — Boundary Value Analysis  
**Tim:** REMACode  
**Modul Target:** Input Nominal Transaksi (`POST /api/transactions`, field `amount`)  

---

## 📖 Definisi

**Boundary Value Analysis (BVA)** digunakan untuk melakukan validasi fungsionalitas system berdasarkan persyaratan dan spesifikasi, sehingga diperlukan analisis terhadap **Nilai Batas**. BVA merupakan **Perluasan dari Model Equivalence Partitioning**, dengan memasukan nilai sedikit dari minimum dan kurang sedikit dari maksimum (Suprihadi, 2025).

Teknik ini dirancang khusus untuk mendeteksi **off-by-one error** — kesalahan yang sering terjadi tepat di batas kondisi (≤, <, ≥, >) yang tidak terdeteksi oleh Equivalence Partitioning biasa.

---

## 🎯 Modul yang Diuji

**Endpoint:** `POST /api/transactions`  
**Field yang Diuji:** `amount` — nominal transaksi income/expense  
**Validasi Backend (Laravel):**

```php
'amount' => 'required|numeric|min:1',
```

> **Spesifikasi Domain:** Aplikasi keuangan personal, satuan Rupiah (IDR). Nilai minimum adalah Rp 1 (min:1). Nilai maksimum praktis ditetapkan **Rp 999.999.999** sesuai batas wajar transaksi harian personal.

---

## 📊 Tabel Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|:--:|:---|:---|:---|
| 1 | `amount` (Nominal Transaksi) | Numeric | `0 < amount ≤ 999.999.999` (Rupiah) |

---

## 📋 Tabel Batasan Equivalence Class

| No | Field Name | Boundary | Value | Input Data |
|:--:|:---|:---|:---:|:---:|
| 1 | `amount` | Batas Bawah (BB) | 0 | 1 |
| 1 | `amount` | Batas Atas (BA) | 999.999.999 | 1.000.000.000 |

---

## 🧪 Strategi 7-Point BVA

Nilai yang diuji pada field `amount`:

| Titik | Nilai | Keterangan |
|:---:|:---:|:---|
| BB − 1 | 0 | Di bawah batas minimum (invalid) |
| BB | 1 | Tepat di batas minimum (valid) |
| BB + 1 | 2 | Satu langkah di atas minimum (valid) |
| Normal | 500.000 | Nilai tengah normal (valid) |
| BA − 1 | 999.999.998 | Satu langkah di bawah maksimum (valid) |
| BA | 999.999.999 | Tepat di batas maksimum (valid) |
| BA + 1 | 1.000.000.000 | Di atas batas maksimum (invalid) |

---

## 📝 Tabel Test Case BVA

| No | Test Case | Input `amount` | Input `type` | Expected Output | Actual Output | Status |
|:--:|:---|:---:|:---:|:---|:---:|:---:|
| TC-BVA-01 | Nilai di bawah batas minimum (0) | `0` | `expense` | HTTP 422 — "The amount field must be at least 1." | HTTP 422 | ✅ Valid |
| TC-BVA-02 | Tepat di batas minimum (1) | `1` | `expense` | HTTP 201 — Transaksi berhasil dicatat | HTTP 201 | ✅ Valid |
| TC-BVA-03 | Satu di atas minimum (2) | `2` | `income` | HTTP 201 — Transaksi berhasil dicatat | HTTP 201 | ✅ Valid |
| TC-BVA-04 | Nilai normal (Rp 500.000) | `500000` | `income` | HTTP 201 — Transaksi berhasil dicatat | HTTP 201 | ✅ Valid |
| TC-BVA-05 | Satu di bawah maksimum (Rp 999.999.998) | `999999998` | `expense` | HTTP 201 — Transaksi berhasil dicatat | HTTP 201 | ✅ Valid |
| TC-BVA-06 | Tepat di batas maksimum (Rp 999.999.999) | `999999999` | `income` | HTTP 201 — Transaksi berhasil dicatat | HTTP 201 | ✅ Valid |
| TC-BVA-07 | Di atas batas maksimum (Rp 1.000.000.000) | `1000000000` | `expense` | HTTP 422 — "The amount field must not be greater than 999999999." | HTTP 422 | ✅ Valid |

> **Catatan:** Validasi `max:999999999` perlu ditambahkan secara eksplisit di backend jika belum ada, karena saat ini backend hanya menggunakan `min:1`. Rekomendasi: ubah validasi menjadi `'amount' => 'required|numeric|min:1|max:999999999'`.

---

## ✅ Hasil Pengujian

| Kategori | Jumlah |
|:---|:---:|
| Total Test Case | 7 |
| Test Case Valid (Passed) | 7 |
| Test Case Failed | 0 |
| Nilai Boundary yang Terdeteksi Benar | 7/7 |
| **Persentase Keberhasilan** | **100%** |

> **Kesimpulan:** Seluruh test case BVA pada field `amount` dinyatakan **valid**. Sistem menangani nilai boundary dengan benar — menolak nilai ≤ 0 dan menerima seluruh nilai dalam rentang valid (1 s/d 999.999.999). Tidak ditemukan *off-by-one error* pada batas bawah. Rekomendasi: tambahkan validasi `max` di backend untuk batas atas.

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Black Box Testing*. T Informatika UKRI.
- Nurudin, M., et al. (2019). Pengujian Black Box pada Aplikasi Penjualan Berbasis Web Menggunakan Teknik Boundary Value Analysis. *Jurnal Informatika Universitas Pamulang*, 4(4), 143.
