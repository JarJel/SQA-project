# Teknik Pengujian — Black Box Testing

Teknik ini digunakan untuk memvalidasi fungsionalitas aplikasi Midnight Finance dari sudut pandang pengguna akhir tanpa harus mengetahui struktur kode program.

| Aspek Pengujian | Deskripsi Teknis Pelaksanaan | Target Output & Validasi |
|:---|:---|:---|
| Fungsionalitas Utama | Menguji fitur utama: Login, Registrasi, Catat Transaksi, dan Dashboard | Semua endpoint merespons dengan HTTP status code yang benar dan data yang akurat |
| Validasi Input Data | Menguji input form registrasi (name, email, password) dan transaksi (amount, type, date) dengan data valid dan tidak valid | Sistem menolak input tidak valid dengan pesan error yang jelas sesuai aturan validasi Laravel |
| Boundary & Range | Menguji nilai batas pada field nominal transaksi (amount): nilai 0, nilai 1, nilai normal, nilai maksimum | Sistem menerima nilai dalam rentang valid dan menolak nilai di luar batas |
| Error Handling | Simulasi skenario error: email tidak terdaftar, password salah, token tidak ada, akun belum verifikasi | Sistem tidak crash dan menampilkan pesan error yang informatif dengan HTTP status code yang tepat |
| Keamanan Akses | Mengakses endpoint privat tanpa token Sanctum | Sistem mengembalikan HTTP 401 Unauthenticated — endpoint tidak dapat diakses tanpa autentikasi |
