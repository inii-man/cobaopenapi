# 📄 Kontrak OpenAPI — Dokumen Interkoneksi Data K/L

> Chapter 3: Desain API Terstruktur (Contract-First)

Repositori ini berisi kontrak **OpenAPI 3.0** untuk resource `documents` — pengelolaan Dokumen Interkoneksi Data Kementerian/Lembaga.

---

## 📁 Struktur File

```
cobaopenapi/
└── documents-openapi.yaml   # Kontrak OpenAPI utama
```

---

## 🔗 Endpoints

| Method | Path | Deskripsi |
|--------|------|-----------|
| `GET` | `/v1/documents` | List dokumen + pagination |
| `GET` | `/v1/documents/{id}` | Detail dokumen by ID |
| `POST` | `/v1/documents` | Buat dokumen baru |

---

## 📦 Model Data

Field `Document`:

| Field | Tipe | Keterangan |
|-------|------|------------|
| `id` | `uuid` | ID unik (generated server) |
| `document_code` | `string` | Kode dokumen, mis. `DOC-2025-001` |
| `title` | `string` | Judul dokumen |
| `category` | `enum` | `PERJANJIAN \| NOTA_KESEPAHAMAN \| SK \| SURAT_EDARAN \| LAINNYA` |
| `status` | `enum` | `DRAFT \| AKTIF \| KADALUARSA \| DICABUT` |
| `file_url` | `uri` | URL ke file dokumen di object storage |
| `created_at` | `date-time` | Waktu dibuat (generated server) |

---

## 📋 Response Envelope

### ✅ Success

```json
{
  "success": true,
  "data": { "...": "..." },
  "meta": {
    "request_id": "req-abc123",
    "timestamp": "2025-03-01T14:00:00+07:00"
  }
}
```

### ❌ Error

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR | DOCUMENT_NOT_FOUND | INTERNAL_ERROR",
    "message": "Pesan error yang mudah dibaca",
    "detail": [
      { "field": "title", "message": "Field wajib diisi" }
    ]
  },
  "meta": {
    "path": "/v1/documents"
  }
}
```

---

## 🚀 Cara Pakai di Swagger Editor

1. Buka **[editor.swagger.io](https://editor.swagger.io)**
2. Hapus konten default
3. Copy-paste isi `documents-openapi.yaml`
4. Panel kanan akan menampilkan dokumentasi interaktif

Atau jalankan lokal dengan Docker:

```bash
docker run -p 8080:8080 swaggerapi/swagger-editor
# Buka http://localhost:8080
```

---

## 📊 HTTP Response Codes

| Code | Kondisi | Error Code |
|------|---------|------------|
| `200 OK` | List / detail berhasil | — |
| `201 Created` | Dokumen baru dibuat | — |
| `404 Not Found` | ID tidak ditemukan | `DOCUMENT_NOT_FOUND` |
| `422 Unprocessable` | Validasi gagal | `VALIDATION_ERROR` |
| `500 Internal` | Error server | `INTERNAL_ERROR` |

---

## 📚 Referensi

- [OpenAPI 3.0 Specification](https://spec.openapis.org/oas/v3.0.3)
- [Swagger Editor Online](https://editor.swagger.io)
- [OpenAPI Best Practices](https://oai.github.io/Documentation/)
