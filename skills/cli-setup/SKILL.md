---
name: cli-setup
description: Panduan install, auth, dan config untuk AI agent CLI tools. Provider-agnostic — bisa pake current LLM atau external. Privacy-first: API key via .env, bukan CLI.
version: 1.0.0
updated: 2026-06-21
---

# CLI Setup & Configuration

> Panduan setup environment untuk AI agent. Foundation skill — dipanggil pertama kali user setup project atau mau pake external LLM.

---

## 🔄 INTERAKTIF — Pilih Setup Mode

```
[1] ⚡ Current LLM (recommended — tidak perlu setup)
    → Pakai Big Pickle / default session LLM
    → Tidak butuh API key, langsung jalan
    → Cocok buat daily development

[2] ✏️ External LLM (setup .env)
    → Pandu setup API key via .env file
    → Privacy terjaga: key di file, bukan di CLI
    → Cocok kalo butuh model tertentu
```

---

## 1. Prerequisites

| Requirement | Version | Check |
|-------------|---------|-------|
| Node.js | ≥18 LTS | `node --version` |
| npm | ≥9 | `npm --version` |
| Git | ≥2.40 | `git --version` |
| OS | Windows: WSL2 / macOS / Linux | — |

### Windows WSL2 Setup

```bash
# 1. Install WSL2
wsl --install -d Ubuntu

# 2. Install Node.js di WSL
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# 3. Verify
node --version
npm --version
```

---

## 2. Authentication (External LLM)

Hanya perlu kalo user pilih external LLM. Dua opsi nyimpen API key:

| Method | Privacy | Use Case |
|--------|---------|----------|
| **.env file** | ✅ Tinggi — file local, .gitignore | Recommended untuk semua case |
| **Environment variable** | ✅ Tinggi — session-only | CI/CD, production |

### .env File (Recommended)

```bash
# 1. Buat .env di root project
touch .env

# 2. Isi di text editor (JANGAN ketik di CLI)
# .env:
LLM_API_KEY=sk-xxx...
LLM_BASE_URL=https://api.openai.com/v1
LLM_MODEL=gpt-4o

# 3. Pastikan di .gitignore
echo ".env" >> .gitignore
```

> 🔒 **Privacy:** API key diisi langsung di file .env pake text editor. Nggak pernah diketik atau ditampilkan di CLI/chat.

### Environment Variable (CI/CD)

```bash
# Set session-only (hilang setelah terminal ditutup)
export LLM_API_KEY="sk-xxx..."
export LLM_BASE_URL="https://api.openai.com/v1"
```

---

## 3. Configuration File

Best practice untuk config agent:

### Global Config (`~/.config/agent/config.toml`)

```toml
[general]
default_model = "gpt-4o"
sandbox = "bubble-wrap"
approval_policy = "request"

[display]
status_line = true
show_tokens = true
theme = "dark"

[session]
auto_compact = true
compact_threshold = 20
```

### Project Config (`./agent.toml`)

Override global per project:

```toml
[general]
default_model = "gpt-4o-mini"
sandbox = "seatbelt"
approval_policy = "untrusted"

[project]
name = "my-ecommerce"
type = "production"
```

### Config Priority

```
1. CLI flag (--model o3-mini)        ← Highest priority
2. Project config (./agent.toml)
3. Global config (~/.config/agent/config.toml)
4. Environment variable (LLM_MODEL)
5. Default                             ← Lowest priority
```

---

## 4. .env Template

Template `.env` untuk external LLM:

```env
# Required — API key (isi pake text editor, jangan di CLI)
LLM_API_KEY=

# Optional — override default endpoint
LLM_BASE_URL=https://api.openai.com/v1

# Optional — model default
LLM_MODEL=gpt-4o

# Optional — sandbox settings
SANDBOX_MODE=bubble-wrap      # bubble-wrap | seatbelt
APPROVAL_POLICY=request       # untrusted | request | never
```

### .gitignore Wajib

```gitignore
# .gitignore — tambahin ini
.env
.env.*
!.env.example
```

---

## 5. opencode.jsonc Integration

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": {
    "paths": ["~/.config/opencode/skills"]
  }
}
```

---

## 6. Global vs Project Config

| Config Level | File | Scope |
|-------------|------|-------|
| Global | `~/.config/opencode/opencode.jsonc` | Semua project |
| Project | `./opencode.jsonc` | Project spesifik |
| AGENTS.md | `./AGENTS.md` | Project + AI instruction |

Priority: Project > Global > Default

---

## 🚨 Self-Enforcement

**Sebelum setup environment baru, WAJIB cek:**

```
□ Pake current LLM atau external?
   → Current LLM: no setup needed
   → External: .env file udah dibuat?
□ .env ada di .gitignore?
   → JANGAN commit API key ke git
□ LLM_API_KEY terisi di .env?
□ Config priority dipahami?
   → Flag > Project > Global > Env > Default
```
