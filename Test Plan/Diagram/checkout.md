# Test Case: Order Checkout

**ID:** TC-ORD-001
**Fitur:** Proses Checkout dan Pemilihan Tipe Pesanan

| Step | Deskripsi Aksi | Input Data | Expected Result | Status |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Klik tombol "Checkout" dari halaman keranjang | Keranjang berisi 2 item | Diarahkan ke halaman konfirmasi pemesanan | Pass |
| 2 | Memilih tipe pesanan "Dine-in" | Radio button "Dine-in" | Sistem meminta input nomor meja | Pass |
| 3 | Memilih tipe pesanan "Take Away" | Radio button "Take Away" | Sistem tidak meminta nomor meja | Pass |
| 4 | Klik "Bayar Sekarang" tanpa memilih metode pembayaran | Belum pilih metode | Muncul pesan error "Harap pilih metode pembayaran" | Pass |
| 5 | Menekan tombol "Finalize Order" | Data lengkap | Data pesanan tersimpan di database dan status menjadi 'Pending Payment' | Pass |
