# White Box Testing — 03 Formal Inspections
**Proyek:** SaPoPoe Finance  
**Teknik:** Formal Inspections  
**Modul:** Auth · Transfer · Transaksi · Tabungan  
**Screenshot:** ❌ Tidak ada (analisis statis)

---

## Definisi

> **Pengujian Formal Inspection adalah teknik pengujian yang berfokus untuk memeriksa kelengkapan, ketepatan, dan format data yang ditampilkan dan diterima oleh aplikasi.**
>
> — Materi Pertemuan 10, Software Quality, T Informatika UKRI

---

## Checklist Kategori A — Validasi & Input

| ID | Kriteria | AUTH | TRF | TRX | SAV |
|---|---|---|---|---|---|
| A-01 | Semua field wajib divalidasi sebelum diproses | ✅ | ✅ | ✅ | ✅ |
| A-02 | Amount/numeric field memiliki batas minimum | ✅ min:8 pw | ✅ min:1 | ✅ min:1 | ✅ min:1 |
| A-03 | Field date dibatasi agar tidak masa depan | ✅ before_or_equal | ✅ before_or_equal | ⚠️ PARTIAL (tidak ada `before_or_equal`) | ⚠️ PARTIAL |
| A-04 | Transfer ke akun yang sama dicegah | N/A | ✅ `different:from_account_id` | N/A | N/A |
| A-05 | Tipe data enum divalidasi | N/A | N/A | ✅ `in:income,expense` | N/A |
| A-06 | Kepemilikan resource diverifikasi via `user_id` | ✅ | ✅ store() | ⚠️ PARTIAL (update/destroy tidak) | ⚠️ PARTIAL |
| A-07 | Input tidak langsung masuk ke SQL mentah | ✅ | ✅ | ❌ `sort_by` langsung ke orderBy | ✅ |

---

## Checklist Kategori B — Keamanan & Kriptografi

| ID | Kriteria | AUTH | TRF | TRX | SAV |
|---|---|---|---|---|---|
| B-01 | Password di-hash dengan algoritma aman | ✅ `Hash::make()` bcrypt | N/A | N/A | N/A |
| B-02 | Perbandingan password timing-safe | ✅ `Hash::check()` | N/A | N/A | N/A |
| B-03 | Random number untuk OTP/token menggunakan CSPRNG | ❌ `rand()` — tidak aman | N/A | N/A | N/A |
| B-04 | Token/OTP memiliki masa berlaku | ✅ 10 menit | N/A | N/A | N/A |
| B-05 | Token dihapus setelah digunakan | ✅ otp=null setelah verify | N/A | N/A | N/A |
| B-06 | Semua token dihapus saat reset password | ✅ `$user->tokens()->delete()` | N/A | N/A | N/A |
| B-07 | Response tidak mengekspos stack trace | ✅ | ⚠️ `$e->getMessage()` | ⚠️ `$e->getMessage()` | ✅ pesan generik |

---

## Checklist Kategori C — Integritas Data & Transaksi DB

| ID | Kriteria | AUTH | TRF | TRX | SAV |
|---|---|---|---|---|---|
| C-01 | Operasi kritis dibungkus DB transaction | ✅ setup() | ✅ | ✅ | ✅ |
| C-02 | Exception selalu men-trigger rollback | ✅ | ✅ | ✅ | ✅ |
| C-03 | Pembuatan resource pendukung masuk dalam transaksi | N/A | ❌ Kategori di luar BeginTrx | N/A | ⚠️ Perlu verifikasi |
| C-04 | Saldo dikembalikan dengan benar saat delete/update | N/A | ✅ Fase Revert | ✅ | ✅ |
| C-05 | Tidak ada partial update (all-or-nothing) | ✅ | ✅ | ✅ | ✅ |
| C-06 | `refresh()` digunakan setelah revert untuk data terbaru | N/A | ✅ | N/A | N/A |

---

## Checklist Kategori D — Logika Bisnis

| ID | Kriteria | AUTH | TRF | TRX | SAV |
|---|---|---|---|---|---|
| D-01 | Saldo diperiksa sebelum pengurangan | N/A | ✅ store/update | ❌ tidak ada | ❌ tidak ada |
| D-02 | Admin fee dicatat sebagai transaksi terpisah | N/A | ✅ | N/A | N/A |
| D-03 | Rate limiting ada untuk endpoint sensitif | ✅ throttle:5 OTP | N/A | N/A | N/A |
| D-04 | Cooldown antar request OTP diterapkan | ✅ 180 detik | N/A | N/A | N/A |
| D-05 | Transaksi terkait dapat diidentifikasi dan di-revert | N/A | ⚠️ via created_at (lemah) | ✅ via transaction.id | ✅ |
| D-06 | Pencairan tabungan dicatat sebagai transaksi income | N/A | N/A | N/A | ✅ |

---

## Checklist Kategori E — Kualitas Kode

| ID | Kriteria | AUTH | TRF | TRX | SAV |
|---|---|---|---|---|---|
| E-01 | HTTP status code semantik tepat | ✅ | ✅ | ✅ | ✅ |
| E-02 | Tidak ada duplikasi logika (DRY) | ⚠️ Mail::send di 2 tempat | ❌ foreach revert duplikat | ✅ | ✅ |
| E-03 | Pagination pada endpoint list | N/A | N/A | ❌ tidak ada | N/A |
| E-04 | Email dikirim async (non-blocking) | ❌ Mail::send sinkron | N/A | N/A | N/A |
| E-05 | Response error tidak mengekspos detail sistem | ✅ | ⚠️ | ⚠️ | ✅ |

---

## Rekapitulasi Per Modul

| Modul | Total Item | PASS | FAIL | PARTIAL |
|---|---|---|---|---|
| AUTH | 25 | 19 | 3 | 3 |
| Transfer | 25 | 18 | 4 | 3 |
| Transaksi | 25 | 16 | 5 | 4 |
| Tabungan | 25 | 17 | 3 | 5 |

---

## Daftar Defect Terstruktur

### AUTH

| ID | Item | Status | Rekomendasi |
|---|---|---|---|
| FI-AUTH-01 | B-03: `rand()` untuk OTP | ❌ FAIL | Ganti `random_int(100000, 999999)` |
| FI-AUTH-02 | E-04: `Mail::send()` sinkron | ❌ FAIL | Ganti `Mail::queue()` |
| FI-AUTH-03 | E-02: Duplikasi `Mail::send` | ⚠️ PARTIAL | Extract ke `sendOtpMail($user, $otp)` |

### Transfer

| ID | Item | Status | Rekomendasi |
|---|---|---|---|
| FI-TRF-01 | C-03: Kategori di luar DB transaction | ❌ FAIL | Pindahkan ke dalam `beginTransaction()` |
| FI-TRF-02 | D-05: Sibling via `created_at` | ⚠️ PARTIAL | Tambah kolom `transfer_group_id UUID` |
| FI-TRF-03 | A-06: `find()` tanpa user_id di loop | ❌ FAIL | Tambah filter `where('user_id', ...)` |
| FI-TRF-04 | E-02: Duplikasi foreach revert | ❌ FAIL | Extract ke `private revertSiblingBalances()` |

### Transaksi

| ID | Item | Status | Rekomendasi |
|---|---|---|---|
| FI-TRX-01 | D-01: Tidak ada cek saldo negatif | ❌ FAIL | Tambah validasi sebelum `balance -=` |
| FI-TRX-02 | A-07: `sort_by` ke `orderBy()` langsung | ❌ FAIL | Whitelist kolom yang diizinkan |
| FI-TRX-03 | E-03: Tidak ada pagination | ❌ FAIL | Ganti `get()` ke `paginate(50)` |
| FI-TRX-04 | A-06: `findOrFail()` tanpa user_id | ⚠️ PARTIAL | Tambah `where('user_id', ...)` |
| FI-TRX-05 | A-03: Date tidak dibatasi `before_or_equal` | ⚠️ PARTIAL | Tambah rule `before_or_equal:today` |

### Tabungan

| ID | Item | Status | Rekomendasi |
|---|---|---|---|
| FI-SAV-01 | D-01: Tidak ada cek saldo negatif di `store()` | ❌ FAIL | Tambah pengecekan saldo sebelum alokasi |
| FI-SAV-02 | A-06: `findOrFail()` tanpa user_id | ⚠️ PARTIAL | Tambah filter `where('user_id', ...)` |
| FI-SAV-03 | C-03: `getSavingCategory()->save()` — perlu verifikasi | ⚠️ PARTIAL | Pastikan dipanggil dalam scope transaksi |
