---
name: web-scraper
description: Use when needing to extract data from public websites. Covers content extraction, markdown conversion, deep technical analysis, image inventory, and link mapping with interactive decision-flow.
version: 1.0.0
updated: 2026-06-21
---

# Web Scraper — Data Mining Skill

## Tujuan
Mengambil data dari public website secara lengkap, presisi, dan bersih — tanpa artifact HTML/JS/CSS. Output siap pakai untuk referensi konten, analisis kompetitor, atau input data ke project.

## Prinsip Dasar

### 1. Interaktif — Tiap Langkah Ditanyakan ke User
Gunakan `question` tool untuk tiap pilihan. Jangan asumsi mode atau format.

### 2. Bersihkan Semua Artifacts
HTML tags, JS code, CSS inline, class names, data attributes — semua harus dibersihkan dari output konten. User minta konten murni, bukan source code.

### 3. Jangan Asumsi — Tanyakan
Ada dua website dengan framework sama belum tentu struktur kontennya sama. Selalu konfirmasi ke user.

### 4. Output Bisa Langsung Dipakai
Hasil scraping harus dalam format yang bisa langsung di-copy-paste atau di-import ke project tanpa perlu dibersihkan lagi.

---

## Workflow — Step by Step

### Step 0: Load Skill
Ketika user bilang "ambil konten dari", "data mine", "scrape", "extract", "analisis website", atau URL langsung, load skill ini.

### Step 1: Validate URL
Gunakan `webfetch` untuk cek apakah URL reachable. Jangan lanjut kalo error (404, timeout, dll).

### Step 2: Interactive Mode Selection
Gunakan `question` tool. Tampilkan pilihan mode:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Web Scraper — Pilih Mode

URL: https://example.com

[1] 📄 Konten aja — text bersih, heading, paragraf
[2] 📝 Markdown bersih — struktur heading, list, links, bold/italic
[3] 🔧 Deep Analysis — CSS behavior, JS patterns, framework, tech stack
[4] 📦 Fetch Content — semua konten mentah apa adanya (markdown penuh)
[5] 🖼️ Images — daftar semua gambar + alt + konteks penempatan
[6] 🔗 Links — semua internal & external links
[7] 📋 Ambil Semua — konten + images + techstack + links sekaligus
[8] ✏️ Tulis detail — request kustom
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 3: Fetch & Process
Gunakan `webfetch` dengan `format: "markdown"` untuk ambil konten.

#### Mode 1 — Konten Aja:
- Parse hasil markdown, ambil hanya: headings, paragraphs, lists
- Hapus: code blocks, URLs mentah, image markdown, tables
- Output: teks murni tanpa markup

#### Mode 2 — Markdown Bersih:
- Parse hasil markdown, pertahankan struktur heading, list, bold/italic
- Hapus: code blocks, raw URLs, image syntax (tapi catat referensi gambarnya)
- Output: markdown yang udah dibersihin

#### Mode 3 — Deep Analysis:
Dari raw HTML (fetch pake `format: "html"`):
- **Framework detection**: cari `data-reactroot`, `__NEXT_DATA__`, `ng-*`, `v-if`, `data-vue-*`
- **CSS framework**: cari class names kaya `flex`, `grid`, `container` (Bootstrap), `space-y-*`, `md:` (Tailwind)
- **JS libraries**: `framer-motion`, `aos`, `swiper`, `jQuery`, `lodash`, `gsap`
- **Form behavior**: action, method, field types, validation attributes
- **Button/Interaction**: onClick patterns, `data-*` attributes, event listeners
- **Animations**: `@keyframes`, `data-aos`, `motion.div`, `transition`, `animate`
- **Accessibility**: aria attributes, role, tabindex, alt text
- **SEO**: meta tags, JSON-LD, Open Graph, canonical
- **Hosting/Infrastructure**: server headers, CDN, SSL (from response headers)
- **Analytics**: Google Analytics, Facebook Pixel, Hotjar, etc (dari script src)

Output per kategori:
```
🔧 Tech Stack:
  - Framework: Next.js 16 (__NEXT_DATA__ detected)
  - CSS: Tailwind CSS v4 (utility classes pattern)
  - Animations: AOS (data-aos attributes), Framer Motion
  - Hosting: Vercel (server header)
  
🔍 Button Analysis:
  Element: <button class="btn-gold hover:bg-gold-600 transition">
  Behavior:
    - onClick: handleSubmit (function)
    - Type: submit button
    - CSS: Tailwind custom + hover effect + transition
  
🔍 Form Analysis:
  action: /api/v1/contact
  method: POST
  Fields: name(text), email(email), phone(tel), message(textarea)
  Validation: required + pattern on phone
  CSRF: meta csrf-token ✅
```

#### Mode 4 — Fetch Content:
- Ambil markdown lengkap dari `webfetch`, jangan di-filter
- Output apa adanya

#### Mode 5 — Images:
- Scan hasil markdown untuk `![alt](url)` patterns
- Scan HTML raw untuk `<img src="...">`, `background-image: url(...)`, `srcSet`
- Output tabel:
```
| # | URL | Alt Text | Width | Height | Context |
|---|-----|----------|-------|--------|---------|
| 1 | /hero.jpg | Hero banner | 1920 | 1080 | Hero section |
| 2 | /logo.png | Logo | - | - | Header |
```

#### Mode 6 — Links:
- Ekstrak semua link dari markdown `[text](url)` dan HTML `<a href="...">`
- Kelompokkan: internal (same domain), external (different domain), anchor (#)
- Output:
```
🔗 Internal Links (24):
  - /about
  - /services
  - /contact
  ...

🔗 External Links (8):
  - https://instagram.com/...
  - https://linkedin.com/...
  ...

🔗 Anchor Links (3):
  - #hero
  - #services
  - #contact
```

#### Mode 7 — Ambil Semua:
- Gabungan dari Mode 2 (markdown bersih) + Mode 3 (deep analysis) + Mode 5 (images) + Mode 6 (links)
- Output lengkap dalam satu report

### Step 4: Interactive Output Format
Setelah data siap, tanya user format output:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Web Scraper — Format Output

[1] Tampilkan di chat — langsung baca hasilnya
[2] Simpen ke file .md — di path yang user tentukan
[3] Simpen ke file .json — structured data, gampang diparse
[4] ✏️ Tulis detail — kustom format
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Kalo pilih file, tanya juga:
```
Path tujuan: (default: docs/scraped/[domain]-[date].md/json)
```

### Step 5: Final Report

```
📋 Web Scraping Report — Selesai

URL: https://example.com
Mode: Deep Analysis
Output: docs/scraped/example-com-2026-06-06.md

Summary:
  📄 Content: 12 headings, 45 paragraphs, 2,450 words
  🖼️ Images: 8 ditemukan (3 tanpa alt text ⚠️)
  🔧 Tech Stack: Next.js 16, Tailwind CSS, AOS, Vercel
  🔗 Links: 24 internal, 8 external

Files:
  ✅ docs/scraped/example-com-2026-06-06.md — full report
```

---

## Output Format — JSON (untuk Mode 3 & 7)

```json
{
  "url": "https://example.com",
  "scrapedAt": "2026-06-06T12:00:00Z",
  "mode": "deep-analysis",
  "content": {
    "title": "Page Title",
    "headings": {
      "h1": ["Main Title"],
      "h2": ["Section 1", "Section 2"],
      "h3": ["Sub Section 1.1"]
    },
    "paragraphs": ["Paragraph 1 text...", "Paragraph 2 text..."],
    "wordCount": 2450
  },
  "images": [
    {
      "src": "/hero.jpg",
      "alt": "Hero banner",
      "width": 1920,
      "height": 1080,
      "context": "hero section"
    }
  ],
  "techStack": {
    "framework": {
      "name": "Next.js",
      "version": "16",
      "evidence": ["__NEXT_DATA__", "next/script"]
    },
    "css": {
      "name": "Tailwind CSS",
      "version": "4",
      "evidence": ["space-y-", "md:", "dark:"]
    },
    "animations": ["AOS", "Framer Motion"],
    "hosting": "Vercel",
    "analytics": ["Google Analytics 4"],
    "forms": [
      {
        "action": "/api/v1/contact",
        "method": "POST",
        "fields": [
          {"name": "email", "type": "email", "required": true}
        ]
      }
    ]
  },
  "links": {
    "internal": [{"text": "About", "href": "/about"}],
    "external": [{"text": "Instagram", "href": "https://instagram.com/..."}],
    "anchors": ["#hero", "#contact"]
  }
}
```

---

## Output Format — Markdown (untuk Mode 1, 2, 5, 6, 7)

```markdown
# Web Scraping Report
**Source**: https://example.com
**Date**: 2026-06-06
**Mode**: Content Only

## Content Structure
### Heading Hierarchy
- H1: Main Title
- H2: Section 1
  - H3: Sub Section 1.1
- H2: Section 2

### Main Content
[Clean text, no artifacts]

## Images
| # | URL | Alt | Context |
|---|-----|-----|---------|
| 1 | /hero.jpg | Hero | Hero section |

## Tech Stack
- **Framework**: Next.js 16
- **CSS**: Tailwind CSS

## Links
### Internal (24)
- /about — About Us
```

---

## Keterbatasan

| Aspek | Status | Workaround |
|-------|--------|------------|
| **SPA / JS-rendered content** | ❌ Gak bisa (webfetch gak jalanin JS) | Coba tambahin `?amp` atau `?_escaped_fragment_=` kalo ada |
| **Login-gated content** | ❌ Gak bisa | Cuma bisa akses public pages |
| **Post-click behavior** | ❌ Gak bisa | Cuma bisa analisis HTML statis |
| **Dynamic infinite scroll** | ❌ Gak bisa | Cuma dapet initial load |
| **Static HTML websites** | ✅ Optimal | Paling akurat hasilnya |
| **SSR frameworks (Next, Nuxt, etc)** | ✅ Bagus | Konten udah di-render di server |

---

## Checklist untuk Agent

```
□ URL sudah di-validasi (reachable, bukan 404/timeout)
□ Mode sudah ditanyakan ke user (jangan asumsi)
□ Output sudah dibersihkan dari artifacts HTML/JS/CSS
□ Tidak ada elemen teknis yang bocor ke output konten
□ Images report includes alt + context
□ Tech stack analysis berdasarkan evidence konkret
□ Format output sudah dikonfirmasi ke user
□ Laporan akhir diberikan
```
