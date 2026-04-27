# Control Flow Testing

**Definisi:**
Strategi pengujian White Box yang berfokus pada urutan eksekusi instruksi dalam program melalui struktur kontrol (percabangan dan perulangan).

**Contoh Struktur Kontrol:**
```python
def cek_status(nilai):
    if nilai >= 75: # Branch 1
        return "Lulus"
    else:           # Branch 2
        return "Gagal"
