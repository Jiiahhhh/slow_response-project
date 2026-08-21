# Roadmap Backlog — Fase 1 sampai 7

Fase 0 punya file checklist sendiri: [[01-Fase-0-Checklist]]. Mulai dari sini
setelah Fase 0 tercentang semua.

Scope terkunci ke **3 modul**: `web_portal`, `admin_portal`, `content_service`
(rename dari konsep `corpweb` di CCWR), plus `shared_lib` sebagai plugin
pendukung. Lihat [[04-Decisions-Log]] untuk alasannya.

Cara pakai: kerjakan satu fase sampai kriteria "Selesai jika" tercapai, baru pindah.
Jangan cicil beberapa fase sekaligus — proyek latihan yang paling sering mandek
adalah yang lompat-lompat fase karena bosan sebelum satu bagian benar-benar solid.

---

## Fase 1 — Modul `shared_lib`

- [ ] `grails create-plugin shared_lib` (bukan `create-app` — ini plugin, bukan
      aplikasi berdiri sendiri, seperti di CCWR)
- [ ] Daftarkan di `settings.gradle` root
- [ ] Tambahkan sebagai dependency di `build.gradle` ketiga modul lain
- [ ] Buat enum `ArticleType` (isinya nanti hasil dari [[03-Concept-Worksheet]] —
      minimal NEWS, REVIEW, GUIDE)
- [ ] Buat enum `PublishStatus` (DRAFT, PUBLISHED, ARCHIVED)
- [ ] Buat DTO kosong `ArticleResponse` (isi field-nya nanti seiring Fase 2)

**Selesai jika:** ketiga modul lain bisa `import shared_lib.enums.ArticleType` dan
tetap compile.

---

## Fase 2 — `content_service` sebagai write side

Ini fase paling berat secara desain — di sinilah [[03-Concept-Worksheet]] harus
sudah terisi, karena kamu butuh tahu field apa saja yang perlu ada di `Article`
dan `Game` sebelum menulis domain class.

- [ ] Ganti datasource H2 → MySQL (`mysql-connector-j`, arahkan ke
      `localhost:3310`, database `capstone_db`)
- [ ] Rancang domain `Article` — field minimal: title, slug, body, type
      (`ArticleType`), status (`PublishStatus`), publishedDate, author
- [ ] Rancang domain `Game` — title, platform(s), developer, publisher,
      releaseDate
- [ ] Rancang domain `Category`, `Tag`, `Author`
- [ ] Relasi `Article` ↔ `Game` (many-to-many — satu artikel bisa bahas
      beberapa game, satu game bisa dibahas banyak artikel)
- [ ] Kalau konsep sudah menentukan ada skor review: domain `Review` terpisah
      atau field di `Article` — putuskan & catat di Decisions Log
- [ ] Controller REST: list artikel (dengan filter type/category), detail
      artikel by slug

**Selesai jika:** `curl localhost:9089/articles` mengembalikan JSON dari MySQL,
bukan H2.

---

## Fase 3 — `admin_portal` CRUD

- [ ] Scaffold dulu (`grails generate-all`) untuk `Article` dan `Game` — biar
      cepat kelihatan hasilnya, rapikan belakangan
- [ ] Feign client ke `content_service` (URL boleh hardcoded di fase ini,
      Consul menyusul di Fase 5)
- [ ] Alur publish: draft → published (minimal transisi status, belum perlu
      versioning serumit CCWR)

**Selesai jika:** artikel bisa dibuat dan di-publish lewat UI admin, dan langsung
kelihatan lewat endpoint `content_service` di Fase 2.

---

## Fase 4 — `web_portal` konsumsi API — MILESTONE UTAMA

- [ ] **Cabut** dependency `hibernate5` dan H2 dari `web_portal/build.gradle` —
      modul ini gak boleh punya datasource sendiri, meniru pola CCWR
      (`web_portal` di CCWR: 0 domain class, 0 datasource)
- [ ] Feign client baca dari `content_service`
- [ ] Homepage — daftar artikel terbaru
- [ ] Halaman detail artikel
- [ ] Halaman listing per kategori/tipe (News / Review / Guide)

**Selesai jika:** alur end-to-end jalan — publish artikel di admin_portal, muncul
di web_portal. **Kalau waktu/energi habis di titik ini, proyek tetap layak
ditunjukkan sebagai capstone.** Fase 5–7 di bawah ini nilai tambah, bukan syarat.

---

## Fase 5 — Consul (opsional, nilai tambah)

- [ ] Daftarkan `content_service` ke Consul saat startup
- [ ] Ganti Feign client di `admin_portal`/`web_portal` dari URL hardcoded jadi
      `@FeignClient(name = "content-service")` yang di-resolve Consul

---

## Fase 6 — OpenSearch (opsional, paling mirip CCWR)

- [ ] Index artikel ke OpenSearch saat `content_service` publish
- [ ] `web_portal` baca listing/search dari OpenSearch, bukan lagi langsung
      Feign ke `content_service`
- [ ] (Kalau sempat) index `preview_articles` terpisah untuk preview draft

---

## Fase 7 — Hazelcast, lalu Kafka (opsional)

- [ ] Hazelcast near-cache di `web_portal` untuk hasil baca yang sering diakses
- [ ] Kafka untuk hal yang genuinely butuh event antar-service (misal: komentar
      pembaca) — ikuti cakupan sempit seperti CCWR, jangan dipakai untuk
      indexing konten (itu tugas OpenSearch langsung)
