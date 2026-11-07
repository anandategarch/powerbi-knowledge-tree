# Glosarium Power BI (A–Z)

> Kamus istilah Power BI, DAX, dan ekosistem Fabric — 90+ entri dari *Aggregation* sampai *Z-Score*, dikelompokkan per huruf awal. Dipakai sebagai referensi cepat saat membaca file lain di knowledge tree ini.

## Cara Pakai

Cari istilah per huruf awal. Istilah teknis dipertahankan dalam bahasa Inggris sesuai konvensi industri; definisi dalam Bahasa Indonesia. Tautan ke file pembahasan lebih dalam disertakan bila topiknya dibahas khusus.

## A

| Istilah | Definisi singkat |
|---|---|
| **Active relationship** | Relasi antar tabel yang sedang aktif dipakai filter propagation; relasi lain antar kolom sama bersifat *inactive* (diaktifkan via `USERELATIONSHIP`). |
| **Aggregation** | Fungsi yang meringkas banyak baris menjadi satu nilai (`SUM`, `AVERAGE`, `COUNT`); dasar semua measure. |
| **Analyze in Excel** | Fitur menghubungkan Excel (PivotTable) langsung ke semantic model di Service tanpa mengekspor data. |
| **Auto date/time** | Opsi default yang membuat tabel tanggal tersembunyi + hierarki per kolom tanggal; memakan memori — sebaiknya dimatikan dan diganti date table resmi. |

## B

| Istilah | Definisi singkat |
|---|---|
| **Best Practice Analyzer (BPA)** | Fitur Tabular Editor yang men-scan model terhadap aturan praktik baik (calculated column berlebih, measure tanpa format, dll). |
| **Bi-directional filter** | Arah filter dua arah pada relasi; kadang perlu, sering memicu ambiguitas dan performa buruk — gunakan hemat. |
| **Bookmark** | Snapshot kondisi halaman (filter, slicer, visible/hide visual, data) yang bisa dipanggil ulang via tombol — dasar navigasi & storytelling report. |

## C

| Istilah | Definisi singkat |
|---|---|
| **CALCULATE** | Fungsi DAX terpenting: mengevaluasi ekspresi dalam filter context yang dimodifikasi. [Bahas mendalam](../01-dax/06-calculate-and-filter-functions.md) |
| **Calculated column** | Kolom baru hasil ekspresi DAX, dihitung per baris saat refresh dan disimpan di model — memakan memori; bedakan dari measure. |
| **Calculated table** | Tabel hasil ekspresi DAX (`DATATABLE`, `CALENDARAUTO`, dst) — mis. tabel `Trend`, `Attribute`, `Periode`, `Mode` di report ini. |
| **Card** | Visual sederhana penampilkan satu angka besar (KPI) — mis. card "Deviasi CM". |
| **Cardinality** | Jumlah nilai unik sebuah kolom (atau sifat relasi 1:1, 1:M, M:M). Kardinalitas tinggi = kompresi VertiPaq buruk. |
| **Composite model** | Model yang menggabungkan tabel Import dan DirectQuery/Direct Lake dalam satu semantic model, biasanya dengan agregasi. |
| **Context transition** | Konversi row context menjadi filter context oleh `CALCULATE`/pemanggilan measure di dalam iterator. [Lihat Variabel](../01-dax/13-variables-and-error-handling.md) |
| **Copilot (Power BI)** | Asisten AI di Service/Desktop untuk membuat report, measure, dan narasi dari semantic model. |
| **Cross-filter** | Filter yang menjalar antar tabel lewat relasi (single/bi-directional); juga opsi visual *Edit interactions*. |

## D

| Istilah | Definisi singkat |
|---|---|
| **DAX** | *Data Analysis Expressions* — bahasa formula untuk measure, calculated column, dan query pada model tabular. |
| **DAX query view** | Panel di Desktop (2024+) untuk menulis & menjalankan query DAX (`EVALUATE`), mengetes measure, dan memvisualkan hasil. |
| **DAX Studio** | Tool eksternal untuk profiling (server timings, xmSQL), benchmark query, dan mengelola model tabular. |
| **Dataflow** | ETL terpusat di cloud (Power Query Online) yang hasilnya bisa dipakai banyak semantic model. |
| **Dataset** | Nama lama *semantic model* (sejak 2023 resmi berganti); masih muncul di berbagai API & dokumentasi. |
| **Deployment pipeline** | Fitur Fabric/PPU untuk mempromosikan konteks Dev → Test → Prod secara terkontrol. |
| **Direct Lake** | Mode Fabric yang membaca file parquet di OneLake langsung tanpa import maupun DirectQuery — performa DQ, kecepatan nyaris Import. |
| **DirectQuery** | Mode koneksi yang tidak mengimpor data; setiap visual meng-query sumber secara langsung (near real-time, tapi bergantung performa sumber). |
| **Drill down / up** | Menelusuri hierarki visual (tahun → bulan → tanggal) tanpa berpindah halaman. |
| **Drillthrough** | Tombol/kanan-klik yang membawa filter konteks ke halaman detail terpisah. |
| **Dynamic format strings** | Fitur (2024) membuat format measure ditentukan ekspresi DAX — satu measure bisa tampil `Rp 1.234` atau `12 kg` sesuai slicer. [Bahas mendalam](../01-dax/15-format-and-currency.md) |

## E

| Istilah | Definisi singkat |
|---|---|
| **ETL / ELT** | *Extract, Transform, Load* — tahap menyiapkan data sebelum masuk model; di Power BI dikerjakan Power Query. |
| **Power BI Embedded** | Kapabilitas menyematkan report Power BI ke aplikasi pihak ketiga via API (kapasitas Azure/Fabric). |

## F

| Istilah | Definisi singkat |
|---|---|
| **Fabric** | Platform SaaS data Microsoft (OneLake, Lakehouse, Warehouse, Dataflows Gen2, Power BI, dll) di atas kapasitas bersama. |
| **Field parameter** | Fitur mengubah pilihan kolom/measure dari slicer — alternatif modern untuk measure switching. |
| **Filter context** | Himpunan filter aktif saat measure dievaluasi (slicer, baris matrix, filter visual/halaman/laporan). Konsep inti DAX. |
| **Filter pane** | Panel filter di sisi visualisasi: filter visual, halaman, dan seluruh laporan. |

## G

| Istilah | Definisi singkat |
|---|---|
| **Gateway (on-premises data gateway)** | Jembatan aman antara Power BI Service dan sumber data on-prem (file, database) untuk refresh & DirectQuery. |

## H

| Istilah | Definisi singkat |
|---|---|
| **Hierarchy** | Struktur bertingkat pada kolom (Tahun → Bulan → Tanggal; Area → Resto) untuk drill dan navigasi. |
| **Hybrid table** | Tabel yang menggabungkan partisi DirectQuery (data terbaru) dan Import (historis) — dipakai untuk near real-time. |

## I

| Istilah | Definisi singkat |
|---|---|
| **Import mode** | Mode default: data dimuat & dikompresi ke memori (VertiPaq) — interaksi tercepat, data setinggi refresh terakhir. |
| **Incremental refresh** | Refresh hanya partisi periode terbaru (bukan seluruh historis) — mempercepat refresh harian pada tabel besar. |
| **Iterator** | Fungsi keluarga `X` (`SUMX`, `AVERAGEX`, `FILTER`) yang mengevaluasi ekspresi per baris tabel. [Lihat Cheatsheet](./dax-cheatsheet.md) |

## J

| Istilah | Definisi singkat |
|---|---|
| **Join (Merge vs Append)** | *Merge* = gabung antar tabel berdasarkan key (SQL JOIN); *Append* = tumpuk baris (UNION). Di M: `Table.MergeQueries` vs `Table.Combine`. |

## K

| Istilah | Definisi singkat |
|---|---|
| **Key influencers** | Visual AI yang menganalisis faktor mana yang paling memengaruhi sebuah metrik. |
| **KPI** | *Key Performance Indicator* — metrik kunci bisnis; biasanya ditampilkan sebagai card + target/tren. |

## L

| Istilah | Definisi singkat |
|---|---|
| **Lakehouse** | Storage berbasis file (Delta/parquet) di OneLake dengan kemampuan SQL/Spark — sumber populer untuk Direct Lake. |
| **Lazy evaluation** | Evaluasi ditunda sampai nilai benar-benar dibutuhkan — sifat `VAR` DAX dan langkah M. [Lihat Variabel](../01-dax/13-variables-and-error-handling.md) |
| **Lineage view** | Tampilan Service yang menunjukkan alur dependensi dataflow → semantic model → report → dashboard. |
| **Live connection** | Koneksi report langsung ke model yang sudah dipublish (Analysis Services/semantic model lain) tanpa menyalin data. |

## M

| Istilah | Definisi singkat |
|---|---|
| **M language** | Bahasa Power Query (`let...in`, case-sensitive) untuk ETL. [Cheatsheet M](./m-cheatsheet.md) |
| **Matrix** | Visual pivot/bedah silang (baris × kolom × nilai) — dipakai luas di report deviasi per resto/item. |
| **Measure** | Ekspresi DAX yang dihitung saat query sesuai filter context; tidak menyimpan nilai di model (beda dari calculated column). |
| **Merge** | Gabung dua tabel berdasarkan key (SQL JOIN) di Power Query — `Table.MergeQueries`/`Table.NestedJoin`. |
| **Model view** | Area Desktop untuk melihat/mengelola tabel, relasi, dan folder measure. |
| **Measure switching** | Pattern satu measure yang "berubah wujud" via `SWITCH(SELECTEDVALUE(...))` — mis. `[Multi Value CM]`. [Resep](../01-dax/16-dax-patterns-recipes.md) |

## N

| Istilah | Definisi singkat |
|---|---|
| **Natural language query** | Bertanya ke data dengan bahasa manusia — lihat **Q&A**. |

## O

| Istilah | Definisi singkat |
|---|---|
| **Object-level security (OLS)** | Pembatasan akses pada level objek model (tabel/kolom tertentu tidak terlihat sama sekali), di atas RLS. |

## P

| Istilah | Definisi singkat |
|---|---|
| **PBIX (.pbix)** | Format file report + model Power BI Desktop; **PBIT** adalah template-nya. |
| **Paginated reports** | Report format "SSRS" (.rdl) cetak-ramai — layout presisi, banyak halaman, cocok untuk invoice/laporan resmi. |
| **Performance Analyzer** | Panel Desktop untuk mengukur durasi render tiap visual (Visual/DAX Query) dan menyalin query ke DAX Studio. [Lihat Performa](../01-dax/17-dax-performance.md) |
| **Power BI Desktop** | Aplikasi Windows untuk membuat model & report (gratis). |
| **Power BI Service** | Platform cloud (app.powerbi.com) untuk publish, share, refresh, dan kolaborasi. |
| **Power Query** | Mesin ETL (Desktop & Online) berbasis M; tempat pattern *combine folder*, *unpivot*, *parse nama file* dikerjakan. |
| **Premium capacity** | Kapasitas dedicated (F-SKU/PPU) untuk fitur enterprise: deployment pipeline, XMLA read/write, model besar, dsb. |
| **Push dataset** | Semantic model yang barisnya dikirim via REST API (bukan refresh biasa) — untuk real-time sederhana. |

## Q

| Istilah | Definisi singkat |
|---|---|
| **Q&A** | Visual pemahaman bahasa alami — user mengetik pertanyaan, Power BI menghasilkan visual. |
| **Query folding** | Kemampuan langkah Power Query diterjemahkan menjadi query sumber (SQL) — kunci performa refresh. [Lihat M Cheatsheet](./m-cheatsheet.md) |
| **Quick measure** | Generator measure DAX dari template UI (tanpa menulis DAX). |

## R

| Istilah | Definisi singkat |
|---|---|
| **RLS (Row-Level Security)** | Keamanan per baris: ekspresi DAX (mis. `USERPRINCIPALNAME()`) membatasi baris yang bisa dilihat tiap user. |
| **Refresh** | Proses mengambil data ulang ke semantic model (terjadwal di Service, butuh gateway bila sumber on-prem). |
| **Relationship** | Relasi antar tabel (1:M, M:M, 1:1) dengan arah filter; tulang punggung star schema. |
| **Report view** | Kanvas penyusunan visual di Desktop. |
| **Row context** | Konteks "baris saat ini" saat ekspresi dievaluasi (calculated column / iterator); berubah jadi filter context oleh context transition. |

## S

| Istilah | Definisi singkat |
|---|---|
| **Semantic model** | Nama resmi sejak 2023 untuk *dataset* — model data (tabel, relasi, measure, RLS) yang melayani report. |
| **Slicer** | Visual filter interaktif (dropdown, button, tanggal) — report ini punya 292 slicer dengan sync antar halaman. |
| **Smart narrative** | Visual AI yang men-generate narasi teks dinamis mengikuti filter. |
| **Snowflake schema** | Varian star schema dimana dimensi dinormalisasi lagi ke sub-dimensi. |
| **Star schema** | Desain model ideal: tabel fakta di tengah dikelilingi tabel dimensi — performa & kejelasan terbaik. |
| **Streaming dataset** | Dataset real-time dari stream (Event Hubs/IoT) tanpa penyimpanan historis permanen (kecuali hybrid). |
| **Sync slicers** | Sinkronisasi pilihan slicer antar halaman report. |

## T

| Istilah | Definisi singkat |
|---|---|
| **Tabular Editor** | Tool pihak ketiga untuk mengedit model (measure massal, BPA, deployment script C#/Python). |
| **Template (.pbit)** | File template report tanpa data — dipakai untuk duplikasi desain/model ke koneksi lain. |
| **Theme** | File JSON gaya terpusat (warna, font, format) untuk konsistensi seluruh report. |
| **Tie-breaking** | Aturan penanganan nilai kembar pada ranking (`SKIP` menyisakan gap, `DENSE` rapat) — krusial di RANKX/RANK & Pareto. |
| **Time intelligence** | Keluarga fungsi periode (`PREVIOUSMONTH`, `SAMEPERIODLASTYEAR`, `TOTALYTD`, dsb). [Bahas mendalam](../01-dax/07-time-intelligence.md) |
| **Tooltip** | Popup info saat hover; bisa berupa *tooltip page* khusus berisi mini-report. |

## V

| Istilah | Definisi singkat |
|---|---|
| **VertiPaq** | Engine kolom in-memory yang menyimpan data Import, dikompresi & di-query oleh Storage Engine. |
| **Visual** | Unit tampilan report (chart, card, slicer, matrix); *visual calculation* (2024) = DAX yang dihitung langsung pada hasil visual. |
| **Visual calculations** | Measure yang dievaluasi pada hasil visual (bukan model) — memakai `ROWS`/`WINDOW` helpers, dirilis bersama RANK/ROWNUMBER. [Lihat Window Functions](../01-dax/14-window-functions.md) |

## W

| Istilah | Definisi singkat |
|---|---|
| **Warehouse (Fabric)** | Gudang data SQL di Fabric (lakehouse + SQL endpoint). |
| **What-if parameter** | Tabel + slicer + measure (`SELECTEDVALUE`) untuk simulasi dinamis — mis. Top N dinamis. [Resep](../01-dax/16-dax-patterns-recipes.md) |
| **Window functions** | Fungsi DAX `INDEX`, `OFFSET`, `WINDOW`, `RANK`, `ROWNUMBER` + `ORDERBY`/`PARTITIONBY`/`MATCHBY` (DAX 2022+/2024+). [Bahas mendalam](../01-dax/14-window-functions.md) |
| **Workspace** | Wadah kolaborasi di Service berisi semantic model, report, dashboard, dengan akses berbasis role. |

## X

| Istilah | Definisi singkat |
|---|---|
| **XMLA endpoint** | Protokol akses model (read/write) untuk tool eksternal — DAX Studio, Tabular Editor, SSMS, automation. |
| **xmSQL** | Bahasa query internal Storage Engine yang terlihat di DAX Studio — membaca xmSQL = memahami apa yang benar-benar dieksekusi. [Lihat Performa](../01-dax/17-dax-performance.md) |

## Y

| Istilah | Definisi singkat |
|---|---|
| **YoY (Year-over-Year)** | Perbandingan periode vs periode sama tahun lalu (`SAMEPERIODLASTYEAR` + growth). [Lihat Time Intelligence](../01-dax/07-time-intelligence.md) |

## Z

| Istilah | Definisi singkat |
|---|---|
| **Z-score** | Skor standar `(x − mean) / stdev` untuk deteksi anomali — pattern asli report: `SUMMARIZE` + `AVERAGEX` + `STDEVX.P`. [Resep lengkap](../01-dax/16-dax-patterns-recipes.md) |

## Terkait

- [DAX Cheatsheet](./dax-cheatsheet.md) — fungsi DAX per kategori
- [M Cheatsheet](./m-cheatsheet.md) — fungsi Power Query
- [Overview Report Cost Control](../11-user-report-analysis/REPORT_OVERVIEW.md) — istilah dalam konteks report nyata
