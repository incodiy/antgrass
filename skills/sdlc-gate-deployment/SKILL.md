---
name: sdlc-gate-deployment
description: Panduan dan checklist Gate G5 (Production Release & Deployment). Mengatur konfigurasi VPS/Docker Compose, TLS/SSL Caddy/Nginx, backup otomatis, CI/CD GitHub Actions, dan monitoring kesehatan sistem.
---

# SDLC Gate G5 — Production Release & Deployment

> **Tujuan:** Melakukan rilis sistem ke lingkungan produksi (VPS/Cloud) secara aman, terotomasi, dan teruji dengan prosedur backup dan monitoring yang handal.

---

## 1. Komponen Deployment Wajib

1. **Containerization (Docker Compose):**
   - Service container: `api`, `postgres`, `redis`, `frontpage`, `admin`, `proxy`.
   - Healthcheck pada setiap service container.
2. **Keamanan Jaringan & TLS:**
   - Reverse proxy Caddy / Nginx dengan auto-HTTPS TLS 1.3 (Let's Encrypt).
   - Firewall (UFW) hanya membuka port 80, 443, dan SSH terbatas.
3. **Backup Otomatis:**
   - Cron job harian untuk `pg_dump` database + snapshot Redis AOF.
   - Sinkronisasi backup ke penyimpanan *offsite* terenkripsi.
4. **CI/CD Pipeline:**
   - GitHub Actions workflow per repositori: Linter $\rightarrow$ Test $\rightarrow$ Build Image $\rightarrow$ Deploy via SSH.

---

## 2. Checklist Kelulusan Gate G5 (Go-Live Criteria)

- [ ] Seluruh container berjalan sehat (*healthy status*)
- [ ] Sertifikat SSL/TLS aktif dan lolos uji HTTPS
- [ ] Database terisi data seed produksi dan backup otomatis teruji
- [ ] Domain publik terhubung dan endpoint health check (`/health`) mengembalikan status 200 OK
- [ ] **Persetujuan rilis final (*Go-Live Approval*) dari pengguna**
