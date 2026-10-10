---
name: system-audit
description: Comprehensive system audit untuk kode yang sudah jadi. Mencakup security, code quality, performance, error handling, bug detection, database schema, API contract, dan architecture review. Menggunakan arsitektur TWO-PHASE (Discovery read-only → Execution frozen snapshot) dengan convergence gate agar loop audit-fix-terminated. Dipanggil otomatis oleh G4 (QA & Deploy) sebagai exit criteria, atau manual kapan saja via /audit.
version: 2.7.0
updated: 2026-08-10
loads: [security-first, code-standards, performance-optimization, error-handling, bugs-finder, database-design, openapi-first, senior-backend-engineer, senior-react-engineer, spec-compliance]
---

# System Audit

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI

**System audit adalah benteng terakhir sebelum code masuk production.** Satu celah yang terlewat bisa menyebabkan data breach, downtime, atau kerugian finansial.

🔴 **Konsekuensi jika dilanggar:**
- **Audit dilewatkan di G4** → bug/security hole masuk production, reputasi hancur
- **Hanya 1 dimensi yang di-audit** → celah di dimensi lain tidak terdeteksi, false sense of security
- **Finding tanpa severity** → prioritas tidak jelas, fix yang kritis bisa ketunda
- **Finding tanpa rekomendasi** → developer bingung harus fix apa, fix jadi setengah-setengah
- **Audit report tidak disimpan** → tidak ada traceability, audit berikutnya tidak punya baseline
- **Fix-while-audit (REMEDIATE saat Discovery — Full System Audit / Two-Phase)** → tiap fix memicu re-scan, re-scan nemu baru, loop N+ tak berujung. Ini PATOLOGI UTAMA audit lama. DILARANG untuk Full System. (Feature Audit boleh remediasi langsung jika user minta — lihat mode Feature Audit, tapi tetap wajib lewat pipeline finding.)
- **Tanpa convergence gate** → audit berhenti karena "perasaan aman", bukan data. Tidak pernah tahu kapan selesai.

**Model WAJIB** menjalankan SEMUA dimensi audit sesuai mode yang dipilih. JANGAN skip dimensi tanpa alasan yang dicatat.

---

## Canonical Definitions — SINGLE SOURCE OF TRUTH

> **Semua data terstruktur di skill ini didefinisikan HANYA di section ini.** Section lain boleh memakai nilai ini, TAPI JANGAN menulis ulang definisinya. Kalau dua bagian skill kelihatan beda → yang benar adalah yang di sini.
>
> **Aturan maintenance (WAJIB):** ketika mengubah enum/severity/path/threshold, ubah HANYA di sini, lalu SINKRONKAN semua contoh JSON/report yang mereferensikannya. Contoh yang melanggar definisi di bawah = bug dokumen.

### Status Enum (5)

| Status | Arti | Konfirmasi user? |
|--------|------|------------------|
| `open` | Ditemukan, belum ada tindakan | — |
| `fixed` | Kode diperbaiki + diverifikasi (baca kode aktual, bukan asumsi) | — |
| `deferred` | Ditunda ke target jelas (`deferred_to`) — butuh refactor berat / tunggu library | ✅ wajib |
| `accepted` | Accepted risk — sengaja tidak di-fix, biasanya tanpa rencana fix masa depan | ✅ wajib |
| `skipped` | Sengaja dilewati (low impact, style-only, no functional risk) | ✅ wajib |

> **deferred vs accepted:** `deferred` punya target rilis (`deferred_to` + `next_action`) dan AKAN dikerjakan. `accepted` tidak punya target — risiko diterima permanen. KEDUANYA tetap butuh `acceptance_reason` + 3-layer trail.
> **Skenario LOW + butuh refactor berat = `deferred`** (didefer ke WO dengan `deferred_to`), BUKAN `accepted`.

### Severity (4) — threshold tertutup (tanpa gap)

| Severity | Kriteria | Wajib Fix Sebelum |
|----------|----------|-------------------|
| 🔴 **Critical** | Data loss, security breach, crash di production, data corruption | **Deploy — BLOCKING** |
| 🟠 **High** | Major functionality broken, performance degradation **≥50%**, auth bypass | **Deploy — BLOCKING** |
| 🟡 **Medium** | Minor functionality issue, code quality, performance degradation **<50%** | **Next iteration** |
| 🔵 **Low** | Style, naming, minor improvements, documentation | **Note only** |

> Boundary 50% tertutup: degradasi **tepat 50% = High**. Tidak ada "gap" antara High dan Medium.

### Paths — canonical

| Artifact | Path |
|----------|------|
| Audit master tracker | `.opencode/audit/registry.json` |
| Audit surface contract | `.opencode/audit/AUDIT_CONTRACT.json` |
| Findings (per scope) | `.opencode/audit/findings/{source_type}/{scope}.json` |
| Final report (setelah approval) | `.opencode/artifacts/[project-slug]/G4-DEPLOY/YYYY-MM-DD_[mode]-audit.md` |
| 3-layer trail (layer 2 & 3) | `.opencode/phase_state/[project-slug]/decisions.md` + `KNOWLEDGE_BASE.md` |
| Fallback (tanpa `.opencode/`) | Audit → `docs/audit/`; phase_state → root project (`./decisions.md`, `./KNOWLEDGE_BASE.md`) |

### Thresholds — canonical

| Aturan | Nilai |
|--------|-------|
| Convergence bound | maks 3 pass Discovery; pass 3 HARUS 0 temuan baru |
| TRIPWIRE fix-phase | `>20%` dari backlog beku ATAU `>10 item` — **mana yang lebih kecil** |
| Battery | baseline di awal Discovery + proof di akhir Execution |
| Approval gate | report final baru di-copy ke G4-DEPLOY **setelah approval user** |

### Glossary

| Istilah | Definisi |
|---------|----------|
| **WO** | Work Order — unit kerja dari skill `waypoint` (BUG / GAP / AUDIT / ENHANCEMENT / REFACTOR) |
| **SPRINT_BACKLOG** | File backlog aktif (artifact waypoint) — di-regenerate tiap sprint, bukan tempat jejak historis |
| **known-gap** | Tool battery yang tidak tersedia/gagal setup, atau kegagalan yang bukan bug kode (mis. CVE dependency yang sudah punya WO) — WAJIB tercatat eksplisit |
| **fix-phase discoveries** | Temuan baru yang muncul SAAT Execution — dicatat & di-triage, JANGAN di-fix interleaved |
| **accepted risk** | Risiko yang sengaja diterima & terdokumentasi (status `accepted`) |
| **3-layer trail** | Jejak accepted/deferred: finding JSON + decisions.md + KNOWLEDGE_BASE.md |

### ID Pattern

```
Finding ID: {SCOPE}-{NNN}   (NNN = 3 digit, zero-padded: 001, 002, ...)
Contoh valid:   POS-001, TRANSACTIONS-003, PROD-ORD-005 (scope boleh mengandung hyphen)
Contoh INVALID: TRANSACTIONS-ID-001, PROD-ORD-ID-001 (infix -ID-/-LOW- TIDAK dibolehkan)
```

---

## When to Load

| Condition | Action |
|-----------|--------|
| G4 — QA & Deploy (default) | **Full System Audit** — otomatis, exit criteria |
| G5 — Maintenance (periodic) | Quick Scan — rekomendasi tiap 30 hari |
| User minta "audit fitur [X]" | Feature Audit — scope terbatas |
| User minta "audit file [Y]" | Quick Scan — file spesifik |
| User minta "review security" | Security-Only mode (subset) |

## Audit Dimensions

| # | Dimensi | Skill Reference | Fokus Pengecekan |
|---|---------|----------------|------------------|
| 1 | **Code Quality** | `code-standards` | DRY, SOLID, Clean Code, naming conventions, test coverage |
| 2 | **Security** | `security-first` | OWASP Top 10 (injection, XSS, CSRF, auth, encryption) |
| 3 | **Performance** | `performance-optimization` | N+1 queries, caching, query optimization, memory leaks |
| 4 | **Error Handling** | `error-handling` | Exception hierarchy, try-catch patterns, logging, fail-fast |
| 5 | **Bug Detection** | `bugs-finder` | Data loss, crashes, race conditions, null dereferences |
| 6 | **Database Schema** | `database-design` | Indexing, normalization, migration safety, soft delete |
| 7 | **API Contract** | `openapi-first` | OAS compliance, validation, versioning, response format |
| 8 | **Architecture** | `senior-backend-engineer` / `senior-react-engineer` | Pattern consistency, separation of concerns, dependency flow |
| 9 | **Spec Compliance** | `spec-compliance` | Docs↔code kesesuaian: FR/tabel/endpoint/seeder di PRD/ERD/API-SPEC/ARCH terimplementasi di kode (implemented/partial/stub/missing). Output: compliance-registry + gap list |

## Two-Phase Audit Architecture (WAJIB untuk Full System Audit)

**Masalah yang dipecahkan:** Alur audit lama (SCAN → REMEDIATE → VERIFY dalam satu run) menciptakan loop N+ — tiap fix memicu re-scan, re-scan menemukan bug baru, dan siklus tidak pernah terminasi. Audit berhenti karena "perasaan sudah bersih", bukan karena kriteria obyektif.

**Solusi:** Pisahkan audit menjadi 2 phase dengan batas tegas:

```
┌─────────────────────────────────────────────────────────────┐
│ PHASE 1: DISCOVERY (READ-ONLY)                               │
│   SCOPE → SCAN → ANALYZE → REPORT                            │
│   + Audit Surface Contract (definition of done)              │
│   + Tool Battery (deterministik, reproducible)               │
│   + Convergence Gate (≤3 pass, 0 temuan baru = convergen)    │
│   Output: BACKLOG BEKU (frozen snapshot)                     │
└─────────────────────────────────────────────────────────────┘
                              │  (snapshot di-freeze, tidak ada fix)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ PHASE 2: EXECUTION (TERHADAP SNAPSHOT BEKU)                  │
│   Cluster findings per root-cause → impact analysis → fix    │
│   → verify → regression. Scope-freeze + tripwire.            │
│   Output: battery clean + suite hijau + re-scan cluster 0 regresi │
└─────────────────────────────────────────────────────────────┘
```

> **Bahasa:** 6-fase workflow di bawah tetap memakai kata **Fase** (Fase 1–6). Arsitektur Two-Phase memakai **Phase** (Phase 1 Discovery / Phase 2 Execution) agar tidak bentrok istilah. Perhatikan bedanya: `Fase 2: SCAN` ≠ `Phase 2: Execution`.

### Aturan Inti (WAJIB)

1. **JANGAN PERNAH fix kode saat Discovery.** Phase 1 hanya membaca kode, menghasilkan temuan, dan mengukur dengan battery. Fix = pekerjaan Phase 2.
2. **Definition of done = contract coverage + battery + convergence gate**, BUKAN "tidak menemukan apa-apa".
3. **Scope-freeze di Phase 2:** temuan baru yang muncul saat fix dicatat ke daftar "fix-phase discoveries" — JANGAN di-fix interleaved. Ini yang memutus loop.
4. **Convergence harus terukur** (angka pass, 0 temuan baru, battery clean), bukan subjektif.

### Kenapa Ini Menghentikan Loop

| Loop Lama | Loop Baru (Two-Phase) |
|-----------|------------------------|
| Fix → re-scan → nemu baru → fix → ... | Discovery sampai convergen (≤3 pass) → snapshot beku |
| Tiap fix bisa trigger audit ulang penuh | Phase 2 berjalan terhadap snapshot; temuan baru di-triage terpisah |
| Berhenti karena "rasanya sudah aman" | Berhenti karena: contract 100% + battery clean + re-scan 0 temuan |

> **Catatan honest:** loop tidak bisa dihilangkan 100% (setiap fix punya peluang regression). Tapi dengan Two-Phase, loop **ter-bound by construction** — bukan menghilang, tapi terbatas dan terukur.

## Audit Modes

### 🔴 Full System Audit — G4 Default (WAJIB Two-Phase)

- **Scope:** Semua kode di project
- **Dimensi:** 1-8 (semua)
- **Arsitektur:** **WAJIB Two-Phase** (Discovery read-only → Execution frozen snapshot). Lihat section Two-Phase Audit Architecture.
- **Output:** `.opencode/artifacts/[project-slug]/G4-DEPLOY/YYYY-MM-DD_full-system-audit.md` (di-copy ke G4-DEPLOY **setelah approval user**)
- **Waktu estimasi:** Dependent on project size
- **Trigger:** Otomatis saat G4 exit criteria
- **WAJIB lulus:** Audit Surface Contract 100% + Tool Battery clean (known-gap tercatat) + Convergence Gate (re-scan 0 temuan baru)

### 🟡 Feature Audit — Manual

- **Scope:** 1 fitur spesifik (module/controller/service)
- **Dimensi:** 1-5 (code quality, security, performance, error handling, bugs)
- **Arsitektur:** Discovery read-only. Remediasi optional — jika user minta fix langsung, WAJIB tetap lewat pipeline finding (update status finding + registry), BUKAN fix lalu lupa catat.
- **Output:** `.opencode/artifacts/[project-slug]/G4-DEPLOY/YYYY-MM-DD_feature-[name]-audit.md`
- **Trigger:** User minta audit fitur tertentu

### 🔵 Quick Scan — Manual / G5 Periodic

- **Scope:** 1 file atau surface-level scan
- **Dimensi:** 2 (security) + 5 (bugs)
- **Output:** Cukup di chat, simpan ke file hanya jika ada temuan
- **Trigger:** User minta quick scan, atau G5 periodic (30 hari)

### 🔒 Security-Only Mode — Subset

- **Scope:** Semua kode di project ATAU scope yang ditentukan user
- **Dimensi:** 2 (security) saja — subset dari Full System
- **Output:** Sama dengan mode yang memicu (Full System / Feature Audit / Quick Scan), tapi report hanya memuat temuan dimensi security
- **Trigger:** User minta "review security" / "audit keamanan" tanpa dimensi lain

## Audit Workflow — 6 Fase (dipetakan ke Two-Phase)

**Pemetaan fase (WAJIB untuk Full System Audit):**

| Fase | Nama | Fase Audit | Read-Only? |
|------|------|-----------|------------|
| Fase 1 | SCOPE | Discovery | ✅ |
| Fase 2 | SCAN | Discovery | ✅ |
| Fase 3 | ANALYZE | Discovery | ✅ |
| Fase 4 | REPORT | Discovery | ✅ |
| Fase 5 | REMEDIATE | **Execution** | ❌ (boleh fix) |
| Fase 6 | VERIFY | **Execution** | ❌ |

Fase 1–4 **WAJIB read-only** — tidak ada perubahan kode apa pun. Detail Execution (Fase 5–6) diatur di section **Execution Phase Workflow** di bawah.

### Fase 1: SCOPE

Tentukan scope audit berdasarkan mode:

| Mode | Pertanyaan WAJIB |
|------|-----------------|
| Full System | Project root? Fitur apa saja yang termasuk? |
| Feature | Nama fitur? File apa saja terkait? |
| Quick Scan | File path? Dimensi yang diminta? |

**WAJIB** catat scope di awal report.

### Fase 2: SCAN

Jalankan audit per dimensi. WAJIB load skill reference untuk tiap dimensi:

```
Dimensi 1 (Code Quality)  → load skill `code-standards`
Dimensi 2 (Security)      → load skill `security-first`
Dimensi 3 (Performance)   → load skill `performance-optimization`
Dimensi 4 (Error Handling) → load skill `error-handling`
Dimensi 5 (Bug Detection) → load skill `bugs-finder`
Dimensi 6 (Database)      → load skill `database-design`
Dimensi 7 (API Contract)  → load skill `openapi-first`
Dimensi 8 (Architecture)  → load skill `senior-backend-engineer` + `senior-react-engineer` (jika ada frontend)
```

**WAJIB** scan file by file — JANGAN generalisasi tanpa buka kode.

### Fase 3: ANALYZE

Klasifikasi tiap temuan dengan severity matrix — **lihat Canonical Definitions → Severity** (kriteria + threshold tertutup + wajib fix sebelum deploy). Ringkasan cepat:

| Severity | Wajib Fix Sebelum |
|----------|-------------------|
| 🔴 **Critical** | **Deploy — BLOCKING** |
| 🟠 **High** | **Deploy — BLOCKING** |
| 🟡 **Medium** | **Next iteration** |
| 🔵 **Low** | **Note only** |

> Tabel di atas hanyalah ringkasan cepat — **Canonical Definitions → Severity** adalah sumber kebenaran untuk kriteria + threshold. Jika terjadi konflik, canonical menang.

**WAJIB** sertakan untuk tiap finding:
1. **Impact konkret** — apa yang terjadi di production jika tidak di-fix?
2. **Root cause** — kenapa ini bisa terjadi?
3. **Trigger scenario** — langkah apa yang memicu issue?

### Fase 4: REPORT

Generate structured audit report. **WAJIB** ikut template di bawah.

Simpan findings ke `.opencode/audit/findings/{source_type}/{scope}.json` (source-based subdirectories). Setelah semua scope selesai, generate final report (lokasi kerja: draft di root project atau chat). **Setelah approval user**, copy report ke `.opencode/artifacts/[project-slug]/G4-DEPLOY/` — G4-DEPLOY hanya berisi report yang sudah disetujui:
```
.opencode/audit/
├── registry.json
├── AUDIT_CONTRACT.json            ← Surface contract (definition of done)
└── findings/
    ├── SYSTEM_AUDIT/          ← Findings dari system-audit skill
    │   └── {scope}.json
    ├── GAP_ANALYSIS/          ← Findings dari gap analysis
    │   └── {scope}.json
    ├── CODE_REVIEW/           ← Findings dari code review
    │   └── {scope}.json
    └── MANUAL/                ← Findings dari manual review
        └── {scope}.json

.opencode/artifacts/[project-slug]/G4-DEPLOY/
    └── YYYY-MM-DD_[mode]-audit.md   ← Final report setelah approval
```

### Source Type Classification

| Source Type | Trigger | Directory |
|------------|---------|-----------|
| **SYSTEM_AUDIT** | system-audit skill runs (G4 default) | `findings/SYSTEM_AUDIT/` |
| **GAP_ANALYSIS** | sdlc-project-templates gap detection | `findings/GAP_ANALYSIS/` |
| **CODE_REVIEW** | Manual code review, PR review | `findings/CODE_REVIEW/` |
| **MANUAL** | Ad-hoc discovery during development | `findings/MANUAL/` |

### Discovery Sub-Phase 1: Audit Surface Contract (definition of done)

> **Urutan eksekusi (WAJIB dipahami):** Sub-phase Discovery tidak berjalan SEKUENSIAL setelah Fase 4. Mereka menyatu dalam Fase 1–4:
> - **Contract** → dibuat di **Fase 1 (SCOPE)**, SEBELUM scan dimulai
> - **Tool Battery baseline** → dijalankan di awal **Fase 2 (SCAN)**
> - **Impact Analysis** → dipakai di **Phase 2 (Execution)**, tapi polanya disiapkan di sini
> - **Convergence Gate** → dijalankan SETELAH **Fase 4 (REPORT)**, sebelum REMEDIATE
>
> Section ini sengaja diletakkan berurutan (bukan terpisah) supaya satu tempat baca.

**Masalah:** Tanpa contract, tiap pass audit nge-scan slice yang beda → "temuan terus muncul" karena coverage tidak merata, bukan karena kode jelek. Ini sumber loop yang paling sering diabaikan.

**Solusi:** Sebelum SCAN dimulai, **WAJIB** buat contract enumerable di `.opencode/audit/AUDIT_CONTRACT.json`:

> Contoh ini memperlihatkan status post-scan (beberapa sel sudah `verified` atau `scanned`). Saat dibuat pertama kali (sebelum scan), semua status harus `pending`.

```json
{
  "project": "[project-slug]",
  "mode": "full-system",
  "generated_at": "YYYY-MM-DD",
  "modules": [
    {
      "name": "auth",
      "paths": ["app/Http/Controllers/Auth", "app/Services/AuthService.php"],
      "dimensions": {
        "code-quality": {"tool": "manual", "status": "verified"},
        "security": {"tool": "sub-agent:agent-1", "status": "verified"},
        "performance": {"tool": "manual", "status": "pending"},
        "error-handling": {"tool": "manual", "status": "pending"},
        "bug-detection": {"tool": "manual", "status": "pending"},
        "database": {"tool": "manual", "status": "pending"},
        "api-contract": {"tool": "known-gap", "status": "verified", "note": "openapi-first belum di-install — manual review diperkuat"},
        "architecture": {"tool": "manual", "status": "pending"}
      }
    }
  ],
  "gaps": [
    {"item": "openapi-first", "reason": "belum terpasang — diganti manual review diperkuat", "resolved_at": null}
  ]
}
```

Aturan contract (WAJIB):
1. **Setiap module × dimensi = 1 sel** dengan status `pending → scanned → verified`.
   - `pending` = belum di-scan
   - `scanned` = sub-agent selesai scan sel ini + findings ditulis ke file (bisa masih ada temuan)
   - `verified` = main model sudah baca findings file sel ini + konfirmasi coverage benar (TIDAK berarti sel bersih — hanya berarti sel tercakup dan tercatat)
   - `tool`: `manual` | `sub-agent:{id}` | nama tool (mis. `phpstan`) | `known-gap`
2. **"Audit selesai" = SEMUA sel `verified`**, BUKAN "tidak menemukan apa-apa".
3. Sel yang di-scan via sub-agent → tandai `"tool": "sub-agent:{id}"` + `status: scanned`.
4. Tool yang tidak tersedia (mis. phpstan belum terpasang) → tandai `"tool": "known-gap"` + isi `note` di sel, dan tambahkan entry ke array `gaps[]` di root contract (reason + tanggal resolve). Ganti dengan manual review yang diperkuat. JANGAN asumsikan cakupan.
5. Jika Discovery menemukan banyak temuan baru setelah pass ke-3 → contract tidak lengkap → update contract (tambah module/sel) → restart.

### Discovery Sub-Phase 2: Tool Battery (deterministik, reproducible)

**Kenapa:** Audit berbasis baca-kode SAJA selalu punya blind spot — contoh nyata: `composer audit` menemukan 12 security advisories (CVE dompdf) yang luput dari code-review manual karena lokasinya di `vendor/`. Battery = jaring pengaman untuk blind spot itu.

**WAJIB** jalankan battery ini di awal Discovery (baseline) dan di akhir Execution (proof). Satu perintah script per repo (project-level), output tercatat:

| Stack | Perintah | Fail = |
|-------|----------|--------|
| PHP/Laravel | `composer audit` | ⚠️ Known vuln → finding |
| PHP/Laravel | `phpstan analyse --level=max` (larastan) | ⚠️ Error → finding |
| PHP/Laravel | `vendor/bin/pint --test` | ⚠️ Style violation → finding (low) |
| PHP/Laravel | `php -l` semua file | ⚠️ Parse error → finding |
| PHP/Laravel | `vendor/bin/phpunit` | ⚠️ Test fail → finding |
| TS/Next.js | `npm audit` | ⚠️ Known vuln → finding |
| TS/Next.js | `tsc --noEmit --strict` | ⚠️ Type error → finding |
| TS/Next.js | `eslint` | ⚠️ Error/warning → finding |
| TS/Next.js | `npm run build` | ⚠️ Build fail → finding |

Aturan battery (WAJIB):
1. **Battery = 1 perintah** per repo (script `audit-battery.sh` / npm script). JANGAN jalanin ad-hoc perintah satu-satu. Template minimal:
   ```bash
   #!/usr/bin/env bash
   # audit-battery.sh — [project-root]/bin/, jalankan dari root repo
   set -e
   # SEMUA perintah memakai `|| true` — output dikumpulkan sebagai backlog,
   # bukan menghentikan script (lihat aturan battery #2)
   echo "[battery] composer audit";      composer audit || true
   echo "[battery] phpstan";             vendor/bin/phpstan analyse --level=max --no-progress || true
   echo "[battery] pint --test";         vendor/bin/pint --test || true
   echo "[battery] php -l";              find app database routes -name '*.php' -exec php -l {} \; || true
   echo "[battery] phpunit";             vendor/bin/phpunit || true
   # (TS/Next.js tambahkan: npm audit, tsc --noEmit --strict, eslint, npm run build — semuanya `|| true`)
   ```
2. **Semua output battery adalah input backlog** — setiap error/warning/vuln = finding ATAU known-gap yang tercatat eksplisit.
3. **Tooling setup ≠ fix kode.** Memasang phpstan/pint di **pass-0** (persiapan tooling sebelum scan dimulai, bukan bagian dari pemburu temuan) TIDAK termasuk menemukan bug. Upgrade dependency (mis. dompdf) adalah WO sendiri dengan regression test — lewat pipeline normal, bukan diam-diam.
4. Battery yang tidak bisa clean (mis. phpstan belum tersedia) → catat `known-gap` di contract — bukan dianggap "aman".

> Definisi `known-gap`: **lihat Canonical Definitions → Glossary** (JANGAN menulis ulang definisi di sini).

### Discovery Sub-Phase 3: Impact Analysis (anti fix-induced bugs)

**Kenapa:** Fix yang menyentuh fungsi dengan hidden consumer → bug baru → audit baru. Ini sumber loop kedua.

**Solusi (deterministik, tanpa tool LLM graph yang mahal):**

| Kebutuhan | Metode |
|-----------|--------|
| Enumerate call sites (pre-fix) | Script `impact.sh` berbasis `rg`: `rg -n 'MethodName' --glob '!vendor' --glob '!node_modules'` → daftar `file:line` |
| Verifikasi "apa yang rusak" (post-fix) | **Compiler/static analyzer itu sendiri**: `phpstan analyse` + `tsc --noEmit` otomatis menunjuk semua caller yang rusak saat signature berubah |
| Circular dependency (opsional) | `madge` (TS, static) / `deptrac` (PHP) |

Rule (WAJIB): **SEBELUM fix, enumerate call sites. SETELAH fix, verify semua call sites via compiler.** JANGAN fix simbol tanpa dua langkah ini.

### Discovery Sub-Phase 4: Convergence Gate (kriteria berhenti obyektif)

Discovery **WAJIB** berhenti hanya jika dua bukti independen terpenuhi:

```
✅ Bukti A — Tool Battery clean:
    composer/npm audit 0 vuln + phpstan/tsc/eslint 0 error + suite hijau
    (atau semua kegagalan tercatat sebagai finding/known-gap)

✅ Bukti B — Full-surface re-scan = 0 temuan baru:
    Satu sub-agent FRESH (context beda, tidak ikut scan sebelumnya)
    mengulang seluruh contract coverage → 0 finding baru

🚧 Bound keras (HARUS dipenuhi SEMUA):
    Pass 1 → baseline backlog
    Pass 2 → re-scan coverage penuh (contract tidak berubah)
    Pass 3 → HARUS 0 temuan baru
    Pass 3 masih nemu → STOP, update contract, restart Discovery
```

- **Bound keras = ≤ 3 pass + pass 3 HARUS 0.** TIDAK ada pengecualian. Ini yang memutus loop, bukan "harapan temuan menurun".
- **Diminishing returns adalah indikator diagnostik, BUKAN gate.** Pasangannya di pasal contract aturan 5: kalau pass 2 menemukan banyak temuan baru (mis. > ½ total pass 1) → itu tanda contract belum lengkap → cek coverage module/sel yang terlewat → update contract → restart. Bukan berarti wajib stop di angka tertentu.
- Gate-nya: **Bukti A + Bukti B + bound 3 pass.** Semua HARUS lulus baru lanjut Execution.
- Setelah convergen → **FREEZE snapshot**: catat commit ref + timestamp tiap module, urutan eksekusi dari module paling lama di-scan (anti basi).
- Discovery belum convergen → **JANGAN** lanjut ke Execution. LARANGAN keras.

### Fase 5: REMEDIATE

> **Discovery sudah selesai & convergen?** Jika belum — KEMBALI ke Discovery. Fase 5/6 adalah bagian **Execution Phase** (Phase 2 Two-Phase), detail lengkap di section **Execution Phase Workflow** di bawah.

Untuk tiap finding, WAJIB berikan:
1. **Recommended fix** — langkah konkret, bukan abstraksi
2. **Reference skill** — skill apa yang relevan untuk fix ini
3. **Example code** — ❌ before + ✅ after (**WAJIB** untuk 🔴 dan 🟠; strongly recommended untuk 🟡)

### Fase 6: VERIFY

🚨 **GATE: SEBELUM tandai finding sebagai "fixed", WAJIB baca kode aktual di `location` field.**

Jangan pernah claim "fixed" hanya berdasarkan:
- Finding file status → "udah di-update jadi fixed" ❌
- Ingatan samar → "saya rasa udah dibenerin" ❌
- Perubahan di file lain → "ini mirip, pasti udah" ❌

Hanya claim "fixed" jika:
- Kode di `location` field sudah dibaca dan fix benar-benar terlihat ✅
- Tidak ada false-positive — fix address root cause, bukan cuma gejala ✅

Setelah remediasi, WAJIB:
1. **Baca kode aktual** di setiap `location` field finding yang diklaim fixed
2. **Re-scan** dimensi yang di-fix
3. **Konfirmasi** finding sudah resolved — berdasarkan kode aktual, bukan asumsi
4. **Update finding JSON** — tambah `verified_at`, `verified_by`
5. **Update report** — tandai finding (post-remediasi — `open` tidak relevan di sini) sebagai `✅ FIXED` / `⏸️ DEFERRED` / `🟢 Accepted` / `⏭️ Skipped` (sesuai status aktual)

> Fase 6 VERIFY adalah bagian dari **Execution Phase Workflow**. Execution berjalan terhadap snapshot beku — VERIFY memastikan snapshot-fix benar-benar menghapus finding, dan battery proof dijalankan di akhir Execution.

### Contoh

❌ **Verifikasi Salah (asumsi, bukan baca kode):**
```
Finding PROD-ORD-001: "Modal tidak pakai Button component"
→ "Saya rasa udah fix karena saya edit file page.tsx kemarin"
→ ❌ FALSE CLAIM — ternyata cuma 3 dari 7 modal yang diubah
```

✅ **Verifikasi Benar (baca kode aktual):**
```
Finding PROD-ORD-001: "Modal tidak pakai Button component"
→ Baca src/app/products/[id]/orders/page.tsx → cek 7 modal
    → Baris 89 ✅, 120 ✅, 145 ✅, 178 ✅, 210 ✅, 245 ✅, 280 ✅
→ Kode sudah pakai <Button> component di semua modal
→ ✅ FIXED — verified_at: 2026-07-07, verified_by: code-review
```

---

## Execution Phase Workflow (Phase 2 Two-Phase)

**Prasyarat WAJIB:** Discovery sudah convergen + snapshot sudah di-freeze (commit ref + timestamp tiap module). Execution **berjalan terhadap snapshot beku** — fix sesuai daftar, JANGAN membuka ulang audit penuh di tengah jalan.

### Step 1: Cluster Findings

Kelompokkan semua finding berdasarkan root-cause/module sehingga **tiap file disentuh SATU KALI saja** (minimize regression surface):

```
Cluster 1 — DocumentService (DOC-002, DOC-006, DOC-010) → file yang sama
Cluster 2 — NotificationService (NOTIF-018, ...)
Cluster 3 — api-proxy.ts (WEB-025, ...)
...
```

- Dua finding di file yang sama = 1 cluster. JANGAN pecah cluster hanya karena severity beda.
- Urutan eksekusi: module yang paling lama di-scan lebih dulu (anti basi snapshot).

### Step 2: Per Cluster — Fix dengan Impact Analysis

```
1. impact.sh <symbol>  → enumerate semua call sites (rg, file:line)
2. Fix (behavior-preserving jika memungkinkan)
3. phpstan analyse + tsc --noEmit  → verify semua call sites tidak rusak
4. Focused tests (test yang menyentuh cluster ini)
5. Update finding JSON: status → fixed + verified_at + verified_by
```

- **Behavior-preserving fix didahulukan** (refactor aman) sebelum perubahan perilaku.
- Fix yang menyentuh dependency upgrade → WAJIB regression test sendiri (mis. dompdf ≥3.1.6).

### Step 3: Scope-Freeze + Fix-Phase Discoveries

**Temuan baru yang muncul SAAT fix** (fix-induced atau ketahuan belakangan):

```
DICATAT ke daftar "fix-phase discoveries" — JANGAN di-fix interleaved!
├── same root-cause dengan finding yang sedang dikerjakan → ikut cluster (catat)
└── beda root-cause → WO terpisah / siklus berikutnya (JANGAN masuk sini)

🚨 TRIPWIRE: fix-phase discoveries > 20% dari total backlog beku ATAU > 10 item (mana yang lebih kecil)
   → snapshot terlalu basi → STOP
   → jalankan re-discovery pass terbatas (hanya kode yang berubah) → merge → lanjut
```

Scope-freeze inilah yang memutus loop N+ lama (fix memicu audit baru tanpa batas).

### Step 4: Regression Proof (akhir Execution)

Setelah SEMUA cluster selesai, WAJIB jalankan ulang:
1. **Tool Battery penuh** (semua stack) → clean / known-gap tercatat
2. **Full test suite** → hijau
3. **Build** (next build / composer install fresh) → sukses

### Step 5: Kebijakan LOW (penentu kecepatan konvergensi)

| Kategori LOW | Aksi |
|--------------|------|
| Fix murah (style, naming, nit) | Fix bareng cluster berjalan |
| Perlu refactor berat | **WAJIB konfirmasi user** → `deferred` + `acceptance_reason` + `deferred_to` + `next_action` |
| Risiko rendah, tidak relevan lagi | **WAJIB konfirmasi user** → `skipped` + alasan |

**JANGAN auto-accept tanpa konfirmasi user** (aturan: kalau ragu → tanya user). Setiap accepted/deferred WAJIB punya **3-layer trail** (lihat Finding Lifecycle): finding JSON + decisions.md + KNOWLEDGE_BASE.md (Accepted Debt).

### Definition of Done — Execution Selesai

```
□ Semua cluster dieksekusi dengan impact analysis + verify per cluster
□ Battery ulang clean / known-gap tercatat
□ Full suite hijau + build sukses
□ Re-scan (sub-agent fresh) terhadap CLUSTER yang sudah di-fix = 0 regresi/temuan baru
□ Temuan baru di luar cluster → fix-phase discoveries → triage (BUKAN blocking DoD)
□ Sisa open = hanya accepted/deferred dengan rationale + 3-layer trail
□ Fix-phase discoveries di-triage (bukan di-drop diam-diam)
```

---

## Finding Lifecycle Management

### Finding States

Setiap finding melewati lifecycle berikut:

```
[OPEN] ──► [FIXED]       (code fix applied + verified)
    │          │
    ├──► [DEFERRED]        (known issue, postponed with rationale)
    ├──► [ACCEPTED]        (accepted risk — deliberately not fixed, documented)
    └──► [SKIPPED]         (intentionally not fixed — documented why)
```

Definisi tiap status → **lihat Canonical Definitions → Status Enum** (JANGAN menulis ulang di sini). Catatan lifecycle (bukan definisi):

- Semua transisi dari `open` wajib **konfirmasi user** (terutama `accepted` — JANGAN auto-accept).
- `deferred` punya target rilis jelas (`deferred_to` + `next_action`); `accepted` TIDAK punya rencana fix — hanya `acceptance_reason`.

### Accepted Risk — 3-Layer Trail (WAJIB)

Setiap finding `ACCEPTED`/`DEFERRED` WAJIB punya jejak di 3 tempat agar historical context tidak hilang lintas sprint:

| Layer | Lokasi | Isi |
|-------|--------|-----|
| 1 | Finding JSON | `status: accepted/deferred` + `acceptance_reason`. **Jika deferred:** + `deferred_to` + `next_action`. |
| 2 | `decisions.md` | Entry per keputusan — alasan + dampak + "untuk solve next time butuh X" |
| 3 | `KNOWLEDGE_BASE.md` (kategori **Accepted Debt**) | Agregat 1-baris per item — `ID \| alasan \| trigger solve` |

SPRINT_BACKLOG di-regenerate tiap sprint → bukan tempat menyimpan jejak accepted. 3-layer di atas adalah sumber historical context yang persisten.

> **Lokasi default:** `.opencode/phase_state/[project-slug]/decisions.md` dan `.opencode/phase_state/[project-slug]/KNOWLEDGE_BASE.md` (dipakai bersama waypoint/phase_state). Jika project tidak punya `.opencode/`, fallback ke root project (`./decisions.md`, `./KNOWLEDGE_BASE.md`).

### Kapan Update Finding Status

| Kondisi | Wajib Update | Field yang Diisi |
|---------|-------------|-----------------|
| Finding 🔴/🟠 di-fix | ✅ | `status: "fixed"`, `fixed_at`, `fix_cycle`, `fix_summary`, `verified_at`, `verified_by` |
| Finding 🟡/🔵 di-fix (bonus) | ✅ | Sama seperti di atas |
| Finding di-defer | ✅ | `status: "deferred"`, `acceptance_reason`, `deferred_to`, `next_action`, `fix_summary` + 3-layer trail |
| Finding di-accept (accepted risk) | ✅ | `status: "accepted"`, `acceptance_reason`, `fix_summary` + 3-layer trail (**TANPA** `deferred_to`/`next_action`) |
| Finding di-skip sengaja | ✅ | `status: "skipped"`, `fix_summary` berisi alasan skip |
| Akhir fix sprint | ✅ | Batch update SEMUA finding yang di-fix di sprint tsb |
| Registry.json fix_sprint — per finding fixed | ✅ | Update `fixed_*` counts + `findings_fixed[]` |
| Registry.json fix_sprint — per finding deferred | ✅ | Update `deferred_*` counts + `findings_deferred[]` |
| Registry.json fix_sprint — per finding accepted | ✅ | Update `accepted_*` counts + `findings_accepted[]` |
| Registry.json fix_sprint — per finding skipped | ✅ | Update `skipped_*` counts + `findings_skipped[]` |

### Finding File Schema Extension

Field di tiap finding object dalam `findings/{source_type}/{scope}.json`. **Tabel field rules LENGKAP (semua field, base + extension) hanya ada di Appendix A → Field Rules** — jangan salin di sini. Field extension berikut menambah field di atas base fields:

```json
{
  "id": "TRANSACTIONS-006",
  "severity": "high",
  "dimension": "bug-detection",
  "description": "Print template tidak meng-escape user-controlled fields",
  "location": "src/lib/print-utils.ts:42",
  "impact": "Stored XSS lewat field judul dokumen",
  "root_cause": "escapeHtml() tidak dipanggil di 3 print template",
  "trigger": "Buka dokumen dengan judul berisi <script>",
  "recommendation": "Terapkan escapeHtml() di semua print template",
  "status": "deferred",
  "fix_summary": "Di-defer ke WO-2026-08-001 (refactor template)",
  "acceptance_reason": "Refactor besar, butuh shared component dulu",
  "deferred_to": "WO-2026-08-001",
  "next_action": "Extract shared component saat refactor version-history UI"
}
```

> ⚠️ **Base fields tetap wajib** (`id`, `severity`, `dimension`, `description`, `location`, `impact`, `root_cause`, `trigger`, `recommendation`, `status`). Contoh di atas memperlihatkan objek LENGKAP — contoh lama yang cuma menampilkan field extension TIDAK valid sebagai objek berdiri sendiri.
> Field extension (di atas base fields): `fixed_at`, `fix_cycle`, `fix_summary`, `fix_ref`, `verified_at`, `verified_by`, `acceptance_reason`, `deferred_to`, `next_action`. Validasi per-status → Appendix A → Field Rules.

### Registry.json — fix_sprint Enhancement

Field `fix_sprint` di `registry.json` WAJIB di-update setelah setiap fix sprint. **Counts WAJIB sama dengan panjang array** (`findings_fixed[]` dst) — ini yang bikin cross-check [12] bisa diverifikasi:

**Pola count:** `{status}_{severity}` dengan status ∈ `fixed|deferred|accepted|skipped` dan severity ∈ `critical|high|medium|low`. Field count TIDAK wajib ada semua — cukup kombinasi status×severity yang muncul di sprint itu; yang 0 boleh dicantumkan atau dihilangkan (cuma dokumentatif). `findings_skipped[]` + count `skipped_*` wajib ada jika ada finding skipped di sprint tsb:

```json
{
  "fix_sprint": {
    "cycle": "v2.6.3",
    "status": "completed | in_progress | pending",
    "fixed_critical": 1,
    "fixed_high": 2,
    "fixed_medium": 1,
    "fixed_low": 1,
    "deferred_high": 1,
    "accepted_low": 1,
    "skipped_low": 0,
    "findings_fixed": [
      "TRANSACTIONS-001",
      "TRANSACTIONS-002",
      "TRANSACTIONS-003",
      "TRANSACTIONS-004",
      "TRANSACTIONS-005"
    ],
    "findings_deferred": [
      "TRANSACTIONS-006"
    ],
    "findings_accepted": [
      "TRANSACTIONS-007"
    ],
    "findings_skipped": [],
    "details": "5 fixed (1 critical, 2 high, 1 medium, 1 low). 1 high deferred ke WO-2026-08-001. 1 low accepted risk."
  }
}
```

> Catatan: field `remaining_*` boleh dipakai untuk laporan (e.g. `remaining_high`), tapi source of truth tetap counts + array `findings_fixed[]` / `findings_deferred[]` / `findings_accepted[]` / `findings_skipped[]`. Schema lengkap di Appendix A → `registry.json`.

---

## Sub-Agent Protocol — WAJIB untuk Batch/Delegated Audit

Ketika scope audit > 1 file/page dan perlu delegasi ke task/sub-agent, protokol ini WAJIB diikuti untuk mencegah findings loss.

### Kenapa Ini Penting

Tanpa protokol ini, sub-agent return findings via message → lossy karena:
- Context window terbatas — temuan LOW sering di-drop saat summarization
- Merge manual antar sub-agent rawan kelewat
- Tidak ada persistensi — sekali ter-skip, hilang selamanya

**Solusi: File-First Protocol** — sub-agent WAJIB tulis findings ke file, bukan return di message.

### Contract Wajib

**Sebelum deploy sub-agent:**
1. Buat entry di `.opencode/audit/registry.json` — scope, expected findings, status `pending`
2. Siapkan path output file: `.opencode/audit/findings/SYSTEM_AUDIT/{scope-slug}.json` (or appropriate source type)
3. Pastikan folder `findings/SYSTEM_AUDIT/` sudah ada

**Saat deploy sub-agent, prompt WAJIB include:**
```
Scope: [page/module/file]
Output file: .opencode/audit/findings/SYSTEM_AUDIT/[scope-slug].json
Wajib scan: [dimensi minimal]
Contract:
  - Tulis SEMUA temuan ke file — LOW sekalipun
  - Gunakan JSON schema sesuai Appendix A
  - JANGAN return raw findings di message
  - Return hanya:
    {completed, file_path, finding_count, summary}
```

**Setelah sub-agent selesai:**
1. Baca file findings — verifikasi semua severity ter-capture
2. Update `registry.json` — status `completed`, update finding counts
3. Jika sub-agent tidak nulis file → **ANGGAP GAGAL** — deploy ulang

### Self-Enforcement — Sub-Agent

```
□ File findings sudah dibuat di path yang benar?
□ SEMUA severity ter-capture (low sekalipun)?
□ Field wajib (impact, root_cause, trigger, recommendation) terisi?
□ registry.json sudah di-update?
```

---

## Audit Report Template

**WAJIB** gunakan template ini. JANGAN modifikasi struktur tanpa alasan.

```markdown
# System Audit Report: [Mode]
- **Date:** YYYY-MM-DD
- **Project:** [Nama Project]
- **Scope:** [Full System / Feature: nama / Quick: file]
- **Auditor:** system-audit

## Summary

| Severity | Count | Status |
|----------|-------|--------|
| 🔴 Critical | N | Blocking |
| 🟠 High | N | Blocking |
| 🟡 Medium | N | Next iteration |
| 🔵 Low | N | Note only |
| ✅ Passed dimensions | N/{total sesuai mode} | — |

## Dimensions Status

| # | Dimensi | Status | Temuan |
|---|---------|--------|--------|
| 1 | Code Quality | ✅ / ⚠️ / ❌ | N findings |
| 2 | Security | ✅ / ⚠️ / ❌ | N findings |
| 3 | Performance | ✅ / ⚠️ / ❌ | N findings |
| 4 | Error Handling | ✅ / ⚠️ / ❌ | N findings |
| 5 | Bug Detection | ✅ / ⚠️ / ❌ | N findings |
| 6 | Database Schema | ✅ / ⚠️ / ❌ | N findings |
| 7 | API Contract | ✅ / ⚠️ / ❌ | N findings |
| 8 | Architecture | ✅ / ⚠️ / ❌ | N findings |
| 9 | Spec Compliance | ✅ / ⚠️ / ❌ | N gap (lihat compliance-registry) |

## Findings

### [SEVERITY] [{SCOPE}-{NNN}] [Judul Finding]
- **Dimension:** Security | Performance | Code Quality | etc
- **Location:** `path/to/file.php:42`
- **Severity:** 🔴 Critical | 🟠 High | 🟡 Medium | 🔵 Low
- **Issue:** [Deskripsi singkat masalah]
- **Impact:** [Apa yang terjadi di production jika tidak di-fix]
- **Root Cause:** [Kenapa ini terjadi]
- **Trigger:** [Langkah konkret untuk memicu issue]
- **Recommendation:** [Langkah fix konkret]
- **Reference:** `security-first` / OWASP Top 10 A1 — Injection
- **Status:** ❌ Open | ✅ Fixed | ⏸️ Deferred | 🟢 Accepted (risk) — harus ada `Acceptance Reason` | ⏭️ Skipped

### [SEVERITY] [{SCOPE}-{NNN+1}] [Judul Finding]
— ... —

## Remediation Summary

| ID | Severity | Status | Fix Summary | Verified? |
|----|----------|--------|-------------|-----------|
| SA-001 | 🔴 | ✅ Fixed | Add prepared statement | ✅ |
| SA-002 | 🟠 | ⏸️ Deferred | Tunggu API rate limit library | ⏳ |
| SA-003 | 🔵 | 🟢 Accepted | Non-exposed dev-only route — risk diterima user | ✅ |
```

> `SA-001` dst menggunakan scope default `SA` (System Audit) untuk report template. Saat mengisi, gunakan scope aktual project.

### ❌ Contoh Report Buruk

```markdown
Finding: Ada masalah security
Severity: High
Fix: Perbaiki
```

**❌ Masalah:** Tidak ada impact, lokasi, root cause, atau rekomendasi konkret. Developer tidak bisa action.

### ✅ Contoh Report yang Benar

```markdown
### 🔴 SA-001 SQL Injection di User Login
- **Dimension:** Security
- **Location:** `app/Http/Controllers/AuthController.php:34`
- **Severity:** 🔴 Critical
- **Issue:** Raw SQL concatenation pada query login — input `$email` langsung di-interpolasi
- **Impact:** Attacker bisa inject `' OR 1=1 --` untuk bypass login tanpa password valid. Akses ke semua akun.
- **Root Cause:** Developer menggunakan `DB::raw()` tanpa parameter binding
- **Trigger:** POST /login dengan email `' OR 1=1 --` dan password sembarang
- **Recommendation:** Ganti dengan Eloquent atau parameterized query
- **Reference:** `security-first` / OWASP Top 10 A1 — Injection
- **Status:** ❌ Open

❌ Before:
    ```php
    $user = DB::select("SELECT * FROM users WHERE email = '$email' LIMIT 1");
    ```

✅ After:
    ```php
    $user = User::where('email', $email)->first();
    ```
```

> Catatan teknik: fence dalam (```php) di-indent 4 spasi agar tidak menutup fence luar (```markdown).

---

## Remediation Patterns per Dimension

| Dimensi | Common Fix Pattern |
|---------|-------------------|
| Code Quality | Extract method, rename variable, add interface, write test |
| Security | Parameterized query, add validation, implement rate limit, escape output |
| Performance | Add index, eager loading, implement cache, pagination |
| Error Handling | Add try-catch, structured logging, custom exception class |
| Bug Detection | Null check, boundary validation, fix race condition |
| Database Schema | Add missing index, normalize table, fix migration order |
| API Contract | Add missing field, fix response format, add validation rule |
| Architecture | Extract service class, implement repository pattern, fix dependency direction |

---

## Output Location

### Struktur Direktori

Semua audit findings **WAJIB** disimpan di `.opencode/audit/` (gitignored via `.opencode/`):

```
.opencode/audit/
├── registry.json                        ← Master tracker semua scope audit
├── AUDIT_CONTRACT.json                  ← Surface contract (definition of done)
└── findings/
    ├── SYSTEM_AUDIT/                    ← Findings dari system-audit skill
    │   ├── pos.json                     ← 1 file per scope (JSON, strict schema)
    │   └── products.json
    ├── GAP_ANALYSIS/                    ← Findings dari gap analysis
    ├── CODE_REVIEW/                     ← Findings dari code review
    └── MANUAL/                          ← Findings dari manual review
```

Final report **setelah approval user** di-copy ke:
```
.opencode/artifacts/[project-slug]/G4-DEPLOY/YYYY-MM-DD_[mode]-audit.md
```

### Alur Data

```
Sub-agent → tulis findings ke .opencode/audit/findings/{source_type}/{scope}.json
     ↓
Main model → baca semua findings → agregasi → update registry.json
     ↓
Generate final report (draft) → approval user → copy ke G4-DEPLOY
```

### Fallback

Jika project tidak punya `.opencode/` (non-SDLC project): semua audit artifacts (registry, AUDIT_CONTRACT.json, findings) pindah ke `docs/audit/`; artifacts phase_state (`decisions.md`, `KNOWLEDGE_BASE.md`) pindah ke root project. Pemetaan lengkap di Canonical Definitions → Paths.

Untuk Quick Scan tanpa temuan → **tidak perlu simpan file**. Cukup informasikan di chat.

---

## 🚨 Self-Enforcement — System Audit

**Sebelum menyelesaikan audit, WAJIB cek:**

```
□ [1] SCOPE sudah ditentukan? — Full / Feature / Quick?
□ [2] SEMUA dimensi sesuai mode sudah di-scan?
    → Full System: 1-8 semua
    → Feature: 1-5 minimal
    → Quick: 2 + 5 minimal
□ [2b] TWO-PHASE (Full System):
    □ Discovery read-only? — TIDAK ADA fix kode saat Discovery? 🔴
    □ Audit Surface Contract dibuat (AUDIT_CONTRACT.json)?
    □ SEMUA sel contract `verified` (definition of done)?
    □ Tool Battery dijalankan & hasil tercatat (baseline)?
    □ Convergence Gate lulus:
        □ Battery clean / known-gap tercatat
        □ Fresh sub-agent re-scan = 0 temuan baru
        □ Maks 3 pass (bound keras)
    □ Backlog di-freeze + snapshot (commit ref + timestamp)?
    □ Execution: scope-freeze dipatuhi?
        □ Fix-phase discoveries di-triage, bukan di-drop / di-fix interleaved
        □ Tripwire >20% ATAU >10 item (mana yang lebih kecil) → re-discovery terbatas dijalankan?
    □ Per cluster: impact.sh → fix → phpstan/tsc → focused tests?
    □ Akhir Execution: battery ulang + full suite + build hijau?
□ [3] Tiap finding punya:
    □ Severity jelas (🔴🟠🟡🔵)?
    □ Impact konkret?
    □ Root cause?
    □ Trigger scenario?
    □ Recommendation?
□ [4] ❌ Before + ✅ After code disertakan untuk finding 🔴 dan 🟠?
□ [5] Findings disimpan ke file JSON di `.opencode/audit/findings/`?
    □ registry.json sudah ter-update?
    □ Semua severity ter-capture (low sekalipun)?
□ [6] Final report di-generate ke `.opencode/artifacts/[project-slug]/G4-DEPLOY/`?
□ [7] Finding 🔴 dan 🟠 sudah di-remediate atau di-defer dengan alasan?
□ [7b] Finding files sudah di-update statusnya (fixed_at/fix_cycle jika fixed; fix_summary untuk semua non-open)?
□ [7c] Registry.json fix_sprint sudah disync dengan finding files?
□ [7d] **SEBELUM tandai "fixed"** — kode aktual di `location` field sudah dibaca dan diverifikasi?
    □ `verified_at` dan `verified_by` sudah diisi di finding JSON?
    □ Tidak ada false-positive claim (cuma update file tanpa baca kode)?
□ [7e] Finding ACCEPTED/DEFERRED:
    □ WAJIB konfirmasi user (JANGAN auto-accept)?
    □ 3-layer trail lengkap: finding JSON + decisions.md + KNOWLEDGE_BASE.md?
□ [8] Kalau ada finding 🟠 atau 🔴 yang belum di-fix → JANGAN approve deploy
□ [9] Ada temuan yang perlu di-catat di decisions.md? (misal: defer alasan teknis)
□ [10] Self-update: ada gap dalam skill ini? → tanya user, update via self-update workflow
□ [11] Batch audit via sub-agent?
    □ Semua sub-agent return file findings (bukan message)?
    □ File findings diverifikasi isinya?
    □ Tidak ada scope yang terlewat?
□ [12] Finding lifecycle CROSS-CHECK (satu sumber kebenaran antar file):
    □ Count & ID di finding JSON == registry.json fix_sprint (findings_fixed[]/findings_deferred[]/findings_accepted[]/findings_skipped[])?
    □ Panjang tiap array == count fix/defer/accept/skip di fix_sprint?
    □ Status di finding JSON == status di report (Audit Report)?
    □ Item ACCEPTED/DEFERRED ada di decisions.md + KNOWLEDGE_BASE.md (3-layer trail)?
    □ Tidak ada finding `open` yang tersisa di registry padahal report bilang "semua fixed"?
```

**Kalau ragu dengan severity atau rekomendasi → tanya user dulu. JANGAN asumsi.**

---

## ✅ Final Checklist

```
□ Criticality Assessment: STRICT (security, data, dan production risk)
  □ 🔴 Banner terpasang di header?
  □ Konsekuensi nyata disebut?
  □ Kata WAJIB/HARUS/JANGAN digunakan?
  □ 🚨 Self-Enforcement section ada?
  □ Checklist ada di akhir?
□ Two-Phase (Full System Audit):
  □ Discovery read-only (NO fix saat discovery)?
  □ Audit Surface Contract + battery + convergence gate?
  □ Execution terhadap snapshot beku + scope-freeze?
  □ Definition of done terukur, bukan subjektif?
□ Anti-patterns sudah dicek:
  □ #1: Gak ada passive language?
  □ #2: Gak ada instruksi tanpa konsekuensi?
  □ #3: Self-enforcement ada?
  □ #4: Criticality sudah strict?
  □ #5: Instruksi spesifik, gak ambigu?
  □ #6: Ada contoh konkret (❌ + ✅)?
□ Self-update workflow sudah diingat?
  □ "Kalau nemu celah, aku bakal tanya user"

---

## Appendix A: Findings JSON Schema

### `registry.json` — Master Tracker

```json
{
  "cycle": "v2.6.3",
  "started_at": "2026-07-07",
  "updated_at": "2026-07-07",
  "scopes": [
    {
      "slug": "pos",
      "page": "/pos",
      "expected_findings": 0,
      "actual_findings": 3,
      "severity_breakdown": { "critical": 1, "high": 1, "medium": 0, "low": 1 },
      "status": "pending | in_progress | completed"
    }
  ],
  "summary": {
    "total_scopes": 48,
    "completed": 48,
    "total_findings": 224,
    "critical": 3,
    "high": 20,
    "medium": 79,
    "low": 122
  },
  "fix_sprint": {
    "cycle": "v2.6.3",
    "status": "completed | in_progress | pending",
    "fixed_critical": 1,
    "fixed_high": 1,
    "fixed_medium": 0,
    "fixed_low": 0,
    "deferred_high": 0,
    "accepted_medium": 0,
    "accepted_low": 1,
    "findings_fixed": [
      "TRANSACTIONS-001",
      "TRANSACTIONS-002"
    ],
    "findings_deferred": [],
    "findings_accepted": [
      "TRANSACTIONS-003"
    ],
    "findings_skipped": [],
    "details": "2 fixed (1 critical, 1 high). 1 low accepted risk. Summary di-sync dengan counts & arrays."
  }
}
```

### `findings/{source_type}/{scope}.json` — Per Scope Findings

File diletakkan di subdirektori source type (lihat Source Type Classification): `findings/SYSTEM_AUDIT/{scope}.json`, `findings/CODE_REVIEW/{scope}.json`, dst.

```json
{
  "scope": "/pos",
  "audit_date": "2026-07-07",
  "dimensions_scanned": ["security", "bug-detection", "code-quality"],
  "findings": [
    {
      "id": "POS-001",
      "severity": "critical",
      "dimension": "security",
      "description": "Branch selector — useState snapshot never syncs with store",
      "location": "posmid-web/src/app/pos/page.tsx:55",
      "impact": "User switch branch but see old branch data until hard refresh",
      "root_cause": "useState(currentBranchId) pattern — takes initial value but never subscribes to store changes",
      "trigger": "1. Open /pos with branch A\n2. Switch to branch B via header\n3. POS still shows branch A products",
      "recommendation": "Replace useState with direct useAppStore subscription",
      "status": "fixed",
      "fixed_at": "2026-07-07",
      "fix_cycle": "v2.1",
      "fix_summary": "Replaced useState with zustand subscription pattern",
      "fix_ref": "posmid-web/src/app/pos/page.tsx",
      "verified_at": "2026-07-07",
      "verified_by": "code-review"
    }
  ],
  "summary": {
    "total": 1,
    "critical": 1,
    "high": 0,
    "medium": 0,
    "low": 0
  }
}
```

> Contoh di atas menampilkan objek dengan `status: "fixed"` — field `fixed_at`/`fix_cycle`/`fix_summary`/`verified_at`/`verified_by` berlaku. Untuk status lain, lihat kewajiban field di Field Rules (`acceptance_reason` hanya untuk `accepted`/`deferred`; `deferred_to`/`next_action` hanya untuk `deferred`).

### Field Rules

| Field | Wajib? | Validasi |
|-------|--------|----------|
| `scope` | ✅ | String, slug format |
| `audit_date` | ✅ | ISO date: `YYYY-MM-DD` |
| `dimensions_scanned` | ✅ | Array enum: subset dari 9 dimensi yang di-scan |
| `findings[].id` | ✅ | `{SCOPE}-{NNN}` pattern |
| `findings[].severity` | ✅ | Enum: `critical`, `high`, `medium`, `low` |
| `findings[].dimension` | ✅ | Enum: 9 dimensi (lihat Audit Dimensions) |
| `findings[].description` | ✅ | Deskripsi singkat masalah |
| `findings[].location` | ✅ | Path file + baris: `path/to/file.php:42` |
| `findings[].impact` | ✅ | Minimal 1 kalimat deskriptif |
| `findings[].root_cause` | ✅ | Minimal 1 kalimat |
| `findings[].trigger` | ✅ | Step-by-step konkret |
| `findings[].recommendation` | ✅ | Actionable, bukan abstraksi |
| `findings[].status` | ✅ | Enum: `open`, `fixed`, `deferred`, `accepted`, `skipped` |
| `findings[].fixed_at` | ✅ jika `status: "fixed"` | Format ISO date: `YYYY-MM-DD` |
| `findings[].fix_cycle` | ✅ jika `status: "fixed"` | Cycle ID: `v2.1` |
| `findings[].fix_summary` | ✅ jika `status != "open"` | 1 kalimat deskriptif tentang fix/defer/accept/skip |
| `findings[].fix_ref` | 🔶 Opsional | Path file atau commit ref |
| `findings[].verified_at` | ✅ jika `status: "fixed"` | ISO date when code was verified |
| `findings[].verified_by` | ✅ jika `status: "fixed"` | `"code-review"` (default) — who verified the fix |
| `findings[].acceptance_reason` | ✅ jika `status: "accepted"/"deferred"` | Rationale — kenapa sengaja tidak di-fix |
| `findings[].deferred_to` | ✅ jika `status: "deferred"` | Fase/WO target berikutnya — WAJIB untuk deferred (bukan accepted) |
| `findings[].next_action` | ✅ jika `status: "deferred"` | Trigger konkret untuk solve nanti — WAJIB untuk deferred (bukan accepted) |
```
