# Performa DAX: Optimasi Query & Model

> Query DAX lambat hampir selalu karena kerja yang seharusnya dikerjakan Storage Engine dipaksa ke Formula Engine, atau context transition yang berulang jutaan kali. File ini memetakan pola lambat vs cepat, tools profiling, dan checklist optimasi — dengan contoh before/after dari report Cost Control.

## Konsep & Penjelasan

### Dua engine di balik DAX

| | Formula Engine (FE) | Storage Engine (SE) |
|---|---|---|
| Tugas | Logika kompleks, iterasi, callback | Scan & agregasi data dari VertiPaq |
| Sifat | Single-thread, lambat pada baris banyak | Multi-thread, sangat cepat, punya cache |
| Bahasa | Ekspresi DAX | xmSQL (terlihat di DAX Studio) |

**Prinsip #1:** dorong sebanyak mungkin pekerjaan ke SE (predikat kolom, agregasi sederhana) dan minimalkan loop FE. **Prinsip #2:** SE punya cache — query identik berikutnya nyaris gratis, jadi buat pola query yang konsisten dan *cache-friendly*.

### Pola lambat vs cepat

| Pola lambat | Pola cepat | Kenapa |
|---|---|---|
| `FILTER('TabelFakta', [Kol]="X")` | `'TabelFakta'[Kol]="X"` langsung di `CALCULATE` | Predikat kolom di-*push down* ke SE; FILTER tabel penuh diiterasi FE |
| `FILTER('Fakta', [Measure] > 0)` | `FILTER(VALUES('Dim'[Key]), [Measure] > 0)` | Iterasi dimensi kecil (ratusan) bukan fakta (jutaan) |
| Measure berat dipanggil 2–3× di `RETURN` | Disimpan sekali di `VAR` | Evaluasi sekali + cache |
| `IFERROR([A]/[B], 0)` | `DIVIDE([A], [B], 0)` | `DIVIDE` dioptimalkan; IFERROR membayar jalur exception |
| `COUNT('Fakta'[Kolom])` | `COUNTROWS('Fakta')` | COUNT memindai kolom + filter BLANK; COUNTROWS semantik jelas |
| `IF(ISERROR([A]/[B]), 0, [A]/[B])` | `DIVIDE([A], [B], 0)` | Pembagian dievaluasi dua kali |
| `CALCULATE` di calculated column atas fakta jutaan baris | Pindah ke measure, atau *merge* di Power Query | Context transition per baris = sangat mahal |
| Iterator bersarang yang memanggil `CALCULATE` per baris | VAR per baris + predikat kolom, atau re-desain | Context transition massal membengkakkan query FE |
| `EARLIER` berlapis | `VAR` | Sama hasil, jauh lebih terbaca dan sering lebih cepat |

### Context transition: senjata dan beban

Setiap `CALCULATE` (termasuk pemanggilan measure) di dalam iterator mengubah baris menjadi filter context — fitur powerful (resep Z-Score & Nilai PI memakainya), tapi pada 5 juta baris berarti jutaan sub-konteks. Aturan: **iterasi pada tabel hasil agregasi** (`SUMMARIZE`, `VALUES`, `SUMMARIZECOLUMNS`), bukan tabel fakta mentah.

### Modifier & fungsi yang perlu diawasi

- **Murah:** `REMOVEFILTERS`/`ALL` (rewrite query), predikat kolom, `KEEPFILTERS` (moderat, sering menyelamatkan correctness).
- **Moderat:** `TREATAS` (materialisasi daftar nilai — ok untuk ratusan–ribuan nilai), `USERELATIONSHIP`, `CROSSFILTER`.
- **Mahal:** `FILTER` atas tabel fakta, iterator dengan `CALCULATE` per baris, `DISTINCTCOUNT` kolom ber-kardinalitas tinggi, `LOOKUPVALUE` di kolom fakta besar, `RANKX` tanpa `ALLSELECTED` yang tepat.

### Tools profiling

| Tool | Dipakai untuk |
|---|---|
| **Performance Analyzer** (Desktop) | Mengukur durasi per visual (Visual/DAX Query), *Copy DAX Query* untuk dibawa ke DAX Studio |
| **DAX Studio** | Server Timings (durasi FE vs SE, jumlah SE query), lihat xmSQL, benchmark batch, *run parallel* |
| **VertiPaq Analyzer** (di dalam DAX Studio) | Ukuran tabel/kolom, cardinality, encoding — temukan kolom gemuk |
| **Tabular Editor — Best Practice Analyzer** | Lint model otomatis: calculated column berlebih, measure tanpa format, kolom tak terpakai |
| **Query Diagnostics** (Power Query) | Step mana di M yang lambat / tidak bisa *fold* |

### Bottleneck yang umum

- Kolom ber-kardinalitas tinggi (datetime dengan jam, nomor transaksi) — VertiPaq memampatkan buruk.
- **Auto date/time aktif** — membuat tabel tanggal tersembunyi + hierarki per kolom tanggal (report ini punya 109 calculated columns, mayoritas berasal dari sini).
- Relasi bi-directional dan many-to-many yang memicu ekspansi filter.
- Puluhan visual + slicer sinkron di satu halaman = puluhan query per render.
- `DISTINCTCOUNT` atas kolom jutaan nilai unik.
- RLS dengan `FILTER` kompleks yang dievaluasi per query.

## Sintaks & Contoh

### Before/after 1 — predikat kolom vs FILTER tabel

```dax
-- SEBELUM: FILTER atas seluruh tabel fakta (FE mengiterasi jutaan baris)
BOM CM Lambat =
CALCULATE(
    ABS(SUM('Deviasi All'[Value])),
    FILTER('Deviasi All', 'Deviasi All'[Jenis] = "BOM")
)

-- SESUDAH: predikat kolom langsung (SE scan + cache)
BOM CM Cepat =
CALCULATE(
    ABS(SUM('Deviasi All'[Value])),
    'Deviasi All'[Jenis] = "BOM"
)
-- Dampak ilustratif pada fakta 5-10 juta baris: 5-20x lebih cepat
```

### Before/after 2 — VAR mencegah evaluasi ganda

```dax
-- SEBELUM: cabang IF mengevaluasi [Deviasi CM] dua kali
Status Lambat =
IF([Deviasi CM] > 0, [Deviasi CM], ABS([Deviasi CM]))

-- SESUDAH: sekali ke VAR
Status Cepat =
VAR Nilai = [Deviasi CM]
RETURN IF(Nilai > 0, Nilai, ABS(Nilai))
```

### Before/after 3 — iterator di dimensi, bukan fakta

```dax
-- SEBELUM: FILTER memindai seluruh 'Deviasi All' untuk mengevaluasi measure
Resto Aktif Lambat =
COUNTROWS(FILTER('Deviasi All', [Deviasi CM] <> 0))

-- SESUDAH: iterasi VALUES dimensi (puluhan-ratusan baris) - pattern asli report
Count BM Resto =
COUNTROWS(
    FILTER(VALUES('Date'[Date]), [BM Resto] <> 0)
)
```

### Before/after 4 — context transition massal di calculated column

```dax
-- SEBELUM (calculated column '3243'[Jenis]): CALCULATE per baris tabel besar
Jenis =
VAR _NamaBarang = '3243'[Nama Barang]
RETURN
CALCULATE(
    MAX('9901'[Jenis Barang Barang & Jasa]),
    FILTER('9901', '9901'[Nama Barang] = _NamaBarang)
)

-- SESUDAH: lakukan lookup saat refresh, bukan saat query -
-- Merge '3243' ke '9901' di Power Query (Table.NestedJoin),
-- atau jadikan measure bila hanya dipakai di visual
```

### Before/after 5 — DIVIDE

```dax
-- SEBELUM
Growth Lambat = IFERROR([Selisih] / [Deviasi LM], BLANK())

-- SESUDAH (pattern asli report, +ABS agar arah growth benar)
Growth = DIVIDE([Selisih], ABS([Deviasi LM]))
```

## Kesalahan Umum

- **Mengoptimasi DAX padahal masalahnya model** — kolom tak terpakai, auto date/time, relasi M:M. Profil dengan VertiPaq Analyzer dulu, baru tulis ulang DAX.
- **Mengukur dengan refresh visual pertama** — first render selalu termasuk cold cache; ukur render kedua (warm cache) dan catat durasi DAX Query, bukan Visual.
- **Menyalahkan jumlah measure** — 161 measure bukan masalah; measure yang tidak dipakai visual tidak dievaluasi.
- **`FILTER` tanpa agregasi di tabel fakta** — hampir selalu bisa diganti predikat kolom atau `VALUES` dimensi.
- **Context transition tersembunyi** — pemanggilan measure di dalam iterator adalah `CALCULATE`; sadari dan batasi.
- **`COUNT` vs `COUNTROWS` tertukar semantik** — COUNT kolom melewati BLANK, COUNTROWS tidak; hasil berbeda di data kotor.
- **Testing dengan slicer terbuka semua** — data terpotong oleh filter justru sering mempercepat; benchmark pada worst case (tanpa filter).

## Tips Pro

- Buka **DAX Studio → Server Timings** untuk setiap measure baru: rasio FE:SE yang sehat adalah SE dominan dan jumlah SE query kecil (<10 per measure dalam visual sederhana).
- Jalankan Best Practice Analyzer Tabular Editor berkala — menangkap calculated column, format kosong, dan nama measure tidak konsisten secara massal.
- Matikan *Auto date/time* (Options → Data Load) dan tandai tabel `Date` sebagai date table resmi — menghemat memori & kolom tersembunyi.
- `Table.Buffer` di Power Query untuk sumber kecil yang dipakai berulang; hindari untuk tabel besar (memori).
- Ubah visual "tabel mentah 10.000 baris" menjadi Top N + tombol *drillthrough* — query lebih ringan, UX lebih baik.

## Checklist Optimasi

| Area | Item | Status |
|---|---|---|
| Model | Star schema, tabel Date ditandai, auto date/time OFF | ☐ |
| Model | Kolom tak terpakai dihapus, cardinality did Audit | ☐ |
| Measure | Semua pembagian pakai `DIVIDE` | ☐ |
| Measure | Ekspresi berulang disimpan di `VAR` | ☐ |
| Measure | `FILTER` hanya atas `VALUES`/tabel kecil | ☐ |
| Measure | Iterator bekerja di level agregat (`SUMMARIZE`), bukan fakta mentah | ☐ |
| Visual | Jumlah visual/halaman wajar, Top N untuk tabel besar | ☐ |
| Monitoring | Server Timings dicatat untuk halaman terberat | ☐ |
| Monitoring | BPA dijalankan tiap release model | ☐ |

## Terkait

- [Measures vs Calculated Columns](./03-measures-vs-calculated-columns.md) — pilihan berdampak performa
- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md) — mekanika filter context
- [Variabel & Error Handling](./13-variables-and-error-handling.md) — VAR & DIVIDE sebagai optimasi
- [Window Functions](./14-window-functions.md) — pengganti pattern TOPN/EARLIER mahal
- [Resep DAX](./16-dax-patterns-recipes.md) — resep yang sudah apply prinsip di sini
- [M Cheatsheet](../00-quick-reference/m-cheatsheet.md) — optimasi sisi ETL & query folding
- [Glosarium](../00-quick-reference/glossary.md) — VertiPaq, xmSQL, cardinality
- [Overview Report](../11-user-report-analysis/REPORT_OVERVIEW.md) — temuan performa PBIX nyata (145 MB, 109 calc columns)
