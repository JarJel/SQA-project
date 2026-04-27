## **Equivalence Partitioning**
### **Deskripsi**
Equivalence Partitioning adalah teknik membagi atau **mempartisi input data** ke dalam beberapa kelompok yang dianggap setara. Sistem diasumsikan akan memberikan respon yang sama untuk setiap data di dalam satu kelompok tersebut.
### **Tujuan**
* **Reduksi jumlah pengujian:** Menghindari pengetesan data yang berulang-ulang untuk kategori yang memiliki perilaku serupa.
* **Cakupan luas:** Memastikan setiap kategori input (baik yang diterima maupun ditolak) telah diwakili dalam pengujian.
* **Optimasi waktu:** Mempercepat proses QA dengan hanya mengambil perwakilan data dari setiap partisi.

### **Digunakan pada**
* **Kategori Nilai:** Mengelompokkan rentang nilai ujian menjadi grade (misal: 80-100 adalah "A").
* **Pilihan Dropdown:** Menguji perwakilan dari daftar negara, jenis kelamin, atau kategori produk.
* **Validasi Tipe Data:** Memisahkan kelompok input yang hanya boleh berisi angka, huruf, atau simbol.

### **Teknik**
* **Valid Partition:** Memilih satu perwakilan dari kelompok data yang seharusnya diterima oleh sistem.
* **Invalid Partition:** Memilih satu perwakilan dari kelompok data yang seharusnya ditolak oleh sistem.

### **Kesimpulan**
Equivalence Partitioning memungkinkan **pengujian yang komprehensif** dengan jumlah kasus uji yang minimal melalui klasifikasi data yang cerdas.
