# Fase 0 — Perbaiki Fondasi

Tujuan fase ini: bikin `./gradlew projects` di root berhasil menampilkan ketiga
modul, dan `git status` bersih dari noise. Murni teknis, tidak butuh konsep produk
apapun. Urutan di bawah ini penting — tiap task dibangun di atas task sebelumnya.

Centang tiap task di file ini begitu selesai, biar kamu bisa lihat progress kalau
kembali besok/minggu depan.

---

## 1. Rename `setting.gradle` → `settings.gradle`

**Kenapa:** Gradle secara default hanya mencari file bernama persis
`settings.gradle`. File saat ini bernama `setting.gradle` (tanpa "s") — jadi baris
`include 'web_portal'`, `include 'admin_portal'`, `include 'content_service'` di
dalamnya **tidak pernah terbaca**. Root build selama ini buta terhadap modul-modulmu.

- [ ] `git mv setting.gradle settings.gradle`
- [ ] Buka isinya, pastikan masih 3 baris `include` yang benar (harusnya belum
      berubah, cuma nama filenya)

**Cara verifikasi:** belum bisa langsung diverifikasi — tunggu sampai task 2 selesai.

---

## 2. Selesaikan konflik versi Gradle

**Kenapa:** wrapper di root (`gradle/wrapper/gradle-wrapper.properties`) menunjuk
ke Gradle **9.3.0**. Wrapper di tiap modul (`web_portal/gradle/wrapper/...`, dst)
menunjuk ke Gradle **7.6.4**. Grails 6.2.3 + `grails-gradle-plugin:6.2.4` (lihat
`build.gradle` tiap modul) tidak kompatibel dengan Gradle 9 — build dari root akan
gagal atau berperilaku aneh.

CCWR (`/Users/ilal/canon-corporate-website-revamp/gradle/wrapper/gradle-wrapper.properties`)
memakai satu wrapper konsisten di 7.6.4 untuk semua level. Ini yang perlu kamu tiru.

**Keputusan yang perlu kamu ambil** (bukan saya — ini trade-off kecil, latihan
bikin keputusan teknis):

- **Opsi A** — turunkan wrapper root ke 7.6.4, samakan dengan modul. Root jadi bisa
  menjalankan build multi-project beneran (`./gradlew :web_portal:bootRun` dari
  root, dsb). Ini yang dipakai CCWR.
- **Opsi B** — hapus wrapper root sepenuhnya (`gradlew`, `gradlew.bat`, folder
  `gradle/` di root), dan tetap `cd` ke tiap modul untuk build seperti yang mungkin
  sudah kamu lakukan selama ini. Lebih simpel, tapi kamu kehilangan kemampuan
  `./gradlew projects` dan build lintas-modul dari satu tempat.

Untuk meniru pola CCWR secara utuh, **Opsi A lebih disarankan** — tapi keputusan
tetap di tanganmu, catat pilihanmu di [[04-Decisions-Log]].

- [ ] Putuskan Opsi A atau B, catat di Decisions Log
- [ ] Kalau Opsi A: edit `gradle/wrapper/gradle-wrapper.properties`, ganti
      `distributionUrl` ke versi 7.6.4 (format URL-nya sama seperti yang ada di
      wrapper tiap modul — tinggal contek pola URL-nya, ganti angka versi)
- [ ] Kalau Opsi A: jalankan `./gradlew wrapper --gradle-version 7.6.4` sekali dari
      root supaya `gradle-wrapper.jar` ikut ter-regenerate dengan benar (bukan cuma
      edit properties manual)
- [ ] Kalau Opsi B: hapus `gradlew`, `gradlew.bat`, `gradle/` di root

**Cara verifikasi:**
```
./gradlew --version
```
harus melaporkan 7.6.4 (Opsi A), lalu
```
./gradlew projects
```
harus menampilkan `:web_portal`, `:admin_portal`, `:content_service`.

---

## 3. Root `.gitignore` + untrack `.gradle/`

**Kenapa:** `.gradle/` (cache & lock file biner) ter-commit ke git sejak commit
awal. Itu sebabnya `fileHashes.lock` selalu muncul sebagai "modified" tiap kali
kamu build — bukan bug, itu memang file yang berubah tiap build dan seharusnya
tidak pernah masuk git.

Tiap modul (`web_portal/.gitignore`, dst) sudah punya `.gitignore` sendiri yang
menangani `build/` di level modul. Yang belum ada cuma di level **root**.

Untuk referensi isi yang wajar, buka `/Users/ilal/canon-corporate-website-revamp/.gitignore`
langsung — lihat pola apa yang mereka ignore, lalu putuskan sendiri mana yang
relevan buat repo ini (gak perlu 100% sama).

- [ ] Buat `.gitignore` di root, minimal cover `.gradle/` dan `build/` di level root
- [ ] `git rm -r --cached .gradle` (untrack tanpa hapus file fisiknya)
- [ ] Cek `git status` — pastikan `.gradle/...` tidak lagi muncul

**Cara verifikasi:** build ulang salah satu modul, lalu `git status` — kalau bersih
(tidak ada noise dari `.gradle/`), berarti berhasil.

---

## 4. Naikkan `sourceCompatibility` ke 17

**Kenapa:** ketiga `build.gradle` set `sourceCompatibility = JavaVersion.toVersion("11")`,
padahal JDK yang terpasang di mesinmu adalah 17 (Corretto), dan CCWR sendiri target
17. Gak ada alasan tetap di 11.

- [ ] Edit baris `sourceCompatibility` di `web_portal/build.gradle`,
      `admin_portal/build.gradle`, `content_service/build.gradle` — ganti `"11"` ke
      `"17"`

**Cara verifikasi:** per modul, jalankan `./gradlew compileGroovy` — harus sukses
tanpa warning versi Java.

---

## 5. Commit pekerjaan yang menggantung

**Kenapa:** sejak commit awal (`0e81bc5`), ada perubahan uncommitted: 3 baris
`server.port` di tiap `application.yml`, file baru `docker/docker-compose.yml`,
dan `.idea/vcs.xml`. Ini kerjaan beneran (bukan noise) — sayang kalau hilang atau
ketumpuk task lain.

- [ ] `git status` — review satu-satu apa yang berubah
- [ ] Commit **terpisah** dari commit "untrack .gradle" di task 3 — jangan
      digabung, supaya riwayat git jelas mana perbaikan tooling vs mana fitur
      infra beneran
- [ ] Pesan commit contoh: `chore: add docker compose infra and per-module ports`

---

## 6. Sanity check akhir

- [ ] `./gradlew projects` dari root sukses menampilkan 3 modul
- [ ] `git status` bersih (tidak ada noise `.gradle/`, tidak ada perubahan
      menggantung yang lupa di-commit)
- [ ] Ketiga modul masih bisa `bootRun` sendiri-sendiri seperti sebelumnya (tidak
      ada yang rusak akibat perubahan di atas)
- [ ] Update baris "Status saat ini" di [[00-Index]] kalau semua sudah beres

Begitu semua tercentang, lanjut ke [[02-Roadmap-Backlog]] bagian Fase 1.
