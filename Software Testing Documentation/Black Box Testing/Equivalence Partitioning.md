# 🔢 Equivalence Partitioning

> **Model Black Box Testing #1** — *Input-Based Testing*
> **Modul Target:** Form Registrasi User (Email, Password, Name)
> **Tim:** REMACode

---

## 📖 1. Definisi

**Equivalence Partitioning (EP)** digunakan untuk **mencari seluruh kesalahan atau kehilangan dalam fungsi**. Kesalahan dapat berupa tampilan struktur data, akses menuju database, atau performa. Keadaan masukan bisa berupa **range, harga khusus, suatu kumpulan, atau boolean**. Jika input merupakan beberapa keadaan tersebut, maka kasus ujinya adalah **satu benar dan dua tidak benar** (Suprihadi, 2025).

> *"Equivalence Partioning digunakan untuk mencari seluruh kesalahan atau kehilangan dalam fungsi. Kesalahan dapat tampilan struktur data atau akses menuju database serta performa. Keadaan masukan bisa berupa range, harga khusus, suatu kumpulan atau boolean."* — (Suprihadi, 2025)

### Prinsip Dasar

Membagi domain input menjadi **partisi/kelas ekuivalensi** dimana semua nilai dalam satu partisi diasumsikan menghasilkan **perilaku sistem yang sama**. Cukup pilih **1 representative value** dari tiap partisi untuk testing.

---

## 🎯 2. Tujuan Pengujian

| No | Tujuan |
|---|---|
| 1 | Mengurangi jumlah test case tanpa kehilangan coverage |
| 2 | Memastikan validasi input bekerja untuk seluruh range data |
| 3 | Menemukan input yang tidak ter-handle dengan benar |
| 4 | Mendeteksi missing validation atau improper error messages |
| 5 | Memberikan struktur sistematis untuk test design |

---

## 💻 3. Modul yang Diuji

**Endpoint:** `POST /api/register`
**Form Fields:** Email, Password, Name, Phone Number

> ⚠️ **TODO:** Pastikan field-field ini sesuai dengan implementasi asli di `midnight-finance-backend`.

### Spesifikasi Validasi

| Field | Tipe | Constraint |
|---|---|---|
| `email` | string | Format email valid, unique, max 255 char |
| `password` | string | Min 8 char, harus ada huruf besar + angka + simbol |
| `name` | string | Min 3 char, max 100 char, hanya huruf & spasi |
| `phone_number` | string | Format Indonesia (+62 atau 08), 10-13 digit |

---

## 🔍 4. Identifikasi Partisi Ekuivalensi

### 4.1 Field: `email`

| Partisi ID | Kelas | Range/Contoh | Valid? |
|---|---|---|---|
| `EP-EMAIL-V1` | Email valid lengkap | `user@example.com` | ✅ Valid |
| `EP-EMAIL-V2` | Email dengan subdomain | `user@mail.example.com` | ✅ Valid |
| `EP-EMAIL-V3` | Email dengan angka | `user123@example.com` | ✅ Valid |
| `EP-EMAIL-I1` | Email tanpa `@` | `userexample.com` | ❌ Invalid |
| `EP-EMAIL-I2` | Email tanpa domain | `user@` | ❌ Invalid |
| `EP-EMAIL-I3` | Email kosong | `""` | ❌ Invalid |
| `EP-EMAIL-I4` | Email sudah terdaftar | `existing@test.com` | ❌ Invalid (duplicate) |
| `EP-EMAIL-I5` | Email > 255 karakter | `aaa...@test.com` (256 char) | ❌ Invalid |

### 4.2 Field: `password`

| Partisi ID | Kelas | Contoh | Valid? |
|---|---|---|---|
| `EP-PASS-V1` | Password kuat (lengkap) | `Pass@1234` | ✅ Valid |
| `EP-PASS-I1` | Hanya huruf kecil | `password` | ❌ Invalid |
| `EP-PASS-I2` | Tanpa angka | `Password@` | ❌ Invalid |
| `EP-PASS-I3` | Tanpa simbol | `Password1` | ❌ Invalid |
| `EP-PASS-I4` | Tanpa huruf besar | `password@1` | ❌ Invalid |
| `EP-PASS-I5` | Kurang dari 8 karakter | `Pa@1` | ❌ Invalid |
| `EP-PASS-I6` | Password kosong | `""` | ❌ Invalid |

### 4.3 Field: `name`

| Partisi ID | Kelas | Contoh | Valid? |
|---|---|---|---|
| `EP-NAME-V1` | Nama valid | `Muhammad Dzaki` | ✅ Valid |
| `EP-NAME-I1` | Mengandung angka | `Dzaki 123` | ❌ Invalid |
| `EP-NAME-I2` | Mengandung simbol | `Dzaki@!` | ❌ Invalid |
| `EP-NAME-I3` | Kurang dari 3 karakter | `Dz` | ❌ Invalid |
| `EP-NAME-I4` | Lebih dari 100 karakter | `A...A` (101 char) | ❌ Invalid |
| `EP-NAME-I5` | Nama kosong | `""` | ❌ Invalid |

### 4.4 Field: `phone_number`

| Partisi ID | Kelas | Contoh | Valid? |
|---|---|---|---|
| `EP-PHONE-V1` | Format `+62` | `+6281234567890` | ✅ Valid |
| `EP-PHONE-V2` | Format `08` | `081234567890` | ✅ Valid |
| `EP-PHONE-I1` | Format internasional non-Indonesia | `+19876543210` | ❌ Invalid |
| `EP-PHONE-I2` | Mengandung huruf | `08123abc7890` | ❌ Invalid |
| `EP-PHONE-I3` | Kurang dari 10 digit | `0812345` | ❌ Invalid |
| `EP-PHONE-I4` | Lebih dari 13 digit | `081234567890123` | ❌ Invalid |
| `EP-PHONE-I5` | Nomor kosong | `""` | ❌ Invalid |

---

## 🧪 5. Test Case Design

### 5.1 Representative Test Cases

Sesuai prinsip EP: **pilih 1 representative dari tiap partisi**.

| TC ID | Email | Password | Name | Phone | Expected | Status |
|---|---|---|---|---|---|---|
| `EP-TC-01` | `user@test.com` | `Pass@1234` | `Muhammad Dzaki` | `081234567890` | ✅ Sukses register | — |
| `EP-TC-02` | `userexample.com` | `Pass@1234` | `Muhammad Dzaki` | `081234567890` | ❌ Error: email invalid | — |
| `EP-TC-03` | `user@test.com` | `password` | `Muhammad Dzaki` | `081234567890` | ❌ Error: password harus mengandung simbol/angka/huruf besar | — |
| `EP-TC-04` | `user@test.com` | `Pa@1` | `Muhammad Dzaki` | `081234567890` | ❌ Error: password min 8 karakter | — |
| `EP-TC-05` | `user@test.com` | `Pass@1234` | `Dz` | `081234567890` | ❌ Error: nama min 3 karakter | — |
| `EP-TC-06` | `user@test.com` | `Pass@1234` | `Dzaki123` | `081234567890` | ❌ Error: nama hanya huruf | — |
| `EP-TC-07` | `user@test.com` | `Pass@1234` | `Muhammad Dzaki` | `0812345` | ❌ Error: phone min 10 digit | — |
| `EP-TC-08` | `user@test.com` | `Pass@1234` | `Muhammad Dzaki` | `+19876543210` | ❌ Error: format phone tidak Indonesia | — |
| `EP-TC-09` | `existing@test.com` | `Pass@1234` | `Muhammad Dzaki` | `081234567890` | ❌ Error: email sudah terdaftar | — |
| `EP-TC-10` | `""` | `""` | `""` | `""` | ❌ Error: semua field wajib diisi | — |

---

## 📸 6. Screenshot yang Diperlukan

> **📸 SCREENSHOT NEEDED #1:** **Form Registrasi Kosong**
> Buka halaman `/register` di Midnight Finance, screenshot tampilan form dalam kondisi default (belum diisi).
> *File suggested name:* `screenshot/EP-form-registrasi-default.png`

> **📸 SCREENSHOT NEEDED #2:** **Sukses Registrasi (TC-01)**
> Isi form dengan data valid (`user@test.com`, `Pass@1234`, `Muhammad Dzaki`, `081234567890`), klik submit, screenshot tampilan setelah berhasil registrasi.
> *File suggested name:* `screenshot/EP-tc01-success.png`

> **📸 SCREENSHOT NEEDED #3:** **Error Email Invalid (TC-02)**
> Isi email dengan `userexample.com` (tanpa @), screenshot pesan error yang muncul.
> *File suggested name:* `screenshot/EP-tc02-email-invalid.png`

> **📸 SCREENSHOT NEEDED #4:** **Error Password Lemah (TC-03)**
> Isi password dengan `password` (tanpa simbol/angka/huruf besar), screenshot pesan error.
> *File suggested name:* `screenshot/EP-tc03-weak-password.png`

> **📸 SCREENSHOT NEEDED #5:** **Error Phone Invalid (TC-08)**
> Isi phone dengan `+19876543210`, screenshot pesan error.
> *File suggested name:* `screenshot/EP-tc08-phone-invalid.png`

**Cara menyertakan screenshot di file ini:**
```markdown
![EP-TC-01 Success](./screenshot/EP-tc01-success.png)
```

---

## 🚀 7. Implementasi Pengujian

### 7.1 Manual Testing (Postman/Insomnia)

```http
POST /api/register HTTP/1.1
Host: midnight-finance.local
Content-Type: application/json

{
    "email": "user@test.com",
    "password": "Pass@1234",
    "name": "Muhammad Dzaki",
    "phone_number": "081234567890"
}
```

**Expected Response (TC-01):**
```json
{
    "status": "success",
    "message": "Registration successful",
    "data": {
        "user": {
            "id": 1,
            "email": "user@test.com",
            "name": "Muhammad Dzaki"
        },
        "access_token": "..."
    }
}
```

**Expected Response (TC-02 — Email Invalid):**
```json
{
    "status": "error",
    "message": "The given data was invalid.",
    "errors": {
        "email": ["The email must be a valid email address."]
    }
}
```

### 7.2 Automated Testing (PHPUnit Feature Test)

```php
<?php

namespace Tests\Feature\Auth;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class RegisterEquivalencePartitioningTest extends TestCase
{
    use RefreshDatabase;

    /** @test EP-TC-01: Valid input — all partitions valid */
    public function it_registers_user_with_valid_data(): void
    {
        $response = $this->postJson('/api/register', [
            'email'        => 'user@test.com',
            'password'     => 'Pass@1234',
            'name'         => 'Muhammad Dzaki',
            'phone_number' => '081234567890',
        ]);

        $response->assertStatus(201)
                 ->assertJsonPath('status', 'success');

        $this->assertDatabaseHas('users', ['email' => 'user@test.com']);
    }

    /** @test EP-TC-02: Invalid email partition */
    public function it_rejects_invalid_email_format(): void
    {
        $response = $this->postJson('/api/register', [
            'email'        => 'userexample.com',  // missing @
            'password'     => 'Pass@1234',
            'name'         => 'Muhammad Dzaki',
            'phone_number' => '081234567890',
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['email']);
    }

    /** @test EP-TC-03: Weak password partition */
    public function it_rejects_weak_password(): void
    {
        $response = $this->postJson('/api/register', [
            'email'        => 'user@test.com',
            'password'     => 'password',  // no uppercase, no number, no symbol
            'name'         => 'Muhammad Dzaki',
            'phone_number' => '081234567890',
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['password']);
    }

    /** @test EP-TC-09: Duplicate email partition */
    public function it_rejects_duplicate_email(): void
    {
        User::factory()->create(['email' => 'existing@test.com']);

        $response = $this->postJson('/api/register', [
            'email'        => 'existing@test.com',
            'password'     => 'Pass@1234',
            'name'         => 'Muhammad Dzaki',
            'phone_number' => '081234567890',
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['email']);
    }
}
```

---

## 📊 8. Hasil Eksekusi

| TC ID | Partisi yang Diuji | Input | Expected | Actual | Status |
|---|---|---|---|---|---|
| `EP-TC-01` | All valid | Valid data | 201 Created | ⏳ Pending | — |
| `EP-TC-02` | EP-EMAIL-I1 | Email tanpa @ | 422 Invalid | ⏳ Pending | — |
| `EP-TC-03` | EP-PASS-I1 | Password lemah | 422 Invalid | ⏳ Pending | — |
| `EP-TC-04` | EP-PASS-I5 | Password < 8 char | 422 Invalid | ⏳ Pending | — |
| `EP-TC-05` | EP-NAME-I3 | Nama < 3 char | 422 Invalid | ⏳ Pending | — |
| `EP-TC-06` | EP-NAME-I1 | Nama dengan angka | 422 Invalid | ⏳ Pending | — |
| `EP-TC-07` | EP-PHONE-I3 | Phone < 10 digit | 422 Invalid | ⏳ Pending | — |
| `EP-TC-08` | EP-PHONE-I1 | Format non-ID | 422 Invalid | ⏳ Pending | — |
| `EP-TC-09` | EP-EMAIL-I4 | Email duplikat | 422 Invalid | ⏳ Pending | — |
| `EP-TC-10` | All empty | Empty form | 422 Invalid | ⏳ Pending | — |

> **Catatan:** Update kolom **Actual** dan **Status** setelah eksekusi manual atau automated test.

---

## 🐛 9. Temuan & Analisis

> **Catatan:** Section ini diisi setelah eksekusi test. Berikut template & contoh prediksi temuan berdasarkan pola umum Laravel:

| ID | Severity | Deskripsi (Predicted) | Rekomendasi |
|---|---|---|---|
| `EP-001` | 🟡 Medium | Validasi nama dengan regex `[A-Za-z\s]` mungkin tidak mengakomodasi nama dengan tanda hubung (`-`) atau apostrof (`'`) — misal "O'Brien" atau "Mary-Jane" | Update regex menjadi `[A-Za-z\s'-]` |
| `EP-002` | 🟡 Medium | Validasi phone Indonesia mungkin terlalu strict — tidak terima `62812...` (tanpa `+`) | Tambah pattern alternatif |
| `EP-003` | 🟢 Low | Pesan error default Laravel berbahasa Inggris | Translate ke Bahasa Indonesia di `lang/id/validation.php` |

---

## ⚖️ 10. Kelebihan & Kekurangan

### ✅ Kelebihan
- **Mengurangi jumlah test case** secara signifikan tanpa kehilangan coverage
- **Sistematis & terukur** — partisi terdefinisi jelas
- **Mudah dipahami** oleh QA non-developer
- Cocok dikombinasikan dengan **Boundary Value Analysis**
- Output bisa langsung jadi **test data spreadsheet**

### ❌ Kekurangan
- **Asumsi homogenitas** — semua nilai dalam partisi diasumsikan behave sama, padahal tidak selalu
- Tidak menangkap bug **boundary** (gunakan BVA)
- Tidak menangkap bug **kombinasi field** (gunakan Decision Table)
- Partisi yang **kurang lengkap** menyebabkan gap coverage
- Sulit untuk input non-trivial (mis. file upload, image)

---

## 🛠️ 11. Tools Pendukung

| Tool | Kegunaan |
|---|---|
| **Postman / Insomnia** | Manual API testing |
| **PHPUnit** | Automated feature test |
| **Laravel Form Request** | Centralized validation rules |
| **Google Sheets / Excel** | Manage test data partition |
| **Faker** | Generate sample data untuk tiap partisi |

---

## 📚 Referensi

1. Suprihadi, D. (2025). *Materi Software Quality Pertemuan 11*. Universitas Kristen Indonesia.
2. Myers, G. J., Sandler, C., & Badgett, T. (2011). *The Art of Software Testing* (3rd ed.). Wiley.
3. Beizer, B. (1995). *Black-Box Testing: Techniques for Functional Testing of Software and Systems*. Wiley.
4. ISTQB. (2023). *Certified Tester Foundation Level Syllabus v4.0*.

---

<div align="center">

[⬅ Kembali ke README](./README.md) · [Lanjut ke Boundary Value Analysis ➡](./Boundary_Value_Analysis.md)

**Tim REMACode** — Midnight Finance SQA Documentation

</div>
