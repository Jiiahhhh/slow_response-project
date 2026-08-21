# CLAUDE.md

Konteks proyek untuk Claude Code — dan catatan rujukan untuk pemilik proyek.

Ini file arsitektur/referensi. Untuk task harian, checklist, dan konsep produk
yang masih digodok, lihat **[docs/vault/](docs/vault/00-Index.md)**.

---

## Cara kerja di repo ini (PENTING)

Proyek ini adalah **latihan pribadi**. Pemilik repo sengaja menulis kodenya sendiri
tanpa bantuan AI, untuk mengukur kemampuannya sebagai software developer.

Aturan untuk Claude:

- **Jangan menulis atau mengubah kode aplikasi kecuali diminta eksplisit.**
- Peran default Claude di sini: menjelaskan konsep, menjawab pertanyaan arsitektur,
  mereview kode yang sudah ditulis pemilik, dan menunjukkan letak pola tertentu di
  proyek rujukan CCWR.
- Kalau pemilik bertanya "bagaimana caranya X", jawab dengan penjelasan dan arah,
  bukan dengan langsung menulis implementasinya.
- Perbaikan konfigurasi/tooling (Gradle, Docker, .gitignore) boleh dikerjakan jika
  diminta — itu bukan bagian dari latihan.

---

## Tujuan proyek

Membangun **situs konten game** (referensi rasa produk: The Lazy Monday, IGN) —
artikel news / review / guide / feature, dengan database game.

Tapi tujuan sebenarnya bukan produknya. Tujuannya adalah **mereplikasi pola arsitektur
CCWR** (proyek Canon Corporate Website Revamp di kantor) dalam skala kecil, supaya
pemilik memahami sistem yang sedang ia kerjakan secara profesional.

Jadi: **arsitektur lebih penting daripada fitur.** Sebuah situs dengan 3 jenis artikel
tapi arsitekturnya benar jauh lebih bernilai daripada situs dengan 20 fitur yang
monolitik.

---

## Proyek rujukan: CCWR

Lokasi: `/Users/ilal/canon-corporate-website-revamp` (read-only, jangan diubah)

### Struktur modul CCWR

| Modul | Domain | Controller | Service | GSP | DB |
|---|---|---|---|---|---|
| `web_portal` (:8080) | **0** | 45 | 49 | 377 | **tidak ada** |
| `admin_portal` | 20 | 84 | 48 | 410 | `canon_admin_portal` |
| `api` | 17 | 14 | 13 | 4 | `canon_admin_portal` |
| `services:corpweb` | 205 | 54 | 99 | 64 | `canon_corpweb` |
| `services:pim` | 72 | 25 | 56 | 0 | `canon_pim` |
| `services:sims` | 62 | 15 | 44 | 0 | `canon_sims` |
| `shared_lib` | – | – | 2 | – | Grails **plugin**, bukan app |

### Alur data CCWR

```
admin_portal (editor input konten)
      | tulis
      v
services:corpweb / pim / sims  --->  MySQL          (write side)
      | index
      v
                OpenSearch                          (read side)
      | baca
      v
   web_portal  -->  Hazelcast near-cache  -->  render GSP
      \
       `--> Feign client (via Consul) --> services:*  (untuk aksi write)
```

**Inti yang harus dipahami:** `web_portal` tidak punya domain class dan tidak punya
datasource sama sekali. Halaman publik tidak pernah JOIN ke MySQL saat request.
Ini read/write split.

### Pola CCWR yang layak direplikasi

1. **`shared_lib` sebagai lapisan kontrak.** Grails plugin berisi enum, DTO
   request/response, Kafka template, OpenSearch client trait, `HazelcastService`.
   Ini yang membuat portal bisa bicara ke service tanpa saling import domain class.

2. **Feign + Consul.** Client dideklarasikan sebagai interface, tanpa URL hardcoded:
   ```groovy
   @FeignClient(name = "services-corpweb", path = "/cps-renewal/")
   interface CpsRenewalClient { ... }
   ```
   Rujukan: `web_portal/src/main/groovy/web_portal/http/client/`

3. **Multi-site & multi-bahasa di level data.** Ada `Site`, `Language`,
   `SiteLanguage`, dan tabel `*Translation`. Index OpenSearch dipecah per site+bahasa:
   `articles__{siteId}__{lang}`.
   Rujukan: `shared_lib/src/main/groovy/shared_lib/opensearch/OpenSearchIndex.groovy`

4. **Draft / Publish / Preview + versioning.** `corpweb.Page` punya `status`
   (DRAFT / RELEASE / PUBLISHED / ARCHIVED) dan tiga self-reference: `parent`
   (terjemahan), `master` (copy asli), `origin` (shadow/draft). Preview pakai index
   OpenSearch terpisah `preview_articles`.
   Rujukan: `services/corpweb/grails-app/domain/corpweb/Page.groovy`

5. **Kafka dipakai sempit.** Topic enum CCWR hanya `CORPWEB_DISCUSSION`,
   `SIMS_DISCUSSION`, `PIM_DISCUSSION`, plus event user & accessTag. Kafka untuk
   sinkronisasi user dan komentar antar-service — **bukan** untuk indexing konten.

6. **Audit trail.** Hampir semua domain `extends AuditFields implements Auditable`,
   memakai plugin `audit-logging`.

### Anti-pola CCWR — jangan ditiru

`web_portal/grails-app/conf/application.yml` di CCWR memuat AWS access key, Adyen API
key & HMAC key, serta reCAPTCHA secret key dalam plaintext dan ter-commit ke git.
**Di proyek ini, semua secret wajib lewat environment variable sejak awal.**

---

## Pemetaan CCWR ke situs game

| CCWR | Proyek ini |
|---|---|
| `corpweb.Page` (+ `PageDetail`, `PageComponent`) | `Article` — news / review / guide / feature |
| `corpweb.ContentType` | `ArticleType` |
| `pim.Product` (+ spec, media, kategori) | `Game` — judul, platform, developer, publisher, tanggal rilis |
| `services:sims` | **dibuang** — terlalu Canon-spesifik, tidak ada analoginya |
| `Site` / `Language` | boleh disederhanakan ke satu site, tapi pertahankan konsepnya |
| `api` | opsional, prioritas terendah |

Yang khas situs game dan **tidak ada** di CCWR — ini bagian orisinal yang perlu
dirancang sendiri:

- Skor review (dan agregasinya per game)
- Relasi `Article` ↔ `Game` (satu artikel bisa membahas beberapa game)
- Platform / genre sebagai dimensi filter utama

---

## Kondisi saat ini

Per commit `0e81bc5` ("feat: initial project setup"), branch `develop`:

**Sudah ada:**
- Tiga modul Grails hasil `grails create-app`, belum disentuh isinya
- Konfigurasi port di ketiga `application.yml`
- `docker/docker-compose.yml` — MySQL, OpenSearch + Dashboard, Kafka + Zookeeper,
  Hazelcast, Consul (semua versi ter-pin, ARM64)

**Belum ada:**
- Domain class (nol, di semua modul — folder `grails-app/domain/` belum dibuat)
- Controller (selain `UrlMappings` bawaan), service, view kustom
- Modul `shared_lib`
- Dependency apapun untuk MySQL, OpenSearch, Hazelcast, Kafka, Feign, Consul
- Seed data (`BootStrap.groovy` masih kosong)

Infrastruktur di docker-compose saat ini menyala tanpa ada yang memakainya.

---

## Peta modul proyek ini

| Modul | Port | Peran (target) |
|---|---|---|
| `web_portal` | 9080 | frontend publik, tanpa datasource |
| `admin_portal` | 9081 | CMS editor + auth |
| `content_service` | 9089 | write side, pemilik MySQL |
| `shared_lib` | – | **belum ada** — Grails plugin, lapisan kontrak |

### Port infrastruktur (docker-compose)

Sengaja digeser dari port default agar tidak bentrok dengan CCWR yang jalan bersamaan.

| Service | Port |
|---|---|
| MySQL | 3310 |
| OpenSearch | 9201 (+ 9601) |
| OpenSearch Dashboard | 5602 |
| Zookeeper | 2182 |
| Kafka | 9093 |
| Hazelcast | 5702 |
| Consul | 8501 (+ 8601/udp) |

Kredensial dev: db `capstone_db`, user `capstone_user` / `capstone_pass`,
root password `rootpassword`.

---

## Stack & versi

- Grails **6.2.3**, grails-gradle-plugin 6.2.4
- Gradle **7.6.4** (wrapper per modul)
- Java **17** (Corretto 17 terpasang di mesin)
- MySQL 8.0.28, OpenSearch 2.19.0, Kafka 7.8.1, Hazelcast 5.5.0, Consul 1.20

Menyamai stack CCWR, kecuali grails-gradle-plugin (CCWR pakai 6.2.3).

---

## Masalah yang perlu dibereskan lebih dulu

Tiga hal ini akan menghambat begitu mulai coding serius:

1. **`setting.gradle` salah nama.** Gradle mencari `settings.gradle` (dengan "s").
   Akibatnya baris `include 'web_portal'` dst. tidak pernah terbaca dan build
   multi-project dari root tidak berfungsi.

2. **Konflik versi Gradle.** Wrapper root = 9.3.0, wrapper tiap modul = 7.6.4.
   Grails 6.2.3 tidak kompatibel dengan Gradle 9.

3. **`.gradle/` ter-commit.** 12 file lock dan cache biner masuk ke git; tidak ada
   `.gitignore` di root.

Selain itu: `sourceCompatibility` masih 11 padahal target Java 17, dan branch
`develop` belum pernah di-push (remote hanya punya `main`).

---

## Roadmap

Fase 0–4 adalah inti. Fase 5–7 adalah nilai tambah — kerjakan satu per satu,
jangan diborong.

### Fase 0 — Perbaiki fondasi
Rename `setting.gradle` ke `settings.gradle`. Samakan wrapper root ke 7.6.4 (atau
hapus wrapper root dan tetap build per modul). Tambah `.gitignore` root dan
`git rm -r --cached .gradle`. Naikkan `sourceCompatibility` ke 17. Commit pekerjaan
docker-compose + port yang masih menggantung.

*Selesai jika:* `./gradlew projects` dari root menampilkan ketiga modul.

### Fase 1 — Modul `shared_lib`
Buat sebagai Grails plugin (`grails create-plugin`), bukan app. Daftarkan di
`settings.gradle` dan jadikan dependency ketiga modul lain. Isi awal cukup minimal:
enum `ArticleType`, enum `PublishStatus`, DTO `ArticleResponse`.

*Selesai jika:* ketiga modul bisa meng-import kelas dari `shared_lib` dan tetap
compile.

### Fase 2 — `content_service` sebagai write side
Ganti H2 ke MySQL (dependency `mysql-connector-j`, datasource ke `localhost:3310`).
Rancang domain inti: `Article`, `Game`, `Category`, `Tag`, `Author`, plus relasi
`Article` ↔ `Game`. Sertakan field `status` (DRAFT / PUBLISHED / ARCHIVED) sejak awal
— jangan ditambahkan belakangan. Expose REST controller untuk list dan detail artikel.

*Selesai jika:* `curl` ke endpoint mengembalikan artikel dari MySQL.

### Fase 3 — `admin_portal` CRUD
Pakai scaffolding Grails dulu supaya cepat kelihatan hasilnya. Tambahkan Feign client
ke `content_service` (URL masih boleh hardcoded di fase ini).

*Selesai jika:* artikel bisa dibuat dan dipublikasikan lewat UI admin.

### Fase 4 — `web_portal` konsumsi API
**Cabut `hibernate5` dan H2 dari `web_portal`** — modul ini tidak boleh punya
datasource, meniru CCWR. Baca via Feign client. Bangun homepage dan halaman detail
artikel.

*Selesai jika:* sistem jalan end-to-end — editor publish di admin, artikel muncul di
web portal. **Ini milestone terpenting.** Kalau waktu habis di sini, proyek tetap
layak ditunjukkan.

### Fase 5 — Consul
Daftarkan tiap service ke Consul, ganti URL hardcoded jadi service discovery
(`@FeignClient(name = "content-service")`).

### Fase 6 — OpenSearch
Index artikel dari `content_service` saat publish. Ubah `web_portal` agar listing dan
search membaca dari OpenSearch, bukan Feign. Ini bagian yang paling menyerupai CCWR.
Kalau sempat, tambahkan index `preview_*` terpisah.

### Fase 7 — Hazelcast, lalu Kafka
Hazelcast near-cache di `web_portal` dengan TTL per jenis konten. Kafka untuk
komentar / sinkronisasi user antar-service — ikuti cakupan sempit seperti CCWR.

---

## Konvensi

- Bahasa commit: Conventional Commits (`feat:`, `fix:`, `chore:`)
- Branch: `develop` sebagai basis, `feature/<nama>` untuk pekerjaan
- Package: nama modul sebagai root package (`web_portal`, `admin_portal`,
  `content_service`, `shared_lib`) — mengikuti pola CCWR
- Secret: environment variable, tidak pernah plaintext di `application.yml`
