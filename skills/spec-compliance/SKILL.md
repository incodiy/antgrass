---
name: spec-compliance
description: Spesifikasi-kesuaian audit (docs-to-code compliance) — memastikan setiap FR/tabel/endpoint/seeder/feature yang terdokumentasi di PRD, ERD, API-SPEC, ARCHITECTURE, MODULE-SYSTEM, SEEDER-SPEC, dan dokumen plan inti benar-benar terimplementasi di kode (atau tercatat sebagai gap). Metode 2-arah (docs→code & code→docs) dengan output compliance-registry + gap list. Dipanggil oleh system-audit & sdlc-gate-deployment sebagai dimensi ke-9, oleh sdlc-gate-development tiap WO close, atau manual via /compliance.
version: 1.0.0
updated: 2026-08-10
loads: [sdlc-project-templates]
---

# Spec Compliance Audit (Docs↔Code)

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI

**Skill ini menutup blind spot yang TIDAK dicakup system-audit.** System-audit mengukur KUALITAS kode yang ADA (security, performance, dsb). Skill ini mengukur KESESUAIAN: apakah semua yang DIJANJIKAN di dokumen plan benar-benar ada di kode. Dua hal berbeda — kode yang "bagus" tetap bisa gagal total jika fitur yang dijanjikan tidak pernah diimplementasi.

🔴 **Konsekuensi jika dilanggar:**
- **Audit compliance dilewatkan** → fitur PRD yang tidak pernah dikoding tetap ditandai "COMPLETED" → user percaya fitur ada padahal tidak (kejadian nyata: Sprint 10-12 POSMID)
- **Verifikasi 1-arah saja** → modul hilang tidak terdeteksi, atau endpoint yatim (tanpa pasangan spec) dianggap aman
- **Status di-klaim tanpa baca kode aktual** → false COMPLETED, gap mengendap tanpa disadari
- **Gap tanpa WO** → gap tidak pernah dieksekusi, utang teknis menumpuk
- **Registry tidak di-update tiap WO close** → gap balik lagi di sesi berikutnya

**Model WAJIB** menjalankan SEMUA langkah dua-arah + menulis status ke compliance-registry. JANGAN pernah klaim "implemented" tanpa membaca kode aktual di lokasi yang ditunjuk.

---

## Canonical Definitions — SINGLE SOURCE OF TRUTH

> **Semua definisi terstruktur di skill ini HANYA di section ini.** Section lain boleh memakai nilai ini, TAPI JANGAN menulis ulang definisinya.

### Status Compliance (4)

| Status | Arti | Bukti yang Diperlukan |
|--------|------|----------------------|
| `implemented` | Fitur/tabel/endpoint ada di kode DAN berfungsi sesuai spec | Baca kode aktual + endpoint/tabel/skema cocok + (jika ada) test |
| `partial` | Ada di kode tapi tidak lengkap/tidak ter-wire/tidak sesuai spec | Sebagian sesuai, sebagian hilang/salah — sebutkan detail apa yang kurang |
| `stub` | Hanya kerangka (method kosong, return dummy, CRUD shell tanpa logika, route tanpa controller method) | Baca controller/service — body tidak melakukan perilaku yang dijanjikan |
| `missing` | Tidak ada di kode sama sekali (tabel, model, controller, route, feature) | Pencarian file/route/migration tidak menemukan apa pun |

### Severity Gap (3)

| Severity | Kriteria | Prioritas WO |
|----------|----------|--------------|
| 🔴 **Critical** | Data loss/duplicate, transaksi salah hitung, auth/tenant bypass, route 500, fitur inti POS/Inventory hilang | Batch 1 — blocking |
| 🟠 **High** | Fitur yang dijanjikan PRD tidak berfungsi (engines, module gating), ±tabel ERD tanpa migrasi | Batch 2 — segera |
| 🟡 **Low** | Partial cosmetic, seeds belum lengkap, doc-status tidak sinkron | Batch 3+ |

### Sumber Dokumen Plan (canonical)

| Dokumen | Lokasi | Isi yang Diverifikasi |
|---------|--------|----------------------|
| PRD | `.opencode/artifacts/{slug}/G0-PRD/PRD.md` | FR (FR-XXX-NNN), business types, layer modul |
| ERD | `.opencode/artifacts/{slug}/G1-ARCH/ERD.md` | Semua tabel + relasi |
| API-SPEC | `.opencode/artifacts/{slug}/G1-ARCH/API-SPEC.md` | Semua endpoint |
| ARCHITECTURE | `.opencode/artifacts/{slug}/G1-ARCH/ARCHITECTURE.md` | Desain sistem, layanan |
| MODULE-SYSTEM | `.opencode/artifacts/{slug}/G1-ARCH/MODULE-SYSTEM.md` | Layer modul, manifest, dependencies |
| SEEDER-SPEC | `.opencode/artifacts/{slug}/G3-DEV/SEEDER-SPEC.md` | Seeder wajib + isi demo |
| IMPLEMENTATION-PLAN | `.opencode/artifacts/{slug}/G3-DEV/IMPLEMENTATION-PLAN.md` | Task/sprint status (jangan dipercaya — verifikasi ulang) |
| MODULE-CATALOG | `.opencode/artifacts/{slug}/G3-DEV/MODULE-CATALOG.md` | Daftar modul + build order (output skill ini) |

> **Aturan ketidakpercayaan:** IMPLEMENTATION-PLAN adalah *klaim*, bukan bukti. Status COMPLETED di sana WAJIB diverifikasi ulang ke kode aktual. Kejadian nyata: Sprint 10-12 ditandai COMPLETED tapi kode masih stub.

### Paths — canonical

| Artifact | Path |
|----------|------|
| Compliance registry | `.opencode/audit/compliance/compliance-registry.json` |
| Compliance report (ringkas) | `.opencode/audit/compliance/COMPLIANCE.md` |
| Findings compliance (per scope) | `.opencode/audit/findings/COMPLIANCE/{scope}.json` |
| Final compliance report (approved) | `.opencode/artifacts/{slug}/G4-DEPLOY/YYYY-MM-DD_spec-compliance.md` |
| WO untuk gap | via waypoint: `.opencode/phase_state/{slug}/` (SPRINT_BACKLOG) |
| Development history | `.opencode/phase_state/{slug}/DEVELOPMENT-HISTORY.md` |
| 3-layer trail (decisions/KNOWLEDGE_BASE) | `.opencode/phase_state/{slug}/decisions.md` + `KNOWLEDGE_BASE.md` |
| Fallback (tanpa `.opencode/`) | audit → `docs/audit/compliance/`; phase_state → root project |

### ID Pattern Gap

```
Gap ID: {MODULE}-{NNN}   (MODULE = kode modul, NNN = 3 digit zero-padded)
Contoh: PLT-001 (platform), WFL-002 (workflow engine), CRM-001 (customer)
Prefix wajib di registry: GAP-{MODULE}-{NNN}
```

---

## When to Load

| Condition | Action |
|-----------|--------|
| G4 — QA & Deploy exit criteria | Compliance audit penuh 4-layer (bersama system-audit) |
| Tiap WO close (G3/G3-MNT) | Re-verify scope WO — update status registry dari `stub/missing/partial` ke `implemented` (atau sebaliknya) |
| User minta `/compliance` | Compliance audit manual |
| Setelah refactor besar / module baru | Re-verify modul tsb |
| Sesi baru dengan gap status "IMPLEMENTED" di plan | Verifikasi spot-check |

---

## Audit Metodologi — 2 Arah (WAJIB)

**Kedua arah WAJIB dijalankan.** Satu arah saja = blind spot.

```
ARAH 1: DOCS → CODE
  Tiap FR / tabel / endpoint / seeder yang dijanjikan dokumen
  → cari di kode (route, controller, service, model, migration, seeder, test)
  → beri status: implemented | partial | stub | missing
  → catat lokasi kode + bukti

ARAH 2: CODE → DOCS
  Tiap route / controller method / migration / tabel / model / feature yang ADA
  → cari pasangannya di dokumen (spec FR / ERD / API-SPEC / PRD)
  → tidak ada pasangan = "orphan code" (bukan gap, tapi catat — bisa jadi feature tak terdokumentasi
    atau kode yang sudah tidak relevan → kandidat cleanup WO)
```

### Metode Verifikasi Per Artifact

| Artifact Dokumen | Verifikasi di Kode | Contoh Perintah |
|------------------|--------------------|------------------|
| Endpoint (API-SPEC) | Route terdaftar? Controller method ada & bukan kosong? | `rg "route" routes/api_v1.php`, baca controller |
| Tabel (ERD) | Migration ada? | `ls database/migrations` + grep nama tabel |
| FR (PRD) | Behavior ada di service/controller? (BUKAN cuma nama method) | baca body method |
| Seeder (SEEDER-SPEC) | Seeder mengisi sesuai spec? | baca `database/seeders` |
| Model & relasi | Model ada + relasi sesuai ERD? | baca `app/Models` |

**Aturan verifikasi (WAJIB):**
1. **Baca body method/service — JANGAN cuma lihat nama method.** Method `store()` yang hanya CRUD shell (tanpa logika bisnis yang dijanjikan FR) = `stub`, bukan `implemented`.
2. **Route vs controller cross-check:** route terdaftar tapi method controller tidak ada → `stub` + 🔴 (500 error). Controller method ada tapi kosong/return dummy → `stub`.
3. **Check wiring (engines):** fitur diklaim ada tapi tidak dipanggil dari alur bisnis → `partial`. Contoh: pricing rules ada tapi POS tidak pernah memanggilnya → `partial`.
4. **Enum/branching check:** fitur ber-varian (business_type, module) → pastikan ada enum + branching nyata, bukan string dekoratif.
5. **Battery ringan (opsional saat perlu bukti runtime):** jalankan test terkait / `php artisan route:list`.

---

## Workflow — 6 Langkah

### Langkah 1: SCOPE
Tentukan scope compliance: layer (Platform/Engines/Core/Modules/Addons), modul, atau app (api/web/mobile). Baca MODULE-CATALOG untuk daftar lengkap modul + build order.

### Langkah 2: Ekstrak Requirement dari Dokumen
Dari dokumen plan (canonical paths), kumpulkan per scope:
- FR list (dari PRD) + tabel (dari ERD) + endpoint (dari API-SPEC) + seeds (SEEDER-SPEC)
- Fitur per modul + dependensi (MODULE-CATALOG)

### Langkah 3: Verifikasi Arah 1 (docs→code)
Untuk tiap requirement → cari di kode → tentukan status (implemented/partial/stub/missing) → catat lokasi bukti. Sub-agent disarankan untuk scope besar (lihat Sub-Agent Protocol).

### Langkah 4: Verifikasi Arah 2 (code→docs)
Enumerasi kode (routes, controller methods, migrations, models) → cari pasangan spec → catat orphan code.

### Langkah 5: Registry + Gap List
- Update `compliance-registry.json` (semua item + status + bukti)
- Gap (`missing`/`stub`/`partial`) → `findings/COMPLIANCE/{scope}.json` + update `COMPLIANCE.md`
- Setiap gap diberi severity + rekomendasi WO (ikut kanonik waypoint: WO BUG/GAP/ENHANCEMENT)
- Tiap gap 🔴/🟠 → wajib ada WO di SPRINT_BACKLOG

### Langkah 6: REPORT + Approval
Generate report ringkas di chat + update `DEVELOPMENT-HISTORY.md`. Setelah approval user → copy ke `G4-DEPLOY/`. Execution gap = via feature-workflow (bukan di sini).

---

## Compliance Registry — Schema

File: `.opencode/audit/compliance/compliance-registry.json`

```json
{
  "project": "posmid",
  "generated_at": "2026-08-10",
  "version": "1",
  "sources": ["PRD", "ERD", "API-SPEC", "ARCHITECTURE", "MODULE-SYSTEM", "SEEDER-SPEC"],
  "layers": [
    {
      "layer": "platform",
      "order": 1,
      "modules": [
        {
          "module": "auth",
          "code": "AUTH",
          "status": "implemented",
          "items": [
            {
              "item": "FR-AUTH-001",
              "source": "PRD §4",
              "spec": "login dengan email+password",
              "status": "implemented",
              "evidence": ["app/Http/Controllers/Api/V1/AuthController.php", "app/Services/Tenant/AuthService.php"],
              "note": ""
            }
          ]
        }
      ]
    }
  ],
  "summary": {
    "total_items": 0,
    "implemented": 0,
    "partial": 0,
    "stub": 0,
    "missing": 0,
    "gap_critical": 0,
    "gap_high": 0,
    "gap_low": 0,
    "orphan_code_items": 0
  }
}
```

> **Aturan maintenance (WAJIB):** status hanya berubah setelah verifikasi kode aktual. Tiap WO close → re-verify → update `status` + `evidence` + `updated_at`. Registry ini = satu-satunya sumber kebenaran status compliance (BUKAN IMPLEMENTATION-PLAN).

---

## Sub-Agent Protocol (untuk scope besar)

Ketika scope compliance > 1 modul dan perlu delegasi, protokol File-First WAJIB dipakai (mencegah findings loss):

1. Siapkan output file per scope: `.opencode/audit/findings/COMPLIANCE/{scope}.json`
2. Prompt sub-agent WAJIB berisi: scope + output file + daftar requirement yang harus diverifikasi + aturan (baca body method, cross-check route, wire-check) + JANGAN return raw findings di message
3. Setelah selesai → baca file → verifikasi → update registry
4. Sub-agent yang tidak menulis file = GAGAL → deploy ulang

---

## 🚨 Self-Enforcement — Spec Compliance

**Sebelum menyelesaikan compliance audit, WAJIB cek:**

```
□ [1] Kedua arah dijalankan? (docs→code DAN code→docs)
□ [2] Setiap status di-klaim setelah baca KODE AKTUAL (bukan nama method / status plan)?
□ [3] Route vs controller cross-check dilakukan? (route tanpa method = stub 🔴)
□ [4] Wiring check dilakukan untuk engines? (fitur tidak dipanggil = partial, bukan implemented)
□ [5] Enum/branching dicek untuk fitur ber-varian? (string dekoratif = stub)
□ [6] Gap 🔴/🟠 punya WO di SPRINT_BACKLOG?
□ [7] compliance-registry.json + COMPLIANCE.md + findings/COMPLIANCE/{scope}.json ter-update?
□ [8] DEVELOPMENT-HISTORY.md tercatat?
□ [9] IMPLEMENTATION-PLAN tidak dipercaya mentah — diverifikasi ulang?
□ [10] Orphan code dicatat (bukan di-drop)?
```

**Kalau ragu dengan status atau severity → tanya user dulu. JANGAN asumsi.**

---

## ✅ Final Checklist

```
□ Criticality Assessment: STRICT (fitur hilang = utang teknis & user kecewa)
  □ 🔴 Banner terpasang
  □ Konsekuensi nyata disebut
  □ Kata WAJIB/HARUS/JANGAN digunakan
  □ 🚨 Self-Enforcement ada
  □ Checklist ada
□ Anti-patterns dicek:
  □ #1: Tidak ada passive language
  □ #2: Tidak ada instruksi tanpa konsekuensi
  □ #3: Self-enforcement ada
  □ #4: Strict/fleksibel dibedakan
  □ #5: Instruksi spesifik
  □ #6: Ada contoh konkret
□ Terintegrasi: dipanggil system-audit (dimensi 9) + gate-deployment + tiap WO close
□ Self-update: kalau nemu celah → tanya user
```

---

## Contoh Penerapan (POSMID)

**Contoh status yang benar (dari audit Batch 0):**

| Item | Spec | Status | Alasan |
|------|------|--------|--------|
| `business_type` enum 12 tipe | PRD §2.1 | `stub` 🟠 | Disimpan sebagai string, 0 branching, tanpa enum (`app/Enums` kosong) |
| Workflow Engine (FR-WF-001..005) | PRD §9 | `stub` 🔴 | `WorkflowController@show/update/destroy` di-route tapi tak ada di controller → 500 |
| Pricing rules → dipakai di POS | PRD §13 | `partial` 🟠 | CRUD ada, tapi POS tidak pernah memanggilnya (base_price langsung) |
| `EnsureModuleEnabled` gating | MODULE-SYSTEM §2 | `stub` 🟠 | 0 usage di route; `tenant_modules` tanpa migrasi |
| Tabel `bookings` | ERD | `missing` 🟠 | Tidak ada migration |
