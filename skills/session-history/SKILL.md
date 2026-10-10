---
name: session-history
description: Use when managing chat session history across any project. Automatically creates periodic structured checkpoints of the conversation, tracks decisions and reasoning, and preserves context across sessions. Supports discovery/change tracking (T1/T2/T3) with pre/post checkpoint variants. Loads automatically on every new session — check current.json first.
version: 1.2.0
updated: 2026-07-02
---

# Session History Manager

Mengelola chat session history dengan periodic checkpoint yang terstruktur. Skill ini otomatis aktif di setiap sesi chat.

## ⚠️ PERINGATAN — HARUS DIIKUTI

**Workflow di skill ini WAJIB diikuti. Ini BUKAN saran. Ini BUKAN optional.**
Setiap instruksi di bawah menggunakan kata **WAJIB**, **HARUS**, atau **JANGAN** — itu artinya model harus mematuhinya tanpa pengecualian.

Konsekuensi jika tidak diikuti:
- Session history terputus → user kehilangan konteks
- User harus tracking manual
- Keputusan dan reasoning hilang
- **Model dianggap gagal menjalankan tugasnya**

Model **HARUS** membaca dan mengikuti setiap langkah di bawah secara berurutan. Jangan lompat, jangan skip.

---

## Direktori & File

### Folder Data (per project — root project)

```
.opencode/session_chats/
  current.json                         # Session journal
  [topic-slug]/                        # Per topic session
    [YYYY-MM-DD]/
      checkpoint-NNN_deskripsi.md       # Checkpoint file
```

### Journal File (`current.json`)

`current.json` adalah file utama yang tracking state session. Selalu update setelah nulis checkpoint.

**Field `development` (optional):** Diisi oleh skill `waypoint` — berisi development progress. WAJIB di-preserve saat update.

```json
{
  "topic": "i18n Implementation",
  "topic_slug": "i18n-implementation",
  "started_at": "2026-06-06T10:00:00.000Z",
  "updated_at": "2026-06-06T14:30:00.000Z",
  "message_count": 42,
  "assistant_response_count": 21,
  "checkpoint_count": 5,
  "last_checkpoint": "checkpoint-005",
  "last_checkpoint_file": "checkpoint-005_fix-middleware.md",
  "checkpoint_dir": ".opencode/session_chats/i18n-implementation/2026-06-06/",

  "development": {
    "project": "dms-propertindo",
    "gate": "G3",
    "active_phase": 3,
    "active_task": "3a",
    "overall": "Phase 1 ✅ | Phase 2 ✅ | Phase 3 🚧 (0/6) | Phase 4 ⏳...",
    "last_handoff": "2026-06-15 13:00 WITA",
    "blueprint_path": "docs/theme-engine/BLUEPRINT.md"
  }
}
```

### Checkpoint File

```markdown
---
session: "i18n Implementation"
checkpoint: 5
date: 2026-06-06
time: "14:30"
topic: "Fix middleware cookie detection"
message_count: 42
previous_checkpoint: "checkpoint-004"
subagent_task: false
---

## Ringkasan

Deskripsi singkat apa yang terjadi di segmen ini.

## Percakapan

Rekaman percakapan per subtopik. Bukan verbatim, tapi captures esensi tiap exchange secara detail termasuk pertanyaan user dan jawaban/analisa model.

## Key Decisions

Tiap keputusan penting yang diambil, format:
- **Keputusan:** [keputusan]
- **Alasan:** [mengapa ambil keputusan itu]
- **Alternatif ditolak:** [opsi lain yang dipertimbangkan dan kenapa ditolak]

## Model Reasoning

Thought process dan logika teknis yang dipakai selama segmen ini. Termasuk analisa, pertimbangan trade-off, dan kesimpulan.

## Files Changed

- `path/file.ts` — [deskripsi perubahan dan alasan]
- `path/file2.ts` — [deskripsi perubahan dan alasan]

## Commands Run

- `npx tsc --noEmit` — type check, passed

## Current Status

- ✅ [selesai]
- 🚧 [sedang dikerjakan]
- ❌ [blocked]
- 📋 [next: langkah selanjutnya]
```

---

## Workflow Detail

### A. Init Session — **WAJIB di awal**

**HARUS** dilakukan di response pertama dari sesi baru (setelah restart opencode). JANGAN lewati langkah ini.

1. **WAJIB Cek** apakah `.opencode/session_chats/current.json` ada
   - **Kalau ADA** → **WAJIB baca**, **WAJIB tampilkan** detail ke user:

     ```
     ─────────────────────────────────────────────
     📋 Session sebelumnya ditemukan:
       Topic:        [topic]
       Checkpoint:   #[n] — [title]
       Terakhir:     [timestamp]
       Progress:     [status singkat]
       Files:        [jumlah file diubah]

     Mau lanjut session ini, atau mulai baru?
     [1] ✅ Lanjut — resume dari terakhir
     [2] 🆕 Baru — close session lama, start fresh
     ─────────────────────────────────────────────
     ```

     - Pilih **Lanjut** → **WAJIB baca** checkpoint terakhir, inject konteks ke reasoning
     - Pilih **Baru** → **WAJIB arsipkan** current.json (rename jadi `.opencode/session_chats/current-archived-[timestamp].json`), buat baru

   - **Kalau TIDAK ADA** → **WAJIB bikin** folder `.opencode/session_chats/` + `current.json` baru

2. **WAJIB Cek `.gitignore`** — pastikan `.opencode/` sudah di-gitignore
   - Kalau belum → **WAJIB tambahkan** baris `.opencode/`

3. **WAJIB Auto-generate title session** dari chat pertama / intent user
   - **WAJIB tampilkan** ke user buat konfirmasi: "Session ini judulnya: **[title]** — setuju?"
   - User bisa edit atau approve

4. **WAJIB Init `current.json`** dengan data session baru

5. **WAJIB tulis checkpoint-001** segera setelah session ter-init (sebelum lanjut ke task lain)
   - Folder: `.opencode/session_chats/[topic-slug]/[YYYY-MM-DD]/`
   - File: `checkpoint-001_session-init.md`
   - Jangan mulai ngerjakan task apapun sebelum checkpoint-001 selesai ditulis

### B. Checkpoint Triggers — **WAJIB dipantau**

**WAJIB** buat checkpoint ketika SALAH SATU kondisi terpenuhi. JANGAN tunda:

1. **Message-based** — **WAJIB** setiap ~5-7 assistant responses (hitung dari context window). Ini adalah trigger PALING PENTING — jangan sampai terlewat.

2. **Length estimation** — **WAJIB** jika model memperkirakan percakapan sudah mencapai ~4000+ line equivalent

3. **Topic shift** — **WAJIB** ketika user mengubah topik secara signifikan

4. **Before/after sub-agent task** — **WAJIB** sebelum delegasi ke task agent dan setelah dapat hasilnya

5. **On demand** — **WAJIB** ketika user mengatakan "save", "checkpoint", "simpen", atau "catet"

Jika checkpoint terlewat (misal karena lupa), **WAJIB buat checkpoint segera** di response berikutnya sebelum ngapa-ngapain.

### C. Sub-Agent Handling

Ketika delegasi pekerjaan ke `task` (sub-agent):

**Sebelum delegasi:**
1. Tulis checkpoint terlebih dahulu (capture state sebelum sub-agent)
2. Pass session context ke sub-agent via prompt:
   ```
   Session context:
   - Topic: [topic]
   - Checkpoint terakhir: [checkpoint-n]
   - Journal path: [path ke current.json]
   - Yang udah dikerjain: [ringkasan progress]
   ```
3. Instruct sub-agent untuk return summary terstruktur:
   ```
   Return format:
   - Task completed: yes/no/partial
   - Files changed: [list]
   - Key decisions: [list]
   - Commands run: [list]
   - Status: [detail]
   ```

**Setelah sub-agent selesai:**
1. Baca summary dari sub-agent
2. Tulis checkpoint baru — merge hasil kerja sub-agent
3. Update `current.json`

### D. Write Checkpoint — **WAJIB dieksekusi**

Saat nulis checkpoint:

1. **WAJIB tentukan path:**
   ```
   .opencode/session_chats/[topic-slug]/[YYYY-MM-DD]/checkpoint-NNN_deskripsi.md
   ```
   - `topic-slug`: lowercase, `-` sebagai separator
   - `NNN`: auto-increment dari `checkpoint_count + 1`, padded 3 digit
   - `deskripsi`: 2-5 kata, lowercase, `-` separator

2. **WAJIB isi lengkap** dengan template checkpoint di atas. JANGAN ngasih checkpoint kosong atau setengah-setengah.

3. **WAJIB Update `current.json`**:
   - `message_count` ++
   - `assistant_response_count` ++ (hitung dari context)
   - `checkpoint_count` ++
   - `last_checkpoint`, `last_checkpoint_file`, `checkpoint_dir`
   - `updated_at` = now
   - **PRESERVE** field `development` — jika sudah ada, JANGAN overwrite.
     Baca dulu `current.json` existing, update hanya field session, 
     lalu tulis kembali dengan `development` tetap utuh.

4. **JANGAN lanjut ke task lain** sebelum proses write + update selesai.

### E. Safety & Continuity

- Setiap checkpoint punya `previous_checkpoint` — jaga chain
- Kalau session interrupt mendadak, `current.json` tetap ada
- Session baru tinggal baca `current.json` → resume
- Checkpoint sebelum sub-agent task — antisipasi kalau sub-agent gagal
- Jangan overwrite file — selalu increment nomor checkpoint

---

## Integrasi dengan Skill Lain

Skill ini kompatibel dengan skill project-specific (`.project-skillset/nextjs-dms`) dan skill lainnya — jalan di background tanpa konflik. Checkpoint fokus ke **proses dan keputusan**, bukan detail teknis yang udah tercatat di file.

### Integrasi dengan Gate Orchestrator (solo-dev-workflow / sdlc-*)

Jika gate orchestrator aktif — ditandai dengan file `.opencode/phase_state/projects.json` DAN ada project `in_progress` atau `paused`:
- **Session init di-handle oleh gate orchestrator** — session-history TIDAK perlu melakukan init flow sendiri (tanya lanjut/baru, auto-generate title, checkpoint-001 session-init) di response pertama, melainkan cukup jalan di background untuk checkpointing.
- **Checkpoint WAJIB menyertakan** info gate/phase saat ini di section "Current Status" jika ada:
  ```markdown
  ## Current Status
  - 📍 Gate: G3 — Development (Checkout Module)
  - 📋 Project: E-Commerce Platform
  ```
- **Gate transition = checkpoint trigger** — setiap kali gate berubah dalam orchestrator, WAJIB buat checkpoint baru.

---

### Discovery & Change Management

Session-history menangani **capture** dari perubahan mid-development. Integrasi dengan Change Management Protocol (T1/T2/T3).

#### Trigger: Discovery Auto-Detection

Ketika user menyebut kata kunci berikut, **WAJIB** buat checkpoint pre/post:

| Kata Kunci | Kemungkinan | Action |
|------------|-------------|--------|
| "bug", "error", "ga jalan" | Bug discovery | Checkpoint pre-fix, catat finding |
| "ternyata", "oh", "rupanya" | Technical discovery (T1) | Checkpoint + tag `type: "T1"` |
| "tambah", "butuh fitur", "gimana kalau" | New requirement (T3) | Checkpoint + flag untuk PRD Amendment |
| "refactor", "ubah", "ganti" | Scope refinement (T2/T3) | Checkpoint pre/post change |
| "gap", "miss", "terlewat" | Gap detection | Checkpoint + decisions.md linkage |

**Flow:**
```
1. Deteksi keyword di input user
2. Sebelum proses: buat checkpoint pre-discovery (snapshot state)
3. Proses discovery bersama user
4. Setelah selesai: buat checkpoint post-discovery (capture hasil)
5. Update current.json → field active_discoveries
```

#### Checkpoint Template — Discovery Variants

**Pre-discovery checkpoint:**
```markdown
---
session: "i18n Implementation"
checkpoint: 12
type: "discovery-pre"
related_decision: null
active_discoveries: []
---
## Discovery Snapshot (Pre)
- **Trigger:** User said "ternyata..."
- **Context:** [apa yang sedang dikerjakan saat discovery muncul]
- **State sebelum:** [task status, file yang terbuka]
```

**Post-discovery checkpoint:**
```markdown
---
session: "i18n Implementation"
checkpoint: 13
type: "discovery-post"
related_decision: "#14"
active_discoveries: [{
  type: "T1",
  title: "Multi-repo architecture",
  decision_ref: "#14",
  status: "resolved"
}]
---
## Discovery Resolution (Post)
- **Tipe:** T1 — Technical Discovery
- **Temuan:** [deskripsi]
- **Keputusan:** [apa yang diputuskan]
- **Dampak:** [timeline, task, arsitektur]
- **Status:** ✅ Resolved / 🔄 Open
```

#### Current.json — Active Discoveries

Field tambahan di `current.json`:

```json
{
  "active_discoveries": [
    {
      "type": "T1",
      "title": "Multi-repo architecture",
      "decision_ref": "#14",
      "status": "open"
    }
  ]
}
```

Aturan:
- `open` = sedang diproses
- `resolved` = selesai, dicatat di decisions.md
- `deferred` = ditunda ke cycle berikutnya
- Max 1 `open` discovery at a time (fokus)

#### Checkpoint Frequency During Change

Saat ada discovery `active_discoveries` dengan status `open`:
- Frequency naik: **setiap 3 assistant responses** (default 5-7)
- WAJIB buat checkpoint sebelum dan sesudah setiap significant step
- Jika discovery selesai → balik ke frequency normal

#### decisions.md Linkage

Setiap checkpoint yang terkait discovery WAJIB punya `related_decision`:

```
Checkpoint #12 → Decision #14 → T1: Multi-repo → BLUEPRINT updated
```

Format di checkpoint frontmatter:
```yaml
related_decision: "#14"     # atau null jika belum ada decision
```

---

### Manual Commands (Alternatif)

Selain auto-checkpoint, user bisa pake session management commands secara manual:

| Situasi | Auto (session-history) | Manual Command | Skill Ref |
|---------|----------------------|----------------|-----------|
| Context penuh | Auto-checkpoint tiap 5-7 responses | `/compact` kompres manual | `session-commands` |
| Ganti topik total | Detect topic shift → checkpoint | `/clear` reset context | `session-commands` |
| Mau session baru | — | `/new` | `session-commands` |
| Balik ke session lama | Resume flow di init | `/resume` | `session-commands` |
| Eksperimen | — | `/fork` | `session-commands` |

**Detail lengkap command:** Lihat skill `session-commands` — context window commands, session commands, status commands, dan session architecture.

---

## 🚨 Self-Enforcement — **WAJIB dibaca di setiap response**

**Setiap kali model akan memulai response baru, WAJIB:**

1. **Hitung** jumlah assistant responses di context saat ini
2. **Cek trigger:** apakah sudah ≥5 sejak checkpoint terakhir? Atau topik berubah? Atau user minta?
3. **Kalau trigger terpenuhi → JANGAN jawab pertanyaan user dulu.** Tulis checkpoint dulu, baru jawab.
4. **Kalau checkpoint berhasil ditulis** → lanjut jawab user
5. **Kalau checkpoint GAGAL ditulis** → beritahu user: "Checkpoint gagal ditulis, [alasan]. Saya lanjut dulu, nanti saya coba lagi."

**Pengecekan ini WAJIB dilakukan di SETIAP response. JANGAN skip.**

---

## Catatan Penting

- **WAJIB baca `current.json` di awal setiap sesi** — ini pintu masuk utama
- **WAJIB ikuti flow init** — JANGAN skip meskipun lagi terburu-buru
- **WAJIB tulis checkpoint sesuai trigger** — jangan nunda, jangan lupa
- **Checkpoint WAJIB informatif** — tulis seakan-akan user akan bacanya 1 bulan kemudian
- **WAJIB nulis checkpoint daripada kehilangan konteks** — better terlalu sering
- **Sub-agent tidak punya akses skill ini** — main model yang **WAJIB** bertanggung jawab nulis checkpoint untuk sub-agent
- **Kalau ragu apakah trigger terpenuhi → tulis checkpoint.** Lebih baik kelebihan daripada kekurangan.
