# M Language Basics — Bahasa Power Query

> M (Power Query Formula Language) adalah bahasa fungsional di balik setiap tombol transformasi. Memahami `let...in`, tipe data, `each`, error handling, dan query folding membuat Anda tidak sekadar menekan tombol — tapi membaca, memperbaiki, dan menulis query sendiri.

## Apa Itu M

- M adalah bahasa **fungsional, case-sensitive, dan lazy evaluation**.
- Setiap klik di UI Power Query editor menghasilkan kode M; **Advanced Editor** menampilkan semuanya.
- Satu query = rantai langkah (Applied Steps); setiap langkah adalah ekspresi yang menghasilkan tabel untuk langkah berikutnya.

## Struktur let...in

```powerquery
let
    Langkah1 = ...,
    Langkah2 = f(Langkah1),
    Hasil    = g(Langkah2)
in
    Hasil
```

- `let` mendefinisikan serangkaian nama (step), `in` menentukan nilai yang dikembalikan.
- Nama step boleh mengandung spasi dengan `#"Nama Step"` — alasan mengapa rename step di UI menghasilkan tanda pagar.

## Case Sensitivity & #identifier

- `Table.SelectRows` benar; `table.selectrows` error — nama fungsi case-sensitive.
- Identifier ber-spasi atau diawali angka harus ditulis `#"Nama Step"` atau `#"2025 Data"`.
- String memakai tanda kutip ganda: `"AYAM"`.

## Tipe Data

| Tipe | Literal / contoh | Catatan |
|---|---|---|
| `text` | `"AYAM"` | Perbandingan `==` case-sensitive |
| `number` | `42`, `3.14` | Mencakup int/decimal/currency |
| `logical` | `true`, `false` | Huruf kecil semua |
| `date` | `#date(2025, 3, 1)` | Tanpa jam |
| `datetime` | `#datetime(2025, 3, 1, 8, 0, 0)` | Dengan jam |
| `duration` | `#duration(1, 0, 0, 0)` | 1 hari |
| `list` | `{1, 2, 3}` | Akses `list{0}` (zero-based) |
| `record` | `[A = 1, B = "x"]` | Akses `rec[A]` |
| `table` | `#table({...}, {...})` | Kolom + baris |
| `any` | — | Tidak pasti; hindari |

Tipe bisa nullable: `type nullable text`. Kolom bertipe `any` menandakan change type terlewat — beri tipe eksplisit agar load lebih cepat dan relasi tidak bermasalah.

## Operator Penting

- `=` : perbandingan sama dengan (case-insensitive) — dan sekaligus simbol binding di `let`. `"ayam" = "AYAM"` menghasilkan `true`.
- `==` : sama dengan **case-sensitive**. `"ayam" == "AYAM"` menghasilkan `false` — berguna membedakan key master yang mirip.
- `??` : coalesce — `[Nilai] ?? 0` mengembalikan 0 bila null.
- `?` : akses item/field opsional (postfix) — `Tabel{0}?` atau `Rec[Kolom]?` mengembalikan null alih-alih error bila tidak ada.
- `<>` : tidak sama dengan; logika memakai `and`, `or`, `not`.

## each dan Underscore

`each` adalah kependekan fungsi dengan parameter `_`:

```powerquery
Table.SelectRows(Tabel, each [Nilai] > 0)
-- identik dengan:
Table.SelectRows(Tabel, (_) => _[Nilai] > 0)
```

Di dalam `each`, penulisan `[Nilai]` sama artinya dengan `_[Nilai]` — kolom `Nilai` dari baris (record) saat ini.

## Komentar

```powerquery
// komentar satu baris
/* komentar
   multi baris */
```

Tulis komentar di Advanced Editor untuk mendokumentasikan langkah non-obvious (mis. alasan file tertentu dikecualikan).

## Query Folding — Apa & Kenapa Penting

Query folding = kemampuan Power Query **menerjemahkan langkah transformasi menjadi query asli** (mis. SQL `SELECT`/`WHERE` ke database) sehingga pekerjaan berat dilakukan di sumber.

- **Support**: SQL Server, Snowflake, OData, dan sebagian connector lainnya. `Table.SelectRows`, `Table.Group`, `Table.Sort` umumnya bisa dilipat ke SQL.
- **TIDAK support**: `Folder.Files`, Excel/CSV lokal, Google Drive, dan mayoritas `Web.Contents` — semua data diunduh dulu lalu diproses di memori mashup engine. Akibatnya refresh lebih berat; satu-satunya strategi adalah **filter dan buang kolom sedini mungkin**.
- Model user (folder Google Drive berisi file Excel per periode) berada di kategori tanpa folding — makanya polanya: filter file dulu, lalu per file diproses (unpivot + type), lalu langsung dirampingkan.
- Cara cek: klik kanan step → **View Native Query**. Bila aktif, folding masih utuh sampai step itu.

## Privacy Levels & Fast Combine

Saat query menggabungkan beberapa sumber (folder Google Drive + Google Sheets), mesin PQ mencegah kebocoran data antar sumber dengan **privacy levels** (Private/Organizational/Public). Konflik level memicu error `Formula.Firewall`. Opsi "Ignore privacy levels" (fast combine) memaksa jalan tetapi **berisiko kebocoran data** — gunakan hanya untuk data internal yang aman, dengan kesadaran penuh.

## Lazy Evaluation

Langkah M tidak dieksekusi berurutan saat Anda membuka editor — mesin mengevaluasi sesuai kebutuhan (lazy). Konsekuensi:

- Error bisa muncul "terlambat" — hanya saat data benar-benar diminta (preview/load).
- Optimasi dilakukan menyeluruh, bukan per step — jangan berasumsi step awal selalu dievaluasi penuh.
- `Table.Buffer(Tabel)` memaksa materialisasi ke memori — berguna bila tabel yang sama dipakai berulang (mis. dedup lalu dipakai untuk lookup).

## Error Handling: try ... otherwise

```powerquery
// Bila gagal, kembalikan null
NilaiBersih = try Number.From([Angka]) otherwise null

// Atau tangkap detail error sebagai record
Hasil = try Number.From([Angka]),
Cek   = if Hasil[HasError] then Hasil[Error][Message] else Hasil[Value]
```

`try` menghasilkan record `[HasError, Value, Error]` — pola dasar untuk menangani file korup dan konversi gagal (lihat [Resep PQ](./04-pq-patterns-recipes.md)).

## Sintaks & Contoh — Query Lengkap dengan Penjelasan Per Baris

```powerquery
let
    // 1. Ambil file Excel dari path (parameter FolderPath)
    File      = Excel.Workbook(File.Contents(FolderPath & "2025_01_W1.xlsx"), null, true),

    // 2. Ambil sheet bernama "Deviasi" dari workbook
    Sheet     = File{[Item = "Deviasi"]}[Data],

    // 3. Jadikan baris pertama sebagai header kolom
    DenganHdr = Table.PromoteHeaders(Sheet, [PromoteAllScalars = true]),

    // 4. Pastikan kolom Nilai bertipe number (locale id-ID: koma desimal)
    Bertipe   = Table.TransformColumnTypes(DenganHdr, {{"Nilai", type number}}, "id-ID"),

    // 5. Buang baris sampah: Nilai null atau 0
    Bersih    = Table.SelectRows(Bertipe, each [Nilai] <> null and [Nilai] <> 0),

    // 6. Standarkan key untuk join (TRIM + rapikan spasi ganda)
    KeyRapi   = Table.TransformColumns(Bersih,
        {{"Nama Barang", each Text.Trim(Text.Replace(_, "  ", " ")), type text}}),

    // 7. Simpan nama file sebagai kolom audit
    Audit     = Table.AddColumn(KeyRapi, "File Sumber", each "2025_01_W1.xlsx", type text)
in
    Audit
```

Penjelasan:

1. `Excel.Workbook` membongkar workbook menjadi objek (sheets/tables); argumen kedua-ketiga memakai default UI.
2. `{[Item="Deviasi"]}[Data]` — akses item di list berdasarkan kondisi field, lalu ambil field `Data`-nya.
3. `Table.PromoteHeaders` menaikkan baris pertama menjadi nama kolom.
4. `Table.TransformColumnTypes` dengan culture `"id-ID"` agar format Indonesia terbaca benar.
5. `each [Nilai] <> null and [Nilai] <> 0` — filter baris per baris.
6. `Table.TransformColumns` mengubah isi kolom tanpa menambah kolom baru.
7. `Table.AddColumn` dengan tipe eksplisit `type text` — jangan biarkan `any`.

## Debugging Step by Step

- Klik tiap step di **Applied Steps** — preview kanan menunjukkan bentuk data saat itu; cari step pertama di mana keanehan mulai muncul.
- Step diagnosa sementara: `Table.RowCount(NamaStep)`, `Table.FirstN(NamaStep, 10)`, `Table.ColumnNames(NamaStep)`.
- Cell merah: klik untuk melihat detail error. Kolom error: klik ikon error di header → **Keep Errors** untuk mengisolasi baris bermasalah.
- Rename step sesuai fungsi (`Filter Baris Sampah` lebih jelas daripada `Filtered Rows1`).

## Kesalahan Umum

- **Menulis nama step ber-spasi tanpa `#"..."`** → parse error. Field access seperti `[Nama Barang]` aman tanpa pagar; nama variabel/step tidak.
- **Mengira M seperti DAX (case-insensitive)** — `Each` atau `table.selectrows` → error.
- **Privacy levels diabaikan** sampai error `Formula.Firewall` muncul saat refresh di server. Perbaikan: rapikan level per sumber, hindari fast combine bila data sensitif.
- **Semua kolom dibiarkan `any`** — relasi gagal dibuat dan load lambat. Perbaikan: change type eksplisit untuk semua kolom.
- **Filter dilakukan paling akhir** pada sumber tanpa folding — memori membengkak. Perbaikan: filter rows dan remove columns di step paling awal yang memungkinkan.

## Tips Pro

- Buka **Advanced Editor** secara rutin dan baca kode hasil UI — cara tercepat mempelajari idiom M.
- Duplikat query (right-click → Duplicate) sebelum eksperimen besar.
- `??` untuk nilai default: `Table.AddColumn(Src, "Qty", each [QTY] ?? 0, type nullable number)`.
- Bungkus transformasi berulang jadi custom function (lihat [Parameters & Functions](./03-parameters-and-functions.md)).

## Terkait

- [Common Transformations](./02-common-transformations.md)
- [Parameters & Functions](./03-parameters-and-functions.md)
- [Resep PQ](./04-pq-patterns-recipes.md)
- [Star Schema](../02-data-modeling/01-star-schema.md)
- [M Cheatsheet](../00-quick-reference/m-cheatsheet.md)
