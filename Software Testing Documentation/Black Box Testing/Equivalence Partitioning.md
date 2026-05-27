# 🔢 Equivalence Partitioning — Midnight Finance

**Mata Kuliah:** Software Quality Assurance  
**Model Pengujian:** Black Box Testing — Equivalence Partitioning  
**Tim:** REMACode  
**Modul Target:** Form Registrasi User (`POST /api/register`)  

---

## 📖 Definisi

**Black Box Testing** adalah teknik pengujian perangkat lunak yang melibatkan pengujian sistem tanpa mengetahui desain internalnya. Teknik pengujian Black Box berfokus pada informasi dari perangkat lunak, menghasilkan test case dengan cara mempartisi masukan dan keluaran dari sebuah program dengan cara mencakup pengujian yang menyeluruh (Suprihadi, 2025).

**Equivalence Partitioning** digunakan untuk mencari seluruh kesalahan atau kehilangan dalam fungsi. Kesalahan dapat berupa tampilan struktur data atau akses menuju database serta performa. Keadaan masukan bisa berupa range, harga khusus, suatu kumpulan atau boolean. Jika input merupakan beberapa keadaan tersebut, maka kasus ujinya adalah **satu benar dan dua tidak benar** (Suprihadi, 2025).

---

## 🎯 Modul yang Diuji

**Endpoint:** `POST /api/register`  
**Validasi Backend (Laravel):**

```php
$request->validate([
    'name'     => 'required|string|max:255',
    'email'    => 'required|email',
    'password' => ['required', 'confirmed',
                   Password::min(8)->letters()->mixedCase()->numbers()->symbols()],
]);
```

---

## 📊 Tabel Equivalence Class

| No | Nama Kolom | Tipe Data | Batasan Data |
|:--:|:---|:---|:---|
| 1 | `name` | String | Wajib diisi, panjang 1–255 karakter |
| 2 | `email` | String (Email) | Wajib diisi, format email valid, unik di database |
| 3 | `password` | String | Min 8 karakter, huruf besar+kecil, angka, simbol |
| 4 | `password_confirmation` | String | Harus sama persis dengan `password` |

---

## 📋 Identifikasi Partisi Ekuivalensi

### Field: `name`

| Partisi ID | Kelas | Contoh Nilai | Valid? |
|:---:|:---|:---|:---:|
| EP-N-V1 | Nama normal (huruf + spasi) | `Budi Santoso` | ✅ Valid |
| EP-N-V2 | Nama satu kata | `Budi` | ✅ Valid |
| EP-N-I1 | Nama kosong (empty string) | `` | ❌ Invalid |
| EP-N-I2 | Nama melebihi 255 karakter | `Budi...` (256 kar) | ❌ Invalid |
| EP-N-I3 | Null / tidak dikirim | *(field tidak ada)* | ❌ Invalid |

### Field: `email`

| Partisi ID | Kelas | Contoh Nilai | Valid? |
|:---:|:---|:---|:---:|
| EP-E-V1 | Email valid standar | `budi@gmail.com` | ✅ Valid |
| EP-E-V2 | Email valid dengan subdomain | `budi@mail.midnight.com` | ✅ Valid |
| EP-E-I1 | Tanpa karakter `@` | `budigmail.com` | ❌ Invalid |
| EP-E-I2 | Tanpa domain | `budi@` | ❌ Invalid |
| EP-E-I3 | Email sudah terdaftar (duplikat aktif) | `existing@midnight.com` | ❌ Invalid |
| EP-E-I4 | Field kosong | `` | ❌ Invalid |

### Field: `password`

| Partisi ID | Kelas | Contoh Nilai | Valid? |
|:---:|:---|:---|:---:|
| EP-P-V1 | Password memenuhi semua syarat | `Midnight@2025` | ✅ Valid |
| EP-P-I1 | Kurang dari 8 karakter | `Ab1!` | ❌ Invalid |
| EP-P-I2 | Tanpa huruf besar | `midnight@2025` | ❌ Invalid |
| EP-P-I3 | Tanpa angka | `Midnight@abc` | ❌ Invalid |
| EP-P-I4 | Tanpa simbol | `Midnight2025` | ❌ Invalid |
| EP-P-I5 | Tidak cocok dengan konfirmasi | `Midnight@2025` vs `Midnight@2026` | ❌ Invalid |

---

## 📝 Tabel Test Case Equivalence Partitioning

| No | Test Case | Input `name` | Input `email` | Input `password` | Input `conf.` | Expected Output | Actual Output | Status |
|:--:|:---|:---:|:---:|:---:|:---:|:---|:---:|:---:|
| TC-EP-01 | Semua input valid | `Budi Santoso` | `budi@gmail.com` | `Midnight@2025` | `Midnight@2025` | HTTP 201 — "Registrasi berhasil. Kode OTP dikirim." | HTTP 201 | ✅ Valid |
| TC-EP-02 | Nama kosong | *(kosong)* | `budi@gmail.com` | `Midnight@2025` | `Midnight@2025` | HTTP 422 — "The name field is required." | HTTP 422 | ✅ Valid |
| TC-EP-03 | Format email salah (tanpa @) | `Budi Santoso` | `budigmail.com` | `Midnight@2025` | `Midnight@2025` | HTTP 422 — "The email field must be a valid email address." | HTTP 422 | ✅ Valid |
| TC-EP-04 | Email sudah terdaftar & aktif | `Budi Santoso` | `existing@midnight.com` | `Midnight@2025` | `Midnight@2025` | HTTP 400 — "Alamat email sudah terdaftar." | HTTP 400 | ✅ Valid |
| TC-EP-05 | Password < 8 karakter | `Budi Santoso` | `budi2@gmail.com` | `Ab1!` | `Ab1!` | HTTP 422 — Password terlalu pendek | HTTP 422 | ✅ Valid |
| TC-EP-06 | Password tanpa huruf besar | `Budi Santoso` | `budi3@gmail.com` | `midnight@2025` | `midnight@2025` | HTTP 422 — Password harus ada huruf besar | HTTP 422 | ✅ Valid |
| TC-EP-07 | Password tidak cocok dengan konfirmasi | `Budi Santoso` | `budi4@gmail.com` | `Midnight@2025` | `Midnight@2026` | HTTP 422 — "The password field confirmation does not match." | HTTP 422 | ✅ Valid |
| TC-EP-08 | Password tanpa simbol | `Budi Santoso` | `budi5@gmail.com` | `Midnight2025` | `Midnight2025` | HTTP 422 — Password harus ada simbol | HTTP 422 | ✅ Valid |

---

## ✅ Hasil Pengujian

| Kategori | Jumlah |
|:---|:---:|
| Total Test Case | 8 |
| Test Case Valid (Passed) | 8 |
| Test Case Failed | 0 |
| **Persentase Keberhasilan** | **100%** |

> **Kesimpulan:** Seluruh test case pada pengujian Equivalence Partitioning modul Registrasi dinyatakan **valid**. Sistem berhasil menangani seluruh partisi input (valid maupun invalid) dengan respons yang tepat sesuai spesifikasi validasi backend Laravel.

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Black Box Testing*. T Informatika UKRI.
- Destiningrum, M., & Adrian, Q. J. (2017). Sistem Informasi Penjadwalan Dokter Berbassis Web dengan Menggunakan Framework Codeigniter. *Jurnal TEKNOINFO*, 11(2), 30.
