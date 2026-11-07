# PQ Patterns & Recipes — Resep Power Query Siap Pakai

> Kumpulan resep M untuk masalah nyata di pipeline data cost control: combine folder, parse nama file, unpivot berprefiks, cleaning key, Google Sheets, fuzzy matching, rekonsiliasi dua versi data, dan lainnya. Setiap resep: masalah → kode → penjelasan.

## Cara Memakai File Ini

Semua resep diasumsikan berjalan di Advanced Editor. Nama seperti `fnTransformFile` dan parameter `FolderPath` merujuk ke pola di [Parameters & Functions](./03-parameters-and-functions.md). Sesuaikan nama kolom dengan file Anda — prinsipnya yang penting, bukan nama variabelnya.

## Sintaks & Contoh — 12 Resep

### 1. Combine Semua File Excel dalam Folder (per Periode YYYY_MM)

Masalah: ratusan file `2025_01_W1.xlsx`, `2025_02_EOM.xlsx`, ... harus jadi satu tabel, tanpa file sampah.

```powerquery
let
    Sumber  = Folder.Files(FolderPath),
    Bersih  = Table.SelectRows(Sumber, each
                  [Extension] = ".xlsx" and not Text.StartsWith([Name], "~$")
                  and not [Attributes][Hidden]),
    Isi     = Table.AddColumn(Bersih, "Data", each fnTransformFile([Content])),
    Meta    = Table.AddColumn(Isi, "PeriodeFile", each fnParseNamaFile([Name]), type date),
    Rapikan = Table.RemoveColumns(Meta, {"Content", "Attributes", "Folder Path"}),
    Hasil   = Table.ExpandTableColumn(Rapikan, "Data",
                  {"Resto", "Nama Barang", "Attribute", "Jenis", "Nilai"})
in
    Hasil
```

Penjelasan: filter dulu (file lock `~$` dan hidden adalah penyebab error refresh paling umum), invoke function per file, simpan metadata, lalu expand menjadi satu fact table.

### 2. Parse Tanggal dari Nama File dengan Text.Split

Masalah: tanggal tidak ada di dalam data — hanya di nama file `2025_03_W1.xlsx`.

```powerquery
fnParseNamaFile = (nama as text) as date =>
    let
        Bagian = Text.Split(Text.BeforeDelimiter(nama, "."), "_"),
        Tahun  = Number.From(Bagian{0}),
        Bulan  = Number.From(Bagian{1})
    in
        #date(Tahun, Bulan, 1)
```

Penjelasan: potong ekstensi, split `_`, ambil dua elemen pertama. `"03"` otomatis menjadi 3 lewat `Number.From`. Selalu pakai `[Name]`, bukan `[Date modified]`.

### 3. Unpivot Kolom Berprefiks (QTY_ / NOMINAL_)

Masalah: file wide dengan kolom `QTY_*` dan `NOMINAL_*` yang bisa bertambah jenisnya tiap periode.

```powerquery
let
    Semua   = Table.ColumnNames(Sumber),
    KeyCols = List.Select(Semua, each
                  not Text.StartsWith(_, "QTY_") and not Text.StartsWith(_, "NOMINAL_")),
    Unpivot = Table.UnpivotOtherColumns(Sumber, KeyCols, "Attribute", "Nilai"),
    Split   = Table.SplitColumn(Unpivot, "Attribute",
                  Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv),
                  {"Attribute", "Jenis"})
in
    Split
```

Penjelasan: daftar key dihitung dinamis — kolom prefiks baru otomatis ikut di-unpivot tanpa mengubah query. Hasil: `Attribute` (QTY/NOMINAL) + `Jenis` (STOK/WASTE/...).

### 4. Cleaning Key untuk Join Mulus

Masalah: key join kotor — spasi depan/belakang, spasi ganda di tengah, beda kapitalisasi.

```powerquery
// Bersihkan spasi (termasuk spasi ganda) lalu UPPER
RapikanSpasi = (s as text) as text =>
    Text.Combine(
        List.Select(Text.Split(Text.Trim(s), " "), each _ <> ""),
        " ")

KeyBersih = Table.TransformColumns(Sumber, {
    {"Nama Barang", each Text.Upper(RapikanSpasi(Text.Clean(_))), type text}
})
```

Penjelasan: `Text.Trim` hanya memotong pangkal; pola split-select-join membersihkan semua spasi ganda. `Text.Upper` menyatukan "Ayam" vs "AYAM" — pakai bila master memang konsisten uppercase.

### 5. Google Sheets → Power Query (URL Export)

Masalah: master barang dirawat di Google Sheets dan harus ikut refresh.

```powerquery
let
    Sumber = Web.Contents(
        "https://docs.google.com",
        [RelativePath = "spreadsheets/d/" & SheetID & "/export",
         Query = [format = "xlsx"], IsRetry = true]),
    Buku   = Excel.Workbook(Sumber),
    Sheet  = Buku{[Item = "Master Barang"]}[Data]
in
    Sheet
```

Penjelasan: sharing harus "Anyone with the link" (Viewer). Base URL statis + RelativePath mencegah masalah dynamic data source di gateway. Alternatif CSV: `gviz/tq?tqx=out:csv`.

### 6. Fuzzy Matching Singkat

Masalah: nama barang dari resto tidak persis sama dengan master ("AYAM WHOLE" vs "AYAM WHOLE ").

```powerquery
= Table.FuzzyNestedJoin(Fakta, {"Nama Barang"},
      Inventory, {"(Old) Nama Barang"},
      "Inv", JoinKind.LeftOuter,
      [IgnoreCase = true, IgnoreSpace = true, Threshold = 0.7, NumberOfMatches = 1])
```

Penjelasan: threshold 0–1 makin tinggi makin ketat. Selalu sampel-check hasilnya — fuzzy bisa salah cocok; idealnya hasilnya dibekukan menjadi tabel mapping statis, bukan di-refresh tiap kali.

### 7. Filter Kolom Dinamis dengan List.Accumulate / Table.SelectColumns

Masalah: struktur kolom beda antar file; `Table.SelectColumns` error bila kolom hilang.

```powerquery
// (a) Amankan kolom wajib yang mungkin tidak ada di file lama
Wajib   = {"Resto", "Nama Barang", "Nilai"},
Amankan = Table.SelectColumns(Sumber,
              List.Intersect({Wajib, Table.ColumnNames(Sumber)}),
              MissingField.UseNull),

// (b) Buang kolom berpola ("Backup ...", "... (2)") dengan List.Accumulate
Buang  = List.Accumulate(Table.ColumnNames(Sumber), {},
            (acc, col) => if Text.StartsWith(col, "Backup") or Text.EndsWith(col, "(2)")
                          then acc & {col} else acc),
Bersih = Table.RemoveColumns(Sumber, Buang)
```

Penjelasan: `List.Accumulate(list, seed, accumulator)` melipat list menjadi satu hasil — di sini daftar kolom yang akan dibuang.

### 8. Custom Calendar Table di Power Query

Masalah: butuh tabel Date kontinu bertipe pasti sebelum masuk model.

```powerquery
let
    Awal  = #date(2024, 1, 1),
    Akhir = #date(2026, 12, 31),
    Tgl   = List.Dates(Awal, Duration.Days(Akir - Awal) + 1, #duration(1, 0, 0, 0)),
    Kal   = Table.FromList(Tgl, Splitter.SplitByNothing(), {"Date"}),
    Tip   = Table.TransformColumnTypes(Kal, {{"Date", type date}}),
    Kolom = Table.AddColumn(Tip, "Tahun", each Date.Year([Date]), Int64.Type)
in
    Kolom
```

Penjelasan: tambahkan kolom lain (`Bulan`, `Kuartal`) dengan pola `Table.AddColumn` yang sama. Keunggulan dibanding DAX: bisa langsung di-merge dengan tabel hari libur.

### 9. Handle File Korup / Baris Kosong dengan try ... otherwise

Masalah: satu file korup membuat seluruh refresh gagal.

```powerquery
// Per file: gagal → tabel penanda, refresh tetap lanjut
Aman = Table.AddColumn(Bersih, "Data", each
    try fnTransformFile([Content])
    otherwise #table({"Resto", "Nama Barang", "Nilai"}, {{null, null, null}}))

// Per baris: konversi gagal → null, bukan error
Nilai = Table.AddColumn(Sumber, "Qty", each
    try Number.From(Text.Trim([QtyText])) otherwise null, type nullable number)
```

Penjelasan: pola pertama menjaga pipeline tetap hidup, pola kedua membersihkan sel kotor. Kombinasikan dengan filter `each [Data] <> null` untuk mengaudit file gagal.

### 10. Dedup Table

Masalah: master punya duplikat key sehingga relasi menjadi M:M.

```powerquery
// Duplikat penuh (semua kolom identik)
Bersih = Table.Distinct(Sumber)

// Keep first per key setelah sort terbaru
KeepFirst = Table.Distinct(
    Table.Sort(Sumber, {{"Tanggal", Order.Descending}}), {"Nama Barang"})
```

Penjelasan: `Table.Distinct` mempertahankan kemunculan pertama — sort dulu agar "pertama" berarti terbaru. Tanpa sort, hasil keep-first tidak deterministik.

### 11. Merge dengan Perbandingan Dua Versi (Rekonsiliasi)

Masalah: membandingkan BM versi Sistem vs BM versi Resto — mana yang cocok, selisih, atau hanya ada di satu versi.

```powerquery
let
    // BMSistem & BMResto: sama-sama {Resto, Nama Barang, Qty}
    Join   = Table.NestedJoin(BMSistem, {"Resto", "Nama Barang"},
                              BMResto,  {"Resto", "Nama Barang"},
                              "R", JoinKind.FullOuter),
    Expand = Table.ExpandTableColumn(Join, "R", {"Qty"}, {"Qty Resto"}),
    Status = Table.AddColumn(Expand, "Status", each
                 if [Qty] = null             then "Hanya Sistem"
                 else if [Qty Resto] = null  then "Hanya Resto"
                 else if [Qty] = [Qty Resto] then "Cocok"
                 else "Selisih", type text),
    Delta  = Table.AddColumn(Status, "Delta",
                 each ([Qty] ?? 0) - ([Qty Resto] ?? 0), type nullable number)
in
    Delta
```

Penjelasan: FullOuter memastikan kedua sisi terlihat; kolom `Status` dan `Delta` menjadi dasar visual rekonsiliasi. Selalu bandingkan jumlah baris sebelum/sesudah untuk memastikan tidak ada yang hilang.

### 12. Konversi Satuan dengan Lookup Table

Masalah: qty datang campuran satuan (KG, GR, LTR) dan harus dinormalisasi.

```powerquery
let
    // Lookup Konversi: {Dari, Ke, Faktor} mis. {"KG","GR",1000}, {"GR","GR",1}
    Join  = Table.NestedJoin(Fakta, {"Satuan"}, Konversi, {"Dari"},
                             "K", JoinKind.LeftOuter),
    Exp   = Table.ExpandTableColumn(Join, "K", {"Faktor"}),
    Std   = Table.AddColumn(Exp, "QTY Std", each [Qty] * ([Faktor] ?? 1), type nullable number)
in
    Std
```

Penjelasan: `?? 1` memberi faktor netral bila satuan tidak ada di tabel konversi — sekaligus sinyal untuk mengaudit baris tersebut.

## Kesalahan Umum

- **Fuzzy match dijalankan setiap refresh** untuk data yang sebenarnya statis — lambat dan tidak deterministik. Perbaikan: bekukan hasilnya jadi tabel mapping.
- **Combine tanpa filter `~$`/hidden** — refresh gagal saat file terbuka. Perbaikan: resep 1 langkah 2.
- **Parse tanggal dari `Date modified`** — tanggal berubah saat file disentuh ulang. Perbaikan: parse dari `[Name]`.
- **FullOuter merge langsung dipercaya** — perbaikan: bandingkan `Table.RowCount` sebelum/sesudah join.
- **Dedup tanpa sort** — keep-first tidak deterministik antar refresh. Perbaikan: sort eksplisit dulu.

## Tips Pro

- Bungkus resep yang sering dipakai menjadi `fn*` function — repo pribadi function yang bisa di-copy antar PBIX.
- Simpan resep ini di file `.pbit` template agar proyek baru langsung siap.
- `Table.Buffer` untuk lookup kecil yang dipakai berulang dalam satu query.
- Setiap resep yang menulis ulang data (merge, fuzzy, konversi) wajib punya kolom audit — jangan percaya transformasi tanpa jejak.

## Terkait

- [M Language Basics](./01-m-language-basics.md)
- [Common Transformations](./02-common-transformations.md)
- [Parameters & Functions](./03-parameters-and-functions.md)
- [Star Schema](../02-data-modeling/01-star-schema.md)
- [Relationships](../02-data-modeling/02-relationships.md)
- [M Cheatsheet](../00-quick-reference/m-cheatsheet.md)
