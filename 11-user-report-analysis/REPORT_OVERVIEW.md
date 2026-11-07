# 📊 Overview Report Cost Control — Analisis PBIX User

> File sumber: `report.pbix` (148 MB, format **PBIR-enhanced**, base theme `CY26SU02`).
> Analisis per 19 Okt 2026 via ekstraksi `Report/definition/*` (pages & visuals) + `DataModel` (pbixray).

## Ringkasan Eksekutif

Ini adalah **dashboard Cost Control untuk jaringan restoran (F&B)** yang sangat komprehensif — mencakup analisis **deviasi stok (BOM vs COM)**, **waste/susut**, **purchasing (barang masuk, PO belum PI)**, dan **sales growth** per resto/area/PIC/item.

| Metrik | Nilai |
|---|---|
| Total halaman | **36** (semua visible) |
| Total visual | **539** |
| Total measures DAX | **161** |
| Calculated columns | **109** (mayoritas auto date/time) |
| Calculated tables (DAX) | `Date`, `Trend`, `Attribute`, `Periode` (DATATABLE) |
| Tabel model (PQ) | 16 + helper queries (Parameter/Sample/Transform) |
| Ukuran halaman dominan | 1280×720 (16:9) |

## Struktur Halaman (36)

Halaman dapat dikelompokkan ke dalam **7 area fungsi**:

### 1. Navigasi & Home
- **Home** (10 visual) — landing page dengan card "Last Update Dev", navigasi via 33 action button keseluruhan report
- **End** (1 visual), **Kosongan** (13 visual) — halaman penutup/blank template

### 2. Analisa Deviasi (inti report)
- **Summary Deviasi** (48 visual) — halaman terpadat: KPI deviasi CM vs LM, tren, breakdown per jenis
- **Detail Deviasi** (18) — drill-down detail per item
- **Deviasi by Area** (16) / **by PIC** (16) / **by Resto** (18) / **by Item** (18) — slicing dimensi analisis
- **Data Deviasi** (8) — tabel data mentah

### 3. Perbandingan Antar Periode
- **Antar Periode** (14), **Kontribusi Dev** (15), **Perbandingan** (12), **Deviasi vs BOM** (15), **Last 13 Months** (13)
- Pattern: `Deviasi CM` (current month) vs `Deviasi LM` (last month) via `PREVIOUSMONTH`, plus `TREATAS` untuk mode perbandingan dinamis

### 4. Growth & Top N
- **Growth by Resto** (15), **Growth by Item** (15), **Sales Growth** (8)
- **Top 10 Item Dev** (31), **Top 5 Growth** (29), **Top 5 Waste** (18) — pattern ranking & kontribusi (`% to Total`, `Rank Dev`)

### 5. Waste & Susut
- **Analisa Susut** (12), **Waste by Resto** (16), **Waste by Item** (16), **Data Waste** (8)
- **COM** (15) — consumption & cost of materials
- Fitur menarik: **deteksi anomali via Z-Score** (`Kategori Anomali Wasutri`)

### 6. Purchasing / Barang Masuk
- **Rekap Brg Masuk** (34), **Detail Brg Masuk** (10), **Rekon Brg Masuk** (11) — rekonsiliasi barang masuk sistem vs resto (`BM Sistem` vs `BM Resto`, `Input Salah/Benar`)
- **Rekap PO blm PI** (14), **Detail PO blm PI** (13), **Detail PI** (12) — PO yang belum punya Purchase Invoice, aging `Belum Dibayar`
- **Barang Baru** (11) — monitoring item baru

### 7. Support & Admin
- **Tooltip** (3) — tooltip page khusus
- **ID Dokumen** (6), **Data Sales** (7)

## Komposisi Visual (539 total)

| Tipe | Jumlah | Catatan |
|---|---|---|
| `slicer` | 292 | Dominan! Hampir semua halaman punya slicer sinkron (sync slicers) — date, resto, area, PIC, periode |
| `shape` | 76 | Elemen desain (header, garis, background) |
| `actionButton` | 33 | Navigasi antar halaman (button + bookmark) |
| `barChart` | 28 | Bar/column chart utama untuk deviasi & growth |
| `pivotTable` (Matrix) | 27 | Matrix/pivot untuk summary per dimensi |
| `cardVisual` | 24 | KPI cards (Deviasi CM, % BOM CM, dll.) |
| `tableEx` (Table) | 20 | Tabel detail |
| `lineClusteredColumnComboChart` | 10 | Combo line+column (tren vs target) |
| `textbox` | 9 | Judul & narasi |
| `donutChart` | 5 | Komposisi (kontribusi deviasi) |
| Custom: Text Filter | 2 | Visual custom untuk filter teks |
| Custom: Scrolling Text | 1 | Ticker/running text |
| `waterfallChart` | 1 | Analisa susut (bridge) |

## Data Model

### Skema (star-schema modifikasi)
- **Dimensi**: `Inventory` (master barang), `Resto` (master cabang: Resto/Area/PIC/AM/Type Kitchen), `Date` (CALENDARAUTO), `Periode` (WEEK 1/2/3/EOM), `Attribute` (NOMINAL/QTY), `Trend` (switch kategori), `Standar Susut`
- **Fakta**: `Deviasi All` (BOM/COM/USAGE/WASTE/SUSUT/TRIAL/LOSS-SURPLUS × QTY/NOMINAL, unpivot), `Waste All` (6 kategori waste), `Sales All` (penjualan bersih per cabang per bulan)
- **Purchasing**: `9901` (PO detail), `3243` (status pesanan pembelian), `4205`, `3224`, `Rekap Mentah BM`, `Update` (log update)

### Relasi utama (semua M:1 single-direction)
```
Deviasi All[Nama Barang] → Inventory[(Old) Nama Barang]
Waste All[Nama new item] → Inventory[(Old) Nama Barang]
Standar Susut[Nama Barang] → Inventory[(Old) Nama Barang]
Sales All[Nama Cabang] → Resto[Resto]
Deviasi All[RESTO] → Resto[Resto]
```

### Sumber data (Power Query)
- **Folder Google Drive** (`H:\My Drive\COST CONTROL\...`) — Excel per periode dengan konvensi nama file `YYYY_MM_periode.xlsx` (contoh: `Deviasi`, `Kategori Waste`, `1316` untuk sales)
- **Google Sheets publish** — master `Inventory` via `Web.Contents("https://docs.google.com/spreadsheets/d/e/...")`
- Pattern PQ khas: `Folder.Files` → filter hidden/~$ → `Transform File` function → expand → **Unpivot** `QTY_*`/`NOMINAL_*` → split delimiter `_` → kolom `Attribute`+`Jenis` → parse tanggal dari nama file

## Teknik DAX Unggulan yang Dipakai Report Ini

1. **Dynamic period comparison via TREATAS** — `Deviasi CM` / `Deviasi LM` memakai tabel `Mode` + `TREATAS(VALUES(...))` agar bisa memilih rentang pembanding secara dinamis
2. **Z-Score anomaly detection** — `Z Score` dengan `SUMMARIZE` + `AVERAGEX` untuk flag resto/item anomali waste
3. **YTD dengan cutoff periode mingguan** — `Selected Cutoff Date` menggabungkan `SELECTEDVALUE(Date[Year])`, `MonthNumber`, dan `Periode` (WEEK 1/2/3/EOM) untuk YTD yang berhenti di cutoff
4. **SWITCH(TRUE()) pattern** — kategorisasi (Surplus/Loss, Anomali/Normal, kategori analisa)
5. **Measure switching multi-metric** — `Multi Value CM` mengarahkan satu visual ke DEVIASI/BOM/%BOM/%SALES/VARIANCE sesuai slicer `Trend`
6. **Weekly growth ladder** — `Week 1`/`Week 2`/`Week 3`/`EoM` + `Growth W2`/`Growth W3`/`Growth EOM` per tahap periode
7. **DIVIDE dengan ABS denominator** — `Growth = DIVIDE([Selisih], ABS([Deviasi LM]))` menghindari growth negatif-negatif jadi positif
8. **Rekonsiliasi sistem vs aktual** — keluarga measure `BM Sistem`/`BM Resto`/`Input Salah`/`Input Benar`/`% BM`

## Peluang Perbaikan (Rekomendasi)

1. **292 slicer** — pastikan memakai *sync slicers* & grup hierarki (Year→Month→Periode) supaya konsisten antar halaman
2. Kolom kunci `(Old) Nama Barang` & `Nama new item ` (dengan trailing space!) — rawan typo & mismatch join; pertimbangkan ** cleansing key** (TRIM/UPPER) di PQ atau composite key
3. Nama file sebagai sumber tanggal (`Text.Split([Source.Name],"_")`) — rapuh; pertimbangkan kolom periode eksplisit di sumber
4. Beberapa nama measure singkat (`A Dev`, `B Dev`, `Show`, `Trial`) — dokumentasikan di folder ini agar tim paham konteks A/B (Antar periode page memakai pasangan A/B untuk periode pembanding)
5. `DataModel` 145 MB — pertimbangkan *incremental refresh* + import hanya kolom terpakai, atau mode DirectQuery/Composite untuk tabel besar

## File Terkait
- [pages-and-visuals.md](./pages-and-visuals.md) — detail 36 halaman + komposisi visual
- [model-and-dax.md](./model-and-dax.md) — relasi, semua 161 measures DAX lengkap, Power Query inti
