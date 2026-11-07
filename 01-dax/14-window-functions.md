# Window Functions (INDEX, OFFSET, WINDOW, RANK, ROWNUMBER)

> Window functions (DAX 2022+) menghitung "baris sebelumnya", "peringkat", "running total", dan "moving average" langsung di dalam measure — tanpa lagi pattern `TOPN`/`EARLIER`/`FILTER(ALL(...))` yang rapuh. Dikelola dengan `ORDERBY`, `PARTITIONBY`, dan `MATCHBY`.

## Konsep & Penjelasan

### Apa itu window function

Window function bekerja atas **kumpulan baris yang sudah terurut** (*window*): ia tahu posisi baris saat ini, lalu bisa "melompat" ke baris lain (sebelum/sesudah) atau mengambil rentang baris. Ini meniru `LAG`/`LEAD`/`RANK`/`ROW_NUMBER` dari SQL, tapi dievaluasi dalam konteks measure Power BI.

| Fungsi | Mengembalikan | Kegunaan |
|---|---|---|
| `INDEX(posisi, ...)` | Tabel baris | Baris ke-`posisi` (1 = pertama, -1 = terakhir) — ambil champion/juara terakhir |
| `OFFSET(delta, ...)` | Tabel baris | Baris yang berjarak `delta` dari baris saat ini (`-1` = sebelumnya) |
| `WINDOW(dari, tipeDari, sampai, tipeSampai, ...)` | Tabel baris | Rentang baris dari `dari` s/d `sampai` |
| `RANK(ties, ...)` | Skalar | Peringkat baris saat ini menurut `ORDERBY` |
| `ROWNUMBER(...)` | Skalar | Nomor urut baris saat ini |
| `ORDERBY(kolom/measure, ASC/DESC, ...)` | Helper | Menentukan urutan — **wajib eksplisit** |
| `PARTITIONBY(kolom, ...)` | Helper | Hitung terpisah per grup (mis. ranking per resto) |
| `MATCHBY(kolom, ...)` | Helper | Kolom identitas bila kombinasi ORDERBY/PARTITIONBY tidak unik (baris kembar) |

**Riwayat versi:** `INDEX`/`OFFSET`/`WINDOW` rilis Desember 2022; `RANK`/`ROWNUMBER` menyusul Februari 2024 (fondasi *visual calculations*). Butuh Power BI Desktop modern; file PBIX lama dibuka di versi baru tetap kompatibel.

### Konsep kunci

- **Relation** — tabel tempat window dibangun. Bisa eksplisit (`ALLSELECTED('Date'[Period])`) atau dibiarkan kosong agar diambil otomatis dari kolom visual saat ini.
- **REL vs ABS** — `REL` = posisi **relatif** terhadap baris saat ini (`-2` = dua baris sebelumnya, `0` = baris ini); `ABS` = posisi **absolut** dari hasil urut (1 = baris pertama, -1 = baris terakhir).
- Window function **mengembalikan baris**, bukan mengubah filter context — untuk "memindahkan" nilai, bungkus dengan `CALCULATE` atau jadikan input iterator (`AVERAGEX`, `SUMX`).
- Window function dirancang untuk konteks **query/visual** (measure), bukan calculated column.

### Keunggulan dibanding pattern lama

| Kebutuhan | Pattern lama | Window function |
|---|---|---|
| Nilai bulan/item sebelumnya | `VAR` + `TOPN`/`MAX` tanggal + `CALCULATE` berlapis | `OFFSET(-1, ORDERBY(...))` |
| Running total | `FILTER(ALL('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date]))` | `WINDOW(1, ABS, 0, REL, ...)` |
| Ranking | `RANKX(ALL(...), [M], , DESC)` + atur ulang filter | `RANK(DENSE, ORDERBY([M], DESC))` — otomatis ikut visual |
| Moving average non-tanggal (per resto/item) | `AVERAGEX` + `TOPN` manual, rawan salah | `WINDOW(-2, REL, 0, REL, ...)` |
| Ranking per grup | `RANKX` + `ALLEXCEPT`, `EARLIER` | `PARTITIONBY('Resto'[Area])` |

Keuntungan umum: lebih sedikit baris kode, urutan eksplisit lewat `ORDERBY`, mendukung partisi, dan tidak perlu memanipulasi `ALL()` secara manual.

## Sintaks & Contoh

### 1. Bulan ini vs bulan sebelumnya dengan OFFSET

```dax
-- Visual dikelompokkan per 'Date'[Period] (teks "yyyy-mm", urut leksikografis aman)
Deviasi LM (Window) =
CALCULATE(
    [Deviasi CM],
    OFFSET(
        -1,                                    -- satu baris sebelum baris saat ini
        ALLSELECTED('Date'[Period]),           -- relation: semua periode yang dipilih
        ORDERBY('Date'[Period], ASC)
    )
)
```

Untuk urutan berbasis kolom angka (lebih aman), ganti relation & ORDERBY dengan `'Date'[Year]` + `'Date'[MonthNumber]` atau kolom kunci `YYYYMM`.

### 2. Ranking dengan RANK (vs RANKX)

```dax
-- RANKX klasik — masih valid, tapi harus mengelola ALLSELECTED sendiri
Rank Dev (RANKX) =
RANKX(ALLSELECTED('Inventory'[Nama Barang]), [Nominal Dev], , DESC, DENSE)

-- RANK modern — relation + ORDERBY eksplisit, hasil sama
Rank Dev (Window) =
RANK(DENSE, ALLSELECTED('Inventory'[Nama Barang]), ORDERBY([Nominal Dev], DESC))
```

### 3. Ranking per grup dengan PARTITIONBY

```dax
-- Di matrix Resto × Nama Barang: ranking item di DALAM tiap resto
Rank Item per Resto =
RANK(
    DENSE,
    ORDERBY([Deviasi CM], DESC),
    PARTITIONBY('Deviasi All'[RESTO])
)
-- relation tidak ditulis -> diambil dari kolom visual yang aktif
```

### 4. Running total dengan WINDOW

```dax
-- Akumulasi dari baris pertama sampai baris saat ini
Running Total Deviasi =
CALCULATE(
    [Deviasi CM],
    WINDOW(
        1, ABS,                                -- mulai dari baris pertama (absolut)
        0, REL,                                -- berakhir di baris saat ini (relatif)
        ALLSELECTED('Date'[Period]),
        ORDERBY('Date'[Period], ASC)
    )
)
```

### 5. Moving average 3 bulan dengan WINDOW

```dax
-- Rata-rata 3 baris terakhir termasuk baris saat ini
-- (di baris awal, window otomatis dipangkas ke baris yang tersedia)
MA 3 Bulan Deviasi =
AVERAGEX(
    WINDOW(
        -2, REL, 0, REL,
        ALLSELECTED('Date'[Period]),
        ORDERBY('Date'[Period], ASC)
    ),
    [Deviasi CM]
)
```

Karena window bekerja atas **urutan baris** (bukan kolom tanggal), pattern yang sama bisa untuk moving average per **resto** atau per **item** — hal yang sulit dilakukan `DATESINPERIOD`.

### 6. Best & worst performer dengan INDEX

```dax
Deviasi Tertinggi =
CALCULATE(
    [Deviasi CM],
    INDEX(1, ALLSELECTED('Resto'[Resto]), ORDERBY([Deviasi CM], DESC))
)

Deviasi Terendah =
CALCULATE(
    [Deviasi CM],
    INDEX(-1, ALLSELECTED('Resto'[Resto]), ORDERBY([Deviasi CM], DESC))
)
```

### 7. Nomor urut & MATCHBY untuk data transaksi

```dax
-- ROWNUMBER wajib menyebut relation
No Urut PO =
ROWNUMBER(
    ALLSELECTED('9901'[Nomor #], '9901'[Tanggal #Buat]),
    ORDERBY('9901'[Tanggal #Buat], ASC),
    MATCHBY('9901'[Nomor #])       -- identitas unik bila ada baris kembar
)
```

`MATCHBY` dibutuhkan saat kombinasi kolom PARTITIONBY/ORDERBY tidak unik (nilai kembar) — ia memberitahu engine kolom mana yang mengidentifikasi baris secara unik. Parameter terakhir `reset` bergunakan di *visual calculation* untuk memulai ulang penomoran per level matriks.

## Kesalahan Umum

- **Lupa ORDERBY.** Window function butuh urutan deterministik; tanpa `ORDERBY` eksplisit muncul error atau hasil tidak terduga. Selalu tulis, jangan andalkan urutan default.
- **Tertukar REL vs ABS.** `OFFSET(-1)` = satu baris **sebelum baris saat ini**; `WINDOW(1, ABS, ...)` = mulai dari **baris pertama**. Salah pilih membuat "bulan lalu" berubah jadi "bulan pertama".
- **ORDERBY kolom teks bulan tanpa sort-by-column.** `"Apr"` terurut sebelum `"Jan"` (alfabetis) — gunakan `yyyy-mm`, kolom angka, atau *Sort by column* yang benar.
- **Mengira window function memfilter otomatis.** `OFFSET` mengembalikan tabel baris; tanpa `CALCULATE` (atau iterator), nilai measure tidak "pindah" ke baris tersebut.
- **Memakai window function di calculated column.** Tidak didukung — ia dirancang untuk konteks query/visual. Pindahkan logika ke measure.
- **Relation ber-kardinalitas tinggi.** Jangan pakai window atas tabel transaksi jutaan baris; agregasi dulu (`SUMMARIZE`/`VALUES`) lalu window atas hasilnya.
- **PARTITIONBY/ORDERBY memakai kolom yang tidak ada di relation/visual** → error *column not found*; pastikan kolom tersebut bagian dari relation eksplisit atau sumbu visual.

## Tips Pro

- Untuk visual bulanan, satu kolom kunci urut seperti `'Date'[Period]` (`"yyyy-mm"`) menyederhanakan ORDERBY dan aman terhadap locale; untuk drill-in harian pakai `Year` + `MonthNumber` + `Date`.
- Padukan dengan variabel tabel: `VAR T = WINDOW(-2, REL, 0, REL, ...)` lalu inspeksi `COUNTROWS(T)` saat debugging — lihat [Variabel & Error Handling](./13-variables-and-error-handling.md).
- `RANK(DENSE, ...)` untuk ranking yang sering seri (banyak nilai 0); `SKIP` (default) bila gap ranking penting.
- Window functions membuat banyak pattern klasik usang — sebelum menulis `TOPN`/`EARLIER` panjang, cek dulu apakah `OFFSET`/`WINDOW` 3 baris sudah cukup.
- Untuk perbandingan bulan lalu pada model dengan tabel Date terpisah (pattern Mode/TREATAS), `OFFSET` tetap bekerja karena cukup menggeser baris pada relation — tidak menyentuh relasi antar tabel.

## Terkait

- [Time Intelligence](./07-time-intelligence.md) — alternatif klasik berbasis tanggal (PREVIOUSMONTH, SAMEPERIODLASTYEAR)
- [Table Functions](./12-table-functions.md) — SUMMARIZE/VALUES sebagai relation window
- [Variabel & Error Handling](./13-variables-and-error-handling.md) — variabel tabel untuk window
- [Resep DAX](./16-dax-patterns-recipes.md) — running total, moving average, Pareto versi resep
- [Performa DAX](./17-dax-performance.md) — biaya materialisasi window
- [DAX Cheatsheet](../00-quick-reference/dax-cheatsheet.md) — tabel fungsi window ringkas
- [Glosarium](../00-quick-reference/glossary.md) — istilah partition, tie-breaking, xmSQL
