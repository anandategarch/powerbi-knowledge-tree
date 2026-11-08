# Komponen Detail per Halaman — Field Lengkap 539 Visual

> Ekstraksi otomatis dari pbix asli (audit 7 Nov 2026, snapshot pbix 19 Okt 2026).
> Setiap visual dicatat: **tipe**, **judul** (kalau ditampilkan), **field/measure** per role, **sync group** slicer, dan **filter** yang menempel.
> Format field: `Entity[Nama]` (tabel[kolom/measure]). Judul slicer umumnya disembunyikan — label slicer = field-nya.

Lihat juga: [pages-and-visuals.md](./pages-and-visuals.md) (komposisi ringkas) • [model-and-dax.md](./model-and-dax.md) (definisi semua measure) • [REPORT_OVERVIEW.md](./REPORT_OVERVIEW.md)

## Filter Level Report (berlaku ke semua halaman)

- `Resto[Resto]: NOT(Resto = 'Non Resto')`

## Indeks Halaman

| No | Halaman | Visual | Komposisi |
|---|---|---|---|
| 1 | [Home](#1-home) | 10 | 4 shape, 4 cardVisual, 2 emptyContainer (placeholder) |
| 2 | [Summary Deviasi](#2-summary-deviasi) | 48 | 17 shape, 11 slicer, 9 cardVisual, 5 barChart, 4 emptyContainer (placeholder), 1 actionButton, 1 waterfallChart |
| 3 | [Detail Deviasi](#3-detail-deviasi) | 18 | 11 slicer, 3 pivotTable, 2 donutChart, 1 shape, 1 actionButton |
| 4 | [Deviasi by Area](#4-deviasi-by-area) | 16 | 12 slicer, 1 shape, 1 actionButton, 1 pivotTable, 1 lineClusteredColumnComboChart |
| 5 | [Deviasi by PIC](#5-deviasi-by-pic) | 16 | 12 slicer, 1 shape, 1 actionButton, 1 pivotTable, 1 lineClusteredColumnComboChart |
| 6 | [Deviasi by Resto](#6-deviasi-by-resto) | 18 | 13 slicer, 2 shape, 1 actionButton, 1 pivotTable, 1 lineClusteredColumnComboChart |
| 7 | [Deviasi by Item](#7-deviasi-by-item) | 18 | 13 slicer, 2 shape, 1 actionButton, 1 pivotTable, 1 lineClusteredColumnComboChart |
| 8 | [Antar Periode](#8-antar-periode) | 14 | 11 slicer, 1 shape, 1 actionButton, 1 pivotTable |
| 9 | [Kontribusi Dev](#9-kontribusi-dev) | 15 | 10 slicer, 2 pivotTable, 1 shape, 1 actionButton, 1 donutChart |
| 10 | [Perbandingan](#10-perbandingan) | 12 | 9 slicer, 1 shape, 1 actionButton, 1 pivotTable |
| 11 | [Deviasi vs BOM](#11-deviasi-vs-bom) | 15 | 11 slicer, 2 pivotTable, 1 shape, 1 actionButton |
| 12 | [Growth by Resto](#12-growth-by-resto) | 15 | 11 slicer, 1 shape, 1 actionButton, 1 pivotTable, 1 lineClusteredColumnComboChart |
| 13 | [Growth by Item](#13-growth-by-item) | 15 | 11 slicer, 1 shape, 1 actionButton, 1 pivotTable, 1 lineClusteredColumnComboChart |
| 14 | [Analisa Susut](#14-analisa-susut) | 12 | 7 slicer, 2 pivotTable, 1 shape, 1 actionButton, 1 lineClusteredColumnComboChart |
| 15 | [Last 13 Months](#15-last-13-months) | 13 | 10 slicer, 1 shape, 1 actionButton, 1 pivotTable |
| 16 | [Top 10 Item Dev](#16-top-10-item-dev) | 31 | 9 shape, 9 slicer, 5 cardVisual, 5 barChart, 2 emptyContainer (placeholder), 1 actionButton |
| 17 | [Top 5 Growth](#17-top-5-growth) | 29 | 11 slicer, 8 barChart, 8 textbox, 1 shape, 1 actionButton |
| 18 | [Top 5 Waste](#18-top-5-waste) | 18 | 8 slicer, 7 barChart, 1 shape, 1 actionButton, 1 cardVisual |
| 19 | [Waste by Resto](#19-waste-by-resto) | 16 | 11 slicer, 1 shape, 1 actionButton, 1 pivotTable, 1 donutChart, 1 lineClusteredColumnComboChart |
| 20 | [COM](#20-com) | 15 | 10 slicer, 1 shape, 1 actionButton, 1 pivotTable, 1 donutChart, 1 lineClusteredColumnComboChart |
| 21 | [Waste by Item](#21-waste-by-item) | 16 | 11 slicer, 2 pivotTable, 1 shape, 1 actionButton, 1 lineClusteredColumnComboChart |
| 22 | [Rekap Brg Masuk](#22-rekap-brg-masuk) | 34 | 14 shape, 7 slicer, 5 cardVisual, 3 barChart, 3 emptyContainer (placeholder), 1 actionButton, 1 textbox |
| 23 | [Detail Brg Masuk](#23-detail-brg-masuk) | 10 | 7 slicer, 1 shape, 1 actionButton, 1 tableEx |
| 24 | [Rekon Brg Masuk](#24-rekon-brg-masuk) | 11 | 7 slicer, 2 tableEx, 1 shape, 1 actionButton |
| 25 | [Sales Growth](#25-sales-growth) | 8 | 5 slicer, 1 shape, 1 actionButton, 1 pivotTable |
| 26 | [Rekap PO blm PI](#26-rekap-po-blm-pi) | 14 | 9 slicer, 3 tableEx, 1 shape, 1 actionButton |
| 27 | [Detail PO blm PI](#27-detail-po-blm-pi) | 13 | 9 slicer, 1 shape, 1 textFilter25A4896A83E0487089E2B90C9AE57C8A, 1 actionButton, 1 tableEx |
| 28 | [Detail PI](#28-detail-pi) | 12 | 8 slicer, 1 shape, 1 textFilter25A4896A83E0487089E2B90C9AE57C8A, 1 actionButton, 1 tableEx |
| 29 | [End](#29-end) | 1 | 1 ScrollingTextVisual1448795304508 |
| 30 | [Data Deviasi *(hidden)*](#30-data-deviasi) | 8 | 5 slicer, 1 shape, 1 actionButton, 1 pivotTable |
| 31 | [Data Waste *(hidden)*](#31-data-waste) | 8 | 4 slicer, 2 pivotTable, 1 shape, 1 actionButton |
| 32 | [Data Sales *(hidden)*](#32-data-sales) | 7 | 4 slicer, 1 shape, 1 actionButton, 1 pivotTable |
| 33 | [Tooltip *(hidden)*](#33-tooltip) | 3 | 3 tableEx |
| 34 | [ID Dokumen *(hidden)*](#34-id-dokumen) | 6 | 4 tableEx, 1 shape, 1 actionButton |
| 35 | [Barang Baru *(hidden)*](#35-barang-baru) | 11 | 5 tableEx, 4 slicer, 1 shape, 1 actionButton |
| 36 | [Kosongan *(hidden)*](#36-kosongan) | 13 | 11 slicer, 1 shape, 1 actionButton |

## 1. Home (10 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Deviasi | — | — |
| 2 | shape | Barang Masuk & Sales | — | — |
| 3 | cardVisual |  | **Data**: Update[Last Update BM] | — |
| 4 | cardVisual |  | **Data**: Update[Last Update Dev] | — |
| 5 | cardVisual |  | **Data**: Update[Update on Dev] | — |
| 6 | cardVisual |  | **Data**: Update[Update on BM] | — |
| 7 | shape | Cost Control | — | — |
| 8 | shape | Laporan Deviasi | — | — |
| 9 | emptyContainer (placeholder) |  | — | — |
| 10 | emptyContainer (placeholder) |  | — | — |

## 2. Summary Deviasi (48 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Summary Deviasi | — | — |
| 2 | shape | Turun | — | — |
| 3 | shape | Loss | — | — |
| 4 | shape | Loss | — | — |
| 5 | shape | Turun | — | — |
| 6 | shape | Naik | — | — |
| 7 | shape | Surplus | — | — |
| 8 | shape | Naik | — | — |
| 9 | shape | Surplus | — | — |
| 10 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 11 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 12 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 13 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 14 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 15 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 16 | shape | Loss | — | — |
| 17 | shape | Loss | — | — |
| 18 | shape | Loss | — | — |
| 19 | shape | Loss | — | — |
| 20 | shape | Surplus | — | — |
| 21 | shape | Surplus | — | — |
| 22 | shape | Surplus | — | — |
| 23 | shape | Surplus | — | — |
| 24 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 25 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 26 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 27 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 28 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 29 | actionButton | Reset | — | — |
| 30 | barChart | Overview | **Y**: Deviasi All[ABS Deviasi] | — |
| 31 | barChart | Top 5 Resto Deviasi Tertinggi | **Category**: Resto[Resto]<br>**Y**: Deviasi All[ABS Deviasi] | `TOP 5 by Deviasi All[ABS Deviasi] DESC` |
| 32 | barChart | Top 5 Item Deviasi Tertinggi | **Category**: Inventory[Nama Barang]<br>**Y**: Deviasi All[ABS Deviasi] | `TOP 5 by Deviasi All[ABS Deviasi] DESC` |
| 33 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Deviasi CM] | `SUM(Deviasi All[Value])` |
| 34 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Growth] | `SUM(Deviasi All[Value])` |
| 35 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Growth YoY] | `SUM(Deviasi All[Value])` |
| 36 | emptyContainer (placeholder) |  | — | — |
| 37 | emptyContainer (placeholder) |  | — | — |
| 38 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[BOM CM] | `SUM(Deviasi All[Value])` |
| 39 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[% BOM CM] | `SUM(Deviasi All[Value])` |
| 40 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[% BOM CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |
| 41 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Sales All[Sales CM] | `SUM(Deviasi All[Value])` |
| 42 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Sales All[% Sales CM] | `SUM(Deviasi All[Value])` |
| 43 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Sales All[% Sales CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |
| 44 | waterfallChart | Kontribusi | **Category**: Deviasi All[Jenis]<br>**Y**: Deviasi All[Deviasi CM] | `Deviasi All[Deviasi CM]: NOT(Deviasi CM = 0)` |
| 45 | barChart | Top 5 Resto Naik/Turun Deviasi | **Category**: Resto[Resto]<br>**Y**: Deviasi All[Selisih Abs] | `TOP 5 by Deviasi All[Selisih Abs] DESC`<br>`Deviasi All[Selisih Abs]` |
| 46 | barChart | Top 5 Item Naik/Turun Deviasi | **Category**: Inventory[Nama Barang]<br>**Y**: Deviasi All[Selisih Abs] | `TOP 5 by Deviasi All[Selisih Abs] DESC` |
| 47 | emptyContainer (placeholder) |  | — | — |
| 48 | emptyContainer (placeholder) |  | — | — |

## 3. Detail Deviasi (18 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Detail Deviasi | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable | Penyebab Deviasi (antar periode) | **Rows**: Inventory[Tipe]<br>**Rows**: Deviasi All[Jenis]<br>**Values**: Deviasi All[Week 1]<br>**Values**: Deviasi All[Week 2]<br>**Values**: Deviasi All[Growth W2]<br>**Values**: Deviasi All[Week 3]<br>**Values**: Deviasi All[Growth W3]<br>**Values**: Deviasi All[EoM]<br>**Values**: Deviasi All[Growth EOM] | `Deviasi All[Show Detail]: Show Detail = 'SHOW'` |
| 15 | pivotTable | Kontribusi Deviasi (antar periode) | **Columns**: Periode[Periode]<br>**Rows**: Inventory[Tipe]<br>**Rows**: Deviasi All[Jenis]<br>**Values**: Deviasi All[Kontribusi CM] | `Deviasi All[Show Detail]: Show Detail = 'SHOW'` |
| 16 | pivotTable | Penyebab & Kontribusi Deviasi (antar bulan) | **Rows**: Inventory[Tipe]<br>**Rows**: Deviasi All[Jenis]<br>**Values**: Deviasi All[Deviasi LM]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[Selisih]<br>**Values**: Deviasi All[Growth]<br>**Values**: Deviasi All[Kontribusi LM]<br>**Values**: Deviasi All[Kontribusi CM] | `Deviasi All[Deviasi YTD]`<br>`Deviasi All[Deviasi CM]`<br>`Deviasi All[Show Detail]: Show Detail = 'SHOW'` |
| 17 | donutChart | Deviasi Raw Material | **Category**: Deviasi All[Jenis]<br>**Y**: Deviasi All[Kontribusi CM] | `SUM(Deviasi All[Value])`<br>`Inventory[Tipe]: IN ('COM DEVIASI')` |
| 18 | donutChart | Deviasi Packaging | **Category**: Deviasi All[Jenis]<br>**Y**: Deviasi All[Kontribusi CM] | `SUM(Deviasi All[Value])`<br>`Inventory[Tipe]: IN ('PACKAGING DEVIASI')` |

## 4. Deviasi by Area (16 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Deviasi by Area | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable |  | **Rows**: Resto[Area]<br>**Rows**: Resto[AM]<br>**Rows**: Resto[Resto]<br>**Values**: DISTINCTCOUNT(Deviasi All[Resto])<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[Deviasi LM]<br>**Values**: Deviasi All[Analisa Deviasi]<br>**Values**: Deviasi All[Selisih]<br>**Values**: Deviasi All[Growth]<br>**Values**: Deviasi All[Perubahan]<br>**Values**: Deviasi All[Deviasi YTD]<br>**Values**: Deviasi All[Deviasi YoY]<br>**Values**: Deviasi All[Growth YoY]<br>**Values**: SUM(Sales All[Penjualan Bersih])<br>**Values**: Sales All[% Sales CM]<br>**Values**: Sales All[Sales LM]<br>**Values**: Sales All[% Sales LM]<br>**Values**: Sales All[Sales GAP]<br>**Values**: Sales All[% Sales Growth]<br>**Values**: Sales All[% Sales GAP]<br>**Values**: Sales All[Sales YTD]<br>**Values**: Sales All[% Sales YTD] | `Sales All[% Sales CM]`<br>`Sales All[% Sales LM]`<br>`Sales All[% Sales YTD]`<br>`Deviasi All[ABS Deviasi]`<br>`Resto[Area 2]`<br>`Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Deviasi YoY]`<br>`Deviasi All[Deviasi YTD]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Growth YoY]`<br>`Resto[Restoo]`<br>`DISTINCTCOUNT(Deviasi All[Resto])`<br>`Deviasi All[Resto YoY]`<br>`SUM(Sales All[Penjualan Bersih])`<br>`Sales All[Sales LM]`<br>`Sales All[Sales YTD]`<br>`Sales All[% Sales GAP]`<br>`Deviasi All[Selisih]`<br>`Resto[Team]`<br>`Deviasi All[Analisa Deviasi]`<br>`Deviasi All[Perubahan]`<br>`Sales All[Sales GAP]`<br>`Sales All[% Sales Growth]`<br>`Deviasi All[Show Value]: NOT(Show Value = null)` |
| 15 | slicer ⭃sync:Kategori | Slicer Trend[Kategori] | **Values**: Trend[Kategori] | — |
| 16 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Tooltips**: Deviasi All[Multi Value LM]<br>**Tooltips**: Deviasi All[Multi Value Gap]<br>**Tooltips**: Deviasi All[Multi Value Growth]<br>**Tooltips**: Deviasi All[Perubahan Multi Value]<br>**Y**: Deviasi All[Multi Value CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 5. Deviasi by PIC (16 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Deviasi by PIC | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | slicer ⭃sync:Date | Slicer Date[Date] | **Values**: Date[Date] | — |
| 14 | actionButton | Reset | — | — |
| 15 | pivotTable |  | **Rows**: Resto[PIC]<br>**Rows**: Resto[Resto]<br>**Values**: DISTINCTCOUNT(Deviasi All[Resto])<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[Deviasi LM]<br>**Values**: Deviasi All[Analisa Deviasi]<br>**Values**: Deviasi All[Selisih]<br>**Values**: Deviasi All[Growth]<br>**Values**: Deviasi All[Perubahan]<br>**Values**: Deviasi All[Deviasi YTD]<br>**Values**: Deviasi All[Deviasi YoY]<br>**Values**: Deviasi All[Growth YoY]<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[% BOM CM]<br>**Values**: Deviasi All[BOM LM]<br>**Values**: Deviasi All[% BOM LM]<br>**Values**: Deviasi All[BOM GAP]<br>**Values**: Deviasi All[% BOM GAP]<br>**Values**: Deviasi All[BOM YTD]<br>**Values**: Deviasi All[% BOM YTD] | `Deviasi All[Growth]`<br>`Deviasi All[Show Value]: NOT(Show Value = null)` |
| 16 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Y**: Deviasi All[BOM ABS]<br>**Y**: Deviasi All[ABS Deviasi]<br>**Y2**: Deviasi All[% BOM CM Abs] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 6. Deviasi by Resto (18 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Deviasi by Resto | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | slicer ⭃sync:Date | Slicer Date[Date] | **Values**: Date[Date] | — |
| 14 | actionButton | Reset | — | — |
| 15 | pivotTable |  | **Rows**: Resto[Resto]<br>**Rows**: Inventory[Nama Barang]<br>**Values**: Deviasi All[% Dev Contribution]<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[Deviasi LM]<br>**Values**: Deviasi All[Analisa Deviasi]<br>**Values**: Deviasi All[Selisih]<br>**Values**: Deviasi All[Growth]<br>**Values**: Deviasi All[Perubahan]<br>**Values**: Deviasi All[Deviasi YTD]<br>**Values**: Deviasi All[Deviasi YoY]<br>**Values**: Deviasi All[Growth YoY]<br>**Values**: SUM(Sales All[Penjualan Bersih])<br>**Values**: Sales All[% Sales CM]<br>**Values**: Sales All[Sales LM]<br>**Values**: Sales All[% Sales LM]<br>**Values**: Sales All[Sales GAP]<br>**Values**: Sales All[% Sales Growth]<br>**Values**: Sales All[% Sales GAP]<br>**Values**: Sales All[Sales YTD]<br>**Values**: Sales All[% Sales YTD] | `Sales All[% Sales LM]`<br>`Sales All[% Sales YTD]`<br>`Deviasi All[ABS Deviasi]`<br>`Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Resto[Restoo]`<br>`SUM(Sales All[Penjualan Bersih])`<br>`Sales All[Sales YTD]`<br>`Sales All[% Sales GAP]`<br>`Inventory[(Old) Nama Barang]`<br>`Deviasi All[% Dev Contribution]`<br>`Deviasi All[Perubahan]`<br>`Deviasi All[Analisa Deviasi]`<br>`Sales All[Sales GAP]`<br>`Inventory[Nama Barang]`<br>`Deviasi All[Selisih]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Deviasi YTD]`<br>`Deviasi All[Deviasi YoY]`<br>`Deviasi All[Growth YoY]`<br>`Sales All[% Sales CM]`<br>`Sales All[Sales LM]`<br>`Sales All[% Sales Growth]`<br>`Deviasi All[Show Value]: NOT(Show Value = null)` |
| 16 | slicer ⭃sync:Kategori | Slicer Trend[Kategori] | **Values**: Trend[Kategori] | — |
| 17 | shape | Untuk melihat perbandingan trend, pilih minimal 2 resto | — | — |
| 18 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Series**: Resto[Resto]<br>**Y**: Deviasi All[Multi Value CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]` |

## 7. Deviasi by Item (18 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Deviasi by Item | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | slicer ⭃sync:Date | Slicer Date[Date] | **Values**: Date[Date] | — |
| 14 | actionButton | Reset | — | — |
| 15 | pivotTable |  | **Rows**: Inventory[Nama Barang]<br>**Values**: Deviasi All[% Dev Contribution]<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[Deviasi LM]<br>**Values**: Deviasi All[Analisa Deviasi]<br>**Values**: Deviasi All[Rank Dev]<br>**Values**: Deviasi All[Rank BOM]<br>**Values**: Deviasi All[Selisih]<br>**Values**: Deviasi All[Growth]<br>**Values**: Deviasi All[Perubahan]<br>**Values**: Deviasi All[Deviasi YTD]<br>**Values**: Deviasi All[Deviasi YoY]<br>**Values**: Deviasi All[Growth YoY]<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[% BOM CM]<br>**Values**: Deviasi All[BOM LM]<br>**Values**: Deviasi All[% BOM LM]<br>**Values**: Deviasi All[BOM GAP]<br>**Values**: Deviasi All[% BOM GAP]<br>**Values**: Deviasi All[BOM YTD]<br>**Values**: Deviasi All[% BOM YTD] | `Deviasi All[Growth]`<br>`Deviasi All[Show Value]: NOT(Show Value = null)` |
| 16 | slicer ⭃sync:Kategori | Slicer Trend[Kategori] | **Values**: Trend[Kategori] | — |
| 17 | shape | Untuk melihat perbandingan trend, pilih minimal 2 item | — | — |
| 18 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Series**: Inventory[Nama Barang]<br>**Y**: Deviasi All[Multi Value CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]` |

## 8. Antar Periode (14 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Antar Periode | — | — |
| 2 | slicer | Slicer Date A[Period] | **Values**: Date A[Period] | `Date A[Date]: Date > ` |
| 3 | slicer | Slicer Date B[Period] | **Values**: Date B[Period] | `Date B[Date]: Date > `<br>`Date B[Period]` |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable |  | **Rows**: Inventory[Nama Barang]<br>**Rows**: Resto[Resto]<br>**Values**: Deviasi All[% Dev Contribution]<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[Deviasi LM]<br>**Values**: Deviasi All[Analisa Deviasi]<br>**Values**: Deviasi All[Selisih]<br>**Values**: Deviasi All[Growth]<br>**Values**: Deviasi All[Perubahan]<br>**Values**: Deviasi All[Deviasi YTD]<br>**Values**: Deviasi All[Deviasi YoY]<br>**Values**: Deviasi All[Growth YoY]<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[% BOM CM]<br>**Values**: Deviasi All[BOM LM]<br>**Values**: Deviasi All[% BOM LM]<br>**Values**: Deviasi All[BOM GAP]<br>**Values**: Deviasi All[% BOM GAP]<br>**Values**: Deviasi All[BOM YTD]<br>**Values**: Deviasi All[% BOM YTD] | `Deviasi All[Growth]`<br>`Deviasi All[Show]: Show = 'Show'` |

## 9. Kontribusi Dev (15 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Kontribusi Dev | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 7 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 8 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 9 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 10 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 11 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 12 | actionButton | Reset | — | — |
| 13 | pivotTable |  | **Rows**: Inventory[Nama Barang]<br>**Values**: Deviasi All[Waste]<br>**Values**: Deviasi All[% Waste]<br>**Values**: Deviasi All[Susut]<br>**Values**: Deviasi All[% Susut]<br>**Values**: Deviasi All[Trial]<br>**Values**: Deviasi All[% Trial]<br>**Values**: Deviasi All[Loss/Surplus]<br>**Values**: Deviasi All[% Loss/Suprlus]<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[% Dev Contribution] | `Deviasi All[Growth]` |
| 14 | pivotTable |  | **Rows**: Resto[Resto]<br>**Values**: Deviasi All[Waste]<br>**Values**: Deviasi All[% Waste]<br>**Values**: Deviasi All[Susut]<br>**Values**: Deviasi All[% Susut]<br>**Values**: Deviasi All[Trial]<br>**Values**: Deviasi All[% Trial]<br>**Values**: Deviasi All[Loss/Surplus]<br>**Values**: Deviasi All[% Loss/Suprlus]<br>**Values**: Deviasi All[ABS Deviasi]<br>**Values**: Deviasi All[% Dev Contribution] | `Deviasi All[Growth]` |
| 15 | donutChart | Kontribusi | **Y**: Deviasi All[% Waste]<br>**Y**: Deviasi All[% Susut]<br>**Y**: Deviasi All[% Trial]<br>**Y**: Deviasi All[% Loss/Suprlus] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 10. Perbandingan (12 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Perbandingan | — | — |
| 2 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 3 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 4 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 5 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 6 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 7 | slicer | Slicer Inventory A[(Old) Nama Barang] | **Values**: Inventory A[(Old) Nama Barang] | `Inventory[Tipe]: NOT(Tipe = null)` |
| 8 | slicer | Slicer Resto A[Resto] | **Values**: Resto A[Resto] | `Inventory[Tipe]: NOT(Tipe = null)` |
| 9 | slicer | Slicer Inventory B[(Old) Nama Barang] | **Values**: Inventory B[(Old) Nama Barang] | `Inventory[Tipe]: NOT(Tipe = null)` |
| 10 | slicer | Slicer Resto B[Resto] | **Values**: Resto B[Resto] | `Inventory[Tipe]: NOT(Tipe = null)` |
| 11 | actionButton | Reset | — | — |
| 12 | pivotTable |  | **Rows**: Date[Period]<br>**Values**: Inventory A[A Deviasi]<br>**Values**: Inventory A[A BOM]<br>**Values**: Inventory A[A % BOM]<br>**Values**: Inventory A[A Sales]<br>**Values**: Inventory A[A % Sales]<br>**Values**: Inventory A[B Deviasi]<br>**Values**: Inventory A[B BOM]<br>**Values**: Inventory A[B % BOM]<br>**Values**: Inventory A[B Sales]<br>**Values**: Inventory A[B % Sales]<br>**Values**: Inventory A[GAP Deviasi]<br>**Values**: Inventory A[GAP ABS Dev]<br>**Values**: Inventory A[GAP BOM]<br>**Values**: Inventory A[GAP % BOM]<br>**Values**: Inventory A[GAP Sales]<br>**Values**: Inventory A[GAP % Sales] | `Deviasi All[Growth]` |

## 11. Deviasi vs BOM (15 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Deviasi vs BOM | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable |  | **Rows**: Resto[Resto]<br>**Rows**: Inventory[Nama Barang]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[% BOM CM]<br>**Values**: Deviasi All[% BOM CM Abs] | — |
| 15 | pivotTable |  | **Rows**: Inventory[Nama Barang]<br>**Rows**: Deviasi All[Jenis]<br>**Values**: Deviasi All[Deviasi CM]<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[% BOM CM]<br>**Values**: Deviasi All[% BOM CM Abs] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |

## 12. Growth by Resto (15 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Growth by Resto | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 11 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 12 | actionButton | Reset | — | — |
| 13 | pivotTable |  | **Rows**: Resto[Resto]<br>**Rows**: Inventory[Nama Barang]<br>**Values**: Deviasi All[Week 1]<br>**Values**: Deviasi All[Week 2]<br>**Values**: Deviasi All[Growth W2]<br>**Values**: Deviasi All[Perubahan W2]<br>**Values**: Deviasi All[Week 3]<br>**Values**: Deviasi All[Growth W3]<br>**Values**: Deviasi All[Perubahan W3]<br>**Values**: Deviasi All[EoM]<br>**Values**: Deviasi All[Growth EOM]<br>**Values**: Deviasi All[Perubahan EOM]<br>**Values**: Deviasi All[Week 1 Abs]<br>**Values**: Deviasi All[Week 2 Abs]<br>**Values**: Deviasi All[Week 3 Abs]<br>**Values**: Deviasi All[EoM Abs] | `Deviasi All[Growth]` |
| 14 | slicer ⭃sync:Kategori | Slicer Trend[Kategori] | **Values**: Trend[Kategori] | — |
| 15 | lineClusteredColumnComboChart | Trend | **Category**: Periode[Periode]<br>**Tooltips**: Deviasi All[Multi Value LM]<br>**Tooltips**: Deviasi All[Multi Value Gap]<br>**Tooltips**: Deviasi All[Multi Value Growth]<br>**Y**: Deviasi All[Multi Value CM] | `Date[Date]`<br>`Periode[Periode]: NOT(IN (null))` |

## 13. Growth by Item (15 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Growth by Item | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 11 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 12 | slicer ⭃sync:Date | Slicer Date[Date] | **Values**: Date[Date] | — |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable |  | **Rows**: Inventory[Nama Barang]<br>**Rows**: Resto[Resto]<br>**Values**: Deviasi All[Week 1]<br>**Values**: Deviasi All[Week 2]<br>**Values**: Deviasi All[Growth W2]<br>**Values**: Deviasi All[Perubahan W2]<br>**Values**: Deviasi All[Week 3]<br>**Values**: Deviasi All[Growth W3]<br>**Values**: Deviasi All[Perubahan W3]<br>**Values**: Deviasi All[EoM]<br>**Values**: Deviasi All[Growth EOM]<br>**Values**: Deviasi All[Perubahan EOM]<br>**Values**: Deviasi All[Week 1 Abs]<br>**Values**: Deviasi All[Week 2 Abs]<br>**Values**: Deviasi All[Week 3 Abs]<br>**Values**: Deviasi All[EoM Abs] | `Deviasi All[Growth]` |
| 15 | lineClusteredColumnComboChart | Trend | **Category**: Periode[Periode]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Y**: Deviasi All[BOM ABS]<br>**Y**: Deviasi All[ABS Deviasi]<br>**Y2**: Deviasi All[% BOM CM Abs] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 14. Analisa Susut (12 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Analisa Susut | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 7 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 8 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 9 | actionButton | Reset | — | — |
| 10 | pivotTable | AYAM CINCANG (V.20) | **Rows**: Resto[Type Kitchen]<br>**Rows**: Resto[Area]<br>**Rows**: Resto[Resto]<br>**Values**: DISTINCTCOUNT(Deviasi All[Resto])<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[Susut]<br>**Values**: Deviasi All[% BOM CM Abs]<br>**Values**: AVG(Standar Susut[Standar Susut])<br>**Values**: Deviasi All[Analisa Susut] | `Deviasi All[% BOM CM]: NOT(% BOM CM = null)`<br>`Deviasi All[Analisa Susut]`<br>`Resto[Area 2]`<br>`Deviasi All[BOM CM]`<br>`Inventory[Nama Barang]: IN ('AYAM CINCANG (V.20)')`<br>`Deviasi All[Growth]`<br>`Resto[Restoo]`<br>`DISTINCTCOUNT(Deviasi All[Resto])`<br>`AVG(Standar Susut[Standar Susut])`<br>`Deviasi All[Susut]`<br>`Resto[Type Kitchen]: NOT(IN (null, '#N/A', ''))` |
| 11 | pivotTable | CABE RAWIT - RESTO (V.20) & CABAI FROZEN | **Rows**: Resto[Type Kitchen]<br>**Rows**: Resto[Area]<br>**Rows**: Resto[Resto]<br>**Values**: DISTINCTCOUNT(Deviasi All[Resto])<br>**Values**: Deviasi All[BOM CM]<br>**Values**: Deviasi All[Susut]<br>**Values**: Deviasi All[% BOM CM Abs]<br>**Values**: AVG(Standar Susut[Standar Susut])<br>**Values**: Deviasi All[Analisa Susut] | `Deviasi All[Growth]`<br>`Resto[Type Kitchen]: NOT(IN (null, '#N/A', ''))`<br>`Deviasi All[% BOM CM]: NOT(% BOM CM = null)`<br>`Inventory[Nama Barang]: IN ('CABE RAWIT - RESTO (V.20)')`<br>`Resto[Area]`<br>`Resto[Resto]`<br>`DISTINCTCOUNT(Deviasi All[Resto])`<br>`Deviasi All[BOM CM]`<br>`Deviasi All[Susut]`<br>`Deviasi All[% BOM CM Abs]`<br>`AVG(Standar Susut[Standar Susut])`<br>`Deviasi All[Analisa Susut]` |
| 12 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Series**: Inventory[Nama Barang]<br>**Y**: Deviasi All[% BOM CM Abs] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 15. Last 13 Months (13 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Last 13 Months | — | — |
| 2 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 3 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 4 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 5 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 6 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 7 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 8 | slicer ⭃sync:Kategori | Slicer Trend[Kategori] | **Values**: Trend[Kategori] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 11 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 12 | actionButton | Reset | — | — |
| 13 | pivotTable |  | **Columns**: Date[Period]<br>**Rows**: Inventory[Nama Barang]<br>**Rows**: Resto[Resto]<br>**Values**: Deviasi All[Multi Value CM] | — |

## 16. Top 10 Item Dev (31 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Top 10 Item Dev | — | — |
| 2 | shape | Loss | — | — |
| 3 | shape | Loss | — | — |
| 4 | shape | Surplus | — | — |
| 5 | shape | Surplus | — | — |
| 6 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 7 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 8 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 9 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 10 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 11 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 12 | shape | Loss | — | — |
| 13 | shape | Loss | — | — |
| 14 | shape | Surplus | — | — |
| 15 | shape | Surplus | — | — |
| 16 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 17 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 18 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 19 | actionButton | Reset | — | — |
| 20 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Nominal Dev] | `SUM(Deviasi All[Value])`<br>`Deviasi All[Jenis]: IN ('WASTE')` |
| 21 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Nominal Dev] | `SUM(Deviasi All[Value])`<br>`Deviasi All[Jenis]: IN ('SUSUT')` |
| 22 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Nominal Dev] | `SUM(Deviasi All[Value])`<br>`Deviasi All[Jenis]: IN ('TRIAL')` |
| 23 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Top10Dev] | `SUM(Deviasi All[Value])`<br>`Deviasi All[Jenis]: IN ('LOSS/SURPLUS')`<br>`Attribute[Attribute]: IN ('NOMINAL')` |
| 24 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Top10Dev] | `SUM(Deviasi All[Value])`<br>`Attribute[Attribute]: IN ('NOMINAL')` |
| 25 | barChart | Top 10 Waste | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[% GT Deviasi]<br>**Tooltips**: Deviasi All[BOM CM]<br>**Tooltips**: Deviasi All[% BOM CM]<br>**Y**: Deviasi All[Nominal Dev] | `Deviasi All[Jenis]: IN ('WASTE')`<br>`Deviasi All[Nominal Dev]: NOT(Nominal Dev = 0)`<br>`TOP 10 by Deviasi All[Nominal Dev] DESC` |
| 26 | barChart | Top 10 Susut | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[% GT Deviasi]<br>**Tooltips**: Deviasi All[BOM CM]<br>**Tooltips**: Deviasi All[% BOM CM]<br>**Y**: Deviasi All[Nominal Dev] | `Deviasi All[Jenis]: IN ('SUSUT')`<br>`Deviasi All[Nominal Dev]: NOT(Nominal Dev = 0)`<br>`TOP 10 by Deviasi All[Nominal Dev] DESC` |
| 27 | barChart | Top 10 Trial | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[% GT Deviasi]<br>**Tooltips**: Deviasi All[BOM CM]<br>**Tooltips**: Deviasi All[% BOM CM]<br>**Y**: Deviasi All[Nominal Dev] | `Deviasi All[Jenis]: IN ('TRIAL')`<br>`Deviasi All[Nominal Dev]: NOT(Nominal Dev = 0)`<br>`TOP 10 by Deviasi All[Nominal Dev] DESC` |
| 28 | barChart | Top 10 Loss/Surplus | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[% GT Deviasi]<br>**Tooltips**: Deviasi All[BOM CM]<br>**Tooltips**: Deviasi All[% BOM CM]<br>**Y**: Deviasi All[ABS Deviasi] | `Deviasi All[Jenis]: IN ('LOSS/SURPLUS')`<br>`Deviasi All[Nominal Dev]: NOT(Nominal Dev = 0)`<br>`Attribute[Attribute]: IN ('NOMINAL')`<br>`TOP 10 by Deviasi All[ABS Deviasi] DESC` |
| 29 | barChart | Top 10 Deviasi | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[% GT Deviasi]<br>**Tooltips**: Deviasi All[BOM CM]<br>**Tooltips**: Deviasi All[% BOM CM]<br>**Y**: Deviasi All[ABS Deviasi] | `Deviasi All[Nominal Dev]: NOT(Nominal Dev = 0)`<br>`Attribute[Attribute]: IN ('NOMINAL')`<br>`TOP 10 by Deviasi All[ABS Deviasi] DESC` |
| 30 | emptyContainer (placeholder) |  | — | — |
| 31 | emptyContainer (placeholder) |  | — | — |

## 17. Top 5 Growth (29 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Top 5 Growth | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 10 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 11 | slicer | Slicer Deviasi All[Group Satuan] | **Values**: Deviasi All[Group Satuan] | `Inventory[Tipe]` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | barChart | Kenaikan Deviasi Terbesar | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Tooltips**: Deviasi All[Deviasi LM]<br>**Tooltips**: Deviasi All[Growth]<br>**Tooltips**: Deviasi All[Perubahan]<br>**Y**: Deviasi All[Selisih] | `Deviasi All[Selisih]`<br>`Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Selisih]: NOT(Selisih = 0)`<br>`TOP 5 by Deviasi All[Selisih] DESC`<br>`Deviasi All[Perubahan]` |
| 15 | barChart | Growth Deviasi Tertinggi (%) | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Tooltips**: Deviasi All[Deviasi LM]<br>**Tooltips**: Deviasi All[Selisih]<br>**Tooltips**: Deviasi All[Perubahan]<br>**Y**: Deviasi All[Growth] | `Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Selisih]: NOT(Selisih = 0)`<br>`TOP 5 by Deviasi All[Growth] DESC`<br>`Deviasi All[Perubahan]` |
| 16 | barChart | Deviasi Semakin Memburuk | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Tooltips**: Deviasi All[Deviasi LM]<br>**Tooltips**: Deviasi All[Selisih]<br>**Tooltips**: Deviasi All[Growth]<br>**Y**: Deviasi All[Perubahan] | `Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Selisih]: NOT(Selisih = 0)`<br>`TOP 5 by Deviasi All[Perubahan] DESC`<br>`Deviasi All[Perubahan]` |
| 17 | barChart | Growth Sales Tertinggi | **Category**: Resto[Resto]<br>**Tooltips**: Sales All[Sales CM]<br>**Tooltips**: Sales All[Sales LM]<br>**Tooltips**: Sales All[Sales GAP]<br>**Y**: Sales All[% Sales Growth] | `Attribute[Attribute]: IN ('NOMINAL')`<br>`TOP 5 by Sales All[% Sales Growth] DESC` |
| 18 | textbox |  | — | — |
| 19 | textbox |  | — | — |
| 20 | textbox |  | — | — |
| 21 | textbox |  | — | — |
| 22 | barChart | Penurunan Deviasi Terbesar | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Tooltips**: Deviasi All[Deviasi LM]<br>**Tooltips**: Deviasi All[Growth]<br>**Tooltips**: Deviasi All[Perubahan]<br>**Y**: Deviasi All[Selisih] | `Deviasi All[Selisih]`<br>`Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Selisih]: NOT(Selisih = 0)`<br>`TOP 5 by Deviasi All[Selisih] ASC`<br>`Deviasi All[Perubahan]` |
| 23 | barChart | Growth Deviasi Terendah (%) | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Tooltips**: Deviasi All[Deviasi LM]<br>**Tooltips**: Deviasi All[Selisih]<br>**Tooltips**: Deviasi All[Perubahan]<br>**Y**: Deviasi All[Growth] | `Deviasi All[Selisih]: NOT(Selisih = 0)`<br>`TOP 5 by Deviasi All[Growth] ASC` |
| 24 | barChart | Deviasi Paling Membaik | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Deviasi All[Deviasi CM]<br>**Tooltips**: Deviasi All[Deviasi LM]<br>**Tooltips**: Deviasi All[Selisih]<br>**Tooltips**: Deviasi All[Growth]<br>**Y**: Deviasi All[Perubahan] | `Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Selisih]: NOT(Selisih = 0)`<br>`TOP 5 by Deviasi All[Perubahan] ASC`<br>`Deviasi All[Perubahan]` |
| 25 | barChart | Growth Sales Terendah | **Category**: Resto[Resto]<br>**Tooltips**: Sales All[Sales CM]<br>**Tooltips**: Sales All[Sales LM]<br>**Tooltips**: Sales All[Sales GAP]<br>**Y**: Sales All[% Sales Growth] | `Attribute[Attribute]: IN ('NOMINAL')`<br>`TOP 5 by Sales All[% Sales Growth] ASC` |
| 26 | textbox |  | — | — |
| 27 | textbox |  | — | — |
| 28 | textbox |  | — | — |
| 29 | textbox |  | — | — |

## 18. Top 5 Waste (18 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Top 5 Waste | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 9 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 10 | actionButton | Reset | — | — |
| 11 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Deviasi All[Nominal Dev] | `SUM(Deviasi All[Value])`<br>`Deviasi All[Jenis]: IN ('WASTE')` |
| 12 | barChart | Top 5 Alat/Mesin | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Waste All[% to Subtotal]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: Waste All[Nominal Waste] | `Waste All[Jenis]: IN ('Alat/Mesin')`<br>`Waste All[Nominal Waste]: NOT(Nominal Waste = 0)`<br>`TOP 5 by Waste All[Nominal Waste] DESC` |
| 13 | barChart | Top 5 Cancel Nota | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Waste All[% to Subtotal]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: Waste All[Nominal Waste] | `Waste All[Jenis]: IN ('Cancel Nota')`<br>`Waste All[Nominal Waste]: NOT(Nominal Waste = 0)`<br>`TOP 5 by Waste All[Nominal Waste] DESC` |
| 14 | barChart | Top 5 Handling | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Waste All[% to Subtotal]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: Waste All[Nominal Waste] | `Waste All[Jenis]: IN ('Handling')`<br>`Waste All[Nominal Waste]: NOT(Nominal Waste = 0)`<br>`TOP 5 by Waste All[Nominal Waste] DESC` |
| 15 | barChart | Penyebab Waste | **Category**: Waste All[Jenis]<br>**Y**: Waste All[Nominal Waste] | `TopN (kondisi kosong)` |
| 16 | barChart | Top 5 Kualitas Eksternal | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Waste All[% to Subtotal]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: Waste All[Nominal Waste] | `Waste All[Jenis]: IN ('Kualitas Eksternal')`<br>`Waste All[Nominal Waste]: NOT(Nominal Waste = 0)`<br>`TOP 5 by Waste All[Nominal Waste] DESC` |
| 17 | barChart | Top 5 Kualitas Internal | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Waste All[% to Subtotal]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: Waste All[Nominal Waste] | `Waste All[Jenis]: IN ('Kualitas Internal')`<br>`Waste All[Nominal Waste]: NOT(Nominal Waste = 0)`<br>`TOP 5 by Waste All[Nominal Waste] DESC` |
| 18 | barChart | Top 5 Over Stock | **Category**: Inventory[Nama Barang]<br>**Tooltips**: Waste All[% to Subtotal]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: Waste All[Nominal Waste] | `Waste All[Jenis]: IN ('Over Stock')`<br>`Waste All[Nominal Waste]: NOT(Nominal Waste = 0)`<br>`TOP 5 by Waste All[Nominal Waste] DESC` |

## 19. Waste by Resto (16 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Waste by Resto | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable |  | **Rows**: Resto[Resto]<br>**Rows**: Inventory[Nama Barang]<br>**Rows**: Waste All[Jenis]<br>**Values**: SUM(Waste All[Value])<br>**Values**: % CM<br>**Values**: Waste All[Waste LM]<br>**Values**: % LM<br>**Values**: Waste All[Waste Gap]<br>**Values**: Waste All[Waste Growth]<br>**Values**: Waste All[Waste YTD]<br>**Values**: % YTD | — |
| 15 | donutChart | Kontribusi Penyebab Waste | **Category**: Waste All[Jenis]<br>**Y**: SUM(Waste All[Value]) | `SUM(Deviasi All[Value])` |
| 16 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Y**: SUM(Waste All[Value]) | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 20. COM (15 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | COM | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 7 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 11 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 12 | actionButton | Reset | — | — |
| 13 | pivotTable |  | **Rows**: Inventory[Nama Barang]<br>**Rows**: Resto[Resto]<br>**Values**: Deviasi All[COM LM]<br>**Values**: %<br>**Values**: Deviasi All[COM CM]<br>**Values**: %1<br>**Values**: Deviasi All[COM Gap]<br>**Values**: Deviasi All[COM Growth] | `Sales All[% Sales CM]`<br>`Sales All[% Sales LM]`<br>`Sales All[% Sales YTD]`<br>`Deviasi All[ABS Deviasi]`<br>`Resto[Area 2]`<br>`Deviasi All[Deviasi CM]`<br>`Deviasi All[Deviasi LM]`<br>`Deviasi All[Deviasi YoY]`<br>`Deviasi All[Deviasi YTD]`<br>`Deviasi All[Growth]`<br>`Deviasi All[Growth YoY]`<br>`Resto[Restoo]`<br>`DISTINCTCOUNT(Deviasi All[Resto])`<br>`Deviasi All[Resto YoY]`<br>`SUM(Sales All[Penjualan Bersih])`<br>`Sales All[Sales LM]`<br>`Sales All[Sales YTD]`<br>`Sales All[% Sales GAP]`<br>`Deviasi All[Selisih]`<br>`Resto[Team]`<br>`Deviasi All[Analisa Deviasi]`<br>`Deviasi All[Perubahan]`<br>`Sales All[Sales GAP]`<br>`Sales All[% Sales Growth]`<br>`Deviasi All[Show Value]: NOT(Show Value = null)` |
| 14 | donutChart | Kontribusi | **Category**: Inventory[Nama Barang]<br>**Y**: Deviasi All[COM CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |
| 15 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Tooltips**: Deviasi All[Multi Value LM]<br>**Tooltips**: Deviasi All[Multi Value Gap]<br>**Tooltips**: Deviasi All[Multi Value Growth]<br>**Tooltips**: Deviasi All[Perubahan Multi Value]<br>**Y**: Deviasi All[COM CM] | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?`<br>`Deviasi All[Multi Value LM]`<br>`Deviasi All[Multi Value Gap]`<br>`Deviasi All[Multi Value Growth]`<br>`Deviasi All[Perubahan Multi Value]`<br>`Deviasi All[COM CM]` |

## 21. Waste by Item (16 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Waste by Item | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
| 14 | pivotTable |  | **Columns**: Waste All[Jenis]<br>**Rows**: Inventory[Nama Barang]<br>**Rows**: Resto[Resto]<br>**Values**: SUM(Waste All[Value]) | — |
| 15 | pivotTable |  | **Rows**: Waste All[Jenis]<br>**Values**: % | — |
| 16 | lineClusteredColumnComboChart | Trend | **Category**: Date[Date]<br>**Series**: Waste All[Jenis]<br>**Tooltips**: Waste All[% to Total]<br>**Y**: SUM(Waste All[Value]) | `SUM(Deviasi All[Value])`<br>`Date[Date]: ?` |

## 22. Rekap Brg Masuk (34 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Rekap Brg Masuk | — | — |
| 2 | shape | Minus | — | — |
| 3 | shape | Minus | — | — |
| 4 | shape | Minus | — | — |
| 5 | shape | Plus | — | — |
| 6 | shape | Plus | — | — |
| 7 | shape | Plus | — | — |
| 8 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 9 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 10 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 11 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 12 | shape | Loss | — | — |
| 13 | shape | Loss | — | — |
| 14 | shape | Loss | — | — |
| 15 | shape | Surplus | — | — |
| 16 | shape | Surplus | — | — |
| 17 | shape | Surplus | — | — |
| 18 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 19 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 20 | slicer ⭃sync:Date1 | Slicer Date[Date] | **Values**: Date[Date] | — |
| 21 | actionButton | Reset | — | — |
| 22 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: SUM(Rekap Mentah BM[Value]) | `SUM(Deviasi All[Value])` |
| 23 | barChart | Selisih (Satuan PCS) | **Category**: Inventory[Nama Barang]<br>**Y**: Rekap Mentah BM[Selisih BM] | `Deviasi All[% BOM CM]`<br>`Deviasi All[% GT Deviasi]`<br>`Deviasi All[BOM CM]`<br>`TopN (kondisi kosong)`<br>`Rekap Mentah BM[Selisih BM]: NOT(Selisih BM = 0)`<br>`Inventory[UoM]: IN ('PCS')` |
| 24 | barChart | Selisih (Satuan GR & ML) | **Category**: Inventory[Nama Barang]<br>**Y**: Rekap Mentah BM[Selisih BM] | `Deviasi All[% BOM CM]`<br>`Deviasi All[% GT Deviasi]`<br>`Deviasi All[BOM CM]`<br>`TopN (kondisi kosong)`<br>`Rekap Mentah BM[Selisih BM]: Selisih BM < -1 OR Selisih BM > 1`<br>`Inventory[UoM]: IN ('GR', 'ML')` |
| 25 | barChart | Selisih (Satuan Botol, Pack & KG) | **Category**: Inventory[Nama Barang]<br>**Y**: Rekap Mentah BM[Selisih BM] | `Deviasi All[% BOM CM]`<br>`Deviasi All[% GT Deviasi]`<br>`Deviasi All[BOM CM]`<br>`TopN (kondisi kosong)`<br>`Rekap Mentah BM[Selisih BM]: Selisih BM ≥ -1 OR Selisih BM > 1`<br>`Inventory[UoM]: IN ('BOTOL', 'PACK', 'KG')` |
| 26 | emptyContainer (placeholder) |  | — | — |
| 27 | emptyContainer (placeholder) |  | — | — |
| 28 | emptyContainer (placeholder) |  | — | — |
| 29 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: SUM(4205[#Qty. Terkecil]) | `SUM(Deviasi All[Value])` |
| 30 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: SUM(3224[Kts (Unit#1)]) | `SUM(Deviasi All[Value])` |
| 31 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: 9901[Pembelian di Luar] | `SUM(Deviasi All[Value])` |
| 32 | cardVisual | *(judul disembunyikan: Deviasi)* | **Data**: Rekap Mentah BM[Selisih BM] | `SUM(Deviasi All[Value])` |
| 33 | shape | Klik kanan pada batang grafik, klik Drill Through/Telusuri Paling Detail => ID Dokumen untuk melihat detail barang masuk | — | — |
| 34 | textbox |  | — | — |

## 23. Detail Brg Masuk (10 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Detail Brg Masuk | — | — |
| 2 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 3 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 4 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 5 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 6 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:Date1 | Slicer Date[Date] | **Values**: Date[Date] | — |
| 9 | actionButton | Reset | — | — |
| 10 | tableEx |  | **Values**: Date[Date]<br>**Values**: Resto[Resto]<br>**Values**: Inventory[Nama Barang]<br>**Values**: Inventory[UoM]<br>**Values**: Inventory[Kategori 2]<br>**Values**: SUM(Rekap Mentah BM[Value])<br>**Values**: SUM(4205[#Qty. Terkecil])<br>**Values**: SUM(3224[Kts (Unit#1)])<br>**Values**: 9901[Pembelian di Luar]<br>**Values**: Rekap Mentah BM[Selisih BM] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`SUM(3224[Kts (Unit#1)])`<br>`SUM(4205[#Qty. Terkecil])`<br>`Inventory[(Old) Nama Barang]`<br>`Resto[Restoo]`<br>`Rekap Mentah BM[Selisih BM]`<br>`Date[Date]`<br>`Inventory[Nama Barang]`<br>`Inventory[UoM]`<br>`9901[Pembelian di Luar]`<br>`SUM(Rekap Mentah BM[Value])` |

## 24. Rekon Brg Masuk (11 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Rekon Brg Masuk | — | — |
| 2 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 3 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 4 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 5 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 6 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:Date1 | Slicer Date[Date] | **Values**: Date[Date] | — |
| 9 | actionButton | Reset | — | — |
| 10 | tableEx | Detail Rekonsiliasi Input Barang Masuk | **Values**: Resto[Resto]<br>**Values**: Inventory[Nama Barang]<br>**Values**: Inventory[UoM]<br>**Values**: Rekap Mentah BM[Count BM Resto]<br>**Values**: Rekap Mentah BM[Count BM Sistem]<br>**Values**: Rekap Mentah BM[% BM]<br>**Values**: Rekap Mentah BM[Selisih BM] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`SUM(3224[Kts (Unit#1)])`<br>`SUM(4205[#Qty. Terkecil])`<br>`Inventory[(Old) Nama Barang]`<br>`Date[Date]`<br>`Inventory[UoM]`<br>`9901[Pembelian di Luar]`<br>`SUM(Rekap Mentah BM[Value])`<br>`Resto[Restoo]`<br>`Inventory[Nama Barang]`<br>`Rekap Mentah BM[Count BM Resto]`<br>`Rekap Mentah BM[Show BM]: Show BM = 'Show'`<br>`Rekap Mentah BM[Count BM Sistem]`<br>`Rekap Mentah BM[% BM]`<br>`Rekap Mentah BM[Selisih BM]` |
| 11 | tableEx | Ringkasan Akurasi Rekonsiliasi per Resto | **Values**: Resto[Resto]<br>**Values**: Rekap Mentah BM[Input Benar]<br>**Values**: Rekap Mentah BM[Input Salah]<br>**Values**: Rekap Mentah BM[Total BM]<br>**Values**: Rekap Mentah BM[% BM Benar]<br>**Values**: Rekap Mentah BM[% BM Salah] | `Resto[Restoo]`<br>`Rekap Mentah BM[Input Benar]`<br>`Rekap Mentah BM[Input Salah]`<br>`Rekap Mentah BM[Total BM]`<br>`Rekap Mentah BM[% BM Benar]`<br>`Rekap Mentah BM[% BM Salah]` |

## 25. Sales Growth (8 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Sales Growth | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 6 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 7 | actionButton | Reset | — | — |
| 8 | pivotTable |  | **Rows**: Resto[Area]<br>**Rows**: Resto[Resto]<br>**Values**: Sales All[Sales sd 07 Bulan Lalu]<br>**Values**: Sales All[Sales sd 07 Bulan Ini]<br>**Values**: Sales All[% Sales Growth 07]<br>**Values**: Sales All[Sales sd 14 Bulan Lalu]<br>**Values**: Sales All[Sales sd 14 Bulan Ini]<br>**Values**: Sales All[% Sales Growth 14]<br>**Values**: Sales All[Sales sd 25 Bulan Lalu]<br>**Values**: Sales All[Sales sd 25 Bulan Ini]<br>**Values**: Sales All[% Sales Growth 25]<br>**Values**: Sales All[Sales sd EOM Bulan Lalu]<br>**Values**: Sales All[Sales sd EOM Bulan Ini]<br>**Values**: Sales All[% Sales Growth EOM] | — |

## 26. Rekap PO blm PI (14 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Rekap PO blm PI | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer | Slicer 3243[Status Pesanan Pembelian] | **Values**: 3243[Status Pesanan Pembelian] | — |
| 7 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 8 | slicer | Slicer 3243[Nama Barang] | **Values**: 3243[Nama Barang] | — |
| 9 | slicer | Slicer 3243[Nama Kategori Barang Barang & Jasa] | **Values**: 3243[Nama Kategori Barang Barang & Jasa] | — |
| 10 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 11 | actionButton | Reset | — | — |
| 12 | tableEx |  | **Values**: 3243[Cabang PO]<br>**Values**: SUM(3243[Total Harga])<br>**Values**: 3243[Nilai PI]<br>**Values**: 3243[Belum Dibayar] | — |
| 13 | tableEx |  | **Values**: 3243[Status Pesanan Pembelian]<br>**Values**: SUM(3243[Total Harga])<br>**Values**: 3243[Nilai PI]<br>**Values**: 3243[Belum Dibayar] | — |
| 14 | tableEx |  | **Values**: 3243[Nama Kategori Barang Barang & Jasa]<br>**Values**: SUM(3243[Total Harga])<br>**Values**: 3243[Nilai PI]<br>**Values**: 3243[Belum Dibayar] | — |

## 27. Detail PO blm PI (13 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Detail PO blm PI | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer | Slicer 3243[Status Pesanan Pembelian] | **Values**: 3243[Status Pesanan Pembelian] | — |
| 7 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 8 | slicer | Slicer 3243[Nama Barang] | **Values**: 3243[Nama Barang] | — |
| 9 | slicer | Slicer 3243[Nama Kategori Barang Barang & Jasa] | **Values**: 3243[Nama Kategori Barang Barang & Jasa] | — |
| 10 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 11 | textFilter25A4896A83E0487089E2B90C9AE57C8A | Nomor PO | **field**: 3243[Nomor #] | — |
| 12 | actionButton | Reset | — | — |
| 13 | tableEx |  | **Values**: 3243[Nomor #]<br>**Values**: 3243[Tanggal]<br>**Values**: 3243[Cabang PO]<br>**Values**: 3243[Pemasok]<br>**Values**: 3243[Nama Kategori Barang Barang & Jasa]<br>**Values**: 3243[Kode #]<br>**Values**: 3243[Nama Barang]<br>**Values**: 3243[Nama Barang 9901]<br>**Values**: SUM(3243[Diskon])<br>**Values**: SUM(3243[Total Harga])<br>**Values**: 3243[Nilai PI]<br>**Values**: 3243[Belum Dibayar]<br>**Values**: SUM(3243[Sisa Uang Muka Faktur Uang Muka Terakhir])<br>**Values**: 3243[Keterangan PO]<br>**Values**: 3243[Pembuat Data]<br>**Values**: 3243[Status Pesanan Pembelian] | `3243[Belum Dibayar]`<br>`3243[Status PO]: Status PO = 'Beda'` |

## 28. Detail PI (12 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Detail PI | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer | Slicer 9901[Jenis Barang Barang & Jasa] | **Values**: 9901[Jenis Barang Barang & Jasa] | — |
| 7 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 8 | slicer | Slicer 9901[Nama Barang] | **Values**: 9901[Nama Barang] | — |
| 9 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 10 | textFilter25A4896A83E0487089E2B90C9AE57C8A | Nomor PO | **field**: 9901[Nomor # Pesanan Pembelian] | — |
| 11 | actionButton | Reset | — | — |
| 12 | tableEx |  | **Values**: 9901[Tanggal]<br>**Values**: 9901[Nomor #]<br>**Values**: 9901[Nama Cabang]<br>**Values**: 9901[Nomor # Pesanan Pembelian]<br>**Values**: 9901[Tanggal Penerimaan Barang]<br>**Values**: 9901[Nomor # Penerimaan Barang]<br>**Values**: 9901[Kode #]<br>**Values**: 9901[Nama Barang]<br>**Values**: 9901[#Purch.UoM]<br>**Values**: SUM(9901[#Purch.Qty])<br>**Values**: 9901[#Purch.@Price]<br>**Values**: SUM(9901[#Purch.Discount])<br>**Values**: SUM(9901[#Purch.Total]) | — |

## 29. End (1 visual)

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | ScrollingTextVisual1448795304508 |  | **Category**: Attribute[Attribute] | — |

## 30. Data Deviasi (8 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Data Deviasi | — | — |
| 2 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 3 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 4 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 5 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 6 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 7 | actionButton | Reset | — | — |
| 8 | pivotTable |  | **Columns**: Periode[Periode]<br>**Rows**: Date[Date]<br>**Values**: Deviasi All[Deviasi CM] | — |

## 31. Data Waste (8 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Data Waste | — | — |
| 2 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 3 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 4 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 5 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 6 | actionButton | Reset | — | — |
| 7 | pivotTable |  | **Columns**: Periode[Periode]<br>**Rows**: Date[Date]<br>**Values**: SUM(Waste All[Value]) | — |
| 8 | pivotTable |  | **Columns**: Periode[Periode]<br>**Rows**: Date[Date]<br>**Values**: Deviasi All[Waste]<br>**Values**: SUM(Waste All[Value])<br>**Values**: Waste All[Cek Selisih Waste] | — |

## 32. Data Sales (7 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Data Sales | — | — |
| 2 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 3 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 4 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 5 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 6 | actionButton | Reset | — | — |
| 7 | pivotTable |  | **Columns**: Periode[Periode]<br>**Rows**: Date[Date]<br>**Values**: Sales All[Sales CM] | — |

## 33. Tooltip (3 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | tableEx | 42.05 | **Values**: 4205[Nomor #Kirim]<br>**Values**: 4205[Nomor #Terima]<br>**Values**: SUM(4205[#Qty Kirim])<br>**Values**: 4205[#Sat. Kirim] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`4205[Nomor #Kirim]`<br>`4205[Nomor #Terima]`<br>`SUM(4205[#Qty Kirim])`<br>`4205[#Sat. Kirim]` |
| 2 | tableEx | 32.24 | **Values**: 3224[Nomor # RI]<br>**Values**: 3224[Nomor # PO]<br>**Values**: SUM(3224[Kts Terima])<br>**Values**: 3224[Satuan] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`3224[Nomor # RI]`<br>`3224[Nomor # PO]`<br>`SUM(3224[Kts Terima])`<br>`3224[Satuan]` |
| 3 | tableEx | 99.01 | **Values**: 9901[Nomor #]<br>**Values**: SUM(9901[#Purch.Qty])<br>**Values**: 9901[#Purch.UoM] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`9901[Nomor #]`<br>`SUM(9901[#Purch.Qty])`<br>`9901[#Purch.UoM]`<br>`9901[Nomor # Penerimaan Barang]: Nomor # Penerimaan Barang = null` |

## 34. ID Dokumen (6 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | ID Dok & Detail | — | — |
| 2 | actionButton |  | — | — |
| 3 | tableEx | 42.05 | **Values**: Date[Date]<br>**Values**: 4205[Nomor #Kirim]<br>**Values**: 4205[Nomor #Terima]<br>**Values**: SUM(4205[#Qty Kirim])<br>**Values**: 4205[#Sat. Kirim] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`4205[Nomor #Kirim]`<br>`4205[Nomor #Terima]`<br>`SUM(4205[#Qty Kirim])`<br>`4205[#Sat. Kirim]` |
| 4 | tableEx | 32.24 | **Values**: Date[Date]<br>**Values**: 3224[Nomor # RI]<br>**Values**: 3224[Nomor # PO]<br>**Values**: SUM(3224[Kts Terima])<br>**Values**: 3224[Satuan] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`3224[Nomor # RI]`<br>`3224[Nomor # PO]`<br>`SUM(3224[Kts Terima])`<br>`3224[Satuan]` |
| 5 | tableEx | 99.01 | **Values**: Date[Date]<br>**Values**: 9901[Nomor #]<br>**Values**: SUM(9901[#Purch.Qty])<br>**Values**: 9901[#Purch.UoM] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`9901[Nomor #]`<br>`SUM(9901[#Purch.Qty])`<br>`9901[#Purch.UoM]`<br>`9901[Nomor # Penerimaan Barang]: Nomor # Penerimaan Barang = null`<br>`9901[Jenis Barang Barang & Jasa]: Jenis Barang Barang & Jasa = 'Persediaan'` |
| 6 | tableEx |  | **Values**: Date[Date]<br>**Values**: Resto[Resto]<br>**Values**: Inventory[Nama Barang]<br>**Values**: Inventory[UoM]<br>**Values**: Inventory[Kategori 2]<br>**Values**: SUM(Rekap Mentah BM[Value])<br>**Values**: SUM(4205[#Qty. Terkecil])<br>**Values**: SUM(3224[Kts (Unit#1)])<br>**Values**: 9901[Pembelian di Luar]<br>**Values**: Rekap Mentah BM[Selisih BM] | `Rekap Mentah BM[Clean BM]: NOT(Clean BM = 0)`<br>`SUM(3224[Kts (Unit#1)])`<br>`SUM(4205[#Qty. Terkecil])`<br>`Inventory[(Old) Nama Barang]`<br>`Resto[Restoo]`<br>`Rekap Mentah BM[Selisih BM]`<br>`Date[Date]`<br>`Inventory[Nama Barang]`<br>`Inventory[UoM]`<br>`9901[Pembelian di Luar]`<br>`SUM(Rekap Mentah BM[Value])` |

## 35. Barang Baru (11 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Barang Baru | — | — |
| 2 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 3 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 4 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 5 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 6 | actionButton | Reset | — | — |
| 7 | tableEx | 3243 | **Values**: 3243[Nama Barang] | `3243[Cek Barang]: Cek Barang = null` |
| 8 | tableEx | 9901 | **Values**: 9901[Nama Barang] | `9901[Cek Barang]: Cek Barang = null` |
| 9 | tableEx | 3224 | **Values**: 3224[Nama Barang] | `3224[Cek Barang]: Cek Barang = null` |
| 10 | tableEx | 4205 | **Values**: 4205[Nama Barang] | `4205[Cek Barang]: Cek Barang = null` |
| 11 | tableEx | Deviasi | **Values**: Deviasi All[Nama Barang] | `Deviasi All[Cek Barang]: Cek Barang = null` |

## 36. Kosongan (13 visual) — *hidden page*

| # | Tipe | Judul / Label | Field & Measure | Filter |
|---|---|---|---|---|
| 1 | shape | Kosongan | — | — |
| 2 | slicer ⭃sync:Year | Slicer Date Year | **Values**: Date Year | `Date[Year]: Date[Year] ≠ 2024` |
| 3 | slicer ⭃sync:Month | Slicer Date Month | **Values**: Date Month | — |
| 4 | slicer ⭃sync:Area | Slicer Resto[Area] | **Values**: Resto[Area] | — |
| 5 | slicer ⭃sync:Cabang | Slicer Resto[Resto] | **Values**: Resto[Resto] | — |
| 6 | slicer ⭃sync:JENIS | Slicer Deviasi All[Jenis] | **Values**: Deviasi All[Jenis] | `Deviasi All[Jenis]: NOT(IN ('BOM'))` |
| 7 | slicer ⭃sync:AM | Slicer Resto[AM] | **Values**: Resto[AM] | `Resto[AM]: NOT(AM = null)` |
| 8 | slicer ⭃sync:PIC | Slicer Resto[PIC] | **Values**: Resto[PIC] | — |
| 9 | slicer ⭃sync:Nama Barang2 | Slicer Inventory[Nama Barang] | **Values**: Inventory[Nama Barang] | — |
| 10 | slicer ⭃sync:Week | Slicer Periode[Periode] | **Values**: Periode[Periode] | `Periode[Periode]: NOT(IN (null))` |
| 11 | slicer ⭃sync:Akun Penyesuaian Persediaan | Slicer Inventory[Tipe] | **Values**: Inventory[Tipe] | `Inventory[Tipe]: IN ('COM DEVIASI', 'CONSUME', 'PACKAGING DEVIASI', 'PACKAGING USAGE')` |
| 12 | slicer ⭃sync:Attribute | Slicer Attribute[Attribute] | **Values**: Attribute[Attribute] | `Attribute[Attribute]: NOT(IN (null))` |
| 13 | actionButton | Reset | — | — |
