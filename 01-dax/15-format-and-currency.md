# Format Angka, Currency & Dynamic Format Strings

> Format menentukan bagaimana angka **tampil**, bukan nilainya — dari pemisah ribuan, Rupiah, abbreviations jutaan/miliar, sampai *dynamic format strings* yang berubah mengikuti slicer. File ini juga membahas jebakan terbesar: `FORMAT()` mengubah angka menjadi teks.

## Konsep & Penjelasan

### Tiga lapisan format

| Lapisan | Di mana diatur | Karakter |
|---|---|---|
| **Format model** (kolom & measure) | Model view → pilih kolom/measure → properti Format | Display-only; nilai tetap presisi penuh |
| **`FORMAT()` function** | Di dalam ekspresi DAX | **Mengembalikan teks** — nilai asli hilang untuk perhitungan lanjutan |
| **Dynamic format strings** (fitur 2024) | Model view → measure → Format → *Dynamic* | Ekspresi DAX menghasilkan string format, nilai **tetap numerik** |

### Format kolom vs measure

- **Kolom**: format menempel pada kolom (termasuk implicit measure saat kolom di-drag ke visual).
- **Measure**: format menempel pada measure — measure yang sama bisa tampil beda format dari kolom sumbernya.
- **Dynamic format strings** hanya tersedia untuk **measure eksplisit** (tidak untuk kolom/implicit measure) dan dipakai ketika satu measure harus tampil berbeda per konteks (NOMINAL vs QTY, IDR vs USD).

### Placeholder format string kustom

| Placeholder | Arti | Contoh string | Input → Hasil |
|---|---|---|---|
| `0` | Digit, selalu tampil (nol ditampilkan) | `0.00` | `1.5` → `1.50` |
| `#` | Digit signifikan saja | `#.##` | `1.5` → `1.5` |
| `.` | Pemisah desimal | `#,##0.00` | `1234567.8` → `1.234.567,80` (locale id-ID) |
| `,` | Pemisah ribuan | `#,##0` | `1234567` → `1.234.567` |
| `,` di akhir integer | Bagi 1.000 per koma | `#,##0,` | `1234567` → `1.235` |
| `,,` | Bagi 1.000.000 | `#,##0.0,,` | `1234567` → `1,2` |
| `,,,` | Bagi 1.000.000.000 | `#,##0.0,,,` | `1234567` → `0,0` |
| `%` | Kali 100 + tampil `%` | `0.0%` | `0.1234` → `12,3%` |
| `"teks"` / `'teks'` | Literal di dalam format | `"Rp "#,##0` | `1234` → `Rp 1.234` |
| `\` | Escape karakter berikutnya | `\R\p #,##0` | `1234` → `Rp 1.234` |
| `;` | Section: positif;negatif;nol | `#,##0;(#,##0);"–"` | `-1234` → `(1.234)` |

> **Catatan locale:** string format selalu ditulis dengan placeholder standar (`#,##0.00`), tapi tampilannya mengikuti locale model — di Indonesia ribuan jadi titik dan desimal jadi koma. Jangan "koreksi" placeholder menjadi `#.##0,00`.

### Rupiah & abbreviations

- Rupiah: `"Rp "#,##0` → `Rp 12.345.678` (gaya BI tanpa spasi: `"Rp"#,##0`).
- Jutaan: `"Rp "#,##0.0,," Jt"` → `Rp 12,3 Jt`.
- Miliar (konvensi Indonesia "M"): `"Rp "#,##0.0,,," M"` → `Rp 1,2 M`.
- Konvensi Inggris memakai `"M"` untuk juta (million) dan `"B"` untuk miliar — tentukan glossary tim sebelum campur.

### Format tanggal

| String | Hasil (contoh 17 Feb 2026) |
|---|---|
| `dd/mm/yyyy` | `17/02/2026` |
| `dd mmmm yyyy` | `17 Februari 2026` (ikuti locale) |
| `mmm yyyy` | `Feb 2026` |
| `yyyy-mm` | `2026-02` — dipakai kolom `Date[Period]` report ini |
| `ddddd` / predefined `"Long Date"` | Tanggal panjang locale |

## Sintaks & Contoh

### 1. Angka + satuan (pattern asli [Qty Waste])

```dax
-- Kartu/tabel display: teks gabungan angka + satuan
Qty Waste =
FORMAT(
    CALCULATE(
        SUM('Waste All'[Value]),
        Attribute[Attribute] = "QTY"
    ),
    "#,##0"
)
& " " &
MAX('Waste All'[SATUAN])
-- Hasil: "1.234 kg"  (PERHATIAN: ini sudah TEXT, bukan angka)
```

### 2. Format kondisional di dalam teks (pattern asli [Dynamic Dev])

```dax
Dynamic Dev =
SWITCH(
    SELECTEDVALUE(Attribute[Attribute]),
    "QTY", FORMAT([Deviasi CM], "#,##0") & " " & MAX('Deviasi All'[Satuan]),
    "NOMINAL",
        IF(
            [Deviasi CM] < 0,
            "-Rp" & FORMAT(ABS([Deviasi CM]), "#,##0"),   -- minus eksplisit
            "Rp" & FORMAT([Deviasi CM], "#,##0")
        )
)
```

### 3. Dynamic format strings (nilai TETAP numerik)

```dax
-- Langkah: pilih measure -> properti Format -> Dynamic -> tulis ekspresi:
Format String Multi Value =
SWITCH(
    SELECTEDVALUE('Attribute'[Attribute]),
    "QTY", "#,##0",
    "NOMINAL", """Rp""#,##0",     -- kutip ganda di-escape dengan ""
    "#,##0"                        -- default
)
```

Measure `[Multi Value CM]` tetap numerik — bisa dijumlahkan, dibandingkan, dan dipakai KPI/conditional formatting — sementara tampilannya berganti antara `12.345 kg` dan `Rp 12.345` mengikuti slicer Attribute. Trik `'"Rp"#,##0'` (kutip tunggal) menghindari escape ganda.

### 4. Dynamic currency dengan SELECTEDVALUE

```dax
-- Tabel 'Currency'[Code] = {"IDR","USD","EUR"} sebagai slicer (disconnected)
Format Mata Uang =
VAR Curr = SELECTEDVALUE('Currency'[Code], "IDR")
RETURN
SWITCH(
    Curr,
    "USD", "'$'#,##0.00",
    "EUR", "'€'#,##0.00",
    "'Rp'#,##0"                    -- default Rupiah
)
```

### 5. Format section untuk negatif & nol

```dax
-- positif;negatif;nol — negatif dalam kurung, nol jadi strip
-- (dipakai di properti Format measure, bukan di dalam FORMAT)
#,##0;(#,##0);"–"

-- FORMAT() versi-nya (hasil teks):
FORMAT([Selisih], "#,##0;(#,##0)")
```

## Kesalahan Umum

- **`FORMAT()` lalu dijumlahkan/di-chart.** Hasilnya teks: `SUM` gagal, sumbu X chart jadi kategori, dan sortir alfabetis (`"10 kg"` terurut sebelum `"9 kg"`). Perbaikan: pisahkan *measure inti* (numerik) dan *measure display* (teks), atau pakai **dynamic format strings** agar tetap numerik.
- **Mengira `%` hanya menempel simbol.** Placeholder `%` **mengalikan 100** — `FORMAT(0.12, "0%")` → `12%`. Kalau measure Anda sudah dikali 100 manual, hasilnya 1.200%.
- **Menulis placeholder ikut locale** (`#,##0.00` diubah jadi `#.##0,00`) → format rusak. Placeholder selalu gaya invariant.
- **Mismatch total karena rounding display.** Nilai asli 1.234,56 tampil `1.235` tapi total menjumlah nilai asli → "tidak nyambung". Jika ingin konsisten, bulatkan di measure dengan `ROUND(..., 0)`.
- **`MAX('Waste All'[SATUAN])` kosong saat multi-item.** Konteks mencakup banyak satuan → `MAX` mengembalikan satu nilai acak atau BLANK, teks jadi `"1.234 "`. Perbaikan: bungkus `SELECTEDVALUE('Waste All'[SATUAN], "")` atau tampilkan satuan sebagai kolom terpisah.
- **Format diterapkan di kolom, lalu heran implicit measure ikut berubah.** Format kolom menular ke semua visual yang memakai kolom itu; gunakan measure eksplisit bila butuh format khusus.
- **Mengira format mengubah nilai.** Format 100% display; audit & export tetap membaca nilai penuh.

## Tips Pro

- Kombinasi juara untuk report multi-metrik: `[Multi Value CM]` (numerik) + **dynamic format strings** + slicer `Trend`/`Attribute` — satu visual melayani DEVIASI/BOM/%BOM/%SALES tanpa measure teks per metrik.
- Abbreviation terbaik untuk KPI card besar: `"Rp "#,##0.0,," Jt"` dan `"Rp "#,##0.0,,," M"` — hemat ruang, tetap angka.
- Simpan konvensi format di satu tempat (mis. tabel `Format` atau documentation page) supaya `Rp` konsisten di 36 halaman.
- Dynamic format strings + `SELECTEDVALUE` adalah pengganti bersih untuk `FORMAT()`-dan-`&`-satuan; gunakan teks hanya untuk label gabungan yang benar-benar naratif (mis. `"Last Update : " & FORMAT(...)`).
- Untuk kolom bulan teks, selalu pasangkan *Sort by column* ke `MonthNumber` agar urutan visual & `ORDERBY` window function benar — lihat [Window Functions](./14-window-functions.md).

## Terkait

- [Text Functions](./11-text-functions.md) — FORMAT & manipulasi teks lebih lanjut
- [Variabel & Error Handling](./13-variables-and-error-handling.md) — `SELECTEDVALUE` + `COALESCE` untuk label
- [Resep DAX](./16-dax-patterns-recipes.md) — resep Dynamic Dev & measure switching
- [Performa DAX](./17-dax-performance.md) — biaya `FORMAT` di iterator besar
- [DAX Cheatsheet](../00-quick-reference/dax-cheatsheet.md) — snippet teks angka + satuan
- [Analisis Model & DAX Report](../11-user-report-analysis/model-and-dax.md) — measure `Qty Waste`, `Dynamic Dev`, `Dynamic Gap Dev` versi asli
