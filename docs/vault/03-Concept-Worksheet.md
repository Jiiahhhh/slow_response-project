# Concept Worksheet

Ini PR/homework kamu, bukan sesuatu yang bisa diisikan AI — konsepnya harus
keluar dari kepalamu, karena ini yang bakal kamu presentasikan sebagai capstone.
Isinya boleh berantakan/draft dulu, revisi belakangan.

Gak nge-block Fase 0. Tapi **harus** sudah ada jawaban minimal sebelum mulai
Fase 2 (desain domain `Article`/`Game`), karena field-field domain class
langsung diturunkan dari jawaban di sini.

---

## 1. Riset pembanding

Kamu sebut 3 situs: kokanggaming.com, thelazy.media, jagatplay.com. Buka
ketiganya, dan isi tabel ini sambil browsing (bukan dari ingatan):

| Situs | Jenis konten apa saja yang ada? (news/review/guide/opini/...) | Ada skor review? Skalanya berapa? | Struktur navigasi utama (menu apa saja) | Yang paling kamu suka dari situs ini |
|---|---|---|---|---|
| kokanggaming.com | | | | |
| thelazy.media | | | | |
| jagatplay.com | | | | |

Pertanyaan lanjutan setelah tabel di atas terisi:
- Dari ketiganya, pola mana yang paling cocok buat proyekmu — dan kenapa?
- Ada sesuatu yang mereka SEMUA punya (berarti kemungkinan itu memang wajib ada
  di situs game manapun)?
- Ada sesuatu yang cuma satu situs punya (berarti itu pembeda, bukan wajib)?

---

## 2. Identitas

- **Nama publik situs** (bukan "slow-response" — itu nama teknis repo, boleh
  tetap dipakai sebagai nama folder/package, tapi brand yang tampil ke
  pembaca beda hal):
- **Tagline satu kalimat:**
- **Target pembaca** (casual gamer? hardcore/esports? platform spesifik?):
- **Cakupan platform** (PC / console / mobile / semua):

---

## 3. Pilar konten

Centang yang kamu mau ada (boleh lebih dari satu, tapi tiap pilar = 1 nilai
enum `ArticleType` di Fase 1 — makin banyak pilar makin banyak yang harus kamu
bangun, jadi jangan pilih semua kalau waktu terbatas):

- [ ] News — berita singkat, rilis, update
- [ ] Review — penilaian game dengan skor
- [ ] Guide — walkthrough, tips
- [ ] Feature/Opinion — artikel panjang, analisis
- [ ] (lainnya, sebutkan): ___________

---

## 4. Skema skor review

Kalau pilar Review dicentang di atas, wajib diputuskan sebelum Fase 2:

- **Skala:** 1–10? 1–100? Kategori huruf (A/B/C)?
- **Skor tunggal atau breakdown per aspek** (misal Gameplay/Graphics/Story/Sound,
  lalu dirata-rata)?
- **Siapa yang kasih skor:** editor situs saja, atau ada rating pembaca juga?

Catat hasil keputusan ini ke [[04-Decisions-Log]] begitu diputuskan — ini
langsung menentukan bentuk domain `Review` di Fase 2.

---

## 5. Relasi Article ↔ Game

Ini elemen yang gak ada padanannya di CCWR (CCWR gak jualan produk konsumen
seperti game) — jadi ini bagian yang murni desainmu sendiri:

- Satu artikel review, itu tentang **satu** game atau bisa lebih dari satu
  (misal artikel "10 game terbaik 2026")?
- Kalau bisa lebih dari satu game per artikel → relasinya many-to-many
- Apakah tiap `Game` juga butuh halaman sendiri (index semua artikel yang
  menyinggung game itu), atau `Game` cuma metadata pelengkap artikel review?

---

## 6. Taksonomi

- **Genre** — daftar awal (RPG, FPS, Strategy, dst) — ini `Category` atau `Tag`?
- **Platform** — PC/PS5/Xbox/Switch/Mobile — field di `Game` atau tag terpisah?
- Butuh berapa level kategori (flat, atau ada sub-kategori)?

---

Setelah semua terisi, ringkas jadi 1 keputusan resmi di
[[04-Decisions-Log]]: "Domain model Article & Game — versi 1", supaya Fase 2
tinggal eksekusi tanpa mikir ulang.
