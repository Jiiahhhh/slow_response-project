# Tahap 4 — Release

**Status keseluruhan:** BELUM DIMULAI — jauh ke depan, ±3 sesi terakhir dari ±47

Ini tracker kerja untuk Tahap 4, tahap terakhir grand plan. Alasan dan urutannya
ada di [[00-Grand-Plan]]. File ini disiapkan sekarang bukan supaya dikerjakan
sekarang, tapi supaya struktur "satu tahap satu tracker" konsisten sejak awal —
dan supaya nanti, saat Tahap 3 hampir tuntas, tidak perlu bingung lagi harus mulai
dari mana.

Prasyarat: seluruh milestone M1–M5 di [[03-Tahap3-Build]] selesai dan sudah
didemokan.

---

## R1 — UAT

**Status:** BELUM

Checklist yang dicoba sendiri oleh "klien" — dalam hal ini kamu, dengan sudut
pandang pengguna, bukan pembuat. Tiap Acceptance di M1–M5 (lihat
[[00-Grand-Plan]]) jadi satu baris centang di sini saat waktunya tiba.

### Checklist

- [ ] Acceptance M1 dicoba ulang dari sudut pandang pembaca
- [ ] Acceptance M2 dicoba ulang dari sudut pandang editor
- [ ] Acceptance M3–M5 dicoba ulang
- [ ] Bug yang ketemu dicatat dan diperbaiki sebelum lanjut ke R2

---

## R2 — Deploy

**Status:** BELUM

**Selesai jika:** sistem jalan di luar laptop, minimal satu lingkungan staging.

### Checklist

- [ ] Target deployment diputuskan (lihat X4 batch "ditunda sampai dibutuhkan")
- [ ] Runbook: cara menyalakan, cara mematikan
- [ ] Runbook: apa yang dilakukan kalau macet
- [ ] Environment variable dan secret dipindah dari dev ke staging dengan aman

---

## R3 — Handover

**Status:** BELUM

**Selesai jika:** orang asing bisa mengikuti dokumen ini sampai sistemnya hidup,
tanpa bertanya.

### Checklist

- [ ] README lengkap di root repo
- [ ] Dokumen arsitektur final (CLAUDE.md sudah sebagian, perlu ditinjau ulang)
- [ ] Catatan "apa yang dikerjakan setengah dan kenapa" — ini yang paling
      dihargai orang yang mewarisi proyek, jangan dilewatkan demi terlihat rapi

---

## Selesai jika (Tahap 4 keseluruhan)

R1, R2, R3 selesai. Proyek ini resmi bisa disebut selesai dari nol sampai
diserahkan — bukan cuma "jalan di laptop saya".
