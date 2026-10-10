---
name: sdlc-project-templates
description: Project-type registry, gap detection engine, compliance engine, and default scaffolding templates with canonical docs/G[N]-[NAME]/ structure. Loaded by sdlc-gate-prd during G0 to detect gaps and scaffold project structure per type. Supports mid-cycle gap re-run on T3/change triggers. Integrates with Work Orders and lean context system.
version: 1.3.0
updated: 2026-08-05
---

# SDLC — Project Templates & Gap Detection

> Loaded by `sdlc-gate-prd` during G0. Provides project-type-specific scaffolding, detects gaps (features, infrastructure, compliance, skills), and generates default project structure.

**Criticality: FLEKSIBEL** — skill pendukung untuk template scaffolding dan gap detection. Panduan di bawah bersifat rekomendasi. Jika project sudah jelas tipenya, bisa langsung pakai tanpa melalui semua step.

## When to Load

| Condition | Action |
|-----------|--------|
| User belum pilih project type | Load this skill, suggest type from registry |
| User pilih project type | Load this skill, scaffold structure + run gap detection |
| FE gap, infra gap, compliance gap needed | Load this skill for gap engine |
| User tanya "best practice untuk project X" | Load this skill |

## Project Type Registry

| Type | Code | Typical Client | Complexity |
|------|------|----------------|------------|
| E-Commerce / Marketplace | `e-commerce` | Retail, dropshipper | High |
| Microservices / SaaS | `microservices` | Startup, enterprise | Very High |
| Mobile App (API backend) | `mobile-app` | Startup, agency | Medium |
| Internal Dashboard | `dashboard` | Enterprise, HR | Low-Medium |
| CMS / Company Profile | `cms` | Small business, org | Low |
| Mobile App (with auth) | `mobile-app-auth` | Fintech, health | Medium-High |

## Gap Detection Engine

Run gap detection AFTER project type is selected. Scan 4 gap categories:

### 1. Feature Gap

Compare user PRD against default feature set for the type.

| Type | Default Must-Have Features |
|------|---------------------------|
| **e-commerce** | Product catalog, Cart, Checkout, Payment gateway, Order management, User accounts, Admin panel |
| **microservices** | API Gateway, Service discovery, Auth service, Config management, Logging aggregation, Monitoring |
| **mobile-app** | Auth (JWT/OAuth), Push notifications, Offline sync, Data caching, Profile management |
| **dashboard** | Auth (RBAC), Data tables, Charts/metrics, Export (CSV/PDF), Search/filter, Audit log |
| **cms** | Auth, Pages CRUD, Media library, SEO meta, User roles |

Format output:

```
⚠️ FEATURE GAP — E-Commerce
- Product catalog: ✅ Ada di PRD
- Cart: ✅ Ada di PRD
- Payment gateway: ❌ TIDAK ADA di PRD
  → Risiko: Bisnis tidak bisa transaksi
  → Saran: Tambah "Payment gateway integration" ke Must Have
```

### 2. Infrastructure Gap

| Gap | Check | Impact |
|-----|-------|--------|
| Hosting | User punya server/cloud? | Deployment delay |
| CI/CD | Pipeline automation? | Manual deploy rawan error |
| CDN | Static assets perlu CDN? | Site slow for global users |
| Database | DB type & hosting clear? | Schema design blocked |
| SSL/Cert | Domain + SSL? | Security compliance fail |

### 3. Compliance Gap

Run compliance checklist from `sdlc-gate-prd`. Plus project-type-specific:

| Type | Mandatory Compliance |
|------|---------------------|
| e-commerce (payment) | PCI-DSS, UU PDP |
| fintech mobile app | OJK regulation, UU PDP |
| health / HIPAA | HIPAA, UU PDP |
| enterprise dashboard | UU PDP (employee data) |
| marketplace | UU PDP (seller + buyer data) |

### 4. Skill Gap

Check whether existing global skills cover the project needs:

```
⚠️ SKILL GAP — Microservices SaaS
- Senior Backend: ✅ Available
- Senior React: ❌ Mobile app but no mobile skill
  → Saran: Pastikan user punya React Native experience
- Database Design: ✅ Available
- Security: ✅ Available
- Error Handling: ✅ Available
```

## Project Scaffold Defaults

### E-Commerce

```
.opencode/artifacts/[project-slug]/G0-PRD/
├── PRD.md
├── PRD-v1.0.md
└── PRD-CHANGELOG.md
.opencode/artifacts/[project-slug]/G1-ARCH/
├── ERD.md
├── SYSTEM.md
├── STACK.md
├── openapi.yaml
└── repository-strategy.md
.opencode/artifacts/[project-slug]/G3-DEV/
└── BLUEPRINT.md
.opencode/artifacts/[project-slug]/G4-DEPLOY/
└── audit-YYYY-MM-DD.md
.opencode/artifacts/[project-slug]/G5-MAINT/
└── ROADMAP.md
```

### Microservices

```
.opencode/artifacts/[project-slug]/G0-PRD/
├── PRD.md
├── PRD-v1.0.md
└── PRD-CHANGELOG.md
.opencode/artifacts/[project-slug]/G1-ARCH/
├── ERD.md
├── SYSTEM.md
├── SERVICE-MAP.md      ← Service boundaries & communication
├── STACK.md
├── openapi/
│   ├── gateway.yaml
│   └── services/
│       ├── auth.yaml
│       ├── product.yaml
│       └── order.yaml
├── repository-strategy.md
└── devops/
    ├── CI-CD.md
    └── INFRA.md
.opencode/artifacts/[project-slug]/G3-DEV/
└── BLUEPRINT.md
.opencode/artifacts/[project-slug]/G4-DEPLOY/
└── audit-YYYY-MM-DD.md
.opencode/artifacts/[project-slug]/G5-MAINT/
└── ROADMAP.md
```

### Mobile App

```
.opencode/artifacts/[project-slug]/G0-PRD/
├── PRD.md
├── PRD-v1.0.md
└── PRD-CHANGELOG.md
.opencode/artifacts/[project-slug]/G1-ARCH/
├── ERD.md
├── SYSTEM.md
├── STACK.md
├── openapi.yaml
└── repository-strategy.md
.opencode/artifacts/[project-slug]/G3-DEV/
└── BLUEPRINT.md
.opencode/artifacts/[project-slug]/G4-DEPLOY/
└── audit-YYYY-MM-DD.md
.opencode/artifacts/[project-slug]/G5-MAINT/
└── ROADMAP.md
```

### Dashboard / CMS

```
.opencode/artifacts/[project-slug]/G0-PRD/
├── PRD.md
├── PRD-v1.0.md
└── PRD-CHANGELOG.md
.opencode/artifacts/[project-slug]/G1-ARCH/
├── ERD.md
├── SYSTEM.md
├── STACK.md
├── openapi.yaml
└── repository-strategy.md
.opencode/artifacts/[project-slug]/G3-DEV/
└── BLUEPRINT.md
.opencode/artifacts/[project-slug]/G4-DEPLOY/
└── audit-YYYY-MM-DD.md
.opencode/artifacts/[project-slug]/G5-MAINT/
└── ROADMAP.md
```

## Compliance Engine

When compliance is required, generate compliance notes in `.opencode/artifacts/[project-slug]/G0-PRD/PRD.md`:

```markdown
### Compliance: [Regulation]

| Requirement | Status | Notes |
|-------------|--------|-------|
| Consent collection | ✅ / ❌ / N/A | ... |
| Data encryption at rest | ✅ / ❌ / N/A | ... |
| Data encryption in transit | ✅ / ❌ / N/A | ... |
| Right to erasure | ✅ / ❌ / N/A | ... |
| Access logging | ✅ / ❌ / N/A | ... |
| Breach notification | ✅ / ❌ / N/A | ... |
```

## Tech Stack Defaults

| Type | Backend | Frontend | DB | Infra |
|------|---------|----------|----|-------|
| **e-commerce** | Laravel | React + Next.js | PostgreSQL | VPS/Vercel |
| **microservices** | Laravel / Go | React / Next.js | PostgreSQL + Redis | Docker + k8s |
| **mobile-app** | Laravel (API) | React Native | PostgreSQL | VPS/Cloud |
| **dashboard** | Laravel | React + Vite | PostgreSQL | Shared hosting/VPS |
| **cms** | Laravel / WordPress | Blade / React | MySQL / PostgreSQL | Shared hosting |

Jika user menggunakan tech stack berbeda → tetap jalan, catat di decisions.md.

## 🚨 Self-Enforcement — Project Templates

Sebaiknya pastikan hal berikut sebelum menyelesaikan:

```
□ Project type sudah ditentukan?
□ Gap detection sudah dijalankan (feature, infra, compliance, skill)?
□ Jika ada gap — sudah ditanyakan ke user untuk diisi?
□ docs/ folder structure sudah di-scaffold sesuai tipe project?
□ Workspace Scaffold sudah dijalankan (scripts/, AGENTS.md, `.opencode/` init)?
□ Verifier `python scripts/waypoint_check.py --check` PASS?
□ decisions.md mencatat hasil gap assessment?
□ Work Order di-classify jika ada template change?
□ Document Sync Rules diaplikasikan sesuai WO type?
```

## ✅ Final Checklist — Project Templates

```
□ Criticality assessment: FLEKSIBEL
□ Tone rekomendasi, bukan instruksi WAJIB
□ Frontmatter hanya name + description
□ Anti-pattern #4 dipatuhi
□ Work Orders, Document Sync & Lean Context summary ada
```

## Workspace Scaffold (Waypoint & Repo Scanner)

Setiap project workspace baru WAJIB di-scaffold 3 elemen berikut supaya waypoint hard gate + verifier jalan sejak awal. Ini mencegah gap seperti yang pernah terjadi (WO hilang, phase=None, schema WO tidak standar).

```
□ scripts/         — copy dari ~/.config/opencode/skills/waypoint/scripts/
                     ├── repo_scan.py       (deteksi repos, tulis .opencode/repo_map.json)
                     └── waypoint_check.py  (verifier hard gate: WO + handoff + phase)
□ AGENTS.md        — generate dari agents-template (sudah termasuk section
                     "Waypoint & Cross-Session (WAJIB)" + hard gate verifier)
□ .opencode/       — init struktur lean context:
                     ├── OVERVIEW.json      (status project, phase aktif)
                     ├── archive/
                     │   ├── sprints/WO-*.json
                     │   └── phases/phase-NN.json
                     └── phase_state/[project]/handoff.md
```

Command verifier (jalan dari root workspace, sama seperti `repo_scan.py`):

```bash
python scripts/repo_scan.py
python scripts/waypoint_check.py --check
```

Referensi path:
- Verifier canonical: `~/.config/opencode/skills/waypoint/scripts/waypoint_check.py` (single-source).
- File di `scripts/` project = copy. Jika ada fix di project, sync balik ke skill (bukan sebaliknya).

## Integration with sdlc-gate-prd

Flow when loaded by G0:

1. User menyebut project type (atau AI rekomendasikan)
2. Load template-specific scaffold structure
3. Run gap detection → output feature, infra, compliance, skill gaps
4. Ask user: [1] ✅ Fill all gaps / [2] Skip gap detection
5. Generate initial `docs/` folder structure
6. Run Workspace Scaffold (lihat section di atas): scripts/ + AGENTS.md + `.opencode/` init
7. Jalankan verifier: `python scripts/repo_scan.py` + `python scripts/waypoint_check.py --check` (harus PASS sebelum lanjut ke G1)
8. Save decisions.md with gap assessment results

---

## Mid-Cycle Gap Assessment

Gap detection TIDAK hanya sekali di G0. Jalankan ulang ketika ada perubahan signifikan:

### Re-Run Triggers

| Trigger | Kapan | Action |
|---------|-------|--------|
| **T3 — New Requirement** | PRD Amendment bertipe T3 | Run ulang gap detection untuk validasi tidak ada gap baru dari requirement baru |
| **BLUEPRINT perubahan signifikan** | >30% task berubah | Gap assessment: feature gaps, infra gaps, compliance gaps |
| **PRD Amendment** | Setiap amendment di PRD | Cek apakah amendment membuka gap baru yang sebelumnya tidak terdeteksi |

### Mid-Cycle Gap Assessment Flow

```
1. Deteksi trigger (T3 / BLUEPRINT change / PRD Amendment)
2. Load project type dari projects.json
3. Run gap detection engine (sama seperti di G0):
   - Feature gaps: fitur wajib yang belum ada task
   - Infra gaps: hosting, CI/CD, monitoring
   - Compliance gaps: regulasi baru yang relevan
   - Skill gaps: skill yang diperlukan tapi belum di-load
4. Output: gap report → tanya user: "Ada [N] gap. Mau di-fill atau deferred?"
5. Jika di-fill → update BLUEPRINT dengan task baru
6. Catat di decisions.md
```

### Traceability Validation

Setiap `PRD-F-[ID]` WAJIB punya traceability ke BLUEPRINT task:

```
Assert: Untuk setiap PRD-F-[ID] di PRD.md
├── Cari task dengan PRD-F-[ID] yang sama di BLUEPRINT
├── ✅ Ketemu → OK
└── ❌ Tidak ketemu → flag gap: "PRD-F-[03] tidak punya task di BLUEPRINT"
```

Ini dijalankan setelah gap detection re-run untuk memastikan semua fitur PRD tercover.

## Work Orders, Document Sync & Lean Context (Project Templates)

> **Canonical definitions:** `sdlc` SKILL.md §4.6 + `waypoint` SKILL.md §5, §7, §15. Short summary below for standalone use.

### WO Classification (Template Context)

| WO Type | When to Use | Required Docs |
|---------|-------------|---------------|
| ENHANCEMENT | New feature requiring template update | Scaffolding, BLUEPRINT, decisions |
| REFACTOR | Template refactoring | Scaffolding, BLUEPRINT, decisions |
| AUDIT | Compliance audit finding | Gap detection, BLUEPRINT, decisions |

### Lean Context

> Read: OVERVIEW.json + ACTIVE_SPRINT.md on session init. Write: OVERVIEW.json + ACTIVE_SPRINT.md + archive WO on session exit. Full rules: `waypoint` §15.
