# Debugging — kamus error

Tabel rujukan untuk membantu tanpa mengambil alih. Cara pakainya: temukan barisnya,
jelaskan **kolom "artinya"**, lalu suruh dia sendiri yang mencari di kolom "biasanya
karena". Jangan langsung menyebutkan penyebab spesifik di kodenya.

Kalau error-nya tidak ada di sini, jangan mengarang — cari di dokumentasi Grails 6.2
atau grep pola serupa di `/Users/ilal/canon-corporate-website-revamp`.

---

## GORM & Hibernate

| Pesan | Artinya | Biasanya karena |
|---|---|---|
| `LazyInitializationException: could not initialize proxy - no Session` | Relasi disentuh setelah session Hibernate ditutup | Domain class dikembalikan langsung sebagai response JSON, atau objek dibawa keluar dari method transaksional. Ini alasan utama DTO ada. |
| Objek hasil `save()` bernilai `null`, tanpa exception | Validasi gagal dan `save()` menyerah diam-diam | `save()` dipanggil tanpa `failOnError: true`. Cek `obj.errors`. |
| `StackOverflowError` / recursion saat render JSON | Serializer bolak-balik menyusuri relasi dua arah | Article → Game → Article. Perbaikannya bukan memutus relasi, tapi memakai DTO. |
| `Table 'capstone_db.xxx' doesn't exist` | Skema tidak dibuat | `dbCreate` bukan `update`/`create`, atau app tersambung ke database lain dari yang kamu cek. |
| `Validation error ... unique` | Constraint unik dilanggar | Slug bentrok. Pertanyaan desainnya: apa yang seharusnya terjadi saat dua artikel punya judul sama? |
| Jumlah query membengkak sesuai jumlah baris | N+1 | Relasi diakses di dalam loop. Nyalakan `logSql: true` sementara untuk melihatnya. |
| `object references an unsaved transient instance` | Menyimpan induk yang menunjuk anak yang belum tersimpan | Urutan insert di seed salah, atau cascade tidak diset. |

## Startup & konfigurasi

| Pesan | Artinya | Biasanya karena |
|---|---|---|
| `NoSuchBeanDefinitionException: dataSource` | Grails mencari datasource, tidak ketemu | `hibernate5` dicabut tapi masih ada yang butuh (ini justru **diharapkan** di `web_portal` Fase 4 — yang salah adalah yang masih memanggilnya). |
| `Communications link failure` | Tidak bisa menyambung ke MySQL | Container belum hidup, atau port salah. Proyek ini pakai **3310**, bukan 3306. |
| `Access denied for user` | Kredensial ditolak | `capstone_user` / `capstone_pass`, database `capstone_db`. Cek juga apakah kamu tersambung ke MySQL milik CCWR. |
| `Web server failed to start. Port 9089 was already in use` | Port dipakai proses lain | Instance lama belum mati, atau CCWR sedang jalan. `lsof -i :9089`. |
| Konfigurasi terlihat diabaikan | Blok environment menimpa blok atas | Di `application.yml`, `environments.development.dataSource` menang atas `dataSource` di root. Ini jebakan langganan. |

## Gradle & JDK

| Pesan | Artinya | Biasanya karena |
|---|---|---|
| `Unsupported class file major version` | Bytecode dari JDK yang lebih baru dari yang dipakai | Mesin ini punya beberapa JDK. Proyek ini Java 17. Cek `java -version` dan `JAVA_HOME`. |
| `Could not find method ... for arguments` di build.gradle | Sintaks DSL tidak dikenali versi Gradle ini | Grails 6.2.3 butuh Gradle 7.6.4. Jangan pakai wrapper Gradle 9. |
| `Could not resolve <dependency>` | Artefak tidak ketemu di repository | Salah koordinat, atau repository-nya belum didaftarkan. |
| Perubahan di `shared_lib` tidak terasa di modul lain | Build cache atau `include` tidak terbaca | Cek `settings.gradle` (dengan "s"), dan coba `./gradlew clean`. |

## Test

| Pesan | Artinya | Biasanya karena |
|---|---|---|
| Test lulus sendiri, gagal saat dijalankan bareng | Ada state yang bocor antar test | Data dari test sebelumnya tidak dibersihkan. |
| `No such property` di Spec | Domain tidak di-mock | `@Domain` / `implements DomainUnitTest` belum dipasang. |
| Test pakai H2 tapi kode mengandalkan perilaku MySQL | Beda dialek | Sadari ini sejak awal: test unit pakai H2, verifikasi akhir tetap harus di MySQL. |

---

## Cara mempersempit, bukan menebak

Ajarkan urutan ini — ini yang membedakan menebak dari mendiagnosis:

1. **Baca dari bawah ke atas.** Penyebab asli (`Caused by:`) ada di bawah, bukan di
   baris pertama.
2. **Buang baris framework.** Yang penting adalah baris yang memuat nama package
   proyek (`content_service`, `shared_lib`). Sisanya jalur Grails/Spring.
3. **Persempit ruang masalahnya.** Apakah ini gagal saat compile, saat startup, atau
   saat request? Tiga tempat berbeda, tiga cara mencari berbeda.
4. **Ubah satu hal.** Kalau mengubah tiga hal lalu berhasil, kamu tidak tahu yang mana
   yang memperbaikinya — dan besok ia akan kembali.
5. **Bisakah diulang?** Error yang tidak bisa diulang biasanya soal state atau urutan,
   bukan soal logika.

Alat yang tersedia: `--stacktrace` dan `--info` di Gradle, `logSql: true` di
`application.yml`, `curl -i` untuk melihat status dan header, `docker compose logs -f`
untuk masalah infrastruktur.
