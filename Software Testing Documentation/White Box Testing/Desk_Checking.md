#  Desk Checking

**Model White Box Testing \#1** — *Static Testing* **Modul Target:** Kalkulasi Saldo Transaksi (Income & Expense) **Tim:** REMACode

---

##  1\. Definisi

**Desk Checking** adalah teknik pengujian *static* paling fundamental dalam White Box Testing, di mana developer atau penguji **menelusuri (trace) kode secara manual baris demi baris** tanpa menjalankan program. Pemeriksaan ini berfokus pada **logika kode dan perubahan nilai variabel** saat program dieksekusi secara hipotetis di atas kertas (Pressman & Maxim, 2020).

*"Desk Checking adalah salah satu pengujian bagi para pembuat software yang telah mempelajari bahasa pemrograman dengan sangat baik, karena pemeriksaan berfokus pada logika dan nilai variabel pada input dan output yang diperlukan oleh aplikasi."* — (Suprihadi, 2025\)

---

##  2\. Tujuan Pengujian

| No | Tujuan |
| :---- | :---- |
| 1 | Memverifikasi logika kode sebelum dieksekusi |
| 2 | Melacak perubahan nilai variabel pada setiap langkah eksekusi |
| 3 | Menemukan kesalahan logika dan *off-by-one error* |
| 4 | Memvalidasi kesesuaian output dengan ekspektasi |
| 5 | Mengurangi biaya debugging di tahap lanjut |

---

##  3\. Source Code yang Diuji

**File:** `app/Http/Controllers/Api/TransactionController.php` **Method:** `store()` — fungsi menambah transaksi income/expense yang otomatis update saldo akun.

 **TODO:** Snippet di bawah adalah representasi pola umum Laravel. Ganti dengan controller asli dari `midnight-finance-backend` saat finalisasi.

public function store(Request $request)

{

    $validated \= $request-\>validate(\[

        'account\_id'   \=\> 'required|exists:accounts,id',

        'category\_id'  \=\> 'required|exists:categories,id',

        'type'         \=\> 'required|in:income,expense',

        'amount'       \=\> 'required|numeric|min:0.01',

        'description'  \=\> 'nullable|string|max:255',

    \]);

    $account \= Account::findOrFail($validated\['account\_id'\]);

    $currentBalance \= $account-\>balance;

    $amount \= $validated\['amount'\];

    // Core logic: hitung saldo akhir berdasarkan tipe transaksi

    if ($validated\['type'\] \=== 'income') {

        $newBalance \= $currentBalance \+ $amount;

    } else {

        $newBalance \= $currentBalance \- $amount;

    }

    $account-\>update(\['balance' \=\> $newBalance\]);

    return Transaction::create(\[

        ...$validated,

        'user\_id' \=\> auth()-\>id(),

    \]);

}

---

##  4\. Proses Desk Checking

### 4.1 Identifikasi Variabel Kunci

| Variabel | Tipe | Sumber | Peran |
| :---- | :---- | :---- | :---- |
| `$currentBalance` | decimal | DB (`accounts.balance`) | Saldo awal akun |
| `$amount` | decimal | Request input | Nominal transaksi |
| `$validated['type']` | string | Request input | `income` atau `expense` |
| `$newBalance` | decimal | Computed | Saldo akhir |

### 4.2 Trace Eksekusi — Skenario Income

**Input:** `currentBalance = 1.000.000`, `amount = 500.000`, `type = income` **Ekspektasi:** `newBalance = 1.500.000`

| Step | Baris Kode | $currentBalance | $amount | $newBalance | Kondisi | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | `$currentBalance = $account->balance` | 1.000.000 | — | — | — | ✓ |
| 2 | `$amount = $validated['amount']` | 1.000.000 | 500.000 | — | — | ✓ |
| 3 | `if (type === 'income')` | 1.000.000 | 500.000 | — | TRUE | ✓ |
| 4 | `$newBalance = $currentBalance + $amount` | 1.000.000 | 500.000 | 1.500.000 | — | ✓ |
| 5 | `$account->update(...)` | 1.000.000 | 500.000 | 1.500.000 | — | ✅ **PASSED** |

### 4.3 Trace Eksekusi — Skenario Expense

**Input:** `currentBalance = 1.000.000`, `amount = 300.000`, `type = expense` **Ekspektasi:** `newBalance = 700.000`

| Step | Baris Kode | $currentBalance | $amount | $newBalance | Kondisi | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | `$currentBalance = $account->balance` | 1.000.000 | — | — | — | ✓ |
| 2 | `$amount = $validated['amount']` | 1.000.000 | 300.000 | — | — | ✓ |
| 3 | `if (type === 'income')` | 1.000.000 | 300.000 | — | FALSE | ✓ |
| 4 | `$newBalance = $currentBalance - $amount` | 1.000.000 | 300.000 | 700.000 | — | ✓ |
| 5 | `$account->update(...)` | 1.000.000 | 300.000 | 700.000 | — | ✅ **PASSED** |

### 4.4 Trace Eksekusi — Skenario Edge Case (Saldo Tidak Cukup)

**Input:** `currentBalance = 100.000`, `amount = 500.000`, `type = expense` **Ekspektasi:** Harus ada validasi (saldo negatif tidak boleh terjadi)

| Step | Baris Kode | $currentBalance | $amount | $newBalance | Kondisi | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | `$currentBalance = $account->balance` | 100.000 | — | — | — | ✓ |
| 2 | `$amount = $validated['amount']` | 100.000 | 500.000 | — | — | ✓ |
| 3 | `if (type === 'income')` | 100.000 | 500.000 | — | FALSE | ✓ |
| 4 | `$newBalance = $currentBalance - $amount` | 100.000 | 500.000 | **\-400.000** | — | ❌ **FAILED** |
| 5 | `$account->update(...)` | 100.000 | 500.000 | \-400.000 | — | ⚠️ **BUG** |

---

##  5\. Temuan Bug

| ID | Severity | Deskripsi | Lokasi | Rekomendasi |
| :---- | :---- | :---- | :---- | :---- |
| `DC-001` |  High | Saldo dapat bernilai negatif saat expense melebihi balance | Line: `if/else` block | Tambahkan validasi `$amount <= $currentBalance` untuk type expense |
| `DC-002` |  Medium | Tidak ada database transaction (rollback) | Seluruh method | Wrap dalam `DB::transaction()` agar atomic |
| `DC-003` |  Low | Tidak ada logging perubahan saldo | Setelah `$account->update()` | Tambahkan audit log untuk traceability |

---

##  6\. Rekomendasi Perbaikan Kode

public function store(Request $request)

{

    $validated \= $request-\>validate(\[

        'account\_id'   \=\> 'required|exists:accounts,id',

        'category\_id'  \=\> 'required|exists:categories,id',

        'type'         \=\> 'required|in:income,expense',

        'amount'       \=\> 'required|numeric|min:0.01',

        'description'  \=\> 'nullable|string|max:255',

    \]);

    return DB::transaction(function () use ($validated) {

        $account \= Account::lockForUpdate()-\>findOrFail($validated\['account\_id'\]);

        $amount \= $validated\['amount'\];

        // Validasi saldo cukup untuk expense

        if ($validated\['type'\] \=== 'expense' && $account-\>balance \< $amount) {

            throw new InsufficientBalanceException('Saldo tidak mencukupi');

        }

        $newBalance \= $validated\['type'\] \=== 'income'

            ? $account-\>balance \+ $amount

            : $account-\>balance \- $amount;

        $account-\>update(\['balance' \=\> $newBalance\]);

        return Transaction::create(\[

            ...$validated,

            'user\_id' \=\> auth()-\>id(),

        \]);

    });

}

---

##  7\. Ringkasan Hasil Pengujian

| Skenario | Input | Expected | Actual | Status |
| :---- | :---- | :---- | :---- | :---- |
| Income normal | balance=1jt, amount=500rb | 1.5jt | 1.5jt | ✅ PASSED |
| Expense normal | balance=1jt, amount=300rb | 700rb | 700rb | ✅ PASSED |
| Expense \> balance | balance=100rb, amount=500rb | Error | \-400rb | ❌ FAILED |
| Amount \= 0 | balance=1jt, amount=0 | Validation error | Validation error | ✅ PASSED |

**Total:** 4 skenario | **Passed:** 3 | **Failed:** 1 | **Coverage:** 75%

---

##  8\. Kelebihan & Kekurangan

###  Kelebihan

- Murah dan tidak butuh tools khusus (cukup pen & paper atau spreadsheet)  
- Mendeteksi bug logika lebih awal sebelum runtime  
- Meningkatkan pemahaman developer terhadap kode sendiri  
- Cocok untuk fungsi kecil dengan logika linear

###  Kekurangan

- **Tidak scalable** untuk codebase besar  
- Rentan terhadap *human error* — developer bisa "mengasumsikan" hasil yang salah  
- Tidak menangkap error runtime (DB connection, network, dll)  
- Bias: developer cenderung yakin kode-nya sudah benar  
- Tidak menguji integrasi antar modul

---

##  9\. Tools Pendukung

| Tool | Kegunaan dalam Desk Checking |
| :---- | :---- |
| **Spreadsheet (Excel/Sheets)** | Tabel trace variabel |
| **VS Code Debugger** | Step-through manual saat curiga |
| **PHPStan / Larastan** | Otomatis cek tipe & logic statik |
| **Pen & Paper** | Tracing tradisional |

---

##  Referensi

1. Pressman, R. S., & Maxim, B. R. (2020). *Software Engineering: A Practitioner's Approach* (9th ed.). McGraw-Hill.  
2. Suprihadi, D. (2025). *Materi Software Quality Pertemuan 10*. Universitas Kristen Indonesia.  
3. Ndaumanu, R. I. (2023). *Pengujian Sistem Informasi Perpustakaan Berbasis Website dengan Basis Path Testing*. Justek.

---

[ Kembali ke README](http://./README.md) · [Lanjut ke Code Walkthrough ➡](http://./Code_Walkthrough.md)

**Tim REMACode** — Midnight Finance SQA Documentation  
