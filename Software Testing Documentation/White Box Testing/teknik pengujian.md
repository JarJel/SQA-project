---

### 2.2.2 White Box Testing

Teknik ini melibatkan pemeriksaan terhadap struktur internal dan logika kode program.

| Aspek Pengujian        | Deskripsi Teknis Pelaksanaan                                                                 | Target Output & Validasi                                                                 |
|----------------------|---------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| Logika Percabangan   | Menguji if-else dan switch pada proses transaksi.                                           | Semua kondisi (diskon, pajak, dll) berjalan dengan benar.                                |
| Security Code Review | Audit kode untuk mendeteksi hardcoded credentials.                                          | Data sensitif disimpan di environment variable.                                           |
| Database Query       | Menguji performa query SQL.                                                                 | Query efisien dan tidak membebani database.                                               |
| Unit Testing Logic   | Menguji fungsi seperti perhitungan total harga.                                              | Fungsi menghasilkan output yang benar.                                                    |
| Code Coverage        | Mengukur seberapa banyak kode yang teruji.                                                   | Minimal 80% code coverage tercapai.                                                       |

---
