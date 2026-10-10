---
name: session-commands
description: Reference untuk session management commands — /clear, /compact, /new, /resume, /fork, /rename. Quick cheat sheet untuk context window management.
version: 1.0.0
updated: 2026-06-21
---

# Session Commands

> Cheat sheet command untuk manage context window, session, dan thread. Panggil skill ini kalo user lupa command atau butuh manage context.

---

## 1. Context Window Commands

### /clear — Reset context

```
/clear
→ Hapus semua riwayat chat di context saat ini
→ Tapi TIDAK hapus session — masih bisa /resume

Kapan pake:
- Topik berubah total (ganti task beda project)
- Context penuh dan mau mulai fresh
- Sebelum mulai task baru yang berat
```

### /compact — Kompres context

```
/compact
→ Kompres context yang udah ada — ringkas percakapan
→ Pertahankan informasi penting, buang yang gak relevan

Kapan pake:
- Context mulai panjang (15-20+ responses)
- Mau lanjut task yang sama tapi context penuh
- Alternatif: tulis manual checkpoint via session-history
```

### /plan (Plan Mode) — Aktifin thinking step

```
/plan
→ Force agent untuk planning dulu sebelum execute
→ Bakal ngajuin clarifying questions kalo ada yang ambigu

Kapan pake:
- Task kompleks yang butuh breakdown
- User gak yakin approach yang tepat
- Mau approval user dulu sebelum nulis kode
```

---

## 2. Session Commands

### /new — Start new session

```
/new [session-title]
→ Buat session baru dengan title opsional
→ Session lama remain accessible

Kapan pake:
- Mulai project baru
- Mau pisah context per feature

Contoh:
  /new Implementasi Cart Checkout
```

### /resume — Resume existing session

```
/resume [session-name-or-index]
→ Balik ke session sebelumnya — context-nya di-restore

Kapan pake:
- Ganti task dan pengen balik ke sesi sebelumnya
- Session terputus (restart IDE/CLI)

Contoh:
  /resume Implementasi Cart Checkout
  /resume 2  (resume session index ke-2)
```

### /fork — Fork current session

```
/fork [new-session-title]
→ Duplikat session saat ini ke session baru
→ Berguna buat eksperimen tanpa takut ngerusak

Kapan pake:
- Mau coba approach berbeda tanpa kehilangan context asli
- Eksplorasi "what-if" scenario

Contoh:
  /fork Eksperimen Redis instead of MySQL
```

### /rename — Rename session

```
/rename [new-title]
→ Ganti nama session (gak ngaruh ke context)

Kapan pake:
- Session title gak deskriptif
- Mau organize sessions biar rapi

Contoh:
  /rename Checkout Module - Phase 2
```

---

## 3. Status & Info Commands

### /status — Current session info

```
/status
→ Tampilin session info: title, message count, token usage, model
```

### /sessions — List all sessions

```
/sessions
→ Tampilin semua session — pilih yang mau di-resume
```

---

## 4. Session Architecture

```
Behind the scenes:

/clear  →  Hapus messages dari context array
            Tapi session config + state tetap ada di SQLite

/compact → Kompres message history
            Algorithm: summarization dari oldest messages
            Keep: last N messages (configurable)

/new    →  INSERT session baru di SQLite
            UPDATE active_session pointer

/resume →  SELECT session dari SQLite
            Load messages & config
            Set active_session

/fork   →  COPY session record + messages
            INSERT sebagai session baru
            Tambah suffix "(fork)" di title

/rename →  UPDATE session title di SQLite
```

### Storage Layer

```
SQLite database:
  ~/.opencode/sessions.db

Tabel:
  sessions:
    id, title, model, created_at, updated_at, message_count

  messages:
    id, session_id, role, content, tokens, created_at

  state:
    id, session_id, key, value (JSON)
```

---

## 5. Integration dengan session-history

`session-history` skill handle auto-checkpointing. Commands di atas bisa dipake sebagai **manual alternatives**:

| Situasi | Auto (session-history) | Manual (command) |
|---------|----------------------|-------------------|
| Context penuh | Auto-checkpoint tiap 5-7 responses | `/compact` kompres manual |
| Ganti topik | Detect topic shift → checkpoint | `/clear` reset manual |
| Mau session baru | — | `/new` |
| Balik ke session | Resume flow di init | `/resume` |
| Eksperimen | — | `/fork` |

---

## 🚨 Self-Enforcement (Quick Reference)

**Saat context mulai berat atau user ganti topik:**

```
□ Context penuh? (≥15-20 responses) → saranin /compact atau checkpoint
□ Topik beda total? → saranin /new atau /clear
□ User minta balik ke task lama? → saranin /resume
□ Mau eksperimen? → saranin /fork
```
