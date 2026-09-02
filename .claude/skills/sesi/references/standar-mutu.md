# Standar Mutu

Ini patokan untuk menjawab satu pertanyaan: **"ini sudah selesai belum?"** — supaya
jawabannya tidak bergantung pada perasaan hari itu.

---

## Definition of Done — per slice

Slice baru boleh dicentang kalau kelima ini terpenuhi:

1. **Bisa dibuktikan dengan satu perintah.** Ada `curl`, `./gradlew test`, query SQL,
   atau URL yang menunjukkan hasilnya. Kalau buktinya cuma "compile sukses", itu
   bukan bukti — compile sukses cuma berarti sintaksnya benar.
2. **Tidak ada kode uji-coba tertinggal.** Field percobaan, `println`, import tak
   terpakai, blok yang dikomentari. (Ini pernah nyaris kebawa ke commit di Fase 1 —
   lihat journal 31 Agustus.)
3. **Sudah di-commit** dengan pesan yang menjelaskan *kenapa*, bukan cuma *apa*.
4. **Working tree bersih** setelahnya. `git status` tidak menyisakan file misterius.
5. **Journal terisi** 4 baris.

Kalau salah satu gagal, slice-nya belum selesai. Katakan begitu terang-terangan —
mencentang slice yang belum beres adalah cara paling halus untuk kehilangan
kepercayaan pada backlog sendiri.

## Definition of Done — per fase

Selain seluruh slice-nya tercentang:

- Kriteria "Selesai jika" di `docs/vault/01-rencana/03-Tahap3-Build.md` tercapai dan sudah
  dijalankan sungguhan, bukan diasumsikan
- Ada tag git (lihat `gitflow.md`)
- `CLAUDE.md` bagian "Kondisi saat ini" diperbarui — bagian ini gampang basi dan
  bikin sesi berikutnya salah orientasi
- Bisa didemokan dari nol: `git clone` → `docker compose up` → jalankan → terlihat

---

## Harus diperbaiki sekarang vs boleh ditunda

Waktu review, pisahkan dua kelompok ini dengan tegas. Menuntut kesempurnaan di
tiap slice sama merusaknya dengan tidak menuntut apa-apa.

### Harus sekarang

Hal-hal yang biayanya naik berlipat kalau ditunda — karena kode setelahnya akan
menumpuk di atas keputusan yang salah:

- Relasi domain salah arah atau salah kardinalitas
- Domain class bocor keluar sebagai response API (harus lewat DTO di `shared_lib`)
- Secret atau kredensial plaintext di `application.yml` yang ter-commit
  (ini anti-pola CCWR yang eksplisit dilarang di `CLAUDE.md`)
- Query di dalam loop / N+1 di jalur yang jelas akan dipakai berat
- Field `status` atau `publishedDate` yang tidak dipakai padahal endpoint publik
  seharusnya menyaringnya — artinya draft bisa bocor ke publik
- Transaksi hilang di operasi tulis yang menyentuh lebih dari satu tabel

### Boleh ditunda

Hal yang biaya perbaikannya tetap sama besok atau bulan depan:

- Penamaan variabel, urutan method, formatting
- Optimasi yang belum terbukti perlu
- Coverage test yang belum lengkap (asal jalur utamanya sudah ada)
- Duplikasi kecil yang belum muncul tiga kali

---

## Standar data "rasa klien"

Data adalah bagian yang boleh kamu tulis, dan justru di sinilah proyek latihan
paling sering terlihat seperti proyek latihan. Aturannya:

**Larangan keras**

- `Test Article 1`, `Game A`, `lorem ipsum`, `asdf`, `foo@bar.com`
- Semua record punya tanggal yang sama
- Semua artikel berstatus `PUBLISHED`
- Slug yang tidak konsisten dengan judulnya

**Yang wajib ada supaya sistem benar-benar teruji**

| Dimensi | Minimal |
|---|---|
| Game | 25-30 judul nyata, tersebar di beberapa platform, developer & publisher benar |
| Artikel | 25-30, campuran NEWS / REVIEW / GUIDE |
| Status | ada DRAFT, PUBLISHED, dan ARCHIVED — kalau semua PUBLISHED, filter statusmu tidak pernah teruji |
| Author | 3-5 penulis berbeda, supaya halaman "artikel oleh X" punya arti |
| Rentang tanggal | tersebar 6-12 bulan, supaya sorting & pagination kelihatan bekerja |
| Relasi | ada artikel yang membahas 1 game, ada yang membahas 3-4, ada yang tidak membahas game sama sekali (contoh: opini industri) |
| Skor review | tersebar realistis (rata-rata ~7,5), bukan semua 9 |

**Edge case yang sengaja dimasukkan** — ini yang membedakan seed data serius dari
seed data hiasan. Tiap satu di bawah ini pernah bikin halaman produksi rusak:

- Judul sangat panjang (>120 karakter) — menguji layout dan `maxSize`
- Judul dengan karakter khusus: `Marvel's ...`, `Pokémon ...`, `NieR:Automata`
  — menguji slug generator dan encoding
- Game yang belum rilis (`releaseDate` di masa depan) — menguji filter tanggal
- Game multi-platform (5+ platform) dan game single-platform
- Artikel tanpa `publishedDate` karena masih DRAFT — menguji null-handling di sorting
- Review dengan skor di dua ujung (3,0 dan 9,5)

**Seed harus idempoten.** Dijalankan dua kali tidak boleh menggandakan data — cek
dulu apakah sudah ada sebelum menyisipkan. Dan seed hanya boleh hidup di environment
`development`, tidak pernah di `production`.

---

## Standar kode

Cukup ini, jangan ditambah sampai ada alasan nyata:

- Package root = nama modul (`content_service`, `shared_lib`, dst) — mengikuti CCWR
- Logika bisnis di **service**, bukan di controller. Controller cuma menerjemahkan
  HTTP ke pemanggilan service dan sebaliknya
- Response API selalu DTO dari `shared_lib`, tidak pernah domain class langsung
- Enum yang dipakai lebih dari satu modul tinggal di `shared_lib`
- Secret lewat environment variable, tanpa kecuali
- Tiap domain punya minimal satu Spec yang menguji constraint-nya

---

## Checklist "orang lain bisa jalanin"

Jalankan checklist ini tiap akhir fase. Kalau ada yang gagal, itu utang yang akan
menagih pada saat paling tidak enak — waktu proyek ini mau ditunjukkan ke orang.

- [ ] `git clone` lalu ikuti README, tanpa bertanya apa pun, sistemnya hidup
- [ ] `docker compose up -d` menyalakan semua yang dibutuhkan
- [ ] Ada perintah tunggal untuk mengisi data awal
- [ ] Port dan kredensial dev tertulis di satu tempat
- [ ] Tidak ada secret asli di repo
- [ ] Ada satu URL atau `curl` yang langsung memperlihatkan fitur utamanya bekerja
