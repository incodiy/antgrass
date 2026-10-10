---
name: waypoint
description: Lean Cross-Session Context Manager. Mengatur kesinambungan status proyek lintas sesi percakapan dengan konsumsi token minimal (~90% token reduction). Menggunakan hierarki OVERVIEW.json (~10 baris) -> ACTIVE_SPRINT.md -> archive/. Mendukung klasifikasi Work Order (BUG, GAP, AUDIT, ENHANCEMENT, REFACTOR) dan aturan sinkronisasi dokumen.
---

# Waypoint — Lean Cross-Session Context Manager

> **Tujuan:** Menjaga kesinambungan pengembangan lintas sesi chat dengan konsumsi token minimal dan audit trail yang rapi.

---

## 1. Arsitektur State Lean (Token-Efficient Context)

```
OVERVIEW.json          (~10 baris)    ◄── Dibaca PERTAMA KALI (cepat & hemat)
     │
     ├── ACTIVE_SPRINT.md (~100 baris) ◄── Dokumen kerja sprint yang sedang aktif
     │
     ├── decisions.md      (~50 baris)  ◄── Keputusan arsitektural terkunci
     │
     └── archive/                       ◄── Dibaca HANYA saat dibutuhkan (on-demand)
         ├── sprints/      (riwayat sprint lampau)
         └── handoffs/     (narasi checkpoint sesi)
```

---

## 2. Struktur File `OVERVIEW.json`

File ini adalah *single source of truth* status sesi:

```json
{
  "project": "qoffah",
  "gate": "G3",
  "gate_status": "in_progress",
  "mode": "greenfield",
  "current_phase": 1,
  "current_task": "M1-foundation-api",
  "current_wo": null,
  "overall": "PRD v1.1 ✅ | G0-G2 ✅ | G3 🚧 M1 Foundation API | G4-G5 ⏳",
  "last_session": "2026-10-10T12:00:00+07:00",
  "last_handoff_id": null
}
```

---

## 3. Klasifikasi Work Order (Task Classification)

Setiap task di luar alur normal wajib diklasifikasikan untuk menjaga integritas dokumen:

| Tipe Work Order | Deskripsi | Dokumen Wajib Disinkronkan |
|---|---|---|
| **ENHANCEMENT** | Penambahan fitur baru atau perubahan scope | `PRD.md`, `ERD.md`, `openapi.yaml`, `BLUEPRINT.md`, `decisions.md` |
| **REFACTOR** | Restrukturisasi kode tanpa mengubah fungsionalitas | `BLUEPRINT.md`, `decisions.md` |
| **BUG** | Perbaikan kesalahan kode atau defect | `BLUEPRINT.md`, `decisions.md` |
| **GAP** | Ketidaksesuaian antara kode dan spesifikasi gate | Artefak gate terkait + `decisions.md` |
| **AUDIT** | Temuan security, performa, atau audit kepatuhan | Laporan audit + `decisions.md` |

---

## 4. Alur Sesi Waypoint (Session Lifecycle)

### A. Session Init (Awal Sesi)
1. Baca `OVERVIEW.json` untuk mengetahui posisi terkini.
2. Jika ada `ACTIVE_SPRINT.md`, baca task yang sedang berjalan.
3. Tampilkan banner status lean ke user.

### B. Session Exit / Checkpoint (Akhir Sesi)
1. Perbarui field `current_task`, `gate_status`, dan `overall` pada `OVERVIEW.json`.
2. Catat keputusan baru di `decisions.md`.
3. Jika sprint selesai, arsipkan ke `archive/sprints/`.
