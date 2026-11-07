# Katalog Visual Bawaan (Native Visuals) Power BI

> Katalog lengkap visual bawaan Power BI yang dikelompokkan berdasarkan tujuan analisa: perbandingan, komposisi, distribusi, relasi, ranking, KPI, tabel, trend, peta, dan visual ekstra/AI. Setiap visual dijelaskan dari sisi kegunaan, fields wells, kapan dipakai, kapan harus dihindari, plus contoh nyata di report cost-control restoran (F&B).

## Prinsip Umum Memilih Visual

- Mulai dari **pertanyaan bisnis**, bukan dari visual: "bandingkan X antar kategori" → bar chart; "apa kontribusi X ke total" → waterfall/donut; "berapa nilai KPI sekarang" → card.
- Satu visual = satu pesan. Halaman yang terlalu padat (misal *Summary Deviasi* dengan 48 visual) sebaiknya diringkas dengan drillthrough, bookmark toggle, atau small multiples.
- Kenali batas kognitif pembaca: pie > 5 kategori, clustered chart > 3 series, dan combo chart > 2 sumbu mulai sulit dibaca.
- Nama internal di file `.pbix` sering berbeda dari nama di UI — contoh di report user: `pivotTable` = Matrix, `cardVisual` = Card (new), `tableEx` = Table, `lineClusteredColumnComboChart` = Line and clustered column chart.

## 1. Perbandingan — Bar & Column

### Bar Chart (horizontal)
- **Kegunaan:** membandingkan nilai antar kategori, ideal saat nama kategori panjang (nama item, nama resto).
- **Fields wells:** Y axis (kategori), X axis (measure), Legend & Small multiples (opsional), Tooltips.
- **Kapan pakai:** ranking resto berdasarkan `[Deviasi CM]`, daftar item penyumbang waste terbesar.
- **Kapan hindari:** sumbu kategorinya adalah waktu (pakai column/line); jumlah kategori puluhan tanpa Top N filter.

### Column Chart (vertikal)
- **Kegunaan:** perbandingan antar kategori diskrit, terutama periode waktu (bulan, minggu).
- **Fields wells:** X axis (kategori/tanggal), Y axis (measure).
- **Kapan pakai:** deviasi per bulan (Jan–Des), jumlah resto bermasalah per area.
- **Kapan hindari:** nama kategori panjang akan miring/terpotong — pindah ke bar chart.

### Clustered Bar/Column
- **Kegunaan:** membandingkan 2–3 series antar kategori, misal `Deviasi CM` vs `Deviasi LM` per resto.
- **Kapan hindari:** lebih dari 3 series menjadi ramai → gunakan small multiples atau filter.

### Stacked Bar/Column & 100% Stacked
- **Kegunaan:** total per kategori sekaligus komposisi penyusunnya (misal COM per kategori bahan).
- **Kapan hindari:** membaca nilai segmen tengah pada stacked chart sulit; untuk proporsi murni pakai 100% stacked, untuk nilai pasti pakai clustered/matrix.

## 2. Komposisi — Donut, Pie, Treemap, Waterfall

### Donut Chart
- **Fields wells:** Legend (kategori), Values (measure).
- **Kapan pakai:** kontribusi maksimal 4–5 kategori ke satu total, misal komposisi deviasi per jenis (Over/Short/Adjustment). Lubang tengah donut bisa dipakai menaruh card/label total.
- **Kapan hindari:** > 5 kategori, atau pembaca perlu membandingkan selisih antar kategori secara presisi (sudut sulit dibandingkan) → bar chart.

### Pie Chart
- **Kapan hindari:** hampir selalu — donut atau bar lebih baik. Dipakai hanya jika publik sudah terbiasa dan kategorinya sedikit.

### Treemap
- **Kegunaan:** komposisi hierarkis banyak kategori (rect area = nilai), misal deviasi per Area > Resto dalam satu kotak.
- **Kapan hindari:** butuh membaca nilai tepat (rect sulit dibanding presisi bar) — aktifkan data labels.

### Waterfall Chart
- **Fields wells:** Category (X), Breakdown (detail opsional), Y axis (value), Tooltips.
- **Kegunaan:** bridge kontribusi positif/negatif ke total — visual wajib analisa margin: `Stok Awal → Barang Masuk → COM → Waste → Stok Akhir`, atau jembatan `Deviasi LM → Deviasi CM` (kenaikan/turunan).
- **Kapan hindari:** hanya satu komponen; pastikan measure bisa positif dan negatif agar biru/merahnya muncul.

## 3. Distribusi — Scatter & Histogram

### Scatter Chart
- **Fields wells:** X axis, Y axis (keduanya measure numerik), Legend, Size (bubble), Play axis (animasi waktu), Details.
- **Kapan pakai:** korelasi dua metrik, misal `Sales` vs `% Waste` per resto — kuartan kanan bawah = resto ramai tapi boros.
- **Kapan hindari:** X axis bukan numerik kontinu; data terlalu sedikit (< 20 titik).

### Histogram
- **Cara:** column chart dari field numerik yang di-*binning* (klik kanan field di Fields pane → New group → Size of bins), atau visual histogram di AppSource.
- **Kegunaan:** distribusi frekuensi, misal sebaran Z-Score waste antar resto untuk deteksi anomali.

## 4. Relasi — Combo Chart (Line + Column)

### Line and Clustered Column Chart
- **Fields wells:** X axis (waktu), Column values (misal `COM` dalam Rp), Line values (misal `% Deviasi`), secondary axis otomatis untuk line.
- **Kapan pakai:** dua metrik dengan satuan berbeda pada satu timeline — user memakai 10 combo chart untuk pola "nilai (Rp) vs persentase".
- **Kapan hindari:** jika dua sumbu Y tidak dilabeli jelas, pembaca bisa salah membaca skala; jangan pakai secondary axis untuk "menyesuaikan" garis agar terlihat sejajar.

## 5. Ranking — Bar Chart Ter-sort + Top N

- **Pola:** bar chart horizontal + sort descending by measure + **Top N visual-level filter** (Filters pane → filter pada field → Top N → Top items → By value: measure ranking).
- **Contoh:** halaman *Top 10 Item Deviasi* — 10 item dengan `[Deviasi CM]` terburuk; *Top 5 Waste* — kontributor susut terbesar.
- **Kapan hindari:** ranking yang di-*hardcode* di DAX `TOPN` pada measure jika user butuh fleksibilitas N — kombinasikan dengan what-if parameter.

## 6. KPI — Card, Multi-row Card, Gauge, KPI

### Card (new) / Card
- **Kegunaan:** satu angka kunci; Card (new) mendukung reference labels (target, rata-rata, min/max), data bars mini, dan sparkline.
- **Fields wells:** Fields (1 measure + opsional breakdown).
- **Contoh:** `Total Deviasi CM`, `Last Update Dev` di halaman Home; user punya 24 card.
- **Kapan hindari:** KPI tanpa konteks pembanding — selalu dampingi vs LM / vs target.

### Multi-row Card
- **Kegunaan:** beberapa KPI dalam satu visual, hemat kanvas.

### Gauge
- **Fields wells:** Min, Max, Target, Value.
- **Kapan pakai:** progress terhadap target yang jelas (misal susut aktual vs standar susut 2%).
- **Kapan hindari:** tanpa target bermakna, gauge menyesatkan — pakai card.

### KPI Visual
- **Fields wells:** Indicator (measure), Trend axis (tanggal), Target goals.
- **Kapan pakai:** satu metrik + arah tren + target, misal `Sales Growth` per bulan.

## 7. Tabel — Table & Matrix

### Table
- **Kegunaan:** nilai detail presisi, export-friendly (Analyse in Excel / Export data). User punya 20 table (*Data Deviasi*, *Data Waste*).
- **Kapan hindari:** > 7 kolom atau untuk analisa pola — pindah ke matrix/chart.

### Matrix
- **Kegunaan:** pivot dengan hierarki (Area > PIC > Resto > Item), +/- drill expanders, subtotal, stepped layout, dan heat map via conditional formatting. User punya 27 matrix — visual paling serbaguna untuk laporan operasional.
- **Fields wells:** Rows, Columns, Values, Tooltips (+ Drill through di level page).
- **Kapan hindari:** hierarki sangat dalam tanpa collapse default — pengguna tenggelam di baris.

## 8. Trend — Line & Area

### Line Chart
- **Fields wells:** X axis (tanggal), Y axis (measure), Legend, Small multiples.
- **Kapan pakai:** tren kontinu antar periode — *Last 13 Months* deviasi, trend growth. Aktifkan markers hanya saat titik sedikit; matikan smooth line agar bentuk kurva tidak "dihaluskan" menyesatkan.
- **Kapan hindari:** X axis kategorikal non-waktu; terlalu banyak series (> 5) → small multiples.

### Area Chart
- **Kegunaan:** komposisi kumulatif antar waktu; stacked area untuk volume, bukan untuk nilai yang saling tumpang tindih.

## 9. Peta — Map, Filled Map, Azure Map

- **Map (bubble):** lokasi = bubble dengan ukuran nilai (sales per outlet); **Filled Map (choropleth):** shading per wilayah (deviasi per area/kota); **Azure Map:** peta modern default dengan layer heat map, dan lebih baik geocoding-nya.
- **Fields wells:** Location (atau Latitude/Longitude), Size/Legends, Color saturation.
- **Kapan hindari:** data tanpa dimensi geografis yang jelas; koordinat tidak konsisten (ada yang pakai nama, ada yang pakai lat/long).

## 10. Ekstra & AI Visuals

| Visual | Kegunaan | Catatan cost-control |
|---|---|---|
| Ribbon Chart | stacked column + re-ranking antar periode | melihat pergeseran ranking item penyumbang deviasi bulan ke bulan |
| Funnel Chart | tahapan berurutan | pipeline PO → PI → pembayaran (halaman *PO blm PI*) |
| Decomposition Tree (AI) | dekomposisi measure ad-hoc by dimensi apa pun + split "High value" | **sangat cocok untuk analisa deviasi**: Total deviasi → Area → Resto → PIC → Item, interaktif tanpa bikin halaman baru |
| Key Influencers (AI) | faktor yang paling memengaruhi metrik | driver utama kenapa resto deviasi negatif |
| Smart Narrative (AI) | ringkasan teks otomatis dinamis mengikuti filter | narasi otomatis di halaman Summary |
| Q&A Visual | pertanyaan bahasa natural | perlu naming kolom/measure bersih |
| R/Python Visual | statistik lanjutan via script | tidak ikut cross-filter; butuh runtime lokal |
| Play Axis (dynamic) | animasi scatter per waktu | evolusi posisi resto di kuadran Sales vs Waste |
| Paginated Report (.rdl) | bukan visual di PBIX, tapi jenis report pixel-perfect terpisah | untuk cetak/export massal (faktur, laporan rinci per resto) |

## Sintaks & Contoh

Measure pendamping yang paling sering dipakai bersama katalog visual di atas:

```dax
-- Ranking untuk Top N (dipakai Top 10 Item Dev)
Rank Dev = RANKX ( ALL ( Item[Nama Item] ), [Deviasi CM], , DESC, DENSE )

-- Selisih antar periode untuk waterfall bridge
Bridge CM vs LM = [Deviasi CM] - [Deviasi LM]

-- Persentase kontribusi untuk donut/treemap
% to Total = DIVIDE ( [Deviasi CM], CALCULATE ( [Deviasi CM], ALLSELECTED ( Resto ) ) )
```

## Kesalahan Umum

- **Pie/donut > 5 kategori** → ganti bar chart ter-sort atau treemap.
- **Gauge tanpa target yang jelas** → angka bisa terlihat "bagus" padahal tidak ada basis pembanding; pakai card + varian.
- **Line chart untuk kategori non-waktu** → pakai bar/column.
- **Stacked chart untuk membandingkan nilai antar segmen** → segmen tengah sulit dibaca; pakai clustered.
- **Combo chart dua sumbu tanpa label satuan** → pembaca tidak tahu mana Rp dan mana %; selalu aktifkan judul sumbu + satuan.
- **Mengandalkan urutan default (alfabetis)** → sort by measure (menu `…` pada visual → Sort axis → pilih measure → descending).
- **Halaman jadi "gudang visual"** (48 visual dalam satu halaman) → pecah via drillthrough/bookmark, atau ringkas dengan small multiples.

## Tips Pro

- Konsistensi warna lintas halaman: merah = loss, hijau = surplus, kuning = perhatian — kunci lewat theme JSON + conditional formatting, bukan manual per visual.
- Matrix + heat map (conditional formatting background) adalah "bar chart dalam bentuk tabel" — padat informasi untuk 27 matrix user.
- Card (new) dengan sparkline menggantikan kombinasi card + line chart kecil.
- Decomposition tree untuk analisa ad-hoc saat rapat — tidak perlu desain halaman tambahan.
- Matikan efek dekoratif berlebihan (shadow/glow) pada visual yang banyak — mengurangi render time.

## Terkait

- [Custom Visuals](./02-custom-visuals.md)
- [Visual Configuration (Format Pane)](./03-visual-configuration.md)
- [Conditional Formatting](./04-conditional-formatting.md)
- [Small Multiples](./05-small-multiples.md)
- [Visual Interactions](./06-visual-interactions.md)
- [Page Layout](../05-report-pages/01-page-layout.md)
- [Detail visual per halaman report user](../11-user-report-analysis/pages-and-visuals.md)
