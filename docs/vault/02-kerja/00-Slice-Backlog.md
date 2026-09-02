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
- **Blocker:** Gate Tahap. [[01-Tahap1-Discovery]] dan [[02-Tahap2-Design]] belum
  SELESAI — `/sesi` akan mengalihkan ke `/planning`, bukan menawarkan slice di
  bawah ini.
- **Riwayat:** draft domain pertama (8 domain + 8 Spec) ditulis lalu dihapus
  2026-09-02 karena mendahului Tahap 1-2. Lihat [[00-Decisions-Log]].
- **S2-1 akan ditulis ulang** setelah X2 ([[02-Tahap2-Design]]) final — isinya
  di bawah masih versi lama, jangan dikerjakan apa adanya.

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

- Keputusan: `Review` sekarang one-to-one dengan `Article` (`article unique: true`).
  Konsekuensinya untuk agregasi skor per `Game` belum dipikirkan — perlu dibahas
  sebelum Fase 4.
- `Category` disebut di roadmap Fase 2 tapi tidak ada di domain yang sudah ditulis.
  Sengaja dibuang atau kelewat? Perlu diputuskan dan dicatat di [[00-Decisions-Log]].

---

## Fase 3 ke atas

Belum dipotong jadi slice — potong saat Fase 2 hampir selesai, bukan sekarang.
Memotong fase yang masih jauh selalu meleset, karena keputusan di fase sebelumnya
mengubah bentuknya.
