# Rekonsiliasi — Codebase vs Vault

Prosedur yang dijalankan di Langkah 1 tiap sesi, sebelum menawarkan pekerjaan apa
pun. Tujuannya: vault bisa salah, klaim status bisa basi, dan kode bisa lebih maju
atau lebih mundur dari yang dicatat. Jangan pernah mempercayai satu sumber saja.

Ini **metode**, bukan snapshot. Jangan menyalin contoh drift di bawah sebagai
"kondisi sekarang" — jalankan ulang tiap sesi, karena kondisinya berubah tiap sesi.

## Yang dibandingkan

**1. Klaim status vs isi folder sungguhan**

Baca baris `**Status keseluruhan:**` di:
- `docs/vault/01-rencana/01-Tahap1-Discovery.md`
- `docs/vault/01-rencana/02-Tahap2-Design.md`
- `docs/vault/01-rencana/03-Tahap3-Build.md`
- `docs/vault/01-rencana/04-Tahap4-Release.md`

Lalu cek isi folder kode sungguhan (`find`/`ls`) untuk modul yang relevan —
minimal `content_service/grails-app/{domain,services,controllers}` dan
`content_service/src/test/groovy`. Kalau ada file yang isinya lebih maju dari
status yang tercatat (misal domain sudah ditulis padahal Tahap 2 masih BELUM),
atau sebaliknya (status bilang SELESAI tapi filenya tidak ada), itu **drift** —
sebutkan di ringkasan pembuka, jangan didiamkan.

**2. Checkbox Slice-Backlog vs isi folder**

`docs/vault/02-kerja/00-Slice-Backlog.md` punya slice S2-1 dst dengan checkbox.
Cocokkan tiap slice yang tercentang dengan bukti nyata di kode (file yang
disebutnya ada, endpoint yang disebutnya jalan). Slice tercentang tanpa bukti di
kode adalah tanda paling jelas backlog sudah tidak dipercaya — perbaiki sebelum
lanjut, jangan menumpuk kebohongan kecil di atasnya.

**3. Working tree**

`git status` dan `git log --oneline -5`. File yang menggantung lama (lihat
tanggal via `git log -1 --format=%cd -- <file>` kalau perlu) adalah sinyal sesi
lalu berhenti di tengah jalan.

**4. Sanity compile (opsional, kondisional)**

Kalau Journal entry terakhir menyebut sesuatu belum selesai/macet, atau ada file
yang diubah tapi belum pernah dijalankan sejak commit terakhir, jalankan compile
cepat untuk modul yang tersentuh:

```
./gradlew :content_service:compileGroovy -q
```

Jangan jalankan ini tiap sesi tanpa alasan — ini bukan test suite, cuma sanity
check murah untuk menangkap "sesi lalu berhenti di tengah kode yang rusak". Kalau
lambat atau timeout, lewati dan sebutkan bahwa itu belum dicek.

## Melaporkan hasilnya

Satu baris di ringkasan pembuka, cukup jujur:

```
Rekonsiliasi: <cocok / ada drift — sebutkan apa>
```

Kalau ada drift yang cukup besar (misal kode sudah jauh di depan status tercatat,
seperti yang pernah terjadi dengan domain `content_service` ditulis sebelum
Tahap 1-2 kelar), itu sendiri bisa jadi pekerjaan hari ini: merapikan
pencatatan sebelum lanjut ke apa pun yang lain. Jangan menutup mata demi cepat
mulai ngoding — itu persis kebiasaan yang bikin vault ini pernah tidak
dipercaya.
