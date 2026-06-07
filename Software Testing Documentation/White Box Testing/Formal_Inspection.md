# White Box Testing — 03 Formal Inspections
**Proyek:** SaPoPoe Finance  
**Teknik:** Formal Inspections  
**Modul:** Auth · Transfer · Transaksi · Tabungan

---

## Definisi

> **Pengujian Formal Inspection adalah teknik pengujian yang berfokus untuk memeriksa kelengkapan, ketepatan, dan format data yang ditampilkan dan diterima oleh aplikasi.**
>
> — Materi Pertemuan 10, Software Quality, T Informatika UKRI

---

## Kategori A — Validasi & Input

| ID | Kriteria Inspeksi | AUTH | TRANSFER | TRANSAKSI | TABUNGAN |
|---|---|---|---|---|---|
| A-01 | Semua field wajib divalidasi sebelum diproses | ✅ PASS | ✅ PASS | ✅ PASS | ✅ PASS |
| A-02 | Field numerik memiliki batas minimum (min:1) | ✅ PASS | ✅ PASS | ✅ PASS | ✅ PASS |
| A-03 | Field tanggal dibatasi agar tidak melebihi hari ini | ✅ `before_or_equal` | ✅ `before_or_equal` | ⚠️ PARTIAL | ⚠️ PARTIAL |
| A-04 | Transfer ke akun yang sama dicegah | — | ✅ `different:from_account_id` | — | — |
| A-05 | Tipe data enum divalidasi (`in:income,expense`) | — | — | ✅ PASS | — |
| A-06 | Kepemilikan resource diverifikasi via `user_id` | ✅ PASS | ⚠️ PARTIAL (loop) | ⚠️ PARTIAL (update/destroy) | ⚠️ PARTIAL |
| A-07 | Input tidak langsung masuk ke SQL mentah | ✅ PASS | ✅ PASS | ❌ FAIL (`sort_by` → `orderBy`) | ✅ PASS |

---

> ### 📋 Analisis SQA — Kategori Validasi & Input
>
> **Kondisi Sistem Saat Ini**
> Validasi input di seluruh modul sudah menggunakan Laravel Form Request dengan baik. Field wajib, tipe data, dan batas minimum sudah terdefinisi. Namun ada dua celah konsisten: (1) tidak semua modul memvalidasi batas tanggal `before_or_equal:today`, dan (2) kepemilikan resource (`user_id`) tidak selalu diverifikasi ulang ketika melakukan lookup di dalam loop atau di fase update.
>
> **Dampak**
> Celah paling kritis ada di A-07 — `$sortBy` dari `$request->input('sort_by')` langsung diteruskan ke `$query->orderBy($sortBy, ...)`. Eloquent ORM tidak mem-bind nama kolom di `orderBy()`, sehingga value ini dapat dimanipulasi untuk melakukan **column-based SQL injection**. Misalnya input `(SELECT password FROM users LIMIT 1)` berpotensi mengekspos data sensitif melalui error message atau response ordering.
>
> **Cara Baca Tabel**
> Status ✅ PASS berarti kriteria terpenuhi sepenuhnya. ⚠️ PARTIAL berarti terpenuhi sebagian — ada kondisi tertentu yang tidak tercakup. ❌ FAIL berarti kriteria sama sekali tidak terpenuhi. Tanda `—` berarti kriteria tidak relevan untuk modul tersebut (N/A).

---

## Kategori B — Keamanan & Kriptografi

| ID | Kriteria Inspeksi | AUTH | TRANSFER | TRANSAKSI | TABUNGAN |
|---|---|---|---|---|---|
| B-01 | Password di-hash dengan algoritma aman (bcrypt) | ✅ `Hash::make()` | — | — | — |
| B-02 | Perbandingan password bersifat timing-safe | ✅ `Hash::check()` | — | — | — |
| B-03 | OTP/token menggunakan CSPRNG | ❌ FAIL (`rand()`) | — | — | — |
| B-04 | OTP/token memiliki masa berlaku | ✅ 10 menit | — | — | — |
| B-05 | Token dihapus setelah digunakan | ✅ `otp_code = null` | — | — | — |
| B-06 | Semua token revoked saat reset password | ✅ `tokens()->delete()` | — | — | — |
| B-07 | Response error tidak mengekspos stack trace | ✅ PASS | ⚠️ `$e->getMessage()` | ⚠️ `$e->getMessage()` | ✅ PASS |

---

> ### 📋 Analisis SQA — Kategori Keamanan & Kriptografi
>
> **Kondisi Sistem Saat Ini**
> Secara umum, implementasi keamanan autentikasi sudah cukup baik: `Hash::make()` menggunakan bcrypt, `Hash::check()` timing-safe, token dihapus setelah digunakan. Satu kelemahan signifikan: `rand()` digunakan untuk membangkitkan OTP 6 digit alih-alih `random_int()`.
>
> **Dampak**
> `rand()` di PHP menggunakan Mersenne Twister yang bukan kriptografis aman. Attacker yang dapat mengamati beberapa OTP berurutan berpotensi memprediksi OTP berikutnya. Ini adalah kerentanan yang nyata meski eksploitasi memerlukan akses jaringan dan timing yang presisi. Selain itu, `$e->getMessage()` di catch block Transfer dan Transaksi bisa mengekspos nama tabel, nama kolom, atau path file di environment produksi.
>
> **Cara Baca Tabel**
> Kolom AUTH mendapat paling banyak inspeksi di kategori ini karena Auth adalah satu-satunya modul yang mengelola credential dan token. Modul lain mendapat tanda `—` karena tidak memiliki logika kriptografi. Fokus inspeksi pada B-03 dan B-07 sebagai temuan yang membutuhkan perbaikan segera.

---

## Kategori C — Integritas Data & Transaksi DB

| ID | Kriteria Inspeksi | AUTH | TRANSFER | TRANSAKSI | TABUNGAN |
|---|---|---|---|---|---|
| C-01 | Operasi kritis dibungkus `DB::beginTransaction()` | ✅ `setup()` | ✅ PASS | ✅ PASS | ✅ PASS |
| C-02 | Exception selalu men-trigger `DB::rollBack()` | ✅ PASS | ✅ PASS | ✅ PASS | ✅ PASS |
| C-03 | Pembuatan resource pendukung masuk dalam transaksi | — | ❌ FAIL (Kategori di luar) | — | ⚠️ PARTIAL |
| C-04 | Saldo dikembalikan dengan benar saat update/delete | — | ✅ Fase Revert | ✅ PASS | ✅ PASS |
| C-05 | Tidak ada partial update (all-or-nothing) | ✅ PASS | ✅ PASS | ✅ PASS | ✅ PASS |

---

> ### 📋 Analisis SQA — Kategori Integritas Data
>
> **Kondisi Sistem Saat Ini**
> Penggunaan `DB::beginTransaction()` dan `DB::rollBack()` sudah konsisten di semua modul — ini adalah fondasi integritas data yang baik. Namun C-03 menjadi temuan kritikal: di `TransferController::store()`, pembuatan `Category` dilakukan sebelum `beginTransaction()`. Ini berarti operasi pembuatan kategori tidak terproteksi rollback.
>
> **Dampak**
> Skenario gagal yang paling berbahaya: transfer diproses, kategori baru dibuat, kemudian transaksi di dalam DB gagal (misalnya karena deadlock). `rollBack()` membatalkan perubahan saldo dan transaksi, tetapi kategori "Transfer Internal" yang baru saja dibuat tetap ada. Akumulasi kategori orphan ini mencemari data master dan dapat memunculkan duplikat yang membingungkan user.
>
> **Cara Baca Tabel**
> C-03 adalah kriteria paling sensitif di kategori ini — periksa apakah setiap `save()` yang dibuat dalam rangkaian proses ada di dalam scope `beginTransaction()`. Cara termudah memverifikasinya dari kode adalah mencari `DB::beginTransaction()` dan memeriksa apakah semua operasi tulis ada setelahnya, bukan sebelumnya.

---

## Kategori D — Logika Bisnis

| ID | Kriteria Inspeksi | AUTH | TRANSFER | TRANSAKSI | TABUNGAN |
|---|---|---|---|---|---|
| D-01 | Saldo diperiksa sebelum pengurangan | — | ✅ `store()` & `update()` | ❌ FAIL | ❌ FAIL |
| D-02 | Biaya admin dicatat sebagai transaksi terpisah | — | ✅ PASS | — | — |
| D-03 | Rate limiting pada endpoint sensitif | ✅ `throttle:5` OTP | — | — | — |
| D-04 | Cooldown antar request OTP diterapkan | ✅ 180 detik | — | — | — |
| D-05 | Identifikasi transaksi terkait untuk revert | — | ⚠️ via `created_at` | ✅ via `transaction.id` | ✅ PASS |
| D-06 | Pencairan tabungan dicatat sebagai income | — | — | — | ✅ PASS |

---

> ### 📋 Analisis SQA — Kategori Logika Bisnis
>
> **Kondisi Sistem Saat Ini**
> Transfer adalah modul terbaik dalam hal logika bisnis: cek saldo ada, admin fee dicatat terpisah, rollback ada. Namun Transaksi dan Tabungan sama-sama tidak melakukan pengecekan saldo sebelum pengurangan (D-01 FAIL), yang merupakan **defect fungsional dengan severity tinggi**. D-05 di Transfer menggunakan `created_at` untuk mengelompokkan sibling transaction — metode yang rawan race condition pada concurrent request di detik yang sama.
>
> **Dampak**
> Absennya cek saldo di Transaksi dan Tabungan memungkinkan saldo akun menjadi negatif tanpa peringatan. Ini fatal untuk sistem keuangan personal — user tidak mendapat feedback bahwa dana tidak mencukupi, dan laporan saldo akan menunjukkan angka negatif yang membingungkan. Sibling detection via `created_at` berisiko salah memasangkan transaksi pada beban tinggi.
>
> **Cara Baca Tabel**
> Perhatikan kolom TRANSFER vs TRANSAKSI/TABUNGAN di D-01: ini adalah inkonsistensi implementasi dalam satu proyek. Idealnya seluruh operasi pengurangan saldo mengikuti pola yang sama (seperti Transfer) — ini yang disebut **DRY principle** di level logika bisnis. Inkonsistensi seperti ini adalah indikator bahwa fitur dikembangkan oleh developer yang berbeda tanpa standar bersama.

---

## Kategori E — Kualitas Kode

| ID | Kriteria Inspeksi | AUTH | TRANSFER | TRANSAKSI | TABUNGAN |
|---|---|---|---|---|---|
| E-01 | HTTP status code semantik tepat | ✅ PASS | ✅ PASS | ✅ PASS | ✅ PASS |
| E-02 | Tidak ada duplikasi logika (DRY) | ⚠️ PARTIAL | ❌ FAIL (foreach revert duplikat) | ✅ PASS | ✅ PASS |
| E-03 | Pagination pada endpoint list | — | — | ❌ FAIL | — |
| E-04 | Email dikirim asinkron (non-blocking) | ❌ FAIL (`Mail::send`) | — | — | — |
| E-05 | Error response tidak ekspos detail internal | ✅ PASS | ⚠️ `$e->getMessage()` | ⚠️ `$e->getMessage()` | ✅ PASS |

---

> ### 📋 Analisis SQA — Kategori Kualitas Kode
>
> **Kondisi Sistem Saat Ini**
> HTTP status code sudah digunakan dengan semantik yang benar di semua modul — 201 untuk create, 200 untuk update/delete, 400/401/403/404 untuk error. Duplikasi logika (E-02) ditemukan di Transfer: kode `foreach ($siblings)` yang identik ada di method `update()` dan `destroy()`. Ini pelanggaran prinsip DRY yang bisa menyebabkan bug asimetris jika hanya satu method yang diperbaiki.
>
> **Dampak**
> `Mail::send()` sinkron (E-04) adalah risiko performa yang paling terasa di production: jika email provider lambat, semua request register akan ikut tertahan. Absennya pagination di `index()` Transaksi (E-03) berarti user dengan ribuan transaksi akan menerima semua data sekaligus, berpotensi menyebabkan response time > 10 detik dan memory exhaustion di server.
>
> **Cara Baca Tabel**
> Kategori E berfokus pada **maintainability** dan **scalability** kode — bukan sekadar apakah fitur berfungsi, tetapi apakah kode akan bertahan saat di-scale. FAIL di E-03 dan E-04 tidak menyebabkan bug saat development dengan data kecil, tapi akan menjadi masalah serius saat production dengan ribuan user.

---

## Rekapitulasi Hasil Inspeksi

| Modul | Total Item | ✅ PASS | ❌ FAIL | ⚠️ PARTIAL |
|---|---|---|---|---|
| AUTH | 25 | 19 | 3 | 3 |
| Transfer | 25 | 18 | 4 | 3 |
| Transaksi | 25 | 16 | 5 | 4 |
| Tabungan | 25 | 17 | 3 | 5 |

**Total defect:** 15 item FAIL + 15 item PARTIAL = **30 poin perhatian**  
**Prioritas perbaikan:** D-01 (saldo negatif), A-07 (SQL injection), C-03 (kategori di luar transaksi), E-04 (mail sinkron)
