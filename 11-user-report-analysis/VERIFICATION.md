# AUDIT — Verifikasi Knowledge Tree vs File PBIX Asli

> **Pertanyaan user:** "knowledge sudah 100% sama dengan file pbix asli? cek ulang"
> **Metode:** File `report.pbix` (148.214.218 bytes, last modified 19 Okt 2026) di-download ulang dari Google Drive, diekstrak, lalu dibandingkan item-per-item dengan dokumentasi di repo ini.
> **Tanggal audit:** 7 Nov 2026.

## Hasil: ✅ 100% COCOK

| Item | PBIX Asli | Knowledge Tree | Status |
|---|---|---|---|
| Jumlah halaman | 36 | 36 | ✅ Nama, urutan pageOrder, dan visibility (29 visible + 7 `HiddenInViewMode`) semuanya identik |
| Total visual | 539 | 539 | ✅ |
| Visual per halaman | — | — | ✅ 36/36 halaman jumlah visual sama persis (Home 10, Summary Deviasi 48, …, Kosongan 13) |
| Komposisi tipe visual | 14 tipe | 14 tipe | ✅ Semua count identik: slicer 292, shape 76, actionButton 33, barChart 28, pivotTable 27, cardVisual 24, tableEx 20, unknown 11, lineClusteredColumnComboChart 10, textbox 9, donutChart 5, textFilter 2, waterfallChart 1, ScrollingText 1 |
| Measures | 161 | 161 | ✅ **Semua 161 nama terdokumentasi, SEMUA ekspresi 100% identik** (termasuk `BM Sistem`, `BM Resto`, `Selisih BM`, `Deviasi CM/LM`, `Growth`, `Clean BM`, `% BM`, `Update on BM`, `Last Update BM`, `Pembelian di Luar`) |
| Calculated columns non-auto | 25 | 25 | ✅ Semua terdokumentasi & ekspresi identik (84 kolom auto date/time `LocalDateTable*`/`DateTableTemplate*` dikecualikan by design — sesuai judul section "di luar auto date/time") |
| Calculated tables riil | 7 | 7 | ✅ `Date`, `Date A`, `Date B` (CALENDARAUTO), `Trend`, `Attribute`, `Periode`, `Mode` (DATATABLE) — ekspresi identik semua. Total 21 di pbix = 7 riil + 14 auto-generated |
| Relasi (relationships) | 17 | 17 | ✅ Semua from/to column & kardinalitas identik (8×M:1 ke Inventory, 9×M:M ke Resto) |
| Tabel Power Query | 16 | 16 | ✅ Inventory, Deviasi All, Waste All, Sales All, Resto, Standar Susut, Inventory A/B, Resto A/B, 3224, 4205, Rekap Mentah BM, Update, 9901, 3243 |
| Klaim penting di Q&A 001/002 | — | — | ✅ **Terkonfirmasi**: `Rekap Mentah BM` memang TIDAK punya relasi ke `Date` (hanya ke `Inventory` & `Resto`) — saran relasi/TREATAS di Q&A tetap valid |

## Cara Verifikasi (reprodusibel)

1. Download pbix dari Google Drive (bypass virus-scan warning via `drive.usercontent.google.com/download?...&confirm=t`).
2. Ekstrak ZIP → parse `Report/definition/pages/pages.json` + `page.json` + `visuals/*/visual.json` (format PBIR-enhanced, report definition v1.32).
3. DataModel dibaca via **pbixray 0.15.5**: `dax_measures`, `dax_columns`, `dax_tables`, `relationships`.
4. Normalisasi whitespace ekspresi → bandingkan string identik dengan isi `model-and-dax.md`.

## Catatan & Batasan

- **Snapshot per 19 Okt 2026** — sesuai tanggal modifikasi file pbix yang diaudit. Kalau pbix diedit setelah tanggal itu (tambah measure/halaman/visual), repo ini perlu re-sync. Jalankan ulang audit ini setiap ada perubahan besar.
- `pages-and-visuals.md` tidak mencatat **field/judul tiap visual** (kolom `fields: -` kosong untuk sebagian besar visual) — yang terdokumentasi adalah jumlah, tipe, dan komposisi visual per halaman, plus judul shape/button. Detail field per visual bisa ditambahkan kalau dibutuhkan.
- 84 calculated columns auto date/time & 14 auto LocalDateTable tidak didokumentasikan satu per satu (by design, tidak relevan untuk analisis).
- Measure `Deviasi LM` sempat terdeteksi "hilang" saat audit pertama — terbukti **false negative** akibat parsing regex; verifikasi manual mengonfirmasi measure tersebut ada dan ekspresinya identik.

## Kesimpulan

**Knowledge tree = 100% representasi akurat dari pbix per 19 Okt 2026** untuk: struktur halaman, jumlah & tipe visual, seluruh 161 measures (nama + formula), 25 calculated columns penting, 7 calculated tables riil, 17 relasi, dan 16 tabel Power Query.
