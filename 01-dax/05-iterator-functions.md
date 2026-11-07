# Fungsi Iterator (X-Suffix)

> Iterator mengevaluasi expression **baris demi baris** di dalam row context lalu mengagregasi hasilnya — membuka kalkulasi yang mustahil dilakukan agregasi kolom biasa (ABS per baris, ranking, penggabungan teks). `SUMX`, `RANKX`, `CONCATENATEX`, `FILTER`, dan `ADDCOLUMNS` semuanya adalah iterator.

## Konsep Iterator

### Anatomi

```text
SUMX ( <tabel>            , <expression> )
      └───┬───┘             └─────┬─────┘
   diiterasi per baris    dievaluasi dalam row context
   (satu row context per baris)
```

Alur eksekusi:

1. `<tabel>` dievaluasi lebih dulu di filter context pemanggil → menghasilkan daftar baris.
2. Setiap baris membuat **row context**.
3. `<expression>` dievaluasi dalam row context itu — boleh membaca kolom; bila memanggil **measure**, terjadi **context transition** per baris (lihat [Row Context vs Filter Context](./02-row-context-vs-filter-context.md)).
4. Hasil per baris diagregasi sesuai fungsinya: sum, average, max, count, join, atau rank.

### Keluarga X — Referensi

| Function | Deskripsi | Contoh |
|---|---|---|
| `SUMX` | Menjumlahkan expression per baris | `SUMX ( 'Deviasi All', ABS ( 'Deviasi All'[Value] ) )` |
| `AVERAGEX` | Rata-rata expression (Blank di-skip) | `AVERAGEX ( VALUES ( 'Resto'[Resto] ), [% BOM CM] )` |
| `MINX` | Nilai minimum expression | `MINX ( VALUES ( 'Resto'[Resto] ), [Deviasi CM] )` |
| `MAXX` | Nilai maksimum expression | `MAXX ( VALUES ( 'Date'[Date] ), [Deviasi CM] )` |
| `COUNTX` | Menghitung baris yang hasil expression-nya numerik | `COUNTX ( 'Sales All', 'Sales All'[Penjualan Bersih] )` |
| `PRODUCTX` | Mengalikan hasil expression per baris | `PRODUCTX ( GENERATESERIES ( 1, 5 ), [Value] )  -- 5! = 120` |
| `CONCATENATEX` | Menggabungkan hasil text per baris (delimiter + urutan) | `CONCATENATEX ( VALUES ( 'Resto'[PIC] ), 'Resto'[PIC], ", " )` |
| `RANKX` | Melakukan ranking nilai dalam tabel | `RANKX ( ALL ( 'Resto'[Resto] ), [% BOM CM], , DESC, Dense )` |

### FILTER adalah Iterator

`FILTER ( tabel, kondisi )` mengevaluasi kondisi per baris dan mengembalikan baris yang lolos — bentuk iterator paling umum dipakai sebagai filter argumen `CALCULATE`:

```dax
Jumlah Resto Rugi =
COUNTROWS ( FILTER ( VALUES ( 'Resto'[Resto] ), [Deviasi CM] < 0 ) )
```

Kuncinya: iterasi pada **granularity kolom dimensi** (`VALUES`), bukan seluruh tabel fakta. Detail di [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md).

### ADDCOLUMNS / SELECTCOLUMNS / SUMMARIZE — Iterator Tabel

Fungsi-fungsi ini mengiterasi tabel dan **menghasilkan tabel baru** (bukan angka) — fondasi tabel virtual:

```dax
-- Tabel resto + deviasinya (untuk TOPN, ranking, tooltip)
Tabel Resto =
ADDCOLUMNS (
    VALUES ( 'Resto'[Resto] ),   -- tabel yang diiterasi
    "Deviasi", [Deviasi CM],     -- kolom baru: measure per baris
    "% BOM", [% BOM CM]          -- (context transition per baris)
)

-- SELECTCOLUMNS: membentuk ulang kolom per baris
Nama Resto Saja = SELECTCOLUMNS ( 'Resto', "Resto", 'Resto'[Resto] )

-- SUMMARIZE: group-by kombinasi unik
Per Item Resto =
SUMMARIZE ( 'Deviasi All', 'Inventory'[Nama Barang], 'Resto'[Resto] )
```

### Early vs Late Binding (Singkat)

- **Early binding** — expression dievaluasi **sekali di context pemanggil** sebelum iterasi: `VAR` sebelum iterator, dan argumen `value` pada `RANKX`.
- **Late binding** — expression dievaluasi **per baris di dalam iterasi**: argumen `expression` pada `RANKX`, dan kolom baru `ADDCOLUMNS` saat berisi measure (hasilnya berbeda per baris karena context transition).
- Implikasi: hasil `RANKX` yang "aneh" hampir selalu karena lupa bahwa satu argumen dievaluasi di luar iterasi, sementara yang lain di dalamnya.

## RANKX Secara Lengkap

### Sintaks

```dax
RANKX ( <table>, <expression> [, <value>] [, <order>] [, <ties>] )
```

| Argumen | Arti | Default |
|---|---|---|
| `table` | Daftar nilai yang ikut diranking (biasanya `ALL ( kolom )`) | wajib |
| `expression` | Nilai tiap baris `table` — dievaluasi per baris + context transition | wajib |
| `value` | Nilai yang dicari posisinya (**early binding** — di context pemanggil) | `expression` di context saat ini |
| `order` | `ASC` / `DESC` | `ASC` |
| `ties` | `Skip` / `Dense` | `Skip` |

### Ties: Skip vs Dense

| Nilai | Rank (Skip) | Rank (Dense) |
|---|---|---|
| 100 | 1 | 1 |
| 100 | 1 | 1 |
| 80 | **3** | **2** |
| 70 | **4** | **3** |

- **Skip** (competition ranking): nilai yang seri berbagi rank, rank berikutnya melompat sejumlah anggota seri.
- **Dense**: rank berikutnya langsung bertambah satu, tanpa lompatan.
- Tanpa nilai duplikat, keduanya identik — pilih sesuai definisi bisnis "peringkat".

### Ranking Resto per Deviasi — Step by Step

```dax
Rank Deviasi =
IF (
    ISINSCOPE ( 'Resto'[Resto] ),      -- jangan ikut me-ranking baris total
    VAR NilaiSekarang = [% BOM CM]     -- early binding: context baris visual
    RETURN
        RANKX (
            ALL ( 'Resto'[Resto] ),    -- 1. semua resto; filter resto visual dihapus
            [% BOM CM],                -- 2. dievaluasi per resto (context transition)
            NilaiSekarang,             -- 3. nilai yang dicari posisinya
            DESC,                      -- 4. % BOM terbesar = rank 1
            Dense                      -- 5. resto seri tidak membuat rank melompat
        )
)
```

Penjelasan langkah:

1. `ALL ( 'Resto'[Resto] )` → daftar semua resto; slicer `Area` tetap berlaku (hanya filter kolom resto yang dihapus).
2. `[% BOM CM]` dievaluasi untuk **setiap** resto di daftar — context transition membuat masing-masing dihitung sendiri-sendiri.
3. `NilaiSekarang` sudah terikat lebih awal di context baris visual — inilah early binding.
4. Posisi `NilaiSekarang` dicari dalam daftar yang telah diurutkan `DESC`.
5. `ISINSCOPE` menjaga baris total tidak ikut diranking (total hampir selalu rank 1 — menyesatkan).

## Performa: Iterator vs Agregasi Vektor

- `SUM ( kolom )` dieksekusi storage engine secara vektorial dengan cache → sangat cepat bahkan di tabel raksasa.
- Iterator dengan expression kompleks (string, `FORMAT`, `IF` bertingkat) memaksa formula engine bekerja **row by row** → lambat pada tabel besar.
- Hierarki pilihan (tercepat → terlambat):
  1. Agregasi kolom: `SUM ( 'Deviasi All'[Value] )`
  2. Iterator dimensi: `SUMX ( VALUES ( 'Resto'[Resto] ), [Deviasi CM] )` — ratusan baris
  3. Iterator fakta: `SUMX ( 'Deviasi All', ... )` — jutaan baris
- Di dalam iterator, panggil **measure** daripada menulis ulang ekspresi — lebih mudah dibaca, di-cache, dan di-debug.

## Sintaks & Contoh

```dax
-- ========== SUMX bertahap: total magnitudo deviasi ==========
-- Langkah 1: total signed
Deviasi = SUM ( 'Deviasi All'[Value] )

-- Langkah 2 (SALAH): absolut setelah dijumlah
-- surplus & loss saling meniadakan
Salah = ABS ( SUM ( 'Deviasi All'[Value] ) )

-- Langkah 3 (BENAR): absolut per baris, baru dijumlah
ABS Deviasi = SUMX ( 'Deviasi All', ABS ( 'Deviasi All'[Value] ) )

-- Langkah 4 (optimal): cukup per resto, tidak perlu per baris fakta
ABS Deviasi v2 =
SUMX ( VALUES ( 'Resto'[Resto] ), ABS ( [Deviasi CM] ) )

-- ========== CONCATENATEX: daftar PIC dengan urutan ==========
PIC Area =
CONCATENATEX (
    VALUES ( 'Resto'[PIC] ),
    'Resto'[PIC],
    ", ",
    'Resto'[PIC],
    ASC
)

-- ========== Z Score: iterator + CALCULATETABLE (pattern report user) ==========
Z Score =
VAR SummaryTable =
    CALCULATETABLE (
        SUMMARIZE ( 'Deviasi All', 'Date'[Date] ),
        ALLSELECTED ( 'Deviasi All' )
    )
VAR MeanValue = AVERAGEX ( SummaryTable, [Multi Value CM] )
VAR StdDevValue = STDEVX.P ( SummaryTable, [Multi Value CM] )
RETURN
    DIVIDE ( [Multi Value CM] - MeanValue, StdDevValue )
```

## Kesalahan Umum

- **`RANKX` tanpa `ALL`** — daftar yang diranking hanya berisi baris visual itu sendiri → semua rank 1. Perbaikan: `RANKX ( ALL ( 'Resto'[Resto] ), ... )`.
- **`RANKX` di baris total** — total adalah agregat semua baris dan hampir selalu rank 1. Perbaikan: guard `ISINSCOPE` / `HASONEVALUE`.
- **Menulis ekspresi mentah (bukan measure) sebagai argumen expression** — `RANKX ( ALL ( 'Resto'[Resto] ), SUM ( 'Deviasi All'[Value] ) )` dievaluasi tanpa filter per-resto → semua baris bernilai sama → semua rank 1. Perbaikan: panggil measure `[Deviasi CM]` agar context transition bekerja.
- **Mengira iterator memfilter tabel** — `SUMX ( 'Deviasi All', ... )` tetap menjumlahkan **semua** baris yang terlihat; untuk mengecualikan baris gunakan `FILTER` atau filter `CALCULATE`.
- **Iterasi tabel fakta raksasa padahal cukup dimensi** — `SUMX ( 'Deviasi All', ... )` vs `SUMX ( VALUES ( 'Resto'[Resto] ), ... )`: hasil sama, performa beda jauh.
- **`CONCATENATEX` tanpa delimiter** — teks menempel tanpa pemisah; selalu isi argumen delimiter.

## Tips Pro

- Pecah logika rumit menjadi `VAR` per tahap (tabel → agregat → hasil) — lihat pattern [Z Score] di atas dan di report user.
- `VALUES` vs `ALL` dalam iterator: `VALUES` menghormati slicer (ranking di dalam seleksi user), `ALL` me-reset — pilih secara sadar, bukan kebetulan.
- Kombinasi juara: `TOPN` + `CONCATENATEX` untuk ringkasan "Top 3 resto penyumbang waste" di tooltip/card.
- DAX Studio → *Server Timings*: bila Formula Engine mendominasi dan query memuat iterator besar, cari cara menurunkan granularity iterasi.
- Iterator bersarang adalah bau kode — biasanya bisa diratakan dengan `VAR` tabel virtual + `ADDCOLUMNS`.

## Terkait

- [Row Context vs Filter Context](./02-row-context-vs-filter-context.md) — context transition dijelaskan penuh
- [Fungsi Agregasi](./04-aggregation-functions.md)
- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md)
- [Time Intelligence](./07-time-intelligence.md)
