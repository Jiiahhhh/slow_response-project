# Tahap 2 — Design

**Status keseluruhan:** SEDANG BERJALAN, belum ada yang tuntas

Ini tracker kerja untuk Tahap 2. Alasan tahap ini ada — cetak biru sebelum kode
ditulis, dan kenapa kesalahan di sini paling mahal diperbaiki belakangan — ada di
[[00-Grand-Plan]]. File ini menjawab: *sejauh mana Tahap 2 ini selesai.*

Prasyarat: [[01-Tahap1-Discovery]] selesai dulu. X2 (model data) juga butuh
sebagian X4 (keputusan teknologi) selesai lebih dulu — urutan sebenarnya:
**X1 → X4 batch 1 → X2 → X3 → X4 batch 2.** Lihat bagian urutan di bawah.

Dipakai lewat `/planning`.

---

## X1 — Information Architecture

**Status:** BELUM

**Selesai jika:** kamu bisa menyebut semua jenis halaman dan apa isinya.

### Checklist

- [ ] Sitemap — semua halaman publik dan admin
- [ ] Daftar jenis halaman (homepage, listing, detail artikel, halaman game, dst)
- [ ] Taksonomi: kategori vs tag vs platform vs genre — bedanya apa, kapan pakai
      yang mana
- [ ] Wireframe kasar homepage + halaman artikel (coretan tangan cukup — yang
      penting elemen apa saja yang wajib ada, karena itu menentukan field di
      model data)

### Catatan kerja

_(diisi saat sesi berjalan)_

---

## X2 — Content Model / ERD

**Status:** BELUM — mulai dari nol

**Selesai jika:** ERD lengkap dengan semua field, dan tiap keputusan relasi punya
satu kalimat alasan di [[00-Decisions-Log]].

**Blocker:** butuh X4 batch 1 (penyimpanan gambar) selesai dulu — bentuk field
gambar di `Article` bergantung pada keputusan itu.

`content_service/grails-app/domain` sekarang kosong. Draft pertama (8 domain
class) sudah dihapus 2026-09-02 karena ditulis sebelum tahap ini selesai — lihat
[[00-Decisions-Log]] dan catatan di [[01-Tahap1-Discovery]]. Jangan menganggap
checklist di bawah sebagai "yang tersisa dari draft lama" — ini target field
untuk rancangan yang benar-benar baru, disusun dari X1 (IA) dan X4 batch 1, bukan
revisi dari yang sudah dihapus. Yang sudah diketahui perlu ada, dari tinjauan
draft pertama kemarin:

### Checklist

- [ ] Featured image di `Article` (bentuknya bergantung X4 batch 1)
- [ ] Excerpt/lead di `Article`
- [ ] Meta SEO: meta description, OG image, canonical URL
- [ ] `dateCreated` / `lastUpdated` di semua domain (terkait X4 — audit trail)
- [ ] Keputusan `Category`: dipakai atau dibuang? (disebut di roadmap lama, tidak
      ada di domain sekarang)
- [ ] Keputusan `Review`: tetap 1:1 dengan `Article`, atau ada konsekuensi ke
      agregasi skor per `Game`?
- [ ] `Game`: tambah genre dan deskripsi (disebut CLAUDE.md sebagai dimensi
      filter utama, belum ada di domain)
- [ ] `Tag.slug maxSize` — sekarang 65535, kemungkinan salah ketik dari `Tag.name`
- [ ] `ArticleGame` — belum ada constraint unique untuk pasangan (article, game)
- [ ] `Author` — perlu slug dan avatar untuk halaman "artikel oleh X"?
- [ ] ERD digambar (boleh sederhana — tabel + panah cukup)

### Catatan kerja

_(diisi saat sesi berjalan)_

---

## X3 — Kontrak API

**Status:** BELUM

**Selesai jika:** `admin_portal` dan `web_portal` bisa dibangun dari dokumen ini
tanpa membaca kode `content_service`.

### Checklist

- [ ] Daftar endpoint (list artikel, detail by slug, CRUD admin, dst)
- [ ] Bentuk request/response tiap endpoint
- [ ] Format error yang konsisten
- [ ] Aturan pagination
- [ ] Aturan versioning (`/api/v1/...` atau tidak — lihat daftar-keputusan #7)

### Catatan kerja

_(diisi saat sesi berjalan)_

---

## X4 — Keputusan Teknologi

**Status:** BELUM — 0 dari 10 keputusan terbuka sudah diputuskan

Daftar lengkap 10 keputusan, hasil grep langsung ke CCWR, dan urutan pembahasan
yang disarankan ada di
`.claude/skills/planning/references/daftar-keputusan.md`. Jangan duplikasi
isinya di sini — file ini cuma menandai progres.

### Batch 1 — menghambat X2, M1, M2 (bahas duluan)

- [ ] Penyimpanan gambar (S3 / MinIO / filesystem lokal)
- [ ] Autentikasi admin (Spring Security Core — seberapa banyak ditiru dari CCWR)
- [ ] Editor teks di admin (WYSIWYG / Markdown)
- [ ] Audit trail (plugin penuh / `dateCreated`+`lastUpdated` saja)

### Batch 2 — boleh sesudah X2

- [ ] Migrasi database (Liquibase / Flyway / tetap `dbCreate: update`)
- [ ] Frontend `web_portal` (GSP, konsisten dengan CCWR)
- [ ] Versioning API
- [ ] Multi-site & multi-bahasa — sejauh mana konsepnya dipertahankan

### Ditunda sampai dibutuhkan

- [ ] Target deployment (dibahas di akhir M4)
- [ ] Strategi testing (dibahas bareng mentoring F-07)

---

## Selesai jika (Tahap 2 keseluruhan)

X1, X2, X3 selesai, dan X4 batch 1 minimal selesai (batch 2 boleh menyusul sambil
Build jalan, karena tidak memblokir M1/M2). Begitu tercapai, lanjut ke
[[03-Tahap3-Build]].
