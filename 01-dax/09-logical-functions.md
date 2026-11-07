# Logical Functions: IF, SWITCH, dan Perilaku BLANK

> Fungsi logika memilih hasil berdasarkan kondisi — dasar dari kategorisasi ("Surplus"/"Loss"), guard pembagian, sampai measure multi-mode. Pahami juga perilaku BLANK dalam aritmatika: sebagian besar "angka aneh" di report cost-control berasal dari BLANK yang diam-diam berubah menjadi 0 atau Infinity.

## Konsep: Cabang Logis dan Evaluasi Malas

`IF` mengevaluasi cabang yang **dibutuhkan saja** (lazy / short-circuit) — cabang lain tidak dihitung. Hal yang sama berlaku pada operator `&&` dan `||`. Konsekuensinya: cabang yang berpotensi error atau mahal tidak akan dieksekusi selama kondisinya tidak tercapai. Ini penting untuk performa dan untuk guard error.

## Function Reference

| Function | Deskripsi | Contoh |
|---|---|---|
| IF | Cabang kondisional: benar/salah | `IF([Deviasi CM] >= 0, "Surplus", "Loss")` |
| IF.EAGER | IF yang selalu mengevaluasi kedua cabang | `IF.EAGER([A] > 0, [B], [C])` |
| AND / OR | Gabung dua kondisi (fungsi, maks. 2 argumen) | `AND([A] > 0, [B] > 0)` |
| `&&` / `\|\|` | Operator AND / OR, bisa dirantai | `[A] > 0 && [B] > 0` |
| NOT | Negasi kondisi | `NOT ISBLANK([Deviasi CM])` |
| SWITCH | Cabang banyak kondisi / dictionary lookup | `SWITCH(TRUE(), ...)` |
| IFERROR | Tangkap error, kembalikan nilai pengganti | `IFERROR([Growth], BLANK())` |
| COALESCE | Kembalikan argumen pertama yang tidak BLANK | `COALESCE([Deviasi CM], [Deviasi LM], 0)` |
| BLANK | Nilai kosong | `IF([X] < 0, BLANK())` |
| TRUE / FALSE | Konstanta boolean | `SWITCH(TRUE(), ...)` |

## Sintaks & Contoh

### IF — kategorisasi sederhana

```dax
-- Measure [Kategori] asli di report:
Kategori = IF([Deviasi CM] >= 0, "Surplus", "Loss")

-- Guard kombinasi kondisi dengan operator (bukan fungsi AND/OR):
Flag Loss =
IF(
    NOT ISBLANK([Deviasi CM]) && [Deviasi CM] < 0,
    1,
    0
)
```

### SWITCH — dictionary dan rule list

Dua wajah SWITCH: mencocokkan **nilai** (dictionary), atau mencocokkan **kondisi** lewat `SWITCH(TRUE(), ...`:

```dax
-- Dictionary: satu nilai → satu measure (asli [Multi Value CM], dipadatkan)
Multi Value CM =
SWITCH(
    SELECTEDVALUE('Trend'[Kategori]),
    "DEVIASI", [Deviasi CM],
    "BOM",     [BOM CM],
    "% BOM",   [% BOM CM],
    "% SALES", IF(SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL", [% Sales CM], BLANK()),
    BLANK()    -- nilai lain / tanpa seleksi
)

-- Rule list: kategorisasi deviasi bertingkat (grade)
Grade Deviasi =
SWITCH(
    TRUE(),
    [% BOM CM] <= -0.05, "Kritis",   -- BOM minus lebih dari 5%: investigasi wajib
    [% BOM CM] <  0,     "Waspada",
    [% BOM CM] <  0.02,  "Aman",
    "Surplus"
)

-- Kategorisasi anomali berbasis Z Score (asli):
Kategori Anomali Wasutri =
SWITCH(
    TRUE(),
    ABS([Z Score]) > 1, "Anomali",
    "Normal"
)
```

`SWITCH(TRUE(), ...)` adalah daftar prioritas: kondisi **pertama** yang benar menang — urutkan dari paling spesifik/paris ke paling longgar.

### IFERROR dan COALESCE

```dax
-- Pencegahan terbaik ada di sumber: pakai DIVIDE, bukan pembagian biasa
Growth = DIVIDE([Selisih], ABS([Deviasi LM]))

-- IFERROR hanya untuk operasi yang benar-benar bisa error tak terduga:
Growth (fallback) = IFERROR([Growth], BLANK())

-- COALESCE: nilai pengganti berantai (display)
Deviasi LM (tampil) = COALESCE([Deviasi LM], 0)
Nilai Alternatif = COALESCE([Deviasi CM], [Deviasi LM], 0)
```

### Perilaku BLANK dalam aritmatika

BLANK bukan nol — tetapi sebagian operasi memperlakukannya seperti nol, sebagian lagi menularkannya:

| Ekspresi | Hasil | Catatan |
|---|---|---|
| `BLANK() + 5` | 5 | BLANK dianggap 0 pada penjumlahan |
| `BLANK() - 5` | -5 | sama, pada pengurangan |
| `[CM] - [LM]` saat [LM] blank | [CM] penuh | sumber "gap palsu" klasik |
| `BLANK() * 5` | BLANK | perkalian menularkan BLANK |
| `5 / BLANK()` | Infinity | inilah alasan pakai DIVIDE |
| `DIVIDE(5, BLANK())` | BLANK | DIVIDE menelan kasus ini dengan aman |
| `BLANK() = 0` | TRUE | koersi saat perbandingan — hati-hati di IF |
| `BLANK() & " gr"` | " gr" | BLANK jadi string kosong saat konkatenasi |

```dax
-- Guard klasik: jangan biarkan BLANK berubah jadi 0 di pengurangan
Selisih (aman) =
IF(
    ISBLANK([Deviasi LM]) || ISBLANK([Deviasi CM]),
    BLANK(),
    [Deviasi CM] - [Deviasi LM]
)
```

Catatan agregasi: `SUM` dan `AVERAGE` **mengabaikan** baris BLANK (bukan dihitung 0). Kalau baris tanpa data harus dihitung sebagai 0, gunakan `COALESCE` per baris di dalam iterator.

### IF.EAGER dan nested IF

```dax
-- IF.EAGER mengevaluasi KEDUA cabang walau kondisi sudah pasti.
-- Pakai hanya saat kedua cabang sangat murah dan evaluasi kondisional
-- justru menjadi overhead — selain itu, tetap pakai IF biasa.

-- Nested IF lebih dari 2 level = tanda ganti SWITCH:
-- buruk:
IF([A] > 10, "Tinggi", IF([A] > 5, "Sedang", "Rendah"))
-- baik:
SWITCH(TRUE(), [A] > 10, "Tinggi", [A] > 5, "Sedang", "Rendah")
```

## Kesalahan Umum

- **Lupa BLANK dianggap 0 pada + dan −** → `[Selisih] = [Deviasi CM] - [Deviasi LM]` dengan LM blank menghasilkan Selisih = CM penuh; Growth jadi +100% palsu. *Perbaikan*: guard `ISBLANK` seperti contoh di atas, atau biarkan BLANK memang "belum ada data".
- **SWITCH(TRUE()) dengan urutan kondisi salah** → kondisi umum di atas kondisi spesifik menelan semua kasus. *Perbaikan*: urutkan dari paling spesifik/paris ke paling longgar; kasus default di posisi terakhir.
- **IF dengan tipe hasil beda** (`IF(x, "teks", 0)`) → implicit conversion; hasil jadi variant, sorting dan agregasi berperilaku aneh, kadang error saat dipakai operasi numerik. *Perbaikan*: samakan tipe semua cabang — pakai `BLANK()` atau `0` secara konsisten.
- **`[X] = BLANK()` dipakai untuk cek kosong** → bernilai TRUE juga saat `[X] = 0` (koersi blank = 0). *Perbaikan*: pakai `ISBLANK([X])`.
- **Fungsi AND()/OR() diberi 3+ argumen** → error (maksimal 2). *Perbaikan*: gunakan operator `&&` / `||`.
- **Menyelimuti semuanya dengan IFERROR** → menyembunyikan bug dan membayar biaya evaluasi error. *Perbaikan*: cegah error di sumber (DIVIDE, ISBLANK, guard tipe); IFERROR hanya jaring pengaman terakhir.

## Tips Pro

- Lebih dari 2 cabang → SWITCH; tambahkan komentar di tiap cabang agar terbaca seperti tabel aturan bisnis (grade deviasi, kategori anomali).
- **VAR untuk kondisi mahal**: hitung sekali, pakai berkali — `VAR Cond = ... RETURN IF(Cond, ...)`.
- Untuk status "belum ada data", kembalikan `BLANK()` (bukan string "0" atau "-") — BLANK otomatis rapi di visual (sel kosong) dan tidak mengotori agregasi.
- Tipe hasil konsisten antar cabang: pilih "semua angka" atau "semua teks" sejak desain.
- Short-circuit berguna untuk guard: tarik kondisi paling murah/paling sering salah di depan pada rantai `&&`.

## Terkait

- [CALCULATE & Filter](./06-calculate-and-filter-functions.md) — IF sebagai pembungkus CALCULATE bercabang
- [Time Intelligence](./07-time-intelligence.md) — guard ISBLANK pada measure LM/YoY
- [Information Functions](./10-information-functions.md) — ISBLANK, ISFILTERED sebagai kondisi
- [Text Functions](./11-text-functions.md) — menggabungkan hasil IF dengan FORMAT dan label dinamis
- [Data Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md) — measure [Kategori], [Multi Value CM], [Kategori Anomali Wasutri]
