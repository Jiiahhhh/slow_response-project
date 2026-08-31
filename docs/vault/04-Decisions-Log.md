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

## 2026-08-28 Domain model Article & Game — versi 1

**Konteks:** riset pembanding 3 situs media game lokal (kokanggaming.com,
thelazy.media, jagatplay.com — detail lengkap di
[[03-Concept-Worksheet]] bagian 1) selesai, sekaligus jadi dasar keputusan
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
[[03-Concept-Worksheet]]. Yang menonjol untuk dicatat di sini: skor numerik
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
