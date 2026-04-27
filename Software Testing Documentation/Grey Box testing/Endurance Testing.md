
# ⏱️ Endurance Testing Documentation

---

## 📖 Overview

**Endurance Testing** (juga dikenal sebagai *Soak Testing*) adalah jenis pengujian non-fungsional yang bertujuan untuk memastikan bahwa sistem mampu beroperasi secara stabil dalam jangka waktu yang panjang tanpa mengalami degradasi performa, memory leak, atau kegagalan sistem.

Pada konteks **Midnight Finance**, pengujian ini sangat penting karena aplikasi menangani data finansial yang harus selalu tersedia, konsisten, dan real-time.

---

## 🎯 Objectives

Tujuan utama dari Endurance Testing:

- Mengidentifikasi **memory leak** pada backend (Laravel API)
- Menguji **stabilitas sistem** dalam durasi panjang
- Memastikan tidak terjadi **penurunan performa signifikan**
- Mengamati **resource usage** (CPU, RAM, Database connection)
- Memvalidasi kemampuan sistem dalam menangani **request berulang dalam jangka panjang**

---

## 🧪 Scope of Testing

Endurance Testing difokuskan pada modul kritikal berikut:

| Modul | Fokus Pengujian |
|---|---|
| 🔐 Autentikasi | Konsistensi login/logout dalam waktu lama |
| 💳 Transaksi | Stabilitas input transaksi berulang |
| 📊 Anggaran | Perhitungan terus-menerus tanpa error |
| 📒 Hutang & Piutang | Sinkronisasi data berkala |
| 📈 Dashboard | Render data real-time secara terus-menerus |

---

## ⚙️ Testing Environment

| Komponen | Spesifikasi |
|---|---|
| Backend | Laravel API |
| Frontend | React.js |
| Database | MySQL |
| Tools | Apache JMeter / K6 / Locust |
| OS | Windows / Linux |
| Network | Localhost / Staging Server |

---

## 🛠️ Testing Tools

| Tools | Fungsi |
|---|---|
| **Apache JMeter** | Simulasi request berulang dalam durasi panjang |
| **K6** | Load testing berbasis script modern |
| **New Relic / Grafana** | Monitoring performa & resource |
| **MySQL Monitor** | Observasi query dan koneksi database |

---

## 📋 Test Scenario

### 📌 Scenario 1: Continuous Login Request

- **Deskripsi**: Simulasi login user secara terus-menerus
- **Durasi**: 2 - 6 jam
- **Expected Result**:
  - Tidak terjadi crash
  - Response time stabil
  - Tidak ada memory leak

---

### 📌 Scenario 2: Repeated Transaction Input

- **Deskripsi**: Input transaksi (income & expense) secara berulang
- **Durasi**: 4 jam
- **Expected Result**:
  - Saldo tetap konsisten
  - Tidak ada duplicate data
  - Database tetap stabil

---

### 📌 Scenario 3: Dashboard Auto Refresh

- **Deskripsi**: Dashboard melakukan refresh data setiap beberapa detik
- **Durasi**: 3 jam
- **Expected Result**:
  - Grafik tetap akurat
  - Tidak terjadi lag signifikan
  - API response tetap stabil

---

## 📊 Metrics & Monitoring

Parameter yang diamati selama pengujian:

| Metric | Deskripsi |
|---|---|
| Response Time | Waktu respon API |
| Throughput | Jumlah request per detik |
| Error Rate | Persentase error |
| CPU Usage | Penggunaan CPU server |
| Memory Usage | Konsumsi RAM |
| DB Connections | Jumlah koneksi database aktif |

---

## ⚠️ Potential Risks

Beberapa risiko yang mungkin ditemukan:

- Memory leak pada Laravel service
- Koneksi database tidak tertutup (connection leak)
- Penurunan performa setelah durasi tertentu
- Crash akibat resource exhaustion
- Cache yang tidak ter-clear

---

## ✅ Exit Criteria

Pengujian dinyatakan **berhasil** jika:

- Sistem berjalan stabil selama durasi pengujian
- Tidak ada peningkatan error rate signifikan
- Resource usage tetap dalam batas normal
- Tidak ditemukan memory leak

---

## 📄 Sample Result (Template)

```md
### Endurance Test Result

- Durasi: 4 Jam
- Total Request: 120,000
- Average Response Time: 210 ms
- Error Rate: 0.5%
- CPU Usage: Stabil (40% - 60%)
- Memory Usage: Stabil (tidak meningkat signifikan)

**Kesimpulan:**
Sistem stabil dan tidak ditemukan indikasi memory leak.
