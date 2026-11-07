# Filter Types — Panduan Lengkap Filters Pane

> Filters pane adalah pusat kontrol filter report Power BI: tiga level (visual, page, report) yang selalu digabung dengan logika **AND**, lima tipe filter (basic, advanced, relative date, Top N, measure), plus mode Apply yang menentukan kapan perubahan dieksekusi terhadap model.

## Konsep & Penjelasan

### Tiga level filter di Filters pane

| Level | Nama di pane | Scope | Contoh di report Cost Control |
|---|---|---|---|
| **Visual-level** | Filters on this visual | Satu visual saja | Filter Top N di halaman *Top 10 Item Dev* agar bar chart hanya menampilkan 10 item teratas |
| **Page-level** | Filters on this page | Semua visual satu halaman | Halaman *Waste by Resto* dikunci `Periode = EoM` untuk analisa end-of-month |
| **Report-level** | Filters on all pages | Semua halaman (36 halaman) | `Date[Year]` dipatok ke tahun berjalan agar semua halaman konsisten |

Semua perubahan tersimpan di dalam file .pbix — user di Service tetap bisa melihat (dan jika diizinkan, mengedit) filter ini.

### Logika penggabungan: AND, bukan override

Urutan penerapan: **report-level → page-level → visual-level → slicer di canvas**. Semua digabung dengan **AND** — tidak ada yang menimpa yang lain.

Contoh: report-level `Resto[Area] = "JAKARTA"` + visual-level `Resto[Area] = "SURABAYA"` → visual kosong (AND tidak terpenuhi), bukan menampilkan SURABAYA. Ini penyebab paling umum visual tiba-tiba blank.

Konsekuensi praktis: semakin banyak level filter, semakin sempit konteks data. Tidak ada mekanisme OR bawaan antar filter card — jika butuh OR, gunakan satu card dengan multi-select (`IN` beberapa nilai), atau ubah logika ke dalam measure (lihat [Dynamic Filtering](./03-dynamic-filtering.md)).

### Lima tipe filter (dropdown "Filter type")

1. **Basic filtering** — daftar checkbox; bisa single-select atau multi-select; ada opsi "Select all". Cocok untuk kolom dengan jumlah nilai wajar (Area, PIC, `Periode` WEEK 1/2/3/EOM).
2. **Advanced filtering** — operator pembanding:
   - Text: `contains`, `begins with`, `is (not) blank`, dll.
   - Number: `equals`, `greater than`, `between`, `is (not) blank`.
   - Date: `is`, `is on or before`, `is after`, dll.
   - Multi-value: pada basic filter pilih beberapa nilai sekaligus (setara `IN`).
3. **Relative date** — `last/this/next` N days/weeks/months/quarters/years, dengan toggle **Include today** dan **Anchor date** (titik acuan tetap, bukan hari ini). Dipakai untuk halaman *Last 13 Months*.
4. **Top N** — Top/Bottom N item, **By value** wajib berupa measure (mis. `[Deviasi CM]`). Nilai N di panel ini **statis**; untuk Top N dinamis lihat [Dynamic Filtering](./03-dynamic-filtering.md).
5. **Measure filter** — filter berdasarkan hasil measure (mis. `[Deviasi CM] < 0`). **Hanya tersedia di visual-level** — Power BI tidak mengizinkan measure di page/report filter karena hasilnya tergantung konteks tiap visual.

### Mode Apply perubahan (basic vs advanced)

- **Mode default (basic)**: setiap klik checkbox langsung dieksekusi → satu klik = satu query baru ke model. Di report besar (model 145 MB, 48 visual di satu halaman *Summary Deviasi*), ini terasa berat.
- **Mode single Apply**: File → Options and settings → Options → **Query Reduction** → *Slicers and filters* → "Require a single Apply button to apply changes". Semua perubahan filter/slicer ditunda sampai tombol **Apply** ditekan. Wajib dipertimbangkan untuk DirectQuery, dan nyaman untuk model besar di Import mode.

Catatan: mode ini juga mengurangi jumlah query saat user "iseng" mengkombinasikan banyak filter sebelum melihat hasil.

### Filter vs slicer — bagaimana keduanya berinteraksi

| Aspek | Filter (pane) | Slicer (canvas) |
|---|---|---|
| Lokasi | Panel Filters | Visual di halaman |
| Sinkron antar halaman | Report-level otomatis berlaku ke semua halaman | Perlu fitur **Sync slicers** (lihat [Slicer Guide](./02-slicer-guide.md)) |
| Visibilitas ke user | Bisa disembunyikan / read-only / editable di Service | Selalu terlihat, tempat interaksi utama |
| Cocok untuk | Filter teknis yang user tidak perlu sentuh (periode EoM, exclude kategori trial) | Interaksi harian (pilih bulan, pilih resto) |

Keduanya digabung AND dengan semua filter lain. Jangan duplikasi filter yang sama di pane *dan* slicer — pilih satu agar tidak membingungkan.

### Filter dengan measure (visual-level only)

Kasus klasik di report cost-control: **tampilkan hanya resto yang deviasinya negatif** (`[Deviasi CM] < 0`).

Langkah:
1. Tarik measure `[Deviasi CM]` ke area *Filters on this visual* dari matrix *Deviasi by Resto*.
2. Filter type: Advanced → `is less than` → nilai `0`.
3. Baris dengan `[Deviasi CM] >= 0` hilang dari visual tersebut saja.

Karena measure filter tidak bisa naik ke page level, alternatifnya: (a) pasang filter serupa di tiap visual, atau (b) buat flag sebagai kolom/calculated column lalu difilter di page level (ingat beban model — lihat [VertiPaq](../07-performance/01-vertipaq-engine.md)).

### Relative date (last N months/weeks)

- Relative date dihitung dari **TODAY()** saat report dibuka di Service — artinya hasilnya bergeser setiap hari tanpa perlu republish. Aktifkan "Include today" agar data hari ini ikut.
- Gunakan **Anchor date** jika periode harus tetap (mis. dibandingkan ke akhir bulan lalu), misal untuk halaman *Last 13 Months* agar 13 bulan tidak "bergerak".
- Relative date bisa dipasang di semua level (visual/page/report) maupun sebagai slicer.

### Perilaku filter: Import vs DirectQuery

| Aspek | Import (model user, 145 MB) | DirectQuery |
|---|---|---|
| Eksekusi filter | In-memory VertiPaq — mikrodetik, tanpa network | Diterjemahkan menjadi SQL WHERE ke sumber — tergantung latensi DB |
| Tiap perubahan filter | 1 query ke engine lokal (cepat, tapi kalau visual banyak tetap terasa) | 1 round-trip ke database — aktifkan single Apply |
| Operator teks `contains` | Scan dictionary kolom | Menjadi `LIKE '%..%'` di SQL — bisa berat tanpa index |
| Cache | Data model sudah di RAM | Hasil bisa di-cache sebentar di Service, tapi tetap kembali ke sumber saat data berubah |

### NULL handling di filter

- Nilai kosong muncul sebagai **(Blank)** di daftar basic filter — hati-hati "Select all" ikut mencentang (Blank).
- Advanced filter punya operator **`is blank` / `is not blank`** (text & number).
- **"Show items with no data"** (opsi pada field/filter card): memaksa baris dimensi tanpa transaksi tetap tampil (mis. cabang baru belum ada deviasi bulan ini berguna untuk audit). Berhati-hatilah di matrix besar — bisa memblowup jumlah baris.
- Banyak (Blank) yang muncul biasanya gejala **join miss** di model — contoh nyata: kolom kunci `Waste All[Nama new item ]` punya trailing space sehingga tidak match ke `Inventory[(Old) Nama Barang]`. Perbaiki di Power Query (TRIM/UPPER), bukan dihabiskan dengan filter manual.

## Sintaks & Contoh

Contoh susunan filter untuk halaman *Summary Deviasi* (pola yang bisa ditiru):

```text
REPORT-LEVEL (all pages)
  Date[Year]        Basic  = 2026            → konsisten di 36 halaman
  Resto[Type Kitchen] Basic, exclude "TRIAL" → buang kategori uji

PAGE-LEVEL (Summary Deviasi)
  Date[Month]       Basic = bulan berjalan
  Periode[Periode]  Basic = EoM

VISUAL-LEVEL
  Matrix by Resto:  [Deviasi CM]  is less than 0   → hanya resto loss
  Bar chart Item:   Top N, Top 10 by [Deviasi CM]
  Card tren:        Date[Date] relative date, last 3 months
```

Measure yang lazim dipakai sebagai field filter visual:

```dax
-- Flag baris rugi (measure filter: is less than 0 → hanya resto loss)
Deviasi Negatif =
IF ( [Deviasi CM] < 0, [Deviasi CM] )
```

Pengaturan single Apply (Query Reduction):

```text
File → Options and settings → Options
  → Query Reduction
     → Slicers and filters: (•) Require a single Apply button to apply changes
```

## Kesalahan Umum

- **Mengira filter page-level menimpa visual-level** — semuanya AND; jika hasil kosong, cek konflik nilai antar level, hapus salah satu.
- **Menaruh measure ke page/report filter** — tidak bisa; measure hanya visual-level. Perbaikan: filter per visual, atau jadikan kolom flag (dengan sadar beban model).
- **Top N dengan By value berupa kolom mentah** — ranking jadi tidak bermakna; By value harus measure agregat.
- **Relative date lupa "Include today"** — transaksi hari ini hilang, user komplain "data kurang".
- **"Select all" ikut menceklis (Blank)** — baris join-miss ikut terpilih dan menurunkan akurasi angka; bersihkan kunci di PQ.
- **DirectQuery: setiap klik filter memukul database** — aktifkan single Apply dan edukasi user.
- **"Show items with no data" menyala di semua visual** — jumlah baris visual meledak; nyalakan hanya untuk visual audit tertentu.

## Tips Pro

- **Rename filter card** (klik kanan → Rename) agar user paham maksudnya; atur "Users can edit filters in Service" di Filtering experience sesuai kebijakan.
- Untuk filter global yang jarang berubah, pilih **report-level filter** daripada menaruh slicer yang sama di 36 halaman (mengurangi jumlah slicer dari 292).
- Simpan kondisi default dengan **bookmark Reset** per halaman — report user sudah punya tombol Reset di banyak halaman; pastikan bookmark merekam kondisi filter bersih.
- Filter yang paling restriktif tidak mengurangi jumlah query — yang mengurangi waktu adalah jumlah visual. Prioritaskan: kurangi visual dulu (lihat [Optimization Checklist](../07-performance/03-optimization-checklist.md)).
- Saat debugging "kenapa angkanya beda", baca Filters pane dari atas (report) ke bawah (visual) — sumber perbedaan hampir selalu ada di salah satu level yang terlupakan.

## Terkait

- [Slicer Guide](./02-slicer-guide.md) — pasangan filter pane di canvas
- [Dynamic Filtering](./03-dynamic-filtering.md) — TREATAS, what-if, field parameters
- [VertiPaq Engine](../07-performance/01-vertipaq-engine.md) — dampak model besar terhadap kecepatan filter
- [Performance Analyzer](../07-performance/02-performance-analyzer.md) — mengukur dampak filter terhadap durasi
- [Row-Level Security](../08-security/01-rls.md) — "filter" yang menempel pada user
- [Report Overview](../11-user-report-analysis/REPORT_OVERVIEW.md) — konteks report cost-control
