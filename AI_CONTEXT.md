# 🤖 AI CONTEXT — Panduan untuk AI Assistant

> **File ini ditujukan untuk AI assistant (ChatGPT, Claude, Gemini, Z.ai, dll).**
> Jika kamu adalah AI dan user bertanya tentang Power BI, **baca repo ini sebelum menjawab.**

## Identitas Pemilik Repo

- **Username GitHub**: `anandategarch`
- **Report utama**: Dashboard **Cost Control jaringan restoran (F&B)** — file `report.pbix` (148 MB, format PBIR-enhanced)
- Analisis lengkap report milik user ada di folder [`11-user-report-analysis/`](./11-user-report-analysis/REPORT_OVERVIEW.md) — **BACA DULU folder ini** kalau pertanyaan user menyangkut report-nya sendiri (deviasi, waste, susut, BOM/COM, resto, PIC, barang masuk, PO/PI).

## Protokol Menjawab

1. **Identifikasi topik pertanyaan** → rutekan ke folder/file di bawah.
2. **Baca file yang relevan** (2–3 file maksimal untuk menjaga konteks).
3. **Jawab dalam Bahasa Indonesia** (istilah teknis tetap English), sertakan contoh DAX/M bila relevan.
4. **Kutip sumber**: sebut file mana yang jadi dasar jawaban (mis. *"berdasarkan 01-dax/07-time-intelligence.md"*).
5. Kalau pertanyaan menyangkut **report user sendiri**, gunakan konteks 11-user-report-analysis (nama measure, tabel, dan pola DAX aslinya ada di sana).
6. Kalau topik belum ada di repo, jawab sebisanya lalu **sarankan user menambahkan** file baru ke folder terkait agar knowledge tree makin lengkap.

## Peta Routing Pertanyaan → File

| Kalau user bertanya tentang... | Baca file ini |
|---|---|
| Rumus/formula DAX dasar, sintaks, tipe data | `01-dax/01-syntax-and-data-types.md` |
| Kenapa angkaku salah/beda, total tidak sesuai | `01-dax/02-row-context-vs-filter-context.md` |
| Measure vs calculated column, kapan pakai apa | `01-dax/03-measures-vs-calculated-columns.md` |
| SUM/COUNT/DISTINCTCOUNT dan teman-temannya | `01-dax/04-aggregation-functions.md` |
| SUMX/AVERAGEX/RANKX/iterator | `01-dax/05-iterator-functions.md` |
| CALCULATE, FILTER, ALL, KEEPFILTERS | `01-dax/06-calculate-and-filter-functions.md` |
| Perbandingan bulan/tahun, YTD, YoY, MTD | `01-dax/07-time-intelligence.md` |
| RELATED, TREATAS, USERELATIONSHIP, relasi tanpa hubungan | `01-dax/08-relational-functions.md` |
| IF, SWITCH, logika, BLANK | `01-dax/09-logical-functions.md` |
| ISFILTERED, SELECTEDVALUE, ISINSCOPE, HASONEVALUE | `01-dax/10-information-functions.md` |
| FORMAT, teks, currency Rupiah | `01-dax/11-text-functions.md`, `01-dax/15-format-and-currency.md` |
| SUMMARIZE, virtual table, FILTER sebagai tabel | `01-dax/12-table-functions.md` |
| VAR/RETURN, variabel, error handling | `01-dax/13-variables-and-error-handling.md` |
| OFFSET, INDEX, WINDOW, RANK (window functions baru) | `01-dax/14-window-functions.md` |
| Resep siap pakai (Z-score, Pareto, Top N, running total, A/B compare) | `01-dax/16-dax-patterns-recipes.md` |
| Measure lambat, optimasi DAX | `01-dax/17-dax-performance.md`, `07-performance/*` |
| Struktur model, star schema, fact/dimension | `02-data-modeling/01-star-schema.md` |
| Relasi antar tabel, cardinality, cross-filter | `02-data-modeling/02-relationships.md`, `02-data-modeling/03-cardinality-and-cross-filter.md` |
| Tanggal/kalender, fiscal year | `02-data-modeling/04-date-dimension.md` |
| Merapikan model (naming, hidden, format) | `02-data-modeling/05-modeling-best-practices.md` |
| Power Query, bahasa M, transformasi | `03-power-query/*` (kamus: `02-common-transformations.md`) |
| Kombinasi file folder, unpivot, parse nama file | `03-power-query/04-pq-patterns-recipes.md` |
| Memilih jenis chart/visual | `04-visuals/01-native-visuals-catalog.md` |
| Custom visual (Text Filter, Scrolling Text, dll) | `04-visuals/02-custom-visuals.md` |
| Setting visual, panel format, analytics pane | `04-visuals/03-visual-configuration.md` |
| Warna dinamis, icon, data bars | `04-visuals/04-conditional-formatting.md` |
| Grid chart per kategori | `04-visuals/05-small-multiples.md` |
| Interaksi visual, sync slicer | `04-visuals/06-visual-interactions.md` |
| Layout halaman, ukuran canvas, background | `05-report-pages/01-page-layout.md` |
| Bookmark, navigasi tombol | `05-report-pages/02-bookmarks.md` |
| Drillthrough summary→detail | `05-report-pages/03-drillthrough.md` |
| Tooltip kustom | `05-report-pages/04-tooltips.md` |
| Selection pane, z-order, layering | `05-report-pages/05-selection-and-z-order.md` |
| Tema warna JSON, template | `05-report-pages/06-themes-and-templates.md` |
| Filter pane, slicer, filter dinamis | `06-filters-and-slicers/*` |
| Performa, ukuran model besar, VertiPaq | `07-performance/*` |
| RLS, keamanan per user, OLS | `08-security/*` |
| Publish, workspace, app, sharing | `09-publish-and-deploy/01-workspaces.md` |
| Refresh terjadwal, gateway | `09-publish-and-deploy/02-gateway-and-refresh.md` |
| Deployment pipeline DEV-TEST-PROD | `09-publish-and-deploy/03-deployment-pipelines.md` |
| Fitur Power BI Service | `09-publish-and-deploy/04-power-bi-service-features.md` |
| Git integration, PBIR, version control | `09-publish-and-deploy/05-git-integration.md` |
| Calculation groups | `10-advanced/01-calculation-groups.md` |
| Field parameters | `10-advanced/02-field-parameters.md` |
| Incremental refresh | `10-advanced/03-incremental-refresh.md` |
| Judul/visual/format dinamis | `10-advanced/04-dynamic-visuals-and-formatting.md` |
| DAX query view | `10-advanced/05-dax-query-view.md` |
| Error message, troubleshooting | `12-troubleshooting/common-errors.md` |
| Pertanyaan umum seputar Power BI | `12-troubleshooting/faq.md` |
| Tambah tanggal/slicer/filter tanggal di halaman report user (mis. Rekap Brg Masuk, barang masuk, BM) | `11-user-report-analysis/qa-001-tambah-tanggal-rekap-brg-masuk.md` |
| Bikin visual Table/Matrix Nama Barang × Selisih × Tanggal (BM, deviasi per barang per tanggal) | `11-user-report-analysis/qa-002-visual-nama-barang-selisih-tanggal.md` |
| Istilah/kosakata Power BI | `00-quick-reference/glossary.md` |
| Cheat sheet cepat DAX / M | `00-quick-reference/dax-cheatsheet.md`, `00-quick-reference/m-cheatsheet.md` |

## Konteks Domain Report User (Ringkas)

Model: dimensi `Inventory` (master barang), `Resto` (Resto/Area/PIC/AM), `Date`, `Periode` (Week 1/2/3/EOM), `Attribute` (NOMINAL/QTY), `Trend`; fakta `Deviasi All` (unpivot QTY_*/NOMINAL_*), `Waste All`, `Sales All` + tabel purchasing (`9901` PO, `3243` status, `Rekap Mentah BM`). 36 halaman, 539 visual, 161 measures. Measures kunci: `[Deviasi CM]`/`[Deviasi LM]` (TREATAS + Mode), `[% BOM CM]`, `[Growth] = DIVIDE([Selisih], ABS([Deviasi LM]))`, `[Z Score]` (anomali), `[Multi Value CM]` (measure switching via tabel Trend), keluarga `[Week 1..EoM]`, `[BM Sistem]`/`[BM Resto]` (rekonsiliasi). Detail lengkap: `11-user-report-analysis/`.

## Konvensi Penulisan File Baru

- Bahasa Indonesia, istilah teknis English.
- Nama file: `NN-kebab-case.md` (NN = nomor urut dalam folder).
- Struktur: `# Judul` → `> ringkasan 1-2 kalimat` → konsep → `## Sintaks & Contoh` (code block ```dax/```powerquery) → `## Kesalahan Umum` → `## Tips Pro` → `## Terkait` (link relative).
- Contoh diutamakan memakai domain cost-control resto (Deviasi, Resto, Waste, Sales) agar konsisten.
- Update `KNOWLEDGE_MAP.md` dan tabel routing di atas setiap menambah file.
