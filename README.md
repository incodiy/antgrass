# Antgrass — Native Antigravity Skillset & Configuration

> **Repository:** [incodiy/antgrass](https://github.com/incodiy/antgrass)  
> **Deskripsi:** Kumpulan skill native, aturan global (*always-on rules*), dan orchestrator SDLC / Waypoint yang dioptimalkan untuk Google Antigravity IDE.

---

## 📂 Struktur Repositori

```
antgrass/
├── rules/                           (STANDAR PASIF — Selalu Aktif Otomatis)
│   ├── code-standards.md            (Standar kode, DRY, SOLID, konvensi penamaan)
│   ├── indonesian-interaction.md    (Gaya komunikasi santai di chat, formal di dokumen)
│   ├── security-first.md            (Prinsip zero-trust, sanitasi input, env-only, UU PDP)
│   └── error-handling.md            (Pola response envelope JSON, mapping status code)
│
├── skills/                          (SKILL AKTIF — On-Demand via @mention)
│   ├── sdlc/SKILL.md                (Master Orchestrator Gate G0–G5)
│   ├── waypoint/SKILL.md            (Cross-Session Context Manager & Lean State)
│   ├── sdlc-gate-prd/SKILL.md       (Gate G0: Product Requirements)
│   ├── sdlc-gate-architecture/SKILL.md (Gate G1: System Design & ERD)
│   ├── sdlc-gate-ux/SKILL.md        (Gate G2: Design System & Wireframes)
│   ├── sdlc-gate-development/SKILL.md (Gate G3: Implementation & TDD)
│   ├── sdlc-gate-deployment/SKILL.md  (Gate G5: Production Release)
│   ├── sdlc-gate-maintenance/SKILL.md (Gate G3-MNT: Bugfix & Maintenance)
│   ├── database-design/SKILL.md     (ERD, indexing, PostgreSQL best practices)
│   ├── openapi-first/SKILL.md       (REST API specification standards)
│   ├── senior-backend-engineer/SKILL.md
│   ├── senior-react-engineer/SKILL.md
│   ├── senior-ui-ux-designer/SKILL.md
│   └── ... (Total 45 skills lengkap)
│
├── plugins/                         (Custom Plugins & Extensions)
├── sidecars/                        (Sidecar configurations)
├── mcp_config.json                  (Model Context Protocol configuration)
├── .gitignore
└── README.md
```

---

## 🚀 Cara Instalasi / Sync ke Mesin Baru

Untuk memasang konfigurasi ini pada mesin atau laptop baru:

```bash
# Clone langsung ke direktori config Antigravity
git clone https://github.com/incodiy/antgrass.git C:\Users\<username>\.gemini\config
```

---

## 💡 Fitur Unggulan

1. **Efisiensi Token 90%:** Menggunakan model state lean Waypoint v2 (`OVERVIEW.json` ~10 baris).
2. **Native Tooling:** 100% kompatibel dengan tool native Antigravity (`view_file`, `replace_file_content`, `run_command`).
3. **Cross-Platform & Windows Friendly:** Perintah terminal dan script disesuaikan dengan Windows PowerShell.
4. **Gate-Enforced SDLC:** Menjamin setiap fase pengembangan (PRD $\rightarrow$ Architecture $\rightarrow$ UX $\rightarrow$ Development $\rightarrow$ Deployment) terekam dan disetujui secara terstruktur.
