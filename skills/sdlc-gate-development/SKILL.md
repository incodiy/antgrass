---
name: sdlc-gate-development
description: Panduan dan checklist Gate G3 (Implementation & Development). Mengatur pembuatan master plan BLUEPRINT.md, eksekusi bertahap Foundation-First, penerapan Test-Driven Development (TDD), penegakan Zero-Mock / Zero-Hardcode, dan validasi kontrak OpenAPI.
---

# SDLC Gate G3 — Development & Implementation

> **Tujuan:** Mengimplementasikan kode program secara terstruktur berdasarkan artefak G0, G1, dan G2, dengan urutan pengerjaan *Foundation-First* dan kepatuhan kontrak API.

---

## 1. Artefak Wajib Gate G3

- `BLUEPRINT.md` — Master plan pengembangan bertahap (Milestone M0 s/d MN) di root proyek.
- Kode Program Teruji pada masing-masing repositori.
- Test suite (Unit test, contract test, schema validator).

---

## 2. Aturan Eksekusi Pengerjaan (Engineering Discipline)

```
[Milestone M0: Setup & Blueprint]
          │
          ▼
[Milestone M1: Foundation API] ──▶ (Scaffold, Config Loader Fail-Fast, Migrasi DB 100%, Auth & Audit)
          │
          ▼
[Milestone M2: Foundation Front] ──▶ (UI Primitives, Tailwind Tokens, I18n, Auth Screens)
          │
          ▼
[Milestone M3+: Feature Modules] ──▶ (Pola per modul: Model -> Service -> Endpoint -> Test -> UI)
```

1. **Foundation-First:** Backend API dan migrasi database lengkap diselesaikan sebelum frontend agar frontend selalu mengonsumsi data nyata (**NFR-C2: Zero Mock**).
2. **Fail-Fast Configuration (NFR-C1):** Konfigurasi wajib dibaca dari `.env`. Jika variabel penting hilang, aplikasi wajib menolak start.
3. **Contract-Driven:** Implementasi endpoint wajib mematuhi schema pada `openapi.yaml`.

---

## 3. Checklist Kelulusan Gate G3 (Exit Criteria)

- [ ] Seluruh tabel database termigrasi sukses via migration tool (Alembic/Prisma)
- [ ] Seluruh endpoint API Must Have berfungsi dan teruji via automated test
- [ ] Frontend dan Mobile terintegrasi penuh ke API real tanpa mock/dummy data
- [ ] Pengujian grep `mockData|dummyResponse|fallbackData` menghasilkan **0 match**
- [ ] **Persetujuan eksplisit (*Exit Approval*) dari pengguna sebelum lanjut ke G4 (QA & Audit)**
