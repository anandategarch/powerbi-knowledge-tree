# Fungsi Agregasi DAX

> Fungsi agregasi merangkum nilai **kolom** sesuai filter context: menjumlahkan, merata-ratakan, mencari min/max, dan menghitung. Aturan terpentingnya: fungsi tanpa akhiran X (`SUM`, `COUNT`, dst.) hanya menerima **kolom** — untuk expression per baris gunakan versi ber-akhiran X (`SUMX`, `AVERAGEX`, dst.).

## Dua Keluarga Agregasi

### Agregasi Kolom (Tanpa X)

Dijalankan di storage engine (VertiPaq) secara vektorial — cepat dan bisa di-cache — tetapi argumennya **harus kolom fisik**:

```dax
Total Deviasi = SUM ( 'Deviasi All'[Value] )   -- jumlahkan isi kolom
Sales CM      = SUM ( 'Sales All'[Penjualan Bersih] )
```

### Iterator (Ber-Akhiran X)

Versi X menerima **tabel + expression** dan mengevaluasi expression baris demi baris dalam row context:

```dax
ABS Deviasi = SUMX ( 'Deviasi All', ABS ( 'Deviasi All'[Value] ) )
```

Detail lengkap di [Fungsi Iterator](./05-iterator-functions.md). Pembagian tugasnya:

| Kebutuhan | Pakai |
|---|---|
| Menjumlahkan isi satu kolom | `SUM ( kolom )` |
| Menjumlahkan hasil expression (`ABS`, `IF`, harga × qty) | `SUMX ( tabel, expression )` |
| Mengagregasi hasil measure per grup | `SUMX ( VALUES ( kolom ), [measure] )` |

## Tabel Referensi Lengkap

| Function | Deskripsi | Contoh |
|---|---|---|
| `SUM` | Menjumlahkan kolom numerik; Blank diabaikan; kolom teks error | `SUM ( 'Deviasi All'[Value] )` |
| `AVERAGE` | Rata-rata kolom numerik; Blank tidak masuk pembagi | `AVERAGE ( 'Sales All'[Penjualan Bersih] )` |
| `MIN` | Nilai terkecil — angka, tanggal, atau **teks** (alfabetis) | `MIN ( 'Date'[Date] )` |
| `MAX` | Nilai terbesar — angka, tanggal, atau teks | `MAX ( 'Waste All'[SATUAN] )` |
| `COUNT` | Menghitung sel **numerik** saja (tanggal termasuk; teks tidak) | `COUNT ( 'Deviasi All'[Value] )` |
| `COUNTA` | Menghitung sel tidak Blank — termasuk teks | `COUNTA ( 'Resto'[PIC] )` |
| `COUNTBLANK` | Menghitung sel Blank (termasuk empty string `""`) | `COUNTBLANK ( 'Deviasi All'[Value] )` |
| `DISTINCTCOUNT` | Jumlah nilai unik (Blank dihitung sebagai 1 nilai unik) | `DISTINCTCOUNT ( 'Resto'[Resto] )` |
| `COUNTROWS` | Jumlah baris tabel — termasuk tabel hasil ekspresi | `COUNTROWS ( 'Deviasi All' )` |
| `SUMX` | Iterator: menjumlahkan expression per baris | `SUMX ( 'Deviasi All', ABS ( 'Deviasi All'[Value] ) )` |
| `AVERAGEX` | Iterator: rata-rata expression per baris | `AVERAGEX ( VALUES ( 'Resto'[Resto] ), [Sales CM] )` |
| `MINX` / `MAXX` | Iterator: min/max expression per baris | `MAXX ( VALUES ( 'Date'[Date] ), [Deviasi CM] )` |
| `COUNTX` | Iterator: menghitung baris yang hasil expression-nya numerik | `COUNTX ( 'Sales All', 'Sales All'[Penjualan Bersih] )` |
| `SUMMARIZE` | Tabel kombinasi unik dari kolom-kolom yang disebut | `SUMMARIZE ( 'Deviasi All', 'Resto'[Resto] )` |
| `COALESCE` | Mengganti BLANK dengan nilai default | `COALESCE ( [Deviasi CM], 0 )` |

## Perilaku Terhadap BLANK

BLANK "menular" ke mana-mana bila tidak dipahami. Perilaku standarnya:

| Fungsi | Perlakuan sel Blank | Konteks kosong (tak ada baris/nilai) |
|---|---|---|
| `SUM` | Diabaikan (dianggap tidak ada) | **BLANK**, bukan 0 |
| `AVERAGE` | Diabaikan — tidak masuk pembagi | BLANK |
| `MIN` / `MAX` | Diabaikan | BLANK |
| `COUNT` | Hanya sel numerik yang dihitung | BLANK |
| `COUNTA` | Blank tidak dihitung | BLANK |
| `COUNTBLANK` | Blank **dan** `""` dihitung | BLANK |
| `DISTINCTCOUNT` | Blank dihitung sebagai 1 nilai unik | BLANK |
| `COUNTROWS` | — (menghitung baris) | BLANK |

Konsekuensi praktis:

- Resto tanpa transaksi bulan ini → `[Deviasi CM]` tampil **kosong**, bukan 0. Kalau bisnis ingin melihat 0: `Deviasi CM (0) = COALESCE ( [Deviasi CM], 0 )`.
- `AVERAGE ( 'Sales All'[Penjualan Bersih] )` adalah rata-rata **hanya resto yang punya baris** — bukan rata-rata seluruh resto. Bila definisi bisnisnya "semua resto", iterasi dimensinya dan beri nilai 0 per baris: `AVERAGEX ( VALUES ( 'Resto'[Resto] ), COALESCE ( [Sales CM], 0 ) )`.

## DISTINCTCOUNT — Menghitung yang Unik

- `COUNT ( 'Deviasi All'[Resto] )` menghitung **baris transaksi** (bisa jutaan) — hampir selalu salah untuk "jumlah resto".
- `DISTINCTCOUNT ( 'Deviasi All'[Resto] )` menghitung **resto berbeda** yang punya transaksi.
- Contoh nyata dari report user — deviasi rata-rata per cabang aktif:

```dax
Deviasi per Resto = DIVIDE ( [Deviasi CM], DISTINCTCOUNT ( 'Deviasi All'[Resto] ) )
```

### SUMMARIZE untuk Kombinasi Unik (Brief)

```dax
-- Berapa kombinasi unik resto × barang di konteks saat ini?
Kombinasi =
COUNTROWS (
    SUMMARIZE ( 'Deviasi All', 'Resto'[Resto], 'Inventory'[Nama Barang] )
)
```

`SUMMARIZE` mengembalikan tabel kombinasi unik dari kolom-kolom yang disebut — fondasi banyak pattern tabel virtual.

## Agregasi Kolom Saja — Bukan Measure/Expression

```dax
SUM ( [Deviasi CM] )                        -- SALAH: argumen bukan kolom
SUM ( 'Deviasi All'[Value] * 2 )            -- SALAH: expression, bukan kolom
SUMX ( 'Deviasi All', 'Deviasi All'[Value] * 2 )   -- BENAR: versi iterator
```

Bila yang ingin dijumlahkan memang isi sebuah measure — cukup panggil measure-nya langsung, atau bungkus dalam iterator bila perlu per grup.

## Sintaks & Contoh

```dax
-- Total deviasi semua Jenis (BOM, COM, WASTE, SUSUT, TRIAL)
Total Deviasi = SUM ( 'Deviasi All'[Value] )

-- Hanya nilai nominal (hasil unpivot: Attribute = NOMINAL / QTY)
Deviasi Nominal =
CALCULATE ( SUM ( 'Deviasi All'[Value] ), Attribute[Attribute] = "NOMINAL" )

-- Berapa cabang yang bertransaksi bulan ini?
Resto Aktif = DISTINCTCOUNT ( 'Deviasi All'[Resto] )

-- Rata-rata penjualan bersih per resto (yang aktif)
Avg Sales per Resto = AVERAGEX ( VALUES ( 'Resto'[Resto] ), [Sales CM] )

-- Deviasi terburuk (paling negatif) di antara resto yang terlihat
Min Deviasi Resto = MINX ( VALUES ( 'Resto'[Resto] ), [Deviasi CM] )

-- Pertumbuhan vs bulan lalu (pattern report user)
Growth = DIVIDE ( [Selisih], ABS ( [Deviasi LM] ) )

-- Kualitas data: berapa baris deviasi tanpa nilai?
Baris Kosong = COUNTBLANK ( 'Deviasi All'[Value] )
```

## Kesalahan Umum

- **`SUM` pada kolom teks** — `SUM ( 'Resto'[Area] )` error: *"The function SUM cannot work with values of type String"*. Untuk teks hanya `MIN`/`MAX`.
- **`COUNT` untuk menghitung baris teks** — `COUNT ( 'Resto'[PIC] )` selalu kosong karena PIC bertipe teks. Gunakan `COUNTA` (atau `COUNTROWS` pada tabelnya).
- **Tertukar `DISTINCTCOUNT` vs `COUNT`** — menghitung transaksi padahal maksudnya jumlah resto/item → angka membengkak. Selalu tanya: "saya menghitung baris atau nilai unik?"
- **`SUM` terhadap measure/expression** — error argumen; pakai `SUMX` atau panggil measure langsung.
- **Mengharapkan 0** — semua agregasi mengembalikan BLANK pada konteks kosong; rapikan tampilan dengan `COALESCE` bila memang butuh 0.
- **`AVERAGE` diam-diam mengecualikan yang kosong** — pembaginya hanya baris yang punya nilai; bila definisi bisnis "semua resto", iterasikan `VALUES ( 'Resto'[Resto] )`.

## Tips Pro

- `COUNTROWS ( 'Deviasi All' )` adalah stetoskop debugging filter context — taruh di visual kecil untuk memverifikasi slicer bekerja sebelum menyalahkan datanya.
- `DISTINCTCOUNT` pada kolom berkardinalitas sangat tinggi (nomor transaksi) mahal — persempit konteksnya dulu (bulan/area) atau hitung lewat dimensi.
- `MIN`/`MAX` teks sangat berguna untuk tooltip/label: `MAX ( 'Waste All'[SATUAN] )` mengambil satu satuan yang valid (pattern measure `[Qty Waste]` report user).
- Kolom angka yang tersimpan sebagai teks (jejak ETL) sebaiknya diperbaiki tipenya di Power Query — jangan membungkus `VALUE()` berulang-ulang di DAX.
- Selalu `DIVIDE`, bukan operator `/` — pembagi nol menghasilkan BLANK dengan aman, bukan error (lihat [Sintaks DAX & Tipe Data](./01-syntax-and-data-types.md)).

## Terkait

- [Fungsi Iterator](./05-iterator-functions.md)
- [Row Context vs Filter Context](./02-row-context-vs-filter-context.md)
- [CALCULATE & Filter Functions](./06-calculate-and-filter-functions.md)
- [Measures vs Calculated Columns](./03-measures-vs-calculated-columns.md)
