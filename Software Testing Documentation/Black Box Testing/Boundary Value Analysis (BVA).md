# 📏 Boundary Value Analysis (BVA) — Midnight Finance

**Proyek:** Midnight Finance
**Modul yang Diuji:** Transaction — `POST /api/transactions`
**Tanggal Pengujian:** 27 Mei 2026
**Penguji:** QA Team — REMACode
**Metode:** Boundary Value Analysis (BVA) — 7-Point Strategy

---

## 1. Definisi

**Boundary Value Analysis (BVA)** digunakan untuk melakukan validasi fungsionalitas system berdasarkan persyaratan dan spesifikasi, sehingga diperlukan analisis terhadap **Nilai Batas**. BVA merupakan **Perluasan dari Model Equivalence Partitioning**, dengan memasukan nilai sedikit dari minimum dan kurang sedikit dari maksimum (Suprihadi, 2025).

**Field yang diuji:** `amount` (jumlah transaksi)
**Range valid:** `1 ≤ amount ≤ 999.999.999`
**Endpoint:** `POST /api/transactions`
**Auth:** Bearer Token

---

## 2. Tabel Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|:--:|:---|:---|:---|
| 1 | `amount` (Nominal Transaksi) | Numeric | `0 < amount ≤ 999.999.999` (Rupiah) |

---

## 3. Tabel Batasan Equivalence Class

| No | Field Name | Boundary | Value | Input Data |
|:--:|:---|:---|:---:|:---:|
| 1 | `amount` | Batas Bawah (BB) | 0 | 1 |
| 1 | `amount` | Batas Atas (BA) | 999.999.999 | 1.000.000.000 |

---

## 4. Visualisasi 7-Point BVA

```
         INVALID │◄──────── VALID RANGE ────────►│ INVALID
                 │                               │
  ───────────────┼───────────────────────────────┼───────────────
  ... 0(✗) │ 1(✓) │ 2(✓) │ ... 500.000(✓) ... │ 999.999.998(✓) │ 999.999.999(✓) │ 1.000.000.000(✗) ...
           BB-1  BB    BB+1        NORMAL           BA-1              BA                BA+1
           ❌    ✅    ✅           ✅               ✅                ✅                ❌(BUG!)
```

```mermaid
graph LR
    A["0\nBB−1\n❌ 422"] --> B["1\nBB\n✅ 201"]
    B --> C["2\nBB+1\n✅ 201"]
    C --> D["500.000\nNormal\n✅ 201"]
    D --> E["999.999.998\nBA−1\n✅ 201"]
    E --> F["999.999.999\nBA\n✅ 201"]
    F --> G["1.000.000.000\nBA+1\n🐛 201"]

    style A fill:#ffcccc,stroke:#cc0000,color:#7a0000
    style B fill:#ccffcc,stroke:#007700,color:#004400
    style C fill:#ccffcc,stroke:#007700,color:#004400
    style D fill:#ccffcc,stroke:#007700,color:#004400
    style E fill:#ccffcc,stroke:#007700,color:#004400
    style F fill:#ccffcc,stroke:#007700,color:#004400
    style G fill:#ff9900,stroke:#cc6600,color:#5c2d00
```

> 🐛 **BVA-07 ditemukan BUG** — sistem seharusnya menolak amount 1 miliar tapi malah menerima (HTTP 201)

---

## 5. Flowchart Alur Validasi Amount

```mermaid
flowchart TD
    A([▶ START]) --> B[/Input: amount/]
    B --> C{amount\ndikirim?}
    C -- Tidak --> Z1[/⚠ HTTP 422\nThe amount field is required/]
    C -- Ya --> D{amount\nnumeric?}
    D -- Tidak --> Z2[/⚠ HTTP 422\nMust be numeric/]
    D -- Ya --> E{amount\n≥ 1?}
    E -- Tidak\namount = 0 atau negatif --> Z3[/⚠ HTTP 422\nMust be at least 1/]
    E -- Ya --> F{"amount\n≤ 999.999.999?\n⚠ VALIDASI INI\nBELUM ADA!"}
    F -- Tidak\n🐛 BUG: lolos --> G[/🐛 HTTP 201\nDiterima padahal\nseharusnya ditolak/]
    F -- Ya --> H[Proses transaksi\nUpdate saldo]
    H --> I[/✅ HTTP 201\nTransaksi berhasil/]
    Z1 --> J([⏹ END])
    Z2 --> J
    Z3 --> J
    G --> J
    I --> J

    style G fill:#ff9900,stroke:#cc6600,color:#5c2d00
    style F fill:#ffffcc,stroke:#cccc00,color:#555500
```

---

## 6. Test Cases dan Hasil Pengujian

### TC-BVA-01 — Amount = 0 (Di bawah batas minimum)

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA01 |
| **Input** | `amount: 0`, `type: expense` |
| **Expected Output** | HTTP 422 — "The amount field must be at least 1." |
| **Actual Output** | HTTP 422 — `{"message":"The amount field must be at least 1.","errors":{"amount":["The amount field must be at least 1."]}}` |
| **Status** | ✅ PASSED |

### TC-BVA-02 — Amount = 1 (Batas minimum tepat)

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA02 |
| **Input** | `amount: 1`, `type: expense` |
| **Expected Output** | HTTP 201 — Transaksi berhasil |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 56`, balance diperbarui |
| **Status** | ✅ PASSED |

### TC-BVA-03 — Amount = 2 (Batas minimum + 1)

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA03 |
| **Input** | `amount: 2`, `type: income` |
| **Expected Output** | HTTP 201 — Transaksi berhasil |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 57`, balance diperbarui |
| **Status** | ✅ PASSED |

### TC-BVA-04 — Amount = 500.000 (Nilai normal)

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA04 |
| **Input** | `amount: 500000`, `type: income` |
| **Expected Output** | HTTP 201 — Transaksi berhasil |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 58`, balance diperbarui |
| **Status** | ✅ PASSED |

### TC-BVA-05 — Amount = 999.999.998 (Batas maksimum − 1)

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA05 |
| **Input** | `amount: 999999998`, `type: income` |
| **Expected Output** | HTTP 201 — Transaksi berhasil |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 59`, balance diperbarui |
| **Status** | ✅ PASSED |

### TC-BVA-06 — Amount = 999.999.999 (Batas maksimum tepat)

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA06 |
| **Input** | `amount: 999999999`, `type: income` |
| **Expected Output** | HTTP 201 — Transaksi berhasil |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 60`, balance diperbarui |
| **Status** | ✅ PASSED |

### TC-BVA-07 — Amount = 1.000.000.000 (Di atas batas maksimum) 🐛

| Atribut | Detail |
|:---|:---|
| **ID Test** | BB02-BVA07 |
| **Input** | `amount: 1000000000`, `type: expense` |
| **Expected Output** | HTTP 422 — Validasi gagal (amount melebihi batas maksimum) |
| **Actual Output** | HTTP 201 — **Transaksi BERHASIL dibuat** (`id: 61`, amount 1 miliar tersimpan) |
| **Status** | ❌ **FAILED — BUG DITEMUKAN** |

> **⚠️ BUG #001 — Missing Max Validation on Amount Field**
>
> **Root Cause:** `TransactionController.php` hanya punya rule `'amount' => 'required|numeric|min:1'` tanpa `max`.
>
> **Rekomendasi Fix:** Ubah menjadi `'amount' => 'required|numeric|min:1|max:999999999'`

---

## 7. Ringkasan Hasil Pengujian

| ID Test | Nilai Amount | Expected HTTP | Actual HTTP | Status |
|:---:|:---:|:---:|:---:|:---:|
| BVA-01 | 0 | 422 | 422 | ✅ PASSED |
| BVA-02 | 1 | 201 | 201 | ✅ PASSED |
| BVA-03 | 2 | 201 | 201 | ✅ PASSED |
| BVA-04 | 500.000 | 201 | 201 | ✅ PASSED |
| BVA-05 | 999.999.998 | 201 | 201 | ✅ PASSED |
| BVA-06 | 999.999.999 | 201 | 201 | ✅ PASSED |
| BVA-07 | 1.000.000.000 | 422 | **201** | ❌ **FAILED** |

**Total: 7 test case | ✅ 6 Passed | ❌ 1 Failed | 🐛 1 Bug Ditemukan**

---

## 8. Bukti Pengujian (Test Evidence)

File JSON hasil pengujian tersimpan di `screenshots/black-box/`:

| File | HTTP | Keterangan |
|:---|:---:|:---|
| `BB02-BVA01-amount0.json` | 422 | Amount 0 ditolak ✅ |
| `BB02-BVA02-amount1.json` | 201 | Amount 1 diterima ✅ |
| `BB02-BVA03-amount2.json` | 201 | Amount 2 diterima ✅ |
| `BB02-BVA04-amount500000.json` | 201 | Amount 500.000 diterima ✅ |
| `BB02-BVA05-amount999999998.json` | 201 | Amount 999.999.998 diterima ✅ |
| `BB02-BVA06-amount999999999.json` | 201 | Amount 999.999.999 diterima ✅ |
| `BB02-BVA07-amount1000000000.json` | 201 | **BUG: 1 miliar diterima** ❌ |

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Black Box Testing*. T Informatika UKRI.
- Nurudin, M., et al. (2019). Pengujian Black Box pada Aplikasi Penjualan Berbasis Web Menggunakan Teknik Boundary Value Analysis. *Jurnal Informatika Universitas Pamulang*, 4(4), 143.
