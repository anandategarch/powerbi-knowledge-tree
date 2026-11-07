# Modeling Best Practices — Checklist Model yang Rapi

> Model yang rapi mempercepat pengembangan report, mengurangi salah drag kolom, dan memudahkan penyerahan ke orang lain. File ini adalah checklist lengkap — dari hide key columns sampai display folder dan tabel khusus untuk measure.

## 1. Star Schema Dulu, Baru Yang Lain

Semua best practice berikut percuma kalau model masih one big table. Pastikan fondasi benar: fact di tengah, dimensi mengelilingi, semua relasi M:1 single direction → lihat [Star Schema](./01-star-schema.md).

## 2. Hide Key Columns dari Report View

Kolom key — `(Old) Nama Barang` di Inventory, key fact di `Deviasi All` — hanya untuk relasi, bukan untuk visual. Right-click kolom → **Hide in report view**. Manfaat:

- Panel Fields bersih — user tidak salah drag key duplikat dari fact saat yang dimaksud label dari dimensi.
- Key fact (`Deviasi All[Nama Barang]`) juga sebaiknya di-hide; untuk label gunakan kolom dimensi via relasi.

Pengecualian: key yang memang sekaligus label dan slicer, seperti `Resto[Resto]`.

## 3. Format Kolom: Summarization, Format Angka, Data Category

- **Summarization**: set **Do not summarize** (Column tools → Summarization → None) untuk semua kolom label dan ID — `Nama Barang`, `PIC`, `Area`, kode resto. Bila tidak, kolom bisa muncul sebagai agregasi tak berarti di visual (kode cabang 9901 di-sum = angka sampah).
- **Format angka**: set sekali di kolom/measure (ribuan separator, jumlah desimal), jangan diformat manual di setiap visual.
- **Persen**: set format "Percentage" di properti measure — jangan `FORMAT()` di dalam DAX (hasil `FORMAT` adalah teks, tidak bisa dipakai sebagai axis numerik atau conditional formatting).
- **Data category**: City untuk kolom kota (mengaktifkan map visual), **Web URL** / **Image URL** untuk link dan gambar produk (Column tools → Data category).

## 4. Naming Convention

| Objek | Konvensi | Contoh |
|---|---|---|
| Tabel dimensi | Nama bisnis apa adanya | `Inventory`, `Resto`, `Date`, `Periode` |
| Tabel fact | Nama + konteks | `Deviasi All`, `Waste All`, `Sales All` |
| Measure | Nama bisnis, tanpa prefix | `Deviasi CM`, `Waste %` — konsisten dengan gaya user |
| Kolom | Tanpa prefix tabel | `Area`, `PIC`, `Nilai` |

Untuk model besar (user punya 160+ measures), measure tetap boleh tanpa prefix (mis. `Deviasi CM`) asalkan **semua measure ditaruh di satu tabel khusus** dan dikelompokkan dengan display folder — lihat bagian 5.

## 5. Tabel Khusus _Measures (Measure-Only Approach)

Buat satu tabel kosong sebagai "rumah" semua measure agar panel Fields punya satu tempat pencarian:

```dax
-- Home -> New table
_Measures = { BLANK() }
```

Lalu setiap membuat measure: pilih tabel `_Measures` dulu → **New measure**. Measure tidak menyimpan data, jadi tabel ini tidak menambah ukuran file. Karakter `_` membuat tabel muncul paling atas di panel Fields (urutan alfabetis).

Kelompokkan dengan **display folders**: pilih beberapa measure → isi properti **Display folder**: `Deviasi`, `Waste`, `Sales`, `Purchasing` — panel Fields menjadi berstruktur seperti file explorer.

## 6. Description & Documentation

- Isi **Description** pada measure dan kolom penting (Properties pane) — muncul sebagai tooltip di panel Fields. Tulis definisi bisnis, bukan pengulangan nama: `Deviasi CM = (CM aktual - CM standar) dalam rupiah; positif = boros`.
- Simpan definisi grain fact di description tabel (`satu baris = resto x barang x periode x attribute x jenis`).
- Cantumkan satuan pada measure kunci ("dalam Rp", "dalam %") untuk mengurangi salah baca.

## 7. Matikan Auto Date/Time & Hapus yang Tak Terpakai

- **File → Options → Current File → Data Load → uncheck Auto date/time** — tabel tanggal tersembunyi per kolom datetime memboroskan memori.
- Hapus kolom tak terpakai **di Power Query**, bukan hanya di-hide: hide hanya menyembunyikan, data tetap dimuat → ukuran file dan waktu refresh tetap berat.
- Hapus juga kolom helper antara (`Helper`, `Baris Kosong`, kolom debug) setelah transformasi selesai.

## 8. Data Categorization & Layout

- Data category untuk geografi/URL (City, Web URL, Image URL) agar visual khusus aktif dan ikon kolom jelas beda.
- Rapikan **Model view**: dimensi di atas, fact di bawah, garis relasi pendek tanpa silang — diagram star yang terbaca mempermudah review.
- Urutan halaman report juga bagian kerapian: kelompokkan per topik (Ringkasan, Deviasi, Waste, Purchasing).

## Sintaks & Contoh

```dax
-- 1) Tabel rumah measure
_Measures = { BLANK() }

-- 2) Contoh measure rapi: nama bisnis, format persen di properti, description terisi
Waste % :=
VAR NilaiWaste =
    CALCULATE ( SUM ( 'Waste All'[Nilai] ), 'Attribute'[Attribute] = "NOMINAL" )
VAR NilaiSales =
    CALCULATE ( SUM ( 'Sales All'[Nilai] ), 'Attribute'[Attribute] = "NOMINAL" )
RETURN
    DIVIDE ( NilaiWaste, NilaiSales )
-- Set di Properties: Format = Percentage, 1 desimal
-- Set di Properties: Description = "Waste nominal / Sales nominal, per periode"
```

## Audit Checklist

| # | Item | Cara Cek | Status |
|---|---|---|---|
| 1 | Star schema: fact dan dimensi terpisah | Model view | [ ] |
| 2 | Semua relasi M:1 single direction (kecuali bridge disengaja) | Manage relationships | [ ] |
| 3 | Key columns hidden dari report view | Panel Fields | [ ] |
| 4 | Auto date/time OFF | File → Options → Data Load | [ ] |
| 5 | Tabel Date custom + Mark as date table | Table tools | [ ] |
| 6 | Kolom teks/ID: Summarization = None | Column tools | [ ] |
| 7 | Format angka/persen diset di kolom & measure | Column/Measure tools | [ ] |
| 8 | Semua measure di tabel _Measures + display folder | Panel Fields | [ ] |
| 9 | Description terisi untuk measure kunci | Properties pane | [ ] |
| 10 | Kolom tak terpakai dihapus di Power Query | Power Query editor | [ ] |
| 11 | Data category untuk City / Web URL | Column tools | [ ] |
| 12 | Tidak ada (Blank) misterius di slicer dan visual | Tabel debug | [ ] |

## Kesalahan Umum

- **Measure tersebar di banyak tabel** — user kesulitan mencari `Deviasi CM` di antara ratusan kolom. Perbaikan: tabel `_Measures` + display folder per topik.
- **Format persen pakai `FORMAT` di dalam DAX** — hasil jadi teks: axis, sorting, dan conditional formatting rusak. Perbaikan: set format "Percentage" di properti measure.
- **Kolom kode numeric ikut ter-sum** — kode resto dijumlah jadi angka nonsens. Perbaikan: Summarization → None, atau ubah jadi teks di Power Query.
- **Meng-hide kolom mengira menghapusnya** — data tetap dimuat dan ikut refresh. Perbaikan: hapus di Power Query (`Table.RemoveColumns`).
- **Description kosong semua** — penyerahan report jadi menyiksa dan terjadi salah interpretasi angka. Perbaikan: isi minimal untuk 10 measure utama.

## Tips Pro

- Audit berkala: buka Model view dan tanyakan "bila saya orang baru, apakah 10 detik cukup untuk memahami model ini?" Bila tidak, rapikan.
- Prefix `_` juga cocok untuk tabel parameter/debug: `_Config`, `_Temp` — selalu di atas panel Fields.
- Gunakan **Performance Analyzer** (View → Performance Analyzer) sebelum dan sesudah pembersihan untuk membuktikan dampaknya.
- Pertahankan gaya penamaan yang sudah ada bila tim sudah terbiasa (user: `Deviasi All`, `Deviasi CM`) — konsistensi mengalahkan selera personal.

## Terkait

- [Star Schema](./01-star-schema.md)
- [Relationships](./02-relationships.md)
- [Cardinality & Cross Filter](./03-cardinality-and-cross-filter.md)
- [Date Dimension](./04-date-dimension.md)
- [M Language Basics](../03-power-query/01-m-language-basics.md)
- [Common Transformations](../03-power-query/02-common-transformations.md)
- [M Cheatsheet](../00-quick-reference/m-cheatsheet.md)
- [Analisis model report cost control](../11-user-report-analysis/model-and-dax.md)
