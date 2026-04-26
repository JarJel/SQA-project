## **Black Box Testing**

### **Deskripsi**
Black Box Testing adalah metode pengujian perangkat lunak yang berfokus pada **fungsionalitas aplikasi** tanpa harus mengetahui struktur internal atau detail implementasi kode programnya. Penguji hanya melihat input yang diberikan dan output yang dihasilkan.

### **Tujuan**
* **Memastikan fitur berjalan sesuai kebutuhan:** Memverifikasi apakah sistem bekerja sesuai dengan spesifikasi fungsional (SRS).
* **Menemukan kesalahan antarmuka (UI/UX):** Mendeteksi kesalahan pada tampilan, navigasi, atau interaksi pengguna.
* **Validasi input dan output:** Memastikan sistem menerima data yang benar dan menolak data yang salah dengan pesan error yang tepat.

---

### **Digunakan pada**
* **Pengujian Antarmuka (UI):** Mengecek tombol, menu, dan tata letak.
* **Input Pengguna:** Mengetes form pendaftaran, pencarian, atau pengunggahan file.
* **Alur Bisnis (End-to-End):** Memastikan proses dari awal hingga akhir (seperti belanja online hingga pembayaran) berjalan mulus.

---

### **Teknik**
* **Equivalence Partitioning:** Membagi input menjadi kelompok-kelompok data untuk menguji perwakilan dari setiap kelompok.
* **Boundary Value Analysis:** Fokus pada pengujian nilai batas (misal: jika batas umur 17-60, maka tes di angka 16, 17, 60, dan 61).
* **Decision Table Testing:** Menguji kombinasi input yang berbeda untuk melihat hasil yang dihasilkan.
* **State Transition Testing:** Menguji perubahan status aplikasi saat diberikan input tertentu.
* **Error Guessing:** Menebak di mana letak kesalahan berdasarkan pengalaman penguji.

---

### **Kesimpulan**
Black Box Testing membantu memastikan bahwa aplikasi **mudah digunakan** dan **berfungsi sebagaimana mestinya** dari sudut pandang pengguna akhir (user).
