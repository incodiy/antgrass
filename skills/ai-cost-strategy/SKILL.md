---
name: ai-cost-strategy
description: Strategi milih model AI, effort level, dan mode eksekusi buat balance quality-speed-cost. Provider-agnostic — bisa pake current LLM atau external. Interactive mode saat milih tier.
version: 1.0.0
updated: 2026-06-21
---

# AI Cost Strategy

> Panduan milih tier model, effort level, dan mode eksekusi. Provider-agnostic — nggak terikat ke API key tertentu. Berguna bareng `sdlc-context-optimizer`.

---

## 🔄 INTERAKTIF — Pilih Model Tier

```
Pilih model untuk session ini:

[1] ⚡ Gunakan LLM saat ini (Current LLM — no setup)
    → Big Pickle / default session LLM
    → Tidak perlu API key, tidak ada biaya tambahan
    → Cocok buat daily development, general tasks

[2] ✏️ External LLM (set sendiri)
    → Saya akan pandu setup .env file
    → Pilih tier: Small / Medium / Large
    → API key disimpan di .env (privacy terjaga)
```

### Recommended Defaults by Task Type

| Task | Recommended Tier | Effort | Fast Mode | Rationale |
|------|-----------------|--------|-----------|-----------|
| Coding / Feature | Large | Auto | Off | Best quality, worth it |
| Bug fix | Medium | Auto | On | Fast turnaround |
| Refactor | Large | High | Off | Needs deep reasoning |
| Test generation | Medium | Low | On | Bulk operation |
| Code review | Large | High | Off | Thorough analysis needed |
| Documentation | Medium | Low | On | Simple generation |
| Exploratory | Medium | Auto | On | Iterative, many attempts |
| CI/CD pipeline | Medium | Low | On | Automated, non-interactive |

---

## Detail Custom Mode

### Pilih Tier Model

```
[a] 🏆 Large Model (best quality)
    → Complex reasoning, architecture, code review
    → Relatif lebih lambat & mahal (kalo external)
    → Contoh: GPT-4 class, Claude 3 Opus, Llama 405B

[b] ⚖️ Medium Model (balanced)
    → Daily dev, refactor, test generation
    → Recommended default untuk most tasks
    → Contoh: GPT-4o-mini, Claude 3 Haiku, Llama 70B

[c] 🚀 Small Model (fastest, cheapest)
    → Simple tasks, linting, formatting, bulk operations
    → Cocok untuk automation, draft, prototyping
    → Contoh: o3-mini, Gemini Flash, Phi-3
```

### Pilih Effort Level

```
[a] Auto — model decides effort (default)
    → Best for most tasks

[b] Low — faster execution, less thorough
    → Cocok: simple edits, copilot-style suggestions
    → Gak cocok: complex refactoring, security audit

[c] High — more reasoning, better quality
    → Cocok: architecture decisions, code review, bug hunting
    → Gak cocok: quick fixes, simple generation
```

### Pilih Fast Mode

Fast mode = skip non-essential checks (validation, linting, type checking) untuk akselerasi.

```
[y] ✅ On — skip non-essential validation
    → Cocok: early prototyping, draft generation
    → Hemat: ~30-50% token usage
    → Risiko: output mungkin perlu manual review

[n] ❌ Off — full pipeline (recommended untuk production)
    → Cocok: final code, PR submission, deployment
    → Kualitas lebih terjamin
```

### Setup External LLM

Jika pilih external LLM, saya akan:

```
1. Buat file .env di root project
2. Kasih tau env var yang perlu diisi, contoh:

   LLM_API_KEY=your-key-here
   LLM_MODEL=gpt-4o
   LLM_BASE_URL=https://api.openai.com/v1

3. Tanya tier yang dipilih (Small/Medium/Large)
4. API key nggak pernah ditampilkan di chat
   → User isi sendiri di file .env pake text editor
```

---

## 2. Budget Strategy

### Relative Cost Comparison

| Tier | Relative Cost | Use When |
|------|--------------|----------|
| Small | 💵 1x | Simple, bulk, draft |
| Medium | 💵💵 5-10x | Daily dev, most tasks |
| Large | 💵💵💵 20-50x | Complex reasoning, production code |

> **Catatan:** Biaya aktual tergantung provider. Current LLM = $0 (no additional cost).

### Budget Optimization Tips

```
💰 Tips hemat token & biaya:

1. Pake Small/Medium untuk task sederhana
   → Simpan Large untuk complex reasoning aja

2. Aktifin fast mode pas early prototyping
   → Matiin pas final code / PR

3. /compact secara berkala — kompres context window

4. Session management:
   → Fork session kalo context udah penuh
   → /clear kalo bahas topik baru

5. Cache-aware prompting:
   → Input yang sama berulang kali di-cache
   → Kirim context yang reusable akan lebih efisien
```

---

## 3. Integration dengan sdlc-context-optimizer

`Ai-cost-strategy` — model tier selection + effort level + fast mode
→ Bantu milih tier biar budget kepake optimal

```
sdlc-context-optimizer:
    Token budget per gate + trimming rules

        ↕ referensi

ai-cost-strategy:
    Model tier selection + effort level + fast mode
```

**Contoh:** G3 Development budget 10K tokens. Pilih tier:
- Simple CRUD feature → Medium + low effort + fast mode (2-3K tokens)
- Complex domain logic → Large + high effort (8-10K tokens)

---

## 🚨 Self-Enforcement

**Sebelum milih tier/effort mode di session baru, WAJIB pertimbangkan:**

```
□ Task complexity → complex = Large + high effort
□ Token budget → sisa budget cukup? kalau gak → turunin tier/effort
□ Urgency → need speed? → Small + fast mode
□ Production vs exploration → production = jangan fast mode
□ External LLM → sudah setup .env? API key valid?
```
