#  Code Walkthrough

**Model White Box Testing \#2** — *Static Testing* **Modul Target:** Autentikasi & Login Flow (Laravel Sanctum) **Tim:** REMACode

---

##  1\. Definisi

**Code Walkthrough** adalah teknik review kode secara **formal maupun informal** yang dilakukan bersama-sama antara developer dan tim terkait untuk **memahami logika kode**, mengidentifikasi potensi error, dan meningkatkan kualitas keseluruhan program (Suprihadi, 2025). Berbeda dengan Desk Checking yang dilakukan individual, Code Walkthrough bersifat **kolaboratif** dan menggunakan diskusi tim sebagai mekanisme deteksi error.

*"Teknik review kode secara formal atau informal yang dilakukan bersama-sama antara developer dan tim terkait untuk memahami logika kode, kemudian mengidentifikasi potensi error, dan meningkatkan kualitas keseluruhan program."* — (Suprihadi, 2025\)

### Perbedaan dengan Desk Checking

| Aspek | Desk Checking | Code Walkthrough |
| :---- | :---- | :---- |
| Pelaku | Individu (developer sendiri) | Tim (developer \+ reviewer) |
| Sifat | Privat | Kolaboratif |
| Fokus | Trace variabel | Diskusi logika & arsitektur |
| Output | Tabel trace | Notulen review \+ action items |

---

##  2\. Tujuan Pengujian

| No | Tujuan |
| :---- | :---- |
| 1 | Memastikan semua anggota tim memahami logika kode yang ditulis |
| 2 | Menemukan bug, code smell, dan security issue secara dini |
| 3 | Knowledge sharing antar developer (mengurangi *bus factor*) |
| 4 | Memvalidasi adherence terhadap coding standard tim |
| 5 | Mengidentifikasi *edge case* yang belum di-handle |

---

##  3\. Peserta Review (Tim REMACode)

| Peran | Nama | Tanggung Jawab |
| :---- | :---- | :---- |
| **Author** | Muhammad Dzaki Awaludin | Penulis kode yang akan di-review |
| **Moderator** | Muhammad Fajar Munandar | Memimpin sesi review, mencatat issue |
| **Reviewer 1** | Mochamad Fikri Ghifari | Fokus arsitektur & desain |
| **Reviewer 2** | Raka Zilva Inggia | Fokus security & validasi logika |

---

##  4\. Source Code yang Direview

**File:** `app/Http/Controllers/Api/AuthController.php` **Method:** `login()` — autentikasi user via email & password menggunakan Sanctum.

 **TODO:** Ganti dengan controller asli dari `midnight-finance-backend` saat finalisasi.

public function login(Request $request)

{

    $credentials \= $request-\>validate(\[

        'email'    \=\> 'required|email',

        'password' \=\> 'required|string|min:8',

    \]);

    $user \= User::where('email', $credentials\['email'\])-\>first();

    if (\!$user) {

        return response()-\>json(\['message' \=\> 'User not found'\], 404);

    }

    if (\!Hash::check($credentials\['password'\], $user-\>password)) {

        return response()-\>json(\['message' \=\> 'Invalid password'\], 401);

    }

    $token \= $user-\>createToken('auth\_token')-\>plainTextToken;

    return response()-\>json(\[

        'access\_token' \=\> $token,

        'token\_type'   \=\> 'Bearer',

        'user'         \=\> $user,

    \], 200);

}

---

##  5\. Proses Walkthrough

### 5.1 Penjelasan Author (Step-by-Step)

flowchart TD

    A(\[Mulai: POST /api/login\]) \--\> B\[Validasi email & password\]

    B \--\> C{User ditemukan?}

    C \--\>|Tidak| D\[Return 404: User not found\]

    C \--\>|Ya| E{Password match?}

    E \--\>|Tidak| F\[Return 401: Invalid password\]

    E \--\>|Ya| G\[Generate Sanctum token\]

    G \--\> H\[Return 200: token \+ user data\]

    style D fill:\#7f1d1d,stroke:\#ef4444,color:\#fff

    style F fill:\#7f1d1d,stroke:\#ef4444,color:\#fff

    style H fill:\#14532d,stroke:\#22c55e,color:\#fff

**Narasi author:**

*"Method `login` menerima request berisi email dan password. Setelah validasi format, kami query user berdasarkan email. Jika tidak ditemukan return 404, jika password salah return 401\. Jika valid, generate token Sanctum dan return ke client."*

### 5.2 Diskusi Tim — Issue yang Diidentifikasi

####  Issue \#1: User Enumeration Vulnerability

**Pengamat:** Raka Zilva Inggia (Security)

**Masalah:**

if (\!$user) {

    return response()-\>json(\['message' \=\> 'User not found'\], 404);

}

if (\!Hash::check(...)) {

    return response()-\>json(\['message' \=\> 'Invalid password'\], 401);

}

**Dampak:** Attacker dapat **enumerate email valid** dengan membandingkan response. Email yang ada return 401, yang tidak ada return 404\. Ini violation terhadap **OWASP A07:2021 — Identification & Authentication Failures**.

**Rekomendasi:** Gunakan pesan generic yang sama untuk kedua kasus:

return response()-\>json(\['message' \=\> 'Invalid credentials'\], 401);

####  Issue \#2: Tidak Ada Rate Limiting

**Pengamat:** Raka Zilva Inggia (Security)

**Masalah:** Endpoint `/api/login` tidak memiliki throttle middleware, rentan terhadap **brute force attack**.

**Rekomendasi:** Tambahkan middleware throttle pada route:

Route::post('/login', \[AuthController::class, 'login'\])

    \-\>middleware('throttle:5,1'); // 5 attempt per menit

####  Issue \#3: Token Tidak Punya Expiration

**Pengamat:** Mochamad Fikri Ghifari (Architecture)

**Masalah:** Token Sanctum di-generate tanpa `expiresAt`, artinya token valid selamanya sampai user logout manual.

**Rekomendasi:**

$token \= $user-\>createToken('auth\_token', \['\*'\], now()-\>addDays(7))

    \-\>plainTextToken;

####  Issue \#4: Mengembalikan Seluruh Data User

**Pengamat:** Mochamad Fikri Ghifari (Architecture)

**Masalah:** `return ... 'user' => $user` mengembalikan **semua field** termasuk timestamp internal dan kemungkinan field sensitif.

**Rekomendasi:** Gunakan API Resource:

'user' \=\> new UserResource($user),

####  Issue \#5: Inconsistent HTTP Status Code

**Pengamat:** Mochamad Fikri Ghifari (Architecture)

**Masalah:** Status `404 User not found` untuk login endpoint kurang tepat secara semantik. Standar RESTful menggunakan `401 Unauthorized` untuk semua kegagalan autentikasi.

---

##  6\. Ringkasan Temuan

| ID | Severity | Issue | Kategori | Reviewer |
| :---- | :---- | :---- | :---- | :---- |
| `CW-001` |  High | User enumeration vulnerability | Security | Raka |
| `CW-002` |  High | Tidak ada rate limiting | Security | Raka |
| `CW-003` |  Medium | Token tanpa expiration | Architecture | Fikri |
| `CW-004` |  Medium | Expose seluruh data user | Architecture | Fikri |
| `CW-005` |  Low | Inconsistent HTTP status code | Code Quality | Fikri |

**Total Issue:** 5 (2 High, 2 Medium, 1 Low)

---

##  7\. Rekomendasi Perbaikan Kode

public function login(Request $request)

{

    $credentials \= $request-\>validate(\[

        'email'    \=\> 'required|email',

        'password' \=\> 'required|string|min:8',

    \]);

    $user \= User::where('email', $credentials\['email'\])-\>first();

    // Generic error message — prevent user enumeration

    if (\!$user || \!Hash::check($credentials\['password'\], $user-\>password)) {

        return response()-\>json(\[

            'message' \=\> 'Invalid credentials',

        \], 401);

    }

    // Token dengan expiration 7 hari

    $token \= $user-\>createToken(

        'auth\_token',

        \['\*'\],

        now()-\>addDays(7)

    )-\>plainTextToken;

    return response()-\>json(\[

        'access\_token' \=\> $token,

        'token\_type'   \=\> 'Bearer',

        'expires\_at'   \=\> now()-\>addDays(7)-\>toIso8601String(),

        'user'         \=\> new UserResource($user),

    \], 200);

}

**Tambahan di `routes/api.php`:**

Route::post('/login', \[AuthController::class, 'login'\])

    \-\>middleware('throttle:5,1')

    \-\>name('auth.login');

---

##  8\. Checklist Walkthrough

Checklist standar yang digunakan Tim REMACode dalam sesi walkthrough:

| Kategori | Item | Status |
| :---- | :---- | :---- |
| **Functionality** | Logika sesuai requirement? | ✅ |
| **Functionality** | Edge case ter-handle? | ⚠️ Partial |
| **Security** | Input divalidasi? | ✅ |
| **Security** | Tidak ada SQL injection risk? | ✅ |
| **Security** | Tidak expose informasi sensitif? | ❌ |
| **Security** | Ada rate limiting? | ❌ |
| **Performance** | Tidak ada N+1 query? | ✅ |
| **Performance** | Index database sesuai? | ✅ |
| **Maintainability** | Coding style konsisten? | ✅ |
| **Maintainability** | Method tidak terlalu panjang? | ✅ |
| **Maintainability** | Penamaan variabel jelas? | ✅ |
| **Testing** | Sudah ada unit test? | ❌ |

**Skor Walkthrough:** 9/12 (75%) — perlu perbaikan di Security dan Testing.

---

##  9\. Kelebihan & Kekurangan

###  Kelebihan

- Mendapatkan **multiple perspective** sekaligus  
- Knowledge transfer antar developer berjalan natural  
- Membangun *coding standard* tim secara konsisten  
- Lebih efektif menemukan bug *security* yang sulit terlihat sendirian  
- Mengurangi *bus factor* (tidak ada single point of knowledge)

###  Kekurangan

- **Membutuhkan waktu** semua peserta secara sinkron  
- Bisa menjadi tidak produktif jika tidak ada moderator  
- Risiko *bikeshedding* — diskusi panjang untuk hal trivial  
- Bergantung pada skill reviewer (junior reviewer kurang efektif)  
- Tidak menggantikan testing otomatis

---

##  10\. Tools Pendukung

| Tool | Kegunaan |
| :---- | :---- |
| **GitHub Pull Request** | Async code review dengan comment thread |
| **GitLab Merge Request** | Alternatif PR dengan inline discussion |
| **VS Code Live Share** | Real-time collaborative review |
| **Zoom / Google Meet** | Sesi walkthrough sinkron |
| **Notion / Confluence** | Dokumentasi notulen & action items |

---

##  11\. Template Notulen Walkthrough

\# Walkthrough Session — \[Tanggal\]

\*\*File:\*\* \[path/to/file.php\]

\*\*Author:\*\* \[nama\]

\*\*Moderator:\*\* \[nama\]

\*\*Reviewers:\*\* \[nama1, nama2\]

\*\*Durasi:\*\* \[X menit\]

\#\# Ringkasan Kode

\[1-2 paragraf penjelasan singkat\]

\#\# Issue Ditemukan

| ID | Severity | Deskripsi | Owner | Due Date |

|---|---|---|---|---|

\#\# Keputusan Tim

\- \[ \] \[Action item 1\]

\- \[ \] \[Action item 2\]

\#\# Sign-off

\- \[ \] Author

\- \[ \] Reviewer 1

\- \[ \] Reviewer 2

---

##  Referensi

1. Suprihadi, D. (2025). *Materi Software Quality Pertemuan 10*. Universitas Kristen Indonesia.  
2. Fagan, M. E. (1976). *Design and code inspections to reduce errors in program development*. IBM Systems Journal.  
3. OWASP Foundation. (2021). *OWASP Top 10:2021 — A07: Identification and Authentication Failures*. [https://owasp.org/Top10/](https://owasp.org/Top10/)  
4. Pressman, R. S., & Maxim, B. R. (2020). *Software Engineering: A Practitioner's Approach* (9th ed.). McGraw-Hill.

---

[ Desk Checking](http://./Desk_Checking.md) · [Kembali ke README](http://./README.md) · [Lanjut ke Formal Inspection ➡](http://./Formal_Inspection.md)

**Tim REMACode** — Midnight Finance SQA Documentation  
