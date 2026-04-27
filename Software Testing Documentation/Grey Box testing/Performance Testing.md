
# 🚀 Performance Testing Documentation

---

## 📖 Overview

**Performance Testing** adalah jenis pengujian non-fungsional yang bertujuan untuk mengevaluasi **kecepatan, stabilitas, dan skalabilitas** sistem dalam berbagai kondisi beban (*load*).

Dalam sistem **Midnight Finance**, performance testing dilakukan untuk memastikan bahwa aplikasi mampu menangani banyak pengguna secara bersamaan tanpa mengalami penurunan performa yang signifikan, terutama pada modul keuangan yang bersifat real-time.

---

## 🎯 Objectives

Tujuan utama dari Performance Testing:

- Mengukur **response time** setiap endpoint API
- Menentukan **batas maksimum kapasitas sistem (throughput)**
- Mengidentifikasi **bottleneck** pada sistem
- Menguji **stabilitas sistem pada berbagai tingkat beban**
- Memastikan sistem tetap responsif saat digunakan banyak user

---

## 🧪 Scope of Testing

Pengujian difokuskan pada modul utama berikut:

| Modul | Fokus Pengujian |
|---|---|
| 🔐 Autentikasi | Response login & register |
| 💳 Transaksi | Insert & kalkulasi data finansial |
| 📊 Anggaran | Perhitungan limit & persentase |
| 📒 Hutang & Piutang | Proses cicilan & update saldo |
| 📈 Dashboard | Load data grafik & filter |

---

## ⚙️ Testing Environment

| Komponen | Spesifikasi |
|---|---|
| Backend | Laravel API |
| Frontend | React.js |
| Database | MySQL |
| Tools | Apache JMeter / K6 / Locust |
| Server | Localhost / Staging |
| Network | Stable Connection |

---

## 🛠️ Testing Tools

| Tools | Fungsi |
|---|---|
| **Apache JMeter** | Simulasi beban tinggi (multi-user) |
| **K6** | Script-based performance testing |
| **Postman Runner** | Testing sederhana API |
| **Grafana / Prometheus** | Monitoring performa |
| **Chrome DevTools** | Analisis performa frontend |

---

## 📋 Test Types

### 1. 🔹 Load Testing
Mengukur performa sistem pada **beban normal hingga tinggi**

### 2. 🔹 Stress Testing
Menguji sistem hingga melewati batas kapasitas

### 3. 🔹 Spike Testing
Menguji lonjakan user secara tiba-tiba

### 4. 🔹 Volume Testing
Menguji performa dengan data dalam jumlah besar

---

## 📌 Test Scenarios

### Scenario 1: Login Load Test

- **User Virtual**: 100 - 500 user
- **Endpoint**: `/api/login`
- **Expected Result**:
  - Response time < 500 ms
  - Error rate < 1%

---

### Scenario 2: Transaction Processing

- **User Virtual**: 200 user
- **Endpoint**: `/api/transactions`
- **Expected Result**:
  - Tidak ada data corrupt
  - Response stabil
  - Perhitungan saldo akurat

---

### Scenario 3: Dashboard Data Load

- **User Virtual**: 150 user
- **Endpoint**: `/api/dashboard`
- **Expected Result**:
  - Data tampil < 1 detik
  - Grafik tidak error

---

### Scenario 4: Spike Test (Traffic Surge)

- **User Virtual**: 0 → 500 user dalam 10 detik
- **Expected Result**:
  - Sistem tetap berjalan
  - Tidak crash
  - Recovery cepat

---

## 📊 Metrics & KPI

| Metric | Target |
|---|---|
| Response Time | < 500 ms |
| Throughput | ≥ 100 request/sec |
| Error Rate | < 1% |
| CPU Usage | < 70% |
| Memory Usage | Stabil |
| Latency | Minimal |

---

## ⚠️ Potential Bottlenecks

Beberapa potensi masalah:

- Query database lambat (N+1 problem)
- API tidak optimal (loop berat)
- Kurangnya caching
- Overload pada server
- Inefficient state management di frontend

---

## ✅ Exit Criteria

Pengujian dianggap berhasil jika:

- Semua endpoint memiliki response time sesuai target
- Error rate dalam batas toleransi
- Sistem tetap stabil pada beban tinggi
- Tidak terjadi crash atau downtime

---

## 📄 Sample Result (Template)

```md
### Performance Test Result

- Total Virtual Users: 300
- Total Request: 50,000
- Average Response Time: 320 ms
- Max Response Time: 900 ms
- Throughput: 120 request/sec
- Error Rate: 0.8%

**Kesimpulan:**
Sistem mampu menangani beban tinggi dengan performa yang masih dalam batas normal.
