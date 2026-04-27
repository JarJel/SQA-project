Boundary Value Analysis (BVA)
Deskripsi
Boundary Value Analysis adalah teknik pengujian yang berfokus pada nilai-nilai batas dari rentang input yang diizinkan. Teknik ini didasarkan pada asumsi bahwa kesalahan lebih sering terjadi di tepi rentang (batas) daripada di tengah-tengahnya.

Tujuan

Mendeteksi Kesalahan Operator: Menemukan kesalahan logika seperti penggunaan < padahal seharusnya <=.

Efisiensi Test Case: Meminimalkan jumlah pengujian dengan hanya mengambil nilai di titik kritis yang paling rawan bug.

Validasi Batas Sistem: Memastikan sistem tetap stabil saat menerima input tepat di batas maksimum atau minimum.

Digunakan pada

Input Numerik: Mengecek batasan angka seperti umur, saldo, atau jumlah stok barang.

Input Tanggal: Menguji batas awal dan akhir bulan atau tahun dalam sistem reservasi.

Ukuran File: Memastikan sistem menolak atau menerima file tepat pada batas ukuran yang ditentukan (misal: 2MB).

Teknik

Minimum & Maximum: Mengambil nilai tepat di batas bawah dan batas atas (misal: 1 dan 100).

Just Below/Above: Menguji satu angka di bawah dan di atas batas (misal: 0, 2, 99, 101) untuk melihat respon sistem terhadap data tidak valid.

Kesimpulan
Boundary Value Analysis membantu penguji menemukan kesalahan logika pada transisi nilai dengan cara yang sangat efektif dan terukur.
