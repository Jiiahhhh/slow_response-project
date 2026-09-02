# Tahap 1 — Discovery

**Status keseluruhan:** SEDANG BERJALAN

Ini tracker kerja untuk Tahap 1. Alasan tahap ini ada dan kenapa dia mendahului
semuanya ada di [[00-Grand-Plan]] — baca itu dulu kalau belum. File ini cuma
menjawab satu hal: *sejauh mana Tahap 1 ini benar-benar selesai, dan apa isinya.*

Tujuannya satu kalimat: **tahu apa yang TIDAK dibuat.** Proyek gagal lebih sering
karena scope melar diam-diam daripada karena teknis sulit — dan scope cuma bisa
dijaga kalau batasnya tertulis, bukan diingat-ingat.

Dipakai lewat `/planning`.

---

## D1 — Product Brief

**Status:** BELUM

**Selesai jika:** ada satu halaman yang bisa dibaca orang lain dan mereka paham
produknya apa, tanpa penjelasan lisan.

### Checklist

- [ ] Pengguna terdefinisi: pembaca, editor, admin — apa yang masing-masing lakukan
- [ ] Daftar tegas **yang TIDAK dibuat** (komentar pembaca? newsletter? akun
      pembaca? multi-bahasa? sistem iklan?)
- [ ] Satu halaman ringkas, bisa dibaca tanpa penjelasan tambahan

### Isi brief

_(diisi saat sesi `/planning` D1 berjalan)_

**Pengguna & peran:**


**Yang TIDAK dibuat (dan kenapa):**


---

## D2 — Analisis Pembanding

**Status:** SEDANG — riset sudah ada, kesimpulannya belum

**Selesai jika:** ada tabel fitur dengan kolom "wajib untuk rilis" / "nanti".

Sebagian besar sudah dikerjakan di [[05-Concept-Worksheet]] (riset The Lazy Monday,
IGN, dan pembanding lain). Yang kurang: temuan itu belum diubah jadi keputusan.
Riset yang tidak berujung keputusan cuma jadi bacaan.

### Checklist

- [x] Riset 3 situs pembanding (lihat [[05-Concept-Worksheet]])
- [ ] Tabel fitur: wajib untuk rilis vs nanti
- [ ] Fitur yang **sengaja tidak ditiru** dari pembanding, dan alasannya

### Tabel fitur

_(diisi saat sesi `/planning` D2 berjalan)_

| Fitur | Wajib rilis? | Alasan |
|---|---|---|
| | | |

---

## Selesai jika (Tahap 1 keseluruhan)

D1 dan D2 keduanya selesai, dan keputusan-keputusannya sudah masuk ke
[[00-Decisions-Log]]. Begitu tercapai, lanjut ke [[02-Tahap2-Design]].

## Catatan realita — pelajaran dari 2026-09-02

Delapan domain class sempat ditulis di `content_service` **sebelum** D1/D2
ditutup — itu sinyal kenapa proyek terasa lompat ke Build padahal Discovery dan
Design belum selesai. File-nya sudah dihapus (belum pernah di-commit, jadi tanpa
sunk cost) — lihat [[00-Decisions-Log]] entry 2026-09-02. `content_service`
sekarang kosong lagi, sengaja, sampai Tahap 1 dan 2 ini benar-benar tuntas.

Ini bukan kemunduran, ini koreksi urutan. Jangan tulis domain apa pun di
`content_service` sebelum X2 (Content Model/ERD) di [[02-Tahap2-Design]] selesai
— Gate Tahap di `/sesi` sekarang menahan ini secara otomatis.
