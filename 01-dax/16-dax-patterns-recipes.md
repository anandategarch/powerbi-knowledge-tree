# Resep DAX Siap Pakai (DAX Patterns Recipes)

> Kumpulan 12 resep "emas" yang terbukti dipakai report Cost Control F&B — setiap resep: **masalah → solusi DAX → penjelasan**. Semua contoh memakai tabel nyata: `Deviasi All`, `Waste All`, `Sales All`, `Resto`, `Inventory`, `Date`, `Periode`, `Mode`, `Trend`.

## Konsep & Penjelasan

Resep disusun dari yang paling sering dipakai (trio periode, % total) sampai advanced (TREATAS A/B, Z-Score). Prinsip semua resep: **measure inti numerik** + `VAR` + `DIVIDE` — sisanya variasi konteks. Salin, ganti nama tabel/kolom, lalu uji di DAX query view.

## Sintaks & Contoh

### 1. Trio Periode: CM / LM / YoY + Growth

**Masalah:** membandingkan bulan berjalan, bulan lalu, dan tahun lalu dalam satu visual.
```dax
Deviasi CM = [Deviasi Value]
Deviasi LM = CALCULATE([Deviasi CM], PREVIOUSMONTH('Date'[Date]))
Deviasi YoY = CALCULATE([Deviasi CM], SAMEPERIODLASTYEAR('Date'[Date]))
Selisih = [Deviasi CM] - [Deviasi LM]
Growth = DIVIDE([Selisih], ABS([Deviasi LM]))
```
**Penjelasan:** `PREVIOUSMONTH`/`SAMEPERIODLASTYEAR` menggeser filter tanggal; `DIVIDE` aman saat LM kosong; `ABS` di penyebut mencegah pembagian `-/-` terbaca growth positif (deviasi -5 vs -10 seharusnya growth **-50%**, bukan +50%).

### 2. Market Share / % to Total

**Masalah:** kontribusi tiap item/resto terhadap total.
```dax
-- Versi total keseluruhan (ALL mengabaikan semua filter kolom tsb)
% to Total =
VAR Total = CALCULATE([Nominal Waste], ALL('Inventory'[Nama Barang]), ALL('Waste All'[Jenis]))
RETURN
DIVIDE([Nominal Waste], Total)

-- Versi market share (menghormati slicer user)
% Share Area =
DIVIDE([Sales CM], CALCULATE([Sales CM], ALLSELECTED('Resto'[Area])))
```
**Penjelasan:** `ALL` = pangsa dari semuanya; `ALLSELECTED` = pangsa dari pilihan user saat ini — pilih sesuai definisi bisnis.

### 3. Pareto 80/20

**Masalah:** menandai item "vital few" penyumbang 80% waste.
```dax
Pareto Kumulatif % =
VAR NilaiItem = [Nominal Waste]
VAR TotalAll = CALCULATE([Nominal Waste], ALLSELECTED('Inventory'[Nama Barang]))
VAR Kumulatif =
    CALCULATE(
        [Nominal Waste],
        FILTER(ALLSELECTED('Inventory'[Nama Barang]), [Nominal Waste] >= NilaiItem)
    )
RETURN
DIVIDE(Kumulatif, TotalAll)

Kategori Pareto =
SWITCH(TRUE(), [Pareto Kumulatif %] <= 0.8, "Vital Few", "Trivial Many")
```
**Penjelasan:** akumulasi dilakukan lewat "jumlahkan semua item yang nilainya ≥ item saat ini". Item dengan nilai sama mendapat kumulatif identik (*tie-breaking*) — jika mengganggu, pecahkan seri dengan `RANK` + kolom unik, atau pakai running total `WINDOW` dari [Window Functions](./14-window-functions.md).

### 4. Ranking + Top N Dinamis (What-if Parameter)

**Masalah:** "Top 10" yang bisa diubah user jadi Top 5/15/20 tanpa edit report.
```dax
-- What-if parameter: 'Top N' (1..20) -> slicer + measure 'Top N Value'
Top N Value = SELECTEDVALUE('Top N'[Top N], 10)

Rank Dev =
RANKX(ALLSELECTED('Inventory'[Nama Barang]), [Nominal Dev], , DESC, DENSE)

Is Top N = [Rank Dev] <= [Top N Value]   -- terapkan Is Top N = 1 sebagai filter visual
```
**Penjelasan:** `RANKX` menghitung peringkat terhadap seluruh pilihan; filter visual `Is Top N` menyembunyikan baris di luar N. Slicernya dibuat otomatis via *Modeling → New parameter → Numeric range*.

### 5. Running Total (Akumulasi)

**Masalah:** garis kumulatif per bulan.
```dax
Running Total Waste =
CALCULATE(
    SUM('Waste All'[Value]),
    FILTER(ALLSELECTED('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date]))
)
```
**Penjelasan:** untuk tiap bulan, jumlahkan semua tanggal ≤ bulan berjalan; `ALLSELECTED` menjaga slicer periode tetap dihormati. Alternatif modern 1 baris: `CALCULATE([Waste CM], WINDOW(1, ABS, 0, REL, ORDERBY(...)))`.

### 6. Moving Average

**Masalah:** haluskan tren bulanan dari noise.
```dax
MA 3 Bulan =
AVERAGEX(
    DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -3, MONTH),
    [Waste CM]
)
```
**Penjelasan:** rata-rata 3 bulan berakhir di bulan berjalan. Butuh moving average per kolom non-tanggal (per resto/item)? Pakai `WINDOW(-2, REL, 0, REL, ORDERBY(...))` — lihat [Window Functions](./14-window-functions.md).

### 7. Anomaly Detection dengan Z-Score (pattern asli)

**Masalah:** menandai tanggal/resto yang nilainya menyimpang dari populasi.
```dax
Z Score =
VAR CurrentValue = [Multi Value CM]
VAR SummaryTable =
    CALCULATETABLE(
        SUMMARIZE('Deviasi All', 'Date'[Date]),   -- populasi: 1 baris per tanggal
        ALLSELECTED('Deviasi All')
    )
VAR MeanValue = AVERAGEX(SummaryTable, [Multi Value CM])
VAR StdDevValue = STDEVX.P(SummaryTable, [Multi Value CM])
RETURN
DIVIDE(CurrentValue - MeanValue, StdDevValue)

Kategori Anomali =
SWITCH(TRUE(), ABS([Z Score]) > 1, "Anomali", "Normal")
```
**Penjelasan:** `ALLSELECTED` menjadikan populasi = pilihan user saat ini; `SUMMARIZE` + iterator mengevaluasi measure per tanggal (context transition). `STDEVX.P` (populasi) dipakai karena barisnya memang seluruh populasi yang dianalisis — pakai `STDEVX.S` bila baris adalah sampel. Threshold 1 σ menandai ±32% ekor distribusi normal; 2 σ lebih konservatif (±5%).

### 8. Perbandingan Dua Periode Dinamis (A vs B) dengan TREATAS

**Masalah:** user memilih dua set (resto/item/tanggal) bebas lalu dibandingkan — tanpa membuat relasi fisik.
```dax
-- 'Date A'/'Date B', 'Resto A'/'Resto B', 'Inventory A'/'B' = tabel DISCONNECTED
A Deviasi =
CALCULATE(
    [Deviasi CM],
    TREATAS(VALUES('Inventory A'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang]),
    TREATAS(VALUES('Resto A'[Resto]), 'Deviasi All'[Resto])
)
B Deviasi =
CALCULATE(
    [Deviasi CM],
    TREATAS(VALUES('Inventory B'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang]),
    TREATAS(VALUES('Resto B'[Resto]), 'Deviasi All'[Resto])
)
GAP Deviasi = [A Deviasi] - [B Deviasi]
```
**Penjelasan:** `TREATAS` mengubah nilai tabel pertama menjadi filter pada kolom kedua — *virtual relationship* sekali pakai. Report ini memadukannya dengan tabel `Mode` (nilai "DINAMIS") + `ISFILTERED` agar measure otomatis mundur ke `PREVIOUSMONTH` ketika slicer Mode tidak dipakai (lihat [Variabel & Error Handling](./13-variables-and-error-handling.md)).

### 9. Dynamic Measure Switching

**Masalah:** satu visual menampilkan metrik berbeda sesuai slicer.
```dax
Multi Value CM =
VAR SalesCM = [% Sales CM]
VAR HasSalesCM = NOT ISBLANK(SalesCM)
RETURN
SWITCH(
    SELECTEDVALUE('Trend'[Kategori]),
    "DEVIASI", [Deviasi CM],
    "BOM", [BOM CM],
    "% BOM", [% BOM CM],
    "% SALES", IF(SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL", SalesCM, BLANK()),
    "VARIANCE", IF(SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL" && HasSalesCM, [% Sales GAP], BLANK()),
    BLANK()
)
```
**Penjelasan:** tabel `Trend` (DATATABLE, disconnected) memberi pilihan DEVIASI/BOM/%BOM/%SALES/VARIANCE; `SWITCH` mengarahkan measure; guard `ISBLANK` menyembunyikan metrik yang tak relevan. Karena semua cabang numerik, padukan dengan [dynamic format strings](./15-format-and-currency.md) agar format ikut berganti. Alternatif modern: *field parameters*.

### 10. Barang Baru (New vs Returning Item)

**Masalah:** menghitung item yang **pertama kali** muncul di periode berjalan.
```dax
Barang Baru =
VAR AwalPeriode = MIN('Date'[Date])
VAR AkhirPeriode = MAX('Date'[Date])
RETURN
COUNTROWS(
    FILTER(
        VALUES('Deviasi All'[Nama Barang]),
        VAR TanggalPertama =
            CALCULATE(MIN('Deviasi All'[Tanggal]), REMOVEFILTERS('Date'))
        RETURN
            TanggalPertama >= AwalPeriode && TanggalPertama <= AkhirPeriode
    )
)
-- Returning item = total distinct item - barang baru
Barang Lama = DISTINCTCOUNT('Deviasi All'[Nama Barang]) - [Barang Baru]
```
**Penjelasan:** `REMOVEFILTERS('Date')` melihat transaksi pertama selamanya **tanpa** menghapus filter resto/item (jadi "baru di resto ini"). Tambahkan `REMOVEFILTERS('Resto')` bila definisinya "baru secara global".

### 11. Semi-Additive Measures (Stok Akhir Periode)

**Masalah:** stok tidak bisa dijumlahkan antar tanggal — yang benar adalah nilai **snapshot terakhir**.
```dax
Stok Akhir =
LASTNONBLANKVALUE(
    'Date'[Date],
    SUM(Inventory[Stok Gudang])
)
```
**Penjelasan:** `LASTNONBLANKVALUE` mengevaluasi `SUM` per tanggal lalu mengambil nilai terakhir yang tidak BLANK — aman saat tanggal snapshot terakhir tidak rapi. Untuk cutoff mingguan (Week 1/2/3/EOM), kombinasikan dengan pattern `[Selected Cutoff Date]`. Varian lain: `LASTNONBLANK`, `CLOSINGBALANCEMONTH`.

### 12. Kontribusi Kumulatif (Resto × Item)

**Masalah:** kontribusi ABS deviasi per sel matriks (Area × Jenis) terhadap total pilihan slicer.
```dax
% Dev Contribution =
-- Pembilang: ikut konteks baris matriks yang aktif
VAR _Numerator =
    SUMX(
        CROSSJOIN(VALUES('Resto'[Resto]), VALUES('Inventory'[Nama Barang])),
        [ABS Deviasi]
    )
-- Penyebut: seluruh kombinasi sesuai slicer
VAR _TotalABS =
    SUMX(
        CROSSJOIN(ALLSELECTED('Resto'[Resto]), ALLSELECTED('Inventory'[Nama Barang])),
        [ABS Deviasi]
    )
RETURN
DIVIDE(_Numerator, _TotalABS, 0)
```
**Penjelasan:** `VALUES` mengikuti baris matriks (mis. satu Area), `ALLSELECTED` melebarkan ke seluruh pilihan slicer — pembilang/penyebut konsisten walau matriks bertingkat. `DIVIDE(..., 0)` menjaga hasil 0 saat penyebut kosong.

## Kesalahan Umum

- **Menghitung growth tanpa `ABS` penyebut** → perbaikan dari Loss ke Loss terbaca growth positif (menyesatkan); selalu `DIVIDE([Selisih], ABS([Basis]))`.
- **Memakai `ALL` padahal maksudnya `ALLSELECTED`** → % total tidak menghormati slicer dan terasa "salah" bagi user.
- **Pareto dengan nilai seri** → kumulatif melempar beberapa item ke kategori yang sama; pecahkan seri dengan kolom unik.
- **TREATAS pada tabel berjuta baris** → materialisasi mahal; `TREATAS` paling hemat untuk daftar nilai (ratusan–ribuan), bukan tabel fakta.
- **Measure switching tanpa guard `BLANK()` / menjumlahkan snapshot stok** (`SUM` antar tanggal) → angka menyesatkan; gunakan guard & semi-additive (resep 11).

## Tips Pro

- Susun measure berlapis: inti (`[Deviasi Value]`) → periode (`[Deviasi CM]`, `[Deviasi LM]`) → turunan (`[Selisih]`, `[Growth]`) — resep mana pun tinggal menyambung.
- Uji setiap resep di **DAX query view** dengan `EVALUATE ROW("Hasil", [Measure])` sebelum dipasang di visual.
- Resep 7 + 9 + dynamic format strings = satu halaman analisa yang bisa "berubah wujud" via slicer.

## Terkait

- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md) — mesin di balik semua resep
- [Time Intelligence](./07-time-intelligence.md) — resep 1, 5, 6, 11
- [Iterator Functions (X)](./05-iterator-functions.md) — iterator di resep 7 & 12
- [Variabel & Error Handling](./13-variables-and-error-handling.md) — fondasi semua resep
- [Window Functions](./14-window-functions.md) — alternatif modern resep 3, 4, 5, 6
- [Format & Currency](./15-format-and-currency.md) — display resep 9
- [Performa DAX](./17-dax-performance.md) — resep 8 & 12 dari sisi biaya
- [161 Measures Asli Report](../11-user-report-analysis/model-and-dax.md) — sumber semua pattern di atas
