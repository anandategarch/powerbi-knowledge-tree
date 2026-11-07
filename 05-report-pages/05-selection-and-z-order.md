# Selection Pane, Z-Order & Tab Order

> Selection pane adalah "command center" objek di halaman: melihat semua objek (termasuk yang tersembunyi), mengatur urutan layer (z-order), hide/show, lock, group, multi-select untuk bulk formatting, hingga tab order untuk aksesibilitas keyboard. Pada halaman dengan puluhan objek (background PNG + slicer + shape + visual) seperti report user, pane ini wajib dikuasai.

## Membuka Selection Pane

- Ribbon **View → Selection pane** (muncul di sisi kanan, sering berdampingan dengan Bookmarks/Visualizations).
- Daftar objek ditampilkan **dari atas = layer terdepan** ke bawah = layer paling belakang — sama seperti panel Layers di Photoshop/Figma.

## Anatomi Panel

Per baris objek tersedia:

- **Ikon mata** — hide/show objek di canvas (tidak menghapus).
- **Ikon gembok** — lock: objek tidak bisa diklik/digeser di canvas (bisa diatur hanya lewat pane) — melindungi background & elemen fix.
- **Nama objek** — double-click untuk rename; penamaan jelas ("BG Header", "Slicer Periode", "Card Total Deviasi") memudahkan hidup.
- **Drag & drop** untuk mengubah urutan layer (z-order).
- **Ikon group** untuk objek yang dikelompokkan (bisa di-expand).

## Z-Order — Bring Forward / Send Backward

- Urutan tumpukan menentukan objek mana yang terlihat/menerima klik saat bertindih.
- Cara cepat: klik kanan objek → **Order** → Bring to front / Bring forward / Send backward / Send to back.
- Layering pattern untuk halaman report bertema (dipakai user):

```text
Layer paling belakang → terdepan:
  1. Background PNG (lock, atau set sebagai Canvas Background)
  2. Shape dekoratif: header bar, panel, garis
  3. Slicer panel (di atas shape-nya)
  4. Visual konten (chart, matrix, card)
  5. Tombol navigasi (action button)
  6. Popup layer (group tersembunyi, muncul via bookmark)
```

- Background sebagai **Canvas Background** (bukan image object) lebih aman — tidak bisa terklik/tergeser; image object fleksibel untuk komposisi parsial.

## Group Objects

- Multi-select beberapa objek → klik kanan → **Group** (atau Ctrl+G).
- Group bisa di-hide/show, dikunci, digeser bersama — ideal untuk header (logo + judul + garis), strip navigasi (33 action button user jadi satu group!), dan popup panel.
- Di pane, group bisa di-expand untuk mengedit anggotanya; rename group ("Nav Bar", "Header KPI").

## Multi-Select & Bulk Formatting

- **Ctrl+klik** objek di canvas, atau **Ctrl/Shift+klik** nama di Selection pane.
- Drag dari area kosong canvas (marquee select) untuk memilih area.
- Setelah multi-select objek sejenis: ubah Format sekali → berubah untuk semua terpilih (misal semua card samakan font/callout, semua slicer samakan background).
- Ini cara tercepat menyeragamkan puluhan visual — jauh lebih baik daripada format satu per satu, dan mengurangi risiko inkonsistensi pada 539 visual.

## Tab Order (Aksesibilitas)

- Di Selection pane ada toggle **"Tab order"** — berganti mode pengurutan navigasi keyboard.
- Susun urutan sesuai alur membaca: judul → slicer periode → slicer area → KPI → chart → tombol.
- Tiap objek bisa ditandai **interactive** (fokus keyboard, biasanya tombol/slicer) atau **non-interactive/dekoratif** (shape, background — dilewati keyboard tapi tetap ada).
- Hidden object otomatis keluar dari tab order.
- Mengapa penting: pengguna keyboard/screen reader menavigasi report dengan Tab — urutan acak membuat report "tidak terbaca" bagi mereka, dan audiens eksekutif dengan impairment visual adalah pengguna nyata report korporat.

## Alt Text

- Setiap visual penting diberi **General → Alt text** — deskripsi yang dibacakan screen reader.
- Alt text bisa **dinamis** (fx → field value measure) sehingga deskripsi mengikuti filter aktif:

```dax
Alt Chart Deviasi =
    "Bar chart deviasi per resto, total "
    & FORMAT ( [Deviasi CM], "#,0" )
    & ", filter: " & SELECTEDVALUE ( Area[Nama], "Semua Area" )
```

- Shape dekoratif: kosongkan alt text (atau tandai dekoratif) agar tidak "dibacakan" sia-sia.

## Hidden Objects & Dampaknya

- Objek yang di-hide lewat mata di Selection pane **tidak dirender saat tersembunyi**, tetapi tetap menjadi bagian halaman — muncul kembali via bookmark (Display) atau toggle manual.
- Catatan penting: **objek visible yang bertumpuk semuanya tetap dievaluasi** — menumpuk 10 visual saling menimpa dalam keadaan visible membebani query; pola yang benar adalah hide/show via bookmark.
- Ada nuansa performa yang sering diperdebatkan terkait apakah objek tersembunyi tetap dievaluasi di semua kondisi — jadikan Performance Analyzer alat bukti: rekam, bandingkan waktu render saat objek di-hide vs dihapus.
- Hidden **page** berbeda dari hidden object: halaman tersembunyi tidak dimuat sampai diakses (tooltip, drillthrough, template).

## Popup Layer Pattern

1. Buat group "Popup Info" di layer teratas (rectangle semi-transparan + textbox + tombol ✕).
2. Hide group → buat bookmark `Popup OFF` (Display ON, Data OFF).
3. Show group → bookmark `Popup ON`.
4. Tombol ℹ → Action → Bookmark `Popup ON`; tombol ✕ → `Popup OFF`.
- Lihat [Bookmarks](./02-bookmarks.md) untuk pola lengkap.

## Sintaks & Contoh

Konvensi penamaan & urutan layer yang bisa langsung dipakai tim:

```text
HDR_  : header (judul, logo, garis)          → layer 2, locked
NAV_  : action button navigasi               → layer 3, group + locked
SLC_  : slicer (periode, area, PIC, resto)   → layer 4
VIZ_  : visual konten (chart, matrix, card)  → layer 5
POP_  : popup layer (info/help)              → layer 6, hidden default

Workflow per halaman:
1. Insert objek → rename langsung dengan prefix.
2. Group per zona (HDR/NAV/SLC/VIZ/POP).
3. Atur z-order: drag di Selection pane (paling atas = paling depan).
4. Lock zona statis (HDR, NAV).
5. Set Tab order: judul → SLC_ → VIZ_ → NAV_ (tombol setelah konten).
```

## Kesalahan Umum

- **Objek tanpa nama** ("TextBox 7", "Image 3") → Selection pane jadi teka-teki; rename segera setelah insert.
- **Background image object yang tidak dikunci** → sekali drag tak sengaja menggeser seluruh tampilan; lock atau pindahkan ke Canvas Background.
- **Popup dibuat dengan menumpuk visual visible** alih-alih hide/show → beban query dan risiko klik menembus layer.
- **Tab order diabaikan** → urutan keyboard kacau; audiens aksesibilitas terganggu.
- **Group berukuran raksasa** (seluruh halaman satu group) → sulit mengedit bagian dalam; pecah per zona (Header, Nav, Konten).
- **Banyak objek duplikat tersembunyi** sisa iterasi desain → file membengkak dan membingungkan; bersihkan berkala.

## Tips Pro

- Standarkan nama objek dengan prefix zona: `HDR_`, `NAV_`, `SLC_`, `VIZ_`, `POP_` — Selection pane langsung terbaca seperti daftar isi halaman.
- Kunci (lock) semua elemen statis: background, header shape, logo — canvas jadi aman untuk editing visual.
- Sebelum finalisasi halaman, buka Selection pane dan lakukan "audit hidden": setiap objek tersembungi harus punya alasan (popup/bookmark) — sisanya hapus.
- Gunakan multi-select + bulk format sebelum copy visual antar halaman agar konsistensi terjaga dengan usaha minimal.
- Saat desain banyak halaman sejenis, buat halaman template dengan layer & group siap pakai (pola halaman "Kosongan" user).

## Terkait

- [Bookmarks](./02-bookmarks.md)
- [Page Layout](./01-page-layout.md)
- [Visual Configuration (Format Pane)](../04-visuals/03-visual-configuration.md)
- [Conditional Formatting](../04-visuals/04-conditional-formatting.md)
- [Themes & Templates](./06-themes-and-templates.md)
