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
