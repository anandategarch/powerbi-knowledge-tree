# Table Functions: FILTER, SUMMARIZE, dan Virtual Table

> Di DAX, segala sesuatu pada akhirnya adalah tabel: filter context adalah kumpulan tabel, filter argument CALCULATE adalah tabel, dan measure adalah agregasi atas tabel. Menguasai fungsi tabel berarti bisa membangun "tabel virtual" di dalam measure — pola Z Score, Top N, dan YTD kustom semuanya hidup di sini.

## Konsep: Segala Sesuatu Mengembalikan Tabel

Filter argument CALCULATE adalah tabel: `CALCULATE([M], 'Deviasi All'[Jenis] = "BOM")` secara internal diubah menjadi `FILTER(ALL('Deviasi All'[Jenis]), 'Deviasi All'[Jenis] = "BOM")`. Iterator (SUMX, AVERAGEX, ...) berjalan di atas tabel. Karena itu, membangun tabel yang tepat = mengendalikan seluruh perhitungan.

## Function Reference

| Function | Deskripsi | Contoh |
|---|---|---|
| FILTER | Tabel + kondisi per baris (iterator) | `FILTER(ALL('Date'), 'Date'[Year] = 2025)` |
| ALL | Semua baris, abaikan filter | `ALL('Deviasi All')` |
| ALLSELECTED | Abaikan filter visual, pertahankan pilihan slicer | `ALLSELECTED('Deviasi All')` |
| VALUES / DISTINCT | Nilai unik yang terlihat / nilai unik kolom | `VALUES('Date'[Date])` |
| SUMMARIZE | Grouping kolom-kolom | `SUMMARIZE('Deviasi All', 'Date'[Date])` |
| SUMMARIZECOLUMNS | Grouping modern + IGNORE + rollup | `SUMMARIZECOLUMNS('Date'[Year], "D", [Deviasi CM])` |
| ADDCOLUMNS | Tambah kolom kalkulasi | `ADDCOLUMNS(t, "M", [Deviasi CM])` |
| SELECTCOLUMNS | Proyeksi / rename kolom | `SELECTCOLUMNS(t, "Tgl", 'Date'[Date])` |
| CROSSJOIN | Produk kartesius dua tabel | `CROSSJOIN(t1, t2)` |
| UNION | Gabung tabel (duplikat dipertahankan) | `UNION(t1, t2)` |
| EXCEPT | Baris tabel pertama tanpa yang ada di tabel kedua (hasil distinct) | `EXCEPT(t1, t2)` |
| INTERSECT | Baris yang ada di kedua tabel (hasil distinct) | `INTERSECT(t1, t2)` |
| GENERATE / GENERATEALL | Join berkorelasi per baris (inner / left join) | `GENERATE(t1, t2)` |
| ROW | Tabel satu baris | `ROW("Label", "Deviasi", "Nilai", [Deviasi CM])` |
| DATATABLE | Tabel statis bertipe | `DATATABLE("Periode", STRING, ...)` |
| TOPN | N baris teratas | `TOPN(10, t, [@Dev], DESC)` |
| CALCULATETABLE | CALCULATE yang mengembalikan tabel | `CALCULATETABLE(VALUES('Date'[Date]), ...)` |

## FILTER dan Keluarga ALL

```dax
-- FILTER dibutuhkan saat kondisinya per-baris dan kompleks:
Deviasi 2024 NonJakarta =
CALCULATE(
    [Deviasi CM],
    FILTER(
        ALL('Date'),
        'Date'[Year] = 2024
    ),
    FILTER(
        VALUES(Resto[Area]),
        NOT CONTAINSSTRING(Resto[Area], "Jakarta")
    )
)

-- Kalau kondisinya sederhana, tulis langsung (sugar untuk FILTER di atas kolom):
CALCULATE([Deviasi CM], 'Deviasi All'[Jenis] = "BOM")
```

**VALUES vs DISTINCT**: keduanya mengembalikan nilai unik, tetapi `VALUES` juga menyertakan baris BLANK tambahan saat ada referential integrity violation (nilai di fakta yang tidak ada di dimensi) — `DISTINCT` tidak. Jika total keduanya berbeda, model Anda "bocor" dan perlu dibersihkan.

## SUMMARIZE vs SUMMARIZECOLUMNS

| Aspek | SUMMARIZE | SUMMARIZECOLUMNS |
|---|---|---|
| Grouping | ya | ya, dengan auto-exist antar kolom dari tabel yang sama |
| Kolom hasil (row expression) | bisa, tapi hindari (jebakan context transition) | ya, dengan `IGNORE()` untuk BLANK |
| Rollup / subtotal | `ROLLUP`, `ROLLUPGROUP` | `ROLLUPADDISSUBTOTAL`, `ROLLUPGROUP` |
| Dipakai dalam measure (virtual table) | aman — menghormati konteks luar | hindari: tidak menghormati konteks luar dengan benar |
| Cocok untuk | virtual table di measure | calculated table / tabel statis |

```dax
-- Pola yang benar: SUMMARIZE untuk grouping, ADDCOLUMNS di LUAR untuk measure
VAR SummaryTable =
    ADDCOLUMNS(
        SUMMARIZE('Deviasi All', 'Date'[Date]),
        "@Deviasi", [Deviasi CM]
    )

-- SUMMARIZECOLUMNS dengan IGNORE dan rollup (untuk calculated table):
Tabel Ringkasan =
SUMMARIZECOLUMNS(
    ROLLUPADDISSUBTOTAL('Date'[Year], "IsTotal"),
    "Deviasi", IGNORE([Deviasi CM])
)
```

## ADDCOLUMNS, SELECTCOLUMNS, dan TOPN

```dax
-- Top 10 barang penyusut (dasar measure Top10Dev):
Top 10 Barang Deviasi =
VAR Ranking =
    ADDCOLUMNS(
        VALUES('Inventory'[Nama Barang]),
        "@Dev", [ABS Deviasi]
    )
VAR TopItems =
    TOPN(10, Ranking, [@Dev], DESC)   -- hati-hati ties: bisa lebih dari 10 baris
RETURN
    SUMX(TopItems, [@Dev])
```

`SELECTCOLUMNS` memproyeksikan dan mengganti nama kolom — penting untuk menyiapkan tabel sebelum UNION/JOIN yang mensyaratkan nama kolom sama.

## Set Operations: UNION, EXCEPT, INTERSECT

```dax
-- UNION: gabung, duplikat DIPERTAHANKAN (beda dari SQL UNION)
Semua Resto Aktif = UNION(VALUES(Resto[Resto]), VALUES('Deviasi All'[RESTO]))

-- EXCEPT: master minus yang sudah bertransaksi = item/resto "belum ada data"
Barang Tanpa Deviasi =
EXCEPT(
    VALUES(Inventory[(Old) Nama Barang]),
    VALUES('Deviasi All'[Nama Barang])
)

-- INTERSECT: irisan (hasil distinct)
Barang Ada Keduanya =
INTERSECT(
    VALUES('Deviasi All'[Nama Barang]),
    VALUES('Waste All'[Nama new item])
)
```

## GENERATE dan GENERATEALL — Join Berkorelasi

```dax
-- GENERATE: untuk tiap Area, ambil resto-nya (inner join per baris)
Area x Resto =
GENERATE(
    VALUES(Resto[Area]),
    CALCULATETABLE(VALUES(Resto[Resto]))
)

-- GENERATEALL: Area tanpa resto TETAP muncul (LEFT JOIN — sisi kanan blank).
-- Inilah dasar "show data for empty periods" versi measure:
Periode x Barang =
GENERATEALL(
    VALUES('Date'[Period]),
    CALCULATETABLE(VALUES('Inventory'[Nama Barang]))
)
```

`GENERATE` mengevaluasi tabel kedua **dalam row context tabel pertama** — join yang selalu berkorelasi, bukan kartesius. `CROSSJOIN` sebaliknya: semua × semua.

## ROW dan DATATABLE

```dax
-- Tabel helper statis (asli di report cost-control):
Periode =
DATATABLE(
    "Periode", STRING,
    "Sort", INTEGER,
    {
        {"WEEK 1", 1},
        {"WEEK 2", 2},
        {"WEEK 3", 3},
        {"EOM", 4}
    }
)

Trend =
DATATABLE(
    "Kategori", STRING,
    {
        {"DEVIASI"},
        {"BOM"},
        {"% BOM"},
        {"% SALES"},
        {"VARIANCE"}
    }
)

-- ROW: tabel satu baris untuk mengemas hasil sementara
VAR Hasil = ROW("Mean", [MeanDev], "StdDev", [StdDevDev])
```

DATATABLE mendukung tipe STRING, INTEGER, DOUBLE, CURRENCY, BOOLEAN, DATETIME. Kolom `Sort` + *Sort By Column* membuat WEEK 1/2/3/EOM terurut benar. Tabel helper seperti `Mode`, `Attribute`, `Trend` dirancang *disconnected* — dipakai lewat SELECTEDVALUE/ISFILTERED/TREATAS.

## Table Variables, Virtual Table, dan CALCULATETABLE

```dax
-- Pattern Z Score asli: tabel virtual + iterator + context transition
Z Score =
VAR CurrentValue = [Multi Value CM]

VAR SummaryTable =
    CALCULATETABLE(
        SUMMARIZE('Deviasi All', 'Date'[Date]),   -- satu baris per tanggal
        ALLSELECTED('Deviasi All')                -- latar: semua yang dipilih user
    )

VAR MeanValue =
    AVERAGEX(SummaryTable, [Multi Value CM])      -- measure per baris = context transition

VAR StdDevValue =
    STDEVX.P(SummaryTable, [Multi Value CM])

RETURN
    DIVIDE(CurrentValue - MeanValue, StdDevValue)

-- CALCULATETABLE = CALCULATE versi tabel (filter argumen, hasil tabel):
VAR TanggalBulanIni =
    CALCULATETABLE(
        VALUES('Date'[Date]),
        'Date'[Year] = 2025,
        'Date'[MonthNumber] = 3
    )
```

Cara baca Z Score: `SummaryTable` adalah daftar tanggal dalam lingkup ALLSELECTED; `AVERAGEX`/`STDEVX.P` mengevaluasi `[Multi Value CM]` untuk **tiap tanggal** (measure reference dalam row context memicu context transition), lalu merata-ratakan — menghasilkan rata-rata dan simpangan baku lintas waktu untuk menandai anomali.

## Kesalahan Umum

- **SUMMARIZE menambahkan measure di dalamnya** (extension column) → jebakan context transition: measure dievaluasi dalam konteks grup yang belum lengkap, hasil bisa salah dan tidak konsisten antar versi. *Perbaikan*: `ADDCOLUMNS(SUMMARIZE(...), "M", [Measure])` — grouping dan kalkulasi dipisah.
- **SUMMARIZECOLUMNS sebagai virtual table di dalam measure** → tidak menghormati filter context luar dengan benar (dan bisa error di row context). *Perbaikan*: dalam measure gunakan SUMMARIZE + ADDCOLUMNS; SUMMARIZECOLUMNS untuk calculated table.
- **CROSSJOIN meledak** — `CROSSJOIN(ALL('Date'[Date]), ALL('Inventory'[Nama Barang]))` = jutaan baris, timeout. *Perbaikan*: persempit dulu dengan VALUES/TOPN/EXCEPT, atau pakai GENERATE yang berkorelasi.
- **TOPN dengan ties** → mengembalikan lebih dari N baris saat nilai terikat. *Perbaikan*: tambahkan tiebreaker unik: `TOPN(10, t, [@Dev], DESC, 'Inventory'[Nama Barang], ASC)`.
- **VALUES vs DISTINCT menyelipkan baris BLANK** (referential integrity) → total dan jumlah baris tak sinkron. *Perbaikan*: bersihkan master/relasi; gunakan DISTINCT sadar-beda bila memang ingin membuang blank tambahan.
- **Menaruh ekspresi skalar sebagai filter CALCULATE** (`CALCULATE([M], IF(...))`) → filter argument harus tabel. *Perbaikan*: bungkus dengan FILTER/VALUES/TREATAS.

## Tips Pro

- **Berpikir dalam tabel**: AVERAGEX(SUMMARIZE(...), ...) adalah pola dasar hampir semua measure analitis (Z Score, rata-rata per resto, indeks) — susun tabelnya dulu di kepala, agregasi belakangan.
- **VAR tabel dievaluasi sekali** lalu dipakai berkali-kali (MeanValue dan StdDevValue di Z Score memakai SummaryTable yang sama) — murah dan terbaca.
- `ALLSELECTED` adalah sahabat "% of total" dan ranking dalam visual: menghormati slicer tetapi melepas filter baris visual.
- `EXCEPT(master, transaksi)` adalah cara termurah membuat daftar "yang belum" — barang tanpa deviasi, resto tanpa waste.
- Tabel statis kecil via DATATABLE (Periode, Attribute, Trend, Mode) + kolom Sort adalah pola standar slicer-helper: deterministik, tanpa query ke sumber, dan mudah di-maintain.

## Terkait

- [CALCULATE & Filter](./06-calculate-and-filter-functions.md) — filter argument = tabel
- [Time Intelligence](./07-time-intelligence.md) — fungsi waktu juga penghasil tabel tanggal
- [Relational Functions](./08-relational-functions.md) — TREATAS dan NATURAL joins antar tabel
- [Information Functions](./10-information-functions.md) — SELECTEDVALUE untuk membaca tabel helper
- [Data Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md) — tabel kalkulasi asli: Periode, Attribute, Trend, Mode
