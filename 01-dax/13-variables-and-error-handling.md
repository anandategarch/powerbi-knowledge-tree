# Variabel (VAR/RETURN) & Error Handling

> `VAR` menyimpan hasil sebuah ekspresi **sekali**, lalu dipakai ulang sepanjang `RETURN` — kode lebih cepat, lebih rapi, dan bebas perhitungan ganda. File ini membahas scope, nesting, variabel tabel, `DIVIDE` vs operator `/`, serta strategi menangani error (`IFERROR`, `COALESCE`, TRY...CATCH).

## Konsep & Penjelasan

### Tiga sifat dasar VAR

- **Evaluasi sekali (evaluate once).** Variabel dihitung paling banyak satu kali per konteks evaluasi, lalu hasilnya di-*cache*. Measure yang direferensikan 3× tanpa `VAR` berarti 3× evaluasi; lewat `VAR` cukup 1×.
- **Lazy evaluation.** Variabel hanya dihitung **saat benar-benar dipakai** oleh jalur `RETURN` yang dieksekusi. `VAR` yang didefinisikan tapi tidak tersentuh cabang `IF`/`SWITCH` yang aktif tidak akan dievaluasi sama sekali.
- **Scope lexikal.** Variabel hidup **dari baris deklarasinya sampai akhir ekspresi `RETURN` pembungkusnya**. Measure lain, visual lain, dan calculated column tidak bisa membacanya — satu-satunya cara berbagi hasil antar measure adalah membuat measure perantara.

### Penamaan variabel

Konvensi yang dipakai report Cost Control (dan disarankan untuk measure baru):

| Gaya | Contoh | Kapan dipakai |
|---|---|---|
| PascalCase deskriptif | `Filtered`, `SelectedYear`, `SummaryTable`, `MeanValue` | Default — mudah dibaca |
| Prefiks underscore | `_Numerator`, `_Show`, `_TotalABS` | Measure panjang, menandai variabel "internal" |
| Nama pendek bermakna | `Nilai`, `EOM`, `CutoffDate` | Measure kecil |

Aturan: tanpa spasi, jangan bentrok dengan nama kolom/measure, hindari `temp`/`x` — nama variabel adalah dokumentasi termurah yang Anda miliki.

### Nesting VAR dan scope bersarang

`VAR` boleh dideklarasikan bertingkat dan **mereferensikan variabel yang dideklarasikan di atasnya** (bukan sebaliknya — DAX membaca dari atas ke bawah). Variabel di dalam scope iterator (mis. callback `SUMX`) hanya berlaku di scope tersebut, tapi boleh membaca variabel scope luar.

### Variabel tabel

`VAR` tidak terbatas skalar — bisa juga **tabel**. Ini fondasi pattern anomaly detection `[Z Score]`: tabel ringkasan dibangun sekali, lalu di-iterasi oleh `AVERAGEX` dan `STDEVX.P` tanpa dibangun ulang.

### VAR di dalam iterator = evaluasi per iterasi

Variabel yang dideklarasikan **di dalam** callback iterator (`SUMX`, `FILTER`, `AVERAGEX`) dievaluasi ulang **setiap baris**. Ini bukan bug — justru berguna agar nilai per-baris dipakai konsisten di dalam baris itu — tapi sadari biayanya: ekspresi berat di dalam iterator × jutaan baris = lambat. Kalau nilainya tidak bergantung baris, **naikkan VAR ke luar iterator** (hoisting).

### DIVIDE vs operator `/`

| Aspek | `[A] / [B]` | `DIVIDE([A], [B])` |
|---|---|---|
| Penyebut 0 / BLANK | Error (query gagal) | Mengembalikan BLANK (atau nilai alternatif) |
| Performa | Butuh jalur penanganan error | Dioptimalkan khusus oleh engine |
| Keterbacaan | Perlu `IF`/`IFERROR` tambahan | Satu baris selesai |

Aturan praktis: **selalu `DIVIDE`** untuk measure, kecuali Anda *sengaja* ingin error muncul saat debugging.

### Error handling: IFERROR, ISERROR, COALESCE

- `IFERROR(nilai, nilai_jika_error)` — menelan **semua** jenis error, termasuk bug logika Anda. Pakai sesempit mungkin.
- `ISERROR(x)` — hanya mengetes, biasanya dipadukan `IF`; hati-hati `IF(ISERROR([A]/[B]), 0, [A]/[B])` mengevaluasi pembagian **dua kali** → ganti `DIVIDE`.
- `COALESCE(a, b, c, ...)` — mengembalikan argumen pertama yang bukan BLANK; ideal untuk label/nilai default, bukan untuk menangkap error.
- `ISBLANK(x)` vs `x = BLANK()` — perbandingan `= BLANK()` menganggap `0 = BLANK()` sebagai TRUE (BLANK dikonversi ke 0), jadi nilai 0 "tertelan". Gunakan `ISBLANK` untuk presisi.
- **TRY...CATCH (DAX query view)** — panel kueri DAX terbaru mengizinkan blok `TRY ... CATCH` saat eksplorasi measure berisiko (fitur baru; ketersediaan tergantung versi Desktop — lihat contoh di bawah). Untuk measure produksi, andalkan `DIVIDE`/`COALESCE`/`IFERROR`.

## Sintaks & Contoh

### 1. Pattern dasar — measure asli [Deviasi CM]

```dax
-- Simpan kondisi sekali, gunakan di IF; cabang tidak aktif tidak dievaluasi
Deviasi CM =
VAR Filtered = ISFILTERED('Mode'[Mode])
RETURN
IF(
    Filtered,
    CALCULATE(
        [Deviasi Value],
        TREATAS(VALUES('Date A'[Date]), 'Date'[Date])
    ),
    [Deviasi Value]
)
```

### 2. VAR dipakai dua kali — measure asli [Week 1]

```dax
Week 1 =
VAR Nilai =
    CALCULATE([Deviasi CM], 'Deviasi All'[Periode] = "Week 1")
RETURN
IF(ABS(Nilai) < 0.1, BLANK(), Nilai)
-- "Nilai" dievaluasi sekali, dipakai di kondisi DAN hasil
```

### 3. Nesting VAR bertingkat — measure asli [Selected Cutoff Date]

```dax
Selected Cutoff Date =
VAR SelectedYear = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek = SELECTEDVALUE('Periode'[Periode])

VAR EOM = EOMONTH(DATE(SelectedYear, SelectedMonth, 1), 0)

VAR CutoffDate =
    SWITCH(
        SelectedWeek,
        "Week 1", DATE(SelectedYear, SelectedMonth, 7),
        "Week 2", DATE(SelectedYear, SelectedMonth, 14),
        "Week 3", DATE(SelectedYear, SelectedMonth, 25),
        "EoM", EOM,
        EOM                                -- default kalau multi-pilihan
    )
RETURN
CutoffDate
```

### 4. Variabel tabel — pattern [Z Score]

```dax
Z Score =
VAR CurrentValue = [Multi Value CM]

VAR SummaryTable =                        -- tabel: 1 baris per tanggal
    CALCULATETABLE(
        SUMMARIZE('Deviasi All', 'Date'[Date]),
        ALLSELECTED('Deviasi All')
    )

VAR MeanValue = AVERAGEX(SummaryTable, [Multi Value CM])
VAR StdDevValue = STDEVX.P(SummaryTable, [Multi Value CM])

RETURN
DIVIDE(CurrentValue - MeanValue, StdDevValue)
```

### 5. VAR per iterasi — measure asli [Nilai PI]

```dax
Nilai PI =
SUMX(
    SUMMARIZE('3243', '3243'[Nomor #], '3243'[Nama Barang]),
    VAR _PO = '3243'[Nomor #]             -- nilai baris berjalan
    VAR _Barang = '3243'[Nama Barang]
    RETURN
        CALCULATE(
            SUM('9901'[#Purch.Total]),
            REMOVEFILTERS('Date'),
            '9901'[Nomor # Pesanan Pembelian] = _PO,
            '9901'[Nama Barang] = _Barang
        )
)
```

### 6. DIVIDE & COALESCE — pengganti IFERROR

```dax
-- Hindari: IFERROR menelan semua error dan mengevaluasi ekspresi via jalur exception
Growth = IFERROR([Selisih] / [Deviasi LM], BLANK())

-- Lebih baik: DIVIDE + ABS agar growth -/- tidak terbaca positif
Growth = DIVIDE([Selisih], ABS([Deviasi LM]))

-- COALESCE untuk nilai default berantai
Label Periode =
COALESCE(SELECTEDVALUE('Periode'[Periode]), "Semua Periode")
```

### 7. TRY...CATCH di DAX query view (eksplorasi)

```dax
-- Ilustrasi sintaks DAX query view terbaru; cek ketersediaan di versi Anda
EVALUATE
TRY
    SUMMARIZECOLUMNS('Date'[Period], "Dev", [Deviasi CM])
CATCH
    ROW("Pesan", "Query gagal - periksa measure dan filter context")
```

## Kesalahan Umum

- **Mengira variabel bisa dibaca measure lain.** Tidak bisa — `VAR` hanya hidup di dalam ekspresinya. Solusi: pecah jadi beberapa measure perantara (`[Deviasi CM]`, `[Deviasi LM]`, `[Selisih]`).
- **VAR berat di dalam iterator.** Dievaluasi per baris; pindahkan ke luar iterator jika tidak bergantung baris (hoisting) — lihat [Performa DAX](./17-dax-performance.md).
- **Mengulang ekspresi di RETURN.** `[Deviasi CM] - [Deviasi LM] * 2 / [Deviasi LM]` mengevaluasi `[Deviasi LM]` dua kali → simpan di `VAR`.
- **`IF(ISERROR([A]/[B]), 0, [A]/[B])`** — evaluasi ganda + jalur error; ganti `DIVIDE([A], [B], 0)`.
- **`[M] = BLANK()` dipakai untuk cek kosong** — nilai `0` ikut terdeteksi; pakai `ISBLANK([M])`.
- **Mendeklarasikan VAR setelah dipakai** → error *cannot find name*. Urutan deklarasi harus dari atas.
- **Mengira VAR mengubah filter context.** VAR hanya snapshot nilai; memodifikasi konteks tetap tugas `CALCULATE`, `TREATAS`, `REMOVEFILTERS` — lihat [CALCULATE](./06-calculate-and-filter-functions.md).

## Tips Pro

- **Debug bertahap:** saat measure rusak, pindahkan tiap `VAR` ke `RETURN` satu per satu (atau jalankan di DAX query view) untuk melihat nilai antara mana yang salah.
- VAR membuat logika `SWITCH(TRUE())` panjang (mis. `[Analisa Deviasi]` dengan 8 cabang) tetap terbaca — beri nama variabel yang bercerita: `_CurrSign`, `_Magnitude`.
- Konsistensi nama lintas measure (`SelectedYear`, `SelectedWeek`, `_Show`) memudahkan tim merawat 161+ measure sekaligus.
- Kombinasi `VAR` + `DIVIDE` menghapus ±90% kasus IFERROR dari kode — sisanya biasanya adalah bug yang justru perlu diperbaiki, bukan disembunyikan.
- Variabel tabel + `COUNTROWS` adalah cara cepat memverifikasi "populasi" measure (mis. cek berapa baris `SummaryTable` saat tuning Z-Score).

## Terkait

- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md) — mengubah filter context setelah nilai VAR disimpan
- [Row Context vs Filter Context](./02-row-context-vs-filter-context.md) — scope & context transition
- [Iterator Functions (X)](./05-iterator-functions.md) — VAR di dalam iterator
- [Window Functions](./14-window-functions.md) — padukan VAR tabel dengan OFFSET/WINDOW
- [Format & Currency](./15-format-and-currency.md) — SELECTEDVALUE untuk format dinamis
- [Resep DAX](./16-dax-patterns-recipes.md) — hampir semua resep dibangun di atas VAR
- [Performa DAX](./17-dax-performance.md) — dampak VAR terhadap kecepatan query
- [DAX Cheatsheet](../00-quick-reference/dax-cheatsheet.md) — ringkasan fungsi & snippet
