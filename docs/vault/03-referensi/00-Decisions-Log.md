# Decisions Log

Catatan tiap keputusan yang gak trivial. Format singkat, gak perlu formal —
tujuannya supaya 3 bulan lagi kamu (atau siapapun baca repo ini) tahu **kenapa**
sesuatu dibuat begitu, bukan cuma apa yang dibuat.

Tambahkan entry baru di atas (terbaru duluan). Template:

```
## [tanggal] Judul keputusan singkat

**Konteks:** situasi/masalah yang memicu keputusan ini
**Opsi yang dipertimbangkan:** apa saja pilihannya
**Keputusan:** apa yang dipilih
**Alasan:** kenapa
```

---

## 2026-09-03 Peran pengguna (D1) — 4 peran, Admin sengaja belum dipecah

**Konteks:** D1 (Product Brief) butuh peran pengguna terdefinisi sebelum X1/X2/X3
bisa mulai. Awalnya diusulkan 1 peran gabungan "Editor/Admin" (alasan: satu
orang ngerjain semuanya, hindari kompleksitas RBAC tanpa skenario nyata untuk
mengujinya). Ditolak — proyek ini soal belajar arsitektur CCWR, bukan
efisiensi produk, dan alokasi role adalah pola yang eksplisit mau dipelajari.

**Opsi yang dipertimbangkan:**
- 1 peran gabungan Editor/Admin (efisien, tapi nggak ada nilai belajar RBAC)
- 2 peran (Editor vs Admin)
- 3 peran (Editor / Admin / Super Admin) dengan Admin sebagai wadah sementara
  untuk hal yang nanti dipecah lebih halus (per master data: Game/Category/Tag)

**Keputusan:** 4 peran total — **Pembaca** (publik, tanpa login, bukan role
`admin_portal`), **Editor** (create/edit/publish artikel — satu wewenang
gabungan, belum dipecah create-only vs publish-only), **Admin** (Editor +
kelola master data `Game`/`Category`/`Tag`/`Author` — placeholder, nanti
dipecah per master data), **Super Admin** (Admin + kelola akun user + akses
semua modul tanpa kecuali). Editor **publish langsung**, tidak ada gate
persetujuan — status artikel tetap 3 tingkat (`DRAFT`/`PUBLISHED`/`ARCHIVED`)
seperti yang sudah dipatok roadmap, bukan 4 tingkat ala `RELEASE` CCWR.

**Alasan:** dicek langsung ke CCWR (`admin_portal/grails-app/domain/{Role,
Permission,UserRole,PermissionRole}.groovy`, `shared_lib/.../RoleConstant.groovy`)
— polanya permission-table (Spring Security Core), granular per-modul/region,
bukan role hardcoded dua tingkat. 3 role eksplisit + Admin sebagai wadah
sementara meniru semangat itu dalam skala kecil: struktur permission-based
disiapkan sekarang, granularitas ditambah belakangan tanpa ubah arsitektur.
Mekanisme implementasinya (tabel `Role`/`Permission` vs `enum role` di-hardcode)
sengaja **belum diputuskan di sini** — itu keputusan X4, bukan D1.

**Dampak ke tracker:** [[01-Tahap1-Discovery]] D1 checklist item 1 selesai.
[[00-Slice-Backlog]] "Belum dijadwalkan" nambah item: X4 perlu putuskan
mekanisme role (tabel Role/Permission ala CCWR vs enum sederhana), dipicu oleh
rencana pecah Admin jadi role per master data.

---

## 2026-09-03 Komentar pembaca — fitur orisinal (bukan pola CCWR), auto-publish + filter tolak-atau-sensor

**Konteks:** D1 checklist item 2 ("yang TIDAK dibuat") awalnya menganggap
komentar termasuk yang dibuang, dengan asumsi CCWR punya pola `Discussion`
yang bisa dicontek. Dicek langsung ke kode: `Discussion` di CCWR
(`services/corpweb/grails-app/domain/corpweb/Discussion.groovy`,
`admin_portal/grails-app/controllers/admin_portal/DiscussionController.groovy`)
ternyata diskusi **internal antar-staf CMS** (`@Secured("isFullyAuthenticated()")`,
`DiscussionType`: PAGE/PRODUCT/PROMOTION/dst — nempel ke item konten, bukan
komentar pembaca publik). `web_portal` (situs publik) sama sekali gak
menyentuh `Discussion`. Jadi komentar pembaca **gak ada rujukan CCWR** —
ini fitur orisinal, kategori sama seperti skor review & relasi Article↔Game
(lihat CLAUDE.md bagian "khas situs game, tidak ada di CCWR").

**Opsi yang dipertimbangkan:**
- Dibuang dari scope sama sekali vs dibangun sebagai fitur orisinal (di luar
  milestone inti Tahap 3, dikerjakan belakangan)
- Alur publish: moderasi dulu (`PENDING` → admin approve) vs auto-publish
  langsung (`APPROVED` sejak submit, dihapus belakangan kalau bermasalah)
- Aksi Editor/Admin ke komentar: boleh edit isi vs cuma boleh hapus
- Filter kata SARA/kasar: tolak submit (validasi gagal, gak pernah tersimpan)
  vs sensor tampilan (`***`, tetap tersimpan & tayang)
- Lokasi data kata terlarang & cara web_portal mengeceknya: cache Hazelcast
  vs query langsung ke DB dari service yang nulis komentar

**Keputusan:**
- **Dibangun**, sebagai fitur orisinal — bukan Tahap 3 inti, masuk
  [[00-Slice-Backlog]] "Belum dijadwalkan", dikerjakan setelah lima milestone
  utama selesai
- Field: `content`, `authorName`, `authorEmail` (disimpan, **tidak pernah**
  masuk response publik/DTO — konsisten sama alasan S2-4), `authorWebsite`
  (opsional), `status` (`APPROVED`/`DELETED` — **tanpa** `PENDING`, meniru
  `Discussion.Status` CCWR persis), FK ke `Article`
- **Auto-publish** — komentar langsung tayang saat submit, gak ada antrian
  moderasi
- Editor/Admin **cuma bisa hapus** (soft delete via `status = DELETED`),
  **tidak ada** endpoint edit isi komentar orang lain — integritas komentar
  terjaga
- Filter kata: **tolak saat submit** direkomendasikan Claude (validasi gagal,
  gak pernah tersimpan), tapi **dipilih sensor** oleh pemilik proyek — alasan
  eksplisit: nilai belajar implementasi regex-replace, bukan soal mana yang
  teknisnya lebih baik
- Tabel `BadWord` (field `word`) tinggal di **DB `content_service`** (satu-
  satunya DB yang pasti ada di roadmap sekarang), dicek **langsung via query**
  oleh `content_service` saat `Comment` disimpan — **bukan** lewat Hazelcast

**Alasan:** moderasi manual gak realistis untuk proyek solo (komentar bisa
nunggu berhari-hari). Hazelcast ditolak karena salah problem — dia
menyelesaikan beban baca berulang di jalur publik tanpa datasource
(`web_portal`), sementara cek kata terlarang itu satu kali per submit di
jalur tulis (`content_service`, yang sudah punya akses DB langsung) — nge-
cache sesuatu yang cuma dicek sekali per aksi nambah kerumitan (invalidasi
cache saat admin update daftar kata) tanpa manfaat performa nyata. Ini juga
menyingkap gap: X4 di [[02-Tahap2-Design]] masih punya checklist terbuka
"Autentikasi admin (Spring Security Core)" yang nentuin apakah `admin_portal`
butuh DB sendiri (kayak `canon_admin_portal` di CCWR) — `BadWord` sengaja
ditaruh di `content_service` dulu supaya keputusan datasource `admin_portal`
gak diputuskan sepotong-sepotong lewat fitur samping kayak ini.

**Dampak ke tracker:** [[01-Tahap1-Discovery]] D1 — komentar dipindah dari
draf "TIDAK dibuat" ke "dibangun, ditunda". [[00-Slice-Backlog]] nambah slice
komentar orisinal + penajaman ke item X4 (datasource `admin_portal`).

---

## 2026-09-03 Sapuan sistematis CCWR + keputusan turunan (iklan, search, media, newsletter)

**Konteks:** setelah beberapa kali temuan CCWR muncul cuma pas ditanya reaktif
(Discussion, Role, Banner, GTM), pemilik proyek nanya langsung apakah sudah
dicek semua — jawabannya jujur belum. Dijalankan sapuan sistematis (subagent
Explore) ke seluruh modul CCWR di luar yang sudah dibahas, difilter relevansi
ke situs media/konten publik. Laporan lengkap ada di riwayat sesi ini (bukan
disalin penuh ke sini — terlalu panjang untuk log keputusan).

**Keputusan yang diambil dari temuan ini:**
- **Iklan/monetisasi:** ad network eksternal (AdSense-style), **bukan** sistem
  Banner internal CCWR. Cukup script tag + slot placement di template,
  hampir tanpa domain/backend sendiri. Dikerjakan pas `web_portal` dibangun
  (Fase 4), bukan sekarang.
- **Newsletter:** dikonfirmasi ditunda **tapi direncanakan** (bukan dibuang).
  Tidak ada rujukan CCWR (situs korporat, bukan media) — orisinal, kelas sama
  dengan Komentar. Butuh infra kirim email + scheduler + subscriber storage,
  belum ada di roadmap manapun.
- **Search bar bebas teks (D1):** **ya, dipakai** — CCWR punya
  `web_portal/.../SearchController.groovy` + `OpenSearchService` publik
  (`/search`). Menjawab item "TIDAK dibuat" D1 yang masih ngambang; selaras
  Fase 6 (OpenSearch) yang sudah direncanakan.
- **Penyimpanan gambar (X4 batch 1, blocker X2 "Featured image"):** ikut pola
  CCWR — S3 (`AmazonS3Service`, `MediaProcessingService`). Ini membuka
  blocker X2 yang sebelumnya nunggu keputusan ini.
- **Meta SEO fields (X2):** dikonfirmasi nama field konkret dari `Page`
  CCWR — `metaTitle`, `metaDescription`, `searchKeyword` — jadi rujukan
  penamaan field `Article` nanti.

**Sengaja BELUM diputuskan** (masuk [[00-Slice-Backlog]] "Belum dijadwalkan",
dipertimbangkan pas fase yang relevan, bukan sekarang):
- Patch notes/changelog artikel (analogi `ApiChangeLog` CCWR — relevan buat
  situs game, tiap game punya riwayat update)
- Related content component (analogi `PageRelatedReadsComp` dkk)
- Social share component
- Popup on-site (analogi CCWR `Popup`)
- Important Notice — overlap konseptual dengan Banner, perlu diputuskan
  fitur terpisah atau tidak
- Audit trail generik (CLAUDE.md pattern #6 sudah menyebut ini layak
  direplikasi — masih belum ada keputusan kapan)

**Alasan menunda sisanya:** sesi ini sudah melebar jauh dari D1 (role,
komentar+filter, iklan, GTM/consent, newsletter, lalu sapuan besar ini) —
memutuskan 6 kandidat baru sekaligus berisiko mengulang pola scope-melar yang
sudah pernah bikin proyek ini macet (lihat entry 2026-09-02). Kandidat yang
di luar item D1 asli sengaja diparkir sebagai backlog, dipertimbangkan satu-
satu pas fase yang relevan.

---

## 2026-09-03 Multi-site/multi-bahasa — struktur tabel dipertahankan, isi 1 baris

**Konteks:** item D1 terakhir ("TIDAK dibuat") sekaligus X4 batch 2
("Multi-site & multi-bahasa — sejauh mana konsepnya dipertahankan"). CLAUDE.md
sejak awal bilang "boleh disederhanakan ke satu site, tapi pertahankan
konsepnya" — ambigu antara struktur tabel tetap ada vs dibuang total.

**Opsi yang dipertimbangkan:**
1. Tabel `Site`/`Language`/`SiteLanguage` tetap ada, cuma diisi 1 baris
   (site="Slow Response", language="id")
2. Dibuang total — gak ada tabel, cukup dicatat sebagai penyimpangan
   terdokumentasi dari CCWR

**Keputusan:** opsi 1 — struktur tabel dipertahankan, single-row.

**Alasan:** nilai belajarnya justru di desain data yang mengantisipasi
multi-tenant walau faktanya cuma dipakai satu — sama alasannya kayak
keputusan role (opsi 2, bukan 1) sebelumnya: proyek ini soal belajar pola
CCWR, bukan efisiensi produk. Biayanya kecil (satu tabel ekstra + FK ke
`Article`/`Page` setara).

**Dampak ke tracker:** [[01-Tahap1-Discovery]] D1 checklist item 2 (semua
sub-item "TIDAK dibuat") selesai. [[02-Tahap2-Design]] X4 batch 2 item
"Multi-site & multi-bahasa" bisa dicentang.

---

## 2026-09-04 D2 — tabel fitur pembanding, Hardware/Tech dibuang

**Konteks:** D2 (Analisis Pembanding) butuh ubah temuan riset 3 situs
pembanding ([[05-Concept-Worksheet]] §1) jadi tabel wajib-rilis/nanti.
Ketemu satu gap: ketiga situs pembanding (Kokang Gaming, The Lazy Media,
JagatPlay) semuanya punya pilar **Hardware/Tech** sejajar News/Review/
Feature — worksheet proyek ini (§3) belum pernah menyebutnya sama sekali.

**Opsi yang dipertimbangkan:** tambah Hardware/Tech sebagai pilar ke-4
(butuh nilai baru di `ArticleType` — kontrak `shared_lib` yang sudah dipakai
3 modul sejak Fase 1) vs tetap fokus game software murni.

**Keputusan:** **tidak** — tetap fokus game, Hardware/Tech tidak ditambahkan.
`ArticleType` tetap NEWS/REVIEW/GUIDE (+ Feature ditunda, sudah diputuskan
2026-08-28).

**Alasan:** konsisten sama identitas produk yang sudah dipatok (target
pembaca core/single-player narrative gamers, bukan pembaca umum gadget/tech).
Nambah pilar berarti ubah kontrak `ArticleType` yang sudah "jalan" di 3
modul — biaya nyata untuk sesuatu yang di luar fokus produk.

**Tabel fitur wajib-rilis/nanti (D2), hasil final:**

| Fitur | Wajib rilis? | Alasan |
|---|---|---|
| Hardware/Tech sebagai pilar konten | **Tidak** | Di luar fokus (lihat di atas) |
| Kategori Gacha | Tidak | Fokus PC/konsol modern, mobile/gacha bukan target ([[05-Concept-Worksheet]] §2) |
| Rubrik "Time Machine" (kilas balik game lawas) | Nanti | Masuk pilar Feature/Opinion yang sudah ditunda, bukan rubrik terpisah |
| Review multi-halaman long-form | — (bukan keputusan D2) | Soal IA — dibahas di X1 |
| Dark mode UI | — (bukan keputusan D2) | Soal desain visual — dibahas pas `web_portal` template dibangun |
| Ekosistem komunitas + integrasi YouTube + Awards | **Tidak ditiru** | Bangun ekosistem eksternal, bukan pola arsitektur — di luar tujuan belajar CCWR |
| Skor kualitatif tanpa angka (pola ketiga pembanding) | **Tidak ditiru** | Sudah diputuskan 2026-08-28 — sengaja pakai skor numerik 1-10 buat range query OpenSearch |

**Dampak ke tracker:** [[01-Tahap1-Discovery]] D2 selesai — kedua checklist
tersisa (tabel fitur, daftar tidak-ditiru) sudah terisi lewat tabel di atas.
Tahap 1 keseluruhan (D1 + D2) **SELESAI** — Tahap 2 (Design, X1 dst) bisa
mulai.

---

## 2026-09-04 X1 — `Category` dibuang, taksonomi final 3 sumbu

**Konteks:** X1 (taksonomi) perlu nutup celah lama — CLAUDE.md roadmap awal
nyebut domain inti `Article, Game, Category, Tag, Author`, tapi Concept
Worksheet §6 (2026-08-28) sudah mutusin tiga sumbu (`ArticleType`, Platform,
Tag) tanpa pernah menyebut `Category` lagi. Item ini nggantung di
[[00-Slice-Backlog]] "Belum dijadwalkan" sejak draf 8 domain dihapus.

**Opsi yang dipertimbangkan:** tetap bikin `Category` generik (ala CCWR,
sengaja dibuang di §6 buat hierarki bersarang tapi kategorinya sendiri
gak pernah eksplisit dibuang) vs `Category` dibuang total, fungsinya cukup
diwakili `ArticleType`.

**Keputusan:** **`Category` dibuang**, tidak jadi domain terpisah.

**Alasan:** fungsi Category (mengelompokkan jenis konten) sudah diambil alih
`ArticleType` — kontrak `shared_lib` yang sudah jalan sejak Fase 1. Bikin
`Category` di atasnya cuma jadi sumbu keempat yang tumpang tindih, bukan
menambah kemampuan baru.

**Taksonomi final (3 sumbu):**
- Jenis konten → `ArticleType` (enum, `shared_lib`)
- Platform → enum/set field di `Game`
- Genre/topik → `Tag` (M:N ke `Game`, bebas/boleh ganda)

**Dampak ke tracker:** [[02-Tahap2-Design]] X1 checklist taksonomi selesai.
[[00-Slice-Backlog]] item lama soal `Category` ditutup — dihapus dari
"Belum dijadwalkan".

---

## 2026-09-04 X1 — wireframe homepage: featured Game, bukan featured Article

**Konteks:** wireframe kasar homepage (X1) butuh jawab: apa ada kurasi
editorial, atau murni kronologis? Claude merekomendasikan murni kronologis
tanpa field tambahan. Pemilik proyek punya arah lebih spesifik.

**Keputusan:**
- **Featured Game** (bukan featured Article) — homepage nampilin game
  unggulan pilihan editor, butuh mekanisme kurasi di `Game` (field/tabel,
  detail bentuknya nyusul di X2)
- **Carousel review terbaru** — artikel tipe REVIEW terbaru, tampil khusus
  bukan cuma nyampur di list biasa
- **Artikel non-review** — cukup list kronologis ke bawah, tanpa kurasi
  (sesuai rekomendasi awal Claude)

**Alasan:** kurasi di level Game (bukan Article) lebih pas — Game itu
first-class entity yang siklus hidupnya lebih panjang dari satu artikel,
jadi "unggulan" lebih masuk akal nempel ke situ. Review dapat perlakuan
khusus (carousel) karena itu pilar konten paling berat (skor, Pros/Cons)
dan paling representatif buat identitas situs.

**Dampak ke tracker:** [[02-Tahap2-Design]] X1 checklist wireframe selesai —
X1 keseluruhan tuntas (sitemap, jenis halaman, taksonomi, wireframe).

---

## 2026-09-04 "Popular Searches" CCWR — bukan analitik search internal

**Konteks:** pemilik proyek lihat fitur "Popular Searches" di situs Canon
publik (screenshot dari canon.com.sg atau serupa), mengira itu tracking
kata kunci pencarian yang sering diketik pengunjung, mau direplikasi buat
belajar. Dicek ke kode: `PopularSearchSource` enum (`web_portal/src/main/
groovy/web_portal/enums/PopularSearchSource.groovy`) cuma punya `STATIC`/
`DYNAMIC`. `DYNAMIC` manggil `ProductService.getPopularPages()` →
`GoogleAnalyticsService.getPopularPages()` (`services/corpweb`) — itu
**Google Analytics API**, ngambil halaman **paling sering dikunjungi**
(page view), bukan kata kunci paling sering dicari. `STATIC` adalah daftar
manual lewat setting/CMS, gak dihitung dari apa pun. Gak ada domain/tabel
pencatat query pencarian (`SearchLog` dkk) di seluruh CCWR.

**Keputusan:** kalau direplikasi, pakai **mode STATIC** (daftar manual
config-driven) — **bukan** integrasi GA beneran, karena capstone ini gak
akan punya trafik asli buat mode DYNAMIC ada artinya. **Belum diputuskan
in/out scope** — diparkir ke [[00-Slice-Backlog]], dibahas detail pas
Fase 6 (search dibangun), bukan sekarang (bagian dari halaman search, bukan
homepage/detail artikel yang lagi dibahas X1).

---

## 2026-09-04 X2 — batch field cepat + Audit trail plugin penuh

**Konteks:** X2 (ERD) checklist, item-item kecil yang gak butuh diskusi
panjang, plus satu keputusan X4 batch 1 yang masih kosong (audit trail).

**Keputusan batch field:**
- `Article.excerpt` — `String`, maxSize ~300
- `Article.metaTitle`/`metaDescription`/`ogImage`/`canonicalUrl` — semua
  nullable, fallback ke `title`/`excerpt`/`featuredImage` kalau kosong
- `Tag.slug maxSize` — diturunin dari 65535 (salah ketik nyalin `Tag.name`)
  ke 255
- `ArticleGame` — tambah `unique: ['article', 'game']`
- `Author` — tambah `slug` + `avatar`

**Keputusan Audit trail (X4 batch 1):** pakai **plugin `audit-logging` penuh**
ala CCWR (`AuditFields`/`Auditable`), bukan cuma `dateCreated`/`lastUpdated`
GORM bawaan.

**Alasan:** dulu (single dev) `dateCreated`/`lastUpdated` saja cukup, tapi
sekarang ada 4 role (Editor/Admin/Super Admin bisa ubah data yang sama) —
"siapa yang ubah apa" jadi relevan beneran, bukan cuma nice-to-have. Pola
CCWR jelas dan nilai belajarnya nyata (audit log adalah concern arsitektur
umum di kerjaan profesional).

**Dampak ke tracker:** [[02-Tahap2-Design]] X2 checklist batch field selesai,
X4 batch 1 "Audit trail" selesai (2 dari 4 item batch 1 — penyimpanan gambar
& audit trail — sudah diputuskan; autentikasi admin & editor teks masih
terbuka).

---

## 2026-09-04 X2 — `Game` dapat `gameType`/`baseGame`, gantiin ide agregasi skor

**Konteks:** item lama sejak 2026-08-31 ("Review 1:1 Article, konsekuensinya
ke agregasi skor per Game belum dipikirkan") akhirnya dibahas. Draf awal
sesi ini (Claude): skor resmi Game = skor dari `Review` milik artikel
REVIEW ter-`isPrimary`+terbaru yang nyinggung game itu (query, tanpa field
baru). **Ditarik balik** sebelum ditulis final — pemilik proyek nunjukin
pola Metacritic (base game/DLC/edition masing-masing skor SENDIRI, bukan
satu skor yang "menang") dan minta dicek ke pola `pim.Product` CCWR.

Dicek: `Product` CCWR (`services/pim/grails-app/domain/pim/Product.groovy`)
punya `variantOf`/`kitOf` (self-FK) + `ProductType` (`MAIN`, `VARIANT`,
`BUNDLE`, `KITSET` — komentar kode: *"Kitset is using same logic as MAIN
with additional attribute kitOf"*). Tiap varian/kit itu **baris `Product`
sendiri** (harga, SKU, semua data sendiri-sendiri), cuma di-link balik ke
produk induk buat pengelompokan — bukan satu produk yang datanya
diagregasi dari beberapa child. Ini persis pola Metacritic (base
game/DLC/edition = entri terpisah, skor masing-masing, dikelompokkan pas
search karena nama mirip).

**Opsi yang dipertimbangkan:**
1. Satu `Game`, banyak `Review` (via `Article` M:N), skor resmi = query
   "review REVIEW ter-primary + terbaru" (draf awal, ditarik)
2. `Game` dapat `gameType` + self-FK `baseGame`, tiap edition/DLC jadi baris
   `Game` terpisah dengan `Review` sendiri — ala `pim.Product`

**Keputusan:** opsi 2.
- `Game.gameType` — enum `MAIN` / `EDITION` / `DLC` (disederhanakan dari 4
  nilai `ProductType` CCWR — `BUNDLE` gak dipakai, proyek ini bukan
  e-commerce, gak butuh line-item quantity ala `ProductBundleItem`)
- `Game.baseGame` — self-FK nullable, diisi kalau `EDITION`/`DLC`, nunjuk
  ke `MAIN`-nya
- `Review` tetap 1:1 `Article` (gak berubah) — sekarang gak perlu logika
  agregasi sama sekali, karena tiap edition/DLC punya `Review` sendiri

**Alasan:** memisah tiap edition/DLC jadi baris sendiri itu benar secara
domain (mereka beneran produk/entitas berbeda dengan skor berbeda, bukan
satu entitas dengan banyak nilai yang perlu dipilih salah satu) — bukan
cuma soal skema lebih gampang. `isPrimary` di `ArticleGame` **tidak
berubah** — itu urusan terpisah (artikel nyinggung banyak game sekaligus),
tidak berkaitan dengan hierarki base/edition/DLC ini.

**Dampak ke tracker:** [[02-Tahap2-Design]] X2 checklist "Review vs
agregasi skor" selesai. [[00-Slice-Backlog]] item lama (2026-08-31,
"agregasi skor per Game belum dipikirkan") ditutup.

---

## 2026-09-04 X3 — Kontrak API: endpoint, response envelope, pagination, error

**Konteks:** X3 (Kontrak API) — `admin_portal`/`web_portal` harus bisa
dibangun dari dokumen ini tanpa baca kode `content_service`. Disusun dari
sitemap X1, ERD X2, dan keputusan lama (DTO-only S2-4, filter+pagination
S2-6, 404 JSON S2-5).

**Keputusan:**
- **Versioning:** **tanpa prefix** `/v1` — `/api/articles`, bukan
  `/api/v1/articles`. Pintu dua arah murah selama belum ada konsumen di
  luar kendali (cuma `web_portal`/`admin_portal`, dua-duanya proyek sendiri).
- **Endpoint publik:** `GET /api/articles` (filter `type`/`platform`/`game`,
  pagination), `GET /api/articles/{slug}`, `GET /api/games` (filter
  `platform`/`genre`, pagination), `GET /api/games/{slug}` (termasuk skor,
  artikel terkait, daftar edition/DLC via `baseGame`)
- **Endpoint admin (Fase 3):** CRUD standar per domain
  (`POST`/`PUT`/`DELETE` di `/api/articles`, `/api/games`, `/api/tags`,
  `/api/authors`) — detail auth nunggu X4
- **Response envelope:** `{ "data": [...], "meta": {...} }` untuk list,
  `{ "data": {...} }` untuk detail, `{ "error": { "code", "message" } }`
  untuk error. `data` selalu DTO `shared_lib`, gak pernah domain class
  langsung.
- **Pagination:** `max`/`offset` (istilah GORM), default `max=20`, **hard
  cap `max=100`**
- **Format error:** selalu JSON `code`+`message`, status HTTP yang benar
  (404/400/422), gak pernah halaman error HTML Grails
- **HTTP method `QUERY` (draf IETF, GET semantik + body kompleks):**
  **tidak dipakai**. CCWR nihil (grep kosong), filter proyek ini cukup
  sederhana buat query string biasa, dan Grails 6.2.3/Spring MVC kemungkinan
  besar gak dukung verb ini tanpa konfigurasi custom.

**Alasan:** semua turunan langsung dari keputusan yang udah ada sebelumnya
(bukan pilihan baru yang perlu ditimbang) — X3 di sini sifatnya merapikan
jadi kontrak eksplisit, bukan membuka diskusi arah baru.

**Dampak ke tracker:** [[02-Tahap2-Design]] X3 SELESAI seluruhnya.

---

## 2026-09-04 X4 batch 1/2 — autentikasi admin, migrasi DB, editor teks, frontend

**Konteks:** X4 batch 1 sisa (autentikasi admin, editor teks) dan batch 2
(migrasi database, frontend `web_portal`) dibahas sekaligus — sebagian
besar turunan langsung dari referensi `daftar-keputusan.md` yang sudah
nge-grep CCWR duluan.

**Keputusan cepat:**
- **Frontend `web_portal`:** GSP — konsisten CCWR, dan SSR memang argumen
  teknis sendiri buat situs media (SEO)
- **Migrasi database:** **Liquibase** (plugin resmi `database-migration`
  Grails). CCWR sendiri gak pakai migrasi (`database-migration` di-exclude
  di semua module, gak ada folder `migrations`) — ini kasus sengaja
  **tidak** ikut CCWR, karena `dbCreate: update` gak layak produksi dan
  nutup jalan ke M5 (Release). Liquibase dipilih atas Flyway karena native
  ekosistem Grails
- **Editor teks admin:** **WYSIWYG generik** — bukan replikasi
  `content-builder` CCWR (block-builder drag-drop custom di
  `admin_portal/grails-app/assets/editor/content-builder/`). Sengaja
  menyimpang — content-builder itu proyek UI berminggu-minggu sendiri,
  gak proporsional buat body artikel yang linear (bukan halaman marketing
  modular kayak CCWR)

**Keputusan Autentikasi admin (paling besar):**

Dicek ke CCWR: domain auth lengkapnya `User, Role, UserRole, Permission,
PermissionRole, Profile, UserLoginHistory, UserPasswordHistory,
UserResetPasswordAttempt, UserToken` (10 domain, plugin
`spring-security-core`), semuanya di DB `canon_admin_portal` — terpisah
dari `canon_corpweb` (konten).

Flow forgot-password CCWR ditelusuri langsung dari
`ForgotPasswordController`/`ForgotPasswordService`/`UserService`:
submit email → cek `User` (`enabled`, `!accountLocked`, `!passwordExpired`)
dengan **pesan error sama** baik email gak ketemu maupun kirim gagal
(cegah user enumeration) → cek `UserResetPasswordAttempt` (rate-limit) →
generate `UserToken` (`FORGOT_PASSWORD`, expiry 48 jam) → email link
(Mandrill) → `ResetPasswordController` validasi token, set password baru.
Password expiry: `User.passwordExpired` dicek pas login, ambang batas
`SettingType.PASSWORD_EXPIRY_TIME_IN_DAYS` (config-driven, default seed
CCWR 120 hari) + `PASSWORD_EXPIRY_EXCLUDED_USER` (whitelist). Reuse
password lama dicegah lewat `UserPasswordHistory` (nyimpen hash tiap ganti
via `UserListener`, dicek `UserService` pas password baru disubmit).
Akun **tidak** self-register — dibikin admin lewat User Management.

**Keputusan:**
- **7 dari 10 domain dipakai:** `User`, `Role`, `UserRole`, `Permission`,
  `PermissionRole` (RBAC — mendukung rencana pecah Admin per master data),
  `UserToken` (forgot-password + set-password pertama kali saat akun baru
  dibikin admin), `UserPasswordHistory` (cegah reuse password)
- **Skip:** `Profile` (field profil ekstra, gak esensial), `UserLoginHistory`
  (tumpang tindih sama audit-logging yang udah diputuskan), `UserResetPasswordAttempt`
  (rate-limit lockout — hardening ekstra di atas token-expiry yang udah ada)
- **`admin_portal` dapat DB sendiri** (misal `capstone_admin_portal`),
  terpisah dari `capstone_db` (`content_service`) — ikut pola CCWR
  (`canon_admin_portal` vs `canon_corpweb`), auth itu bounded context beda
  dari konten
- **Password expiry: 90 hari** (bukan 120 seperti default seed CCWR — pilihan
  sadar pemilik proyek, bukan ikut CCWR persis), plus
  `PASSWORD_EXPIRY_EXCLUDED_USER` whitelist tetap dipakai
- **Akun tidak self-register** — dibikin admin lewat User Management,
  konsisten CCWR
- **Email beneran dikirim** (bukan cuma di-log) — pakai plugin `mail`
  Grails standar (bukan Mandrill CCWR, itu berbayar) ke **MailHog** lokal
  di `docker-compose` (SMTP catcher, email keliatan di web UI lokal, gak
  butuh kredensial asli). Infra ini dipakai bareng buat forgot-password
  dan Newsletter (nanti) — satu keputusan buat dua kebutuhan, gak diputuskan
  dua kali

**Alasan:** RBAC + token flow + password rotation ada nilai belajar nyata
(pola arsitektur yang sering muncul di kerjaan profesional), dan
`UserPasswordHistory`/`UserToken` itu bukan sekadar tabel pasif — CCWR
make mereka aktif buat validasi, jadi layak direplikasi. Yang di-skip
(`Profile`, `UserLoginHistory`, `UserResetPasswordAttempt`) itu hardening
lapis tambahan di atas yang udah ada, bukan konsep baru — nambahnya gak
nambah pelajaran, cuma nambah tabel.

**Dampak ke tracker:** [[02-Tahap2-Design]] X4 batch 1 & batch 2 SELESAI
semua — Tahap 2 (Design) tuntas sepenuhnya.

---

## 2026-09-02 Hapus domain draft `content_service`, mulai ulang setelah Tahap 2 tuntas

**Konteks:** delapan domain class (`Article`, `Game`, `Author`, `Review`, `Tag`,
plus 3 join table) dan 8 file Spec sudah ditulis sebelum Tahap 1 (Discovery) dan
Tahap 2 (Design) — khususnya X2 (Content Model/ERD) dan X4 (keputusan teknologi,
terutama penyimpanan gambar) — selesai. Ketahuan lewat rekonsiliasi vault vs
codebase: model itu sudah diketahui belum final (belum ada featured image,
excerpt, meta SEO, dan beberapa keputusan relasi menggantung — lihat checklist
X2 di [[02-Tahap2-Design]]).

**Opsi yang dipertimbangkan:** (a) revisi domain yang sudah ada begitu X2
final, (b) hapus semuanya dan tulis ulang dari nol setelah X2 final.

**Keputusan:** opsi (b) — hapus. `shared_lib` (enum `ArticleType`,
`PublishStatus`, `GamePlatform`, DTO `ArticleResponse`) **tidak** dihapus —
isinya kategorikal/kontrak, tidak terikat ke bentuk field `Article`/`Game`
yang mana pun, jadi tetap relevan apa pun hasil X2 nanti.

**Alasan:** file domain itu belum pernah di-commit, jadi tidak ada sunk cost.
Merevisi draft yang ditulis sebelum desain final berisiko menambatkan
keputusan baru pada asumsi lama (nama field, bentuk relasi) yang sebenarnya
belum pernah benar-benar diputuskan — cuma kebetulan ditulis lebih dulu. Mulai
bersih memaksa domain final benar-benar lahir dari ERD hasil X2, bukan dari
patch di atas draft yang terburu-buru.

**Dampak ke tracker:** `content_service/grails-app/domain` sekarang kosong
lagi. [[00-Slice-Backlog]] S2-1 perlu ditulis ulang setelah Tahap 1-2 selesai —
jangan dikerjakan dari isi lama.

---

## 2026-08-31 Struktur relasi Article↔Game — join domain, bukan hasMany native

**Konteks:** penyempurnaan dari entry "Domain model Article & Game — versi 1"
di bawah — sudah diputuskan many-to-many, tapi belum diputuskan *cara*
implementasinya di GORM. Dipicu diskusi pola `PageProduct` di CCWR
([[01-Glossary-CCWR-Patterns]]) sebagai referensi.

**Opsi yang dipertimbangkan:**
- `hasMany`/`belongsTo` native GORM dua arah (`Article` ↔ `Game` langsung)
  vs domain join eksplisit (`ArticleGame`) seperti pola `Page`↔`PageProduct`↔`Product`
  di CCWR
- Kalau join eksplisit: `Game` deklarasi `hasMany` balik ke `ArticleGame`
  atau tidak
- Field `isPrimary` di `ArticleGame`: constraint keunikan (maks 1 primer per
  artikel) atau `Boolean` polos tanpa constraint

**Keputusan:**
- Pakai **domain join eksplisit** `ArticleGame` (`Article article`,
  `Game game`, `Boolean isPrimary`), bukan `hasMany`/`belongsTo` native
- `Article` **hasMany** `articleGames: ArticleGame` (jumlah game per artikel
  kecil, aman di-load)
- `Game` **tidak** deklarasi apapun balik ke `ArticleGame` — feed artikel per
  game (buat halaman hub `/games/{slug}`) diambil lewat query eksplisit
  dengan sort & pagination (mis. `ArticleGame.findAllByGame(...)`), bukan
  navigasi koleksi
- `isPrimary` — **`Boolean` polos, tanpa constraint keunikan** untuk sekarang

**Alasan:** relasi butuh field tambahan (`isPrimary`) yang gak punya tempat
di `hasMany`/`belongsTo` native GORM — begitu butuh kolom di relasi itu
sendiri (bukan di salah satu entitas), harus lompat ke domain join eksplisit.
`Game` sengaja gak deklarasi balik karena dua alasan: (1) query feed butuh
sort & pagination yang gak bisa didapat dari sekadar navigasi koleksi
`game.articleGames`, jadi properti itu gak akan pernah benar-benar dipakai;
(2) mencegah risiko koleksi termuat gak sengaja (JSON serialization ke
`ArticleResponse`, render GSP, audit-logging) yang bisa jadi besar untuk game
populer — persis alasan `Product` di CCWR gak deklarasi balik ke
`PageProduct`. `isPrimary` sengaja dibiarkan tanpa constraint dulu — kalau
nanti jadi masalah nyata (dua primer di satu artikel), lebih murah ditambah
`validator` belakangan daripada didesain berlebihan sekarang.

---

## 2026-08-28 Domain model Article & Game — versi 1

**Konteks:** riset pembanding 3 situs media game lokal (kokanggaming.com,
thelazy.media, jagatplay.com — detail lengkap di
[[05-Concept-Worksheet]] bagian 1) selesai, sekaligus jadi dasar keputusan
skema `Review`, relasi `Article`↔`Game`, dan taksonomi sebelum Fase 2 mulai.

**Opsi yang dipertimbangkan:**
- Skor review: skala 1–10 vs 1–100 vs kualitatif tanpa angka (pola yang
  dipakai ketiga situs pembanding) vs breakdown skor per aspek
  (Gameplay/Graphics/Story/Sound)
- Relasi Article↔Game: one-to-many vs many-to-many; `Game` sebagai metadata
  pasif vs first-class entity dengan halaman sendiri
- Taksonomi: Genre sebagai Category vs Tag; Platform sebagai teks bebas vs
  enum terstruktur; kategori berjenjang vs flat

**Keputusan:**
- Skor: skala **1–10** (desimal 1 digit, `BigDecimal`/`DECIMAL(3,1)`), **skor
  tunggal editorial** + ringkasan Pros/Cons/Verdict (bukan breakdown per
  aspek), dinilai **editor saja** (rating pembaca ditunda)
- Relasi Article↔Game: **many-to-many**, `Game` **first-class entity** dengan
  halaman hub sendiri (`/games/{slug}`)
- Taksonomi: **Genre → Tag**, **Platform → enum/set field di `Game`**
  (diwariskan ke payload Kafka → index OpenSearch → filter Web Portal),
  **kategori flat 1 level**
- Pilar konten Fase 1: **News, Review, Guide** dicentang; **Feature/Opinion
  ditunda**

**Alasan:** tiap keputusan diprioritaskan berdasarkan nilai pengujian
arsitektur (bukan cuma selera produk) — lihat rincian lengkap per poin di
[[05-Concept-Worksheet]]. Yang menonjol untuk dicatat di sini: skor numerik
**sengaja menyimpang** dari pola ketiga situs pembanding (yang semuanya
kualitatif tanpa angka) karena proyek ini butuh field terindeks untuk uji
range query OpenSearch — bukan mengikuti tren pasar. Kategori flat dan skor
tunggal (tanpa breakdown) sama-sama sengaja menghindari kompleksitas yang
belum perlu di tahap ini (nested category tree, tabel anak buat rata-rata
aspek).

---

## 2026-08-28 Identitas produk: nama, tagline, target pembaca

**Konteks:** nama "slow-response" sejauh ini cuma nama teknis repo. Perlu
diputuskan apakah brand publik situs beda dari nama teknisnya.

**Opsi yang dipertimbangkan:** pakai nama publik baru yang lebih "aman" untuk
situs berita (menghindari kesan lambat) vs pertahankan "Slow Response" dengan
positioning yang direframe.

**Keputusan:** nama publik tetap **Slow Response**, tagline "Ruang baca
tenang bagi gamer yang mencari ulasan mendalam dan cerita di balik layar",
target pembaca **core & single-player/narrative gamers (dewasa/pekerja)**,
cakupan platform **PC & konsol modern** (PlayStation/Nintendo/Xbox) — mobile
dan gacha sengaja bukan fokus utama.

**Alasan:** "Slow Response" direframe jadi *unhurried/reflektif* (selaras
tagline "ulasan mendalam"), bukan "situs berita yang telat". Risikonya nama
ini tetap bisa dibaca negatif oleh orang yang belum lihat taglinenya —
kalau dipresentasikan (capstone, portfolio), siapkan satu kalimat penjelas
biar konteksnya gak ilang.

---

## 2026-08-17 Scope dikunci ke 3 modul

**Konteks:** CCWR punya 6 modul (`web_portal`, `admin_portal`, `api`,
`services:corpweb`, `services:pim`, `services:sims`) + `shared_lib`. Meniru
semuanya untuk proyek latihan solo gak realistis dalam waktu terbatas.

**Opsi yang dipertimbangkan:** replikasi penuh 6 modul vs ambil subset yang
paling merepresentasikan pola inti (read/write split, shared contract layer).

**Keputusan:** hanya 3 modul — `web_portal`, `admin_portal`, `content_service`
(pengganti nama untuk konsep `services:corpweb`) — plus `shared_lib` sebagai
plugin pendukung (bukan dihitung sebagai "service"). `api`, `services:pim`,
`services:sims` di-drop dari rencana.

**Alasan:** `services:corpweb` (205 domain class) adalah modul paling umum dan
paling dekat dengan kebutuhan situs konten (halaman, kategori, tag) —
sementara `pim` (produk fisik/consumer goods) dan `sims` (support info Canon)
terlalu spesifik ke bisnis Canon, gak ada padanan wajar di situs game. `api`
(integrasi third-party) prioritas paling rendah karena gak menyentuh pola
inti (read/write split via OpenSearch) yang jadi fokus pembelajaran.

---

<!-- Entry baru ditambahkan di atas garis ini -->
