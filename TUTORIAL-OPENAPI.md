# 📖 Tutorial OpenAPI — Membuat Kontrak API dari Nol

> **Tujuan:** Memahami elemen-elemen OpenAPI Specification (OAS 3.x) dan mampu menulis kontrak API sederhana untuk satu use case dengan 2 endpoint.
>
> **Use Case yang dipakai:** *Manajemen Dokumen Interkoneksi Data*, mengacu pada file [`documents-openapi.yaml`](./documents-openapi.yaml).

---

## Daftar Isi

1. [Apa itu OpenAPI?](#1-apa-itu-openapi)
2. [Mengapa Perlu Kontrak API?](#2-mengapa-perlu-kontrak-api)
3. [Elemen-Elemen OpenAPI](#3-elemen-elemen-openapi)
4. [Step-by-Step: Membuat Kontrak](#4-step-by-step-membuat-kontrak)
5. [Hasil Akhir — Gambaran Besar](#5-hasil-akhir--gambaran-besar)
6. [Tools yang Berguna](#6-tools-yang-berguna)

---

## 1. Apa itu OpenAPI?

**OpenAPI Specification (OAS)** adalah standar deskripsi API berbasis format YAML atau JSON. Tujuannya adalah mendokumentasikan "kontrak" antara:

| Pihak | Peran |
|---|---|
| **API Producer** (Back-end) | Mendefinisikan apa yang disediakan |
| **API Consumer** (Front-end / Klien) | Tahu apa yang bisa dipakai |

> **Analogi:** Bayangkan kontrak API seperti menu restoran. Menu menjelaskan *apa yang tersedia*, *bahan-bahannya*, dan *apa yang akan kamu terima* — sebelum kamu memesan.

Versi yang umum dipakai: **OpenAPI 3.0.x** dan **3.1.x** (tutorial ini memakai **3.0.3**).

---

## 2. Mengapa Perlu Kontrak API?

- ✅ **Alignment Tim** — Front-end dan back-end tidak perlu menunggu satu sama lain untuk mulai coding
- ✅ **Auto-generate** — Bisa menghasilkan SDK, mock server, dan dokumentasi interaktif (Swagger UI)
- ✅ **Validasi Otomatis** — Tools bisa memvalidasi request/response sesuai kontrak
- ✅ **Single Source of Truth** — Satu file jadi acuan seluruh tim

---

## 3. Elemen-Elemen OpenAPI

Struktur file OpenAPI terdiri dari beberapa blok utama:

```
openapi.yaml
├── openapi          ← versi spesifikasi
├── info             ← metadata API
├── servers          ← daftar URL server
├── tags             ← pengelompokan endpoint
├── paths            ← daftar endpoint (INTI)
│   └── /v1/resource
│       └── get / post / put / delete
│           ├── parameters
│           ├── requestBody
│           └── responses
└── components       ← reusable schemas, security
    ├── schemas
    └── securitySchemes
```

---

### 3.1 `openapi` — Versi Spesifikasi

```yaml
openapi: "3.0.3"
```

Wajib ada. Menentukan versi OAS yang digunakan.

---

### 3.2 `info` — Metadata API

```yaml
info:
  title: Dokumen Interkoneksi Data K/L API
  description: |
    Kontrak API untuk pengelolaan Dokumen Interkoneksi Data.
  version: "1.0.0"
  contact:
    name: Tim Integrasi Data NQA
    email: api-support@nqa.go.id
```

| Field | Keterangan |
|---|---|
| `title` | Nama API (tampil di Swagger UI) |
| `description` | Penjelasan panjang, support Markdown |
| `version` | Versi kontrak API (bukan versi OAS) |
| `contact` | Info kontak tim pemilik API |

---

### 3.3 `servers` — Daftar URL Server

```yaml
servers:
  - url: https://api.nqa.go.id
    description: Production Server
  - url: https://staging-api.nqa.go.id
    description: Staging Server
  - url: http://localhost:8000
    description: Local Development
```

Bisa mendaftarkan beberapa server (prod, staging, local). Swagger UI akan memunculnya sebagai dropdown.

---

### 3.4 `tags` — Pengelompokan Endpoint

```yaml
tags:
  - name: Documents
    description: Operasi CRUD untuk Dokumen Interkoneksi Data K/L
```

`tags` dipakai untuk mengelompokkan endpoint di dokumentasi. Setiap operasi di `paths` akan merujuk ke nama tag ini.

---

### 3.5 `paths` — Definisi Endpoint (Bagian INTI)

Ini adalah bagian terpenting. Setiap key adalah URL path, lalu di dalamnya ada **HTTP method**.

```yaml
paths:
  /v1/documents:       # ← URL path
    get:               # ← HTTP method
      ...
    post:
      ...
  /v1/documents/{id}:  # ← path parameter dengan {}
    get:
      ...
```

#### Anatomi Satu Operasi (mis. `GET /v1/documents`)

```yaml
get:
  tags:
    - Documents                  # pengelompokan
  summary: List Dokumen          # judul singkat
  description: Mengambil daftar dokumen dengan pagination.  # penjelasan panjang
  operationId: listDocuments     # ID unik untuk code generation

  parameters:                    # input dari URL / header / cookie
    - name: page
      in: query                  # lokasi: query, path, header, cookie
      required: false
      schema:
        type: integer
        default: 1

  requestBody:                   # input dari body (khusus POST/PUT/PATCH)
    required: true
    content:
      application/json:
        schema:
          $ref: "#/components/schemas/CreateDocumentRequest"

  responses:                     # semua kemungkinan response
    "200":
      description: Berhasil
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/DocumentListResponse"
    "404":
      description: Tidak ditemukan
```

#### Sub-elemen `parameters`

| Field | Nilai yang mungkin | Keterangan |
|---|---|---|
| `in` | `query`, `path`, `header`, `cookie` | Lokasi parameter |
| `required` | `true` / `false` | Wajib atau tidak |
| `schema.type` | `string`, `integer`, `boolean`, `array`, `object` | Tipe data |

#### Sub-elemen `responses`

Kode HTTP yang umum dipakai:

| Kode | Makna |
|---|---|
| `200` | OK — berhasil |
| `201` | Created — resource baru berhasil dibuat |
| `400` | Bad Request — format salah |
| `401` | Unauthorized — belum autentikasi |
| `403` | Forbidden — tidak punya izin |
| `404` | Not Found — resource tidak ada |
| `422` | Unprocessable Entity — validasi gagal |
| `500` | Internal Server Error |

---

### 3.6 `components` — Reusable Schemas

Hindari duplikasi! Definisikan schema sekali, pakai berkali-kali dengan `$ref`.

```yaml
components:
  schemas:
    Document:            # schema model utama
      type: object
      properties:
        id:
          type: string
          format: uuid
        title:
          type: string
        status:
          type: string
          enum: [DRAFT, AKTIF, KADALUARSA, DICABUT]
      required:
        - id
        - title
        - status

  securitySchemes:       # definisi metode autentikasi
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

#### Cara merujuk schema dengan `$ref`

```yaml
# Di dalam paths:
schema:
  $ref: "#/components/schemas/Document"
# Artinya: gunakan schema 'Document' yang ada di components
```

---

## 4. Step-by-Step: Membuat Kontrak

Kita akan membuat kontrak untuk **use case: Manajemen Dokumen** dengan **2 endpoint**:

| # | Endpoint | Method | Deskripsi |
|---|---|---|---|
| 1 | `/v1/documents` | `GET` | Ambil daftar dokumen |
| 2 | `/v1/documents` | `POST` | Buat dokumen baru |

---

### ✅ Step 1 — Tentukan Use Case & Endpoint

Sebelum menulis YAML, jawab dulu pertanyaan ini:

```
Use Case   : Manajemen Dokumen Interkoneksi K/L
Resource   : Documents
Endpoint 1 : GET  /v1/documents  → list semua dokumen (dengan pagination)
Endpoint 2 : POST /v1/documents  → buat dokumen baru
```

---

### ✅ Step 2 — Buat Kerangka File

Buat file baru, misalnya `documents-openapi.yaml`, lalu isi kerangkanya:

```yaml
openapi: "3.0.3"

info:
  title: Documents API
  version: "1.0.0"
  description: Kontrak API untuk manajemen dokumen.

servers:
  - url: http://localhost:8000
    description: Local Development

tags:
  - name: Documents
    description: Operasi untuk dokumen

paths:
  # Endpoint akan diisi di sini

components:
  schemas:
    # Schema akan diisi di sini
```

---

### ✅ Step 3 — Definisikan Schema di `components`

Selalu definisikan schema **sebelum** menulis paths, agar mudah dirujuk.

**3a. Model utama `Document`:**

```yaml
components:
  schemas:
    Document:
      type: object
      properties:
        id:
          type: string
          format: uuid
          readOnly: true
          example: "d1a2b3c4-e5f6-7890-abcd-ef1234567890"
        document_code:
          type: string
          maxLength: 50
          example: "DOC-2025-001"
        title:
          type: string
          maxLength: 255
          example: "Perjanjian Interkoneksi Data Kependudukan"
        status:
          type: string
          enum: [DRAFT, AKTIF, KADALUARSA, DICABUT]
          example: "AKTIF"
        created_at:
          type: string
          format: date-time
          readOnly: true
          example: "2025-01-15T08:30:00+07:00"
      required:
        - id
        - document_code
        - title
        - status
        - created_at
```

**3b. Request body untuk POST:**

```yaml
    CreateDocumentRequest:
      type: object
      properties:
        document_code:
          type: string
          example: "DOC-2025-003"
        title:
          type: string
          example: "Perjanjian Interkoneksi Data Kesehatan 2025"
        status:
          type: string
          enum: [DRAFT, AKTIF]
          default: "DRAFT"
      required:
        - document_code
        - title
```

**3c. Response envelopes** (struktur baku API):

```yaml
    # Response untuk list
    DocumentListResponse:
      type: object
      properties:
        success:
          type: boolean
          example: true
        data:
          type: array
          items:
            $ref: "#/components/schemas/Document"
        meta:
          type: object
          properties:
            request_id:
              type: string
            total_items:
              type: integer

    # Response untuk detail / create
    DocumentDetailResponse:
      type: object
      properties:
        success:
          type: boolean
          example: true
        data:
          $ref: "#/components/schemas/Document"
        meta:
          type: object
          properties:
            request_id:
              type: string

    # Response untuk error
    ErrorResponse:
      type: object
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          properties:
            code:
              type: string
              example: "VALIDATION_ERROR"
            message:
              type: string
              example: "Data tidak valid"
```

---

### ✅ Step 4 — Tulis Endpoint 1: `GET /v1/documents`

```yaml
paths:
  /v1/documents:
    get:
      tags:
        - Documents
      summary: List Dokumen
      description: Mengambil daftar dokumen dengan dukungan pagination.
      operationId: listDocuments

      parameters:
        - name: page
          in: query
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
          example: 1

        - name: page_size
          in: query
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 10
          example: 10

      responses:
        "200":
          description: Berhasil mengambil daftar dokumen
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/DocumentListResponse"
              example:
                success: true
                data:
                  - id: "d1a2b3c4-e5f6-7890-abcd-ef1234567890"
                    document_code: "DOC-2025-001"
                    title: "Perjanjian Interkoneksi Data Kependudukan"
                    status: "AKTIF"
                    created_at: "2025-01-15T08:30:00+07:00"
                meta:
                  request_id: "req-abc123"
                  total_items: 1

        "422":
          description: Parameter tidak valid
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
```

---

### ✅ Step 5 — Tulis Endpoint 2: `POST /v1/documents`

```yaml
    post:
      tags:
        - Documents
      summary: Buat Dokumen Baru
      description: Membuat entri dokumen baru.
      operationId: createDocument

      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateDocumentRequest"
            example:
              document_code: "DOC-2025-003"
              title: "Perjanjian Interkoneksi Data Kesehatan 2025"
              status: "DRAFT"

      responses:
        "201":
          description: Dokumen berhasil dibuat
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/DocumentDetailResponse"
              example:
                success: true
                data:
                  id: "f1e2d3c4-b5a6-9870-dcfe-123456789abc"
                  document_code: "DOC-2025-003"
                  title: "Perjanjian Interkoneksi Data Kesehatan 2025"
                  status: "DRAFT"
                  created_at: "2025-03-01T14:05:00+07:00"
                meta:
                  request_id: "req-xyz789"

        "422":
          description: Data request tidak valid
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
              example:
                success: false
                error:
                  code: "VALIDATION_ERROR"
                  message: "Field title wajib diisi"
```

---

### ✅ Step 6 — Validasi File YAML

Setelah semua selesai ditulis, validasi kontrak kamu:

**Cara 1 — Online (tercepat):**
1. Buka [editor.swagger.io](https://editor.swagger.io)
2. Paste isi file YAML kamu
3. Panel kiri = editor, panel kanan = preview Swagger UI
4. Jika ada error, akan muncul tanda ⚠️ di panel kiri

**Cara 2 — CLI dengan `openapi-generator` atau `spectral`:**
```bash
# Install spectral (linter OpenAPI)
npm install -g @stoplight/spectral-cli

# Jalankan validasi
spectral lint documents-openapi.yaml
```

**Cara 3 — VS Code Extension:**
Install extension **"OpenAPI (Swagger) Editor"** dari Marketplace → akan ada preview dan validasi langsung di editor.

---

## 5. Hasil Akhir — Gambaran Besar

```
documents-openapi.yaml
│
├── openapi: "3.0.3"
│
├── info
│   ├── title, description, version
│   └── contact
│
├── servers
│   ├── production
│   ├── staging
│   └── localhost
│
├── tags
│   └── Documents
│
├── paths
│   └── /v1/documents
│       ├── GET  → listDocuments   (Step 4)
│       │   ├── parameters: page, page_size
│       │   └── responses: 200, 422
│       └── POST → createDocument  (Step 5)
│           ├── requestBody: CreateDocumentRequest
│           └── responses: 201, 422
│
└── components
    └── schemas
        ├── Document                  ← model utama
        ├── CreateDocumentRequest     ← input POST
        ├── DocumentListResponse      ← output GET list
        ├── DocumentDetailResponse    ← output POST / GET detail
        └── ErrorResponse             ← output error
```

Lihat implementasi lengkapnya di: [`documents-openapi.yaml`](./documents-openapi.yaml)

---

## 6. Tools yang Berguna

| Tool | Fungsi | Link |
|---|---|---|
| **Swagger Editor** | Edit & preview online | [editor.swagger.io](https://editor.swagger.io) |
| **Swagger UI** | Dokumentasi interaktif | [swagger.io/tools/swagger-ui](https://swagger.io/tools/swagger-ui) |
| **Redoc** | Dokumentasi yang rapi | [redocly.com](https://redocly.com) |
| **Spectral** | Linting & validasi | [stoplight.io/open-source/spectral](https://stoplight.io/open-source/spectral) |
| **openapi-generator** | Generate SDK/client | [openapi-generator.tech](https://openapi-generator.tech) |
| **VS Code OpenAPI Extension** | Edit + preview di IDE | Swagger/OpenAPI di Marketplace |

---

## Ringkasan Checklist

Gunakan checklist ini setiap kali membuat kontrak API baru:

- [ ] **Step 1** — Tentukan use case, resource, dan daftar endpoint
- [ ] **Step 2** — Buat kerangka file dengan `openapi`, `info`, `servers`, `tags`
- [ ] **Step 3** — Definisikan semua schema di `components.schemas`
- [ ] **Step 4** — Tulis endpoint GET (dengan `parameters` dan `responses`)
- [ ] **Step 5** — Tulis endpoint POST (dengan `requestBody` dan `responses`)
- [ ] **Step 6** — Validasi di Swagger Editor atau CLI

---

*Tutorial ini dibuat berdasarkan use case nyata di file [`documents-openapi.yaml`](./documents-openapi.yaml) — silakan jadikan referensi implementasi lengkap.*
