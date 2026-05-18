# 🔬 White Box Testing — Midnight Finance

**Mata Kuliah:** Software Quality Assurance **Pertemuan:** 10 — White Box Testing **Tim:** REMACode **Sistem yang Diuji:** Midnight Finance (Private Wealth Management Platform) — Laravel 11 \+ React.js

---

## 📖 Apa itu White Box Testing?

**White Box Testing** adalah metode pengujian perangkat lunak yang **memeriksa dan menganalisis struktur internal kode program** untuk menemukan kesalahan atau kekurangan dalam aplikasi (Ndaumanu, 2023). Berbeda dengan Black Box Testing yang hanya melihat input-output, White Box Testing membutuhkan pemahaman penuh terhadap *source code*, alur logika, dan struktur internal modul (Andriyadi et al., 2022).

### Karakteristik Utama

| Aspek | Penjelasan |
| :---- | :---- |
| **Akses Kode** | Penguji memiliki akses penuh ke source code |
| **Fokus** | Struktur internal, alur logika, jalur eksekusi |
| **Level Pengujian** | Unit, Integration, dan sebagian System Testing |
| **Pelaku** | Developer atau QA Engineer dengan kemampuan coding |
| **Tujuan** | Memvalidasi logika, menemukan dead code, memastikan coverage |

---

## 🎯 Tujuan Dokumentasi

Dokumentasi ini berisi **7 model White Box Testing** yang diterapkan pada modul-modul kritis sistem **Midnight Finance**. Setiap model menjelaskan:

- ✅ Definisi formal & konsep dasar  
- ✅ Tujuan pengujian  
- ✅ Contoh kode Laravel dari modul Midnight Finance  
- ✅ Diagram alur (Mermaid)  
- ✅ Tabel test case & hasil eksekusi  
- ✅ Kelebihan & kekurangan  
- ✅ Tools yang relevan  
- ✅ Analisis hasil pengujian

---

## 📂 Daftar Model Pengujian

| \# | Model | File | Modul Target | Tingkat Kompleksitas |
| :---- | :---- | :---- | :---- | :---- |
| 1 | **Desk Checking** | [`Desk_Checking.md`](http://./Desk_Checking.md) | Kalkulasi Saldo Transaksi | 🟢 Low |
| 2 | **Code Walkthrough** | [`Code_Walkthrough.md`](http://./Code_Walkthrough.md) | Autentikasi & Login Flow | 🟢 Low |
| 3 | **Formal Inspection** | [`Formal_Inspection.md`](http://./Formal_Inspection.md) | Form Input Transaksi | 🟡 Medium |
| 4 | **Control Flow Testing** | [`Control_Flow_Testing.md`](http://./Control_Flow_Testing.md) | Validasi Limit Anggaran | 🟡 Medium |
| 5 | **Basis Path Testing** | [`Basis_Path_Testing.md`](http://./Basis_Path_Testing.md) | Login \+ Role Check | 🔴 High |
| 6 | **Data Flow Testing** | [`Data_Flow_Testing.md`](http://./Data_Flow_Testing.md) | Transfer Antar Wallet | 🔴 High |
| 7 | **Loop Testing** | [`Loop_Testing.md`](http://./Loop_Testing.md) | Kalkulasi Cicilan Hutang | 🟡 Medium |

---

## 🗺️ Klasifikasi Model

graph TD

    A\[White Box Testing\] \--\> B\[Static Testing\]

    A \--\> C\[Dynamic Testing\]

    B \--\> B1\[Desk Checking\]

    B \--\> B2\[Code Walkthrough\]

    B \--\> B3\[Formal Inspection\]

    C \--\> C1\[Control Flow Testing\]

    C \--\> C2\[Basis Path Testing\]

    C \--\> C3\[Data Flow Testing\]

    C \--\> C4\[Loop Testing\]

    style A fill:\#1e293b,stroke:\#3b82f6,color:\#fff

    style B fill:\#0f766e,stroke:\#14b8a6,color:\#fff

    style C fill:\#7c2d12,stroke:\#ea580c,color:\#fff

**Static Testing** → dilakukan tanpa menjalankan kode (review manual). **Dynamic Testing** → dilakukan dengan mengeksekusi kode menggunakan test case.

---

## 📊 Tabel Perbandingan Singkat

| Model | Tipe | Fokus Utama | Output |
| :---- | :---- | :---- | :---- |
| Desk Checking | Static | Logika & nilai variabel | Tabel trace variabel |
| Code Walkthrough | Static | Review kode bersama tim | Checklist issue |
| Formal Inspection | Static | Kelengkapan & format data | Laporan defect |
| Control Flow | Dynamic | Jalur eksekusi kontrol | Tabel kondisi vs hasil |
| Basis Path | Dynamic | Independent paths | Cyclomatic Complexity \+ path |
| Data Flow | Dynamic | Siklus hidup variabel | Define-Use chain |
| Loop Testing | Dynamic | Iterasi & batas loop | Tabel boundary cases |

---

## 🛠️ Tools yang Digunakan

| Kategori | Tool | Kegunaan |
| :---- | :---- | :---- |
| **Unit Testing** | PHPUnit | Testing function/method Laravel |
| **Static Analysis** | PHPStan, Larastan | Deteksi error sebelum runtime |
| **Code Coverage** | Xdebug \+ PHPUnit Coverage | Mengukur statement/branch coverage |
| **Code Quality** | PHP\_CodeSniffer, PHP Mess Detector | Code style & complexity |
| **Flowchart** | Mermaid, draw.io | Visualisasi control flow graph |
| **Manual Review** | GitHub PR Review | Code walkthrough & inspection |

---

## 🚀 Cara Membaca Dokumentasi

1. **Mulai dari file ini** untuk memahami big picture  
2. **Buka file per model** sesuai urutan tabel di atas (low → high complexity)  
3. **Jalankan test case** menggunakan PHPUnit pada repo backend  
4. **Verifikasi hasil** dengan tabel ekspektasi yang disediakan

\# Menjalankan unit test (dari root midnight-finance-backend)

php artisan test

\# Dengan coverage report

php artisan test \--coverage

---

## 📚 Referensi

1. Ndaumanu, R. I. (2023). *Pengujian Sistem Informasi Perpustakaan Berbasis Website dengan Basis Path Testing*. Justek: Jurnal Sains dan Teknologi.  
2. Andriyadi, A., Zulkarnaini, Fikri, R. R. N., & Saputri, E. F. (2022). *Evaluasi Sistem Informasi Perpustakaan Institut Informatika Darmajaya Dengan WhiteBox Testing*. Journal of Innovation Research and Knowledge.  
3. Pressman, R. S., & Maxim, B. R. (2020). *Software Engineering: A Practitioner's Approach* (9th ed.). McGraw-Hill.  
4. McCabe, T. J. (1976). *A Complexity Measure*. IEEE Transactions on Software Engineering.  
5. Suprihadi, D. (2025). *Materi Software Quality Pertemuan 10 — White Box Testing*. Universitas Kristen Indonesia.

---

**Dokumentasi disusun oleh Tim REMACode**

*"Quality is not an act, it is a habit." — Aristotle*  
