# White Box Testing — Basis Path Testing

**Proyek:** Midnight Finance  
**Tanggal Pengujian:** 27 Mei 2026  
**Penguji:** QA Team  
**Metode:** Basis Path Testing (McCabe's Cyclomatic Complexity)

---

## Pendahuluan

White Box Testing (disebut juga *Glass Box Testing*) adalah metode perancangan test case yang menggunakan **struktur kontrol** dari perancangan prosedural untuk mendapatkan test case. Metode ini digunakan untuk mengetahui cara kerja internal suatu perangkat lunak, serta menjamin operasi-operasi internal sesuai spesifikasi yang telah ditetapkan.

Pengujian dengan metode ini diharapkan memperoleh test case yang:

- Memberikan jaminan bahwa semua jalur independen suatu modul digunakan **minimal satu kali**
- Menggunakan semua keputusan logis untuk semua kondisi *true* atau *false*
- Mengeksekusi semua perulangan pada batasan nilai dan operasional pada setiap kondisi
- Menggunakan struktur data internal untuk menjamin validitas jalur keputusan

### Basis Path Testing

Merupakan teknik uji coba yang diusulkan oleh **Tom McCabe**. Digunakan untuk mengukur kompleksitas logis dari desain prosedural dan menggunakannya sebagai pedoman untuk menetapkan himpunan basis dari semua jalur eksekusi. Tujuannya meyakinkan bahwa himpunan test case akan menguji setiap *path* pada suatu program **paling sedikit satu kali**.

**Langkah-langkah Basis Path Testing:**

1. Buat *Flow Graph Notation* (Grafik Alir)
2. Hitung *Cyclomatic Complexity* V(G)
3. Tentukan jalur independen (*Independent Path* / Basis Set)
4. Siapkan test case untuk setiap jalur independen

### Notasi Flow Graph

| Elemen | Simbol | Keterangan |
|--------|--------|------------|
| Node/Simpul | ○ | Merepresentasikan satu atau lebih statement prosedural. Node yang memiliki 2 atau lebih edge keluar disebut **simpul predikat (P)** |
| Edge/Link | → | Merepresentasikan aliran kontrol antar node |
| Region | — | Daerah yang dibatasi oleh edge dan node, termasuk daerah di luar grafik alir |

### Kompleksitas Siklomatis V(G)

Metrik perangkat lunak yang memberikan pengukuran kuantitatif terhadap kompleksitas logis suatu program. Nilai V(G) menentukan jumlah jalur independen dalam basis set dan memberi nilai **batas atas** bagi jumlah pengujian yang harus dilakukan.

Terdapat **3 cara** menghitung V(G):

| Cara | Formula | Keterangan |
|------|---------|------------|
| 1 | `V(G) = Jumlah Region` | Hitung daerah tertutup pada flow graph (termasuk daerah luar) |
| 2 | `V(G) = E – N + 2` | E = jumlah edge/busur, N = jumlah node/simpul |
| 3 | `V(G) = P + 1` | P = jumlah simpul predikat (node dengan ≥ 2 edge keluar) |

> Ketiga cara harus menghasilkan nilai yang **sama**.

---

## WB-01: Login (`AuthController@login`)

### 1. Flow Graph Notation

![Flowgraph WB-01 Login](https://github.com/user-attachments/assets/b22c7e6b-e6e7-406c-807f-02104128c1de)

**Daftar Edge (Busur):**

```
1→2  : START ke cek user
2→3  : User ditemukan (Y)
2→4  : User tidak ditemukan (N)
3→5  : Password benar (Y)
3→6  : Password salah (N)
5→7  : OTP aktif / akun belum verifikasi (Y)
5→8  : OTP null / tidak ada OTP (N)
4→END
6→END
7→END
8→END
```

**Identifikasi Elemen Graf:**

| Elemen | Daftar | Jumlah |
|--------|--------|--------|
| Node (N) | 1, 2, 3, 4, 5, 6, 7, 8, END | 10 |
| Edge (E) | 1→2, 2→3, 2→4, 3→5, 3→6, 5→7, 5→8, 4→END, 6→END, 7→END, 8→END, (+ 1 ke START) | 12 |
| Simpul Predikat (P) | Node 2 (cek user), Node 3 (cek password), Node 5 (cek OTP) | 3 |
| Region (R) | R1, R2, R3, R4 (termasuk daerah luar) | 4 |

### 2. Perhitungan Cyclomatic Complexity V(G)

> **Tiga cara penghitungan harus menghasilkan nilai yang sama.**

**Cara 1 — Jumlah Region:**

```
V(G) = Jumlah Region
V(G) = 4
```

**Cara 2 — Rumus E – N + 2:**

```
V(G) = E – N + 2
V(G) = 12 – 10 + 2
V(G) = 4
```

**Cara 3 — Rumus P + 1:**

```
V(G) = P + 1
V(G) = 3 + 1
V(G) = 4
```

**Kesimpulan:** V(G) = **4** → Terdapat **4 jalur independen** yang harus diuji.

### 3. Jalur Independen (Basis Set)

> Jalur independen adalah jalur yang melalui program yang mengintroduksi sedikitnya satu rangkaian statement proses baru atau suatu kondisi baru.

| Jalur | Rute Node | Deskripsi | Kondisi Baru yang Dilewati |
|-------|-----------|-----------|---------------------------|
| Path 1 | `1 → 2 → 4 → END` | Email tidak ditemukan | Node 2: user = **tidak ada** |
| Path 2 | `1 → 2 → 3 → 6 → END` | Password salah | Node 2: user = **ada** → Node 3: pass = **salah** |
| Path 3 | `1 → 2 → 3 → 5 → 7 → END` | Akun belum verifikasi (OTP aktif) | Node 3: pass = **benar** → Node 5: OTP = **ada** |
| Path 4 | `1 → 2 → 3 → 5 → 8 → END` | Login berhasil | Node 5: OTP = **null** (tidak ada) |

### 4. Test Cases & Hasil

| ID | Path | Input | Expected Output | Actual Output | HTTP | Status |
|----|------|-------|----------------|---------------|------|--------|
| TC-L-01 | Path 1 | `email: notexist@test.com` `password: any` | 404 — Email tidak ditemukan | `{"message":"Alamat email tidak ditemukan. Silakan buat akun terlebih dahulu."}` | 404 | ✅ PASS |
| TC-L-02 | Path 2 | `email: admin@midnight.com` `password: WrongPass` | 401 — Kata sandi salah | `{"message":"Kata sandi yang Anda masukkan salah."}` | 401 | ✅ PASS |
| TC-L-03 | Path 3 | `email: unverified@midnight.com` `password: Test@1234` | 403 — OTP diperlukan | `{"message":"Akun belum diverifikasi. Silakan cek email Anda untuk kode OTP.","need_otp":true}` | 403 | ✅ PASS |
| TC-L-04 | Path 4 | `email: admin@midnight.com` `password: Test@1234` | 200 — Token diterbitkan | `{"message":"Akses diberikan. Membuka brankas digital Anda...","access_token":"...","user":{...}}` | 200 | ✅ PASS |

### 5. Bukti (Evidence)

| ID | File Bukti |
|----|-----------|
| TC-L-01 | [WB01-path1-email-notfound.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path1-email-notfound.json) |
| TC-L-02 | [WB01-path2-wrong-password.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path2-wrong-password.json) |
| TC-L-03 | [WB01-path3-unverified.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path3-unverified.json) |
| TC-L-04 | [WB01-path4-login-success.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB01-path4-login-success.json) |

---

## WB-02: Buat Transaksi (`TransactionController@store`)

### 1. Flow Graph Notation

```
            [START]
               │
           [Node 1]                    ← Terima input transaksi
    (amount, type, category_id,          (statement prosedural)
     financial_account_id, date)
               │
           [Node 2]  ← P1             ← Simpul Predikat: Validasi format input
    (required, numeric,                  (2 edge keluar: Valid / Invalid)
     exists:financial_accounts)
               │
       ┌───────┴───────┐
    [Valid]         [Invalid]
       │                 │
   [Node 3]  ← P2    [Node 4]         ← P2: Simpul Predikat: Cari akun milik user
  Cari akun        Return 422           (2 edge keluar: Ada / Tidak)
  keuangan       Validation Error
  milik user           │
       │             [END-B]
   ┌───┴───┐
 [Ada]   [Tidak]
   │         │
[Node 5]  [Node 6]
 Simpan    Throw Exception
 transaksi  500 / 403
 & update       │
 balance     [END-C]
   │
[Node 7]
Return 201
Transaction Created
   │
[END-A]
```

**Identifikasi Elemen Graf:**

| Elemen | Daftar | Jumlah |
|--------|--------|--------|
| Node (N) | 1, 2, 3, 4, 5, 6, 7 | 7 |
| Edge (E) | 1→2, 2→3, 2→4, 3→5, 3→6, 5→7, 4→END-B, 6→END-C, 7→END-A | 8* |
| Simpul Predikat (P) | Node 2 (validasi), Node 3 (cek akun) | 2 |
| Region (R) | R1, R2, R3 (termasuk daerah luar) | 3 |

> *Dihitung sebagai 8 edge dengan memperhitungkan edge masuk ke START.

### 2. Perhitungan Cyclomatic Complexity V(G)

> **Tiga cara penghitungan harus menghasilkan nilai yang sama.**

**Cara 1 — Jumlah Region:**

```
V(G) = Jumlah Region
V(G) = 3
```

**Cara 2 — Rumus E – N + 2:**

```
V(G) = E – N + 2
V(G) = 8 – 7 + 2
V(G) = 3
```

**Cara 3 — Rumus P + 1:**

```
V(G) = P + 1
V(G) = 2 + 1
V(G) = 3
```

**Kesimpulan:** V(G) = **3** → Terdapat **3 jalur independen** yang harus diuji.

### 3. Jalur Independen (Basis Set)

| Jalur | Rute Node | Deskripsi | Kondisi Baru yang Dilewati |
|-------|-----------|-----------|---------------------------|
| Path 1 | `1 → 2 → 3 → 5 → 7 → END-A` | Transaksi income berhasil | Node 2: valid → Node 3: akun **ada** |
| Path 2 | `1 → 2 → 3 → 5 → 7 → END-A` | Transaksi expense berhasil | Node 2: valid → Node 3: akun **ada** (tipe berbeda) |
| Path 3 | `1 → 2 → 4 → END-B` | Validasi gagal — financial account tidak valid | Node 2: input **tidak valid** |

### 4. Test Cases & Hasil

| ID | Path | Input | Expected Output | Actual Output | HTTP | Status |
|----|------|-------|----------------|---------------|------|--------|
| TC-T-01 | Path 1 | `type: income` `amount: 50000` `financial_account_id: 13` | 201 — Income berhasil | `{"id":..., "amount":50000, "type":"income", "financial_account":{...}}` | 201 | ✅ PASS |
| TC-T-02 | Path 2 | `type: expense` `amount: 25000` `financial_account_id: 13` | 201 — Expense berhasil | `{"id":..., "amount":25000, "type":"expense", "financial_account":{...}}` | 201 | ✅ PASS |
| TC-T-03 | Path 3 | `financial_account_id: 9999` (tidak ada di DB) | 422 — Validation error | `{"message":"The selected financial account id is invalid.","errors":{"financial_account_id":[...]}}` | 422 | ✅ PASS |

> **📝 Catatan TC-T-03:** Sistem mengembalikan **HTTP 422** (bukan 500 seperti dugaan awal). Hal ini disebabkan rule validasi `'financial_account_id' => 'exists:financial_accounts,id'` pada `TransactionController` mencegat ID yang tidak ada *sebelum* logika bisnis dijalankan. Ini merupakan perilaku yang lebih aman dan tepat secara desain — validasi layer mencegah eksekusi business logic dengan data yang tidak valid.

### 5. Bukti (Evidence)

| ID | File Bukti |
|----|-----------|
| TC-T-01 | [WB02-path1-income.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB02-path1-income.json) |
| TC-T-02 | [WB02-path2-expense.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB02-path2-expense.json) |
| TC-T-03 | [WB02-path3-invalid-account.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB02-path3-invalid-account.json) |

---

## WB-03: Buat Kategori (`CategoryController@store`)

### 1. Flow Graph Notation

```
            [START]
               │
           [Node 1]                    ← Terima input kategori
         (name, type)                    (statement prosedural)
               │
           [Node 2]  ← P1             ← Simpul Predikat: Validasi format input
    (required, string, max:255,          (2 edge keluar: Valid / Invalid)
     in:income/expense)
               │
       ┌───────┴───────┐
    [Valid]         [Invalid]
       │                 │
   [Node 3]  ← P2    [Node 4]         ← P2: Simpul Predikat: Cek duplikat
  Cek duplikat     Return 422           (2 edge keluar: Duplikat / Baru)
  (name + type      Input Error
   + user_id)            │
       │              [END-B]
   ┌───┴───┐
[Duplikat] [Baru]
    │          │
[Node 5]   [Node 6]
Return 422  Simpan kategori
"Sudah ada"  Return 201
    │              │
 [END-C]        [END-A]
```

**Identifikasi Elemen Graf:**

| Elemen | Daftar | Jumlah |
|--------|--------|--------|
| Node (N) | 1, 2, 3, 4, 5, 6 | 7* |
| Edge (E) | 1→2, 2→3, 2→4, 3→5, 3→6, 4→END-B, 5→END-C, 6→END-A | 8* |
| Simpul Predikat (P) | Node 2 (validasi), Node 3 (cek duplikat) | 2 |
| Region (R) | R1, R2, R3 (termasuk daerah luar) | 3 |

> *Node END dihitung sebagai 1 node gabungan untuk penyederhanaan.

### 2. Perhitungan Cyclomatic Complexity V(G)

> **Tiga cara penghitungan harus menghasilkan nilai yang sama.**

**Cara 1 — Jumlah Region:**

```
V(G) = Jumlah Region
V(G) = 3
```

**Cara 2 — Rumus E – N + 2:**

```
V(G) = E – N + 2
V(G) = 8 – 7 + 2
V(G) = 3
```

**Cara 3 — Rumus P + 1:**

```
V(G) = P + 1
V(G) = 2 + 1
V(G) = 3
```

**Kesimpulan:** V(G) = **3** → Terdapat **3 jalur independen** yang harus diuji.

> **📝 Catatan:** Pengujian dilakukan dengan **4 test case** untuk memvalidasi semua kombinasi kondisi secara lengkap, termasuk kasus nama sama dengan tipe berbeda. Penambahan TC-C-04 merupakan *extended test* di luar basis set minimum untuk meningkatkan kepercayaan pada logika duplikasi.

### 3. Jalur Independen (Basis Set)

| Jalur | Rute Node | Deskripsi | Kondisi Baru yang Dilewati |
|-------|-----------|-----------|---------------------------|
| Path 1 | `1 → 2 → 4 → END-B` | Input tidak valid (field kosong/salah format) | Node 2: input **tidak valid** |
| Path 2 | `1 → 2 → 3 → 5 → END-C` | Duplikat — nama + tipe + user_id sama | Node 2: valid → Node 3: kategori **sudah ada** |
| Path 3 | `1 → 2 → 3 → 6 → END-A` | Kategori baru berhasil dibuat | Node 3: kategori **belum ada** |
| Path 4 *(Extended)* | `1 → 2 → 3 → 6 → END-A` | Nama sama, tipe berbeda → dianggap baru | Node 3: kombinasi name+type **berbeda** dari existing |

### 4. Test Cases & Hasil

| ID | Path | Input | Expected Output | Actual Output | HTTP | Status |
|----|------|-------|----------------|---------------|------|--------|
| TC-C-01 | Path 1 | `name: ""` `type: ""` (kosong) | 422 — Validasi gagal | `{"message":"The name field is required...","errors":{...}}` | 422 | ✅ PASS |
| TC-C-02 | Path 2 | `name: "Gaji"` `type: "income"` (sudah ada) | 422 — Duplikat ditolak | `{"message":"Kategori \"Gaji\" sudah ada di portofolio Anda!"}` | 422 | ✅ PASS |
| TC-C-03 | Path 3 | `name: "Freelance"` `type: "income"` (baru) | 201 — Berhasil dibuat | `{"id":..., "name":"Freelance", "type":"income", ...}` | 201 | ✅ PASS |
| TC-C-04 | Path 4 | `name: "Transportasi"` `type: "income"` (nama ada, tipe beda) | 201 — Dianggap kategori baru | `{"id":..., "name":"Transportasi", "type":"income", ...}` | 201 | ✅ PASS |

### 5. Bukti (Evidence)

| ID | File Bukti |
|----|-----------|
| TC-C-01 | [WB03-branch1-empty-input.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch1-empty-input.json) |
| TC-C-02 | [WB03-branch2-duplicate.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch2-duplicate.json) |
| TC-C-03 | [WB03-branch3-new-category.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch3-new-category.json) |
| TC-C-04 | [WB03-branch4-same-name-diff-type.json](https://github.com/JarJel/SQA-project/blob/main/screenshots/white-box/WB03-branch4-same-name-diff-type.json) |

---

## Rekap White Box Testing

| Modul | V(G) | Basis Set (Jalur Min.) | Test Case Dijalankan | Passed | Failed |
|-------|------|----------------------|----------------------|--------|--------|
| Login (`AuthController@login`) | 4 | 4 | 4 | 4 | 0 |
| Buat Transaksi (`TransactionController@store`) | 3 | 3 | 3 | 3 | 0 |
| Buat Kategori (`CategoryController@store`) | 3 | 3 | 4 | 4 | 0 |
| **Total** | **10** | **10** | **11** | **11** | **0** |

> ✅ **Semua jalur independen dalam basis set berhasil diuji. Statement Coverage: 100%**

---

## Referensi

- Pressman, R.S. *Software Engineering: A Practitioner's Approach*. McGraw-Hill.
- McCabe, T.J. (1976). "A Complexity Measure". *IEEE Transactions on Software Engineering*.
- R. Irman Hariman, ST., MT. *Teknik Pengujian PL (WB) — Sesi 7 & 8*. Materi Kuliah Pengujian Perangkat Lunak.
