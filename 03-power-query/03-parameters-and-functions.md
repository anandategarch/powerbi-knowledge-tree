# Parameters & Custom Functions

> Parameter membuat path dan konfigurasi bisa diganti tanpa menyentuh kode; custom function membuat transformasi "per satu file" ditulis sekali dan dipakai untuk semua file. Kombinasi keduanya adalah tulang punggung pola Combine Files untuk folder Google Drive per periode.

## Parameter di Power Query

Parameter (Home → **Manage Parameters → New**) adalah nilai bernama yang bisa diganti dari luar query — termasuk lewat dataset settings di Power BI Service saat refresh terjadwal.

Perbedaan penting dua jenis "parameter":

| Jenis | Dibuat di | Kegunaan |
|---|---|---|
| **Power Query parameter** | PQ → Manage Parameters | Path folder, URL, SheetID — dipakai di dalam M |
| **What-if parameter** | Modeling → New Parameter | Slider numerik di report untuk skenario DAX |

Keduanya berbeda total; jangan tertukar. Untuk kebutuhan pipeline data (folder Google Drive user), yang dipakai adalah **Power Query parameter**:

```powerquery
// Parameter: FolderPath (Text) = "C:\Users\z\Google Drive\Cost Control\"
Sumber = Folder.Files(FolderPath)
```

Manfaat: pindah laptop / ganti environment (dev → produksi) cukup mengganti satu nilai, bukan mengedit banyak query.

## Custom Function — Bentuk Dasar

```powerquery
// Query terpisah bernama fnParseNamaFile
(nama as text) as date =>
    let
        Bagian = Text.Split(Text.BeforeDelimiter(nama, "."), "_"),
        Tahun  = Number.From(Bagian{0}),
        Bulan  = Number.From(Bagian{1})
    in
        #date(Tahun, Bulan, 1)
```

- `(arg as text) as date =>` — signature: tipe argumen dan tipe kembalian.
- Body hanyalah ekspresi `let...in` biasa.
- Simpan sebagai query; panggil dari query lain: `fnParseNamaFile("2025_03_W1.xlsx")` → `#date(2025, 3, 1)`.

## List.Transform — Fungsi Tanpa Query

Untuk transformasi kecil yang tidak perlu query terpisah:

```powerquery
NamaBersih = List.Transform(Files[Name], each Text.Lower(Text.Trim(_)))
```

`List.Transform(list, fungsi)` menerapkan fungsi ke setiap elemen — konsep `each` sama seperti di custom column, tapi untuk list.

## Pola Combine Files & Helper Queries Otomatis

Saat memilih folder lalu klik **Combine Files** (ikon dua panah bawah di kolom Content), Power Query otomatis membuat helper queries:

1. **Sample File** (parameter) — binary content satu file contoh yang Anda pilih di dialog.
2. **Transform Sample File** (function) — seluruh langkah transformasi untuk **satu** file; edit query ini untuk mengubah cara semua file diproses.
3. **Transform File** (function) — penghubung yang memanggil Transform Sample File untuk setiap file; dipakai oleh query utama.

Poin terpenting: **bila ada file yang gagal diproses, perbaikiannya ada di Transform Sample File / function pembungkusnya, bukan di query utama.** Query utama hanya: invoke function → expand.

## Sintaks & Contoh — Pola Folder Lengkap (Persis Kasus User)

Function untuk satu file (isi `fnTransformFile`):

```powerquery
fnTransformFile = (file as binary) as table =>
    let
        Buku   = Excel.Workbook(file, true),          // true = pakai baris 1 sebagai header
        Sheet  = Buku{[Item = "Deviasi"]}[Data],
        Bersih = Table.SelectRows(Sheet, each [Nama Barang] <> null)
    in
        Bersih
```

Query utama:

```powerquery
let
    // 1. Folder Google Drive yang di-sync lokal
    Sumber  = Folder.Files(FolderPath),

    // 2. Buang file tersembunyi, file lock Excel (~$), dan non-xlsx
    Bersih  = Table.SelectRows(Sumber, each
                  [Extension] = ".xlsx"
                  and not Text.StartsWith([Name], "~$")
                  and not [Attributes][Hidden]),

    // 3. Panggil custom function per file (Add Column → Invoke Custom Function)
    Invoked = Table.AddColumn(Bersih, "Data", each fnTransformFile([Content])),

    // 4. Parse periode dari nama file untuk audit & filtering
    Meta    = Table.AddColumn(Invoked, "PeriodeFile",
                  each fnParseNamaFile([Name]), type date),

    // 5. Buang kolom binary dan metadata tak perlu
    Rapikan = Table.SelectColumns(Meta,
                  {"Name", "Date modified", "PeriodeFile", "Data"}),

    // 6. Expand hasil transformasi menjadi satu tabel
    Hasil   = Table.ExpandTableColumn(Rapikan, "Data",
                  {"Resto", "Nama Barang", "Attribute", "Jenis", "Nilai"})
in
    Hasil
```

Penjelasan kunci: langkah 2 sering dilupakan sampai refresh error karena file `~$` sisa Excel terbuka ikut terbaca; langkah 3 adalah inti pola; langkah 6 menyatukan semua file menjadi satu fact table.

## Error Handling dalam Function

Function yang dipanggil untuk ratusan file harus tahan banting:

```powerquery
// Pembungkus aman: file korup tidak menghentikan seluruh refresh
fnTransformAman = (file as binary) as table =>
    try fnTransformFile(file)
    otherwise #table(
        {"Resto", "Nama Barang", "Attribute", "Jenis", "Nilai"},
        {{null, null, null, "ERROR FILE", null}}
    )
```

Baris "ERROR FILE" membuat file bermasalah tetap terlihat saat audit — lebih baik daripada refresh gagal total atau baris hilang diam-diam.

## Dynamic Google Sheets Source via Web.Contents

Master `Inventory` user berasal dari Google Sheets. Pola yang benar untuk refresh stabil (termasuk via gateway):

```powerquery
let
    SheetID = "1AbC…xyz",  // simpan sebagai parameter agar mudah diganti
    Sumber  = Web.Contents(
        "https://docs.google.com",                     // base URL statis
        [
            RelativePath = "spreadsheets/d/" & SheetID & "/export",
            Query        = [format = "xlsx"],
            IsRetry      = true
        ]),
    Buku    = Excel.Workbook(Sumber),
    Sheet   = Buku{[Item = "Master Barang"]}[Data]
in
    Sheet
```

Kenapa bentuk **base URL statis + RelativePath + Query**? Karena `Web.Contents` dengan URL gabungan dinamis dianggap **dynamic data source** — refresh di Power BI Service/gateway bisa ditolak atau menuntut approval ulang setiap berubah. Dengan base URL statis, perubahan `RelativePath`/`Query` tidak memicu konfigurasi ulang data source.

## Relative Path & Data Source Settings

- **Data source settings** (File → Options → Data source settings) menyimpan kredensial per base URL/path; bersihkan sumber lama agar tidak menumpuk.
- Path relatif bisa jalan di Desktop, tapi **refresh di Service membutuhkan gateway** yang mengakses path yang sama persis — pasangkan parameter `FolderPath` dengan gateway folder.
- Google Drive: pastikan folder ter-sync lokal (Google Drive for Desktop) dan gateway membaca path sync tersebut; sharing Google Sheets harus "Anyone with the link" (Viewer) agar endpoint `export` bisa diakses tanpa login browser.

## Kesalahan Umum

- **Path/URL hardcode** — pindah komputer = semua query rusak. Perbaikan: parameter `FolderPath` / `SheetID`.
- **`Web.Contents` dengan URL dinamis penuh** — error "dynamic data source" saat refresh di Service. Perbaikan: base URL statis + RelativePath + Query.
- **Lupa filter `~$` dan hidden file** — refresh gagal saat ada file sedang terbuka. Perbaikan: filter `[Name]`, `[Extension]`, `[Attributes][Hidden]`.
- **Sample file tidak representatif** — transformasi dibuat dari file lama yang strukturnya beda; file baru error. Perbaikan: pilih sample terbaru dan gunakan `MissingField.UseNull` untuk variasi kolom.
- **Function tanpa annotation tipe** — `(x) => ...` lolos ditulis tapi error muncul acak di runtime. Perbaikan: `(file as binary) as table => ...`.

## Tips Pro

- Konvensi penamaan: prefix `fn` untuk function (fnTransformFile), nama jelas untuk parameter (FolderPath, SheetID).
- Simpan `[Name]` dan `[Date modified]` dari Folder.Files sebagai kolom audit — jejak data sampai file aslinya.
- Parse periode dari `[Name]` (stabil, ikut konvensi YYYY_MM_periode.xlsx), bukan dari `[Date modified]` (berubah setiap kali file disentuh ulang).
- Bila mengembangkan dengan banyak file besar, matikan background data preview download (File → Options → Data Load) agar editor tetap responsif.

## Terkait

- [M Language Basics](./01-m-language-basics.md)
- [Common Transformations](./02-common-transformations.md)
- [Resep PQ](./04-pq-patterns-recipes.md)
- [Date Dimension](../02-data-modeling/04-date-dimension.md)
- [Star Schema](../02-data-modeling/01-star-schema.md)
- [M Cheatsheet](../00-quick-reference/m-cheatsheet.md)
