# Tahap 3 — Build (Roadmap Teknis, Fase 1 sampai 7)

**Prasyarat: [[01-Tahap1-Discovery]] dan [[02-Tahap2-Design]] harus selesai dulu.**
Fase 2 di bawah sempat mulai berjalan sebelum itu tercapai — domainnya sudah
dihapus lagi 2026-09-02 (lihat [[00-Decisions-Log]]) supaya tidak dibangun di
atas desain yang belum final. `/sesi` sekarang menahan ini otomatis lewat
Gate Tahap — jangan buka slice baru dari sini sebelum Tahap 1-2 tuntas.

**Sudut pandang file ini: komponen teknis apa yang dibangun, per lapisan.**
Kalau kamu mau tahu "apa yang bisa ditunjukkan ke orang berikutnya", itu
[[00-Grand-Plan]] — file itu yang mengurutkan berdasarkan nilai, bukan lapisan.
Keduanya dipakai bersama: satu milestone di Grand Plan biasanya memotong
beberapa fase di sini sekaligus.

Untuk pekerjaan hari ini, jangan buka file ini — buka
[../02-kerja/00-Slice-Backlog.md](../02-kerja/00-Slice-Backlog.md). Checklist di
bawah sengaja ringkas (per fase, bukan per 60-90 menit) supaya tidak dobel
dengan slice backlog yang lebih rinci dan lebih sering di-update.

Fase 0 punya file arsip sendiri: [[Fase-0-Checklist]] (sudah selesai).

Scope terkunci ke **3 modul**: `web_portal`, `admin_portal`, `content_service`
(rename dari konsep `corpweb` di CCWR), plus `shared_lib` sebagai plugin
pendukung. Lihat [[00-Decisions-Log]] untuk alasannya.

Cara pakai: kerjakan satu fase sampai kriteria "Selesai jika" tercapai, baru pindah.
Jangan cicil beberapa fase sekaligus — proyek latihan yang paling sering mandek
adalah yang lompat-lompat fase karena bosan sebelum satu bagian benar-benar solid.

---

## Fase 1 — Modul `shared_lib` · **SELESAI** (2026-08-31)

- [x] `grails create-plugin shared_lib`
- [x] Daftarkan di `settings.gradle` root, dependency di ketiga modul lain
- [x] Enum `ArticleType`, `PublishStatus`, `GamePlatform`
- [x] DTO `ArticleResponse` (kerangka — isi field menyusul di slice S2-4)

**Selesai jika:** ketiga modul lain bisa `import shared_lib.enums.ArticleType` dan
tetap compile. — terbukti, lihat [[02-Journal]] entri 31 Agustus.

---

## Fase 2 — `content_service` sebagai write side · **BELUM DIMULAI**

`grails-app/domain` kosong. Ini fase paling berat secara desain — jangan mulai
sebelum [[02-Tahap2-Design]] (khususnya X2) selesai. Pekerjaannya sudah dipecah
jadi slice 60-90 menit — lihat [[00-Slice-Backlog]] (S2-1 sampai S2-8), tapi
S2-1 perlu ditulis ulang begitu X2 final, karena isinya masih mengacu ke domain
draft yang sudah dihapus. Jangan menandai apa pun selesai di sini; status yang
benar ada di file itu.

**Selesai jika:** `curl localhost:9089/api/articles` mengembalikan JSON dari
MySQL, bukan H2, dengan data lengkap dan tanpa artikel DRAFT bocor ke publik.

---

## Fase 3 — `admin_portal` CRUD

- [ ] Scaffold dulu (`grails generate-all`) untuk `Article` dan `Game` — biar
      cepat kelihatan hasilnya, rapikan belakangan
- [ ] Feign client ke `content_service` (URL boleh hardcoded di fase ini,
      Consul menyusul di Fase 5)
- [ ] Alur publish: draft → published (minimal transisi status, belum perlu
      versioning serumit CCWR)

**Selesai jika:** artikel bisa dibuat dan di-publish lewat UI admin, dan langsung
kelihatan lewat endpoint `content_service` di Fase 2.

---

## Fase 4 — `web_portal` konsumsi API — MILESTONE UTAMA

- [ ] **Cabut** dependency `hibernate5` dan H2 dari `web_portal/build.gradle` —
      modul ini gak boleh punya datasource sendiri, meniru pola CCWR
      (`web_portal` di CCWR: 0 domain class, 0 datasource)
- [ ] Feign client baca dari `content_service`
- [ ] Homepage — daftar artikel terbaru
- [ ] Halaman detail artikel
- [ ] Halaman listing per kategori/tipe (News / Review / Guide)

**Selesai jika:** alur end-to-end jalan — publish artikel di admin_portal, muncul
di web_portal. **Kalau waktu/energi habis di titik ini, proyek tetap layak
ditunjukkan sebagai capstone.** Fase 5–7 di bawah ini nilai tambah, bukan syarat.

---

## Fase 5 — Consul (opsional, nilai tambah)

- [ ] Daftarkan `content_service` ke Consul saat startup
- [ ] Ganti Feign client di `admin_portal`/`web_portal` dari URL hardcoded jadi
      `@FeignClient(name = "content-service")` yang di-resolve Consul

---

## Fase 6 — OpenSearch (opsional, paling mirip CCWR)

- [ ] Index artikel ke OpenSearch saat `content_service` publish
- [ ] `web_portal` baca listing/search dari OpenSearch, bukan lagi langsung
      Feign ke `content_service`
- [ ] (Kalau sempat) index `preview_articles` terpisah untuk preview draft

---

## Fase 7 — Hazelcast, lalu Kafka (opsional)

- [ ] Hazelcast near-cache di `web_portal` untuk hasil baca yang sering diakses
- [ ] Kafka untuk hal yang genuinely butuh event antar-service (misal: komentar
      pembaca) — ikuti cakupan sempit seperti CCWR, jangan dipakai untuk
      indexing konten (itu tugas OpenSearch langsung)

---

## Fase 8 — Block-based content editor (stretch goal, setelah Fase 4 aman)

CCWR punya pola `PageComponent` — isi halaman disusun dari blok-blok
bertipe (Hero, Banner, Text, dst), bukan satu field teks panjang. **CCWR
sendiri punya 110 jenis komponen (`ComponentType`)** — itu skala tim
beneran selama bertahun-tahun, jangan ditiru penuh.

Rencana: adopsi **pola**-nya saja dalam skala kecil (3-5 jenis blok, misal
TEXT/IMAGE/QUOTE), bukan jumlahnya. Baru dikerjakan **setelah** Fase 4
(milestone utama) aman — jangan jadi pengganti field `body` polos di
`Article` sekarang.

**Kenapa ditunda, bukan sekarang:** dampaknya nyebar ke 3 modul sekaligus
(domain komponen di `content_service`, UI penyusun blok di `admin_portal`,
logic render per jenis blok di `web_portal`) — resiko molor dari milestone
utama kalau dikerjakan lebih dulu.

- [ ] (belum dikerjakan — cek kembali setelah Fase 4 selesai)
