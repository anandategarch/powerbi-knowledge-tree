# Time Intelligence Functions

> Keluarga fungsi DAX untuk "berpindah waktu" — bulan lalu (LM), tahun lalu (YoY), year-to-date (YTD) — tanpa menulis filter tanggal manual. Semuanya bertumpu pada satu fondasi: **date table yang benar**; kalau fondasinya salah, hasilnya salah secara diam-diam.

## Konsep: Time Intelligence Adalah Manipulasi Filter Tanggal

Semua fungsi time intelligence (`PREVIOUSMONTH`, `DATESYTD`, `SAMEPERIODLASTYEAR`, dst.) sebenarnya **mengembalikan tabel tanggal** yang dipakai sebagai filter argument di `CALCULATE`. Tidak ada magic: `PREVIOUSMONTH('Date'[Date])` artinya "beri saya semua tanggal bulan sebelum bulan terakhir yang sedang terlihat". Kalau error muncul, selalu reasoning-kan lewat pertanyaan: *tabel tanggal apa yang sedang dibentuk?*

### Syarat wajib sebelum memakai time intelligence

1. **Date table terpisah dan contiguous** — satu baris per hari tanpa gap, mis. `Date = CALENDARAUTO()`. Fungsi seperti `DATESYTD` bekerja pada *tanggal yang ada di tabel*, bukan tanggal yang ada datanya.
2. **Mark as date table** — setelah kolom date bertipe Date, klik kanan tabel → *Mark as date table*. Engine akan memvalidasi uniqueness dan contiguity. Matikan juga *Auto date/time* di File → Options → Data Load supaya tidak ada hidden date table per kolom tanggal.
3. **Relasi ke tabel fakta** — `Date[Date]` harus berelasi ke kolom tanggal di `Deviasi All`, `Waste All`, dan `Sales All`. Tanpa relasi, filter tanggal tidak pernah sampai ke fakta.
4. **Kolom tanggal di fakta bertipe Date** — bukan text `"2025-01-15"`. Text tidak bisa dimapping benar ke date table.

> Di report cost-control, `Date`, `Date A`, dan `Date B` semuanya `CALENDARAUTO()`. Ingat: CALENDARAUTO memindai *semua* kolom bertipe date/time di seluruh model — satu kolom tanggal liar (mis. di tabel PO) bisa melar rentang date table dan membuat label tahun ikut melar.

## Function Reference

| Function | Deskripsi | Contoh |
|---|---|---|
| PREVIOUSMONTH | Semua tanggal bulan sebelum bulan terakhir di konteks | `CALCULATE([M], PREVIOUSMONTH('Date'[Date]))` |
| SAMEPERIODLASTYEAR | Geser tanggal di konteks mundur 1 tahun | `CALCULATE([M], SAMEPERIODLASTYEAR('Date'[Date]))` |
| DATEADD | Geser tanggal di konteks per interval (day/month/quarter/year) | `DATEADD('Date'[Date], -1, MONTH)` |
| PARALLELPERIOD | Geser periode PENUH per interval | `PARALLELPERIOD('Date'[Date], -1, YEAR)` |
| TOTALYTD / TOTALQTD / TOTALMTD | Akumulasi sejak awal tahun/kuartal/bulan | `TOTALYTD([M], 'Date'[Date])` |
| DATESYTD / DATESQTD / DATESMTD | Tabel tanggal akumulasi (versi tabel dari TOTALxTD) | `CALCULATE([M], DATESYTD('Date'[Date]))` |
| DATESBETWEEN | Rentang tanggal eksplisit, inklusif dua ujung | `DATESBETWEEN('Date'[Date], d1, d2)` |
| DATESINPERIOD | N interval dari titik jangkar | `DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -3, MONTH)` |
| FIRSTDATE / LASTDATE | Tanggal pertama/terakhir di konteks (tabel 1 baris, bisa dipakai sebagai skalar) | `LASTDATE('Date'[Date])` |
| STARTOFMONTH / ENDOFMONTH | Tanggal pertama/terakhir bulan (dengan offset opsional) | `STARTOFMONTH('Date'[Date], -1)` |
| CALENDAR / CALENDARAUTO | Membuat date table | `CALENDAR(DATE(2023,1,1), DATE(2025,12,31))` |

## DATEADD vs PREVIOUSMONTH vs SAMEPERIODLASTYEAR vs PARALLELPERIOD

Empat fungsi ini paling sering tertukar. Kuncinya: **bentuk tanggal di konteks saat ini** yang menentukan hasil.

| Konteks saat ini | DATEADD(-1,MONTH) | PREVIOUSMONTH | SAMEPERIODLASTYEAR | PARALLELPERIOD(-1,MONTH) |
|---|---|---|---|---|
| 1 bulan (Mar 2025) | Feb 2025 | Feb 2025 | Mar 2024 | Feb 2025 |
| Beberapa bulan (Jan–Mar) | Des 2024–Feb 2025 (semua geser) | Feb 2025 **saja** | Jan–Mar 2024 | Des 2024–Feb 2025 |
| 1 hari (15 Mar 2025) | 15 Feb 2025 | Feb 2025 **penuh** | 15 Mar 2024 | Feb 2025 **penuh** |
| 1 tahun (2025) | 2024 penuh | **Des 2024 saja** | 2024 penuh | 2024 penuh |

- **DATEADD** mempertahankan *bentuk* konteks (3 bulan tetap 3 bulan, 1 hari tetap 1 hari).
- **PREVIOUSMONTH** selalu mengembalikan bulan penuh, dan selalu relatif ke *bulan dari tanggal terakhir* di konteks — pada konteks satu tahun penuh dia hanya mengembalikan Desember tahun lalu!
- **SAMEPERIODLASTYEAR** setara `DATEADD(..., -1, YEAR)`; hati-hati 29 Feb (tidak punya pasangan di tahun biasa → BLANK).
- **PARALLELPERIOD** seperti DATEADD tapi selalu mengembalikan **periode penuh** pada level interval-nya (konteks 15 hari + PARALLELPERIOD(-1,MONTH) = seluruh bulan lalu).

## Sintaks & Contoh

### Measure pembanding standar (pattern asli report cost-control)

```dax
-- Bulan lalu (LM = Last Month)
Deviasi LM =
CALCULATE(
    [Deviasi CM],
    PREVIOUSMONTH('Date'[Date])
)

-- Tahun lalu (YoY = Year over Year)
Deviasi YoY =
CALCULATE(
    [Deviasi CM],
    SAMEPERIODLASTYEAR('Date'[Date])
)

-- YTD sederhana: akumulasi dari 1 Januari sampai tanggal terakhir di konteks
% BOM YTD =
CALCULATE([% BOM CM], DATESYTD('Date'[Date]))

-- Growth yang aman untuk baseline negatif maupun nol
Growth =
DIVIDE([Selisih], ABS([Deviasi LM]))
-- ABS di penyebut: kalau LM = -100 dan Selisih = -50,
-- Growth = -50% (benar-benar turun setengah), bukan +50% yang menyesatkan
```

### YTD dengan cutoff mingguan (Week 1/2/3/EOM)

`DATESYTD` murni menjumlahkan *semua* minggu di bulan-bulan lalu — padahal data disimpan sebagai snapshot mingguan (WEEK 1–3) plus snapshot akhir bulan (EOM). YTD yang benar = **jumlah EOM bulan-bulan lalu + minggu terpilih bulan ini** (pattern asli [Deviasi YTD]):

```dax
Deviasi YTD =
VAR SelectedYear  = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek  = SELECTEDVALUE('Periode'[Periode])   -- "WEEK 1" .. "EOM"

-- 1. Akumulasi snapshot EOM dari Januari s/d bulan lalu
VAR SumEoMUntilLastMonth =
    CALCULATE(
        [Deviasi CM],
        ALL('Date'), ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] < SelectedMonth,
        'Periode'[Periode] = "EOM"
    )

-- 2. Bulan berjalan: hanya minggu yang sedang dipilih
VAR SumCurrentWeek =
    CALCULATE(
        [Deviasi CM],
        ALL('Date'), ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] = SelectedMonth,
        'Periode'[Periode] = SelectedWeek
    )
RETURN
    SumEoMUntilLastMonth + SumCurrentWeek
```

### DATESBETWEEN dan DATESINPERIOD

```dax
-- Rolling 3 bulan ke belakang dari tanggal terakhir yang terlihat di konteks
Deviasi Rolling 3M =
CALCULATE(
    [Deviasi CM],
    DATESINPERIOD('Date'[Date], MAX('Date'[Date]), -3, MONTH)
)

-- Rentang eksplisit (inklusif kedua ujung): semester 1 2025
Deviasi H1 =
CALCULATE(
    [Deviasi CM],
    DATESBETWEEN('Date'[Date], DATE(2025, 1, 1), DATE(2025, 6, 30))
)

-- STARTOFMONTH dengan offset: tanggal 1 bulan lalu (untuk cek stok opname)
VAR AwalBulanLalu = STARTOFMONTH('Date'[Date], -1)
```

Catatan: keduanya adalah filter argument — mereka **menggantikan** filter tanggal yang sedang aktif, bukan menambah.

### Fiscal year

```dax
-- Opsi 1: argumen string "bulan-tanggal" akhir tahun fiskal (tahun fiskal berakhir 30 Juni)
Sales FYTD =
TOTALYTD([Sales CM], 'Date'[Date], "6-30")
-- identik dengan: CALCULATE([Sales CM], DATESYTD('Date'[Date], "6-30"))

-- Opsi 2: kolom fiscal month dengan offset (tahun fiskal mulai Juli),
-- sebagai calculated column di tabel 'Date':
FiscalMonth = MONTH(EOMONTH('Date'[Date], -6))   -- Juli = 1, Juni = 12
FiscalYear  = YEAR(EOMONTH('Date'[Date], -6))
```

Opsi 1 paling murah (satu argumen). Opsi 2 diperlukan kalau user butuh slicer/hierarchy tahun fiskal sendiri.

### Show data for empty periods

Visual secara default bersifat "non-empty": bulan tanpa transaksi hilang dari baris visual. Tiga cara mengatasinya:

1. **Show items with no data** — aktifkan pada field di visual; bulan muncul dengan nilai BLANK.
2. **TREATAS + VALUES** (pattern asli report) — tabel tanggal *disconnected* seperti `Date A` menampung pilihan periode pembanding, lalu nilai terlihatnya dilempar ke `Date` utama di dalam measure:

```dax
-- Slicer 'Date A' TIDAK memfilter visual; hanya mengubah isi [Deviasi CM]
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

3. **LEFT JOIN via measure** — bangun daftar periode lengkap di dalam measure dengan `GENERATEALL` (sisi kanan boleh kosong, sisi kiri tetap dievaluasi) — detail pola di [Table Functions](./12-table-functions.md).

### CALENDAR vs CALENDARAUTO

```dax
-- CALENDARAUTO: rentang = min..max SEMUA kolom date/time di model,
-- lalu digenapkan ke batas tahun penuh (31 Des)
Date = CALENDARAUTO()

-- CALENDAR: rentang eksplisit — deterministik, disarankan untuk production
Date =
CALENDAR(
    DATE(2023, 1, 1),
    DATE(2025, 12, 31)
)
```

CALENDARAUTO praktis saat model masih berubah, tapi rentangnya "ikut data" — satu kolom tanggal anomali bisa menggeser semua label tahun dan YTD.

## Kesalahan Umum

- **Date table bergap atau memakai kolom tanggal tabel fakta** (`PREVIOUSMONTH('Deviasi All'[Tanggal])`) → hasil tidak stabil, mark as date table tidak bisa dipasang. *Perbaikan*: date table terpisah + relasi + mark as date table; time intelligence selalu merujuk `'Date'[Date]`.
- **PREVIOUSMONTH pada konteks beberapa bulan / setahun** → hanya mengembalikan bulan sebelum bulan terakhir (Desember tahun lalu), sehingga angka setahun "berbanding" hanya dengan Desember. *Perbaikan*: pakai `DATEADD('Date'[Date], -1, MONTH)` agar semua bulan ikut bergeser.
- **Time intelligence di calculated column** → calculated column tidak punya filter context per baris; `PREVIOUSMONTH` mengembalikan hasil yang sama untuk semua baris. *Perbaikan*: pindahkan ke measure.
- **Lupa mark as date table / biarkan auto date/time aktif** → engine memakai hidden date table per kolom, hasil bisa berbeda antar visual. *Perbaikan*: matikan auto date/time, tandai date table.
- **SAMEPERIODLASTYEAR pada 29 Februari** → tidak ada 29 Feb tahun biasa, hasil BLANK. *Perbaikan*: terima BLANK atau geser manual dengan DATEADD plus mapping khusus.
- **YTD "terpotong"** karena tanggal terakhir date table = tanggal data terakhir. *Perbaikan*: pastikan date table menutup sampai akhir tahun (CALENDARAUTO membulatkan ke tahun penuh; CALENDAR eksplisit lebih terkontrol).

## Tips Pro

- **Pattern "base measure + CALCULATE + fungsi waktu"**: tulis [Deviasi CM] sekali, lalu [Deviasi LM], [Deviasi YoY], [% BOM YTD] masing-masing cukup satu baris CALCULATE. Konsisten, mudah diaudit, tidak ada logika duplikat.
- **Growth selalu `DIVIDE(x, ABS(baseline))`** — aman dari pembagian nol DAN tidak membalik tanda saat baseline negatif (deviasi sering negatif!).
- **Fiscal year cukup satu argumen** `"6-30"` di DATESYTD/TOTALYTD kalau tidak butuh slicer tahun fiskal; tambahkan kolom offset kalau user perlu memilih fiscal year secara eksplisit.
- **Performance**: fungsi time intelligence dioptimalkan di storage engine. Hindari `FILTER(ALL('Date'), 'Date'[Date] >= ... && 'Date'[Date] <= ...)` sebagai pengganti — itu formula engine, jauh lebih lambat.
- **Kolom sort** seperti `Periode[Sort]` (1–4) wajib di-set sebagai *Sort By Column* agar WEEK 1/2/3/EOM tidak terurut alfabetis (EOM naik duluan).
- Uji setiap measure pembanding di visual bulanan dan satu kartu tunggal sebelum dipercaya — sebagian besar bug time intelligence tidak terlihat di level total.

## Terkait

- [CALCULATE & Filter](./06-calculate-and-filter-functions.md) — dasar filter context yang dipakai semua fungsi di atas
- [Relational Functions](./08-relational-functions.md) — TREATAS + VALUES untuk date table disconnected
- [Information Functions](./10-information-functions.md) — ISFILTERED/SELECTEDVALUE pada pattern Mode DINAMIS
- [Window Functions](./14-window-functions.md) — alternatif modern (OFFSET/INDEX/WINDOW) untuk perbandingan periode
- [Data Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md) — daftar lengkap measure time intelligence di report
