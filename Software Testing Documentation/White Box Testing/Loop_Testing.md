# Loop Testing

**Definisi:**
Teknik White Box yang fokus secara eksklusif pada validitas struktur perulangan (while, for, do-while).

**Skenario Pengujian Perulangan:**
1. **Zero Iteration:** Loop tidak dijalankan sama sekali.
2. **One Iteration:** Loop dijalankan tepat satu kali.
3. **M Iterations:** Loop dijalankan sebanyak $M$ kali ($M < \text{max}$).
4. **Maximum Iterations:** Loop dijalankan sampai batas maksimal.
5. **Boundary - 1 & + 1:** Menguji tepat di bawah dan di atas batas maksimal.

**Tujuan:** Mencegah terjadinya *infinite loop* atau kesalahan *off-by-one*.
