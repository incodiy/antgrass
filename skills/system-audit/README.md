# System Audit

Comprehensive system audit for finalized code. Covers security vulnerability scanning, code quality analysis, performance profiling, error handling review, bug detection, database schema validation, API contract compliance, and architecture consistency checks.

**Arsitektur v2.6.3 — Two-Phase:** Audit dipecah menjadi Discovery (read-only, lengkap, sampai convergen via Audit Surface Contract + Tool Battery + Convergence Gate) dan Execution (fix batch terhadap frozen snapshot dengan scope-freeze). Ini memutus loop audit-fix-tak-berujung. v2.6.0 menambahkan section **Canonical Definitions** (single source of truth untuk enum/severity/path/threshold) sehingga data terstruktur tidak lagi dirawat ganda di banyak section. v2.6.1 (verification pass-4): melengkapi Field Rules, menggabungkan cross-check [12] ke checklist Self-Enforcement utama, menstandarkan pola count. v2.6.3 (verification pass-5): sinkronisasi `deferred_to`/`next_action` (wajib untuk `deferred`, bukan `accepted`), `status` dipindahkan ke base fields, lifecycle jadi pointer ke canonical, contoh finding/contract/registry disinkronkan untuk konsistensi cross-check.

**Loaded:** Automatically by `sdlc-gate-deployment` (G4) as an exit criteria gate, or manually at any time via `/audit` command.

**Produces:** Security audit report, code quality scorecard, performance profiling results, error handling review, bug inventory, database schema validation report, API contract compliance matrix, and architecture consistency report.

[Full instructions → SKILL.md](./SKILL.md)
