# 🔄 Regression Testing — Midnight Finance

**Mata Kuliah:** Software Quality Assurance  
**Model Pengujian:** Gray Box Testing — Regression Testing  
**Tim:** REMACode  
**Modul Target:** Sinkronisasi Saldo Otomatis (`TransactionController` + `FinancialAccount`)  

---

## 📖 Definisi

**Regression Testing** adalah teknik pengujian perangkat lunak yang berfokus pada **memastikan bahwa perubahan yang dilakukan pada kode program tidak menyebabkan bug atau masalah baru** pada fungsionalitas yang sudah ada. Teknik ini penting untuk **menjaga stabilitas dan kualitas perangkat lunak** selama proses pengembangan (Suprihadi, 2025).

**Skenario yang dapat dipergunakan dalam Regression Testing:**
1. Menambah Fitur Baru
2. Memperbaiki Bug dan Gangguan
3. Mengubah Infrastruktur

---

## 🎯 Skenario Regression Testing di Midnight Finance

Fitur kritis yang diuji: setiap kali transaksi **dicatat / diedit / dihapus**, saldo `FinancialAccount` harus otomatis diperbarui. Ini adalah logika inti yang harus tetap benar setelah setiap perubahan kode.

```php
// Sinkronisasi saldo di TransactionController@store
if ($validated['type'] === 'income') {
    $account->balance += $validated['amount'];
} else {
    $account->balance -= $validated['amount'];
}
```

---

## 📝 Test Case Regression Testing — Sinkronisasi Saldo

| No | Test Case | Aksi | Kondisi Awal | Input | Expected Saldo | Actual Saldo | Status |
|:--:|:---|:---:|:---:|:---|:---:|:---:|:---:|
| TC-RG-01 | Catat income → saldo bertambah | POST /api/transactions | Saldo: Rp 1.000.000 | type: `income`, amount: `500000` | Rp 1.500.000 | Rp 1.500.000 | ✅ Valid |
| TC-RG-02 | Catat expense → saldo berkurang | POST /api/transactions | Saldo: Rp 1.500.000 | type: `expense`, amount: `200000` | Rp 1.300.000 | Rp 1.300.000 | ✅ Valid |
| TC-RG-03 | Edit transaksi (ubah amount) | PUT /api/transactions/{id} | Saldo: Rp 1.300.000, tx lama: income Rp 500.000 | type: `income`, amount: `700000` | Rp 1.500.000 | Rp 1.500.000 | ✅ Valid |
| TC-RG-04 | Hapus transaksi income | DELETE /api/transactions/{id} | Saldo: Rp 1.500.000, tx: income Rp 700.000 | *(hapus)* | Rp 800.000 | Rp 800.000 | ✅ Valid |
| TC-RG-05 | Hapus transaksi expense | DELETE /api/transactions/{id} | Saldo: Rp 800.000, tx: expense Rp 200.000 | *(hapus)* | Rp 1.000.000 | Rp 1.000.000 | ✅ Valid |
| TC-RG-06 | Tidak ada regresi setelah hapus | POST /api/transactions | Saldo: Rp 1.000.000 | type: `income`, amount: `250000` | Rp 1.250.000 | Rp 1.250.000 | ✅ Valid |

---

## ✅ Hasil Pengujian

| Kategori | Jumlah |
|:---|:---:|
| Total Test Case | 6 |
| Test Case Passed | 6 |
| Regresi Ditemukan | 0 |
| **Persentase Keberhasilan** | **100%** |

> **Kesimpulan:** Tidak ditemukan regresi pada fitur sinkronisasi saldo setelah operasi CRUD transaksi. Saldo `FinancialAccount` selalu diperbarui dengan benar sesuai tipe dan jumlah transaksi.

---

## 📚 Referensi

- Suprihadi, D. (2025). *Software Quality — Gray Box Testing*. T Informatika UKRI.
