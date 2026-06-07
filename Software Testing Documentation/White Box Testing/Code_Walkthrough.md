# White Box Testing — 02 Code Walkthrough
**Proyek:** SaPoPoe Finance  
**Teknik:** Code Walkthrough  
**Modul:** Auth · Transfer · Transaksi · Tabungan

---

## Definisi

> **Teknik review kode secara formal atau informal yang dilakukan bersama-sama antara developer dan tim terkait untuk memahami logika kode, kemudian mengidentifikasi potensi error, dan meningkatkan kualitas keseluruhan program.**
>
> — Materi Pertemuan 10, Software Quality, T Informatika UKRI

---

## Modul A — Autentikasi (`AuthController.php`)

### Review Blok 1 — Method `register()`

```php
public function register(Request $request)
{
    $request->validate([
        'name'                  => 'required|string|max:255',
        'email'                 => 'required|string|email|max:255',
        'password'              => 'required|string|min:8|confirmed',
        'password_confirmation' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if ($user && $user->status === 'active') {
        return response()->json(['error' => 'Email sudah terdaftar dan aktif'], 400);
    }

    $this->checkCooldown($user);

    $otp = rand(100000, 999999);

    $user = User::updateOrCreate(
        ['email' => $request->email],
        [
            'name'            => $request->name,
            'password'        => Hash::make($request->password),
            'otp_code'        => (string) $otp,
            'otp_expires_at'  => now()->addMinutes(10),
        ]
    );

    Mail::send('emails.otp', ['otp' => $otp, 'name' => $user->name],
        function ($message) use ($user) {
            $message->to($user->email)->subject('Kode OTP Verifikasi Akun');
        }
    );

    return response()->json(['message' => 'Registrasi berhasil. Silakan cek email.'], 201);
}
```

**Walkthrough — Anotasi Per Blok:**

- **`$request->validate([...])`**
  Blok ini adalah gerbang pertama — Laravel memvalidasi semua field sebelum kode lain berjalan. Field `password` menggunakan rule `min:8|confirmed`, artinya password minimal 8 karakter dan harus ada field `password_confirmation` yang nilainya sama. Jika validasi gagal, Laravel otomatis return 422 tanpa masuk ke logika di bawah.
  - Variabel terlibat: `$request->name`, `$request->email`, `$request->password`
  - Potensi error: tidak ada rule `unique:users,email` — artinya email yang sudah terdaftar bisa masuk ke baris berikutnya dan diproses ulang.

- **`$user = User::where('email', $request->email)->first()`**
  Query ini mencari apakah email sudah ada di database. Hasilnya bisa `null` (belum pernah daftar) atau objek `User` (pernah daftar).
  - Variabel terlibat: `$user` — tipe `?User` (nullable)
  - Logika bisnis: sistem mendukung re-registrasi. User yang pernah daftar tapi belum aktif bisa daftar ulang untuk mendapat OTP baru.

- **`if ($user && $user->status === 'active')`**
  Kondisi ini memblokir re-registrasi jika akun sudah aktif. Jika TRUE, return 400.
  - Kondisi Is T: email sudah ada dan status = `active` → tolak → 400
  - Kondisi Is F: email belum ada (null) ATAU email ada tapi belum aktif → lanjut proses
  - Potensi error: tidak ada penanganan untuk kasus `$user` ada tapi status bukan `active` dan bukan `inactive` — jika ada nilai status lain di DB, bisa masuk ke proses yang tidak diinginkan.

- **`$this->checkCooldown($user)`**
  Memanggil method private yang mengecek apakah user sudah meminta OTP terlalu sering. Jika `$user` null (user baru), cooldown tidak berlaku.
  - Variabel terlibat: `$user` diteruskan sebagai argumen
  - Logika bisnis: mencegah brute-force request OTP berulang dalam 180 detik
  - Potensi error: jika `checkCooldown` melempar exception tapi tidak di-catch di sini, request akan return 500 tanpa pesan yang informatif.

- **`$otp = rand(100000, 999999)`**
  Membangkitkan kode OTP 6 digit secara acak.
  - Variabel terlibat: `$otp` — tipe integer
  - **Potensi error kritis:** `rand()` menggunakan Mersenne Twister, bukan kriptografis aman. Seharusnya `random_int(100000, 999999)` untuk keamanan kriptografis (CSPRNG).
  - Logika bisnis: OTP digunakan untuk membuktikan kepemilikan email.

- **`$user = User::updateOrCreate([...], [...])`**
  Jika email sudah ada di DB → UPDATE record yang ada. Jika belum ada → INSERT baru. Variabel `$user` di sini **mendefinisikan ulang** variabel `$user` yang sebelumnya sudah ada (anomali DD).
  - Variabel terlibat: `$user` (overwrite), `$otp`, `Hash::make($request->password)`
  - Field yang ditulis: `name`, `password` (di-hash), `otp_code` (string OTP), `otp_expires_at` (+10 menit)
  - Logika bisnis: setiap request register memperpanjang masa berlaku OTP dan meng-update nama/password — berarti password bisa diubah hanya dengan request register ulang selama akun belum aktif.
  - Potensi error: jika email user sudah aktif tapi lolos guard di atas (bug di guard), password-nya bisa di-overwrite oleh siapapun yang tahu emailnya.

- **`Mail::send('emails.otp', [...], function(...) { ... })`**
  Mengirim email berisi OTP ke user secara **sinkron (blocking)**.
  - Variabel terlibat: `$otp`, `$user->name`, `$user->email`
  - **Potensi error:** `Mail::send()` memblokir response hingga SMTP server merespons. Jika koneksi email lambat atau gagal, user menunggu lama atau menerima 500. Seharusnya `Mail::queue()` agar pengiriman dilakukan di background.
  - Logika bisnis: OTP hanya berguna jika email terkirim. Jika email gagal namun kode tidak throw exception, OTP tersimpan di DB tapi user tidak menerima — akun terkunci.

---

### Review Blok 2 — Method `login()`

```php
public function login(Request $request)
{
    $request->validate([
        'email'    => 'required|email',
        'password' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (!$user) {
        return response()->json(['error' => 'Alamat email tidak ditemukan'], 404);
    }

    if (!Hash::check($request->password, $user->password)) {
        return response()->json(['error' => 'Kata sandi salah'], 401);
    }

    if ($user->otp_code) {
        return response()->json([
            'error'    => 'Akun belum diverifikasi. Silakan cek email Anda.',
            'need_otp' => true,
        ], 403);
    }

    $token = $user->createToken('auth_token')->plainTextToken;

    return response()->json([
        'access_token' => $token,
        'token_type'   => 'Bearer',
        'user'         => $user,
    ], 200);
}
```

**Walkthrough — Anotasi Per Blok:**

- **`$user = User::where('email', $request->email)->first()`**
  Mencari user berdasarkan email. Tidak ada index yang disebutkan secara eksplisit — perlu dipastikan kolom `email` sudah ter-index di database untuk performa optimal.
  - Variabel terlibat: `$user` — nullable
  - Logika bisnis: email adalah identifier utama akun.

- **`if (!$user)`**
  Guard pertama: jika user tidak ditemukan, langsung tolak.
  - Kondisi Is T: `$user = null` → return 404 "Alamat email tidak ditemukan"
  - Kondisi Is F: user ditemukan → lanjut ke verifikasi password
  - Logika bisnis yang baik: return 404 bukan 401 untuk membedakan "tidak ada akun" vs "password salah" — memudahkan debugging user tapi bisa menjadi **user enumeration vector** (attacker tahu email terdaftar atau tidak dari response berbeda).

- **`if (!Hash::check($request->password, $user->password))`**
  Guard kedua: verifikasi password menggunakan timing-safe comparison.
  - Variabel terlibat: `$request->password` (plaintext dari input), `$user->password` (bcrypt hash dari DB)
  - Kondisi Is T: hash tidak cocok → return 401 "Kata sandi salah"
  - Kondisi Is F: hash cocok → lanjut ke cek OTP
  - Logika bisnis yang baik: `Hash::check()` menggunakan bcrypt constant-time comparison — aman dari timing attack.
  - Potensi error: tidak ada rate limiting khusus di method ini (throttle middleware ada di route level — perlu diverifikasi).

- **`if ($user->otp_code)`**
  Guard ketiga: cek apakah OTP masih tersisa di kolom — jika ada, berarti akun belum diverifikasi.
  - Variabel terlibat: `$user->otp_code` — string atau null
  - Kondisi Is T: `otp_code` berisi nilai → return 403 + flag `need_otp: true`
  - Kondisi Is F: `otp_code = null` → akun sudah terverifikasi → lanjut ke pembuatan token
  - **Potensi error:** logika bergantung pada nilai `otp_code` di DB. Jika karena bug `otp_code` tidak di-null-kan setelah verifikasi, user yang valid tidak bisa login selamanya. Ini sudah terjadi di akun uji (`dzakiawaludin0@gmail.com`).

- **`$token = $user->createToken('auth_token')->plainTextToken`**
  Membuat Sanctum token baru untuk sesi login.
  - Variabel terlibat: `$token` — string token Sanctum
  - Logika bisnis: setiap login berhasil membuat token baru. Token lama tidak di-revoke — satu user bisa memiliki banyak token aktif sekaligus (multi-device login diizinkan).
  - Potensi error: tidak ada limit jumlah token per user. Jika user login berkali-kali tanpa logout, tabel `personal_access_tokens` bisa terakumulasi banyak record.

---

## Modul B — Transfer (`TransferController.php`)

### Review Blok 1 — Method `store()`

```php
public function store(Request $request)
{
    $user = $request->user();

    $validated = $request->validate([
        'from_account_id' => 'required|integer',
        'to_account_id'   => 'required|integer|different:from_account_id',
        'amount'          => 'required|numeric|min:1',
        'admin_fee'       => 'nullable|numeric|min:0',
        'date'            => 'required|date|before_or_equal:today',
        'description'     => 'nullable|string',
    ]);

    $adminFee       = $validated['admin_fee'] ?? 0;
    $totalDeduction = $validated['amount'] + $adminFee;

    $fromAccount = FinancialAccount::where('id', $validated['from_account_id'])
                                   ->where('user_id', $user->id)->firstOrFail();
    $toAccount   = FinancialAccount::where('id', $validated['to_account_id'])
                                   ->where('user_id', $user->id)->firstOrFail();

    if ($fromAccount->balance < $totalDeduction) {
        return response()->json(['error' => 'Saldo tidak mencukupi untuk melakukan transfer'], 400);
    }

    $transferCategory = Category::where('user_id', $user->id)
                                ->where('name', 'Transfer Internal')->first();
    if (!$transferCategory) {
        $transferCategory = new Category();
        $transferCategory->user_id = $user->id;
        $transferCategory->name    = 'Transfer Internal';
        $transferCategory->save(); // ⚠️ DI LUAR TRANSACTION
    }

    if ($adminFee > 0) {
        $adminCategory = Category::where('user_id', $user->id)
                                 ->where('name', 'Biaya Admin Bank')->first();
        if (!$adminCategory) {
            $adminCategory = new Category();
            $adminCategory->user_id = $user->id;
            $adminCategory->name    = 'Biaya Admin Bank';
            $adminCategory->save(); // ⚠️ DI LUAR TRANSACTION
        }
    }

    DB::beginTransaction();
    try {
        $fromAccount->balance -= $totalDeduction;
        $fromAccount->save();

        $toAccount->balance += $validated['amount'];
        $toAccount->save();

        Transaction::create([
            'user_id'             => $user->id,
            'financial_account_id'=> $fromAccount->id,
            'category_id'         => $transferCategory->id,
            'type'                => 'transfer',
            'amount'              => $validated['amount'],
            'description'         => 'Transfer Keluar ke ' . $toAccount->name,
            'date'                => $validated['date'],
        ]);

        Transaction::create([
            'user_id'             => $user->id,
            'financial_account_id'=> $toAccount->id,
            'category_id'         => $transferCategory->id,
            'type'                => 'transfer',
            'amount'              => $validated['amount'],
            'description'         => 'Transfer Masuk dari ' . $fromAccount->name,
            'date'                => $validated['date'],
        ]);

        if ($adminFee > 0) {
            Transaction::create([
                'user_id'             => $user->id,
                'financial_account_id'=> $fromAccount->id,
                'category_id'         => $adminCategory->id,
                'type'                => 'expense',
                'amount'              => $adminFee,
                'description'         => 'Biaya Admin Transfer',
                'date'                => $validated['date'],
            ]);
        }

        DB::commit();
    } catch (\Exception $e) {
        DB::rollBack();
        return response()->json(['error' => 'Transfer gagal, silakan coba lagi.'], 500);
    }

    return response()->json(['message' => 'Transfer berhasil!'], 200);
}
```

**Walkthrough — Anotasi Per Blok:**

- **`$adminFee = $validated['admin_fee'] ?? 0` dan `$totalDeduction = $validated['amount'] + $adminFee`**
  Menghitung total yang akan dipotong dari akun asal. `??` memastikan jika `admin_fee` tidak dikirim, nilainya 0 bukan null.
  - Variabel terlibat: `$adminFee` (biaya tambahan), `$totalDeduction` (total pemotongan)
  - Logika bisnis: `$totalDeduction` memisahkan jumlah yang dipotong dari pengirim dengan jumlah yang diterima penerima — keduanya bisa berbeda karena admin fee.

- **`FinancialAccount::where('id',...)->where('user_id', $user->id)->firstOrFail()`**
  Mengambil akun sumber dan tujuan, difilter juga dengan `user_id` — memastikan akun benar-benar milik user yang login.
  - Variabel terlibat: `$fromAccount`, `$toAccount`
  - Logika bisnis yang baik: double filter (`id` + `user_id`) mencegah user mengakses akun milik user lain.
  - Potensi error: jika akun tidak ditemukan, `firstOrFail()` lempar `ModelNotFoundException` → return 404. Tidak ada pesan error custom untuk kasus ini.

- **`if ($fromAccount->balance < $totalDeduction)`**
  Guard saldo sebelum proses transfer.
  - Variabel terlibat: `$fromAccount->balance`, `$totalDeduction`
  - Kondisi Is T: saldo kurang → return 400 "Saldo tidak mencukupi"
  - Kondisi Is F: saldo cukup → lanjut ke proses kategori dan transaksi
  - Logika bisnis: satu-satunya guard finansial di seluruh method — letaknya tepat, sebelum apapun yang mengubah data.

- **Blok pembuatan `$transferCategory` dan `$adminCategory`**
  Mencari kategori di DB, jika belum ada dibuat baru.
  - Variabel terlibat: `$transferCategory`, `$adminCategory`
  - **Potensi error kritis:** kedua `$category->save()` berada **sebelum** `DB::beginTransaction()`. Jika transfer gagal di dalam blok try-catch dan `DB::rollBack()` dieksekusi, rollback **tidak** membatalkan pembuatan kategori ini — data kategori orphan tetap tersimpan.
  - Logika bisnis: kategori diperlukan sebagai foreign key (`category_id`) di tabel transactions.

- **`DB::beginTransaction()` dan blok `try`**
  Semua operasi perubahan saldo dan pencatatan transaksi dibungkus dalam satu transaksi DB.
  - Variabel terlibat: `$fromAccount->balance`, `$toAccount->balance`, semua field Transaction
  - Logika bisnis yang baik: jika salah satu INSERT gagal, semua perubahan saldo ikut dibatalkan — tidak ada partial update.

- **Dua `Transaction::create(...)` untuk KELUAR dan MASUK**
  Setiap transfer menghasilkan **dua** record transaksi: satu di akun pengirim (keluar), satu di akun penerima (masuk).
  - Variabel terlibat: `$fromAccount->id`, `$toAccount->id`, `$transferCategory->id`, `$validated['amount']`, `$validated['date']`
  - Logika bisnis: pasangan KELUAR-MASUK inilah yang kemudian diidentifikasi sebagai "siblings" di method `update()` dan `destroy()`.
  - **Potensi error:** identifikasi siblings menggunakan `created_at` timestamp — jika dua transfer terjadi dalam milidetik yang sama, siblings bisa tercampur.

- **`if ($adminFee > 0) { Transaction::create(...) }`**
  Jika ada biaya admin, dicatat sebagai transaksi terpisah bertipe `expense`.
  - Variabel terlibat: `$adminFee`, `$adminCategory->id`, `$fromAccount->id`
  - Logika bisnis: admin fee dipotong dari akun pengirim, bukan dari amount yang diterima penerima.

---

## Modul C — Transaksi (`TransactionController.php`)

### Review Blok 1 — Method `index()`

```php
public function index(Request $request)
{
    $user  = $request->user();
    $query = Transaction::where('user_id', $user->id);

    if ($request->filled('start_date'))
        $query->where('date', '>=', $request->start_date);

    if ($request->filled('end_date'))
        $query->where('date', '<=', $request->end_date);

    if ($request->filled('financial_account_id'))
        $query->where('financial_account_id', $request->financial_account_id);

    if ($request->filled('category_id'))
        $query->where('category_id', $request->category_id);

    if ($request->filled('type'))
        $query->where('type', $request->type);

    $sortBy    = $request->input('sort_by', 'date');
    $sortOrder = $request->input('sort_order', 'desc');
    $query->orderBy($sortBy, $sortOrder);

    $transactions = $query->get();

    return response()->json(['data' => $transactions], 200);
}
```

**Walkthrough — Anotasi Per Blok:**

- **`$query = Transaction::where('user_id', $user->id)`**
  Query dasar yang selalu difilter berdasarkan user yang sedang login. Ini adalah basis keamanan data — tidak ada user yang bisa melihat transaksi user lain.
  - Variabel terlibat: `$query` — Eloquent Builder object yang akan dimodifikasi
  - Logika bisnis yang baik: user isolation diterapkan di baris pertama, sebelum filter apapun.

- **Lima blok `if ($request->filled(...))`**
  Setiap blok menambahkan clause `where()` ke `$query` hanya jika parameter dikirim dan tidak kosong.
  - Variabel terlibat: `$request->start_date`, `$request->end_date`, `$request->financial_account_id`, `$request->category_id`, `$request->type`
  - Kondisi: `filled()` berbeda dari `has()` — `filled()` juga mengecek bahwa nilai bukan string kosong
  - Logika bisnis: filter bersifat kumulatif (AND), bukan OR — semakin banyak filter, semakin sedikit hasil.
  - Potensi error: tidak ada validasi tipe data pada nilai filter. Jika `financial_account_id` berisi string, query tidak akan error tapi hasilnya kosong tanpa pesan yang informatif.

- **`$sortBy = $request->input('sort_by', 'date')` dan `$query->orderBy($sortBy, $sortOrder)`**
  Menentukan urutan hasil query berdasarkan input user.
  - Variabel terlibat: `$sortBy` (nama kolom), `$sortOrder` (asc/desc)
  - **Potensi error kritis — SQL Injection:** `$sortBy` langsung dimasukkan ke `orderBy()` tanpa whitelist. Nama kolom di `orderBy()` tidak di-binding oleh PDO, sehingga nilai arbitrer bisa dieksekusi sebagai bagian dari SQL query.
  - Rekomendasi: `$allowed = ['date','amount','created_at']; $sortBy = in_array($sortBy, $allowed) ? $sortBy : 'date';`

- **`$transactions = $query->get()`**
  Mengeksekusi query dan mengambil semua hasil sekaligus.
  - Variabel terlibat: `$transactions` — Collection Eloquent
  - **Potensi error:** `get()` tanpa `limit()` atau `paginate()` mengambil semua record. User dengan 10.000+ transaksi akan menyebabkan response besar, memory exhaustion, dan timeout.
  - Logika bisnis: tidak ada batas tampilan — ini berbahaya di production.

---

### Review Blok 2 — Method `store()`

```php
public function store(Request $request)
{
    $user = $request->user();

    $validated = $request->validate([
        'financial_account_id' => 'required|integer',
        'category_id'          => 'required|integer',
        'type'                 => 'required|in:income,expense',
        'amount'               => 'required|numeric|min:1',
        'date'                 => 'required|date',
        'description'          => 'nullable|string',
    ]);

    DB::beginTransaction();
    try {
        $account = FinancialAccount::where('id', $validated['financial_account_id'])
                                   ->where('user_id', $user->id)->firstOrFail();

        if ($validated['type'] === 'income') {
            $account->balance += $validated['amount'];
        } else {
            $account->balance -= $validated['amount'];
        }
        $account->save();

        $transaction = Transaction::create([
            'user_id'             => $user->id,
            'financial_account_id'=> $validated['financial_account_id'],
            'category_id'         => $validated['category_id'],
            'type'                => $validated['type'],
            'amount'              => $validated['amount'],
            'date'                => $validated['date'],
            'description'         => $validated['description'] ?? null,
        ]);

        DB::commit();
        return response()->json(['data' => $transaction], 201);

    } catch (\Exception $e) {
        DB::rollBack();
        return response()->json(['error' => $e->getMessage()], 500);
    }
}
```

**Walkthrough — Anotasi Per Blok:**

- **`$account = FinancialAccount::where('id',...)->where('user_id', $user->id)->firstOrFail()`**
  Mengambil akun yang akan dimodifikasi, diverifikasi kepemilikannya.
  - Variabel terlibat: `$account` — objek FinancialAccount
  - Logika bisnis yang baik: double filter memastikan user tidak bisa memanipulasi akun orang lain di endpoint ini.

- **`if ($validated['type'] === 'income') { ... } else { ... }`**
  Satu-satunya percabangan di method ini — menentukan arah perubahan saldo.
  - Variabel terlibat: `$validated['type']`, `$account->balance`, `$validated['amount']`
  - Kondisi Is T (income): `$account->balance += $validated['amount']` — saldo bertambah
  - Kondisi Is F (expense): `$account->balance -= $validated['amount']` — saldo berkurang
  - **Potensi error kritis:** tidak ada pengecekan `$account->balance >= $validated['amount']` sebelum pengurangan. Saldo bisa menjadi negatif jika `amount > balance`. Berbeda dengan Transfer yang sudah memiliki guard ini.
  - Logika bisnis yang hilang: seharusnya ada validasi `if ($account->balance < $validated['amount']) return 400`.

- **`Transaction::create([...])`**
  Menyimpan record transaksi ke database.
  - Variabel terlibat: semua field dari `$validated` + `$user->id`
  - Potensi error: `category_id` dari request tidak diverifikasi apakah milik user yang sama — user bisa memasukkan `category_id` milik user lain.

- **`catch (\Exception $e) { return response()->json(['error' => $e->getMessage()], 500); }`**
  Menangkap semua exception dan mengembalikan pesan error.
  - **Potensi error:** `$e->getMessage()` di production bisa mengekspos nama tabel, nama kolom, atau path file — informasi sensitif yang seharusnya tidak terlihat user. Seharusnya pesan generik: `"Terjadi kesalahan sistem"`.

---

## Modul D — Tabungan (`SavingController.php`)

### Review Blok 1 — Method `update()`

```php
public function update(Request $request, $id)
{
    $user = $request->user();

    $validated = $request->validate([
        'name'           => 'sometimes|string|max:255',
        'target_amount'  => 'sometimes|numeric|min:1',
        'current_amount' => 'sometimes|numeric|min:0',
        'deadline'       => 'sometimes|date',
    ]);

    $saving    = $user->savings()->findOrFail($id);
    $oldAmount = $saving->current_amount;

    DB::beginTransaction();
    try {
        if (isset($validated['current_amount'])
            && $oldAmount !== $validated['current_amount'])
        {
            $selisih = $validated['current_amount'] - $oldAmount;
            $account = FinancialAccount::findOrFail($saving->financial_account_id);

            if ($selisih > 0) {
                $account->balance -= $selisih;
                $category = $this->getSavingCategory($user->id, 'expense');
                Transaction::create([
                    'user_id'              => $user->id,
                    'financial_account_id' => $account->id,
                    'category_id'          => $category->id,
                    'type'                 => 'expense',
                    'amount'               => $selisih,
                    'description'          => 'Top Up Tabungan: ' . $saving->name,
                    'date'                 => now()->toDateString(),
                ]);
            } else {
                $account->balance += abs($selisih);
                $category = $this->getSavingCategory($user->id, 'income');
                Transaction::create([
                    'user_id'              => $user->id,
                    'financial_account_id' => $account->id,
                    'category_id'          => $category->id,
                    'type'                 => 'income',
                    'amount'               => abs($selisih),
                    'description'          => 'Penarikan Tabungan: ' . $saving->name,
                    'date'                 => now()->toDateString(),
                ]);
            }
            $account->save();
        }

        $saving->update($validated);
        DB::commit();
        return response()->json(['data' => $saving->fresh()], 200);

    } catch (\Exception $e) {
        DB::rollBack();
        return response()->json(['error' => $e->getMessage()], 500);
    }
}
```

**Walkthrough — Anotasi Per Blok:**

- **`$saving = $user->savings()->findOrFail($id)` dan `$oldAmount = $saving->current_amount`**
  Mengambil data tabungan yang akan diubah, sekaligus membuat snapshot nilai lama.
  - Variabel terlibat: `$saving` (objek Saving), `$oldAmount` (nilai sebelum update)
  - Logika bisnis: `$user->savings()` memastikan tabungan yang diambil pasti milik user yang login.
  - Logika bisnis: `$oldAmount` diperlukan untuk menghitung selisih perubahan saldo — tanpa ini tidak bisa tahu apakah user top up atau menarik.

- **`if (isset($validated['current_amount']) && $oldAmount !== $validated['current_amount'])`**
  Gate yang menentukan apakah ada perubahan finansial (bukan hanya update nama atau deadline).
  - Variabel terlibat: `$validated['current_amount']`, `$oldAmount`
  - Kondisi Is T: `current_amount` dikirim DAN berbeda dari nilai lama → proses kalkulasi saldo
  - Kondisi Is F: `current_amount` tidak dikirim, atau nilainya sama → hanya update metadata (nama, target, deadline)
  - Logika bisnis yang baik: memisahkan update finansial dari update metadata secara eksplisit.

- **`$selisih = $validated['current_amount'] - $oldAmount`**
  Menghitung perbedaan nilai tabungan baru vs lama.
  - Variabel terlibat: `$selisih` — bisa positif (top up) atau negatif (penarikan)
  - Logika bisnis: `$selisih` menjadi pivotal variable — nilainya menentukan semua yang terjadi selanjutnya.
  - **Potensi error:** tidak ada validasi apakah `$validated['current_amount'] >= 0` setelah dikurangi. Jika user memasukkan `current_amount: -100`, `$selisih` bisa sangat negatif, menyebabkan saldo akun bertambah besar secara tidak wajar.

- **`$account = FinancialAccount::findOrFail($saving->financial_account_id)`**
  Mengambil akun rekening yang terhubung ke tabungan ini.
  - Variabel terlibat: `$account` — objek FinancialAccount
  - **Potensi error:** `findOrFail()` tanpa filter `user_id` — hanya mencari berdasarkan ID. Meski dalam skenario normal ID ini pasti milik user yang sama, ini tetap berpotensi IDOR jika data `savings` bisa dimanipulasi.

- **`if ($selisih > 0)` — Blok Top Up**
  Jika nilai baru lebih besar dari nilai lama, ini adalah top up.
  - Variabel terlibat: `$account->balance`, `$selisih`, `$category`
  - `$account->balance -= $selisih` — saldo rekening berkurang sebesar selisih
  - `$this->getSavingCategory($user->id, 'expense')` — ambil/buat kategori pengeluaran tabungan
  - Catat transaksi tipe `expense` sebesar `$selisih`
  - **Potensi error:** tidak ada pengecekan `$account->balance >= $selisih` — saldo rekening bisa negatif jika top up melebihi saldo tersedia.

- **`else` — Blok Penarikan**
  Jika `$selisih < 0`, ini adalah penarikan sebagian dari tabungan.
  - Variabel terlibat: `$account->balance`, `abs($selisih)`, `$category`
  - `$account->balance += abs($selisih)` — saldo rekening bertambah sebesar nilai penarikan
  - Catat transaksi tipe `income` sebesar `abs($selisih)`
  - **Potensi error:** tidak ada pengecekan apakah `$validated['current_amount'] >= 0` — penarikan bisa membuat `current_amount` tabungan menjadi negatif.
  - Logika bisnis yang baik: `abs()` memastikan amount transaksi selalu positif meskipun `$selisih` negatif.

- **`$saving->update($validated)`**
  Update semua field yang dikirim sekaligus (batch update).
  - Variabel terlibat: `$validated` — array field yang sudah lolos validasi
  - Potensi error: jika `$validated` berisi field yang tidak ada di `$fillable` model Saving, Laravel akan mengabaikannya secara diam-diam tanpa error — tidak ada feedback bahwa field tertentu diabaikan.

---

## Ringkasan Temuan Code Walkthrough

| Modul | Method | Temuan | Severity |
|---|---|---|---|
| Auth | `register()` | `rand()` untuk OTP — tidak CSPRNG | 🔴 Tinggi |
| Auth | `register()` | `Mail::send()` sinkron — blocking response | 🟡 Sedang |
| Auth | `register()` | `updateOrCreate()` bisa overwrite password akun lama | 🟡 Sedang |
| Auth | `login()` | Response 404 vs 401 berbeda → user enumeration risk | 🟡 Sedang |
| Auth | `login()` | Token lama tidak di-revoke saat login baru | 🟢 Rendah |
| Transfer | `store()` | `Category::save()` di luar `DB::beginTransaction()` | 🔴 Tinggi |
| Transfer | `store()` | Siblings diidentifikasi via `created_at` — race condition | 🔴 Tinggi |
| Transaksi | `index()` | `$sortBy` langsung ke `orderBy()` — SQL injection | 🔴 Tinggi |
| Transaksi | `index()` | `get()` tanpa pagination — memory exhaustion | 🟡 Sedang |
| Transaksi | `store()` | Tidak ada guard saldo minimum untuk expense | 🔴 Tinggi |
| Transaksi | `store()` | `$e->getMessage()` di catch — ekspos info internal | 🟡 Sedang |
| Tabungan | `update()` | Tidak ada guard saldo rekening saat top up | 🔴 Tinggi |
| Tabungan | `update()` | `findOrFail()` tanpa `user_id` filter | 🟡 Sedang |
| Tabungan | `update()` | `current_amount` bisa minus jika input negatif | 🟡 Sedang |
