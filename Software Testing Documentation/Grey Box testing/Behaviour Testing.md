
# 🔄 Behaviour Testing
**Midnight Finance · Grey Box Testing Suite**  
*Tim REMAKO · Software Quality Assurance*

---

## Pendahuluan

**Behaviour Testing** memverifikasi bahwa sistem berperilaku sesuai dengan spesifikasi untuk setiap kondisi dan alur yang mungkin terjadi. Pengujian ini berbasis **State Machine** — memodelkan sistem sebagai kumpulan *state* yang berpindah berdasarkan *event/trigger* tertentu.

Dengan pengetahuan parsial tentang arsitektur Midnight Finance (skema DB + alur API), penguji dapat memvalidasi tidak hanya tampilan UI tetapi juga **konsistensi state di level database**.

---

## Modul 1 — Autentikasi & Keamanan

### State Machine: Akun Pengguna

```
[Belum Terdaftar]
       │ Register (email, password, nama)
       ↓
[Terdaftar - Belum Verifikasi]
       │ Klik link OTP / masukkan kode OTP
       ↓
[Aktif]
       │ Login berhasil             │ Logout / Token expired
       ↓                            ↓
[Sesi Aktif] ────────────────▶ [Sesi Berakhir]
       │ Terlalu banyak percobaan login gagal
       ↓
[Terkunci Sementara]
       │ Waktu lockout habis
       ↓
[Aktif]
```

### Test Cases — Behaviour Autentikasi

| ID | State Awal | Event / Aksi | State Akhir Ekspektasi | Verifikasi DB |
|---|---|---|---|---|
| BT-AUTH-01 | Belum Terdaftar | Register dengan data valid | Terdaftar - Belum Verifikasi | Record di tabel `users` dengan `email_verified_at = NULL` |
| BT-AUTH-02 | Terdaftar - Belum Verifikasi | Masukkan OTP valid | Aktif | `email_verified_at` terisi timestamp |
| BT-AUTH-03 | Terdaftar - Belum Verifikasi | Masukkan OTP salah 3x | Belum Verifikasi (kode diinvalidasi) | OTP lama tidak dapat digunakan |
| BT-AUTH-04 | Aktif | Login dengan kredensial benar | Sesi Aktif | Token Sanctum tersimpan di tabel `personal_access_tokens` |
| BT-AUTH-05 | Sesi Aktif | Logout | Sesi Berakhir | Token dihapus dari `personal_access_tokens` |
| BT-AUTH-06 | Sesi Berakhir | Akses endpoint dengan token lama | Ditolak (401) | Respons `{"message": "Unauthenticated"}` |
| BT-AUTH-07 | Aktif | Login gagal 5x berturut-turut | Terkunci Sementara | Rate limiter aktif, respons 429 |

---

## Modul 2 — Portofolio & Transaksi

### State Machine: Dompet

```
[Baru Dibuat]
  saldo = nilai awal (bisa 0)
       │ Tambah Income
       ↓
[Saldo Positif]
       │ Tambah Expense (nominal ≤ saldo)
       ↓
[Saldo Berkurang / Bisa 0]
       │ Tambah Expense (nominal > saldo)
       ↓
[DITOLAK — Saldo Tidak Cukup]
       │ Terima Transfer dari dompet lain
       ↓
[Saldo Bertambah]
```

### Test Cases — Behaviour Transaksi

| ID | State Awal | Event | State Akhir Ekspektasi | Verifikasi DB |
|---|---|---|---|---|
| BT-TRX-01 | Saldo = Rp 500.000 | Tambah income Rp 200.000 | Saldo = Rp 700.000 | `wallets.balance` = 700000, record di `transactions` |
| BT-TRX-02 | Saldo = Rp 500.000 | Tambah expense Rp 300.000 | Saldo = Rp 200.000 | `wallets.balance` = 200000, record di `transactions` |
| BT-TRX-03 | Saldo = Rp 100.000 | Tambah expense Rp 200.000 | DITOLAK | Saldo tidak berubah, tidak ada record transaksi baru |
| BT-TRX-04 | Dompet A = Rp 500.000, B = Rp 100.000 | Transfer A→B Rp 200.000 | A = Rp 300.000, B = Rp 300.000 | Kedua `wallets.balance` terupdate secara atomik |
| BT-TRX-05 | Saldo = Rp 0 | Tambah expense Rp 1 | DITOLAK | Saldo tetap 0 |
| BT-TRX-06 | Transaksi ada | Hapus transaksi income | Saldo berkurang sesuai nominal | Saldo di-rollback, record dihapus |

---

## Modul 3 — Anggaran Bulanan

### State Machine: Status Anggaran

```
[AMAN]
pemakaian < 80%
       │ Pengeluaran kategori bertambah → pemakaian ≥ 80%
       ↓
[WARNING]
80% ≤ pemakaian < 100%
       │ Pengeluaran bertambah lagi → pemakaian ≥ 100%
       ↓
[OVERBUDGET]
pemakaian ≥ 100%
       │ Bulan baru / Reset manual
       ↓
[AMAN] (pemakaian = 0%)
```

### Test Cases — Behaviour Anggaran

| ID | State Awal | Event | State Akhir Ekspektasi | Verifikasi |
|---|---|---|---|---|
| BT-BGT-01 | Limit = Rp 500.000, Pemakaian = 0 | Expense Rp 350.000 di kategori ini | Status: AMAN (70%) | Persentase = 70% |
| BT-BGT-02 | Limit = Rp 500.000, Pemakaian = Rp 350.000 | Expense Rp 100.000 | Status: WARNING (90%) | Persentase = 90%, notifikasi warning |
| BT-BGT-03 | Limit = Rp 500.000, Pemakaian = Rp 450.000 | Expense Rp 100.000 | Status: OVERBUDGET (110%) | Persentase = 110%, notifikasi overbudget |
| BT-BGT-04 | Status: OVERBUDGET | Hapus transaksi terakhir | Status kembali sesuai perhitungan ulang | Persentase dihitung ulang dari DB |
| BT-BGT-05 | Status: WARNING (80%) | Tambah expense hingga tepat 100% | Status: OVERBUDGET | Boundary tepat di 100% |
| BT-BGT-06 | Status: AMAN (79%) | Tambah expense hingga tepat 80% | Status: WARNING | Boundary tepat di 80% |

---

## Modul 4 — Hutang & Piutang

### State Machine: Record Hutang/Piutang

```
[AKTIF]
remaining_amount > 0
       │ Bayar cicilan (cicilan < remaining)
       ↓
[AKTIF - Sisa Berkurang]
       │ Bayar cicilan (cicilan = remaining)
       ↓
[LUNAS]
remaining_amount = 0
       │ (status final, tidak bisa diubah)
       ─
[LUNAS]
```

### Test Cases — Behaviour Hutang & Piutang

| ID | State Awal | Event | State Akhir Ekspektasi | Verifikasi DB |
|---|---|---|---|---|
| BT-DEBT-01 | Hutang baru Rp 1.000.000, 0 cicilan | Bayar cicilan Rp 200.000 | Sisa = Rp 800.000, Status: AKTIF | `remaining_amount` = 800000 |
| BT-DEBT-02 | Hutang Rp 200.000 sisa | Bayar lunas Rp 200.000 | Sisa = Rp 0, Status: LUNAS | `status` = 'lunas', `remaining_amount` = 0 |
| BT-DEBT-03 | Hutang dengan bunga 10%/bulan | Lewati 1 bulan | Nominal bertambah 10% | `amount` terupdate sesuai kalkulasi bunga |
| BT-DEBT-04 | Bayar cicilan | Cek saldo dompet | Saldo berkurang sesuai nominal cicilan | `wallets.balance` berkurang sinkron |
| BT-DEBT-05 | Hutang LUNAS | Coba bayar cicilan lagi | DITOLAK — hutang sudah lunas | Status tidak berubah |

---

## Modul 5 — Analitik Dashboard

### State Machine: Filter Waktu Dashboard

```
[All Time]
       │ User pilih filter "7 Hari"
       ↓
[7 Hari Terakhir]
       │ User pilih filter "1 Bulan"
       ↓
[1 Bulan Terakhir]
       │ User pilih filter "All Time"
       ↓
[All Time]
```

### Test Cases — Behaviour Analitik

| ID | State Awal | Event | State Akhir Ekspektasi | Verifikasi |
|---|---|---|---|---|
| BT-ANL-01 | Filter: All Time | Pilih filter 7 Hari | Grafik hanya tampilkan 7 hari terakhir | Query range = today-7 s/d today |
| BT-ANL-02 | Filter: 7 Hari | Pilih filter 1 Bulan | Grafik tampilkan 30 hari terakhir | Query range = today-30 s/d today |
| BT-ANL-03 | Tidak ada transaksi dalam 7 hari | Pilih filter 7 Hari | Grafik kosong / data nol | Tidak ada error, chart tampil kosong |
| BT-ANL-04 | Ada 10 transaksi hari ini | Cek pie chart | Proporsi sesuai kategori | Persentase per kategori = (sum kategori / total) × 100% |

---

## Ringkasan Hasil Eksekusi

| Modul | Total TC | Passed | Failed | Pending |
|---|---|---|---|---|
| Autentikasi | 7 | — | — | — |
| Transaksi | 6 | — | — | — |
| Anggaran | 6 | — | — | — |
| Hutang & Piutang | 5 | — | — | — |
| Analitik | 4 | — | — | — |
| **Total** | **28** | **—** | **—** | **—** |

*Tabel diperbarui setelah eksekusi pengujian.*

---

*Midnight Finance SQA · Tim REMAKO*
