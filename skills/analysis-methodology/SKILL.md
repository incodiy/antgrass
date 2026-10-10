---
name: analysis-methodology
description: Use when investigating bugs, performance issues, architecture decisions, or any "why is this happening?" question. Enforces structured root cause analysis with Confidence Bar, mandatory code path tracing, hypothesis listing, and verification before proposing fixes. Prevents premature conclusions and shallow analysis.
version: 1.0.0
updated: 2026-07-16
---

# Analysis Methodology

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI

**Skill ini adalah framework wajib untuk semua tipe analisa.** Shallow analysis → wrong fix → new bugs → trust erosion.

🔴 **Konsekuensi jika dilanggar:**
- Premature conclusion → fix yang tidak perlu, root cause tidak teratasi
- Pattern-matching tanpa trace → salah identifikasi masalah
- Hypothesis present sebagai fakta → user trust hilang
- Fix tanpa verifikasi → introduce new bugs, regression

**Model WAJIB** mengikuti Confidence Bar dan Root Cause Framework SEBELUM propose fix apapun.

---

## When to Load

| Condition | Action |
|-----------|--------|
| Bug investigation | Load immediately |
| Performance analysis | Load immediately |
| Architecture decision | Load immediately |
| "Why is this happening?" | Load immediately |
| User report masalah | Load immediately |
| Sebelum propose fix apapun | Load immediately |

---

## Confidence Bar

**WAJIB** lolos Confidence Bar SEBELUM propose fix:

| Kondisi | Tindakan |
|---------|----------|
| Bisa trace code path + yakin root cause + bisa describe trigger scenario | ✅ Propose fix |
| Bisa trace tapi ragu / tidak bisa describe trigger lengkap | 📢 Report hypothesis, JANGAN fix |
| Tidak bisa trace / tidak ada bukti cukup | ❌ STOP — minta data lebih dari user |

**JANGAN** present hypothesis sebagai fakta. Kalau ragu, bilang:
- "Saya tidak yakin ini root cause-nya"
- "Ada beberapa kemungkinan, perlu verifikasi lebih lanjut"
- "Bisa describe langkah-langkah yang menghasilkan masalah?"

---

## Root Cause Analysis Framework

### Phase 1: SYMPTOM — Apa yang terjadi?

Deskripsikan masalah secara objektif:
- Apa yang user lihat / alami?
- Apa error message / log output?
- Kapan masalah terjadi? (konsisten / intermittent)

**JANGAN** langsung tebak kenapa. Fokus ke "apa" dulu.

### Phase 2: EVIDENCE — Bukti apa yang ada?

Kumpulkan semua bukti:
- Error logs, stack traces
- Timing logs, performance metrics
- Code changes (diff, commits)
- User behavior / reproduction steps
- Browser dev tools, network tab

**WAJIB** baca kode yang terlibat — jangan cuma baca error log atau diff.

### Phase 3: HYPOTHESIS — List semua kemungkinan

Buat minimal **2 hypothesis** sebelum pilih satu:

```
Hypothesis A: [deskripsi]
  Evidence yang support: [...]
  Evidence yang contra: [...]
  Confidence: [tinggi/sedang/rendah]

Hypothesis B: [deskripsi]
  Evidence yang support: [...]
  Evidence yang contra: [...]
  Confidence: [tinggi/sedang/rendah]
```

**JANGAN** pick satu hypothesis langsung. List dulu, baru verifikasi.

### Phase 4: VERIFICATION — Trace code path

Untuk setiap hypothesis, trace execution path:

```
1. Mulai dari symptom → ikuti alur kode
2. Identify state dependencies (untuk React: deps array, useEffect, state flow)
3. Cari bukti yang support atau contra hypothesis
4. Document: "Jika hypothesis X benar, maka seharusnya [expected behavior]"
5. Cek: apakah expected behavior terjadi di reality?
```

**WAJIB** baca kode aktual — jangan pattern-match pada diff atau log saja.

**Contoh trace untuk React:**
```
Component mount → state awal → useEffect deps → state update → re-render → useEffect deps lagi?
```

### Phase 5: CONCLUSION — Pilih hypothesis yang terverifikasi

Setelah verifikasi:
- Pilih hypothesis dengan evidence terkuat
- Jelaskan kenapa hypothesis lain ditolak
- Confidence level: yakin / cukup yakin / ragu

**Kalau ragu → kembali ke Confidence Bar. JANGAN propose fix.**

### Phase 6: FIX — Minimal, high-confidence

Jika lolos Confidence Bar:
- **Satu fix per PR** — jangan gabung multiple fixes
- **Minimal change** — jangan refactor + fix sekaligus
- **Test wajib ada** — minimal test untuk scenario yang di-fix
- **Verify fix** — reproduksi masalah → apply fix → verify hilang

---

## Mandatory Steps Before Any Fix

```
1. Baca kode yang terlibat (bukan cuma diff/log)
2. Trace state dependencies (React: deps, useEffect, re-render chain)
3. List minimal 2 hypothesis
4. Verify setiap hypothesis dengan code path trace
5. Bisa describe trigger scenario yang konkret
6. Kalau ragu → tanya user, JANGAN assume
```

---

## Self-Enforcement

**Sebelum propose fix apapun, WAJIB cek:**

```
□ Trace code path sudah dilakukan? (bukan cuma baca error log)
□ Minimal 2 hypothesis terdaftar?
□ Trigger scenario bisa dideskripsikan step-by-step?
□ Fix minimal? (tidak refactor + fix sekaligus)
□ False positive risk dicek? (ada kemungkinan ini expected behavior?)
□ Kalau ragu → sudah tanya user?
```

**Kalau ada ❌ → JANGAN propose fix. Selesaikan dulu.**

---

## Common Anti-Patterns

| Anti-Pattern | Contoh | Yang Benar |
|--------------|--------|------------|
| **Premature conclusion** | "Timing log ada X, pasti masalah di X" | Trace code path dulu, list hypothesis |
| **Pattern-matching** | "Error ini mirip sama yang dulu" | Baca kode yang terlibat, jangan asumsi sama |
| **Single hypothesis** | "Ini pasti karena X" | List 2+ hypothesis, verifikasi semua |
| **Assume without verify** | "Pasti X, langsung fix" | Verify dulu, baru fix |
| **Reactor + fix** | "Sekalian refactor ya" | Fix minimal dulu, refactor terpisah |

---

## Integration with Other Skills

| Skill | Relationship |
|-------|-------------|
| `bugs-finder` | Confidence Bar dari skill ini dipakai sebagai gate sebelum fix |
| `feature-workflow` | Verification framework complement validity check di pipeline |
| `system-audit` | Audit findings harus lolos Confidence Bar sebelum mark sebagai fixed |
| `sdlc-gate-development` | Pre-Code Gate termasuk analysis verification |
| `sdlc-gate-deployment` | Audit findings remediasi harus pakai framework ini |

---

## Example: Infinite Refresh Loop Investigation

**SYMPTOM:** Halaman refresh berulang setiap load

**EVIDENCE:**
- 60+ GET / requests di network tab
- Timing log menunjukkan `proxy.ts` berjalan
- Terjadi di semua browser

**HYPOTHESIS:**
1. Middleware naming salah (proxy.ts vs middleware.ts)
2. React state cycle di ThemeProvider
3. Cache strategy salah (`cache: 'no-store'`)

**VERIFICATION:**
- Hypothesis 1: Middleware tetap jalan meskipun nama `proxy.ts` (karena next-intl handle registration) → CONTRA
- Hypothesis 2: `hasSsrData = theme !== null` dihitung ulang setiap render → flip trigger useEffect lagi → SUPPORTED
- Hypothesis 3: `cache: 'no-store'` menyebabkan 7 fetch per load → SUPPORTED tapi bukan loop cause

**CONCLUSION:** Hypothesis 2 adalah root cause utama. Hypothesis 3 adalah performance issue, bukan loop cause.

**FIX:** Stabilkan `hasSsrData` dengan `useRef` atau `useState` freeze.

---

## Final Checklist

```
□ Confidence Bar lolos?
□ Root Cause Framework 6 phase dijalankan?
□ Minimal 2 hypothesis terdaftar?
□ Code path trace dilakukan (bukan pattern-match)?
□ Trigger scenario bisa dideskripsikan?
□ Fix minimal (satu fix per PR, no refactor)?
□ Test ada untuk scenario yang di-fix?
□ Kalau ragu → tanya user?
```

**Kalau ada yang terlewat → koreksi SEBELUM lanjut ke fix.**
