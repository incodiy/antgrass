---
name: sdlc-gate-ux
description: Panduan dan checklist Gate G2 (UX & Design System). Menghasilkan Design System tokens (DESIGN-SYSTEM.md), hirarki komponen React/Next.js (COMPONENT-TREE.md), spesifikasi state management (STATE-SPECS.md), dan low-fidelity wireframes (WIREFRAMES.md).
---

# SDLC Gate G2 — UX & Design System

> **Tujuan:** Menerjemahkan kebutuhan PRD dan arsitektur data menjadi spesifikasi antarmuka visual yang terstruktur, responsif, dan siap dikoding di frontend/mobile.

---

## 1. Artefak Wajib Gate G2

Semua artefak disimpan di `[artifacts-root]/[project-slug]/G2-UX/`:
- `DESIGN-SYSTEM.md` — Tokens warna, tipografi, spacing, radius, breakpoints, dark mode, dan aksesibilitas.
- `COMPONENT-TREE.md` — Hirarki shared UI primitives, domain components, dan page trees.
- `STATE-SPECS.md` — Spesifikasi query keys React Query, local UI state, URL params, dan aturan anti-mock.
- `WIREFRAMES.md` — Low-fidelity wireframe untuk seluruh screen Must Have lengkap dengan layout, data source endpoint, dan state (loading/empty/error).

---

## 2. Prinsip Desain UX G2

1. **Aturan 4 State Universal:** Setiap layar wajib mendefinisikan layout untuk:
   - **Loading:** Skeleton yang meniru layout konten asli (bukan spinner layar kosong).
   - **Empty:** Pesan spesifik + ilustrasi + tombol CTA aksi.
   - **Error:** Pesan error jujur + tombol *Retry* (**Nol dummy/fallback data**).
   - **Optimistic UI:** Terbatas pada aksi ringan dengan mekanisme rollback jika API gagal.
2. **Tipografi Ramah Konten Khusus:** Untuk teks Arab/Quran, wajib menerapkan line-height longgar (contoh: 64px untuk ukuran 30px) agar harakat tidak bertumpuk.

---

## 3. Checklist Kelulusan Gate G2 (Exit Criteria)

- [ ] Design System tokens (warna, font, responsive breakpoints) selesai
- [ ] Hirarki Component Tree selaras dengan modul bounded context G1
- [ ] State management & React Query keys terpetakan ke endpoint `openapi.yaml`
- [ ] Wireframe seluruh layar Must Have selesai dengan spesifikasi data source
- [ ] **Persetujuan eksplisit (*Exit Approval*) dari pengguna diperoleh**
