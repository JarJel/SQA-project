# White Box Testing — 01 Desk Checking
**Proyek:** SaPoPoe Finance  
**Teknik:** Desk Checking  
**Modul:** Auth · Transfer · Transaksi · Tabungan  
**Screenshot:** ❌ Tidak ada (analisis statis)

---

## Definisi

> **Desk Checking adalah salah satu pengujian bagi para pembuat software yang telah mempelajari bahasa pemrograman dengan sangat baik, karena Pemeriksaan berfokus pada logika dan nilai variabel, pada input dan output yang diperlukan oleh aplikasi.**
>
> — Materi Pertemuan 10, Software Quality, T Informatika UKRI

---

## Modul A — Autentikasi (`AuthController.php`)

### Skenario: Method `login()`

**Test Data:**
- Input Email = `dzaki@mail.com`
- Input Password = `Midnight@2026`
- Data di DB: user ada, password hash cocok, `otp_code` = NULL (sudah terverifikasi)
- Output yang diharapkan = `200 OK` + `access_token`

| $email | $password | $user | $otp\_code | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| "dzaki@mail.com" | | | | | | |
| | "Midnight@2026" | | | | | |
| | | {id:1, email:"dzaki@mail.com", password:"\$2y\$..."} | | | | |
| | | | NULL | | | |
| | | | | C1 : `$user === null` ? Is T | return 404 "Alamat email tidak ditemukan" | Passed |
| | | | | C1 : `$user === null` ? Is F | Lanjut validasi password → C2 | Passed |
| | | | | C2 : `!Hash::check($password, $user->password)` ? Is T | return 401 "Kata sandi salah" | Passed |
| | | | | C2 : `!Hash::check($password, $user->password)` ? Is F | Lanjut cek verifikasi → C3 | Passed |
| | | | | C3 : `$user->otp_code != null` ? Is T | return 403 "Akun belum diverifikasi" | Passed |
| | | | | C3 : `$user->otp_code != null` ? Is F | `$user->createToken('auth_token')` | Passed |
| | | | | | return 200 { access\_token, user } | Passed |

**Kesimpulan:** Semua jalur logika login() berjalan sesuai ekspektasi. Variabel `$user` dan `$otp_code` digunakan dengan tepat sebagai penjaga akses.

---

## Modul B — Transfer (`TransferController.php`)

### Skenario: Method `store()` — Transfer dengan Biaya Admin

**Test Data:**
- Saldo BCA (from) = Rp 600.000
- Nominal Transfer = Rp 200.000
- Admin Fee = Rp 5.000
- Total Deduction = 200.000 + 5.000 = Rp 205.000
- Output yang diharapkan = `200 OK` "Transfer berhasil! Biaya admin Rp 5.000 dicatat."

| $saldoBCA | $amount | $adminFee | $totalDeduction | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| 600000 | | | | | | |
| | 200000 | | | | | |
| | | 5000 | | | | |
| | | | 205000 | | | |
| | | | | C1 : `$saldoBCA < $totalDeduction` ? Is T | return 400 "Saldo tidak mencukupi" | Passed |
| | | | | C1 : `$saldoBCA < $totalDeduction` ? Is F | Lanjut proses transfer → DB::beginTransaction() | Passed |
| | | | | C2 : `$adminFee > 0` ? Is T | INSERT trx biaya admin Rp 5.000 ke tabel transactions | Passed |
| | | | | C2 : `$adminFee > 0` ? Is F | Transfer selesai tanpa biaya admin | Passed |
| | | | | | Saldo BCA = 600.000 − 205.000 = **395.000** | Passed |
| | | | | | Saldo Mandiri = 100.000 + 200.000 = **300.000** | Passed |
| | | | | | return 200 "Transfer berhasil! Biaya admin Rp 5.000 dicatat." | Passed |

**Kesimpulan:** Logika pemotongan saldo dan pencatatan biaya admin berjalan benar. Variabel `$totalDeduction` menggabungkan amount + adminFee dengan tepat.

---

## Modul C — Transaksi (`TransactionController.php`)

### Skenario: Method `store()` — Transaksi Income

**Test Data:**
- Saldo Awal BCA = Rp 300.000
- Nominal = Rp 100.000
- Tipe = `income`
- Saldo Akhir yang diharapkan = 300.000 + 100.000 = Rp 400.000

| $saldoAwal | $amount | $type | $saldoAkhir | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| 300000 | | | | | | |
| | 100000 | | | | | |
| | | "income" | | | | |
| | | | | C1 : `$type === 'income'` ? Is T | `$account->balance += 100000` → saldo bertambah | Passed |
| | | | | C1 : `$type === 'income'` ? Is F | `$account->balance -= 100000` → saldo berkurang | Passed |
| | | | 400000 | | | |
| | | | | | `Transaction::create({type:'income', amount:100000})` | Passed |
| | | | | | return 201 { transaksi, saldo\_akhir: 400.000 } | Passed |

> ⚠️ **Catatan Desk Checking:** Pada Is F (type=expense), tidak ada pengecekan apakah `$saldoAwal >= $amount` sebelum pengurangan. Saldo bisa menjadi negatif jika `$amount > $saldoAwal`.

**Kesimpulan:** Logika percabangan income/expense benar, namun ditemukan tidak adanya validasi saldo minimum untuk kasus expense — berpotensi menghasilkan saldo negatif.

---

## Modul D — Tabungan (`SavingController.php`)

### Skenario: Method `update()` — Top Up Tabungan

**Test Data:**
- Saldo BCA = Rp 1.500.000
- Tabungan "Dana Darurat" current_amount lama = Rp 200.000
- Tabungan current_amount baru = Rp 350.000
- Selisih = 350.000 − 200.000 = Rp 150.000 (top up)
- Output yang diharapkan: Saldo BCA berkurang 150.000 → Rp 1.350.000

| $oldAmount | $newAmount | $selisih | $saldoBCA | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| 200000 | | | | | | |
| | 350000 | | | | | |
| | | 150000 | | | | |
| | | | 1500000 | | | |
| | | | | C1 : `$newAmount !== $oldAmount` ? Is T | Proses kalkulasi selisih dan update saldo | Passed |
| | | | | C1 : `$newAmount !== $oldAmount` ? Is F | Tidak ada perubahan saldo, hanya update metadata | Passed |
| | | | | C2 : `$selisih > 0` ? Is T (top up) | `$saldoBCA -= 150000` → BCA = 1.350.000 | Passed |
| | | | | C2 : `$selisih > 0` ? Is F (penarikan) | `$saldoBCA += abs($selisih)` → BCA bertambah | Passed |
| | | | 1350000 | | | |
| | | | | | INSERT trx { type:'expense', amount:150000, desc:'Top Up Tabungan Dana Darurat' } | Passed |
| | | | | | return 200 { saving, saldo\_bca: 1.350.000 } | Passed |

**Kesimpulan:** Logika percabangan top up vs penarikan pada `update()` berjalan benar. Variabel `$selisih` menentukan arah perubahan saldo dengan tepat.

---

## Ringkasan Temuan Desk Checking

| Modul | Method | Temuan | Severity |
|---|---|---|---|
| Auth | `login()` | Semua kondisi (C1, C2, C3) berfungsi sebagai gatekeeper yang tepat | ✅ OK |
| Transfer | `store()` | Pengecekan saldo dan pencatatan admin fee berjalan benar | ✅ OK |
| Transaksi | `store()` | Tidak ada validasi saldo minimum sebelum expense — saldo bisa negatif | 🔴 Bug |
| Tabungan | `update()` | Logika top up / penarikan menggunakan `$selisih` dengan benar | ✅ OK |
