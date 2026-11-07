# Dynamic Visuals & Formatting: Report yang Beradaptasi

> Dynamic visuals membuat satu halaman melayani banyak konteks: judul yang ikut filter, format NOMINAL/QTY yang berganti sendiri, warna merah-hijau otomatis, visual yang muncul-tenggelam, dan Top-N yang bisa diatur pengguna. Report Cost Control sudah memakai pola ini (measure `Show`, switching `Trend`, halaman by Area/by PIC) — materi ini merangkum pola-pola tersebut menjadi satu playbook.

## Konsep & Penjelasan

### Dynamic title (SELECTEDVALUE & UNICHAR)
- Judul visual/text box bisa diisi **measure** (Format > Title > fx > Field value).
- Bahan utama: `SELECTEDVALUE()` untuk dimensi terpilih, `FORMAT()` untuk periode, `UNICHAR(10)` untuk pindah baris.
- Contoh hasil: `Deviasi per AREA — Jan 2026` yang berubah sendiri saat slicer diganti.

### Dynamic format strings (NOMINAL vs QTY)
- Report ini punya tabel `Attribute` (NOMINAL/QTY) hasil unpivot — satu visual harus bisa tampil `Rp 12.500.000` (nominal) atau `250,5 kg` (QTY).
- Dua jalur:
  1. **Dynamic format string pada measure** (Desktop: pilih measure > Format > opsi dinamis, tulis ekspresi DAX yang mengembalikan string format).
  2. **Format string expression pada calculation group** (klasik, paling portabel).
- Format string mengubah **tampilan** — nilai mentah (export/CSV) tetap angka asli.

### Dynamic coloring & field value conditional formatting
- Buat measure yang mengembalikan **string warna CSS/hex** (`"#E53935"`), lalu di visual: Conditional formatting > Background color > **Format by: Field value** > pilih measure warna.
- Bisa dipakai untuk background, font, border, data bar. SVG/ikon (mis. panah naik-turun) dapat ditampilkan via custom visual HTML Content dengan `data:` URI SVG — pastikan dari sumber tepercaya.

### Show/hide visual via bookmark + selection pane
- **View > Selection pane**: daftar semua objek halaman; klik ikon mata untuk hide.
- **Bookmark** menyimpan kondisi halaman; centang opsi *Display* agar status hidden ikut tersimpan.
- Pola toggle: dua tombol (Tampilan Grafik / Tampilan Tabel) masing-masing memicu bookmark yang menyembunyikan salah satu visual — untuk data yang sama, dua cara pandang.

### Dynamic display units (jutaan/miliar)
- Display unit (rb/jt/M) di properti visual **tidak bisa diatur lewat measure** — ini keterbatasan.
- Workaround yang jalan: (a) format string dengan pemisah ribuan-triliun (`"#,##0,,,"` → tampil dalam miliar), atau (b) measure teks (`"Rp " & FORMAT([Deviasi CM]/1e6, "#,0") & " jt"`) — konsekuensinya bukan angka lagi (tidak bisa di-sort numerik di table visual).

### Smart narrative
- Visual **Smart narrative** (AI) menghasilkan paragraf ringkasan otomatis (turun/naik, kontributor terbesar) dan ikut berubah saat filter berubah; teks bisa diedit manual.
- Cocok di halaman Summary Deviasi untuk direksi yang hanya membaca.

### Dynamic RLS-driven content
- Konten halaman bisa personal mengikuti user: `USERNAME()`/`USERPRINCIPALNAME()` + `LOOKUPVALUE` untuk judul "Area Anda: Jabodetabek", dikombinasikan dengan RLS di tabel Resto (AM hanya melihat area-nya).
- Bukan pengaman semata — proteksi data tetap dari RLS; ini hanya personalisasi tampilan.

### Page navigator custom
- Pilihan cepat: visual **Page navigator** bawaan (otomatis ikut halaman).
- Kustom penuh: deretan tombol/shape dengan Action = Page navigation, di-highlight lewat bookmark — report ini memakai 33 action button untuk navigasi antar 36 halaman; tambahkan pola ini saat merapikan.

### Dynamic Top N with what-if
- Buat **what-if parameter** (`N`, range 1–20) → slicer angka.
- Measure ranking: `RANKX(ALL(Inventory[...]), [Deviasi CM], , DESC)` dan filter visual `[Rank] <= [N Value]` — Top 10 bisa jadi Top 5 dalam sekali geser, tanpa ganti halaman (report ini punya halaman Top 10 Item Dev yang cocok di-refactor ke pola ini).

### Tooltip page dinamis
- Halaman khusus berukuran kecil (mis. 320×240), aktifkan **Allow use as tooltip**; saat hover di visual lain, tooltip menampilkan mini-report yang kaya (tren 13 bulan item yang di-hover).
- Report ini sudah punya halaman Tooltip — pertimbangkan memperluas ke tooltip per-metric.

### Pattern `[Show]` (dari report ini)
- Measure `[Show]` mengembalikan BLANK bila tidak ada data yang berarti untuk baris tersebut; dipakai di visual-level filter ("Show is not blank") sehingga **baris kosong menyusut** dari visual — matriks tidak penuh baris (blank) item tanpa transaksi.

## Sintaks & Contoh

Paket measure untuk dashboard deviasi adaptif:

```dax
-- 1. Judul dinamis (multi-baris dengan UNICHAR)
Title Dinamis =
"Deviasi per " & SELECTEDVALUE('Dim Parameter'[Dim Parameter], "Resto")
& " — " & FORMAT([Selected Cutoff Date], "mmm yyyy")
& UNICHAR(10) & "Filter: " &
    CONCATENATEX(VALUES(Resto[Area]), Resto[Area], ", ")

-- 2. Format string dinamis (property Format measure — ekspressi):
--    (dipakai bila satu visual melayani NOMINAL & QTY sekaligus)
Fmt Deviasi =
SWITCH(SELECTEDVALUE('Attribute'[Attribute]),
    "NOMINAL", "#,##0",
    "QTY",     "#,##0.0",
    "#,##0"
)

-- 3. Warna dinamis (field value conditional formatting)
Color Deviasi =
SWITCH(TRUE(),
    [Deviasi CM] < 0, "#E53935",   -- merah: over-budget
    [Deviasi CM] > 0, "#43A047",   -- hijau: under
    "#9E9E9E"                      -- netral
)

-- 4. Pattern [Show] — baris tanpa data menyusut
Show =
IF(
    ISBLANK([Deviasi CM]) && ISBLANK([Deviasi LM]),
    BLANK(),
    1
)
-- terapkan di visual-level filter: [Show] is not blank

-- 5. Top N dinamis (what-if parameter "N")
Rank Dev = RANKX(ALL(Inventory[(Old) Nama Barang]), [Deviasi CM], , DESC)
Top Items = CALCULATE(
    [Deviasi CM],
    TOPN([N Value], ALL(Inventory[(Old) Nama Barang]), [Deviasi CM], DESC)
)

-- 6. Judul personal RLS
Title AM =
"Area Anda: " &
LOOKUPVALUE(Resto[Area], Resto[Email AM], USERPRINCIPALNAME())
```

Struktur halaman "Summary Deviasi adaptif":

```text
[Slicer Dim: Resto|Area|PIC|Item]  [Slicer N: 5|10|15]  [Slicer Attribute: NOMINAL|QTY]
┌────────────────────────────┬────────────────────────────────┬────────────────────┐
│ Card: [Deviasi CM]         │ Judul: Title Dinamis           │ Smart narrative    │
│ (fmt: Fmt Deviasi)         │ (fx field value)               │ (ringkasan otomatis)│
├────────────────────────────┼────────────────────────────────┼────────────────────┤
│ Bar chart (warna: Color    │ Matrix rows=Dim, filter        │ Tooltip page:      │
│  Deviasi) axis=Dim Param   │  [Show] not blank              │  tren 13 bulan     │
└────────────────────────────┴────────────────────────────────┴────────────────────┘
Tombol: [Tampilan Grafik] [Tampilan Tabel] → bookmark (display)
```

## Kesalahan Umum
- **Menaruh measure teks ber-format di table lalu mengeluh sort jadi abjad** → teks tidak bisa di-sort numerik; perbaikan: sort pakai kolom/measure angka tersembunyi, atau display unit via format string.
- **Field value warna mengembalikan nama bebas** (`"dark red"`) → kadang gagal render; perbaikan: selalu hex `#RRGGBB`.
- **Bookmark lupa centang opsi Display** → toggle tidak menyembunyikan apa-apa; perbaikan: centang *Display* (dan biasanya matikan *Data* agar filter user tidak ikut ter-reset).
- **Pattern [Show] dipasang di page filter** → semua visual kehilangan baris sekaligus tak terduga; perbaikan: pasang di visual-level filter per visual yang butuh.
- **Dynamic format string dianggap mengubah nilai** → export/analyze tetap angka mentah; perbaikan: sampaikan di dokumentasi bahwa format hanya tampilan.
- **Smart narrative dibiarkan tanpa edit** → kalimat generik membingungkan; perbaikan: edit teks + tambah konteks manual.

## Tips Pro
- Kelompokkan semua measure dinamis di display folder `01 Dynamic` — field list tetap terbaca.
- Konsisten palet warna per rol: merah `#E53935` = over, hijau `#43A047` = saving; simpan di theme JSON report.
- Uji tooltip di ukuran 320×240; kepanjangan = tooltip memotong.
- Untuk direksi: halaman Summary memakai smart narrative + subscription PDF — dibaca 30 detik tanpa buka slicer.
- Setelah menambah toggle/FP, selalu cek mobile layout (lihat [Fitur Service](../09-publish-and-deploy/04-power-bi-service-features.md)).

## Terkait
- [Field Parameters](./02-field-parameters.md)
- [Calculation Groups](./01-calculation-groups.md)
- [DAX Query View](./05-dax-query-view.md)
- [Analisis Halaman & Visual](../11-user-report-analysis/pages-and-visuals.md)
- [Errors Umum](../12-troubleshooting/common-errors.md)
