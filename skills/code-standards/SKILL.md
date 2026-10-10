---
name: code-standards
description: Use when writing or reviewing code. Ensures code follows DRY, SOLID, Clean Code principles with proper naming conventions (PHP, TypeScript/JavaScript, React/Next.js, Database) and testing standards. Adapts to your project's tech stack — examples in PHP and JS/TS.
version: 1.0.0
updated: 2026-06-21
---

# Code Standards & Best Practices

## Tujuan
Memastikan setiap kode yang dikembangkan mengikuti standard global programmer convention dengan kualitas tinggi dan mudah di-maintain.

## Prinsip Utama

### 1. DRY (Don't Repeat Yourself)
- **Selalu cek** apakah ada fungsi dengan penamaan dan/atau fungsionalitas yang sama sebelum membuat fungsi baru
- Identifikasi pola yang berulang dan buat reusable function
- Gunakan inheritance, traits, atau composition untuk menghindari duplikasi

**Checklist sebelum membuat fungsi baru:**
```
□ Apakah ada fungsi serupa di codebase?
□ Bisakah fungsi existing di-refactor untuk handle case baru?
□ Apakah fungsi ini akan digunakan di lebih dari satu tempat?
□ Jika ya, tempatkan di utility/helper yang tepat
```

### 2. SOLID Principles
- **S**ingle Responsibility: Satu fungsi/class = satu tanggung jawab
- **O**pen/Closed: Terbuka untuk extension, tertutup untuk modifikasi
- **L**iskov Substitution: Subclass harus bisa menggantikan parent class
- **I**nterface Segregation: Interface spesifik lebih baik dari general
- **D**ependency Inversion: Depend on abstractions, not concretions

### 3. Clean Code
- Nama variabel/fungsi harus **deskriptif dan meaningful**
- Hindari singkatan yang ambigu
- Fungsi harus pendek dan fokus (max 20-30 baris)
- Maksimal 3-4 parameter per fungsi (gunakan object jika lebih)

## Tech-Stack Awareness

This skill adapts to your project's tech stack. Examples show PHP and JS/TS patterns side by side — apply the relevant one for your project.

### Universal Principles
All principles below (DRY, SOLID, Clean Code, naming, structure) apply regardless of language or framework. See your tech-specific sections for concrete examples.

---

## Naming Conventions

### PHP (for Laravel projects)
```php
// Class: PascalCase
class UserAuthentication {}

// Method/Function: camelCase
public function getUserById($id) {}

// Variable: camelCase
$userName = "John";

// Constant: UPPER_SNAKE_CASE
const MAX_LOGIN_ATTEMPTS = 5;

// Interface: PascalCase dengan prefix "I" atau suffix "Interface"
interface IUserRepository {}
interface UserRepositoryInterface {}
```

### JavaScript / TypeScript (for Next.js / React projects)
```typescript
// Types & Interfaces: PascalCase (no I prefix — idiomatic TS)
type UserProfile = { id: string; name: string }
interface AuthResponse { token: string; user: User }

// React Components: PascalCase
function UserProfileCard() {}
const LoginForm: React.FC = () => {}

// Hooks: camelCase, prefixed with "use"
const [user, setUser] = useState<User | null>(null)
const useAuth = () => { /* ... */ }

// Regular functions: camelCase
export async function fetchProjects() {}
const calculateTotal = (items: CartItem[]) => {}

// Variables: camelCase
const itemCount = 10
const isAuthenticated = false

// Constants: UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com'
const MAX_RETRY_COUNT = 3

// Event handlers: camelCase, prefixed with "handle" or "on"
const handleSubmit = (e: FormEvent) => {}
const onChange = (value: string) => {}

// Server Actions / Route Handlers: camelCase
export async function GET(request: NextRequest) {}
export async function createProject(formData: FormData) {}
```

### Database (when applicable)
```sql
-- Table: snake_case (plural)
CREATE TABLE user_accounts;
CREATE TABLE order_items;

-- Column: snake_case
user_id, created_at, first_name

-- Primary Key: SELALU gunakan UUID
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

## File & Folder Structure

> Apply the section relevant to your project's tech stack.

### Backend (PHP/Laravel)
```
project-root/
├── app/
│   ├── Http/
│   │   ├── Controllers/     # PascalCase, suffix "Controller"
│   │   ├── Middleware/
│   │   └── Requests/        # Form validation
│   ├── Models/              # PascalCase, singular
│   ├── Services/            # Business logic, suffix "Service"
│   ├── Repositories/        # Data access, suffix "Repository"
│   └── Helpers/             # Utility functions
├── database/
│   ├── migrations/
│   └── seeders/
├── tests/
│   ├── Unit/
│   └── Feature/
└── docs/
```

### Frontend — Next.js (App Router)
```
project-root/
├── app/                     # App Router pages & layouts
│   ├── layout.tsx           # Root layout
│   ├── page.tsx             # Entry page
│   ├── api/                 # Route Handlers (backend-for-frontend)
│   │   └── v1/
│   └── [slug]/              # Dynamic routes
├── components/
│   ├── ui/                  # Shared UI primitives
│   ├── features/            # Feature-specific components
│   └── layouts/             # Layout components
├── lib/
│   ├── types/               # TypeScript interfaces & types
│   ├── data/                # Data layer (dummy or API helpers)
│   └── utils.ts             # Utility functions
├── hooks/                   # Custom React hooks
├── public/                  # Static assets
└── docs/
    └── openapi/             # OpenAPI schema
```

### Frontend — Generic (React/Vue)
```
src/
├── components/
│   ├── common/             # Button, Input, etc
│   └── features/           # Feature-specific components
├── pages/                  # Page components
├── services/               # API calls
├── hooks/                  # Custom hooks
├── utils/                  # Helper functions
├── store/                  # State management
└── types/                  # TypeScript types/interfaces
```

## Code Quality Checklist

Setiap kali menulis kode, pastikan:

```
□ Tidak ada hardcoded values (gunakan config/env)
□ Tidak ada dummy data
□ Error handling sudah proper
□ Input validation sudah ada
□ Logging di tempat yang strategis
□ Komentar untuk logika kompleks
□ Type hints/declarations (PHP 7.4+, TypeScript)
□ Null safety checks
□ Resource cleanup (close connections, files, etc)
```

## Anti-Patterns yang Harus Dihindari

❌ **Magic Numbers**
```php
// Buruk
if ($user->age > 17) {}

// Baik
const MINIMUM_AGE = 18;
if ($user->age >= MINIMUM_AGE) {}
```

❌ **God Objects/Functions**
```php
// Buruk: Satu function melakukan terlalu banyak
function processOrder($data) {
    // validate
    // calculate
    // save to db
    // send email
    // update inventory
    // create invoice
    // ... 200 lines of code
}

// Baik: Pecah menjadi fungsi-fungsi kecil
function processOrder(OrderData $data) {
    $this->validateOrder($data);
    $order = $this->createOrder($data);
    $this->sendConfirmationEmail($order);
    $this->updateInventory($order);
    return $order;
}
```

❌ **Nested Conditions (Arrow Anti-Pattern)**
```php
// Buruk
if ($user) {
    if ($user->isActive) {
        if ($user->hasPermission) {
            // logic
        }
    }
}

// Baik: Early return
if (!$user || !$user->isActive || !$user->hasPermission) {
    throw new UnauthorizedException();
}
// logic
```

## Documentation dalam Kode

### PHP DocBlock
```php
/**
 * Memproses pembayaran pesanan
 * 
 * @param string $orderId UUID pesanan
 * @param array $paymentData Data pembayaran dari gateway
 * @return Order Model pesanan yang telah diperbarui
 * @throws PaymentException Jika pembayaran gagal
 */
public function processPayment(string $orderId, array $paymentData): Order
{
    // Implementation
}
```

### TypeScript JSDoc / TSDoc
```typescript
/** Processes an order payment and updates inventory. */
export async function processPayment(
  /** UUID of the order */
  orderId: string,
  /** Payment data from the gateway */
  paymentData: PaymentPayload
): Promise<Order> {
  // Implementation
}
```

### React Component (JSDoc)
```typescript
/**
 * Full-screen scene with animated entrance.
 * Used as one of 15 scenes in the cinematic experience.
 * 
 * @param props.activeScene - Current active scene index (0-based)
 * @param props.onNavigate - Callback when user requests navigation
 */
export function SceneContainer({
  activeScene,
  onNavigate,
}: SceneContainerProps) {
  return (
    <AnimatePresence mode="wait">
      {/* ... */}
    </AnimatePresence>
  )
}
```

### API Route Handler (JSDoc)
```typescript
// app/api/v1/projects/route.ts
/**
 * GET /api/v1/projects
 * Returns paginated list of property development projects.
 * Supports ?page=1&limit=10 query parameters.
 */
export async function GET(request: NextRequest) {
  // Implementation
}
```

## Testing Standards

- Setiap fungsi bisnis kritis **HARUS** ada unit test
- Test cases harus cover: happy path, edge cases, error cases
- Mock external dependencies (API, database, etc)
- Test name harus deskriptif

### PHP (PHPUnit/Pest)
```php
// Naming: test_methodName_scenario_expectedResult
public function test_calculateDiscount_withValidCoupon_returnsCorrectAmount()
{
    $order = new Order(['total' => 1000]);
    $coupon = new Coupon(['discount' => 10]);
    
    $result = $this->service->calculateDiscount($order, $coupon);
    
    $this->assertEquals(900, $result);
}
```

### TypeScript (Vitest / Jest)
```typescript
// describe/it pattern
describe('calculateDiscount', () => {
  it('returns discounted total with valid coupon', () => {
    const result = calculateDiscount(1000, { discount: 10 })
    expect(result).toBe(900)
  })

  it('returns original total with no coupon', () => {
    const result = calculateDiscount(1000, null)
    expect(result).toBe(1000)
  })
})
```

### React Component Test (Vitest + Testing Library)
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { NavigationDots } from './NavigationDots'

describe('NavigationDots', () => {
  it('renders correct number of dots', () => {
    render(
      <NavigationDots
        totalScenes={5}
        activeScene={0}
        onDotClick={vi.fn()}
      />
    )
    const dots = screen.getAllByRole('button')
    expect(dots).toHaveLength(5)
  })

  it('calls onDotClick when clicked', () => {
    const onDotClick = vi.fn()
    render(
      <NavigationDots
        totalScenes={3}
        activeScene={0}
        onDotClick={onDotClick}
      />
    )
    fireEvent.click(screen.getAllByRole('button')[2])
    expect(onDotClick).toHaveBeenCalledWith(2)
  })
})
```

### API Route Test (Vitest)
```typescript
import { GET } from './route'

describe('GET /api/v1/projects', () => {
  it('returns paginated project list', async () => {
    const response = await GET(new Request('http://localhost/api/v1/projects'))
    const body = await response.json()
    expect(body.success).toBe(true)
    expect(Array.isArray(body.data)).toBe(true)
    expect(body.meta).toHaveProperty('total')
  })
})
```

---

## Interactive Audit Mode

Ketika user minta audit atau review code, jalankan workflow interaktif ini:

### Step 1 — Scan & Detect Violations

Gunakan `grep` + `glob` untuk scan codebase dan deteksi:
- **Magic numbers**: angka literal (kecuali 0, 1) yang bukan konstanta
- **God functions**: fungsi >30 baris
- **Nested conditions**: `if` di dalam `if` di dalam `if` (3+ level)
- **Duplikasi**: pola kode yang muncul >2 kali
- **Hardcoded values**: string literal yang harusnya config/env
- **Naming violations**: camelCase/snake_case/PascalCase salah
- **Missing type annotations**: parameter atau return tanpa type
- **`any` types**: TypeScript `any` yang bisa diganti specific type

Kelompokkan temuan dalam daftar.

### Step 2 — Tanya User Per Point

Tiap kategori ditanyakan satu per satu ke user pake `question` tool:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Code Audit — Point 1/4: Magic Numbers (5 ditemukan)

  File: src/utils/calculator.ts:15 — `if (age > 17)`
  → Saran: const MINIMUM_AGE = 18

  File: src/services/payment.ts:42 — `return total * 0.1`
  → Saran: const TAX_RATE = 0.1

Saran fix: Extract magic numbers ke constants

Pilihan kamu:
[1] ✅ Setuju — pake saran fix
[2] ❌ Tidak Setuju — skip point ini
[3] ✏️ Tulis detail — instruksi manual
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Setiap point memiliki 3 opsi yang sama:
1. **Setuju** — pake default fix
2. **Tidak Setuju** — skip
3. **Tulis detail** — user kasih instruksi manual

### Step 3 — Record All Answers

Simpan semua jawaban user. Contoh:
```
📋 Code Audit Decisions:
  Point 1 (Magic numbers): ✅ Setuju — 5 konstanta dibuat
  Point 2 (God functions): ✏️ Manual — "skip fungsi calculateTotal, sisanya refactor"
  Point 3 (Nested conditions): ❌ Skip
  Point 4 (Any types): ✅ Setuju — 3 any → specific types
```

### Step 4 — Execute

Hanya eksekusi sesuai jawaban user. Gunakan `edit` tool untuk perubahan kode.

### Step 5 — Final Report

```
📋 Code Audit Report — Selesai

Ringkasan:
  Point 1: Magic numbers — ✅ 5 constants created
  Point 2: God functions — ⏭️ 1 skipped, 2 refactored
  Point 3: Nested conditions — ❌ Skipped
  Point 4: Any types — ✅ 3 any → specific types

Detail perubahan:
  - File: src/utils/calculator.ts:15 — age > 17 → age >= MINIMUM_AGE
  - File: src/services/payment.ts:42 — * 0.1 → * TAX_RATE
  ...
```
