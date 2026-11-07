# DAX Cheatsheet — Kartu Hafalan

> Satu file berisi semua fungsi DAX penting (dikelompokkan per kategori) + 15 snippet siap *copy-paste*. Contoh memakai tabel report Cost Control: `Deviasi All`, `Waste All`, `Sales All`, `Resto`, `Inventory`, `Date`, `Periode`, `Attribute`, `Trend`.

## Konsep: Peta Fungsi DAX

Kolom: **Fungsi | Kegunaan | Contoh singkat**. `X` di akhir nama = iterator (baris-per-baris). Versi lengkap tiap topik ada di file `01-dax/`.

## Sintaks & Contoh

### Fungsi Agregasi

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `SUM` | Total kolom numerik | `SUM('Waste All'[Value])` |
| `AVERAGE` | Rata-rata kolom | `AVERAGE('Standar Susut'[Standar Susut])` |
| `MIN` / `MAX` | Nilai terkecil/terbesar | `MAX('Waste All'[SATUAN])` |
| `COUNT` | Hitung non-blank kolom numerik | `COUNT('Deviasi All'[Value])` |
| `COUNTA` | Hitung non-blank kolom apa pun | `COUNTA('Resto'[PIC])` |
| `COUNTROWS` | Hitung baris tabel (paling jelas) | `COUNTROWS('Deviasi All')` |
| `DISTINCTCOUNT` | Hitung nilai unik | `DISTINCTCOUNT('Deviasi All'[RESTO])` |
| `DISTINCTCOUNTNOBLANK` | Unik tanpa BLANK | `DISTINCTCOUNTNOBLANK('Waste All'[CABANG])` |
| `APPROXIMATEDISTINCTCOUNT` | Unik (SE-optimized, DQ) | `APPROXIMATEDISTINCTCOUNT(Sales[No])` |

### Iterator (fungsi X)

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `SUMX` | Jumlah per baris ekspresi | `SUMX('Sales All', [Qty]*[Harga])` |
| `AVERAGEX` | Rata-rata per baris | `AVERAGEX(VALUES('Date'[Date]), [Deviasi CM])` |
| `MINX` / `MAXX` | Min/Max per baris ekspresi | `MAXX('Resto', [Deviasi CM])` |
| `COUNTX` / `COUNTAX` | Hitung per baris ekspresi | `COUNTAX('Resto', [PIC])` |
| `CONCATENATEX` | Gabung teks dengan pemisah | `CONCATENATEX(VALUES('Resto'[Area]), [Area], ", ")` |
| `RANKX` | Ranking per baris tabel | `RANKX(ALLSELECTED('Resto'[Resto]), [Deviasi CM], , DESC)` |
| `STDEVX.P` / `STDEVX.S` | Std dev populasi/sampel | `STDEVX.P(SummaryTable, [Multi Value CM])` |

### Filter & Context

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `CALCULATE` | Ubah filter context + agregasi | `CALCULATE([Dev CM], 'Deviasi All'[Jenis]="BOM")` |
| `CALCULATETABLE` | `CALCULATE` versi tabel | `CALCULATETABLE(SUMMARIZE(...), ALLSELECTED(...))` |
| `FILTER` | Filter tabel per baris | `FILTER(VALUES('Date'[Date]), [BM Resto]<>0)` |
| `ALL` | Hapus filter (semua/kolom/tabel) | `CALCULATE([Nom], ALL('Inventory'[Nama Barang]))` |
| `ALLEXCEPT` | Hapus filter kecuali kolom tertentu | `ALLEXCEPT('Resto', 'Resto'[Area])` |
| `ALLSELECTED` | Total sesuai pilihan user | `CALCULATE([Sales CM], ALLSELECTED('Resto'[Area]))` |
| `REMOVEFILTERS` | `ALL` versi jelas (filter arg) | `CALCULATE([Min Tgl], REMOVEFILTERS('Date'))` |
| `KEEPFILTERS` | Filter arg ∩ filter ada | `CALCULATE([BOM], KEEPFILTERS('Deviasi All'[Jenis]="BOM"))` |
| `VALUES` | Nilai unik di konteks (dgn baris BLANK) | `VALUES('Deviasi All'[RESTO])` |
| `DISTINCT` | Nilai unik (tanpa baris BLANK) | `DISTINCT('Resto'[Resto])` |
| `SELECTEDVALUE` | Nilai jika tepat satu | `SELECTEDVALUE('Periode'[Periode], "EoM")` |
| `EARLIER` / `EARLIEST` | Akses row context luar | ganti dengan `VAR` bila bisa |
| `TREATAS` | Virtual relationship | `TREATAS(VALUES('Resto A'[Resto]), 'Deviasi All'[Resto])` |

### Time Intelligence

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `TOTALYTD` / `DATESYTD` | Akumulasi sejak awal tahun | `TOTALYTD([Dev CM], 'Date'[Date])` |
| `TOTALMTD` / `DATESMTD` | Akumulasi bulan berjalan | `TOTALMTD([Dev CM], 'Date'[Date])` |
| `PREVIOUSMONTH` / `PREVIOUSYEAR` | Periode sebelumnya | `CALCULATE([Dev CM], PREVIOUSMONTH('Date'[Date]))` |
| `SAMEPERIODLASTYEAR` | Periode sama tahun lalu | `CALCULATE([Dev CM], SAMEPERIODLASTYEAR('Date'[Date]))` |
| `DATEADD` | Geser periode (fleksibel) | `CALCULATE([Dev CM], DATEADD('Date'[Date], -1, MONTH))` |
| `PARALLELPERIOD` | Geser periode utuh | `CALCULATE([Dev CM], PARALLELPERIOD('Date'[Date], -1, YEAR))` |
| `DATESBETWEEN` | Rentang tanggal eksplisit | `DATESBETWEEN('Date'[Date], d1, d2)` |
| `DATESINPERIOD` | Rentang N periode berakhir di tanggal | `DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -3, MONTH)` |
| `FIRSTDATE` / `LASTDATE` | Tanggal awal/akhir konteks | `LASTDATE('Date'[Date])` |
| `FIRSTNONBLANK` / `LASTNONBLANK` | Tanggal dg nilai tidak kosong | `LASTNONBLANK('Date'[Date], [Stok])` |
| `FIRSTNONBLANKVALUE` / `LASTNONBLANKVALUE` | Nilai pertama/terakhir tidak kosong | `LASTNONBLANKVALUE('Date'[Date], SUM(Inv[Stok]))` |
| `CLOSINGBALANCEMONTH` / `OPENINGBALANCEMONTH` | Saldo akhir/awal bulan (stok) | `CLOSINGBALANCEMONTH(SUM(Inv[Stok]), 'Date'[Date])` |
| `CALENDAR` / `CALENDARAUTO` | Buat tabel tanggal | `Date = CALENDARAUTO()` |
| `EOMONTH` / `EDATE` | Akhir bulan / geser bulan | `EOMONTH(DATE(2026,2,1), 0)` |

### Relasi

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `RELATED` | Ambil kolom sisi "one" (row context) | `RELATED(Resto[Area])` |
| `RELATEDTABLE` | Ambil tabel sisi "many" | `COUNTROWS(RELATEDTABLE('Deviasi All'))` |
| `LOOKUPVALUE` | Lookup tanpa relasi | `LOOKUPVALUE(Resto[Resto], Resto[Resto], [RESTO])` |
| `USERELATIONSHIP` | Aktifkan relasi inactive | `CALCULATE([M], USERELATIONSHIP(T[TglKirim], 'Date'[Date]))` |
| `CROSSFILTER` | Ubah arah/filter relasi | `CALCULATE([M], CROSSFILTER(A[K], B[K], Both))` |
| `COMBINEVALUES` | Gabung kolom jadi key relasi | `COMBINEVALUES("-", [Area], [Resto])` |

### Logika

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `IF` | Cabang kondisional | `IF([Dev CM] >= 0, "Surplus", "Loss")` |
| `SWITCH` | Multi-cabang | `SWITCH([Jenis], "WASTE", [Waste], "SUSUT", [Susut], BLANK())` |
| `SWITCH(TRUE(), ...)` | Kondisi berurutan | `SWITCH(TRUE(), ABS([Z])>1, "Anomali", "Normal")` |
| `AND` / `OR` / `NOT` (atau `&&`, `\|\|`, `!`) | Logika boolean | `[A] > 0 && [B] > 0` |
| `IN` | Keanggotaan list | `[Jenis] IN {"Waste","Susut","Trial"}` |
| `BLANK` / `ISBLANK` | Nilai kosong | `IF(ISBLANK([Sales CM]), BLANK(), ...)` |
| `COALESCE` | Nilai pertama non-blank | `COALESCE([LM], 0)` |
| `IFERROR` / `ISERROR` | Tangkap/tes error | `IFERROR([A]/[B], 0)` → ganti `DIVIDE` |
| `DIVIDE` | Pembagian aman | `DIVIDE([Selisih], ABS([Dev LM]))` |

### Informasi & Konteks

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `ISFILTERED` / `ISCROSSFILTERED` | Apakah kolom difilter | `ISFILTERED('Mode'[Mode])` |
| `ISINSCOPE` | Level hierarki di matrix | `IF(ISINSCOPE('Date'[Month]), ...)` |
| `HASONEVALUE` | Tepat satu nilai di konteks | `IF(HASONEVALUE('Periode'[Periode]), ...)` |
| `ISEMPTY` | Tabel kosong | `ISEMPTY(RELATEDTABLE('9901'))` |
| `USERPRINCIPALNAME` | Email user login (RLS) | `USERPRINCIPALNAME()` |
| `SELECTEDMEASURE` / `ISSELECTEDMEASURE` | Calc group | `IF(ISSELECTEDMEASURE([Dev CM]), ...)` |

### Teks

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `LEFT` / `RIGHT` / `MID` | Potong teks | `RIGHT('Deviasi All'[RESTO], 6)` |
| `LEN` | Panjang teks | `LEN([Nama Barang])` |
| `SEARCH` / `FIND` | Cari posisi teks | `SEARCH(" - ", [Nama], 1, 0)` |
| `REPLACE` / `SUBSTITUTE` | Ganti teks | `SUBSTITUTE([Nama], "  ", " ")` |
| `TRIM` / `UPPER` / `LOWER` | Bersih-bersih teks | `UPPER(TRIM([CABANG]))` |
| `CONCATENATE` / `&` | Gabung teks | `"Rp " & FORMAT([M], "#,##0")` |
| `CONTAINSSTRING` | Tes substring | `CONTAINSSTRING([Nama], "FROZEN")` |
| `UNICHAR` / `REPT` | Simbol & pengulangan | `UNICHAR(9650) & REPT("*", [Z])` |
| `VALUE` | Teks → angka | `VALUE("1234")` |

### Tabel

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `SUMMARIZE` | Tabel ringkasan per grup | `SUMMARIZE('Deviasi All', 'Date'[Date])` |
| `SUMMARIZECOLUMNS` | Ringkasan (query) | `SUMMARIZECOLUMNS('Resto'[Area], "D", [Dev CM])` |
| `ADDCOLUMNS` | Tambah kolom hasil ekspresi | `ADDCOLUMNS(VALUES('Resto'[Resto]), "Dev", [Dev CM])` |
| `SELECTCOLUMNS` | Pilih/ubah nama kolom | `SELECTCOLUMNS('Resto', "Cabang", [Resto])` |
| `TOPN` | N baris teratas | `TOPN(10, ALLSELECTED('Inventory'[Nama]), [Nom Dev])` |
| `UNION` / `EXCEPT` / `INTERSECT` | Operasi himpunan tabel | `UNION(TabelA, TabelB)` |
| `CROSSJOIN` / `GENERATE` | Kombinasi baris | `CROSSJOIN(VALUES('Resto'[Resto]), VALUES(Inv[Nama]))` |
| `ROW` | Tabel 1 baris | `ROW("Total", [Deviasi CM])` |
| `DATATABLE` | Tabel literal | `DATATABLE("Periode", STRING, {{"WEEK 1"}})` |
| `GROUPBY` | Group + iterasi dalam | `GROUPBY('Fakta', 'Fakta'[RESTO], "N", COUNTX(CURRENTGROUP(), 1))` |

### Window (DAX 2022+/2024+)

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `OFFSET` | Baris relatif saat ini | `CALCULATE([Dev CM], OFFSET(-1, ORDERBY('Date'[Period])))` |
| `INDEX` | Baris ke-N absolut | `CALCULATE([Dev CM], INDEX(1, ORDERBY([Dev CM], DESC)))` |
| `WINDOW` | Rentang baris (REL/ABS) | `WINDOW(-2, REL, 0, REL, ORDERBY('Date'[Period]))` |
| `RANK` | Peringkat | `RANK(DENSE, ORDERBY([Dev CM], DESC))` |
| `ROWNUMBER` | Nomor urut | `ROWNUMBER(ALLSELECTED('Resto'[Resto]), ORDERBY([Dev CM]))` |
| `ORDERBY` / `PARTITIONBY` / `MATCHBY` | Helper urutan/grup/identitas | `ORDERBY([Dev CM], DESC), PARTITIONBY('Resto'[Area])` |

### Format & Matematika

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `FORMAT` | Angka/tanggal → teks berformat | `FORMAT([Qty], "#,##0")` |
| `FIXED` | Angka → teks dgn desimal tetap | `FIXED([M], 0)` |
| `ROUND` / `ROUNDUP` / `ROUNDDOWN` / `INT` / `TRUNC` | Pembulatan | `ROUND([M], 0)` |
| `ABS` / `SIGN` | Nilai absolut / tanda | `ABS([Deviasi CM])` |
| `MOD` / `QUOTIENT` | Sisa / hasil bagi bulat | `MOD([N], 2)` |
| `CONVERT` / `CURRENCY` | Konversi tipe | `CONVERT([M], STRING)` |

### 15 Snippet Favorit (siap copy)

```dax
-- 1. YTD
Deviasi YTD = TOTALYTD([Deviasi CM], 'Date'[Date])

-- 2. Bulan lalu + growth (ABS agar arah benar)
Deviasi LM = CALCULATE([Deviasi CM], PREVIOUSMONTH('Date'[Date]))
Growth = DIVIDE([Deviasi CM] - [Deviasi LM], ABS([Deviasi LM]))

-- 3. YoY
Deviasi YoY = CALCULATE([Deviasi CM], SAMEPERIODLASTYEAR('Date'[Date]))

-- 4. % to total
% to Total =
VAR T = CALCULATE([Nominal Waste], ALL('Inventory'[Nama Barang]))
RETURN DIVIDE([Nominal Waste], T)

-- 5. Market share (hormati slicer)
% Share Area =
DIVIDE([Sales CM], CALCULATE([Sales CM], ALLSELECTED('Resto'[Area])))

-- 6. Ranking
Rank Dev = RANKX(ALLSELECTED('Inventory'[Nama Barang]), [Nominal Dev], , DESC, DENSE)

-- 7. Top N dinamis (what-if 'Top N')
Is Top N = [Rank Dev] <= SELECTEDVALUE('Top N'[Top N], 10)

-- 8. Switch measure (slicer 'Trend')
Nilai Aktif =
SWITCH(SELECTEDVALUE('Trend'[Kategori]),
    "DEVIASI", [Deviasi CM], "BOM", [BOM CM], "% BOM", [% BOM CM], BLANK())

-- 9. Z-Score + kategori anomali
Z Score =
VAR Curr = [Multi Value CM]
VAR T = CALCULATETABLE(SUMMARIZE('Deviasi All', 'Date'[Date]), ALLSELECTED('Deviasi All'))
VAR Mean = AVERAGEX(T, [Multi Value CM])
VAR Std = STDEVX.P(T, [Multi Value CM])
RETURN DIVIDE(Curr - Mean, Std)
Kategori Anomali = SWITCH(TRUE(), ABS([Z Score]) > 1, "Anomali", "Normal")

-- 10. Running total
Running Total =
CALCULATE([Waste CM],
    FILTER(ALLSELECTED('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date])))

-- 11. Moving average 3 bulan
MA 3 Bulan = AVERAGEX(DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -3, MONTH), [Waste CM])

-- 12. Angka + satuan (display saja - hasil TEXT)
Qty Waste =
FORMAT(CALCULATE(SUM('Waste All'[Value]), Attribute[Attribute] = "QTY"), "#,##0")
    & " " & MAX('Waste All'[SATUAN])

-- 13. GAP A vs B (TREATAS, tabel A/B disconnected)
A Deviasi = CALCULATE([Deviasi CM], TREATAS(VALUES('Resto A'[Resto]), 'Deviasi All'[Resto]))
B Deviasi = CALCULATE([Deviasi CM], TREATAS(VALUES('Resto B'[Resto]), 'Deviasi All'[Resto]))
GAP Deviasi = [A Deviasi] - [B Deviasi]

-- 14. Kategorisasi kondisional berurutan
Analisa =
SWITCH(TRUE(),
    ISBLANK([% BOM CM]), BLANK(),
    [% BOM CM] < AVERAGE('Standar Susut'[Standar Susut]), "Under Standard",
    [% BOM CM] = AVERAGE('Standar Susut'[Standar Susut]), "Good",
    "Over Standard")

-- 15. Stok akhir periode (semi-additive)
Stok Akhir = LASTNONBLANKVALUE('Date'[Date], SUM(Inventory[Stok Gudang]))
```

## Kesalahan Umum

- **`FORMAT()` di measure yang dijumlahkan/chart** → teks tidak bisa diagregasi; pakai measure numerik terpisah atau dynamic format strings.
- **`RANKX` tanpa `ALL`/`ALLSELECTED`** → semua baris rank 1 (membandingkan dengan dirinya sendiri).
- **`PREVIOUSMONTH` pada tabel non-tanggal / date table tidak ditandai** → hasil BLANK.
- **`VALUES` vs `DISTINCT` tertukar** saat ada BLANK row (relasi terputus) — `VALUES` bisa menghasilkan baris ekstra.
- **Lupa `ABS` di penyebut growth** → perbaikan Loss→Loss terbaca positif.

## Tips Pro

- Hafalkan urutan tuning: **model → measure → visual**; DAX terbaik tidak menolong model bengkak (lihat [Performa DAX](../01-dax/17-dax-performance.md)).
- Semua snippet di atas adalah pola dari report nyata — versi lengkap + penjelasan ada di [Resep DAX](../01-dax/16-dax-patterns-recipes.md).
- Uji snippet di DAX query view: `EVALUATE ROW("Hasil", [Measure])`.

## Terkait

- [Variabel & Error Handling](../01-dax/13-variables-and-error-handling.md)
- [Window Functions](../01-dax/14-window-functions.md)
- [Format & Currency](../01-dax/15-format-and-currency.md)
- [Resep DAX](../01-dax/16-dax-patterns-recipes.md)
- [Performa DAX](../01-dax/17-dax-performance.md)
- [M Cheatsheet](./m-cheatsheet.md) — sisi Power Query
