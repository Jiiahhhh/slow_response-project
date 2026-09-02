# Peta Kompetensi

Ini alat ukur, bukan rapor. Gunanya supaya kalimat "kamu belum cukup paham" punya dasar
yang bisa diperiksa — bukan perasaan Claude hari itu.

Cara kerjanya: tiap slice di [[00-Slice-Backlog]] punya baris **Prasyarat**. Sebelum
slice dimulai, statusnya dicek di sini. Ada satu saja yang belum `LULUS`, slice ditahan
dan kita pindah ke `/mentoring` dulu.

Isi tiap kompetensi (apa isinya, uji pahamnya apa) ada di
`.claude/skills/mentoring/references/kurikulum-fondasi.md`.

**Status:** `BELUM` (belum dibahas) · `SEDANG` (sudah dibahas, penilaian belum lulus) ·
`LULUS` (dengan tanggal dan bukti)

---

## Tier 1 — dipakai di Fase 2

| Kode | Kompetensi | Status | Tanggal | Bukti / catatan |
|---|---|---|---|---|
| F-01 | Membaca error dan mendiagnosis | BELUM | – | – |
| F-02 | Version control sebagai alat berpikir | BELUM | – | – |
| F-03 | Pemodelan data | BELUM | – | – |
| F-04 | Layering dan separation of concerns | BELUM | – | – |
| F-06 | Transaksi dan konsistensi | BELUM | – | – |
| F-07 | Menulis test yang berarti | BELUM | – | – |

## Tier 2 — dipakai di Fase 3 dan 4

| Kode | Kompetensi | Status | Tanggal | Bukti / catatan |
|---|---|---|---|---|
| F-05 | Kontrak dan batas antar modul | BELUM | – | – |
| F-08 | Membaca kode orang lain | BELUM | – | – |
| F-10 | Keamanan dasar | BELUM | – | – |
| F-12 | Menulis untuk developer lain | BELUM | – | – |
| F-13 | Trade-off dan keputusan arsitektur | BELUM | – | – |

## Tier 3 — dipakai di Fase 5 sampai 7

| Kode | Kompetensi | Status | Tanggal | Bukti / catatan |
|---|---|---|---|---|
| F-09 | Mengukur sebelum mengoptimasi | BELUM | – | – |
| F-11 | Caching dan invalidasi | BELUM | – | – |
| F-14 | Observability | BELUM | – | – |

---

## Catatan penilaian

Diisi tiap selesai sesi mentoring — sudut penjelasan mana yang dipakai, di soal mana
jatuh kalau belum lulus. Gunanya supaya percobaan berikutnya tidak mengulang cara yang
sudah terbukti tidak masuk.

_(belum ada)_

---

## Cara membaca peta ini dengan jujur

Semua `BELUM` di atas **bukan berarti kamu tidak tahu apa-apa.** Artinya cuma: belum
diuji. Sangat mungkin beberapa di antaranya lulus di percobaan pertama dalam 30 menit.

Yang penting justru sebaliknya — jangan tergoda mencentang sendiri sesuatu yang terasa
"kayaknya udah paham". Rasa paham dan paham adalah dua hal berbeda, dan seluruh
gunanya file ini adalah memisahkan keduanya. Kalau memang sudah paham, ujinya cepat.

Jawaban latihan disimpan di `docs/vault/latihan/`.
