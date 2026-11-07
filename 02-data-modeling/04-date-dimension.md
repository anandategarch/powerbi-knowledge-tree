# Date Dimension — Tabel Kalender Wajib

> Tabel Date custom adalah prasyarat time intelligence yang benar, hierarki tanggal yang rapi, dan format bulan Indonesia. Tanpa tabel Date yang di-mark, fungsi DAX tanggal bisa salah atau error, dan model membengkak karena tabel tanggal otomatis tersembunyi.

## Kenapa Wajib Date Table Custom

- **Time intelligence butuh tanggal kontinu** — fungsi seperti `DATEADD`, `SAMEPERIODLASTYEAR`, dan `DATESYTD` membutuhkan rentang tanggal tanpa bolong (contiguous). Tanggal dari `DISTINCT` kolom transaksi bolong di hari tanpa transaksi → perhitungan periode sebelumnya meleset.
- **Fiscal period & periode custom** — minggu WEEK 1–EOM, tahun fiskal, dan mapping periode bisnis tidak bisa diwakili kolom datetime mentah.
- **Hierarki rapi** — Year → Month → Date yang konsisten untuk drill-down.
- **Satu dimensi untuk semua fact** — Deviasi, Waste, Sales, dan PO berbagi `Date` yang sama (conformed dimension), sehingga slicer periode bekerja seragam di seluruh report.

## Cara Membuat: DAX vs Power Query vs Manual

### 1. DAX — CALENDARAUTO / CALENDAR (dipakai user)

```dax
Date = CALENDARAUTO()
```

`CALENDARAUTO()` memindai semua kolom date/datetime hasil load Power Query (calculated column dan calculated table **tidak** ikut dipindai), lalu membuat rentang tanggal kontinu dari tanggal minimum sampai maksimum. Kolom turunannya (`Tahun`, `Bulan`, dst.) dibuat sebagai calculated columns.

`CALENDAR` lebih eksplisit bila rentang ingin dikunci:

```dax
Date = CALENDAR ( DATE ( 2024, 1, 1 ), DATE ( 2026, 12, 31 ) )
```

Catatan: hapus kolom datetime yang tidak dipakai (mis. timestamp `Modified`) agar rentang CALENDARAUTO tidak melebar tak perlu.

### 2. Power Query

Dibangun dengan `List.Dates` (lihat resep di [PQ Patterns](../03-power-query/04-pq-patterns-recipes.md)). Keunggulan: kolom sudah bertipe pasti sebelum masuk model, dan mudah di-merge dengan tabel hari libur eksternal.

### 3. Tabel manual / dari sumber

Import tabel kalender dari Google Sheets — cocok kalau tim bisnis ikut merawat mapping periode dan hari libur.

## Kolom Penting

| Kolom | Contoh | Fungsi |
|---|---|---|
| `Tahun` | 2025 | Slicer tahun |
| `Bulan` | Jan, Feb | Axis bulan (sort by `BulanNo`) |
| `BulanNo` | 1–12 | Sort key bulan |
| `Kuartal` | Q1 | Grouping kuartal |
| `TahunBulan` | 2025-01 | Label axis kontinu |
| `Minggu` / `Periode` | WEEK 1 | Grain mingguan model user |
| `TahunFiskal` | FY25 | Fiscal year (offset bulan mulai) |
| `IsWeekend` | TRUE | Filter analitik |
| `IsHoliday` | FALSE | Merge tabel hari libur |

## Mark as Date Table (Wajib!)

1. Pilih tabel `Date` → **Table tools → Mark as date table** → pilih kolom `Date`.
2. Syarat validasi: kolom tanggal **unik, tidak blank, kontinu, tanpa bagian jam**.
3. Setelah di-mark, fungsi time intelligence dijamin memakai tabel Anda.

Bila auto date/time dimatikan secara global (memang harus — lihat bawah), **mark as date table menjadi wajib**; kalau tidak, fungsi time intelligence akan error.

## Sort By Column — Termasuk Periode

Label teks diurutkan alfabetis secara default, padahal urutan bisnis berbeda. Solusinya **Sort by column**:

1. Buat kolom urutan numerik.
2. Pilih kolom label → **Column tools → Sort by column** → pilih kolom urutan.

Contoh dari model user — tabel `Periode` sebagai calculated table lalu di-sort:

```dax
Periode =
DATATABLE (
    "Periode", STRING,
    "Sort", INTEGER,
    {
        { "WEEK 1", 1 },
        { "WEEK 2", 2 },
        { "WEEK 3", 3 },
        { "EOM", 4 }
    }
)
-- Lalu: pilih kolom Periode -> Column tools -> Sort by column -> Sort
```

Tanpa kolom `Sort`, dropdown Periode tampil "EOM, WEEK 1, WEEK 2, WEEK 3" (alfabetis) — salah secara bisnis. Dengan sort by column: WEEK 1 → WEEK 2 → WEEK 3 → EOM. Prinsip sama berlaku untuk `Bulan` (sort by `BulanNo`).

## Hierarchy Year → Month → Date

Di Model view, drag kolom `Bulan` ke `Tahun`, lalu `Date` ke `Bulan` → hierarki Year-Month-Date. Visual bar/line otomatis mendukung drill-down. Jangan pakai hierarki otomatis auto date/time — buat manual agar nama, urutan, dan isi terkontrol penuh.

## Auto Date/Time: Kenapa Dimatikan

Fitur ini otomatis membuat tabel tanggal tersembunyi **untuk setiap kolom datetime** di model. Akibatnya:

- Ukuran file membengkak (bisa 2–3x untuk model dengan banyak kolom tanggal).
- Muncul hierarki ganda di panel Fields yang membingungkan.
- Time intelligence lintas fact jadi tidak konsisten — tiap kolom punya kalender sendiri.

Matikan: **File → Options and settings → Options → Current File → Data Load → uncheck "Auto date/time for new files"**. Kompensasinya: tabel `Date` custom + Mark as date table.

## Sintaks & Contoh — Format Indonesia

Dua pendekatan nama bulan/hari Indonesia:

```dax
-- 1) FORMAT dengan locale id-ID (dipakai sebagai calculated column)
Bulan = FORMAT ( 'Date'[Date], "MMM", "id-ID" )   -- Jan, Feb, ..., Des
Hari  = FORMAT ( 'Date'[Date], "ddd", "id-ID" )   -- Sen, Sel, ...

-- 2) Mapping manual via SWITCH (paling terkontrol)
Bulan = SWITCH (
    MONTH ( 'Date'[Date] ),
    1, "Jan", 2, "Feb", 3, "Mar",  4, "Apr",  5, "Mei",  6, "Jun",
    7, "Jul", 8, "Agu", 9, "Sep", 10, "Okt", 11, "Nov", 12, "Des"
)

-- Contoh tabel Date lengkap (pola calculated columns setelah CALENDARAUTO)
TahunBulan = FORMAT ( 'Date'[Date], "yyyy-MM" )
Kuartal    = "Q" & FORMAT ( 'Date'[Date], "Q" )
IsWeekend  = WEEKDAY ( 'Date'[Date], 2 ) >= 6
```

Untuk label "Jan 25" gunakan `FORMAT('Date'[Date], "MMM yy", "id-ID")`. Selalu siapkan `BulanNo` sebagai sort key agar urutan tidak alfabetis.

## Kesalahan Umum

- **Lupa Mark as date table** padahal auto date/time sudah dimatikan → fungsi time intelligence error. Perbaikan: mark tabel + pastikan kolom Date unik dan kontinu.
- **Tabel tanggal dari `DISTINCT` transaksi** — ada bolong di tanggal tanpa penjualan; line chart terputus dan `SAMEPERIODLASTYEAR` meleset. Perbaikan: pakai `CALENDARAUTO`/`CALENDAR`/`List.Dates` agar kontinu.
- **Rentang CALENDARAUTO aneh (tahun 1899/2099)** — ada kolom datetime sampah (mis. angka `0` dikonversi jadi 30 Desember 1899) di salah satu fact. Perbaikan: hapus/perbaiki kolom itu, atau pakai `CALENDAR` eksplisit.
- **Kolom tanggal bertipe teks** ("01/02/2025") → tidak bisa dijadikan relasi. Perbaikan: change type dengan locale di Power Query.
- **Sort by column gagal** karena kolom sort tidak unik per label (mis. "W1" muncul di dua tahun). Perbaikan: jadikan label unik per tahun atau sort key gabungan.
- **Dua relasi aktif ke Date dari dua kolom tanggal** — jadikan satu inactive + `USERELATIONSHIP`.

## Tips Pro

- Simpan grain periode mingguan sebagai dimensi `Periode` terpisah (seperti user) — jangan nomor minggu bebas dalam teks; sort key menjaga urutan.
- `IsHoliday` lebih mudah dirawat lewat merge di Power Query ke tabel hari libur di Google Sheets, bukan hardcode di DAX.
- Bila report mingguan (WEEK 1–EOM), buat kolom `PeriodeIndex` berurutan agar perbandingan "periode vs periode sebelumnya" tidak terikat kalender.
- Kolom `TahunBulan` ("2025-01") sangat membantu untuk axis kontinu dan sort aman lintas tahun.

## Terkait

- [Star Schema](./01-star-schema.md)
- [Relationships](./02-relationships.md)
- [Modeling Best Practices](./05-modeling-best-practices.md)
- [Custom calendar di Power Query](../03-power-query/04-pq-patterns-recipes.md)
- [Change type & locale](../03-power-query/02-common-transformations.md)
- [CALCULATE](../01-dax/06-calculate-and-filter-functions.md)
- [Analisis model report cost control](../11-user-report-analysis/model-and-dax.md)
