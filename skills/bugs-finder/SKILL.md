---
name: bugs-finder
description: You are a deep bug-finding automation focused on high-severity issues.
version: 1.0.0
updated: 2026-06-21
---

# Bugs Finder

## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI

**Skill ini untuk inspeksi kode secara mendalam dan melaporkan bug severity tinggi saja.**

🔴 **Konsekuensi jika dilanggar:**
- Bug kritis terlewat → data loss, security breach di production
- False positive dikirim ke user → AI kehilangan kredibilitas
- Fix terlalu agresif (refactor + bug fix sekaligus) → introduces new bugs

**Model WAJIB** mengikuti setiap aturan di bawah dengan disiplin.

---

## Goal

Inspect recent commits and identify critical correctness bugs that escaped review. Hanya surface issues yang menyebabkan data loss, crashes, security holes, atau significant user-facing breakage.

## Investigation strategy

**WAJIB** load skill `analysis-methodology` sebelum mulai investigate. Skill ini menyediakan:
- Confidence Bar untuk menentukan apakah boleh propose fix
- Root Cause Analysis Framework (Symptom → Evidence → Hypothesis → Verification → Fix)
- Self-enforcement checklist

**JANGAN** propose fix sebelum lolos Confidence Bar.

**WAJIB:**
- Focus on behavioral changes with meaningful blast radius
- Trace through the full code path — jangan pattern-match pada diff doang. Pahami caller chain dan downstream effects
- Look for: data corruption, race conditions yang bisa lose writes, null dereferences di critical paths, auth/permission bypasses, infinite loops, resource leaks, dan silent data truncation

**JANGAN:**
- Report style issues, minor edge cases, theoretical concerns tanpa concrete trigger, atau low-severity issues yang cuma degrade UX

## Confidence bar

**WAJIB** bisa mendeskripsikan skenario konkret yang memicu bug.

| Kondisi | Tindakan |
|---------|----------|
| Bisa deskripsikan trigger scenario + yakin bug real | ✅ Buka PR |
| Bisa deskripsikan trigger scenario tapi ragu | 📢 Report di Slack, JANGAN buka PR |
| Tidak bisa konstruksi trigger scenario | ❌ Tutup — jangan report apapun |

## Fix strategy

- **WAJIB** implementasi minimal, high-confidence fix — JANGAN sekalian refactor
- **WAJIB** tambah/update tests untuk lock in behavior
- **JANGAN** gabung multiple bug fix dalam 1 PR

## Safety rules

- **JANGAN** buka PR unless highly confident bug is real dan fix-nya correct
- Jika tidak ada critical bug found: **WAJIB** post "no critical bugs found" summary. Ini expected outcome most days

## Output

Jika fix dibuat, WAJIB include:
- **Bug dan impact** — apa yang terjadi dan siapa yang terdampak
- **Root cause** — kenapa ini bisa terjadi
- **Fix dan validation** — apa yang diubah dan gimana nge-verify

---

## 🚨 Self-Enforcement

**Sebelum report atau buka PR, WAJIB cek:**

1. **Trigger scenario jelas?** — bisa deskripsikan langkah demi langkah yang memicu bug?
2. **Severity cukup tinggi?** — data loss, crash, security hole, atau major breakage?
3. **Fix minimal?** — gak ada refactor atau perubahan lain di luar bug?
4. **Test added?** — minimal ada test untuk scenario yang di-fix?
5. **False positive risk?** — ada kemungkinan ini expected behavior? Kalau ragu, tanya dulu.

**Kalau ragu → tanya user dulu. JANGAN asumsi.**
