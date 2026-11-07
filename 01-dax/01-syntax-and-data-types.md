# Sintaks DAX & Tipe Data

> Fondasi semua formula DAX: anatomi formula, aturan referensi objek, operator, dan tipe data beserta konversi implisitnya. Banyak "bug aneh" di report — filter tidak jalan, angka tiba-tiba Blank, error *String* — sebenarnya berakar dari kesalahan sintaks dasar, bukan dari logika bisnis.

## Anatomi Formula DAX

### Bagian-Bagian Formula

Setiap formula DAX — measure maupun calculated column — terdiri atas tiga bagian: **nama objek**, operator **`=`**, dan **ekspresi**.

```text
% BOM CM  =  DIVIDE ( [Deviasi CM] , [Sales CM] )
└───┬────┘ └┬┘ └───┬───┘ └─────┬─────┘ └─────┬─────┘
  nama      sama    fungsi    argumen 1     argumen 2
 (measure)  dengan           (referensi    (referensi
            operator         measure)      measure)
```

- **Nama** — boleh mengandung spasi dan angka; harus unik dalam satu model; tidak peka kapital.
- **`=`** — operator assignment; DAX tidak mengenal `:=`.
- **Ekspresi** — kombinasi fungsi, operator, dan referensi yang menghasilkan satu nilai skalar (measure) atau satu nilai per baris (calculated column).

### Referensi Tabel, Kolom, dan Measure

| Objek | Sintaks | Contoh | Catatan |
|---|---|---|---|
| Kolom | `'Tabel'[Kolom]` | `'Deviasi All'[Value]` | Selalu sertakan nama tabel |
| Measure | `[Nama]` | `[Deviasi CM]` | Tanpa nama tabel (best practice) |
| Tabel | `'Nama Tabel'` | `'Resto'` | Quote wajib bila ada spasi |
| Kolom tanpa kualifikasi | `[Value]` | — | Hanya valid di calculated column tabel yang sama; hindari |

Aturan penting:

- **Single quote** `'...'` membungkus nama objek (tabel/kolom) yang mengandung spasi — `'Deviasi All'`, `'Resto'`. Tanpa spasi sebenarnya boleh tanpa quote (`Resto[Resto]`), tapi konsisten memakai quote lebih aman.
- **Double quote** `"..."` untuk string literal — `"WASTE"`, `"NOMINAL"`. DAX tidak mengenal string berkutip satu.
- Dalam measure, `[X]` selalu diartikan **measure**. Bila nama kolom dan nama measure bentrok, tulis kolom secara eksplisit `'Tabel'[Kolom]` supaya tidak ambigu.

### Case-Insensitivity

- Nama fungsi, keyword, dan objek tidak peka kapital: `sum(...)` sama dengan `SUM(...)`; `[deviasi cm]` sama dengan `[Deviasi CM]`.
- Dua objek tidak boleh berbeda hanya karena kapitalisasi.
- Perbandingan string pun tidak peka kapital: `'Deviasi All'[Jenis] = "waste"` identik dengan `"WASTE"`; `DISTINCTCOUNT` menganggap `"Waste"` dan `"WASTE"` sebagai satu nilai.

### Komentar, Multi-baris, dan IntelliSense

```dax
-- Komentar satu baris
//  Alternatif satu baris (didukung Power BI Desktop versi modern)
/* Komentar multi-baris —
   berguna untuk menonaktifkan sebagian formula saat debugging */
```

- **Alt+Enter** membuat baris baru di formula bar — formula panjang seperti [Z Score] wajib ditulis rapi.
- IntelliSense: ketik `'` untuk daftar tabel, `[` untuk daftar measure, **Ctrl+Space** untuk memunculkan ulang saran.
- Format otomatis: DAX Studio, Tabular Editor, atau daxformatter.com.

## Operator DAX

### Tabel Operator

| Kategori | Operator | Arti | Contoh |
|---|---|---|---|
| Aritmatika | `+` `-` `*` `/` | tambah, kurang, kali, bagi | `[Deviasi CM] - [Deviasi LM]` |
| Aritmatika | `^` | pangkat | `2 ^ 10` |
| Perbandingan | `=` | sama dengan (dengan konversi implisit) | `'Deviasi All'[Jenis] = "WASTE"` |
| Perbandingan | `==` | sama dengan ketat (tanpa konversi implisit) | `BLANK() == 0` menghasilkan FALSE |
| Perbandingan | `>` `<` `>=` `<=` `<>` | lainnya | `[Sales CM] > 0` |
| Logika | `&&` | AND | `[Sales CM] > 0 && [Deviasi CM] < 0` |
| Logika | `||` | OR | `[Deviasi CM] > 0 \|\| ISBLANK ( [Deviasi CM] )` |
| Logika | `IN` | anggota daftar nilai | `'Resto'[Area] IN { "BALI", "JATIM" }` |
| Logika | `NOT` | negasi | `NOT ISBLANK ( [Deviasi CM] )` |
| Teks | `&` | penggabungan string | `"Rank " & [Rank Dev]` |
| Unary | `-` | negasi | `-[Deviasi CM]` |

Catatan: DAX **tidak punya operator `%`** — persen hanyalah format tampilan. Tulis `0.02 * [BOM CM]`, bukan `2% * [BOM CM]`.

### `=` vs `==`

| Ekspresi | Hasil | Alasan |
|---|---|---|
| `"5" = 5` | TRUE | konversi implisit text menjadi number |
| `"5" == 5` | FALSE | tipe berbeda, tanpa konversi |
| `BLANK() = 0` | TRUE | BLANK dikonversi menjadi 0 |
| `BLANK() == 0` | FALSE | tipe berbeda, tanpa konversi |
| `"WASTE" = "waste"` | TRUE | string case-insensitive |

Gunakan `==` saat Anda perlu benar-benar membedakan BLANK/kosong dari nol — misalnya filter `'Deviasi All'[Value] == 0` agar baris Blank tidak ikut terpilih.

### Prioritas Operator

Urutan dari yang paling kuat mengikat: `^` → unary `-` → `*` `/` → `+` `-` → `&` → perbandingan (`=`, `==`, `<`, `>`, `<=`, `>=`, `<>`, `IN`) → `NOT` → `&&` → `||`.

Tulis tanda kurung untuk semua kombinasi non-trivial — pembaca formula (dan diri Anda tiga bulan kemudian) tidak perlu menghafal tabel prioritas.

## Tipe Data

### Daftar Tipe Data

| Tipe (di UI Power BI) | Penyimpanan | Catatan |
|---|---|---|
| Integer (Whole Number) | 64-bit | presisi penuh untuk hitungan |
| Decimal (Decimal Number) | double 64-bit floating point | presisi ~15 digit; rawan pembulatan |
| Currency (Fixed Decimal) | fixed-point, 4 desimal | ideal untuk uang: pembulatan 4 desimal, rentang ±922 triliun |
| DateTime | double — bagian bulat = hari sejak 1899-12-30, fraksi = jam | bisa dijumlah/dikurang angka |
| Boolean | TRUE / FALSE | otomatis menjadi 1/0 dalam aritmatika |
| String (Text) | Unicode | tidak bisa di-SUM; bisa MIN/MAX |
| BLANK / null | penanda "kosong" | bukan `0`, bukan `""` — tapi bisa dikonversi menjadi keduanya |
| Binary | blob dari Power Query | tidak dipakai dalam formula DAX biasa |

### Konversi Implisit

- Aritmatika: teks dikonversi ke angka bila memungkinkan — `"5" * 2` menghasilkan `10`; konversi gagal berarti error (`"lima" * 2`).
- Boolean dalam aritmatika: `TRUE + 1` menghasilkan `2`.
- BLANK dalam aritmatika: `BLANK() + 10` menghasilkan `10`, `BLANK() * 10` menghasilkan `0`.
- Perbandingan: `"5" = 5` TRUE; BLANK dibandingkan dengan angka menjadi 0, dengan teks menjadi `""`, dengan Boolean menjadi FALSE.
- Pencegahan: gunakan konversi eksplisit `VALUE()`, `FORMAT()`, `INT()`, `DATEVALUE()` bila tipe data sumber tidak pasti.

### Perilaku BLANK / Null

- `SUM` atas kolom yang seluruhnya Blank menghasilkan **BLANK, bukan 0** → sel visual tampil kosong. Butuh tampilan 0: `COALESCE ( [Deviasi CM], 0 )`.
- `DIVIDE ( 1, 0 )` menghasilkan BLANK (aman); operator `/` dengan pembagi nol dan pembilang bukan nol menghasilkan **error**. Utamakan `DIVIDE`.
- `ISBLANK ( "" )` FALSE — empty string bukan BLANK; `COUNTBLANK` menghitung keduanya.
- Fungsi terkait: `BLANK()`, `ISBLANK()`, `COALESCE()`.

### DateTime

- Tersimpan sebagai angka: `'Date'[Date] + 7` menghasilkan tanggal tujuh hari kemudian.
- Tabel `Date = CALENDARAUTO()` membuat rentang tanggal dari data model; kolom turunan (`Year`, `Month`, `MonthNumber`) biasanya dibuat sebagai calculated column.
- `FORMAT ( 'Date'[Date], "MMM-YYYY" )` menghasilkan **teks** — bagus untuk label, tidak untuk axis kontinu atau time intelligence.

## Sintaks & Contoh

```dax
-- Measure dengan operator aritmatika dan referensi measure
Selisih = [Deviasi CM] - [Deviasi LM]

-- Operator logika && , NOT , IN , dan SELECTEDVALUE
Resto Prioritas =
IF (
    [Sales CM] > 0
        && NOT ISBLANK ( [Deviasi CM] )
        && SELECTEDVALUE ( 'Resto'[Area] ) IN { "BALI", "JABAR", "JATIM" },
    "Prioritas",
    "Reguler"
)

-- Penggabungan teks; kutip ganda di dalam string ditulis dua kali
Label Waste =
    "Waste """
        & SELECTEDVALUE ( 'Resto'[Resto] )
        & """ = "
        & FORMAT ( [Qty Waste], "#,##0" )

-- Konversi eksplisit text menjadi number
Standar Susut Pct = VALUE ( "0.02" )   -- hasil 0.02 (Decimal)

-- Komentar sebagai dokumentasi measure
/* Kategori: klasifikasi arah deviasi bulan berjalan.
   Positif = surplus stok, negatif = loss. */
Kategori = IF ( [Deviasi CM] >= 0, "Surplus", "Loss" )
```

## Kesalahan Umum

- **Mengira `BLANK() = 0` FALSE** — padahal TRUE karena konversi implisit; filter `= 0` ikut menangkap baris kosong. Perbaikan: gunakan `==`, `ISBLANK()`, atau `COALESCE()`.
- **Single quote untuk string** — `'Deviasi All'[Jenis] = 'WASTE'` error. String wajib double quote: `"WASTE"`.
- **Nama tabel ber-spasi tanpa quote** — `Deviasi All[Value]` error sintaks. Harus `'Deviasi All'[Value]`.
- **Membandingkan teks dengan angka** — `'Periode'[Periode] = 1` padahal isinya `"WEEK 1"` → konversi gagal, hasil kosong/error. Perbaikan: samakan tipe kedua sisi.
- **Kolom tanpa nama tabel dalam measure** — `[Value]` dianggap measure bernama Value → error "column cannot be found". Kolom selalu `'Deviasi All'[Value]`.
- **Menganggap `%` operator** — `50% * [Sales CM]` error; tulis `0.5 * [Sales CM]`.
- **Tercampur `""` dan BLANK** — `ISBLANK("")` FALSE tapi `COUNTBLANK` menghitung keduanya; bersihkan string kosong di Power Query.

## Tips Pro

- Dua aturan emas yang menghilangkan 90% ambiguitas: kolom selalu berkualifikasi `'Tabel'[Kolom]`, measure selalu `[Nama]` tanpa nama tabel.
- Untuk kolom uang (rupiah), pilih tipe Currency/Fixed Decimal agar pembulatan deterministik 4 desimal.
- Beri nama measure dengan pola "metrik + konteks periode" seperti di report cost-control: `[Deviasi CM]`, `[Deviasi LM]`, `[% BOM CM]` — mudah dicari di antara ratusan measure.
- Alt+Enter + indentasi 4 spasi + komentar `--` per blok logika sebelum menyimpan formula.
- Formula panjang bukan masalah; formula yang tidak terbaca barulah masalah — bongkar dengan `VAR` (lihat [Fungsi Iterator](./05-iterator-functions.md)).

## Terkait

- [Row Context vs Filter Context](./02-row-context-vs-filter-context.md)
- [Measures vs Calculated Columns](./03-measures-vs-calculated-columns.md)
- [Fungsi Agregasi](./04-aggregation-functions.md)
- [Analisis Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md)
