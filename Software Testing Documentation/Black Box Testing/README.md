# Dokumentasi Black Box Testing — SaPoPoe FINANCE (Midnight Finance)
### Ujian Akhir Semester — Software Quality Assurance
**Program Studi Teknik Informatika — Universitas Kebangsaan RI (UKRI)**

---

## Tentang Sistem

**SaPoPoe FINANCE** (Midnight Finance) adalah aplikasi manajemen keuangan pribadi berbasis web dengan arsitektur:

| Komponen | Teknologi |
|---|---|
| Backend | Laravel 11 + Laravel Sanctum (REST API) |
| Frontend | React.js + Vite |
| Database | MySQL |
| Auth | Token-based (Sanctum Bearer Token) |

**Modul yang Diuji:**

| Modul | Endpoint Utama | Fungsi |
|---|---|---|
| Autentikasi | `POST /api/login` | Login pengguna |
| Transfer | `POST /api/transfers` | Pindah dana antar brankas |
| Transaksi | `POST /api/transactions` | Catat pemasukan / pengeluaran |
| Tabungan | `POST /api/savings` | Buat & kelola target tabungan |

---

---

## Daftar Pengujian Black Box (BB)

| Kode | Teknik | Modul Diuji | TC | Passed | Failed | Diagram |
|---|---|---|---|---|---|---|
| [BB-01](BB-01-boundary-value-analysis.md) | Boundary Value Analysis | Auth · Transfer · Transaksi · Tabungan | 24 | 23 | 1 | Class Diagram |
| [BB-02](BB-02-equivalence-partitioning.md) | Equivalence Partitioning | Auth · Transfer · Transaksi · Tabungan | 12 | 12 | 0 | — |
| [BB-03](BB-03-decision-table-testing.md) | Decision Table Testing | Auth · Transfer · Transaksi · Tabungan | 13 | 13 | 0 | Decision Table |
| [BB-04](BB-04-comparison-testing.md) | Comparison Testing | Auth · Transfer · Transaksi · Tabungan | — | — | — | Flowchart |
| [BB-05](BB-05-sample-testing.md) | Sample Testing | Auth · Transfer · Transaksi · Tabungan | 12 | 11 | 1 | Class Diagram |
| [BB-06](BB-06-behaviour-testing.md) | Behaviour Testing (BDD) | Auth · Transfer · Transaksi · Tabungan | 12 | 11 | 1 | Sequence + State Diagram |
| [BB-07](BB-07-performance-testing.md) | Performance Testing | Dashboard (`/dashboard`) | 3 | 2 | 0⚠️ | Flowchart |
| [BB-08](BB-08-endurance-testing.md) | Endurance Testing | Auth · Transfer · Transaksi · Tabungan | 20 iter | 15 | 5 | Flowchart |
| [BB-09](BB-09-cause-effect-testing.md) | Cause-Effect Relationship | Auth · Transfer · Transaksi · Tabungan | 20 | 13 | 2 | Fishbone (4 diagram) |

---

## Rekapitulasi Defect yang Ditemukan

| No | Defect | Ditemukan di | Modul | Severity | Status |
|---|---|---|---|---|---|
| 1 | Expense tanpa validasi saldo → saldo negatif | BB-01, BB-05, BB-06, BB-09 | Transaksi | 🔴 **Kritis** | Belum diperbaiki |
| 2 | `GET /api/transfers` selalu return HTTP 500 | BB-08, BB-09 | Transfer | 🔴 **Kritis** | Belum diperbaiki |

### Detail Defect

**Defect #1 — Saldo Negatif pada Modul Transaksi**
- **Cause:** `TransactionController::store()` tidak memvalidasi kecukupan saldo sebelum mencatat expense
- **Effect:** Saldo brankas bisa menjadi negatif (contoh: Rp -994.649.999)
- **Rekomendasi:**
```php
// Tambahkan di TransactionController::store() sebelum simpan transaksi
if ($request->type === 'expense' && $wallet->balance < $request->amount) {
    return response()->json(['error' => 'Saldo tidak mencukupi'], 400);
}
```

**Defect #2 — GET Riwayat Transfer HTTP 500**
- **Cause:** Bug di `TransferController@index()` — kemungkinan relasi Eloquent gagal di-load
- **Effect:** Halaman riwayat transfer tidak bisa ditampilkan, endpoint selalu error
- **Rekomendasi:** Debug query di `TransferController@index()`, periksa relasi `with()` yang digunakan

---

## Ringkasan Hasil Seluruh Pengujian

```
Black Box Testing
─────────────────────────────────────────────────────
Teknik digunakan       : 9 teknik (BB-01 s/d BB-09)
Total Test Case        : 116
Passed                 : 100
Failed / Bug           :   2 defect kritis
Warning / Kelemahan    :   5 area perlu peningkatan

Bukti Pengujian
─────────────────────────────────────────────────────
Total Screenshot       : 32 gambar
Tersimpan di           : screenshots/
```

---

## Screenshot Tersedia

| Prefix | Modul | Jumlah |
|---|---|---|
| `bva-*` | BVA (BB-01) | 9 |
| `ep-*` | Equivalence Partitioning (BB-02) | 5 |
| `sample-*` | Sample Testing (BB-05) | 12 |
| `perf-*` | Performance Testing (BB-07) | 2 |
| `endurance-*` | Endurance Testing (BB-08) | 4 |

---

> Dokumentasi Black Box Testing ini dibuat sebagai bagian dari **Ujian Akhir Semester (UAS) mata kuliah Software Quality**, Program Studi Teknik Informatika, Universitas Kebangsaan RI (UKRI), 2025/2026.
