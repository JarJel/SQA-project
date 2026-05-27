# White Box Testing — Code Walkthrough

**Proyek:** Midnight Finance  
**Modul yang Dikaji:** Authentication — `AuthController.php`  
**Tanggal:** 27 Mei 2026  
**Penguji:** QA Team  
**Metode:** Code Walkthrough (Static White Box Testing)

---

## 1. Deskripsi

Code Walkthrough adalah teknik *static white box testing* di mana tim QA menelusuri kode sumber secara manual bersama pengembang untuk menemukan potensi kesalahan logika, keamanan, atau pelanggaran standar — **tanpa menjalankan program**.

**File yang dikaji:** `app/Http/Controllers/API/AuthController.php`  
**Metode yang ditelaah:** `login()`, `register()`, `checkCooldown()`

---

## 2. Source Code yang Dikaji (Kode Asli)

```php
// AuthController.php — Method: login()
public function login(Request $request)
{
$request->validate(['email' => 'required|email', 'password' => 'required']);
$user = User::where('email', $request->email)->first();

if (!$user) 
    return response()->json(['message' => 'Alamat email tidak ditemukan...'], 404);
if (!Hash::check($request->password, $user->password)) 
    return response()->json(['message' => 'Kata sandi yang Anda masukkan salah.'], 401);
if ($user->otp_code) 
    return response()->json(['message' => 'Akun belum diverifikasi...', 'need_otp' => true], 403);
return response()->json([
    'message' => 'Akses diberikan. Membuka brankas digital Anda...',
    'access_token' => $user->createToken('auth_token')->plainTextToken,
    'user' => $user
]);
}

// AuthController.php — Method: register()
public function register(Request $request)
{
$request->validate([
'name' => 'required|string|max:255',
'email' => 'required|email',
'password' => ['required', 'confirmed',
Password::min(8)->letters()->mixedCase()->numbers()->symbols()->uncompromised()
],
]);

$user = User::where('email', $request->email)->first();
if ($user && $user->status === 'active') {
    return response()->json(['message' => 'Alamat email sudah terdaftar...'], 400);
}
if ($cooldownMsg = $this->checkCooldown($user)) {
    return response()->json(['message' => $cooldownMsg], 429);
}
$otp = rand(100000, 999999);
$user = User::updateOrCreate(
    ['email' => $request->email],
    ['name' => $request->name, 'password' => Hash::make($request->password),
     'status' => 'inactive', 'otp_code' => (string) $otp,
     'otp_expires_at' => now()->addMinutes(10)]
);
Mail::send('emails.otp', ['otp' => $otp, 'user' => $user], function($msg) use ($user) {
    $msg->to($user->email)->subject('Kode Verifikasi Keamanan - Midnight Finance');
});
return response()->json(['message' => 'Registrasi berhasil...'], 201);
}

// AuthController.php — Helper: checkCooldown()
private function checkCooldown($user)
{
if ($user && $user->otp_code && $user->updated_at) {
s
e
c
o
n
d
s
P
a
s
s
e
d
=
n
o
w
(
)
−
>
d
i
f
f
I
n
S
e
c
o
n
d
s
(
secondsPassed=now()−>diffInSeconds(user->updated_at);
if ($secondsPassed < 180) {
$wait = ceil((180 - $secondsPassed) / 60);
return "Tunggu {$wait} menit lagi sebelum meminta kode verifikasi baru.";
}
}
return false;
}

---
## 3. Temuan Walkthrough
### Temuan #1 — `login()`: Status Akun Tidak Dicek
| Atribut | Detail |
|---------|--------|
| **Lokasi** | `login()` — kondisi ketiga |
| **Kondisi** | Cek `$user->otp_code` bukan `$user->status` |
| **Risiko** | User dengan `status = 'inactive'` yang OTP-nya sudah null (expired dibersihkan) bisa login |
| **Tingkat** | 🟠 Medium |
| **Rekomendasi** | Tambahkan `if ($user->status !== 'active') return response()->json([...], 403)` |
### Temuan #2 — `register()`: Email Validation Tidak Cek `unique`
| Atribut | Detail |
|---------|--------|
| **Lokasi** | `register()` — rule validasi |
| **Kondisi** | Tidak ada rule `unique:users,email` di `$request->validate()` |
| **Risiko** | Logika duplikat ditangani manual (`if ($user && $user->status === 'active')`), lebih rawan error dibanding menggunakan Laravel's built-in unique rule |
| **Tingkat** | 🟡 Low |
| **Rekomendasi** | Ini desain yang disengaja (untuk `updateOrCreate`), tapi perlu comment penjelasan |
### Temuan #3 — `checkCooldown()`: `diffInSeconds` bisa Negatif
| Atribut | Detail |
|---------|--------|
| **Lokasi** | `checkCooldown()` |
| **Kondisi** | `now()->diffInSeconds($user->updated_at)` bisa bernilai negatif jika clock server tidak sinkron |
| **Risiko** | Cooldown bisa bypass jika server time bermasalah |
| **Tingkat** | 🟡 Low |
| **Rekomendasi** | Gunakan `abs(now()->diffInSeconds($user->updated_at))` |
### Temuan #4 — `register()`: OTP Menggunakan `rand()` bukan `random_int()`
| Atribut | Detail |
|---------|--------|
| **Lokasi** | `register()` — baris `$otp = rand(100000, 999999)` |
| **Kondisi** | `rand()` adalah pseudo-random, tidak cryptographically secure |
| **Risiko** | OTP berpotensi diprediksi pada sistem dengan seed yang dapat ditebak |
| **Tingkat** | 🟠 Medium |
| **Rekomendasi** | Gunakan `random_int(100000, 999999)` — PHP CSPRNG |
---
## 4. Ringkasan Temuan
| ID | Metode | Temuan | Tingkat |
|----|--------|--------|---------|
| CW-01 | `login()` | Status akun tidak dicek secara eksplisit | 🟠 Medium |
| CW-02 | `register()` | Tidak ada `unique:users,email` rule | 🟡 Low |
| CW-03 | `checkCooldown()` | `diffInSeconds` tanpa `abs()` | 🟡 Low |
| CW-04 | `register()` | `rand()` tidak cryptographically secure | 🟠 Medium |
**Total:** 4 temuan | 🟠 2 Medium | 🟡 2 Low | 🔴 0 Critical
---
## 5. Kesimpulan
Code Walkthrough pada `AuthController.php` menemukan **4 potensi perbaikan**. Tidak ada temuan critical yang memblokir sistem. Temuan CW-01 dan CW-04 direkomendasikan untuk diperbaiki sebelum production deployment. Keseluruhan alur autentikasi logis dan terstruktur dengan baik menggunakan Laravel's built-in security features (Hash, Password rules, Sanctum tokens).
