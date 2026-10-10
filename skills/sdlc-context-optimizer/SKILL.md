---
name: sdlc-context-optimizer
description: Context budget management, loading matrix, auto-recovery, archetype learning, change management archetypes, session continuity optimization. Loaded by sdlc when token budget is tight, context needs trimming, or session recovery is needed. Integrates with Work Orders and lean context system.
version: 1.4.0
updated: 2026-08-05
---

# SDLC — Context Optimizer

> Loaded by `sdlc` when token budget is tight, context needs trimming, session interrupted, or archetype learning required.

**Criticality: FLEKSIBEL** — skill pendukung untuk optimasi konteks. Panduan di bawah bersifat rekomendasi, bukan keharusan absolut. Gunakan sesuai kebutuhan situasi.

## When to Load

| Condition | Action |
|-----------|--------|
| Session >15-20 messages | Load this skill for budget check |
| Token limit approaching | Load this skill for prioritization |
| Session interrupted / resume | Load this skill for recovery |
| New project with similar type as previous | Load this skill for archetype loading |
| User says "this session is slow" | Load this skill |

---
## 🔄 Model & Budget Selection

**SETIAP KALIN** skill ini di-load, tanya user:

```
Pilih strategi budget:

[1] ⚡ Current LLM (recommended — no setup)
    → Pakai Big Pickle / default session LLM
    → Tidak perlu API key, tidak ada biaya tambahan
    → Detail cost strategy: Lihat skill `ai-cost-strategy`

[2] ✏️ External LLM (set sendiri — butuh .env)
    → Pilih tier: Small / Medium / Large
    → Saya pandu setup API key via .env (privacy terjaga)

[3] ⚡ Recommended Budget (context-aware auto)
    → Large tier untuk task kompleks, Medium untuk task ringan
    → Budget otomatis disesuaikan dengan task type
```

**Jika pilih External LLM atau Recommended:**

```
Tier model untuk session ini:
[a] 🏆 Large — best quality, higher cost (GPT-4 class)
[b] ⚖️ Medium — balanced, default (GPT-4o-mini class)
[c] 🚀 Small — fastest, cheapest (o3-mini class)

Token budget multiplier:
[a] 1x (standard) — ikut tabel budget di bawah
[b] 1.5x (generous) — untuk task kompleks
[c] 0.5x (tight) — untuk task sederhana

Fast mode (skip non-essential validation)?
[y] ✅ On — ~30-50% hemat token
[n] ❌ Off — full quality
```

---

## Context Loading Matrix

Prioritize what to load based on current gate:

```markdown
|| G0 PRD | G1 Arch | G2 UX | G3 Dev | G4 Deploy | G5 Maint |
|---|---|---|---|---|---|---|
PRD.md | ✅ Full | ✅ Summary | ✅ Summary | ✅ Ref | ❌ | ❌ |
ERD | ❌ | ✅ Full | ✅ Ref | ✅ Ref | ❌ | ❌ |
OpenAPI | ❌ | ✅ Full | ✅ Ref | ✅ Full | ✅ Ref | ✅ Ref |
Component Tree | ❌ | ❌ | ✅ Full | ✅ Ref | ❌ | ❌ |
STACK.md | ❌ | ✅ Full | ✅ Full | ✅ Full | ✅ Ref | ✅ Ref |
decisions.md | ✅ Ref | ✅ Ref | ✅ Ref | ✅ Ref | ✅ Full | ✅ Full |
OVERVIEW.json | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ✅ Full |
.env / secrets | ❌ | ❌ | ❌ | ❌ | ✅ Ref | ✅ Ref |
```

- **Full** = entire file
- **Summary** = 5-10 line summary (generate from file)
- **Ref** = minimal reference (path + purpose only)
- **❌** = skip entirely

## Token Budget by Gate

Budget default (sebelum model selection):

| Gate | Default Budget | With Large Tier | With Medium Tier | With Small Tier |
|------|--------------|-----------------|------------------|-----------------|
| G0 PRD | 4K tokens | 4K | 2K | 1K |
| G1 Architecture | 6K tokens | 8K | 4K | 2K |
| G2 UX | 4K tokens | 4K | 2K | 1K |
| G3 Dev | 10K tokens | 12K | 6K | 3K |
| G4 Deploy | 3K tokens | 4K | 2K | 1K |
| G5 Maint | 2K tokens | 2K | 1K | 0.5K |

**Catatan:** Budget naik untuk Large tier karena model kelas ini butuh lebih banyak context buat reasoning optimal. Turun untuk Small tier karena task-nya sederhana.

**Integrasi:** Lihat skill `ai-cost-strategy` untuk detail model tier selection, effort levels, fast mode, dan budget strategy.

When budget exceeded → summarize oldest context, keep most recent.

## Auto-Recovery Protocol

### Session Interrupted

```
1. Detect: No previous messages in new session
2. Read: .opencode/phase_state/[project]/OVERVIEW.json
3. Read: .opencode/session_chats/current.json (field development + last_checkpoint)
4. Inject: "Sesi sebelumnya di [GATE], sedang ngerjain [TASK].
   Status: [in_progress/completed/blocked]. Blockers: [list]."
5. Offer: [1] ✅ Lanjut / [2] 🔄 Restart gate / [3] 📋 Lihat handoff
```

### Token Limit Reached Mid-Task

```
1. Generate checkpoint via session-history
2. Write handoff.md:
   - Current task (2 lines max)
   - Key context (file paths, decisions, blockers)
   - Next action (1 sentence)
3. Update OVERVIEW.json:
   - Set phase state = "in_progress" + handoff summary
4. Suggest: "Sesi penuh. Simpan handoff di docs/handoff.md.
   Load ulang dengan skill sdlc-context-optimizer"
```

### Gate Rollback

```json
{
  "rollback": {
    "from_gate": "G3",
    "to_gate": "G2",
    "reason": "Komponen UI berubah signifikan, wireframe perlu revise",
    "preserved_artifacts": ["OpenAPI spec", "ERD"],
    "reverted_artifacts": ["Component tree di G2"],
    "date": "2026-06-14"
  }
}
```

Steps:
1. Update `OVERVIEW.json` → revert gate
2. Preserve artifacts from later gate that are valid
3. Revert artifacts that are invalid
4. Log in `decisions.md`

## Archetype Learning

Cache learnings per project type for future projects:

```
.opencode/archetypes/
├── e-commerce.json       ← Learnings from e-commerce projects
├── microservices.json    ← Learnings from microservices projects
├── mobile-app.json       ← Learnings from mobile app projects
└── dashboard.json        ← Learnings from dashboard projects
```

### Archetype Schema

```json
{
  "project_type": "e-commerce",
  "patterns": {
    "common_pitfalls": [
      "Meletakkan payment logic di controller → harusnya di service",
      "Lupa stock management di checkout flow"
    ],
    "architecture_patterns": [
      "Cart → Service pattern dengan Redis cache"
    ],
    "tech_stack": {
      "preferred": ["Laravel", "PostgreSQL", "Redis", "Meilisearch"],
      "avoid": ["MySQL (JSON field untuk product attribute)"]
    },
    "compliance_lessons": [
      "PCI-DSS: card number jangan pernah di-log"
    ]
  },
  "last_updated": "2026-06-14"
}
```

### Loading Archetype

When starting new project of known type:

```
📚 Archetype ditemukan: e-commerce (dari project "Toko Online")
Common mistakes to avoid:
  - Payment logic di controller
  - Lupa stock management
Load archetype? [1] ✅ Yes / [2] Skip
```

### Learning Trigger

Auto-save archetype when:
- Project reached G4 (deploy) or completed
- Significant mistake discovered mid-project
- User explicitly says "catat ini untuk project berikutnya"

## Change Management Context Archetypes

### Change Pattern Archetype

Archetype baru untuk tracking pola discovery/change:

```json
{
  "project_type": "e-commerce",
  "change_patterns": [
    {
      "pattern_id": "CP-001",
      "description": "Payment gateway discovery mid-G3",
      "phase": "G3 — Backend API",
      "trigger": "T1 — Technical discovery",
      "finding": "Payment gateway butuh 3rd party integration yang tidak diantisipasi di ERD",
      "handling": "PRD Amendment (T2), +3 tasks di BLUEPRINT",
      "result": "Timeline +5 hari, sukses deployed",
      "lesson": "Include payment gateway feasibility di G1 assessment"
    }
  ],
  "last_updated": "2026-07-02"
}
```

### Context Loading During Change

Ketika ada discovery aktif (detect dari `current.json` → `active_discoveries`):

| Context | Load Priority | Rationale |
|---------|---------------|-----------|
| PRD.md (relevan section) | HIGH | Cek scope — apakah ini T1/T2/T3? |
| PRD-CHANGELOG.md | HIGH | Cek amendment history untuk konteks |
| decisions.md (recent 5) | HIGH | Cek keputusan terkait |
| BLUEPRINT.md current phase | MEDIUM | Task yang terpengaruh |
| issues.md | MEDIUM | Bug/gap terkait |
| ERD.md / openapi.yaml | LOW | Hanya jika T1 arsitektur |

### Token Budget During Change

Saat `active_discoveries` tidak kosong:
- Temporary budget increase: **+25%** dari normal
- Extra allocation untuk: PRD diff, BLUEPRINT updates, pre/post checkpoints
- Auto-reset ke normal setelah discovery selesai (status "resolved")

### Queryable Archetype — Reuse Decision

Untuk reuse pola discovery dari project sebelumnya:

```
📚 Change pattern match: Project "Toko Online" (e-commerce)
  → Fase G3: Payment gateway discovery (T1) 
  → Handling: PRD Amendment + +3 tasks
  → Result: +5 hari timeline, sukses
  → Lesson: Include payment gateway feasibility di G1
  
[1] ✅ Apply pattern ini
[2] Skip — context berbeda
[3] Lihat detail
```



## Context Trimming Rules

| Can Trim | Keep Always |
|----------|-------------|
| Old conversation turns (>10 messages back) | OVERVIEW.json |
| Full file content (replace with summary) | decisions.md (recent 5 entries) |
| Specific code snippets not currently being edited | Module tracking state |
| Archived archetypes | Active project registry |
| Error messages from resolved issues | SLA targets and thresholds |

## Work Orders, Document Sync & Lean Context

> **Canonical definitions:** `sdlc` SKILL.md §4.6 + `waypoint` SKILL.md §5, §7, §15. Short summary below for standalone use.

### WO Classification (Context Optimizer)

| WO Type | When to Use | Required Docs |
|---------|-------------|---------------|
| ENHANCEMENT | New feature requiring context optimization | Loading matrix, BLUEPRINT, decisions |
| REFACTOR | Context refactoring | Trimming rules, BLUEPRINT, decisions |
| AUDIT | Context audit finding | BLUEPRINT, decisions |

### Lean Context

> Read: OVERVIEW.json + ACTIVE_SPRINT.md on session init. Write: OVERVIEW.json + ACTIVE_SPRINT.md + archive WO on session exit. Full rules: `waypoint` §15.

## 🚨 Self-Enforcement — Context Optimizer

Sebaiknya pertimbangkan hal berikut saat menggunakan skill ini:

```
□ Apakah token budget benar-benar perlu dioptimasi? (cek jumlah pesan)
□ Context loading matrix sudah sesuai gate saat ini?
□ Jika session interrupted — OVERVIEW.json sudah dibaca?
□ Jika token limit — checkpoint sudah dibuat?
□ Archetype learning — ada pattern yang worth disimpan?
□ Trimming rules dipatuhi? (keep OVERVIEW.json, decisions.md recent)
□ Work Order di-classify jika ada context change?
□ Document Sync Rules diaplikasikan sesuai WO type?
```

## ✅ Final Checklist — Context Optimizer

```
□ Criticality assessment: FLEKSIBEL
□ Tone rekomendasi, bukan instruksi WAJIB
□ Frontmatter hanya name + description
□ Anti-pattern #4 dipatuhi
□ Work Orders, Document Sync & Lean Context summary ada
```

### Summary Format

When summarizing a document:

```
📄 [Filename] (summarized)
↳ Purpose: [1 line]
↳ Key decisions: [2-3 bullet points]
↳ Current relevance: [How it connects to current task]
↳ Full path: [file path for reference]
```
