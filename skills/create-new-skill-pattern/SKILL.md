---
name: create-new-skill-pattern
description: Meta-skill untuk pembuatan skill baru. Wajib dipakai saat membuat, mengupdate, atau mengaudit skill opencode. Berisi template standar, enforcement pattern, anti-patterns, dan self-update mechanism.
version: 1.0.0
updated: 2026-06-21
---

# Create New Skill Pattern

## ⚠️ PERINGATAN — SKILL INI WAJIB DIIKUTI

**Skill ini adalah meta-skill — blueprint untuk semua skill lain.** Kualitas skill baru sangat bergantung pada seberapa baik template ini diikuti. Jika template buruk atau ada celah, maka semua skill yang dibuat dari template ini akan turunan jeleknya.

🔴 **Konsekuensi jika dilanggar:**
- **Gak pake enforcement** → model gak nge-follow workflow, skill useless
- **Gak pake self-enforcement** → model gak pernah diingetin, celah terus berulang
- **Gak bedain strict vs fleksibel** → skill yang seharusnya informatif jadi terlalu kaku, atau skill kritis jadi lemah
- **Gak pake self-update** → celah yang ditemukan di masa depan gak pernah diperbaiki
- **Struktur gak konsisten** → model bingung, tiap skill punya format beda-beda

**Model WAJIB** membaca dan mematuhi setiap aturan di bawah SETIAP KALI membuat skill baru.

---

## Daftar Isi

1. [Template Standar Skill](#1-template-standar-skill)
2. [Criticality Assessment Matrix](#2-criticality-assessment-matrix)
3. [Enforcement Pattern](#3-enforcement-pattern)
4. [Anti-Patterns](#4-anti-patterns)
5. [Self-Update Workflow](#5-self-update-workflow)
6. [Final Checklist](#6-final-checklist)
7. [Self-Enforcement](#7-self-enforcement)

---

## 1. Template Standar Skill

Setiap skill baru **WAJIB** memiliki struktur berikut:

### 1a. Frontmatter

```yaml
---
name: skill-name
description: Deskripsi singkat kapan skill ini dipakai (1-2 kalimat).
---
```

Aturan:
- `name` — huruf kecil, pakai hyphen (`-`)
- `description` — jelas, langsung nyambung ke use case

### 1b. Header + Peringatan (jika strict)

```markdown
# Nama Skill

[⚠️ peringatan WAJIB + konsekuensi — lihat Enforcement Pattern]
```

### 1c. Isi Skill

Gunakan struktur yang sesuai dengan fungsinya:

| Tipe Skill | Struktur yang Cocok |
|------------|---------------------|
| **Workflow** | Step-by-step, decision flow, interactive mode |
| **Referensi** | Tabel, daftar, checklist, contoh kode |
| **Guidance** | Prinsip, best practices, anti-patterns |
| **Template** | Template code, struktur folder, boilerplate |

### 1d. Self-Enforcement (jika strict)

```markdown
## 🚨 Self-Enforcement — [judul sesuai konteks]

**Sebelum [action], WAJIB cek:**
1. ...
2. ...
3. ...

[JIKA ada aturan yang dilanggar → koreksi SEBELUM lanjut]
```

### 1e. Checklist (opsional, recommended untuk strict skill)

```markdown
## Checklist

Sebelum selesai, pastikan:
□ Item 1
□ Item 2
□ Item 3
```

---

## 2. Criticality Assessment Matrix

Sebelum membuat skill baru, **WAJIB** tentukan dulu apakah skill ini perlu **strict** atau **fleksibel**.

### Penilaian

| Pertanyaan | Jawaban | Artinya |
|------------|---------|---------|
| Kalau dilanggar, app bisa error / gak jalan? | ✅ YA → **Strict** | ❌ TIDAK → lanjut |
| Berdampak pada security / data pengguna? | ✅ YA → **Strict** | ❌ TIDAK → lanjut |
| Build / compiler / typecheck bisa gagal? | ✅ YA → **Strict** | ❌ TIDAK → lanjut |
| Data bisa korup / hilang permanen? | ✅ YA → **Strict** | ❌ TIDAK → lanjut |
| Cuma best practice / style preference? | ✅ YA → **Fleksibel** | ❌ TIDAK → lanjut |

**Jika ada SATU jawaban "YA" di 4 pertanyaan pertama → SKILL INI STRICT.**

### Karakteristik Strict vs Fleksibel

| Aspek | Strict | Fleksibel |
|-------|--------|-----------|
| Header | ⚠️/🔴 Peringatan + konsekuensi | Cukup deskripsi biasa |
| Bahasa | **WAJIB**, **HARUS**, **JANGAN** | "Disarankan", "Sebaiknya", "Bisa" |
| Self-enforcement | ✅ WAJIB ada 🚨 section | ❌ Tidak perlu |
| Templat | Kaku, harus diikutin tepat | Fleksibel, guidance doang |
| Contoh | ❌ Salah + ✅ Benar | Cukup ✅ contoh baik |
| Checklist | ✅ Ada di akhir | ❌ Tidak perlu |

---

## 3. Enforcement Pattern

Untuk skill **strict**, **WAJIB** gunakan pola konsisten ini:

### Format

```
⚠️/🔴 [PERINGATAN — 1 baris]

[Penjelasan kenapa skill ini penting — 1-2 kalimat]

🔴 [Konsekuensi jika dilanggar — list bullet, dampak nyata]

**Model WAJIB** [instruksi tegas].
```

---
### Komponen Enforcement

**1. Banner Tingkat Keparahan**

| Level | Simbol | Contoh Header |
|-------|--------|--------------|
| Critical | 🔴 | `## 🔴 PERINGATAN KRITIKAL — WAJIB DIIKUTI` |
| High | ⚠️ | `## ⚠️ PERINGATAN — HARUS DIIKUTI` |
| Medium | | Tanpa banner, tapi ada peringatan di deskripsi |

**2. Konsekuensi Nyata**

Jangan cuma bilang "WAJIB" — jelasin **KENAPA** dengan konsekuensi nyata:

```
🔴 Konsekuensi jika dilanggar:
- [Dampak spesifik] → [akibat nyata]
- [Dampak spesifik] → [akibat nyata]
```

Contoh baik:
```
🔴 Konsekuensi jika dilanggar:
- Gak pake UUID → enumerasi ID, celah security
- Gak pake prepared statement → SQL injection, data breach
```

Contoh buruk:
```
🔴 Konsekuensi: bisa menyebabkan masalah
```

**3. Kata Kunci Tegas**

Gunakan kata-kata ini untuk instruksi kritis:

| Kata | Konteks |
|------|---------|
| **WAJIB** | Aturan absolut, gak bisa ditawar |
| **HARUS** | Sama dengan WAJIB |
| **JANGAN PERNAH** | Larangan absolut |
| **SELALU** | Kebiasaan yang gak boleh dilewat |

**4. Self-Enforcement Section**

Setiap skill **strict** **WAJIB** punya section ini di bagian akhir:

```markdown
## 🚨 Self-Enforcement — [konteks]

**Sebelum [action tertentu], WAJIB cek:**
1. ...
2. ...
3. ...

**Kalau ragu → tanya user dulu. JANGAN asumsi.**
```

---

## 4. Anti-Patterns

Daftar celah yang sudah ditemukan dari pengalaman sebelumnya. **WAJIB** dicek setiap kali membuat skill baru.

### ❌ Anti-Pattern #1: Passive Language

```
❌ "Disarankan untuk menggunakan UUID sebagai primary key"
✅ "WAJIB gunakan UUID sebagai primary key — JANGAN auto-increment"
```

**Kenapa bahaya:** Model bisa menganggapnya sebagai opsi, bukan keharusan. Skill jadi gak efektif.

**Perbaiki:** Ubah semua kata pasif jadi instruksi tegas (**WAJIB**, **HARUS**, **JANGAN**).

### ❌ Anti-Pattern #2: Tanpa Konsekuensi

```
❌ "WAJIB pake prepared statement"
✅ "WAJIB pake prepared statement. 🔴 Konsekuensi jika dilanggar: SQL injection, data breach"
```

**Kenapa bahaya:** Tanpa konsekuensi, model gak punya urgensi untuk mematuhi aturan.

**Perbaiki:** Setiap **WAJIB** harus punya alasan "kenapa" yang jelas.

### ❌ Anti-Pattern #3: Tanpa Self-Enforcement

```
❌ Skill cuma berisi instruksi, tanpa checklist di akhir
✅ Ada 🚨 Self-Enforcement section dengan checklist sebelum eksekusi
```

**Kenapa bahaya:** Model tidak pernah diingatkan untuk mengecek kepatuhan di tengah pekerjaan.

**Perbaiki:** Tambahkan 🚨 section di akhir skill strict dengan poin-poin yang harus dicek SEBELUM melanjutkan.

### ❌ Anti-Pattern #4: Semua Skill Disamakan

```
❌ Skill security dan skill code-style pakai format yang sama (sama-sama strict)
✅ Security → strict, Code-style → fleksibel
```

**Kenapa bahaya:** Skill yang terlalu strict untuk hal sepele bikin model lamban. Skill yang terlalu longgar untuk hal kritis bikin celah.

**Perbaiki:** Gunakan [Criticality Assessment Matrix](#2-criticality-assessment-matrix) untuk menentukan level.

### ❌ Anti-Pattern #5: Instruksi Ambigu

```
❌ "Handle error dengan baik"
✅ "WAJIB gunakan try-catch untuk setiap async operation. JANGAN swallow exception."
```

**Kenapa bahaya:** "Dengan baik" itu subjektif. Model bisa interpretasi beda-beda.

**Perbaiki:** Gunakan instruksi yang spesifik, terukur, dan bisa diverifikasi.

### ❌ Anti-Pattern #6: Tidak Ada Contoh Konkret

```
❌ Hanya teori tanpa contoh kode
✅ Teori + ❌ contoh salah + ✅ contoh benar
```

**Kenapa bahaya:** Model butuh referensi konkret untuk mengikuti pola dengan tepat.

**Perbaiki:** Selalu sertakan contoh kode (❌ salah + ✅ benar) untuk setiap instruksi teknis.

---

## 5. Self-Update Workflow

Skill ini dirancang untuk **selalu berkembang** — setiap kali ditemukan celah/gap baru dalam pembuatan skill, workflow ini yang akan dijalankan.

### Flow

```
[Celah/Gap Terdeteksi]
       │
       ▼
[Aku identifikasi gap-nya]
  - Apa celahnya?
  - Di konteks apa muncul?
  - Apa akar masalahnya?
       │
       ▼
[Tanya User — tampilkan di chat]

  ┌─────────────────────────────────────────────┐
  │ 🔍 Gap Baru Ditemukan di Skill Creation     │
  │                                             │
  │ Celah: [nama gap]                           │
  │ Konteks: [di mana/situasi apa]              │
  │ Dampak: [kenapa ini masalah]                │
  │ Saran update: [usulan perubahan]            │
  │                                             │
  │ [1] ✅ Update — gas                         │
  │ [2] ✏️ Tulis detail — revisi manual        │
  │ [3] ❌ Skip — gak relevan                   │
  └─────────────────────────────────────────────┘
       │
       ▼
[Eksekusi sesuai pilihan user]
  - ✅ Update → tambah anti-pattern / refine template
  - ✏️ Revisi → ikuti instruksi manual user
  - ❌ Skip → catat kenapa di-skip (opsional)
       │
       ▼
[Selesai — skill udah versi baru]
```

### Contoh Skenario

Misalnya bulan depan nemuin kasus di mana model salah interpretasi instruksi karena skill gak punya bagian "Contoh Kode":

```
🔍 Gap Baru Ditemukan

Celah: Instruksi abstrak tanpa contoh kode
Konteks: Di skill database-design, model bikin index
        tapi formatnya salah karena gak ada contoh SQL
Dampak: Index yang dibuat gak optimal, query lambat
Saran: Tambah Anti-Pattern #6: "Tidak Ada Contoh Konkret"
       ke daftar anti-patterns

[1] ✅ Update
[2] ✏️ Tulis detail
[3] ❌ Skip
```

---

## 6. Final Checklist

Sebelum menyelesaikan pembuatan skill baru, **WAJIB** cek:

```
□ Criticality Assessment sudah dilakukan?
□ Kalau strict:
  □ ⚠️/🔴 Banner terpasang di header?
  □ Konsekuensi nyata disebut?
  □ Kata WAJIB/HARUS/JANGAN digunakan?
  □ 🚨 Self-Enforcement section ada?
  □ Checklist ada?
□ Kalau fleksibel:
  □ Bahasa yang digunakan santai / informatif?
  □ Tidak ada instruksi WAJIB yang gak perlu?
  □ Tidak ada self-enforcement yang berlebihan?
□ Struktur sesuai template?
□ Anti-patterns sudah dicek?
  □ #1: Gak ada passive language?
  □ #2: Gak ada instruksi tanpa konsekuensi?
  □ #3: Self-enforcement ada (kalau strict)?
  □ #4: Criticality sudah dibedain?
  □ #5: Instruksi spesifik, gak ambigu?
  □ #6: Ada contoh konkret (❌ + ✅)?
□ README.md ada? (WAJIB untuk skill >500 lines atau complex workflow)
□ Self-update workflow sudah diingat?
  □ "Kalau nemu celah, aku bakal tanya user"
□ Tanya user review dulu sebelum finalisasi (untuk skill baru)?
```

---

## 7. Self-Enforcement

**Sebelum membuat skill baru, WAJIB baca ulang skill ini dari atas sampai bawah.**

Poin yang WAJIB dicek setiap kali:
1. **Udah baca semua anti-patterns?** — jangan sampe ngulang kesalahan yang sama
2. **Udah nentuin strict vs fleksibel?** — pake Criticality Assessment Matrix
3. **Udah siapin template sesuai struktur?** — frontmatter → header → isi → enforcement
4. **Udah tanya user?** — sebelum bikin skill baru, presentasiin dulu, jangan langsung create
5. **Kalau nemu celah, bakal inget self-update?** — ini yang paling penting

**Kalau ragu dengan struktur atau enforcement level → tanya user dulu. JANGAN asumsi.**
