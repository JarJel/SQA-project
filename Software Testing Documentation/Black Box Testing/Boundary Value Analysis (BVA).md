Boundary Value Analysis (BVA)
Deskripsi
Teknik pengujian yang berfokus pada nilai batas atau titik transisi di mana perilaku sistem biasanya berubah.

Tujuan
Memastikan sistem menangani kondisi batas dengan benar, karena bug sering kali muncul di angka-angka tepi (ujung).

Contoh
Jika sistem menerima nilai 1 - 100, maka titik kritis yang diuji adalah:

Batas bawah: 0 (luar), 1 (tepat), 2 (dalam).

Batas atas: 99 (dalam), 100 (tepat), 101 (luar).

Kesimpulan
Meminimalkan risiko error pada logika perbandingan (seperti salah menggunakan operator < yang seharusnya <=).
