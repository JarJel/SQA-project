# Black Box Testing — Equivalence Partitioning (EP)

**Proyek:** Midnight Finance  
**Modul yang Diuji:** Authentication — `POST /api/v1/register`  
**Tanggal Pengujian:** 27 Mei 2026  
**Penguji:** QA Team  
**Metode:** Equivalence Partitioning

---

## 1. Deskripsi Pengujian

Equivalence Partitioning (EP) membagi domain input menjadi kelas-kelas ekuivalen, di mana setiap anggota kelas diharapkan menghasilkan respons yang sama dari sistem. Metode ini meminimalkan jumlah test case yang diperlukan sekaligus memaksimalkan coverage.

**Endpoint yang diuji:** `POST /api/v1/register`  
**Field yang diuji:** `name`, `email`, `password`, `password_confirmation`

---

## 2. Identifikasi Kelas Ekuivalen

### Field: `name`

| Kelas | Tipe | Deskripsi | Contoh |
|-------|------|-----------|--------|
| EP-N1 | ✅ Valid | String non-kosong | `"Test User"` |
| EP-N2 | ❌ Invalid | Kosong / null | `""` |

### Field: `email`

| Kelas | Tipe | Deskripsi | Contoh |
|-------|------|-----------|--------|
| EP-E1 | ✅ Valid | Format email benar, belum terdaftar | `"newuser@test.com"` |
| EP-E2 | ❌ Invalid | Format email salah | `"bukan-email"` |
| EP-E3 | ❌ Invalid | Email sudah terdaftar | `"admin@midnight.com"` |

### Field: `password`

| Kelas | Tipe | Deskripsi | Contoh |
|-------|------|-----------|--------|
| EP-P1 | ✅ Valid | ≥8 karakter, huruf besar, kecil, angka, simbol | `"X9#kQz7@"` |
| EP-P2 | ❌ Invalid | Kurang dari 8 karakter | `"Ab1!"` |
| EP-P3 | ❌ Invalid | Tidak ada huruf besar | `"abcd1234!"` |
| EP-P4 | ❌ Invalid | `password_confirmation` tidak cocok | password ≠ confirmation |
| EP-P5 | ❌ Invalid | Tidak ada simbol | `"AbcdEfgh1"` |

---

## 3. Test Cases & Hasil Pengujian

### TC-EP-01 — Input Valid (Registrasi Berhasil)

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP01 |
| **Kelas EP** | EP-N1, EP-E1, EP-P1 |
| **Input** | `name: "EP Test User"`, `email: "eptest01@test.com"`, `password: "X9#kQz7@mVb2$nP4"`, `password_confirmation: "X9#kQz7@mVb2$nP4"` |
| **Expected** | HTTP 201 — Registrasi berhasil |
| **Actual Output** | HTTP 201 — `{"message":"Registrasi berhasil. Sistem sedang mengirimkan kode verifikasi ke email Anda."}` |
| **Status** | ✅ PASSED |

---

### TC-EP-02 — Name Kosong

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP02 |
| **Kelas EP** | EP-N2 |
| **Input** | `name: ""`, `email: "eptest02@test.com"`, `password: "X9#kQz7@"`, `password_confirmation: "X9#kQz7@"` |
| **Expected** | HTTP 422 — Validasi gagal (name required) |
| **Actual Output** | HTTP 422 — `{"message":"The name field is required.","errors":{"name":["The name field is required."]}}` |
| **Status** | ✅ PASSED |

---

### TC-EP-03 — Format Email Tidak Valid

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP03 |
| **Kelas EP** | EP-E2 |
| **Input** | `name: "Test User"`, `email: "bukan-email"`, `password: "X9#kQz7@"`, `password_confirmation: "X9#kQz7@"` |
| **Expected** | HTTP 422 — Format email tidak valid |
| **Actual Output** | HTTP 422 — `{"message":"The email field must be a valid email address.","errors":{"email":[...]}}` |
| **Status** | ✅ PASSED |

---

### TC-EP-04 — Email Sudah Terdaftar

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP04 |
| **Kelas EP** | EP-E3 |
| **Input** | `name: "Duplicate User"`, `email: "admin@midnight.com"`, `password: "X9#kQz7@"`, `password_confirmation: "X9#kQz7@"` |
| **Expected** | HTTP 422 — Email sudah terdaftar |
| **Actual Output** | HTTP 422 — `{"message":"The email has already been taken.","errors":{"email":["The email has already been taken."]}}` |
| **Status** | ✅ PASSED |

---

### TC-EP-05 — Password Terlalu Pendek

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP05 |
| **Kelas EP** | EP-P2 |
| **Input** | `name: "Test User"`, `email: "eptest05@test.com"`, `password: "Ab1!"`, `password_confirmation: "Ab1!"` |
| **Expected** | HTTP 422 — Password minimal 8 karakter |
| **Actual Output** | HTTP 422 — `{"message":"The password field must be at least 8 characters.","errors":{"password":[...]}}` |
| **Status** | ✅ PASSED |

---

### TC-EP-06 — Password Tidak Ada Huruf Besar

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP06 |
| **Kelas EP** | EP-P3 |
| **Input** | `name: "Test User"`, `email: "eptest06@test.com"`, `password: "abcd1234!"`, `password_confirmation: "abcd1234!"` |
| **Expected** | HTTP 422 — Password harus mengandung huruf besar |
| **Actual Output** | HTTP 422 — `{"message":"The password field must contain at least one uppercase and one lowercase letter.","errors":{...}}` |
| **Status** | ✅ PASSED |

---

### TC-EP-07 — Password Confirmation Tidak Cocok

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP07 |
| **Kelas EP** | EP-P4 |
| **Input** | `name: "Test User"`, `email: "eptest07@test.com"`, `password: "X9#kQz7@"`, `password_confirmation: "DifferentPass1!"` |
| **Expected** | HTTP 422 — Konfirmasi password tidak cocok |
| **Actual Output** | HTTP 422 — `{"message":"The password field confirmation does not match.","errors":{"password":[...]}}` |
| **Status** | ✅ PASSED |

---

### TC-EP-08 — Password Tidak Ada Simbol

| Atribut | Detail |
|---------|--------|
| **ID Test** | BB01-EP08 |
| **Kelas EP** | EP-P5 |
| **Input** | `name: "Test User"`, `email: "eptest08@test.com"`, `password: "AbcdEfgh1"`, `password_confirmation: "AbcdEfgh1"` |
| **Expected** | HTTP 422 — Password harus mengandung simbol |
| **Actual Output** | HTTP 422 — `{"message":"The password field must contain at least one symbol.","errors":{"password":[...]}}` |
| **Status** | ✅ PASSED |

---

## 4. Ringkasan Hasil EP

| ID Test | Kelas EP | Input | Expected HTTP | Actual HTTP | Status |
|---------|----------|-------|---------------|-------------|--------|
| EP-01 | Valid | Semua field valid | 201 | 201 | ✅ PASSED |
| EP-02 | Invalid | Name kosong | 422 | 422 | ✅ PASSED |
| EP-03 | Invalid | Email format salah | 422 | 422 | ✅ PASSED |
| EP-04 | Invalid | Email sudah ada | 422 | 422 | ✅ PASSED |
| EP-05 | Invalid | Password < 8 char | 422 | 422 | ✅ PASSED |
| EP-06 | Invalid | No uppercase | 422 | 422 | ✅ PASSED |
| EP-07 | Invalid | Confirmation mismatch | 422 | 422 | ✅ PASSED |
| EP-08 | Invalid | No symbol | 422 | 422 | ✅ PASSED |

**Total:** 8 test case | ✅ 8 Passed | ❌ 0 Failed

---

## 5. Bukti Pengujian (Test Evidence)

File JSON hasil pengujian tersimpan di direktori `screenshots/black-box/`:

| File | Keterangan |
|------|------------|
| `BB01-EP01-valid.json` | HTTP 201 — Registrasi berhasil |
| `BB01-EP02-empty-name.json` | HTTP 422 — Name kosong |
| `BB01-EP03-invalid-email.json` | HTTP 422 — Format email salah |
| `BB01-EP04-email-exists.json` | HTTP 422 — Email terdaftar |
| `BB01-EP05-short-password.json` | HTTP 422 — Password pendek |
| `BB01-EP06-no-uppercase.json` | HTTP 422 — Tidak ada huruf besar |
| `BB01-EP07-password-mismatch.json` | HTTP 422 — Konfirmasi tidak cocok |
| `BB01-EP08-no-symbol.json` | HTTP 422 — Tidak ada simbol |
