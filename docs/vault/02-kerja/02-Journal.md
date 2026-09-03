# Journal

Log sesi kerja. Tiap habis ngoding, tambahin entry baru di atas — 3-5 baris
cukup, gak perlu panjang. Gunanya: pas kamu balik lagi setelah jeda (kayak
sebelum sesi ini, sempat lama gak sentuh proyek), kamu punya jejak "terakhir
kali aku ngapain dan macet di mana".

Template:

```
## [tanggal]
- Ngerjain: ...
- Macet di: ...
- Kepelajari: ...
- Lanjut ke: ...
```

---

## 2026-09-03 s.d. 2026-09-04
- Ngerjain: **Tahap 1 dan Tahap 2 tuntas semua** dalam satu sesi panjang. D1
  (4 peran: Pembaca/Editor/Admin/Super Admin), D2 (tabel fitur, Hardware/Tech
  dibuang), X1 (sitemap + `/games` listing, taksonomi — `Category` dibuang,
  wireframe: featured Game bukan featured Article), X2 (ERD lengkap +
  `Game.gameType`/`baseGame` ala `pim.Product`), X3 (kontrak API, tanpa
  versioning, `QUERY` di-skip), X4 (S3, Liquibase, WYSIWYG, GSP, Spring
  Security Core 7 domain + DB terpisah `admin_portal`, expiry 90 hari,
  MailHog). Ditutup dengan **audit menyeluruh 7 modul CCWR** →
  [[02-Peta-CCWR-Lengkap]].
- Macet di: bukan macet, tapi dua kali harus menarik balik rekomendasi sendiri —
  agregasi skor Review (dikoreksi pola Metacritic + `pim.Product`) dan
  `UserToken`/`UserPasswordHistory` (tadinya mau di-skip, ternyata inti flow).
- Kepelajari: grep reaktif per topik itu rapuh — cuma nemu apa yang sudah
  ditanyakan. Audit menyeluruh nemu 12 gap, termasuk **CLAUDE.md sendiri salah**
  soal Kafka (ternyata transport indexing, bukan cuma komentar/user sync).
- Lanjut ke: tutup G1-G3 (Kafka di CLAUDE.md, domain `Setting`, versioning
  konten di X2) sebelum ngoding, lalu tulis ulang slice S2-1 dst dari hasil
  X1-X4 — isi lama di [[00-Slice-Backlog]] sudah usang.

---

## 2026-09-02
- Ngerjain: reorganisasi vault jadi 4 folder (rencana/kerja/referensi/arsip),
  bikin Grand Plan + tracker per Tahap (1-4), bikin skill `/planning`,
  `/mentoring`, `/debug` di samping `/sesi`, dan pasang Gate Tahap + Gate
  Kompetensi di `/sesi` supaya dia jadi hub yang mengarahkan sendiri. Ketahuan
  lewat rekonsiliasi: 8 domain class ditulis sebelum Tahap 1-2 selesai — dihapus
  (belum pernah commit, tanpa sunk cost), `content_service` kembali kosong.
- Macet di: tidak ada — ini sesi perencanaan/tooling, bukan sesi ngoding.
- Kepelajari: potongan kerja yang benar itu bukan cuma soal ukuran (slice
  60-90 menit), tapi juga soal urutan — nulis domain sebelum desain final itu
  bentuk lain dari lompat fase, walau terasa produktif waktu itu.
- Lanjut ke: `/sesi` akan mengarahkan ke `/planning` untuk D1 (Product Brief) —
  Tahap 1 dan 2 harus tuntas dulu sebelum `content_service` disentuh lagi.

---

## 2026-08-31
- Ngerjain: nutup Fase 1 — bikin `shared_lib` sebagai Grails plugin (bukan app),
  isi enum `ArticleType` (NEWS/REVIEW/GUIDE), `PublishStatus`
  (DRAFT/PUBLISHED/ARCHIVED), DTO kosong `ArticleResponse`. Disambungkan ke
  ketiga modul (`web_portal`, `admin_portal`, `content_service`) lewat
  `implementation project(':shared_lib')`, dibuktikan jalan lewat import uji
  coba di `BootStrap.groovy` masing-masing (sudah dibersihkan lagi setelahnya).
  Juga selesai isi seluruh Concept Worksheet (riset 3 situs pembanding,
  identitas produk, skema skor review, relasi Article↔Game, taksonomi) dan
  rangkum jadi 2 entry di Decisions Log.
- Macet di: sempat lupa bersihkan kode uji-coba (`ArticleType articleType`
  field) di BootStrap sebelum commit — nyaris kebawa jadi dead code permanen.
- Kepelajari: buat kontrak baru (enum/DTO di shared plugin) itu gak selesai
  cuma di compile sukses — harus dibuktikan ada modul lain yang beneran
  import & pakai, baru bisa dibilang "kontraknya jalan".
- Lanjut ke: Fase 2 — `content_service` jadi write side. Ganti H2 → MySQL,
  desain domain `Article`, `Game`, `Category`, `Tag`, `Author`, `Review`
  berdasarkan keputusan yang sudah dicatat di Concept Worksheet & Decisions
  Log.

---

## 2026-08-17
- Ngerjain: setup dokumentasi — CLAUDE.md (arsitektur & rujukan CCWR) dan
  vault ini (task tracking, concept worksheet, decisions log).
- Macet di: gak tahu harus mulai dari mana setelah lama gak sentuh proyek —
  bingung bedain "belum ada konsep produk" vs "fondasi teknis belum beres".
- Kepelajari: dua hal itu independen — Fase 0 (Gradle/git) gak butuh konsep
  produk sama sekali, bisa dikerjakan duluan.
- Lanjut ke: [[Fase-0-Checklist]] task 1 (rename `setting.gradle`).

---

<!-- Entry baru ditambahkan di atas garis ini -->
