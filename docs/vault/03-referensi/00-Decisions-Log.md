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
