# Teknik Pengujian — Gray Box Testing

Gray Box Testing menggabungkan Black Box dan White Box dengan fokus pada integrasi sistem. Penguji memiliki pengetahuan parsial tentang struktur internal aplikasi, cukup untuk merancang test case yang lebih cerdas dari Black Box murni.

| Aspek Pengujian | Deskripsi Teknis Pelaksanaan | Target Output & Validasi |
|:---|:---|:---|
| Orthogonal Array Testing | Menguji kombinasi parameter filter transaksi (`type`, `sort_by`, `sort_order`) dengan array ortogonal L6 | Semua kombinasi menghasilkan data yang benar dan konsisten |
| Matrix Testing | Menguji semua kombinasi matriks parameter filter (`type`, `start_date`, `category_id`) pada GET /api/transactions | Tidak ada bug interaksi antar parameter |
| Regression Testing | Menguji sinkronisasi saldo otomatis setelah operasi CRUD transaksi (store, update, destroy) | Saldo `FinancialAccount` selalu diperbarui dengan benar, tidak ada regresi |
| Pattern Testing | Eksplorasi edge case pada endpoint Dashboard dan Analytics (user baru, tanpa wallet, zero-fill chart) | Sistem stabil, JSON konsisten, tidak ada crash pada skenario tidak terduga |
