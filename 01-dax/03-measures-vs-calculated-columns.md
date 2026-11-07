# Measures vs Calculated Columns

> Measure dievaluasi **on the fly** saat visual merequest angka (mengikuti filter context), sedangkan calculated column dihitung **sekali per baris saat refresh** dan disimpan permanen di VertiPaq. Aturan praktis: jika hasilnya bergantung pada pilihan filter user — bulan, area, resto — ia hampir pasti harus menjadi measure.

## Measure

### Definisi dan Karakteristik

- Formula dievaluasi **saat query** — ketika halaman dirender — bukan saat refresh dataset.
- Satu definisi menghasilkan nilai berbeda di setiap sel visual sesuai filter context.
- **Tidak menyimpan data** → tidak menambah ukuran file PBIX sama sekali.
- Menghasilkan satu nilai skalar per evaluasi.
- Muncul di panel Data dengan ikon kalkulator.

```dax
Deviasi CM = SUM ( 'Deviasi All'[Value] )

% BOM CM = DIVIDE ( [Deviasi CM], [Sales CM] )   -- rasio = selalu measure!
```

Karena `[% BOM CM]` bergantung pada `[Sales CM]` yang berubah tiap bulan/area/resto, menyimpannya sebagai kolom statis akan langsung salah begitu user memfilter apa pun.

## Calculated Column

### Definisi dan Karakteristik

- Dihitung **saat refresh dataset**, sekali per baris; hasilnya disimpan dan dikompresi di VertiPaq.
- Dievaluasi dalam **row context** (lihat [Row Context vs Filter Context](./02-row-context-vs-filter-context.md)).
- Cocok untuk nilai yang **statis per baris** dan **tidak bergantung filter**: kategori, key gabungan, turunan tanggal.
- Bisa dipakai sebagai slicer, axis, legend, kondisi relationship, dan filter row-level security (RLS).
- Di panel Data tampak seperti kolom biasa dengan ikon formula kecil.

```dax
-- Kolom klasifikasi statis di tabel Inventory
Kelompok Barang =
SWITCH (
    TRUE (),
    'Inventory'[Nama Barang] IN { "AYAM CINCANG (V.20)", "CABAI FROZEN" }, "Khusus",
    "Reguler"
)
```

## Tabel Keputusan: Kapan Pakai yang Mana

| Situasi | Pilihan | Alasan |
|---|---|---|
| Hasil bergantung slicer (bulan, area, resto, periode) | **Measure** | Dievaluasi mengikuti filter context |
| Perlu jadi slicer/axis/kategori di visual | Calculated column | Measure tidak bisa menjadi axis |
| Nilai berbeda per baris dan tidak berubah oleh filter (key, label) | Calculated column | Row context tersedia |
| Hasil agregasi (total, rata-rata, rasio, ranking) | **Measure** | Kolom agregasi bersifat statis |
| Perlu kolom untuk relationship atau RLS | Calculated column | Relationship hanya antar kolom fisik |
| Bisa dihitung saat ETL di Power Query/sumber | **Power Query dulu** | Dihitung di tahap load, model tetap ramping |

Rule of thumb komunitas: **"kalau ragu, buat measure."** Calculated column adalah pengecualian dengan alasan spesifik — bukan pilihan default.

## Dampak ke Storage (VertiPaq)

- VertiPaq adalah **columnar store**: setiap kolom dikompresi terpisah; ukurannya ditentukan kardinalitas (jumlah nilai unik) dan distribusinya.
- **Measure = 0 byte** di storage; biayanya hanya CPU saat query.
- **Calculated column membengkakkan model secara permanen**:
  - Kolom `Year` dengan 2 nilai unik → nyaris gratis.
  - Kolom gabungan teks `Resto & " - " & Nama Barang` dengan jutaan kombinasi → mahal: dictionary besar, kompresi buruk.
- Report cost-control milik user punya 109 calculated columns — mayoritas berasal dari fitur **Auto date/time** (setiap kolom tanggal otomatis membentuk tabel tanggal tersembunyi). Untuk model besar, matikan via *File → Options → Data Load → Auto date/time*, lalu gunakan tabel `Date = CALENDARAUTO()` eksplisit.
- Kolom yang tidak pernah dipakai di visual/relationship/hierarki adalah beban murni — hapus.

## Implicit vs Explicit Measures

| Aspek | Implicit (drag kolom → "Sum of Value") | Explicit (`[Deviasi CM] = SUM(...)`) |
|---|---|---|
| Reuse | Tidak — diulang di setiap visual | Ya — satu definisi untuk semua |
| Bisa direferensikan DAX? | Tidak | Ya |
| Format konsisten | Manual per visual | Diatur sekali di model |
| Description/dokumentasi | Tidak ada | Ada (muncul sebagai tooltip) |
| Dikelola calculation groups | Tidak | Ya |

**Explicit measures harus menjadi default.** Implicit measure adalah utang teknis: begitu logika berubah, Anda harus mengedit visual satu per satu di 36 halaman report.

## Calculated Tables (Brief)

```dax
Date      = CALENDARAUTO ()
Attribute = DATATABLE ( "Attribute", STRING, { { "NOMINAL" }, { "QTY" } } )
Periode   = DATATABLE ( "Periode", STRING,
              { { "WEEK 1" }, { "WEEK 2" }, { "WEEK 3" }, { "EOM" } } )
```

- Dihitung saat refresh seperti calculated column, tapi menghasilkan **tabel**.
- Ideal untuk tabel parameter/label kecil (seperti `Attribute` dan `Periode` di report user) dan tabel kalender.
- Ukurannya kecil, tetap perlu di-refresh setiap kali data berubah.

## Hidden Measures, Organisasi, dan Formatting

- **Hidden measures**: sembunyikan measure intermediate (`[Selisih]`, `[Deviasi Value]`) agar field list bersih — measure tetap bisa dipanggil formula lain.
- **Measure table**: buat tabel kosong khusus (mis. `_Measures`), jadikan home semua measure, lalu hide tabelnya — model besar jadi rapi dan mudah dinavigasi.
- **Description**: isi deskripsi measure (mis. `[Z Score]`: "skor anomali; |Z| > 1 = Anomali") — muncul sebagai tooltip di field list.
- **Formatting**: set format (persen, ribuan, desimal) pada measure, bukan per visual — angka konsisten di semua halaman.

## Sintaks & Contoh

```dax
-- ========= MEASURE (dihitung saat query, mengikuti filter context) =========
Deviasi CM = SUM ( 'Deviasi All'[Value] )

Sales CM   = SUM ( 'Sales All'[Penjualan Bersih] )

% BOM CM   = DIVIDE ( [Deviasi CM], [Sales CM] )

Growth     = DIVIDE ( [Selisih], ABS ( [Deviasi LM] ) )

-- ====== CALCULATED COLUMN (dihitung saat refresh, per baris) ======
-- Di tabel Resto: key gabungan ber-kardinalitas rendah
Key Resto = 'Resto'[Resto] & " - " & 'Resto'[Area]

-- Di tabel Deviasi All: nilai statis berbasis kolom baris
Nilai Abs = ABS ( 'Deviasi All'[Value] )

-- ====== CALCULATED TABLE ==========================================
Trend = DATATABLE ( "Kategori", STRING,
    {
        { "DEVIASI" }, { "BOM" }, { "% BOM" }, { "% SALES" }, { "VARIANCE" }
    }
)
```

## Kesalahan Umum

- **Membuat calculated column untuk nilai yang seharusnya measure** — misalnya kolom "Total Deviasi per Resto" berisi `SUM('Deviasi All'[Value])`: hasilnya statis, mengabaikan slicer bulan, dan membengkakkan model. Perbaikan: hapus kolomnya, buat measure.
- **Circular reference** — calculated column memanggil measure `[X]`, sementara `[X]` (langsung atau tidak langsung) membaca kolom tersebut → error *"A circular dependency was detected"*. Perbaikan: pisahkan arah — kolom hanya membaca kolom, measure hanya membaca measure/kolom.
- **Kolom kardinalitas tinggi** — `FORMAT('Date'[Date], "YYYYMMDD") & Resto & Item` per baris fakta membuat ukuran model meledak. Perbaikan: jadikan measure, atau pindahkan ke tabel dimensi.
- **Menyimpan klasifikasi hasil measure sebagai kolom** — `IF([Deviasi CM] >= 0, "Surplus", "Loss")` sebagai kolom akan membekukan nilai saat refresh. Perbaikan: buat measure `[Kategori]` (persis seperti di report user).
- **Mengandalkan implicit measure** — tidak bisa direferensikan DAX, format tidak konsisten. Perbaikan: konversi menjadi explicit measure.
- **Menambahkan kolom lalu tidak dipakai** — sampah VertiPaq; audit berkala dengan DAX Studio (VertiPaq Analyzer).

## Tips Pro

- Urutan berpikir default: **measure first** → kalau butuh nilai statis per baris, tanya "bisakah Power Query melakukannya?" → terakhir baru calculated column.
- Buat kolom turunan tanggal (`Year`, `MonthName`) sekali saja di tabel `Date` — jangan duplikasi di setiap tabel fakta.
- Kumpulkan measure di tabel `_Measures`; gunakan Tabular Editor untuk display folder bila perlu pengelompokan lebih rapi.
- Isi Description pada measure yang dipakai lintas halaman — tim dan reviewer model akan berterima kasih.
- Matikan Auto date/time dan ganti dengan tabel Date eksplisit — satu langkah kecil, dampak besar ke ukuran model.
- Cek ukuran model setelah menambah kolom baru dengan DAX Studio / VertiPaq Analyzer — kejutan 50 MB lebih baik ketahuan pagi ini daripada bulan depan.

## Terkait

- [Row Context vs Filter Context](./02-row-context-vs-filter-context.md)
- [Fungsi Agregasi](./04-aggregation-functions.md)
- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md)
- [Star Schema](../02-data-modeling/01-star-schema.md)
