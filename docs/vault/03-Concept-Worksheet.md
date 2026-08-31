# Concept Worksheet

**Status: terisi — 2026-08-28.** Diringkas dari
`slow-response_3-concept workout.docx` yang dikerjakan sendiri di luar repo.
Draft asli (Word) masih ada di root repo tapi belum ditrack di git — lihat
catatan di [[00-Index]] soal ini.

Isinya boleh direvisi kapan saja kalau nanti ada yang berubah pikiran di
tengah Fase 2 — bukan kontrak mati, cuma keputusan berjalan.

---

## 1. Riset pembanding

| Situs | Jenis konten | Skor review | Navigasi utama | Yang paling menarik |
|---|---|---|---|---|
| kokanggaming.com | News, Reviews, Features & Previews, Gacha News, Hardware & Gadget, Technology & Entertainment | Tidak pakai skor angka — kualitatif: Kesimpulan, Pros, Cons, rekomendasi audiens | Latest Post, Reviews, Features, Gaming (PlayStation/Xbox-PC/Nintendo/Gacha), Hardware, Technology, Entertainment | Gaya tulis matang & mendalam (jurnalisme game berpengalaman), format review fokus esensi tanpa angka, UI/UX bersih + dark mode, kategori tersegregasi rapi (Gacha, Hardware) |
| thelazy.media | Gaming News & Updates, Game Reviews & Impressions, Tech & Hardware, Pop Culture & Entertainment, Editorial/Opini & Features | Umumnya tanpa skor angka kaku — verdict + Pros/Cons + "worth it atau tidak" | Home, Games, Tech/Hardware, Reviews/Lazy Review, Pop Culture/Entertainment, Features/Editorial | Gaya bahasa santai & komunikatif, ekosistem multi-platform kuat (terhubung YouTube), dukungan kuat ke industri & komunitas lokal (The Lazy Game Awards) |
| jagatplay.com | News (game & industri, termasuk hardware/teknologi pendukung), Reviews (multi-halaman, long-form), Previews, Gaming Gear Reviews, Top List & Features | Tidak pakai skor angka — Kesimpulan, Kelebihan, Kekurangan, rekomendasi persona pemain | Home, Channels (PC/PlayStation/Nintendo/Gaming Gear/Top List/Time Machine), Networks (Jagat Review/Gadget/Overclocking/Review TV), About/Contact | Ulasan sangat detail & objektif (multi-halaman, tidak ragu kritik game hyped), rubrik Top List & Time Machine terkurasi, sinergi hardware & software (Jagat Review Network) |

**Pola mana yang paling cocok, dan kenapa:** Gabungan pola Kokang Gaming +
JagatPlay ("Editorial & Structured Long-form"). Format review mereka punya
entitas jelas dan konsisten (Plot/Gameplay/Visual → Kesimpulan → Pros/Cons →
Rekomendasi Persona) yang ideal dipetakan ke skema relasional MySQL + DTO
shared library. Format multi-halaman keduanya juga memberi skenario nyata
untuk uji indexing OpenSearch, cache Hazelcast per halaman, dan payload Kafka
saat artikel publish/update. Taksonomi platform (PC/PlayStation/Nintendo)
mereka juga memudahkan query agregasi & filter di OpenSearch.

**Yang mereka SEMUA punya (kemungkinan wajib di situs game manapun):**
- Format review berbasis Pros/Cons/Verdict tanpa skor angka mutlak — ketiganya
  sama-sama meninggalkan rating numerik kaku
- Navigasi berbasis platform/hardware (PC, PlayStation, Nintendo dipisah jadi
  kanal sendiri)
- Tiga pilar konten standar: News (arus cepat), Review (analisis mendalam),
  Features/Opini (sudut pandang editorial)

**Yang cuma SATU situs punya (pembeda, bukan wajib):**
- Kategori "Gacha" sejajar platform konsol utama (Kokang Gaming)
- Rubrik "Time Machine" — kilas balik game lawas (JagatPlay)
- Integrasi erat ke ekosistem komunitas & awards, terhubung ke kanal video
  (The Lazy Media)

---

## 2. Identitas

- **Nama publik situs:** Slow Response — dipertahankan sebagai brand publik,
  bukan cuma nama teknis repo. Diposisikan sebagai *unhurried/reflektif*
  (selaras tagline), bukan "lambat merilis berita" — kalau dipresentasikan,
  siapkan satu kalimat penjelas biar gak salah baca.
- **Tagline:** "Ruang baca tenang bagi gamer yang mencari ulasan mendalam dan
  cerita di balik layar."
- **Target pembaca:** Core & Single-Player / Narrative Gamers (gamer
  dewasa/pekerja) — bukan casual atau kompetitif/esports.
- **Cakupan platform:** fokus utama PC & konsol modern (PlayStation, Nintendo,
  Xbox). Mobile/gacha sengaja tidak jadi fokus utama.

---

## 3. Pilar konten

- [x] News — berita singkat, rilis, update
      (read-heavy, throughput tinggi, payload kecil — uji listing kronologis
      cepat via OpenSearch/Hazelcast)
- [x] Review — penilaian game mendalam, kesimpulan, pros/cons
      (model relasional kompleks di MySQL: rich-text, metadata game, pagination)
- [x] Guide — walkthrough, tips & trik
      (uji full-text search OpenSearch berdasarkan keyword game & sub-bab konten)
- [ ] Feature/Opinion — **sengaja ditunda ke fase berikutnya**, biar kontrak
      enum `ArticleType` di Fase 1 gak over-engineered dari awal

---

## 4. Skema skor review

- **Skala:** 1–10, desimal 1 digit (misal 8.5). Tipe data `BigDecimal` /
  `DECIMAL(3,1)` di MySQL — cukup granular tanpa serumit skala 1–100, dan
  gampang dipakai buat range query di OpenSearch (`score >= 8.0`).
- **Model skor:** skor tunggal editorial (Overall Score), didukung ringkasan
  kualitatif Pros/Cons/Verdict — **bukan** breakdown per aspek
  (Gameplay/Graphics/Story/Sound). Sengaja dihindari karena gak semua game
  relevan dinilai dengan bobot aspek yang sama, dan breakdown butuh tabel
  anak tambahan cuma buat kalkulasi rata-rata.
- **Siapa yang kasih skor:** editor situs saja (lewat Admin Portal), untuk
  fase awal. Rating publik/pembaca ditunda — menghindari kompleksitas
  write-heavy (auth publik, proteksi review-bombing, agregasi skor,
  cache invalidation storm di Hazelcast) sebelum waktunya.

**Catatan penting:** ketiga situs pembanding di bagian 1 semuanya **tidak**
pakai skor angka. Keputusan pakai skala 1–10 di sini sengaja menyimpang dari
pola itu — alasannya murni kebutuhan arsitektur (field numerik buat uji
range query OpenSearch), bukan mengikuti tren pasar. Simpan alasan ini kalau
nanti ditanya kenapa beda dari referensi.

---

## 5. Relasi Article ↔ Game

- **Kardinalitas:** many-to-many. Satu artikel REVIEW umumnya terikat ke 1
  game primer, tapi NEWS bisa menyinggung crossover 2 game, dan GUIDE/Top
  List bisa mengulas banyak game sekaligus. Satu game juga muncul di banyak
  artikel sepanjang siklusnya (NEWS pengumuman → REVIEW rilis → GUIDE tips).
- **Status `Game`:** first-class entity, bukan sekadar metadata pasif. Punya
  halaman index sendiri (`/games/{slug}`) yang mengagregasi metadata dasar
  (title, slug, developer, publisher, release date, platforms, cover image),
  skor review resmi (kalau sudah ada artikel REVIEW-nya), dan feed semua
  artikel (NEWS/REVIEW/GUIDE) yang terhubung ke game itu.

**Catatan ke depan (belum urgent):** karena `Game` first-class dengan halaman
hub sendiri, di Fase 6 kemungkinan butuh index OpenSearch terpisah untuknya
(mirip `PRODUCT` di enum `OpenSearchIndex` CCWR), bukan cuma index `ARTICLE`.

---

## 6. Taksonomi

- **Genre → Tag**, bukan Category. Genre adalah sifat `Game` (RPG, Action,
  Roguelike), sering ganda/hybrid pada satu game — kalau dipaksa jadi
  Category artikel akan bingung ditaruh di mana. Sebagai Tag/relasi M:N di
  `Game`, sistem tetap fleksibel.
- **Platform → field Enum/Set di `Game`**, bukan tag teks bebas. Di
  `content_service` (MySQL) jadi relasi M:N sederhana (`game_platforms`)
  berbasis enum (`PC`, `PLAYSTATION_5`, `XBOX_SERIES`, `NINTENDO_SWITCH`,
  `MOBILE`). Field ini diwariskan ke payload Kafka saat publish, lalu ke
  dokumen OpenSearch (`platforms: ["PC", "PLAYSTATION_5"]`), dan dipakai
  langsung sebagai term filter/aggregation di menu navigasi Web Portal.
- **Level hirarki kategori:** flat, 1 level saja. Sumbu pengelompokan sudah
  dibagi tiga: `ArticleType` (NEWS/REVIEW/GUIDE) untuk jenis konten, Platform
  untuk filter, Tag/Game untuk topik/genre — jadi kategori bersarang
  (self-referencing FK, recursive CTE) gak diperlukan. Routing URL Web Portal
  jadi bersih: `/{articleType}/{slug}` atau `/{articleType}?platform=pc`.

---

Ringkasan resmi hasil worksheet ini sudah dicatat sebagai satu entry di
[[04-Decisions-Log]] ("Domain model Article & Game — versi 1") — buka itu
kalau cuma butuh versi ringkas buat eksekusi Fase 2, worksheet ini untuk
alasan lengkapnya.
