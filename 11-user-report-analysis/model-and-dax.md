# Analisis Data Model & DAX — Report Cost Control

> Di-generate otomatis dari DataModel PBIX (via pbixray).
> Total **161 measures**, **109 calculated columns**, **21 calculated tables/DAX tables**.

## Tabel (Urutan Query Power Query)

### Tabel inti

- **Inventory**
- **Deviasi All**
- **Waste All**
- **Sales All**
- **Resto**
- **Standar Susut**
- **Inventory A**
- **Inventory B**
- **Resto A**
- **Resto B**
- **3224**
- **4205**
- **Rekap Mentah BM**
- **Update**
- **9901**
- **3243**

### Helper query (internal Power Query)



## Relasi (Relationships)

| Dari (many) | Kolom | Ke (one) | Kolom | Kardinalitas |
|---|---|---|---|---|
| Waste All | Nama  new item  | Inventory | (Old) Nama Barang | M:1 |
| Standar Susut | Nama Barang | Inventory | (Old) Nama Barang | M:1 |
| Deviasi All | Nama Barang | Inventory | (Old) Nama Barang | M:1 |
| 3224 | Nama Barang | Inventory | (Old) Nama Barang | M:1 |
| 4205 | Nama Barang | Inventory | (Old) Nama Barang | M:1 |
| Rekap Mentah BM | NAMA BARANG | Inventory | (Old) Nama Barang | M:1 |
| 9901 | Nama Barang | Inventory | (Old) Nama Barang | M:1 |
| 3243 | Nama Barang | Inventory | (Old) Nama Barang | M:1 |
| 3243 | Cabang PO | Resto | Resto | M:M |
| Rekap Mentah BM | NAMA RESTO | Resto | Resto | M:M |
| 9901 | Nama Cabang | Resto | Resto | M:M |
| 4205 | Cabang | Resto | Resto | M:M |
| 3224 | Nama Cabang Penerimaan Barang | Resto | Resto | M:M |
| Standar Susut | Type Kitchen | Resto | Type Kitchen | M:M |
| Sales All | Nama Cabang | Resto | Resto | M:M |
| Waste All | CABANG | Resto | Resto | M:M |
| Deviasi All | Resto | Resto | Resto | M:M |

## Calculated Tables (DAX)

### Date

```dax
Date = CALENDARAUTO()
```

### Trend

```dax
Trend = DATATABLE(
    "Kategori", STRING,
    {
        {"DEVIASI"},
        {"BOM"},
        {"% BOM"},
        {"% SALES"},
        {"VARIANCE"}
    }
)
```

### Attribute

```dax
Attribute = DATATABLE(
    "Attribute", STRING,
    {
        {"NOMINAL"},
        {"QTY"}
    }
)
```

### Periode

```dax
Periode = DATATABLE(
    "Periode", STRING,
    "Sort", INTEGER,
    {
        {"WEEK 1", 1},
        {"WEEK 2", 2},
        {"WEEK 3", 3},
        {"EOM", 4}
    }
)
```

### Mode

```dax
Mode = DATATABLE(
    "Mode", STRING,
    {
        {"DINAMIS"}
    }
)
```

### Date A

```dax
Date A = CALENDARAUTO()
```

### Date B

```dax
Date B = CALENDARAUTO()
```

## Calculated Columns Penting (di luar auto date/time)

### Deviasi All

- **Tipe** = `SWITCH(
    'Deviasi All'[Akun Penyesuaian Persediaan],
    "COM DEVIASI - RESTO", "RAW MATERIAL",
    "Biaya Packaging - RESTO", "PACKAGING",
    "COM CONSUME - RESTO", "CONSUME",
    BLANK()
)`
- **Cek Resto** = `LOOKUPVALUE(Resto[Resto],Resto[Resto],'Deviasi All'[RESTO])`
### Waste All

- **Tipe** = `SWITCH(
    'Waste All'[KATEGORI],
    "COM DEVIASI - RESTO", "DEVIASI",
    "Biaya Packaging - RESTO", "PACKAGING",
    "COM CONSUME - RESTO", "CONSUME",
    BLANK()
)`
### Date

- **Period** = `FORMAT('Date'[Date],"yyyy-mm")`
### Deviasi All

- **Cek Barang** = `LOOKUPVALUE(Inventory[Nama Barang],Inventory[(Old) Nama Barang],'Deviasi All'[Nama Barang])`
### Sales All

- **Date** = `DATE(VALUE(MID('Sales All'[Source.Name],7,4)),VALUE(MID('Sales All'[Source.Name],12,2)),1)`
### Resto

- **Tipe** = `VAR Code = LEFT('Resto'[Nama],1)
RETURN
SWITCH(
    Code,
    "1", "Mie Gacoan",
    "B", "Bakso Gacoan",
    "H", "HO",
    "Non Resto"
)`
### Date

- **Year** = `YEAR('Date'[Date])`
- **MonthNumber** = `MONTH('Date'[Date])`
### Deviasi All

- **Kode** = `RIGHT('Deviasi All'[RESTO],6)`
### Sales All

- **Kode** = `RIGHT('Sales All'[Nama Cabang],6)`
### Deviasi All

- **Group Satuan** = `SWITCH(
    TRUE(),
    'Deviasi All'[Satuan] IN {"GR", "ML"}, "GR & ML",
    'Deviasi All'[Satuan] IN {"BOTOL", "PACK", "KG"}, "BOTOL, PACK & KG",
    'Deviasi All'[Satuan] IN {"PCS"}, "PCS",
    "La`
### 4205

- **Cabang** = `VAR _Gudang = '4205'[Gudang #Terima]

VAR _KodeCabang =
    IF(
        LEFT(_Gudang, 1) = "B",
        LEFT(
            _Gudang,
            SEARCH(".1-G", _Gudang, 1, LEN(_Gudang)) - 1
        ),
 `
### Rekap Mentah BM

- **TANGGAL** = `VAR Tahun = VALUE(MID('Rekap Mentah BM'[Source.Name],7,4))
VAR Bulan = VALUE(MID('Rekap Mentah BM'[Source.Name],12,2))
RETURN
DATE(Tahun,Bulan,'Rekap Mentah BM'[DAY])`
### Date

- **Periode Cut Off** = `VAR Tgl = 'Date'[Date]

VAR TglAwal =
    IF(
        DAY(Tgl) >= 26,
        DATE(YEAR(Tgl), MONTH(Tgl), 26),
        DATE(YEAR(EDATE(Tgl,-1)), MONTH(EDATE(Tgl,-1)), 26)
    )

VAR TglAkhir =
    EDA`
### Inventory

- **Nama Barang** = `SWITCH(Inventory[(Old) Nama Barang],"KRUPUK MIE (V.30)","KRUPUK MIE (V.40)",Inventory[(Old) Nama Barang])`
### Date A

- **Period** = `FORMAT('Date A'[Date],"yyyy-mm")`
### Date B

- **Period** = `FORMAT('Date B'[Date],"yyyy-mm")`
### Deviasi All

- **Cek Jenis** = `LOOKUPVALUE(Inventory[Tipe],Inventory[Nama Barang],'Deviasi All'[Nama Barang])`
### Waste All

- **Cek Barang** = `LOOKUPVALUE(Inventory[Nama Barang],Inventory[Nama Barang],'Waste All'[Nama  new item ])`
### 3243

- **Jenis** = `VAR _NamaBarang = '3243'[Nama Barang]
RETURN
CALCULATE(
    MAX('9901'[Jenis Barang Barang & Jasa]),
    FILTER(
        '9901',
        '9901'[Nama Barang] = _NamaBarang
    )
)`
### 3224

- **Cek Barang** = `LOOKUPVALUE(Inventory[Nama Barang],Inventory[(Old) Nama Barang],'3224'[Nama Barang])`
### 4205

- **Cek Barang** = `LOOKUPVALUE(Inventory[Nama Barang],Inventory[(Old) Nama Barang],'4205'[Nama Barang])`
### 9901

- **Cek Barang** = `LOOKUPVALUE(Inventory[Nama Barang],Inventory[(Old) Nama Barang],'9901'[Nama Barang])`
### 3243

- **Cek Barang** = `LOOKUPVALUE(Inventory[Nama Barang],Inventory[(Old) Nama Barang],'3243'[Nama Barang])`

## Semua Measures (DAX Lengkap)

<details><summary><b>Klik untuk membuka 161 measures lengkap</b></summary>

### Deviasi LM

```dax
VAR Filtered = ISFILTERED('Mode'[Mode])

RETURN
IF(
    Filtered,
    CALCULATE(
        [Deviasi Value],
        TREATAS(
            VALUES('Date B'[Date]),
            'Date'[Date]
        )
    ),
    CALCULATE(
        [Deviasi Value],
        PREVIOUSMONTH('Date'[Date])
    )
)
```

### Growth

```dax
DIVIDE([Selisih],ABS([Deviasi LM]))
```

### Selisih

```dax
[Deviasi CM]-[Deviasi LM]
```

### Deviasi CM

```dax
VAR Filtered = ISFILTERED('Mode'[Mode])

RETURN
IF(
    Filtered,
    CALCULATE(
        [Deviasi Value],
        TREATAS(
            VALUES('Date A'[Date]),
            'Date'[Date]
        )
    ),
    [Deviasi Value]
)
```

### ABS LM

```dax
CALCULATE([ABS Deviasi],PREVIOUSMONTH('Date'[Date]))
```

### Nominal Dev

```dax
CALCULATE([Top10Dev],Attribute[Attribute]="NOMINAL")
```

### Kategori

```dax
IF([Deviasi CM] >= 0, "Surplus", "Loss")
```

### BOM CM

```dax
VAR Filtered = ISFILTERED('Mode'[Mode])

RETURN
IF(
    Filtered,
    CALCULATE(
        ABS(SUM('Deviasi All'[Value])),
        'Deviasi All'[Jenis] = "BOM",
        TREATAS(
            VALUES('Date A'[Date]),
            'Date'[Date]
        )
    ),
    CALCULATE(
        ABS(SUM('Deviasi All'[Value])),
        'Deviasi All'[Jenis] = "BOM"
    )
)
```

### % BOM CM

```dax
VAR AvgSusut =
    AVERAGE('Standar Susut'[Standar Susut])

VAR IsKhusus =
    SELECTEDVALUE(Inventory[Nama Barang]) IN {
        "AYAM CINCANG (V.20)",
        "CABE RAWIT - RESTO (V.20)",
        "CABAI FROZEN"
    }

RETURN
IF(
    IsKhusus,
    DIVIDE([Deviasi CM] * (1 - AvgSusut), [BOM CM]),
    DIVIDE([Deviasi CM], [BOM CM])
)
```

### % BOM LM

```dax
CALCULATE([% BOM CM],PREVIOUSMONTH('Date'[Date]))
```

### Growth % BOM

```dax
DIVIDE([% BOM GAP],ABS([% BOM LM]))
```

### % BOM GAP

```dax
[% BOM CM]-[% BOM LM]
```

### Multi Value CM

```dax
VAR SalesCM = [% Sales CM]
VAR HasSalesCM = NOT ISBLANK(SalesCM)

RETURN
SWITCH(
    SELECTEDVALUE('Trend'[Kategori]),

    "DEVIASI", [Deviasi CM],
    "BOM", [BOM CM],
    "% BOM", [% BOM CM],

    "% SALES",
        IF(
            SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL",
            SalesCM,
            BLANK()
        ),

    "VARIANCE",
        IF(
            SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL"
                && HasSalesCM,
            [% Sales GAP],
            BLANK()
        ),

    BLANK()
)
```

### Multi Value LM

```dax
CALCULATE([Multi Value CM],PREVIOUSMONTH('Date'[Date]))
```

### Multi Value Gap

```dax
[Multi Value CM]-[Multi Value LM]
```

### Multi Value Growth

```dax
DIVIDE([Multi Value Gap],ABS([Multi Value LM]))
```

### ABS Deviasi

```dax
ABS([Deviasi CM])
```

### Z Score

```dax
VAR CurrentValue = [Multi Value CM]

VAR SummaryTable = 
    CALCULATETABLE(
        SUMMARIZE(
            'Deviasi All',
            'Date'[Date]
        ),
        ALLSELECTED('Deviasi All')
    )

VAR MeanValue = 
    AVERAGEX(
        SummaryTable,
        [Multi Value CM]
    )

VAR StdDevValue = 
    STDEVX.P(
        SummaryTable,
        [Multi Value CM]
    )

RETURN
DIVIDE(
    CurrentValue - MeanValue,
    StdDevValue
)
```

### Qty Waste

```dax
FORMAT(
    CALCULATE(
        SUM('Waste All'[Value]),
        Attribute[Attribute] = "QTY"
    ),
    "#,##0"
)
& " " &
MAX('Waste All'[SATUAN])
```

### Kategori Anomali Wasutri

```dax
SWITCH(
    TRUE(),
    ABS([Z Score]) > 1, "Anomali",
    "Normal"
)
```

### Deviasi YTD

```dax
VAR SelectedYear = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek = SELECTEDVALUE('Periode'[Periode])

-- 1. Hitung Akumulasi EoM dari Januari s/d Bulan Lalu
VAR SumEoMUntilLastMonth = 
    CALCULATE(
        [Deviasi CM],
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] < SelectedMonth,
        'Periode'[Periode] = "EoM"
    )

-- 2. Hitung Deviasi untuk Bulan Berjalan sesuai Week yang dipilih
VAR SumCurrentWeek = 
    CALCULATE(
        [Deviasi CM],
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] = SelectedMonth,
        'Periode'[Periode] = SelectedWeek
    )

RETURN
SumEoMUntilLastMonth + SumCurrentWeek
```

### Selisih Abs

```dax
ABS([Selisih])
```

### % BOM CM Abs

```dax
ABS([% BOM CM])
```

### Waste LM

```dax
CALCULATE(SUM('Waste All'[Value]),PREVIOUSMONTH('Date'[Date]))
```

### Waste Gap

```dax
SUM('Waste All'[Value])-[Waste LM]
```

### Waste Growth

```dax
DIVIDE([Waste Gap],ABS([Waste LM]))
```

### Waste YTD

```dax
VAR SelectedYear = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek = SELECTEDVALUE('Periode'[Periode])

-- 1. Hitung Akumulasi EoM dari Januari s/d Bulan Lalu
VAR SumEoMUntilLastMonth = 
    CALCULATE(
        SUM('Waste All'[Value]),
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] < SelectedMonth,
        'Periode'[Periode] = "EoM"
    )

-- 2. Hitung Deviasi untuk Bulan Berjalan sesuai Week yang dipilih
VAR SumCurrentWeek = 
    CALCULATE(
        SUM('Waste All'[Value]),
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] = SelectedMonth,
        'Periode'[Periode] = SelectedWeek
    )

RETURN
SumEoMUntilLastMonth + SumCurrentWeek
```

### % Sales CM

```dax
DIVIDE([Deviasi CM],[Sales CM])
```

### Sales LM

```dax
CALCULATE([Sales CM],PREVIOUSMONTH('Date'[Date]))
```

### % Sales LM

```dax
DIVIDE([Deviasi LM],[Sales LM])
```

### Sales YTD

```dax
VAR SelectedYear = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek = SELECTEDVALUE('Periode'[Periode])

-- 1. Hitung Akumulasi EoM dari Januari s/d Bulan Lalu
VAR SumEoMUntilLastMonth = 
    CALCULATE(
        [Sales CM],
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] < SelectedMonth,
        'Periode'[Periode] = "EoM"
    )

-- 2. Hitung Deviasi untuk Bulan Berjalan sesuai Week yang dipilih
VAR SumCurrentWeek = 
    CALCULATE(
        [Sales CM],
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] = SelectedMonth,
        'Periode'[Periode] = SelectedWeek
    )

RETURN
SumEoMUntilLastMonth + SumCurrentWeek
```

### % Sales YTD

```dax
DIVIDE([Deviasi YTD],[Sales YTD])
```

### Growth YoY

```dax
DIVIDE([Selisih YoY],ABS([Deviasi YoY]))
```

### Sales CM

```dax
IF(
    SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL",
    SUM('Sales All'[Penjualan Bersih]),
    BLANK()
)
```

### Deviasi YoY

```dax
CALCULATE([Deviasi CM], SAMEPERIODLASTYEAR('Date'[Date]))
```

### Avg Dev

```dax
DIVIDE([Deviasi CM],DISTINCTCOUNT('Deviasi All'[RESTO]))
```

### Resto YoY

```dax
CALCULATE(DISTINCTCOUNT('Deviasi All'[RESTO]),SAMEPERIODLASTYEAR('Date'[Date]))
```

### BOM LM

```dax
VAR Filtered = ISFILTERED('Mode'[Mode])

RETURN
IF(
    Filtered,
    CALCULATE(
        ABS(SUM('Deviasi All'[Value])),
        KEEPFILTERS(
            'Deviasi All'[Jenis] = "BOM"
        ),
        TREATAS(
            VALUES('Date B'[Date]),
            'Date'[Date]
        )
    ),
    CALCULATE(
        ABS(SUM('Deviasi All'[Value])),
        KEEPFILTERS(
            'Deviasi All'[Jenis] = "BOM"
        ),
        PREVIOUSMONTH('Date'[Date])
    )
)
```

### BOM YTD

```dax
VAR SelectedYear = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek = SELECTEDVALUE('Periode'[Periode])

-- 1. Hitung Akumulasi EoM dari Januari s/d Bulan Lalu
VAR SumEoMUntilLastMonth = 
    CALCULATE(
        [BOM CM],
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] < SelectedMonth,
        'Periode'[Periode] = "EoM"
    )

-- 2. Hitung Deviasi untuk Bulan Berjalan sesuai Week yang dipilih
VAR SumCurrentWeek = 
    CALCULATE(
        [BOM CM],
        ALL('Date'),
        ALL('Periode'),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] = SelectedMonth,
        'Periode'[Periode] = SelectedWeek
    )

RETURN
SumEoMUntilLastMonth + SumCurrentWeek
```

### % BOM YTD

```dax
CALCULATE([% BOM CM],DATESYTD('Date'[Date]))
```

### Week 1

```dax
VAR Nilai =
    CALCULATE(
        [Deviasi CM],
        'Deviasi All'[Periode] = "Week 1"
    )
RETURN
IF(
    ABS(Nilai) < 0.1,
    BLANK(),
    Nilai
)
```

### Week 2

```dax
VAR Nilai =
    CALCULATE(
        [Deviasi CM],
        'Deviasi All'[Periode] = "Week 2"
    )
RETURN
IF(
    ABS(Nilai) < 0.1,
    BLANK(),
    Nilai
)
```

### EoM

```dax
VAR Nilai =
    CALCULATE(
        [Deviasi CM],
        'Deviasi All'[Periode] = "EoM"
    )
RETURN
IF(
    ABS(Nilai) < 0.1,
    BLANK(),
    Nilai
)
```

### Growth W2

```dax
IF(
    OR(
        ISBLANK([Week 2]),
        ISBLANK([Week 1])
    ),
    BLANK(),
    DIVIDE(
        [Week 2] - [Week 1],
        ABS([Week 1])
    )
)
```

### Growth EOM

```dax
IF(
    OR(
        ISBLANK([EoM]),
        ISBLANK([Week 2])
    ),
    BLANK(),
    DIVIDE(
        [EoM] - [Week 2],
        ABS([Week 2])
    )
)
```

### Selected Cutoff Date

```dax
VAR SelectedYear = SELECTEDVALUE('Date'[Year])
VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber])
VAR SelectedWeek = SELECTEDVALUE('Periode'[Periode])

VAR EOM =
    EOMONTH(DATE(SelectedYear, SelectedMonth, 1), 0)

VAR CutoffDate =
    SWITCH(
        SelectedWeek,
        "Week 1", DATE(SelectedYear, SelectedMonth, 7),
        "Week 2", DATE(SelectedYear, SelectedMonth, 14),
        "Week 3", DATE(SelectedYear, SelectedMonth, 25),
        "EoM", EOM,
        EOM
    )

RETURN
CutoffDate
```

### % Sales GAP

```dax
[% Sales CM]-[% Sales LM]
```

### Loss/Surplus

```dax
ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="LOSS/SURPLUS"))
```

### Susut

```dax
ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="SUSUT"))
```

### Trial

```dax
ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="TRIAL"))
```

### Waste

```dax
ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="WASTE"))
```

### % Waste

```dax
DIVIDE([Waste],[Deviasi Total])
```

### % Susut

```dax
DIVIDE([Susut],[Deviasi Total])
```

### % Trial

```dax
DIVIDE([Trial],[Deviasi Total])
```

### % Loss/Suprlus

```dax
DIVIDE([Loss/Surplus],[Deviasi Total])
```

### Analisa Susut

```dax
SWITCH(
    TRUE(),
    ISBLANK([% BOM CM Abs]), BLANK(),
    [% BOM CM Abs] < AVERAGE('Standar Susut'[Standar Susut]), "Under Standard",
    [% BOM CM Abs] = AVERAGE('Standar Susut'[Standar Susut]), "Good",
    [% BOM CM Abs] > AVERAGE('Standar Susut'[Standar Susut]), "Over Standard"
)
```

### A Deviasi

```dax
CALCULATE(
    [Deviasi CM],
    TREATAS(
        VALUES('Inventory A'[(Old) Nama Barang]),
        'Deviasi All'[Nama Barang]
    ),
     TREATAS(
        VALUES('Resto A'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### A BOM

```dax
CALCULATE(
    [BOM CM],
    TREATAS(
        VALUES('Inventory A'[(Old) Nama Barang]),
        'Deviasi All'[Nama Barang]
    ),
     TREATAS(
        VALUES('Resto A'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### A % BOM

```dax
CALCULATE(
    [% BOM CM],
    TREATAS(
        VALUES('Inventory A'[(Old) Nama Barang]),
        'Deviasi All'[Nama Barang]
    ),
     TREATAS(
        VALUES('Resto A'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### B % BOM

```dax
CALCULATE(
    [% BOM CM],
    TREATAS(
        VALUES('Inventory B'[(Old) Nama Barang]),
        'Deviasi All'[Nama Barang]
    ),
     TREATAS(
        VALUES('Resto B'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### B BOM

```dax
CALCULATE(
    [BOM CM],
    TREATAS(
        VALUES('Inventory B'[(Old) Nama Barang]),
        'Deviasi All'[Nama Barang]
    ),
     TREATAS(
        VALUES('Resto B'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### B Deviasi

```dax
CALCULATE(
    [Deviasi CM],
    TREATAS(
        VALUES('Inventory B'[(Old) Nama Barang]),
        'Deviasi All'[Nama Barang]
    ),
     TREATAS(
        VALUES('Resto B'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### GAP Deviasi

```dax
[A Deviasi]-[B Deviasi]
```

### GAP BOM

```dax
[A BOM]-[B BOM]
```

### GAP % BOM

```dax
[A % BOM]-[B % BOM]
```

### GAP ABS Dev

```dax
ABS([A Deviasi])-ABS([B Deviasi])
```

### Kategori L/S

```dax
VAR LS = CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="LOSS/SURPLUS")
RETURN
IF(LS>=0,"Surplus","Loss")
```

### Selisih YoY

```dax
[Deviasi CM]-[Deviasi YoY]
```

### Deviasi Total

```dax
[Waste]+[Susut]+[Trial]+[Loss/Surplus]
```

### % Deviasi

```dax
[% Waste]+[% Susut]+[% Trial]+[% Loss/Suprlus]
```

### QTY Dev

```dax
FORMAT(
    CALCULATE(
        [Top10Dev],
        Attribute[Attribute] = "QTY"
    ),
    "#,##0"
)
& " " &
MAX('Deviasi All'[Satuan])
```

### Nominal Waste

```dax
CALCULATE(SUM('Waste All'[Value]),Attribute[Attribute]="NOMINAL")
```

### Kontribusi CM

```dax
IF(
    HASONEVALUE('Deviasi All'[Jenis]),
    SWITCH(
        SELECTEDVALUE('Deviasi All'[Jenis]),
        "WASTE", [% Waste],
        "SUSUT", [% Susut],
        "TRIAL", [% Trial],
        "USAGE", [% Usage],
        "LOSS/SURPLUS", [% Loss/Suprlus],
        BLANK()
    ),
    SUMX(
        VALUES('Deviasi All'[Jenis]),
        SWITCH(
            'Deviasi All'[Jenis],
            "WASTE", [% Waste],
            "SUSUT", [% Susut],
            "TRIAL", [% Trial],
            "USAGE", [% Usage],
            "LOSS/SURPLUS", [% Loss/Suprlus]
        )
    )
)
```

### Kontribusi LM

```dax
CALCULATE([Kontribusi CM],PREVIOUSMONTH('Date'[Date]))
```

### Kontribusi GAP

```dax
[Kontribusi CM]-[Kontribusi LM]
```

### Usage

```dax
ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="USAGE"))
```

### % Usage

```dax
DIVIDE([Usage],[Deviasi Total])
```

### Show

```dax
VAR Nilai =
[Deviasi CM]+[BOM CM]+
[Deviasi LM]+[BOM LM]+
[Deviasi YTD]+[BOM YTD]+
[Deviasi YoY]+[BOM YoY]

RETURN
IF(ABS(Nilai)<0,"Hide","Show")
```

### BOM YoY

```dax
CALCULATE([BOM CM], SAMEPERIODLASTYEAR('Date'[Date]))
```

### % BOM YoY

```dax
CALCULATE([% BOM CM], SAMEPERIODLASTYEAR('Date'[Date]))
```

### Sales YoY

```dax
CALCULATE([Sales CM], SAMEPERIODLASTYEAR('Date'[Date]))
```

### % Sales YoY

```dax
DIVIDE([Deviasi YoY],[Sales YoY])
```

### A Sales

```dax
CALCULATE(
    [Sales CM],
     TREATAS(
        VALUES('Resto A'[Resto]),
        'Sales All'[Nama Cabang]
    )
)
```

### B Sales

```dax
CALCULATE(
    [Sales CM],
     TREATAS(
        VALUES('Resto B'[Resto]),
        'Sales All'[Nama Cabang]
    )
)
```

### A % Sales

```dax
CALCULATE(
    [% Sales CM],
     TREATAS(
        VALUES('Resto A'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### B % Sales

```dax
CALCULATE(
    [% Sales CM],
     TREATAS(
        VALUES('Resto B'[Resto]),
        'Deviasi All'[Resto]
    )
)
```

### GAP Sales

```dax
[A Sales]-[B Sales]
```

### GAP % Sales

```dax
[A % Sales]-[B % Sales]
```

### Week 1 Abs

```dax
ABS([Week 1])
```

### Week 2 Abs

```dax
ABS([Week 2])
```

### EoM Abs

```dax
ABS([EoM])
```

### Sales GAP

```dax
[Sales CM]-[Sales LM]
```

### BOM GAP

```dax
[BOM CM]-[BOM LM]
```

### BOM ABS

```dax
ABS([BOM CM])
```

### Kontribusi Waste

```dax
DIVIDE(
    [Nominal Waste],
    CALCULATE(
        [Nominal Waste],
        REMOVEFILTERS('Waste All'[Jenis])
    )
)
```

### Dynamic Gap Dev

```dax
SWITCH(
        SELECTEDVALUE(Attribute[Attribute]),
        "QTY", FORMAT([Selisih], "#,##0") & " " & MAX('Deviasi All'[Satuan]),
        "NOMINAL", 
            IF(
                [Selisih] < 0,
                "-Rp" & FORMAT(ABS([Selisih]), "#,##0"),
                "Rp" & FORMAT([Selisih], "#,##0")
            ),
        BLANK()
    )
```

### % Sales Growth

```dax
DIVIDE([Sales GAP],ABS([Sales LM]))
```

### % GT Deviasi

```dax
VAR Total = CALCULATE([Nominal Dev],ALL(Inventory[Nama Barang]))
RETURN
DIVIDE([Nominal Dev],Total)
```

### % to Subtotal

```dax
VAR Total = CALCULATE([Nominal Waste],ALL('Inventory'[Nama Barang]))
RETURN
DIVIDE([Nominal Waste],Total)
```

### % to Total

```dax
VAR Total = CALCULATE([Nominal Waste],ALL('Inventory'[Nama Barang]),ALL('Waste All'[Jenis]))
RETURN
DIVIDE([Nominal Waste],Total)
```

### Perubahan

```dax
DIVIDE(ABS([Deviasi CM])-ABS([Deviasi LM]),ABS([Deviasi LM]))
```

### Perubahan EOM

```dax
IF(
    ISBLANK([EoM]),
    BLANK(),
    DIVIDE(
        ABS([EoM]) - ABS([Week 2]),
        ABS([Week 2])
    )
)
```

### Perubahan W2

```dax
IF(
    ISBLANK([Week 2]),
    BLANK(),
    DIVIDE(
        ABS([Week 2]) - ABS([Week 1]),
        ABS([Week 1])
    )
)
```

### Perubahan Multi Value

```dax
DIVIDE(ABS([Multi Value CM])-ABS([Multi Value LM]),ABS([Multi Value LM]))
```

### Dynamic Dev

```dax
SWITCH(
        SELECTEDVALUE(Attribute[Attribute]),
        "QTY", FORMAT([Deviasi CM], "#,##0") & " " & MAX('Deviasi All'[Satuan]),
        "NOMINAL", 
            IF(
                [Deviasi CM] < 0,
                "-Rp" & FORMAT(ABS([Deviasi CM]), "#,##0"),
                "Rp" & FORMAT([Deviasi CM], "#,##0")
            ),
        BLANK()
    )
```

### Top10Dev

```dax
VAR JenisDev = SELECTEDVALUE('Deviasi All'[Jenis])
RETURN
IF(
    JenisDev IN {"Waste","Susut","Trial"},
    [ABS Deviasi],
    [Deviasi CM]
)
```

### Selisih BM

```dax
VAR Sistem = SUM('3224'[Kts (Unit#1)])+SUM('4205'[#Qty. Terkecil])+[Pembelian di Luar]
VAR Inputan = SUM('Rekap Mentah BM'[Value])
RETURN
Inputan-Sistem
```

### Clean BM

```dax
SUM('3224'[Kts (Unit#1)])+SUM('4205'[#Qty. Terkecil])+[Pembelian di Luar]+SUM('Rekap Mentah BM'[Value])
```

### Last Update Dev

```dax
VAR Last =
    MAX ( 'Deviasi All'[Tanggal] )

VAR LastPeriodRank =
    CALCULATE (
        MAXX (
            'Deviasi All',
            SWITCH (
                'Deviasi All'[Periode],
                "Week 1", 1,
                "Week 2", 2,
                "Week 3", 3,
                "EoM", 4,
                0
            )
        ),
        'Deviasi All'[Tanggal] = Last
    )

VAR LastPeriod =
    SWITCH (
        LastPeriodRank,
        1, "Week 1",
        2, "Week 2",
        3, "Week 3",
        4, "EoM"
    )

RETURN
    "Last Update : "
        & FORMAT(Last, "mmmm yyyy")
        & " - "
        & LastPeriod
```

### Update on Dev

```dax
"Updated on : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Update on Deviasi"),"dd mmmm yyyy")
```

### Update on BM

```dax
"Updated on : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Update on Barang Masuk"),"dd mmmm yyyy")
```

### Last Update BM

```dax
"Last Update : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Last Update Barang Masuk"),"dd mmmm yyyy")
```

### Pembelian di Luar

```dax
CALCULATE(
    SUM('9901'[#Prime.Qty]),
    ISBLANK('9901'[Nomor # Penerimaan Barang]),
    '9901'[Jenis Barang Barang & Jasa] = "Persediaan"
)
```

### % Dev Contribution

```dax
// Pembilang: menjumlahkan ABS Deviasi sesuai konteks baris matriks yang aktif
VAR _Numerator = 
    SUMX(
        CROSSJOIN(
            VALUES('Resto'[Resto]),
            VALUES('Inventory'[Nama Barang])
        ),
        [ABS Deviasi]
    )

// Penyebut: total seluruh ABS Deviasi dari filter/slicer yang terpilih
VAR _TotalABS = 
    SUMX(
        CROSSJOIN(
            ALLSELECTED('Resto'[Resto]),
            ALLSELECTED('Inventory'[Nama Barang])
        ),
        [ABS Deviasi]
    )

RETURN
    DIVIDE(
        _Numerator,
        _TotalABS,
        0
    )
```

### Analisa Deviasi

```dax
VAR _Show = [Show]

VAR _Curr = [Deviasi CM]
VAR _Last = [Deviasi LM]

VAR _Tolerance = 0.01

VAR _AbsCurr = ABS(_Curr)
VAR _AbsLast = ABS(_Last)

VAR _CurrSign = SIGN(_Curr)
VAR _LastSign = SIGN(_Last)

VAR _Magnitude =
    SWITCH(
        TRUE(),
        ABS(_AbsCurr - _AbsLast) <= _Tolerance, "tetap",
        _AbsCurr > _AbsLast, "membesar",
        "mengecil"
    )

RETURN
IF(
    _Show = "Hide",
    BLANK(),

    SWITCH(
        TRUE(),

        -- Tidak ada data bulan ini
        ISBLANK(_Curr),
            "Tidak ada deviasi",

        -- Bulan lalu tidak ada data
        ISBLANK(_Last),
            SWITCH(
                _CurrSign,
                1, "Surplus",
                -1, "Loss",
                "Tidak ada deviasi"
            ),

        -- Keduanya netral
        _CurrSign = 0 && _LastSign = 0,
            "Tidak ada deviasi",

        -- Bulan ini netral
        _CurrSign = 0,
            IF(
                _LastSign > 0,
                "Surplus mengecil",
                "Loss mengecil"
            ),

        -- Bulan lalu netral
        _LastSign = 0,
            IF(
                _CurrSign > 0,
                "Surplus membesar",
                "Loss membesar"
            ),

        -- Flip
        _CurrSign <> _LastSign,
            IF(
                _CurrSign > 0,
                "Flip surplus " & _Magnitude,
                "Flip loss " & _Magnitude
            ),

        -- Tidak flip
        IF(
            _CurrSign > 0,
            "Surplus " & _Magnitude,
            "Loss " & _Magnitude
        )
    )
)
```

### Week 3

```dax
VAR Nilai =
    CALCULATE(
        [Deviasi CM],
        'Deviasi All'[Periode] = "Week 3"
    )
RETURN
IF(
    ABS(Nilai) < 0.1,
    BLANK(),
    Nilai
)
```

### Week 3 Abs

```dax
ABS([Week 3])
```

### Perubahan W3

```dax
IF(
    ISBLANK([Week 3]),
    BLANK(),
    DIVIDE(
        ABS([Week 3]) - ABS([Week 2]),
        ABS([Week 2])
    )
)
```

### Growth W3

```dax
IF(
    OR(
        ISBLANK([Week 3]),
        ISBLANK([Week 2])
    ),
    BLANK(),
    DIVIDE(
        [Week 3] - [Week 2],
        ABS([Week 2])
    )
)
```

### BM Sistem

```dax
SUM('3224'[Kts (Unit#1)])+SUM('4205'[#Qty. Terkecil])+[Pembelian di Luar]
```

### BM Resto

```dax
SUM('Rekap Mentah BM'[Value])
```

### Count BM Resto

```dax
COUNTROWS(
    FILTER(
        VALUES('Date'[Date]),
        [BM Resto] <> 0
    )
)
```

### Count BM Sistem

```dax
COUNTROWS(
    FILTER(
        VALUES('Date'[Date]),
        [BM Sistem] <> 0
    )
)
```

### % BM

```dax
DIVIDE(
    MIN([Count BM Resto], [Count BM Sistem]),
    MAX([Count BM Resto], [Count BM Sistem])
)
```

### ABS Selisih BM

```dax
ABS([Selisih BM])
```

### Show BM

```dax
IF(AND([% BM]=1,[Selisih BM]=0),"Hide","Show")
```

### Input Salah

```dax
CALCULATE(
    DISTINCTCOUNT(Inventory[Nama Barang]),
    FILTER(
        VALUES(Inventory[Nama Barang]),
        [Count BM Resto] > 0 &&
        [Count BM Sistem] > 0 &&
        ([Selisih BM] <> 0)
    )
)
```

### Input Benar

```dax
CALCULATE(
    DISTINCTCOUNT(Inventory[Nama Barang]),
    FILTER(
        VALUES(Inventory[Nama Barang]),
        [Count BM Resto] > 0 &&
        [Count BM Sistem] > 0 &&
        ([Selisih BM] = 0)
    )
)
```

### Total BM

```dax
[Input Benar]+[Input Salah]
```

### % BM Benar

```dax
DIVIDE([Input Benar],[Total BM])
```

### % BM Salah

```dax
DIVIDE([Input Salah],[Total BM])
```

### COM CM

```dax
[BOM CM]+[Deviasi CM]
```

### Sales sd 07 Bulan Ini

```dax
CALCULATE([Sales CM],Periode[Periode]="WEEK 1")
```

### Sales sd 07 Bulan Lalu

```dax
CALCULATE([Sales LM],Periode[Periode]="WEEK 1")
```

### Sales sd 14 Bulan Ini

```dax
CALCULATE([Sales CM],Periode[Periode]="WEEK 2")
```

### Sales sd 14 Bulan Lalu

```dax
CALCULATE([Sales LM],Periode[Periode]="WEEK 2")
```

### Sales sd 25 Bulan Ini

```dax
CALCULATE([Sales CM],Periode[Periode]="WEEK 3")
```

### Sales sd 25 Bulan Lalu

```dax
CALCULATE([Sales LM],Periode[Periode]="WEEK 3")
```

### Sales sd EOM Bulan Ini

```dax
CALCULATE([Sales CM],Periode[Periode]="EOM")
```

### Sales sd EOM Bulan Lalu

```dax
CALCULATE([Sales LM],Periode[Periode]="EOM")
```

### % Sales Growth 07

```dax
IF(
    ISBLANK([Sales sd 07 Bulan Ini]),
    BLANK(),
    DIVIDE(
        [Sales sd 07 Bulan Ini] - [Sales sd 07 Bulan Lalu],
        [Sales sd 07 Bulan Lalu]
    )
)
```

### % Sales Growth 14

```dax
IF(
    ISBLANK([Sales sd 14 Bulan Ini]),
    BLANK(),
    DIVIDE(
        [Sales sd 14 Bulan Ini] - [Sales sd 14 Bulan Lalu],
        [Sales sd 14 Bulan Lalu]
    )
)
```

### % Sales Growth 25

```dax
IF(
    ISBLANK([Sales sd 25 Bulan Ini]),
    BLANK(),
    DIVIDE(
        [Sales sd 25 Bulan Ini] - [Sales sd 25 Bulan Lalu],
        [Sales sd 25 Bulan Lalu]
    )
)
```

### % Sales Growth EOM

```dax
IF(
    ISBLANK([Sales sd EOM Bulan Ini]),
    BLANK(),
    DIVIDE(
        [Sales sd EOM Bulan Ini] - [Sales sd EOM Bulan Lalu],
        [Sales sd EOM Bulan Lalu]
    )
)
```

### Show Value

```dax
VAR Nilai =
[Deviasi CM]+[BOM CM]+
[Deviasi LM]+[BOM LM]+
[Deviasi YTD]+[BOM YTD]+
[Deviasi YoY]+[BOM YoY]

RETURN
Nilai
```

### Cek Selisih Waste

```dax
[Waste]-SUM('Waste All'[Value])
```

### Nilai PI

```dax
SUMX(
    SUMMARIZE(
        '3243',
        '3243'[Nomor #],
        '3243'[Nama Barang]
    ),
    VAR _PO = '3243'[Nomor #]
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

### Belum Dibayar

```dax
SUM('3243'[Total Harga])-[Nilai PI]
```

### Status PO

```dax
VAR _TotalHargaPO =
    CALCULATE(
        SUM('3243'[Total Harga]),
        ALLEXCEPT(
            '3243',
            '3243'[Nomor #]
        )
    )

VAR _NilaiPIPO =
    CALCULATE(
        [Nilai PI],
        ALLEXCEPT(
            '3243',
            '3243'[Nomor #]
        )
    )

RETURN
    IF(
        _TotalHargaPO = _NilaiPIPO,
        "Sama",
        "Beda"
    )
```

### Nama Barang 9901

```dax
VAR _PO = SELECTEDVALUE('3243'[Nomor #])
VAR _Kode = SELECTEDVALUE('3243'[Kode #])
RETURN
CONCATENATEX(
    DISTINCT(
        SELECTCOLUMNS(
            FILTER(
                '9901',
                '9901'[Nomor # Pesanan Pembelian] = _PO &&
                '9901'[Kode #] = _Kode
            ),
            "Nama", '9901'[Nama Barang]
        )
    ),
    [Nama],
    ", ",
    [Nama],
    ASC
)
```

### Rank Dev

```dax
IF(
    ISINSCOPE('Inventory'[Nama Barang]),
    VAR CurrentValue = [ABS Deviasi]
    RETURN
        IF(
            NOT ISBLANK(CurrentValue) && CurrentValue <> 0,
            RANKX(
                ALL('Resto'[Resto]),
                [ABS Deviasi],
                ,
                DESC,
                Dense
            )
        )
)
```

### Rank BOM

```dax
IF(
    ISINSCOPE('Inventory'[Nama Barang]),
    VAR CurrentValue = [BOM CM]
    RETURN
        IF(
            NOT ISBLANK(CurrentValue) && CurrentValue <> 0,
            RANKX(
                ALL('Resto'[Resto]),
                [BOM CM],
                ,
                DESC,
                Dense
            )
        )
)
```

### Kontribusi CM Week 1

```dax
CALCULATE([Kontribusi CM],Periode[Periode]="WEEK 1")
```

### Kontribusi CM Week 2

```dax
CALCULATE([Kontribusi CM],Periode[Periode]="WEEK 2")
```

### Kontribusi CM Week 3

```dax
CALCULATE([Kontribusi CM],Periode[Periode]="WEEK 3")
```

### Kontribusi CM EOM

```dax
CALCULATE([Kontribusi CM],Periode[Periode]="EOM")
```

### Show Detail

```dax
VAR _Tipe = MAX(Inventory[Tipe])
VAR _Jenis = MAX('Deviasi All'[Jenis])
RETURN
SWITCH(
    TRUE(),
    _Tipe = "COM DEVIASI" && _Jenis IN {"WASTE", "TRIAL", "SUSUT", "LOSS/SURPLUS"}, "SHOW",
    _Tipe = "COM DEVIASI" && _Jenis = "USAGE", "HIDE",

    _Tipe = "CONSUME" && _Jenis IN {"USAGE", "LOSS/SURPLUS"}, "SHOW",
    _Tipe = "CONSUME" && _Jenis IN {"WASTE", "TRIAL", "SUSUT"}, "HIDE",

    _Tipe = "PACKAGING DEVIASI" && _Jenis IN {"WASTE", "TRIAL", "LOSS/SURPLUS"}, "SHOW",
    _Tipe = "PACKAGING DEVIASI" && _Jenis IN {"USAGE", "SUSUT"}, "HIDE",

    _Tipe = "PACKAGING USAGE" && _Jenis IN {"USAGE", "LOSS/SURPLUS"}, "SHOW",
    _Tipe = "PACKAGING USAGE" && _Jenis IN {"WASTE", "TRIAL", "SUSUT"}, "HIDE",

    BLANK()
)
```

### Deviasi Value

```dax
CALCULATE(
    SUM('Deviasi All'[Value]),
    KEEPFILTERS(
        'Deviasi All'[Jenis] IN {"WASTE", "SUSUT", "TRIAL", "LOSS/SURPLUS", "USAGE"}
    )
)
```

### COM LM

```dax
CALCULATE([COM CM], PREVIOUSMONTH('Date'[Date]))
```

### COM Gap

```dax
[COM CM]-[COM LM]
```

### COM Growth

```dax
DIVIDE([COM Gap],[COM LM])
```

</details>

## Power Query (M) — Tabel Inti

### Inventory

```powerquery
let
    Source = Excel.Workbook(Web.Contents("https://docs.google.com/spreadsheets/d/e/2PACX-1vRyQx4UUMgt9PX8hy-AMAiEUg93FfMiZCyUcb94m8do2mS44EBGDRnZdjCGKznUEQ/pub?output=xlsx"), null, true),
    Inventory_Sheet = Source{[Item="Inventory",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(Inventory_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Nama Barang", type text}, {"Tipe", type text}, {"Kategori 1", type text}, {"Jenis", type text}, {"Source", type text}, {"Status", type text}, {"Kategori 2", type text}, {"Satuan", type text}, {"Isi Pack/Botol/Kaleng", type text}, {"Satuan Terkecil", type text}, {"Satuan Terkecil2", type text}}),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each [Nama Barang] <> null and [Nama Barang] <> ""),
    #"Renamed Columns" = Table.RenameColumns(#"Filtered Rows",{{"Nama Barang", "(Old) Nama Barang"}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"UoM", type text}})
in
    #"Changed Type1"
```

### Deviasi All

```powerquery
let
    Source = Folder.Files("H:\My Drive\COST CONTROL\Deviasi"),
    #"Filtered Rows" = Table.SelectRows(Source, each not Text.StartsWith([Name], "~$")),
    #"Filtered Hidden Files1" = Table.SelectRows(#"Filtered Rows", each [Attributes]?[Hidden]? <> true),
    #"Invoke Custom Function1" = Table.AddColumn(#"Filtered Hidden Files1", "Transform File", each #"Transform File"([Content])),
    #"Renamed Columns1" = Table.RenameColumns(#"Invoke Custom Function1", {"Name", "Source.Name"}),
    #"Removed Other Columns1" = Table.SelectColumns(#"Renamed Columns1", {"Source.Name", "Transform File"}),
    #"Expanded Table Column1" = Table.ExpandTableColumn(#"Removed Other Columns1", "Transform File", Table.ColumnNames(#"Transform File"(#"Sample File"))),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Table Column1",{{"Source.Name", type text}, {"Akun Penyesuaian Persediaan", type text}, {"Status", type text}, {"Resto", type text}, {"Nama Barang", type text}, {"Satuan", type text}, {"QTY_BOM", type number}, {"QTY_COM", type number}, {"QTY_DEVIASI", type number}, {"QTY_USAGE", Int64.Type}, {"QTY_WASTE", type number}, {"QTY_SUSUT", Int64.Type}, {"QTY_TRIAL", Int64.Type}, {"QTY_LOSS/SURPLUS", type number}, {"NOMINAL_BOM", type number}, {"NOMINAL_COM", type number}, {"NOMINAL_DEVIASI", type number}, {"NOMINAL_USAGE", Int64.Type}, {"NOMINAL_WASTE", type number}, {"NOMINAL_SUSUT", Int64.Type}, {"NOMINAL_TRIAL", type number}, {"NOMINAL_LOSS/SURPLUS", type number}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type", "Periode", each Text.BeforeDelimiter(Text.Split([Source.Name], "_"){2},".")),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",{{"Source.Name", type text}, {"Akun Penyesuaian Persediaan", type text}, {"Status", type text}, {"Resto", type text}, {"Nama Barang", type text}, {"Satuan", type text}, {"QTY_BOM", type number}, {"QTY_COM", type number}, {"QTY_DEVIASI", type number}, {"QTY_USAGE", Int64.Type}, {"QTY_WASTE", Int64.Type}, {"QTY_SUSUT", Int64.Type}, {"QTY_TRIAL", Int64.Type}, {"QTY_LOSS/SURPLUS", type number}, {"NOMINAL_BOM", type number}, {"NOMINAL_COM", type number}, {"NOMINAL_DEVIASI", type number}, {"NOMINAL_USAGE", Int64.Type}, {"NOMINAL_WASTE", type number}, {"NOMINAL_SUSUT", Int64.Type}, {"NOMINAL_TRIAL", type number}, {"NOMINAL_LOSS/SURPLUS", type number}, {"Periode", type text}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type1",{"QTY_COM","QTY_DEVIASI","NOMINAL_COM","NOMINAL_DEVIASI"}),
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Removed Columns", {"Source.Name", "Akun Penyesuaian Persediaan", "Status", "Resto", "Nama Barang", "Satuan", "Periode"}, "Attribute", "Value"),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Unpivoted Columns", "Attribute", Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv), {"Attribute.1", "Attribute.2"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Split Column by Delimiter",{{"Attribute.1", type text}, {"Attribute.2", type text}}),
    #"Renamed Columns" = Table.RenameColumns(#"Changed Type2",{{"Attribute.1", "Attribute"}, {"Attribute.2", "Jenis"}}),
    #"Added Custom1" = Table.AddColumn(#"Renamed Columns", "Tanggal", each #date(
    Number.FromText(Text.Split([Source.Name], "_"){0}),
    Number.FromText(Text.Split([Source.Name], "_"){1}),
    1
)),
    #"Changed Type3" = Table.TransformColumnTypes(#"Added Custom1",{{"Tanggal", type date}})
in
    #"Changed Type3"
```

### Waste All

```powerquery
let
    Source = Folder.Files("H:\My Drive\COST CONTROL\Kategori Waste"),
    #"Filtered Hidden Files1" = Table.SelectRows(Source, each [Attributes]?[Hidden]? <> true),
    #"Invoke Custom Function1" = Table.AddColumn(#"Filtered Hidden Files1", "Transform File (2)", each #"Transform File (2)"([Content])),
    #"Renamed Columns1" = Table.RenameColumns(#"Invoke Custom Function1", {"Name", "Source.Name"}),
    #"Removed Other Columns1" = Table.SelectColumns(#"Renamed Columns1", {"Source.Name", "Transform File (2)"}),
    #"Expanded Table Column1" = Table.ExpandTableColumn(#"Removed Other Columns1", "Transform File (2)", Table.ColumnNames(#"Transform File (2)"(#"Sample File (2)"))),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Table Column1",{{"Source.Name", type text}, {"KATEGORI", type text}, {"CABANG", type text}, {"Nama  new item ", type text}, {"SATUAN", type text}, {"QTY_Alat/Mesin", Int64.Type}, {"QTY_Cancel Nota", type number}, {"QTY_Handling", type number}, {"QTY_Kualitas Eksternal", type number}, {"QTY_Kualitas Internal", type number}, {"QTY_Over Stock", Int64.Type}, {"NOMINAL_Alat/Mesin", type number}, {"NOMINAL_Cancel Nota", type number}, {"NOMINAL_Handling", type number}, {"NOMINAL_Kualitas Eksternal", type number}, {"NOMINAL_Kualitas Internal", type number}, {"NOMINAL_Over Stock", type number}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type", "Periode", each Text.Split([Source.Name], "_"){2}),
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Added Custom", {"Source.Name", "KATEGORI", "CABANG", "Nama  new item ", "SATUAN", "Periode"}, "Attribute", "Value"),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Unpivoted Columns", "Attribute", Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv), {"Attribute.1", "Attribute.2"}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Split Column by Delimiter",{{"Attribute.1", type text}, {"Attribute.2", type text}}),
    #"Renamed Columns" = Table.RenameColumns(#"Changed Type1",{{"Attribute.1", "Attribute"}, {"Attribute.2", "Jenis"}}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Renamed Columns",{{"Source.Name", type text}, {"KATEGORI", type text}, {"CABANG", type text}, {"Nama  new item ", type text}, {"SATUAN", type text}, {"Periode", type text}, {"Attribute", type text}, {"Jenis", type text}, {"Value", type number}}),
    #"Added Custom1" = Table.AddColumn(#"Changed Type2", "Tanggal", each #date(
    Number.FromText(Text.Split([Source.Name], "_"){0}),
    Number.FromText(Text.Split([Source.Name], "_"){1}),
    1
)),
    #"Changed Type3" = Table.TransformColumnTypes(#"Added Custom1",{{"Tanggal", type date}})
in
    #"Changed Type3"
```

### Sales All

```powerquery
let
    Source = Folder.Files("H:\My Drive\COST CONTROL\1316"),
    #"Filtered Hidden Files1" = Table.SelectRows(Source, each [Attributes]?[Hidden]? <> true),
    #"Invoke Custom Function1" = Table.AddColumn(#"Filtered Hidden Files1", "Transform File (7)", each #"Transform File (7)"([Content])),
    #"Renamed Columns1" = Table.RenameColumns(#"Invoke Custom Function1", {"Name", "Source.Name"}),
    #"Removed Other Columns1" = Table.SelectColumns(#"Renamed Columns1", {"Source.Name", "Transform File (7)"}),
    #"Expanded Table Column1" = Table.ExpandTableColumn(#"Removed Other Columns1", "Transform File (7)", Table.ColumnNames(#"Transform File (7)"(#"Sample File (7)"))),
    #"Changed Type" = Table.TransformColumnTypes(#"Expanded Table Column1",{{"Source.Name", type text}, {"Bulan", type date}, {"Nama Cabang", type text}, {"Biaya Bahan Pembantu - RESTO", type number}, {"Biaya Packaging - RESTO", type number}, {"Pendapatan Penjualan", type number}, {"Potongan Lain-lain", type number}, {"Potongan Penjualan Barang", Int64.Type}, {"Retur Makanan dan Minuman", Int64.Type}, {"Penjualan Bersih", type number}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type", "Periode", each Text.Split(Text.BeforeDelimiter([Source.Name], ".xlsx"), "_"){3}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",{{"Periode", type text}}),
    #"Added Custom1" = Table.AddColumn(#"Changed Type1", "Tanggal", each #date(
    Number.FromText(Text.Split([Source.Name], "_"){1}),
    Number.FromText(Text.Split([Source.Name], "_"){2}),
    1
)),
    #"Changed Type2" = Table.TransformColumnTypes(#"Added Custom1",{{"Tanggal", type date}})
in
    #"Changed Type2"
```

### Resto

```powerquery
let

    Source = Excel.Workbook(
        Web.Contents("https://docs.google.com/spreadsheets/d/e/2PACX-1vRyQx4UUMgt9PX8hy-AMAiEUg93FfMiZCyUcb94m8do2mS44EBGDRnZdjCGKznUEQ/pub?output=xlsx"),
        null,
        true
    ),

    #"4218" = Source{[Item="4218",Kind="Sheet"]}[Data],
    #"Filtered Rows" = Table.SelectRows(#"4218", each [Column5] <> null and [Column5] <> ""),
    #"Promoted Headers" = Table.PromoteHeaders(#"Filtered Rows", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Column1", type any}, {"Column2", type any}, {"No. Urut", Int64.Type}, {"Column4", type any}, {"Nama", type text}, {"Column6", type any}, {"Deskripsi", type text}, {"Column8", type any}, {"Jalan Alamat", type text}, {"Column10", type any}, {"Kota Alamat", type text}, {"Column12", type any}, {"Provinsi Alamat", type text}, {"Column14", type any}, {"K.Pos Alamat", Int64.Type}, {"Column16", type any}, {"Column17", type any}, {"Resto", type text}, {"PIC", type text}, {"Provinsi", type text}, {"Area", type text}, {"Type Kitchen", type text}, {"AM", type text}}),
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type",{"No. Urut", "Nama", "Deskripsi", "Jalan Alamat", "Kota Alamat", "Provinsi Alamat", "K.Pos Alamat", "Resto", "PIC", "Provinsi", "Area", "Type Kitchen", "AM"})
in
    #"Removed Other Columns"
```

### Update

```powerquery
let
    Source = Excel.Workbook(Web.Contents("https://docs.google.com/spreadsheets/d/e/2PACX-1vRyQx4UUMgt9PX8hy-AMAiEUg93FfMiZCyUcb94m8do2mS44EBGDRnZdjCGKznUEQ/pub?output=xlsx"), null, true),
    Update_Sheet = Source{[Item="Update",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(Update_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Keterangan", type text}, {"Tanggal", type date}}),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each [Keterangan] <> null and [Keterangan] <> "")
in
    #"Filtered Rows"
```
