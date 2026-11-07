# Dynamic Filtering — Filter Dinamis dengan DAX

> Filter dinamis mengubah **perilaku query** lewat DAX: TREATAS untuk men-"injeksi" pilihan dari tabel disconnected (pola `[Deviasi CM]` report ini), what-if parameter untuk measure yang menghitung ulang, dan field parameters sebagai pengganti modern measure-switching.

## Konsep & Penjelasan

### Kenapa perlu filter dinamis

Filters pane dan slicer bekerja pada *data*. Filter dinamis bekerja pada *logika*: satu visual bisa membaca pilihan user (dari tabel yang bahkan tidak berelasi) lalu mengubah cara measure dihitung. Ini fondasi halaman *Antar Periode*: pilih periode pembanding bebas A/B, tanpa menduplikasi halaman.

### TREATAS pattern — pola `[Deviasi CM]` report ini

Inti polanya: ada **tabel disconnected** `Date A` (clone tabel `Date`, tanpa relasi). Slicer ditujukan ke `Date A`; measure membaca pilihan itu lalu "ditransplantasikan" ke `Date[Date]` yang sebenarnya melalui TREATAS.

Ini measure asli dari model (lihat [Model & DAX](../11-user-report-analysis/model-and-dax.md)):

```dax
Deviasi CM =
VAR Filtered = ISFILTERED ( 'Mode'[Mode] )
RETURN
IF (
    Filtered,
    CALCULATE (
        [Deviasi Value],
        TREATAS (
            VALUES ( 'Date A'[Date] ),   -- pilihan user di tabel disconnected
            'Date'[Date]                 -- diterapkan ke tabel terhubung
        )
    ),
    [Deviasi Value]
)
```

Cara membacanya:

- `VALUES('Date A'[Date])` = daftar tanggal yang sedang terseleksi di slicer `Date A`.
- `TREATAS(..., 'Date'[Date])` menjadikan daftar itu sebagai filter atas `Date[Date]` **dalam konteks CALCULATE ini saja** — tanpa perlu relasi di model.
- `ISFILTERED('Mode'[Mode])` memutus: mode perbandingan aktif (pakai `Date A`) atau tidak (perilaku normal).
- Hasil: halaman bisa membandingkan **bulan ini vs bulan apa pun yang dipilih user** — persis kebutuhan halaman *Antar Periode* / *Perbandingan*.

Kapan TREATAS lebih baik daripada relasi: (1) model tidak boleh diubah (relasi sudah dipakai untuk hal lain), (2) butuh banyak-to-satu sementara arah relasi terkunci, (3) dua tabel sengaja tidak berhubungan (A/B comparison).

### Disconnected tables — `Mode`, `Trend`, `Attribute`, `Date A`

Report ini memakai empat tabel disconnected sebagai "remote control":

| Tabel | Isi | Fungsi |
|---|---|---|
| `Mode` | Mode pembanding | Toggle `ISFILTERED('Mode'[Mode])` di `Deviasi CM`/`BOM CM` |
| `Trend` | Kategori analisa | Measure switching: `Multi Value CM` mengarahkan visual ke DEVIASI/BOM/%BOM/%SALES/VARIANCE |
| `Attribute` | NOMINAL / QTY | Memilih satuan analisa; measure cek `SELECTEDVALUE(Attribute[Attribute])` |
| `Date A` | Tanggal pembanding | Sumber nilai TREATAS di atas |

Pola umumnya membaca pilihan dengan `SELECTEDVALUE` (BLANK jika multi-select) atau `ISFILTERED` (deteksi "apakah field ini sedang difilter sama sekali").

### What-if parameter — measure menghitung ulang

What-if parameter (Modeling → New parameter → **Numeric range**) membuat: tabel kecil hasil `GENERATESERIES`, measure pembaca nilai, dan slider. Setiap geseran slider memicu evaluasi ulang semua measure yang membacanya.

Dua kegunaan paling umum:

1. **Top N dinamis** — panel Top N filter hanya menerima N statis, jadi N dinamis dibuat di dalam measure (pola "blank-out"): baris di luar Top N bernilai BLANK sehingga otomatis hilang dari visual.
2. **Target persentase** — garis target di combo chart dan ambang klasifikasi tanpa perlu edit model.

### Field parameters — pengganti modern measure-switching

Field parameters (Modeling → New parameter → **Fields**) membuat satu slicer yang **mengganti field/metric di visual**: pilih `[Deviasi CM]`, `[BOM CM]`, atau `[% BOM CM]` — axis/value visual berganti mengikutinya.

Pola lama report ini (`Trend` + `Multi Value CM` dengan `SWITCH`) adalah pendahulunya. Field parameters unggul ketika:

- butuh ganti **dimensi** (Area ↔ PIC ↔ Resto) atau **metric** tanpa menulis SWITCH;
- urutan dan label bisa diatur langsung di tabel parameter;
- konsisten di banyak visual sekaligus (satu slicer mengendalikan semuanya).

Batasan: tidak semua slot visual menerima field parameter; dan switching tetap terbatas pada daftar field yang didaftarkan.

### Cross-filtering dari tabel mentah

Tabel data mentah (mis. halaman *Data Deviasi*) bisa jadi sumber filter: klik satu baris `Resto` → matrix dan chart di halaman itu terfilter. Atur lewat **Format → Edit interactions** per visual: *Filter* (potong data), *Highlight* (redupkan yang lain), atau *None* (kebal).

Pola berguna: tabel "mentah" sebagai navigator — baris yang diklik menjadi konteks semua visual pendukung, lebih murah daripada slicer tambahan.

### Filter baris + conditional formatting

Gabungkan filter visual-level dengan conditional formatting untuk menandai baris yang lolos kriteria:

- Visual filter: `[Deviasi CM] is less than 0` (hanya resto loss — lihat [Filter Types](./01-filter-types.md)).
- Conditional formatting (background color, rules): `[Deviasi CM] < 0 → merah`, `[Z Score]` anomali → ikon warning.

Keduanya membaca measure yang sama, jadi konsisten otomatis.

### Security filter (pengantar)

RLS (Row-Level Security) pada dasarnya adalah **filter yang menempel pada user** dan dievaluasi sebelum DAX visual berjalan — karena itu RLS tidak bisa dimatikan lewat interaksi report. Detail lengkap di [RLS](../08-security/01-rls.md).

### Dynamic M query parameter (DirectQuery)

Untuk DirectQuery, parameter bisa diturunkan sampai query SQL sumber ("WHERE Year = @param"): buat parameter Power Query, rujuk di dalam native query, lalu tautkan slicer ke parameter tersebut (aktifkan fitur Dynamic M query parameters di Options; letak menunya berbeda antar versi). Gunakan hanya di DirectQuery — dan hati-hati SQL injection: jangan string-concat input bebas; biarkan nilai datang dari slicer (enumerated values).

## Sintaks & Contoh

Top N dinamis dengan what-if (pola blank-out):

```dax
-- Dibuat otomatis oleh what-if parameter (tabel 'Parameter TopN'):
TopN Value = SELECTEDVALUE ( 'Parameter TopN'[Parameter TopN], 10 )

-- Measure yang hanya bernilai untuk Top N:
Deviasi Top N =
VAR N = SELECTEDVALUE ( 'Parameter TopN'[Parameter TopN], 10 )
VAR Ranking =
    RANKX ( ALLSELECTED ( 'Resto'[Resto] ), [Deviasi CM], , DESC )
RETURN
    IF ( Ranking <= N, [Deviasi CM] )   -- di luar Top N → BLANK → baris hilang
```

Target persentase dinamis (ambang susut):

```dax
Target Susut % = SELECTEDVALUE ( 'Parameter Target'[Parameter Target], 0.03 )

Status Susut =
IF ( [% Susut] > [Target Susut %], "Over Target", "OK" )
```

Measure switching pola lama vs field parameter:

```dax
-- POLA LAMA (yang dipakai report ini, tabel Trend):
Multi Value CM =
SWITCH ( SELECTEDVALUE ( 'Trend'[Trend] ),
    "DEVIASI", [Deviasi CM],
    "BOM",      [BOM CM],
    "%BOM",     [% BOM CM]
)

-- PENGGANTI MODERN: buat Field parameter berisi measure
-- [Deviasi CM], [BOM CM], [% BOM CM] → satu slicer mengganti metric visual.
```

TREATAS untuk kasus "relasi tak bisa" (filter dari tabel tanpa relasi):

```dax
-- Contoh: terapkan pilihan Area ke tabel fact tanpa mengubah model
CALCULATE (
    [Waste Value],
    TREATAS ( VALUES ( 'Resto'[Area] ), 'Waste All'[Area] )
)
```

## Kesalahan Umum

- **Mengira panel Top N bisa dinamis** — nilai N di panel statis; yang dinamis harus lewat measure (pola blank-out di atas).
- **SELECTEDVALUE mengembalikan BLANK saat multi-select** — user memilih 2 bulan lalu measure "hilang"; tambahkan fallback (`, default`) atau gunakan `VALUES` + `ISINSCOPE` bila multi harus didukung.
- **TREATAS dipakai padahal relasi biasa memungkinkan** — relasi lebih murah dan dimengerti engine; TREATAS adalah jalan terakhir, bukan default.
- **Field parameter di-register tanpa urutan/label** — slicer tampil acak; atur kolom Order dan rename label di tabel parameter.
- **Tabel disconnected tidak di-hide** — muncul di field list dan membingungkan; hide tabel, tampilkan hanya slicernya.
- **What-if range terlalu lebar** — slider 1–1000 sulit disetel; sesuaikan increment dan min/max dengan kebutuhan nyata.
- **Dynamic M parameter di Import mode** — tidak berpengaruh; fitur ini DirectQuery only.

## Tips Pro

- Kombinasi **`Date A` + TREATAS + bookmark** memberi user "periode pembanding bebas" — ini pengganti elegan untuk menduplikasi halaman A/B; simpan preset kombinasi periode sebagai bookmark.
- Baca `VALUES` sebelum `TREATAS` — jika user belum memilih apa pun, `VALUES` mengembalikan semua baris (perilaku = tanpa filter). Beri default eksplisit jika periode kosong tidak boleh terjadi.
- What-if parameter hemat model: tabel `GENERATESERIES` hanya beberapa baris — jangan takut memakainya (bandingkan dengan calculated column, lihat [VertiPaq](../07-performance/01-vertipaq-engine.md)).
- Untuk switching metric yang sering dipakai lintas 36 halaman, field parameter + sync slicers menggantikan puluhan visual duplikat (lihat [Slicer Guide](./02-slicer-guide.md)).
- Dokumentasikan tabel disconnected di file overview model — tim baru sering bingung kenapa ada `Date A`/`Mode` tanpa relasi.

## Terkait

- [Filter Types](./01-filter-types.md) — filter pane vs logika DAX
- [Slicer Guide](./02-slicer-guide.md) — sync slicers untuk kontrol lintas halaman
- [Performance Analyzer](../07-performance/02-performance-analyzer.md) — mengukur biaya TREATAS
- [Optimization Checklist](../07-performance/03-optimization-checklist.md) — DAX checklist
- [RLS](../08-security/01-rls.md) — security filter lanjutan
- [Model & DAX](../11-user-report-analysis/model-and-dax.md) — definisi asli `Deviasi CM`, `Z Score`, `Multi Value CM`
