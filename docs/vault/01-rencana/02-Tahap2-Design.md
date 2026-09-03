# Tahap 2 — Design

**Status keseluruhan:** SELESAI (2026-09-04) — X1, X2, X3, X4 semua tuntas.
Tahap 2 (Design) resmi selesai, boleh mulai Fase 2 (Build) — lihat
[[03-Tahap3-Build]] dan [[00-Slice-Backlog]] S2-1 (perlu ditulis ulang dari
hasil X2/X4, jangan pakai isi lama).

Ini tracker kerja untuk Tahap 2. Alasan tahap ini ada — cetak biru sebelum kode
ditulis, dan kenapa kesalahan di sini paling mahal diperbaiki belakangan — ada di
[[00-Grand-Plan]]. File ini menjawab: *sejauh mana Tahap 2 ini selesai.*

Prasyarat: [[01-Tahap1-Discovery]] selesai dulu. X2 (model data) juga butuh
sebagian X4 (keputusan teknologi) selesai lebih dulu — urutan sebenarnya:
**X1 → X4 batch 1 → X2 → X3 → X4 batch 2.** Lihat bagian urutan di bawah.

Dipakai lewat `/planning`.

---

## X1 — Information Architecture

**Status:** SELESAI (2026-09-04)

**Selesai jika:** kamu bisa menyebut semua jenis halaman dan apa isinya.

### Checklist

- [x] Sitemap — semua halaman publik dan admin
- [x] Daftar jenis halaman (homepage, listing, detail artikel, halaman game, dst)
- [x] Taksonomi: kategori vs tag vs platform vs genre — bedanya apa, kapan pakai
      yang mana
- [x] Wireframe kasar homepage + halaman artikel (coretan tangan cukup — yang
      penting elemen apa saja yang wajib ada, karena itu menentukan field di
      model data)

### Catatan kerja

**Sitemap — draf sesi 2026-09-04:**

Publik (`web_portal`):
| Halaman | URL | Isi |
|---|---|---|
| Homepage | `/` | Artikel terbaru lintas tipe |
| Listing artikel per tipe | `/{articleType}` | Filter `?platform=`/`?game=` |
| Detail artikel | `/{articleType}/{slug}` | Isi, skor (review), komentar (nanti) |
| Listing game | `/games` | Browse semua game, filter platform/genre |
| Hub game | `/games/{slug}` | Metadata, skor resmi, feed artikel terkait |
| Hasil pencarian | `/search?q=` | Full-text OpenSearch (Fase 6) |

Admin (`admin_portal`):
| Halaman | Untuk siapa |
|---|---|
| Login, Dashboard | semua role |
| Artikel: list/create/edit/publish | Editor ke atas |
| Game/Tag/Author: list/create/edit | Admin ke atas |
| User management | Super Admin |

**Taksonomi — final (2026-09-04, detail di [[00-Decisions-Log]]):** 3 sumbu —
`ArticleType` (jenis konten), Platform (enum field di `Game`), `Tag`
(genre/topik, M:N ke `Game`). `Category` generik **dibuang**, fungsinya
diwakili `ArticleType`.

**Wireframe homepage — final (2026-09-04, detail di [[00-Decisions-Log]]):**
Featured **Game** (kurasi editorial, bukan featured Article), carousel
review terbaru, artikel non-review list kronologis biasa.

**Wireframe detail artikel — final:** judul, featured image, author,
tanggal, `ArticleType` badge, isi, platform & game terkait, meta SEO,
khusus REVIEW (skor + Pros/Cons/Verdict), slot iklan in-content, placeholder
komentar.

**X1 SELESAI** — sitemap, jenis halaman, taksonomi, wireframe semua tuntas.

---

## X2 — Content Model / ERD

**Status:** SELESAI (2026-09-04)

**Selesai jika:** ERD lengkap dengan semua field, dan tiap keputusan relasi punya
satu kalimat alasan di [[00-Decisions-Log]].

**Blocker:** ~~butuh X4 batch 1 (penyimpanan gambar) selesai dulu~~ — sudah
kebuka 2026-09-04, penyimpanan gambar diputuskan S3 (lihat [[00-Decisions-Log]]).

`content_service/grails-app/domain` sekarang kosong. Draft pertama (8 domain
class) sudah dihapus 2026-09-02 karena ditulis sebelum tahap ini selesai — lihat
[[00-Decisions-Log]] dan catatan di [[01-Tahap1-Discovery]]. Jangan menganggap
checklist di bawah sebagai "yang tersisa dari draft lama" — ini target field
untuk rancangan yang benar-benar baru, disusun dari X1 (IA) dan X4 batch 1, bukan
revisi dari yang sudah dihapus. Yang sudah diketahui perlu ada, dari tinjauan
draft pertama kemarin:

### Checklist

- [x] Featured image di `Article` — S3 (X4 batch 1, diputuskan 2026-09-04)
- [x] Excerpt/lead di `Article` — `String excerpt`, maxSize ~300
- [x] Meta SEO: `metaTitle`/`metaDescription`/`ogImage`/`canonicalUrl`,
      nullable dengan fallback
- [x] `dateCreated` / `lastUpdated` — dijawab lewat plugin `audit-logging`
      penuh (lebih dari sekadar dua field ini), lihat [[00-Decisions-Log]]
- [x] Keputusan `Category`: **dibuang** — diputuskan 2026-09-04 (X1),
      fungsinya diwakili `ArticleType`. Detail [[00-Decisions-Log]].
- [x] Keputusan `Review`: tetap 1:1 `Article` — konsekuensinya bukan
      agregasi skor, tapi `Game` dapat `gameType`/`baseGame` (ala
      `pim.Product` CCWR). Detail [[00-Decisions-Log]].
- [x] `Game`: genre sudah kejawab lewat `Tag` (taksonomi X1) — tambah
      `description` (String, teks panjang). `gameType`/`baseGame` juga
      ditambah, lihat item di atas.
- [x] `Tag.slug maxSize` — diturunin ke 255
- [x] `ArticleGame` — tambah `unique: ['article', 'game']`
- [x] `Author` — tambah `slug` + `avatar`
- [x] ERD digambar (boleh sederhana — tabel + panah cukup)

### Catatan kerja

**Batch field & audit trail — final (2026-09-04, detail di
[[00-Decisions-Log]]):** lihat checklist di atas untuk nilai tiap field.
Audit trail pakai plugin `audit-logging` penuh ala CCWR (`AuditFields`/
`Auditable`), bukan cuma `dateCreated`/`lastUpdated`.

**`Game.gameType`/`baseGame` — final (2026-09-04, detail di
[[00-Decisions-Log]]):** ala `pim.Product` CCWR (`variantOf`/`kitOf` →
disederhanakan jadi satu `baseGame`). `gameType`: `MAIN`/`EDITION`/`DLC`.
Tiap edition/DLC = baris `Game` sendiri dengan `Review` 1:1 sendiri —
gak ada agregasi skor.

**ERD final:**

```mermaid
erDiagram
    ARTICLE ||--o| REVIEW : "punya (kalau tipe REVIEW)"
    ARTICLE ||--o{ ARTICLE_GAME : "tautan"
    GAME ||--o{ ARTICLE_GAME : "tautan"
    ARTICLE }o--|| AUTHOR : "ditulis oleh"
    GAME }o--o{ TAG : "genre"

    ARTICLE {
        Long id
        String title
        String slug
        String excerpt
        String body
        ArticleType articleType
        PublishStatus status
        String featuredImage
        String metaTitle
        String metaDescription
        String ogImage
        String canonicalUrl
        Date publishedDate
    }
    REVIEW {
        Long id
        Article article FK_unique
        BigDecimal score
        String pros
        String cons
        String verdict
    }
    ARTICLE_GAME {
        Long id
        Article article FK
        Game game FK
        Boolean isPrimary
    }
    GAME {
        Long id
        String title
        String slug
        String description
        String developer
        String publisher
        Date releaseDate
        GameType gameType
        Game baseGame FK_nullable_self
    }
    TAG {
        Long id
        String name
        String slug
    }
    AUTHOR {
        Long id
        String name
        String slug
        String avatar
    }
```

Catatan yang gak masuk sintaks ER standar: `Game.baseGame` itu self-referencing FK
(`EDITION`/`DLC` nunjuk balik ke `MAIN`-nya) — lihat [[00-Decisions-Log]] 2026-09-04.
Platform gak digambar sebagai entitas — dia enum/set field langsung di `Game`
(tabel `game_platforms` implisit dari GORM `hasMany` enum), bukan tabel taksonomi
terpisah kayak `Tag`. `Comment`/`BadWord` sengaja **tidak** masuk ERD ini — keduanya
fitur ditunda (lihat [[00-Slice-Backlog]]), didesain detail pas dikerjakan nanti,
bukan sekarang biar ERD Fase 2 tetap fokus ke yang benar-benar dibangun duluan.

**X2 SELESAI** — semua checklist tuntas.

---

## X3 — Kontrak API

**Status:** SELESAI (2026-09-04)

**Selesai jika:** `admin_portal` dan `web_portal` bisa dibangun dari dokumen ini
tanpa membaca kode `content_service`.

### Checklist

- [x] Daftar endpoint (list artikel, detail by slug, CRUD admin, dst)
- [x] Bentuk request/response tiap endpoint
- [x] Format error yang konsisten
- [x] Aturan pagination
- [x] Aturan versioning (`/api/v1/...` atau tidak — lihat daftar-keputusan #7)

### Catatan kerja

**Kontrak final (2026-09-04, detail & alasan lengkap di [[00-Decisions-Log]]):**

*Versioning:* tanpa prefix `/v1` — `/api/articles`.

*Endpoint publik:*
| Endpoint | Fungsi |
|---|---|
| `GET /api/articles?type=&platform=&game=&max=&offset=` | Listing, filter opsional |
| `GET /api/articles/{slug}` | Detail — 404 JSON kalau slug gak ada |
| `GET /api/games?platform=&genre=&max=&offset=` | Listing game |
| `GET /api/games/{slug}` | Hub game — skor, artikel terkait, edition/DLC |

*Endpoint admin (Fase 3):* CRUD standar (`POST`/`PUT`/`DELETE`) di
`/api/articles`, `/api/games`, `/api/tags`, `/api/authors` — auth detail
nunggu X4.

*Response envelope:*
```json
{ "data": [...], "meta": { "total": 42, "max": 20, "offset": 0 } }  // list
{ "data": {...} }                                                    // detail
{ "error": { "code": "ARTICLE_NOT_FOUND", "message": "..." } }       // error
```
`data` selalu DTO `shared_lib`, gak pernah domain class langsung (S2-4).

*Pagination:* `max`/`offset`, default `max=20`, hard cap `max=100`.

*Error:* selalu JSON `code`+`message`, status HTTP benar, gak pernah HTML
Grails default.

*HTTP method `QUERY`:* dipertimbangkan, **tidak dipakai** — CCWR nihil,
filter proyek ini cukup sederhana, Grails 6.2.3 kemungkinan gak dukung
tanpa konfigurasi custom.

**X3 SELESAI.**

---

## X4 — Keputusan Teknologi

**Status:** SELESAI (2026-09-04) — 8 dari 10 diputuskan (batch 1 & 2 tuntas);
2 sisanya (deployment, testing) sengaja **Ditunda sampai dibutuhkan**, lihat
bagian di bawah — bukan blocker Tahap 2.

Daftar lengkap 10 keputusan, hasil grep langsung ke CCWR, dan urutan pembahasan
yang disarankan ada di
`.claude/skills/planning/references/daftar-keputusan.md`. Jangan duplikasi
isinya di sini — file ini cuma menandai progres.

### Batch 1 — menghambat X2, M1, M2 (bahas duluan)

- [x] Penyimpanan gambar — **S3**, ikut pola CCWR (`AmazonS3Service`,
      `MediaProcessingService`). Diputuskan 2026-09-03, detail di
      [[00-Decisions-Log]]. Ini membuka blocker X2 "Featured image".
- [x] Autentikasi admin — Spring Security Core, 7/10 domain CCWR
      (`User`/`Role`/`UserRole`/`Permission`/`PermissionRole`/`UserToken`/
      `UserPasswordHistory`), `admin_portal` dapat DB sendiri, password
      expiry 90 hari, email lewat MailHog lokal. Detail [[00-Decisions-Log]].
- [x] Editor teks di admin — **WYSIWYG generik**, sengaja bukan replikasi
      `content-builder` CCWR (block-builder, gak proporsional)
- [x] Audit trail — **plugin `audit-logging` penuh** ala CCWR, diputuskan
      2026-09-04. Detail [[00-Decisions-Log]].

### Batch 2 — boleh sesudah X2

- [x] Migrasi database — **Liquibase**, sengaja **tidak** ikut CCWR
      (CCWR gak pakai migrasi sama sekali — `dbCreate: update` gak layak
      produksi). Detail [[00-Decisions-Log]].
- [x] Frontend `web_portal` — **GSP**, konsisten CCWR
- [x] Versioning API — tanpa prefix `/v1`, diputuskan lewat X3 (2026-09-04)
- [x] Multi-site & multi-bahasa — struktur tabel `Site`/`Language`/
      `SiteLanguage` dipertahankan, diisi 1 baris. Diputuskan 2026-09-03,
      detail di [[00-Decisions-Log]].

### Ditunda sampai dibutuhkan

- [ ] Target deployment (dibahas di akhir M4)
- [ ] Strategi testing (dibahas bareng mentoring F-07)

---

## Selesai jika (Tahap 2 keseluruhan)

X1, X2, X3 selesai, dan X4 batch 1 minimal selesai (batch 2 boleh menyusul sambil
Build jalan, karena tidak memblokir M1/M2). Begitu tercapai, lanjut ke
[[03-Tahap3-Build]].
