# 🛡️ SQA Project: Midnight Finance

Selamat datang di repositori **Software Quality Assurance (SQA)** untuk proyek **Midnight Finance**. 

Repositori ini didedikasikan untuk menyimpan seluruh dokumentasi rekayasa perangkat lunak, perencanaan pengujian, dan laporan hasil uji coba dari aplikasi Midnight Finance—sebuah platform *Private Wealth Management* berbasis web (Laravel & React.js) yang dirancang untuk mengelola portofolio keuangan, anggaran, serta hutang-piutang secara komprehensif.

---

## 👥 Tim Pengembang & Penjamin Mutu (Tim REMAKO)

Proyek ini dikembangkan dan diuji oleh **Tim REMAKO**, dengan pembagian peran sebagai berikut:

- **Muhammad Fajar Munandar** (20231310025)
  *Team Leader & Software Engineer* — Memimpin manajemen proyek tim REMAKO, melakukan koordinasi tugas, serta fokus pada kolaborasi pengembangan *codebase* dan eksekusi *test case*.
- **Muhammad Dzaki Awaludin** (20231310012)
  *Lead Developer & QA Automation* — Fokus pada rekayasa kode, implementasi fitur inti, dan penyusunan skrip pengujian.
- **Mochamad Fikri Ghifari** (20231310037)
  *System Designer & Technical Documentation* — Fokus pada perancangan arsitektur sistem, pembuatan UML/ERD, dan dokumentasi desain.
- **Raka Zilva Inggia** (20231310026)
  *System Analyst & QA Analyst* — Fokus pada analisis kebutuhan (*requirements*), pemodelan bisnis logika, dan validasi fungsionalitas sistem.

---

## 📂 Struktur Repositori

Repositori ini diatur ke dalam beberapa direktori utama sesuai dengan siklus hidup pengembangan perangkat lunak (SDLC) dan standar SQA:

- **📁 `Software Requirements Specification` (SRS)**
  Berisi dokumen spesifikasi kebutuhan perangkat lunak. Mendefinisikan apa saja fitur yang harus ada, batasan sistem, *use case*, dan *business rule* (termasuk logika sistem bunga dan cicilan).
  
- **📁 `Software Design Documentation` (SDD)**
  Berisi rancangan arsitektur sistem. Meliputi *Entity Relationship Diagram* (ERD) database, arsitektur API, desain UI/UX, dan *flowchart* sistem kas.

- **📁 `Software User Documentation`**
  Berisi panduan penggunaan aplikasi (*User Manual*) untuk *end-user*, menjelaskan langkah demi langkah cara menggunakan fitur-fitur Midnight Finance.

- **📁 `Test Plan`**
  Ini adalah inti dari SQA. Direktori ini berisi:
  - Skenario Pengujian (*Test Scenarios*)
  - Kasus Uji (*Test Cases*)
  - Laporan Bug / *Defect Tracking*
  - Hasil Pengujian Otomatis & Manual

---

## 🎯 Ruang Lingkup Pengujian (System Under Test)

Pengujian pada proyek ini akan difokuskan pada modul-modul krusial dari aplikasi Midnight Finance, antara lain:

1. **Modul Autentikasi & Keamanan:** Register, Login, pembatasan akses (*middleware*), dan manajemen sesi pengguna.
2. **Modul Portofolio & Transaksi:** Validasi input pemasukan/pengeluaran, sinkronisasi antar dompet (transfer), dan perhitungan saldo akhir (*Core Banking Logic*).
3. **Modul Anggaran Bulanan (Budgets):** Pengujian limit anggaran, kalkulasi persentase pemakaian, dan status peringatan (Aman/Warning/Overbudget).
4. **Modul Buku Relasi (Debts & Receivables):** Pengujian sistem cicilan hutang/piutang, penambahan bunga (persentase/fix), dan sinkronisasi otomatis ke saldo dompet.
5. **Modul Analitik (Dashboard):** Pengujian filter rentang waktu (7 Hari, 1 Bulan, All Time) dan akurasi *render* grafik (*Line Chart & Pie Chart*).

---

## 🛠️ Alat Pengujian (Testing Tools)

*(Dapat disesuaikan dengan alat yang digunakan oleh Tim REMAKO)*
- **API Testing:** Postman / Insomnia
- **Automation UI Testing:** Cypress / Selenium / Katalon Studio
- **Unit/Feature Testing:** PHPUnit (Backend) & Jest/React Testing Library (Frontend)
- **Bug Tracking:** GitHub Issues / Trello / Jira

> *"Quality is not an act, it is a habit."* - Aristotle
