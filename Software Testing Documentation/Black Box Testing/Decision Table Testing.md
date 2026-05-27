# 📋 Decision Table Testing — Midnight Finance

**Mata Kuliah:** Software Quality Assurance  
**Model Pengujian:** Black Box Testing — Decision Table Testing  
**Tim:** REMACode  
**Modul Target:** Fitur Login (`POST /api/login`)  

---

## 📖 Definisi

**Decision Table Testing** adalah teknik Black Box Testing yang digunakan untuk menguji perilaku sistem berdasarkan **kombinasi kondisi input** yang berbeda-beda. Teknik ini efektif untuk fitur yang memiliki banyak kondisi logika yang saling mempengaruhi output, sehingga semua kombinasi kondisi penting dapat terwakili dalam test case (Suprihadi, 2025).

---

## 🎯 Modul yang Diuji

**Endpoint:** `POST /api/login`  
**Logika Backend:**

```
1. Email valid & terdaftar?
2. Password cocok?
3. Akun sudah verifikasi OTP?
```

---

## 📊 Decision Table Login

| | R1 | R2 | R3 | R4 | R5 |
|:---|:---:|:---:|:---:|:---:|:---:|
| **KONDISI** | | | | | |
| Email terdaftar di database | ❌ | ✅ | ✅ | ✅ | ✅ |
| Password cocok | — | ❌ | ✅ | ✅ | ✅ |
| Akun sudah verifikasi OTP | — | — | ❌ | ✅ | — |
| **AKSI** | | | | | |
| Return 404 (email tidak ditemukan) | ✅ | ❌ | ❌ | ❌ | ❌ |
| Return 401 (password salah) | ❌ | ✅ | ❌ | ❌ | ❌ |
| Return 403 (perlu verifikasi OTP) | ❌ | ❌ | ✅ | ❌ | ❌ |
| Return 200 (login berhasil + token) | ❌ | ❌ | ❌ | ✅ | ❌ |
| Return 422 (format email salah) | ❌ | ❌ | ❌ | ❌ | ✅ |

> R5 = input email dengan format tidak valid (validasi Laravel sebelum query DB)

---

## 📝 Test Case Decision Table

| No | Test Case | Rule | Input Email | Input Password | Expected Output | Actual Output | Status |
|:--:|:---|:---:|:---:|:---:|:---|:---:|:---:|
| TC-DT-01 | Email tidak terdaftar | R1 | `ghost@mail.com` | `Test@1234` | HTTP 404 — "Alamat email tidak ditemukan" | HTTP 404 | ✅ Valid |
| TC-DT-02 | Email benar, password salah | R2 | `user@midnight.com` | `WrongPass!` | HTTP 401 — "Kata sandi yang Anda masukkan salah" | HTTP 401 | ✅ Valid |
| TC-DT-03 | Email benar, password benar, belum OTP | R3 | `unverified@midnight.com` | `Test@1234` | HTTP 403 — "Akun belum diverifikasi", `need_otp: true` | HTTP 403 | ✅ Valid |
| TC-DT-04 | Login lengkap berhasil | R4 | `user@midnight.com` | `Test@1234` | HTTP 200 — `access_token` + data user | HTTP 200 | ✅ Valid |
| TC-DT-05 | Format email tidak valid | R5 | `userATmail` | `Test@1234` | HTTP 422 — "The email field must be a valid email address" | HTTP 422 | ✅ Valid |

---

## ✅ Hasil Pengujian

| Kategori | Jumlah |
|:---|:---:|
| Total Test Case | 5 |
| Test Case Valid (Passed) | 5 |
| Test Case Failed | 0 |
| **Persentase Keberhasilan** | **100%** |

> **Kesimpulan:** Semua kombinasi kondisi login berhasil ditangani sistem dengan benar. Decision Table Testing mengkonfirmasi bahwa logika autentikasi Midnight Finance sudah sesuai spesifikasi untuk semua skenario yang relevan.

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Black Box Testing*. T Informatika UKRI.
