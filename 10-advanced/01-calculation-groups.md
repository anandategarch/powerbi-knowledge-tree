# Calculation Groups: Satu Measure, Banyak Transformasi Waktu

> Calculation group adalah tabel khusus berisi *calculation items* yang memodifikasi measure apa pun di visual lewat `SELECTEDMEASURE()`. Ini jawaban langsung untuk masalah "ledakan measure": keluarga `[Deviasi CM] [Deviasi LM] [Deviasi YTD] [Deviasi YoY] [% BOM CM] [% BOM LM] …` di report Cost Control (total 161 measures) dapat diringkas menjadi beberapa base measure + satu calculation group.

## Konsep & Penjelasan

### Masalah yang diselesaikan
Pola duplikasi di report ini (dan hampir semua report cost control):

```text
Base metric      Transformasi waktu      →  Measures duplikat
Deviasi       ×  CM / LM / YTD / YoY     →  Deviasi CM, Deviasi LM, Deviasi YTD, Deviasi YoY
% BOM         ×  CM / LM / YTD / YoY     →  % BOM CM, % BOM LM, % BOM YTD, % BOM YoY
Waste         ×  CM / LM                 →  Waste CM, Waste LM …
```

Matriks ini tumbuh kuadratik: 5 metric × 4 transformasi = 20 measure, padahal logikanya hanya 5 + 4. Setiap perbaikan rumus harus disuntik ke banyak tempat — sumber bug klasik "CM sudah diperbaiki tapi YTD lupa".

### Cara kerja calculation group
- Calculation group = tabel dengan satu kolom (misal `Periode Comparison`) berisi baris items: `CM`, `LM`, `YTD`, `YoY`.
- Setiap item punya **ekspresi DAX** yang mengubah nilai measure di visual: di dalamnya, `SELECTEDMEASURE()` mewakili "measure apa pun yang sedang dievaluasi".
- Saat kolom calculation group dipakai di slicer/matrix/kolom visual, **semua measure** pada visual itu otomatis dihitung ulang untuk setiap item terpilih.
- Setiap item bisa punya **format string sendiri** (dynamic format strings), termasuk berbeda antar item.

### Di mana dibuat
- **Power BI Desktop model view** (Model > New calculation group) — cukup untuk kasus umum.
- **Tabular Editor 2 (gratis, external tool)** — kontrol penuh: precedence, format string expression, penamaan teknis. Untuk report ini, TE2 sepadan karena butuh koordinasi dua calculation group.

### Application: slicer dari kolom calculation group
- Drag kolom `Periode Comparison` ke slicer → pengguna memilih `LM` → semua card/matrix di halaman menampilkan versi last-month.
- Bisa juga ditaruh sebagai kolom matrix: baris = `Resto[Area]`, kolom = `Periode Comparison`, values = `[Deviasi]` → satu visual menggantikan empat kolom measure.

### Precedence (calc group vs calc group)
- Jika ada dua calculation group aktif dalam satu konteks (mis. `Periode Comparison` + `Attribute` untuk NOMINAL/QTY), urutan penerapan diatur **precedence**.
- Aturan: calculation group dengan **angka precedence lebih kecil diterapkan lebih dulu** (paling dalam), lalu group berikutnya melihat hasilnya lewat `SELECTEDMEASURE()`.
- Contoh: set `Attribute` = 1 (diterapkan dulu: memilih NOMINAL/QTY pada base), `Periode Comparison` = 10 (diterapkan kemudian: CM/LM/YTD/YoY pada hasilnya).

### Batasan yang harus diketahui
- `SELECTEDMEASURE()` **hanya bermakna di dalam calculation item** — dipakai di measure biasa = error/vaue kosong.
- **Klien MDX klasik (mis. pivot Excel lama via Analyze in Excel) tidak menerapkan calculation group** — angka tampil mentah. Uji dulu di kanal yang dipakai manajemen.
- Kolom calculation group **tidak bisa dipakai dalam relationship** dan tabelnya tidak boleh direlasikan.
- Satu measure hanya dimodifikasi **sekali oleh tiap calculation group** (tidak rekursif dalam satu group).
- Calculation group **tidak bisa dibuat lewat DAX calculated table** di Desktop — harus lewat model view/Tabular Editor.
- Format string item mengubah **tampilan**, bukan nilai (export raw tetap angka asli).

## Sintaks & Contoh

Calculation group `Periode Comparison` untuk domain deviasi (persis pola report ini):

```dax
-- ============ CALCULATION GROUP: Periode Comparison ============
-- Item "CM" (identitas — WAJIB ada sebagai pilihan dasar)
CM  = SELECTEDMEASURE()

-- Item "LM" — menggantikan measure [Deviasi LM] dst.
LM  = CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Date'[Date]))

-- Item "YTD" — berhenti di cutoff periode (pola Selected Cutoff report ini)
YTD = CALCULATE(
    SELECTEDMEASURE(),
    DATESYTD('Date'[Date]),
    'Date'[Date] <= [Selected Cutoff Date]
)

-- Item "YoY"
YoY = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Date'[Date]))

-- Item "Growth %" — sambil memakai ABS denominator ala [Growth] report ini
Growth % =
VAR Base   = SELECTEDMEASURE()
VAR Prev    = CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Date'[Date]))
RETURN DIVIDE(Base - Prev, ABS(Prev))

-- Format string per item (dynamic format string):
--   CM/LM/YTD/YoY  : "#,##0"
--   Growth %       : "0.0%;-0.0%;0.0%"
```

Sebelum–sesudah refactor (semangatnya, bukan teks DAX persis model):

```dax
-- SEBELUM (4 + 4 measures duplikat):
Deviasi CM  = CALCULATE([Value Dev All], Date[BulanIni])
Deviasi LM  = CALCULATE([Value Dev All], PREVIOUSMONTH(Date[Date]))
Deviasi YTD = CALCULATE([Value Dev All], DATESYTD(Date[Date]))
Deviasi YoY = CALCULATE([Value Dev All], SAMEPERIODLASTYEAR(Date[Date]))
% BOM CM    = DIVIDE([BOM CM], [Sales CM])   … (dan seterusnya tiap metric)

-- SESUDAH (2 base measures + 1 calc group):
Deviasi := [Value Dev All]
%BOM    := DIVIDE([Value BOM All], [Value Sales All])
-- kolom slicer: 'Periode Comparison'[Periode Comparison]
-- → satu visual matrix: Area × (CM|LM|YTD|YoY|Growth%) dengan [Deviasi] saja
```

Langkah cepat di Tabular Editor 2:

```text
1. Buka model lewat External Tools > Tabular Editor
2. Klik kanan Tables > Create new > Calculation group
3. Rename → "Periode Comparison"; tambahkan calculation items
4. Isi Format String Expression bila perlu
5. Save (write ke model) → refresh metadata di Desktop
```

## Kesalahan Umum
- **Lupa item identitas (CM/current)** → slicer tidak punya pilihan "bulan berjalan"; perbaikan: item pertama = `SELECTEDMEASURE()`.
- **`SELECTEDMEASURE()` ditulis di measure biasa** → tidak valid; perbaikan: hanya dalam calculation item.
- **Precedence salah** → kombinasi dua calc group menghasilkan angka aneh; perbaikan: uji matriks kecil, kecil = diterapkan dulu.
- **Format string dilupakan** → item Growth % tampil seperti ribuan; perbaikan: isi format string per item.
- **Berganti-ganti antara calc group dan measure switching lama** (`Trend[Multi Value CM]`) pada visual yang sama → dobel transformasi; perbaikan: pilih satu mekanisme per visual (lihat [Field Parameters](./02-field-parameters.md)).
- **Migrasi big-bang 161 measures** → risiko tinggi; perbaikan: bertahap per keluarga halaman, jadikan measure lama wrapper sementara.

## Tips Pro
- Refactor bertahap: buat calc group, biarkan measure lama tetap ada sebagai wrapper (`Deviasi LM := CALCULATE([Deviasi], 'Periode Comparison'[…] = "LM")` — atau cukup alias), migrasi visual per halaman, baru hapus yang lama di commit terpisah.
- Beri prefix `_` / display folder `00 Base` untuk base measures supaya field list tetap rapi.
- Dua calc group (Attribute: NOMINAL/QTY + Periode: CM/LM/YTD/YoY) + [Field Parameters](./02-field-parameters.md) untuk dimensi → menggabung halaman by Area/by PIC/by Resto/by Item menjadi satu halaman adaptif.
- Audit hasil refactor dengan [DAX Query View](./05-dax-query-view.md): bandingkan grid measure lama vs calc group sebelum hapus yang lama.
- Catat setiap item calc group di dokumentasi repo (bukan hanya di kepala pembuat).

## Terkait
- [Field Parameters](./02-field-parameters.md)
- [Dynamic Visuals & Formatting](../10-advanced/04-dynamic-visuals-and-formatting.md)
- [DAX Query View](./05-dax-query-view.md)
- [Analisis Model & DAX](../11-user-report-analysis/model-and-dax.md)
- [Errors Umum](../12-troubleshooting/common-errors.md)
