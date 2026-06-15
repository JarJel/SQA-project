# Dokumentasi SQA — SaPoPoe FINANCE (Midnight Finance)
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

## Struktur Dokumentasi

```
midnight-sqa/
├── README.md                        ← file ini
│
├── BB-01-boundary-value-analysis.md
├── BB-02-equivalence-partitioning.md
├── BB-03-decision-table-testing.md
├── BB-04-comparison-testing.md
├── BB-05-sample-testing.md
├── BB-06-behaviour-testing.md
├── BB-07-performance-testing.md
├── BB-08-endurance-testing.md
├── BB-09-cause-effect-testing.md
│
├── WB-01-desk-checking.md
├── WB-02-code-walkthrough.md
├── WB-03-formal-inspections.md
├── WB-04-control-flow.md
├── WB-05-basic-path.md
├── WB-06-data-flow.md
├── WB-07-loop-testing.md
│
└── screenshots/                     ← 32 screenshot bukti pengujian
```

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

## Daftar Pengujian White Box (WB)

| Kode | Teknik | Fokus Analisis | Diagram |
|---|---|---|---|
| [WB-01](WB-01-desk-checking.md) | Desk Checking | Review kode manual tanpa eksekusi | — |
| [WB-02](WB-02-code-walkthrough.md) | Code Walkthrough | Penjelasan alur kode per fungsi | — |
| [WB-03](WB-03-formal-inspections.md) | Formal Inspections | Inspeksi formal dengan checklist | — |
| [WB-04](WB-04-control-flow.md) | Control Flow Testing | Alur percabangan & kondisi | Flowchart |
| [WB-05](WB-05-basic-path.md) | Basic Path Testing | Jalur independen (cyclomatic complexity) | Graph |
| [WB-06](WB-06-data-flow.md) | Data Flow Testing | Definisi & penggunaan variabel | — |
| [WB-07](WB-07-loop-testing.md) | Loop Testing | Pengujian struktur perulangan | — |

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
Total Test Case        : 116
Passed                 : 100
Failed / Bug           :   2 defect kritis
Warning / Kelemahan    :   5 area perlu peningkatan

White Box Testing
─────────────────────────────────────────────────────
Teknik digunakan       : 7 teknik
Modul dianalisis       : 4 modul (Auth, Transfer, Transaksi, Tabungan)
Temuan                 : Konsisten dengan temuan Black Box

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

> Dokumentasi ini dibuat sebagai bagian dari **Ujian Akhir Semester (UAS) mata kuliah Software Quality**, Program Studi Teknik Informatika, Universitas Kebangsaan RI (UKRI), 2025/2026.
