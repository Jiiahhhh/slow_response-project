# Kurikulum Teknologi

Peta konsep per fase: apa yang perlu dipahami sebelum menulis kode, dan jebakan
yang paling sering menjegal. Dipakai di langkah 2 protokol sesi (briefing).

Ini indeks, bukan naskah. Penjelasan lengkapnya kamu susun langsung di sesi pakai
format 9 langkah di `SKILL.md` — supaya bisa disesuaikan dengan apa yang sudah dia
pahami hari itu.

**Soal rujukan CCWR:** hanya empat path yang sudah terverifikasi di `CLAUDE.md`
(`web_portal/src/main/groovy/web_portal/http/client/`,
`shared_lib/src/main/groovy/shared_lib/opensearch/OpenSearchIndex.groovy`,
`services/corpweb/grails-app/domain/corpweb/Page.groovy`, dan
`web_portal/grails-app/conf/application.yml` sebagai contoh anti-pola). Selain itu,
**cari dulu dengan grep di `/Users/ilal/canon-corporate-website-revamp` sebelum
menyebut path** — menunjuk file yang ternyata tidak ada langsung merusak kepercayaan
pada seluruh sesi.

## Daftar isi

- [Fase 2 — content_service](#fase-2--content_service-write-side)
- [Fase 3 — admin_portal](#fase-3--admin_portal)
- [Fase 4 — web_portal](#fase-4--web_portal)
- [Fase 5 — Consul](#fase-5--consul)
- [Fase 6 — OpenSearch](#fase-6--opensearch)
- [Fase 7 — Hazelcast & Kafka](#fase-7--hazelcast--kafka)
- [Konsep lintas fase](#konsep-lintas-fase)

---

## Fase 2 — content_service (write side)

**GORM & Hibernate**
Inti: domain class bukan sekadar struct — tiap objek dikelola Hibernate session, dan
banyak hal terjadi tanpa kamu memanggilnya (dirty checking, flush otomatis di akhir
transaksi).
Jebakan: `save()` tanpa `failOnError: true` gagal diam-diam kalau validasi tidak
lolos. Ini penyebab paling umum "kok datanya nggak masuk tapi nggak ada error".

**Relasi: `hasMany`/`belongsTo` vs join table eksplisit**
Inti: many-to-many bawaan Grails cepat ditulis tapi tidak bisa menyimpan atribut
tambahan di relasinya. Join table eksplisit (seperti `ArticleGame`) lebih bertele-tele
tapi bisa menampung, misalnya, peran game di artikel itu (utama/sekunder).
Jebakan: `belongsTo` menentukan arah cascade delete. Salah arah → hapus satu Tag
ikut menghapus Article.
Rujukan: `corpweb.Page` di CCWR punya tiga self-reference (`parent`, `master`,
`origin`) — contoh bagaimana relasi memodelkan konsep bisnis, bukan cuma foreign key.

**Cascade & orphan removal**
Inti: apa yang terjadi ke anak saat induknya dihapus atau disimpan.
Jebakan: cascade `all` pada relasi yang dipakai bersama (Tag, Game) hampir selalu
salah — itu data milik bersama, bukan milik satu artikel.

**Service layer & `@Transactional`**
Inti: service adalah tempat logika bisnis dan batas transaksi. Grails membungkus
service dalam proxy; itu sebabnya memanggil method transaksional dari method lain di
kelas yang sama tidak membuka transaksi baru.
Jebakan: `readOnly = true` untuk query — bukan optimasi kosmetik, ia mematikan dirty
checking dan mencegah update tak sengaja.

**Constraint & validasi**
Inti: validasi tinggal di domain supaya berlaku untuk semua pemanggil, termasuk yang
belum ditulis.
Jebakan: `unique: true` pada slug tidak cukup — masih perlu tahu apa yang terjadi
saat bentrok, dan pesan errornya harus bisa dibaca klien API.

**REST controller & respond()**
Inti: controller menerjemahkan HTTP ke pemanggilan service dan sebaliknya. Tidak ada
logika bisnis di sini.
Jebakan: mengembalikan domain class langsung. Selain membocorkan field internal, ia
memicu `LazyInitializationException` saat serializer menyentuh relasi di luar
transaksi. Ini alasan sebenarnya DTO ada.

**DTO & lapisan kontrak**
Inti: `ArticleResponse` di `shared_lib` adalah kontrak antar modul. Bentuk API jadi
bisa stabil walau domain berubah.
Jebakan: DTO yang cuma menyalin semua field domain satu-satu tidak memberi
perlindungan apa pun — pilih field secara sadar.

**N+1 query**
Inti: satu query untuk daftar, lalu satu query lagi per baris saat relasinya
disentuh.
Cara lihat: nyalakan `logSql: true` sementara, lalu hitung query untuk satu request.
Jebakan: baru terasa setelah data banyak — makanya seed data harus 25-30 record,
bukan 3.

---

## Fase 3 — admin_portal

**Scaffolding**
Inti: `generate-all` bikin CRUD lengkap dalam hitungan detik — sangat berguna untuk
melihat hasil cepat.
Jebakan: kode hasil generate itu utang. Putuskan sejak awal mana yang akan
dipertahankan dan mana yang dibuang.

**Feign client**
Inti: mendeklarasikan pemanggilan HTTP sebagai interface, bukan menulis HTTP client
manual. Implementasinya dibuat runtime.
Jebakan: DTO-nya harus dipakai bersama (dari `shared_lib`) — kalau tiap sisi punya
salinan sendiri, kontraknya bisa melenceng tanpa ketahuan sampai runtime.
Rujukan: `web_portal/src/main/groovy/web_portal/http/client/`

**Transisi status DRAFT → PUBLISHED**
Inti: publish bukan sekadar `status = PUBLISHED` — biasanya ada efek samping
(set `publishedDate`, nanti trigger indexing).
Jebakan: transisi yang tidak divalidasi. Bolehkah ARCHIVED kembali ke PUBLISHED?
Putuskan sadar-sadar, catat di Decisions Log.

---

## Fase 4 — web_portal

**Read/write split**
Inti: ini pola paling penting di seluruh CCWR. `web_portal` tidak punya domain class
dan tidak punya datasource sama sekali. Halaman publik tidak pernah JOIN ke MySQL
saat request.
Kenapa: sisi baca dan sisi tulis punya pola beban yang berbeda jauh, dan
memisahkannya membuat masing-masing bisa dioptimasi sendiri.
Jebakan: godaan menambahkan `hibernate5` "cuma untuk satu query". Begitu satu masuk,
polanya bocor dan tidak pernah kembali.

**Mencabut datasource dari modul Grails**
Inti: mencabut `hibernate5` dan H2 membuat sejumlah auto-configuration ikut mati.
Jebakan: pesan errornya menyesatkan dan tidak menyebut datasource sama sekali. Siapkan
mental untuk itu.

**GSP & layout**
Inti: server-side rendering, taglib, layout dekorator.
Jebakan: memanggil service atau HTTP client dari dalam GSP. Ambil datanya di
controller.

---

## Fase 5 — Consul

**Service discovery**
Inti: service mendaftarkan diri ke registry; pemanggil mencari berdasarkan *nama*,
bukan host:port. URL hardcoded hilang.
Jebakan: health check yang salah bikin service dianggap mati padahal hidup — dan
gejalanya terlihat seperti bug jaringan.

**Client-side load balancing**
Inti: pemanggil yang memilih instance mana yang dipakai, bukan load balancer terpusat.
Jebakan: butuh strategi retry yang sadar idempotensi — retry otomatis pada request
tulis bisa menggandakan data.

---

## Fase 6 — OpenSearch

**Search index vs database**
Inti: MySQL sumber kebenaran, OpenSearch salinan yang dioptimasi untuk baca dan cari.
Jebakan: memperlakukan index sebagai sumber kebenaran. Index harus selalu bisa
dibangun ulang dari MySQL — kalau tidak bisa, itu bukan index, itu database kedua
tanpa backup.

**Denormalisasi**
Inti: dokumen index memuat data yang sudah digabung (nama author, judul game) supaya
tidak perlu JOIN saat baca.
Jebakan: saat nama author berubah, semua dokumen yang memuatnya harus diperbarui.
Putuskan di awal apa yang boleh didenormalisasi.

**Eventual consistency**
Inti: ada jeda antara publish dan artikel muncul di listing.
Jebakan: editor yang publish lalu tidak melihat hasilnya langsung akan mengira sistem
rusak. Perlu jawaban desain, bukan cuma penjelasan lisan.

**Strategi penamaan index**
Rujukan: `shared_lib/src/main/groovy/shared_lib/opensearch/OpenSearchIndex.groovy` —
CCWR memecah index per site+bahasa (`articles__{siteId}__{lang}`) dan memisahkan
`preview_articles` untuk draft.

---

## Fase 7 — Hazelcast & Kafka

**Near-cache**
Inti: cache di dalam proses aplikasi, jadi hit tidak menyentuh jaringan sama sekali.
Jebakan: invalidasi. TTL per jenis konten adalah kompromi yang dipilih CCWR — pahami
apa yang dikorbankan (data basi selama TTL).

**Kafka dan cakupannya**
Inti: message broker untuk komunikasi asinkron antar service.
Jebakan terbesar: memakai Kafka untuk segalanya. CCWR sengaja memakainya sempit —
sinkronisasi user dan komentar antar service, **bukan** untuk indexing konten. Tiru
kesempitan itu; ia hasil pengalaman, bukan keterbatasan.

---

## Konsep lintas fase

**Kenapa `shared_lib` plugin, bukan library biasa**
Grails plugin bisa membawa artefak Grails (domain, service, taglib, konfigurasi),
bukan cuma kelas biasa. Itu yang bikin ia bisa jadi lapisan kontrak sungguhan.

**Manajemen secret**
Environment variable sejak awal. `CLAUDE.md` mencatat anti-pola CCWR yang eksplisit:
`web_portal/grails-app/conf/application.yml` di sana memuat AWS key, Adyen key, dan
reCAPTCHA secret dalam plaintext dan ter-commit. Ini contoh berharga justru karena
salah — proyek nyata pun membuat kesalahan ini.

**Audit trail**
CCWR memakai plugin `audit-logging` dengan `extends AuditFields implements Auditable`
di hampir semua domain. Layak ditiru kalau waktunya cukup — dan pertanyaan desain
yang menarik: kenapa "siapa mengubah apa dan kapan" itu kebutuhan bisnis, bukan
kebutuhan teknis.
