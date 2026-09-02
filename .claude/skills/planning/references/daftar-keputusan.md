# Daftar Keputusan

Semua keputusan arah yang perlu diambil, mana yang sudah, mana yang masih terbuka.
Isi keputusan final tetap ditulis di `docs/vault/03-referensi/00-Decisions-Log.md` — file ini
cuma daftar antrean dan bahan mentahnya.

Kolom CCWR di bawah adalah hasil pemeriksaan langsung ke
`/Users/ilal/canon-corporate-website-revamp`. Kalau kamu perlu detail lebih jauh,
**grep dulu** — jangan menambah klaim dari ingatan.

---

## Sudah diputuskan

| Keputusan | Pilihan | Catatan |
|---|---|---|
| Framework | Grails 6.2.3 | mengikuti CCWR |
| Java / Gradle | 17 / 7.6.4 | Grails 6.2.3 tidak kompatibel dengan Gradle 9 |
| Modul | 3 aplikasi + `shared_lib` plugin | `api`, `pim`, `sims` dibuang — lihat Decisions Log 2026-08-17 |
| Database | MySQL 8.0 | mengikuti CCWR |
| `shared_lib` | Grails plugin, bukan library biasa | supaya bisa membawa artefak Grails |
| Search | OpenSearch 2.19 | mengikuti CCWR |
| Cache | Hazelcast 5.5 | mengikuti CCWR |
| Service discovery | Consul | mengikuti CCWR |
| Messaging | Kafka, cakupan sempit | CCWR memakainya hanya untuk user & komentar, bukan indexing |
| Relasi Article↔Game | join domain eksplisit | Decisions Log 2026-08-31 |
| Secret | environment variable | menolak anti-pola CCWR |

---

## Terbuka — wajib diputuskan di Tahap 2 (X4)

Diurut dari yang paling menghambat. Empat yang pertama menghalangi M2, dan M2 tidak
bisa dimulai sebelum keempatnya selesai.

### 1. Penyimpanan gambar — *menghambat M1 dan M2*

**CCWR:** domain `Media` di `shared_lib/src/main/groovy/shared_lib/domain/Media.groovy`,
diunggah ke S3 lewat `shared_lib/src/main/groovy/shared_lib/amazons3/AmazonS3Service.groovy`
(SDK `software.amazon.awssdk:s3`). Ada `VariantMediaGroup` — artinya satu gambar
disimpan dalam beberapa ukuran. Ada `S3ObjectStatus`, jadi status unggahan dilacak.

**Yang perlu diputuskan:** ikut S3 (butuh akun AWS, biaya, kredensial) atau pakai
S3-compatible lokal seperti MinIO di docker-compose (gratis, API sama persis, bisa
ditukar ke S3 asli nanti tanpa ubah kode), atau filesystem lokal (paling sederhana,
tapi polanya beda dari CCWR dan tidak bisa dipakai kalau nanti ada lebih dari satu
instance).

**Bahan pertimbangan:** ini pintu satu arah kalau salah pilih abstraksinya, tapi pintu
dua arah kalau abstraksinya benar. Pertanyaan sebenarnya bukan "S3 atau lokal", tapi
"apakah kode kita bicara ke antarmuka penyimpanan atau ke S3 langsung".

**Ikutannya:** perlu domain `Media` sendiri atau cukup kolom `String` di `Article`?
Perlu varian ukuran (thumbnail, cover, OG image)? Untuk situs media, jawabannya hampir
pasti perlu.

### 2. Autentikasi admin — *menghambat M2*

**CCWR:** `org.grails.plugins:spring-security-core:6.0.0`. Domainnya lengkap:
`User`, `Role`, `UserRole`, `Permission`, `PermissionRole`, `Profile`,
`UserLoginHistory`, `UserPasswordHistory`, `UserResetPasswordAttempt`, `UserToken`.
Jadi CCWR punya RBAC penuh plus riwayat login, riwayat password, dan pembatasan
percobaan reset.

**Yang perlu diputuskan:** seberapa banyak dari itu yang ditiru. Sepuluh domain auth
untuk proyek satu orang jelas berlebihan; tapi `User`/`Role`/`UserRole` adalah inti
Spring Security Core dan tidak bisa dikurangi.

**Bahan pertimbangan:** peran apa yang benar-benar ada di situs media — Admin, Editor,
Author? Bedanya apa? Kalau tidak ada bedanya, satu peran saja dulu dan tambahkan saat
perbedaannya nyata.

### 3. Migrasi database — *menghambat M5, tapi makin mahal makin ditunda*

**CCWR:** tidak ditemukan folder `migrations` di mana pun. Plugin database-migration
cuma disinggung di baris `excludes`. Artinya CCWR pun tampaknya tidak memakai migrasi
skema — ini contoh di mana **rujukan tidak layak diikuti**.

**Yang perlu diputuskan:** Liquibase (plugin resmi Grails, XML/Groovy DSL, mendukung
rollback) atau Flyway (SQL polos, lebih sederhana, lebih mudah dibaca) atau tetap
`dbCreate: update` (tidak layak produksi, dan menutup pintu ke M5).

**Bahan pertimbangan:** ini pintu satu arah dalam arti waktu — makin banyak tabel yang
sudah ada, makin repot memulainya. Kalau mau dipakai, mulai sekarang selagi baru
delapan tabel.

### 4. Editor teks di admin — *menghambat M2*

Artikel media butuh isi kaya: heading, gambar di tengah teks, kutipan, embed video.
Pilihannya editor WYSIWYG (TinyMCE, CKEditor) atau Markdown.

**Bahan pertimbangan:** WYSIWYG lebih ramah editor non-teknis; Markdown jauh lebih
mudah disimpan, di-diff, dan diindeks. Pertanyaannya: siapa yang akan menulis di situs
ini dalam skenario latihan ini?

### 5. Frontend `web_portal`

**CCWR:** GSP server-side, 377 file view, plus asset-pipeline.

**Bahan pertimbangan:** konsistensi dengan CCWR menyarankan GSP, dan untuk situs media
server-side rendering memang pilihan yang benar — SEO adalah nyawa produk ini, dan itu
argumen teknis yang berdiri sendiri, bukan sekadar ikut CCWR.

### 6. Audit trail

**CCWR:** `org.grails.plugins:audit-logging:4.0.3`, domain `AuditTrail`, dan
`TimestampFields`/`BaseDomain` di `shared_lib`.

**Yang perlu diputuskan:** pakai plugin penuh, atau cukup `dateCreated`/`lastUpdated`
bawaan GORM. Yang jelas: sekarang domain proyek ini **tidak punya keduanya** — bahkan
`dateCreated` pun belum ada, dan itu kekurangan yang harus ditutup apa pun keputusannya.

### 7. Versioning API

Bentuk URL (`/api/v1/articles` atau `/api/articles`), format error, aturan pagination.
Pintu dua arah selama belum ada konsumen di luar kendali — putuskan cepat, jangan
dibesarkan.

### 8. Multi-site dan multi-bahasa

**CCWR:** `Site`, `Language`, `SiteLanguage`, tabel `*Translation`, dan index OpenSearch
dipecah per site+bahasa.

**CLAUDE.md** sudah mengarahkan: boleh disederhanakan ke satu site, tapi konsepnya
dipertahankan. Yang perlu diputuskan: "mempertahankan konsep" itu artinya apa secara
konkret — menyimpan kolom `siteId` yang selalu bernilai sama, atau cukup memahami
polanya tanpa menuliskannya.

### 9. Target deployment — *dibutuhkan di R2*

Belum mendesak, tapi jangan sampai baru dipikirkan di bulan keempat. Cukup putuskan
arahnya sekarang: docker-compose di VPS, atau cukup berhenti di lingkungan lokal dan
mendokumentasikan cara deploy-nya.

### 10. Strategi testing

Unit saja, atau unit plus integration. Terkait erat dengan F-07 di kurikulum fondasi.
Kondisi sekarang: delapan file Spec ada tapi seluruhnya masih stub bawaan yang gagal.

---

## Urutan pembahasan yang disarankan

Sesi X4 dijadwalkan 1-2 sesi. Tidak semua sepuluh dibahas sekaligus.

**Sesi pertama** — yang menghambat M1 dan M2: penyimpanan gambar (1), autentikasi (2),
editor teks (4), audit trail (6).

**Sesi kedua** — migrasi (3), frontend (5), versioning API (7), multi-site (8).

**Ditunda sampai dibutuhkan** — deployment (9) dibahas di akhir M4; testing (10)
dibahas bersamaan dengan mentoring F-07.
