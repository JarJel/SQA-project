<div align="center">

<br/>

```
███╗   ███╗██╗██████╗ ███╗   ██╗██╗ ██████╗ ██╗  ██╗████████╗
████╗ ████║██║██╔══██╗████╗  ██║██║██╔════╝ ██║  ██║╚══██╔══╝
██╔████╔██║██║██║  ██║██╔██╗ ██║██║██║  ███╗███████║   ██║   
██║╚██╔╝██║██║██║  ██║██║╚██╗██║██║██║   ██║██╔══██║   ██║   
██║ ╚═╝ ██║██║██████╔╝██║ ╚████║██║╚██████╔╝██║  ██║   ██║   
╚═╝     ╚═╝╚═╝╚═════╝ ╚═╝  ╚═══╝╚═╝ ╚═════╝ ╚═╝  ╚═╝   ╚═╝  
                  F I N A N C E
```

### Software Quality Assurance Documentation
**Private Wealth Management Platform** · Laravel & React.js

<br/>

[![SRS](https://img.shields.io/badge/Docs-SRS-blue?style=for-the-badge&logo=bookstack&logoColor=white)](./Software%20Requirements%20Specification)
[![SDD](https://img.shields.io/badge/Docs-SDD-6366f1?style=for-the-badge&logo=blueprint&logoColor=white)](./Software%20Design%20Documentation)
[![Test Plan](https://img.shields.io/badge/Docs-Test%20Plan-22c55e?style=for-the-badge&logo=testinglibrary&logoColor=white)](./Test%20Plan)
[![User Docs](https://img.shields.io/badge/Docs-User%20Manual-f59e0b?style=for-the-badge&logo=gitbook&logoColor=white)](./Software%20User%20Documentation)

[![PHP](https://img.shields.io/badge/Backend-Laravel-FF2D20?style=flat-square&logo=laravel&logoColor=white)](https://laravel.com)
[![React](https://img.shields.io/badge/Frontend-React.js-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev)
[![MySQL](https://img.shields.io/badge/Database-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)](https://mysql.com)
[![License](https://img.shields.io/badge/License-Academic-gray?style=flat-square)](./LICENSE)
[![Status](https://img.shields.io/badge/Status-Active%20Testing-brightgreen?style=flat-square)]()

<br/>

> *"Quality is not an act, it is a habit."*
> — Aristotle

<br/>

</div>

---

## 📋 Table of Contents

- [📖 About The Project](#-about-the-project)
- [👥 Tim REMAKO](#-tim-remako)
- [📂 Repository Structure](#-repository-structure)
- [🎯 Scope of Testing](#-scope-of-testing)
- [🛠️ Testing Tools & Stack](#️-testing-tools--stack)
- [🚀 Getting Started](#-getting-started)
- [📊 Testing Progress](#-testing-progress)
- [📄 Documents Overview](#-documents-overview)

---

## 📖 About The Project

**Midnight Finance** adalah platform *Private Wealth Management* berbasis web yang dirancang untuk membantu pengguna mengelola seluruh aspek keuangan pribadi dalam satu ekosistem yang terintegrasi.

```
Portofolio & Aset  ─┐
Anggaran Bulanan   ─┤──▶  Midnight Finance  ──▶  Keputusan Keuangan Terstruktur
Hutang & Piutang   ─┤
Analitik Real-time ─┘
```

Repositori ini didedikasikan sepenuhnya untuk keperluan **Software Quality Assurance (SQA)** — mencakup dokumentasi rekayasa perangkat lunak, perencanaan pengujian sistematis, pelacakan defect, dan pelaporan hasil uji dari seluruh modul aplikasi.

| Atribut | Detail |
|---|---|
| **Platform** | Web Application |
| **Tech Stack** | Laravel (Backend API) · React.js (Frontend) · MySQL (Database) |
| **Metodologi** | Agile / Iterative Development |
| **Standar QA** | SDLC-based · Black-box & White-box Testing |
| **Institusi** | — |
| **Mata Kuliah** | Software Quality Assurance |

---

## 👥 Tim REMAKO

<table>
  <thead>
    <tr>
      <th align="center">NIM</th>
      <th>Nama</th>
      <th>Peran</th>
      <th>Fokus Kontribusi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><code>20231310025</code></td>
      <td><strong>Muhammad Fajar Munandar</strong></td>
      <td>🏆 Team Leader & Software Engineer</td>
      <td>Manajemen proyek, koordinasi tim, kolaborasi <em>codebase</em>, eksekusi <em>test case</em></td>
    </tr>
    <tr>
      <td align="center"><code>20231310012</code></td>
      <td><strong>Muhammad Dzaki Awaludin</strong></td>
      <td>⚙️ Lead Developer & QA Automation</td>
      <td>Rekayasa kode, implementasi fitur inti, penyusunan skrip pengujian otomatis</td>
    </tr>
    <tr>
      <td align="center"><code>20231310037</code></td>
      <td><strong>Mochamad Fikri Ghifari</strong></td>
      <td>🗂️ System Designer & Technical Writer</td>
      <td>Arsitektur sistem, pembuatan UML/ERD, dokumentasi teknis & desain</td>
    </tr>
    <tr>
      <td align="center"><code>20231310026</code></td>
      <td><strong>Raka Zilva Inggia</strong></td>
      <td>🔍 System Analyst & QA Analyst</td>
      <td>Analisis kebutuhan (<em>requirements</em>), pemodelan logika bisnis, validasi fungsionalitas</td>
    </tr>
  </tbody>
</table>

---

## 📂 Repository Structure

```
midnight-finance-sqa/
│
├── 📁 Software Design Documentation/    # Arsitektur & Desain Sistem
│   ├── 📁 Deskripsi Umum/
│   ├── 📁 Pendahuluan/
│   └── 📁 Software Design/              # ERD, UML, UI/UX, Flowchart
│
├── 📁 Software Requirements Specification/  # Spesifikasi Kebutuhan
│   ├── Use Case Diagram & Narasi
│   ├── Functional & Non-Functional Requirements
│   └── Business Rules & Constraints
│
├── 📁 Software User Documentation/      # Panduan Pengguna Akhir
│   └── User Manual (fitur per modul)
│
├── 📁 Test Plan/                        # Inti Aktivitas SQA
│   ├── 📄 Test Scenarios
│   ├── 📄 Test Cases
│   ├── 📄 Defect / Bug Report
│   └── 📄 Test Execution Result
│
├── 📄 231164_SystemDesignDocument....   # Dokumen SDD utama (PDF/DOCX)
└── 📄 README.md                         # Dokumen ini
```

---

## 🎯 Scope of Testing

Pengujian difokuskan pada **5 modul krusial** aplikasi Midnight Finance:

<table>
  <thead>
    <tr>
      <th>#</th>
      <th>Modul</th>
      <th>Fokus Pengujian</th>
      <th>Tingkat Kekritisan</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center">1</td>
      <td><strong>🔐 Autentikasi & Keamanan</strong></td>
      <td>Register, Login, OTP flow, pembatasan akses middleware, manajemen sesi</td>
      <td align="center">🔴 Critical</td>
    </tr>
    <tr>
      <td align="center">2</td>
      <td><strong>💳 Portofolio & Transaksi</strong></td>
      <td>Validasi input income/expense, transfer antar dompet, kalkulasi saldo akhir (<em>Core Banking Logic</em>)</td>
      <td align="center">🔴 Critical</td>
    </tr>
    <tr>
      <td align="center">3</td>
      <td><strong>📊 Anggaran Bulanan</strong></td>
      <td>Limit anggaran per kategori, kalkulasi persentase pemakaian, status: <em>Aman / Warning / Overbudget</em></td>
      <td align="center">🟠 High</td>
    </tr>
    <tr>
      <td align="center">4</td>
      <td><strong>📒 Hutang & Piutang</strong></td>
      <td>Sistem cicilan, penambahan bunga (persentase/fix), sinkronisasi otomatis ke saldo dompet</td>
      <td align="center">🟠 High</td>
    </tr>
    <tr>
      <td align="center">5</td>
      <td><strong>📈 Analitik Dashboard</strong></td>
      <td>Filter rentang waktu (7 Hari / 1 Bulan / All Time), akurasi render grafik (Line Chart & Pie Chart)</td>
      <td align="center">🟡 Medium</td>
    </tr>
  </tbody>
</table>

### Batasan Pengujian

> Pengujian **tidak mencakup** infrastruktur deployment (server, CDN, DNS), konfigurasi environment production, dan pengujian performa skala besar (*load testing*) di luar skenario yang terdefinisi.

---

## 🛠️ Testing Tools & Stack

### Backend & API Testing

| Tool | Kegunaan |
|---|---|
| **Postman / Insomnia** | Pengujian endpoint API (REST), validasi response, dan autentikasi token |
| **PHPUnit** | Unit testing & feature testing pada layer backend Laravel |
| **Laravel Sanctum** | Validasi mekanisme autentikasi Bearer Token |

### Frontend & UI Testing

| Tool | Kegunaan |
|---|---|
| **Cypress** | End-to-end (E2E) testing antarmuka React.js |
| **Jest + React Testing Library** | Unit testing komponen frontend |
| **Selenium / Katalon Studio** | Alternatif automation UI testing lintas browser |

### Manajemen & Pelaporan

| Tool | Kegunaan |
|---|---|
| **GitHub Issues** | Pelacakan bug dan defect secara terpusat |
| **GitHub Projects** | Manajemen sprint dan progres pengujian |
| **Markdown Reports** | Dokumentasi hasil test di dalam repositori ini |

---

## 🚀 Getting Started

Untuk mulai berkontribusi atau menelaah dokumentasi ini:

### 1. Clone Repositori

```bash
git clone https://github.com/<username>/midnight-finance-sqa.git
cd midnight-finance-sqa
```

### 2. Navigasi Dokumen

```bash
# Lihat spesifikasi kebutuhan
cd "Software Requirements Specification"

# Lihat rancangan sistem
cd "Software Design Documentation"

# Lihat test plan dan test cases
cd "Test Plan"
```

### 3. Konvensi Penulisan Dokumen

| Jenis Dokumen | Format | Konvensi Penamaan |
|---|---|---|
| Test Case | `.md` / `.xlsx` | `TC_[ModulCode]_[NomorUrut].md` |
| Bug Report | `.md` | `BUG_[ID]_[DeskripsiSingkat].md` |
| Test Result | `.md` | `RESULT_[Sprint]_[Tanggal].md` |
| UML Diagram | `.md` (Mermaid) | `[NomorBab]_[NamaDiagram].md` |

---

## 📊 Testing Progress

> Status pengujian diperbarui secara berkala oleh Tim REMAKO mengikuti sprint aktif.

| Modul | Test Cases | Passed | Failed | Pending |
|---|---|---|---|---|
| 🔐 Autentikasi | — | — | — | — |
| 💳 Transaksi | — | — | — | — |
| 📊 Anggaran | — | — | — | — |
| 📒 Hutang & Piutang | — | — | — | — |
| 📈 Analitik | — | — | — | — |
| **Total** | **—** | **—** | **—** | **—** |

*Tabel ini diperbarui setelah setiap siklus eksekusi pengujian.*

---

## 📄 Documents Overview

| Dokumen | Deskripsi | Lokasi |
|---|---|---|
| **Software Requirements Specification (SRS)** | Definisi fitur, *use case*, *business rules*, batasan sistem | [`/Software Requirements Specification`](./Software%20Requirements%20Specification) |
| **Software Design Documentation (SDD)** | ERD, arsitektur API, UML, desain UI/UX, *flowchart* | [`/Software Design Documentation`](./Software%20Design%20Documentation) |
| **Software User Documentation** | Panduan penggunaan fitur untuk *end-user* | [`/Software User Documentation`](./Software%20User%20Documentation) |
| **Test Plan** | Skenario uji, *test cases*, laporan bug, hasil eksekusi | [`/Test Plan`](./Test%20Plan) |
| **System Design Document (PDF)** | Dokumen SDD kompilasi lengkap | [`231164_SystemDesignDocument....`](./231164_SystemDesignDocument) |

---

<div align="center">

<br/>

**Tim REMAKO** · Software Quality Assurance

*Dokumen ini dikelola secara aktif selama siklus pengujian berlangsung.*

<br/>

[![Made with Markdown](https://img.shields.io/badge/Made%20with-Markdown-1f425f?style=flat-square&logo=markdown)](https://www.markdownguide.org)
[![Maintained](https://img.shields.io/badge/Maintained-Yes-brightgreen?style=flat-square)]()

</div>
