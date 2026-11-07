# Small Multiples — Grid Mini Chart per Kategori

> Small multiples memecah satu chart menjadi grid chart kecil, satu panel per kategori (misal satu sparkline deviasi per resto dalam satu layar). Ini teknik peringkas halaman yang ampuh — pola halaman "satu chart per resto" bisa menjadi satu visual saja, sangat relevan untuk report 36 halaman seperti milik user.

## Konsep & Cara Kerja

- Small multiples mengulang visual yang sama untuk **setiap nilai dari field kategori** yang ditaruh ke field well **Small multiples**, dengan filter konteks otomatis per panel.
- Semua panel berbagi **sumbu yang sama** — itulah yang membuat perbandingan antar panel valid dan cepat.
- Panel yang datanya kosong tetap dirender (bisa dimatikan di pengaturan) menjaga struktur grid tetap terbaca.

## Visual yang Mendukung

Small multiples tersedia pada visual: **Line, Area, Column (clustered/stacked/100% stacked), Bar (varian), Scatter**. Tidak didukung pada: **matrix, table, donut/pie, combo chart, map, dan mayoritas custom visual**.

- Line + small multiples adalah kombinasi paling umum: tren 13 bulan per resto dalam satu visual (alternatif kompak dari halaman *Last 13 Months*).
- Scatter + small multiples: posisi Sales vs % Waste per resto per area.
- Column + small multiples: deviasi bulanan per area.

## Cara Setup

1. Buat visual line/column seperti biasa (X axis = bulan, Y axis = `[Deviasi CM]`).
2. Tarik field kategori (misal `Resto[Nama]`) ke field well **Small multiples**.
3. Grid otomatis terbentuk. Atur layout di **Format → Small multiples**.

## Styling & Pengaturan Grid

- **Grid layout** — `Max small multiples per row` (atau kolom untuk bar horizontal); panel akan mem-wrap otomatis melebihi jumlah itu.
- **Small multiples title** — judul per panel (nama resto), pengaturan font/warna; bisa diganti menampilkan nilai lain via tooltip? Tidak — judul selalu nilai kategori panel; angka ringkasan bisa ditaruh via data labels.
- **Background & border per panel** — pemisah antar panel; **padding** antar panel agar tidak padat.
- **Fill** — mode *fill* merentaskan panel terakhir memenuhi sisa ruang.
- Bila jumlah kategori melebihi kapasitas, muncul navigasi **halaman grid** (panah kiri/kanan) di atas visual — batasi dengan filter Top N bila tidak ingin pagination.

## Kapan Sangat Berguna

- **Membandingkan puluhan resto dalam satu layar** — pengganti "satu chart per resto" atau 5 halaman *Waste by Resto / Growth by Resto / Deviasi by Resto* yang strukturnya identik.
- Menggantikan banyak visual duplikat: 1 visual small multiples menggantikan N visual → halaman lebih ringan dan konsisten secara format.
- Menemukan **anomali bentuk** — satu panel yang polanya beda dari tetangganya langsung terlihat (spike waste di satu resto).
- Dikombinasikan dengan **field parameter** (switch measure): satu grid yang bisa berubah dari Deviasi → Waste → Growth tanpa halaman baru.

## Batasan

- Hanya line/bar/column/area/scatter (lihat daftar di atas) — analisa tabel harus tetap pakai matrix.
- Tidak bisa mengatur format berbeda **per panel** (semua panel mengikuti satu set format).
- Interaksi drillthrough dari panel kecil bisa kurang presisi di panel mungil — pastikan target klik cukup besar.
- Panel sangat banyak (> 50) membuat tiap panel terlalu kecil untuk dibaca; gunakan Top N / slicer terlebih dahulu.
- Beberapa fitur analytics pane dan tooltip kustom berperilaku berbeda di dalam small multiples — uji sebelum desain final.

## Kombinasi dengan Slicer & Filter

- Small multiples mengikuti **seluruh filter halaman & slicer** (periode, area, PIC) — grid akan ter-render ulang saat filter berubah.
- Pasangkan dengan slicer Area (sync slicers antar halaman) supaya grid per resto konsisten konteksnya dengan halaman lain.
- Gunakan **visual-level filter Top N** pada field di Small multiples well (misal 12 resto dengan deviasi terburuk) untuk menjaga grid terbaca.

## Alternatif di Luar Native

- **Trellis / Multipeta custom visual** (mis. *Small Multiples* dari AppSource) untuk tipe visual yang tidak didukung native — ingat batasan custom visual (lihat [Custom Visuals](./02-custom-visuals.md)).
- **Matrix + data bars + conditional formatting** sebagai "small multiples versi tabel": baris = resto, kolom = bulan, warna = nilai — sangat padat informasi dan selalu ter-render cepat.
- Beberapa card + sparkline (Card (new)) meniru "KPI grid per resto".

## Sintaks & Contoh

```text
[Contoh: Tren deviasi 13 bulan per resto]
  Visual      : Line chart
  X axis      : Date[Tanggal]  (Month level, continuous)
  Y axis      : [Deviasi CM]
  Legend      : (kosong — hindari legend di dalam panel kecil)
  Small multiples : Resto[Nama]
  Format:
    Max per row = 6        → 2 baris untuk 12 resto
    Title = ON, 10pt       → nama resto di tiap panel
    Background panel = #F5F5F5, padding 4px
    Markers = OFF, line width tipis (2px)
```

Measure pendukung yang sering dipakai bersama grid:

```dax
-- Highlight panel anomali lewat warna garis (fx pada line color)
Color Line Deviasi =
IF ( [Z Score] > 2, "#D64550", "#118DFF" )
```

## Kesalahan Umum

- **Memasukkan kategori dengan puluhan/ ratusan nilai** (misal `Item`) → panel jadi postang perangko; filter Top N dulu.
- **Menaruh field ke Legend sekaligus Small multiples** → tiap panel penuh legend kecil yang tidak terbaca; pilih salah satu.
- **Mengharapkan format berbeda per panel** (warna garis berbeda manual per resto) → tidak didukung; gunakan conditional formatting berbasis measure sebagai gantinya.
- **Lupa bahwa sumbu dibagi bersama** — perbedaan skala antar resto hilang; bila itu justru yang ingin dilihat, jangan pakai small multiples.
- **Grid terlalu rapat tanpa padding/border** → panel saling menempel dan sulit dipisahkan secara visual.

## Tips Pro

- 12–16 panel adalah sweet spot per layar (4×3 atau 6×2) pada kanvas 1280×720.
- Judul panel + satu angka ringkasan (via data labels pada titik terakhir) membuat grid berfungsi seperti "card + sparkline" massal.
- Untuk presentasi: small multiples + bookmark (ganti field di well via dua bookmark dengan visual berbeda) memberi efek "ganti dimensi" tanpa halaman baru.
- Bandingkan pola grid dengan rata-rata: tarik average line di analytics pane agar panel menyimpang dari normal langsung mencolok.
- Bila report Anda sudah punya banyak halaman per-dimensi (by Area/PIC/Resto/Item ala user), small multiples adalah kandidat konsolidasi halaman — diskusikan dengan end user mana yang benar-benar dipakai.

## Terkait

- [Katalog Visual Bawaan](./01-native-visuals-catalog.md)
- [Visual Configuration (Format Pane)](./03-visual-configuration.md)
- [Visual Interactions](./06-visual-interactions.md)
- [Bookmarks](../05-report-pages/02-bookmarks.md)
- [Page Layout](../05-report-pages/01-page-layout.md)
