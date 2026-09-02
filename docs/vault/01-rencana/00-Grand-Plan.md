# Grand Plan

Rencana induk proyek ini, disusun seperti proyek yang dikerjakan untuk klien.

**Bedanya dengan [[03-Tahap3-Build]]:** roadmap fase menjawab *"komponen teknis apa
yang dibangun berikutnya"*. File ini menjawab *"apa yang bisa ditunjukkan ke orang
berikutnya"*. Keduanya dipakai bersama — roadmap jadi urutan teknis di dalam tiap
milestone.

**Tiap tahap di bawah punya file tracker sendiri** — checklist, status, dan catatan
kerja yang berubah tiap sesi. File ini (Grand Plan) isinya yang jarang berubah: alasan
urutannya begini, dan kriteria selesainya. Kalau mau tahu *sejauh apa progresnya
sekarang*, buka tracker-nya, bukan file ini.

**Kenapa perlu dua sudut pandang:** tidak ada klien yang memesan "Consul". Yang dipesan
klien adalah "editor saya bisa menerbitkan artikel". Kalau pekerjaan cuma diurut per
teknologi, pertanyaan "sudah sampai mana?" hanya bisa dijawab dengan istilah internal —
dan proyek yang tidak bisa menjawab pertanyaan itu dengan bahasa manusia akan selalu
terasa tidak selesai-selesai.

---

## Empat tahap

| Tahap | Isi | Keluaran | Perkiraan | Tracker |
|---|---|---|---|---|
| **1 · Discovery** | menetapkan apa yang dibuat dan apa yang tidak | Product brief | 2 sesi | [[01-Tahap1-Discovery]] |
| **2 · Design** | cetak biru sebelum satu baris kode ditulis | IA, ERD, kontrak API, keputusan tech | 4 sesi | [[02-Tahap2-Design]] |
| **3 · Build** | lima milestone, tiap satunya bisa didemokan | Sistem yang jalan | 38 sesi | [[03-Tahap3-Build]] |
| **4 · Release** | dari "jalan di laptop" ke "bisa diserahkan" | UAT, deploy, handover | 3 sesi | [[04-Tahap4-Release]] |

Total ±47 sesi. Pada ritme 3 sesi seminggu, itu **sekitar 4 bulan**. Angka ini sengaja
ditulis supaya tidak ada kejutan di tengah jalan — bukan untuk menakuti, tapi supaya
tiap milestone yang selesai terasa seperti kemajuan nyata, bukan seperti terlambat.

---

# Tahap 1 — Discovery

**Tracker & status kerja: [[01-Tahap1-Discovery]]**

Tujuannya satu: **tahu apa yang TIDAK dibuat.** Proyek gagal lebih sering karena scope
melar diam-diam daripada karena teknis sulit.

### D1 — Product brief
Siapa penggunanya (pembaca, editor, admin), apa yang masing-masing lakukan, dan daftar
tegas "tidak termasuk" (komentar? newsletter? multi-bahasa? akun pembaca?).

**Selesai jika:** ada satu halaman yang bisa dibaca orang lain dan mereka paham
produknya apa, tanpa penjelasan lisan.

### D2 — Analisis pembanding
Sebagian besar sudah dikerjakan di [[05-Concept-Worksheet]] (riset tiga situs).
Yang perlu ditambah: ubah temuan itu jadi **daftar fitur wajib vs opsional**, bukan
sekadar catatan pengamatan.

**Selesai jika:** ada tabel fitur dengan kolom "wajib untuk rilis" / "nanti".

---

# Tahap 2 — Design

**Tracker & status kerja: [[02-Tahap2-Design]]**

Cetak biru. Tahap yang paling sering dilewati orang, dan yang paling mahal kalau
dilewati — karena kesalahan di sini baru ketahuan setelah puluhan file menumpuk di
atasnya.

### X1 — Information architecture
Sitemap, daftar jenis halaman, taksonomi (kategori vs tag vs platform vs genre — apa
bedanya dan kapan pakai yang mana), dan wireframe kasar homepage + halaman artikel.
Wireframe boleh coretan tangan; yang penting tahu elemen apa saja yang harus ada,
karena itu yang menentukan field di model data.

**Selesai jika:** kamu bisa menyebut semua jenis halaman dan apa isinya.

### X2 — Content model / ERD
Model data final. Ini merevisi domain yang sudah ada — `Article` sekarang belum punya
featured image, excerpt, dan meta SEO, dan ketiganya tidak opsional untuk situs media.
Lebih murah memperbaikinya sekarang selagi belum ada kode yang bergantung padanya.

**Selesai jika:** ERD lengkap dengan semua field, dan tiap keputusan relasi punya satu
kalimat alasan di [[00-Decisions-Log]].

### X3 — Kontrak API
Daftar endpoint, bentuk request/response, format error, aturan pagination dan
versioning. Ditulis **sebelum** implementasi — itulah gunanya kontrak.

**Selesai jika:** `admin_portal` dan `web_portal` bisa dibangun dari dokumen ini tanpa
membaca kode `content_service`.

### X4 — Keputusan teknologi
Tiap komponen dipilih sadar: apa yang dipakai CCWR, apa alternatifnya, kenapa proyek
ini ikut atau menyimpang. Termasuk yang belum ada jawabannya sekarang — penyimpanan
gambar, autentikasi admin, dan migrasi database.

**Selesai jika:** tiap baris di tabel tech punya kolom "alasan" yang terisi.

---

# Tahap 3 — Build

**Tracker & status kerja: [[03-Tahap3-Build]]** (checklist per fase teknis) dan
[[00-Slice-Backlog]] (pekerjaan mingguan)

Lima milestone, diurut berdasarkan **apa yang bisa ditunjukkan**, bukan berdasarkan
lapisan teknis. Tiap milestone punya skenario demo — kalimat yang benar-benar kamu
ucapkan sambil menunjuk layar.

---

### M1 — Pembaca bisa membaca *(±8 sesi)*

Situs publik hidup dengan konten nyata. Belum ada admin; datanya dari seed.

**Demo:** *"Ini homepage-nya. Ada artikel terbaru lengkap dengan gambar dan ringkasan.
Saya klik satu — ini halaman artikelnya."*

**Acceptance:**
- Homepage menampilkan minimal 10 artikel terbaru dengan featured image dan excerpt
- Halaman artikel menampilkan isi lengkap, penulis, tanggal, dan game yang dibahas
- Artikel berstatus DRAFT tidak pernah muncul di mana pun
- `web_portal` tidak punya datasource — datanya lewat HTTP dari `content_service`

**Menyentuh:** Fase 2 seluruhnya, Fase 4 sebagian (Feign, URL masih hardcoded)

**Kenapa ini duluan:** ini satu-satunya milestone yang membuktikan pola inti CCWR —
pemisahan sisi baca dan sisi tulis. Kalau ini jalan, sisanya penambahan. Kalau ini
tidak jalan, sisanya tidak ada artinya.

---

### M2 — Editor bisa menerbitkan *(±8 sesi)*

**Demo:** *"Saya login sebagai editor, tulis artikel baru, unggah gambarnya, simpan
sebagai draft — belum muncul di situs. Saya klik Publish — sekarang muncul di
homepage."*

**Acceptance:**
- Admin terlindungi login; pengunjung tanpa login tidak bisa masuk sama sekali
- Editor bisa CRUD artikel, mengunggah featured image, dan mengatur meta SEO
- Transisi DRAFT → PUBLISHED mengisi `publishedDate` otomatis
- Perubahan di admin terlihat di web portal tanpa restart

**Menyentuh:** Fase 3, plus dua hal yang belum ada di roadmap: **autentikasi** dan
**penyimpanan gambar**. Keduanya diputuskan di X4.

---

### M3 — Database game punya nyawa *(±6 sesi)*

**Demo:** *"Ini halaman game Elden Ring — ada info rilis, developer, dan semua artikel
kami yang membahasnya. Skor review kami 9,2."*

**Acceptance:**
- Halaman game menampilkan detail, artikel terkait, dan skor review agregat
- Satu artikel bisa terhubung ke beberapa game, dengan satu game ditandai utama
- Bisa telusuri dua arah: dari artikel ke game, dari game ke artikel

---

### M4 — Bisa ditemukan *(±8 sesi)*

Milestone yang menentukan situs media ini hidup atau mati. Trafik media datang dari
Google dan share sosmed — bukan dari orang yang mengetik alamatnya.

**Demo:** *"Saya cari 'soulslike' — keluar artikel dan game yang relevan. Saya filter
ke PS5. Dan ini kalau link-nya saya share ke WhatsApp — muncul judul, ringkasan, dan
gambarnya."*

**Acceptance:**
- Pencarian teks berjalan di artikel dan game
- Filter platform dan genre
- Tiap halaman punya meta title, meta description, canonical URL, dan OG tags
- `sitemap.xml` dan `robots.txt` tersedia dan benar
- Listing publik dibaca dari OpenSearch, bukan dari `content_service`

**Menyentuh:** Fase 6

---

### M5 — Siap menahan beban *(±8 sesi)*

**Demo:** *"Ini yang terjadi kalau satu berita tiba-tiba ramai — halaman disajikan dari
cache, database tidak tersentuh. Dan ini cara saya menambah instance tanpa mengubah
konfigurasi apa pun."*

**Acceptance:**
- Near-cache aktif dengan TTL per jenis konten
- Service saling menemukan lewat Consul, tanpa URL hardcoded
- Perubahan skema lewat migrasi, bukan `dbCreate: update`
- Ada healthcheck endpoint dan log yang bisa ditelusuri

**Menyentuh:** Fase 5 dan 7, plus migrasi database

---

# Tahap 4 — Release

**Tracker & status kerja: [[04-Tahap4-Release]]**

### R1 — UAT
Checklist yang dicoba sendiri oleh "klien" (dalam hal ini kamu, dengan sudut pandang
pengguna, bukan pembuat). Tiap acceptance criteria di atas jadi satu baris centang.

### R2 — Deploy
Sistem jalan di luar laptop. Minimal satu lingkungan staging. Runbook: cara
menyalakan, cara mematikan, apa yang dilakukan kalau macet.

### R3 — Handover
README yang bisa diikuti orang asing sampai sistemnya hidup, dokumen arsitektur, dan
catatan "apa yang saya kerjakan setengah dan kenapa" — dokumen terakhir ini yang paling
dihargai orang yang mewarisi proyek.

---

## Peta milestone ke fase

| | Fase 2 | Fase 3 | Fase 4 | Fase 5 | Fase 6 | Fase 7 |
|---|---|---|---|---|---|---|
| M1 | ●●● | | ●● | | | |
| M2 | ● | ●●● | ● | | | |
| M3 | ●● | ● | ●● | | | |
| M4 | | | ●● | | ●●● | |
| M5 | | | | ●●● | | ●●● |

Satu milestone memotong beberapa fase — itu memang maksudnya. Fase adalah urutan
teknis; milestone adalah urutan nilai.

---

## Aturan main

1. **Satu milestone selesai penuh sebelum pindah.** Milestone setengah jadi tidak bisa
   didemokan, dan yang tidak bisa didemokan tidak bisa dihitung selesai.
2. **Tiap milestone ditutup dengan demo sungguhan** — jalankan skenarionya dari awal,
   ucapkan kalimatnya. Kalau ada yang tersendat, milestone-nya belum selesai.
3. **Tahap 1 dan 2 tidak boleh dilewati**, tapi juga tidak boleh melar. Enam sesi.
   Kalau lewat, itu tanda sedang menghindari menulis kode.
4. **Perubahan scope dicatat**, tidak diselundupkan. Ide baru masuk ke "Belum
   dijadwalkan" di [[00-Slice-Backlog]], bukan ke milestone yang sedang berjalan.
