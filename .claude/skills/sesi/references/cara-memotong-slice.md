# Cara Memotong Slice

Baca ini kalau backlog habis, kalau slice ternyata kebesaran di tengah sesi, atau
kalau mau memecah fase berikutnya.

---

## Apa itu slice

Satu slice punya empat sifat:

1. **Muat dalam 60-90 menit** termasuk waktu briefing dan review — jadi porsi
   ngoding-nya realistis cuma 40-50 menit.
2. **Punya kriteria selesai yang bisa dibuktikan satu perintah.** Kalau kriterianya
   tidak bisa ditulis sebagai perintah, slice-nya belum jelas.
3. **Menghasilkan satu commit yang berdiri sendiri.**
4. **Menggerakkan sesuatu yang terlihat.** Idealnya ada yang bisa dilihat, dijalankan,
   atau di-`curl` sesudahnya.

## Aturan paling penting: potong vertikal, bukan horizontal

Ini akar masalah yang bikin proyek ini mandek di Fase 2.

**Potongan horizontal** = kerjakan satu lapisan sampai habis dulu. "Tulis semua
domain class", lalu "tulis semua service", lalu "tulis semua controller". Kelihatan
rapi dan efisien. Tapi setelah 8 domain class ditulis, tidak ada satu pun yang bisa
dijalankan atau dilihat. Tidak ada umpan balik, tidak ada rasa selesai, dan tiap
keputusan desain baru terbukti benar-salahnya berminggu-minggu kemudian.

**Potongan vertikal** = ambil satu hal, tembus semua lapisan. `Article` saja: domain →
service → controller → `curl` berhasil. Baru setelah itu `Game`. Lebih lambat di atas
kertas, tapi tiap slice berakhir dengan sesuatu yang hidup, dan kesalahan desain
ketahuan di hari yang sama.

Aturan praktisnya: **kalau sebuah slice tidak mengubah apa yang bisa dilihat atau
dijalankan, dia terlalu horizontal.** Cari cara memotongnya lebih tipis tapi tembus.

## Cara memotong: mundur dari kriteria fase

1. Tulis kriteria "Selesai jika" fase itu sebagai satu perintah nyata.
   Contoh Fase 2: `curl localhost:9089/api/articles` mengembalikan JSON dari MySQL.
2. Tanya: apa hal **paling kecil** yang membuat perintah itu berhasil, walau
   hasilnya masih jelek? Itu slice pertama sampai selesai — biasanya satu jalur
   tipis yang tembus dari database ke HTTP.
3. Sisa pekerjaannya jadi slice-slice penyempurnaan: filter, DTO, pagination,
   error handling, data lengkap.
4. Urutkan supaya **jalur tipis itu hidup sedini mungkin**. Setelah ada yang hidup,
   semua slice sesudahnya terasa seperti perbaikan, bukan seperti mendaki.

## Tanda slice kebesaran

Hentikan dan potong lagi kalau ketemu salah satu ini:

- Kriteria selesainya pakai kata "dan" lebih dari sekali
- Menyentuh lebih dari 3-4 file baru
- Butuh dua konsep baru sekaligus (misal: transaksi *dan* pagination)
- Kamu tidak bisa membayangkan pesan commit-nya sebelum mulai
- Di menit ke-50 belum ada apa pun yang bisa dijalankan

Memotong di tengah sesi itu normal dan bukan kegagalan. Commit yang sudah jalan,
sisanya jadi slice baru di backlog.

## Tanda slice kekecilan

Digabung saja kalau:

- Selesai di bawah 20 menit termasuk review
- Pesan commit-nya jadi terdengar konyol berdiri sendiri (`feat: tambah satu field`)
- Tidak bisa dibuktikan tanpa slice sesudahnya

## Format menulis slice di backlog

```markdown
### S2-3 — ArticleService dengan filter status

- **Konsep:** service layer, `@Transactional(readOnly)`, kenapa penyaringan
  status tidak boleh di controller
- **Sentuh:** `content_service/grails-app/services/content_service/ArticleService.groovy`
- **Selesai jika:** `./gradlew :content_service:test --tests '*ArticleServiceSpec*'`
  lulus, dan `findPublished()` tidak pernah mengembalikan artikel DRAFT
- **Perkiraan:** 60 menit
```

Empat baris ini yang bikin sesi bisa dimulai dalam 2 menit, bukan 20 menit
mengingat-ingat.

## Kalau muncul ide di tengah sesi

Catat sebagai slice baru di bagian "Belum dijadwalkan" di backlog, lalu lanjutkan
slice yang sedang berjalan. Ide bagus yang dikerjakan di waktu yang salah adalah
cara paling umum sebuah sesi berakhir tanpa commit.
