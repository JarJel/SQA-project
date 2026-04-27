# Basis Path Testing

**Definisi:**
Teknik White Box testing yang diusulkan oleh Tom McCabe. Teknik ini memungkinkan tester untuk mengukur kompleksitas logis dari suatu kode program dan menggunakan pengukuran ini sebagai panduan untuk menentukan kumpulan jalur eksekusi dasar (*basis set*) yang harus diuji.

**Metode: Cyclomatic Complexity**
Metrik ini menentukan jumlah jalur independen minimum yang menjamin setiap baris perintah dieksekusi setidaknya satu kali.

**Rumus Perhitungan:**
Terdapat tiga cara untuk menghitung kompleksitas $V(G)$:

1. **Berdasarkan Region:** Jumlah region (wilayah tertutup) pada Control Flow Graph.
2. **Berdasarkan Edges & Nodes:** $$V(G) = E - N + 2$$
   *Dimana $E$ = jumlah edge, $N$ = jumlah node.*
3. **Berdasarkan Predicate Nodes:**
   $$V(G) = P + 1$$
   *Dimana $P$ = jumlah titik keputusan (if, while, for).*

**Langkah Implementasi:**
1. Gambar **Control Flow Graph (CFG)** dari kode sumber.
2. Hitung nilai **Cyclomatic Complexity**.
3. Tentukan **Independent Paths** (jalur-jalur yang berbeda).
4. Buat **Test Case** untuk setiap jalur tersebut.

**Tujuan:**
Memastikan cakupan pengujian (*test coverage*) mencapai maksimal dan meminimalkan risiko adanya jalur logika yang tidak pernah dieksekusi (untested paths).
