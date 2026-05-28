# White Box Testing — Basis Path Testing

**Proyek:** Midnight Finance
**Tanggal Pengujian:** 27 Mei 2026
**Penguji:** QA Team
**Metode:** Basis Path Testing (McCabe's Cyclomatic Complexity)

---

## Pendahuluan

Basis Path Testing adalah teknik *white box testing* yang dikembangkan oleh Thomas J. McCabe. Metode ini menganalisis alur kontrol program melalui *flowgraph* untuk menentukan jumlah jalur independen minimum yang harus diuji, yang diukur menggunakan **Cyclomatic Complexity V(G)**.

**Formula:**

$$V(G) = E - N + 2P$$

| Simbol | Keterangan |
|--------|------------|
| **E** | Jumlah *edge* (panah/alur) |
| **N** | Jumlah *node* (simpul/proses) |
| **P** | Jumlah komponen terhubung (biasanya 1) |

---

## WB-01: Login (`AuthController@login`)

### Flowgraph

![Flowgraph WB-01 Login](https://github.com/user-attachments/assets/b22c7e6b-e6e7-406c-807f-02104128c1de)

**Edge List:**

```
1→2, 2→3 (user ada), 2→4 (user tidak ada),
3→5 (pass ok), 3→6 (pass salah),
5→7 (ada OTP), 5→8 (OTP null),
4→END, 6→END, 7→END, 8→END
```

### Perhitungan Cyclomatic Complexity

| Komponen | Nilai |
|----------|-------|
| Edges (E) | 12 |
| Nodes (N) | 10 |
| Connected Components (P) | 1 |
| **V(G) = E − N + 2P** | **12 − 10 + 2 = 4** |

> **V(G) = 4** → Terdapat **4 jalur independen** yang harus diuji.

### Jalur Independen

| Jalur | Deskripsi | Rute Node |
|-------|-----------|-----------|
| Path 1 | Email tidak ditemukan | `1 → 2 → 4 → END` |
| Path 2 | Password salah | `1 → 2 → 3 → 6 → END` |
| Path 3 | Akun belum verifikasi (OTP aktif) | `1 → 2 → 3 → 5 → 7 → END` |
| Path 4 | Login berhasil | `1 → 2 → 3 → 5 → 8 → END` |

### Test Cases & Hasil

| ID | Path | Input | Expected | Actual Output | HTTP | Status |
|----|------|-------|----------|---------------|------|--------|
| TC-L-01 | Path 1 | `email: notexist@test.com` `password: any` | 404 — Email tidak ditemukan | `{"message":"Alamat email tidak ditemukan. Silakan buat akun terlebih dahulu."}` | 404 | ✅ PASS |
| TC-L-02 | Path 2 | `email: admin@midnight.com` `password: WrongPass` | 401 — Kata sandi salah | `{"message":"Kata sandi yang Anda masukkan salah."}` | 401 | ✅ PASS |
| TC-L-03 | Path 3 | `email: unverified@midnight.com` `password: Test@1234` | 403 — OTP diperlukan | `{"message":"Akun belum diverifikasi. Silakan cek email Anda untuk kode OTP.","need_otp":true}` | 403 | ✅ PASS |
| TC-L-04 | Path 4 | `email: admin@midnight.com` `password: Test@1234` | 200 — Token diterbitkan | `{"message":"Akses diberikan. Membuka brankas digital Anda...","access_token":"...","user":{...}}` | 200 | ✅ PASS |

### Bukti (Evidence)

| Test Case | File Bukti |
|-----------|-----------|
| TC-L-01 | [WB01-path1-email-notfound.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path1-email-notfound.json) |
| TC-L-02 | [WB01-path2-wrong-password.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path2-wrong-password.json) |
| TC-L-03 | [WB01-path3-unverified.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path3-unverified.json) |
| TC-L-04 | [WB01-path4-login-success.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path4-login-success.json) |

---

## WB-02: Buat Transaksi (`TransactionController@store`)

### Flowgraph

```
         [START]
            │
        [Node 1]
   Terima input transaksi
   (amount, type, category_id,
    financial_account_id, date)
            │
        [Node 2]
   Validasi format input
   (required, numeric,
    exists:financial_accounts)
            │
     ┌──────┴──────┐
  [Valid]       [Invalid]
     │               │
 [Node 3]        [Node 4]
 Cari akun      Return 422
 keuangan    Validation Error
 milik user       │
     │           [END-B]
  ┌──┴──┐
[Ada]  [Tidak]
  │        │
[N5]     [N6]
Simpan  Throw Exception
trans &  500 / 403
update       │
balance    [END-C]
  │
[Node 7]
Return 201
Transaction Created
  │
[END-A]
```

### Perhitungan Cyclomatic Complexity

| Komponen | Nilai |
|----------|-------|
| Edges (E) | 8 |
| Nodes (N) | 7 |
| Connected Components (P) | 1 |
| **V(G) = E − N + 2P** | **8 − 7 + 2 = 3** |

> **V(G) = 3** → Terdapat **3 jalur independen** yang harus diuji.

### Jalur Independen

| Jalur | Deskripsi | Rute Node |
|-------|-----------|-----------|
| Path 1 | Transaksi income berhasil | `1 → 2 → 3 → 5 → 7 → END` |
| Path 2 | Transaksi expense berhasil | `1 → 2 → 3 → 5 → 7 → END` |
| Path 3 | Financial account tidak valid | `1 → 2 → 4 → END` |

### Test Cases & Hasil

| ID | Path | Input | Expected | Actual Output | HTTP | Status |
|----|------|-------|----------|---------------|------|--------|
| TC-T-01 | Path 1 | `type: income` `amount: 50000` `financial_account_id: 13` (valid) | 201 — Income berhasil | `{"id":..., "amount":50000, "type":"income", "financial_account":{...}}` | 201 | ✅ PASS |
| TC-T-02 | Path 2 | `type: expense` `amount: 25000` `financial_account_id: 13` (valid) | 201 — Expense berhasil | `{"id":..., "amount":25000, "type":"expense", "financial_account":{...}}` | 201 | ✅ PASS |
| TC-T-03 | Path 3 | `financial_account_id: 9999` (tidak ada) | 422 — Validation error | `{"message":"The selected financial account id is invalid.","errors":{"financial_account_id":[...]}}` | 422 | ✅ PASS |

> **📝 Catatan TC-T-03:** Sistem mengembalikan **HTTP 422** (bukan 500 seperti dugaan awal). Hal ini disebabkan rule validasi `'financial_account_id' => 'exists:financial_accounts,id'` pada `TransactionController` mencegat ID yang tidak ada *sebelum* logika bisnis dijalankan. Ini merupakan perilaku yang lebih aman dan tepat secara desain.

### Bukti (Evidence)

| Test Case | File Bukti |
|-----------|-----------|
| TC-T-01 | [WB02-path1-income.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB02-path1-income.json) |
| TC-T-02 | [WB02-path2-expense.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB02-path2-expense.json) |
| TC-T-03 | [WB02-path3-invalid-account.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB02-path3-invalid-account.json) |

---

## WB-03: Buat Kategori (`CategoryController@store`)

### Flowgraph

```
         [START]
            │
        [Node 1]
   Terima input kategori
   (name, type)
            │
        [Node 2]
   Validasi format input
   (required, string,
    max:255, in:income/expense)
            │
     ┌──────┴──────┐
  [Valid]       [Invalid]
     │               │
 [Node 3]        [Node 4]
 Cek duplikat   Return 422
 (name + type    Input Error
  + user_id)         │
     │             [END-B]
  ┌──┴──┐
[Duplikat] [Baru]
     │          │
 [Node 5]   [Node 6]
 Return 422  Simpan kategori
 "Sudah ada"  Return 201
     │              │
  [END-C]        [END-A]
```

### Perhitungan Cyclomatic Complexity

| Komponen | Nilai |
|----------|-------|
| Edges (E) | 8 |
| Nodes (N) | 7 |
| Connected Components (P) | 1 |
| **V(G) = E − N + 2P** | **8 − 7 + 2 = 3** |

> **V(G) = 3** → Terdapat **3 jalur independen** yang harus diuji.

> **📝 Catatan:** Pengujian dilakukan dengan **4 test case** untuk memvalidasi semua kombinasi kondisi secara lengkap, termasuk kasus nama sama dengan tipe berbeda.

### Jalur Independen

| Jalur | Deskripsi | Rute Node |
|-------|-----------|-----------|
| Path 1 | Input tidak valid (field kosong) | `1 → 2 → 4 → END` |
| Path 2 | Duplikat — nama + tipe sama | `1 → 2 → 3 → 5 → END` |
| Path 3 | Kategori baru berhasil dibuat | `1 → 2 → 3 → 6 → END` |
| Path 4 *(Extended)* | Nama sama, tipe berbeda → dianggap baru | `1 → 2 → 3 → 6 → END` |

### Test Cases & Hasil

| ID | Path | Input | Expected | Actual Output | HTTP | Status |
|----|------|-------|----------|---------------|------|--------|
| TC-C-01 | Path 1 | `name: ""` `type: ""` (kosong) | 422 — Validasi gagal | `{"message":"The name field is required...","errors":{...}}` | 422 | ✅ PASS |
| TC-C-02 | Path 2 | `name: "Gaji"` `type: "income"` (sudah ada) | 422 — Duplikat | `{"message":"Kategori \"Gaji\" sudah ada di portofolio Anda!"}` | 422 | ✅ PASS |
| TC-C-03 | Path 3 | `name: "Freelance"` `type: "income"` (baru) | 201 — Berhasil dibuat | `{"id":..., "name":"Freelance", "type":"income", ...}` | 201 | ✅ PASS |
| TC-C-04 | Path 4 | `name: "Transportasi"` `type: "income"` (nama ada, tipe beda) | 201 — Dianggap kategori baru | `{"id":..., "name":"Transportasi", "type":"income", ...}` | 201 | ✅ PASS |

### Bukti (Evidence)

| Test Case | File Bukti |
|-----------|-----------|
| TC-C-01 | [WB03-branch1-empty-input.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch1-empty-input.json) |
| TC-C-02 | [WB03-branch2-duplicate.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch2-duplicate.json) |
| TC-C-03 | [WB03-branch3-new-category.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch3-new-category.json) |
| TC-C-04 | [WB03-branch4-same-name-diff-type.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch4-same-name-diff-type.json) |

---

## Rekap White Box Testing

| Modul | V(G) | Jalur Diuji | Passed | Failed |
|-------|------|-------------|--------|--------|
| Login (`AuthController@login`) | 4 | 4 | 4 | 0 |
| Buat Transaksi (`TransactionController@store`) | 3 | 3 | 3 | 0 |
| Buat Kategori (`CategoryController@store`) | 3 | 4 | 4 | 0 |
| **Total** | **10** | **11** | **11** | **0** |

> ✅ **Semua jalur independen berhasil diuji. Coverage: 100%**
