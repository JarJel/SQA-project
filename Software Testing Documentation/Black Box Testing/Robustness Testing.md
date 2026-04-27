## **Robustness Testing**
### **Deskripsi**
Robustness Testing atau pengujian ketahanan adalah metode untuk mengecek seberapa baik sistem dalam **menangani kondisi tidak normal**, input yang rusak, atau gangguan lingkungan kerja.
### **Tujuan**
* **Mencegah sistem crash:** Memastikan aplikasi tidak berhenti bekerja secara mendadak saat menerima data yang salah.
* **Keamanan data:** Menjamin integritas data tetap terjaga meskipun proses transaksi terganggu (misal: koneksi terputus).
* **Error Handling:** Memastikan pesan kesalahan yang muncul mudah dimengerti oleh pengguna dan memberikan solusi.

### **Digunakan pada**
* **Kondisi Jaringan:** Menguji perilaku aplikasi saat sinyal internet lemah atau terputus mendadak.
* **Input Ekstrem:** Memasukkan karakter yang tidak didukung atau mengunggah file yang rusak.
* **Integritas Hardware:** Melihat respon aplikasi jika database atau server mengalami gangguan teknis.

### **Teknik**
* **Negative Testing:** Sengaja memberikan input yang salah untuk melihat mekanisme pertahanan sistem.
* **Fault Injection:** Menyuntikkan gangguan pada sistem untuk melihat seberapa cepat aplikasi melakukan pemulihan (recovery).

### **Kesimpulan**
Robustness Testing memberikan jaminan bahwa aplikasi adalah **produk yang matang** dan mampu bertahan dalam berbagai kondisi penggunaan yang tidak ideal.
