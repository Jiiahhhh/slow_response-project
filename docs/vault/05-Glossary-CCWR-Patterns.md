# Glossary — Pola CCWR (Cheat Sheet)

Versi ringkas untuk dicek cepat saat lagi ngoding. Detail lengkap + path file
rujukan ada di [CLAUDE.md](../../CLAUDE.md) bagian "Proyek rujukan: CCWR".

Format tanya-jawab — coba tutup jawabannya dan tes diri sendiri sebelum lihat,
lebih nempel di ingatan daripada cuma dibaca.

---

**Q: Kenapa `web_portal` di CCWR gak punya domain class atau datasource sama sekali?**
A: Karena `web_portal` murni read-side. Semua baca lewat OpenSearch (atau Feign
ke service lain), gak pernah JOIN langsung ke MySQL saat serve request publik.

**Q: Apa itu `shared_lib` dan kenapa penting?**
A: Grails plugin (bukan app) berisi enum, DTO request/response, Kafka
template, OpenSearch client trait, dan `HazelcastService`. Ini yang bikin
`web_portal` dan `content_service` bisa "bicara" pakai kontrak yang sama tanpa
saling import domain class satu sama lain.

**Q: Bagaimana `web_portal` manggil `content_service` tanpa hardcode URL?**
A: Pakai Feign client — interface dengan anotasi `@FeignClient(name = "...")`.
Nama logis di-resolve jadi alamat asli lewat Consul (service discovery).

**Q: Apa bedanya status DRAFT / RELEASE / PUBLISHED / ARCHIVED di CCWR?**
A: `corpweb.Page` pakai 4 status ini plus 3 self-reference (`parent` untuk
terjemahan, `master` untuk copy asli, `origin` untuk shadow/draft) buat
menangani draft-preview-publish-versioning sekaligus. Proyek ini gak wajib
seserius itu — versi sederhana (DRAFT/PUBLISHED/ARCHIVED tanpa versioning)
cukup untuk Fase 2.

**Q: Kenapa ada index OpenSearch `preview_articles` terpisah dari `articles`?**
A: Supaya editor bisa preview draft tanpa draft itu ketarik ke hasil pencarian
publik. Dua index, satu untuk live, satu untuk preview.

**Q: Kafka dipakai buat apa di CCWR? Buat index konten ke OpenSearch?**
A: Bukan. Topic-nya cuma `CORPWEB_DISCUSSION`, `SIMS_DISCUSSION`,
`PIM_DISCUSSION` + event user/accessTag — jadi buat sinkronisasi user &
komentar antar-service. Indexing konten ke OpenSearch itu jalur terpisah,
bukan lewat Kafka.

**Q: Kenapa hampir semua domain CCWR punya `dateCreated`/`lastUpdated`/`createdBy`?**
A: Pola `extends AuditFields implements Auditable`, pakai plugin
`audit-logging`. Semua perubahan data tercatat otomatis.

**Q: Apa yang HARUS diganti, bukan ditiru mentah-mentah, dari CCWR?**
A: `web_portal/grails-app/conf/application.yml` CCWR nyimpen AWS key, Adyen
key, reCAPTCHA secret **plaintext di git**. Jangan ditiru — proyek ini pakai
environment variable dari awal.
