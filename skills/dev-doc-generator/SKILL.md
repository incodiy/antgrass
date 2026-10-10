---
name: dev-doc-generator
description: Generate and maintain development documentation — versioned changelogs, CHANGELOG.md, README.md, and DEVELOPMENT_PROCESS.md. Auto-detects version boundaries from commit patterns, intelligently updates existing or creates new changelog entries. Works on single or multi-repo workspaces.
version: 1.0.0
updated: 2026-07-18
---

# Dev Documentation Generator

On-demand skill untuk generate dan memelihara dokumentasi development: versioned changelogs, CHANGELOG index, README, dan DEVELOPMENT_PROCESS. Mendeteksi versi secara otomatis dari pola commit, lalu chain-update semua dokumen terkait.

---

## Daftar Isi

1. [Workspace Detection](#1-workspace-detection)
2. [Git Scan & Commit Classification](#2-git-scan--commit-classification)
3. [Version Detection Logic](#3-version-detection-logic)
4. [Changelog Generation](#4-changelog-generation)
5. [Chain Update Rules](#5-chain-update-rules)
6. [Format Standards](#6-format-standards)
7. [Interactive Workflow](#7-interactive-workflow)
8. [Anti-Patterns](#8-anti-patterns)
9. [Self-Enforcement](#9-self-enforcement)
10. [Checklist](#10-checklist)

---

## 1. Workspace Detection

**Setiap kali skill di-trigger, WAJIB deteksi dulu workspace structure sebelum lakukan apapun.**

### Flow Deteksi

```
User: /skill dev-doc-generator
         │
         ▼
    [CWD = git repo?]
      │           │
     YES          NO
      │           │
      ▼           ▼
  SINGLE REPO   [Scan subdirs untuk .git]
                  │
                  ▼
              [Baca AGENTS.md jika ada]
              (parse repo declarations)
                  │
                  ▼
              [Found repos?]
                │         │
               1 repo    2+ repos
                │         │
                ▼         ▼
            AUTO SELECT  Tanya user:
                         "Generate untuk repo mana?"
                         (list semua repo yang ditemukan)
```

### Cara Detect

1. **Cek CWD**: `git rev-parse --is-inside-work-tree 2>/dev/null`
   - Output `true` → CWD adalah git repo → **Single Repo Mode**
   - Output error → lanjut ke step 2

2. **Scan subdirectories**: Cari folder yang mengandung `.git`
   ```bash
   find . -maxdepth 2 -name ".git" -type d 2>/dev/null
   ```
   - Setiap `.git` ditemukan = 1 repo

3. **Baca AGENTS.md** (jika ada di CWD atau parent):
   - Parse tabel repository declarations (kolom `Path`)
   - Contoh: `| Backend API | persija-manual-api/ |` → repo di `persija-manual-api/`

4. **Prompt user** (jika 2+ repos ditemukan):
   ```
   Ditemukan {N} repos dalam workspace:
   [1] {repo-folder-1}
   [2] {repo-folder-2}
   [3] Semua
   
   Pilih repo untuk generate dokumentasi:
   ```

### Output Path Resolution

Semua output ditulis ke **di dalam repo target**, bukan di workspace root:

| File | Path |
|------|------|
| CHANGELOG.md | `{repo-path}/CHANGELOG.md` |
| README.md | `{repo-path}/README.md` |
| DEVELOPMENT_PROCESS.md | `{repo-path}/docs/DEVELOPMENT_PROCESS.md` |
| Versioned changelogs | `{repo-path}/docs/changelog/vX.Y.Z.md` |
| Changelog index | `{repo-path}/docs/changelog/README.md` |

---

## 2. Git Scan & Commit Classification

### Scan Unmapped Commits

**WAJIB** cari commits yang belum ada di changelog manapun:

1. **Baca semua existing versioned changelogs** dari `{repo-path}/docs/changelog/v*.md`
2. **Extract commit references** yang sudah di-map (cari pola commit hash, feat/fix subject, atau fase/phase number di dalam changelog)
3. **Jalankan git log** untuk dapat semua commits:
   ```bash
   git log --format="%H|%s|%ai" --since="{last-changelog-date}" 
   ```
   Atau jika tidak ada date reference:
   ```bash
   git log --format="%H|%s|%ai"
   ```
4. **Filter**: hanya commits yang TIDAK ada di changelog manapun yang akan di-process

### Classify Commits

Setiap unmapped commit WAJIB diklasifikasikan:

| Type | Keyword | Kategori |
|------|---------|----------|
| `feat` | New feature | **Feature** |
| `fix` | Bug fix | **Fix** |
| `refactor` | Code restructuring | **Improvement** |
| `docs` | Documentation | **Documentation** |
| `style` | Code style | **Improvement** |
| `test` | Adding tests | **Quality** |
| `chore` | Maintenance | **Maintenance** |
| `perf` | Performance | **Improvement** |
| `ci` | CI/CD | **Infrastructure** |
| `build` | Build system | **Infrastructure** |

### Extract Phase Numbers

Jika commit subject mengandung pola `Phase XX` atau `phase XX`, extract angka phase-nya. Ini digunakan untuk version boundary detection.

---

## 3. Version Detection Logic

**Ini adalah intelligence core dari skill.** Harus akurat menentukan: update existing version atau buat baru.

### Decision Tree

```
[Unmapped commits ditemukan]
         │
         ▼
[Baca latest versioned changelog yang ada]
         │
         ▼
[Apply 4 Signal Checks — lihat tabel di bawah]
         │
    ┌────┴────┐
    │         │
 UPDATE     CREATE NEW
 EXISTING   VERSION
    │         │
    ▼         ▼
[Tambah    [Tentukan
 section    versi baru
 baru di     + generate
 existing    file baru]
 changelog]
```

### 4 Signal Checks

| # | Signal | Weight | Update Existing | Create New |
|---|--------|--------|-----------------|------------|
| 1 | **Phase Number Jump** | High | Phase naik 1-2 angka dalam tema sama | Phase naik 3+ angka, atau theme berubah total |
| 2 | **Thematic Shift** | High | Commits masih soal fitur yang sama (misal: fix CKEditor terus) | Commits pindah ke fitur baru yang fundamental |
| 3 | **Time Gap** | Medium | Commits within 3 hari dari latest changelog date | Gap > 5 hari antara latest changelog dan commits baru |
| 4 | **Commit Density** | Low | < 10 commits, mostly fixes/refactors | 15+ commits dengan multiple new features |

### Decision Rules

**UPDATE existing version JIKA:**
- Semua unmapped commits bertema SAMA dengan yang sudah ada di latest changelog
- Hanya berisi `fix`, `refactor`, `style`, `docs` commits (tidak ada `feat` baru yang signifikan)
- Phase number naik <= 2 dan masih dalam scope fitur yang sama
- Gap waktu <= 3 hari dari latest changelog

**CREATE NEW version JIKA:**
- Ada `feat` commits untuk fitur baru yang FUNDAMENTAL berbeda dari yang sudah ada
- Phase number naik >= 3, atau melompati phase
- Gap waktu > 5 hari dari latest changelog
- Latest changelog sudah berisi 15+ commits (versi terlalu besar)
- Theme commits berubah total (misal: dari CKEditor → Role Management)

### Version Numbering

Jika buat versi baru, tentukan nomor versi:

| Tipe Perubahan | Version Bump | Contoh |
|----------------|-------------|--------|
| Breaking changes, architectural overhaul | Major (X.0.0) | v1.0.0 → v2.0.0 |
| New features, backwards-compatible | Minor (0.X.0) | v2.0.0 → v2.1.0 |
| Bug fixes only, no new features | Patch (0.0.X) | v2.1.0 → v2.1.1 |

**Referensi**: Baca existing CHANGELOG.md untuk tahu versi terakhir, lalu increment sesuai tipe perubahan.

---

## 4. Changelog Generation

### 4a. Versioned Changelog Format

**WAJIB** ikuti format ini persis — ini standar yang sudah teruji:

```markdown
# Changelog vX.Y.Z - {Judul dalam Bahasa Indonesia}

**Release Date**: YYYY-MM-DD  
**Type**: {Initial Release|Major Release|Minor Release|Patch Release}  
**Status**: {Alpha|Beta|Production Ready}

---

## 📢 Ringkasan Bisnis & Fitur Pengguna

{Penjelasan dalam Bahasa Indonesia, non-teknis, untuk stakeholder. 
Jelaskan APA yang berubah dan KENAPA ini penting bagi pengguna. 
Gunakan numbering untuk fitur utama.}

### Fitur Utama untuk Pengguna:
1. **Nama Fitur**:
   - Penjelasan fitur 1
   - Penjelasan fitur 2
2. **Nama Fitur Lain**:
   - Penjelasan

---

## 🛠️ Detail Implementasi Teknis (Developer)

<details>
<summary><b>Lihat Detail Teknis ({Deskripsi singkat content})</b></summary>

### {Section Title}
```{language}
{Code snippet atau contoh}
```

### {Section Title 2}
- Bullet point teknis
- Detail implementation

</details>

---

**Previous Version**: [{vX.Y.Z}](vX.Y.Z.md)  
**Next Version**: TBD
```

### Format Rules

| Elemen | Aturan |
|--------|--------|
| **Judul** | `# Changelog vX.Y.Z - {Judul}` — Judul harus deskriptif dalam Bahasa Indonesia |
| **Release Date** | Format `YYYY-MM-DD` |
| **Type** | Pilih: `Initial Release`, `Major Release`, `Minor Release`, `Patch Release` |
| **Status** | `Alpha` (v0.x), `Beta` (v1.0-1.x sebelum production), `Production Ready` |
| **Ringkasan Bisnis** | WAJIB Bahasa Indonesia, non-teknis, untuk stakeholder |
| **Detail Teknis** | WAJIB dalam `<details>` collapsible, berisi code snippets |
| **Previous/Next Version** | Link ke file changelog tetangga (relative path) |
| **Commit mapping** | Catat commit hashes atau subjects di bagian teknis jika perlu |

### 4b. Update Existing Changelog

Jika keputusan adalah UPDATE, WAJIB:

1. **Baca** file existing versioned changelog
2. **Tambah** section baru di dalam `<details>` block (sebelum `</details>`)
3. **Update** Ringkasan Bisnis jika ada fitur baru yang signifikan
4. **JANGAN hapus** konten yang sudah ada
5. **Preserve** format yang sudah ada

---

## 5. Chain Update Rules

**Setelah versioned changelog di-generate atau di-update, WAJIB chain-update 3 dokumen berikut:**

### Update Order (Priority)

```
1. Versioned Changelog (vX.Y.Z.md)  ← SOURCE OF TRUTH
         │
         ▼
2. CHANGELOG.md (root index)
         │
         ▼
3. README.md (project overview)
         │
         ▼
4. DEVELOPMENT_PROCESS.md (full history)
```

### 5a. CHANGELOG.md Update Rules

| Elemen | Action |
|--------|--------|
| Version History section | Tambah/update entry versi baru di posisi yang benar (latest di atas) |
| Entry format | `- **[vX.Y.Z](docs/changelog/vX.Y.Z.md)** - YYYY-MM-DD - {Short title}` |
| Quick Links | Update jika ada category baru |
| By Category | Tambah kategori baru jika ada fitur baru |
| Last Updated | Update ke tanggal hari ini |
| Current Version | Update ke versi terbaru |

### 5b. README.md Update Rules

| Elemen | Action |
|--------|--------|
| Version badge/header | Update versi di header |
| Feature table | Tambah baris jika ada fitur baru |
| Tech stack | Update jika ada dependency baru |
| API Overview | Update endpoint count jika berubah |
| Project Status | Update metrics (commits, tables, endpoints, dll) |
| Changelog section | Pastikan link ke CHANGELOG.md benar |
| Roadmap | Tandai item yang sudah selesai |

### 5c. DEVELOPMENT_PROCESS.md Update Rules

| Elemen | Action |
|--------|--------|
| Header metadata | Update `Current Version` dan `Status` |
| Development Timeline | Tambah phase baru atau update existing phase |
| Phase section | Tambah sub-section untuk versi baru (format konsisten) |
| Architecture Evolution | Update jika ada perubahan schema/endpoint |
| Key Milestones | Tambah milestone baru jika ada pencapaian signifikan |
| Project Metrics | Update counts (commits, lines, coverage, dll) |
| Last Updated | Update timestamp |

### DEVELOPMENT_PROCESS Phase Section Format

```markdown
#### vX.Y.Z - {Title} (Mon DD)
**{Focus description}**

**Implemented:**
1. **Feature Name**:
   - Detail 1
   - Detail 2

2. **Feature Name 2**:
   - Detail

**Technical Implementation:**
- Bullet technical detail
```

---

## 6. Format Standards

### File Structure (WAJIB)

```
{repo-path}/
├── CHANGELOG.md
├── README.md
└── docs/
    ├── DEVELOPMENT_PROCESS.md
    └── changelog/
        ├── README.md          ← version table index
        ├── vX.Y.Z.md         ← individual changelogs
        └── ...
```

### docs/changelog/README.md Format

```markdown
# Changelog — {Project Name}

Dokumentasi perubahan untuk {Project Name}.

File changelog utama berada di [`CHANGELOG.md`](../../CHANGELOG.md) di root proyek.

## Daftar Versi

| Versi | Tanggal | File |
|-------|---------|------|
| vX.Y.Z | YYYY-MM-DD | [vX.Y.Z.md](./vX.Y.Z.md) |
| ... | ... | ... |

## Format

Setiap file changelog berisi:
- **Ringkasan Bisnis** — penjelasan non-teknis untuk stakeholder
- **Detail Implementasi Teknis** — daftar lengkap perubahan dengan code snippets (collapsible)
```

### Language Convention

| Context | Language |
|---------|----------|
| Ringkasan Bisnis / Fitur Pengguna | **Bahasa Indonesia** |
| Detail Implementasi Teknis | Campuran (judul section English, penjelasan boleh campur) |
| Code snippets | **English** (variable names, comments in code) |
| Section headers in changelog | **Mixed** — `## 📢 Ringkasan Bisnis & Fitur Pengguna` (Indonesian) |
| README.md | **English** (project documentation standard) |
| DEVELOPMENT_PROCESS.md | **English** with Indonesian phase titles acceptable |

---

## 7. Interactive Workflow

### Full Workflow (Step by Step)

```
STEP 1: DETECT WORKSPACE
├─ Run git detection
├─ Identify repos
└─ Confirm with user (if multi-repo)

STEP 2: SCAN & CLASSIFY
├─ Read existing changelogs
├─ Run git log
├─ Filter unmapped commits
└─ Classify by type + phase

STEP 3: VERSION DECISION
├─ Read latest changelog
├─ Apply 4 Signal Checks
├─ Present decision to user:
│   "Commits X-Y terdeteksi. 
│    [1] Update existing vX.Y.Z
│    [2] Create new vX.Y.Z
│    [3] Skip"
└─ User confirms

STEP 4: GENERATE/UPDATE CHANGELOG
├─ Write/update versioned changelog (vX.Y.Z.md)
├─ Follow format template persis
└─ Confirm to user

STEP 5: CHAIN UPDATE
├─ Update CHANGELOG.md (index)
├─ Update README.md (stats, features)
├─ Update DEVELOPMENT_PROCESS.md (phase detail)
└─ Update docs/changelog/README.md (version table)

STEP 6: REPORT
├─ Summary of changes:
│   - Files created/updated
│   - Version bump type
│   - Commits mapped
│   - Next steps (if any)
└─ Done
```

### User Prompts

Skill ini **interactive** — setiap step kunci minta konfirmasi user:

**Step 3 prompt** (version decision):
```
🔍 Detected {N} unmapped commits since vX.Y.Z ({date}):

{list commits singkat}

Signal analysis:
- Phase: {current} → {detected} ({same theme?})
- Theme: {description}
- Time gap: {X} days
- Density: {N} commits

Keputusan: [1] Update vX.Y.Z  [2] Create v{new}  [3] Skip
```

**Step 4 confirmation** (setelah generate):
```
✅ Versioned changelog vX.Y.Z.md {created/updated}

Ringkasan:
- {N} commits documented
- Fitur utama: {list}
- Type: {Minor/Major/Patch} Release

Lanjut chain update? [Y/n]
```

---

## 8. Anti-Patterns

### ❌ Anti-Pattern #1: Skip Workspace Detection

```
❌ Langsung asumsi CWD adalah repo
✅ Selalu run git detection dulu, baru tentukan output path
```

### ❌ Anti-Pattern #2: Hardcode Repo Names

```
❌ "Generate untuk persija-manual-api"
✅ "Generate untuk {detected repo name}" — dynamic dari workspace scan
```

### ❌ Anti-Pattern #3: Generate Tanpa Baca Existing

```
❌ Langsung buat changelog baru tanpa cek yang sudah ada
✅ Selalu baca existing changelogs dulu, extract mapped commits, baru process unmapped
```

### ❌ Anti-Pattern #4: Format Tidak Konsisten

```
❌ Setiap project punya format changelog beda-beda
✅ Selalu pakai format standar dari Section 6 — persis seperti persija-manual-api
```

### ❌ Anti-Pattern #5: Skip Chain Update

```
❌ Cuma update versioned changelog, lupakan CHANGELOG.md/README/DEVELOPMENT_PROCESS
✅ Selalu chain update semua 4 dokumen setiap kali ada perubahan
```

### ❌ Anti-Pattern #6: Bahasa Tidak Konsisten

```
❌ Ringkasan Bisnis dalam English
✅ Ringkasan Bisnis WAJIB Bahasa Indonesia — untuk stakeholder non-teknis
```

---

## 9. Self-Enforcement

**Sebelum eksekusi chain update, WAJIB cek:**

1. **Workspace sudah terdeteksi?** — tahu exact output path untuk semua files
2. **Unmapped commits sudah di-classify?** — tahu tipe dan phase-nya
3. **Version decision sudah dikonfirmasi user?** — update existing atau create new
4. **Versioned changelog sudah ditulis dengan format benar?** — cek template di Section 4
5. **CHANGELOG.md sudah di-update?** — version entry + last updated
6. **README.md sudah di-update?** — version badge + feature table + metrics
7. **DEVELOPMENT_PROCESS.md sudah di-update?** — phase section + timeline + metrics
8. **docs/changelog/README.md sudah di-update?** — version table

**Kalau ada step yang terlewat → koreksi SEBELUM present hasil ke user.**

---

## 10. Checklist

Sebelum menyelesaikan workflow, pastikan:

```
□ Workspace detection selesai (single/multi repo diketahui)
□ User sudah pilih target repo (jika multi-repo)
□ Git log sudah di-scan, unmapped commits teridentifikasi
□ Commits sudah diklasifikasikan (type + phase)
□ Existing changelogs sudah dibaca
□ Version decision sudah dikonfirmasi user
□ Versioned changelog sudah ditulis/diupdate dengan format benar
□ 📢 Ringkasan Bisnis dalam Bahasa Indonesia
□ 🛠️ Detail Teknis dalam <details> block
□ Previous/Next Version links benar
□ CHANGELOG.md sudah di-update (index entry + metadata)
□ README.md sudah di-update (version, features, metrics)
□ DEVELOPMENT_PROCESS.md sudah di-update (phase, timeline, metrics)
□ docs/changelog/README.md sudah di-update (version table)
□ Semua paths benar (di dalam repo target, bukan workspace root)
□ Report sudah disampaikan ke user
```
