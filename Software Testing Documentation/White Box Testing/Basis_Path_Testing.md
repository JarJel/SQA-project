# Basis Path Testing

## Definisi

**Basis Path Testing** adalah teknik *White Box Testing* yang diusulkan oleh **Tom McCabe**. Teknik ini memungkinkan tester untuk:

- Mengukur kompleksitas logis dari suatu kode program.
- Menggunakan pengukuran tersebut sebagai panduan untuk menentukan kumpulan jalur eksekusi dasar (*basis set*) yang harus diuji.

---

## Metode: Cyclomatic Complexity

**Cyclomatic Complexity** adalah metrik yang menentukan **jumlah jalur independen minimum** yang menjamin setiap baris perintah dieksekusi setidaknya satu kali.

### Rumus Perhitungan $V(G)$

Terdapat tiga cara untuk menghitung kompleksitas $V(G)$:

| No. | Pendekatan | Rumus | Keterangan |
|-----|------------|-------|------------|
| 1 | **Berdasarkan Region** | $V(G) = R$ | $R$ = jumlah region (wilayah tertutup) pada Control Flow Graph |
| 2 | **Berdasarkan Edges & Nodes** | $V(G) = E - N + 2$ | $E$ = jumlah edge, $N$ = jumlah node |
| 3 | **Berdasarkan Predicate Nodes** | $V(G) = P + 1$ | $P$ = jumlah titik keputusan (`if`, `while`, `for`) |

> **Catatan:** Ketiga rumus di atas akan selalu menghasilkan nilai yang sama jika diterapkan pada CFG yang sama.

---

## Langkah-Langkah Implementasi

### 1. Gambar Control Flow Graph (CFG)

Buat representasi grafik dari alur eksekusi kode sumber:

- Setiap **node** merepresentasikan satu atau beberapa pernyataan (*statement*) berurutan.
- Setiap **edge** merepresentasikan alur kontrol antar node.
- Tandai semua **predicate node** (titik keputusan/percabangan).

### 2. Hitung Cyclomatic Complexity

Gunakan salah satu dari tiga rumus di atas. Nilai $V(G)$ yang dihasilkan menentukan **jumlah test case minimum** yang diperlukan.

Interpretasi nilai $V(G)$:

| Nilai $V(G)$ | Tingkat Risiko |
|:---:|---|
| 1 – 10 | Sederhana, risiko rendah |
| 11 – 20 | Moderat, risiko sedang |
| 21 – 50 | Kompleks, risiko tinggi |
| > 50 | Sangat kompleks, tidak stabil |

### 3. Tentukan Independent Paths

Identifikasi semua **jalur independen** (*basis set*), yaitu jalur yang memperkenalkan setidaknya satu edge baru yang belum dilalui jalur sebelumnya.

**Contoh notasi jalur:**

```
Path 1: Node 1 → Node 2 → Node 4 → Node 6 (END)
Path 2: Node 1 → Node 2 → Node 3 → Node 5 → Node 6 (END)
Path 3: Node 1 → Node 2 → Node 4 → Node 5 → Node 6 (END)
```

### 4. Buat Test Case

Untuk setiap jalur independen, rancang test case yang memaksa eksekusi melewati jalur tersebut:

| Jalur | Input / Kondisi | Output yang Diharapkan |
|-------|-----------------|------------------------|
| Path 1 | ... | ... |
| Path 2 | ... | ... |
| Path N | ... | ... |

---

## Contoh Sederhana

### Kode Sumber

```python
def cek_nilai(nilai):        # Node 1
    if nilai >= 75:          # Node 2 (predicate)
        print("Lulus")       # Node 3
    else:
        print("Tidak Lulus") # Node 4
    print("Selesai")         # Node 5
```

### Control Flow Graph

```
       [Node 1] start
           |
       [Node 2] nilai >= 75?
       /         \
  Ya /           \ Tidak
    /             \
[Node 3]       [Node 4]
"Lulus"     "Tidak Lulus"
    \             /
     \           /
      [Node 5] end
```

### Perhitungan

- **Edges (E):** 4 &nbsp;|&nbsp; **Nodes (N):** 4 &nbsp;|&nbsp; **Predicate (P):** 1

$$V(G) = E - N + 2 = 4 - 4 + 2 = \mathbf{2}$$

$$V(G) = P + 1 = 1 + 1 = \mathbf{2}$$

### Independent Paths & Test Case

| Jalur | Kondisi | Input | Output |
|-------|---------|-------|--------|
| Path 1 | `nilai >= 75` bernilai **True** | `nilai = 80` | `"Lulus"`, `"Selesai"` |
| Path 2 | `nilai >= 75` bernilai **False** | `nilai = 60` | `"Tidak Lulus"`, `"Selesai"` |

---

## Tujuan

> Memastikan cakupan pengujian (*test coverage*) mencapai **maksimal** dan meminimalkan risiko adanya jalur logika yang tidak pernah dieksekusi (*untested paths*).

### Manfaat Utama

- ✅ Menjamin setiap baris kode dieksekusi minimal satu kali.
- ✅ Memberikan dasar yang terukur dalam menentukan jumlah test case.
- ✅ Membantu mendeteksi kesalahan logika yang tersembunyi di cabang-cabang tertentu.
- ✅ Meningkatkan kualitas dan kepercayaan terhadap perangkat lunak.

### Keterbatasan

- ❌ Tidak menjamin pengujian semua kombinasi kondisi (perlu teknik lain seperti *Condition Coverage*).
- ❌ Kompleksitas tinggi pada program besar membutuhkan banyak test case.
- ❌ Tidak mendeteksi kesalahan yang berhubungan dengan data (*data flow errors*) secara langsung.

---

## Ringkasan

```
Kode Sumber
    ↓
Control Flow Graph (CFG)
    ↓
Cyclomatic Complexity V(G)
    ↓
Independent Paths (basis set)
    ↓
Test Cases
    ↓
Eksekusi & Validasi
```

---

*Referensi: McCabe, T.J. (1976). "A Complexity Measure". IEEE Transactions on Software Engineering.*
