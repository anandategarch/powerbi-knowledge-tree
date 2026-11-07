# Field Parameters: Slicer yang Mengganti Kolom & Measure

> Field parameter memungkinkan satu slicer mengganti kolom/measure yang dipakai visual — misalnya satu halaman "Deviasi by Dimensi" yang bisa berubah dari *by Resto* → *by Area* → *by PIC* → *by Item* tanpa menduplikasi halaman. Ini pola yang bisa memangkas halaman `Deviasi by Area / by PIC / by Resto / by Item` di report Cost Control menjadi satu halaman adaptif.

## Konsep & Penjelasan

### Apa itu field parameter
- Tabel kecil yang berisi **referensi field** (kolom atau measure) + label tampilan + urutan.
- Ketika kolom parameter dipakai di slicer, **field well visual** (Axis/Rows/Legend/Values) ikut berganti sesuai pilihan — visual engine yang menggantinya, bukan trik filter.
- Bisa berisi campuran: kolom dimensi (`Resto[Area]`) sekaligus measure (`[Deviasi CM]`) dalam satu parameter.

### Cara buat
- Desktop: **Modeling > New parameter > Field parameters** (di ribbon Model), pilih field, atur urutan, OK.
- Hasilnya: satu tabel calculated + slicer otomatis terpasang di halaman aktif.

### Dibanding measure switching lama (ala `Trend[Multi Value CM]` report ini)
Report ini sudah lama memakai pola *measure switching* dengan tabel `Trend` (DEVIASI/BOM/%BOM/%SALES/VARIANCE) dan measure `Multi Value CM` yang membaca `VALUES(Trend[Kategori])` (dibungkus `TREATAS`/`SWITCH`).

| Aspek | Field parameter | Measure switching lama |
|---|---|---|
| Yang diganti | Field itu sendiri (kolom/measure) di field well | Hanya nilai measure (satu rumus besar SWITCH) |
| Bisa ganti **dimensi/axis** | ✔ (Resto/Area/PIC/Item) | ✗ (axis tetap) |
| Bisa ganti **measure** | ✔ | ✔ |
| Logika berbeda per pilihan | ✗ (hanya memilih objek) | ✔ (tiap cabang SWITCH boleh beda rumus) |
| Format mengikuti sumber | ✔ (format measure asli terbawa) | Harus disatukan manual |
| Ukuran di model | 1 tabel kecil per parameter | Measure switching makin panjang tiap opsi |
| Klien non-Power BI (Excel MDX) | Binding tidak berlaku | Tetap jalan (hanya DAX biasa) |

Kesimpulan praktis: ganti *dimensi* → field parameter; butuh *logika khusus per pilihan* (mis. % BOM vs % Sales punya rumus sangat beda dan saling bergantung) → measure switching atau [Calculation Groups](./01-calculation-groups.md).

### Penggunaan di slicer & bookmark
- Slicer: single-select (recommended) atau multi-select; pilihan tersimpan dalam **bookmark** — pasangan yang pas untuk tombol preset (mis. tombol "Mode Area" menyimpan pilihan + filter lain).
- Field parameter juga bisa di-sync slicers antar halaman seperti slicer biasa.
- Pilihan default saat pertama buka halaman: pastikan ada satu nilai terpilih (set lewat slicer default/bookmark awal), karena visual tanpa field terpilih menampilkan blank/error.

### Dynamic axis & measure in one slicer
- Satu parameter boleh berisi kolom dan measure sekaligus → slicer yang sama bisa menggeser visual dari "Deviasi per Area (jumlah Rp)" ke "Top item (measure ranking)".
- Kombinasi lanjutan: parameter dimensi (`Resto/Area/PIC/Item`) × [Calculation Group](./01-calculation-groups.md) periode (`CM/LM/YTD/YoY`) → satu halaman melayani 4×4 kombinasi yang sebelumnya butuh belasan halaman.

## Sintaks & Contoh

DAX yang di-generate Power BI saat membuat field parameter (list literal + `NAMEOF`):

```dax
Dim Parameter = {
    ("Resto", NAMEOF('Resto'[Resto]), 0),
    ("Area",  NAMEOF('Resto'[Area]),  1),
    ("PIC",   NAMEOF('Resto'[PIC]),   2),
    ("Item",  NAMEOF('Inventory'[(Old) Nama Barang]), 3)
}
```

Yang terjadi di balik layar:
- Tiap baris = (label tampilan, referensi field, angka urutan).
- UI menamai kolom: kolom utama (dipakai di slicer), `… Order` (Sort by), `… Value` (referensi asli; biasanya disembunyikan).
- **Penting**: set kolom `… Order` sebagai *Sort by column* agar urutan Resto→Area→PIC→Item konsisten.

Varian lama berbasis `DATATABLE` (persis gaya tabel `Trend`/`Attribute` di report ini) — untuk switching berbasis teks tanpa referensi field:

```dax
Metric = DATATABLE(
    "Kategori", STRING,
    {
        {"DEVIASI"},
        {"BOM"},
        {"% BOM"},
        {"% SALES"},
        {"VARIANCE"}
    }
)
-- lalu measure switching:
Multi Value CM =
SWITCH(SELECTEDVALUE(Metric[Kategori]),
    "DEVIASI", [Deviasi CM],
    "BOM",     [BOM CM],
    "% BOM",   DIVIDE([BOM CM], [Sales CM]),
    "% SALES", DIVIDE([BOM CM], [Sales CM]) * [Deviasi CM] / [Deviasi LM],
    [Deviasi CM]
)
```

Layout halaman gabungan (menggantikan 4 halaman by Area/by PIC/by Resto/by Item):

```text
[ Slicer Dim: Resto|Area|PIC|Item ]   [ Slicer Periode: CM|LM|YTD|YoY ]  (calc group)
┌───────────────────────────────┬────────────────────────────────────┐
│ Judul dinamis: "Deviasi per   │ Matrix: Rows = Dim Parameter       │
│ AREA — Jan 2026"             │        Values = [Deviasi]           │
├───────────────────────────────┼────────────────────────────────────┤
│ Bar chart (axis = Dim Param) │ Donut kontribusi (legend = Dim)    │
└───────────────────────────────┴────────────────────────────────────┘
```

## Kesalahan Umum
- **Lupa set Sort by kolom Order** → urutan pilihan acak (Area, Resto, Item, PIC…); perbaikan: Sort by column = `… Order`.
- **Menaruh kolom Value (referensi mentah) di slicer** → slicer tampil aneh/duplikat; perbaikan: gunakan kolom utama, sembunyikan Value.
- **Tidak ada pilihan default** → halaman terbuka blank; perbaikan: set default di slicer / bookmark awal.
- **Mengganti nama kolom sumber setelah FP dibuat** (`Resto[Area]` → `Region`) → DAX `NAMEOF` rusak; perbaikan: buat ulang parameter (atau edit barisnya), jangan rename asal-asalan.
- **Memakai FP untuk logika yang butuh rumus berbeda per pilihan** → tidak bisa; perbaikan: measure switching / calculation group untuk kasus itu.
- **FP + measure switching pada field well yang sama** → tumpang tindih membingungkan; perbaikan: satu mekanisme per visual.

## Tips Pro
- Label tampilan boleh berbeda dari nama teknis: tampilkan "Item" untuk `Inventory[(Old) Nama Barang]` — lebih ramah manajemen, sumber tetap teknis.
- Kombinasi FP + bookmark preset ("Mode Ringkas" / "Mode Detail") memberi kesan report mahal tanpa menulis ulang halaman.
- Untuk report 292 slicer: konsolidasi dimensi ke FP akan memangkas jumlah slicer sekaligus menyederhanakan sync slicers.
- Simpan parameter di tabel bernama jelas (`Dim Parameter`, `Metric Parameter`) dan dokumentasikan di repo.
- Uji FP di mobile layout — slicer panjang sering perlu dirombak di ponsel.

## Terkait
- [Calculation Groups](./01-calculation-groups.md)
- [Dynamic Visuals & Formatting](./04-dynamic-visuals-and-formatting.md)
- [DAX Query View](./05-dax-query-view.md)
- [Analisis Halaman & Visual](../11-user-report-analysis/pages-and-visuals.md)
- [FAQ](../12-troubleshooting/faq.md)
