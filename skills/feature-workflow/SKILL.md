---
name: feature-workflow
description: FSM controller untuk eksekusi fitur enterprise. Pipeline 6 fase: PM → Architect → Dev Planning → PM Verify → Tech Lead Audit → Execution. Safety latch + retry mechanism + fallback. Wajib di-load di G3 saat task fitur baru.
version: 1.1.1
updated: 2026-07-13
---

# Feature Workflow — FSM Orchestrator

> Pipeline FSM untuk eksekusi fitur enterprise. Di-load `on-demand` dari `sdlc-gate-development` saat task terdeteksi sebagai fitur baru.

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI

Skill ini adalah **state machine controller** untuk pipeline multi-agent. Pelanggaran terhadap state transition bisa menyebabkan eksekusi kode sebelum approval — risiko keamanan tinggi.

🔴 **Konsekuensi jika dilanggar:**
- Safety latch dilewati → kode ditulis tanpa review → bug/security hole ke production
- State transisi salah → agent manggil output yang belum ada → pipeline gagal
- Retry limit diabaikan → infinite loop, token boros, user frustration
- Fallback tidak dijalankan saat 2+ consecutive failures → user stuck tanpa solusi

**Model WAJIB** mematuhi state machine, safety latch, dan retry mechanism di bawah. JANGAN skip fase. JANGAN tulis kode sebelum USER_APPROVAL_PENDING.

---

## 1. Trigger Detection

Sebelum memulai workflow, **klasifikasi task** dari user request:

| Kata Kunci / Intent | Mode | Fase yang Jalan |
|---------------------|------|-----------------|
| "Bikin fitur", "Buat [x]", "Tambah [x]", "Implementasi [x]", "Feature request" | **FULL** | PM → Architect → Dev → PM Verify → TL Audit → Execute |
| "Bug", "Error", "Gagal", "Broken", "Fix [x]", "Typo" | **QUICK** | Dev → PM Verify → TL Audit |
| "Hotfix", "Critical", "Production down", "Urgent" | **HOTFIX** | Dev → Execute (skip review, log sebagai hotfix) |
| "Refactor", "Refactor [x]", "Clean up", "Optimize" | **QUICK** | Dev → PM Verify → TL Audit |

User bisa override: "pake pipeline full" atau "gas langsung" — ikuti instruksi eksplisit user.

### Init Workflow

```
WORKFLOW_DIR = ".opencode/workflows/[feature-name-slug]/"

1. Tentukan PROJECT_ROOT dari system prompt ("Working directory")
2. Buat folder: {{PROJECT_ROOT}}/{{WORKFLOW_DIR}}/artifacts/
3. Init state.md dari template di bawah
4. Mode: sesuai klasifikasi
5. Mulai pipeline sesuai mode
```

---

## 1a. The Agentic Loop

> Skill ini mengimplementasikan **Agentic Loop**: siklus **Plan → Execute → Observe → Repeat** untuk tiap fase pipeline.

```
Loop per fase:
  ┌──────────────────────┐
  │ PLAN                 │ ← Baca input artifacts, tentuin approach
  │   @agent dispatch    │
  ▼                      │
  ┌──────────────────────┐
  │ EXECUTE              │ ← Agent ngerjain task
  │   @agent nulis file  │
  ▼                      │
  ┌──────────────────────┐
  │ OBSERVE              │ ← Baca output, validity check
  │   validasi artifacts │
  ▼                      │
  ┌──────────────────────┐
  │ REPEAT / HANDOFF     │ ← Pass ke fase berikutnya atau retry
  │   update state.md    │
  └──────────────────────┘
```

Mapping ke FSM states:
- **Plan** = `*_PENDING` states (PM_ANALYSIS_PENDING, dll)
- **Execute** = @agent dispatch phase
- **Observe** = validity check + user verification
- **Repeat** = retry mechanism (balik ke fase sama)
- **Handoff** = transition ke state berikutnya

---

## 2. FSM States & Transitions

### FULL Mode

```
START
  │
  ▼
┌─────────────────────┐
│ PM_ANALYSIS_PENDING │ ◄── Dispatch @agent PM
└─────────┬───────────┘
          │ @agent selesai + valid
          ▼
┌──────────────────────────┐
│ PM_ANALYSIS_COMPLETE     │
│ → user verify output     │
└─────────┬────────────────┘
          │ user approve
          ▼
┌───────────────────────────┐
│ ARCHITECT_DESIGN_PENDING  │ ◄── Dispatch @agent Architect
└───────────┬───────────────┘
            │ @agent selesai + valid
            ▼
┌─────────────────────────────┐
│ ARCHITECT_DESIGN_COMPLETE   │
└───────────┬─────────────────┘
            │ auto
            ▼
┌───────────────────────┐
│ DEV_PLANNING_PENDING  │ ◄── Dispatch @agent Developer
└───────────┬───────────┘
            │ @agent selesai + valid
            ▼
┌───────────────────────────┐
│ DEV_PLANNING_COMPLETE     │
└───────────┬───────────────┘
            │ auto
            ▼
┌───────────────────────────┐
│ PM_VERIFICATION_PENDING   │ ◄── Dispatch @agent PM (review mode)
└───────────┬───────────────┘
            │
      ┌─────┴─────┐
      │           │
      ▼           ▼
 APPROVED    REJECTED
      │           │
      │      ┌────┴────┐
      │      │ retry < │
      │      │    3?   │
      │      └──┬──┬───┘
      │    YES  │  │ NO
      │         ▼  ▼
      │    → Dev   → ESKALASI
      │    retry
      ▼
┌──────────────────────────┐
│ TECH_LEAD_AUDIT_PENDING  │ ◄── Dispatch @agent Tech Lead
└───────────┬──────────────┘
            │
      ┌─────┼─────┐
      │     │     │
      ▼     ▼     ▼
 APPROVED  VETOED  CRITICAL
      │     │     │
      │     │     └────────► DEV_PLANNING_PENDING
      │     └──────────────► ARCHITECT_DESIGN_PENDING
      ▼
┌───────────────────────────┐
│ USER_APPROVAL_PENDING     │ ◄── Tanya user: "Approve?"
└───────────┬───────────────┘
            │ user approve
            ▼
┌───────────────────────┐
│ 🔐 SAFETY LATCH CHECK │
└───────────┬───────────┘
            │ pass
            ▼
┌──────────────────────┐
│ EXECUTION_IN_PROGRESS │ ◄── @agent Developer (execution mode)
└──────────┬───────────┘
           │ selesai
           ▼
┌──────────────────────┐
│ EXECUTION_COMPLETE   │ → END
└──────────────────────┘

           │ gagal 3x
           ▼
┌──────────────────────┐
│ FAILED → CANCELLED    │
└──────────────────────┘
```

### QUICK Mode

```
DEV_PLANNING_PENDING → PM_VERIFICATION_PENDING → TECH_LEAD_AUDIT_PENDING → USER_APPROVAL_PENDING → EXECUTION
```

### HOTFIX Mode

```
DEV_PLANNING_PENDING → (log: "hotfix — no review") → EXECUTION
```

---

## 3. Pipeline Phases

| # | Fase | Role Skill | Input | Output | Handoff | Retry |
|---|------|------------|-------|--------|---------|-------|
| 1 | PM Analysis | `role-pm` | user request | `artifacts/feature.md` | user approve → phase 2 | 3x → eskalasi |
| 2 | Architect Design | `role-architect` | `feature.md` | `artifacts/solution.md` | auto → phase 3 | 3x → eskalasi |
| 3 | Dev Planning | `role-developer` | `feature.md` + `solution.md` | `artifacts/implementation_steps.md` | auto → phase 4 | 3x → eskalasi |
| 4 | PM Verification | `role-pm` | `implementation_steps.md` | `artifacts/pm_view.md` | approved → phase 5, rejected → phase 3 | 3x → eskalasi |
| 5 | Tech Lead Audit | `role-tech-lead` | semua artifacts | `artifacts/audit.md` | approved → phase 6, vetoed → phase 2, critical → phase 3 | 1x (no retry) |
| 6 | Execution | `role-developer` | semua artifacts + user approval | kode + `walkthrough.md` | — | 3x → eskalasi |

---

## 4. Safety Latch

```
## 🔴 SAFETY LATCH — WAJIB SEBELUM EKSEKUSI

Sebelum fase EXECUTION berjalan, WAJIB lakukan 3 check:

CHECK 1: Stage Validation
  Baca {{WORKFLOW_DIR}}/state.md
  → Apakah "Current Stage" == "USER_APPROVAL_PENDING"?
    ✅ YA → lanjut
    ❌ TIDAK → STOP. Output:
      "> [FAILURE] Unauthorized execution. Stage must be USER_APPROVAL_PENDING."

CHECK 2: Artifact Completeness
  → Apakah semua artifacts di registry berstatus "completed"?
    ✅ YA → lanjut
    ❌ TIDAK → STOP. Output:
      "> [FAILURE] Missing artifacts: [list yang belum completed]"

CHECK 3: User Confirmation (final gate sebelum kode ditulis)
  → Tanya user: "Feature [nama] sudah siap di-execute. Approve?"
    ✅ YA → update state ke EXECUTION_IN_PROGRESS, mulai nulis kode
    ❌ TIDAK → stay di USER_APPROVAL_PENDING, jangan nulis kode

  > Distinction: USER_APPROVAL_PENDING adalah STATE yang mencatat bahwa pipeline sudah selesai.
  > Safety Latch CHECK 3 adalah PROMPT terakhir yang minta user konfirmasi sebelum kode beneran ditulis.
  > Jadi walaupun state sudah USER_APPROVAL_PENDING, tetap ada prompt konfirmasi sebagai safety net.
```

---

## 5. Retry & Recovery

### Layer 1: Auto Retry (per @agent)

```
@agent output
  → validity check
    ✅ PASS → update state, handoff ke next phase
    ❌ FAIL → retry_count++
      → < max_retry: dispatch @agent ulang dengan pesan:
          "Output sebelumnya gagal validasi: [alasan]. Fix dan ulangi."
      → >= max_retry: eskalasi ke Layer 2
```

**Catatan:** Tiap fase punya `max_retry` sendiri. Cek kolom **Retry** di pipeline phase table:
- Fase 1-4 & 6: max 3 retries
- Fase 5 (Tech Lead Audit): max 1 retry (no retry — sekali gagal langsung eskalasi)

### Layer 2: User Eskalasi (1 fase gagal 3x)

```
┌─────────────────────────────────────────────────────┐
│ ⚠️ FASE GAGAL 3x                                     │
│ Fase: [nama fase] — Role: [nama role]                │
│ Alasan: [penyebab kegagalan]                         │
│                                                      │
│ Pilih opsi:                                          │
│ [1] Skip fase — lanjut ke fase berikutnya             │
│ [2] Manual input — kamu isi output-nya sendiri       │
│ [3] 🔄 Rollback single model —                       │
│     pipeline bubar, lanjut tanpa FSM                 │
│ [4] ❌ Batal total — cancel fitur ini                │
└─────────────────────────────────────────────────────┘
```

### Layer 3: Auto Fallback (2+ consecutive failures)

```
KETIKA: 2 fase berbeda gagal berturut-turut (consecutive)
MAKA:
  1. STOP pipeline
  2. Output: "> [WARNING] FSM unstable — 2+ consecutive failures detected."
  3. Log warning di state.md
  4. Auto rollback ke single model mode
  5. Lanjut eksekusi tanpa pipeline
```

### Layer 4: State Machine Violation

```
KETIKA: State transition tidak valid terdeteksi
  (misal: lompat dari PM_ANALYSIS_PENDING ke EXECUTION_IN_PROGRESS)
MAKA:
  1. STOP semua aktivitas
  2. Output:
     "> [FAILURE] Invalid state transition: [current] → [attempted].
      Pipeline terminated. Manual intervention required."
  3. Update state ke FAILED
```

---

## 6. @Agent Dispatch Pattern

Tiap @agent WAJIB dipanggil dengan pola berikut:

### Sequential Dispatch

```
@agent [Role]-[Fase]:
  📋 ROLE: Kamu adalah [Product Manager|System Architect|Software Engineer|Tech Lead]
  📋 SKILL: Baca {{path ke role skill}}
  📋 MODE: [FULL|QUICK|HOTFIX]
  📋 TASK: [deskripsi spesifik fase]
  📋 INPUT:
    [isi artifact sebelumnya atau user request]
  📋 OUTPUT: Simpan ke {{WORKFLOW_DIR}}/artifacts/[file]
  📋 TEMPLATE OUTPUT: Ikuti template di SKILL.md role-mu
```

### Penting

1. @agent dipanggil **SATU PER SATU** — sequential, bukan parallel. Tiap fase dependen pada output fase sebelumnya.
2. Setelah @agent selesai, **BACA output** dari file artifact.
3. **VALIDATE** output dengan Validity Check di bawah.
4. **UPDATE state.md** dengan stage baru.
5. **LOG** di execution log state.md.

---

## 7. Validity Check per Fase

WAJIB dijalankan SETELAH tiap @agent selesai:

### feature.md (PM Analysis)
```
required:
  - title: non-empty
  - description: non-empty
  - acceptance_criteria: >= 3 items
  - nfrs: >= 2 items
  - sad_paths: >= 2 items
forbidden:
  - "TBD", "TODO", "TBD", "to be determined"
```

### solution.md (Architect)
```
required:
  - architecture_diagram (mermaid): present
  - proposed_solutions: >= 2 options
  - recommendation: present
  - components_affected: non-empty
  - security_considerations: non-empty
```

### implementation_steps.md (Dev Planning)
```
required:
  - steps: >= 3
  - files_changed (create + modify): non-empty
  - test_plan: non-empty
  - rollback_plan: non-empty
```

### pm_view.md (PM Verification)
```
required:
  - verdict: "approved" | "rejected"
  - gaps: array (empty if approved)
```

### audit.md (Tech Lead)
```
required:
  - verdict: "approved" | "vetoed" | "critical"
  - blocking_issues: array (empty if approved)
```

---

## 7a. Sandbox & Approval Integration

Setiap fase di pipeline wajib ngecek sandbox mode dan approval policy:

| Fase | Sandbox Recomended | Approval Policy |
|------|-------------------|-----------------|
| 1-5 (Planning & Review) | Bubble Wrap | Request |
| 6 (Execution) | Seatbelt (production) / Bubble Wrap (exploratory) | Untrusted |
| CI/CD / Headless | Seatbelt | Never |

**Detail selengkapnya:** Lihat skill `sandbox-security` — sandbox modes, approval policies, permission overrides, dan safety rules.

---

## 8. state.md Template

WAJIB init file ini saat workflow dimulai:

```markdown
# Feature Workflow State

## Metadata
- **Feature:** [nama fitur]
- **Task ID:** [unique id]
- **Started At:** [timestamp]
- **Mode:** [FULL | QUICK | HOTFIX]

## Current State
- **Stage:** `PM_ANALYSIS_PENDING`
- **Active Agent:** `none`
- **Retry Count:** 0
- **Consecutive Failures:** 0
- **Phase Retry Count:** 0

## Artifact Registry
| Artifact | Status | Path |
|----------|--------|------|
| artifacts/feature.md | pending | artifacts/feature.md |
| artifacts/solution.md | pending | artifacts/solution.md |
| artifacts/implementation_steps.md | pending | artifacts/implementation_steps.md |
| artifacts/pm_view.md | pending | artifacts/pm_view.md |
| artifacts/audit.md | pending | artifacts/audit.md |

## Execution Log
| # | Timestamp | Agent | Action | Result |
|---|-----------|-------|--------|--------|
| 1 | [timestamp] | - | Init workflow | OK |

## User Approval
- **Status:** pending | approved | rejected
- **Approved At:** -
- **Notes:** -
```

---

## 9. Self-Enforcement

**Sebelum memulai workflow, WAJIB cek:**
```
□ Task terklasifikasi (FULL/QUICK/HOTFIX)?
□ Folder {{WORKFLOW_DIR}} sudah dibuat?
□ state.md sudah ter-init dengan stage awal yang benar?
□ User sudah konfirmasi mode?
```

**Sebelum tiap handoff antar fase, WAJIB cek:**
```
□ Output fase sebelumnya lolos validity check?
□ state.md sudah di-update dengan stage baru?
□ Active agent di-reset ke "none"?
□ Retry count tidak melebihi limit?
```

**Sebelum eksekusi kode (fase 6), WAJIB cek:**
```
□ Safety latch CHECK 1 passed? (stage == USER_APPROVAL_PENDING)
□ Safety latch CHECK 2 passed? (semua artifacts completed)
□ Safety latch CHECK 3 passed? (user approved)
□ Kalau ada ❌ → JANGAN NULIS KODE
```

---

## 10. Workflow Completion

Setelah EXECUTION_COMPLETE:

```
1. Update state.md: Stage = EXECUTION_COMPLETE
2. Update BLUEPRINT.md: Add task to completed tasks section, mark scope, link findings
3. Log hasil akhir di execution log
4. Archive folder:
   → Pindahkan semua file ke {{WORKFLOW_DIR}}/archive/
5. Report ke user:
   "Pipeline selesai — [nama fitur] telah diimplementasi.
    - Fase: [jumlah fase yang dijalani]
    - Retries: [total retry]
    - Errors: [jumlah error]"
```

## ✅ Final Checklist

```
□ Trigger detection implemented (FULL/QUICK/HOTFIX)
□ FSM states defined dengan transitions yang jelas
□ Safety latch di semua 3 check points
□ Retry mechanism (4 layers)
□ Validity check per fase
□ @Agent dispatch pattern dengan role skill reference
□ state.md template
□ 🚨 Self-Enforcement section
□ Workflow completion & archive
```
