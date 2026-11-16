# Turnpack MCP Global Workflow Rules

> **Status:** Global Rule (Always Active)
> **Berlaku:** Seluruh project dan workspace di Antigravity saat MCP server `turnpack` aktif.

---

## 1. Prioritas Penggunaan Tool Turnpack (`pack`, `ship`, `batch`)

Saat MCP server `turnpack` tersedia di sesi percakapan, prioritaskan ketiga tool berikut untuk memangkas jumlah *round-trip* turn dan menjaga *working tree* tetap bersih:

1. **Context Gathering (`pack`)**:
   - Gunakan `pack` dalam **1 panggilan** saat perlu melakukan pencarian regex/literal sekaligus membaca kode di sekitar temuan (`context`), melihat `git_status`, melakukan listing glob, atau membaca `>= 2` file/simbol fungsi sekaligus.
   - Jika skema tool meminta parameter `root`, isi dengan *absolute path* dari Active Workspace proyek yang sedang dikerjakan.

2. **Atomic Multi-File Edit & Test-Gated Ship (`ship`)**:
   - Gunakan `ship` dalam **1 panggilan** saat mengedit `>= 2` file sekaligus atau saat melakukan perubahan kode yang wajib lulus *test* / *typecheck*.
   - Sertakan parameter `verify` sesuai *tech stack* proyek aktif (contoh: `go test ./...`, `pytest`, `npm test`, `npx tsc --noEmit`, `cargo check`, atau `php artisan test`).
   - Jika status hasil `ship` adalah `FAILED_AND_ROLLED_BACK` atau `ABORTED_BEFORE_WRITE`, seluruh file telah otomatis dikembalikan (*byte-exact*) ke kondisi semula. Baca pesan error pada output yang dikembalikan, perbaiki edit, lalu jalankan `ship` kembali tanpa perlu me-*revert* manual.

3. **Sandboxed Bulk Inspection (`batch`)**:
   - Gunakan `batch` (dengan parameter `code`, serta `root` jika diminta) untuk menjalankan kueri atau skrip analisis *read-only* lintas banyak file sekaligus di dalam repositori dan kembalikan ringkasannya.
