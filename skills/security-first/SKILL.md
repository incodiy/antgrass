---
name: security-first
description: Use when writing security-sensitive code. Covers SQL injection prevention, XSS/CSRF protection, authentication, authorization, encryption, input validation, and security headers.
version: 1.1.0
updated: 2026-06-21
---

# Security-First Development

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI TANPA PENGECUALIAN

**Security adalah PRIORITAS PERTAMA.** Satu celah keamanan bisa menyebabkan kerugian finansial, hukum, dan reputasi yang tidak bisa diperbaiki.

🔴 **Konsekuensi nyata jika dilanggar:**
- **SQL injection** → data breach, jutaan data pengguna bocor, ganti rugi milyaran
- **XSS (Cross-Site Scripting)** → session user dicuri, account takeover
- **Password salah simpan (MD5/plaintext)** → compliance violation (UU PDP), denda administratif
- **Hardcoded secrets/api keys** → billing abuse, penyalahgunaan API, kebocoran ke GitHub
- **Tidak validasi input** → berbagai serangan lanjutan (RCE, path traversal, dll)

**Celah keamanan sering tidak terdeteksi di development — baru ketahuan setelah di exploit.**
Model **WAJIB** mematuhi setiap aturan di bawah. JANGAN skip, JANGAN tawar-menawar.

## Tujuan
Memastikan setiap aplikasi dibangun dengan security best practices untuk melindungi data pengguna dan sistem dari berbagai ancaman.

## Gate-Aware Security Scope

Jika diaktifkan oleh `solo-dev-workflow`, fokuskan review sesuai gate:

| Gate | Fokus Security | Yang BUKAN fokus |
|------|---------------|-----------------|
| **G1 (Architecture)** | Auth architecture, encryption strategy, API security design, environmental secrets setup | Code-level implementation details |
| **G3 (Development)** | Input validation, SQL injection, XSS, password hashing, error response security | Deployment / hosting security configurations |
| **G4 (Quality & Deploy)** | Dependency audit, CORS, SSL/HTTPS headers, rate limiting, backup strategy, penetration test | Architecture redesign |

---

## Prinsip Dasar Security

### 1. Defense in Depth (Berlapis)
Jangan hanya mengandalkan satu layer security. Implementasikan multiple layers:
- Input validation
- Authentication & Authorization
- Encryption
- Rate limiting
- Logging & Monitoring

### 2. Principle of Least Privilege
- User hanya punya akses ke resource yang benar-benar dibutuhkan
- Service accounts dengan minimal permissions
- Database user dengan limited privileges

### 3. Fail Securely
- Default action adalah DENY
- Error messages tidak expose sensitive information
- Graceful degradation saat security check gagal

## Common Security Vulnerabilities & Prevention

### 1. SQL Injection

❌ **Vulnerable Code:**
```php
// JANGAN PERNAH LAKUKAN INI
$query = "SELECT * FROM users WHERE email = '" . $_POST['email'] . "'";
$result = mysqli_query($conn, $query);
```

✅ **Secure Code:**
```php
// Gunakan Prepared Statements
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
$stmt->execute([$email]);
$user = $stmt->fetch();

// Atau dengan Named Parameters
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $email]);
```

### 2. XSS (Cross-Site Scripting)

❌ **Vulnerable Code:**
```php
// Output langsung tanpa escaping
echo "<div>Welcome, " . $_GET['name'] . "</div>";
```

✅ **Secure Code:**
```php
// Escape output
echo "<div>Welcome, " . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8') . "</div>";

// Atau gunakan template engine dengan auto-escaping
// Blade (Laravel), Twig, dll
```

```javascript
// Frontend: Gunakan textContent, bukan innerHTML
element.textContent = userInput; // Safe
// element.innerHTML = userInput; // Dangerous!

// Jika HARUS pakai HTML, sanitize dulu
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

### 3. CSRF (Cross-Site Request Forgery)

✅ **Implementation:**
```php
// Laravel otomatis handle dengan @csrf token
<form method="POST" action="/transfer">
    @csrf
    <input name="amount" value="1000">
    <button type="submit">Transfer</button>
</form>

// Verify CSRF token di backend
// Laravel middleware: VerifyCsrfToken
```

```javascript
// SPA: Include CSRF token di header
axios.defaults.headers.common['X-CSRF-TOKEN'] = document
  .querySelector('meta[name="csrf-token"]')
  .getAttribute('content');
```

### 4. Authentication & Password Security

✅ **Password Hashing:**
```php
// JANGAN gunakan MD5, SHA1, atau plain text!

// Gunakan bcrypt (recommended)
$hashedPassword = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);

// Verify password
if (password_verify($inputPassword, $hashedPassword)) {
    // Login success
}

// Password requirements
- Minimum 8 karakter
- Kombinasi huruf besar, kecil, angka, symbol
- Tidak sama dengan username/email
- Check against common password list
```

### 5. Authorization & Access Control

✅ **Implement Proper Authorization:**
```php
// Check authorization SEBELUM action
public function updateUser(Request $request, $userId)
{
    $user = User::findOrFail($userId);
    
    // Authorization check
    if (auth()->user()->id !== $user->id && !auth()->user()->isAdmin()) {
        abort(403, 'Unauthorized action');
    }
    
    // Proceed with update
    $user->update($request->validated());
}

// Atau gunakan Policy (Laravel)
$this->authorize('update', $user);
```

**Role-Based Access Control (RBAC):**
```php
// Database: users, roles, permissions, role_user, permission_role

// Check permission
if ($user->hasPermission('delete-post')) {
    // Allow action
}

// Check role
if ($user->hasRole('admin')) {
    // Admin action
}
```

### 6. Input Validation & Sanitization

✅ **Server-Side Validation (WAJIB):**
```php
// JANGAN hanya validate di frontend!

$request->validate([
    'email' => 'required|email|max:255',
    'age' => 'required|integer|min:1|max:150',
    'website' => 'nullable|url',
    'phone' => 'required|regex:/^[0-9]{10,15}$/',
]);

// Whitelist allowed values
$allowedStatuses = ['active', 'inactive', 'pending'];
if (!in_array($status, $allowedStatuses)) {
    throw new ValidationException('Invalid status');
}
```

### 7. File Upload Security

✅ **Secure File Upload:**
```php
public function uploadFile(Request $request)
{
    $request->validate([
        'file' => 'required|file|max:10240|mimes:jpg,jpeg,png,pdf', // 10MB max
    ]);
    
    $file = $request->file('file');
    
    // Generate unique filename (don't trust user filename)
    $filename = Str::uuid() . '.' . $file->getClientOriginalExtension();
    
    // Store di luar public directory
    $path = $file->storeAs('uploads', $filename, 'private');
    
    // Scan virus (jika ada ClamAV integration)
    // $this->virusScanner->scan($path);
    
    return $path;
}

// Serve file dengan authorization check
public function downloadFile($fileId)
{
    $file = File::findOrFail($fileId);
    
    // Authorization check
    $this->authorize('download', $file);
    
    return Storage::disk('private')->download($file->path);
}
```

### 8. API Security

✅ **JWT Token Best Practices:**
```php
// Short expiration time
$token = JWTAuth::claims(['exp' => time() + 3600])->fromUser($user); // 1 hour

// Use refresh token untuk long sessions
[
    'access_token' => $accessToken,  // Short-lived (15 min)
    'refresh_token' => $refreshToken // Long-lived (7 days)
]

// Store refresh token di database dengan expiration
// Dapat di-revoke kapan saja
```

✅ **Rate Limiting:**
```php
// Laravel throttle middleware
Route::middleware('throttle:60,1')->group(function () {
    // Max 60 requests per minute
});

// Stricter untuk sensitive endpoints
Route::post('/login')->middleware('throttle:5,1'); // 5 attempts per minute
```

✅ **API Key Management:**
```php
// JANGAN hardcode API keys di code!

// Gunakan environment variables
$apiKey = env('EXTERNAL_API_KEY');

// Rotate API keys secara berkala
// Store di secure vault (AWS Secrets Manager, HashiCorp Vault)
```

### 9. Data Encryption

✅ **Encryption at Rest:**
```php
// Encrypt sensitive data di database
use Illuminate\Support\Facades\Crypt;

// Encrypt
$encrypted = Crypt::encryptString($sensitiveData);

// Decrypt
$decrypted = Crypt::decryptString($encrypted);

// Cast model attribute
class User extends Model {
    protected $casts = [
        'ssn' => 'encrypted', // Social Security Number
        'credit_card' => 'encrypted',
    ];
}
```

✅ **Encryption in Transit:**
```
- WAJIB gunakan HTTPS (TLS 1.2+)
- Disable HTTP, atau redirect ke HTTPS
- Use HSTS header
- Certificate dari trusted CA
```

### 10. Security Headers

✅ **Implement Security Headers:**
```php
// middleware atau server config
return response($content)
    ->header('X-Content-Type-Options', 'nosniff')
    ->header('X-Frame-Options', 'DENY')
    ->header('X-XSS-Protection', '1; mode=block')
    ->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains')
    ->header('Content-Security-Policy', "default-src 'self'")
    ->header('Referrer-Policy', 'strict-origin-when-cross-origin');
```

### 11. Logging & Monitoring

✅ **Security Logging:**
```php
// Log security events
Log::channel('security')->info('Login attempt', [
    'user_id' => $user->id,
    'ip_address' => $request->ip(),
    'user_agent' => $request->userAgent(),
    'status' => 'success',
    'timestamp' => now(),
]);

// Log failed attempts
Log::channel('security')->warning('Failed login attempt', [
    'email' => $request->email,
    'ip_address' => $request->ip(),
    'reason' => 'invalid_credentials',
]);

// Monitor untuk suspicious activities
// - Multiple failed login attempts
// - Access to unauthorized resources
// - Unusual data access patterns
```

### 12. Dependency Management

✅ **Keep Dependencies Updated:**
```bash
# Regular security audit
composer audit
npm audit

# Update dependencies
composer update
npm update

# Check for known vulnerabilities
# Use tools: Snyk, Dependabot, OWASP Dependency-Check
```

## Environment Variables Security

✅ **Never Commit Secrets:**
```bash
# .gitignore WAJIB include:
.env
.env.local
.env.production
*.key
*.pem
secrets/
```

✅ **.env File Structure:**
```env
# Application
APP_NAME="My Application"
APP_ENV=production
APP_DEBUG=false
APP_KEY=base64:random-32-char-key

# Database - JANGAN commit production credentials!
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=mydb
DB_USERNAME=dbuser
DB_PASSWORD=super-secure-password-here

# External APIs
PAYMENT_GATEWAY_KEY=sk_live_xxxxx
SMTP_PASSWORD=xxxxx
```

## 🚨 Self-Enforcement — WAJIB di Setiap Response

**Sebelum menulis kode apa pun yang berhubungan dengan:**
- Input pengguna / form handling
- Database query / SQL
- Password / autentikasi
- API keys / secrets
- File upload
- Output rendering / HTML

**WAJIB cek:**
1. Apakah input sudah divalidasi server-side? JANGAN cuma frontend.
2. Apakah SQL pake prepared statements? JANGAN concatenate string.
3. Apakah output sudah di-escape? JANGAN innerHTML mentah.
4. Apakah password pake bcrypt/argon2? JANGAN MD5/SHA1/plaintext.
5. Apakah secrets disimpen di env? JANGAN hardcode di kode.

**Kalau ragu dengan security implikasi → tanya user dulu. JANGAN asumsi aman.**

---

## Security Checklist

Sebelum deployment, pastikan:

```
Authentication & Authorization
□ Password hashing menggunakan bcrypt/argon2
□ CSRF protection enabled
□ Session timeout implemented
□ Authorization check di setiap endpoint
□ Multi-factor authentication (untuk critical apps)

Input & Output
□ All user input di-validate di server-side
□ Output di-escape untuk prevent XSS
□ File upload validation & virus scanning
□ SQL queries menggunakan prepared statements
□ No eval() atau exec() dengan user input

Data Protection
□ Sensitive data di-encrypt at rest
□ HTTPS/TLS enforced
□ Security headers configured
□ Database credentials tidak di-hardcode
□ API keys di environment variables

Monitoring & Logging
□ Security events logged
□ Failed login attempts tracked
□ Anomaly detection enabled
□ Regular security audit
□ Backup & disaster recovery plan

Dependencies
□ All dependencies up-to-date
□ No known vulnerabilities (npm audit, composer audit)
□ Using official packages only
□ License compliance checked
```

---

## 13. Sandbox Security

> Bagian ini nge-cover security dari sisi execution sandbox — sandbox modes, approval policies, dan permission overrides. Referensi dari `feature-workflow` dan `sandbox-security`.

### Sandbox Modes

| Mode | Proteksi | Use Case |
|------|----------|----------|
| **Bubble Wrap** | Ringan — baca/tulis file diizinkan | Daily dev, exploratory |
| **Seatbelt** | Ketat — semua operasi butuh approval | Production, untrusted codebase |

**Rule:** Production task WAJIB pake Seatbelt.

### Approval Policies

| Policy | Behavior |
|--------|----------|
| **Untrusted** | Semua operasi berbahaya ditanya — safety first |
| **Request** | Agent saran, user approve/reject — collaborative |
| **Never** | Auto-mode tanpa izin — CI/CD only |

🔴 **JANGAN** pakai Never policy di interactive session. Risiko unintended code execution tinggi.

### Permission Overrides

| Override | Efek | Safety |
|----------|------|--------|
| Auto Mode | Skip approval — gas langsung | ⚠️ Only di CI/CD |
| Full Access | Allow semua operasi | 🔴 Risiko tinggi |

**Kombinasi terlarang:** Full Access + Never Policy + Interactive Session = NO SAFETY NET.

**Detail selengkapnya:** Lihat skill `sandbox-security` — flowchart sandbox, OS-specific settings, network access control, dan safety rules.

---

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [CWE Top 25](https://cwe.mitre.org/top25/)
- [Security Headers](https://securityheaders.com/)
- [Sandbox Security](skills/sandbox-security/SKILL.md) — internal skill reference
