---
name: sdlc-gate-maintenance
description: Panduan dan checklist Gate G3-MNT (Post-Launch Maintenance, Bugfix & Continuous Enhancements). Mengatur Work Order handling, regression testing, patch versioning, dan sinkronisasi artefak SDLC.
---

# SDLC Gate G3-MNT — Maintenance & Continuous Improvement

> **Tujuan:** Menangani perbaikan bug pasca-rilis, optimasi performa, dan penambahan fitur kecil secara terkontrol tanpa merusak integritas arsitektur yang sudah berjalan.

---

## 1. Alur Penanganan Work Order Pemeliharaan

```
Laporan Isu / Request Baru
           │
           ▼
Klasifikasi Work Order (BUG / GAP / ENHANCEMENT / REFACTOR)
           │
           ▼
Cek Dampak ke Dokumen Arsitektur (ERD / OpenAPI / PRD)
           │
           ▼
Tulis Regression Test (Reproduce Bug)
           │
           ▼
Implementasi Perbaikan & Test Suite Lolos 100%
           │
           ▼
Update CHANGELOG.md & Bump Version (SemVer: PATCH / MINOR)
```

---

## 2. Aturan Kepatuhan Pemeliharaan

1. **Anti-Regression:** Setiap perbaikan bug wajib disertai unit test baru yang memvalidasi bahwa bug tersebut tidak akan muncul kembali di masa depan.
2. **Document Sync:** Jika perbaikan mengubah struktur data atau endpoint, wajib memperbarui `openapi.yaml`, `ERD.md`, dan `decisions.md` sebelum melakukan deployment patch.
