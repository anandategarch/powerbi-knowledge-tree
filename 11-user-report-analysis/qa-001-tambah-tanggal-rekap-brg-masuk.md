# Q&A 001 — Tambah Tanggal di Halaman Rekap Brg Masuk

> **Pertanyaan user:** "Tambah tanggal di page 17 atau rekap brg masuk gimana ya"
> **Halaman terkait:** `Rekap Brg Masuk` (34 visual: 14 shape, 7 slicer, 5 KPI card, 3 bar/column chart, drill-through → `ID Dokumen`, tombol Reset). Halaman satu paket dengan `Detail Brg Masuk` dan `Rekon Brg Masuk`.
> **Ditanyakan:** 7 Nov 2026.

Ada 3 interpretasi "tambah tanggal" — ketiganya dibahas di sini.

---

## Interpretasi A — Mau FILTER halaman berdasarkan tanggal (Slicer Tanggal)

### Langkah menambahkan slicer tanggal

1. Klik area kosong di kanvas halaman `Rekap Brg Masuk`.
2. Panel **Visualizations** → klik ikon **Slicer**.
3. Drag field tanggal ke Field well. Pilihan field yang tersedia di model ini:
   - `Date[Date]` — tanggal granular (harian)
   - `Date[Period]` — format `"yyyy-mm"` (dipakai halaman lain, konsisten)
   - `Date[Periode Cut Off]` — logika cutoff tanggal 26 (fiscal month)
4. Klik **⋮** (titik tiga) pojok kanan atas slicer → pilih gaya:
   - **Between** — rentang bebas (kalender dua sisi)
   - **Relative** — "N bulan/hari terakhir" (tidak perlu setting manual tiap bulan)
   - **Dropdown / List** — pilih tanggal satu per satu
5. Ganti judul slicer di Format → **Slicer header** (mis. "Pilih Tanggal").

### WAJIB: Sync slicers

Report ini punya 292 slicer di 36 halaman yang sudah disinkronkan (`Date[Year]`, `Date[Month]`, `Periode`, `Resto[Area]`, `Resto[PIC]`). Slicer baru yang dibuat manual **tidak otomatis ikut grup sync**:

1. **View → Sync slicers**.
2. Cari grup slicer tanggal yang sama (field identik).
3. Centang **Sync** + **Visible** untuk `Rekap Brg Masuk`, `Detail Brg Masuk`, dan `Rekon Brg Masuk` (ketiganya satu paket).
4. Kalau slicer tanggal baru beda grup dengan yang sudah ada → hapus, lalu copy-paste slicer tanggal dari halaman lain (mis. `Summary Deviasi`) lalu atur sync-nya.

### ⚠️ Peringatan model — `Rekap Mentah BM` TIDAK berelasi ke `Date`

Berdasarkan analisis DataModel (lihat [model-and-dax.md](./model-and-dax.md)), relasi `Rekap Mentah BM` hanya:

| Dari | Kolom | Ke | Kardinalitas |
|---|---|---|---|
| `Rekap Mentah BM` | `NAMA BARANG` | `Inventory[(Old) Nama Barang]` | M:1 |
| `Rekap Mentah BM` | `NAMA RESTO` | `Resto[Resto]` | M:M |

**Tidak ada relasi ke `Date`.** Konsekuensi: slicer `Date` **tidak akan memfilter** `[BM Resto] = SUM('Rekap Mentah BM'[Value])`. Gejala: angka KPI card di halaman tidak berubah saat tanggal diganti.

Padahal kolom tanggalnya sudah ada: calculated column `Rekap Mentah BM[TANGGAL]` (dirakit dari `Source.Name` tahun-bulan + kolom `DAY`).

**Solusi 1 — Relasi fisik (paling bersih):**

1. Model view → drag `Rekap Mentah BM[TANGGAL]` → `Date[Date]`.
2. Cardinality: **Many to One (\*)**, cross filter: **Single**.
3. Validasi dulu isi kolom (nama file sebagai sumber tanggal itu rapuh — salah format nama file = tanggal nyasar):

```dax
-- Cek di card / DAX query view
Min Tgl BM = MIN('Rekap Mentah BM'[TANGGAL])
Max Tgl BM = MAX('Rekap Mentah BM'[TANGGAL])
Jml Tgl Nyasar = COUNTROWS(FILTER('Rekap Mentah BM', 'Rekap Mentah BM'[TANGGAL] < DATE(2024,1,1)))
```

**Solusi 2 — TREATAS (pola yang sudah dipakai report ini, ala `[Deviasi CM]`):**
Tambahkan TREATAS di measure BM, tanpa menyentuh model:

```dax
BM Resto (by Date) =
CALCULATE(
    SUM('Rekap Mentah BM'[Value]),
    TREATAS(
        VALUES('Date'[Date]),
        'Rekap Mentah BM'[TANGGAL]
    )
)
```

**Catatan untuk `[BM Sistem]`:** measure ini menjumlahkan `3224` + `4205` + `[Pembelian di Luar]` (dari `9901`). Tabel `3224`/`4205` juga **tidak punya relasi tanggal**. Kalau hanya `Rekap Mentah BM` yang di-hook ke `Date`, maka `[Selisih BM] = [BM Resto] - [BM Sistem]` jadi timpang (satu sisi terfilter tanggal, sisi lain tidak). Pastikan sisi sistem punya kolom tanggal yang juga berelasi/TREATAS sebelum memakai slicer tanggal untuk rekonsiliasi.

---

## Interpretasi B — Mau MENAMPILKAN tanggal di visual (axis chart / kolom tabel)

- **Kolom siap pakai:** `Rekap Mentah BM[TANGGAL]` — drag ke:
  - **Axis** bar/column chart (X-axis) untuk tren harian, atau
  - **Rows/Values** di table `Detail Brg Masuk`.
- **Kalau mau tampilan bulanan** konsisten seperti halaman lain (`yyyy-mm`), buat calculated column baru:

```dax
Periode BM = FORMAT('Rekap Mentah BM'[TANGGAL], "yyyy-mm")
```

- Kalau drag `Date[Period]` ke axis **tidak mengubah** chart BM → itu gejala yang sama dengan Interpretasi A (tidak ada relasi); solusinya sama (relasi fisik atau TREATAS).

---

## Interpretasi C — Mau menampilkan TANGGAL UPDATE DATA di header halaman

Measure-nya **sudah ada** di model (sumber: tabel `Update`):

```dax
-- Sudah ada, tinggal dipakai
Update on BM =
"Updated on : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Update on Barang Masuk"),"dd mmmm yyyy")

Last Update BM =
"Last Update : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Last Update Barang Masuk"),"dd mmmm yyyy")
```

Cara pakai (3 opsi):

1. **Card visual:** Insert → Card → drag `[Update on BM]` ke Fields.
2. **Text box + nilai:** Insert → Text box → ketik teks statis, atau Insert → Smart narrative.
3. **Judul visual dinamis (paling rapi):** klik visual KPI → Format (cat icon) → **Title** → teks → klik **fx (Conditional formatting)** → *Field value* → pilih `[Update on BM]` → judul visual otomatis menampilkan tanggal update terakhir data BM.

---

## Kesalahan Umum

| Kesalahan | Akibat | Pencegahan |
|---|---|---|
| Slicer `Date` dibuat tanpa cek relasi | KPI BM diam tidak berubah meski tanggal diganti | Cek Model view dulu; baca peringatan model di atas |
| Lupa Sync slicers | Pilihan tanggal hilang saat pindah halaman Detail/Rekon | View → Sync slicers, centang Sync+Visible 3 halaman BM |
| Relasi `TANGGAL` dibuat tanpa validasi | Tanggal nyasar dari nama file salah format → filter meleset | Cek `MIN/MAX('Rekap Mentah BM'[TANGGAL])` dulu |
| Hanya hook `Rekap Mentah BM` ke Date | `[Selisih BM]` timpang (resto terfilter, sistem tidak) | Hook `3224`/`4205` juga, atau jangan filter tanggal untuk rekon |
| Drag `Date[Date]` ke chart BM axis | Hierarki Year-Quarter-Month muncul & axis berantakan | Matikan hierarki (klik kanan field → hilangkan "Show as hierarchy") atau pakai kolom `TANGGAL` langsung |

## Tips Pro

- Untuk halaman rekon yang butuh "semua data bulan berjalan", lebih aman pakai **relative date slicer** ("This month") daripada between manual.
- Patuhi pola laporan: halaman lain memakai `Date[Period]` (`yyyy-mm`) sebagai bahasa periode — samakan supaya user tidak bingung.
- Kalau sering perlu drill tanggal, pertimbangkan **drillthrough** dari chart BM → `ID Dokumen` sudah ada; cukup pastikan field tanggal ikut sebagai drillthrough field.

## Terkait

- [REPORT_OVERVIEW.md](./REPORT_OVERVIEW.md) — struktur 36 halaman
- [model-and-dax.md](./model-and-dax.md) — relasi & measure BM lengkap
- [../06-filters-and-slicers/02-slicer-guide.md](../06-filters-and-slicers/02-slicer-guide.md) — panduan slicer & sync
- [../02-data-modeling/04-date-dimension.md](../02-data-modeling/04-date-dimension.md) — tabel Date
- [../01-dax/08-relational-functions.md](../01-dax/08-relational-functions.md) — TREATAS pattern
