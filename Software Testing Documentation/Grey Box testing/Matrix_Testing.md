# 📐 Orthogonal Array Testing — Midnight Finance

**Mata Kuliah:** Software Quality Assurance  
**Model Pengujian:** Gray Box Testing — Orthogonal Array Testing (OAT)  
**Tim:** REMACode  
**Modul Target:** Filter & Sort Riwayat Transaksi (`GET /api/transactions`)  

---

## 📖 Definisi

**Orthogonal Array Testing (OAT)** adalah teknik pengujian perangkat lunak yang menggunakan **array ortogonal untuk membuat kasus uji**. Ini adalah pendekatan pengujian statistik yang sangat berguna ketika sistem yang akan diuji memiliki **input data yang besar**. Pengujian susunan ortogonal membantu memaksimalkan cakupan pengujian dengan **memasangkan dan menggabungkan input** serta menguji sistem dengan jumlah kasus pengujian yang relatif lebih sedikit untuk menghemat waktu (Suprihadi, 2025).

**Langkah-Langkah dalam pengujian OAT:**
1. Identifikasi variabel independen untuk skenario tersebut
2. Temukan array terkecil dengan jumlah proses
3. Petakan faktor-faktor tersebut ke dalam array
4. Pilih nilai untuk level "sisa" mana pun
5. Transkripsikan proses ke dalam kasus uji, tambahkan kombinasi mencurigakan yang tidak dihasilkan

---

## 🎯 Modul yang Diuji

**Endpoint:** `GET /api/transactions`  
**Deskripsi:** Endpoint ini mendukung kombinasi filter dan sorting simultan.

```php
// Parameter yang tersedia di TransactionController@index
$request->filled('start_date')           // Filter tanggal mulai
$request->filled('end_date')             // Filter tanggal akhir
$request->filled('type')                 // Filter tipe: income / expense
$request->input('sort_by', 'date')       // Kolom sort: date / amount
$request->input('sort_order', 'desc')    // Arah sort: asc / desc
```

---

## 📊 Identifikasi Faktor & Level

| No | Faktor | Level 1 | Level 2 | Level 3 |
|:--:|:---|:---:|:---:|:---:|
| 1 | `type` (Filter tipe) | `income` | `expense` | *(kosong/semua)* |
| 2 | `sort_by` (Kolom sort) | `date` | `amount` | — |
| 3 | `sort_order` (Arah sort) | `asc` | `desc` | — |

- **Jumlah Faktor** = 3 (type, sort_by, sort_order)
- **Jumlah Level** = 3 level untuk factor 1, 2 level untuk factor 2 & 3
- **Tipe Array** = L6 (6 kasus uji)

---

## 📋 Tabel Orthogonal Array L6

| Kasus Uji # | `type` | `sort_by` | `sort_order` |
|:---:|:---:|:---:|:---:|
| 1 | `income` | `date` | `asc` |
| 2 | `income` | `amount` | `desc` |
| 3 | `expense` | `date` | `desc` |
| 4 | `expense` | `amount` | `asc` |
| 5 | *(semua)* | `date` | `asc` |
| 6 | *(semua)* | `amount` | `desc` |

---

## 📝 Test Case OAT

| No | Kasus Uji | Parameter Request | Expected Output | Actual Output | Status |
|:--:|:---|:---|:---|:---:|:---:|
| TC-OAT-01 | Income, sort date ASC | `type=income&sort_by=date&sort_order=asc` | HTTP 200 — hanya income, urut tanggal terlama → terbaru | HTTP 200 | ✅ Valid |
| TC-OAT-02 | Income, sort amount DESC | `type=income&sort_by=amount&sort_order=desc` | HTTP 200 — hanya income, urut jumlah terbesar → terkecil | HTTP 200 | ✅ Valid |
| TC-OAT-03 | Expense, sort date DESC | `type=expense&sort_by=date&sort_order=desc` | HTTP 200 — hanya expense, urut tanggal terbaru → terlama | HTTP 200 | ✅ Valid |
| TC-OAT-04 | Expense, sort amount ASC | `type=expense&sort_by=amount&sort_order=asc` | HTTP 200 — hanya expense, urut jumlah terkecil → terbesar | HTTP 200 | ✅ Valid |
| TC-OAT-05 | Semua tipe, sort date ASC | `sort_by=date&sort_order=asc` | HTTP 200 — semua transaksi, urut tanggal terlama → terbaru | HTTP 200 | ✅ Valid |
| TC-OAT-06 | Semua tipe, sort amount DESC | `sort_by=amount&sort_order=desc` | HTTP 200 — semua transaksi, urut jumlah terbesar → terkecil | HTTP 200 | ✅ Valid |

---

## ✅ Hasil Pengujian

| Kategori | Jumlah |
|:---|:---:|
| Total Faktor | 3 |
| Total Level | Maks. 3 |
| Total Test Case (OAT) | 6 |
| Test Case Passed | 6 |
| **Coverage Pasangan (Pairwise)** | **100%** |

> **Kesimpulan:** OAT berhasil mereduksi jumlah kasus uji dari 12 kemungkinan kombinasi menjadi hanya 6 kasus uji tanpa kehilangan coverage pasangan. Semua kombinasi faktor filter dan sorting pada endpoint `GET /api/transactions` Midnight Finance berjalan dengan benar.

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Gray Box Testing*. T Informatika UKRI.
