# Text Functions: FORMAT, CONCATENATE, dan Manipulasi Teks

> Fungsi teks mengubah angka dan tanggal menjadi tampilan (format rupiah, "12,5 kg", panah tren) serta membersihkan dan mengelompokkan data teks. Aturan emasnya: hasil fungsi teks adalah **TEXT** — pisahkan measure angka dari measure tampilan, atau visual Anda berhenti bisa menghitung dan mengurutkan.

## Konsep: Teks untuk Tampilan, Angka untuk Kalkulasi

DAX memisahkan tegas nilai dan tampilannya. `FORMAT(1234567.89, "#,##0")` menghasilkan string `"1.234.568"` — enak dilihat, tapi tidak bisa dijumlah, dirata-rata, atau diurutkan secara numerik. Pattern yang sehat: **measure angka** untuk kalkulasi/axis/sort, **measure tampilan** (ber-format) untuk kartu, label, tooltip.

## Function Reference

| Function | Deskripsi | Contoh |
|---|---|---|
| FORMAT | Ubah angka/tanggal menjadi teks berformat | `FORMAT([Deviasi CM], "#,##0")` |
| CONCATENATE / `&` | Gabung dua teks (operator `&` bisa dirantai) | `"Rp " & [Label]` |
| LEFT / RIGHT / MID | Ambil potongan teks dari kiri/kanan/posisi | `LEFT([Kode], 3)` |
| LEN | Panjang teks | `LEN([Nama Barang])` |
| UPPER / LOWER / PROPER | Kapitalisasi | `UPPER([Area])` |
| TRIM | Buang spasi berlebih (depan-belakang dan ganda di tengah) | `TRIM([Nama Barang])` |
| SUBSTITUTE | Ganti teks **berdasarkan isi** (semua kemunculan) | `SUBSTITUTE([Nama], "  ", " ")` |
| REPLACE | Ganti teks **berdasarkan posisi** | `REPLACE([Kode], 1, 2, "XX")` |
| SEARCH | Posisi substring, case-insensitive, dukung wildcard | `SEARCH("-", [Kode], 1, BLANK())` |
| FIND | Posisi substring, case-sensitive, tanpa wildcard | `FIND("-", [Kode], 1, BLANK())` |
| CONTAINSSTRING / CONTAINSSTRINGEXACT | Cek substring (case-insensitive / exact) | `CONTAINSSTRING([Nama], "RESTO")` |
| VALUE | Teks → angka (parsing sesuai locale) | `VALUE("123")` |
| FIXED | Angka → teks dengan desimal tetap | `FIXED([X], 2, TRUE)` |
| UNICHAR | Karakter dari kode Unicode (simbol, panah, line break) | `UNICHAR(9650)` |

## Format String yang Perlu Dihafal

| Format string | Contoh input | Output (locale Indonesia) |
|---|---|---|
| `"#,##0"` | 1234567.8 | 1.234.568 |
| `"#,##0.00"` | 1234567.8 | 1.234.567,80 |
| `"#,##0.0%"` | 0.1234 | 12,3% |
| `"0.0%"` | -0.052 | -5,2% |
| `"""Rp""#,##0"` | 1234567 | Rp 1.234.567 |
| `"#,##0;(#,##0)"` | -1234 | (1.234) — section negatif |
| `"dd MMM yyyy"` | 2025-03-15 | 15 Mar 2025 |
| `"yyyy-mm"` | 2025-03-15 | 2025-03 (key yang sortable) |
| `"000"` | 7 | 007 (jaga leading zero) |

Dalam format string, `,` adalah *placeholder* ribuan dan `.` placeholder desimal — **karakter yang dirender mengikuti locale model** (Indonesia: titik ribuan, koma desimal). Literal seperti `Rp` di-escape dengan tiga tanda kutip: `"""Rp""#,##0"`.

## Sintaks & Contoh

### Measure tampilan — uang, persen, label dinamis

```dax
-- Measure angka (untuk kalkulasi, axis, sorting):
Deviasi CM = ...   -- tetap numerik

-- Measure tampilan (kartu/label):
Deviasi CM Display =
"Deviasi: " & FORMAT([Deviasi CM], """Rp""#,##0")

-- Qty Waste dengan satuan (pattern asli report):
Qty Waste =
FORMAT(
    CALCULATE(
        SUM('Waste All'[Value]),
        Attribute[Attribute] = "QTY"
    ),
    "#,##0"
) & " " & MAX('Waste All'[SATUAN])
-- hasil contoh: "12,5 kg" — angka rapi + satuan dari baris terlihat

-- Indikator tren dengan panah Unicode:
Growth Label =
IF(
    ISBLANK([Growth]),
    "",
    UNICHAR(IF([Growth] >= 0, 9650, 9660)) & " " & FORMAT([Growth], "0.0%")
)
-- 9650 = panah naik, 9660 = panah turun, UNICHAR(10) = line break
```

### Membersihkan dan mengelompokkan teks

```dax
-- Standarisasi nama barang: buang spasi ganda + kapitalisasi
Nama Bersih = PROPER(TRIM('Inventory'[Nama Barang]))

-- Grouping dari kode: "10-AYAM-01" → "10"
Grup Kode = LEFT('Inventory'[Kode], 2)

-- Ambil segmen setelah tanda "-"
-- SEARCH case-insensitive; NotFoundValue mencegah error saat tidak ketemu:
Nama Setelah Kode =
VAR Posisi = SEARCH("-", [Kode], 1, BLANK())
RETURN
IF(ISBLANK(Posisi), [Kode], MID([Kode], Posisi + 1, LEN([Kode])))

-- Flag nama yang mengandung kata tertentu:
Is Resto Item = CONTAINSSTRING('Inventory'[Nama Barang], "RESTO")
```

### VALUE / FIXED dan round-trip FORMAT

```dax
VALUE("123")          -- 123 (angka)
FIXED([X], 2, TRUE)   -- teks "1234.57" (2 desimal, tanpa pemisah ribuan)

-- ANTI-POLA: format lalu dijumlah
-- SUMX(t, FORMAT([X], "#,##0"))  -- error: FORMAT menghasilkan TEXT

-- Kalau butuh keduanya, buat dua measure:
-- [Deviasi CM]  (angka, untuk kalkulasi)
-- [Deviasi CM Display]  (teks, untuk tampilan)
```

`FORMAT(...)` lalu `VALUE(...)` untuk "pembulatan tampilan" juga anti-pola — pembulatan seharusnya `ROUND`, dan hasil VALUE bergantung locale (lihat kesalahan umum).

## Kesalahan Umum

- **FORMAT untuk angka lalu dijumlah/diagregasi** → hasil sudah TEXT; SUM/SUMX error atau menyusun agregasi yang salah, dan sorting kolom jadi alfabetis (1000 < 900). *Perbaikan*: dua measure terpisah — angka dan tampilan; visual di-sort dengan measure angka.
- **Regional setting comma vs dot** → `VALUE("1.5")` di locale Indonesia dibaca 15 (titik = pemisah ribuan), di locale English dibaca 1,5. *Perbaikan*: jangan parsing teks angka di DAX; konversi tipe di Power Query dengan locale eksplisit.
- **Leading zero hilang** → "007" di-VALUE/di-load sebagai angka menjadi 7. *Perbaikan*: simpan kode sebagai TEXT, atau kembalikan format `FORMAT([Kode], "000")`.
- **SEARCH/FIND tanpa NotFoundValue** → error saat substring tidak ditemukan. *Perbaikan*: selalu isi parameter keempat (mis. `BLANK()`).
- **SUBSTITUTE vs REPLACE tertukar** → SUBSTITUTE bekerja berdasar isi teks (semua kemunculan), REPLACE berdasar posisi karakter. *Perbaikan*: pilih sesuai sifat datanya.
- **Perbandingan teks case-sensitive dianggap sama** → `=` pada teks bersifat case-insensitive di DAX, tapi SEARCH vs FIND dan CONTAINSSTRING vs EXACT berbeda perilaku — pahami mana yang dibutuhkan.

## Tips Pro

- Pisahkan selalu **measure angka vs measure tampilan** (mis. `[Deviasi CM]` dan `[Deviasi CM Display]`) — konvensi nama `- Display` membuat niatnya jelas.
- Kolom key seperti `'Date'[Period] = FORMAT('Date'[Date], "yyyy-mm")` (asli di report) menghasilkan key yang **sortable secara alfabetis sekaligus kronologis** — cocok untuk label sumbu.
- `UNICHAR(10)` untuk multi-baris pada tooltip/kartu; gabungkan dengan `&` dan label dinamis dari `SELECTEDVALUE` (lihat [Information Functions](./10-information-functions.md)).
- Untuk daftar teks agregat (mis. "Top 3 resto penyusut: A, B, C") gunakan `CONCATENATEX` — table function yang dijelaskan di [Table Functions](./12-table-functions.md).
- Kolom sort numerik (mis. `Periode[Sort]`) + *Sort By Column* menggantikan trik teks untuk urutan kustom.

## Terkait

- [Logical Functions](./09-logical-functions.md) — IF/SWITCH menghasilkan teks label
- [Information Functions](./10-information-functions.md) — SELECTEDVALUE sebagai sumber label dinamis
- [Table Functions](./12-table-functions.md) — CONCATENATEX untuk agregasi teks
- [Time Intelligence](./07-time-intelligence.md) — kolom Period yyyy-mm di date table
- [Data Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md) — measure [Qty Waste] dan kolom teks di report asli
