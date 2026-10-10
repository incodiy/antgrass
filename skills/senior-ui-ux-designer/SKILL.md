---
name: senior-ui-ux-designer
description: Use when designing UI layouts, building responsive React components, implementing design systems, and optimizing UX flows with utility-first CSS. Handles theming, a11y, motion performance, and production-grade component patterns.
version: 1.0.0
updated: 2026-06-21
---

# ⚠️ Senior UI/UX Designer — Agent Skill

## ⚠️ PERINGATAN — WAJIB BACA SEBELUM BUAT KOMPONEN

Skill ini mengatur **pola desain UI/UX standar senior** untuk project full-stack React + utility-first CSS. Melanggar aturan di sini menghasilkan **komponen tidak konsisten, aksesibilitas buruk, performa rendering jelek, dan utang teknis**.

🔴 **Konsekuensi jika dilanggar:**
- Mobile-first diabaikan → layout broken di perangkat kecil, pengguna hilang
- UI states (loading/empty/error/disabled) tidak ada → user frustrasi, bounce rate tinggi
- a11y diabaikan → eksklusi pengguna difabel, potensi tuntutan hukum (ADA/WCAG)
- Animasi pake properti non-GPU → jank/lag, UX menderita, Core Web Vitals gagal
- Theming tidak terpusat → 100+ file diubah hanya untuk ganti warna primer

---

## Daftar Isi

1. [Interactive Setup — Framework & Pola](#1-interactive-setup--framework--pola)
2. [CSS Architecture — Satu File Pusat](#2-css-architecture--satu-file-pusat-design-tokenscss)
3. [Core Design Principles](#3-core-design-principles)
4. [Design Tokens Reference](#4-design-tokens-reference)
5. [Typography Scale](#5-typography-scale--reference-dari-designmd)
6. [Layout & Spacing](#6-layout--spacing--reference-dari-designmd)
7. [Elevation & Shadows](#7-elevation--shadows--reference-dari-designmd)
8. [Component Specifications + Contoh Kode](#8-component-specifications--contoh-kode)
9. [🚨 Self-Enforcement Checklist](#9--self-enforcement-checklist)
10. [Self-Update Workflow](#10-self-update-workflow)

---

## 1. Interactive Setup — Framework & Pola

**SETIAP KALI** skill ini dipanggil, WAJIB tanya user secara berurutan:

```
Pertanyaan 1: Design language reference?
  [1] ✅ Pilih brand dari awesome-design-md (load design-md skill)
  [2] 📄 Custom DESIGN.md (paste path)
  [3] ⏭️ Skip — pake token default

Pertanyaan 2: Utility-first CSS framework?
  [1] Tailwind CSS (v3/v4)
  [2] Bootstrap 5
  [3] Other (sebutin)

Pertanyaan 3: Glassmorphism & Shine Effect?
  [1] 🔄 Ikuti DESIGN.md (kalo brand pake glass → pake, gak → solid)
  [2] ✅ Force enable glass + shine
  [3] ❌ Force disable (solid-only)

Pertanyaan 4: Dark Mode?
  [1] ☯️ Dark + Light (default)
  [2] ☀️ Light only
  [3] 🌙 Dark only
```

Gunakan jawaban user untuk menentukan template kode dan level enforcement di bawah.

**Catatan:** Jika user pilih [1] di Pertanyaan 1, chain ke `design-md` skill untuk milih brand. `design-md` akan return design tokens yang dipake untuk generate `design-tokens.css`. Jika user pilih [3], fallback ke token default di bawah.

---

## 2. CSS Architecture — Satu File Pusat: `design-tokens.css`

### Prinsip

**Semua variabel tema, utility glassmorphism, shine effect, dan keyframes WAJIB di satu file:** `styles/design-tokens.css`.

```
styles/
├── design-tokens.css       # ➡️ SATU FILE INI: CSS vars, glass utilities, shine effect, keyframes, prefers-reduced-motion
├── globals.css              #     import design-tokens.css, reset, base styles
└── components/              #     (opsional) komponen-specific CSS kalau Tailwind gak cukup
```

### Kenapa 1 file?

| Masalah | Solusi `design-tokens.css` |
|---------|---------------------------|
| Warna tersebar di 50+ file | Ganti 1 var → semua komponen berubah |
| Dark mode susah maintain | `[data-theme="dark"]` override di 1 tempat |
| Glassmorphism dan shine di-copy paste | Utility class `.glass` dan `.shine` reusable |
| Animasi redudansi | Keyframes + `prefers-reduced-motion` di 1 tempat |
| Tailwind `@theme` terbatas | Hybrid: CSS vars + Tailwind `var(--)` referensi |

### Template `design-tokens.css` — Adaptive, Parametric

Token CSS vars **WAJIB** di-generate dari DESIGN.md jika tersedia. Template di bawah adalah **fallback default** — ganti nilai `--color-*`, `--font-*` sesuai brand.

Token flow:
1. Ada DESIGN.md → generate dari token hasil analisis
2. Gak ada DESIGN.md → pake fallback di bawah
3. User modify → override token tertentu

Template Tailwind v4 + CSS vars:

```css
/* ==========================================
   DESIGN TOKENS — satu sumber kebenaran
   ==========================================
   🎯 Generated dari DESIGN.md jika ada.
   Ganti nilai di bawah sesuai brand hasil analisis.
   ========================================== */

@import "tailwindcss";

@theme {
  --color-primary: var(--color-primary);
  --color-surface: var(--color-surface);
}

:root, [data-theme="light"] {
  /* --- Colors --- */
  --color-primary: #6366f1;
  --color-primary-hover: #4f46e5;
  --color-on-primary: #ffffff;
  --color-surface: rgba(255, 255, 255, 0.7);
  --color-surface-solid: #ffffff;
  --color-text: #1e293b;
  --color-text-muted: #64748b;
  --color-border: #e2e8f0;

  /* --- Glassmorphism Tokens --- */
  --glass-bg: rgba(255, 255, 255, 0.15);
  --glass-border: rgba(255, 255, 255, 0.25);
  --glass-blur: 12px;

  /* --- Shine Effect Tokens --- */
  --shine-color: rgba(255, 255, 255, 0.4);
  --shine-angle: 45deg;
  --shine-speed: 0.6s;

  /* --- Feedback Colors --- */
  --color-success: #22c55e;
  --color-error: #ef4444;
  --color-warning: #f59e0b;
  --color-info: #3b82f6;
}

[data-theme="dark"] {
  --color-primary: #818cf8;
  --color-primary-hover: #6366f1;
  --color-on-primary: #0f172a;
  --color-surface: rgba(15, 23, 42, 0.8);
  --color-surface-solid: #1e293b;
  --color-text: #f1f5f9;
  --color-text-muted: #94a3b8;
  --color-border: #334155;

  --glass-bg: rgba(15, 23, 42, 0.25);
  --glass-border: rgba(255, 255, 255, 0.1);
  --glass-blur: 12px;

  --shine-color: rgba(255, 255, 255, 0.2);
}

/* === UTILITY: Glassmorphism (opsional — aktif hanya jika brand pake glass aesthetic) === */
.glass {
  background: var(--glass-bg);
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  border: 1px solid var(--glass-border);
}

/* === UTILITY: Shine Effect (hover) === */
.shine {
  position: relative;
  overflow: hidden;
  isolation: isolate;
}

.shine::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    var(--shine-angle, 45deg),
    transparent 0%,
    var(--shine-color) 50%,
    transparent 100%
  );
  transform: translateX(-100%);
  transition: transform var(--shine-speed, 0.6s) ease;
  pointer-events: none;
}

.shine:hover::after {
  transform: translateX(100%);
}

/* === ANIMATIONS: GPU-accelerated only === */
@keyframes fadeScaleIn {
  from { opacity: 0; transform: scale(0.95); }
  to   { opacity: 1; transform: scale(1); }
}

@keyframes slideUp {
  from { opacity: 0; transform: translateY(16px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes skeletonPulse {
  0%, 100% { opacity: 0.4; }
  50%      { opacity: 0.8; }
}

/* === REDUCED MOTION: Hormati user preference === */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Bootstrap 5 Template (alternatif)

Kalau user pilih Bootstrap, gunakan Bootstrap CSS vars + utility override:

```css
:root, [data-theme="light"] {
  --glass-bg: rgba(255, 255, 255, 0.15);
  --glass-blur: 12px;
  /* ... sama seperti di atas ... */
}

.glass {
  background: var(--glass-bg) !important;
  backdrop-filter: blur(var(--glass-blur)) !important;
  border: 1px solid rgba(255, 255, 255, 0.25) !important;
}
```

---

## 3. Core Design Principles

### 3a. Mobile-First Responsive

**WAJIB** gunakan pendekatan mobile-first: tulis style base untuk mobile, lalu tambah breakpoint untuk layar lebih besar.

| Breakpoint | Tailwind | Bootstrap | Target Device |
|------------|----------|-----------|---------------|
| Base (0+) | (tanpa prefix) | (tanpa prefix) | Smartphone potrait |
| sm | `sm:` | `576px` | Smartphone landscape |
| md | `md:` | `768px` | Tablet |
| lg | `lg:` | `992px` | Laptop |
| xl | `xl:` | `1200px` | Desktop |
| 2xl | `2xl:` | `1400px` | Large desktop |

```
❌ Desktop-first (salah):
  <div class="flex lg:flex-col p-8 lg:p-4">

✅ Mobile-first (benar):
  <div class="flex-col lg:flex-row p-4 lg:p-8">
```

### 3b. UI States — WAJIB Lengkap

Setiap komponen interaktif WAJIB punya state berikut:

| State | Wajib? | Contoh Implementasi |
|-------|--------|---------------------|
| **Loading** | ✅ WAJIB | Skeleton UI (pulse animation) untuk card/list, spinner untuk button |
| **Empty** | ✅ WAJIB | Ilustrasi + pesan "Belum ada data" + CTA |
| **Error** | ✅ WAJIB | Pesan error + tombol retry |
| **Hover** | ✅ WAJIB | Pointer cursor, perubahan visual (shine effect) |
| **Active** | ✅ WAJIB | Scale down 95%, bg darker |
| **Focus** | ✅ WAJIB | Outline ring (Tailwind: `focus-visible:ring-2`) |
| **Disabled** | ✅ WAJIB | Opacity 50%, cursor not-allowed, no hover effect |

```tsx
// ❌ Salah: loading & empty diabaikan
function UserList() {
  const { data } = useQuery(...);
  return data.map(user => <UserCard user={user} />);
}

// ✅ Benar: complete states
function UserList() {
  const { data, isLoading, error, refetch } = useQuery(...);

  if (isLoading) return <UserListSkeleton />;
  if (error) return <ErrorState message={error.message} onRetry={refetch} />;
  if (!data?.length) return <EmptyState />;
  return data.map(user => <UserCard user={user} />);
}

function UserCard({ user }) {
  return (
    <div className="group glass shine rounded-xl p-4
                    transition-transform duration-200
                    active:scale-[0.98]
                    focus-visible:ring-2 focus-visible:ring-primary">
      {/* ... */}
    </div>
  );
}
```

### 3c. DRY & Reusable Components

**WAJIB** buat komponen kecil yang reusable. JANGAN ulang pattern yang sama di multiple file.

```
❌ Button pattern di-copy ke 5 file berbeda
✅ Satu <Button> component dengan variant prop: primary | secondary | ghost | danger

❌ Modal pattern ditulis inline tiap halaman
✅ Satu <Modal> component dengan open/close, focus trap, escape handler built-in
```

### 3d. Accessibility (a11y) — WCAG 2.1 AA

**WAJIB** penuhi standar ini untuk setiap komponen:

| Aturan | Implementasi |
|--------|-------------|
| Keyboard navigation | Semua interaktif bisa diakses via Tab/Enter/Escape |
| Focus indicator | `focus-visible:ring-2 ring-primary` — jangan pernah `outline: none` tanpa fallback |
| ARIA labels | `<button aria-label="Close">`, `<nav aria-label="Main navigation">` |
| Role attributes | `role="dialog"` untuk modal, `role="alert"` untuk error |
| Color contrast | Minimal 4.5:1 untuk text normal, 3:1 untuk large text (18px+ bold / 24px) |
| Reduced motion | `@media (prefers-reduced-motion: reduce)` — matikan/shadow animasi |

```tsx
// WAJIB: focus trap di modal
<dialog
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
  onKeyDown={(e) => {
    if (e.key === 'Escape') onClose();
    // Trap focus logic...
  }}
>
```

### 3e. Motion Performance — GPU-Only

⚠️ **Animasi WAJIB pakai properti GPU: `transform` + `opacity` SAJA.**

```
✅ Boleh:  transform: scale(), translate(), rotate();  opacity: 0→1
❌ JANGAN:  width, height, top, left, margin, padding, box-shadow, border-radius
```

---

## 4. Design Tokens Reference

### Color System

```css
--color-primary          /* Tombol, link, accent */
--color-primary-hover    /* Hover state primary */
--color-on-primary        /* Text di atas primary */
--color-surface           /* Glassmorphism bg */
--color-surface-solid     /* Solid fallback */
--color-text              /* Body text */
--color-text-muted        /* Secondary text */
--color-border            /* Divider, border */
--color-success           /* Hijau */
--color-error             /* Merah */
--color-warning           /* Kuning */
--color-info              /* Biru */
```

### Glassmorphism Tokens

```css
--glass-bg       /* background-color */
--glass-border   /* border-color */
--glass-blur     /* backdrop-filter blur px */
```

### Shine Effect Tokens

```css
--shine-color    /* Warna pantulan */
--shine-angle    /* Arah gradien */
--shine-speed    /* Durasi animasi */
```

---

## 5. Typography Scale — Reference dari DESIGN.md

Jika ada DESIGN.md, ambil typography rules dari sana. Fallback di bawah:

### Font Family Hierarchy

| Role | Font | Fallback |
|------|------|----------|
| Display / Hero | `[brand-font-display]` | `system-ui, sans-serif` |
| Heading | `[brand-font-heading]` | `system-ui, sans-serif` |
| Body | `[brand-font-body]` | `system-ui, sans-serif` |
| Mono / Code | `[brand-font-mono]` | `'Fira Code', monospace` |

### Type Scale

| Token | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| `--text-display` | `3.5rem` (56px) | `1.1` | `700` | Hero headline |
| `--text-h1` | `2.5rem` (40px) | `1.2` | `700` | Page title |
| `--text-h2` | `2rem` (32px) | `1.25` | `600` | Section heading |
| `--text-h3` | `1.5rem` (24px) | `1.3` | `600` | Card title |
| `--text-body-lg` | `1.125rem` (18px) | `1.6` | `400` | Lead text |
| `--text-body` | `1rem` (16px) | `1.6` | `400` | Body text |
| `--text-body-sm` | `0.875rem` (14px) | `1.5` | `400` | Meta / caption |
| `--text-label` | `0.75rem` (12px) | `1.4` | `500` | Badge / label |

```css
:root {
  --font-display: 'Inter', system-ui, sans-serif;
  --font-heading: 'Inter', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --font-mono: 'Fira Code', monospace;

  --text-display: 3.5rem;
  --text-h1: 2.5rem;
  --text-h2: 2rem;
  --text-h3: 1.5rem;
  --text-body-lg: 1.125rem;
  --text-body: 1rem;
  --text-body-sm: 0.875rem;
  --text-label: 0.75rem;

  --leading-display: 1.1;
  --leading-heading: 1.25;
  --leading-body: 1.6;

  --weight-display: 700;
  --weight-heading: 600;
  --weight-body: 400;
}
```

### WAJIB: Apply di Tailwind config atau CSS

```css
@theme {
  --font-display: var(--font-display);
  --font-heading: var(--font-heading);
  --font-body: var(--font-body);
  --font-mono: var(--font-mono);

  --text-display: var(--text-display);
  --text-h1: var(--text-h1);
  --text-h2: var(--text-h2);
  --text-h3: var(--text-h3);
  --text-body: var(--text-body);
  --text-body-sm: var(--text-body-sm);
}
```

## 6. Layout & Spacing — Reference dari DESIGN.md

### Spacing Scale

Gunakan skala berbasis 4px. Jika DESIGN.md punya spacing sendiri, override:

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | `0.25rem` (4px) | Micro spacing, icon gap |
| `--space-2` | `0.5rem` (8px) | Tight gap, button padding |
| `--space-3` | `0.75rem` (12px) | Input padding, small card |
| `--space-4` | `1rem` (16px) | Standard gap, section padding |
| `--space-6` | `1.5rem` (24px) | Card padding, section gap |
| `--space-8` | `2rem` (32px) | Large section padding |
| `--space-12` | `3rem` (48px) | Page section gap |
| `--space-16` | `4rem` (64px) | Hero/feature padding |

```css
:root {
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;
  --space-16: 4rem;
  --max-width: 1280px;
  --grid-columns: 12;
}
```

### Grid System

| Property | Default | Note |
|----------|---------|------|
| Max width | `1280px` | Content container |
| Columns | `12` | CSS Grid column count |
| Gap | `--space-6` | Grid gap |
| Padding | `--space-4` | Page padding (mobile: 16px) |

## 7. Elevation & Shadows — Reference dari DESIGN.md

### Box Shadow Scale

Jika DESIGN.md punya shadow tokens, override nilai di bawah:

| Token | Value | Elevation |
|-------|-------|-----------|
| `--shadow-sm` | `0 1px 2px rgba(0,0,0,0.05)` | Card default |
| `--shadow-md` | `0 4px 6px rgba(0,0,0,0.07)` | Card hover, dropdown |
| `--shadow-lg` | `0 10px 15px rgba(0,0,0,0.1)` | Modal, dialog |
| `--shadow-xl` | `0 20px 25px rgba(0,0,0,0.15)` | Toast, overlay |

### Border Radius Scale

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | `0.375rem` (6px) | Badge, label |
| `--radius-md` | `0.5rem` (8px) | Button, input |
| `--radius-lg` | `0.75rem` (12px) | Card |
| `--radius-xl` | `1rem` (16px) | Modal, dialog |
| `--radius-full` | `9999px` | Pill, avatar |

```css
:root {
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.15);

  --radius-sm: 0.375rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;
}
```

---

## 8. Component Specifications + Contoh Kode

> **Adaptive Components:** Gunakan `design-tokens.css` vars untuk styling. Jika DESIGN.md brand pake glass aesthetic → pake `.glass`. Jika solid → pake `bg-surface-solid shadow-sm`. Jangan hardcode gaya visual.

### 8a. Header — Sticky + Adaptive Background

| Aturan | Detail |
|--------|--------|
| Position | `sticky top-0 z-50` |
| Background | `var(--glass-bg)` jika glass, `var(--color-surface-solid)` jika solid |
| Height | `h-16` (64px) |
| Dark mode | Auto switch via `[data-theme]` |
| Transitions | `backdrop-filter` + `background-color` smooth |

```tsx
export function Header() {
  const [scrolled, setScrolled] = useState(false);

  useEffect(() => {
    const onScroll = () => setScrolled(window.scrollY > 0);
    window.addEventListener('scroll', onScroll, { passive: true });
    return () => window.removeEventListener('scroll', onScroll);
  }, []);

  return (
    <header
      className={`sticky top-0 z-50 h-16 transition-all duration-300
        ${scrolled
          ? 'shadow-sm backdrop-blur-[var(--glass-blur)]'
          : 'bg-transparent'}`}
      style={{
        backgroundColor: scrolled ? 'var(--glass-bg)' : 'transparent',
        borderBottom: scrolled ? '1px solid var(--color-border)' : 'none'
      }}
    >
      <nav className="mx-auto flex h-full max-w-7xl items-center justify-between px-4"
           aria-label="Main navigation">
        {/* ... */}
      </nav>
    </header>
  );
}
```

### 8b. Sidebar — Resizable (Drag-and-Drop)

| Aturan | Detail |
|--------|--------|
| Default width | `w-64` (256px) |
| Min/Max | `min-w-[200px]` / `max-w-[480px]` |
| Background | `var(--color-surface)` jika glass aktif, `var(--color-surface-solid)` jika solid |
| Resize handle | Vertical line di ujung kanan, `cursor-col-resize` |
| Storage | Simpan width ke localStorage |

```tsx
export function ResizableSidebar() {
  const [width, setWidth] = useState(() =>
    Number(localStorage.getItem('sidebar-width')) || 256
  );
  const isResizing = useRef(false);

  const handleMouseDown = useCallback(() => {
    isResizing.current = true;
    document.body.style.cursor = 'col-resize';
    document.body.style.userSelect = 'none';

    const handleMouseMove = (e: MouseEvent) => {
      if (!isResizing.current) return;
      const newWidth = Math.min(Math.max(e.clientX, 200), 480);
      setWidth(newWidth);
      localStorage.setItem('sidebar-width', String(newWidth));
    };

    const handleMouseUp = () => {
      isResizing.current = false;
      document.body.style.cursor = '';
      document.body.style.userSelect = '';
      document.removeEventListener('mousemove', handleMouseMove);
      document.removeEventListener('mouseup', handleMouseUp);
    };

    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', handleMouseUp);
  }, []);

  return (
    <aside
      className="relative flex-shrink-0 overflow-hidden transition-[width] duration-100"
      style={{ width, backgroundColor: 'var(--color-surface)' }}
      aria-label="Sidebar navigation"
    >
      <div className="flex h-full flex-col gap-2 p-4">
        {/* ... konten sidebar ... */}
      </div>

      <div
        className="absolute right-0 top-0 h-full w-1 cursor-col-resize
                   bg-transparent hover:bg-primary transition-colors"
        onMouseDown={handleMouseDown}
        role="separator"
        tabIndex={0}
        aria-label="Resize sidebar"
        onKeyDown={(e) => {
          if (e.key === 'ArrowRight') setWidth(w => Math.min(w + 20, 480));
          if (e.key === 'ArrowLeft') setWidth(w => Math.max(w - 20, 200));
        }}
      />
    </aside>
  );
}
```

### 8c. Modal — Smooth + Adaptive + Focus Trap

| Aturan | Detail |
|--------|--------|
| Backdrop | `rgba(0,0,0,0.5)` click luar → close |
| Panel | `var(--color-surface)` + shadow-lg, animated |
| Close | Escape key + tombol X + click backdrop |
| Focus trap | Tab cycle di dalam modal saja |
| a11y | `role="dialog"`, `aria-modal="true"`, `aria-labelledby` |

```tsx
export function Modal({ open, onClose, title, children }: ModalProps) {
  const dialogRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!open) return;
    const dialog = dialogRef.current;
    if (!dialog) return;

    const focusable = dialog.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const first = focusable[0];
    const last = focusable[focusable.length - 1];

    const trap = (e: KeyboardEvent) => {
      if (e.key === 'Tab') {
        if (e.shiftKey && document.activeElement === first) {
          e.preventDefault();
          last?.focus();
        } else if (!e.shiftKey && document.activeElement === last) {
          e.preventDefault();
          first?.focus();
        }
      }
    };

    dialog.addEventListener('keydown', trap);
    first?.focus();
    return () => dialog.removeEventListener('keydown', trap);
  }, [open]);

  if (!open) return null;

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      style={{ backgroundColor: 'rgba(0,0,0,0.5)' }}
      onClick={onClose}
      role="presentation"
    >
      <div
        ref={dialogRef}
        className="mx-4 w-full max-w-lg rounded-2xl p-6 shadow-xl
                   animate-[fadeScaleIn_0.2s_ease-out]"
        style={{ backgroundColor: 'var(--color-surface-solid)' }}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        onClick={(e) => e.stopPropagation()}
      >
        <div className="mb-4 flex items-center justify-between">
          <h2 id="modal-title" className="text-lg font-semibold"
              style={{ color: 'var(--color-text)' }}>
            {title}
          </h2>
          <button
            onClick={onClose}
            className="rounded-lg p-1.5 transition-colors
                       focus-visible:ring-2 focus-visible:ring-primary"
            style={{ color: 'var(--color-text-muted)' }}
            aria-label="Close modal"
          >
            ✕
          </button>
        </div>
        {children}
      </div>
    </div>
  );
}
```

### 8d. Scroll-to-Top Button

| Aturan | Detail |
|--------|--------|
| Muncul | Setelah scroll > 300px (`translate-y-0` vs `translate-y-20`) |
| Posisi | `fixed bottom-6 right-6 z-40` |
| Animasi | `transition-transform duration-300` |
| Aksi | `window.scrollTo({ top: 0, behavior: 'smooth' })` |
| Style | `glass shine rounded-full p-3 shadow-lg` |

```tsx
export function ScrollToTop() {
  const [visible, setVisible] = useState(false);

  useEffect(() => {
    const onScroll = () => setVisible(window.scrollY > 300);
    window.addEventListener('scroll', onScroll, { passive: true });
    return () => window.removeEventListener('scroll', onScroll);
  }, []);

  return (
    <button
      onClick={() => window.scrollTo({ top: 0, behavior: 'smooth' })}
      className={`glass shine fixed bottom-6 right-6 z-40 rounded-full p-3 shadow-lg
                  transition-transform duration-300
                  focus-visible:ring-2 focus-visible:ring-primary
                  ${visible ? 'translate-y-0' : 'translate-y-20 pointer-events-none'}`}
      aria-label="Scroll to top"
    >
      ↑
    </button>
  );
}
```

### 8e. Form Components — Validation States

| State | Visual | a11y |
|-------|--------|------|
| Valid | Border hijau (`border-success`) | `aria-invalid="false"` |
| Invalid | Border merah (`border-error`) + pesan error | `aria-invalid="true"` + `aria-describedby="error-id"` |
| Disabled | Opacity 50%, `cursor-not-allowed` | `disabled` attribute |
| Focus | Ring primary | `focus-visible:ring-2` |

```tsx
export function Input({
  label, error, ...props
}: InputProps) {
  const id = useId();
  const errorId = `${id}-error`;

  return (
    <div className="space-y-1">
      <label
        htmlFor={id}
        className="block text-sm font-medium text-text"
      >
        {label}
      </label>
      <input
        id={id}
        aria-invalid={!!error}
        aria-describedby={error ? errorId : undefined}
        disabled={props.disabled}
        className={`w-full rounded-lg border bg-surface-solid px-3 py-2 text-text
                    placeholder:text-text-muted
                    transition-all duration-200
                    focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary
                    disabled:cursor-not-allowed disabled:opacity-50
                    ${error ? 'border-error' : 'border-border'}
                    ${!error && 'focus:border-primary'}`}
        {...props}
      />
      {error && (
        <p id={errorId} className="text-sm text-error" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}
```

### 8f. Error Boundary + Fallback UI

```tsx
'use client';

export class ErrorBoundary extends React.Component<
  { children: ReactNode; fallback?: ReactNode },
  { hasError: boolean; error: Error | null }
> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="flex flex-col items-center justify-center gap-4 p-8 text-center">
          <div className="text-4xl">⚠️</div>
          <h2 className="text-lg font-semibold text-text">Terjadi Kesalahan</h2>
          <p className="text-sm text-text-muted">
            {this.state.error?.message || 'Something went wrong'}
          </p>
          <button
            onClick={() => this.setState({ hasError: false, error: null })}
            className="glass shine rounded-lg px-4 py-2 text-sm font-medium
                       hover:bg-surface-solid transition-colors"
          >
            Coba Lagi
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

---

## 9. 🚨 Self-Enforcement Checklist

**Sebelum menyelesaikan komponen UI baru, WAJIB cek:**

### Design Tokens
```
□ Apakah warna/margin/font yg dipakai sudah pakai CSS var dari design-tokens.css?
□ JANGAN hardcode value — misal "text-[#6366f1]" — pakai var(--color-primary)
□ Kalau ada token baru, tambahkan ke design-tokens.css, JANGAN di file lain
□ Token dari DESIGN.md sudah di-apply? (color, typography, spacing, shadow, radius)
```

### Typography
```
□ Font family sudah sesuai DESIGN.md / fallback?
□ Type scale konsisten? (display → h1 → h2 → h3 → body → body-sm → label)
□ Line height konsisten?
```

### Responsive
```
□ Apakah layout sudah mobile-first? (base = mobile, breakpoint = larger)
□ Cek di 3 breakpoint: mobile (320px), tablet (768px), desktop (1280px)
□ Apakah touch target minimal 44x44px di mobile?
```

### States
```
□ Loading state ada? (Skeleton untuk card/list, spinner untuk button)
□ Empty state ada? (ilustrasi + pesan + CTA)
□ Error state ada? (pesan error + retry)
□ Hover/Active/Focus/Disabled state ada untuk semua elemen interaktif?
```

### a11y
```
□ Keyboard navigable? (Tab, Enter, Escape)
□ Focus indicator visible? (jangan outline: none)
□ ARIA labels terisi untuk icon-only button?
□ Color contrast ≥ 4.5:1?
□ prefers-reduced-motion dihormati?
```

### Motion
```
□ Semua animasi cuma pakai transform + opacity?
□ JANGAN ada animasi width/height/top/left/box-shadow
□ Durasi animasi ≤ 300ms (kecuali shine 600ms)?
```

### Komponen Spesifik
```
□ Header: sticky + glass terpasang?
□ Sidebar: resizable handle ada? min/max width ditentukan? disimpan ke localStorage?
□ Modal: focus trap berfungsi? Escape close? backdrop click close? aria-modal="true"?
□ Scroll-to-top: muncul setelah 300px? smooth scroll?
□ Form: validation states + aria-invalid + error message?
□ Error boundary: wrapped di root atau tiap section independen?
```

**Kalau ragu → tanya user dulu. JANGAN asumsi.**

---

## 10. Self-Update Workflow

Skill ini akan terus berkembang. Kalau nemu celah/gap baru dalam proses implementasi:

```
🔍 Gap Baru Ditemukan

Celah: [nama gap]
Konteks: [di situasi apa]
Dampak: [kenapa masalah]
Saran: [usulan perubahan]

[1] ✅ Update skill
[2] ✏️ Tulis detail
[3] ❌ Skip
```
