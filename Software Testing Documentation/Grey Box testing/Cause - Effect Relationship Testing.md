
# ⚡ Cause-Effect Relationship Testing
**Midnight Finance · Grey Box Testing Suite**  
*Tim REMAKO · Software Quality Assurance*

---

## Pendahuluan

**Cause-Effect Relationship Testing** adalah teknik Grey Box Testing yang secara sistematis memetakan relasi antara **kondisi input** (*cause*) dan **hasil output** (*effect*). Teknik ini memastikan setiap kombinasi kondisi yang relevan menghasilkan output yang tepat sesuai logika bisnis.

Dengan pengetahuan tentang skema database dan logika API Midnight Finance, penguji dapat memverifikasi tidak hanya respons HTTP tetapi juga **dampak terhadap data di database**.

---

## Notasi

| Simbol | Arti |
|---|---|
| **T** | True / Kondisi terpenuhi |
| **F** | False / Kondisi tidak terpenuhi |
| **✓** | Effect terjadi / Expected output |
| **✗** | Effect tidak terjadi |
| **—** | Tidak relevan (*don't care*) |

---

## Modul 1 — Autentikasi: Login

### Causes (Kondisi Input)

| ID | Kondisi |
|---|---|
| C1 | Email terdaftar di database |
| C2 | Password cocok dengan hash di DB |
| C3 | Akun sudah terverifikasi email |
| C4 | Akun tidak terkunci |

### Effects (Output yang Diharapkan)

| ID | Output |
|---|---|
| E1 | Login berhasil — token Sanctum diterbitkan |
| E2 | Error: "Email atau password salah" |
| E3 | Error: "Email belum diverifikasi" |
| E4 | Error: "Akun terkunci sementara" |

### Decision Table

| Kondisi / Effect | TC-L01 | TC-L02 | TC-L03 | TC-L04 | TC-L05 |
|---|:---:|:---:|:---:|:---:|:---:|
| C1: Email terdaftar | T | T | T | T | F |
| C2: Password cocok | T | T | T | F | — |
| C3: Sudah verifikasi | T | T | F | — | — |
| C4: Tidak terkunci | T | F | — | — | — |
| **E1: Login berhasil** | **✓** | ✗ | ✗ | ✗ | ✗ |
| **E2: Error kredensial** | ✗ | ✗ | ✗ | **✓** | **✓** |
| **E3: Error verifikasi** | ✗ | ✗ | **✓** | ✗ | ✗ |
| **E4: Error terkunci** | ✗ | **✓** | ✗ | ✗ | ✗ |

### Verifikasi Database

| TC | Tabel yang Diverifikasi | Kondisi Diverifikasi |
|---|---|---|
| TC-L01 | `personal_access_tokens` | Token baru tersimpan |
| TC-L02 | `users` | `locked_until` masih aktif |
| TC-L03 | `users` | `email_verified_at` = NULL |
| TC-L04 | Tidak ada perubahan DB | Tidak ada token baru |
| TC-L05 | Tidak ada perubahan DB | Tidak ada token baru |

---

## Modul 2 — Transaksi: Tambah Expense

### Causes

| ID | Kondisi |
|---|---|
| C1 | Saldo dompet mencukupi (saldo ≥ nominal) |
| C2 | Kategori dipilih (tidak null) |
| C3 | Nominal > 0 |
| C4 | Tanggal valid (format & tidak di masa depan) |
| C5 | Dompet dimiliki oleh user yang login |

### Effects

| ID | Output |
|---|---|
| E1 | Transaksi tersimpan di tabel `transactions` |
| E2 | Saldo dompet berkurang sebesar nominal |
| E3 | Error: "Saldo tidak mencukupi" |
| E4 | Error: "Field wajib tidak lengkap" |
| E5 | Error: "Nominal tidak valid" |
| E6 | Error: "Akses ditolak (401/403)" |

### Decision Table

| Kondisi / Effect | TC-E01 | TC-E02 | TC-E03 | TC-E04 | TC-E05 | TC-E06 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| C1: Saldo cukup | T | F | T | T | T | T |
| C2: Kategori dipilih | T | — | F | T | T | T |
| C3: Nominal > 0 | T | — | — | F | T | T |
| C4: Tanggal valid | T | — | — | — | F | T |
| C5: Dompet milik user | T | — | — | — | — | F |
| **E1: Transaksi tersimpan** | **✓** | ✗ | ✗ | ✗ | ✗ | ✗ |
| **E2: Saldo berkurang** | **✓** | ✗ | ✗ | ✗ | ✗ | ✗ |
| **E3: Error saldo** | ✗ | **✓** | ✗ | ✗ | ✗ | ✗ |
| **E4: Error field** | ✗ | ✗ | **✓** | **✓** | ✗ | ✗ |
| **E5: Error nominal** | ✗ | ✗ | ✗ | ✗ | **✓** | ✗ |
| **E6: Error akses** | ✗ | ✗ | ✗ | ✗ | ✗ | **✓** |

---

## Modul 2 — Transfer Antar Dompet

### Causes

| ID | Kondisi |
|---|---|
| C1 | Dompet sumber memiliki saldo cukup |
| C2 | Dompet tujuan berbeda dari dompet sumber |
| C3 | Kedua dompet dimiliki user yang sama |
| C4 | Nominal > 0 |

### Effects

| ID | Output |
|---|---|
| E1 | Saldo dompet sumber berkurang |
| E2 | Saldo dompet tujuan bertambah |
| E3 | Dua record transaksi tersimpan (debit + kredit) |
| E4 | Error: "Saldo tidak cukup" |
| E5 | Error: "Dompet sumber dan tujuan sama" |
| E6 | Error: "Akses ditolak" |

### Decision Table

| Kondisi / Effect | TC-T01 | TC-T02 | TC-T03 | TC-T04 | TC-T05 |
|---|:---:|:---:|:---:|:---:|:---:|
| C1: Saldo sumber cukup | T | F | T | T | T |
| C2: Dompet berbeda | T | — | F | T | T |
| C3: Dompet milik user | T | — | — | F | T |
| C4: Nominal > 0 | T | — | — | — | F |
| **E1+E2: Transfer berhasil** | **✓** | ✗ | ✗ | ✗ | ✗ |
| **E3: 2 record tersimpan** | **✓** | ✗ | ✗ | ✗ | ✗ |
| **E4: Error saldo** | ✗ | **✓** | ✗ | ✗ | ✗ |
| **E5: Error dompet sama** | ✗ | ✗ | **✓** | ✗ | ✗ |
| **E6: Error akses** | ✗ | ✗ | ✗ | **✓** | ✗ |

---

## Modul 3 — Anggaran: Evaluasi Status

### Causes

| ID | Kondisi |
|---|---|
| C1 | Total pengeluaran kategori < 80% dari limit |
| C2 | Total pengeluaran kategori antara 80% - 99% dari limit |
| C3 | Total pengeluaran kategori ≥ 100% dari limit |

### Effects

| ID | Output |
|---|---|
| E1 | Status anggaran: **AMAN** (hijau) |
| E2 | Status anggaran: **WARNING** (kuning) + notifikasi |
| E3 | Status anggaran: **OVERBUDGET** (merah) + notifikasi |

### Decision Table

| Kondisi / Effect | TC-B01 | TC-B02 | TC-B03 |
|---|:---:|:---:|:---:|
| C1: Pengeluaran < 80% | T | F | F |
| C2: 80% ≤ Pengeluaran < 100% | F | T | F |
| C3: Pengeluaran ≥ 100% | F | F | T |
| **E1: Status AMAN** | **✓** | ✗ | ✗ |
| **E2: Status WARNING** | ✗ | **✓** | ✗ |
| **E3: Status OVERBUDGET** | ✗ | ✗ | **✓** |

### Kasus Uji Batas Kritis

| ID | Input | Expected Output |
|---|---|---|
| CE-BGT-01 | Pemakaian = 79% dari limit | Status: AMAN |
| CE-BGT-02 | Pemakaian = 80% dari limit | Status: WARNING |
| CE-BGT-03 | Pemakaian = 99% dari limit | Status: WARNING |
| CE-BGT-04 | Pemakaian = 100% dari limit | Status: OVERBUDGET |
| CE-BGT-05 | Pemakaian = 150% dari limit | Status: OVERBUDGET |

---

## Modul 4 — Hutang: Pencatatan Cicilan

### Causes

| ID | Kondisi |
|---|---|
| C1 | Nominal cicilan > 0 |
| C2 | Nominal cicilan ≤ `remaining_amount` |
| C3 | Status hutang masih AKTIF (bukan LUNAS) |
| C4 | Saldo dompet cukup untuk membayar cicilan |

### Effects

| ID | Output |
|---|---|
| E1 | Cicilan tersimpan, `remaining_amount` berkurang |
| E2 | Status berubah ke LUNAS (`remaining_amount` = 0) |
| E3 | Saldo dompet berkurang sebesar cicilan |
| E4 | Error: "Nominal cicilan melebihi sisa hutang" |
| E5 | Error: "Saldo tidak mencukupi" |
| E6 | Error: "Hutang sudah lunas" |

### Decision Table

| Kondisi / Effect | TC-D01 | TC-D02 | TC-D03 | TC-D04 | TC-D05 |
|---|:---:|:---:|:---:|:---:|:---:|
| C1: Nominal > 0 | T | T | T | F | T |
| C2: Cicilan ≤ sisa hutang | T (sebagian) | T (lunas) | F | — | T |
| C3: Status AKTIF | T | T | T | — | F |
| C4: Saldo dompet cukup | T | T | — | — | — |
| **E1: Cicilan tersimpan** | **✓** | ✗ | ✗ | ✗ | ✗ |
| **E2: Status LUNAS** | ✗ | **✓** | ✗ | ✗ | ✗ |
| **E3: Saldo berkurang** | **✓** | **✓** | ✗ | ✗ | ✗ |
| **E4: Error melebihi sisa** | ✗ | ✗ | **✓** | ✗ | ✗ |
| **E5: Error saldo** | ✗ | ✗ | ✗ | **✓** | ✗ |
| **E6: Error sudah lunas** | ✗ | ✗ | ✗ | ✗ | **✓** |

---

## Ringkasan Test Cases

| Modul | Jumlah TC | Passed | Failed | Pending |
|---|---|---|---|---|
| Login | 5 | — | — | — |
| Tambah Expense | 6 | — | — | — |
| Transfer | 5 | — | — | — |
| Anggaran | 8 | — | — | — |
| Hutang Cicilan | 5 | — | — | — |
| **Total** | **29** | **—** | **—** | **—** |

---

*Midnight Finance SQA · Tim REMAKO*
