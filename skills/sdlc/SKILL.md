---
name: sdlc
description: Master orchestrator untuk Enterprise SDLC berbasis Gate (G0 s/d G5). Mengatur alur PRD -> Architecture -> UX -> Development -> QA/Deploy -> Maintenance, checklist kelulusan gate, decision locking, dan sinkronisasi state proyek. Panggil skill ini saat memulai inisialisasi sesi SDLC, transisi gate, atau audit kepatuhan workflow.
---

# SDLC — Master Enterprise Development Orchestrator

> **Fungsi:** Mengatur siklus gerbang pengembangan (Gate G0–G5), memastikan tidak ada tahapan arsitektur atau desain yang terlewat sebelum koding dimulai, dan menjaga konsistensi state proyek.

---

## 1. Peta Gerbang SDLC (Gate Roadmap)

```
[G0 — PRD] ──▶ [G1 — Architecture] ──▶ [G2 — UX & Wireframe]
                                               │
                                               ▼
[G5 — Release] ◀── [G4 — QA & Audit] ◀── [G3 — Development]
       │
       ▼
[G3-MNT — Maintenance & Enhancement]
```

| Gate | Nama Gerbang | Fokus Utama | Artefak Wajib |
|---|---|---|---|
| **G0** | **Product Requirements** | Problem statement, user personas, roadmap, MoSCoW, NFRs | `PRD.md`, `PRD-CHANGELOG.md` |
| **G1** | **Architecture & Design** | ERD E2E, OpenAPI contract, C4 system diagram, tech stack | `ERD.md`, `SYSTEM.md`, `STACK.md`, `openapi.yaml`, `repository-strategy.md` |
| **G2** | **UX & Design System** | Visual tokens, component tree, state specs, wireframes | `DESIGN-SYSTEM.md`, `COMPONENT-TREE.md`, `STATE-SPECS.md`, `WIREFRAMES.md` |
| **G3** | **Implementation & TDD** | Foundation API, UI kit, modular development, unit tests | `BLUEPRINT.md`, Kode Sumber, Test Suites |
| **G4** | **QA & System Audit** | E2E integration test, load test, security audit, UU PDP audit | Laporan audit & test report |
| **G5** | **Production & Deploy** | CI/CD pipeline, VPS/Docker setup, domain & SSL, monitoring | Release notes, deployment log |

---

## 2. Inisialisasi Sesi & Pemeriksaan State (Session Init)

Setiap kali sesi baru dimulai atau skill `@sdlc` dipanggil, jalankan langkah berikut:

### Langkah 1: Cek State Proyek Aktif
Baca file state lean pada project root (baik di folder `.opencode/` maupun `.agents/`):
1. Periksa file `OVERVIEW.json` untuk mengetahui Gate dan Task yang sedang aktif.
2. Periksa file `decisions.md` untuk melihat keputusan yang sudah terkunci (*locked decisions*).

### Langkah 2: Tampilkan Banner Status Sesi
Tampilkan ringkasan status dalam format banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 SDLC Session Status — [Nama Proyek]
  Gate Aktif: [G0 / G1 / G2 / G3 / G4 / G5]
  Status Gate: [in_progress / approved]
  Task Saat Ini: [Nama Task / Milestone]
  State: OVERVIEW.json ✅ | decisions.md ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Aturan Transisi Gate (Gate Transition Rules)

1. **Kelulusan 100% Checklist:** Dilarang berpindah ke gate berikutnya sebelum seluruh artefak wajib di gate saat ini lengkap dan lolos audit.
2. **Persetujuan Pengguna (User Approval):** Sebelum gate dinyatakan selesai, mintakan konfirmasi eksplisit dari pengguna.
3. **Pencatatan Audit Trail:** Catat transisi gate di `decisions.md` dan perbarui `OVERVIEW.json`.

---

## 4. Sub-Skill Per-Gate (Chaining via On-Demand Reading)

Ketika memasuki gate tertentu, gunakan panduan detail dari sub-skill terkait:
- Gate G0: Baca `skills/sdlc-gate-prd/SKILL.md`
- Gate G1: Baca `skills/sdlc-gate-architecture/SKILL.md`
- Gate G2: Baca `skills/sdlc-gate-ux/SKILL.md`
- Gate G3: Baca `skills/sdlc-gate-development/SKILL.md`
- Gate G5: Baca `skills/sdlc-gate-deployment/SKILL.md`
- Maintenance: Baca `skills/sdlc-gate-maintenance/SKILL.md`
