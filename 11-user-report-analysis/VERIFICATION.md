# AUDIT — Verifikasi Knowledge Tree vs File PBIX Asli

> **Pertanyaan user:** "Cek lagi secara keseluruhan apakah sudah sama dengan file pbix? lengkapi jika belum"
> **Metode:** File `report.pbix` (±148 MB, snapshot 19 Okt 2026) diunduh ulang dari Google Drive, model diekstrak via **pbixray 0.15.5** (`dax_measures`, `dax_columns`, `dax_tables`, `relationships`, `power_query`), layout report diparse dari format **PBIR-enhanced** (`Report/definition/pages/*/page.json` + `visuals/*/visual.json`). Semua item dibandingkan programatis (bukan manual) dengan dokumentasi di repo.
> **Tanggal audit:** 8 Nov 2026 (audit ke-2, lebih dalam dari audit 7 Nov 2026).

## Hasil Audit ke-2 — Ditemukan & Diperbaiki: 3 Temuan

Audit pertama (7 Nov) hanya memverifikasi `model-and-dax.md`. Audit ini membedah **semua file** termasuk sel formula di `measure-glossary.md`, dan menemukan:

| # | Temuan | Lokasi | Perbaikan |
|---|---|---|---|
| 1 | **42 formula measure terpotong / tidak lengkap** di sel formula (kolom "DAX inti"): 34 di antaranya terpotong dengan tanda "…", 8 lainnya menampilkan hanya bagian `RETURN` tanpa prolog `VAR` (Deviasi CM, % BOM CM, Z Score, Top10Dev, Last Update Dev, Status PO, Rank Dev, Rank BOM) | `measure-glossary.md` | ✅ Semua 42 sel diganti dengan **ekspresi lengkap 100% identik** dari pbix (pipe di-escape `\|` sesuai konvensi tabel). Re-verifikasi otomatis: **161/161 baris = LENGKAP & IDENTIK** |
| 2 | **8 label agregasi salah tulis `COUNT`** — agregasi kolom `Deviasi All[Resto]` di 8 pemakaian (hal. Deviasi by Area ×2, Deviasi by PIC, Analisa Susut ×4, COM) sebenarnya `Function=2` = **Distinct count** per schema resmi Microsoft (`QueryAggregateFunction`: 0=Sum, 1=Average, 2=Distinct count, 3=Min, 4=Max, 5=Count non-null, 6=Median), bukan Count biasa | `pages-detail.md` | ✅ Semua 8 label diganti `COUNT(Deviasi All[Resto])` → `DISTINCTCOUNT(Deviasi All[Resto])` |
| 3 | Catatan metodologi: queryRef di visual.json PBIR banyak yang **basi** (nama lama sebelum rename): `Monthly Deviasi.P1` → sebenarnya `Deviasi All[Week 1]`, `Inventory.CONVERT` → `Inventory[Nama Barang]`, `Date.Last Update` → `Update[Last Update Dev]`, `Barang A.*` → `Inventory A[...]`, `DEVIASI ALL` (huruf besar) → `Deviasi All`. Verifikasi HARUS memakai objek `field` (Entity+Property), bukan `queryRef` | — (temuan teknis) | ✅ Verifikasi ini memakai objek `field` — semua angka di bawah sudah bebas dari queryRef basi |

## Hasil Verifikasi Penuh (setelah perbaikan)

| Item | PBIX Asli | Knowledge Tree | Status |
|---|---|---|---|
| Jumlah halaman | 36 | 36 | ✅ Nama, urutan pageOrder, visibility (29 visible + 7 `HiddenInViewMode`) identik |
| Total visual | 539 | 539 | ✅ |
| Visual per halaman | 36 halaman | 36 halaman | ✅ Jumlah visual sama persis semua halaman (Home 10, Summary Deviasi 48, …, Kosongan 13) |
| Komposisi tipe visual | 14 tipe | 14 tipe | ✅ slicer 292, shape 76, actionButton 33, barChart 28, pivotTable 27, cardVisual 24, tableEx 20, unknown/placeholder 11, lineClusteredColumnComboChart 10, textbox 9, donutChart 5, textFilter 2, waterfallChart 1, ScrollingText 1 |
| **Field per visual** | **878 proyeksi field** (375 measure, 414 kolom, 39 agregasi, 44 level hierarki tanggal, 6 visual calc) | `pages-detail.md` | ✅ **878/878 terverifikasi** via objek `field` otoritatif (Entity+Property) |
| Measures — nama | 161 | 161 | ✅ Semua terdokumentasi di `model-and-dax.md` DAN `measure-glossary.md` |
| Measures — formula | 161 ekspresi | 161 ekspresi | ✅ `model-and-dax.md`: 161/161 identik (whitespace-normalized). `measure-glossary.md`: 161/161 identik **setelah perbaikan 42 sel hari ini** |
| Calculated columns non-auto | 25 | 25 | ✅ Semua terdokumentasi & ekspresi identik (84 kolom auto date/time `LocalDateTable*`/`DateTableTemplate*` dikecualikan by design) |
| Calculated tables riil | 7 | 7 | ✅ `Date`, `Date A`, `Date B` (CALENDARAUTO), `Trend`, `Attribute`, `Periode`, `Mode` (DATATABLE) — ekspresi identik. Total 21 di pbix = 7 riil + 14 auto-generated |
| Relasi | 17 | 17 | ✅ Semua from/to kolom & kardinalitas identik (8×M:1 ke Inventory, 9×M:M ke Resto) |
| Tabel Power Query | 16 | 16 | ✅ Inventory, Deviasi All, Waste All, Sales All, Resto, Standar Susut, Inventory A/B, Resto A/B, 3224, 4205, Rekap Mentah BM, Update, 9901, 3243 |
| Angka pemakaian measure (kolom "Visual" di glossary & measure-usage.md) | — | proyeksi + filter + TopN | ✅ Konsisten & tervalidasi (mis. `Show Value` = 5 visual via filter `NOT(Show Value = null)`; `Deviasi CM` = 19; `Growth` = 21 termasuk TopN) — metodologi: semua kemunculan `Tabel[Measure]` di kolom Field & Filter `pages-detail.md` |
| Link navigasi (KNOWLEDGE_MAP, AI_CONTEXT) | — | — | ✅ Semua file `11-user-report-analysis` tercantum & link valid |

## Klaim Penting yang Terkonfirmasi Ulang

- `Rekap Mentah BM` **TIDAK punya relasi ke `Date`** (hanya M:1 ke Inventory + M:M ke Resto) — saran relasi/TREATAS di qa-001/qa-002 tetap valid.
- `[Selisih BM] = SUM('Rekap Mentah BM'[Value]) − ( SUM('3224'[Kts (Unit#1)]) + SUM('4205'[#Qty. Terkecil]) + [Pembelian di Luar] )` — formula lengkap kini terdokumentasi utuh di glossary (dulu terpotong).
- Urutan halaman resmi (pageOrder PBIR): 3=Detail Deviasi, 5=Deviasi by PIC, 6=Deviasi by Resto, 10=Perbandingan, 22=Rekap Brg Masuk, 23=Detail Brg Masuk, 24=Rekon Brg Masuk, 26=Rekap PO blm PI.

## Cara Verifikasi (reprodusibel)

1. Unduh pbix dari Google Drive (ID `19gNo6e0laEfYlWvqVtKYE3C4ukJRd_E4`) via gdown.
2. Model: pbixray 0.15.5 (`dax_measures`, `dax_columns` — filter `LocalDateTable*`/`DateTableTemplate*`, `dax_tables`, `relationships`).
3. Layout: parse `Report/definition/pages/pages.json` (pageOrder) + per halaman `page.json` + `visuals/*/visual.json`.
4. **Ambil field dari objek `field`** (`Measure`/`Column`/`Aggregation`/`HierarchyLevel`/`Arithmetic` → Entity+Property), JANGAN dari `queryRef` (banyak basi pra-rename).
5. Nomor `Aggregation.Function` → 0=Sum, 1=Average, 2=Distinct count, 3=Min, 4=Max, 5=Count non-null, 6=Median (schema `semanticQuery/1.0.0` Microsoft).
6. Normalisasi whitespace → bandingkan string identik dengan sel formula `measure-glossary.md` & blok dax `model-and-dax.md`.

## Catatan & Batasan

- **Snapshot per 19 Okt 2026.** Kalau pbix diedit setelah tanggal itu, repo perlu re-sync.
- Angka pemakaian measure di glossary/measure-usage menghitung **proyeksi field + filter visual + TopN** (metodologi "dipakai di visual"); pemakaian via *conditional formatting* / sort internal tidak dihitung — untuk analisis dampak penuh, cek langsung `pages-detail.md` per halaman.
- 84 calculated columns auto date/time & 14 auto LocalDateTable tidak didokumentasikan satu per satu (by design).
- `unknown` (11) = emptyContainer placeholder tanpa visualType — tidak berisi field.
- Audit ke-1 (7 Nov) tidak memeriksa sel formula glossary — itulah sebabnya 42 kekurangan lolos. Audit ke-2 ini sudah memeriksa **seluruh sel formula satu per satu secara programatis**.

## Kesimpulan

**Setelah 3 perbaikan hari ini, knowledge tree = 100% representasi lengkap & akurat dari pbix per 19 Okt 2026**: 36 halaman, 539 visual (878 field terverifikasi), 161 measures (nama + formula lengkap di 2 file), 25 calculated columns, 7 calculated tables, 17 relasi, 16 tabel Power Query.
