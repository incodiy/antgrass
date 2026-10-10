---
name: session-export
description: Export OpenCode chat sessions to Markdown. Lists all sessions, lets you pick by number/filter, choose output location (default ./transcripts/), and generates readable transcript with reasoning, tool calls, and token stats.
version: 1.0.0
updated: 2026-06-14
compatibility: opencode
---

# Session Export — Chat Transcript Generator

> Export OpenCode chat sessions ke file Markdown. Otomatis baca SQLite DB, list session, pilih target, generate.

## Prerequisites

Script ada 2:
- `export-transcript.mjs` — export 1 session
- `batch-export.mjs` — export ALL sessions

Keduanya di `~/.config/opencode/scripts/`. Single export:
```
node ~/.config/opencode/scripts/export-transcript.mjs <session-id> <output-path>
```
Batch export:
```
node ~/.config/opencode/scripts/batch-export.mjs <output-folder>
```

## Flow

```
Skill di-load
  │
  ▼
Baca SQLite DB: ~/.local/share/opencode/opencode.db
  │
  ▼
Query: SELECT id, title, time_created, tokens_input, tokens_output FROM session
  │
  ▼
Tampilkan daftar (Table format):
  │
  ▼
User pilih:
  [1] Export session — pilih nomor dari daftar
  [2] Filter — cari by title / date range
  [A] Export All — export semua session
  [C] Cancel
  │
  ▼
Tanya lokasi simpan:
  [1] ./transcripts/        ← default (auto-create folder)
  [2] Custom path           ← user ketik
  │
  ▼
Generate filename: [YYYY-MM-DD]_[slug].md
  │
  ▼
Jalankan script:
- Single export → `node export-transcript.mjs <session-id> <path>`
- Batch export → `node batch-export.mjs <folder>` (otomatis loop semua session, skip yang udah ada)
  │
  ▼
✅ Done — tampilkan path + ringkasan
```

## Detail Step

### 1. Load & Query DB

```js
const DB_PATH = join(homedir(), ".local", "share", "opencode", "opencode.db")
const db = new DatabaseSync(DB_PATH)

const sessions = db
  .prepare("SELECT id, title, time_created, time_updated, agent, model, tokens_input, tokens_output, cost FROM session ORDER BY time_created DESC")
  .all()
```

### 2. Display Session List

Tampilkan dalam format:

```
Daftar Session (12 total):
┌─────┬──────────────────────────────────────┬──────────────┬──────────┬────────┐
│  #  │ Title                                │ Date         │ Messages │ Tokens │
├─────┼──────────────────────────────────────┼──────────────┼──────────┼────────┤
│  1  │ Sapaan malam bro                     │ 2026-06-14   │ 42       │ 12.5K  │
│  2  │ Implementasi auth + roles            │ 2026-06-13   │ 18       │ 4.2K   │
│  3  │ Refactor solo-dev-workflow ke sdlc   │ 2026-06-14   │ 67       │ 28.1K  │
│  4  │ PRD E-Commerce Platform              │ 2026-06-12   │ 31       │ 8.9K   │
└─────┴──────────────────────────────────────┴──────────────┴──────────┴────────┘

[1-4] Pilih session
[F] Filter by title
[D] Filter by date range
[A] Export ALL
[C] Cancel
```

### 3. Filter

Kalau user pilih `[F]`:

```
Search title: [user ketik keyword]
→ Tampilkan ulang daftar yang cocok
→ Pilih nomor
```

Kalau user pilih `[D]`:

```
Start date (YYYY-MM-DD): 2026-06-01
End date (YYYY-MM-DD):   2026-06-14
→ Tampilkan ulang daftar dalam range
→ Pilih nomor
```

### 4. Pilih Lokasi

```
Output folder:
[1] ./transcripts/          ← default: buat folder di root project
[2] Custom path             ← ketik manual
[3] ./                      ← current directory

> 1
→ Auto-create ./transcripts/ (if not exists)
→ Generate filename: 2026-06-14_sapaan-malam-bro.md
```

Naming convention: `[YYYY-MM-DD]_[slug].md`
- Slug: lowercased, strip non-alphanumeric, ` ` → `-`, max 50 chars

### 5. Execute

```bash
node ~/.config/opencode/scripts/export-transcript.mjs ses_xxx ./transcripts/2026-06-14_sapaan-malam-bro.md
```

### 6. Output Confirmation

```
✅ Export selesai!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 File:  ./transcripts/2026-06-14_sapaan-malam-bro.md
📊 Size:  48 KB
💬 Messages: 42
📥 Tokens: 12.5K in / 3.2K out / 1.1K reasoning
💰 Cost:  0.0082
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[1] ✅ Export lagi
[2] 📂 Buka file (explorer)
[3] ❌ Selesai
```

## Error Handling

| Error | Response |
|-------|----------|
| DB not found | "Database OpenCode tidak ditemukan. Pastikan sudah pernah buka OpenCode." |
| No sessions | "Belum ada session tersimpan." |
| Session ID not found | "Session sudah terhapus? Coba list ulang." |
| Path not writable | "Folder tujuan tidak bisa ditulis. Coba path lain." |
| Script missing | "export-transcript.mjs tidak ditemukan. Cek ~/.config/opencode/scripts/" |

## Notes

- Script ini standalone — gak perlu chain ke skill lain
- Tapi kompatibel di-load bareng `session-history` atau `waypoint`
- Export ALL = pakai `batch-export.mjs` — loop semua session, simpan ke `./transcripts/`, skip existing files
- Batch export progres: nampilin ✅ sukses, ❌ gagal, ⏭ skip per file
