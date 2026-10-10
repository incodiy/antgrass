---
name: openapi-first
description: Use when designing or implementing REST APIs. Enforces OpenAPI specification before coding, proper schema definitions, and API versioning.
version: 1.0.0
updated: 2026-06-21
---

# OpenAPI-First Development

## Tujuan
Menggunakan OpenAPI sebagai konsep utama/dasar untuk acuan alur dan schema data dalam setiap project development.

## Prinsip OpenAPI-First

### Mengapa OpenAPI-First?
- **Contract-Driven Development**: API contract didefinisikan dulu sebelum implementasi
- **Documentation as Code**: Dokumentasi API selalu up-to-date
- **Cross-Team Collaboration**: Frontend & Backend bisa develop parallel
- **Code Generation**: Auto-generate client SDK, server stubs, validation
- **Testing & Mocking**: Bisa test & mock API sebelum implementasi selesai

## Workflow

```
1. Define OpenAPI Spec → 2. Review & Approve → 3. Generate Code → 4. Implement → 5. Validate
```

### 1. Define OpenAPI Spec

Setiap project **HARUS** punya file OpenAPI spec di:
```
docs/openapi/
├── openapi.yaml (atau openapi.json)
└── schemas/
    ├── user.yaml
    ├── product.yaml
    └── order.yaml
```

### 2. OpenAPI Structure

```yaml
openapi: 3.0.3
info:
  title: Nama Project API
  description: Deskripsi lengkap project dan tujuannya
  version: 1.0.0
  contact:
    name: Team Name
    email: team@example.com

servers:
  - url: http://localhost:8000/api/v1
    description: Development
  - url: https://staging-api.example.com/api/v1
    description: Staging
  - url: https://api.example.com/api/v1
    description: Production

tags:
  - name: Authentication
    description: Endpoint untuk autentikasi pengguna
  - name: Users
    description: Manajemen pengguna
```

paths:
  /auth/login:
    post:
      tags:
        - Authentication
      summary: Login pengguna
      description: Endpoint untuk autentikasi pengguna dengan email dan password
      operationId: loginUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LoginRequest'
      responses:
        '200':
          description: Login berhasil
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LoginResponse'
        '401':
          description: Kredensial tidak valid
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '422':
          description: Validasi gagal
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ValidationErrorResponse'

components:
  schemas:
    LoginRequest:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
          example: user@example.com
        password:
          type: string
          format: password
          minLength: 8
          example: SecurePassword123
```

    LoginResponse:
      type: object
      properties:
        success:
          type: boolean
          example: true
        message:
          type: string
          example: Login berhasil
        data:
          type: object
          properties:
            token:
              type: string
              example: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
            user:
              $ref: '#/components/schemas/User'
    
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: 550e8400-e29b-41d4-a716-446655440000
        email:
          type: string
          format: email
          example: user@example.com
        name:
          type: string
          example: John Doe
        created_at:
          type: string
          format: date-time
          example: 2024-01-15T10:30:00Z
    
    ErrorResponse:
      type: object
      properties:
        success:
          type: boolean
          example: false
        message:
          type: string
          example: Terjadi kesalahan
        error_code:
          type: string
          example: AUTH_FAILED
    
    ValidationErrorResponse:
      type: object
      properties:
        success:
          type: boolean
          example: false
        message:
          type: string
          example: Validasi gagal
        errors:
          type: object
          additionalProperties:
            type: array
            items:
              type: string

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

### 3. Best Practices

#### Gunakan $ref untuk Reusability
```yaml
# Bad: Duplikasi schema
paths:
  /users:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: string
                  name:
                    type: string

# Good: Gunakan $ref
paths:
  /users:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
```

#### Pisahkan ke File Terpisah (untuk project besar)
```yaml
# openapi.yaml
components:
  schemas:
    User:
      $ref: './schemas/user.yaml'
    Product:
      $ref: './schemas/product.yaml'
```

#### Standard Response Format
Gunakan format response yang konsisten:
```json
{
  "success": true,
  "message": "Operasi berhasil",
  "data": { ... },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100
  }
}
```

### 4. Tools untuk OpenAPI

#### Documentation & UI
- **Swagger UI**: Interactive API documentation
- **ReDoc**: Clean, responsive API documentation
- **Stoplight**: API design & documentation platform

#### Code Generation
- **OpenAPI Generator**: Generate server stubs, client SDKs
- **Swagger Codegen**: Multi-language code generator

#### Validation
- **Spectral**: OpenAPI linter
- **openapi-validator**: Validate spec against OpenAPI standard

#### Contoh penggunaan:
```bash
# Generate PHP Laravel controller
openapi-generator-cli generate \
  -i docs/openapi/openapi.yaml \
  -g php-laravel \
  -o generated/

# Generate TypeScript client
openapi-generator-cli generate \
  -i docs/openapi/openapi.yaml \
  -g typescript-axios \
  -o frontend/src/api/

# Validate OpenAPI spec
spectral lint docs/openapi/openapi.yaml
```

### 5. Integration dengan Development

#### Dalam Project:
1. **Buat OpenAPI spec PERTAMA** sebelum coding
2. **Review dengan team** (Frontend, Backend, QA)
3. **Generate code** dari spec
4. **Implement business logic** berdasarkan generated code
5. **Validate** response sesuai dengan spec

#### Reference OpenAPI dalam Kode:
```php
/**
 * Login User
 * 
 * @OA\Post(
 *     path="/api/v1/auth/login",
 *     operationId="loginUser",
 *     tags={"Authentication"},
 *     @OA\RequestBody(ref="#/components/schemas/LoginRequest"),
 *     @OA\Response(response=200, ref="#/components/responses/LoginResponse")
 * )
 */
public function login(LoginRequest $request)
{
    // Implementation
}
```

### 6. Versioning API

Gunakan versioning di URL:
```yaml
servers:
  - url: http://localhost:8000/api/v1
    description: Version 1
  - url: http://localhost:8000/api/v2
    description: Version 2 (dengan breaking changes)
```

### 7. Documentation dalam Project

File OpenAPI **HARUS** disimpan di:
```
docs/
├── openapi/
│   ├── openapi.yaml
│   ├── schemas/
│   │   ├── user.yaml
│   │   ├── product.yaml
│   │   └── order.yaml
│   └── README.md (panduan cara pakai OpenAPI spec)
└── api/
    └── collection.json (Postman/Insomnia collection)
```

### 8. Testing dengan OpenAPI

Gunakan spec untuk automated testing:
```javascript
// Example: Validate API response against OpenAPI spec
const OpenAPIValidator = require('express-openapi-validator');

app.use(
  OpenAPIValidator.middleware({
    apiSpec: './docs/openapi/openapi.yaml',
    validateRequests: true,
    validateResponses: true,
  })
);
```

## Interactive Design Mode

Ketika user minta buat endpoint API baru, jalankan workflow interaktif ini:

### Step 1 — Scan Existing Spec

Baca `docs/openapi/openapi.yaml` yang ada, identifikasi:
- Endpoints yang sudah ada (biar gak duplikasi)
- Schema/components yang sudah ada (bisa di-`$ref`)
- Pattern response yang dipake (success, error, pagination)

### Step 2 — Tanya Detail Endpoint (Satu per Satu)

Gunakan `question` tool untuk tiap pertanyaan:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API Design — Step 1/5: Method & Path

Method yang tersedia: GET, POST, PUT, DELETE, PATCH

Pilihan kamu:
[1] GET    — read data
[2] POST   — create data
[3] PUT    — update data
[4] DELETE — hapus data
[5] PATCH  — partial update
[6] ✏️ Tulis detail — kustom
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
API Design — Step 2/5: Tag & Summary

Existing tags yang tersedia:
- Authentication
- Users
- Projects
- News

Pilih tag atau buat baru?
[1] Pakai tag existing → (tampilkan daftar)
[2] ✏️ Tulis detail — tag kustom + summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Lanjut ke pertanyaan berikutnya:
- **Step 3**: Request body fields (tanya satu per satu: nama, type, required, example)
- **Step 4**: Response fields (success response, error response)
- **Step 5**: Auth & security (Bearer token? Rate limit?)

### Step 3 — Generate YAML & Show Preview

Setelah semua terjawab, generate YAML snippet dan tampilkan ke user:

```yaml
paths:
  /api/v1/users:
    get:
      tags: [Users]
      summary: Daftar semua pengguna
      parameters:
        - name: page
          in: query
          schema: { type: integer }
      responses:
        '200':
          description: Berhasil
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PaginatedUsers'
```

Tanya user:
```
[1] ✅ Setuju — generate spec + code
[2] ✏️ Tulis detail — revisi manual
[3] ❌ Batal
```

### Step 4 — Generate Code

Jika user setuju, generate:
1. Update `docs/openapi/openapi.yaml` dengan endpoint baru
2. Types di `lib/types/index.ts` (kalau ada schema baru)
3. Data dummy di `lib/data/` (kalau perlu)
4. Route handler di `app/api/v1/[resource]/route.ts`

### Step 5 — Final Report

```
📋 API Design Report — Selesai

Endpoint: GET /api/v1/users
Schema:   User, PaginatedUsers
Files:
  ✅ docs/openapi/openapi.yaml — spec updated
  ✅ lib/types/index.ts — 2 new types
  ✅ lib/data/users.ts — dummy data
  ✅ app/api/v1/users/route.ts — handler

Next: Implement business logic di route handler
```

## Integrasi dengan solo-dev-workflow

Jika skill `solo-dev-workflow` aktif (file `.opencode/phase_state/projects.json` ada):

| Gate | Perilaku OpenAPI |
|------|-----------------|
| **G1 (Architecture)** | Perubahan OpenAPI spec **diperbolehkan** — ini fase desain API contract |
| **G3+ (Development/Deployment)** | Perubahan OpenAPI spec WAJIB trigger warning (lihat di bawah) |

**Warning saat modify spec di G3+:**

```
┌─────────────────────────────────────────────────────┐
│ ⚠️ API Contract Modification Detected                 │
│                                                      │
│ Project ini sudah melewati G1 (Architecture).         │
│ Mengubah API Contract bisa berdampak ke backend,      │
│ frontend, dan mobile yang sudah jalan.                │
│                                                      │
│ [1] 🔄 Revisit G1 — review arsitektur dulu           │
│ [2] 💪 Override — ubah spec, terima konsekuensi      │
│ [3] ❌ Batal — jangan ubah                           │
└─────────────────────────────────────────────────────┘
```

Jika user pilih Override → catat di `decisions.md` dengan format standar.

---

## Checklist OpenAPI-First

Sebelum mulai coding, pastikan:
```
□ OpenAPI spec sudah dibuat dan complete
□ All endpoints terdefinisi dengan jelas
□ Request/Response schemas sudah detail
□ Error responses sudah didefinisikan
□ Authentication/Authorization sudah di-specify
□ Spec sudah di-validate (no errors)
□ Team sudah review dan approve spec
□ Documentation sudah di-generate (Swagger UI/ReDoc)
```

