# Gitflow

Aturan git untuk proyek ini. Tujuannya bukan kerapian demi kerapian — riwayat git
adalah satu-satunya bukti objektif bahwa proyek ini benar-benar jalan, dan nanti
ia jadi bahan cerita saat proyek ini ditunjukkan ke orang.

**Semua perintah di file ini (termasuk yang ada di blok kode) untuk dijalankan
oleh pemilik proyek sendiri, bukan olehmu.** `git commit`, `git push`,
`git merge`, `git tag` — kamu tidak pernah menjalankan ini. Tugasmu berhenti di
menyiapkan pesan commit yang tepat dan daftar file yang relevan; eksekusinya
miliknya, itu bagian dari melatih F-02.

---

## Branch

| Branch | Isi | Aturan |
|---|---|---|
| `main` | hanya kondisi yang layak didemokan | tidak pernah commit langsung; hanya menerima merge dari `develop` |
| `develop` | integrasi harian, basis semua pekerjaan | boleh commit langsung untuk `chore`/`docs` kecil |
| `feature/<nama>` | satu fitur atau satu kelompok slice | dicabang dari `develop`, kembali ke `develop` |
| `fix/<nama>` | perbaikan bug | sama seperti feature |
| `chore/<nama>` | tooling, config, dependency | boleh langsung ke `develop` kalau sepele |

**Satu feature branch menampung beberapa slice yang saling terkait**, bukan satu
branch per slice. Contoh: `feature/article-api` menampung slice service, controller,
DTO, dan error handling — karena keempatnya baru ada artinya kalau digabung.

Penamaan: huruf kecil, pisah dengan tanda hubung, sebut *apa*-nya bukan *bagaimana*-nya.
`feature/article-api` bagus. `feature/tambah-controller-baru` kurang — itu deskripsi
kegiatan, bukan deskripsi hasil.

## Commit

Conventional Commits, dengan scope berupa nama modul:

```
<tipe>(<modul>): <ringkasan imperatif, huruf kecil, tanpa titik>

<kenapa perubahan ini dibuat — 1-3 baris, opsional tapi sangat dianjurkan
untuk keputusan yang tidak jelas dari kodenya sendiri>
```

Tipe: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`.

Contoh yang bagus:

```
feat(content_service): tambah ArticleService dengan filter status

Endpoint publik tidak boleh pernah mengembalikan DRAFT, jadi penyaringan
status ditaruh di service, bukan di controller — supaya semua pemanggil
ikut aman, termasuk yang belum ditulis.
```

```
fix(content_service): pakai join fetch di findPublished untuk hindari N+1

Sebelumnya tiap artikel memicu query terpisah ke author. Dengan 25 artikel
di halaman depan itu 26 query.
```

Contoh yang kurang: `update code`, `fix bug`, `feat: banyak perubahan`.
Enam bulan lagi tidak ada yang bisa dibaca dari situ.

**Satu slice = satu commit.** Kalau di tengah slice kamu terpaksa mengubah hal lain
(rename, config), pisahkan jadi commit `chore` sendiri sebelum atau sesudahnya.
Commit yang mencampur satu fitur dengan sepuluh rename tidak bisa direview dan
tidak bisa di-revert.

## Merge ke develop

Selalu `--no-ff`, supaya batas satu fitur tetap terlihat di riwayat:

```bash
git checkout develop && git merge --no-ff feature/article-api
```

Sebelum merge, cek tiga hal: build lulus, test lulus, dan tidak ada file nyasar
di diff.

## Menutup fase

Tiap Fase selesai (kriteria di `docs/vault/01-rencana/03-Tahap3-Build.md` benar-benar
terbukti), merge `develop` ke `main` dan beri tag:

| Tag | Artinya |
|---|---|
| `v0.1.0` | Fase 1 — `shared_lib` jalan |
| `v0.2.0` | Fase 2 — `content_service` melayani artikel dari MySQL |
| `v0.3.0` | Fase 3 — admin bisa CRUD |
| `v0.4.0` | Fase 4 — end-to-end admin publish → tampil di web portal |
| `v0.5.0` | Fase 5 — Consul |
| `v0.6.0` | Fase 6 — OpenSearch |
| `v0.7.0` | Fase 7 — Hazelcast & Kafka |

```bash
git checkout main && git merge --no-ff develop
git tag -a v0.2.0 -m "Fase 2: content_service melayani artikel dari MySQL"
```

Tag ini bukan formalitas. Ia yang bikin kalimat "proyek ini sudah sampai mana"
punya jawaban tanpa perlu membuka kode.

## Kebiasaan yang menjaga proyek ini hidup

- **Jangan tidur dengan working tree kotor.** Kalau slice belum kelar, commit yang
  sudah jalan dengan pesan `wip(<modul>): ...` dan tulis sisanya di journal. Kerjaan
  menggantung yang menumpuk adalah alasan nomor satu proyek sampingan tidak pernah
  disentuh lagi — bukan karena sulit, tapi karena tidak jelas lagi harus mulai
  dari mana.
- **Push `develop` ke remote tiap akhir sesi.** Sekarang remote baru punya `main`.
  Riwayat yang cuma ada di satu laptop adalah riwayat yang belum aman.
- **Jangan pernah `--force` ke `main` atau `develop`.**
- **Kalau ragu perubahannya terlalu besar untuk satu commit, dia memang terlalu besar.**
