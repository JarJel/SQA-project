# White Box Testing — 01 Desk Checking
**Proyek:** SaPoPoe Finance  
**Teknik:** Desk Checking — pembacaan kode manual tanpa eksekusi  
**Modul:** Auth · Transfer · Transaksi · Tabungan  
**Screenshot:** ❌ Tidak ada

---

## Apa itu Desk Checking?
Pembacaan kode secara manual oleh penguji untuk menemukan bug logika, celah keamanan, dan inkonsistensi **sebelum program dijalankan**.

---

## Modul A — Autentikasi (`AuthController.php`)

**Metode:** `register()`, `verifyOtp()`, `resendOtp()`, `login()`, `setup()`, `forgotPassword()`, `resetPassword()`, `logout()`

### Temuan DC-AUTH-01 — OTP Tidak Cryptographically Secure
```php
$otp = rand(100000, 999999);  // ← MASALAH
```
> `rand()` bukan CSPRNG (Cryptographically Secure Pseudo Random Number Generator). Pada sistem 32-bit, range-nya terbatas dan dapat diprediksi.  
> **Rekomendasi:** Ganti ke `random_int(100000, 999999)`.  
> **Severity:** 🔴 Tinggi

### Temuan DC-AUTH-02 — Email Dikirim Secara Sinkron
```php
Mail::send('emails.otp', [...], function($msg) { ... });  // blocking
```
> `Mail::send()` memblokir response hingga email terkirim. Jika SMTP lambat, user menunggu lama.  
> **Rekomendasi:** Ganti ke `Mail::queue()` + setup queue worker.  
> **Severity:** 🟡 Sedang

### Temuan DC-AUTH-03 — Sibling Detection OTP via `created_at`
```php
$siblings = Transaction::where('user_id', $user->id)
    ->where('created_at', $baseTransaction->created_at)->get();
```
> (di TransferController, tapi pola ini mirip OTP yang juga berbasis timestamp)  
> OTP cooldown hanya cek `updated_at` — bukan token unik. Potensi timing collision pada beban tinggi.  
> **Severity:** 🟡 Sedang

### Temuan DC-AUTH-04 — `$user` Didefinisikan Dua Kali di `register()`
```php
$user = User::where('email', $request->email)->first();   // D1: cek apakah ada
// ... validasi cooldown ...
$user = User::updateOrCreate([...]);                       // D2: overwrite
```
> Pola intentional (D1 untuk validasi, D2 untuk simpan), tapi membingungkan karena `$user` berubah tipe dari nullable ke pasti ada.  
> **Severity:** 🟢 Rendah

---

## Modul B — Transfer (`TransferController.php`)

**Metode:** `store()`, `update()`, `destroy()`

### Temuan DC-TRF-01 — Kategori Dibuat di Luar Transaksi DB
```php
// store() — baris 37-55
$transferCategory = Category::where(...)->first();
if (!$transferCategory) {
    $transferCategory = new Category();
    $transferCategory->save();   // ← Di luar DB::beginTransaction()!
}
// ...
DB::beginTransaction();         // Baru mulai transaksi di sini
```
> Jika transfer gagal di dalam transaksi, `DB::rollBack()` hanya me-revert data di dalam transaksi. Kategori yang baru dibuat **tidak ikut di-rollback** → data orphan di tabel `categories`.  
> **Rekomendasi:** Pindahkan pembuatan kategori ke dalam blok `DB::beginTransaction()`.  
> **Severity:** 🔴 Tinggi

### Temuan DC-TRF-02 — Sibling Detection via `created_at` (Race Condition)
```php
$siblings = Transaction::where('user_id', $user->id)
    ->where('created_at', $baseTransaction->created_at)->get();
```
> Jika dua transfer terjadi di detik yang sama (concurrent requests), siblings bisa saling tercampur → saldo salah terhitung.  
> **Rekomendasi:** Tambahkan kolom `transfer_group_id UUID` untuk menandai pasangan transaksi.  
> **Severity:** 🔴 Tinggi

### Temuan DC-TRF-03 — `FinancialAccount::find()` Tanpa Filter `user_id`
```php
// destroy() dan update() — di dalam foreach loop
$acc = FinancialAccount::find($trx->financial_account_id);  // tanpa user_id
```
> Tidak ada verifikasi kepemilikan akun di dalam loop. Jika data transaksi korup, bisa mengubah saldo akun milik user lain.  
> **Rekomendasi:** `FinancialAccount::where('id', ...)->where('user_id', $user->id)->first()`  
> **Severity:** 🟡 Sedang

### Temuan DC-TRF-04 — DRY Violation: Foreach Revert Terduplikasi
```php
// update() baris 144–158 dan destroy() baris 235–248 — kode identik
foreach ($siblings as $trx) { ... }
```
> Logika revert saldo sama persis di dua tempat. Bug yang diperbaiki di satu method harus diperbaiki juga di method lain.  
> **Severity:** 🟢 Rendah

---

## Modul C — Transaksi (`TransactionController.php`)

**Metode:** `index()`, `store()`, `update()`, `destroy()`

### Temuan DC-TRX-01 — Tidak Ada Pengecekan Saldo Negatif
```php
// store() — langsung kurangi saldo tanpa cek kecukupan
if ($validated['type'] === 'income') {
    $account->balance += $validated['amount'];
} else {
    $account->balance -= $validated['amount'];  // ← bisa negatif!
}
```
> Berbeda dengan Transfer yang mengecek saldo sebelum eksekusi, Transaksi langsung mengurangi tanpa validasi. Saldo bisa menjadi negatif.  
> **Rekomendasi:** Tambahkan `if ($account->balance < $validated['amount']) return 400;` sebelum deduction.  
> **Severity:** 🔴 Tinggi

### Temuan DC-TRX-02 — `findOrFail()` Tanpa Filter `user_id`
```php
// update() baris 106, destroy() baris 144
$oldAccount = FinancialAccount::findOrFail($transaction->financial_account_id);
$account    = FinancialAccount::findOrFail($transaction->financial_account_id);
```
> Sama seperti Transfer — tidak memfilter `user_id`. Data integrity bergantung pada asumsi bahwa `transaction->financial_account_id` selalu milik user yang sama.  
> **Severity:** 🟡 Sedang

### Temuan DC-TRX-03 — `index()` Tidak Ada Pagination
```php
$transactions = $query->get();  // ambil SEMUA tanpa limit
return response()->json(['data' => $transactions], 200);
```
> Jika user memiliki ribuan transaksi, `get()` tanpa pagination bisa timeout dan menghabiskan memori.  
> **Rekomendasi:** Ganti ke `$query->paginate(50)`.  
> **Severity:** 🟡 Sedang (performance)

### Temuan DC-TRX-04 — Kolom `sort_by` Tidak Divalidasi (SQL Injection Risk)
```php
$sortBy    = $request->input('sort_by', 'date');    // langsung dari user
$sortOrder = $request->input('sort_order', 'desc'); // langsung dari user
$query->orderBy($sortBy, $sortOrder);               // ← tidak divalidasi!
```
> `$sortBy` langsung dimasukkan ke `orderBy()` tanpa whitelist. Eloquent memang menggunakan parameter binding untuk nilai, tapi nama kolom di `orderBy()` **tidak di-binding** → potensi SQL injection via column name injection.  
> **Rekomendasi:** Whitelist: `$allowed = ['date','amount','created_at']; $sortBy = in_array($sortBy, $allowed) ? $sortBy : 'date';`  
> **Severity:** 🔴 Tinggi

---

## Modul D — Tabungan (`SavingController.php`)

**Metode:** `index()`, `store()`, `show()`, `update()`, `destroy()` + `getSavingCategory()`

### Temuan DC-SAV-01 — `getSavingCategory()` Dipanggil di Luar Transaksi DB
```php
// store() baris 53
$category = $this->getSavingCategory($user->id, 'expense');  // di dalam try, tapi getSavingCategory->save() tidak terproteksi
```
```php
// getSavingCategory() baris 19-25
if (!$category) {
    $category = new Category();
    $category->save();  // ← save() di dalam private method, tapi apakah ini di dalam scope transaksi?
}
```
> `getSavingCategory()` melakukan `Category::save()` — jika ini dipanggil di dalam blok `try` yang sudah ada `beginTransaction()`, maka terproteksi. Namun jika pernah dipanggil di luar transaksi, ini bisa membuat kategori orphan.  
> **Status:** Perlu verifikasi konteks pemanggilan.  
> **Severity:** 🟡 Sedang

### Temuan DC-SAV-02 — Tidak Ada Pengecekan Saldo di `store()`
```php
// store() baris 50
$account->balance -= $validated['current_amount'];  // ← bisa negatif
```
> Sama seperti Transaksi — tidak ada pengecekan apakah saldo mencukupi sebelum dikurangi untuk alokasi tabungan awal.  
> **Severity:** 🔴 Tinggi

### Temuan DC-SAV-03 — `FinancialAccount::findOrFail()` Tanpa Filter `user_id`
```php
// store() baris 49, update() baris 91, destroy() baris 135
$account = FinancialAccount::findOrFail($saving->financial_account_id);
```
> Tidak ada filter `user_id`. Sama polanya dengan Transaksi dan Transfer.  
> **Severity:** 🟡 Sedang

### Temuan DC-SAV-04 — Status `completed`/`canceled` dari Query Parameter Tidak Divalidasi
```php
// destroy() baris 128
$status = $request->query('status', 'canceled');  // langsung dari URL
// ...
$desc = $status === 'completed' ? 'Target Tercapai...' : 'Batal/Kepepet Cair...';
```
> `$status` hanya dipakai untuk string deskripsi, bukan operasi berbahaya. Tidak ada injection risk, tapi sebaiknya tetap divalidasi untuk kejelasan.  
> **Severity:** 🟢 Rendah

---

## Ringkasan Temuan Lintas Modul

| ID | Modul | Severity | Temuan |
|---|---|---|---|
| DC-AUTH-01 | Auth | 🔴 Tinggi | `rand()` untuk OTP — tidak aman |
| DC-AUTH-02 | Auth | 🟡 Sedang | `Mail::send()` sinkron — blocking |
| DC-TRF-01 | Transfer | 🔴 Tinggi | Kategori dibuat di luar DB transaction |
| DC-TRF-02 | Transfer | 🔴 Tinggi | Sibling detection via `created_at` — race condition |
| DC-TRF-03 | Transfer | 🟡 Sedang | `find()` tanpa filter `user_id` di loop |
| DC-TRF-04 | Transfer | 🟢 Rendah | Duplikasi foreach revert |
| DC-TRX-01 | Transaksi | 🔴 Tinggi | Tidak ada cek saldo negatif |
| DC-TRX-02 | Transaksi | 🟡 Sedang | `findOrFail()` tanpa `user_id` |
| DC-TRX-03 | Transaksi | 🟡 Sedang | Tidak ada pagination di `index()` |
| DC-TRX-04 | Transaksi | 🔴 Tinggi | `sort_by` dari user input langsung ke `orderBy()` — SQL injection |
| DC-SAV-01 | Tabungan | 🟡 Sedang | `getSavingCategory()` perlu verifikasi scope transaksi |
| DC-SAV-02 | Tabungan | 🔴 Tinggi | Tidak ada cek saldo negatif di alokasi awal |
| DC-SAV-03 | Tabungan | 🟡 Sedang | `findOrFail()` tanpa `user_id` |

**Total Temuan:** 13  
**Kritis (🔴):** 6 — **Sedang (🟡):** 5 — **Rendah (🟢):** 2
