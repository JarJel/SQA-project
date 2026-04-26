Equivalence Partitioning
Deskripsi
Teknik untuk membagi data input menjadi beberapa kelompok (partisi) yang dianggap setara, di mana sistem diharapkan memberikan hasil yang sama untuk setiap data dalam kelompok tersebut.

Tujuan
Mengurangi jumlah test case dengan cara memilih satu perwakilan data dari setiap kelompok, sehingga pengujian lebih efisien namun tetap mencakup berbagai skenario.

Contoh
Jika form input hanya menerima angka 1 - 100, maka kita bagi menjadi 3 kelompok:

Data Tidak Valid (< 1)

Data Valid (1 - 100)

Data Tidak Valid (> 100)
Cukup ambil satu angka dari tiap kelompok (misal: 0, 50, dan 101).

Kesimpulan
Membantu penguji meng-cover seluruh kemungkinan jenis input tanpa harus mencoba setiap angka satu per satu.
