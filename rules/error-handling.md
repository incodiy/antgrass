# Error Handling & Response Envelope Rule

> **Status:** Global Rule (Always Active)
> **Tujuan:** Standarisasi struktur respon API dan pola penanganan error yang konsisten di backend dan frontend.

---

## 1. Standar Response Envelope (REST API)

Semua respon endpoint API wajib mengikuti struktur envelope terpadu:

### Respon Sukses (2xx)
```json
{
  "success": true,
  "data": { ... },
  "message": "Operasi berhasil",
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total": 114
  }
}
```

### Respon Error (4xx / 5xx)
```json
{
  "success": false,
  "message": "Validasi input gagal atau data tidak ditemukan",
  "errors": {
    "email": ["Format email tidak valid"],
    "password": ["Password minimal 8 karakter"]
  }
}
```

---

## 2. Pemetaan Kode Status HTTP

| HTTP Code | Kondisi | Penanganan di Client |
|---|---|---|
| **200 / 201** | Request sukses | Render data ke UI |
| **400** | Bad request / business logic error | Tampilkan toast / inline warning |
| **401** | Token expired / unauthorized | Trigger silent refresh token atau redirect ke `/login` |
| **403** | Forbidden / role tidak cukup | Tampilkan pesan akses ditolak |
| **404** | Resource tidak ditemukan | Tampilkan EmptyState spesifik |
| **422** | Unprocessable Entity / validasi form | Petakan pesan error langsung ke input form terkait |
| **500** | Internal Server Error | Log detail di backend, tampilkan pesan error aman ke user |

---

## 3. Aturan Error Handling di Frontend
- Tangkap error via React Query / Axios Interceptors.
- Jangan biarkan layar aplikasi blank (*white screen*). Gunakan `ErrorBoundary` dan komponen `ErrorState` dengan tombol retry yang jujur.
