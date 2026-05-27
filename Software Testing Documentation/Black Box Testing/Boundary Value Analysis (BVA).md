# Black Box Testing — Boundary Value Analysis (BVA)

**Proyek:** Midnight Finance  
**Modul yang Diuji:** Transaction — `POST /api/v1/transactions`  
**Tanggal Pengujian:** 27 Mei 2026  
**Penguji:** QA Team  
**Metode:** Boundary Value Analysis (BVA) — 7-Point Strategy  

---

## 1. Deskripsi Pengujian

Boundary Value Analysis (BVA) adalah teknik *black box testing* yang berfokus pada nilai-nilai di batas partisi input. Nilai batas dan nilai tepat di luar batas cenderung menyebabkan lebih banyak kesalahan daripada nilai yang berada di tengah partisi.

**Field yang diuji:** `amount` (jumlah transaksi)  
**Asumsi batas yang valid:** `1 ≤ amount ≤ 999.999.999`  
**Endpoint:** `POST /api/v1/transactions`  
**Auth:** Bearer Token (user: admin@midnight.com)

---

## 2. Strategi 7-Titik Batas

Dengan asumsi range valid `[1, 999.999.999]`, tujuh titik pengujian yang dipilih adalah:

| Kode | Titik Batas | Nilai | Keterangan |
|------|-------------|-------|------------|
| BVA-01 | BB (Batas Bawah − 1) | 0 | Di bawah batas minimum |
| BVA-02 | BB (Batas Bawah tepat) | 1 | Tepat pada batas minimum |
| BVA-03 | BB+1 (Batas Bawah + 1) | 2 | Tepat di atas batas minimum |
| BVA-04 | Normal | 500.000 | Nilai tengah, jauh dari batas |
| BVA-05 | BA−1 (Batas Atas − 1) | 999.999.998 | Tepat di bawah batas maksimum |
| BVA-06 | BA (Batas Atas tepat) | 999.999.999 | Tepat pada batas maksimum |
| BVA-07 | BA+1 (Batas Atas + 1) | 1.000.000.000 | Di atas batas maksimum |

---

## 3. Test Cases dan Hasil Pengujian

### TC-BVA-01 — Amount = 0 (Di bawah batas minimum)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA01 |
| **Input** | `amount: 0` |
| **Expected Output** | HTTP 422 — Validasi gagal (`amount must be at least 1`) |
| **Actual Output** | HTTP 422 — `{"message":"The amount field must be at least 1.","errors":{"amount":["The amount field must be at least 1."]}}` |
| **Status** | ✅ PASSED |

---

### TC-BVA-02 — Amount = 1 (Batas minimum tepat)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA02 |
| **Input** | `amount: 1` |
| **Expected Output** | HTTP 201 — Transaksi berhasil dibuat |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 56`, balance akun diperbarui |
| **Status** | ✅ PASSED |

---

### TC-BVA-03 — Amount = 2 (Batas minimum + 1)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA03 |
| **Input** | `amount: 2` |
| **Expected Output** | HTTP 201 — Transaksi berhasil dibuat |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 57`, balance akun diperbarui |
| **Status** | ✅ PASSED |

---

### TC-BVA-04 — Amount = 500.000 (Nilai normal)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA04 |
| **Input** | `amount: 500000` |
| **Expected Output** | HTTP 201 — Transaksi berhasil dibuat |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 58`, balance akun diperbarui |
| **Status** | ✅ PASSED |

---

### TC-BVA-05 — Amount = 999.999.998 (Batas maksimum − 1)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA05 |
| **Input** | `amount: 999999998` |
| **Expected Output** | HTTP 201 — Transaksi berhasil dibuat |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 59`, balance akun diperbarui |
| **Status** | ✅ PASSED |

---

### TC-BVA-06 — Amount = 999.999.999 (Batas maksimum tepat)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA06 |
| **Input** | `amount: 999999999` |
| **Expected Output** | HTTP 201 — Transaksi berhasil dibuat |
| **Actual Output** | HTTP 201 — Transaksi berhasil, `id: 60`, balance akun diperbarui |
| **Status** | ✅ PASSED |

---

### TC-BVA-07 — Amount = 1.000.000.000 (Di atas batas maksimum) 🐛

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB02-BVA07 |
| **Input** | `amount: 1000000000` |
| **Expected Output** | HTTP 422 — Validasi gagal (`amount melebihi batas maksimum`) |
| **Actual Output** | HTTP 201 — **Transaksi BERHASIL dibuat** (`id: 61`, amount: 1.000.000.000 tersimpan ke database) |
| **Status** | ❌ **FAILED — BUG DITEMUKAN** |

> **⚠️ BUG #001 — Missing Max Validation on Amount Field**  
> Sistem menerima `amount = 1.000.000.000` yang seharusnya ditolak. Validasi di `TransactionController.php` hanya memiliki aturan `'amount' => 'required|numeric|min:1'` tanpa constraint `max`. Nilai miliaran dapat tersimpan ke database tanpa pembatasan, berpotensi menyebabkan overflow pada kolom balance atau ketidakakuratan laporan keuangan.  
>
> **Root Cause:** `app/Http/Controllers/API/TransactionController.php` — rule validasi `amount` tidak memiliki `max:999999999`  
> **Rekomendasi Fix:** Tambahkan `'amount' => 'required|numeric|min:1|max:999999999'`

---

## 4. Ringkasan Hasil BVA

| ID Test | Nilai Amount | Expected HTTP | Actual HTTP | Status |
|---------|-------------|---------------|-------------|--------|
| BVA-01 | 0 | 422 | 422 | ✅ PASSED |
| BVA-02 | 1 | 201 | 201 | ✅ PASSED |
| BVA-03 | 2 | 201 | 201 | ✅ PASSED |
| BVA-04 | 500.000 | 201 | 201 | ✅ PASSED |
| BVA-05 | 999.999.998 | 201 | 201 | ✅ PASSED |
| BVA-06 | 999.999.999 | 201 | 201 | ✅ PASSED |
| BVA-07 | 1.000.000.000 | 422 | **201** | ❌ **FAILED** |

**Total:** 7 test case | ✅ 6 Passed | ❌ 1 Failed | 🐛 1 Bug Ditemukan

---

## 5. Bukti Pengujian (Test Evidence)

File JSON hasil pengujian tersimpan di direktori `screenshots/black-box/`:

| File | Keterangan |
|------|------------|
| `BB02-BVA01-amount0.json` | HTTP 422 — amount 0 ditolak |
| `BB02-BVA02-amount1.json` | HTTP 201 — amount 1 diterima |
| `BB02-BVA03-amount2.json` | HTTP 201 — amount 2 diterima |
| `BB02-BVA04-amount500000.json` | HTTP 201 — amount 500.000 diterima |
| `BB02-BVA05-amount999999998.json` | HTTP 201 — amount 999.999.998 diterima |
| `BB02-BVA06-amount999999999.json` | HTTP 201 — amount 999.999.999 diterima |
| `BB02-BVA07-amount1000000000.json` | HTTP 201 — **BUG: amount 1 miliar diterima** |
