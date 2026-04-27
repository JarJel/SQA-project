# Data Flow Testing

**Definisi:**
Pengujian yang fokus pada siklus hidup variabel, mulai dari saat variabel didefinisikan, digunakan, hingga dihapus dari memori.

**Status Variabel yang Diuji:**
| Kode | Status | Penjelasan |
|------|--------|------------|
| **d** | Defined | Variabel diberi nilai awal. |
| **u** | Used | Variabel digunakan dalam kalkulasi/logika. |
| **k** | Killed | Variabel dihapus atau keluar dari scope. |

**Tujuan:** Mendeteksi variabel yang didefinisikan tapi tidak pernah digunakan (*Dead Code*).
