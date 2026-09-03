# Peta CCWR Lengkap — Modul, Fitur, Teknologi

Hasil penelusuran menyeluruh ke `/Users/ilal/canon-corporate-website-revamp`
(2026-09-04). Semua angka dan nama file di sini **hasil baca langsung**, bukan
ingatan. Dibuat karena rencana Tahap 1-2 dicurigai ada yang terlewat — bagian
terpenting file ini adalah **Bagian 5 (Gap Analysis)**.

Bedanya dengan [[01-Glossary-CCWR-Patterns]]: glossary itu cheat-sheet tanya-jawab
buat dicek cepat sambil ngoding. File ini peta lengkap + audit rencana.

---

## 1. Peta modul — angka nyata

| Modul | Domain | Controller | Service | TagLib | GSP | src/main | DB sendiri |
|---|---|---|---|---|---|---|---|
| `web_portal` | **0** | 45 | 49 | 3 | 751 | 389 | **tidak ada** |
| `admin_portal` | 20 | 84 | 48 | 4 | 819 | 766 | `canon_admin_portal` |
| `api` | 17 | 14 | 13 | 0 | 8 | 108 | `canon_admin_portal` |
| `services:corpweb` | **206** | 54 | 102 | 1 | 128 | 319 | `canon_corpweb` |
| `services:pim` | 72 | 25 | 56 | 0 | 0 | 335 | `canon_pim` |
| `services:sims` | 62 | 15 | 44 | 0 | 0 | 159 | `canon_sims` |
| `shared_lib` | 0 | 0 | 2 | 0 | 0 | **675** | plugin, bukan app |

Root `settings.gradle` memuat ketujuhnya. Grails 6.2.3, `org.gradle.parallel=true`.

### Isi `shared_lib` per paket (675 file)

| Paket | File | Isi |
|---|---|---|
| `model` | 171 | DTO request/response |
| `component` | 93 | definisi komponen halaman (BannerComp, RelatedReadsComp, dst) |
| `enums` | 69 | seluruh enum kontrak |
| `event` | 61 | Canon-spesifik (event management) |
| `cps` | 57 | Canon-spesifik (membership fotografer) |
| `commandObject` | 48 | object binding/validasi form |
| `utils` | 24 | `ContentUtils`, `SanitizingUtils`, `SlugCodec`, `PreviewUtils`, `LogMaskingUtils`, `HashUtils`, `ImageUtils`, `VideoUtils`, `URLUtils`, `DateUtils` |
| `media` | 22 | pemrosesan media |
| `kafka` | 21 | template producer/consumer, payload, topic enum |
| `microsite` | 15 | Canon-spesifik |
| `dam` | 12 | digital asset management |
| `domain` | 11 | POGO bersama (`Media`, `BaseDomain`) |
| `sdk*` | 24 | Canon-spesifik |
| `email` | 8 | Mandrill + SMTP config |
| `opensearch` | 4 | `OpenSearchIndex`, `OpenSearchConnectionTrait`, `OpenSearchUtils` |
| `amazons3` | 4 | `AmazonS3Service`, config, factory |
| `hazelcast` | 3 | `HazelcastService`, `BannerItemCache` |
| `telegramBot` | 1 | notifikasi Telegram |

**Pelajaran ukuran:** `shared_lib` itu modul **terbesar kedua** setelah corpweb.
Lapisan kontrak di arsitektur seperti ini memang gemuk — bukan folder kecil berisi
2-3 enum.

---

## 2. Inventaris teknologi lengkap

### Inti (semua modul)
Grails 6.2.3 · Java 17 · Gradle 7.6.4 · Spring Boot · GORM/Hibernate5 (kecuali
`web_portal`) · MySQL 8 (`mysql-connector-j:8.4.0`) · Tomcat + `tomcat-jdbc` ·
`spring-boot-starter-actuator` (health check di semua modul) · `janino` (config
logging kondisional)

### Per kapabilitas

| Kapabilitas | Library/plugin | Dipakai di |
|---|---|---|
| Service discovery | `spring-cloud-starter-consul-discovery:3.1.5` | web_portal, admin_portal, corpweb, pim, sims |
| HTTP client antar-service | `spring-cloud-starter-openfeign:3.1.9` + `spring-cloud-loadbalancer` | web_portal, admin_portal, corpweb, sims |
| Search | `opensearch-rest-client:2.14.0` + `opensearch-java:2.6.0` + `jakarta.json` | shared_lib, web_portal, corpweb, pim, sims, api |
| Cache terdistribusi | `hazelcast:5.5.0` | shared_lib, web_portal, admin_portal |
| Cache lokal | `caffeine:3.1.8` | **web_portal saja** (lapis kedua) |
| Messaging | `kafka-clients:3.9.0` + `aws-msk-iam-auth` | shared_lib, admin_portal, corpweb, pim, sims |
| Object storage | `software.amazon.awssdk:s3` (+ `aws-java-sdk` 1.x di corpweb/pim) | shared_lib, web_portal, admin_portal, sims |
| Auth (session) | `spring-security-core:6.0.0` | admin_portal |
| Auth (JWT/REST) | `spring-security-rest:5.0.1` + `-gorm` | **api saja** |
| Audit | `audit-logging:4.0.3` | shared_lib, admin_portal, corpweb, pim, sims, api |
| Email | `org.grails.plugins:mail:4.0.0` + Mandrill (custom service) | shared_lib |
| JSON view | `views-json` + `views-json-templates` (`.gson`) | corpweb, pim, sims, api |
| Render JSON manual | `BaseResponse<T>` + `as JSON` | corpweb, admin_portal |
| Async/paralel | `grails.plugins:async` (corpweb), `events` (pim, sims), `gpars:1.2.1` (corpweb, sims) | — |
| Gambar | `sanselan` (metadata EXIF), `simplemagic` (deteksi MIME), `twelvemonkeys imageio-webp` (WebP) | shared_lib, pim |
| Video | `jcodec` + `jcodec-javase` | shared_lib, admin_portal, corpweb, pim |
| Ekstraksi dokumen | `tika-parsers:1.28.3` | web_portal, admin_portal, pim |
| HTML parsing/sanitasi | `jsoup:1.15.3` | semua modul |
| PDF | `grails.plugins:rendering:2.0.3` + `xhtmlrenderer` | corpweb |
| QR code | `zxing:3.3.0` (core + javase) | corpweb |
| Validasi telepon | `libphonenumber:7.4.5` | corpweb |
| Analytics | `google-analytics-data:0.104.0` (GA4 Data API) | corpweb |
| Pembayaran | `adyen-java-api-library:39.5.0` | **web_portal** |
| Notifikasi chat | `java-telegram-bot-api` | shared_lib, pim |
| Arsip ZIP | `zip4j` | corpweb, pim |
| Frontend asset | `asset-pipeline-grails:4.3.0` | web_portal, admin_portal, api |
| Editor konten | `content-builder` (block editor custom, bukan library publik) | admin_portal |
| Test unit | Spock + `grails-gorm-testing-support` + `views-json-testing-support` | semua |
| Test browser | **Geb + Selenium 4.19.1** (chrome/firefox/safari driver) | web_portal, api, sims |
| File watching | `directory-watcher:0.18.0` | web_portal, admin_portal, pim |

### Yang TIDAK ada di CCWR
Migrasi skema (Liquibase/Flyway **di-exclude** di semua modul, tidak ada folder
`migrations`) · Redis · GraphQL · Docker Compose untuk dev · TinyMCE/CKEditor ·
Markdown · HTTP method `QUERY`.

---

## 3. Fitur per modul

### `web_portal` — read side publik (0 domain, 45 controller)

**Konten:** `Home`, `Page`, `Article`, `ArticleTrait`, `PressRelease`,
`CaseStudy`, `Solutions`, `Search`, `Sitemap`, `Redirect`, `Faq`
**Media:** `Media`, `Dam`, `PhotoLibrary`
**Produk:** `Product`, `ProductPromotion`, `WhereToBuy`, `Tco` (kalkulator biaya)
**Event:** `EventHome`, `EventContent`, `EventNews`, `EventGallery`,
`EventPeople`, `EventSearch`
**Form/lead:** `ContactUs`, `EoiForm`, `WhitePaper`, `PaymentNotification`
**Support:** `SimsContent`, `SimsModel`, `SimsMsds`, `SimsNotices`, `SimsSearch`,
`SimsSupport`, `Sdk`
**CPS:** `CpsApplication`, `CpsEnquiries`, `CpsRenewal`
**Operasional:** `Cache` (endpoint kelola cache), `Error`, `Microsite`

**Tiga interceptor (lapisan lintas-cutting):**
- `SiteInterceptor` — resolve site + bahasa dari path
- `GeneralInterceptor` — bangun `RequestModel` (header, footer, menu, popup,
  notice, consent, GTM ID, popular pages) untuk **setiap** request
- `HttpRequestLogInterceptor` — **rate limiter**: 30 request / 120 detik per IP,
  whitelist `127.0.0.1`, kalau lewat → forward ke `error/tooManyRequests` dengan
  `Cache-Control: no-store`. Implementasi `HttpRequestLogService`:
  `ConcurrentHashMap` ring buffer kapasitas 10.000. Statusnya saat ini
  `HTTP_REQUEST_LOG_ENABLED = false`.

**Pola URL (`UrlMappings`):** site dan bahasa ada **di dalam path**, tipe konten
jadi **sufiks**:
```
/{site}/{lang}/{segment}/articles              → listing
/{site}/{lang}/{segment}/{slug}/article        → detail
/{site}/{lang}/{segment}/{slug}/product        → detail produk
/{site}/{lang}/{segment}/{slug}/{section}      → constraint regex (specification|buy|tco|photos)
/{site}/{lang}/{segment}/web/{structures**}/{slug} → halaman generik, path bertingkat
```

### `admin_portal` — CMS + auth (20 domain, 84 controller)

**Auth (10 domain):** `User`, `Role`, `UserRole`, `Permission`, `PermissionRole`,
`Profile`, `UserLoginHistory`, `UserPasswordHistory`, `UserResetPasswordAttempt`,
`UserToken`
**Operasional:** `AuditTrail`, `Email`, `EmailLog` (template & log kirim email
disimpan di DB), `PendingDownload` (job export async), `InlineAsset`, `Location`,
`Site`, `SiteLanguage`, `SiteSetting`, `Language`

**Controller konten:** `Page`, `PageStructure`, `Article`, `PressRelease`,
`CaseStudy`, `Media`, `Dam`, `Asset`, `PhotoLibrary`, `BannerManagement`,
`PopupManagement`, `ImportantNotice`, `Header`, `Footer`, `Homepage`,
`Redirection`, `CampaignUrl` (generator URL UTM), `Discussion`
**User:** `UserManagement`, `UserTagManagement`, `ForgotPassword`, `ResetPassword`
**Tooling:** `AdminTool`, `Admin` (trigger reindex OpenSearch), `AdminRest`

`AdminInterceptor` menampilkan notice "password kadaluwarsa N hari lagi" sebelum
benar-benar expired.

### `services:corpweb` — write side konten (206 domain)

Dari 206 domain, yang **relevan untuk situs konten** hanya ±40:

| Kelompok | Domain |
|---|---|
| Halaman | `Page`, `PageDetail`, `PageComponent`, `PageDetailPublication`, `PageCategory`, `PageTag`, `PageTopic`, `PageProduct`, `PageProductCategory`, `PageRelation`, `PageStructure`, `PageFile` |
| Taksonomi | `Category`, `CategoryTranslation`, `Subject`, `SubjectTranslation`, `Lookup`, `LookupTranslation` |
| Media | `Media`, `MediaTranslation`, `DamSourceFolder` |
| Navigasi | `Structure`, `StructureDetail`, `Menu` |
| Multi-site | `Site`, `SiteLanguage`, `SiteSetting`, `Language`, `Country`, `District` |
| Marketing | `Banner`, `BannerItem`, `BannerItemDetail`, `BannerOrder`, `Popup`, `PopupDetail`, `Notice`, `NoticeDetail`, `Campaign`, `Promotion` |
| Lain | `Redirect`, `RedirectUrlDetail`, `Discussion`, `Email`, `EmailLog`, `AuditTrail`, `PopularProduct`, `PendingDownload` |

Sisanya (±165 domain): `Event*` (±60), `Cps*` (±20), `Sdk*` (±20), `Sims*`,
`Eoi*`/`Form*`, `Microsite*`, `WhitePaper*`, `B2BContact`/`Company` — semua
Canon-spesifik.

**5 Kafka consumer + 2 producer:** `KafkaIndexingConsumerService`,
`KafkaDiscussionConsumerService`, `KafkaMediaConsumerService`,
`KafkaProductConsumerService`, `KafkaUserConsumerService`, `KafkaProducerService`,
`KafkaMediaProducerService`. Plus `IndexingService`, `GoogleAnalyticsService`,
`PopularProductService`.

### `services:pim` (72 domain) & `services:sims` (62 domain)
Canon-spesifik. Yang layak dicatat cuma **pola `Product`**: `variantOf`/`kitOf`/
`translationOf` (tiga self-FK) + `ProductType` (MAIN/VARIANT/BUNDLE/KITSET) —
sudah diadopsi jadi `Game.gameType`/`baseGame` (lihat [[00-Decisions-Log]] 2026-09-04).

### `api` (17 domain) — API partner
Domainnya **persis salinan auth admin_portal** + `AuthenticationToken`.
Pakai JWT (`spring-security-rest`), bukan session. Punya folder `v2` — jadi API
publik CCWR **memang berversi**, berbeda dari API internal.

---

## 4. Pola arsitektur lintas modul

### 4a. Database-per-service + replikasi reference data
`Site`, `SiteLanguage`, `SiteSetting`, `Language`, `User`, `Role`, `UserRole`,
`AuditTrail`, `Lookup`, `Media`, `Email`, `EmailLog` **ada di hampir setiap
service dengan tabel masing-masing**. Tidak ada shared database, tidak ada JOIN
lintas service. Konsistensinya dijaga lewat Kafka (`Event-admin-user-1.0` dst).

### 4b. Kafka — jauh lebih luas dari "user + komentar"
Topic nyata dari `application.yml`:
```
Event-admin-user-1.0              # sinkronisasi user antar-service
Event-corpweb-discussion-1.0      # komentar/diskusi
Event-corpweb-media-1.0           # event media
Event-corpweb-sync-media-1.0      # sinkronisasi media
Event-corpweb-indexing-1.0        # ← TRIGGER INDEXING OPENSEARCH
Event-pim-product-corp-1.0        # sinkronisasi produk PIM → corpweb
Event-pim-product-1.0 / -category-1.0 / -media-1.0 / -access-tag-1.0 / -channel-1.0
```
`IndexingEventPayload` (`ids`, `type: OpenSearchIndex`, `isPreview`,
`publishedOnly`, `indexAll`, `siteId`) dikirim `AdminController` →
`KafkaIndexingConsumerService` di corpweb → OpenSearch. **Kafka adalah transport
indexing**, bukan cuma komentar.

### 4c. OpenSearch — desain index
`OpenSearchIndex` enum mendeklarasikan per index: `hasPreviewIndex`,
`hasSiteLanguageIndex`, `hasLanguageIndex`.
```
ARTICLE("articles", true, true)  →  articles__{siteId}__{lang}
                                 →  preview_articles__{siteId}__{lang}
```
Ada `prefix` per environment (`dev`), dan preview boleh di **cluster terpisah**
(`preview_hosts`). Index non-konten juga ada: `AUDIT_LOG`, `REQUEST_LOG`,
`SITE_SETTINGS`, `MEDIA`.

Domain `Page` juga mendeklarasikan `ES_SORTABLE_FIELDS` (termasuk sort nested
`publications.published_date`) — kontrak sorting hidup di domain class.

### 4d. Konfigurasi sebagai data, bukan file
`SiteSetting` = `site + language + segment + settingType → settingValue`
(mediumtext, unique komposit). Isinya antara lain `GTM_ID`, `USER_CENTRICS_ID`,
`HEADER_SCRIPT`, `PASSWORD_EXPIRY_TIME_IN_DAYS` (seed: 120),
`PASSWORD_EXPIRY_EXCLUDED_USER`, `POPULAR_SEARCH_SOURCE`,
`MEDICAL_INDUSTRIAL_ENABLED_PER_SITE`. Editor bisa ubah tanpa deploy.

### 4e. Versioning konten (`Page`)
Status **4 nilai**: `DRAFT`, `RELEASE`, `PUBLISHED`, `ARCHIVED`.
Tiga self-FK: `parent` (terjemahan) · `master` (copy asli) · `origin` (shadow/
draft) + `publishedVersion` (Integer) + `getMasterPage()` yang menangani 4
kombinasi (draft-translation, translation, draft-master, master).
`CLONEABLE_FIELDS` — daftar eksplisit field yang ikut tersalin saat clone.
Slug hidup di `PageDetail` (bukan `Page`), karena satu halaman bisa muncul di
beberapa segment dengan slug berbeda; publikasinya di `PageDetailPublication`.

### 4f. Anti-pola & detail operasional
- `autoTimestamp false` + transient `disableTimestamp` di `Page`/`Media` — supaya
  operasi massal (reindex, clone) tidak mengubah `lastUpdated`
- `ContentUtils.hasBlacklistedTags(val)` — validator anti-XSS dipakai di hampir
  semua field teks (`Menu.name`, `Structure.slug`, `Category.name`, dst)
- `Redirect.slug` punya validator **reserved keyword** (`en`, `id`, `consumer`,
  `business`, `api`, `media`, `admin`, …) dan validasi regex
- `Category.metaDescription` maxSize 160 + tolak emoji/tag
- `PageComponent` disimpan lewat **custom Hibernate UserType** yang memetakan satu
  objek ke 4 kolom (`comp_type`, `comp_content` longtext, `comp_order`,
  `comp_row_settings`)

---

## 5. Gap Analysis — apa yang meleset dari rencana kita

Diurut dari yang paling berdampak.

### 🔴 G1. CLAUDE.md salah soal Kafka
**Tertulis:** *"Kafka untuk sinkronisasi user dan komentar antar-service — bukan
untuk indexing konten."*
**Faktanya:** ada topic `Event-corpweb-indexing-1.0`, `IndexingEventPayload`, dan
`KafkaIndexingConsumerService`. Kafka **justru** transport indexing.
**Dampak:** urutan Fase 6 (OpenSearch) dan Fase 7 (Kafka) di roadmap terbalik —
kalau mau meniru CCWR, Kafka datang **bersama** indexing, bukan sesudahnya.

### 🔴 G2. Tidak ada domain `Setting` di rencana
Kita sudah memutuskan minimal 4 hal config-driven (GTM ID, password expiry 90
hari, daftar Popular Searches statis, consent) — di CCWR **semuanya** baris
`SiteSetting`, bukan `application.yml`. Tanpa ini, tiap perubahan butuh deploy.

### 🔴 G3. Versioning konten kosong di X2
CLAUDE.md pattern #4 menyebut draft/publish/preview + versioning sebagai pola yang
layak direplikasi, tapi ERD X2 kita cuma punya `status` 3 nilai — tanpa `origin`/
`master`/`publishedVersion`. Padahal Fase 6 kita berencana bikin index
`preview_articles`, yang **tidak mungkin** tanpa mekanisme shadow/draft ini.

### 🟠 G4. `Media` diperlakukan sebagai `String`
X2 memutuskan `featuredImage` dan `ogImage` sebagai String. CCWR punya domain
`Media` penuh: `alt` (wajib untuk aksesibilitas + SEO), `copyright`,
`urlThumbnail`, `width`/`height`, `fileSize`, `publishDate`/`archiveDate`,
`s3ObjectStatus` (lifecycle unggah). Keputusan X4 kita "pakai S3" praktis
mensyaratkan `s3ObjectStatus`-semacam ini.

### 🟠 G5. Tidak ada keputusan sanitasi input
Rencana kita punya **form komentar publik** (teks bebas) dan **body artikel
WYSIWYG** — dua permukaan XSS paling klasik — tanpa satu pun keputusan sanitasi.
CCWR punya `ContentUtils.hasBlacklistedTags` + `SanitizingUtils` + `jsoup` di
semua modul.

### 🟠 G6. Tidak ada rate limiting
Komentar anonim tanpa akun adalah target spam paling jelas, dan filter kata
terlarang tidak menahan flooding. CCWR punya `HttpRequestLogInterceptor`
(30 req/120 detik per IP).

### 🟠 G7. Navigasi diasumsikan hardcoded
Sitemap X1 kita menyebut menu `News/Review/Guide/Games/Search` tanpa memutuskan
apakah itu data atau kode. CCWR: `Menu` (tree, `orderNo`, `MenuType`,
`MenuLocation`) + `Structure`/`StructureDetail`.

### 🟡 G8. Interceptor tidak pernah disebut
`RequestModel` CCWR (header, footer, menu, popup, consent, GTM) dirakit di
`GeneralInterceptor` untuk setiap request. Rencana kita belum menyebut lapisan ini
sama sekali, padahal semua elemen global di wireframe X1 butuh tempat dirakit.

### 🟡 G9. `views-json` (.gson) tidak ada di rencana dependency
Semua service CCWR memakainya untuk render JSON (`notFound.gson`,
`badRequest.gson`). Envelope mereka `BaseResponse<T>` = `{data, message, errors}`
dengan `@JsonInclude(NON_NULL)`; kita memilih `{data, meta}` + `{error:{code,
message}}`. Deviasi ini sah, tapi harus sadar — dan plugin `views-json` belum
masuk daftar dependency kita.

### 🟡 G10. `Email`/`EmailLog` sebagai domain
Kita memutuskan MailHog + plugin `mail`, tanpa domain. CCWR menyimpan template dan
log pengiriman di DB — relevan untuk forgot-password dan Newsletter nanti.

### 🟡 G11. Strategi testing masih "ditunda", padahal CCWR punya contoh jelas
Spock (unit) + **Geb/Selenium** (browser) di `web_portal`, `api`, `sims`.

### ⚪ G12. Detail kecil yang layak dicatat
Slug di entitas detail (bukan induk) · `autoTimestamp false` + `disableTimestamp` ·
`ES_SORTABLE_FIELDS` di domain · validator reserved-keyword untuk slug redirect ·
`Caffeine` sebagai cache lapis kedua di `web_portal` · index OpenSearch untuk
audit/request log · `PendingDownload` untuk export async.

---

## 6. Yang sudah benar di rencana kita

Supaya seimbang — hasil audit ini **tidak** membatalkan keputusan besar yang sudah
diambil:

- 3 modul + `shared_lib` sebagai plugin — benar, sesuai pola CCWR
- `web_portal` tanpa datasource — benar, terkonfirmasi 0 domain di CCWR
- Feign + Consul — benar
- `Game.gameType`/`baseGame` meniru `Product.variantOf`/`kitOf` — benar
- Spring Security Core dengan `Role`/`Permission`/`UserRole`/`PermissionRole` +
  `UserToken` + `UserPasswordHistory`, `admin_portal` punya DB sendiri — benar,
  persis pola CCWR (dan G-2 di atas justru memperkuatnya)
- `audit-logging` plugin penuh — benar
- Liquibase **meski CCWR tidak punya** — tetap benar, ini penyimpangan sadar
- Tanpa HTTP `QUERY` — benar, CCWR nihil
- Ad network eksternal, bukan `Banner` internal — sah sebagai penyimpangan sadar
