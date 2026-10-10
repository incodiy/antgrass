# Security-First & Compliance Rule

> **Status:** Global Rule (Always Active)
> **Tujuan:** Menjamin integritas keamanan data, perlindungan PII, dan kepatuhan privasi (UU PDP / GDPR).

---

## 1. Prinsip Dasar Keamanan

1. **Defense in Depth:** Terapkan keamanan berlapis (Input Validation $\rightarrow$ Authentication & RBAC $\rightarrow$ Parameterized Query $\rightarrow$ Encryption $\rightarrow$ Rate Limiting).
2. **Principle of Least Privilege:** Setiap endpoint dan service hanya memiliki hak akses minimal yang diperlukan.
3. **Fail Securely:** Default aksi adalah `DENY`. Pesan error ke client tidak boleh membocorkan detail stack trace internal database atau credentials.

---

## 2. Pencegahan Kerentanan Umum

### SQL Injection Prevention
- Dilarang keras melakukan string concatenation pada query SQL.
- Wajib menggunakan ORM (SQLAlchemy / Prisma) atau *Parameterized Prepared Statements*.

### XSS & Output Sanitization
- Escape semua output HTML user-generated.
- Gunakan rendering React yang aman (`textContent`) dan hindari `dangerouslySetInnerHTML` tanpa sanitasi (DOMPurify).

### Secrets & Credentials Management
- **Dilarang keras melakukan hardcode API key, secret token, atau password di dalam kode sumber.**
- Semua secrets wajib dibaca dari file `.env`.
- File `.env` wajib masuk dalam `.gitignore` dan tidak pernah di-commit ke Git. Commit hanya file template `.env.example`.

### Autentikasi & Otorisasi
- Password wajib di-hash menggunakan algoritma modern (`bcrypt` atau `argon2id`). Dilarang keras MD5/SHA1/Plaintext.
- Gunakan JWT Access Token pendek (15–30 menit) + Refresh Token persisten yang di-hash dan dirotasi di database.
- Terapkan OAuth 2.0 PKCE untuk client mobile dan SPA.

---

## 3. Kepatuhan Privasi (UU PDP)
- Registrasi akun wajib menyertakan persetujuan eksplisit terhadap *Privacy Policy* (*consent*).
- Sediakan fitur hak ekspor data pribadi (`/auth/me/export`) dan hak penghapusan akun (`/auth/me/delete-request`).
