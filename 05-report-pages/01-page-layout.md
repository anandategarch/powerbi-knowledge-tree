# Page Layout & Canvas Setup

> Setup halaman menentukan pengalaman pertama pengguna: ukuran canvas (16:9 1280×720 seperti report user), grid & alignment, background PNG, pola layout header-slicer-konten, hingga urutan & penamaan halaman. Layout yang disiplin membuat report 36 halaman terasa seperti satu produk, bukan tumpukan worksheet.

## Page Setup — Canvas Settings

Klik kanan area kosong kanvas → **Edit canvas settings** (atau Format → Edit canvas settings):

- **Canvas Settings**
  - **Type:** 16:9 (default, 1280×720), 4:3 (960×720), Letter, atau **Custom** (atur Width/Height bebas).
  - **Fit:** Fit to canvas / Fit to width — mengatur perilaku scaling latar saat rasio layar pemirsa berbeda.
- **Canvas Background** — warna/gradient/**image** untuk area kanvas; opsi **Image fit**: Normal, Fit, Fill; transparency.
- **Canvas Wallpaper** — tampilan area **di luar** kanvas (terlihat saat zoom-out atau layar lebar); biasanya disamakan dengan background agar mulus.

Report user memakai **1280×720 (16:9)** dan **background PNG custom per halaman** — pola yang baik selama PNG dijaga ringan (lihat bagian Performance).

## Grid, Snap & Alignment

- **View → Gridlines** — tampilkan garis bantu; **Snap to grid** membuat objek "magnet" ke grid — kombinasi ini fondasi layout konsisten.
- **Smart guides** — garis merah putus otomatis saat objek bergerak sejajar/berjarak sama dengan objek lain.
- **Lock objects** (View) — kunci semua posisi saat mengedit objek tertentu; cegah geser tak sengaja.
- Alignment presisi antar objek:
  1. Multi-select (Ctrl+klik atau drag dari kosong).
  2. Ribbon **Format → Align** (Left/Center/Right/Top/Middle/Bottom), **Distribute horizontally/vertically** (jarak merata), **Make same size**.
  3. Atau isi X/Y/Width/Height eksak di panel General — di kanvas 1280×720, grid 8px (margin 24px, gutter 16px) terbukti rapi.

## View Options

- **View → Page view:** Fit to page / Fit to width / Actual size — mengatur zoom kerja di Desktop dan tampilan awal di Service.
- **Actual size** menampilkan 1 canvas pixel = 1 screen pixel; 1280×720 penuh di layar 1366×768 ke atas.
- **High contrast** (View) untuk menguji report bagi pengguna dengan penglihatan berkebutuhan khusus.

## Background Image & Theme

- Background PNG per halaman (dipakai user) sebaiknya di-set sebagai **Canvas Background image**, bukan object image yang di-taruh sendiri — object bisa tergeser/terklik, background tidak.
- Alternatif: image object di layer paling bawah + lock — fleksibel untuk halaman dengan panel background berbeda dalam satu kanvas.
- Konsistensi warna & font dikunci lewat **report theme** (lihat [Themes & Templates](./06-themes-and-templates.md)) — PNG background harus selaras dengan palet theme (warna loss/surplus tidak boleh bertabrakan dengan latar gelap).
- Kompres PNG (8-bit palette / tinypng) — file pbix user mencapai 148 MB dan background beresolusi besar adalah salah satu penyumbang.

## Pola Layout Standar (1280×720)

```text
┌──────────────────────────────────────────────┐
│ HEADER (y: 0–96)                             │
│  Logo/judul halaman | KPI Cards ×4–6         │
│  Scrolling text / last update                │
├────────┬─────────────────────────────────────┤
│ SLICER │ KONTEN UTAMA (grid 2×2 / 3×2)       │
│ PANEL  │  Chart besar kiri, matrix kanan,    │
│ (x: 0– │  card strip bawah                   │
│ 240)   │                                     │
├────────┴─────────────────────────────────────┤
│ NAV BAR (y: 680–720): action buttons ◀ ▶ 🏠  │
└──────────────────────────────────────────────┘
```

- **Header KPI cards di atas** — 4–6 card (Total Deviasi, Resto Loss, Resto Surplus, Last Update) + judul halaman; tinggi konsisten antar halaman.
- **Slicer panel kiri** — periode, area, PIC, resto; versi collapsed (dropdown/bookmark toggle) untuk halaman detail.
- **Konten grid** — chart utama mengambil area terbesar; matrix/table di kanan; jangan menumpuk dua chart besar.
- **Navigasi** — action button next/prev/home (user: 33 action button) ditempatkan konsisten di semua halaman.

## Responsive & Auto-fit

- Power BI Service otomatis menskalakan kanvas ke ukuran browser (fit to page/width) — desain berbasis **proporsi** tetap aman di layar berbeda.
- Hindari teks kecil < 10pt: setelah scaling turun, teks makin sulit dibaca di laptop.
- **Mobile layout** (View → Mobile layout) membuat tata letak ponsel terpisah — tarik visual ke kanvas telepon sesuai prioritas; slicer paling atas. Tidak wajib, tapi pertimbangkan untuk KPI ringkasan eksekutif.
- Halaman **Tooltip** (Page information → Allow use as tooltip) otomatis berukuran 320×240 — lihat [Tooltips](./04-tooltips.md).

## Performance Halaman

- **Jangan menumpuk visual visible** — setiap visual = satu query; halaman 48 visual memicu puluhan query per interaksi. Tumpukan visual bertindih (kecuali popup via bookmark) ikut dievaluasi saat visible.
- Gunakan **bookmark show/hide** untuk menyembunyikan visual yang tidak sedang dibutuhkan (pola toggle) — lihat [Bookmarks](./02-bookmarks.md).
- 76 shape dekoratif aman-aman ringan, tapi 292 slicer adalah beban utama report user: kurangi dengan **sync slicers** (satu definisi, tampilan sinkron) & ganti slicer list panjang dengan Text Filter.
- Ukur dengan **Performance Analyzer** (View → Performance pane): perhatikan DAX query duration vs visual render — bedakan masalah model vs masalah layout.
- Background PNG besar memperlambat render awal setiap halaman — satu PNG dipakai banyak halaman hanya di-load sekali, jadi standardisasi lebih baik daripada PNG unik per halaman.

## Organisasi Halaman

- **Urutan tab** mencerminkan alur cerita: Home → Summary → Detail → By-dimensi → Top N → Data mentah → Tooltip (hidden) → admin (hidden).
- **Penamaan halaman** konsisten dan deskriptif ("Summary Deviasi", "Deviasi by Resto") — nama ini muncul di tab & bookmark navigasi.
- **Hidden pages** (klik kanan tab → Hide page) untuk tooltip, template, dan halaman teknis — pengguna tidak tersesat.
- **Tab order** & aksesibilitas keyboard diatur lewat Selection pane — lihat [Selection & Z-Order](./05-selection-and-z-order.md).

## Sintaks & Contoh

Checklist setup halaman standar 1280×720 (duplikasi dari halaman template, bukan dari nol):

```text
Canvas          : 1280 × 720 (16:9), Fit = Fit to canvas
Grid            : Gridlines ON, Snap to grid ON, Lock objects saat edit konten
Background      : PNG terkompresi, Image fit = Fill, identik di semua halaman
Header (y 0–96) : judul halaman + 4–6 KPI card + Scrolling Text (last update)
Slicer panel    : x 16–240, style dropdown/tile, sync slicers lintas halaman
Konten          : mulai y 112, gutter 16px, chart utama ambil 2/3 lebar kanvas
Nav bar         : y 688–720, action button 36×36px, group "Nav Bar" + lock
```

Semua angka di atas adalah konvensi tim — yang penting bukan angkanya, melainkan disiplin memakai angka yang sama di 36 halaman.

## Kesalahan Umum

- **Ukuran halaman campur** (16:9 + 4:3 dalam satu report) → perpindahan halaman "loncat"; standardisasi 1280×720.
- **Background PNG berat full-res foto** → file pbix membengkak dan render lambat; kompres & standardisasi.
- **Mengandalkan drag tanpa snap/align** → margin antar halaman berbeda 3–5 px — terlihat "tidak rapi" saat pindah halaman.
- **Slicer tersebar di semua sisi kanvas** → pengguna tidak tahu mulai dari mana; kumpulkan di satu panel standar.
- **Judul halaman ditulis manual di textbox** tiap halaman dengan posisi/ukuran beda → jadikan elemen header standar yang di-copy konsisten.
- **Menaruh navigasi di posisi berbeda per halaman** → tetapkan satu strip nav dan duplikasi posisinya persis.

## Tips Pro

- Buat **halaman template kosong** (pola halaman "Kosongan" milik user) berisi header, nav, slicer panel standar — mulai halaman baru dari situ.
- Simpan koordinat standar (header y=16, konten y=112, nav y=688) di dokumentasi tim; konsistensi > kreativitas per halaman.
- Untuk review manajemen, siapkan mode "presentation": bookmark yang menyembunyikan slicer panel & menampilkan visual besar.
- Cek halaman di zoom 75% dan 125% — kalau rusak, biasanya posisi fix/teks kecil.
- Kelompokkan objek header/nav jadi **group** di Selection pane agar sulit tergeser saat mengedit konten.

## Terkait

- [Bookmarks](./02-bookmarks.md)
- [Drillthrough](./03-drillthrough.md)
- [Tooltips](./04-tooltips.md)
- [Selection & Z-Order](./05-selection-and-z-order.md)
- [Themes & Templates](./06-themes-and-templates.md)
- [Visual Interactions](../04-visuals/06-visual-interactions.md)
- [Struktur halaman report user](../11-user-report-analysis/pages-and-visuals.md)
