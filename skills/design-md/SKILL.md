---
name: design-md
description: Use when the user wants a specific brand design language, needs design inspiration, or wants to use a DESIGN.md file from the awesome-design-md collection. Loads before senior-ui-ux-designer to provide design tokens.
version: 1.0.0
updated: 2026-06-21
loads: [senior-ui-ux-designer]
---

# DESIGN.md — Brand Design Language Reference

Skill ini adalah **catalog loader** untuk 73+ DESIGN.md files dari brand terkenal. Ngasih referensi visual identity ke agent, lalu delegate implementation ke `senior-ui-ux-designer`.

## Lokasi Files

```
~/.config/opencode/references/awesome-design-md/design-md/[brand]/
├── DESIGN.md            ← Design system document (yang dibaca agent)
├── preview.html         ← Visual catalog light mode
└── preview-dark.html    ← Visual catalog dark mode
```

## Kategori Brand

| Kategori | Brand |
|----------|-------|
| **AI/LLM** | Claude, Cohere, ElevenLabs, Mistral, Ollama, OpenCode AI, Replicate, Runway, Together AI, VoltAgent, xAI |
| **Developer Tools** | Cursor, Expo, Lovable, Raycast, Superhuman, Vercel, Warp |
| **Backend/DevOps** | ClickHouse, Composio, HashiCorp, MongoDB, PostHog, Sanity, Sentry, Supabase |
| **SaaS/Productivity** | Cal.com, Intercom, Linear, Mintlify, Notion, Resend, Zapier |
| **Design/Creative** | Airtable, Clay, Figma, Framer, Miro, Webflow |
| **Fintech/Crypto** | Binance, Coinbase, Kraken, Mastercard, Revolut, Stripe, Wise |
| **E-commerce/Retail** | Airbnb, Meta, Nike, Shopify, Starbucks |
| **Media/Consumer** | Apple, HP, IBM, NVIDIA, Pinterest, PlayStation, SpaceX, Spotify, The Verge, Uber, Vodafone, WIRED |
| **Automotive** | BMW, BMW M, Bugatti, Ferrari, Lamborghini, Renault, Tesla |
| **Retro Web** | Dell (1996), Nintendo (2001) |

## Interactive Flow

### Step 1 — Tanya sumber DESIGN.md

Setiap kali skill ini di-load, WAJIB tanya user:

```
DESIGN.md — mau pake reference brand atau pake default?

[1] ✅ Pilih dari awesome-design-md (73 brand)
[2] 📄 Custom — path ke DESIGN.md file sendiri
[3] ⏭️ Skip — pake token default dari senior-ui-ux-designer
```

### Step 2 — Brand Selection

*(Hanya muncul jika user pilih [1])*

Tanya user preferensi brand. Kalo user gak tahu, tawarkan pilihan kategori:

```
Kategori brand yang tersedia:
[1] AI/Tech         [2] Developer Tools   [3] Backend/DevOps
[4] SaaS            [5] Design/Creative   [6] Fintech
[7] E-commerce      [8] Media/Consumer    [9] Automotive
[A] Retro Web       [B] Browse semua

Atau ketik nama brand langsung (misal: "Stripe", "Vercel", "Ferrari")
```

### Step 3 — Analisis & Apply

Baca DESIGN.md dari brand terpilih, summarise ke user:

```
Hasil analisis DESIGN.md — [Brand Name]

🎨 Color Palette:
  Primary:   #xxx  — [role]
  Secondary: #xxx  — [role]
  Accent:    #xxx  — [role]
  Neutral:   #xxx  — [role]
  Success/Error/Warning/Info: #xxx

🔤 Typography:
  Heading:   [font], [weights], [size range]
  Body:      [font], [weights], [size range]
  Mono:      [font], [weights]

📐 Layout:
  Spacing:   [scale]
  Grid:      [columns]
  Max width: [value]

☁️ Elevation:
  Shadow:    [sm/md/lg values]
  Radius:    [sm/md/lg values]

[1] ✅ Apply — generate token dan teruskan ke senior-ui-ux-designer
[2] ✏️ Modify — override beberapa token (warna/font spesifik)
[3] ❌ Cancel — balik ke default
```

### Step 4 — Eksekusi

Jika user pilih [1] atau [2], kumpulkan:

```
design_tokens = {
  color: { primary, secondary, accent, neutral, success, error, warning, info },
  typography: { heading_font, body_font, mono_font, scale, weights },
  layout: { spacing, grid_columns, max_width },
  elevation: { shadows, radius },
  glassmorphism: { enabled: bool },
  dark_mode: { enabled: bool }
}
```

Teruskan token ini ke `senior-ui-ux-designer` dengan instruksi:
"Gunakan token dari DESIGN.md ini untuk generate design-tokens.css. Jangan pake hardcoded default."

## Fallback (Tidak Ada DESIGN.md)

Jika user skip atau cancel, agent harus tetep jalan normal:
- Tanya CSS framework (Tailwind vs Bootstrap)
- Tanya glassmorphism preference

## Self-Enforcement Checklist

```
□ SUDAH tanya user soal DESIGN.md?
□ Sudah summarise brand identity sebelum apply?
□ Token sudah diteruskan ke senior-ui-ux-designer?
□ Kalo user skip → fallback ke flow normal?
□ Kalo user pilih custom path → file exists?
```
