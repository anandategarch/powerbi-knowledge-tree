# Slicer Guide — Tipe, Kontrol Seleksi, dan Sinkronisasi

> Slicer adalah filter yang hidup di canvas: dipilih tipe yang tepat (list, tile, dropdown, slider, relative date, hierarchy), dikontrol mode seleksinya, lalu **disinkronkan antar halaman** — hal wajib bagi report dengan 292 slicer seperti report cost-control 36 halaman ini.

## Konsep & Penjelasan

### Semua tipe slicer dan kapan dipakainya

| Tipe slicer | Field | Tampilan | Kapan dipakai |
|---|---|---|---|
| **Vertical list** | Kategori (text) | Daftar vertikal | Opsi default; nilai sedikit & sering dilihat (`Area`, `PIC`, `Periode`) |
| **Tile** | Kategori | Kotak-kotak horizontal | Ruang sempit, pilihan cepat satu klik (gaya button) |
| **Dropdown** | Kategori | Collapse, 1 baris | **Paling ringan**; wajib untuk nilai banyak (`Resto` puluhan cabang, `Nama Barang`) |
| **Between / slider** | Angka | Slider dua handle | Rentang nilai numerik, mis. filter `Value` di rentang tertentu |
| **Relative date** | Tanggal | Last/This/Next N … | Tren berjalan: "3 bulan terakhir" tanpa setting manual |
| **Date range (Between)** | Tanggal | Kalender dua sisi | Periode bebas bebas-batas; dipakai untuk memilih periode pembanding |
| **Hierarchy slicer** | Tanggal berjenjang | Year → Month, expand/collapse | Navigasi waktu bertingkat tanpa memakan tempat dua slicer |

Aturan penting tipe: dropdown/list/tile hanya untuk field kategorikal; slider untuk numerik; relative date dan date range khusus tanggal.

### Selection controls

Di Format slicer → **Selection controls**:

- **Single select** (default ON): satu klik = satu nilai; klik nilai lain me-replace.
- **Multi-select with Ctrl**: nyalakan jika ingin user harus sengaja menahan Ctrl untuk multi-pilih — mencegah salah klik saat single select dimatikan.
- **Show "Select all"**: menambah opsi Select all di list/dropdown — nyaman untuk "pilih semua kecuali satu".

Kombinasi rekomendasi untuk report operasional: Single select OFF + Show Select all ON → user bisa pilih beberapa Area sekaligus lalu buang satu.

### Styling slicer

- **Slicer header**: judul (rename dari default field name), font size, background — samakan dengan tema report.
- **Items**: ukuran font, padding, background terpilih vs tidak.
- **Expand/collapse icon**: hanya relevan untuk hierarchy slicer — nyalakan agar user tahu bisa drill Year → Month.
- Konsistensi visual antar halaman itu bagian dari UX, bukan kosmetik: user yang pindah halaman harus menemukan slicer di posisi yang sama.

### Sync slicers — WAJIB untuk 292 slicer

Report ini punya 292 slicer di 36 halaman (rata-rata 8 per halaman; *Deviasi by Resto* sampai 13). Tanpa sinkronisasi, user mengganti bulan di halaman 1, pindah halaman, dan **pilihan tidak ikut** — pengalaman paling menyebalkan di report multi-halaman.

Cara pakai:

1. View → **Sync slicers**. Muncul pane berisi daftar slicer (per field/group) dan grid halaman.
2. Tiap halaman punya dua tanda: **Sync** (nilai terseleksi disamakan) dan **Visible** (slicer juga ditampilkan di halaman itu).
3. Centang Sync+Visible ke seluruh 36 halaman untuk grup slicer inti: `Date[Year]`, `Date[Month]`, `Periode`, `Resto[Area]`, `Resto[PIC]`.

**Pitfall "slicer sync beda nama/grup":**

- Dua slicer dengan field sama yang dibuat terpisah (copy halaman, insert slicer baru) **tidak otomatis sinkron** — mereka grup berbeda. Sinkronisasi harus lewat pane Sync slicers, bukan dibiarkan.
- Di satu halaman, beberapa slicer field sama otomatis jadi satu grup; nilai di satu akan menyamakan yang lain. Jika memang butuh dua perilaku berbeda untuk field sama (mis. satu filter visual, satu untuk pembanding), pisahkan grupnya secara eksplisit.
- Mengganti slicer lama dengan slicer baru tanpa update pane sync = halaman lain diam-diam tidak lagi sinkron. Audit pane Sync slicers setelah redesign.

### Slicer dengan hierarchy: Year → Month → Periode

Pola waktu report cost-control unik: `Date[Year] → Date[Month]` lalu **`Periode` (WEEK 1 / WEEK 2 / WEEK 3 / EOM)** sebagai tabel terpisah (dipakai measure `Selected Cutoff Date` lewat `SELECTEDVALUE`).

Karena hierarchy harus berada dalam **satu tabel**, ada dua opsi:

1. **Dua slicer bersebelahan** (pola yang sekarang dipakai user): slicer hierarchy `Date` (Year→Month) + slicer list `Periode`. Sederhana, dan `Periode` tetap tabel independent untuk YTD cutoff.
2. **Satu hierarchy penuh** di tabel `Date` (tambahkan kolom Periode ke `Date` via relasi/lookup) → satu slicer expand/collapse. Lebih rapi, tapi kolom Periode di `Date` duplikasi nilai per tanggal — ukur dulu dampaknya ke model (lihat [VertiPaq](../07-performance/01-vertipaq-engine.md)).

### Performance slicer

Setiap slicer mengeluarkan **query sendiri** saat halaman dirender (untuk mengisi daftar nilai). Implikasinya:

- 11 slicer di *Summary Deviasi* = 11 query + query visual — sebelum DAX-nya pun diubah, halaman sudah sibuk.
- **Dropdown lebih ringan dari list/tile** untuk Visual Display (client render); jumlah query tetap sama, tapi tampilannya jauh lebih murah.
- Hindari slicer pada kolom **distinct count besar** — `Inventory[Nama Barang]` ribuan item di list adalah pemboroskan; pakai dropdown, atau custom visual **Text Filter** (report ini sudah memakai 2 buah) untuk pencarian teks.
- Single select mengurangi ruang kombinasi filter user (dan risiko "semua terpilih semua").
- 292 slicer banyak di antaranya duplikasi lintas halaman — dengan sync, satu definisi menggantikan banyak; halaman berat seperti *Summary Deviasi* (48 visual) paling diuntungkan.

### Harmonisasi: grup konsisten + bookmark clear filter

- Susun **slicer strip** di posisi identik di semua halaman (mis. baris atas) — sync slicers menjaga nilai, layout manual menjaga posisi.
- Tiap halaman kasih tombol **Reset** (report user sudah punya action button Reset di hampir semua halaman): tombol ini harus memanggil **bookmark "Clear filter"** yang merekam kondisi slicer bersih — bukan sekadar navigasi.
- Jangan campur filter pane dan slicer untuk hal yang sama di halaman berbeda; pilih satu "sumber kebenaran" per field.

## Sintaks & Contoh

Konfigurasi standar slicer `Periode` (contoh yang dipakai report):

```text
Field      : Periode[Periode]          (WEEK 1, WEEK 2, WEEK 3, EoM)
Style      : Vertical list
Selection  : Single select ON, Show "Select all" OFF
Styling    : Header "Pilih Periode", item 10pt, background tema
Sync       : Sync ON + Visible ON di seluruh 36 halaman
```

Hierarchy slicer tanggal:

```text
Fields     : Date[Year], Date[Month]   (user-defined hierarchy "Yr-Mth")
Style      : Dropdown (hemat ruang) atau Tile
Controls   : Expand/collapse ON ("Expand down one level" icon)
```

Measure yang membaca pilihan slicer (dipakai di `Selected Cutoff Date`):

```dax
Selected Periode =
SELECTEDVALUE ( 'Periode'[Periode] )   -- BLANK jika multi-select
```

Pola bookmark Reset per halaman:

```text
1. Bersihkan semua slicer/filter di halaman (Ctrl+klik Select all OFF).
2. Bookmark → Add → nama "Reset - <halaman>" → centang Data.
3. Action button Reset → Action → Bookmark → pilih "Reset - <halaman>".
```

## Kesalahan Umum

- **Default single select membingungkan** — user pikir bisa pilih 3 bulan sekaligus, ternyata ter-replace. Perbaikan: matikan Single select, atau nyalakan Multi-select with Ctrl sesuai kebutuhan.
- **Sync slicers dicentang tapi kolom Visible lupa** — nilainya sinkron, tapi slicer tidak muncul di halaman tersebut (user bingung kenapa halaman itu berbeda).
- **Slicer di-copy antar halaman tanpa cek pane sync** — dua grup berbeda untuk field sama; pilihan tidak konsisten. Audit lewat pane Sync slicers.
- **Hierarchy slicer tanpa icon expand/collapse** — user tidak tahu bisa drill Year → Month, hanya klik level teratas.
- **Slicer memakai hidden auto date/time table** — datetime bawaan menumbuhkan kolom-kolom tersembunyi dan membengkakkan model; gunakan tabel `Date` sendiri + Mark as date table.
- **List vertikal untuk `Nama Barang` (ribuan item)** — render lambat dan scroll panjang; ganti dropdown + Text Filter.
- **Reset button hanya navigasi, bukan bookmark clear filter** — klik "reset" tidak mengembalikan pilihan; pastikan bookmark merekam kondisi bersih.

## Tips Pro

- Untuk 36 halaman, kelola slicer sebagai **7 grup inti** (Year, Month, Periode, Area, PIC, Resto, Attribute/Trend) — sisanya turunkan jadi filter pane. Target: turunkan 292 slicer ke ±100 tanpa mengurangi kemampuan analisa.
- Gunakan **Selection pane** (View → Selection) untuk meng-group dan menyembunyikan slicer saat desain, supaya canvas tidak penuh.
- Slicer tanggal dengan `Date range` cocok untuk halaman pembanding bebas (pola `Date A` untuk periode A/B di halaman *Antar Periode*) — lihat [Dynamic Filtering](./03-dynamic-filtering.md).
- Uji performa halaman terberat setelah menambah/mengurangi slicer dengan Performance Analyzer — lihat [Performance Analyzer](../07-performance/02-performance-analyzer.md).
- Jika slicer harus banyak, taruh di panel collapse (button show/hide dengan bookmark) — halaman tetap bersih, slicer tetap ada.

## Terkait

- [Filter Types](./01-filter-types.md) — pasangan slicer di filters pane
- [Dynamic Filtering](./03-dynamic-filtering.md) — TREATAS, what-if, field parameters
- [VertiPaq Engine](../07-performance/01-vertipaq-engine.md) — biaya hidden auto date/time
- [Optimization Checklist](../07-performance/03-optimization-checklist.md) — mengurangi visual & slicer per halaman
- [Pages & Visuals](../11-user-report-analysis/pages-and-visuals.md) — distribusi 292 slicer per halaman
