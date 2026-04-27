## **Boundary Value Analysis (BVA)**### **Deskripsi**
Boundary Value Analysis adalah teknik pengujian yang berfokus pada **nilai-nilai batas** (tepi) dari rentang input yang diizinkan. Teknik ini didasarkan pada asumsi bahwa kesalahan lebih sering terjadi di ambang batas daripada di tengah rentang data.### **Tujuan**
* **Mendeteksi kesalahan logika:** Menemukan bug pada operator perbandingan (seperti penggunaan < yang seharusnya <=).
* **Efisiensi kasus uji:** Mengurangi jumlah pengujian dengan hanya mengambil titik-titik kritis yang paling rawan kesalahan.
* **Validasi ketelitian sistem:** Memastikan sistem tetap stabil saat menerima input tepat di angka minimum atau maksimum.### **Digunakan pada**
* **Input Numerik:** Mengecek batasan angka seperti umur, saldo minimal, atau jumlah stok barang.
* **Input Tanggal:** Menguji batas awal dan akhir bulan atau tahun dalam sistem reservasi.
* **Batasan Karakter:** Memastikan kolom teks (seperti password) memenuhi syarat panjang minimal dan maksimal.### **Teknik**
* **Nilai Batas (Boundary):** Mengambil nilai tepat di batas bawah dan batas atas (misal: 1 dan 100).
* **Nilai Luar Batas (Out of Bounds):** Menguji satu angka di bawah batas minimum (0) dan satu angka di atas batas maksimum (101).### **Kesimpulan**
Boundary Value Analysis membantu penguji menemukan **kesalahan logika pada transisi nilai** dengan cara yang sangat efektif tanpa harus mengetes seluruh rentang angka.
