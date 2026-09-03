# Vault Index

Ini panel kontrol proyek. Kalau kamu buka repo ini setelah lama gak nyentuh, mulai
dari sini.

## Cara pakai

- Folder ini bisa dibuka sebagai Obsidian vault (kalau kamu install Obsidian), atau
  cukup dibaca sebagai file markdown biasa di editor manapun.
- **[CLAUDE.md](../../CLAUDE.md)** di root repo = peta arsitektur & referensi CCWR.
  Baca itu kalau lupa *kenapa* sesuatu didesain begitu.
- Folder ini (`docs/vault/`) = **peta kerja**. Isinya task, keputusan, dan konsep
  yang masih kamu susun sendiri.
- Aturan main: **AI cuma boleh nulis dokumen perencanaan di sini, bukan kode.**
  Lihat bagian "Cara kerja di repo ini" di CLAUDE.md.

## Status saat ini

- Scope dikunci ke **3 modul**: `web_portal`, `admin_portal`, `content_service`
  (+ `shared_lib` sebagai plugin pendukung, bukan service). `api`, `pim`, `sims`
  di-drop dari rencana. Lihat [[00-Decisions-Log]].
- Fase 0 dan Fase 1 **selesai**, diarsipkan. Konsep produk sudah diputuskan
  ([[05-Concept-Worksheet]] terisi penuh).
- **Urutan tahap sempat kelewat, sudah dikoreksi**: 8 domain class di
  `content_service` sempat ditulis (Tahap 3 / Build) sebelum Tahap 1 (Discovery)
  dan Tahap 2 (Design) selesai. Filenya sudah dihapus 2026-09-02 (belum pernah
  di-commit, tanpa sunk cost) — `content_service` kembali kosong sampai
  [[01-Tahap1-Discovery]] dan [[02-Tahap2-Design]] tuntas. `/sesi` sekarang
  menahan ini otomatis lewat Gate Tahap. Detail: [[00-Decisions-Log]].

## Empat folder, empat sudut pandang

Bukan sekadar rapi-rapi — tiap folder menjawab pertanyaan yang berbeda. Kalau
bingung dokumen mana yang harus dibuka, mulai dari pertanyaannya:

| Folder | Pertanyaan yang dijawab | Seberapa sering berubah |
|---|---|---|
| **[01-rencana/](01-rencana/)** | Apa yang dibangun, dan kenapa urutannya begitu? | jarang — cuma saat arah proyek berubah |
| **[02-kerja/](02-kerja/)** | Apa yang dikerjakan minggu ini, dan sejauh apa progresnya? | tiap sesi |
| **[03-referensi/](03-referensi/)** | Kenapa dulu kita pilih X, dan istilah CCWR itu artinya apa? | ditambah, jarang diubah |
| **[04-arsip/](04-arsip/)** | Apa yang pernah dikerjakan dan sudah selesai total? | tidak pernah — ini riwayat |

### 01-rencana/ — arah

| File | Isi | Status |
|---|---|---|
| [[00-Grand-Plan]] | **Rencana induk.** 4 tahap, 5 milestone, diurut berdasarkan apa yang bisa didemokan | — |
| [[01-Tahap1-Discovery]] | Tracker Tahap 1 — product brief, batas scope | SEDANG |
| [[02-Tahap2-Design]] | Tracker Tahap 2 — IA, ERD, kontrak API, 10 keputusan teknologi | SEDANG |
| [[03-Tahap3-Build]] | Tracker Tahap 3 — Fase 1–7 per komponen teknis | SEDANG (lompat urutan) |
| [[04-Tahap4-Release]] | Tracker Tahap 4 — UAT, deploy, handover | BELUM DIMULAI |
| [[05-Concept-Worksheet]] | Konsep produk: brand, pilar konten, skema skor | SELESAI |

**Cara membaca empat tracker tahap:** Grand Plan isinya yang jarang berubah — alasan
urutan dan kriteria selesai. Tiap `Tahap-N` isinya yang berubah tiap sesi — checklist,
status per item, catatan kerja. Kalau mau tahu progres sekarang, buka tracker-nya;
kalau mau tahu kenapa urutannya begini, buka Grand Plan.

### 02-kerja/ — yang sedang berjalan

| File | Isi |
|---|---|
| [[00-Slice-Backlog]] | **Mulai dari sini tiap sesi.** Pekerjaan dipotong per 60-90 menit |
| [[01-Peta-Kompetensi]] | Status kompetensi fondasi — gate sebelum tiap slice boleh dimulai |
| [[02-Journal]] | Log sesi kerja — apa yang macet, apa yang kepelajari |

### 03-referensi/ — kenapa dan apa artinya

| File | Isi |
|---|---|
| [[00-Decisions-Log]] | Catatan keputusan — kenapa milih X bukan Y, apa yang dikorbankan |
| [[01-Glossary-CCWR-Patterns]] | Cheat sheet istilah & pola dari CCWR, format tanya-jawab |
| [[02-Peta-CCWR-Lengkap]] | Peta menyeluruh 7 modul CCWR + inventaris teknologi + **gap analysis rencana** (2026-09-04) |

### 04-arsip/ — sudah selesai

| File | Isi |
|---|---|
| [[Fase-0-Checklist]] | Checklist fondasi Gradle/git — selesai 2026-08-17, disimpan sebagai riwayat |

### latihan/

Jawaban task penilaian dari sesi `/mentoring`. Diisi otomatis saat sesi mentoring
berjalan — tidak perlu dibuka manual kecuali mau melihat riwayat penilaian.

## Cara mulai sesi

Ada empat skill di `.claude/skills/`, masing-masing punya pintunya sendiri:

| Perintah | Kapan |
|---|---|
| `/sesi` | mau ngoding. Membaca journal, backlog, dan `git status`, mengecek gate kompetensi, lalu menyodorkan satu slice 60-90 menit |
| `/planning` | memutuskan arah — pilih teknologi, rancang model data, kontrak API. Dipakai terutama di Tahap 1-2 Grand Plan |
| `/mentoring` | belajar konsep fondasi. Dipanggil sendiri, atau otomatis saat gate `/sesi` tidak lolos |
| `/debug` | ada error atau stack trace |

Mulai dari `/sesi`. Kalau prasyarat kompetensinya belum lulus, dia sendiri yang akan
mengalihkan ke `/mentoring` dan menahan slice-nya. Kalau yang mau dibahas soal arah
proyek bukan soal baris kode, mulai dari `/planning`.

Kalau mau manual:

1. Buka [[00-Slice-Backlog]], ambil slice teratas yang belum tercentang.
2. Kerjakan sampai kriteria "Selesai jika"-nya terbukti — satu perintah, bukan
   perasaan.
3. Tiap kali ambil keputusan yang gak trivial (nama field, struktur folder,
   dsb), catat 2-3 baris di [[00-Decisions-Log]]. Kebiasaan ini yang paling
   membedakan kerjaan level junior vs senior — bukan kodenya, tapi jejak
   alasannya.
4. Tulis [[02-Journal]] tiap habis sesi ngoding, walau cuma 3 baris.
