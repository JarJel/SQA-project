# BB-02 — Equivalence Partitioning (EP)
## Sistem: SaPoPoe FINANCE (Midnight Finance)
## Teknik: Black Box Testing — Equivalence Partitioning

---

> **Definisi Teknik:**
> Teknik Equivalence Partitioning (EP) digunakan untuk **membagi data input ke dalam kelas-kelas ekuivalen**, sehingga setiap kelas cukup diwakili oleh **satu test case**. Setiap kelas ekuivalen terdiri dari:
> - **Kelas Valid**: input yang sesuai dengan syarat sistem dan seharusnya diproses
> - **Kelas Tidak Valid**: input yang melanggar syarat sistem dan seharusnya ditolak
>
> — Materi Pertemuan 11, Software Quality, T Informatika UKRI

---

## Modul 1 — Autentikasi: Form Login

### Kelas Ekuivalen

| No | Kondisi Input | Kelas Valid | Kelas Tidak Valid |
|---|---|---|---|
| 1 | Email | Email terdaftar dengan format valid | (KI-1) Email tidak terdaftar di sistem; (KI-2) Password salah |
| 2 | Password | Password sesuai dengan akun | (KI-2) Password tidak sesuai dengan akun |

### Kasus Uji

| No | Test Case | Kelas Ekuivalen | Input Email | Input Password | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|---|
| 1 | TC1 | Valid (KV) | `sultan@test.com` (terdaftar) | `password123` (benar) | Valid / Login berhasil → Dashboard tampil | Dashboard berhasil ditampilkan | ✅ Passed |
| 2 | TC2 | Tidak Valid (KI-1) | `sultan@test.com` (terdaftar) | `salahpassword` (salah) | Invalid / Ditolak | "Kata sandi yang Anda masukkan salah." | ✅ Passed |
| 3 | TC3 | Tidak Valid (KI-2) | `gudangmateri881@gmail.com` (tidak terdaftar) | `password123` | Invalid / Ditolak | "Alamat email tidak ditemukan. Silakan buat akun terlebih dahulu." | ✅ Passed |

### Screenshot Bukti Pengujian

**TC1 — Login Berhasil (Kelas Valid)**

<img width="1920" height="1080" alt="bva-auth-tc4" src="https://github.com/user-attachments/assets/ee9e2284-f017-4ddc-bf07-94975b132198" />


**TC2 — Password Salah (Kelas Tidak Valid 1)**

<img width="1920" height="1080" alt="bva-auth-tc3" src="https://github.com/user-attachments/assets/7acc9539-2e27-40ff-9b0a-f410393fe48e" />


**TC3 — Email Tidak Terdaftar (Kelas Tidak Valid 2)**

<img width="1920" height="1080" alt="ep-auth-tc3" src="https://github.com/user-attachments/assets/c687a7d4-0def-41f8-aa56-0032086e12f8" />


---

> **Analisis SQA — Modul Auth:**
> Ketiga kelas ekuivalen diuji dengan benar. Sistem menerima login saat email terdaftar dan password benar (TC1), dan menolak dengan pesan yang sesuai saat password salah (TC2) maupun email tidak terdaftar (TC3). Seluruh 3 test case Passed.

---

## Modul 2 — Transfer: Form Pindah Dana

### Kelas Ekuivalen

| No | Kondisi Input | Kelas Valid | Kelas Tidak Valid |
|---|---|---|---|
| 1 | Amount (Nominal) | Amount > 0 (ada nilai yang valid) | (KI-1) Amount = 0 atau kosong (di bawah minimum) |
| 2 | Saldo Sumber | Saldo brankas asal ≥ amount + admin fee | (KI-2) Saldo brankas asal < amount (tidak mencukupi) |

### Kasus Uji

| No | Test Case | Kelas Ekuivalen | Input Amount | Sumber → Tujuan | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|---|
| 1 | TC1 | Valid (KV) | 50.000 | Mandiri → BSI | Valid / Transfer berhasil | "BERHASIL Dana berhasil dipindahkan!" | ✅ Passed |
| 2 | TC2 | Tidak Valid (KI-1) | 0 (kosong) | BCA → Mandiri | Invalid / Ditolak | "Harap isi bidang ini." — form tidak dikirim | ✅ Passed |
| 3 | TC3 | Tidak Valid (KI-2) | 999.999.999 | BCA → Mandiri | Invalid / Ditolak (saldo tidak cukup) | "GAGAL Saldo dompet asal tidak mencukupi untuk nominal transfer beserta biaya admin!" | ✅ Passed |

### Screenshot Bukti Pengujian

**TC1 — Transfer Berhasil (Kelas Valid)**

<img width="1920" height="1080" alt="bva-transfer-tc4" src="https://github.com/user-attachments/assets/4260274b-86a5-42db-b803-4aa4ed405a27" />


**TC2 — Amount Kosong (Kelas Tidak Valid 1)**

<img width="1920" height="1080" alt="bva-transaksi-tc1" src="https://github.com/user-attachments/assets/e1dae35f-4ed5-4e28-9d63-e269f8001ee2" />


**TC3 — Saldo Tidak Mencukupi (Kelas Tidak Valid 2)**

<img width="1920" height="1080" alt="bva-transfer-tc6" src="https://github.com/user-attachments/assets/26f19dcc-1d31-4867-80a9-f753a0a91d8a" />


---

> **Analisis SQA — Modul Transfer:**
> Semua kelas ekuivalen ditangani dengan benar. Sistem memproses transfer saat saldo cukup (TC1), menolak amount kosong/0 sebelum request dikirim (TC2), dan menolak amount yang melebihi saldo dengan pesan error yang informatif (TC3). Seluruh 3 test case Passed.

---

## Modul 3 — Transaksi: Form Catat Aliran Dana

### Kelas Ekuivalen

| No | Kondisi Input | Kelas Valid | Kelas Tidak Valid |
|---|---|---|---|
| 1 | Amount (Nominal) | Amount > 0 (ada nilai yang valid) | (KI-1) Amount kosong / tidak diisi |
| 2 | Portofolio | Portofolio dipilih (salah satu brankas valid) | (KI-2) Portofolio tidak dipilih / kosong |

### Kasus Uji

| No | Test Case | Kelas Ekuivalen | Input Amount | Input Type | Input Portofolio | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|---|---|
| 1 | TC1 | Valid (KV) | 50.000 | `income` (MASUK) | Mandiri | Valid / Transaksi tercatat, saldo bertambah | Saldo Mandiri bertambah Rp 50.000 (Rp 3.000.000 → Rp 3.050.000) | ✅ Passed |
| 2 | TC2 | Tidak Valid (KI-1) | — (kosong) | `income` (MASUK) | Mandiri | Invalid / Ditolak | "Harap isi bidang ini." — form tidak dikirim | ✅ Passed |
| 3 | TC3 | Tidak Valid (KI-2) | 50.000 | `expense` (KELUAR) | — (tidak dipilih) | Invalid / Ditolak | "Pilih item pada daftar." — form tidak dikirim | ✅ Passed |

### Screenshot Bukti Pengujian

**TC1 — Income Berhasil Dicatat (Kelas Valid)**

<img width="1920" height="1080" alt="ep-transaksi-tc1" src="https://github.com/user-attachments/assets/e5763e26-0068-41b7-8515-dd7e73d8fa27" />


**TC2 — Amount Kosong (Kelas Tidak Valid 1)**

<img width="1920" height="1080" alt="bva-transaksi-tc1" src="https://github.com/user-attachments/assets/652d1b0a-7322-42e0-a227-2b81981c7b61" />


**TC3 — Portofolio Tidak Dipilih (Kelas Tidak Valid 2)**

<img width="1920" height="1080" alt="ep-transaksi-tc3" src="https://github.com/user-attachments/assets/ae73ea90-e0d4-479e-ad51-be27e005ca52" />


---

> **Analisis SQA — Modul Transaksi:**
> TC1 (valid) berhasil — saldo Mandiri naik dari Rp 3.000.000 ke Rp 3.050.000 setelah income dicatat. TC2 dan TC3 ditolak oleh validasi browser sebelum request dikirim ke server, membuktikan sistem memiliki layer validasi frontend. Seluruh 3 test case Passed.

---

## Modul 4 — Tabungan: Form Target Impian

### Kelas Ekuivalen

| No | Kondisi Input | Kelas Valid | Kelas Tidak Valid |
|---|---|---|---|
| 1 | Nama Tabungan | 1 ≤ Panjang Karakter ≤ 255 | (KI-1) Nama kosong (0 karakter); (KI-2) Panjang > 255 karakter |
| 2 | Target Amount | Amount > 0 (minimal 1) | Amount = 0 (di bawah minimum) |

### Kasus Uji

| No | Test Case | Kelas Ekuivalen | Input Nama | Input Target | Input Sumber | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|---|---|
| 1 | TC1 | Valid (KV) | `TabunganEP` (10 karakter) | 500.000 | BSI | Valid / Tabungan dibuat | Tabungan "TABUNGANEP" muncul di halaman Target Impian | ✅ Passed |
| 2 | TC2 | Tidak Valid (KI-1) | — (kosong) | 500.000 | Mandiri | Invalid / Ditolak | "Harap isi bidang ini." — form tidak dikirim | ✅ Passed |
| 3 | TC3 | Tidak Valid (KI-2) | 300 karakter (`AAA...`) | 500.000 | BSI | Invalid / Ditolak (nama terlalu panjang) | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |

### Screenshot Bukti Pengujian

**TC1 — Tabungan Berhasil Dibuat (Kelas Valid)**

<img width="1920" height="1080" alt="ep-tabungan-tc1" src="https://github.com/user-attachments/assets/5877cb84-552d-44d5-944a-d772e50212be" />


**TC2 — Nama Kosong (Kelas Tidak Valid 1)**

<img width="1920" height="1080" alt="bva-tabungan-tc1" src="https://github.com/user-attachments/assets/021c7ba4-77fe-4029-94d2-89a70941cf39" />


**TC3 — Nama > 255 Karakter (Kelas Tidak Valid 2)**

<img width="1920" height="1080" alt="ep-tabungan-tc3" src="https://github.com/user-attachments/assets/6250e161-4c9d-4630-9398-54463da78620" />


---

> **Analisis SQA — Modul Tabungan:**
> TC1 berhasil — tabungan "TABUNGANEP" berhasil dibuat dan muncul di daftar Target Impian. TC2 ditolak oleh browser native validation (field required). TC3 berhasil mendeteksi nama 300 karakter dan menampilkan pesan error backend yang jelas: "The name field must not be greater than 255 characters." Seluruh 3 test case Passed.

---

## Ringkasan Hasil EP — Seluruh Sistem

| Modul | Jumlah TC | Kelas Valid | Kelas Tidak Valid | Passed | Failed |
|---|---|---|---|---|---|
| Auth — Form Login | 3 | 1 | 2 | 3 | 0 |
| Transfer — Form Pindah Dana | 3 | 1 | 2 | 3 | 0 |
| Transaksi — Form Catat Aliran Dana | 3 | 1 | 2 | 3 | 0 |
| Tabungan — Form Target Impian | 3 | 1 | 2 | 3 | 0 |
| **TOTAL** | **12** | **4** | **8** | **12** | **0** |

> **Kesimpulan:** Semua 12 test case EP menunjukkan hasil **Passed**. Setiap kelas ekuivalen — baik valid maupun tidak valid — ditangani oleh sistem sesuai dengan ekspektasi. Validasi input bekerja di dua layer: browser native validation (mencegah request dikirim untuk field kosong/format salah) dan backend server validation (422 untuk data yang lolos frontend namun tidak sesuai business rule). Tidak ditemukan defect baru dalam pengujian EP ini.
