# Custom Visuals Power BI (AppSource & Organizational)

> Custom visual adalah visual tambahan berformat `.pbiviz` yang di-install dari Microsoft AppSource, organizational store, atau file lokal — memperluas kemampuan native Power BI. Dokumen ini membahas cara import, katalog visual populer, klasifikasi certified vs non-certified, serta risiko performance & security — termasuk dua custom visual yang benar-benar dipakai di report user: **Text Filter** dan **Scrolling Text**.

## Konsep Dasar

- Custom visual adalah paket JavaScript yang berjalan dalam sandbox iframe di dalam report. File-nya berekstensi `.pbiviz`.
- Sumber distribusi ada tiga:
  1. **AppSource / "Import visual from AppSource"** — marketplace publik ratusan visual.
  2. **Organizational store** — visual yang di-upload admin tenant, hanya bisa dipakai organisasi (korporat bisa mengontrol versi & keamanan).
  3. **File lokal** — "Import a visual from a file" untuk `.pbiviz` dari developer/in-house.
- Setelah di-import, visual muncul di panel Visualizations dengan ikon khusus dan bisa dipakai seperti visual native (ada fields wells, format pane, mendukung interaksi & filter).
- Internal name di file `.pbix` adalah nama teknis + GUID, contoh di report user:
  - `textFilter25A4896A83E0487089E2B90C9AE57C8A` (×2) → visual **Text Filter**.
  - `ScrollingTextVisual1448795304508` (×1) → visual **Scrolling Text** (ticker).

## Cara Import Custom Visual

1. Buka report di Power BI Desktop → panel **Visualizations** → klik ikon `…` (More options).
2. Pilih **Get more visuals** (atau *Import a visual from AppSource*).
3. Cari nama visual → **Add**. Visual langsung tersedia di panel Visualizations.
4. Alternatif: `…` → **Import a visual from a file** → pilih file `.pbiviz` dari disk (untuk visual internal/non-publik).
5. Setelah di-upload ke Power BI Service, custom visual otomatis ter-bundle dalam report — viewer tidak perlu install apa pun, cukup izin browser menjalankannya.

## Katalog Visual Populer & Kegunaannya

| Visual | Fungsi | Relevansi untuk cost-control F&B |
|---|---|---|
| **Text Filter** | search box yang memfilter kolom teks (contains), ramah keyboard | cari nama item/resto di halaman Detail — dipakai user ×2; alternatif slicer dengan search |
| **Scrolling Text / Ticker** | teks berjalan (marquee) | banner informasi periode/last update di halaman Home — dipakai user ×1 |
| HierarchySlicer | slicer dengan struktur hierarki (Area → Resto) | navigasi dimensi bertingkat tanpa banyak slicer flat |
| Chiclet Slicer | tombol/tile pilihan bergaya | pilihan PIC/periode yang ringkas dan touch-friendly |
| Photon (ZoomCharts) | drill-down chart interaktif | telusuri deviasi dari area ke resto dengan klik |
| Envelope | drill-down visual multi-level | alternatif explorasi hierarki |
| Synoptic Panel (SQLBI) | gambar custom (denah lantai) dengan area ter-data | denah resto/outlet: area mana yang waste-nya tinggi |
| Bullet Chart | actual vs target vs range | susut aktual vs standar susut 2% |
| Gantt | timeline & durasi | aging PO belum PI |
| Dumbbell Chart | jarak dua nilai per kategori | gap `Deviasi CM` vs `Deviasi LM` per resto |
| Arrow Slider | slider nilai dengan indikator arah | navigasi periode/parameter numerik |
| Heatmap | grid intensitas nilai | kalender/matriks deviasi per resto per hari |
| Word Cloud | frekuensi kata | daftar item bermasalah (dari kolom keterangan) |
| Info Slider | slider dengan deskripsi | what-if parameter (misal threshold anomali) |
| Play Axis (Dynamic) | animasi scatter berbasis waktu | pergerakan posisi resto antar bulan |
| Decomposition Tree (custom) | versi custom sebelum jadi native | kini sudah ada versi native (AI visual) |
| Zebra BI Tables | tabel IBCS dengan variance inline | laporan manajemen dengan variance % otomatis |
| Power KPI | multi-KPI card + sparkline + detail | scorecard per resto |

## Certified vs Non-Certified

- **Certified power BI visuals** punya badge "Certified by Power BI" di AppSource. Sertifikasi berarti visual sudah lulus rangkaian tes Microsoft: kode aman, **tidak melakukan akses jaringan eksternal** (data tidak dikirim keluar), dan lolos uji performance.
- **Non-certified** belum tentu buruk — hanya belum di-audit. Risiko utama: potensi ekfilstrasi data (script bisa memanggil endpoint luar) dan performa tidak terjamin.
- Admin tenant dapat mengatur di Admin portal → Tenant settings → **"Visual from AppSource"** dan **"Certified visuals"** — banyak korporat memblokir non-certified, sehingga visual non-certified gagal render di service.
- Visual dari organizational store adalah jalur kompromi: admin yang menguji dan meng-upload versi terkunci.

## Performance & Security

- Setiap jenis custom visual menambah unduhan bundle JS saat halaman dibuka — **banyak visual unik = load lambat**. Pakai maksimal 2–4 jenis custom visual per report.
- Custom visual berat (canvas drawing, banyak data point) memperbesar waktu render; uji dengan **Performance Analyzer** (View → Performance pane → Start recording).
- Non-certified visual dapat mengirim data ke pihak ketiga — untuk data cost/supplier yang sensitif, pilih certified atau org store.
- Custom visual **tidak selalu** mendukung semua fitur native: sebagian tidak mendukung drillthrough, bookmark, sync slicers, atau export data — cek sebelum mendesain alur analisa.

## Menilai Custom Visual Sebelum Dipakai

- Cek badge **certified**, publisher terpercaya, dan tanggal update terakhir (visual yang tidak di-maintain bisa rusak saat engine update).
- Baca kebutuhan lisensi — sebagian visual "gratis" di AppSource adalah freemium dengan watermark.
- Uji volume data nyata: visual yang mulus dengan 100 baris bisa crash dengan 50.000 baris detail item.
- Uji interaksi: cross-highlight, drillthrough, bookmark, dan perilaku saat slicer disinkronkan antar halaman.

## Sintaks & Contoh

Pola pemakaian dua visual di report user:

```text
Text Filter (×2 di report user)
  Fields: Item[Nama Item]  (atau Resto[Nama])
  Perilaku: mengetik "ayam" memfilter SEMUA visual halaman ke item yang mengandung kata "ayam"
  Nilai tambah: aksesibilitas — bisa dipakai full keyboard, lebih cepat dari slicer list panjang

Scrolling Text (×1 di report user)
  Fields/Properties: teks statis atau measure DAX, kecepatan & arah scroll
  Contoh isi: "Periode analisa: " & SELECTEDVALUE(Periode[Nama]) &
              " | Last update dev: " & [Last Update Dev]
  Nilai tambah: menyampaikan konteks periode tanpa makan tempat di header halaman
```

Perhatikan Text Filter memfilter **seluruh halaman** — pastikan interaksinya diatur (`Format → Edit interactions`) bila tidak semua visual boleh ikut terfilter.

## Kesalahan Umum

- **Menumpuk banyak jenis custom visual karena "keren"** → setiap jenis menambah beban load; pilih yang benar-benar menyelesaikan masalah (seperti Text Filter untuk pencarian).
- **Tidak mengecek status certified di korporat** → report gagal tampil di service saat admin memblokir non-certified.
- **Mengandalkan custom visual untuk fungsi yang sudah ada native** — misal Gantt custom untuk bar chart biasa, atau Decomposition Tree custom padahal sudah jadi visual native.
- **Lupa duplikasi visual antar halaman ikut menduplikasi bundle custom visual** — gunakan sync slicers dan desain konsolidasi, bukan copy-paste slicer/filter manual 292 kali.
- **Tidak menguji export/Analyse in Excel** — sebagian custom visual tidak bisa di-export datanya.

## Tips Pro

- Text Filter sangat cocok untuk halaman detail dengan ratusan item — jauh lebih cepat daripada slicer list yang panjang (dan lebih ramah aksesibilitas).
- Scrolling Text untuk "news ticker" periode & last update memberi konteks di halaman Home tanpa menambah card statis.
- Simpan file `.pbiviz` versi terkunci di repo/SharePoint korporat — jika publisher menarik visual dari AppSource, Anda tetap bisa import dari file.
- Cek halaman AppSource publisher untuk changelog — update custom visual terkadang mengubah default format laporan.
- Jika butuh perilaku unik, custom visual bisa dibuat sendiri (Power BI visual tools berbasis TypeScript/`pbiviz` CLI) — tapi pertimbangkan biaya maintenance jangka panjang.

## Terkait

- [Katalog Visual Bawaan](./01-native-visuals-catalog.md)
- [Visual Configuration (Format Pane)](./03-visual-configuration.md)
- [Visual Interactions](./06-visual-interactions.md)
- [Themes & Templates](../05-report-pages/06-themes-and-templates.md)
- [Overview report user (komposisi visual)](../11-user-report-analysis/REPORT_OVERVIEW.md)
