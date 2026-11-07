# 🗺️ Knowledge Map — Peta Lengkap Pohon Pengetahuan

> Peta navigasi semua file di repo ini. Satu baris = satu file + deskripsi singkat.
> AI assistant: lihat juga [AI_CONTEXT.md](./AI_CONTEXT.md) untuk routing pertanyaan → file.

## 00-quick-reference — ⚡ Kartu Hafalan

| File | Isi |
|---|---|
| [dax-cheatsheet.md](./00-quick-reference/dax-cheatsheet.md) | ±80 fungsi DAX dalam 11 kategori + 15 snippet siap copy (YTD, YoY, % to total, rank, Top N, Z-Score, dll) |
| [m-cheatsheet.md](./00-quick-reference/m-cheatsheet.md) | ±50 fungsi Power Query M per kategori + 4 pattern asli (combine folder, unpivot, parse nama file, Google Sheets) |
| [glossary.md](./00-quick-reference/glossary.md) | 96 istilah Power BI A–Z: dari Aggregation sampai Z-score |

## 01-dax — 🧮 Formula DAX (17 file, urut belajar)

| File | Isi |
|---|---|
| [01-syntax-and-data-types.md](./01-dax/01-syntax-and-data-types.md) | Anatomi formula, operator, 7 tipe data, konversi implisit, komentar, IntelliSense |
| [02-row-context-vs-filter-context.md](./01-dax/02-row-context-vs-filter-context.md) | ⭐ Konsep PALING penting: filter context, row context, context transition + diagram & studi kasus |
| [03-measures-vs-calculated-columns.md](./01-dax/03-measures-vs-calculated-columns.md) | Perbedaan, tabel keputusan, dampak VertiPaq, kapan pakai yang mana |
| [04-aggregation-functions.md](./01-dax/04-aggregation-functions.md) | SUM, AVERAGE, COUNT, DISTINCTCOUNT, COUNTROWS + perilaku BLANK |
| [05-iterator-functions.md](./01-dax/05-iterator-functions.md) | Keluarga X (SUMX, AVERAGEX...), RANKX lengkap, CONCATENATEX, performa iterator |
| [06-calculate-and-filter-functions.md](./01-dax/06-calculate-and-filter-functions.md) | ⭐ CALCULATE (urutan evaluasi!), FILTER, ALL, ALLEXCEPT, KEEPFILTERS, ALLSELECTED |
| [07-time-intelligence.md](./01-dax/07-time-intelligence.md) | PREVIOUSMONTH, SAMEPERIODLASTYEAR, DATEADD, TOTALYTD, fiscal year, YTD cutoff mingguan |
| [08-relational-functions.md](./01-dax/08-relational-functions.md) | RELATED, RELATEDTABLE, USERELATIONSHIP, CROSSFILTER, TREATAS (pattern asli report), LOOKUPVALUE |
| [09-logical-functions.md](./01-dax/09-logical-functions.md) | IF, SWITCH(TRUE()) pattern, && dan \|\|, COALESCE, perilaku BLANK dalam aritmatika |
| [10-information-functions.md](./01-dax/10-information-functions.md) | ISBLANK, ISFILTERED, SELECTEDVALUE, HASONEVALUE, ISINSCOPE (subtotal matrix) |
| [11-text-functions.md](./01-dax/11-text-functions.md) | FORMAT (format string lengkap), CONCATENATE, TRIM, SEARCH vs FIND, CONTAINSSTRING |
| [12-table-functions.md](./01-dax/12-table-functions.md) | FILTER, SUMMARIZE vs SUMMARIZECOLUMNS, ADDCOLUMNS, TOPN, DATATABLE, CROSSJOIN, virtual table |
| [13-variables-and-error-handling.md](./01-dax/13-variables-and-error-handling.md) | VAR/RETURN (lazy evaluation, scope), DIVIDE vs /, IFERROR, COALESCE |
| [14-window-functions.md](./01-dax/14-window-functions.md) | INDEX, OFFSET, WINDOW, RANK, ROWNUMBER + ORDERBY/PARTITIONBY (running total & moving average modern) |
| [15-format-and-currency.md](./01-dax/15-format-and-currency.md) | Format string kustom, Rupiah, abbreviations (Jt/M), dynamic format strings, format NOMINAL vs QTY |
| [16-dax-patterns-recipes.md](./01-dax/16-dax-patterns-recipes.md) | ⭐ 12 resep emas: YTD trio, Pareto 80/20, Top N dinamis, Z-Score anomaly, A/B compare TREATAS, measure switching |
| [17-dax-performance.md](./01-dax/17-dax-performance.md) | Formula Engine vs Storage Engine, pola lambat→cepat, before/after, tools optimasi |

## 02-data-modeling — 🏛️ Desain Model

| File | Isi |
|---|---|
| [01-star-schema.md](./02-data-modeling/01-star-schema.md) | Fact vs dimension, diagram star schema model cost-control, anti-pattern one big table, grain |
| [02-relationships.md](./02-data-modeling/02-relationships.md) | Jenis relasi, active/inactive (USERELATIONSHIP), single vs both direction, blank row |
| [03-cardinality-and-cross-filter.md](./02-data-modeling/03-cardinality-and-cross-filter.md) | M:1, 1:1, M:M, cross filter direction, ambiguitas, hierarki Resto—Area—PIC |
| [04-date-dimension.md](./02-data-modeling/04-date-dimension.md) | Membuat date table (CALENDAR/CALENDARAUTO), kolom penting, mark as date table, sort by column, bulan Indonesia |
| [05-modeling-best-practices.md](./02-data-modeling/05-modeling-best-practices.md) | Checklist: hide key, summarize by, naming convention, display folder, matikan auto date/time |

## 03-power-query — 🔧 ETL & Bahasa M

| File | Isi |
|---|---|
| [01-m-language-basics.md](./03-power-query/01-m-language-basics.md) | let...in, case sensitivity, each, operator ?? dan ?, query folding, privacy levels, try...otherwise |
| [02-common-transformations.md](./03-power-query/02-common-transformations.md) | Kamus transformasi harian: promote headers, change type (locale!), split, unpivot, merge, append, group by |
| [03-parameters-and-functions.md](./03-power-query/03-parameters-and-functions.md) | Parameter, custom function, pattern Combine Files (Sample File/Transform File), Google Sheets via Web.Contents |
| [04-pq-patterns-recipes.md](./03-power-query/04-pq-patterns-recipes.md) | 12 resep: combine folder, parse tanggal dari nama file, unpivot prefiks, cleaning key, rekonsiliasi 2 versi |

## 04-visuals — 📊 Visualisasi

| File | Isi |
|---|---|
| [01-native-visuals-catalog.md](./04-visuals/01-native-visuals-catalog.md) | Katalog semua visual bawaan per tujuan (perbandingan, komposisi, KPI, tabel...) + kapan hindari |
| [02-custom-visuals.md](./04-visuals/02-custom-visuals.md) | 17 custom visual populer (Text Filter, Scrolling Text, Decomposition Tree...), certified vs non-certified |
| [03-visual-configuration.md](./04-visuals/03-visual-configuration.md) | Panel Format lengkap, konfigurasi per visual, analytics pane (trend line, constant line) |
| [04-conditional-formatting.md](./04-visuals/04-conditional-formatting.md) | Color scale, rules, field value, data bars, icon SVG custom, dynamic format |
| [05-small-multiples.md](./04-visuals/05-small-multiples.md) | Grid mini chart per kategori — konsolidasi halaman per-resto jadi satu layar |
| [06-visual-interactions.md](./04-visuals/06-visual-interactions.md) | Edit interactions, drill, sync slicers (kritikal untuk 292 slicer!), don't sum percentages |

## 05-report-pages — 📄 Desain Halaman

| File | Isi |
|---|---|
| [01-page-layout.md](./05-report-pages/01-page-layout.md) | Canvas 1280×720, grid & alignment, background image, pola layout, mobile layout |
| [02-bookmarks.md](./05-report-pages/02-bookmarks.md) | Bookmark data vs display, navigasi tombol, toggle chart, clear filter, groups |
| [03-drillthrough.md](./05-report-pages/03-drillthrough.md) | Halaman drill summary→detail, keep all filters, back button, judul dinamis |
| [04-tooltips.md](./05-report-pages/04-tooltips.md) | Tooltip default vs tooltip page (hover card KPI ringan) |
| [05-selection-and-z-order.md](./05-report-pages/05-selection-and-z-order.md) | Selection pane, hide/show/lock, grouping, tab order (accessibility), layering |
| [06-themes-and-templates.md](./05-report-pages/06-themes-and-templates.md) | JSON theme (contoh lengkap tema kuning #F2C811), .json vs .pbit, brand consistency |

## 06-filters-and-slicers — 🎚️ Filter & Slicer

| File | Isi |
|---|---|
| [01-filter-types.md](./06-filters-and-slicers/01-filter-types.md) | 3 level filter, tipe basic/advanced/Top N/measure filter, relative date, NULL handling |
| [02-slicer-guide.md](./06-filters-and-slicers/02-slicer-guide.md) | Semua tipe slicer, selection controls, sync slicers antar halaman, hierarchy slicer |
| [03-dynamic-filtering.md](./06-filters-and-slicers/03-dynamic-filtering.md) | TREATAS, disconnected table, what-if parameter, field parameters, dynamic M parameter |

## 07-performance — ⚡ Performa

| File | Isi |
|---|---|
| [01-vertipaq-engine.md](./07-performance/01-vertipaq-engine.md) | Arsitektur engine, kompresi VertiPaq, kenapa model membengkak, datatype optimal |
| [02-performance-analyzer.md](./07-performance/02-performance-analyzer.md) | Performance Analyzer, DAX Studio (server timings), studi kasus halaman 48 visual |
| [03-optimization-checklist.md](./07-performance/03-optimization-checklist.md) | Checklist model/DAX/visual/refresh/service + prioritas 10 langkah |

## 08-security — 🔒 Keamanan

| File | Isi |
|---|---|
| [01-rls.md](./08-security/01-rls.md) | RLS statis & dinamis (USERPRINCIPALNAME), tabel mapping user→area, testing, RLS vs workspace |
| [02-object-level-security.md](./08-security/02-object-level-security.md) | OLS (sembunyikan tabel/kolom), workspace roles, app audience, sharing & link risk, checklist |

## 09-publish-and-deploy — 🚀 Publikasi

| File | Isi |
|---|---|
| [01-workspaces.md](./09-publish-and-deploy/01-workspaces.md) | Workspace roles, workspace vs app, sharing, publish dari Desktop |
| [02-gateway-and-refresh.md](./09-publish-and-deploy/02-gateway-and-refresh.md) | Personal vs standard gateway, scheduled refresh, sumber Google Drive/Sheets (catatan penting!), migrasi cloud |
| [03-deployment-pipelines.md](./09-publish-and-deploy/03-deployment-pipelines.md) | Pipeline dev→test→prod, data source rules, deployment selective |
| [04-power-bi-service-features.md](./09-publish-and-deploy/04-power-bi-service-features.md) | Usage metrics, lineage view, endorsement, subscription, alerts, analyze in Excel |
| [05-git-integration.md](./09-publish-and-deploy/05-git-integration.md) | Git integration Fabric, format PBIR (struktur file .pbix modern), branch strategy, konflik |

## 10-advanced — 🎓 Fitur Advanced

| File | Isi |
|---|---|
| [01-calculation-groups.md](./10-advanced/01-calculation-groups.md) | Refactor 161 measures → 1 measure + calc group, SELECTEDMEASURE, precedence |
| [02-field-parameters.md](./10-advanced/02-field-parameters.md) | Slicer ganti dimensi/metric (konsolidasi halaman by Area/PIC/Resto/Item) |
| [03-incremental-refresh.md](./10-advanced/03-incremental-refresh.md) | RangeStart/RangeEnd, policy refresh, hybrid table, catatan folder connector |
| [04-dynamic-visuals-and-formatting.md](./10-advanced/04-dynamic-visuals-and-formatting.md) | Judul dinamis, format string dinamis (NOMINAL vs QTY), warna & icon, show/hide |
| [05-dax-query-view.md](./10-advanced/05-dax-query-view.md) | EVALUATE langsung di Desktop, copy query dari visual, debug measure bertahap |

## 11-user-report-analysis — 📊 Analisis Report Asli (Cost Control)

| File | Isi |
|---|---|
| [REPORT_OVERVIEW.md](./11-user-report-analysis/REPORT_OVERVIEW.md) | ⭐ Ringkasan eksekutif: 36 halaman, 539 visual, 161 measures, model, sumber data, teknik unggulan, rekomendasi |
| [pages-and-visuals.md](./11-user-report-analysis/pages-and-visuals.md) | Detail 36 halaman + komposisi visual + fields yang dipakai |
| [pages-detail.md](./11-user-report-analysis/pages-detail.md) | ⭐ Komponen LENGKAP per halaman (539 visual): tipe, judul, field/measure per role, sync group slicer, filter (termasuk Top N & kondisi) — jawab “apa saja komponen di halaman N” |
| [measure-usage.md](./11-user-report-analysis/measure-usage.md) | ⭐ Index dampak: measure/kolom → dipakai di halaman & visual mana + direferensikan measure apa (analisis “kalau hapus measure X apa yang kena”) + glosarium fungsi measure kunci |
| [measure-glossary.md](./11-user-report-analysis/measure-glossary.md) | ⭐ Glosarium fungsi bisnis SEMUA 161 measure dikelompokkan 16 keluarga: fungsi + DAX inti + dipakai di visual/halaman mana + dipanggil oleh measure apa |
| [model-and-dax.md](./11-user-report-analysis/model-and-dax.md) | Tabel, relasi, SEMUA 161 measures dengan DAX lengkap, Power Query inti |
| [qa-001-tambah-tanggal-rekap-brg-masuk.md](./11-user-report-analysis/qa-001-tambah-tanggal-rekap-brg-masuk.md) | ⭐ Solusi lengkap tambah tanggal di halaman Rekap Brg Masuk: slicer tanggal + sync, tampil kolom TANGGAL, tanggal update; warning relasi `Rekap Mentah BM` ↔ `Date` belum ada |
| [qa-002-visual-nama-barang-selisih-tanggal.md](./11-user-report-analysis/qa-002-visual-nama-barang-selisih-tanggal.md) | ⭐ Bikin visual Table/Matrix Nama Barang × Selisih BM × Tanggal; Cara 1 pakai `Rekap Mentah BM[TANGGAL]` (cepat, sisi sistem tidak terfilter tanggal), Cara 2 pakai TREATAS/relasi fisik untuk akurasi per tanggal |
| [VERIFICATION.md](./11-user-report-analysis/VERIFICATION.md) | ✅ Audit 7 Nov 2026: knowledge tree diverifikasi 100% identik dengan pbix asli (36 halaman, 539 visual, 161 measures, 17 relasi) — snapshot per 19 Okt 2026 |

## 12-troubleshooting — 🩺 Perbaikan Masalah

| File | Isi |
|---|---|
| [common-errors.md](./12-troubleshooting/common-errors.md) | 24 error umum: penyebab + solusi (multiple values, circular, blank, refresh gagal...) |
| [faq.md](./12-troubleshooting/faq.md) | 16 pertanyaan nyata + jawaban langsung (kenapa total % tidak 100%, growth negatif, Google Drive, dll) |

---

*Terakhir diperbarui saat initial commit. Update map ini setiap menambah file baru.*
