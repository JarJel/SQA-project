# BB-09 — Cause-Effect Relationship Testing
## Sistem: SaPoPoe FINANCE (Midnight Finance)
## Teknik: Black Box Testing — Cause-Effect Relationship Testing

---

> **Definisi Teknik:**
> Cause-Effect Relationship Testing adalah teknik pengujian dengan cara **membagi spesifikasi menjadi bagian-bagian yang sesuai dengan kebutuhan**, kemudian menentukan **cause** (penyebab) dan **effect** (akibat) berdasarkan spesifikasi kebutuhan, lalu menganalisis hubungan antar keduanya.
>
> Cause merupakan **kondisi input atau situasi** yang memicu suatu perilaku sistem. Effect merupakan **output atau perubahan kondisi** yang dihasilkan oleh system sebagai respons atas cause tersebut. Teknik ini divisualisasikan menggunakan **diagram fishbone (Ishikawa)**.
>
> — Materi Pertemuan 11, Software Quality, T Informatika UKRI

---

## Modul 1 — Autentikasi
### Effect: *"Mengapa Login Bisa Gagal / Berhasil?"*

```mermaid
graph LR
    classDef cat fill:#fef3c7,stroke:#f59e0b,color:#1a1a1a,font-weight:bold
    classDef sub fill:#dbeafe,stroke:#3b82f6,color:#1a1a1a
    classDef effFail fill:#fee2e2,stroke:#ef4444,color:#7f1d1d,font-weight:bold
    classDef effPass fill:#dcfce7,stroke:#22c55e,color:#14532d,font-weight:bold

    s1["Email tidak\nterdaftar"] --> C1["Input\nPengguna"]
    s2["Password\nsalah"] --> C1
    s3["Field email /\npassword kosong"] --> C1

    s4["Tidak ada validasi\nreal-time di frontend"] --> C2["Validasi\nFrontend"]
    s5["Validasi hanya\nterjadi di backend"] --> C2

    s6["Password hash\ntidak cocok di DB"] --> C3["Proses\nBackend"]
    s7["Email tidak\nditemukan di tabel users"] --> C3

    s8["Tidak ada\nTwo-Factor Auth"] --> C4["Keamanan\nSistem"]
    s9["Tidak ada\nrate limiting login"] --> C4

    C1 --> EF["Login\nGagal"]
    C2 --> EF
    C3 --> EF
    C4 --> EF

    s10["Email terdaftar\n& password benar"] --> C5["Input Valid"]
    s11["Token Sanctum\nditerbitkan"] --> C5
    C5 --> EP["Login\nBerhasil"]

    class C1,C2,C3,C4 cat
    class C5 cat
    class s1,s2,s3,s4,s5,s6,s7,s8,s9,s10,s11 sub
    class EF effFail
    class EP effPass
```

### Tabel Cause → Effect

| No | Cause (Penyebab) | Kategori | Effect (Akibat) | Status |
|---|---|---|---|---|
| 1 | Email tidak terdaftar di database | Input Pengguna | "Alamat email tidak ditemukan." | 🔴 Gagal |
| 2 | Password tidak sesuai akun | Input Pengguna | "Kata sandi yang Anda masukkan salah." | 🔴 Gagal |
| 3 | Field email atau password kosong | Input Pengguna | Form tidak dikirim — "Harap isi bidang ini." | 🔴 Gagal |
| 4 | Tidak ada validasi real-time | Validasi Frontend | Error baru diketahui setelah submit | ⚠️ Kelemahan UX |
| 5 | Email terdaftar & password benar | Input Valid | Token Sanctum diterbitkan → Dashboard tampil | ✅ Berhasil |

### Screenshot Bukti

**Cause: Email valid + Password benar → Effect: Login Berhasil**

<img width="1920" height="1080" alt="sample-auth-tc1" src="https://github.com/user-attachments/assets/c48b3d89-a158-48fb-b0fd-ae7e3b2fff8b" />


**Cause: Password salah → Effect: Login Ditolak**

<img width="1920" height="1080" alt="sample-auth-tc2" src="https://github.com/user-attachments/assets/4cb02a5f-8e27-47f6-a14c-ffc9b3219c38" />


---

## Modul 2 — Transfer (Pindah Dana)
### Effect: *"Mengapa Transfer Bisa Gagal?"*

```mermaid
graph LR
    classDef cat fill:#fef3c7,stroke:#f59e0b,color:#1a1a1a,font-weight:bold
    classDef sub fill:#dbeafe,stroke:#3b82f6,color:#1a1a1a
    classDef effFail fill:#fee2e2,stroke:#ef4444,color:#7f1d1d,font-weight:bold
    classDef effPass fill:#dcfce7,stroke:#22c55e,color:#14532d,font-weight:bold
    classDef effBug fill:#fce7f3,stroke:#ec4899,color:#831843,font-weight:bold

    s1["Amount = 0\n(tidak valid)"] --> C1["Input\nPengguna"]
    s2["Amount melebihi\nsaldo brankas asal"] --> C1
    s3["Brankas sumber\ntidak dipilih"] --> C1

    s4["Cek saldo sebelum\ntransfer sudah ada"] --> C2["Validasi\nBackend"]
    s5["Tidak ada konfirmasi\nsebelum eksekusi"] --> C2

    s6["Bug di\nTransferController@index"] --> C3["Bug\nBackend"]
    s7["Relasi Eloquent\ngagal di-load"] --> C3
    s8["GET /api/transfers\nselalu return 500"] --> C3

    s9["Tidak ada notifikasi\ntransfer masuk"] --> C4["Fitur\nSistem"]

    C1 --> EF["Transfer\nDitolak"]
    C2 --> EF
    C3 --> EB["GET Riwayat\nTransfer Gagal\n(HTTP 500)"]
    C4 --> EW["Pengalaman\nPengguna Kurang"]

    s10["Amount > 0 &\nsaldo mencukupi"] --> C5["Input Valid"]
    C5 --> EP["Transfer\nBerhasil"]

    class C1,C2,C3,C4,C5 cat
    class s1,s2,s3,s4,s5,s6,s7,s8,s9,s10 sub
    class EF effFail
    class EB effBug
    class EP effPass
    class EW effFail
```

### Tabel Cause → Effect

| No | Cause (Penyebab) | Kategori | Effect (Akibat) | Status |
|---|---|---|---|---|
| 1 | Amount = 0 atau kosong | Input Pengguna | "Harap isi bidang ini." — tidak dikirim | 🔴 Ditolak |
| 2 | Amount > saldo brankas asal | Input Pengguna | "GAGAL Saldo tidak mencukupi..." | 🔴 Ditolak |
| 3 | Bug di `TransferController@index()` | Bug Backend | GET /api/transfers → HTTP 500 di semua request | 🔴 Bug Baru |
| 4 | Tidak ada konfirmasi sebelum eksekusi | Validasi Backend | Transfer langsung diproses tanpa verifikasi ulang | ⚠️ Risiko |
| 5 | Amount valid & saldo mencukupi | Input Valid | "BERHASIL Dana berhasil dipindahkan!" | ✅ Berhasil |

### Screenshot Bukti

**Cause: Amount valid, saldo cukup → Effect: Transfer Berhasil**

<img width="1920" height="1080" alt="sample-transfer-tc1" src="https://github.com/user-attachments/assets/2b925ba5-2d1e-4a2d-8f1d-8c966db7c33f" />


**Cause: Amount > saldo → Effect: Transfer Ditolak**

<img width="1920" height="1080" alt="sample-transfer-tc3" src="https://github.com/user-attachments/assets/453d0579-4978-4c04-84e4-16d9d2f408a5" />


---

## Modul 3 — Transaksi (Catat Aliran Dana)
### Effect: *"Mengapa Saldo Bisa Menjadi Negatif?"*

```mermaid
graph LR
    classDef cat fill:#fef3c7,stroke:#f59e0b,color:#1a1a1a,font-weight:bold
    classDef sub fill:#dbeafe,stroke:#3b82f6,color:#1a1a1a
    classDef effFail fill:#fee2e2,stroke:#ef4444,color:#7f1d1d,font-weight:bold
    classDef effPass fill:#dcfce7,stroke:#22c55e,color:#14532d,font-weight:bold
    classDef effBug fill:#fce7f3,stroke:#ec4899,color:#831843,font-weight:bold

    s1["Amount ekstrem\n(misal 999.999.999)"] --> C1["Input\nPengguna"]
    s2["Type = expense\n(transaksi keluar)"] --> C1
    s3["Portofolio dipilih\n(brankas dengan saldo kecil)"] --> C1

    s4["Tidak ada pengecekan\nsaldo di backend"] --> C2["Bug Validasi\nBackend"]
    s5["TransactionController::store()\ntanpa balance check"] --> C2
    s6["Validasi hanya\ndi level frontend (browser)"] --> C2

    s7["Tidak ada CHECK\nconstraint saldo >= 0 di DB"] --> C3["Kelemahan\nDatabase"]
    s8["Saldo langsung\ndikurangi tanpa guard"] --> C3

    s9["Portofolio tidak\ndipilih — form ditolak"] --> C4["Validasi\nFrontend"]
    s10["Amount kosong —\nform tidak dikirim"] --> C4

    C1 --> EB["Saldo Menjadi\nNegatif\n(BUG KRITIS)"]
    C2 --> EB
    C3 --> EB
    C4 --> EF["Transaksi\nDitolak\n(Benar)"]

    s11["Amount valid, type valid\nportofolio dipilih"] --> C5["Input Valid"]
    C5 --> EP["Transaksi Tercatat\nSaldo Diperbarui"]

    class C1,C2,C3,C4,C5 cat
    class s1,s2,s3,s4,s5,s6,s7,s8,s9,s10,s11 sub
    class EB effBug
    class EF effFail
    class EP effPass
```

### Tabel Cause → Effect

| No | Cause (Penyebab) | Kategori | Effect (Akibat) | Status |
|---|---|---|---|---|
| 1 | Amount = 0 atau kosong | Validasi Frontend | "Harap isi bidang ini." — tidak dikirim | ✅ Benar |
| 2 | Portofolio tidak dipilih | Validasi Frontend | "Pilih item pada daftar." — tidak dikirim | ✅ Benar |
| 3 | Tidak ada `balance check` di `TransactionController::store()` | Bug Backend | Expense ekstrem diproses → saldo negatif | 🔴 Bug Kritis |
| 4 | Tidak ada CHECK constraint `saldo >= 0` di database | Kelemahan DB | Database menerima nilai saldo negatif | 🔴 Kelemahan |
| 5 | Amount valid + type valid + portofolio dipilih | Input Valid | Transaksi tercatat, saldo diperbarui sesuai | ✅ Berhasil |

### Screenshot Bukti

**Cause: Income valid → Effect: Transaksi Berhasil, Saldo Bertambah**

sample-transaksi-tc1.png

**Cause: Expense ekstrem tanpa validasi saldo → Effect: Saldo Negatif (Bug Kritis)**



---

## Modul 4 — Tabungan (Target Impian)
### Effect: *"Mengapa Tabungan Bisa Ditolak / Berhasil?"*

```mermaid
graph LR
    classDef cat fill:#fef3c7,stroke:#f59e0b,color:#1a1a1a,font-weight:bold
    classDef sub fill:#dbeafe,stroke:#3b82f6,color:#1a1a1a
    classDef effFail fill:#fee2e2,stroke:#ef4444,color:#7f1d1d,font-weight:bold
    classDef effPass fill:#dcfce7,stroke:#22c55e,color:#14532d,font-weight:bold
    classDef effWarn fill:#fef9c3,stroke:#eab308,color:#713f12,font-weight:bold

    s1["Nama tabungan\nkosong (0 karakter)"] --> C1["Input\nPengguna"]
    s2["Nama tabungan\n> 255 karakter"] --> C1
    s3["Target amount = 0\natau kosong"] --> C1

    s4["Browser menolak\nfield required kosong"] --> C2["Validasi\nFrontend"]
    s5["Form tidak dikirim\nsebelum ke server"] --> C2

    s6["max:255 validation\ndi SavingController"] --> C3["Validasi\nBackend"]
    s7["target amount\nminimal 1 (422)"] --> C3

    s8["Backend tidak cek\nsaldo untuk deposit"] --> C4["Kelemahan\nBackend"]
    s9["Rentan direct\nAPI call bypass"] --> C4

    C1 --> EF["Tabungan\nDitolak"]
    C2 --> EF
    C3 --> EF
    C4 --> EW["Risiko\nManipulasi API"]

    s10["Nama valid ≤ 255 char\ntarget > 0, sumber dipilih"] --> C5["Input Valid"]
    C5 --> EP["Tabungan Berhasil\nDibuat"]

    class C1,C2,C3,C4,C5 cat
    class s1,s2,s3,s4,s5,s6,s7,s8,s9,s10 sub
    class EF effFail
    class EW effWarn
    class EP effPass
```

### Tabel Cause → Effect

| No | Cause (Penyebab) | Kategori | Effect (Akibat) | Status |
|---|---|---|---|---|
| 1 | Nama tabungan kosong | Validasi Frontend | "Harap isi bidang ini." — tidak dikirim | ✅ Benar |
| 2 | Nama tabungan > 255 karakter | Validasi Backend | "GAGAL — The name field must not be greater than 255 characters." | ✅ Benar |
| 3 | Target amount = 0 | Validasi Backend | HTTP 422 — "target amount minimal 1" | ✅ Benar |
| 4 | Backend tidak memvalidasi saldo saat deposit | Kelemahan Backend | Direct API call bisa bypass frontend validation | ⚠️ Risiko |
| 5 | Nama valid, target > 0, sumber dipilih | Input Valid | Tabungan berhasil dibuat, muncul di daftar | ✅ Berhasil |

### Screenshot Bukti

**Cause: Input valid → Effect: Tabungan Berhasil Dibuat**

<img width="1920" height="1080" alt="sample-tabungan-tc1" src="https://github.com/user-attachments/assets/60dc3c2e-143c-4d1e-b3a6-835b500346ab" />


**Cause: Nama > 255 karakter → Effect: Ditolak Backend**

<img width="1920" height="1080" alt="sample-tabungan-tc3" src="https://github.com/user-attachments/assets/4d31b6cb-c63c-402c-9eea-24ee977868d7" />


---

## Ringkasan Temuan Cause-Effect — Seluruh Sistem

| Modul | Jumlah Cause | Effect Benar ✅ | Effect Bug 🔴 | Kelemahan ⚠️ |
|---|---|---|---|---|
| Autentikasi | 5 | 4 | 0 | 1 (tidak ada real-time validation) |
| Transfer | 5 | 2 | 1 (GET 500) | 2 (konfirmasi, notifikasi) |
| Transaksi | 5 | 3 | 1 (saldo negatif) | 1 (constraint DB) |
| Tabungan | 5 | 4 | 0 | 1 (bypass API) |
| **TOTAL** | **20** | **13** | **2** | **5** |

### Rekapitulasi Defect yang Ditemukan

| No | Defect | Modul | Cause | Effect | Rekomendasi |
|---|---|---|---|---|---|
| 1 | Saldo bisa negatif | Transaksi | Tidak ada `balance check` di `TransactionController::store()` | Saldo BCA menjadi Rp -994.649.999 | Tambahkan `if ($wallet->balance < $request->amount) return 400` |
| 2 | GET riwayat transfer error | Transfer | Bug di `TransferController@index()` — relasi gagal | HTTP 500 di setiap request GET /api/transfers | Debug query Eloquent di `TransferController@index()` |

> **Kesimpulan Cause-Effect Testing:** Dari pemetaan 20 hubungan cause-effect pada 4 modul, ditemukan **2 defect aktif** dan **5 area kelemahan** yang berpotensi menjadi risiko. Defect paling kritis adalah saldo negatif pada modul Transaksi yang disebabkan oleh tidak adanya pengecekan saldo di layer backend. Diagram fishbone membantu mengidentifikasi bahwa akar masalah bukan di layer input pengguna, melainkan di **layer validasi backend** yang tidak lengkap.
