---
name: sdlc-gate-prd
description: Panduan dan checklist Gate G0 (Product Requirements Definition). Menghasilkan PRD.md dengan problem statement, personas, roadmap, MoSCoW, NFRs, engineering constraints, compliance, dan cost ROM.
---

# SDLC Gate G0 — Product Requirements Definition

> **Tujuan:** Mendefinisikan kebutuhan produk (WHAT) secara komprehensif, terstruktur, dan disepakati sebelum masuk ke perancangan arsitektur.

---

## 1. Artefak Wajib Gate G0

Semua artefak disimpan di `.opencode/artifacts/[project-slug]/G0-PRD/` atau `.agents/artifacts/[project-slug]/G0-PRD/`:
- `PRD.md` — Dokumen PRD utama (versi hidup)
- `PRD-v1.0.md` — Snapshot baseline immutable saat pertama kali disetujui
- `PRD-CHANGELOG.md` — Catatan perubahan/amendment PRD

---

## 2. Struktur Standar `PRD.md`

1. **Problem Statement & Opportunity:** Current state, pain points, target metrics.
2. **Target Audience & User Personas:** Persona detail dengan goals dan pain points.
3. **Vision & Business Goals:** Sasaran strategis proyek.
4. **Roadmap E2E:** Pemetaan phase pengembangan (Phase 1 s/d Phase N).
5. **Functional Requirements (FR):** Format ID `PRD-F-[NN]` dengan prioritas MoSCoW per phase.
6. **Non-Functional Requirements (NFR):** Zero hardcode, zero mock, fail-fast, security, database encoding.
7. **Feasibility & Compliance:** Evaluasi teknis, UU PDP, GDPR, kebijakan app store.
8. **Cost ROM & Dependencies:** Estimasi man-month dan sumber data eksternal.

---

## 3. Checklist Kelulusan Gate G0 (Exit Criteria)

- [ ] Problem statement & user personas terdefinisi jelas
- [ ] Seluruh fitur E2E memiliki ID `PRD-F-[NN]` dengan prioritas MoSCoW
- [ ] Non-Functional Requirements & Engineering Constraints terkunci
- [ ] Analisa kepatuhan regulasi (UU PDP/GDPR/Store Policies) tercakup
- [ ] `PRD.md` dan `PRD-CHANGELOG.md` tersimpan di folder artefak G0
- [ ] **Persetujuan eksplisit (*Exit Approval*) dari pengguna diperoleh**
