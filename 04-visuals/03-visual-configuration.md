# Visual Configuration — Panel Format & Properties

> Setiap visual di Power BI dikonfigurasi lewat panel Visualizations yang terdiri dari tab **Build/Format** (fields wells, visual elements) dan **General/Properties** (size, position, alt text, effects, header icons). Dokumen ini memetakan seluruh opsi penting besingga konfigurasi per tipe visual (bar, line, matrix, card, slicer, axis, analytics pane) dengan peringatan-peringatan klasik seperti "start at zero".

## Struktur Panel Visualizations

- **Tab Build (ikon kuas/format)** — berisi dua bagian:
  - **Fields wells** — drop zone field (Axis, Legend, Values, Tooltips, Small multiples, Drill through).
  - **Format** — pengaturan spesifik visual (X axis, Data labels, Gridlines, dll) + pengaturan container (Title, Background, Border, Shadow, Tooltip card).
- **Tab General (ikon slider)** — Properties: **Size & position** (X, Y, Width, Height dalam pixel kanvas), Style, Effects, **Header icons**, dan **Alt text** (aksesibilitas).
- Banyak section punya toggle **customize** (ikon pensil) untuk memisahkan pengaturan per series/level.

## Bagian General (Properties)

- **Size & position** — isikan angka eksak (X/Y/Width/Height). Di kanvas 1280×720, kerja presisi pixel membuat grid konsisten lintas halaman; jangan mengandalkan drag manual saja.
- **Alt text** — teks yang dibaca screen reader; bisa dinamis via conditional formatting (field value). Wajib untuk report yang di-share luas.
- **Header icons** — kontrol ikon header visual (More options `…`, Focus mode, Pin, dsb). Untuk visual dekoratif/shape, sembunyikan header agar bersih.
- **Effects** — Background (fill + transparency), Border (color, radius), Shadow, **Visual border (glow)**, dan visual **rotation**. Gunakan seperlunya: glow/rotation adalah "chart junk" untuk report operasional.
- **Title** — teks, font, ukuran, warna, alignment, background, divider; bisa diganti menjadi **teks dinamis** via conditional formatting (fx) dengan measure judul.

## Format Umum Semua Visual

| Section | Fungsi | Catatan praktis |
|---|---|---|
| Title | judul visual | format via fx → bisa measure dinamis (`SELECTEDVALUE`) |
| Background | warna/transparansi body visual | samakan dengan theme; transparansi untuk overlay |
| Border & Shadow | garis tepi & bayangan | cukup salah satu, jangan keduanya |
| Visual border (glow) | efek tepi menyala | hindari di report korporat |
| Tooltip card | pengaturan tooltip visual (default vs report page) | lihat [Tooltips](../05-report-pages/04-tooltips.md) |
| Header icons | ikon di pojok visual | matikan untuk card/shape dekoratif |

## Konfigurasi Penting per Tipe Visual

### Bar / Column Chart
- **Data labels:** posisi (outside end/inside end/center), display units (Thousands/Millions), decimal places, orientation. Aktifkan hanya jika nilai tidak bertumpuk.
- **Sort:** menu `…` visual → Sort axis → pilih field/measure → ascending/descending.
- **Y axis — "Start at zero":** default Power BI memulai sumbu di nol. Mematikan opsi ini membuat perbedaan kecil terlihat dramatis — hanya boleh dengan alasan kuat dan label sumbu yang jelas.
- **Concatenate labels** & label density untuk kategori panjang; **Bars gap/inner padding** untuk ketebalan bar.

### Line Chart
- **Markers:** aktifkan saat titik sedikit (< ~15) agar terbaca; matikan untuk data harian padat.
- **Smooth line: OFF** — kurva halus bisa mengaburkan bentuk data sebenarnya.
- **X axis type:** categorical (per kategori) vs continuous (tren waktu). Untuk tren bulanan pakai continuous agar spacing waktu proporsional.

### Matrix
- **Stepped layout** — layout bertingkat mengikuti hierarki (indented rows); matikan bila ingin flat table style.
- **Column widths** — atur lebar per kolom; **+/- expander** (row headers) untuk drill hierarki Area → Resto → Item.
- **Values row subtotals & column subtotals** — kontrol subtotal per level; matikan subtotal yang tidak bermakna (misal subtotal % yang tidak aditif).
- **Heat map** — aktifkan via conditional formatting background (lihat [Conditional Formatting](./04-conditional-formatting.md)).
- **Style presets** — minimal rows/bold headers/compact.

### Card (new) / Card
- **Callout value** — angka utama: font, warna (bisa fx), display units, decimals.
- **Reference labels** (Card new) — target/average/min/max line kecil; **sparkline** bawaan Card (new).
- **Label & category** — teks di bawah angka; matikan bila judul visual sudah menjelaskan.

### Slicer (konfigurasi detail)
- **Style:** Dropdown (hemat tempat, perlu klik dulu), List (langsung terlihat), **Tile** (tombol — bagus untuk sedikit opsi seperti PIC/Area), Slider (range tanggal/angka: Between/Before/After), Relative date.
- **Selection controls:** **Single select** (pilih satu saja), Multi-select with Ctrl, Show "Select all" option.
- **Orientation & responsive** — layout horizontal untuk slicer periode di header.
- Untuk konsistensi antar 36 halaman: gunakan **sync slicers**, bukan copy-paste slicer berulang (report user punya 292 slicer — kandidat penyederhanaan besar).

### Axis Configuration (bar/column/line/combo)
- **Min/Max** nilai sumbu — dipakai untuk memaksa skala konsisten antar halaman (misal Z-Score -3 s/d 3).
- **Display units & precision**, **Log scale** untuk distribusi miring, **Title** sumbu + satuan (Rp vs %), **Gridlines** minimal.
- **Secondary axis** pada combo chart — selalu tampilkan dan beri satuan berbeda agar tidak tertukar.

## Mini Map & Elemen Lain

- Beberapa visual (line/column) mendukung **mini map**? — tidak ada di native; gunakan visual peta terpisah bila perlu konteks lokasi. (Jangan tertukar dengan *small multiples*.)
- **Legend** — posisi (Top/Right/Bottom), bisa dimatikan bila series dijelaskan data labels.

## Analytics Pane (ikon kaca pembesar)

Tersedia untuk bar/column/line/scatter; opsi utama:

- **Constant line** — garis standar/batas, contoh: garis **standar susut 2%** pada chart analisa susut, atau garis 0 pada chart deviasi.
- **Average line / Min / Max / Median line** — pembanding statistik, misal rata-rata deviasi seluruh resto sebagai referensi per resto.
- **Trend line** — linear/exponential pada line/scatter dengan sumbu tanggal (per series, opsi customize).
- **Forecast** — hanya line chart dengan X = tanggal; atur forecast length + confidence (95%/80%), seasonality.
- **Percentile line & Error bars** — untuk analisa sebaran (misal error bar pada rata-rata waste per area).

Semua garis analytics bisa diberi **fx (conditional formatting)** pada warna, dan garis konstanta bisa diisi measure — misal nilai standar susut yang berbeda per kategori.

## Sintaks & Contoh

Contoh konfigurasi yang direkomendasikan diabadikan sebagai checklist:

```text
[Bar chart deviasi per resto]
  Y axis  : start at zero = ON, display units = Thousands, title = "Rp"
  Data labels = ON (outside end), 1 decimal
  Sort    : by [Deviasi CM] DESC
  Analytics: Constant line = 0 (garis pemisah loss/surplus)
  General : alt text dinamis, position snap ke grid 8px

[Line chart Last 13 Months]
  X axis type = Continuous, Markers = OFF (13 titik → ON juga boleh)
  Smooth line = OFF
  Analytics: Average line = rata-rata 13 bulan

[Matrix Detail Deviasi]
  Stepped layout = ON, +/- expander = ON
  Column widths: Item 220px, Values 90px
  Conditional formatting: background heat map by [Deviasi CM]
```

## Kesalahan Umum

- **Mematikan "start at zero" untuk membuat perbedaan terlihat besar** → menyesatkan pembaca; kembalikan ke default atau beri catatan skala.
- **Smooth line ON di chart analitis** → bentuk puncak/lembah data berubah; gunakan smooth hanya untuk presentasi estetik.
- **Menyalin format manual visual demi visual** → pakai theme JSON + copy-paste visual, bukan format satu-satu (539 visual user tidak mungkin diformat manual konsisten).
- **Lupa alt text** → report gagal standar aksesibilitas saat di-share ke manajemen/regulator.
- **Terlalu banyak efek (glow + shadow + border + background berwarna)** → visual noise dan render lambat.
- **Mengatur slicer style berbeda-beda per halaman** → pengguna harus belajar ulang tiap halaman; konsisten lewat theme.

## Tips Pro

- Klik kanan pada nilai di fields well → pilih aggregation yang benar (Sum/Don't summarize/Average) — jangan biarkan default Sum untuk kolom persentase.
- Gunakan **copy-paste visual** antar halaman + **Format painter**-nya tabel/matrix untuk mempercepat konsistensi.
- Simpan konfigurasi standar (jarak antar visual, posisi header) sebagai halaman template kosong — report user punya halaman "Kosongan" untuk ini.
- Untuk puluhan visual per halaman, matikan animasi/transisi visual (Options → Report settings → visual animation) supaya interaksi terasa cepat.
- Alt text dinamis: `Alt Text = "Deviasi per resto, total " & FORMAT([Deviasi CM],"#,0") & ", pilihan: " & SELECTEDVALUE(Resto[Nama],"Semua")`.

## Terkait

- [Katalog Visual Bawaan](./01-native-visuals-catalog.md)
- [Conditional Formatting](./04-conditional-formatting.md)
- [Small Multiples](./05-small-multiples.md)
- [Tooltips](../05-report-pages/04-tooltips.md)
- [Selection & Z-Order](../05-report-pages/05-selection-and-z-order.md)
- [Themes & Templates](../05-report-pages/06-themes-and-templates.md)
