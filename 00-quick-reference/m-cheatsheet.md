# Power Query M Cheatsheet

> Kartu hafalan bahasa M: struktur `let...in`, tipe data, operator (`??`, `?`), fungsi esensial per kategori, dan pattern siap pakai — termasuk pattern asli report Cost Control (combine folder Google Drive, unpivot `QTY_*`/`NOMINAL_*`, parse tanggal dari nama file).

## Konsep: Struktur & Tipe

### Anatomi `let...in`

Setiap query M adalah ekspresi `let` berisi langkah berurutan, diakhiri `in` yang menentukan output:

```m
let
    Sumber = Excel.Workbook(File.Contents("H:\Data\Deviasi.xlsx"), true),
    Sheet = Sumber{[Item = "Sheet1", Kind = "Sheet"]}[Data],
    Header = Table.PromoteHeaders(Sheet, [PromoteAllScalars = true]),
    Tipe = Table.TransformColumnTypes(Header, {{"Value", type number}}, "id-ID")
in
    Tipe
// Komentar satu baris pakai //
/* komentar
   multi baris */
```

### Tipe data M

| Tipe | Contoh literal | Catatan |
|---|---|---|
| `number` | `1`, `2.5`, `#infinity` | Membedakan `Int64.Type` hanya di *facet* tipe kolom |
| `text` | `"Deviasi"` | Case-sensitive & immutable |
| `logical` | `true` / `false` | |
| `null` | `null` | "Tidak ada nilai" — beda dari error |
| `date` / `time` / `datetime` / `datetimezone` | `#date(2026,2,1)`, `#datetime(2026,2,1,8,0,0)` | |
| `duration` | `#duration(0,8,0,0)` | 8 jam |
| `list` | `{1, 2, 3}`, `{1..5}` | Index mulai 0: `List{0}` |
| `record` | `[Nama = "BOM", Qty = 10]` | Akses: `[Nama]` |
| `table` | `#table({"A","B"}, {{1,2}})` | Kolom → list of records |
| `binary` / `type` / `function` / `any` | `type text`, `each _` | `any` = tipe generik; hindari di kolom (matikan folding) |

### Operator penting

| Operator | Fungsi | Contoh |
|---|---|---|
| `??` | Coalesce — ganti `null` (bukan error!) | `[Qty] ?? 0` |
| `?` | Akses opsional — error jadi `null` | `[Attributes]?[Hidden]?` |
| `=` / `<>` | Sama / tidak sama (case-sensitive) | `[Nama] = "BOM"` |
| `&` | Gabung teks (atau list/record) | `"Rp " & [Kode]` |
| `&&`, `\|\|`, `not` | Logika boolean | `[A] > 0 && not [B]` |
| `=>` dan `each` | Fungsi lambda; `each _` = parameter implisit | `each [Qty] > 0` ≡ `(_) => _[Qty] > 0` |
| `try ... otherwise` | Tangkap error | `try Number.From([X]) otherwise null` |
| `#name` | Literal khusus | `#date(...)`, `#shared` (semua fungsi + kueri) |

> **`??` vs `try...otherwise`:** `??` hanya menangani `null`; `try...otherwise` menangani *error* (mis. konversi gagal). Keduanya berbeda kasus.

## Sintaks & Contoh: Fungsi Esensial

### Akses data

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `Folder.Files` | Daftar file dalam folder (recursive: `Folder.Contents`) | `Folder.Files("H:\My Drive\COST CONTROL")` |
| `Excel.Workbook` | Buka workbook dari binary | `Excel.Workbook([Content], true)` |
| `Csv.Document` | Parse CSV (binary) | `Csv.Document(bin, [Delimiter=",", Encoding=65001])` |
| `Web.Contents` | Ambil konten URL (Google Sheets publish) | `Web.Contents("https://docs.google.com/.../pub?output=csv")` |
| `File.Contents` | Baca file jadi binary | `File.Contents("H:\Data\x.xlsx")` |
| `Excel.CurrentWorkbook` | Tabel dari file Excel aktif | `Excel.CurrentWorkbook(){[Name="Tabel1"]}[Content]` |

### Transformasi tabel

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `Table.SelectRows` | Filter baris | `Table.SelectRows(T, each [Value] > 0)` |
| `Table.SelectColumns` | Pilih kolom | `Table.SelectColumns(T, {"Nama Barang", "Value"})` |
| `Table.RemoveColumns` | Buang kolom | `Table.RemoveColumns(T, {"Helper"})` |
| `Table.TransformColumnTypes` | Set tipe (dengan locale) | `Table.TransformColumnTypes(T, {{"Tanggal", type date}}, "id-ID")` |
| `Table.RenameColumns` | Ganti nama kolom | `Table.RenameColumns(T, {{"QTY", "Qty"}})` |
| `Table.AddIndexColumn` | Kolom nomor urut | `Table.AddIndexColumn(T, "No", 1, 1)` |
| `Table.UnpivotOtherColumns` | Lebar → panjang (sisakan kolom kunci) | `Table.UnpivotOtherColumns(T, {"Nama Barang"}, "Header", "Value")` |
| `Table.Pivot` | Panjang → lebar | `Table.Pivot(T, List.Distinct(T[Header]), "Header", "Value", List.Sum)` |
| `Table.Group` | Agregasi per grup | `Table.Group(T, {"RESTO"}, {{"Total", each List.Sum([Value]), type number}})` |
| `Table.ExpandTableColumn` | Buka kolom bertipe table | `Table.ExpandTableColumn(T, "Detail", {"Qty"})` |
| `Table.Combine` | Gabung banyak tabel (stack) | `Table.Combine({T1, T2})` |
| `Table.MergeQueries` | Join (semua join kind) | `Table.MergeQueries(T, Master, {"Kode"}, {"Kode"}, JoinKind.LeftOuter)` |
| `Table.NestedJoin` | Join tanpa auto-expand | `Table.NestedJoin(T, {"Kode"}, Master, {"Kode"}, "J", JoinKind.LeftOuter)` |
| `Table.Distinct` | Hapus duplikat | `Table.Distinct(T, {"Nama Barang"})` |
| `Table.Sort` | Urutkan | `Table.Sort(T, {{"Tanggal", Order.Ascending}})` |
| `Table.AddColumn` / `Table.TransformColumns` | Kolom baru / ubah isi kolom | `Table.AddColumn(T, "Rp", each [Value] * 1000)` |
| `Table.PromoteHeaders` | Baris 1 jadi header | `Table.PromoteHeaders(T, [PromoteAllScalars=true])` |

### Teks

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `Text.Split` | Pecah jadi list per delimiter | `Text.Split("2026_05_Deviasi.xlsx", "_")` |
| `Text.BeforeDelimiter` / `Text.AfterDelimiter` | Ambil sebelum/sesudah delimiter | `Text.BeforeDelimiter([Nama], ".1-G")` |
| `Text.Start` / `Text.End` | Ambil N karakter awal/akhir | `Text.Start([Kode], 1)` |
| `Text.Trim` / `Text.Upper` | Rapikan spasi / huruf besar | `Text.Upper(Text.Trim([CABANG]))` |
| `Text.Contains` | Cek substring | `Text.Contains([Nama], "FROZEN")` |
| `Text.Length` / `Text.Replace` | Panjang / ganti teks | `Text.Replace([Nama], "  ", " ")` |
| `Number.FromText` / `Text.From` | Konversi teks↔angka | `Number.FromText(Text.Start([File], 4))` |

### List

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `List.Contains` | Cek anggota | `List.Contains({"Waste","Susut"}, [Jenis])` |
| `List.Distinct` | Nilai unik | `List.Distinct(T[Jenis])` |
| `List.Sum` / `List.Average` | Jumlah / rata-rata | `List.Sum([Value])` |
| `List.Transform` | Map tiap elemen | `List.Transform({1..3}, each _ * 2)` |
| `List.Select` | Filter elemen | `List.Select(Files, each Text.EndsWith(_, ".xlsx"))` |
| `List.First` / `List.Last` / `List.Count` | Elemen pertama/terakhir/jumlah | `List.First(Text.Split([Nama], "_"))` |

### Date & logika

| Fungsi | Kegunaan | Contoh singkat |
|---|---|---|
| `Date.From` | Konversi ke date | `Date.From([Tanggal Text])` |
| `#date(y, m, d)` | Literal tanggal | `#date(2026, 2, 1)` |
| `Date.Year` / `Date.Month` / `Date.AddDays` | Komponen & aritmetika tanggal | `Date.Year([Tanggal])` |
| `if...then...else` | Kondisional | `if [Qty] > 0 then [Qty] else 0` |
| `each ...` | Fungsi implisit `_` | `each [Value] / [Total]` |
| `error` | Raise error manual | `error "Kolom tidak ditemukan"` |

### Pattern 1 — Combine folder Excel (pattern asli report)

```m
let
    Sumber = Folder.Files("H:\My Drive\COST CONTROL\Deviasi"),
    FileBersih = Table.SelectRows(
        Sumber,
        each not Text.StartsWith([Name], "~$")          // buang file temp Excel
            and [Extension] = ".xlsx"
            and not ([Attributes]?[Hidden]? = true)     // operator ? yang aman
    ),
    ParseFile = (file) =>
        let
            Buku  = Excel.Workbook(file[Content], true),
            Sheet = Buku{[Item = "Sheet1", Kind = "Sheet"]}[Data],
            DgnNamaFile = Table.AddColumn(Sheet, "Source.Name", each file[Name])
        in
            DgnNamaFile,
    SemuaTabel = Table.AddColumn(FileBersih, "Hasil", each ParseFile(_)),
    Gabung = Table.Combine(SemuaTabel[Hasil])
in
    Gabung
```

### Pattern 2 — Unpivot + split delimiter (NOMINAL/QTY)

```m
// Kolom QTY_BOM, NOMINAL_WASTE, QTY_SUSUT, ... diubah jadi baris
Unpivot = Table.UnpivotOtherColumns(
    Data,
    {"Nama Barang", "RESTO", "Satuan"},   // kolom kunci yang dipertahankan
    "Header",
    "Value"
),
SplitAttr = Table.SplitColumn(
    Unpivot, "Header",
    Splitter.SplitTextByDelimiter("_", QuoteStyle.Csv),
    {"Attribute", "Jenis"}                // "QTY_BOM" -> QTY + BOM
),
Tipe = Table.TransformColumnTypes(
    SplitAttr,
    {{"Attribute", type text}, {"Jenis", type text}, {"Value", type number}},
    "id-ID"
)
```

### Pattern 3 — Parse tanggal dari nama file (pattern asli report)

```m
// Nama file: "2026_05_Deviasi.xlsx"  ->  #date(2026, 5, 1)
TanggalDariNama = (namaFile as text) as date =>
    let
        Bagian = Text.Split(namaFile, "_"),
        Tahun  = Number.FromText(Bagian{0}),
        Bulan  = Number.FromText(Bagian{1})
    in
        #date(Tahun, Bulan, 1)

// Pemakaian di AddColumn:
TambahTanggal = Table.AddColumn(
    Sebelumnya, "Tanggal",
    each TanggalDariNama([Source.Name]),
    type date
)
```

> Alternatif DAX di report ini: `DATE(VALUE(MID([Source.Name],7,4)), ...)` — lakukan di M agar lebih robust dan sekali saat refresh.

### Pattern 4 — Google Sheets publish & error handling

```m
// Master Inventory via Google Sheets publish to web (pattern asli report)
let
    Csv = Csv.Document(
        Web.Contents("https://docs.google.com/spreadsheets/d/e/.../pub?output=csv"),
        [Delimiter = ",", Encoding = 65001]
    ),
    Header = Table.PromoteHeaders(Csv, [PromoteAllScalars = true]),
    Aman = Table.AddColumn(Header, "Qty", each try Number.From([Qty]) otherwise null, type nullable number)
in
    Aman
```

## Kesalahan Umum

- **`??` dipakai untuk menangkap error** — `??` hanya mengganti `null`; konversi gagal tetap error → gunakan `try ... otherwise`.
- **Lupa `in` atau nama step salah kapital** — M **case-sensitive**: `table.selectrows` ≠ `Table.SelectRows`.
- **`each` bersarang menimpa `_`** — `_` di dalam `each` terdalam merujuk scope-nya sendiri; beri nama parameter eksplisit `(r) => r[...]` bila butuh record luar.
- **Index list out of range** — `Text.Split(...){5}` error bila hanya 3 elemen; gunakan `List.First`/`try...otherwise` atau cek `List.Count`.
- **Merge tanpa `Table.Distinct`** di tabel referensi → hasil kartesian (baris mengganda).
- **Transformasi berat sebelum filter** — selalu filter baris/kolom dulu (`Table.SelectRows`/`SelectColumns`) supaya *query folding* tetap hidup dan data diproses minimal.
- **Hardcode locale** di `Table.TransformColumnTypes` tanpa `"id-ID"` — tanggal `01/02/2026` terbaca 2 Januari di locale en-US.

## Tips Pro

- **Query folding:** klik kanan step → *View Native Query*. Bila bisa dilihat, transformasi "difold" ke sumber (SQL) — refresh jauh lebih cepat. Step seperti `Table.Buffer` atau fungsi custom biasanya memutus folding.
- `Table.Buffer(Tabel)` untuk tabel kecil yang dipakai berulang (mis. master) — mencegah M membaca ulang sumber di tiap referensi; jangan dipakai di tabel besar (memori).
- Kelompokkan query: folder `Parameters`, `Functions`, `Master`, `Fact` di panel Queries — report ini memakai *Transform File function* + helper queries.
- `#shared` di blank query menampilkan semua fungsi & environment — cara tercepat belajar signature fungsi.
- Pattern rapuh "tanggal dari nama file" tetap butuh validasi: tambahkan step `try ... otherwise null` + kolom audit bila vendor mengubah konvensi nama.

## Terkait

- [DAX Cheatsheet](./dax-cheatsheet.md) — pasangan sisi model
- [Glosarium](./glossary.md) — istilah ETL, gateway, query folding
- [Performa DAX](../01-dax/17-dax-performance.md) — setelah ETL bersih, optimasi measure
- [Overview Report](../11-user-report-analysis/REPORT_OVERVIEW.md) — pattern PQ asli report Cost Control
