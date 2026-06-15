# BB-01 — Boundary Value Analysis (BVA)
## Sistem: SaPoPoe FINANCE (Midnight Finance)
## Teknik: Black Box Testing — Boundary Value Analysis

---

> **Definisi Teknik:**
> Teknik BVA digunakan untuk **melakukan validasi fungsionalitas system berdasarkan persyaratan dan spesifikasi**, sehingga diperlukan analisis terhadap **Nilai Batas**. BVA merupakan **Perluasan dari Model Equivalence Partitioning**, dengan memasukan nilai sedikit dari minimum dan kurang sedikit dari maksimum.
>
> — Materi Pertemuan 11, Software Quality, T Informatika UKRI

---

## Modul 1 — Autentikasi: Form Login

### Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|---|---|---|---|
| 1 | Email | String | 0 < Panjang Karakter <= 255 dan format valid (mengandung @ dan domain) |
| 2 | Password | String | 0 < Panjang Karakter <= 255 |

### Batasan Equivalence Class

| No | Field Name | Boundary | Value | Input Data |
|---|---|---|---|---|
| 1 | Email | Batas Bawah (BB) | 0 | `""` (kosong / 0 karakter) |
| | | Batas Atas (BA) | 255 | email 300 karakter (melebihi batas) |
| 2 | Password | Batas Bawah (BB) | 0 | `""` (kosong / 0 karakter) |
| | | Batas Atas (BA) | 255 | string 300 karakter (melebihi batas) |

### Table Case

| No | Test Case | Input Email (panjang) | Input Password (panjang) | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | TC1 | 0 (kosong) | 0 (kosong) | Invalid / Tidak Diproses | return 422 "email wajib diisi, password wajib diisi" | ✅ Passed |
| 2 | TC2 | 0 (kosong) | 8 karakter | Invalid / Tidak Diproses | return 422 "email wajib diisi" | ✅ Passed |
| 3 | TC3 | valid (15 karakter) | 0 (kosong) | Invalid / Tidak Diproses | return 422 "password wajib diisi" | ✅ Passed |
| 4 | TC4 | valid (15 karakter) | valid (8 karakter) | Valid / Diproses → 200 + token | return 200 + access_token | ✅ Passed |
| 5 | TC5 | 255 karakter (format email rusak) | 255 karakter | Invalid (format email tidak valid) | return 422 "email harus berformat email" | ✅ Passed |
| 6 | TC6 | 300 karakter | 300 karakter | Invalid (melebihi batas) | return 422 "email harus berformat email" | ✅ Passed |

---

> **Analisis SQA — Modul Auth:**
> Semua 6 test case menunjukkan sistem menolak input di luar batas dengan benar (TC1–TC3 dan TC5–TC6) dan menerima input valid (TC4). Validasi batas pada form Login sudah berjalan sesuai spesifikasi. Laravel melakukan validasi `required` dan `email` sebelum data diproses controller.

---

## Modul 2 — Transfer: Form Pindah Dana

### Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|---|---|---|---|
| 1 | Amount (Nominal) | Numeric | 1 <= Amount (minimal 1, tidak boleh 0 atau negatif) |
| 2 | Admin Fee (Biaya Admin) | Numeric | 0 <= Admin Fee (boleh 0, tidak boleh negatif) |

### Batasan Equivalence Class

| No | Field Name | Boundary | Value | Input Data |
|---|---|---|---|---|
| 1 | Amount | Batas Bawah (BB) | 0 | `0` (nol — di bawah minimum) |
| | | Batas Atas (BA) | tidak terbatas | `999.999.999` (nilai ekstrem) |
| 2 | Admin Fee | Batas Bawah (BB) | -1 | `-1` (negatif — di bawah minimum) |
| | | Batas Atas (BA) | tidak terbatas | `999.999` (nilai ekstrem) |

### Table Case

| No | Test Case | Input Amount | Input Admin Fee | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | TC1 | 0 | 0 | Invalid / Tidak Diproses | return 422 "amount minimal 1" | ✅ Passed |
| 2 | TC2 | -1 | 0 | Invalid / Tidak Diproses | return 422 "amount minimal 1" | ✅ Passed |
| 3 | TC3 | 50.000 | -1 | Invalid / Tidak Diproses | return 422 "admin fee minimal 0" | ✅ Passed |
| 4 | TC4 | 50.000 | 0 | Valid / Diproses → 200 transfer berhasil | return 200 "Transfer berhasil!" | ✅ Passed |
| 5 | TC5 | 100.000 | 2.500 | Valid / Diproses → 200 transfer + biaya admin | return 200 + 3 transaksi dicatat | ✅ Passed |
| 6 | TC6 | 999.999.999 | 0 | Invalid jika saldo kurang → 400 | return 400 "Saldo tidak mencukupi" | ✅ Passed |

---

> **Analisis SQA — Modul Transfer:**
> Validasi batas amount dan admin fee berjalan benar — sistem menolak nilai 0 dan negatif untuk amount (TC1, TC2), serta nilai negatif untuk admin fee (TC3). TC6 mengkonfirmasi sistem juga melakukan pengecekan saldo saat nilai ekstrem diinputkan. Seluruh 6 test case Passed.

---

## Modul 3 — Transaksi: Form Catat Aliran Dana

### Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|---|---|---|---|
| 1 | Amount (Nominal) | Numeric | 1 <= Amount (minimal 1, tidak boleh 0 atau negatif) |
| 2 | Type (Tipe) | String | Hanya `income` atau `expense` (nilai di luar itu tidak valid) |
| 3 | Description (Keterangan) | String | 0 <= Panjang Karakter (boleh kosong, nullable) |

### Batasan Equivalence Class

| No | Field Name | Boundary | Value | Input Data |
|---|---|---|---|---|
| 1 | Amount | Batas Bawah (BB) | 0 | `0` (nol — di bawah minimum) |
| | | Batas Atas (BA) | tidak terbatas | `999.999.999` (nilai ekstrem tanpa cek saldo) |
| 2 | Type | Batas Bawah (BB) | `""` (kosong) | `""` (tidak diisi) |
| | | Batas Atas (BA) | nilai tidak terdaftar | `"transfer"` (di luar enum valid) |
| 3 | Description | Batas Bawah (BB) | 0 | `""` (kosong — boleh kosong) |
| | | Batas Atas (BA) | tidak terbatas | string 1.000 karakter |

### Table Case

| No | Test Case | Input Amount | Input Type | Input Description | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|---|
| 1 | TC1 | 0 | `income` | — | Invalid / Tidak Diproses | return 422 "amount minimal 1" | ✅ Passed |
| 2 | TC2 | -1 | `income` | — | Invalid / Tidak Diproses | return 422 "amount minimal 1" | ✅ Passed |
| 3 | TC3 | 50.000 | `""` (kosong) | — | Invalid / Tidak Diproses | return 422 "type wajib diisi" | ✅ Passed |
| 4 | TC4 | 50.000 | `"transfer"` | — | Invalid / Tidak Diproses | return 422 "type harus income atau expense" | ✅ Passed |
| 5 | TC5 | 50.000 | `income` | `""` (kosong) | Valid / Diproses → 201 + saldo bertambah | return 201 + transaksi tercatat | ✅ Passed |
| 6 | TC6 | 999.999.999 | `expense` | — | Invalid jika saldo kurang → tolak | return 201 **saldo menjadi NEGATIF** ⚠️ | 🔴 **Failed** |

---

> **Analisis SQA — Modul Transaksi:**
> TC1–TC5 semua Passed — sistem menolak amount invalid dan type di luar enum. Namun **TC6 Failed**: saat amount ekstrem diinputkan untuk type=expense, sistem tidak memvalidasi kecukupan saldo. Akibatnya saldo akun bisa menjadi negatif meskipun response HTTP 201 (sukses). Ini adalah **defect kritis** yang ditemukan melalui BVA pada batas atas amount.

---

## Modul 4 — Tabungan: Form Target Impian

### Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|---|---|---|---|
| 1 | Nama Tabungan | String | 0 < Panjang Karakter <= 255 (wajib diisi) |
| 2 | Target Amount | Numeric | 1 <= Target Amount (minimal 1) |
| 3 | Current Amount | Numeric | 0 <= Current Amount (boleh 0, tidak boleh negatif) |

### Batasan Equivalence Class

| No | Field Name | Boundary | Value | Input Data |
|---|---|---|---|---|
| 1 | Nama Tabungan | Batas Bawah (BB) | 0 | `""` (kosong) |
| | | Batas Atas (BA) | 255 | string 300 karakter |
| 2 | Target Amount | Batas Bawah (BB) | 0 | `0` (di bawah minimum) |
| | | Batas Atas (BA) | tidak terbatas | `999.999.999` |
| 3 | Current Amount | Batas Bawah (BB) | -1 | `-1` (negatif) |
| | | Batas Atas (BA) | tidak terbatas | nilai melebihi saldo akun |

### Table Case

| No | Test Case | Input Nama (panjang) | Input Target | Input Current | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|---|
| 1 | TC1 | 0 (kosong) | 500.000 | 0 | Invalid / Tidak Diproses | return 422 "name wajib diisi" | ✅ Passed |
| 2 | TC2 | valid (10 karakter) | 0 | 0 | Invalid / Tidak Diproses | return 422 "target amount minimal 1" | ✅ Passed |
| 3 | TC3 | valid (10 karakter) | 500.000 | -1 | Invalid / Tidak Diproses | return 422 "current amount minimal 0" | ✅ Passed |
| 4 | TC4 | valid (10 karakter) | 500.000 | 0 | Valid / Diproses → 201 (tanpa setoran awal) | return 201 + tabungan dibuat | ✅ Passed |
| 5 | TC5 | valid (10 karakter) | 500.000 | 100.000 | Valid / Diproses → 201 + saldo berkurang 100k | return 201 + saldo berkurang | ✅ Passed |
| 6 | TC6 | 300 karakter | 500.000 | 0 | Invalid (nama melebihi 255 karakter) | return 422 "name maksimal 255 karakter" | ✅ Passed |
| 7 | TC7 | valid (10 karakter) | 500.000 | 999.999.999 | Invalid (melebihi saldo akun) | return 201 **saldo menjadi NEGATIF** ⚠️ | 🔴 **Failed** |

---

> **Analisis SQA — Modul Tabungan:**
> TC1–TC6 semua Passed — sistem menolak nama kosong, target=0, current negatif, dan nama terlalu panjang. Namun **TC7 Failed**: saat current amount melebihi saldo akun diinputkan, sistem tidak memvalidasi kecukupan saldo sebelum menguranginya. Ini konsisten dengan temuan di WB-04 dan WB-05 — bug yang sama ditemukan dari sudut pandang Black Box (tanpa melihat kode) melalui pengujian nilai batas atas.

---

## Ringkasan Hasil BVA — Seluruh Sistem

| Modul | Jumlah TC | Passed | Failed | Temuan |
|---|---|---|---|---|
| Auth — Form Login | 6 | 6 | 0 | Semua batas valid dan invalid ditangani benar |
| Transfer — Form Pindah Dana | 6 | 6 | 0 | Validasi amount dan admin fee berjalan benar |
| Transaksi — Form Catat Aliran Dana | 6 | 5 | **1** | 🔴 TC6: Expense amount ekstrem → saldo negatif |
| Tabungan — Form Target Impian | 7 | 6 | **1** | 🔴 TC7: Current amount > saldo → saldo negatif |
| **TOTAL** | **25** | **23** | **2** | |
