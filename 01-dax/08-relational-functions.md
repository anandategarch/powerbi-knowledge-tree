# Relational Functions: RELATED, USERELATIONSHIP, CROSSFILTER, TREATAS

> Fungsi relasional menghubungkan tabel lewat relasi fisik (RELATED, USERELATIONSHIP, CROSSFILTER) maupun relasi virtual (TREATAS, LOOKUPVALUE). Pattern `[Deviasi CM] = CALCULATE([Deviasi Value], TREATAS(VALUES('Date A'[Date]), 'Date'[Date]))` di report cost-control adalah contoh nyata relasi virtual yang membuat satu measure punya dua "mode".

## Konsep: Relasi Fisik vs Relasi Virtual

Model cost-control memakai dua gaya sekaligus:

- **Relasi fisik** — `Deviasi All` → `Inventory` (many-to-one) dan `Deviasi All` → `Resto` (many-to-many). Filter mengalir otomatis mengikuti relasi: murah, cepat, tapi kaku — satu kolom tanggal di fakta hanya bisa punya satu relasi aktif ke satu date table.
- **Relasi virtual** — tabel helper *disconnected* (`Date A`, `Date B`, `Mode`) sengaja TIDAK berelasi. Pilihan user di slicer-nya tidak memfilter visual; nilai terlihatnya "dilempar" manual ke tabel sungguhan di dalam measure. Inilah yang membuat slicer periode pembanding A/B dan toggle mode DINAMIS mungkin.

## Function Reference

| Function | Deskripsi | Contoh |
|---|---|---|
| RELATED | Ambil nilai skalar dari sisi *one* (dipanggil dari sisi *many*); butuh row context | `RELATED(Inventory[Nama Barang])` |
| RELATEDTABLE | Ambil tabel dari sisi *many* (dipanggil dari sisi *one*) | `COUNTROWS(RELATEDTABLE('Waste All'))` |
| USERELATIONSHIP | Aktifkan relasi inactive hanya di dalam CALCULATE | `USERELATIONSHIP('Date'[Date], 'PO'[Tanggal Terima])` |
| CROSSFILTER | Ubah arah/aktifkan relasi sementara | `CROSSFILTER(A[X], B[X], BOTH)` |
| TREATAS | Terapkan tabel nilai sebagai filter ke kolom lain (relasi virtual) | `TREATAS(VALUES('Date A'[Date]), 'Date'[Date])` |
| LOOKUPVALUE | Cari nilai skalar tanpa relasi sama sekali | `LOOKUPVALUE(Inventory[Nama Barang], Inventory[(Old) Nama Barang], '3243'[Nama Barang])` |
| NATURALINNERJOIN | Join dua tabel pada kolom bernama sama | `NATURALINNERJOIN(A, B)` |
| NATURALLEFTOUTERJOIN | Versi LEFT JOIN dari NATURALINNERJOIN | `NATURALLEFTOUTERJOIN(A, B)` |
| USERNAME / USERPRINCIPALNAME | Identitas user untuk RLS | `Resto[Email] = USERPRINCIPALNAME()` |

## RELATED dan RELATEDTABLE — Arah Menentukan Segalanya

Keduanya hanya jalan jika ada **row context** (calculated column, atau di dalam iterator seperti SUMX) dan hanya pada relasi **many-to-one**:

```dax
-- Calculated column di 'Deviasi All' (sisi many) → tarik master dari Inventory (sisi one)
Nama Barang Resmi = RELATED(Inventory[Nama Barang])

-- Calculated column di 'Inventory' (sisi one) → tabel fakta di sisi many
Jumlah Baris Deviasi = COUNTROWS(RELATEDTABLE('Deviasi All'))

-- Di dalam iterator (measure): tiap baris 'Resto' = row context
Total Waste per Resto =
SUMX(
    'Resto',
    SUMX(RELATEDTABLE('Waste All'), 'Waste All'[Value])
)
```

Pada relasi many-to-many (mis. `Deviasi All` → `Resto`) RELATED tidak tersedia — gunakan filter kolom biasa atau tabel virtual.

## USERELATIONSHIP — Dua Tanggal, Satu Tabel Fakta

Kasus klasik *role-playing dimension*: tabel PO punya **Tanggal Order** dan **Tanggal Terima Barang**, dua-duanya ingin dianalisis per tanggal. Hanya satu relasi ke `Date` yang boleh aktif:

```dax
-- Relasi aktif   : Date[Date] → 'PO'[Tanggal Order]
-- Relasi inactive: Date[Date] → 'PO'[Tanggal Terima]

-- Measure A (default, mengikuti relasi aktif):
Qty Order = SUM('PO'[QTY])

-- Measure B (pinjam relasi inactive):
Qty Diterima =
CALCULATE(
    SUM('PO'[QTY]),
    USERELATIONSHIP('Date'[Date], 'PO'[Tanggal Terima])
)
```

Aturan main:

- Hanya bisa dipakai sebagai **filter argument** CALCULATE/CALCULATETABLE.
- Relasi yang disebut harus **inactive**; dua relasi aktif ke tabel yang sama membuat model ambigu dan DAX menolak.
- Di luar CALCULATE tersebut, keadaan relasi kembali seperti semula — tidak ada efek sisa.

Alternatif tanpa relasi ganda: duplikat date table (`Date A`) + TREATAS — persis alasan report cost-control punya `Date A` dan `Date B`.

## CROSSFILTER — Ubah Arah Sesaat

```dax
-- Area → Resto → Deviasi: aktifkan dua arah HANYA untuk measure ini
Deviasi by Area (2 arah) =
CALCULATE(
    [Deviasi Value],
    CROSSFILTER(Resto[Resto], 'Deviasi All'[RESTO], BOTH)
)

-- Atau sebaliknya: matikan propagasi filter untuk measure tertentu
Total Tanpa Filter Resto =
CALCULATE(
    [Deviasi Value],
    CROSSFILTER(Resto[Resto], 'Deviasi All'[RESTO], NONE)
)
```

Gunakan untuk kebutuhan sesaat di measure. Mengeset "Both direction" permanen di model untuk kasus seperti ini adalah sumber ambiguity dan masalah performa.

## TREATAS — Relasi Virtual

**Mengapa TREATAS, bukan relasi fisik?**

1. **Kontrol per-measure.** Dengan relasi fisik, slicer `Date A` otomatis memfilter SEMUA visual. Dengan TREATAS, pilihan user hanya berlaku pada measure yang secara eksplisit "menerima" nilai tersebut.
2. **Perbandingan periode bebas.** `Date A` (periode A) dan `Date B` (periode B) dipilih independen dari filter `Date` utama — mustahil dengan relasi fisik, karena satu kolom tanggal fakta hanya bisa punya satu relasi aktif per date table.
3. **Performance.** Meneruskan `VALUES()` (nilai unik yang sudah tersaring, mis. satu bulan ≈ 30 tanggal) via TREATAS sering lebih murah daripada relasi many-to-many ganda atau cross-filter dua arah berantai.
4. **Many-to-many tanpa drama.** Filter diterapkan berbasis nilai, bukan arah relasi — tidak ada risiko filter merambat tak terduga.

Pattern asli [Deviasi CM]:

```dax
Deviasi CM =
VAR Filtered = ISFILTERED('Mode'[Mode])
RETURN
IF(
    Filtered,
    CALCULATE(
        [Deviasi Value],
        TREATAS(
            VALUES('Date A'[Date]),  -- nilai tanggal yang TERLIHAT di slicer Date A
            'Date'[Date]             -- diterapkan sebagai filter ke kolom Date utama
        )
    ),
    [Deviasi Value]                  -- mode normal: ikut filter 'Date' biasa
)
```

Cara baca: `VALUES('Date A'[Date])` membentuk tabel satu kolom berisi tanggal pilihan user; `TREATAS` menempelkan tabel itu ke `'Date'[Date]` seolah-olah user memfilter kolom tersebut; `CALCULATE` lalu mengevaluasi `[Deviasi Value]` dengan filter pengganti itu. `[Deviasi LM]` memakai pola yang sama dengan `Date B`.

Syarat penting: **tipe data kolom sumber dan target harus sama** (Date vs Date), dan argumen pertama harus **table expression** — hampir selalu `VALUES(...)` atau `ALL(...)`.

## LOOKUPVALUE — Cari Tanpa Relasi

```dax
-- Calculated column "Cek Barang" (asli di model): petakan nama lama → nama baru
Cek Barang =
LOOKUPVALUE(
    Inventory[Nama Barang],          -- kolom hasil yang diambil
    Inventory[(Old) Nama Barang],    -- kolom kunci di tabel tujuan
    '3243'[Nama Barang]              -- nilai kunci dari baris ini
)
```

- Tidak butuh relasi sama sekali — cocok untuk mapping sesekali atau validasi data.
- Bisa beberapa pasang kunci (digabung AND): `LOOKUPVALUE(hasil, kunci1, nilai1, kunci2, nilai2)`.
- Argumen terakhir opsional adalah hasil alternatif kalau tidak ketemu: `..., "TIDAK KETEMU"`.
- **Error kalau kunci menemukan lebih dari satu baris** — tambahkan pasangan kunci sampai unik.
- Di tabel besar, LOOKUPVALUE di calculated column memperlambat refresh; kalau pemetaannya permanen, relasi + RELATED lebih cepat karena materialized.

## NATURALINNERJOIN / NATURALLEFTOUTERJOIN (Singkat)

```dax
-- Join dua virtual table yang punya kolom bernama sama (tipe harus cocok)
VAR Master =
    SELECTCOLUMNS(
        VALUES(Inventory[(Old) Nama Barang]),
        "Nama", Inventory[(Old) Nama Barang]
    )
VAR AdaData =
    SELECTCOLUMNS(
        VALUES('Deviasi All'[Nama Barang]),
        "Nama", 'Deviasi All'[Nama Barang]
    )
VAR JoinLengkap =
    NATURALLEFTOUTERJOIN(Master, AdaData)  -- semua barang master, plus mana yang punya data
```

Batasan yang perlu diketahui: kedua tabel harus punya **kolom dengan nama sama**, dan join akan error bila kedua kolom punya **lineage** dari kolom sumber yang sama — kasus ini justru lebih mudah diselesaikan TREATAS atau GENERATE. Untuk kebutuhan sehari-hari di measure, prefer TREATAS.

## USERNAME / USERPRINCIPALNAME — RLS

```dax
-- Filter role "PIC Cabang" pada tabel Resto:
-- user hanya melihat baris cabang miliknya
Resto[Email PIC] = USERPRINCIPALNAME()
```

- `USERPRINCIPALNAME()` → selalu bentuk email (user@domain.com), konsisten di Desktop dan Service — **inilah yang disarankan untuk RLS**.
- `USERNAME()` → di Desktop mengembalikan `DOMAIN\user`, di Service mengembalikan UPN. Berguna untuk dev/test, membingungkan di produksi.
- Kombinasi kuat dengan master `Resto` (punya kolom PIC, AM, Email): mapping dinamis cabang per user tanpa duplikasi tabel.

## Kesalahan Umum

- **RELATED dari sisi "one", pada tabel tanpa relasi, atau lewat relasi many-to-many** → error. *Perbaikan*: pastikan dipanggil dari sisi many pada relasi many-to-one; dari sisi one gunakan RELATEDTABLE; cek relasi di Model view.
- **RELATED/RELATEDTABLE di measure tanpa iterator** → error "tidak ada row context". *Perbaikan*: bungkus dalam SUMX/ADDCOLUMNS/dll, atau pindahkan logika ke calculated column.
- **TREATAS tanpa VALUES** — `TREATAS('Date A'[Date], 'Date'[Date])` invalid karena argumen pertama harus tabel, bukan kolom. *Perbaikan*: `TREATAS(VALUES('Date A'[Date]), 'Date'[Date])`.
- **TREATAS antar kolom beda tipe data** (Date vs text) → filter tidak match, hasil kosong atau salah. *Perbaikan*: samakan tipe data kedua kolom.
- **USERELATIONSHIP ke relasi yang sudah aktif** → error. *Perbaikan*: relasi yang disebut harus berstatus inactive; kalau dua-duanya perlu aktif, pisahkan date table (role-playing).
- **CROSSFILTER / USERELATIONSHIP ditulis di luar filter argument CALCULATE** → error. *Perbaikan*: selalu sebagai argumen filter.
- **LOOKUPVALUE dengan kunci tidak unik** → error "beberapa nilai ditemukan". *Perbaikan*: tambah pasangan kunci pencari sampai unik.

## Tips Pro

- **Desain Mode DINAMIS**: tabel helper disconnected (`Mode` berisi satu baris "DINAMIS") + `ISFILTERED` + `TREATAS` = satu measure dua wajah (normal vs pembanding bebas). Semua measure turunan meniru pola yang sama agar konsisten.
- **Kumpulkan TREATAS di measure dasar**: [Deviasi CM] dan [BOM CM] sudah menangani mode; measure turunan cukup `CALCULATE([Deviasi CM], PREVIOUSMONTH(...))` — jangan duplikasi TREATAS di semua measure.
- **Ukur dengan Performance Analyzer**: TREATAS pada VALUES yang mengembalikan sedikit baris (satu bulan ≈ 30 tanggal) sangat murah; kalau VALUES-nya ribuan baris, pertimbangkan agregasi dulu.
- Jangan aktifkan cross-filter "Both" permanen hanya demi satu visual — pakai CROSSFILTER dalam measure.
- LOOKUPVALUE untuk kolom validasi (Cek Barang, Cek Resto) bagus untuk audit ETL; hindari di measure yang dievaluasi jutaan baris.

## Terkait

- [CALCULATE & Filter](./06-calculate-and-filter-functions.md) — tempat semua fungsi ini dipakai
- [Time Intelligence](./07-time-intelligence.md) — TREATAS untuk date table pembanding
- [Information Functions](./10-information-functions.md) — ISFILTERED untuk deteksi mode DINAMIS
- [Table Functions](./12-table-functions.md) — VALUES, CALCULATETABLE, GENERATE untuk virtual table
- [Data Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md) — relasi dan tabel disconnected di report asli
