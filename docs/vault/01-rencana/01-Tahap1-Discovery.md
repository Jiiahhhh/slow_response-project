# Tahap 1 — Discovery

**Status keseluruhan:** SELESAI (2026-09-04) — D1 dan D2 tuntas, lanjut ke [[02-Tahap2-Design]]

Ini tracker kerja untuk Tahap 1. Alasan tahap ini ada dan kenapa dia mendahului
semuanya ada di [[00-Grand-Plan]] — baca itu dulu kalau belum. File ini cuma
menjawab satu hal: *sejauh mana Tahap 1 ini benar-benar selesai, dan apa isinya.*

Tujuannya satu kalimat: **tahu apa yang TIDAK dibuat.** Proyek gagal lebih sering
karena scope melar diam-diam daripada karena teknis sulit — dan scope cuma bisa
dijaga kalau batasnya tertulis, bukan diingat-ingat.

Dipakai lewat `/planning`.

---

## D1 — Product Brief

**Status:** SELESAI (2026-09-03)

**Selesai jika:** ada satu halaman yang bisa dibaca orang lain dan mereka paham
produknya apa, tanpa penjelasan lisan.

### Checklist

- [x] Pengguna terdefinisi: pembaca, editor, admin — apa yang masing-masing lakukan
- [x] Daftar tegas **yang TIDAK dibuat** (komentar pembaca? newsletter? akun
      pembaca? multi-bahasa? sistem iklan?)
- [x] Satu halaman ringkas, bisa dibaca tanpa penjelasan tambahan

### Isi brief

_(diisi saat sesi `/planning` D1 berjalan)_

**Pengguna & peran:** — keputusan lengkap & alasannya di [[00-Decisions-Log]]
(2026-09-03)

| Peran | Login | Wewenang |
|---|---|---|
| Pembaca | Tidak | Baca artikel, filter platform/type/game, lihat halaman game hub. Publik lewat `web_portal`, bukan role `admin_portal`. |
| Editor | Ya | Create/edit/publish artikel (satu wewenang gabungan, belum dipecah create-only vs publish-only). Publish langsung, tanpa gate persetujuan. |
| Admin | Ya | Semua yang Editor bisa + kelola master data `Game`/`Category`/`Tag`/`Author`. Placeholder — nanti dipecah per master data. |
| Super Admin | Ya | Semua yang Admin bisa + kelola akun user (create/nonaktifkan Editor/Admin/Super Admin) + akses semua modul tanpa kecuali. |

Mekanisme implementasi role (tabel `Role`/`Permission` ala CCWR vs enum
sederhana) belum diputuskan — itu keputusan X4, bukan D1.

**Cakupan fitur** — sebagian besar kandidat yang tadinya diduga "TIDAK dibuat"
ternyata masuk scope, cuma beda waktu pengerjaan. Rincian & alasan tiap
keputusan ada di [[00-Decisions-Log]] (2026-09-03).

*Dibangun, Tahap 3 inti (Fase 2-6):*
- Search bar bebas teks (OpenSearch publik, ikut pola CCWR)
- GTM + Consent Mode v2 + cookie banner custom (ikut pola CCWR)
- Iklan — ad network eksternal (AdSense-style), bukan Banner internal CCWR
- Multi-site/multi-bahasa — struktur tabel dipertahankan, diisi 1 baris

*Dibangun, tapi sengaja ditunda sampai setelah milestone inti selesai —
fitur orisinal, CCWR tidak punya rujukan untuk ini:*
- Komentar pembaca (anonim by design, auto-publish, filter sensor kata
  SARA/kasar)
- Newsletter (digest email harian, direncanakan sebagai improvement)

*Benar-benar TIDAK dibuat:*
- Akun/login pembaca — pembaca tetap anonim selamanya, termasuk saat komentar
  (isi nama/email tiap kali, bukan akun persisten)
- Rating/skor dari pembaca — skor cuma dari editor (lihat
  [[05-Concept-Worksheet]] bagian 4)
- Antrian moderasi komentar (status `PENDING`) — auto-publish, moderasi cuma
  lewat hapus setelah tayang
- Edit komentar oleh Editor/Admin — cuma hapus, integritas komentar orang
  lain gak boleh diubah
- Enterprise CMP (Usercentrics) — cukup banner consent custom sederhana
- Sistem Banner internal ala CCWR — dipilih ad network eksternal sebagai
  gantinya, bukan keduanya

*Belum diputuskan, dipertimbangkan nanti (bukan "tidak", bukan "ya"):*
patch notes/changelog artikel, related content component, social share,
popup on-site, Important Notice, audit trail generik, pemecahan role Admin
per master data. Daftar lengkap di [[00-Slice-Backlog]] "Belum dijadwalkan".

---

## D2 — Analisis Pembanding

**Status:** SELESAI (2026-09-04)

**Selesai jika:** ada tabel fitur dengan kolom "wajib untuk rilis" / "nanti".

Sebagian besar sudah dikerjakan di [[05-Concept-Worksheet]] (riset The Lazy Monday,
IGN, dan pembanding lain). Yang kurang: temuan itu belum diubah jadi keputusan.
Riset yang tidak berujung keputusan cuma jadi bacaan.

### Checklist

- [x] Riset 3 situs pembanding (lihat [[05-Concept-Worksheet]])
- [x] Tabel fitur: wajib untuk rilis vs nanti
- [x] Fitur yang **sengaja tidak ditiru** dari pembanding, dan alasannya

### Tabel fitur

Detail & alasan lengkap tiap baris di [[00-Decisions-Log]] (2026-09-04).

| Fitur | Wajib rilis? | Alasan |
|---|---|---|
| Hardware/Tech sebagai pilar konten | Tidak | Di luar fokus — proyek ini tetap murni game |
| Kategori Gacha | Tidak | Fokus PC/konsol modern, mobile/gacha bukan target |
| Rubrik "Time Machine" (kilas balik game lawas) | Nanti | Masuk pilar Feature/Opinion yang sudah ditunda |
| Review multi-halaman long-form | — | Bukan keputusan D2, soal IA — dibahas di X1 |
| Dark mode UI | — | Bukan keputusan D2, soal desain visual — dibahas pas `web_portal` template dibangun |
| Ekosistem komunitas + integrasi YouTube + Awards | Tidak ditiru | Bangun ekosistem eksternal, di luar tujuan belajar CCWR |
| Skor kualitatif tanpa angka (pola ketiga pembanding) | Tidak ditiru | Sudah diputuskan 2026-08-28 — sengaja pakai skor numerik 1-10 |

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
