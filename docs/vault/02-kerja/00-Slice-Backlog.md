# Slice Backlog

Ini backlog kerja harian. [[03-Tahap3-Build]] menjawab "fase ini isinya apa";
file ini menjawab **"sesi 90 menit hari ini ngerjain apa"**.

Dipakai bersama skill `/sesi` di `.claude/skills/sesi/`. Aturan memotongnya ada di
`.claude/skills/sesi/references/cara-memotong-slice.md`.

Baris **Prasyarat** di tiap slice merujuk ke [[01-Peta-Kompetensi]]. Slice tidak boleh
dimulai sebelum prasyaratnya berstatus `LULUS`.

---

## Status sekarang

- **Fase:** 2 — `content_service` sebagai write side — **BELUM DIMULAI**
- **Gate Tahap: LOLOS** (2026-09-04) — [[01-Tahap1-Discovery]] dan
  [[02-Tahap2-Design]] keduanya SELESAI. `/sesi` sekarang boleh menawarkan
  slice di bawah — tapi cek dulu gate kompetensi (Prasyarat tiap slice)
  sebelum mulai.
- **Riwayat:** draft domain pertama (8 domain + 8 Spec) ditulis lalu dihapus
  2026-09-02 karena mendahului Tahap 1-2. Lihat [[00-Decisions-Log]].
- **S2-1 s.d. S2-8 di bawah masih isi LAMA** (ditulis sebelum X1-X4 final) —
  jangan dikerjakan apa adanya. Perlu ditulis ulang dulu dari hasil final
  X1 (sitemap/taksonomi), X2 (ERD — `Article`/`Review`/`Game` dengan
  `gameType`/`baseGame`/`ArticleGame`/`Tag`/`Author`), X3 (kontrak API),
  X4 (S3, Liquibase, Spring Security Core 7 domain, audit-logging) sebelum
  sesi ngoding berikutnya dimulai.

---

## Fase 2 — `content_service`

**Kriteria fase (dari [[03-Tahap3-Build]]):**
`curl localhost:9089/api/articles` mengembalikan JSON artikel dari MySQL, bukan H2.

Urutannya sengaja mengejar satu hal: **bikin jalur tipis dari MySQL sampai HTTP hidup
secepat mungkin (S2-3)**, baru sesudah itu disempurnakan. Draft pertama fase ini
(delapan domain class tanpa satu pun endpoint hidup, lihat [[00-Decisions-Log]]
2026-09-02) adalah contoh potongan horizontal — itu yang bikin fase terasa tidak
selesai-selesai. Jangan ulangi polanya: setelah [[02-Tahap2-Design]] tuntas, kejar
S2-3 dulu, bukan menulis semua domain sekaligus.

### [ ] S2-1 — Buktikan skema tercipta di MySQL

> **USANG.** Ditulis saat 8 domain draft masih ada; domainnya sudah dihapus
> 2026-09-02 ([[00-Decisions-Log]]). Tulis ulang slice ini setelah X2 di
> [[02-Tahap2-Design]] final — jangan dikerjakan dari isi di bawah ini apa adanya.

- **Prasyarat:** F-02 — commit atomik — 16 file menggantung ini justru bahan latihannya
- **Konsep:** `dbCreate: update`, bagaimana GORM menerjemahkan domain jadi DDL,
  penamaan tabel join, kenapa `environments.development` menimpa blok `dataSource`
  di atasnya
- **Sentuh:** `content_service/grails-app/conf/application.yml` (verifikasi saja),
  lalu commit 16 file yang menggantung
- **Selesai jika:** `docker compose up -d` jalan, `content_service` hidup, dan
  `SHOW TABLES` di `capstone_db` menampilkan tabel untuk kedelapan domain —
  lalu `git status` bersih
- **Perkiraan:** 60 menit

### [ ] S2-2 — Mekanisme seed data

- **Prasyarat:** F-03 — urutan insert & foreign key menuntut paham relasi
- **Konsep:** `BootStrap.groovy` dan siklus hidupnya, kenapa seed harus idempoten,
  kenapa seed tidak boleh hidup di environment `production`
- **Pembagian:** kamu tulis mekanismenya (guard environment, cek "sudah ada belum",
  urutan insert supaya foreign key tidak melanggar). Claude yang menyiapkan datanya —
  standar datanya di `.claude/skills/sesi/references/standar-mutu.md`
- **Selesai jika:** app dijalankan dua kali berturut-turut, `SELECT COUNT(*) FROM game`
  tetap sama (tidak menggandakan)
- **Perkiraan:** 90 menit

### [ ] S2-3 — Jalur tipis: MySQL → HTTP

- **Prasyarat:** F-04, F-06 — ini slice yang paling butuh fondasi: kenapa logika di service, dan di mana transaksi digambar
Ini slice terpenting di seluruh Fase 2. Setelah ini sistemnya punya denyut.

- **Konsep:** service layer sebagai batas transaksi, `@Transactional(readOnly = true)`,
  kenapa penyaringan `status` wajib di service bukan di controller, `respond()`
- **Sentuh:** `ArticleService.groovy`, `ArticleController.groovy`, `UrlMappings.groovy`
- **Selesai jika:** `curl localhost:9089/api/articles` mengembalikan JSON berisi
  artikel dari MySQL, dan **tidak ada satu pun artikel DRAFT di dalamnya**
- **Perkiraan:** 90 menit
- **Kalau menit ke-50 service belum jalan:** potong. Commit service + Spec-nya,
  controller jadi slice S2-3b. Jangan dipaksa sampai lewat waktu.

### [ ] S2-4 — Response lewat DTO, bukan domain class

- **Prasyarat:** F-05 — DTO itu soal kontrak, bukan soal menyalin field
- **Konsep:** kenapa mengembalikan domain class itu bocor sekaligus rapuh
  (`LazyInitializationException`), DTO sebagai kontrak antar modul, kenapa dia tinggal
  di `shared_lib` dan bukan di `content_service`
- **Sentuh:** `shared_lib/.../response/ArticleResponse.groovy` (isi field-nya),
  mapper di `ArticleService`
- **Selesai jika:** response JSON tidak lagi memuat field internal
  (`version`, `class`, `errors`), dan `curl` tetap berhasil
- **Perkiraan:** 60 menit

### [ ] S2-5 — Detail artikel by slug + penanganan 404

- **Prasyarat:** — (lanjutan S2-4)
- **Konsep:** slug sebagai identifier publik (kenapa bukan id), status code HTTP yang
  benar, bentuk error response yang bisa dipakai klien
- **Selesai jika:** `curl -i .../api/articles/<slug-valid>` → 200 dengan isi;
  `curl -i .../api/articles/slug-ngawur` → 404 dengan body JSON, bukan halaman
  error HTML Grails
- **Perkiraan:** 60 menit

### [ ] S2-6 — Filter type + pagination

- **Prasyarat:** — (tidak ada konsep fondasi baru)
- **Konsep:** `max`/`offset` di GORM, kenapa endpoint listing tanpa batas itu
  bom waktu, cara mengembalikan total count tanpa query kedua yang mahal
- **Selesai jika:** `curl '.../api/articles?type=REVIEW&max=10&offset=10'`
  mengembalikan tepat 10 artikel REVIEW, dan tanpa parameter pun ada batas default
- **Perkiraan:** 90 menit

### [ ] S2-7 — Data klien penuh + berburu N+1

- **Prasyarat:** F-09 — mengukur dulu sebelum memperbaiki
- **Konsep:** membaca `logSql`, mengenali N+1, `join fetch`
- **Pembagian:** Claude melengkapi seed jadi 25-30 game & 25-30 artikel lengkap
  dengan edge case; kamu yang memperbaiki query-nya
- **Selesai jika:** dengan data penuh, satu request listing memicu jumlah query yang
  tidak bertambah seiring jumlah artikel di halaman
- **Perkiraan:** 90 menit

### [ ] S2-8 — Tutup fase

- **Prasyarat:** F-12 — menutup fase itu soal menulis untuk orang lain
- **Selesai jika:** checklist "orang lain bisa jalanin" di `standar-mutu.md` lolos
  semua, `CLAUDE.md` bagian "Kondisi saat ini" diperbarui, merge ke `main`, tag `v0.2.0`
- **Perkiraan:** 60 menit

---

## Belum dijadwalkan

Tempat menampung ide yang muncul di tengah sesi. Jangan dikerjakan saat muncul —
tulis di sini, lanjutkan slice yang sedang jalan.

- X4 perlu putuskan mekanisme role: tabel `Role`/`Permission`/`UserRole`/
  `PermissionRole` ala CCWR (`admin_portal/grails-app/domain/`) vs `enum role`
  sederhana di `User`. Dipicu keputusan D1 (2026-09-03) — Admin sengaja belum
  dipecah per master data, tapi rencananya iya, jadi mekanismenya perlu siap
  untuk itu. **Terkait:** X4 juga perlu putuskan apakah `admin_portal` punya
  DB sendiri (kayak `canon_admin_portal` CCWR) atau tidak — dua keputusan ini
  saling gantung, putuskan bareng, jangan terpisah.
- **"Popular Searches" di halaman search** — mode STATIC (daftar manual
  config-driven), bukan integrasi Google Analytics beneran (mode DYNAMIC
  CCWR — gak ada gunanya tanpa trafik asli). Dibahas detail pas Fase 6
  (search dibangun). Detail temuan CCWR di [[00-Decisions-Log]] (2026-09-04).
- **Kandidat dari sapuan CCWR 2026-09-03** (dipertimbangkan pas fase relevan,
  belum diputuskan in/out): patch notes/changelog artikel (analogi
  `ApiChangeLog`), related content component (analogi `PageRelatedReadsComp`),
  social share component, popup on-site (analogi `Popup`), Important Notice
  (overlap konsep sama Banner — pilih salah satu atau dua-duanya?), audit
  trail generik (juga disebut CLAUDE.md pattern #6). Detail di
  [[00-Decisions-Log]] (2026-09-03).
- **Slice orisinal (setelah Tahap 3 inti selesai):** fitur komentar pembaca.
  Domain `Comment` (`content`, `authorName`, `authorEmail` — gak pernah masuk
  DTO publik, `authorWebsite` opsional, `status` APPROVED/DELETED, FK ke
  `Article`), auto-publish tanpa moderasi, Editor/Admin cuma bisa hapus
  (bukan edit), filter kata SARA/kasar model **sensor** (regex replace jadi
  `***`, bukan tolak submit) lewat tabel `BadWord` di DB `content_service`,
  dicek langsung tanpa Hazelcast. Detail & alasan lengkap di
  [[00-Decisions-Log]] (2026-09-03).

---

## Fase 3 ke atas

Belum dipotong jadi slice — potong saat Fase 2 hampir selesai, bukan sekarang.
Memotong fase yang masih jauh selalu meleset, karena keputusan di fase sebelumnya
mengubah bentuknya.
