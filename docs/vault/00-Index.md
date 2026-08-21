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
  di-drop dari rencana. Lihat [[04-Decisions-Log]].
- Sedang di **Fase 0** — belum ada kode aplikasi ditulis. Task konkretnya ada di
  [[01-Fase-0-Checklist]].
- Konsep produk (nama publik, pilar konten, skema skor) **belum diputuskan**.
  Worksheet-nya ada di [[03-Concept-Worksheet]] — kerjakan santai, gak nge-block
  Fase 0.

## Peta file

| File | Isi |
|---|---|
| [[01-Fase-0-Checklist]] | Task teknis paling mendesak — beresin fondasi Gradle/git |
| [[02-Roadmap-Backlog]] | Task Fase 1–7, dipecah per checkbox |
| [[03-Concept-Worksheet]] | Pertanyaan untuk merumuskan konsep situs (brand, pilar konten, skor) |
| [[04-Decisions-Log]] | Catatan keputusan — kenapa milih X bukan Y |
| [[05-Glossary-CCWR-Patterns]] | Cheat sheet istilah & pola dari CCWR |
| [[06-Journal]] | Log sesi kerja — apa yang macet, apa yang kepelajari |

## Urutan yang disarankan

1. Kerjakan [[01-Fase-0-Checklist]] sampai selesai — jangan lompat ke Fase 1
   sebelum ini beres, karena Fase 1+ butuh `settings.gradle` yang benar untuk
   build multi-project.
2. Sambil jalan (boleh paralel, gak harus selesai dulu), isi
   [[03-Concept-Worksheet]] pelan-pelan. Kamu butuh ini sebelum mulai desain
   domain model di Fase 2.
3. Tiap kali ambil keputusan yang gak trivial (nama field, struktur folder,
   dsb), catat 2-3 baris di [[04-Decisions-Log]]. Kebiasaan ini yang paling
   membedakan kerjaan level junior vs senior — bukan kodenya, tapi jejak
   alasannya.
4. Tulis [[06-Journal]] tiap habis sesi ngoding, walau cuma 3 baris.
