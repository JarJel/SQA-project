# White Box Testing — 01 Desk Checking
**Proyek:** SaPoPoe Finance  
**Teknik:** Desk Checking  
**Modul:** Auth · Transfer · Transaksi · Tabungan

---

## Definisi

> **Desk Checking adalah salah satu pengujian bagi para pembuat software yang telah mempelajari bahasa pemrograman dengan sangat baik, karena Pemeriksaan berfokus pada logika dan nilai variabel, pada input dan output yang diperlukan oleh aplikasi.**
>
> — Materi Pertemuan 10, Software Quality, T Informatika UKRI

---

## Modul A — Autentikasi (`AuthController.php`)
### Method: `login()`

**Test Data:**
- Input Email = `dzaki@mail.com`
- Input Password = `Midnight@2026`
- Kondisi DB: user ada, hash password cocok, `otp_code` = NULL
- Output yang diharapkan: `200 OK` + `access_token`

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
| | | | | C3 : `$user->otp_code != null` ? Is F | `createToken('auth_token')` → access_token | Passed |
| | | | | | return 200 { access\_token, user } | Passed |

---

> ### 📋 Analisis SQA — Modul Auth
>
> **Kondisi Sistem Saat Ini**
> Method `login()` sudah mengimplementasikan tiga lapis perlindungan yang berurutan: verifikasi keberadaan akun (C1), kecocokan kata sandi (C2), dan status verifikasi email (C3). Ketiga kondisi ini membentuk *chain of guards* yang solid. Nilai variabel `$otp_code = NULL` menjadi sinyal sah bahwa akun sudah melalui verifikasi OTP.
>
> **Dampak**
> Urutan kondisi C1→C2→C3 sudah benar dari sisi keamanan. Jika C1 dilewati terlebih dahulu ke C2, ada risiko timing attack. Namun ada satu celah: jika tabel `users` memiliki record dengan `otp_code` berisi nilai lama (tidak di-null-kan setelah expired), user bisa terkunci permanen. Hal ini telah terjadi di akun uji `dzakiawaludin0@gmail.com`.
>
> **Cara Baca Tabel Ini**
> Tabel dibaca dari atas ke bawah mengikuti alur eksekusi kode. Baris yang hanya berisi nilai di kolom variabel menunjukkan **assignment** (variabel diberi nilai). Baris yang berisi kolom *Condition* menunjukkan **percabangan** — `Is T` berarti kondisi bernilai TRUE dan menjalankan blok pertama, `Is F` berarti FALSE dan menjalankan blok alternatif. Status `Passed` artinya logika kode menghasilkan output sesuai ekspektasi.

---

## Modul B — Transfer (`TransferController.php`)
### Method: `store()` — Transfer dengan Biaya Admin

**Test Data:**
- Saldo BCA (from) = Rp 600.000
- Nominal Transfer = Rp 200.000
- Admin Fee = Rp 5.000
- Total Deduction = 200.000 + 5.000 = Rp 205.000
- Output yang diharapkan: `200 OK` "Transfer berhasil! Biaya admin Rp 5.000 dicatat."

| $saldoBCA | $amount | $adminFee | $totalDeduction | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| 600.000 | | | | | | |
| | 200.000 | | | | | |
| | | 5.000 | | | | |
| | | | 205.000 | | | |
| | | | | C1 : `$saldoBCA < $totalDeduction` ? Is T | return 400 "Saldo tidak mencukupi" | Passed |
| | | | | C1 : `$saldoBCA < $totalDeduction` ? Is F | Lanjut proses → DB::beginTransaction() | Passed |
| | | | | C2 : `$adminFee > 0` ? Is T | INSERT trx biaya admin Rp 5.000 ke tabel transactions | Passed |
| | | | | C2 : `$adminFee > 0` ? Is F | Transfer selesai tanpa pencatatan biaya admin | Passed |
| 395.000 | | | | | Saldo BCA = 600.000 − 205.000 | Passed |
| | | | | | Saldo Mandiri = 100.000 + 200.000 = 300.000 | Passed |
| | | | | | return 200 "Transfer berhasil! Biaya admin Rp 5.000 dicatat." | Passed |

---

> ### 📋 Analisis SQA — Modul Transfer
>
> **Kondisi Sistem Saat Ini**
> Method `store()` sudah melakukan pengecekan saldo (C1) sebelum memotong — ini praktik yang benar. Namun ada celah kritis: pembuatan `Category` (kategori Transfer dan Biaya Admin) terjadi **sebelum** blok `DB::beginTransaction()`. Artinya jika transfer gagal di tengah jalan dan `DB::rollBack()` dijalankan, kategori yang sudah dibuat tidak ikut di-rollback, menghasilkan data orphan di tabel `categories`.
>
> **Dampak**
> Jika server down atau query gagal di dalam transaksi, saldo tidak terpotong (berhasil di-rollback) tapi kategori "Transfer Internal" dan "Biaya Admin Bank" sudah telanjur tersimpan. Duplikasi kategori bisa terjadi pada eksekusi berikutnya tergantung logika `where()->first()`.
>
> **Cara Baca Tabel Ini**
> Kolom `$totalDeduction` adalah variabel yang nilainya merupakan hasil kalkulasi (amount + adminFee). Perhatikan baris C1 — tabel membandingkan nilai ini dengan saldo. Baris terakhir menunjukkan state akhir semua akun setelah seluruh eksekusi selesai. Jika ada baris dengan Status `Failed`, berarti ditemukan inkonsistensi antara logika kode dan hasil yang diharapkan.

---

## Modul C — Transaksi (`TransactionController.php`)
### Method: `store()` — Transaksi Income

**Test Data:**
- Saldo Awal BCA = Rp 300.000
- Nominal = Rp 100.000
- Tipe = `income`
- Saldo Akhir yang diharapkan = 300.000 + 100.000 = Rp 400.000

| $saldoAwal | $amount | $type | $saldoAkhir | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| 300.000 | | | | | | |
| | 100.000 | | | | | |
| | | "income" | | | | |
| | | | | C1 : `$type === 'income'` ? Is T | `$account->balance += 100.000` → saldo bertambah | Passed |
| | | | | C1 : `$type === 'income'` ? Is F (expense) | `$account->balance -= 100.000` → saldo berkurang | **Failed** ⚠️ |
| | | | 400.000 | | `Transaction::create({type:'income', amount:100000})` | Passed |
| | | | | | return 201 { transaksi baru } | Passed |

> ⚠️ **Catatan:** Pada Is F (type=expense), tidak ada validasi apakah `$saldoAwal >= $amount` sebelum pengurangan. Jika `amount > saldoAwal`, saldo akan menjadi **negatif** — ini adalah bug logika bisnis.

---

> ### 📋 Analisis SQA — Modul Transaksi
>
> **Kondisi Sistem Saat Ini**
> Method `store()` di TransactionController mempercayakan validasi hanya pada Laravel Validator (`min:1`), yang hanya memastikan amount adalah angka positif. Tidak ada pengecekan apakah saldo akun mencukupi sebelum dilakukan pengurangan untuk transaksi bertipe `expense`. Ini berbeda dengan `TransferController` yang sudah memiliki guard saldo di baris 27.
>
> **Dampak**
> User dapat mencatat pengeluaran (expense) yang melebihi saldo akun, menghasilkan `balance` negatif di tabel `financial_accounts`. Jika sistem digunakan untuk pelaporan keuangan nyata, angka ini akan menyesatkan. Ini adalah **defect fungsional dengan severity tinggi**.
>
> **Cara Baca Tabel Ini**
> Perhatikan baris dengan Status `Failed` — ini menandai titik di mana logika kode tidak sesuai ekspektasi bisnis. Pada Desk Checking, `Failed` bukan berarti program crash, melainkan **logika tidak sempurna** — program tetap berjalan tapi menghasilkan state yang salah. Ini adalah tujuan utama teknik ini: menemukan cacat logika sebelum pengujian runtime.

---

## Modul D — Tabungan (`SavingController.php`)
### Method: `update()` — Top Up Tabungan

**Test Data:**
- Saldo BCA = Rp 1.500.000
- Tabungan "Dana Darurat" current_amount lama = Rp 200.000
- Tabungan current_amount baru = Rp 350.000
- Selisih = 350.000 − 200.000 = Rp 150.000 (top up)
- Output yang diharapkan: Saldo BCA = 1.500.000 − 150.000 = Rp 1.350.000

| $oldAmount | $newAmount | $selisih | $saldoBCA | Condition | Input / Output | Status / Hasil |
|---|---|---|---|---|---|---|
| 200.000 | | | | | | |
| | 350.000 | | | | | |
| | | 150.000 | | | | |
| | | | 1.500.000 | | | |
| | | | | C1 : `$newAmount !== $oldAmount` ? Is T | Proses kalkulasi selisih dan update saldo | Passed |
| | | | | C1 : `$newAmount !== $oldAmount` ? Is F | Skip update saldo, hanya update metadata | Passed |
| | | | | C2 : `$selisih > 0` ? Is T (top up) | `$saldoBCA -= 150.000` → BCA = 1.350.000 | Passed |
| | | | | C2 : `$selisih > 0` ? Is F (penarikan) | `$saldoBCA += abs($selisih)` → BCA bertambah | **Failed** ⚠️ |
| | | | 1.350.000 | | INSERT trx { type:'expense', amount:150.000 } | Passed |
| | | | | | return 200 { saving, saldo\_bca: 1.350.000 } | Passed |

> ⚠️ **Catatan Is F:** Pada kasus penarikan (`selisih < 0`), tidak ada validasi apakah `current_amount - abs(selisih) >= 0` sebelum dieksekusi. Tabungan bisa menjadi nilai negatif.

---

> ### 📋 Analisis SQA — Modul Tabungan
>
> **Kondisi Sistem Saat Ini**
> `update()` di SavingController menggunakan variabel `$selisih` untuk menentukan arah pergerakan saldo — pendekatan yang elegan. Namun, tidak ada guard untuk memastikan bahwa penarikan tidak melebihi `current_amount` tabungan yang ada. Kondisi `$selisih > 0` (C2) menangani dua cabang dengan benar, tetapi tidak memvalidasi batas bawah.
>
> **Dampak**
> User dapat menarik dana dari tabungan melebihi jumlah yang tersimpan di tabungan tersebut, menyebabkan `current_amount` negatif. Dampaknya berlapis: saldo akun bertambah (income) dari nilai yang sebenarnya tidak ada, mengacaukan laporan keuangan dan integritas data tabungan.
>
> **Cara Baca Tabel Ini**
> Variabel `$selisih` adalah variabel kunci dalam tabel ini — nilainya menentukan alur eksekusi di C2. Perhatikan bahwa `$selisih` bisa positif (top up) atau negatif (penarikan), dan method menggunakan `abs($selisih)` untuk memastikan amount transaksi selalu positif. Baris `Failed` di C2 Is F menunjukkan bahwa cabang penarikan membutuhkan validasi tambahan sebelum dapat dinyatakan aman.
