# 🎭 Behaviour Testing (BDD)

> **Model Black Box Testing #7** — *Logic-Based Testing*
> **Modul Target:** User Journey — Registrasi → Login → Buat Transaksi
> **Tim:** REMACode

---

## 📖 1. Definisi

**Behaviour Testing** (juga dikenal sebagai **Behavior-Driven Development/BDD**) adalah pendekatan pengujian perangkat lunak yang **berfokus pada perilaku yang diharapkan dari suatu program dari sudut pandang pengguna** (Suprihadi, 2025). Teknik ini menggunakan berbagai diagram UML untuk menggambarkan interaksi, alur aktivitas, dan perilaku sistem secara keseluruhan.

> *"Behaviour Testing (juga dikenal sebagai Behavior-Driven Development (BDD)) adalah pendekatan pengujian perangkat lunak yang berfokus pada perilaku yang diharapkan dari suatu program dari sudut pandang pengguna."* — (Suprihadi, 2025)

### Diagram UML yang Digunakan

| Diagram | Fungsi | Tools |
|---|---|---|
| **Sequence Diagram** | Interaksi antar objek berdasarkan urutan waktu | Mermaid `sequenceDiagram` |
| **Activity Diagram** | Alur aktivitas dan keputusan dalam proses | Mermaid `flowchart` |
| **Statechart/State Machine** | Kelakuan sistem secara keseluruhan via state | Mermaid `stateDiagram-v2` |
| **Collaboration Diagram** | Organisasi antar objek dalam interaksi | Mermaid `graph` (approx.) |

---

## 🎯 2. Tujuan Pengujian

| No | Tujuan |
|---|---|
| 1 | Memvalidasi **user journey** end-to-end dari perspektif pengguna |
| 2 | Memastikan **urutan interaksi** sistem sesuai dengan spesifikasi |
| 3 | Mendeteksi **state transition** yang tidak valid |
| 4 | Memverifikasi semua **skenario alternatif** (happy path & sad path) |
| 5 | Memberikan **dokumentasi hidup** yang dipahami semua stakeholder |

---

## 💻 3. Modul yang Diuji

**User Journey:** Registrasi → Verifikasi → Login → Buat Transaksi → Lihat Summary

**Aktor:** User (pengguna Midnight Finance)

> ⚠️ **TODO:** Sesuaikan flow jika ada step tambahan (mis. verifikasi email, onboarding setup akun pertama).

---

## 🔷 4. Sequence Diagram

Sequence diagram menggambarkan **interaksi antar objek dalam berkomunikasi saling mengirim message disusun berdasarkan urutan waktu** (Suprihadi, 2025).

### 4.1 Sequence Diagram — Flow Login

```mermaid
sequenceDiagram
    actor User
    participant App as Frontend (React)
    participant API as Backend (Laravel)
    participant DB as Database

    User->>App: 1. Buka halaman login
    App-->>User: 1.1 Tampilkan form login

    User->>App: 2. Isi email & password, klik Login
    App->>API: 2.1 POST /api/login {email, password}
    API->>DB: 2.2 SELECT * FROM users WHERE email = ?
    DB-->>API: 2.3 Return user record

    alt Kredensial valid
        API->>DB: 2.4 INSERT INTO personal_access_tokens
        DB-->>API: 2.5 Token created
        API-->>App: 2.6 200 OK {access_token, user}
        App-->>User: 2.7 Redirect ke /dashboard
    else Kredensial tidak valid
        API-->>App: 2.8 401 {message: "Invalid credentials"}
        App-->>User: 2.9 Tampilkan pesan error
    end
```

### 4.2 Sequence Diagram — Flow Buat Transaksi

```mermaid
sequenceDiagram
    actor User
    participant App as Frontend (React)
    participant API as Backend (Laravel)
    participant DB as Database

    User->>App: 1. Klik tombol "Tambah Transaksi"
    App-->>User: 1.1 Tampilkan form transaksi

    User->>App: 2. Isi form (account, category, type, amount, date)
    App->>API: 2.1 POST /api/transactions {Bearer Token}
    API->>API: 2.2 Validasi input & auth

    alt Input valid & saldo cukup
        API->>DB: 2.3 BEGIN TRANSACTION
        API->>DB: 2.4 INSERT INTO transactions
        API->>DB: 2.5 UPDATE accounts SET balance
        API->>DB: 2.6 COMMIT
        DB-->>API: 2.7 Success
        API-->>App: 2.8 201 Created {transaction}
        App-->>User: 2.9 Tampilkan notifikasi sukses
    else Input tidak valid
        API-->>App: 2.10 422 {errors}
        App-->>User: 2.11 Tampilkan pesan validasi
    else Saldo tidak cukup
        API-->>App: 2.12 422 {message: "Saldo tidak mencukupi"}
        App-->>User: 2.13 Tampilkan pesan saldo kurang
    end
```

---

## 🔶 5. Activity Diagram

Activity diagram digunakan untuk **menggambarkan urutan kerja masing-masing objek dalam menyelesaikan permasalahan tertentu sesuai dengan fungsi kerja masing-masing objek** (Suprihadi, 2025).

### 5.1 Activity Diagram — Registrasi User

```mermaid
flowchart TD
    START([Mulai]) --> A[User membuka halaman Register]
    A --> B[Isi form: name, email, password, phone]
    B --> C{Validasi\nFormat Input}

    C -->|Invalid| D[Tampilkan pesan error]
    D --> B

    C -->|Valid| E{Email sudah\nterdaftar?}
    E -->|Ya| F[Tampilkan error:\nEmail sudah digunakan]
    F --> B

    E -->|Tidak| G[Simpan user ke database]
    G --> H[Generate token verifikasi]
    H --> I[Kirim email verifikasi]
    I --> J{User klik\nlink verifikasi?}

    J -->|Tidak, expired| K[Tampilkan halaman\nresend verification]
    K --> I

    J -->|Ya| L[Update status user:\nis_verified = true]
    L --> M[Redirect ke halaman Login]
    M --> END([Selesai])

    style START fill:#1e3a8a,stroke:#3b82f6,color:#fff
    style END fill:#14532d,stroke:#22c55e,color:#fff
    style D fill:#7f1d1d,stroke:#ef4444,color:#fff
    style F fill:#7f1d1d,stroke:#ef4444,color:#fff
```

### 5.2 Activity Diagram — Buat Transaksi

```mermaid
flowchart TD
    START([Mulai]) --> AUTH{User\nterautentikasi?}
    AUTH -->|Tidak| LOGIN[Redirect ke Login]
    LOGIN --> END_AUTH([End])

    AUTH -->|Ya| A[Buka form tambah transaksi]
    A --> B[Pilih akun & kategori]
    B --> C[Isi type, amount, tanggal, deskripsi]
    C --> D{Validasi\nInput}

    D -->|Invalid| E[Tampilkan error validasi]
    E --> C

    D -->|Valid| F{Type =\nExpense?}
    F -->|Ya| G{Saldo cukup?}
    G -->|Tidak| H[Error: Saldo tidak mencukupi]
    H --> C

    G -->|Ya| I[Simpan transaksi]
    F -->|Tidak - Income| I

    I --> J[Update saldo akun]
    J --> K{Budget aktif\nuntuk kategori?}

    K -->|Ya| L[Update budget spent]
    L --> M{Budget > 80%?}
    M -->|Ya| N[Kirim notifikasi warning]
    N --> O[Return sukses]
    M -->|Tidak| O
    K -->|Tidak| O

    O --> END([Selesai])

    style START fill:#1e3a8a,stroke:#3b82f6,color:#fff
    style END fill:#14532d,stroke:#22c55e,color:#fff
    style H fill:#7f1d1d,stroke:#ef4444,color:#fff
    style N fill:#854d0e,stroke:#eab308,color:#fff
```

---

## 🔵 6. Statechart Diagram

Statechart UML adalah diagram yang **menggambarkan kelakuan sistem secara keseluruhan**. Karena menggambarkan kelakuan sistem secara keseluruhan, pembangkitan data uji berdasar statechart (state machine) dianggap menguji keseluruhan sistem (Suprihadi, 2025).

### 6.1 Statechart — User Account States

```mermaid
stateDiagram-v2
    [*] --> Unregistered

    Unregistered --> PendingVerification : Register berhasil
    PendingVerification --> PendingVerification : Resend verification
    PendingVerification --> Active : Klik link verifikasi
    PendingVerification --> Unregistered : Token expired (> 24 jam)

    Active --> LoggedIn : Login berhasil
    LoggedIn --> Active : Logout
    LoggedIn --> Suspended : Admin suspend
    LoggedIn --> LoggedIn : Refresh token

    Active --> Suspended : Admin suspend
    Suspended --> Active : Admin unsuspend

    Active --> Deleted : User hapus akun
    Suspended --> Deleted : Admin delete
    Deleted --> [*]
```

### 6.2 Statechart — Transaksi States

```mermaid
stateDiagram-v2
    [*] --> Draft : User mulai isi form

    Draft --> Validating : User submit form
    Validating --> Draft : Validasi gagal (kembali ke form)
    Validating --> Processing : Validasi sukses

    Processing --> Completed : DB transaction commit
    Processing --> Failed : DB transaction rollback

    Completed --> Completed : (final state)
    Failed --> Draft : User retry

    Completed --> [*]
    Failed --> [*]
```

### 6.3 Statechart — Budget States

```mermaid
stateDiagram-v2
    [*] --> Inactive : Budget dibuat (sebelum periode)

    Inactive --> Active : Tanggal mulai tercapai
    Active --> Warning : Pemakaian >= 80%
    Warning --> Overbudget : Pemakaian >= 100%
    Warning --> Active : Transaksi dihapus (turun < 80%)
    Overbudget --> Warning : Transaksi dihapus (turun < 100%)

    Active --> Expired : Tanggal akhir terlewat
    Warning --> Expired : Tanggal akhir terlewat
    Overbudget --> Expired : Tanggal akhir terlewat

    Expired --> [*]
```

---

## 🟣 7. Collaboration Diagram

Collaboration diagram digunakan untuk **menggambarkan susunan organisasi antar objek dalam berinteraksi dan bekerjasama untuk melaksanakan fungsi yang didefinisikan di dalam sistem software** (Suprihadi, 2025).

### 7.1 Collaboration — Proses Buat Transaksi

```mermaid
graph TB
    subgraph Client
        USER[👤 User]
        REACT[⚛️ React Frontend]
    end

    subgraph Server
        CTRL[TransactionController]
        REQ[StoreTransactionRequest]
        SVC[TransactionService]
        BSVC[BudgetService]
        NOTIF[NotificationService]
    end

    subgraph Database
        TXN_TBL[(transactions)]
        ACC_TBL[(accounts)]
        BUDGET_TBL[(budgets)]
    end

    USER -->|"1: Submit form"| REACT
    REACT -->|"2: POST /api/transactions"| CTRL
    CTRL -->|"3: validate()"| REQ
    REQ -->|"4: validated data"| CTRL
    CTRL -->|"5: store(data)"| SVC
    SVC -->|"6: INSERT"| TXN_TBL
    SVC -->|"7: UPDATE balance"| ACC_TBL
    SVC -->|"8: checkBudget()"| BSVC
    BSVC -->|"9: SELECT spent"| BUDGET_TBL
    BSVC -->|"10: UPDATE spent"| BUDGET_TBL
    BSVC -->|"11: notify()"| NOTIF
    NOTIF -->|"12: push notification"| REACT
    SVC -->|"13: return transaction"| CTRL
    CTRL -->|"14: 201 Created"| REACT
    REACT -->|"15: Tampilkan sukses"| USER

    style USER fill:#1e3a8a,stroke:#3b82f6,color:#fff
    style REACT fill:#0f766e,stroke:#14b8a6,color:#fff
    style CTRL fill:#7c2d12,stroke:#ea580c,color:#fff
    style SVC fill:#7c2d12,stroke:#ea580c,color:#fff
    style BSVC fill:#7c2d12,stroke:#ea580c,color:#fff
    style NOTIF fill:#854d0e,stroke:#eab308,color:#fff
```

---

## 🧪 8. BDD Scenarios (Gherkin Format)

```gherkin
Feature: User Authentication & Transaction Management
  Sebagai pengguna Midnight Finance
  Saya ingin bisa login dan membuat transaksi
  Agar saya bisa mencatat keuangan saya

  Background:
    Given saya memiliki akun yang terdaftar dan terverifikasi
    And saya memiliki akun keuangan dengan saldo Rp 1.000.000

  Scenario: Login berhasil sebagai user biasa
    When saya membuka halaman login
    And saya mengisi email "user@test.com" dan password "Pass@1234"
    And saya menekan tombol "Login"
    Then saya diarahkan ke halaman dashboard
    And saya melihat nama saya di navbar

  Scenario: Login gagal dengan password salah
    When saya membuka halaman login
    And saya mengisi email "user@test.com" dan password "wrongpassword"
    And saya menekan tombol "Login"
    Then saya melihat pesan "Invalid credentials"
    And saya tetap di halaman login

  Scenario: Membuat transaksi expense berhasil
    Given saya sudah login
    When saya membuka form tambah transaksi
    And saya memilih tipe "Expense"
    And saya mengisi nominal "Rp 200.000"
    And saya memilih kategori "Makanan"
    And saya menekan tombol "Simpan"
    Then transaksi berhasil disimpan
    And saldo akun saya berkurang menjadi "Rp 800.000"

  Scenario: Transaksi expense gagal karena saldo tidak cukup
    Given saya sudah login
    When saya membuka form tambah transaksi
    And saya memilih tipe "Expense"
    And saya mengisi nominal "Rp 2.000.000"
    And saya menekan tombol "Simpan"
    Then saya melihat pesan "Saldo tidak mencukupi"
    And saldo akun saya tidak berubah

  Scenario: Notifikasi warning budget muncul saat mendekati limit
    Given saya sudah login
    And saya memiliki budget "Makanan" dengan limit "Rp 500.000"
    And saya sudah memakai "Rp 380.000" (76%)
    When saya menambahkan transaksi expense "Makanan" senilai "Rp 50.000"
    Then total pemakaian menjadi 86%
    And saya menerima notifikasi "Anggaran Makanan mendekati batas (86%)"
```

---

## 📸 9. Screenshot yang Diperlukan

> **📸 SCREENSHOT NEEDED #1:** **Halaman Login**
> Screenshot halaman `/login` Midnight Finance.
> *File suggested name:* `screenshot/BHV-login-page.png`

> **📸 SCREENSHOT NEEDED #2:** **Dashboard setelah Login Berhasil**
> Screenshot halaman dashboard yang muncul setelah login sukses.
> *File suggested name:* `screenshot/BHV-dashboard-after-login.png`

> **📸 SCREENSHOT NEEDED #3:** **Form Tambah Transaksi**
> Screenshot form input transaksi.
> *File suggested name:* `screenshot/BHV-form-transaksi.png`

> **📸 SCREENSHOT NEEDED #4:** **Notifikasi Sukses Setelah Transaksi**
> Screenshot notifikasi/toast sukses setelah transaksi berhasil disimpan.
> *File suggested name:* `screenshot/BHV-transaksi-success-notif.png`

> **📸 SCREENSHOT NEEDED #5:** **Notifikasi Budget Warning**
> Screenshot notifikasi warning budget mendekati limit (jika ada push notification / badge).
> *File suggested name:* `screenshot/BHV-budget-warning-notif.png`

---

## 🚀 10. Implementasi Pengujian (PHPUnit)

```php
<?php

namespace Tests\Feature\BDD;

use App\Models\Budget;
use App\Models\User;
use App\Models\Account;
use App\Models\Category;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserJourneyBehaviourTest extends TestCase
{
    use RefreshDatabase;

    /** @test Scenario: Login berhasil */
    public function user_can_login_with_valid_credentials(): void
    {
        $user = User::factory()->create([
            'email'     => 'user@test.com',
            'password'  => bcrypt('Pass@1234'),
            'is_active' => true,
        ]);

        $this->postJson('/api/login', [
            'email'    => 'user@test.com',
            'password' => 'Pass@1234',
        ])
        ->assertStatus(200)
        ->assertJsonStructure(['access_token', 'user']);
    }

    /** @test Scenario: Login gagal password salah */
    public function user_cannot_login_with_wrong_password(): void
    {
        User::factory()->create(['email' => 'user@test.com']);

        $this->postJson('/api/login', [
            'email'    => 'user@test.com',
            'password' => 'wrongpassword',
        ])
        ->assertStatus(401)
        ->assertJson(['message' => 'Invalid credentials']);
    }

    /** @test Scenario: Transaksi expense berhasil + saldo berkurang */
    public function user_can_create_expense_and_balance_decreases(): void
    {
        $user    = User::factory()->create();
        $account = Account::factory()->create([
            'user_id' => $user->id,
            'balance' => 1_000_000,
        ]);
        $category = Category::factory()->create(['user_id' => $user->id]);

        $this->actingAs($user)->postJson('/api/transactions', [
            'account_id'       => $account->id,
            'category_id'      => $category->id,
            'type'             => 'expense',
            'amount'           => 200_000,
            'transaction_date' => now()->toDateString(),
        ])->assertStatus(201);

        $account->refresh();
        $this->assertEquals(800_000, $account->balance);
    }

    /** @test Scenario: Transaksi gagal saldo tidak cukup */
    public function user_cannot_create_expense_exceeding_balance(): void
    {
        $user    = User::factory()->create();
        $account = Account::factory()->create([
            'user_id' => $user->id,
            'balance' => 1_000_000,
        ]);
        $category = Category::factory()->create(['user_id' => $user->id]);

        $this->actingAs($user)->postJson('/api/transactions', [
            'account_id'       => $account->id,
            'category_id'      => $category->id,
            'type'             => 'expense',
            'amount'           => 2_000_000,
            'transaction_date' => now()->toDateString(),
        ])->assertStatus(422);

        // Balance tidak berubah
        $account->refresh();
        $this->assertEquals(1_000_000, $account->balance);
    }

    /** @test Scenario: State machine — budget inactive → active → warning */
    public function budget_transitions_through_states_correctly(): void
    {
        $user     = User::factory()->create();
        $category = Category::factory()->create(['user_id' => $user->id]);
        $account  = Account::factory()->create(['user_id' => $user->id, 'balance' => 10_000_000]);

        $budget = Budget::factory()->create([
            'user_id'      => $user->id,
            'limit_amount' => 1_000_000,
            'start_date'   => now()->startOfMonth(),
            'end_date'     => now()->endOfMonth(),
        ]);

        // State: Active (0%)
        $status = $this->actingAs($user)
            ->getJson("/api/budgets/{$budget->id}/status")
            ->json('data.status');
        $this->assertEquals('safe', $status);

        // Transition → Warning (85%)
        $this->actingAs($user)->postJson('/api/transactions', [
            'account_id'       => $account->id,
            'category_id'      => $category->id,
            'type'             => 'expense',
            'amount'           => 850_000,
            'transaction_date' => now()->toDateString(),
        ]);

        $status = $this->actingAs($user)
            ->getJson("/api/budgets/{$budget->id}/status")
            ->json('data.status');
        $this->assertEquals('warning', $status);
    }
}
```

---

## 📊 11. Hasil Eksekusi

| Scenario | Gherkin | State Transition | Expected | Status |
|---|---|---|---|---|
| Login sukses | ✅ Covered | Active → LoggedIn | 200 + token | ⏳ Pending |
| Login gagal | ✅ Covered | Active (no change) | 401 | ⏳ Pending |
| Transaksi expense sukses | ✅ Covered | Draft → Completed | 201 | ⏳ Pending |
| Transaksi saldo kurang | ✅ Covered | Draft → Failed | 422 | ⏳ Pending |
| Budget safe → warning | ✅ Covered | Active → Warning | status=warning | ⏳ Pending |
| Budget warning → overbudget | ✅ Covered | Warning → Overbudget | status=overbudget | ⏳ Pending |

---

## 🐛 12. Temuan & Analisis

| ID | Severity | Deskripsi (Predicted) | Rekomendasi |
|---|---|---|---|
| `BHV-001` | 🔴 High | State `PendingVerification` tidak di-enforce — user bisa login tanpa verifikasi email | Tambah check `is_verified` di login logic |
| `BHV-002` | 🟡 Medium | Tidak ada state `Deleted` di implementasi — akun hanya di-soft delete | Tambah `SoftDeletes` + state guard |
| `BHV-003` | 🟡 Medium | Notifikasi budget warning tidak real-time (butuh refresh manual) | Implementasi Laravel Broadcasting + Pusher |
| `BHV-004` | 🟢 Low | Tidak ada state history/audit log untuk perubahan budget status | Tambah `budget_status_logs` table |

---

## ⚖️ 13. Kelebihan & Kekurangan

### ✅ Kelebihan
- **Dokumentasi hidup** yang dipahami developer & non-developer
- Menguji sistem **end-to-end** dari perspektif pengguna
- **State machine** mendeteksi transisi ilegal
- BDD scenarios bisa jadi **acceptance criteria** yang clear
- Diagram sequence membantu **debug** alur komunikasi

### ❌ Kekurangan
- **Setup effort tinggi** untuk 4 jenis diagram
- Diagram bisa **outdated** jika tidak di-maintain
- Tidak menguji **internal logic** secara detail (gunakan White Box)
- Gherkin syntax memerlukan **Behat/Cucumber** untuk full automation
- Collaboration diagram sulit di-Mermaid secara exact

---

## 🛠️ 14. Tools Pendukung

| Tool | Kegunaan |
|---|---|
| **Behat** | BDD framework untuk PHP (Gherkin execution) |
| **Cucumber** | BDD framework multi-language |
| **Cypress** | E2E test yang mirror Gherkin scenario |
| **Mermaid** | Semua 4 jenis diagram dalam `.md` |
| **draw.io** | Collaboration diagram yang lebih visual |
| **Laravel Dusk** | Browser automation untuk E2E |

```bash
# Install Behat untuk BDD
composer require --dev behat/behat behat/mink-extension

# Run Gherkin scenarios
./vendor/bin/behat features/authentication.feature
./vendor/bin/behat features/transaction.feature
```

---

## 📚 Referensi

1. Suprihadi, D. (2025). *Materi Software Quality Pertemuan 11*. Universitas Kristen Indonesia.
2. North, D. (2006). *Introducing BDD*. https://dannorth.net/introducing-bdd/
3. Fowler, M. (2004). *UML Distilled* (3rd ed.). Addison-Wesley.
4. Myers, G. J., Sandler, C., & Badgett, T. (2011). *The Art of Software Testing* (3rd ed.). Wiley.

---

<div align="center">

[⬅ Comparison Testing](./Comparison_Testing.md) · [Kembali ke README](./README.md) · [Lanjut ke Performance Testing ➡](./Performance_Testing.md)

**Tim REMACode** — Midnight Finance SQA Documentation

</div>
