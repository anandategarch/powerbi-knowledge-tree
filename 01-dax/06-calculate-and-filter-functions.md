# CALCULATE & Filter Functions

> `CALCULATE` adalah fungsi terpenting di DAX — satu-satunya cara memodifikasi **filter context** sebelum expression dievaluasi. Measure analitik seperti filter per jenis deviasi, % kontribusi, hingga mode perbandingan dinamis semuanya dibangun di atasnya, bersama filter modifier `ALL`, `REMOVEFILTERS`, `KEEPFILTERS`, dan kawan-kawannya.

## CALCULATE

### Cara Kerja

```dax
CALCULATE ( <expression>, <filter1>, <filter2>, ... )
```

Urutan evaluasi — kunci memahami hasilnya:

1. **Filter argumen dievaluasi lebih dulu di filter context LAMA** — context yang aktif saat `CALCULATE` dipanggil, sebelum modifikasi apa pun diterapkan.
2. Bila ada row context → **context transition**: nilai baris aktif menjadi filter (meng-override filter lama pada kolom yang sama). Detail: [Row Context vs Filter Context](./02-row-context-vs-filter-context.md).
3. Filter hasil langkah 1 **diterapkan**: pada kolom yang sama → filter lama **diganti** (override); pada kolom berbeda → **digabungkan AND** dengan filter yang sudah ada.
4. `<expression>` dievaluasi di filter context baru; setelah selesai, context lama dipulihkan.

### Boolean Filter vs Table Filter

```dax
-- Boolean filter: satu kolom, satu kondisi — ringkas & cepat
Waste CM =
CALCULATE ( [Deviasi CM], 'Deviasi All'[Jenis] = "WASTE" )

-- Secara internal di-expand menjadi:
Waste CM =
CALCULATE (
    [Deviasi CM],
    FILTER ( ALL ( 'Deviasi All'[Jenis] ), 'Deviasi All'[Jenis] = "WASTE" )
)
```

| Aspek | Boolean filter | Table filter (`FILTER`) |
|---|---|---|
| Bentuk | `'T'[Kolom] = "nilai"` | `FILTER ( tabel, kondisi per baris )` |
| Cakupan | Satu kolom per predicate | Bebas — banyak kolom, perbandingan antar baris |
| Bisa memakai measure? | Tidak — error | Ya (lewat iterator) |
| Performa | Dioptimalkan storage engine | Tergantung granularity tabel yang difilter |

Kapan **wajib** `FILTER`:

```dax
-- 1. Kondisi membandingkan hasil agregasi (butuh evaluasi per baris)
Jumlah Resto Rugi =
COUNTROWS ( FILTER ( VALUES ( 'Resto'[Resto] ), [Deviasi CM] < 0 ) )

-- 2. Filter yang bergantung pada measure (predicate boolean tidak boleh memuat measure)
Deviasi Bulan Laku =
CALCULATE (
    [Deviasi CM],
    FILTER ( VALUES ( 'Date'[Month] ), [Sales CM] > 0 )
)
```

### Filter Kolom Sama: Override, Bukan Ditambah

```dax
-- Slicer: Area = "BALI". Lalu measure:
Deviasi JATIM = CALCULATE ( [Deviasi CM], 'Resto'[Area] = "JATIM" )
-- Hasil: hanya JATIM — filter "BALI" DIGANTI, bukan digabung.
```

### KEEPFILTERS — Intersection, Bukan Override

```dax
-- Slicer: Area ∈ { "BALI", "JATIM" }
Tanpa KEEPFILTERS =
CALCULATE ( [Deviasi CM], 'Resto'[Area] IN { "JATIM", "JABAR" } )   -- → JATIM + JABAR
Dengan KEEPFILTERS =
CALCULATE ( [Deviasi CM], KEEPFILTERS ( 'Resto'[Area] IN { "JATIM", "JABAR" } ) )
-- → irisan { BALI, JATIM } ∩ { JATIM, JABAR } = hanya JATIM
```

Gunakan `KEEPFILTERS` setiap kali Anda ingin filter baru **dipersempit oleh** pilihan user, bukan menggantikannya.

### Context Transition dalam CALCULATE

`CALCULATE` adalah satu-satunya pemicu context transition — eksplisit, atau implisit lewat pemanggilan measure:

```dax
-- Hitung berapa resto yang rugi (iterator + CALCULATE per baris)
Jumlah Resto Rugi =
SUMX (
    VALUES ( 'Resto'[Resto] ),
    IF ( CALCULATE ( [Deviasi CM] ) < 0, 1, 0 )
)
```

### CALCULATETABLE

Versi tabel dari `CALCULATE` — mengembalikan **tabel** yang dievaluasi dalam context baru. Dipakai untuk membangun tabel virtual, mis. pattern [Z Score] report user:

```dax
VAR SummaryTable =
    CALCULATETABLE (
        SUMMARIZE ( 'Deviasi All', 'Date'[Date] ),
        ALLSELECTED ( 'Deviasi All' )
    )
```

## Filter Modifiers

| Function | Deskripsi | Contoh |
|---|---|---|
| `ALL` | Menghapus semua filter dari tabel/kolom; juga table function (mengembalikan seluruh nilai) | `CALCULATE ( [Deviasi CM], ALL ( 'Resto' ) )` |
| `REMOVEFILTERS` | Identik dengan `ALL` sebagai filter modifier — semantik lebih jelas | `CALCULATE ( [Deviasi CM], REMOVEFILTERS ( 'Resto' ) )` |
| `ALLEXCEPT` | Menghapus semua filter dari tabel KECUALI kolom yang disebut | `CALCULATE ( [Deviasi CM], ALLEXCEPT ( 'Resto', 'Resto'[Area] ) )` |
| `ALLSELECTED` | Menghapus filter internal visual, mempertahankan seleksi luar (slicer) | `CALCULATE ( [Deviasi CM], ALLSELECTED ( 'Resto'[Resto] ) )` |
| `KEEPFILTERS` | Filter argumen di-iriskan (intersect) dengan filter existing | `CALCULATE ( [Deviasi CM], KEEPFILTERS ( ... ) )` |
| `ALLNOBLANKROW` | Seperti `ALL` tetapi mengecualikan blank row dari referential integrity | `ALLNOBLANKROW ( 'Resto' )` |

### ALL vs REMOVEFILTERS

Sebagai filter modifier keduanya identik — bedanya `ALL` punya peran kedua sebagai **table function** (`ALL ( 'Resto' )` bisa dipakai sebagai iterator). Best practice modern: tulis `REMOVEFILTERS` bila tujuannya menghapus filter (lebih eksplisit), dan `ALL` bila Anda memang butuh tabel seluruh nilai.

### ALLEXCEPT — Reset dengan Pengecualian

```dax
-- % kontribusi resto terhadap areanya (Area tetap mengikuti slicer)
% Kontribusi Area =
DIVIDE (
    [Deviasi CM],
    CALCULATE ( [Deviasi CM], ALLEXCEPT ( 'Resto', 'Resto'[Area] ) )
)
```

Filter `Resto[Resto]` dihapus (semua resto di area itu dihitung), tetapi filter `Resto[Area]` dipertahankan.

### ALLSELECTED — Kuat tapi Rumit

`ALLSELECTED` mengembalikan nilai yang "terlihat" berdasarkan seleksi luar (slicer, filter page) sambil menghapus filter internal visual (baris/kolom matriks) — fondasi pola "% of selected total". Peringatan: perilakunya rumit pada visual bertingkat; selalu uji hasilnya di matrix multi-level.

### ALLNOBLANKROW — Blank Row Referential Integrity

Bila `Deviasi All` memuat nama resto yang tidak ada di master `Resto`, engine menambahkan **blank row** pada `Resto` supaya relationship tetap valid. `ALLNOBLANKROW` mengembalikan seluruh nilai tanpa blank row tersebut — berguna saat menghitung "semua resto valid".

## VALUES vs DISTINCT

| Aspek | `VALUES` | `DISTINCT` |
|---|---|---|
| Nilai unik di context saat ini | Ya | Ya |
| Blank row dari referential integrity | Termasuk | Tidak termasuk |
| Blank dari data (nilai kosong asli) | Termasuk | Termasuk |

```dax
Resto Aktif    = COUNTROWS ( DISTINCT ( 'Deviasi All'[Resto] ) )  -- tanpa blank row
Resto Master   = COUNTROWS ( VALUES ( 'Resto'[Resto] ) )
```

Keduanya lazim dipakai sebagai iterator; `VALUES` lebih "lengkap" (ikut blank row), `DISTINCT` lebih bersih.

## ISFILTERED, HASONEFILTER, HASONEVALUE, ISINSCOPE (Singkat)

| Function | Pertanyaan yang dijawab | Pemakaian tipikal |
|---|---|---|
| `ISFILTERED ( 'T' )` / `ISFILTERED ( 'T'[c] )` | Apakah tabel/kolom sedang punya filter (langsung)? | Branching mode perbandingan (pattern `[Deviasi CM]` report user) |
| `HASONEVALUE ( 'T'[c] )` | Apakah tepat satu nilai yang visible? | Guard `SELECTEDVALUE`, judul dinamis |
| `HASONEFILTER ( 'T'[c] )` | Apakah tepat satu filter pada kolom? | Debugging — mirip `HASONEVALUE` tapi menghitung filter eksplisit |
| `ISINSCOPE ( 'T'[c] )` | Apakah kolom menjadi level grouping visual saat ini? | Guard ranking/total di matrix |

```dax
-- SELECTEDVALUE = jalan pintas HASONEVALUE + VALUES
Judul Resto = SELECTEDVALUE ( 'Resto'[Resto], "Semua Resto" )
```

## TREATAS (Overview Singkat)

`TREATAS` menerapkan nilai dari satu kolom/tabel sebagai filter pada kolom lain — **virtual relationship** tanpa relationship fisik di model:

```dax
-- Pattern report user: mode perbandingan tanggal dinamis
Deviasi Mode A =
CALCULATE (
    [Deviasi Value],
    TREATAS ( VALUES ( 'Date A'[Date] ), 'Date'[Date] )
)
```

Pembahasan penuh virtual relationship ada di [Relational Functions & TREATAS](./08-relational-functions.md).

## Sintaks & Contoh — Contoh Bertingkat

```dax
-- 1. Filter satu kolom
Waste CM = CALCULATE ( [Deviasi CM], 'Deviasi All'[Jenis] = "WASTE" )

-- 2. Beberapa filter → selalu dikombinasikan AND
Waste Nominal EOM =
CALCULATE (
    [Deviasi CM],
    'Deviasi All'[Jenis] = "WASTE",
    Attribute[Attribute] = "NOMINAL",
    'Periode'[Periode] = "EOM"
)

-- 3. % of total dengan reset scope minimal
% Kontribusi Area =
DIVIDE (
    [Deviasi CM],
    CALCULATE ( [Deviasi CM], REMOVEFILTERS ( 'Resto'[Resto] ) )
)

-- 4. KEEPFILTERS: irisan dengan pilihan user
Deviasi Prioritas =
CALCULATE (
    [Deviasi CM],
    KEEPFILTERS ( 'Resto'[Area] IN { "BALI", "JATIM" } )
)

-- 5. Pattern [Deviasi YTD] report user: VAR + reset + predicate bertingkat
Deviasi YTD =
VAR SelectedYear = SELECTEDVALUE ( 'Date'[Year] )
VAR SelectedMonth = SELECTEDVALUE ( 'Date'[MonthNumber] )
RETURN
    CALCULATE (
        [Deviasi CM],
        REMOVEFILTERS ( 'Date' ),
        REMOVEFILTERS ( 'Periode' ),
        'Date'[Year] = SelectedYear,
        'Date'[MonthNumber] <= SelectedMonth
    )
```

## Kesalahan Umum

- **`FILTER` seluruh tabel fakta sebagai filter argumen** — `CALCULATE ( [m], FILTER ( 'Deviasi All', ... ) )` mengevaluasi kondisi pada jutaan baris dan memfilter semua kolom tabel sekaligus. Perbaikan: filter pada granularity kolom — `FILTER ( ALL ( 'Deviasi All'[Kolom] ), ... )` atau `FILTER ( VALUES ( dimensi ), [measure] ... )`.
- **Lupa `KEEPFILTERS` saat butuh intersection** — `CALCULATE` meng-override filter pada kolom yang sama; bila maksudnya mempersempit pilihan user, bungkus filter dengan `KEEPFILTERS`.
- **Mengira filter argumen dievaluasi di context BARU** — semua predicate dievaluasi di context lama; `SELECTEDVALUE`/measure di dalam predicate merefleksikan context pemanggil, bukan hasil filter yang sedang dibangun.
- **Predicate boolean berisi measure** — `'Resto'[Resto] = [Top Resto]` error (*"A function 'CALCULATE' has been used in a True/False expression..."*) karena measure adalah `CALCULATE` implisit. Perbaikan: simpan nilai ke `VAR` lalu bandingkan di dalam `FILTER`, atau gunakan `TREATAS`.
- **`ALL ( 'Tabel Fakta' )` untuk % of total** — ikut menghapus filter tanggal → total sepanjang masa. Scope reset harus minimal: kolom/tabel dimensi tertentu saja.
- **Dua predicate pada kolom yang sama diharapkan OR** — keduanya di-AND-kan (irisan) dan biasanya menghasilkan kosong; gunakan satu predicate `IN { ... }` untuk OR pada satu kolom.

## Tips Pro

- Mulai dari filter boolean sederhana; turun ke `FILTER` hanya bila terpaksa — kode lebih pendek dan lebih mudah dioptimalkan storage engine.
- Pilih nama sesuai maksud: `REMOVEFILTERS` untuk "hapus filter", `ALL` untuk "ambil seluruh tabel" — formula menjadi self-documenting.
- Guard setiap measure ranking/rasio dengan `HASONEVALUE`/`ISINSCOPE` agar baris total tidak menampilkan angka menyesatkan.
- Pattern `[Deviasi CM]` report user (`ISFILTERED` + `TREATAS`) adalah contoh nyata *dynamic comparison mode* — pelajari sebelum membangun fitur serupa.
- DAX Studio → *Query Plan*: predicate yang diterjemahkan ke xmSQL dieksekusi storage engine (cepat); yang tertinggal di formula engine adalah kandidat optimasi berikutnya.

## Terkait

- [Row Context vs Filter Context](./02-row-context-vs-filter-context.md)
- [Fungsi Iterator](./05-iterator-functions.md)
- [Time Intelligence](./07-time-intelligence.md)
- [Relational Functions & TREATAS](./08-relational-functions.md)
- [Star Schema](../02-data-modeling/01-star-schema.md)
- [Analisis Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md)
