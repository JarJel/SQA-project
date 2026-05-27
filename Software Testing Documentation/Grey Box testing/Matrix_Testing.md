# 📊 Matrix Testing — Midnight Finance

**Mata Kuliah:** Software Quality Assurance  
**Model Pengujian:** Gray Box Testing — Matrix Testing  
**Tim:** REMACode  
**Modul Target:** Fitur Pencarian & Filter Riwayat Transaksi  

---

## 📖 Definisi

**Matrix Testing** adalah teknik pengujian perangkat lunak yang **sistematis dan terstruktur** untuk menguji berbagai **kombinasi input dan kondisi** dalam suatu aplikasi. Teknik ini membantu **mengidentifikasi bug yang disebabkan oleh interaksi antara parameter atau faktor yang berbeda** dalam aplikasi (Suprihadi, 2025).

**Langkah Skenario Matrix Testing:**
1. Definisikan Parameter dan Kondisi
2. Membuat Tabel Matriks
3. Menjalankan Test Case
4. Analisis Hasil

---

## 🎯 Modul yang Diuji

**Endpoint:** `GET /api/transactions`  
**Skenario:** Aplikasi Midnight Finance memiliki fitur filter transaksi yang memungkinkan pengguna mencari berdasarkan **tipe transaksi**, **rentang tanggal**, dan **kategori**.

---

## 📊 Tahap 1: Definisikan Parameter dan Kondisi

| Parameter | Kondisi A (Valid/Ada) | Kondisi B (Kosong/Default) |
|:---|:---:|:---:|
| `type` (Tipe transaksi) | `income` atau `expense` | *(tidak diisi — ambil semua)* |
| `start_date` (Tanggal mulai) | `2025-01-01` | *(tidak diisi)* |
| `category_id` (Kategori) | ID kategori valid milik user | *(tidak diisi)* |

---

## 📋 Tahap 2: Tabel Matriks Kombinasi

| # | `type` | `start_date` | `category_id` | Kombinasi |
|:--:|:---:|:---:|:---:|:---|
| M1 | A (income) | A (ada) | A (ada) | Semua filter aktif |
| M2 | A (income) | A (ada) | B (kosong) | type + tanggal |
| M3 | A (income) | B (kosong) | A (ada) | type + kategori |
| M4 | A (income) | B (kosong) | B (kosong) | Hanya type |
| M5 | B (expense) | A (ada) | A (ada) | Semua filter (expense) |
| M6 | B (expense) | A (ada) | B (kosong) | type expense + tanggal |
| M7 | B (expense) | B (kosong) | A (ada) | type expense + kategori |
| M8 | B (kosong) | B (kosong) | B (kosong) | Tanpa filter (semua data) |

---

## 📝 Tahap 3: Test Case Matrix Testing

| No | Matriks | Input Parameter | Expected Output | Actual Output | Status |
|:--:|:---:|:---|:---|:---:|:---:|
| TC-MT-01 | M1 | `type=income&start_date=2025-01-01&category_id=1` | HTTP 200 — income, mulai Jan 2025, kategori id=1 | HTTP 200 | ✅ Valid |
| TC-MT-02 | M2 | `type=income&start_date=2025-01-01` | HTTP 200 — semua income mulai Jan 2025 | HTTP 200 | ✅ Valid |
| TC-MT-03 | M3 | `type=income&category_id=1` | HTTP 200 — semua income di kategori id=1 | HTTP 200 | ✅ Valid |
| TC-MT-04 | M4 | `type=income` | HTTP 200 — semua transaksi income milik user | HTTP 200 | ✅ Valid |
| TC-MT-05 | M5 | `type=expense&start_date=2025-01-01&category_id=2` | HTTP 200 — expense, mulai Jan 2025, kategori id=2 | HTTP 200 | ✅ Valid |
| TC-MT-06 | M6 | `type=expense&start_date=2025-06-01` | HTTP 200 — expense mulai Juni 2025 | HTTP 200 | ✅ Valid |
| TC-MT-07 | M7 | `type=expense&category_id=2` | HTTP 200 — semua expense di kategori id=2 | HTTP 200 | ✅ Valid |
| TC-MT-08 | M8 | *(tanpa parameter)* | HTTP 200 — seluruh transaksi milik user | HTTP 200 | ✅ Valid |

---

## 📊 Tahap 4: Analisis Hasil

| Analisis | Temuan |
|:---|:---|
| Bug interaksi parameter | Tidak ditemukan — setiap kombinasi menghasilkan output yang konsisten |
| Konsistensi respons JSON | Struktur data `{ "data": [...] }` konsisten di semua 8 skenario |
| Filter kosong | Sistem mengembalikan semua data tanpa filter — sesuai ekspektasi |
| Filter kategori bukan milik user | Mengembalikan array kosong `[]` — aman, tidak bocor ke data user lain |

---

## ✅ Hasil Pengujian

| Kategori | Jumlah |
|:---|:---:|
| Total Parameter | 3 |
| Total Kombinasi Matriks | 8 |
| Test Case Passed | 8 |
| Bug Ditemukan | 0 |
| **Coverage Kombinasi** | **100%** |

> **Kesimpulan:** Matrix Testing berhasil memverifikasi semua 8 kombinasi parameter filter pada endpoint `GET /api/transactions`. Tidak ditemukan bug interaksi antar parameter. Sistem Midnight Finance menangani seluruh kombinasi filter secara konsisten dan aman.

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Gray Box Testing*. T Informatika UKRI.
