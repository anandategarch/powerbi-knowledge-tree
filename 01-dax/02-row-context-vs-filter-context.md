# Row Context vs Filter Context

> Dua jenis "context" ini menentukan bagaimana setiap formula DAX dievaluasi — dan sebagian besar kebingungan DAX berasal dari mencampuradukkannya. Kaidah inti: **filter context** menentukan *baris mana yang terlihat* oleh agregasi, sedangkan **row context** menentukan *baris mana yang sedang aktif* saat membaca kolom.

## Filter Context

### Apa Itu Filter Context

Filter context adalah **himpunan filter yang aktif ketika sebuah ekspresi dievaluasi**. Sumbernya:

- Baris, kolom, axis, dan legend pada visual — setiap sel evaluasi punya context sendiri
- Slicer dan filter panel (level visual/page/report)
- Propagasi **relationship** — filter dimensi diteruskan ke tabel fakta
- Pemanggilan `CALCULATE`/`CALCULATETABLE` dari dalam measure lain

Measure selalu dievaluasi di dalam suatu filter context; paling jauh context-nya kosong (tanpa filter sama sekali), tapi context itu tetap ada.

### Contoh Step-by-Step: Tabel per Area

Visual tabel sederhana — kolom `Resto[Area]` dan measure `[Deviasi CM]`:

| Area | Deviasi CM |
|---|---|
| BALI | 12.400.000 |
| JABODETABEK | -8.100.000 |
| JATIM | 3.700.000 |
| **Total** | **8.000.000** |

Evaluasi baris "BALI":

1. Visual menetapkan filter context untuk sel tersebut: `Resto[Area] = "BALI"` — digabung dengan slicer bulan, `Periode[Periode] = "EOM"`, dan filter lain yang sedang aktif.
2. Relationship antara `Resto` dan `Deviasi All` meneruskan filter — hanya baris deviasi milik resto area BALI yang terlihat.
3. `[Deviasi CM] = SUM('Deviasi All'[Value])` menjumlahkan baris yang tersisa → 12.400.000.
4. Baris **Total** dievaluasi ulang **tanpa filter Area** — bukan menjumlahkan ketiga sel di atasnya. Karena itu measure non-aditif (rasio, `DISTINCTCOUNT`, ranking) bisa menghasilkan total yang tidak sama dengan penjumlahan barisnya.

### Diagram Evaluasi Filter Context

```text
 Slicer: Date[Month]="Sep"        Slicer: Periode[Periode]="EOM"
        │                                │
        └───────────────┬────────────────┘
                        ▼
     VISUAL baris "BALI"  ──►  FILTER CONTEXT aktif:
                               • Resto[Area]      = "BALI"
                               • Date[Month]      = "Sep"
                               • Periode[Periode] = "EOM"
                               • (slicer / filter lain)
                                     │  diteruskan via relationship
                                     ▼
                              'Deviasi All' terfilter
                                     │
                                     ▼
                 [Deviasi CM] = SUM('Deviasi All'[Value])
                                     │
                                     ▼
                               hasil 12.400.000
```

## Row Context

### Apa Itu Row Context

Row context adalah **baris yang sedang ditunjuk** saat sebuah ekspresi dievaluasi. Row context hanya tercipta di dua tempat:

1. **Calculated column** — formula dievaluasi sekali per baris tabel saat refresh.
2. **Iterator** (`SUMX`, `FILTER`, `AVERAGEX`, `ADDCOLUMNS`, dst.) — expression dievaluasi per baris tabel yang diiterasi (lihat [Fungsi Iterator](./05-iterator-functions.md)).

```dax
-- Calculated column di tabel Deviasi All
-- row context membuat 'Deviasi All'[Value] terbaca baris demi baris
Nilai Abs = ABS ( 'Deviasi All'[Value] )
```

Row context memungkinkan membaca nilai kolom per baris — dan **tidak lebih dari itu**.

### Row Context TIDAK Otomatis Menjadi Filter

Kesalahpahaman nomor satu pemula DAX:

```dax
-- Calculated column di tabel Deviasi All (KONSEP SALAH)
Total per Jenis = SUM ( 'Deviasi All'[Value] )
```

Hasilnya **bukan** total per Jenis — melainkan total seluruh tabel, sama nilainya di semua baris. Alasannya: `SUM` hanya merespons **filter context**, dan calculated column tidak punya filter context (ia punya row context). Row context tidak memfilter tabel — ia hanya menunjuk baris.

## Context Transition

### Definisi

Ketika sebuah **measure** (atau `CALCULATE`) dipanggil dari dalam row context, engine otomatis menjalankan **context transition**: nilai seluruh kolom pada baris aktif dikonversi menjadi filter context `= nilai baris tersebut`.

```dax
-- Calculated column di tabel Resto
Total Deviasi Resto = [Deviasi CM]   -- panggilan measure memicu transition
```

Per baris `Resto`:

1. `[Deviasi CM]` secara internal dibungkus `CALCULATE` → context transition dijalankan.
2. Baris aktif (`Resto = "BCA - Kuta"`, `Area = "BALI"`, `PIC = "Made"`, ...) menjadi filter pada tabel `Resto`.
3. Relationship meneruskan filter ke `Deviasi All` — hanya transaksi resto tersebut yang terlihat.
4. `SUM('Deviasi All'[Value])` menghasilkan deviasi khusus resto itu — nilainya berbeda di setiap baris kolom.

### Diagram Context Transition

```text
ROW CONTEXT: baris Resto "BCA - Kuta"
        │   panggil [Deviasi CM]  (measure = CALCULATE implisit)
        ▼
CONTEXT TRANSITION: semua kolom baris aktif menjadi filter
        │   Resto="BCA - Kuta", Area="BALI", PIC="Made", ...
        ▼
FILTER CONTEXT BARU (meng-override filter lama pada kolom yang sama)
        │   relationship: Resto ──► Deviasi All
        ▼
[Deviasi CM] dievaluasi hanya pada transaksi resto tersebut
```

### Kapan Terjadi

| Situasi | Context transition? |
|---|---|
| Memanggil `[Measure]` di dalam iterator / calculated column | Ya — implisit |
| Menulis `CALCULATE(...)` di dalam row context | Ya — eksplisit |
| Ekspresi agregasi kolom biasa (`SUM('T'[C])`) di dalam row context | **Tidak** |
| Measure dipanggil langsung dari visual (tanpa iterator) | Tidak relevan — sudah berupa filter context |

## ALL / REMOVEFILTERS — Mereset Filter Context

```dax
-- % kontribusi tiap Area terhadap total keseluruhan
% Kontribusi =
DIVIDE (
    [Deviasi CM],
    CALCULATE ( [Deviasi CM], REMOVEFILTERS ( 'Resto' ) )
)
```

- `REMOVEFILTERS('Resto')` (atau `ALL('Resto')`) menghapus semua filter dari tabel `Resto` pada denominator — filter Area dari visual hilang, tetapi filter `Date`/`Periode` tetap berlaku.
- Inilah pola standar "% of total": reset context dengan **scope paling minimal** yang dibutuhkan, bukan seluruh model. Detail filter modifier: [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md).

## Sintaks & Contoh

```dax
-- ========== STUDI KASUS: total magnitudo deviasi ==========
-- Pertanyaan: berapa total risiko deviasi (abaikan tanda) di area ini?

-- Versi 1 (SALAH): absolut SETELAH dijumlah
-- resto A surplus +12 jt, resto B loss -10 jt → |2| jt → risiko understated!
Total Abs Salah = ABS ( SUM ( 'Deviasi All'[Value] ) )

-- Versi 2 (BENAR): absolut per resto, baru dijumlah
Total Abs =
SUMX (
    VALUES ( 'Resto'[Resto] ),   -- tabel yang diiterasi (per resto)
    ABS ( [Deviasi CM] )         -- context transition per baris resto
)
-- |12| + |10| = 22 jt

-- Alur evaluasi Versi 2:
-- 1) VALUES menghasilkan resto yang terlihat di filter context saat ini
-- 2) SUMX membuat row context per baris resto
-- 3) [Deviasi CM] dipanggil → context transition → deviasi resto itu
-- 4) ABS diterapkan per resto → SUMX menjumlahkan hasilnya

-- ========== Measure debug: lihat context yang sedang aktif ==========
Baris Terlihat = COUNTROWS ( 'Deviasi All' )
Resto Terlihat = COUNTROWS ( VALUES ( 'Resto'[Resto] ) )
```

## Tabel Perbandingan

| Aspek | Filter Context | Row Context |
|---|---|---|
| Definisi | Himpunan filter aktif saat evaluasi | Baris aktif saat evaluasi ekspresi |
| Sumber | Visual, slicer, CALCULATE, relationship | Calculated column, iterator |
| Menentukan | Baris tabel mana yang terlihat agregasi | Kolom mana yang dibaca, per baris |
| Mempengaruhi `SUM`/`COUNT`? | Ya — langsung | Tidak, kecuali lewat context transition |
| Cara mengubah | `CALCULATE` + filter modifier (`ALL`, `KEEPFILTERS`, ...) | Diciptakan iterator, berakhir saat iterasi selesai |
| Jumlah hasil | Satu nilai agregat per sel visual | Satu nilai per baris |
| Analogi SQL | klausa `WHERE` | kursor `FOREACH ROW` |

## Kesalahan Umum

- **Mengira filter di baris visual = row context.** Baris pada visual membuat *filter context*, bukan row context. Measure tidak pernah "melihat baris" — ia hanya merespons filter. Row context hanya ada di calculated column dan iterator.
- **Lupa context transition.** Measure dipanggil di dalam iterator dan hasilnya janggal — ingat: setiap pemanggilan measure di dalam iterator dievaluasi dalam filter context hasil transisi per baris, bukan dalam context pemanggil.
- **Mengira agregasi di calculated column otomatis per baris.** `SUM('Deviasi All'[Value])` di calculated column menghasilkan total seluruh tabel, sama di semua baris. Perbaikan: gunakan measure, atau bungkus `CALCULATE` untuk memancing context transition.
- **Bingung total visual tidak sama dengan jumlah baris.** Total dievaluasi ulang tanpa filter terdalam; measure non-aditif tidak "dijumlahkan" begitu saja.
- **Menyalahkan data padahal context berubah.** Angka yang "berubah sendiri" antar halaman biasanya efek slicer/sync-slicers — audit dengan measure debug `COUNTROWS`.

## Tips Pro

- Selalu sediakan measure debug `Baris Terlihat = COUNTROWS('Deviasi All')` di halaman kerja — cara tercepat "melihat" filter context yang sedang aktif.
- Rumus mental: **filter context = WHERE, row context = foreach**.
- Tulis iterator bertahap dengan `VAR` (tabel → agregat → hasil) seperti pola [Z Score] report user — logika context menjadi eksplisit dan mudah diuji.
- Iterasi dimensi (`VALUES('Resto'[Resto])`) jauh lebih murah daripada iterasi fakta (`'Deviasi All'`) — lihat [Fungsi Iterator](./05-iterator-functions.md).
- Performance Analyzer + DAX Studio memperlihatkan query persis yang dikirim visual — context per sel terlihat telanjang di sana.

## Terkait

- [Sintaks DAX & Tipe Data](./01-syntax-and-data-types.md)
- [Measures vs Calculated Columns](./03-measures-vs-calculated-columns.md)
- [Fungsi Iterator](./05-iterator-functions.md)
- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md)
- [Time Intelligence](./07-time-intelligence.md)
