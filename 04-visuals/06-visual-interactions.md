# Visual Interactions — Cross-filter, Highlight, Drill & Sync Slicers

> Interaksi antar visual (klik satu → yang lain terfilter/ter-highlight) adalah inti pengalaman eksplorasi Power BI, dan sepenuhnya bisa dikontrol lewat Edit Interactions. Dokumen ini membahas filter vs highlight vs none, drill down vs drillthrough, sync slicers antar halaman (kritikal untuk 292 slicer milik user), jebakan agregasi persentase, dan interaksi dengan measure switching.

## Default Behavior

Secara default, mengklik bagian dari satu visual akan:

- **Cross-highlight** visual lain: bar/kolom yang tidak terpilih diredupkan (tetap terlihat sebagai konteks), tabel/matrix di-filter.
- Sumber klik menentukan perilaku: bar/column/donut/line → highlight; **slicer, table, dan matrix → filter** (visual lain hanya menampilkan data terpilih).
- Klik lagi area kosong (atau tombol clear/Esc) untuk melepas pilihan.
- Highlight bekerja pada **visual yang berbagi model** — filter diterjemahkan menjadi filter konteks di query tiap visual.

## Edit Interactions — Filter, Highlight, None

Cara mengatur: pilih visual **sumber** → ribbon **Format → Edit interactions** → di tiap visual target muncul tiga ikon:

- **Filter (ikon corong)** — target hanya menampilkan data terkait (visual lain "hilang").
- **Highlight (ikon siluet)** — target meredupkan bagian yang tidak terkait (konteks tetap terlihat).
- **None (ikon lingkaran dicoret)** — target tidak terpengaruh sama sekali.

Pola pemakaian yang sehat untuk report cost-control:

- KPI card di header → set **None** (card total harus tetap menampilkan angka periode penuh, atau justru ikut — putuskan sadar, jangan default).
- Chart breakdown → **Highlight** agar pembaca tetap melihat proporsi terhadap total.
- Tabel detail → **Filter** supaya langsung fokus.
- Visual pendukung seperti Scrolling Text / logo → **None**.

## Cross-highlight di Matrix & Table

- Klik baris pada matrix/table akan memfilter/highlight visual lain berdasarkan baris itu (termasuk level hierarki yang sedang aktif).
- Sebaliknya, matrix/table **sebagai target** hanya bisa di-filter (tidak ada highlight parsial di tabel).
- Hati-hati: klik baris total di matrix bisa mengirim filter yang tidak diharapkan — uji interaksi Anda termasuk baris subtotal.

## Drill Down, Drill Up, Expand

Untuk visual dengan **hierarchy** di satu field well (misal `Area → Resto → Item` di Axis):

- **Drill down mode (panah bawah)** — mode aktif lalu klik bar/kategori untuk turun satu level hanya pada kategori itu.
- **Go to the next level (fork satu tingkat)** — seluruh chart turun ke level berikutnya.
- **Expand all** — menampilkan semua level sekaligus (bisa sangat padat — gunakan untuk spot check saja).
- **Drill up (panah atas)** — kembali ke level sebelumnya.
- Ini berbeda dari **drillthrough** yang melompat ke halaman lain dengan konteks — lihat [Drillthrough](../05-report-pages/03-drillthrough.md).

## Tooltips on Hover

- Hover pada data point menampilkan **default tooltip** (isi field wells + Tooltip well) — lihat [Tooltips](../05-report-pages/04-tooltips.md) untuk tooltip page kustom.

## Sync Slicers Antar Halaman (KRITIS)

Report multi-halaman dengan slicer yang sama (user: 292 slicer di 36 halaman) wajib disinkronkan:

1. Ribbon **View → Sync slicers** (pane muncul di samping).
2. Pane menampilkan slicer per **field** dan grid halaman; centang sel untuk **sync** dan ikon mata untuk **visibility**.
3. Pola terbaik: slicer **master** di halaman Home/Summary terlihat, halaman lain bisa hidden-visible (slicer sync tapi tidak ditampilkan — hemat kanvas).
4. Rename **slicer group** untuk sinkronisasi terbatas (misal grup "Periode" dipisah dari grup "Resto").
5. Requirement: slicer harus berasal dari **field yang sama** (nama & tabel sama) — slicer dari field berbeda tidak bisa disinkronkan.

Mengapa penting: tanpa sync, pilihan periode di halaman Summary **tidak terbawa** ke halaman Detail — pengguna menganalisa data bulan yang berbeda tanpa sadar. Sync juga memangkas jumlah slicer yang harus di-maintain.

## Bookmark Interplay

- Bookmark menyimpan state filter/slicer; interaksi visual (filter/highlight) yang sedang aktif juga ikut tertangkap bila bookmark menyimpan **Data**.
- Tombol **Reset** (user punya action button Reset) biasanya berisi bookmark yang merekam kondisi slicer default — pastikan bookmark dibuat **setelah** semua interaksi dibersihkan.
- Detail pola: lihat [Bookmarks](../05-report-pages/02-bookmarks.md).

## Jangan Menjumlahkan Persentase (Don't Sum Percentages)

- Persentase adalah measure **non-aditif**: `% Waste = Waste / Sales` tidak boleh dijumlahkan antar resto/bulan.
- Jebakan klasik: kolom `%` dari sumber data di-drag ke visual → Power BI default **Sum** → total bulanan menjadi jumlah angka persen (bisa > 100%).
- Solusi: tulis sebagai **measure** agar total dihitung ulang dalam konteks:

```dax
% Waste = DIVIDE ( [Total Waste], [Total Sales] )

-- BENAR: subtotal matrix menghitung ulang SUM(Waste)/SUM(Sales)
-- SALAH: SUM( Tabel[KolomPersen] )  → persen dijumlah persen
```

- Bila terpaksa pakai kolom persen, ganti agregasi default menjadi Average — tapi tetap kurang tepat bila weight antar baris berbeda; measure selalu lebih benar.

## Interaksi dengan Measure Switching

- **Field parameters** (Modeling → New parameter → Fields) membuat slicer yang mengganti measure/field di visual — grid chart yang sama bisa menampilkan Deviasi / Waste / Growth mengikuti pilihan (pattern "Multi Value CM" ala report user).
- Interaksi tetap bekerja normal: klik bar akan memfilter visual lain berdasarkan kategori, bukan berdasarkan measure.
- Kombinasi power: field parameter + bookmarks + sync slicers = satu halaman yang berfungsi seperti tiga halaman.

## Sintaks & Contoh

```text
[Checklist interaksi halaman Summary Deviasi]
1. Pilih slicer Periode → Format → Edit interactions:
   - Semua chart breakdown = Filter
   - Card KPI header = Filter (ikut periode)
   - Scrolling Text = None
2. Pilih chart Deviasi by Area (sumber):
   - Matrix detail = Filter
   - Card header = None  (jaga total tetap tampil)
3. View → Sync slicers:
   - Periode, Area, PIC, Resto → sync + visible di semua halaman analisa
4. Simpan bookmark "Reset" setelah semua pilihan dibersihkan,
   assign ke tombol Reset.
```

## Kesalahan Umum

- **Membiarkan default interaksi di semua visual** → halaman dengan 48 visual jadi kacau: satu klik meredupkan semuanya; atur sadar per target.
- **Lupa sync slicers** → analisa lintas halaman tidak konsisten (periode di halaman A ≠ halaman B).
- **Menjumlahkan kolom persen** (implicit SUM) → subtotal salah; ubah jadi measure `DIVIDE`.
- **Menganggap drill down = drillthrough** → drill down menelusuri hierarki di visual yang sama; drillthrough berpindah halaman.
- **Menaruh slicer field berbeda tapi berharap sync** → sync hanya jalan untuk field identik.
- **Interaksi none pada semua card** tanpa pertimbangan → kadang justru ingin card ikut terfilter; putuskan per KPI.

## Tips Pro

- Audit interaksi per halaman sebagai rutinitas QA: klik setiap slicer & chart sumber, screenshot hasil, pastikan tidak ada visual "mati" atau hilang tak terduga.
- Slicer dropdown/list panjang (ratusan item) memperlambat render — ganti dengan Text Filter (custom visual user) untuk pencarian nama item.
- Gunakan Performance Analyzer setelah mengubah interaksi untuk melihat visual mana yang query-nya ikut membengkak akibat highlight.
- Untuk demo/rapat: mode **Focus mode** (ikon fokus di header visual) mengisolasi satu visual; kombinasi dengan bookmark bisa jadi "presentation mode".
- Pertimbangkan **Optimization presets** (File → Options) saat desain interaksi di halaman padat visual.

## Terkait

- [Katalog Visual Bawaan](./01-native-visuals-catalog.md)
- [Custom Visuals](./02-custom-visuals.md)
- [Visual Configuration (Format Pane)](./03-visual-configuration.md)
- [Bookmarks](../05-report-pages/02-bookmarks.md)
- [Drillthrough](../05-report-pages/03-drillthrough.md)
- [Page Layout](../05-report-pages/01-page-layout.md)
