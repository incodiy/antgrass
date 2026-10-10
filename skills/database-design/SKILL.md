---
name: database-design
description: Use when designing database schemas, writing migrations, or creating tables. Enforces UUID PKs, snake_case naming, 3NF normalization, proper indexing, and soft delete patterns.
version: 1.0.0
updated: 2026-06-21
---

# Database Design Standards

## ⚠️ PERINGATAN — HARUS DIIKUTI

**Aturan di skill ini WAJIB dipatuhi.** Desain database yang buruk adalah salah satu kesalahan termahal dalam software engineering — migration kompleks, data korup, performa anjlok, dan biaya perbaikan tinggi.

🔴 **Konsekuensi jika dilanggar:**
- Gak pake UUID → enumerasi ID, celah security
- Gak pake soft delete (`deleted_at`) → data hilang permanen, gak bisa recovery
- Gak pake index → query lambat, aplikasi terasa lemot, scalability mati
- Gak pake 3NF → data redundant, update anomaly, inconsistency
- Gak bikin ERD → tim bingung struktur data, development molor

Model **WAJIB** membaca dan mematuhi setiap aturan di bawah.

---

## Tujuan
Mendesain database dengan matang dan terukur agar dapat secara presisi handling flow data yang dinamis dengan scalability dan maintainability yang baik.

## Prinsip Dasar

### 1. UUID sebagai Primary Key (WAJIB)
- **SELALU** gunakan UUID untuk semua tabel
- Alasan: Security, scalability, distributed systems friendly
- Format: UUID v4 (random)

```sql
-- PostgreSQL
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- MySQL 8.0+
CREATE TABLE users (
    id BINARY(16) PRIMARY KEY DEFAULT (UUID_TO_BIN(UUID())),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. Naming Conventions

#### Tables
- **snake_case** (lowercase dengan underscore)
- **Plural** untuk entity tables
- Descriptive dan jelas

```sql
✅ Benar:
- users
- order_items
- product_categories
- user_login_histories

❌ Salah:
- Users (PascalCase)
- orderItem (camelCase)
- tbl_user (prefix tidak perlu)
```

#### Columns
- **snake_case**
- Descriptive names
- Avoid abbreviations

```sql
✅ Benar:
- first_name
- email_address
- created_at
- is_active

❌ Salah:
- fname (terlalu singkat)
- emailAddr (camelCase)
- createdAt (camelCase)
```

#### Foreign Keys
- Format: `referenced_table_singular_id`

```sql
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL,
    product_id UUID NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

#### Indexes
- Format: `idx_tablename_columnname` atau `idx_tablename_col1_col2`

```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

### 3. Standard Columns (Timestamps & Audit)

Setiap tabel **HARUS** memiliki:

```sql
CREATE TABLE example_table (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Business columns here
    name VARCHAR(255) NOT NULL,
    
    -- Audit fields (WAJIB)
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL, -- Untuk soft delete
    
    -- Optional audit fields
    created_by UUID NULL,
    updated_by UUID NULL,
    deleted_by UUID NULL,
    
    FOREIGN KEY (created_by) REFERENCES users(id),
    FOREIGN KEY (updated_by) REFERENCES users(id),
    FOREIGN KEY (deleted_by) REFERENCES users(id)
);
```

### 4. Normalization

#### Terapkan 3NF (Third Normal Form)
- Eliminasi redundant data
- Setiap non-key column harus depend on the key
- Gunakan lookup tables untuk data referensi

**Contoh Buruk (Denormalized):**
```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    customer_name VARCHAR(255),
    customer_email VARCHAR(255),
    customer_phone VARCHAR(20),
    customer_address TEXT,
    -- Data customer duplikat di setiap order
);
```

**Contoh Baik (Normalized):**
```sql
CREATE TABLE customers (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    address TEXT
);

CREATE TABLE orders (
    id UUID PRIMARY KEY,
    customer_id UUID NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

### 5. Relationships

#### One-to-Many
```sql
-- One user has many orders
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT
);
```

#### Many-to-Many (Junction Table)
```sql
-- Students can enroll in many courses
-- Courses can have many students

CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL
);

CREATE TABLE courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_name VARCHAR(255) NOT NULL
);

-- Junction table
CREATE TABLE student_courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL,
    course_id UUID NOT NULL,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    grade DECIMAL(3,2) NULL,
    
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE,
    
    -- Prevent duplicate enrollments
    UNIQUE KEY unique_enrollment (student_id, course_id)
);

CREATE INDEX idx_student_courses_student ON student_courses(student_id);
CREATE INDEX idx_student_courses_course ON student_courses(course_id);
```

### 6. Indexes Strategy

**Kapan membuat index:**
- ✅ Foreign keys (WAJIB)
- ✅ Columns used in WHERE clauses frequently
- ✅ Columns used in JOIN conditions
- ✅ Columns used in ORDER BY
- ✅ UNIQUE constraints untuk business logic

**Hindari over-indexing:**
- ❌ Columns yang jarang di-query
- ❌ Tables dengan high write/low read ratio
- ❌ Small tables (< 1000 rows)

```sql
-- Composite index untuk query yang sering dipakai
CREATE INDEX idx_orders_user_status_date 
ON orders(user_id, status, created_at);

-- Untuk query: 
-- SELECT * FROM orders 
-- WHERE user_id = ? AND status = ? 
-- ORDER BY created_at DESC;
```

### 7. Data Types Best Practices

```sql
-- String
VARCHAR(N)      -- Variable length, specify max length
TEXT            -- For long text (descriptions, content)
CHAR(N)         -- Fixed length (codes: ISO country codes, etc)

-- Numbers
INT             -- Integer (-2B to 2B)
BIGINT          -- Large integer
DECIMAL(10,2)   -- Fixed precision (money, prices)
FLOAT/DOUBLE    -- Floating point (avoid for money!)

-- Boolean
BOOLEAN         -- atau TINYINT(1) di MySQL

-- Date & Time
DATE            -- Date only
TIME            -- Time only
TIMESTAMP       -- Date + Time + Timezone
DATETIME        -- Date + Time (no timezone)

-- JSON
JSON            -- Native JSON type (PostgreSQL, MySQL 5.7+)

-- Binary
BINARY          -- Fixed length binary
VARBINARY       -- Variable length binary
BLOB            -- Large binary objects
```

**Gunakan tipe data yang tepat:**
```sql
-- ✅ Benar
email VARCHAR(255)
phone VARCHAR(20)
price DECIMAL(10,2)
quantity INT UNSIGNED
is_active BOOLEAN

-- ❌ Salah
email TEXT              -- Too large
phone INT               -- Loses leading zeros, formatting
price FLOAT             -- Precision issues untuk money
quantity VARCHAR(10)    -- String untuk angka?
is_active TINYINT       -- Gunakan BOOLEAN
```

### 8. Constraints

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- NOT NULL untuk fields wajib
    name VARCHAR(255) NOT NULL,
    sku VARCHAR(50) NOT NULL,
    
    -- UNIQUE untuk business keys
    UNIQUE KEY unique_sku (sku),
    
    -- CHECK constraints
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    stock INT NOT NULL DEFAULT 0 CHECK (stock >= 0),
    
    -- ENUM atau CHECK untuk status
    status ENUM('active', 'inactive', 'discontinued') DEFAULT 'active',
    
    -- Foreign key dengan ON DELETE/UPDATE
    category_id UUID NOT NULL,
    FOREIGN KEY (category_id) REFERENCES categories(id) 
        ON DELETE RESTRICT 
        ON UPDATE CASCADE,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NULL
);
```

### 9. Soft Delete Pattern

```sql
-- Gunakan deleted_at untuk soft delete
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL,
    deleted_at TIMESTAMP NULL,
    
    -- Unique constraint yang ignore soft deleted
    UNIQUE KEY unique_active_email (email, deleted_at)
);

-- Query untuk exclude soft deleted
SELECT * FROM users WHERE deleted_at IS NULL;

-- Restore soft deleted
UPDATE users SET deleted_at = NULL WHERE id = ?;
```

### 10. Audit Tables (Log Tables)

Untuk tabel kritikal, buat audit trail:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL
);

-- Audit/History table
CREATE TABLE users_audit (
    audit_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    action ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    
    -- Snapshot of data
    old_data JSON,
    new_data JSON,
    
    -- Audit info
    changed_by UUID,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    
    INDEX idx_audit_user_id (user_id),
    INDEX idx_audit_changed_at (changed_at)
);
```

## 🚨 Self-Enforcement — WAJIB di Setiap Desain Database

**Setiap kali akan membuat tabel/migration/query baru, WAJIB:**

1. **Cek ERD** yang sudah ada — jangan duplikasi struktur
2. **WAJIB pake UUID** sebagai primary key — JANGAN auto-increment integer
3. **WAJIB snake_case** untuk semua nama tabel dan kolom
4. **WAJIB** `created_at`, `updated_at`, `deleted_at` di setiap tabel
5. **WAJIB foreign key index** — jangan lupa index untuk kolom FK
6. **WAJIB 3NF** — cek dulu apakah ada data redundant sebelum finalisasi
7. **WAJIB tanya user** sebelum bikin ERD atau migration — jangan asumsi struktur

**JIKA ada aturan yang dilanggar → koreksi SEBELUM lanjut ke task lain.**

---

## Database Design Checklist

Sebelum finalize design, pastikan:

```
□ Semua tabel menggunakan UUID sebagai PK
□ Naming convention konsisten (snake_case)
□ Semua tabel punya created_at, updated_at, deleted_at
□ Foreign keys sudah didefinisikan dengan proper ON DELETE/UPDATE
□ Indexes dibuat untuk FK dan query patterns
□ Constraints (NOT NULL, UNIQUE, CHECK) sudah tepat
□ Data types sesuai dengan kebutuhan
□ Normalization sudah proper (minimal 3NF)
□ Junction tables untuk many-to-many relationships
□ Audit fields untuk tracking changes (opsional)
```

## ERD (Entity Relationship Diagram)

Untuk setiap project, **WAJIB** buat ERD yang jelas:
- Gunakan tools: dbdiagram.io, draw.io, atau MySQL Workbench
- Simpan di `docs/architecture/ERD.md` (preferred) atau `docs/architecture/ERD.png`
- Include cardinality (1:1, 1:N, N:M)
- Document business rules dalam ERD description

## Migration Files

Gunakan migration tools untuk version control database:
- Laravel: php artisan make:migration
- Django: python manage.py makemigrations
- Node.js: Knex.js, Sequelize migrations

```php
// Laravel migration example
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->uuid('id')->primary();
        $table->string('name');
        $table->decimal('price', 10, 2);
        $table->uuid('category_id');
        $table->timestamps();
        $table->softDeletes();
        
        $table->foreign('category_id')
              ->references('id')
              ->on('categories')
              ->onDelete('restrict');
              
        $table->index(['category_id', 'created_at']);
    });
}
```
