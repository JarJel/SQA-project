# BB-03 — Decision Table Testing
## Sistem: SaPoPoe FINANCE (Midnight Finance)
## Teknik: Black Box Testing — Decision Table Testing

---

> **Definisi Teknik:**
> Decision Table Testing adalah pengujian **gabungan dari berbagai kondisi dalam pengambilan keputusan**, yang digunakan pada uji software dalam **verifikasi input yang beragam** tetapi saling menggenapi fungsi form.
>
> — Materi Pertemuan 11, Software Quality, T Informatika UKRI

---

## Modul 1 — Autentikasi: Form Login

### Decision Table

| ID | CONDITION / ACTION | TC1 | TC2 | TC3 |
|---|---|---|---|---|
| C1 | Email terdaftar di sistem | T | T | F |
| C2 | Password sesuai akun | T | F | F |
| A1 | Login berhasil → redirect Dashboard | E | | |
| A2 | Error "Kata sandi yang Anda masukkan salah." | | E | |
| A3 | Error "Alamat email tidak ditemukan. Silakan buat akun terlebih dahulu." | | | E |

> T = True, F = False, E = Execute

### Kasus Uji

| ID | Test Case | Input | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|
| TC1 | Email & password benar | email=`sultan@test.com`, pass=`password123` | Redirect to Dashboard | Dashboard berhasil ditampilkan | ✅ Passed |
| TC2 | Email terdaftar, password salah | email=`sultan@test.com`, pass=`salah123` | Error password | "Kata sandi yang Anda masukkan salah." | ✅ Passed |
| TC3 | Email tidak terdaftar | email=`tidakada@test.com`, pass=`apasaja` | Error email | "Alamat email tidak ditemukan. Silakan buat akun terlebih dahulu." | ✅ Passed |

---

## Modul 2 — Transfer: Form Pindah Dana

### Decision Table

| ID | CONDITION / ACTION | TC1 | TC2 | TC3 |
|---|---|---|---|---|
| C1 | Amount > 0 (nominal valid) | T | F | T |
| C2 | Saldo brankas asal mencukupi | T | — | F |
| A1 | Transfer berhasil | E | | |
| A2 | Error "Harap isi bidang ini." (amount kosong) | | E | |
| A3 | Error "GAGAL Saldo dompet asal tidak mencukupi..." | | | E |

> T = True, F = False, E = Execute, — = Tidak relevan (kondisi C1 sudah gugur)

### Kasus Uji

| ID | Test Case | Input Amount | Saldo Sumber | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| TC1 | Amount valid, saldo cukup | 50.000 | Mandiri (Rp 3.050.000) | Transfer berhasil | "BERHASIL Dana berhasil dipindahkan!" | ✅ Passed |
| TC2 | Amount kosong | 0 / kosong | — | Ditolak sebelum dikirim | "Harap isi bidang ini." | ✅ Passed |
| TC3 | Amount valid, saldo tidak cukup | 999.999.999 | BCA (Rp -994.649.999) | Ditolak karena saldo minus | "GAGAL Saldo dompet asal tidak mencukupi untuk nominal transfer beserta biaya admin!" | ✅ Passed |

---

## Modul 3 — Transaksi: Form Catat Aliran Dana

### Decision Table

| ID | CONDITION / ACTION | TC1 | TC2 | TC3 |
|---|---|---|---|---|
| C1 | Amount > 0 (nominal valid) | T | F | T |
| C2 | Portofolio dipilih | T | — | F |
| A1 | Transaksi berhasil dicatat, saldo diperbarui | E | | |
| A2 | Error "Harap isi bidang ini." (amount kosong) | | E | |
| A3 | Error "Pilih item pada daftar." (portofolio kosong) | | | E |

> T = True, F = False, E = Execute, — = Tidak relevan (kondisi C1 sudah gugur)

### Kasus Uji

| ID | Test Case | Input Amount | Portofolio | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| TC1 | Amount valid, portofolio dipilih | 50.000 | Mandiri | Transaksi tercatat, saldo bertambah | Saldo Mandiri bertambah Rp 50.000 | ✅ Passed |
| TC2 | Amount kosong | — (kosong) | Mandiri | Ditolak sebelum dikirim | "Harap isi bidang ini." | ✅ Passed |
| TC3 | Amount valid, portofolio tidak dipilih | 50.000 | — (tidak dipilih) | Ditolak sebelum dikirim | "Pilih item pada daftar." | ✅ Passed |

---

## Modul 4 — Tabungan: Form Target Impian

### Decision Table

| ID | CONDITION / ACTION | TC1 | TC2 | TC3 | TC4 |
|---|---|---|---|---|---|
| C1 | Nama tabungan diisi (tidak kosong) | T | F | T | T |
| C2 | Panjang nama ≤ 255 karakter | T | — | F | T |
| C3 | Target amount > 0 | T | — | — | F |
| A1 | Tabungan berhasil dibuat | E | | | |
| A2 | Error "Harap isi bidang ini." (nama kosong) | | E | | |
| A3 | Error "The name field must not be greater than 255 characters." | | | E | |
| A4 | Error "target amount minimal 1" | | | | E |

> T = True, F = False, E = Execute, — = Tidak relevan (kondisi sebelumnya sudah gugur)

### Kasus Uji

| ID | Test Case | Input Nama | Input Target | Expected Output | Actual Output | Status |
|---|---|---|---|---|---|---|
| TC1 | Semua input valid | `TabunganDT` (10 karakter) | 500.000 | Tabungan berhasil dibuat | Tabungan muncul di daftar Target Impian | ✅ Passed |
| TC2 | Nama kosong | — (kosong) | 500.000 | Ditolak sebelum dikirim | "Harap isi bidang ini." | ✅ Passed |
| TC3 | Nama > 255 karakter | 300 karakter (`AAA...`) | 500.000 | Ditolak karena nama terlalu panjang | "GAGAL — The name field must not be greater than 255 characters." | ✅ Passed |
| TC4 | Target = 0 | `TabunganDT` | 0 | Ditolak karena target minimal 1 | return 422 "target amount minimal 1" | ✅ Passed |

---

## Ringkasan Hasil Decision Table Testing — Seluruh Sistem

| Modul | Jumlah Kondisi | Jumlah TC | Passed | Failed |
|---|---|---|---|---|
| Auth — Form Login | 2 kondisi | 3 | 3 | 0 |
| Transfer — Form Pindah Dana | 2 kondisi | 3 | 3 | 0 |
| Transaksi — Form Catat Aliran Dana | 2 kondisi | 3 | 3 | 0 |
| Tabungan — Form Target Impian | 3 kondisi | 4 | 4 | 0 |
| **TOTAL** | | **13** | **13** | **0** |

> **Kesimpulan:** Seluruh 13 test case Decision Table menunjukkan hasil **Passed**. Setiap kombinasi kondisi ditangani sistem sesuai ekspektasi — input valid diproses, input tidak valid ditolak dengan pesan error yang sesuai. Tidak ditemukan defect baru dalam pengujian ini.
