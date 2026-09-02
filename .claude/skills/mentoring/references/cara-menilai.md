# Cara Menyusun dan Menilai Task

Tujuan penilaian ini bukan memberi nilai. Tujuannya menjawab satu pertanyaan:
**apakah dia akan menulis kode berikutnya dengan paham, atau dengan meniru?**

Batasnya bukan penguasaan penuh. Batasnya: cukup paham untuk mengenali saat dirinya
sedang salah. Menunggu penguasaan penuh sebelum melanjutkan akan menghentikan proyek
selamanya — dan sebagian besar pemahaman memang baru matang setelah dipakai.

---

## Bentuk task: tiga soal

Selalu tiga, selalu urutan ini. Ketiganya menguji lapisan yang berbeda, dan yang
ketiga yang paling jujur.

### Soal 1 — Jelaskan

*"Jelaskan <konsep> ke seorang junior dalam maksimal 5 kalimat, tanpa memakai istilah
<istilah kunci> itu sendiri."*

Larangan memakai istilah kuncinya itu yang bikin soal ini bekerja. "Transaksi memastikan
atomicity" bisa ditulis siapa pun yang membaca satu paragraf. Menjelaskannya tanpa kata
"transaksi" dan "atomicity" menuntut dia benar-benar memegang idenya.

### Soal 2 — Terapkan

*"Ini situasinya: <kasus baru, domain di luar proyek ini>. Apa yang kamu lakukan,
dan kenapa?"*

Kasusnya wajib baru — bukan variasi dari contoh yang tadi dibahas. Pakai domain lain:
Perpustakaan, Invoice, Pendaftaran Kursus, Reservasi Hotel. Kalau kasusnya terlalu mirip
contoh tadi, yang diuji cuma ingatan jangka pendek.

Minta alasannya, bukan cuma jawabannya. Jawaban benar tanpa alasan tidak lulus.

### Soal 3 — Kritik

*"Ini potongan kode/skema. Ada yang salah. Temukan, jelaskan kenapa itu masalah, dan
apa yang akan terjadi di produksi kalau dibiarkan."*

Ini soal yang paling sulit dipalsukan. Hafalan bisa menjelaskan dan kadang bisa
menerapkan, tapi hafalan tidak bisa mengkritik — mengkritik menuntut model mental yang
utuh tentang bagaimana sesuatu seharusnya bekerja.

Menyusunnya: ambil kode yang **kelihatan wajar**, lalu tanam satu cacat utama dan satu
cacat kecil. Cacat yang mencolok tidak menguji apa pun. Sekali lagi: domain di luar
proyek ini.

---

## Menulis file task

Simpan di `docs/vault/latihan/<kode>-<slug>.md`, misal `F-04-layering.md`:

```markdown
# F-04 · Layering dan Separation of Concerns

Dikerjakan sendiri. Kalau ada yang tidak jelas dari soalnya, tanya — itu bukan
curang. Yang dinilai alasanmu, bukan kecocokan dengan kunci jawaban.

---

## Soal 1 — Jelaskan
<pertanyaan>

**Jawaban:**


## Soal 2 — Terapkan
<pertanyaan>

**Jawaban:**


## Soal 3 — Kritik
<kode/skema bercacat>

**Jawaban:**

---

## Penilaian
_(diisi Claude setelah dijawab)_
```

Setelah menulis file, sebutkan lokasinya lalu **berhenti**. Sesi mentoring menggantung
sampai dia kembali.

---

## Rubrik penilaian

Nilai tiap soal dengan tiga tingkat:

| | Artinya |
|---|---|
| **Kokoh** | Alasannya benar dan lengkap. Dia bisa membelanya kalau ditanya lebih jauh. |
| **Goyah** | Arahnya benar tapi alasannya tidak lengkap, atau benar karena kebetulan. |
| **Belum** | Salah arah, atau benar tapi jelas hasil menyalin tanpa dipahami. |

**Lulus** kalau: minimal dua soal **Kokoh**, dan soal 3 (kritik) tidak **Belum**.

Soal 3 diberi bobot khusus karena dialah yang paling sulit dipalsukan. Orang yang lulus
soal 1 dan 2 tapi gagal total di soal 3 biasanya sedang mengulang penjelasan tadi, bukan
memahaminya — dan itu persis kondisi yang gate ini dibuat untuk menangkap.

## Menilai dengan jujur

- **Nilai alasannya, bukan kecocokannya.** Jawaban yang berbeda dari perkiraanmu tapi
  alasannya kokoh itu lulus. Kadang dia benar dan perkiraanmu yang sempit.
- **Jangan melunakkan.** Menaikkan "Goyah" jadi "Kokoh" karena tidak enak akan
  membuatnya membangun di atas fondasi yang dia kira kuat. Biayanya ditagih nanti, dalam
  bentuk yang jauh lebih menyakitkan daripada satu penilaian yang jujur hari ini.
- **Kalau jawaban terasa hafalan** — benar, rapi, tapi tanpa jejak berpikir, atau
  memakai istilah yang tidak pernah dibahas — jangan menuduh. Tanya satu pertanyaan
  lanjutan yang cuma bisa dijawab kalau dia benar-benar paham, misalnya *"kapan yang
  ini justru salah dipakai?"*. Jawabannya akan menjelaskan sendiri.
- **Sebut yang sudah kokoh lebih dulu, secara spesifik.** Bukan basa-basi — dia perlu
  tahu bagian mana dari cara berpikirnya yang layak diulang.

## Kalau belum lulus

Ini keadaan yang normal dan bukan kemunduran. Yang tidak boleh: mengulang penjelasan
yang sama lebih keras.

Yang dilakukan:

1. Cari **di soal mana** dia jatuh — itu menunjukkan lapisan mana yang belum terpasang.
   Gagal di soal 1 artinya konsepnya belum terbentuk. Gagal di soal 3 saja artinya
   konsepnya ada tapi belum jadi model mental.
2. Ganti sudut: analogi lain, domain contoh lain, atau turun ke sub-konsep yang lebih
   kecil. Penjelasan yang tidak masuk dua kali biasanya salah bentuk, bukan salah
   orangnya.
3. Buat task baru — jangan pakai task yang sama. Task yang diulang menguji ingatan,
   bukan pemahaman.
4. Catat di peta kompetensi sebagai `SEDANG` dengan satu baris: sudut mana yang sudah
   dicoba. Supaya percobaan ketiga tidak mengulang yang kedua.
