# Code Standards & Best Practices Rule

> **Status:** Global Rule (Always Active)
> **Tujuan:** Menjaga kebersihan, konsistensi, dan maintainability seluruh codebase.

---

## 1. Prinsip Utama Rekayasa Perangkat Lunak

### DRY (Don't Repeat Yourself)
- Selalu periksa apakah helper, utility, atau fungsi serupa sudah ada sebelum membuat fungsi baru.
- Refactor duplikasi logika menjadi fungsi terpusat di `common/`, `utils/`, atau `services/`.

### SOLID & Clean Architecture
- **Single Responsibility (SRP):** Satu modul/fungsi/class hanya memiliki satu tanggung jawab spesifik.
- **Dependency Inversion:** Pisahkan interface/schema dari implementasi konkrit (gunakan repository pattern & service layer).
- **Keterbacaan Fungsi:** Fungsi harus ringkas, fokus (maksimal 20–30 baris), dan maksimal 3–4 parameter (bungkus ke DTO/Object/Pydantic Model jika lebih).

---

## 2. Standar Penamaan (*Naming Conventions*)

### Python / FastAPI
```python
# Class / Model / Schema: PascalCase
class UserProfile(BaseModel):
    pass

# Function / Method / Variable: snake_case
async def get_user_by_id(user_id: UUID) -> UserResponse:
    user_name = "Ahmad"
    return user_name

# Constant: UPPER_SNAKE_CASE
MAX_RETRY_ATTEMPTS = 3
DEFAULT_PAGE_SIZE = 20
```

### TypeScript / Next.js / React Native
```typescript
// Type / Interface / Component: PascalCase
export type AuthResponse = { token: string; user: User }
export function AyahDisplayCard({ ayah }: AyahProps) {}

// Function / Hook / Variable: camelCase
export async function fetchSurahList() {}
export const useReadingProgress = () => {}
const isCompleted = true

// Constant: UPPER_SNAKE_CASE
export const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL
```

### Database (PostgreSQL)
```sql
-- Tabel: snake_case (plural)
CREATE TABLE ayah_translations;

-- Kolom: snake_case
surah_id, text_uthmani, created_at;

-- Primary Key: Wajib UUID v7 / gen_random_uuid()
id UUID PRIMARY KEY DEFAULT gen_random_uuid();
```

---

## 3. Aturan Bebas Mock & Fail-Fast
- **Zero Mock Policy:** Dilarang meletakkan dummy data / fallback JSON statis di sisi frontend untuk menutupi kegagalan API. Tampilkan *ErrorState* jujur dengan tombol coba lagi (*retry*).
- **Fail-Fast Configuration:** Semua konfigurasi wajib dimuat dari *environment variables* via config loader (`.env`). Jika env variable penting tidak ditemukan, aplikasi wajib menolak start (*crash immediately*) dengan pesan yang jelas.
