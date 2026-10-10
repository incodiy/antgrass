---
name: sdlc-gate-architecture
description: Panduan dan checklist Gate G1 (Architecture & System Design). Menghasilkan ERD E2E lengkap, kontrak OpenAPI (openapi.yaml), diagram sistem C4 level 1-2 (SYSTEM.md), tech stack final (STACK.md), dan repository strategy.
---

# SDLC Gate G1 — Architecture & System Design

> **Tujuan:** Merancang fondasi teknis, skema database lengkap (tanpa schema churn), batas layanan (*service boundaries*), dan kontrak API yang otoritatif.

---

## 1. Artefak Wajib Gate G1

Semua artefak disimpan di `[artifacts-root]/[project-slug]/G1-ARCH/`:
- `ERD.md` — Desain Entity Relationship lengkap untuk semua phase (D-09)
- `SYSTEM.md` — Diagram arsitektur sistem C4 Level 1 & Level 2
- `STACK.md` — Keputusan final tech stack backend, frontend, mobile, DB, cache
- `openapi.yaml` — Spesifikasi kontrak REST API OpenAPI 3.1
- `repository-strategy.md` — Strategi multi-repo vs monorepo dan mapping folder

---

## 2. Prinsip Desain Arsitektur G1

1. **ERD E2E Lengkap sejak Hari Pertama (D-09):** Seluruh entitas untuk semua fase dirancang sejak awal untuk menghindari perubahan skema destruktif di tengah jalan. Gunakan UUID v7 sebagai PK, snake_case, soft delete untuk entitas pengguna, dan index berbasis query pattern.
2. **Contract-First (OpenAPI):** `openapi.yaml` adalah satu-satunya acuan data bagi seluruh client. Respon wajib mengikuti format envelope standar.
3. **Module Boundaries:** Batasi domain model agar siap ditransisikan ke microservices jika diperlukan.

---

## 3. Checklist Kelulusan Gate G1 (Exit Criteria)

- [ ] Seluruh entitas database teridentifikasi dengan PK, FK, dan Indexes di `ERD.md`
- [ ] `openapi.yaml` mencakup seluruh endpoint Must Have dan Should Have
- [ ] Diagram sistem C4 level 1 & 2 terdefinisi di `SYSTEM.md`
- [ ] Tech stack final terkunci di `STACK.md`
- [ ] Strategi repositori dan alur data terdokumentasi di `repository-strategy.md`
- [ ] Keputusan arsitektur dicatat di `decisions.md`
- [ ] **Persetujuan eksplisit (*Exit Approval*) dari pengguna diperoleh**
