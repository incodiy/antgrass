---
name: data-migration
description: Skill untuk migrasi dari data dummy/hardcoded ke real database. Mencakup assessment, schema design, strategi migrasi, execution, validation, performance tuning, dan cleanup. Wajib dipakai saat ada transisi dari mock data ke production database.
version: 1.0.0
updated: 2026-06-21
loads: [database-design, performance-optimization]
---

# Data Migration — Migrasi dari Dummy ke Database Real

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI TANPA PENGECUALIAN

**Skill ini adalah panduan WAJIB untuk setiap proses migrasi data dari sumber dummy/hardcoded menuju database real.** Satu kesalahan dalam migrasi data bisa menyebabkan data korup, hilang permanen, inkonsistensi bisnis, atau downtime produksi yang tidak bisa diperbaiki.

🔴 **Konsekuensi nyata jika dilanggar:**
- **Data mapping salah** → data hilang atau terkorupsi permanen, recovery butuh waktu berhari-hari
- **Tidak backup sebelum migrasi** → rollback impossible, data asli lenyap
- **Transformasi tanpa validasi** → inkonsistensi data, laporan bisnis salah, keputusan manajemen keliru
- **Migration tanpa dry-run** → production langsung crash, user tidak bisa akses aplikasi
- **Hardcoded data tidak dicabut** → dual-source of truth, data stale, kebingungan tim
- **Tidak ada rollback plan** → migrasi gagal tapi gak bisa balik, bencana production

**Model WAJIB** mematuhi setiap aturan di bawah. **JANGAN skip, JANGAN tawar-menawar, JANGAN terima asumsi.**

---

## Daftar Isi

1. [Tujuan & Scope](#1-tujuan--scope)
2. [Core Principles](#2-core-principles)
3. [Data Source Categorization](#3-data-source-categorization)
4. [Stay-as-Is Decision Framework](#4-stay-as-is-decision-framework)
5. [Phase 0 — Assessment & Inventory](#5-phase-0--assessment--inventory)
6. [Phase 1 — Data Modeling & Schema Design](#6-phase-1--data-modeling--schema-design)
7. [Phase 2 — Migration Strategy Catalog](#7-phase-2--migration-strategy-catalog)
8. [Phase 3 — Execution](#8-phase-3--execution)
9. [Phase 4 — Replacement & Integration](#9-phase-4--replacement--integration)
10. [Phase 5 — Validation Contract System](#10-phase-5--validation-contract-system)
11. [Phase 6 — Performance & Caching](#11-phase-6--performance--caching)
12. [Phase 7 — Legacy Cleanup & Rollback](#12-phase-7--legacy-cleanup--rollback)
13. [Edge Cases Catalog](#13-edge-cases-catalog)
14. [Automation Scripts](#14-automation-scripts)
15. [Integration with Other Skills](#15-integration-with-other-skills)
16. [Anti-Patterns](#16-anti-patterns)
17. [Self-Enforcement](#17-self-enforcement)
18. [Final Checklist](#18-final-checklist)

---

## 1. Tujuan & Scope

### 1a. Tujuan

Skill ini bertujuan untuk memastikan **setiap migrasi data dari sumber dummy/hardcoded menuju database real** berjalan dengan aman, terstruktur, dan dapat di-revert. Proses migrasi bukan sekadar copy-paste data — melainkan transformasi yang mempertahankan integritas data bisnis.

### 1b. Scope (Apa yang Dicakup)

| Area | Cakupan |
|------|---------|
| **Sumber Data** | Hardcoded arrays, JSON files, mock objects, CSV/Excel, API response cache, legacy database |
| **Target Database** | PostgreSQL, MySQL, SQLite, MongoDB, Firestore, Supabase |
| **Jenis Data** | Master data (lookup), transactional data, configuration, user data, relationship data |
| **Lingkup Teknis** | Schema design, ETL pipeline, data transformation, validation, rollback, caching strategy |

### 1c. Out of Scope (Apa yang TIDAK Dicakup)

- Migrasi antar-database production tanpa downtime (database-level replication) — lihat skill database-design
- Data warehousing / big data ETL skala enterprise (Apache Spark, Airflow)
- Realtime data sync CDC (Change Data Capture) — hanya batch migration
- Data privacy compliance (GDPR, UU PDP) — lihat skill security-first


## 2. Core Principles

Lima prinsip yang **WAJIB** dipegang dalam setiap migrasi data.

### 2a. Prinsip #1: Data is Gold — Jangan Pernah Hilangkan Data

> **Data yang sudah ada adalah aset. Transformasi BOLEH mengubah format, tapi JANGAN menghilangkan informasi.**

**Salah — Data hilang saat transformasi:**
```typescript
// Data asli: { fullName: 'Budi Santoso' }
const transformed = {
  firstName: data.fullName.split(' ')[0],
  // lastName hilang!
};
```

**Benar — Pertahankan semua informasi:**
```typescript
const [firstName, ...rest] = data.fullName.split(' ');
const transformed = {
  firstName,
  lastName: rest.join(' ') || null,
  fullName: data.fullName,
};
```

### 2b. Prinsip #2: Setiap Migrasi Wajib Reversible

> **Setiap langkah migrasi HARUS bisa di-rollback ke keadaan sebelumnya.**

**Salah — Migration langsung ubah data tanpa backup:**
```sql
-- JANGAN!
UPDATE users SET email = LOWER(email);
```

**Benar — Backup dulu sebelum mutasi:**
```sql
CREATE TABLE users_backup_20260101 AS SELECT * FROM users;
UPDATE users SET email = LOWER(email);
-- Rollback: DROP TABLE users;
-- ALTER TABLE users_backup_20260101 RENAME TO users;
```

### 2c. Prinsip #3: Validasi Sebelum, Saat, dan Sesudah

> **JANGAN percaya data akan benar — validasi di setiap tahap.**

```python
def validate_source(data: list[dict]) -> list[str]:
    errors = []
    for i, row in enumerate(data):
        if not row.get('id'):
            errors.append(f'Row {i}: missing id')
        if row.get('email') and '@' not in row['email']:
            errors.append(f'Row {i}: invalid email {row["email"]}')
    return errors
```

### 2d. Prinsip #4: Satu Source of Truth (SSoT)

> **Setelah migrasi selesai, HARUS hanya ada satu sumber data. Data lama WAJIB dinonaktifkan.**

**Salah — Dual source of truth:**
```typescript
// Fallback ke data lama — JANGAN!
const user = await db.query('SELECT * FROM users WHERE id = ?', [id])
  ?? hardcodedUsers.find(u => u.id === userId);
```

**Benar — Single source of truth:**
```typescript
const user = await userService.findById(userId);
// Data lama sudah dihapus
```

### 2e. Prinsip #5: Migration adalah Code — Version Controlled

> **Skrip migrasi WAJIB masuk git, direview, dan bisa di-replay.**

```
migrations/
├── 2026-01-01-initial-schema.sql
├── 2026-01-01-seed-master-data.sql
├── 2026-01-02-migrate-users.sql
├── 2026-01-02-rollback-users.sql
├── 2026-01-03-migrate-products.sql
├── 2026-01-03-rollback-products.sql
└── validate/
    ├── validate-count.sql
    └── validate-integrity.sql
```


---

## 3. Data Source Categorization

**WAJIB** kategorikan sumber data sebelum menentukan strategi migrasi.

### Tipe A: Hardcoded Array / Object

Data didefinisikan langsung di kode.

```typescript
const COUNTRIES = [
  { code: 'ID', name: 'Indonesia' },
  { code: 'MY', name: 'Malaysia' },
];
```

| Karakteristik | Approach |
|---------------|----------|
| Ukuran kecil (< 100 rows) | Seed langsung di migration |
| Statis, jarang berubah | Bisa jadi lookup table |
| Biasanya reference data | Transform minimal |

### Tipe B: JSON / YAML / Config File

Data di file konfigurasi.

```json
[
  { "id": "prod-1", "name": "Product A", "price": 25000 }
]
```

| Karakteristik | Approach |
|---------------|----------|
| Ukuran sedang (100 - 10K rows) | ETL dengan script |
| Bisa nested/complex | Flattening & normalization |
| Sering version controlled | Migration script reusable |

### Tipe C: Mock Object / Factory

Data dari factory/mock library (Faker, Factory Bot).

| Karakteristik | Approach |
|---------------|----------|
| Generated, gak permanen | Gak perlu migrasi — langsung seeding |
| Untuk testing | Buat seeder baru untuk production |
| Format tidak stabil | Buat schema dulu, baru seeder |

### Tipe D: CSV / Excel Export

Data dari spreadsheet, laporan, export tool.

| Karakteristik | Approach |
|---------------|----------|
| Flat / tabular | Import dengan parser, validasi tipe |
| Sering dirty (inconsistent format) | WAJIB cleansing step |
| No relationships | Tambah foreign key mapping |

### Tipe E: Legacy Database

Data dari database lain (SQLite, Access, DB lama).

| Karakteristik | Approach |
|---------------|----------|
| Sudah punya schema | Reverse engineering schema |
| Punya relationships | Preserve referential integrity |
| Ukuran besar | Batch pagination, progress tracking |

---

## 4. Stay-as-Is Decision Framework

**Tidak SEMUA data perlu dimigrasi.** Gunakan decision tree berikut:

```
Data masih dipakai di production?
  YES ──┬── Sering berubah? ── YES ── WAJIB MIGRASI
         │                    └── NO ── STAY AS-IS (dokumentasi)
  NO  ──┬── Bisa diakses via API? ── YES ── BISA STAY (cache strategy)
         │                          └── NO ── WAJIB MIGRASI (API wrapper gak cukup)
         └── Arsipkan / Hapus
```

**Stay-as-Is** hanya valid jika:
1. Data benar-benar statis dan tidak berubah
2. Tidak ada dual-source of truth
3. Tidak perlu query/filter dari data tersebut
4. Ada dokumentasi kenapa data ini tidak dimigrasi


---

## 5. Phase 0 — Assessment & Inventory

> 🔄 **Mode Interaktif** — Setiap langkah, WAJIB gunakan `question` tool untuk tanya user sebelum eksekusi.

### 5a. Data Inventory — Interactive Flow

```
Step 1: Tanya user — scan mode
Q: "Mau gue scan codebase untuk inventory dummy data?"
[1] ✅ Auto-scan (rekomendasi) — jalankan script scanner
[2] ✏️ Manual — user jelasin manual
[3] 🔄 Hybrid — auto scan + manual completion

Step 2: Tampilkan hasil ke user setelah scan
📊 INVENTORY SCAN COMPLETE
  Type A (Hardcoded):   N files
  Type B (Config):      N files
  Type C (Mock/Test):   N files
  Type D (Env):         N files
  Type E (Mixed):       N files
  Top files by dependency count: ...

Step 3: Tanya user untuk tiap entity
Q: "Gue temukan entity berikut. Setuju?"
[1] ✅ Setuju — lanjut ke priority scoring
[2] ✏️ Tambah/kurang entity
[3] 🔄 Rescan — mungkin ada yang kelewat

Step 4: Priority matrix
Q: "Impact × Effort scoring. Setuju dengan prioritas ini?"
[1] ✅ Setuju — execute sesuai prioritas
[2] ✏️ Adjust prioritas manual
```

Jalankan script scanner (Section 14) jika user pilih auto-scan:

```bash
python scan_hardcoded_data.py --path ./src --format json
```

### 5b. Source Code Scanner

Jalankan script Automation (Section 14):

```bash
python scan_hardcoded_data.py --path ./src --format json
```

### 5c. Inventory Output

Setelah assessment, **WAJIB** buat tabel:

| # | Data Entity | Source Type | Volume | Target Table | Priority | Strategy |
|---|-------------|-------------|--------|--------------|----------|----------|
| 1 | User Roles | A (Hardcoded) | 5 rows | roles | High | Seed |
| 2 | Products | B (JSON) | 200 rows | products | High | ETL |
| 3 | Orders | C (Mock) | 0 (generated) | orders | Medium | Seeder |
| 4 | Categories | A (Hardcoded) | 15 rows | categories | High | Seed |
| 5 | Old Reports | D (CSV) | 5000 rows | reports_archive | Low | Batch Import |

---

## 6. Phase 1 — Data Modeling & Schema Design

> 🔄 **Mode Interaktif** — Tanya user untuk setiap keputusan schema.

### 6a. Entity Relationship Mapping — Interactive Flow

```
Step 1: Tanya user — entity extraction
Q: "Gue extract entity berikut dari struktur data existing. Setuju?"
[1] ✅ Setuju — lanjut ke type mapping
[2] ✏️ Tambah/kurang entity
[3] 🔄 Rescan — mungkin ada yang kelewat

Step 2: Type mapping confirmation
Q: "Mapping tipe data untuk entity User. Setuju?"
    id: number → UUID
    name: string → VARCHAR(255) NOT NULL
    email: string → VARCHAR(255) UNIQUE NOT NULL
    role: string → UUID FK → roles.id
[1] ✅ Setuju
[2] ✏️ Edit mapping

Step 3: Relationship inference
Q: "Gue deteksi relationship berikut dari data patterns:"
    Product.category_id → Category.id (N:1)
    Order.user_id → User.id (N:1)
    OrderItem.order_id → Order.id (N:1)
[1] ✅ Setuju — generate migration files
[2] ✏️ Edit relationships
[3] 🔄 Belum lengkap — tambah manual
```

```typescript
// Data sumber: hardcoded orders
const orders = [
  { id: 'ORD-001', items: ['PROD-A', 'PROD-B'], userId: 'USR-1' }
];
// Target:
// orders(id, order_number, user_id, created_at, status)
// order_items(id, order_id, product_id, quantity, price)
```

### 6b. Type Mapping Rules

| Source Type   | PostgreSQL       | MySQL          | MongoDB     |
|---------------|------------------|----------------|-------------|
| string        | VARCHAR(n)/TEXT  | VARCHAR(n)/TEXT| string      |
| number (int)  | INTEGER/BIGINT   | INT/BIGINT     | number      |
| number (float)| DECIMAL(10,2)    | DECIMAL(10,2)  | number      |
| boolean       | BOOLEAN          | TINYINT(1)     | boolean     |
| Date string   | TIMESTAMPTZ      | DATETIME       | Date        |
| Array         | JSONB/TABLE      | JSON/TABLE     | array       |
| Object        | JSONB            | JSON           | object      |

### 6c. Schema Design Checklist

**Prinsip inti:** UUID PKs, foreign keys, indexes, timestamps, soft delete. 
**Detail lengkap:** Lihat skill `database-design` — schema patterns, indexing, constraints, data types.

### 6d. SQL Schema Example

Lihat skill `database-design` untuk contoh schema lengkap dengan UUID PKs, foreign keys, indexes, dan audit fields.


---

## 7. Phase 2 — Migration Strategy Catalog

> 🔄 **Mode Interaktif** — Pilih strategy per modul, bukan satu untuk semua.

### 7a. Strategy Selection — Interactive Flow

```
Step 1: Tanya user — strategy per modul
Q: "Rekomendasi strategy berdasarkan analisis:"
    Module      Size  Complexity  Recommend
    ───────     ────  ──────────  ──────────
    Users       50    Low         Dual-Write
    Products    1000  Medium      Batch ETL
    Statuses    10    Low         Big Bang
    Settings    30    Low         Lazy Migration
    Orders      5000  High        Feature Flag + Dual-Write
[1] ✅ Setuju — execute sesuai rekomendasi
[2] ✏️ Adjust per module
[3] Pilih manual semua

Step 2: Rollback chain design
Q: "Rollback chain untuk Users module:"
[1] ✅ Simpan backup + DOWN script (rekomendasi)
[2] ✏️ Custom rollback plan
```

### 7b. Strategy Catalog

| Strategy | Downtime | Complexity | Best For |
|----------|----------|------------|----------|
| **Big Bang** | Yes | Low | Small data, reference tables |
| **Dual-Write** | No | High | Production, zero-downtime |
| **Lazy Migration** | No | Medium | Per-module access pattern |
| **Feature Flag** | No | Medium | Rollout bertahap, easy rollback |
| **Shadow Read** | No | High | High confidence needed |
| **Batch ETL** | Maybe | High | Jutaan rows, complex transform |

### 7c. Strategy Detail

**Big Bang**: All old → Transform → Insert → Cutover. Cocok untuk data lookup yang jarang berubah.

**Dual-Write**:
```
Phase 1: Write to BOTH old + new, read from OLD
Phase 2: Write to BOTH old + new, read from NEW (validate)
Phase 3: Write to NEW only, remove OLD write
Phase 4: Remove OLD read + OLD data source
```

**Lazy Migration**: On first access, cek DB → gak ada? ambil dummy → simpan DB → return. Backfill di background.

**Feature Flag**:
```
if (USE_DB) return db.query(...)
return dummyUsers
Rollout: 10% → monitor → 50% → 100% → remove flag
```

**Shadow Read**: Read from OLD (main), shadow read from NEW. Compare & log. No mismatch for X days → cutover.

**Batch ETL**: Extract → Transform → Load (1000 rows/batch). Resume-able, retry-able.

### Direct Seed (Tipe A -- volume kecil)

```python
ROLES = [
    {'code': 'super_admin', 'name': 'Super Admin'},
    {'code': 'admin', 'name': 'Admin'},
    {'code': 'user', 'name': 'User'},
]

async def seed_roles():
    db = await get_db()
    for role in ROLES:
        await db.execute("""
            INSERT INTO roles (code, name)
            VALUES ($1, $2)
            ON CONFLICT (code) DO UPDATE
            SET name = EXCLUDED.name, updated_at = NOW()
        """, role['code'], role['name'])
    print(f'Seeded {len(ROLES)} roles')
```

### Strategi #2: ETL Pipeline (Tipe B & D -- volume sedang)

```python
import json, psycopg2

def extract(filepath):
    with open(filepath) as f:
        return json.load(f)

def transform(products):
    result = []
    for p in products:
        result.append((
            p['id'], p['name'],
            int(p['price'] * 100),
            p.get('category', 'uncategorized'),
        ))
    return result

def load(conn, data):
    with conn.cursor() as cur:
        cur.executemany("""
            INSERT INTO products (id, name, price_cents, category)
            VALUES (%s, %s, %s, %s)
            ON CONFLICT (id) DO UPDATE
            SET name = EXCLUDED.name, price_cents = EXCLUDED.price_cents
        """, data)
    conn.commit()

def run_etl():
    conn = psycopg2.connect('dbname=mydb user=admin')
    try:
        raw = extract('data/products.json')
        data = transform(raw)
        load(conn, data)
        print(f'ETL selesai: {len(data)} products')
    finally:
        conn.close()
```

### Strategi #3: Batch Pagination (Tipe E -- volume besar)

```typescript
const BATCH_SIZE = 1000;
let offset = 0, total = 0;

while (true) {
  const batch = await sourceDb.query(
    'SELECT * FROM orders ORDER BY id LIMIT $1 OFFSET $2',
    [BATCH_SIZE, offset]
  );
  if (batch.rows.length === 0) break;

  const client = await targetDb.beginTransaction();
  try {
    for (const row of batch.rows) {
      await client.query(
        'INSERT INTO orders (id, user_id, total_cents, status) VALUES ($1,$2,$3,$4)',
        [row.id, row.user_id, row.total_cents, row.status]
      );
    }
    await client.commit();
  } catch (err) {
    await client.rollback();
    throw err;
  }
  offset += BATCH_SIZE;
  total += batch.rows.length;
  console.log(`Progress: ${total} orders`);
}
```

### Strategi #4: Seeder Replacement (Tipe C -- mock data)

```typescript
import { faker } from '@faker-js/faker';

export async function seedOrders(count = 100) {
  const orders = Array.from({ length: count }, () => ({
    userId: faker.string.uuid(),
    productId: faker.string.uuid(),
    quantity: faker.number.int({ min: 1, max: 10 }),
    totalCents: faker.number.int({ min: 1000, max: 100000 }),
    status: faker.helpers.arrayElement(['pending', 'completed', 'cancelled']),
  }));
  await db.insert('orders').values(orders);
}
```

### Strategi #5: Upsert / Merge

```sql
INSERT INTO users (id, email, full_name, role_id)
VALUES ($1, $2, $3, $4)
ON CONFLICT (email) DO UPDATE
SET full_name = EXCLUDED.full_name,
    role_id = EXCLUDED.role_id,
    updated_at = NOW();
```

### Strategi #6: Staging Table Pattern

```sql
CREATE TABLE staging_products (LIKE products INCLUDING ALL);
\copy staging_products FROM 'products.csv' WITH CSV HEADER;
INSERT INTO products (id, name, price_cents, category_id, created_at)
SELECT s.id, s.name, s.price_cents, c.id, s.created_at
FROM staging_products s LEFT JOIN categories c ON c.code = s.category_code;
DROP TABLE staging_products;
```


---

## 8. Phase 3 — Execution

> 🔄 **Mode Interaktif** — Step-by-step, tiap batch dikonfirmasi user.

### 8a. Execution Flow

```
Step 1: Pre-flight check
Q: "Semua persiapan udah siap?"
  ✅ Schema migration udah jalan?
  ✅ Backup data asli udah dibuat?
  ✅ Rollback script udah siap?
[1] ✅ Gas execute!
[2] ⏸ Cek lagi

Step 2: Dry-run
Jalankan migration di mode dry-run (tanpa write ke production DB)
Q: "Dry-run sukses: 47 records akan di-insert. Lanjut?"
[1] ✅ Lanjut ke real execution
[2] 🔄 Debug — ada error

Step 3: Real execution
Monitor tiap batch:
  Batch 1/5: 1000 records ✅
  Batch 2/5: 1000 records ✅
  ...
Q: "Migration selesai. 47 records inserted. Verify?"
```

### 8a. Pre-Migration Checklist (WAJIB Sebelum Eksekusi)

```
 Backup database target sudah dilakukan?
 Dry-run sudah berhasil tanpa error?
 Source data sudah di-freeze / gak berubah selama migrasi?
 Semua foreign key constraints sudah siap?
 Migration script sudah di-review oleh minimal 1 orang lain?
 Rollback script sudah siap dan teruji?
 Waktu execution sudah dipilih (luar jam sibuk)?
 Monitoring / logging sudah disiapkan?
```

### 8b. Migration Execution Flow

```python
import time, logging
from contextlib import contextmanager

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger('migration')

@contextmanager
def migration_step(name):
    logger.info(f'Starting: {name}')
    start = time.time()
    try:
        yield
        elapsed = time.time() - start
        logger.info(f'Completed: {name} ({elapsed:.2f}s)')
    except Exception as e:
        logger.error(f'Failed: {name} -- {str(e)}')
        raise

def run_migration():
    with migration_step('backup'): backup_database()
    with migration_step('schemas'): create_schemas()
    with migration_step('roles'): seed_roles()
    with migration_step('products'): etl_products()
    with migration_step('users'): batch_migrate_users()
    with migration_step('validation'): run_all_validations()
    logger.info('Migration selesai!')
```

### 8c. Dry-Run Mode

**WAJIB** jalankan dry-run sebelum production:

```typescript
async function dryRunMigration() {
  console.log('DRY RUN -- Tidak ada data diubah');
  const testDb = await createTestDatabase();

  for (const step of migrationSteps) {
    try {
      await step.execute(testDb);
      console.log(`  ${step.name} -- OK`);
    } catch (err) {
      console.log(`  ${step.name} -- FAILED: ${err.message}`);
      throw err;
    }
  }

  const validation = await runValidations(testDb);
  console.log(`Validasi: ${validation.passed}/${validation.total} passed`);
  await testDb.drop();
}
```

### 8d. Progress Bar

```python
def show_progress(current, total, bar_length=40):
    percent = current / total
    filled = int(bar_length * percent)
    bar = chr(9608) * filled + chr(9617) * (bar_length - filled)
    print(f'\rProgress: |{bar}| {current}/{total} ({percent:.1%})', end='')
```

---

## 9. Phase 4 — Replacement & Integration

> 🔄 **Mode Interaktif** — Replace hardcoded refs satu per satu, dengan konfirmasi.

### 9a. Replacement Flow

```
Step 1: Tanya user — replace mode
Q: "Mau replace hardcoded references dengan service layer?"
[1] ✅ Auto-replace (detect + replace all known patterns)
[2] ✏️ Manual — ganti satu per satu
[3] 🔄 Hybrid — auto untuk simple, manual untuk complex

Step 2: Embedded logic check
Q: "Gue deteksi potensi embedded logic di data berikut:"
  ORDER_STATUSES — state machine dengan transitions
  PAYMENT_METHODS — fee_percent: 2.5% (calculation logic)
[1] ✅ Extract logic ke code, migrate data doang
[2] ✏️ Review manual — mungkin ada yang perlu di-preserve

Step 3: Verification setelah replace
Q: "Setelah replace, test dulu?"
[1] ✅ Run integration test (recommended)
[2] ⏸ Nanti — masih ada module lain
```

### 9a. Code Replacement Pattern

Setelah data di database, **WAJIB** ganti semua referensi data lama.

**Salah -- Data lama masih dipanggil:**
```typescript
class UserService {
  async getUser(id: string) {
    const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    if (user) return user;
    return HARDCODED_USERS.find(u => u.id === id);  // Fallback -- JANGAN!
  }
}
```

**Benar -- Total replacement:**
```typescript
class UserService {
  async getUser(id: string) {
    const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    if (!user) throw new NotFoundError('User not found');
    return user;
  }
}
// Hapus: src/data/hardcoded-users.ts
// Hapus: src/mocks/users.mock.ts
```

### 9b. Feature Flag Strategy

```typescript
class ProductService {
  async getProducts(filters: ProductFilters) {
    if (await featureFlags.isEnabled('new-db-products')) {
      return this.getFromDatabase(filters);
    }
    return this.getFromHardcoded(filters);
  }
}
// Setelah stabil: hapus flag, pake database permanently
```

### 9c. Integration Testing

```typescript
describe('Migration Integration', () => {
  test('hardcoded data matches database', async () => {
    for (const entity of ['roles', 'categories', 'products']) {
      const dbData = await db.query(`SELECT * FROM ${entity}`);
      const hcData = getHardcodedData(entity);
      expect(dbData.rows.length).toBe(hcData.length);
    }
  });
});
```


---

## 10. Phase 5 — Validation Contract System

> 🔄 **Mode Interaktif** — WAJIB validasi SEBELUM cutover.

### 10a. Validation Flow

```
Step 1: Contract definition
Q: "Gue akan validasi data migrated vs data asli. Setuju dengan parameter berikut?"
  ✅ Record count: source = target
  ✅ Sample content: random 10% records
  ✅ Referential integrity: zero orphans
  ✅ Business rule: semua constraint terpenuhi
[1] ✅ Run all validations
[2] ✏️ Custom — pilih validation yang jalan

Step 2: Validation result
Q: "Validation report:"
  ✅ Record count PASS (47 = 47)
  ✅ Sample content PASS (100% match)
  ✅ Referential integrity PASS (0 orphans)
  ✅ Business rule PASS
[1] ✅ All PASS — ready to cutover!
[2] 🔴 Failed items — debug dulu

Step 3: A/B comparison (optional untuk critical data)
Q: "Mau shadow read comparison dulu?"
[1] ✅ Yes — 24 jam shadow mode
[2] ❌ No — langsung cutover (confidence tinggi)
```

**Setiap migrasi WAJIB** memiliki validation contract.

### 10a. Count Validation

```python
def validate_count(source, table, db_conn):
    target_count = db_conn.fetchval(f'SELECT COUNT(*) FROM {table}')
    assert len(source) == target_count, \
        f'{table}: source={len(source)}, target={target_count}'
    print(f'{table}: {target_count} rows -- OK')
```

### 10b. Content Sampling

```typescript
async function validateSample(sourceData, tableName, sampleRate = 0.1) {
  const sample = sourceData
    .sort(() => Math.random() - 0.5)
    .slice(0, Math.max(1, sourceData.length * sampleRate));
  const errors = [];

  for (const item of sample) {
    const dbItem = await db.query(`SELECT * FROM ${tableName} WHERE id = $1`, [item.id]);
    if (!dbItem.rows[0]) { errors.push(`Missing: ${item.id}`); continue; }
    for (const key of Object.keys(item)) {
      if (['created_at','updated_at'].includes(key)) continue;
      if (String(dbItem.rows[0][key]) !== String(item[key])) {
        errors.push(`${item.id}.${key}: "${item[key]}" vs "${dbItem.rows[0][key]}"`);
      }
    }
  }
  return { passed: errors.length === 0, errors };
}
```

### 10c. Referential Integrity

```sql
-- Orphan records check
SELECT 'orders without users', COUNT(*) as violations
FROM orders o LEFT JOIN users u ON o.user_id = u.id WHERE u.id IS NULL;

SELECT 'products without categories', COUNT(*) as violations
FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE c.id IS NULL;
```

### 10d. Business Logic Validation

```typescript
async function validateBusinessRules() {
  const rules = [
    {
      name: 'Total pesanan = jumlah item * harga',
      check: async () => {
        const result = await db.query("""
          SELECT o.id FROM orders o JOIN (
            SELECT order_id, SUM(quantity * price_cents) as calc_total
            FROM order_items oi JOIN products p ON p.id = oi.product_id
            GROUP BY order_id
          ) calc ON calc.order_id = o.id
          WHERE o.total_cents != calc.calc_total
        """);
        return result.rows.length === 0;
      }
    },
    {
      name: 'Email harus unique',
      check: async () => {
        const result = await db.query(
          'SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1'
        );
        return result.rows.length === 0;
      }
    },
  ];
  // Run all rules
  for (const rule of rules) {
    const passed = await rule.check();
    console.log(`${passed ? 'PASS' : 'FAIL'}: ${rule.name}`);
  }
}
```

### 10e. Validation Report

```
+------------------------------------------------------+
| VALIDATION REPORT -- 2026-01-01 14:30:00 UTC         |
+------------------+-----------+-----------+-----------+
| Entity           | Count OK  | Sample OK | Integrity |
+------------------+-----------+-----------+-----------+
| roles            | OK 5/5    | OK 5/5    | N/A       |
| categories       | OK 15/15  | OK 15/15  | N/A       |
| products         | OK 200/200| OK 20/20  | OK 200/200|
| users            | OK 150/150| OK 15/15  | OK 150/150|
| orders           | OK 500/500| OK 50/50  | 3 orphan  |
+------------------+-----------+-----------+-----------+
| Overall: 4/5 entities passed                        |
| Orders: 3 orphan items -- perlu cleanup             |
+------------------------------------------------------+
```

---

## 11. Phase 6 — Performance & Caching

> 🔄 **Mode Interaktif** — Kompensasi latency DB setelah migrasi.

### 11a. Performance Assessment Flow

```
Step 1: Assess impact
Q: "Performance impact assessment:"
  Before (hardcoded): 0.01ms
  After (DB query):   2-50ms
  Impact: [Read-heavy: HIGH] [Admin: LOW] [Real-time: CRITICAL]
[1] ✅ Implement cache + connection pool
[2] ⏸ Optimize query dulu sebelum cache

Step 2: Cache strategy
Q: "Rekomendasi caching: in-memory cache (TTL 5 menit)"
[1] ✅ Setuju — implement
[2] ✏️ Custom — Redis / different TTL
```

### 11b. Index Strategy

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 'abc' AND status = 'completed';
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
```

### 11b. Connection Pooling

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  database: process.env.DB_NAME,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 5000,
});

export async function query(text: string, params?: any[]) {
  const start = Date.now();
  const result = await pool.query(text, params);
  const duration = Date.now() - start;
  if (duration > 1000) {
    console.warn(`Slow query (${duration}ms): ${text.substring(0, 100)}`);
  }
  return result;
}
```

### 11c. Caching Strategy & Optimization

**Prinsip inti:** Cache data after migration to compensate DB latency. In-memory cache with TTL, connection pooling, query optimization.
**Detail lengkap:** Lihat skill `performance-optimization` — caching strategies, query optimization, indexing, connection pooling.


---

## 12. Phase 7 — Legacy Cleanup & Rollback

> 🔄 **Mode Interaktif** — Hapus dengan aman, jangan asal delete.

### 12a. Legacy Cleanup — Interactive Flow

```
Step 1: Confirmation phase
Q: "Konfirmasi legacy cleanup untuk module User:"
  ✅ Data migrated ke DB (47 records)
  ✅ Validation passed
  ✅ No active consumer of hardcoded source
  ✅ Rollback plan ready (backup di /backups/)
[1] ✅ Hapus legacy files & push
[2] ⏸ Tahan dulu — want to test prod first

Step 2: Feature flag removal
Q: "Feature flag USE_DB — sudah 100% rollout?"
[1] ✅ Remove flag — kode bersih
[2] ⏸ Keep flag — masih ada yang pake OLD

Step 3: Cleanup execution
Q: "Gue detect file berikut siap dihapus:"
  src/data/users_legacy.ts
  src/constants/order_statuses.ts
  src/mocks/product_data.mock.ts (keep for tests?)
[1] ✅ Delete all (keep mocks for tests)
[2] ✏️ Pilih manual — hapus sebagian
[3] ⏸ Archive dulu — soft delete
```

### 12b. Legacy Cleanup Checklist

Setelah migrasi sukses dan divalidasi:

```
 Hapus file hardcoded data:
   - src/data/hardcoded-*.ts
   - src/mocks/*.mock.ts (kecuali testing)
   - src/constants/*.ts (berisi data)
 Hapus feature flag (kalau ada)
 Update dokumentasi -- hapus referensi data lama
 Archive backup di storage terpisah (S3 / Google Drive)
 Notifikasi tim -- data lama sudah tidak dipakai
```

### 12b. Rollback Plan

**Setiap migration WAJIB punya rollback plan teruji.**

```bash
#!/bin/bash
# rollback.sh
set -euo pipefail
echo "ROLLBACK: Memulai proses..."

# 1. Restore database dari backup
pg_restore -d mydb --clean backup_20260101.dump

# 2. Restore kode lama
git checkout HEAD~1 -- src/data/
git checkout HEAD~1 -- src/constants/

# 3. Hapus migration artifacts
rm -rf migrations/2026-01-01-*

# 4. Restart service
pm2 restart all

echo "ROLLBACK SELESAI"
```

### 12c. Rollback Decision Matrix

```
Validasi setelah migrasi
    |
    +-- PASS ------> Selesai, semua aman
    +-- FAIL minor -> Fix forward (patch, gak perlu rollback)
    +-- FAIL crit --> ROLLBACK (restore backup)
```

Threshold rollback:
- **Critical**: Data hilang, integrity broken -> ROLLBACK
- **Major**: >5% mismatch, business rule violation -> ROLLBACK
- **Minor**: <1% mismatch, formatting -> Fix forward
- **Cosmetic**: Tampilan, label -> Fix forward

---

## 13. Edge Cases Catalog

### Edge Case #1: Data Duplicate di Source

```typescript
const source = [
  { code: 'ADMIN', name: 'Administrator' },
  { code: 'ADMIN', name: 'Admin' },  // Duplicate!
];

function deduplicate(data, key, strategy) {
  const map = new Map();
  for (const item of data) {
    const k = item[key];
    if (!map.has(k) || strategy === 'last') map.set(k, item);
    else if (strategy === 'merge') map.set(k, { ...map.get(k), ...item });
  }
  return [...map.values()];
}
```

### Edge Case #2: Null / Undefined Values

```typescript
function normalizeNulls(data, field) {
  return data.map(item => ({ ...item, [field]: item[field] ?? null }));
}
```

### Edge Case #3: Type Mismatch

```typescript
function parsePrice(value) {
  if (!value || value.trim() === '') return null;
  const cleaned = value.replace(/[,.]/g, '');
  const parsed = parseInt(cleaned, 10);
  return isNaN(parsed) ? null : parsed;
}
```

### Edge Case #4: Circular References

```typescript
function safeStringify(obj) {
  const seen = new WeakSet();
  return JSON.stringify(obj, (key, value) => {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) return '[Circular]';
      seen.add(value);
    }
    return value;
  }, 2);
}
```

### Edge Case #5: Data Encoding

```python
import chardet
def fix_encoding(filepath):
    with open(filepath, 'rb') as f:
        raw = f.read()
        detected = chardet.detect(raw)
        if detected['encoding'] != 'utf-8':
            return raw.decode(detected['encoding']).encode('utf-8').decode('utf-8')
        return raw.decode('utf-8')
```

### Edge Case #6: Time-Sensitive Data

```typescript
async function migrateWithTimestamp(orders) {
  const now = new Date();
  for (const order of orders) {
    order.status = order.updated_at > now
      ? order.status
      : await getCurrentStatus(order.id);
    order.migrated_at = now;
    await targetDb.insertOrder(order);
  }
}
```

### Edge Case #7: Large BLOB Data

```typescript
// Streaming untuk data besar
async function migrateLargeData() {
  const stream = sourceDb.getStream('SELECT * FROM articles');
  let count = 0;
  for await (const row of stream) {
    await targetDb.insert('articles', transformArticle(row));
    count++;
    if (count % 100 === 0) console.log(`Progress: ${count}`);
  }
}
```

### Edge Case #8: Live Migration (Dual-Write)

```typescript
class DualWriteService {
  async createUser(data) {
    const user = await db.query('INSERT INTO users... RETURNING *');
    await legacyDb.query('INSERT INTO users...'); // Fallback
    return user;
  }

  async getUser(id) {
    let user = await db.query('SELECT * FROM users WHERE id=$1', [id]);
    if (user) return user;
    const legacy = await legacyDb.query('SELECT * FROM users WHERE id=$1', [id]);
    if (legacy) {
      await db.query('INSERT INTO users...', [...legacy]); // Sync
      return legacy;
    }
    throw new Error('User not found');
  }
}
```


---

## 14. Automation Scripts

### 14a. Python Scanner -- Deteksi Hardcoded Data

```python
#!/usr/bin/env python3
"""
Scan project untuk hardcoded data arrays/objects.
Usage: python scan_hardcoded_data.py --path ./src
"""
import os, re, json, argparse
from dataclasses import dataclass, asdict
from typing import List

@dataclass
class HardcodedData:
    filepath: str
    line: int
    name: str
    type: str
    size: int
    preview: str

PATTERNS = {
    'typescript': [
        r'(?:const|let|var)\s+(\w+)\s*[=:]\s*\[',
        r'(?:const|let|var)\s+(\w+)\s*[=:]\s*\{',
    ],
    'python': [
        r'(\w+)\s*=\s*\[',
        r'(\w+)\s*=\s*\{',
    ],
}

def scan_file(filepath):
    results = []
    ext = os.path.splitext(filepath)[1]
    patterns = PATTERNS.get('typescript' if ext in ('.ts','.tsx','.js','.jsx') else
                           'python' if ext == '.py' else '', [])
    if not patterns:
        return results
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            lines = f.readlines()
    except:
        return results
    for i, line in enumerate(lines, 1):
        for p in patterns:
            m = re.search(p, line)
            if m:
                commas = line.count(',')
                results.append(HardcodedData(
                    filepath=filepath, line=i, name=m.group(1),
                    type='array' if '[' in line else 'object',
                    size=commas + 1, preview=line.strip()[:80]
                ))
    return results

def scan_directory(path):
    results = []
    for root, dirs, files in os.walk(path):
        dirs[:] = [d for d in dirs if d not in ('node_modules','dist','.git','__pycache__')]
        for file in files:
            results.extend(scan_file(os.path.join(root, file)))
    return results

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--path', required=True)
    parser.add_argument('--format', choices=['table','json'], default='table')
    parser.add_argument('--output')
    args = parser.parse_args()
    results = scan_directory(args.path)
    report = json.dumps([asdict(r) for r in results], indent=2) if args.format == 'json' else \
        '\n'.join(f'{r.filepath}:{r.line} -- {r.name} ({r.size} elemen)' for r in results)
    if args.output:
        with open(args.output, 'w') as f: f.write(report)
    else:
        print(report)

if __name__ == '__main__':
    main()
```

### 14b. Node.js Schema Extractor

```typescript
#!/usr/bin/env node
/**
 * Extract schema dari hardcoded data, hasilkan SQL DDL.
 * Usage: npx ts-node extract-schema.ts --file data/products.json
 */

function inferType(values: any[]): string {
  const nonNull = values.filter(v => v != null);
  if (nonNull.length === 0) return 'TEXT';
  const types = new Set(nonNull.map(v => typeof v));
  if (types.has('boolean')) return 'BOOLEAN';
  if (types.has('number')) {
    return nonNull.every(v => Number.isInteger(v)) ? 'INTEGER' : 'DECIMAL';
  }
  if (types.has('string')) {
    const maxLen = Math.max(...nonNull.map(v => String(v).length));
    return maxLen > 255 ? 'TEXT' : `VARCHAR(${Math.max(maxLen * 2, 50)})`;
  }
  return 'JSONB';
}

function extractSchema(data: Record<string, any>[], tableName: string) {
  const keys = new Set<string>();
  data.forEach(row => Object.keys(row).forEach(k => keys.add(k)));
  const columns = [];
  for (const key of keys) {
    const values = data.map(row => row[key]);
    columns.push({
      name: key.replace(/([A-Z])/g, '_$1').toLowerCase().replace(/^_/, ''),
      type: inferType(values),
      nullable: values.filter(v => v != null).length < values.length,
    });
  }
  const hasId = columns.some(c => c.name === 'id');
  return { tableName, columns, hasId };
}

function generateDDL(schema) {
  const cols = schema.columns.map(c =>
    `  ${c.name} ${c.type}${c.nullable ? '' : ' NOT NULL'}`
  );
  if (schema.hasId) cols.push('  PRIMARY KEY (id)');
  return `CREATE TABLE ${schema.tableName} (\n${cols.join(',\n')}\n);`;
}

// CLI
const filePath = process.argv[2];
if (!filePath) { console.error('Usage: extract-schema.ts <file>'); process.exit(1); }
const data = require(filePath);
const array = Array.isArray(data) ? data : [data];
const schema = extractSchema(array, 'untitled');
console.log('/* Rows:', array.length, '*/');
console.log(generateDDL(schema));
```

### 14c. Python Migration Validator

```python
#!/usr/bin/env python3
"""
Validasi komprehensif hasil migrasi.
Usage: python validate_migration.py --source data.json --table users --db postgresql://...
"""
import json, argparse, sys, random
from typing import List, Dict
from dataclasses import dataclass, field

try:
    import psycopg2, psycopg2.extras
except ImportError:
    psycopg2 = None

@dataclass
class Result:
    passed: bool = True
    checks: int = 0
    passed_checks: int = 0
    errors: List[str] = field(default_factory=list)

class Validator:
    def __init__(self, source, table, db_url, sample_rate=0.1):
        self.source = source
        self.table = table
        self.db_url = db_url
        self.sample_rate = sample_rate
        self.result = Result()

    def load(self):
        with open(self.source) as f:
            data = json.load(f)
        return data if isinstance(data, list) else [data]

    def connect(self):
        if not psycopg2:
            raise ImportError('Install psycopg2-binary')
        return psycopg2.connect(self.db_url)

    def validate_count(self, source, conn):
        self.result.checks += 1
        with conn.cursor() as cur:
            cur.execute(f'SELECT COUNT(*) FROM {self.table}')
            target = cur.fetchone()[0]
        if len(source) != target:
            self.result.errors.append(f'Count: {len(source)} vs {target}')
            return False
        self.result.passed_checks += 1
        return True

    def validate_sample(self, source, conn):
        self.result.checks += 1
        sample = random.sample(source, max(1, int(len(source) * self.sample_rate)))
        errors = []
        for item in sample:
            id_val = item.get('id')
            if not id_val: continue
            with conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor) as cur:
                cur.execute(f'SELECT * FROM {self.table} WHERE id = %s', (id_val,))
                row = cur.fetchone()
            if not row:
                errors.append(f'Missing: {id_val}')
                continue
            for k, v in item.items():
                if k in ('created_at','updated_at','migrated_at'): continue
                if str(v) != str(row.get(k)):
                    errors.append(f'{id_val}.{k}: "{v}" vs "{row.get(k)}"')
        if errors:
            self.result.errors.extend(errors[:5])
            return False
        self.result.passed_checks += 1
        return True

    def run(self):
        source = self.load()
        conn = self.connect()
        try:
            self.validate_count(source, conn)
            self.validate_sample(source, conn)
        finally:
            conn.close()
        self.result.passed = self.result.passed_checks == self.result.checks
        status = 'PASSED' if self.result.passed else 'FAILED'
        print(f'{status}: {self.result.passed_checks}/{self.result.checks}')
        return self.result

def main():
    p = argparse.ArgumentParser()
    p.add_argument('--source', required=True)
    p.add_argument('--table', required=True)
    p.add_argument('--db', required=True)
    p.add_argument('--sample', type=float, default=0.1)
    args = p.parse_args()
    v = Validator(args.source, args.table, args.db, args.sample)
    r = v.run()
    sys.exit(0 if r.passed else 1)

if __name__ == '__main__':
    main()
```


---

## 15. Integration with Other Skills

Skill ini **WAJIB** digunakan bersama skill lain untuk hasil optimal:

| Skill | Kapan Digunakan |
|-------|-----------------|
| database-design | Sebelum Phase 1 (schema design) |
| security-first | Migration script -- prevent SQL injection |
| senior-backend-engineer | Phase 4 (replacement) -- arsitektur clean |
| code-standards | Migration script -- coding style konsisten |
| error-handling | Migration execution -- error handling robust |
| performance-optimization | Phase 6 (performance) -- query optimization |
| git-workflow | Migration script di-commit properly |

**WAJIB** aktivasi database-design SEBELUM Phase 1, dan security-first SEBELUM migration script menyentuh data sensitif.

---

## 16. Anti-Patterns

### Anti-Pattern #1: Migrasi Langsung di Production Tanpa Dry-Run

```
 Satu query salah bisa menghapus data berharga.
 WAJIB dry-run dulu di staging. JANGAN pernah pertama kali di production.
```

### Anti-Pattern #2: Skip Validasi

```
 Data source sering punya inkonsistensi yang tidak kelihatan.
 Setiap tahap migrasi WAJIB divalidasi.
```

### Anti-Pattern #3: Tanpa Rollback Plan

```
 Kalau migrasi gagal di tengah jalan, data korup dan gak bisa balik.
 Setiap migration WAJIB punya rollback script teruji.
```

### Anti-Pattern #4: Dual Source of Truth

```
 Aplikasi jadi tidak konsisten -- kadang data baru, kadang data lama.
 HAPUS semua referensi ke data lama setelah migrasi.
```

### Anti-Pattern #5: Data Dumping Tanpa Transformasi

```
 Data kotor di database menyebabkan error aplikasi dan laporan bisnis salah.
 WAJIB cleansing dan transformasi sebelum load.
```

### Anti-Pattern #6: Migrasi Raksasa Sekaligus

```
 Satu error di tengah batch 10.000 data bikin semua gagal.
 Pecah per entity, batch maksimal 1.000 rows.
```

### Anti-Pattern #7: Lupa Update Service Layer

```
 API response berubah format, frontend error.
 Integration test yang memverifikasi response setelah migrasi.
```

### Anti-Pattern #8: Tidak Ada Performance Testing

```
 Query yang dulu cepat di hardcoded (in-memory) jadi lambat di database (disk I/O).
 WAJIB test performance dengan EXPLAIN ANALYZE setelah migrasi.
```


---

## 17. Self-Enforcement

**Sebelum menulis kode apa pun yang berhubungan dengan migrasi data, WAJIB cek:**

### Check 1: Assessment
- [ ] Apakah saya sudah melakukan data inventory bersama user? (Phase 0)
- [ ] Apakah saya sudah mengkategorikan tiap sumber data? (Tipe A-E)
- [ ] Apakah ada data yang sebaiknya stay-as-is? (Section 4)

### Check 2: Perencanaan
- [ ] Apakah schema database sudah didesain dan direview?
- [ ] Apakah migration strategy sudah dipilih untuk tiap entity?
- [ ] Apakah rollback plan sudah siap?

### Check 3: Execution
- [ ] Apakah backup sudah dilakukan SEBELUM migrasi?
- [ ] Apakah dry-run sudah berhasil 100%?
- [ ] Apakah migration script menggunakan batch processing?
- [ ] Apakah ada error handling untuk setiap step?

### Check 4: Validasi
- [ ] Apakah count validation sudah jalan? (source = target)
- [ ] Apakah sample content validation sudah jalan?
- [ ] Apakah referential integrity sudah dicek?
- [ ] Apakah business rule validation sudah jalan?

### Check 5: Cleanup
- [ ] Apakah data sumber lama sudah dinonaktifkan?
- [ ] Apakah feature flag sudah dicabut?
- [ ] Apakah dokumentasi sudah diupdate?
- [ ] Apakah performance testing sudah dilakukan?

### Check 6: Mental Check
- [ ] Apakah saya yakin data tidak ada yang hilang?
- [ ] Apakah saya sudah menerapkan 5 Core Principles?
- [ ] Apakah ada anti-pattern yang saya langgar?
- [ ] Apakah saya siap melakukan rollback jika diperlukan?

**Kalau ragu dengan salah satu langkah -> tanya user dulu. JANGAN lanjut tanpa kepastian.**

### Self-Prompt (Automatic Enforcement)

Setiap kali user meminta migrasi data, model **WAJIB** merespons dengan:

> Saya akan membantu migrasi data dari sumber dummy/hardcoded ke database. Sebelum mulai, saya perlu melakukan assessment dulu -- **bisa kita lakukan Data Inventory (Phase 0)?** Saya akan tanya beberapa pertanyaan untuk memetakan data yang perlu dimigrasi.

---

## 18. Final Checklist

Sebelum menyelesaikan proses migrasi data, **WAJIB** pastikan semua item berikut:

```
PHASE 0 -- ASSESSMENT
  Data inventory sudah dilakukan
  Setiap data sudah dikategorikan (Tipe A-E)
  Stay-as-Is decision sudah diambil
  Priority sudah ditentukan

PHASE 1 -- SCHEMA DESIGN
  Schema database sudah dibuat dan direview
  Indices sudah diidentifikasi
  Foreign key constraints sudah terdefinisi
  Timestamps (created_at, updated_at) ada
  UUID strategy sudah ditentukan

PHASE 2 -- STRATEGY
  Migration strategy sudah dipilih per entity
  Rollback script sudah siap
  Backup plan sudah ada

PHASE 3 -- EXECUTION
  Backup sudah dilakukan
  Dry-run sudah sukses
  Migration berjalan tanpa error
  Logging tercatat dengan baik

PHASE 4 -- REPLACEMENT
  Semua referensi data lama sudah diganti
  Feature flag (jika ada) sudah dicabut
  Integration test sudah jalan
  API response sudah sesuai format

PHASE 5 -- VALIDATION
  Count validation: source = target
  Sample content validation: minimal 10%
  Referential integrity: zero orphan records
  Business rule validation: semua pass

PHASE 6 -- PERFORMANCE
  Query performance sudah dicek (EXPLAIN ANALYZE)
  Index yang diperlukan sudah dibuat
  Connection pooling sudah dikonfigurasi
  Caching strategy sudah diimplementasikan

PHASE 7 -- CLEANUP
  Hardcoded data files sudah dihapus
  Backup sudah diarsipkan
  Dokumentasi sudah diperbarui
  Tim sudah diberitahu

SELF-ENFORCEMENT
  5 Core Principles sudah diterapkan
  Tidak ada anti-pattern yang dilanggar
  Validation contract sudah terpenuhi
  Rollback plan siap dieksekusi

  SEMUA ITEM DI ATAS SUDAH TERPENUHI -- MIGRASI SIAP DIPRODUKSIKAN
```
