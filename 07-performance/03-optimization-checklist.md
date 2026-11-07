# Optimization Checklist — Model, DAX, Visual, Refresh, Service

> Checklist praktis untuk mengecilkan dan mempercepat report Power BI, diberi peringkat dampak (High/Medium/Low). Angka acuan dari report cost-control: model 145 MB, 36 halaman, 539 visual (292 slicer), halaman terpadat 48 visual.

## Konsep & Penjelasan

### Cara memakai checklist ini

1. Ukur dulu kondisi awal (Performance Analyzer + VertiPaq Analyzer — lihat file terkait), catat baseline.
2. Kerjakan dari dampak **High** ke bawah; satu perubahan satu pengukuran.
3. Verifikasi angka bisnis tidak berubah setelah tiap langkah.

### MODEL

| # | Aksi | Dampak | Catatan untuk report ini |
|---|---|---|---|
| M1 | Hapus kolom & baris yang tidak dipakai visual mana pun | **High** | Kolom sisa `Combine Files` (expand berlebih) di Deviasi/Waste/9901/3243 — audit lewat VertiPaq Analyzer |
| M2 | Matikan **auto date/time**, pakai tabel `Date` + Mark as date table | **High** | 109 calculated columns mayoritas auto-date → potensi -15% s.d. -35% ukuran |
| M3 | Kunci join teks → **integer surrogate key** (`Inventory[ItemID]`) | **High** | 8 tabel membawa dictionary `Nama Barang` sendiri-sendiri |
| M4 | Datatype minimal: integer untuk Year/MonthNumber; date-only; hindari datetime ber-jam | **Medium** | Text → integer untuk kolom kunci/sort |
| M5 | **Star schema**: dim bersih + fact; dedupe master `Resto` agar M:M menjadi M:1 | **Medium** | M:M saat ini gejala master duplikat (lihat [VertiPaq](./01-vertipaq-engine.md)) |
| M6 | TRIM/UPPER kolom kunci di Power Query | **Medium** | `Nama new item ` trailing space — dictionary mengecil, join akurat |
| M7 | Hapus tabel duplikat pembanding (`Inventory A/B`, `Resto A/B`) — ganti TREATAS | **Medium** | Pola `Date A` + TREATAS lebih murah daripada menyalin master |
| M8 | Fact tetap unpivot (`Attribute`/`Jenis`) | *(sudah baik)* | Pertahankan — kompresi optimal |

### DAX

| # | Aksi | Dampak | Catatan |
|---|---|---|---|
| D1 | Filter argumen `CALCULATE` pakai **kolom** (predikat boolean), bukan `FILTER` atas tabel | **High** | `FILTER('Deviasi All', ...)` memindai seluruh tabel; `FILTER(VALUES(...))` bila terpaksa |
| D2 | **VAR** untuk hasil antara (dievaluasi sekali) | **Medium** | Pola `Deviasi YTD` report ini sudah bagus — tiru |
| D3 | `DIVIDE` bukan `/` | **Low** | Anti divide-by-zero; laporan lebih bersih |
| D4 | Reuse measure (`[Deviasi CM]` dipanggil measure lain, bukan di-copy) | **Medium** | 161 measures — cari duplikasi logika identik |
| D5 | Hindari iterator atas tabel besar (`SUMMARIZE`/`AVERAGEX` di fact) di visual banyak | **Medium** | Keluarga `[Z Score]` — pakai hanya di visual yang benar-benar membutuhkan |
| D6 | `SELECTEDVALUE` dengan default; `HASONEVALUE` untuk guard header | **Low** | Stabilkan tampilan saat multi-select |
| D7 | Measure switching (`Trend` + `Multi Value CM`) → pertimbangkan **field parameters** | **Medium** | Lebih murah dipelihara; lihat [Dynamic Filtering](../06-filters-and-slicers/03-dynamic-filtering.md) |

### VISUAL

| # | Aksi | Dampak | Catatan |
|---|---|---|---|
| V1 | **Kurangi visual per halaman**: 48 → 12–15 di *Summary Deviasi* | **High** | Setiap visual = minimal 1 query; jumlah visual = variabel performa terbesar |
| V2 | Gabungkan KPI card kembar → satu card/matrix multi-baris | **High** | 9 card = 9 query; 1 visual hemat 8 query |
| V3 | Jangan duplikasi visual mahal lintas halaman | **Medium** | Bagian yang sama di *by Area/by PIC/by Resto* — satukan lewat field parameter |
| V4 | Custom visuals berat (Scrolling Text/running text) — evaluasi ulang | **Medium** | Render terus-menerus di client (Visual Display) |
| V5 | Slicer: dropdown > list untuk nilai banyak; hindari kolom high-cardinality | **Medium** | `Nama Barang` di list = scroll panjang + render lambat |
| V6 | Sync slicers lintas halaman — kurangi duplikasi 292 slicer | **Medium** | 7 grup inti; sisanya jadi filter pane (lihat [Slicer Guide](../06-filters-and-slicers/02-slicer-guide.md)) |
| V7 | "Show items with no data" hanya untuk visual audit; matikan di matrix besar | **Low** | Jumlah baris bisa meledak |
| V8 | Conditional formatting via icon/SVG ringan, bukan image URL banyak | **Low** | Kurangi beban render |
| V9 | Shape dekoratif (17 buah di satu halaman) — pertimbangkan background image tunggal | **Low** | Murah, tapi jumlahnya menambah Visual Display |

### REFRESH

| # | Aksi | Dampak | Catatan |
|---|---|---|---|
| R1 | **Filter sumber sebelum load** (skip hidden `~$`, pilih sheet terpakai, filter tanggal) | **High** | `Folder.Files` Google Drive memuat banyak file — potong dulu di PQ |
| R2 | **Incremental refresh** untuk tabel fact yang membesar (simpan riwayat 24 bulan) | **High** | `Deviasi All`/`Waste All` bertambah tiap periode |
| R3 | Hybrid tables (RealTime) hanya jika butuh data intraday | **Low** | Tidak dibutuhkan report bulanan-mingguan ini |
| R4 | Combine files efisien: satu `Transform File` function, hindari expand-all | **Medium** | Sudah pola bagus; audit kolom hasil expand |
| R5 | Parallel loading query (Tools → Options → paralel) | **Low** | Bantu sedikit di refresh banyak tabel |
| R6 | Jangan load tabel helper (disable *Enable load*) | **Medium** | Query Parameter/Sample/Transform tidak perlu masuk model |
| R7 | Halaman hidden & tooltip tidak dirender saat buka report — taruh visual berat di halaman dalam, landing page ringkas | **Low** | *Home* 10 visual sudah pas |

### SERVICE

| # | Aksi | Dampak | Catatan |
|---|---|---|---|
| S1 | Gateway **on-premises data gateway** dekat sumber (bukan personal mode) | **Medium** | Sumber Google Drive via PC/gateway — lokasi menentukan latensi refresh |
| S2 | Schedule refresh di jam sepi + notifikasi failure | **Low** | Hindari bentrok jam kerja |
| S3 | Kapasitas: Pro cukup untuk 145 MB/1 GB; Premium/PPU bila model >700 MB atau butuh XMLA/log analytics | **Low** | Monitor pertumbuhan model per kuartal |
| S4 | Usage metrics: pantau halaman terbuka & durasi — bukti prioritas optimasi | **Low** | Halaman jarang dibuka = jangan dioptimasi dulu |

## Sintaks & Contoh

Prioritas 10 langkah pertama untuk report ini (urutan eksekusi):

```text
1.  Matikan auto date/time, Mark as date table              (M2)  → ukur ukuran model
2.  Hapus kolom tak terpakai hasil Combine Files            (M1)
3.  ItemID integer + relasi ulang                           (M3, M6)
4.  Dedupe master Resto → relasi M:1                        (M5)
5.  Hapus Inventory A/B, Resto A/B → ganti TREATAS          (M7)
6.  Kurangi visual Summary Deviasi 48 → 12-15               (V1)
7.  Gabung KPI card                                        (V2)
8.  Sync slicers + turunkan 292 → ±100                      (V6)
9.  Rewrite measure ber-FILTER besar                        (D1)
10. Incremental refresh fact tables                         (R2)
```

Template pengukuran before/after:

```text
Baseline  : model 145 MB | Summary Deviasi first render ___ ms | refresh ___ min
Sesudah M2: model ___ MB | halaman ___ ms | refresh ___ min
Sesudah V1: halaman ___ ms
Verifikasi : [Deviasi Value] total = angka baseline (angka tidak boleh berubah)
```

## Kesalahan Umum

- **Mengoptimasi DAX lebih dulu padahal masalahnya jumlah visual** — V1/V2 (kurangi & gabung visual) hampir selalu memberi hasil terbesar dengan risiko terkecil.
- **Menghapus kolom yang ternyata dipakai visual tersembunyi/halaman hidden** — audit dulu lewat VertiPaq Analyzer + pencarian field di seluruh halaman; hapah satu-satu sambil terukur.
- **Incremental refresh dipasang tanpa filter tanggal di PQ** — kebijakan RangeStart/RangeEnd harus memotong sumber, bukan hanya memfilter sesudah load.
- **Menambah visual "karena masih ada ruang"** — setiap visual membayar query; halaman padat bukan pencapaian.
- **Semua relasi dibidirectional demi "asal filter jalan"** — ambigu dan performa turun; single direction + TREATAS/CROSSFILTER terukur.

## Tips Pro

- Anggap performa sebagai **fitur produk**: catat target di README report (first render < 3 s, refresh < 20 menit) dan lacak tiap rilis.
- Bongkar pekerjaan besar menjadi beberapa rilis kecil yang terukur; jangan "big-bang optimization".
- Data refresh lambat ≠ query lambat — dua masalah berbeda dengan obat berbeda (R untuk refresh, V/D/M untuk render).
- Sebelum menambah tabel/kolom baru, tanya: "visual apa yang memakainya?" Tanpa jawaban, jangan dimuat.
- Simpan snapshot VertiPaq Analyzer + hasil Performance Analyzer di repo (folder ini) — jadikan lampiran keputusan, bukan ingatan.

## Terkait

- [VertiPaq Engine](./01-vertipaq-engine.md) — dasar aksi M1–M7
- [Performance Analyzer](./02-performance-analyzer.md) — cara mengukur before/after
- [Slicer Guide](../06-filters-and-slicers/02-slicer-guide.md) — aksi V5/V6
- [Filter Types](../06-filters-and-slicers/01-filter-types.md) — mode Apply untuk report besar
- [Report Overview](../11-user-report-analysis/REPORT_OVERVIEW.md) — rekomendasi asli untuk report ini
