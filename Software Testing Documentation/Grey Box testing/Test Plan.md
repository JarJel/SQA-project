# 🔍 Grey Box Testing Report  
## Midnight Finance – Private Wealth Management Platform

---

## 1. Informasi Umum

| Atribut | Detail |
|--------|--------|
| **Nama Aplikasi** | Midnight Finance |
| **Platform** | Web Application |
| **Tech Stack** | Laravel · React.js · MySQL |
| **Metode Testing** | Grey Box Testing |
| **Tanggal Pengujian** | 26 April 2026 |
| **Tim Penguji** | Tim REMAKO |

---

## 2. Deskripsi Grey Box Testing

Grey Box Testing adalah metode pengujian yang menggabungkan pendekatan:
- **Black Box Testing** → fokus pada input/output user
- **White Box Testing** → memahami sebagian struktur internal (API, database, logic)

Pada aplikasi *Midnight Finance*, pengujian dilakukan dengan memahami:
- Struktur API Laravel (endpoint & response)
- Relasi database (saldo, transaksi, anggaran)
- Alur logika bisnis keuangan

---

## 3. Tujuan Pengujian

- Memastikan integrasi frontend (React) dan backend (Laravel API) berjalan dengan baik  
- Memvalidasi logika bisnis keuangan (saldo, transaksi, budgeting)  
- Mengidentifikasi bug pada proses yang melibatkan database dan API  
- Menguji keamanan dasar pada autentikasi  

---

## 4. Scope Pengujian (Berdasarkan Modul)

### 🔐 1. Autentikasi & Keamanan
- Login & register
- Validasi token (Sanctum)
- Session & middleware

### 💳 2. Portofolio & Transaksi
- Input income & expense
- Perhitungan saldo otomatis
- Transfer antar dompet

### 📊 3. Anggaran Bulanan
- Limit budget per kategori
- Status penggunaan (Aman / Warning / Overbudget)

### 📒 4. Hutang & Piutang
- Penambahan data hutang
- Sistem cicilan & bunga
- Sinkronisasi ke saldo

### 📈 5. Analitik Dashboard
- Filter waktu
- Akurasi data grafik

---

## 5. Skenario & Hasil Pengujian

| No | Modul | Skenario | Input | Expected Result | Actual Result | Status |
|----|------|---------|------|----------------|--------------|--------|
| 1 | Autentikasi | Login valid | Email & password benar | Login berhasil & token dibuat | Berhasil | ✅ Pass |
| 2 | Autentikasi | Token invalid | Token diubah manual | Akses ditolak | Akses masih bisa | ❌ Fail |
| 3 | Transaksi | Tambah income | nominal = 1.000.000 | Saldo bertambah | Sesuai | ✅ Pass |
| 4 | Transaksi | Input negatif | nominal = -50000 | Ditolak | Tetap masuk DB | ❌ Fail |
| 5 | Transaksi | Transfer saldo | antar dompet | Saldo sinkron | Tidak sinkron | ❌ Fail |
| 6 | Anggaran | Overbudget | limit 1jt, transaksi 1.2jt | Status "Overbudget" | Status tidak berubah | ❌ Fail |
| 7 | Hutang | Tambah hutang | bunga 10% | Total terhitung benar | Salah hitung | ❌ Fail |
| 8 | Dashboard | Filter 7 hari | range waktu | Data sesuai filter | Data tidak berubah | ❌ Fail |

---

## 6. Analisis Temuan

### 🔴 Critical Issues
- Validasi token tidak aman (bisa bypass autentikasi)
- Input negatif masih tersimpan ke database
- Ketidaksesuaian saldo pada transfer

### 🟠 High Issues
- Perhitungan bunga hutang tidak akurat
- Status anggaran tidak berubah saat overbudget

### 🟡 Medium Issues
- Filter dashboard tidak bekerja sesuai query

---

## 7. Analisis Teknis (Grey Box Insight)

Berdasarkan pemahaman struktur internal:

- **Backend Issue**
  - Validasi input belum diterapkan di layer controller/service
  - Logic transaksi belum menggunakan atomic transaction (DB transaction)

- **Database Issue**
  - Tidak ada constraint untuk nilai negatif
  - Relasi saldo tidak di-lock saat update

- **API Issue**
  - Response tidak konsisten
  - Error handling belum optimal (HTTP 500)

---

## 8. Rekomendasi Perbaikan

### Backend (Laravel)
- Gunakan **Form Request Validation**
- Tambahkan **DB Transaction (`DB::transaction`)**
- Perbaiki middleware autentikasi

### Database (MySQL)
- Tambahkan constraint:
  ```sql
  CHECK (amount >= 0)
