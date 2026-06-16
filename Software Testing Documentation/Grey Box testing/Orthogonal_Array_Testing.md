# GB-01 — Orthogonal Array Testing (OAT)
## Sistem: SaPoPoe FINANCE (Midnight Finance)
## Teknik: Gray Box Testing — Orthogonal Array Testing

---

> **Definisi Teknik:**
> Pengujian Array Ortogonal (OAT) adalah teknik pengujian perangkat lunak yang **menggunakan array ortogonal untuk membuat kasus uji**. Ini adalah pendekatan pengujian statistik yang sangat berguna ketika sistem yang akan diuji memiliki input data yang besar. Pengujian susunan ortogonal membantu memaksimalkan cakupan pengujian dengan **memasangkan dan menggabungkan input serta menguji sistem** dengan jumlah kasus pengujian yang relatif lebih sedikit untuk menghemat waktu.
>
> — Materi Pertemuan, Software Quality, T Informatika UKRI

---

## Langkah-Langkah Pengujian OAT

1. **Identifikasi variabel independen** (faktor) untuk skenario tersebut.
2. **Temukan array terkecil** dengan jumlah proses: `Tipe Array = L(Level²)`.
3. **Petakan faktor-faktor** tersebut ke dalam kolom array.
4. **Pilih nilai untuk level** "sisa" mana pun yang ada.
5. **Transkripsikan proses ke dalam kasus uji**, tambahkan kombinasi mencurigakan apa pun yang tidak dihasilkan.

---

## Modul 1 — Autentikasi (Form Login)

### Langkah 1 — Identifikasi Faktor dan Level

| Faktor | Kode | Level 1 | Level 2 | Level 3 |
|---|---|---|---|---|
| Email | A | Terdaftar & valid (`sultan@test.com`) | Tidak terdaftar (`tidakada@test.com`) | Kosong / tidak diisi (`""`) |
| Password | B | Benar (`password123`) | Salah (`salahpassword`) | Kosong / tidak diisi (`""`) |
| Format Request | C | JSON valid + header lengkap | Tanpa header `Content-Type` | Field tidak lengkap (hanya email) |

### Langkah 2 — Hitung Tipe Array

- **Jumlah Faktor** = 3 (Email, Password, Format Request)
- **Jumlah Level** = 3 (Level 1, Level 2, Level 3)
- **Tipe Array** = L(3²) = L(3×3) = **L(9 Kasus Uji)**

Sehingga berdasarkan tabel yang terbentuk OAT, diperlukan **9 kasus uji**.

### Langkah 3 — Tabel Array Ortogonal

| Kasus Uji # | Email (A) | Password (B) | Format (C) |
|---|---|---|---|
| 1 | 1 | 1 | 1 |
| 2 | 1 | 2 | 2 |
| 3 | 1 | 3 | 3 |
| 4 | 2 | 1 | 2 |
| 5 | 2 | 2 | 3 |
| 6 | 2 | 3 | 1 |
| 7 | 3 | 1 | 3 |
| 8 | 3 | 2 | 1 |
| 9 | 3 | 3 | 2 |

### Langkah 4 & 5 — Kasus Uji Lengkap

| TC | Email (A) | Password (B) | Format (C) | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | `sultan@test.com` (A1) | `password123` (B1) | JSON valid (C1) | Login berhasil → Dashboard | Dashboard berhasil ditampilkan | ✅ Passed |
| 2 | `sultan@test.com` (A1) | `salahpassword` (B2) | Tanpa Content-Type (C2) | Ditolak — password salah | "Kata sandi yang Anda masukkan salah." | ✅ Passed |
| 3 | `sultan@test.com` (A1) | `""` kosong (B3) | Field tidak lengkap (C3) | Ditolak — password kosong | "Harap isi bidang ini." | ✅ Passed |
| 4 | `tidakada@test.com` (A2) | `password123` (B1) | Tanpa Content-Type (C2) | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |
| 5 | `tidakada@test.com` (A2) | `salahpassword` (B2) | Field tidak lengkap (C3) | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |
| 6 | `tidakada@test.com` (A2) | `""` kosong (B3) | JSON valid (C1) | Ditolak — email tidak ditemukan | "Alamat email tidak ditemukan." | ✅ Passed |
| 7 | `""` kosong (A3) | `password123` (B1) | Field tidak lengkap (C3) | Ditolak — email wajib diisi | "Harap isi bidang ini." | ✅ Passed |
| 8 | `""` kosong (A3) | `salahpassword` (B2) | JSON valid (C1) | Ditolak — email wajib diisi | "Harap isi bidang ini." | ✅ Passed |
| 9 | `""` kosong (A3) | `""` kosong (B3) | Tanpa Content-Type (C2) | Ditolak — semua field kosong | "Harap isi bidang ini." | ✅ Passed |

> **Hasil:** 9/9 Passed. Sistem menangani seluruh kombinasi faktor dengan benar. Validasi berjalan konsisten di semua level email, password, dan format request.

---

## Modul 2 — Transfer (Pindah Dana)

### Langkah 1 — Identifikasi Faktor dan Level

| Faktor | Kode | Level 1 | Level 2 | Level 3 |
|---|---|---|---|---|
| Amount (Nominal) | A | Valid — `50.000` (di atas minimum) | Batas bawah — `0` (tidak valid) | Ekstrem — `999.999.999` (melebihi saldo) |
| Brankas Sumber | B | Mandiri (saldo mencukupi) | BCA (saldo tidak mencukupi) | Tidak dipilih |
| Brankas Tujuan | C | BSI (berbeda dengan sumber) | Mandiri (sama dengan sumber) | Tidak dipilih |

### Langkah 2 — Hitung Tipe Array

- **Jumlah Faktor** = 3 (Amount, Brankas Sumber, Brankas Tujuan)
- **Jumlah Level** = 3 (Level 1, Level 2, Level 3)
- **Tipe Array** = L(3²) = L(3×3) = **L(9 Kasus Uji)**

Sehingga berdasarkan tabel yang terbentuk OAT, diperlukan **9 kasus uji**.

### Langkah 3 — Tabel Array Ortogonal

| Kasus Uji # | Amount (A) | Brankas Sumber (B) | Brankas Tujuan (C) |
|---|---|---|---|
| 1 | 1 | 1 | 1 |
| 2 | 1 | 2 | 2 |
| 3 | 1 | 3 | 3 |
| 4 | 2 | 1 | 2 |
| 5 | 2 | 2 | 3 |
| 6 | 2 | 3 | 1 |
| 7 | 3 | 1 | 3 |
| 8 | 3 | 2 | 1 |
| 9 | 3 | 3 | 2 |

### Langkah 4 & 5 — Kasus Uji Lengkap

| TC | Amount (A) | Sumber (B) | Tujuan (C) | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | `50.000` (A1) | Mandiri (B1) | BSI (C1) | Transfer berhasil | "BERHASIL Dana berhasil dipindahkan!" | ✅ Passed |
| 2 | `50.000` (A1) | BCA (B2) | Mandiri (C2) | Ditolak — saldo tidak cukup | "GAGAL Saldo dompet asal tidak mencukupi..." | ✅ Passed |
| 3 | `50.000` (A1) | Tidak dipilih (B3) | Tidak dipilih (C3) | Ditolak — sumber wajib dipilih | "Harap isi bidang ini." | ✅ Passed |
| 4 | `0` (A2) | Mandiri (B1) | Mandiri (C2) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 5 | `0` (A2) | BCA (B2) | Tidak dipilih (C3) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 6 | `0` (A2) | Tidak dipilih (B3) | BSI (C1) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 7 | `999.999.999` (A3) | Mandiri (B1) | Tidak dipilih (C3) | Ditolak — tujuan wajib dipilih | "Harap isi bidang ini." | ✅ Passed |
| 8 | `999.999.999` (A3) | BCA (B2) | BSI (C1) | Ditolak — saldo tidak mencukupi | "GAGAL Saldo dompet asal tidak mencukupi..." | ✅ Passed |
| 9 | `999.999.999` (A3) | Tidak dipilih (B3) | Mandiri (C2) | Ditolak — sumber wajib dipilih | "Harap isi bidang ini." | ✅ Passed |

> **Hasil:** 9/9 Passed. Kombinasi ortogonal membuktikan bahwa validasi amount, sumber, dan tujuan bekerja secara independen maupun bersamaan. Tidak ada kombinasi yang lolos tanpa validasi.

---

## Modul 3 — Transaksi (Catat Aliran Dana)

### Langkah 1 — Identifikasi Faktor dan Level

| Faktor | Kode | Level 1 | Level 2 | Level 3 |
|---|---|---|---|---|
| Amount (Nominal) | A | Valid — `50.000` | Batas bawah — `0` / kosong | Ekstrem — `999.999.999` |
| Tipe Transaksi | B | Income / MASUK | Expense / KELUAR | Tidak dipilih |
| Portofolio | C | BSI (saldo cukup) | BCA (saldo tidak cukup / negatif) | Tidak dipilih |

### Langkah 2 — Hitung Tipe Array

- **Jumlah Faktor** = 3 (Amount, Tipe Transaksi, Portofolio)
- **Jumlah Level** = 3 (Level 1, Level 2, Level 3)
- **Tipe Array** = L(3²) = L(3×3) = **L(9 Kasus Uji)**

Sehingga berdasarkan tabel yang terbentuk OAT, diperlukan **9 kasus uji**.

### Langkah 3 — Tabel Array Ortogonal

| Kasus Uji # | Amount (A) | Tipe (B) | Portofolio (C) |
|---|---|---|---|
| 1 | 1 | 1 | 1 |
| 2 | 1 | 2 | 2 |
| 3 | 1 | 3 | 3 |
| 4 | 2 | 1 | 2 |
| 5 | 2 | 2 | 3 |
| 6 | 2 | 3 | 1 |
| 7 | 3 | 1 | 3 |
| 8 | 3 | 2 | 1 |
| 9 | 3 | 3 | 2 |

### Langkah 4 & 5 — Kasus Uji Lengkap

| TC | Amount (A) | Tipe (B) | Portofolio (C) | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | `50.000` (A1) | Income (B1) | BSI (C1) | Transaksi berhasil, saldo +50.000 | Saldo BSI bertambah Rp 50.000 | ✅ Passed |
| 2 | `50.000` (A1) | Expense (B2) | BCA (C2) | Ditolak — saldo BCA tidak mencukupi | ⚠️ return 201 — **saldo BCA makin negatif** | 🔴 **Failed** |
| 3 | `50.000` (A1) | Tidak dipilih (B3) | Tidak dipilih (C3) | Ditolak — portofolio wajib | "Pilih item pada daftar." | ✅ Passed |
| 4 | `0` (A2) | Income (B1) | BCA (C2) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 5 | `0` (A2) | Expense (B2) | Tidak dipilih (C3) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 6 | `0` (A2) | Tidak dipilih (B3) | BSI (C1) | Ditolak — amount tidak valid | "Harap isi bidang ini." | ✅ Passed |
| 7 | `999.999.999` (A3) | Income (B1) | Tidak dipilih (C3) | Ditolak — portofolio wajib | "Pilih item pada daftar." | ✅ Passed |
| 8 | `999.999.999` (A3) | Expense (B2) | BSI (C1) | Ditolak — saldo tidak mencukupi | ⚠️ return 201 — **saldo BSI menjadi negatif** | 🔴 **Failed** |
| 9 | `999.999.999` (A3) | Tidak dipilih (B3) | BCA (C2) | Ditolak — portofolio wajib | "Pilih item pada daftar." | ✅ Passed |

> **Hasil:** 7/9 Passed, **2 Failed**. OAT berhasil mengekspos defect kritis pada kombinasi Expense + portofolio dengan saldo tidak mencukupi (TC2 & TC8). Kombinasi ortogonal A1×B2×C2 dan A3×B2×C1 sama-sama menghasilkan saldo negatif — membuktikan bug bukan pada nilai amount tertentu, melainkan pada **semua kombinasi type=expense tanpa validasi saldo**.

---

## Modul 4 — Tabungan (Target Impian)

### Langkah 1 — Identifikasi Faktor dan Level

| Faktor | Kode | Level 1 | Level 2 | Level 3 |
|---|---|---|---|---|
| Nama Tabungan | A | Valid — `TabunganOAT` (10 karakter) | Kosong — `""` (0 karakter) | Panjang — 300 karakter (`AAA...`) |
| Target Amount | B | Valid — `500.000` (di atas minimum) | Batas bawah — `0` | Tidak diisi / kosong |
| Brankas Sumber | C | BSI (saldo cukup) | Mandiri (saldo berbeda) | Tidak dipilih |

### Langkah 2 — Hitung Tipe Array

- **Jumlah Faktor** = 3 (Nama, Target Amount, Brankas Sumber)
- **Jumlah Level** = 3 (Level 1, Level 2, Level 3)
- **Tipe Array** = L(3²) = L(3×3) = **L(9 Kasus Uji)**

Sehingga berdasarkan tabel yang terbentuk OAT, diperlukan **9 kasus uji**.

### Langkah 3 — Tabel Array Ortogonal

| Kasus Uji # | Nama (A) | Target Amount (B) | Brankas (C) |
|---|---|---|---|
| 1 | 1 | 1 | 1 |
| 2 | 1 | 2 | 2 |
| 3 | 1 | 3 | 3 |
| 4 | 2 | 1 | 2 |
| 5 | 2 | 2 | 3 |
| 6 | 2 | 3 | 1 |
| 7 | 3 | 1 | 3 |
| 8 | 3 | 2 | 1 |
| 9 | 3 | 3 | 2 |

### Langkah 4 & 5 — Kasus Uji Lengkap

| TC | Nama (A) | Target (B) | Brankas (C) | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| 1 | `TabunganOAT` (A1) | `500.000` (B1) | BSI (C1) | Tabungan berhasil dibuat | Tabungan muncul di daftar Target Impian | ✅ Passed |
| 2 | `TabunganOAT` (A1) | `0` (B2) | Mandiri (C2) | Ditolak — target minimal 1 | HTTP 422 "target amount minimal 1" | ✅ Passed |
| 3 | `TabunganOAT` (A1) | Kosong (B3) | Tidak dipilih (C3) | Ditolak — target wajib diisi | "Harap isi bidang ini." | ✅ Passed |
| 4 | `""` Kosong (A2) | `500.000` (B1) | Mandiri (C2) | Ditolak — nama wajib diisi | "Harap isi bidang ini." | ✅ Passed |
| 5 | `""` Kosong (A2) | `0` (B2) | Tidak dipilih (C3) | Ditolak — nama wajib diisi | "Harap isi bidang ini." | ✅ Passed |
| 6 | `""` Kosong (A2) | Kosong (B3) | BSI (C1) | Ditolak — nama & target kosong | "Harap isi bidang ini." | ✅ Passed |
| 7 | 300 karakter (A3) | `500.000` (B1) | Tidak dipilih (C3) | Ditolak — nama > 255 karakter | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |
| 8 | 300 karakter (A3) | `0` (B2) | BSI (C1) | Ditolak — nama > 255 & target 0 | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |
| 9 | 300 karakter (A3) | Kosong (B3) | Mandiri (C2) | Ditolak — nama > 255 karakter | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |

> **Hasil:** 9/9 Passed. Semua kombinasi ortogonal ditangani sistem dengan benar. Validasi nama (required, max 255) dan target amount (min 1) bekerja secara independen maupun bersamaan di semua level.

---

## Ringkasan Hasil OAT — Seluruh Sistem

| Modul | Faktor | Level | Tipe Array | TC | Passed | Failed | Temuan |
|---|---|---|---|---|---|---|---|
| Autentikasi | 3 | 3 | L(9) | 9 | 9 | 0 | Semua kombinasi ditangani benar |
| Transfer | 3 | 3 | L(9) | 9 | 9 | 0 | Validasi amount & brankas konsisten |
| Transaksi | 3 | 3 | L(9) | 9 | 7 | **2** | 🔴 TC2 & TC8: Expense tanpa validasi saldo → negatif |
| Tabungan | 3 | 3 | L(9) | 9 | 9 | 0 | Validasi nama & target konsisten |
| **TOTAL** | | | | **36** | **34** | **2** | |

> **Kesimpulan OAT:** Dari 36 kombinasi ortogonal yang diuji, ditemukan **2 kasus gagal** yang keduanya berada di modul Transaksi pada kombinasi `type=expense` dengan portofolio yang saldo-nya tidak mencukupi. OAT membuktikan bahwa defect saldo negatif terjadi **pada level kombinasi faktor tertentu**, bukan hanya pada nilai ekstrem — bahkan expense `Rp 50.000` pun menyebabkan saldo negatif jika portofolio yang dipilih sudah minus (TC2). Ini memperkuat temuan bahwa akar masalah ada di absennya validasi saldo di `TransactionController::store()`.
