---
name: sandbox-security
description: Execution sandbox security — sandbox modes, approval policies, permission overrides, and safe execution rules. Referenced by feature-workflow safety latch and security-first.
version: 1.0.0
updated: 2026-06-21
---

# Sandbox Security

> Sandbox adalah mekanisme keamanan yang ngebatasin apa yang bisa dilakukan AI agent di lingkungan eksekusi. Dipakai oleh `feature-workflow` safety latch dan `security-first` skill.

## 🔴 PERINGATAN — WAJIB DIPAHAMI SEBELUM EKSEKUSI

Sandbox yang salah konfigurasi bisa menyebabkan:
- Kode berbahaya jalan tanpa izin → data loss, security breach
- Sandbox terlalu ketat → agent gak bisa ngerjain task
- Approval policy salah → user kewalahan approve/reject tiap langkah

---

## 1. Sandbox Modes

| Mode | Proteksi | Use Case |
|------|----------|----------|
| **Bubble Wrap** | Ringan — baca/tulis file diizinkan, eksekusi command dibatasi | Daily dev, explorasi, trusted codebase |
| **Seatbelt** | Ketat — semua operasi butuh approval eksplisit | Production, untrusted codebase, sensitive data |

### Perbedaan Detail

| Operasi | Bubble Wrap | Seatbelt |
|---------|-------------|----------|
| Read file | ✅ Langsung | ✅ Langsung |
| Write file | ✅ Langsung | ⚠️ Butuh approval |
| Run command | ⚠️ Command aman otomatis, berbahaya ditanya | ❌ Semua ditanya atau diblok |
| Network access | ✅ Default allow | ❌ Default block |
| Install package | ⚠️ Ditanya | ❌ Diblok |

### Rekomendasi

```
🔄 INTERAKTIF — Pilih sandbox mode:

[1] ⚡ Bubble Wrap (recommended untuk daily dev)
    → File write langsung, command aman auto-approved
    → Cocok untuk exploratory coding, refactor, bug fix

[2] 🔒 Seatbelt (recommended untuk production/sensitive)
    → Semua operasi butuh approval
    → Cocok untuk production deploy, sensitive data access

[3] ✏️ Custom mode
    → Set sendiri tiap permission
```

---

## 2. Approval Policies

Mengontrol kapan agent minta izin user:

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **Untrusted** | Semua operasi berbahaya ditanya | Default, safety first |
| **Request** | Agent saran operasi, user approve | Collaborative dev |
| **Never** | Agent jalan tanpa izin (auto-mode) | CI/CD, trusted scripts only |

```
Policy flowchart:

User action request
    │
    ▼
┌──────────────────┐
│ Untrusted?       │───✅──► Tanya user tiap operasi
└──────┬───────────┘
       ▼ NO
┌──────────────────┐
│ Request?         │───✅──► Agent saran, user approve/reject
└──────┬───────────┘
       ▼ NO
┌──────────────────┐
│ Never?           │───✅──► Auto-execute (CI/CD only)
└──────┬───────────┘
       ▼
  Error: no policy
```

### Rekomendasi per Konteks

| Konteks | Policy | Sandbox |
|---------|--------|---------|
| Explorasi ide | Request | Bubble Wrap |
| Feature development | Request | Bubble Wrap |
| Code review / audit | Untrusted | Bubble Wrap |
| Production hotfix | Untrusted | Seatbelt |
| CI/CD pipeline | Never | Seatbelt |
| Data migration | Request | Seatbelt |

---

## 3. Permission Overrides

Override approval policy untuk skenario spesifik:

| Override | Efek |
|----------|------|
| **Auto Mode** | Skip approval — gas langsung (setara Never policy) |
| **Full Access** | Allow semua operasi — override sandbox restriction |

🔴 **PERINGATAN:** Full Access + Never Policy = NO SAFETY NET.
Hanya gunakan di isolated environment (CI/CD, Docker container, DevContainer).

### Safety Rules

```
🚨 SAFETY RULES — WAJIB sebelum override:

□ Override di lingkungan terisolasi? (CI/CD, container, dev server)
   → ❌ Production = JANGAN override
□ Ada rollback plan kalau terjadi kesalahan?
□ User explicitly minta override? (JANGAN asumsi)
□ Durasi override terbatas? (jangan permanent)
```

---

## 4. OS-Specific Settings

| OS | File System | Network | Notes |
|----|-------------|---------|-------|
| macOS | Default sandbox | Full access | SIP protects system files |
| Linux | Default sandbox | Full access | cgroups for container isolation |
| Windows (WSL2) | Cross-OS file access terbatas | Windows firewall | Path translation needed |
| Windows (CMD) | Full FS access | Windows firewall | No sandbox by default |

---

## 5. Network Access Control

| Level | Access | Use Case |
|-------|--------|----------|
| Allow all | Semua koneksi keluar | Daily dev, API calls |
| Allow specific | Whitelist domains | Enterprise, compliance |
| Block all | No network | Air-gapped, sensitive data |

---

## 6. Integration dengan feature-workflow Safety Latch

Safety latch di `feature-workflow` (CHECK 1-3) bekerja bersama sandbox:

```
feature-workflow Safety Latch
    │
    ├── CHECK 1: Stage == USER_APPROVAL_PENDING?
    ├── CHECK 2: Semua artifacts completed?
    ├── CHECK 3: User confirm execution?
    │
    ▼
sandbox-security
    ├── Sandbox mode: Bubble Wrap / Seatbelt
    ├── Approval policy: Untrusted / Request / Never
    └── Permission override: None / Auto / Full
```

**Rule:** Safety latch ngecek apakah sandbox mode sesuai dengan konteks task:
- Production task → Seatbelt Wajib
- Exploratory → Bubble Wrap cukup
- CI/CD → Never policy + Seatbelt

---

## 🚨 Self-Enforcement

**Sebelum nge-set sandbox mode atau approval policy, WAJIB cek:**

```
□ Apakah mode sesuai dengan konteks task?
   → Production → Seatbelt
   → Daily dev → Bubble Wrap
   → CI/CD → Never + Seatbelt
□ Apakah user sudah tau konsekuensi dari override?
□ Apakah ada rollback plan?
□ Kalau Full Access + Never → confirm ke user: "Ini NO SAFETY NET. Yakin?"
```
