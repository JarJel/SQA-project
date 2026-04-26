Teknik ini digunakan untuk memvalidasi fungsionalitas aplikasi dari sudut pandang pengguna akhir tanpa harus mengetahui struktur kode program.

| Aspek Pengujian        | Deskripsi Teknis Pelaksanaan                                                                 | Target Output & Validasi                                                                 |
|----------------------|---------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| Fungsionalitas Utama | Menguji fitur utama seperti Login, Keranjang, dan Checkout.                                 | Semua tombol berfungsi dan mengarah ke halaman yang benar (URL sesuai).                  |
| Validasi Input Data  | Menguji input (Nama, Email, Jumlah Pesanan) dengan data valid dan tidak valid.              | Sistem menolak input tidak valid dengan pesan error yang jelas.                          |
| User Experience (UX) | Menguji kemudahan navigasi dan alur pemesanan.                                              | User dapat menyelesaikan transaksi tanpa kebingungan.                                     |
| Error Handling       | Simulasi error seperti field kosong atau upload file besar.                                 | Sistem tidak crash dan menampilkan pesan error (contoh: "File maksimal 2MB").            |
| Compatibility UI     | Uji tampilan di berbagai device (responsive).                                               | UI tidak overlap di mobile/tablet.                                                        |

---
