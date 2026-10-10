---
name: agents-template
description: Template for creating AGENTS.md project-specific guidelines. Copy to project root and customize for per-project rules, tech stack, and workflows. WAJIB menyertakan Waypoint & Cross-Session section agar AGENTS.md hasil init ter-sync dengan waypoint skill (OVERVIEW.json, WO archival, hard gate).
version: 1.2.0
updated: 2026-08-05
---

# AGENTS.md - Project-Specific Guidelines Template

> **Catatan**: File ini adalah TEMPLATE untuk membuat AGENTS.md di setiap project.
> Copy file ini ke root project dengan nama AGENTS.md dan sesuaikan dengan kebutuhan project.

---

# Project: [Nama Project]

## Waypoint & Cross-Session (WAJIB — JANGAN DIHAPUS)

> Section ini WAJIB ada di AGENTS.md project. Menjamin project ter-sync dengan skill `waypoint` (lean context system) dan skill `session-history`. Referensi lengkap: `skills/waypoint/SKILL.md`.

### Context System (waypoint)

- **Session init** — baca `.opencode/phase_state/[project-slug]/OVERVIEW.json` (~10 lines) + `ACTIVE_SPRINT.md` (~100-200 lines). JANGAN baca full BLUEPRINT di init.
- **Task classification** — setiap task WAJIB jadi Work Order (BUG/GAP/AUDIT/ENHANCEMENT/REFACTOR/DOC) dengan schema kanonik duration-based (`skills/waypoint/SKILL.md` §3/§5).
- **WO archival** — WO selesai → simpan ke `.opencode/archive/sprints/WO-YYYY-MM-DD-NNN.json` (immutable, JANGAN hapus).
- **Archive structure** — `.opencode/archive/{sprints,phases,handoffs,knowledge}/`.

### Waypoint Hard Gate (WAJIB — Per Phase)

Setiap phase selesai WAJIB menghasilkan 3 artefak — JANGAN di-skip:

1. **WO JSON** di `.opencode/archive/sprints/WO-YYYY-MM-DD-NNN.json` — field wajib: `wo_id`, `type`, `task`, `phase` (int), `status`, `started`, `completed`, `duration_minutes`, `files_changed`, `summary`
2. **handoff entry** di `.opencode/phase_state/[project-slug]/handoff.md` — judul `## YYYY-MM-DD — Phase N: <nama>` + ringkasan What was done
3. **Phase JSON** di `.opencode/archive/phases/phase-NN.json` — field `handoff_ref` diisi WO ID dari poin 1

Wajib run verifier sebelum klaim phase selesai:
```bash
python scripts/waypoint_check.py --check
```

### Session History

- `session-history` skill otomatis jalan di setiap sesi — baca `.opencode/session_chats/current.json` di awal.
- Checkpoint tiap ~5-7 responses. Field `development` di current.json diisi oleh waypoint, JANGAN di-overwrite.

### Repo Scanner

- `python scripts/repo_scan.py` — deteksi struktur repo di root, tulis `.opencode/repo_map.json`.
- `python scripts/repo_scan.py --check` — verifikasi konsistensi (CI gate). Harus PASS sebelum deploy.
- Root project sendiri biasanya BUKAN git repo → `.opencode/` tidak ter-version-control. Gunakan `--check` + WO JSON sebagai enforcement.

### Konvensi `scripts/`

- Tooling project-level (repo scanner, verifier, migrasi, dll) disimpan di `scripts/`.
- Script executable + didokumentasikan di AGENTS.md section ini.
- Script di-copy dari `skills/waypoint/scripts/` saat init — jangan edit langsung versi di skill.

---

## Ringkasan Project

### Tujuan Utama
[Jelaskan tujuan utama dari project ini secara singkat dan jelas]

### Target Users
[Siapa pengguna aplikasi ini? B2B, B2C, internal tools, dll]

### Core Features
1. [Feature 1]
2. [Feature 2]
3. [Feature 3]

## Tech Stack

### Backend
- **Language**: [PHP 8.2, Node.js, Python, dll]
- **Framework**: [Laravel 10, Express, FastAPI, dll]
- **Database**: [PostgreSQL, MySQL, MongoDB, dll]
- **Cache**: [Redis, Memcached, dll]
- **Queue**: [RabbitMQ, Redis Queue, dll]

### Frontend
- **Framework**: [React, Vue, Angular, dll]
- **Language**: [TypeScript, JavaScript]
- **State Management**: [Redux, Zustand, Pinia, dll]
- **UI Library**: [Tailwind, Material-UI, Ant Design, dll]

### Infrastructure
- **Hosting**: [AWS, GCP, Azure, Vercel, dll]
- **CI/CD**: [GitHub Actions, GitLab CI, Jenkins, dll]
- **Monitoring**: [Sentry, New Relic, Datadog, dll]

## Project Structure

```
project-root/
├── app/                     # [Deskripsi]
├── database/               # [Deskripsi]
├── docs/                   # Dokumentasi
├── tests/                  # Test files
└── ...
```

## Development Guidelines

### Code Standards
- [Specific code standards untuk project ini]
- [Naming conventions yang berbeda dari standard, jika ada]
- [Design patterns yang digunakan]

### Database Guidelines
- **Migration**: [Cara membuat dan menjalankan migration]
- **Seeding**: [Data seeding strategy]
- **Specific Rules**: [Aturan khusus untuk project ini]

### API Guidelines
- **Versioning**: [Strategi versioning API]
- **Authentication**: [JWT, OAuth, Session, dll]
- **Rate Limiting**: [Rate limit configuration]
- **Response Format**: [Jika berbeda dari standard]

### Testing Strategy
- **Unit Tests**: [Framework dan lokasi]
- **Integration Tests**: [Scope dan coverage]
- **E2E Tests**: [Tools dan strategy]
- **Coverage Target**: [Minimum coverage requirement]

## Development Workflow

### Setup Development Environment

```bash
# Step-by-step setup
1. Clone repository
2. Install dependencies
3. Configure environment
4. Run migrations
5. Start development server
```

### Running Tests

```bash
# Commands untuk run tests
npm run test
npm run test:coverage
```

### Build & Deployment

```bash
# Build commands
npm run build

# Deployment process
[Jelaskan deployment process]
```

## Project-Specific Rules

### Business Rules
1. [Business rule 1 yang harus diikuti]
2. [Business rule 2]
3. [Business rule 3]

### Technical Constraints
1. [Constraint 1 - misal: max file upload 10MB]
2. [Constraint 2 - misal: support browser IE11+]
3. [Constraint 3]

### Third-Party Integrations
- **[Service Name]**: [Purpose, documentation link, credentials location]
- **[Payment Gateway]**: [Configuration, testing guide]
- **[Email Service]**: [SMTP config, templates location]

## Common Tasks

### Menambahkan Endpoint Baru

```php
// 1. Buat migration
php artisan make:migration create_table_name

// 2. Buat model
php artisan make:model ModelName

// 3. Buat controller
php artisan make:controller Api/ModelNameController --api

// 4. Tambahkan route di routes/api.php
// 5. Buat test
// 6. Update OpenAPI spec
```

### Menambahkan Feature Baru

1. [Step 1]
2. [Step 2]
3. [Step 3]

## OpenAPI Specification

### Location
```
docs/openapi/openapi.yaml
```

### Generate Client SDK
```bash
# Command untuk generate SDK
npm run generate:sdk
```

### Validation
```bash
# Validate OpenAPI spec
npm run validate:openapi
```

## Environment Variables

### Required Variables

```env
# Application
APP_NAME=
APP_ENV=
APP_KEY=

# Database
DB_CONNECTION=
DB_HOST=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=

# External Services
[List semua env variables yang diperlukan]
```

### Optional Variables

```env
[List optional variables dan default values]
```

## Known Issues & Limitations

### Current Limitations
1. [Limitation 1 - misal: Max concurrent users: 1000]
2. [Limitation 2]

### Known Bugs
1. [Bug 1 - dengan workaround jika ada]
2. [Bug 2]

### Technical Debt
1. [Technical debt 1 yang perlu di-address]
2. [Technical debt 2]

## Communication & Collaboration

### Team Structure
- **Product Owner**: [Nama]
- **Tech Lead**: [Nama]
- **Backend Developers**: [Nama-nama]
- **Frontend Developers**: [Nama-nama]
- **QA**: [Nama]
- **DevOps**: [Nama]

### Communication Channels
- **Daily Standup**: [Waktu & platform]
- **Sprint Planning**: [Jadwal]
- **Code Review**: [Process & SLA]
- **Bug Reports**: [Where & how to report]

### Documentation
- **Architecture Docs**: `docs/ARCHITECTURE.md`
- **API Docs**: `docs/api/README.md`
- **Deployment Guide**: `docs/guides/deployment.md`

## Troubleshooting

### Common Issues

#### Issue: [Masalah umum 1]
**Symptom**: [Deskripsi masalah]
**Solution**: [Cara mengatasi]

#### Issue: [Masalah umum 2]
**Symptom**: [Deskripsi masalah]
**Solution**: [Cara mengatasi]

### Debug Mode

```bash
# Enable debug mode
[Commands untuk enable debugging]

# Check logs
[Lokasi dan cara check logs]
```

## Performance Considerations

### Critical Paths
1. [Path 1 yang harus optimal - misal: Checkout process]
2. [Path 2]

### Performance Budgets
- **API Response Time**: < 200ms (p95)
- **Page Load Time**: < 3s
- **Database Query**: < 100ms (p95)

### Monitoring
- **Dashboard**: [Link ke monitoring dashboard]
- **Alerts**: [Siapa yang menerima alerts]

## Security Notes

### Sensitive Operations
1. [Operasi 1 yang memerlukan extra security]
2. [Operasi 2]

### Compliance
- [GDPR, PCI-DSS, atau compliance lain yang harus diikuti]

### Security Contacts
- **Security Issues**: [Email untuk report security issues]

## Changelog & Versioning

### Current Version
v[X.Y.Z]

### Versioning Strategy
[Semantic versioning, calendar versioning, dll]

### Release Schedule
[Release cycle - weekly, bi-weekly, monthly, dll]

---

## LLM Configuration

### Model Tier

```yaml
# Tier model untuk project ini
# Pilihan: small (fast/cheap), medium (balanced), large (best quality)
model_tier: medium

# Effort level: auto, low, high
# - auto: model decides
# - low: faster, less thorough
# - high: more reasoning, better quality
effort: auto

# Fast mode: true / false
# - true: skip non-essential validation (~30-50% hemat token)
# - false: full quality (recommended untuk production)
fast_mode: false
```

### Sandbox & Security

```yaml
# Sandbox mode: bubble-wrap / seatbelt
# - bubble-wrap: file write langsung, command aman auto-approved
# - seatbelt: semua operasi butuh approval (WAJIB untuk production)
sandbox: bubble-wrap

# Approval policy: untrusted / request / never
# - untrusted: semua operasi berbahaya ditanya
# - request: agent saran, user approve
# - never: auto-mode (CI/CD only — JANGAN di interactive session)
approval_policy: request
```

### Detail Referensi

- `ai-cost-strategy` — model tier selection, effort levels, budget estimation
- `sandbox-security` — sandbox modes, approval policies, permission overrides
- `session-commands` — context/session management commands
- `cli-setup` — installation, authentication, configuration files

---

## Notes untuk OpenCode AI Agent

### Ketika Develop Feature Baru:
1. Selalu tanyakan requirement secara detail
2. Buat plan yang jelas sebelum coding
3. Update dokumentasi seiring development
4. Buat tests untuk feature baru
5. Update OpenAPI spec jika ada perubahan API

### Ketika Fix Bugs:
1. Reproduce issue terlebih dahulu
2. Identify root cause
3. Buat test untuk prevent regression
4. Fix dengan minimal impact ke existing code

### Setiap Phase Selesai (Waypoint Hard Gate):
1. Buat WO JSON di `.opencode/archive/sprints/` (schema kanonik)
2. Tambah handoff entry di `.opencode/phase_state/[project-slug]/handoff.md`
3. Update phase JSON + `handoff_ref`
4. Jalankan `python scripts/waypoint_check.py --check` → WAJIB exit 0

### Ketika Refactoring:
1. Pastikan tests pass sebelum refactor
2. Refactor in small increments
3. Tests harus tetap pass setelah refactor
4. Update documentation jika ada perubahan architecture

### Red Flags (Harus Konfirmasi dengan User):
- [ ] Breaking changes di API
- [ ] Schema changes yang impact existing data
- [ ] Performance degradation
- [ ] Security implications
- [ ] Perubahan besar di architecture

---

**Last Updated**: [Tanggal]
**Maintained By**: [Tim atau nama]
