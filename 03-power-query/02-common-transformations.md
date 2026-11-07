# Common Transformations — Kamus Transformasi Harian

> Semua klik di UI Power Query adalah fungsi M. File ini memetakan transformasi harian — dari promote headers sampai unpivot — ke kode M yang dihasilkannya, lengkap dengan jebakan umum (locale tanggal, null handling, row explosion saat merge).

## Konsep: UI Editor Hanyalah Generator M

Setiap tombol menulis satu baris M ke Applied Steps. Dengan mengenali polanya, Anda bisa: membaca ulang query orang lain, memperbaiki langkah tanpa klik ulang, dan menulis transformasi yang tidak tersedia di UI. Urutan transformasi juga penting: **filter dan buang kolom sedini mungkin**, terutama untuk sumber tanpa query folding (Folder/Excel) karena semua diproses di memori lokal.

## Sintaks & Contoh — Menu UI → Kode M

### Promote Headers
UI: Home → **Use First Row as Headers**

```powerquery
= Table.PromoteHeaders(Sumber, [PromoteAllScalars = true])
```

Baris pertama sheet menjadi nama kolom. Pastikan tidak ada baris judul laporan di atasnya — buang dulu dengan filter rows, jangan langsung promote.

### Change Type (dan Jebakan Locale)
UI: pilih kolom → dropdown tipe → **Using Locale...**

```powerquery
= Table.TransformColumnTypes(Sebelumnya, {{"Tanggal", type date}}, "id-ID")
```

- `"05/07/2025"` dibaca `MM/dd/yyyy` (en-US) → 7 Mei; dibaca `dd/MM/yyyy` (id-ID) → 5 Juli. Salah locale = tanggal bergeser diam-diam.
- Angka format Indonesia `1.234,56` juga membutuhkan locale.

### Filter Rows
UI: dropdown panah kolom / **Number Filters / Text Filters**

```powerquery
= Table.SelectRows(Sebelumnya, each [Status] = "AKTIF" and [Nilai] > 0)
```

### Remove / Select Columns
UI: right-click kolom → **Remove**; Home → **Choose Columns**

```powerquery
= Table.RemoveColumns(Sebelumnya, {"Helper", "Baris Kosong"})
= Table.SelectColumns(Sebelumnya, {"Resto", "Nama Barang", "Nilai"})
```

`SelectColumns` lebih aman bila daftar kolom yang dipertahankan pasti: file sumber yang menambah kolom baru tidak ikut termuat.

### Rename Columns
UI: double-click header kolom

```powerquery
= Table.RenameColumns(Sebelumnya, {{"Col1", "Nama Barang"}}, MissingField.Ignore)
```

Argumen ketiga `MissingField.Ignore` mencegah error bila kolom sumbernya hilang/berubah nama.

### Replace Values & Null Handling
UI: Transform → **Replace Values** (isian `null` untuk mengganti null)

```powerquery
= Table.ReplaceValue(Sebelumnya, "N/A", null, Replacer.ReplaceText, {"Nama Barang"})
= Table.ReplaceValue(Sebelumnya, null, 0, Replacer.ReplaceValue, {"Nilai"})
```

`Replacer.ReplaceText` mencocokkan substring (case-insensitive); `Replacer.ReplaceValue` mencocokkan nilai persis. Variasi "N/A " dengan spasi tidak tertangkap — `Text.Trim` dulu.

### Split Column by Delimiter
UI: Transform → **Split Column → By Delimiter** (`_`)

Konteks: setelah unpivot, kolom `Attribute` berisi `QTY_WASTE`, `NOMINAL_SALES`, dst.

```powerquery
= Table.SplitColumn(
    Sebelumnya, "Attribute",
    Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv),
    {"Attribute", "Jenis"}
)
```

Hasil: `QTY_WASTE` → `Attribute = "QTY"`, `Jenis = "WASTE"`. Nama kolom hasil ditentukan urutan list.

### Unpivot — Mengapa dan Pola User
UI: pilih kolom key → Transform → **Unpivot Other Columns**

Konteks user: file Excel mentah berformat wide — `QTY_STOK`, `QTY_WASTE`, `NOMINAL_SALES`, dan seterusnya; satu baris per resto per barang dengan belasan kolom nilai.

Mengapa unpivot (long format):

- Kolom berkurang drastis — nilai menjadi baris; model ramping dan jenis baru tinggal menambah baris, bukan kolom.
- Bisa di-slice: `Attribute` (QTY/NOMINAL) dan `Jenis` (STOK/WASTE/...) menjadi dimensi + slicer, bukan nama kolom mati.
- Satu measure `SUM([Nilai])` menggantikan banyak measure per kolom.

```powerquery
let
    Sumber  = Sebelumnya, // wide: Resto, Nama Barang, Periode, QTY_*, NOMINAL_*
    KeyCols = {"Resto", "Nama Barang", "Periode"},
    Unpivot = Table.UnpivotOtherColumns(Sumber, KeyCols, "Attribute", "Nilai"),
    Split   = Table.SplitColumn(Unpivot, "Attribute",
                  Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv),
                  {"Attribute", "Jenis"}),
    Tipe    = Table.TransformColumnTypes(Split, {{"Nilai", type number}})
in
    Tipe
```

### Pivot (Kebalikan Unpivot)
UI: Transform → **Pivot Column**

```powerquery
= Table.Pivot(Sebelumnya, List.Distinct(Sebelumnya[Attribute]), "Attribute", "Nilai", List.Sum)
```

Dipakai untuk membangun tampilan matrix di Power Query — jarang dibutuhkan di model karena visual matrix DAX lebih fleksibel. Wajib ada fungsi agregasi (`List.Sum`) karena data bisa duplikat.

### Merge Queries (Join)
UI: Home → **Merge Queries** → pilih kolom key + Join Kind

| Join Kind | Menghasilkan |
|---|---|
| **Left Outer** | Semua baris kiri + pasangan yang cocok (default) |
| Right Outer | Kebalikan left outer |
| **Inner** | Hanya baris yang cocok di kedua sisi |
| Full Outer | Semua baris kiri dan kanan |
| **Left Anti** | Baris kiri TANPA pasangan — untuk audit master |
| Right Anti | Baris kanan tanpa pasangan |

```powerquery
= Table.NestedJoin(
      Fakta, {"Nama Barang"},
      Inventory, {"(Old) Nama Barang"},
      "Inv", JoinKind.LeftOuter)
= Table.ExpandTableColumn(Sebelumnya, "Inv", {"Kategori", "Satuan"})
```

Jebakan: bila sisi kanan (Inventory) punya key duplikat, hasil merge **meledak** (row explosion) — satu baris fact menjadi banyak. Dedup dimensi dulu. Gunakan Left Anti untuk menemukan barang di fact yang tidak ada di master.

### Append Queries (Stacking)
UI: Home → **Append Queries**

```powerquery
= Table.Combine({Jan, Feb, Mar})
```

Menumpuk tabel berstruktur sama — pola klasik menggabungkan file per periode. Kolom yang tidak ada di salah satu tabel menjadi null. Alternatif skala besar: `Folder.Files` + custom function (lihat [Parameters & Functions](./03-parameters-and-functions.md)).

### Group By
UI: Home → **Group By** (Advanced untuk multi-agregasi)

```powerquery
= Table.Group(Sebelumnya, {"Resto", "Nama Barang"}, {
      {"TotalNilai", each List.Sum([Nilai]), type nullable number},
      {"JumlahBaris", each Table.RowCount(_), Int64.Type}
  })
```

Mengubah grain menjadi agregat (mis. ringkas transaksi menjadi harian per resto per barang).

### Add Custom Column
UI: **Add Column → Custom Column**

```powerquery
= Table.AddColumn(Sebelumnya, "Rp Std", each [QTY] * [HNA], type nullable number)
```

Selalu isi tipe data (argumen keempat) — tanpa itu kolom menjadi `any`.

### Index Column
UI: **Add Column → Index Column**

```powerquery
= Table.AddIndexColumn(Sebelumnya, "No", 1, 1, Int64.Type)
```

Untuk identifikasi baris unik, pola keep-first (sort lalu dedup), dan debugging.

### Conditional Column
UI: **Add Column → Conditional Column** (form, tanpa menulis M)

```powerquery
= Table.AddColumn(Sebelumnya, "Status", each
      if [Nilai] < 0 then "NEGATIF"
      else if [Nilai] = 0 then "NOL"
      else "POSITIF", type text)
```

UI Conditional Column menghasilkan kode yang sama dengan Custom Column berisi `if ... then ... else` — pakai form bila kondisinya sederhana.

### Expand Table Column
UI: tombol expand (dua panah) di header kolom bertipe table — muncul setelah merge atau invoke custom function

```powerquery
= Table.ExpandTableColumn(Sebelumnya, "Inv", {"Kategori", "Satuan", "HNA"})
```

Expand hanya kolom yang dibutuhkan — memuat semua kolom menambah beban refresh.

### Fill Down
UI: pilih kolom → Transform → **Fill → Down**

```powerquery
= Table.FillDown(Sebelumnya, {"Area"})
```

Mengisi sel kosong dengan nilai di atasnya — khas data Excel dengan merged cells (Area hanya tertulis sekali per grup cabang).

## Kesalahan Umum

- **Change type tanpa locale** — tanggal/angka Indonesia salah baca. Perbaikan: Using Locale → `id-ID`.
- **Unpivot salah seleksi** — kolom key ikut menjadi baris. Perbaikan: pilih kolom key dulu, lalu Unpivot **Other** Columns.
- **Merge tanpa dedup sisi kanan** — row explosion. Perbaikan: `Table.Distinct` di tabel dimensi sebelum merge.
- **Replace Values tidak menangkap variasi** ("N/A", "N/A ", "n/a") — perbaikan: `Text.Trim` + `Replacer.ReplaceText` (case-insensitive).
- **Fill down sebelum buang baris footer/grand total** — nilai footer menular ke baris lain. Perbaikan: filter baris total dulu, baru fill.
- **Expand semua kolom** hasil merge — refresh lambat. Perbaikan: sebutkan kolom yang dibutuhkan saja.

## Tips Pro

- Setelah transformasi berat, buat step verifikasi: `Table.RowCount` atau `Table.Profile` untuk mendeteksi anomali (null rate, min/max).
- Kolom audit (`File Sumber`, `Date modified`) dari `Folder.Files` sangat membantu menelusuri baris bermasalah sampai ke file aslinya.
- Rename step sesuai fungsi (`Split Attribute_Jenis`) — Applied Steps adalah dokumentasi hidup query.
- Bila struktur file antar periode berubah-ubah, `Table.SelectColumns` + `MissingField.UseNull` lebih tahan banting daripada rename hardcoded.

## Terkait

- [M Language Basics](./01-m-language-basics.md)
- [Parameters & Functions](./03-parameters-and-functions.md)
- [Resep PQ](./04-pq-patterns-recipes.md)
- [Star Schema](../02-data-modeling/01-star-schema.md)
- [M Cheatsheet](../00-quick-reference/m-cheatsheet.md)
