# Kurikulum Fondasi

Kompetensi yang membuat seseorang developer, bukan yang membuatnya bisa Grails.
Semuanya berlaku di bahasa dan framework apa pun — itu ujinya: kalau sebuah butir di
sini cuma berlaku di Grails, ia salah tempat dan seharusnya ada di
`.claude/skills/sesi/references/kurikulum-tech.md`.

Status tiap kompetensi dicatat di `docs/vault/02-kerja/01-Peta-Kompetensi.md`.

Urutannya bukan selera. Tiap tier dipilih supaya kompetensinya **langsung terpakai di
fase yang sedang berjalan** — konsep yang dipelajari tanpa langsung dipakai akan hilang
sebelum sempat berguna.

---

## Tier 1 — dipakai di Fase 2

### F-01 · Membaca error dan mendiagnosis
Membaca stack trace dari bawah ke atas, memisahkan baris framework dari baris proyek,
mempersempit masalah dengan mengubah satu hal, dan membedakan menebak dari mendiagnosis.
**Kenapa sekarang:** ini kompetensi dengan bunga majemuk tertinggi. Tiap sesi kerja
memakainya.
**Uji paham:** diberi stack trace asing, dia bisa menyebut di mana harus mulai mencari
dan kenapa — tanpa perlu tahu penyebab pastinya.

### F-02 · Version control sebagai alat berpikir
Commit atomik, pesan yang menjelaskan *kenapa*, riwayat yang bisa dibaca orang lain,
kenapa branch itu soal isolasi bukan soal kerapian.
**Kenapa sekarang:** proyek ini pernah menumpuk 16 file tak ter-commit. Itu bukan
kemalasan git — itu tanda pekerjaan tidak punya batas yang jelas.
**Uji paham:** diberi satu tumpukan perubahan campur aduk, dia bisa membaginya jadi
beberapa commit dan menjelaskan dasar pembagiannya.

### F-03 · Pemodelan data
Entitas vs atribut, kardinalitas, kapan butuh tabel penghubung, normalisasi
secukupnya, dan kapan denormalisasi justru benar.
**Kenapa sekarang:** kesalahan model data adalah kesalahan yang paling mahal
diperbaiki — semua kode setelahnya menumpuk di atasnya.
**Uji paham:** diberi deskripsi bisnis dalam bahasa manusia, dia bisa menggambar
entitas dan relasinya, lalu menjelaskan kenapa bukan bentuk lain.

### F-04 · Layering dan separation of concerns
Kenapa logika bisnis tidak di controller, apa artinya sebuah lapisan "tahu" tentang
lapisan lain, arah ketergantungan, dan biaya yang dibayar saat batas itu dilanggar.
**Kenapa sekarang:** dipakai langsung di slice S2-3.
**Uji paham:** diberi controller gemuk berisi logika, dia bisa menunjukkan apa yang
pindah ke mana dan **apa untungnya** — bukan sekadar "biar rapi".

### F-06 · Transaksi dan konsistensi
Atomicity, di mana batas transaksi seharusnya digambar, apa yang terjadi kalau gagal
di tengah, kenapa transaksi yang terlalu lebar sama berbahayanya dengan yang tidak ada.
**Kenapa sekarang:** dipakai di S2-3 dan seluruh operasi tulis sesudahnya.
**Uji paham:** diberi operasi yang menyentuh tiga tabel, dia bisa menunjuk di mana
transaksi dibuka-ditutup dan apa yang rusak kalau salah tempat.

### F-07 · Menulis test yang berarti
Apa yang layak dites dan apa yang tidak, arrange-act-assert, menguji perilaku bukan
implementasi, kenapa test yang selalu hijau sering tidak berguna.
**Kenapa sekarang:** sudah ada 8 file Spec di repo — pertanyaannya apakah mereka
benar-benar menguji sesuatu.
**Uji paham:** diberi satu test, dia bisa menyebut perubahan kode apa yang akan
membuatnya merah — dan kalau tidak ada, dia paham kenapa itu masalah.

---

## Tier 2 — dipakai di Fase 3 dan 4

### F-05 · Kontrak dan batas antar modul
DTO sebagai kontrak, kenapa objek internal tidak boleh bocor keluar, apa artinya
perubahan yang merusak (breaking change), versioning API.
**Uji paham:** dia bisa menyebut satu perubahan yang aman dilakukan di balik kontrak
dan satu yang tidak, beserta alasannya.

### F-08 · Membaca kode orang lain
Masuk ke codebase asing lewat entry point, memakai grep sebagai alat penelusuran,
membaca dari luar ke dalam, tahu kapan berhenti membaca.
**Kenapa penting:** CCWR punya ratusan domain class. Kemampuan menavigasinya jauh lebih
berharga daripada menghafal isinya — dan di pekerjaan nyata, ini yang dipakai tiap hari.

### F-10 · Keamanan dasar
Injection, otorisasi vs autentikasi, manajemen secret, kenapa validasi di sisi klien
bukan validasi.
**Kaitan langsung:** anti-pola CCWR di `CLAUDE.md` — secret plaintext yang ter-commit.

### F-12 · Menulis untuk developer lain
Pesan commit, catatan keputusan (ADR), README yang benar-benar bisa diikuti orang asing.
**Kenapa penting:** ini yang paling membedakan kerja level junior dan senior — bukan
kodenya, tapi jejak alasannya. Dan di proyek ini, jejak itulah yang nanti ditunjukkan
ke orang.

### F-13 · Trade-off dan keputusan arsitektur
Tidak ada pilihan yang benar, cuma pilihan yang konsekuensinya dipahami. Mengenali
sumbu pertukarannya, menuliskan alasan, dan menerima bahwa keputusan bisa berubah.
**Uji paham:** untuk keputusan yang sudah dia ambil di proyek ini, dia bisa
memaparkan kasus terkuat untuk pilihan yang **tidak** dia ambil.

---

## Tier 3 — dipakai di Fase 5 sampai 7

### F-09 · Mengukur sebelum mengoptimasi
N+1, membaca query log, kenapa optimasi tanpa pengukuran biasanya salah sasaran,
kompleksitas dalam praktik.

### F-11 · Caching dan invalidasi
Kenapa invalidasi itu masalah yang terkenal sulit, TTL sebagai kompromi sadar, data
basi sebagai keputusan produk dan bukan bug.

### F-14 · Observability
Log yang berguna vs log sampah, apa yang perlu dicatat, bagaimana melacak satu request
melewati beberapa service.

---

## Peta ke slice

Prasyarat tiap slice dicatat di `docs/vault/02-kerja/00-Slice-Backlog.md`. Ringkasan untuk
Fase 2:

| Slice | Prasyarat |
|---|---|
| S2-1 | F-02 |
| S2-2 | F-03 |
| S2-3 | F-04, F-06 |
| S2-4 | F-05 |
| S2-5 | — |
| S2-6 | — |
| S2-7 | F-09 |
| S2-8 | F-12 |

F-01 (membaca error) dan F-07 (test) tidak diikat ke satu slice — keduanya dipakai
terus-menerus. Jadwalkan F-01 sedini mungkin, F-07 saat Spec yang sudah ada mulai
dipertanyakan.

## Menambah kompetensi baru

Kalau muncul kebutuhan di luar daftar ini, tambahkan dengan kode berikutnya dan isi
tiga hal yang sama: **apa isinya, kenapa dibutuhkan sekarang, dan uji pahamnya apa.**
Butir tanpa uji paham tidak bisa dijadikan gate — dan gate yang tidak bisa dinilai
bukan gate.
