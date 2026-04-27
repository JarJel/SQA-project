
# 🛡️ Robustness Testing Documentation

---

## 📖 Overview

**Robustness Testing** adalah jenis pengujian non-fungsional yang bertujuan untuk mengevaluasi kemampuan sistem dalam menangani **input tidak valid, kondisi ekstrem, dan kesalahan tak terduga** tanpa mengalami crash atau kegagalan sistem.

Dalam konteks **Midnight Finance**, robustness testing sangat penting untuk memastikan bahwa sistem tetap **aman, stabil, dan dapat diandalkan**, bahkan ketika menerima data yang tidak sesuai atau terjadi gangguan sistem.

---

## 🎯 Objectives

Tujuan utama dari Robustness Testing:

- Menguji sistem terhadap **input invalid / abnormal**
- Memastikan sistem tidak mengalami **crash**
- Memvalidasi **error handling & validation**
- Menguji ketahanan terhadap **unexpected user behavior**
- Menjamin sistem tetap memberikan **response yang aman**

---

## 🧪 Scope of Testing

Pengujian difokuskan pada modul kritikal:

| Modul | Fokus Pengujian |
|---|---|
| 🔐 Autentikasi | Input login invalid, SQL injection, brute force |
| 💳 Transaksi | Input nominal negatif, format salah |
| 📊 Anggaran | Nilai limit tidak logis |
| 📒 Hutang & Piutang | Data cicilan tidak valid |
| 📈 Dashboard | Parameter filter tidak valid |

---

## ⚙️ Testing Environment

| Komponen | Spesifikasi |
|---|---|
| Backend | Laravel API |
| Frontend | React.js |
| Database | MySQL |
| Tools | Postman / Burp Suite / OWASP ZAP |
| Server | Localhost / Staging |

---

## 🛠️ Testing Tools

| Tools | Fungsi |
|---|---|
| **Postman** | Uji API dengan berbagai input |
| **Burp Suite** | Manipulasi request & security testing |
| **OWASP ZAP** | Scan vulnerability |
| **Browser DevTools** | Uji manipulasi frontend |

---

## 📋 Test Scenarios

### 📌 Scenario 1: Invalid Login Input

- **Input**:
  - Email kosong
  - Password salah
  - Format email tidak valid
- **Expected Result**:
  - Sistem menolak request
  - Menampilkan error message yang jelas
  - Tidak crash

---

### 📌 Scenario 2: SQL Injection Attempt

- **Input**:
```sql
' OR 1=1 --
