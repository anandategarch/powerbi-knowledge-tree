# VertiPaq Engine — Arsitektur, Kompresi, dan Ukuran Model

> VertiPaq adalah storage engine columnar di balik model Import mode: memahami Formula Engine vs Storage Engine, cara kompresi (dictionary, value encoding, RLE), dan cara mengukurnya dengan VertiPaq Analyzer adalah kunci menjelaskan kenapa model 145 MB itu besar — dan bagaimana menurunkannya.

## Konsep & Penjelasan

### Arsitektur dua engine

Saat sebuah visual meminta data, DAX diproses oleh dua engine yang berbeda:

| | **Formula Engine (FE)** | **Storage Engine (SE) — VertiPaq** |
|---|---|---|
| Threading | Single-threaded | Multi-threaded (paralel per query) |
| Bekerja pada | Hasil SE + logika kompleks (iterasi, konteks) | Kolom terkompresi di RAM |
| Bahasa | DAX | xmSQL (query internal) |
| Cache | Data cache per query | Scan sangat cepat, hasil di-cache |
| Biaya | CPU — sumber "DAX query lambat" | Memory — ukuran model |

Alur: visual → DAX → FE menyusun rencana → FE memanggil SE (xmSQL) → SE scan kolom → data cache dikembalikan ke FE → FE menyelesaikan kalkulasi. Measure sederhana (`SUM`) hampir seluruhnya di SE; measure ber-iterator (`SUMMARIZE`, `FILTER`, `RANKX`) menambah beban FE — pola `[Z Score]` report ini contohnya.

### VertiPaq: columnar + kompresi

Data disimpan **per kolom**, bukan per baris. Setiap kolom dikompresi dengan tiga teknik (berurutan):

1. **Value encoding** (kolom numerik): nilai digeser berdasarkan minimum kolom (misal harga 12.000–12.500 disimpan sebagai 0–500) → butuh bit jauh lebih sedikit.
2. **Dictionary encoding**: nilai unik disimpan sekali di dictionary; baris hanya menyimpan ID kecil. Kolom dengan 4.000 nama barang = dictionary 4.000 string + ID 2-byte per baris.
3. **RLE — Run-Length Encoding**: ID yang berurutan sama dijadikan "nilai × jumlah" (ID 7 sebanyak 500 baris = satu entri). Sangat efektif jika data terurut/grup berdasarkan kolom itu.

Data dipecah menjadi **segment** (±1 juta baris) yang dikompresi terpisah. Konsekuensi: *cardinality* (jumlah nilai unik) adalah variabel terpenting — bukan jumlah baris.

### Kenapa model user mencapai 145 MB

Fakta model (lihat [Report Overview](../11-user-report-analysis/REPORT_OVERVIEW.md)): 16 tabel Power Query, 109 calculated columns, 161 measures, banyak tabel Excel hasil `Folder.Files`. Penyebab borosnya:

- **Auto date/time**: 109 calculated columns "mayoritas auto date/time" — setiap kolom datetime menumbuhkan **tabel kalender tersembunyi** (Date + ~15 kolom turunan: Year, Month, Day, dll.). Ini pengganda ukuran yang paling sering diabaikan.
- **Kolom teks high-cardinality sebagai kunci join**: `Deviasi All[Nama Barang]`, `Inventory[(Old) Nama Barang]`, `Waste All[Nama new item ]` — nama barang nyaris unik per baris → dictionary string besar **di setiap tabel fact** (Deviasi All, Waste All, 9901, 3243, 4205, 3224, Rekap Mentah BM, Standar Susut — 8 dictionary untuk "nama barang yang sama").
- **Kunci teks kotor**: trailing space (`Nama new item `) dan variasi kapital menaikkan distinct value → dictionary membengkak dan join miss.
- **Duplikasi master**: `Inventory A/B`, `Resto A/B` (untuk periode pembanding) — dua salinan tabel demi pola perbandingan.
- **Tabel log/mentah** (`Update`, `Rekap Mentah BM`) ikut dimuat penuh.

### Cara mengukur: VertiPaq Analyzer

Ukuran model jangan ditebak — diukur. Alat: **DAX Studio** (tombol VertiPaq Analyzer) atau Tabular Editor. Langkah:

1. Buka report di Desktop → buka DAX Studio → connect ke "Power BI Desktop – <nama model>".
2. Jalankan **VertiPaq Analyzer** → export ke Excel untuk dibandingkan antar waktu.
3. Baca per tabel: **Rows, Table Size, Cardinality** per kolom, **Encoding** (Hash/Value/RLE), dan % dari total model.

Contoh output (ilustrasi angka, pola khas model seperti ini):

| Tabel | Rows | Size | Kolom terboros | Cardinality | Encoding |
|---|---|---|---|---|---|
| Deviasi All | 248.000 | 26 MB | Nama Barang (text) | 4.200 | Hash |
| Date (auto-date) | 1.460 | 9 MB | 15 kolom turunan | 7–530 | Hash/Value |
| Inventory | 4.200 | 3 MB | (Old) Nama Barang | 4.200 | Hash |
| Resto | 90 | <1 MB | — | rendah | — |

Kolom dengan size besar + cardinality tinggi + encoding Hash = kandidat utama optimalisasi.

### Teknik kompresi & pengaruh datatype

Urutan kompresi dari paling hemat ke paling boros:

1. **Boolean** → 2. **Whole number (integer)** → 3. **Date (disimpan sebagai integer)** → 4. **Decimal/Currency (8 byte)** → 5. **DateTime dengan jam** (cardinality tinggi) → 6. **Text** (paling boros: string dictionary, tanpa value encoding).

Praktik untuk model ini:

- `MonthNumber`, `Year` → integer, bukan teks "Januari…" untuk kolom kunci (label bulan boleh tetap teks untuk display).
- Hindari datetime ber-jam jika tidak dipakai; `Date` + kolom jam terpisah bila perlu.
- **Kunci join teks → integer surrogate key**: buat `Inventory[ItemID]` (index), lalu map semua tabel fact ke `ItemID` di Power Query. Satu langkah ini bisa memangkas belasan MB.
- TRIM/UPPER kolom kunci di PQ → distinct value turun → dictionary mengecil *dan* join lebih akurat.

### Calculated column vs measure — untuk performa

| | Calculated column | Measure |
|---|---|---|
| Dihitung | Sekali per refresh | Setiap query |
| Disimpan | Ya — makan memory permanen | Tidak — makan CPU saat render |
| Cocok untuk | Grouping/slicer/relasi (atribut statis) | KPI, rasio, nilai dinamis |

Aturan: kolom hanya jika harus bisa di-slice/diced atau jadi kunci; selebihnya measure. Contoh: "Surplus/Loss" boleh kolom (di-slicer), tapi `[% BOM]` harus measure.

### Many-to-many relationship: dampaknya

Model ini punya banyak relasi M:M (`3243→Resto`, `Rekap Mentah BM→Resto`, `9901→Resto`, dll. — lihat [Model & DAX](../11-user-report-analysis/model-and-dax.md)). M:M hampir selalu **gejala, bukan fitur**: muncul karena `Resto[Resto] kemungkinan duplikat di master (kotor), sehingga engine menolak M:1.

Dampak: filter propagation lebih lemah dan ambigu, engine menambah baris blank, query plan makin kompleks (beban FE naik). Perbaikan: bersihkan master `Resto` (dedupe di PQ) → relasi jadi M:1 standar; kualitas model naik, performa ikut naik.

### User-defined hierarchies — aman

Hierarchy (`Date[Year] → Date[Month]`) hanya **metadata** — tidak menyimpan data baru, tidak menambah ukuran model. Aman dipakai sebanyak apa pun. Yang boros adalah *kolom*, bukan hierarchy-nya.

### Materialisasi: refresh time vs query time

- **Refresh** (saat data diperbarui): baca sumber (Power Query) → tulis & kompresi ke VertiPaq (CPU, bisa lama di text high-cardinality). Model 145 MB dari ratusan file Excel via `Folder.Files` = refresh yang panjang.
- **Query** (saat report dipakai): scan kolom terkompresi di RAM — cepat, tapi makin banyak kolom & cardinality, makin banyak bit yang di-scan.

Jadi: model besar itu membayar dua kali — refresh lama, dan memory resident tinggi di Service. Mengecilkan model memperbaiki keduanya.

### Kolom mana yang kemungkinan boros di model ini (prakiraan)

1. Semua artefak **auto date/time** (tabel kalender tersembunyik per kolom datetime) — potensi -15% s/d -35% ukuran.
2. Delapan salinan dictionary `Nama Barang` (text, high-cardinality) di seluruh tabel fact — ganti `ItemID` integer.
3. `Inventory A` / `Resto A` (salinan pembanding) — bisa dihapus jika perbandingan dialihkan ke TREATAS (lihat [Dynamic Filtering](../06-filters-and-slicers/03-dynamic-filtering.md)).
4. Kolom hasil expand `Combine Files` yang tidak dipakai visual apa pun — hapus di PQ sebelum load.
5. Tabel log `Update` — pertimbangkan aggregate atau filter rentang di PQ.

## Sintaks & Contoh

Verifikasi ukuran model:

```text
DAX Studio → connect "Power BI Desktop – Cost Control"
  → VertiPaq Analyzer → (jalankan) → Export Excel
  → sort kolom "Size" descending → catat top-10 kolom
```

Membuat surrogate key integer di Power Query:

```text
Inventory: Add Column → Index Column (1) → nama [ItemID]
Deviasi All: Merge ke Inventory by [(Old) Nama Barang] → ambil [ItemID]
             → hapus kolom teks [Nama Barang] dari tabel fact
Relasi model: Deviasi All[ItemID] M:1 Inventory[ItemID]
```

Mematikan auto date/time (untuk file baru dan existing):

```text
File → Options → Data Load → hilangkan centang "Auto date/time for new files"
Modeling → pilih tabel Date → Mark as date table (Date[Date])
→ hapus kolom-kolom kalender tersembunyi bawaan auto-date
```

## Kesalahan Umum

- **Menganggap ukuran .pbix = ukuran model** — .pbix menyimpan model terkompresi; saat berjalan di memory, beberapa struktur tambahan muncul. Ukur dengan VertiPaq Analyzer, bukan file size saja.
- **Menghapus kolom tapi ukuran tidak turun** — perlu save + refresh penuh agar struktur lama dibuang; ukur lagi setelahnya.
- **Datetime ber-jam untuk data harian** — cardinality meledak; cukup date.
- **Calculated column dipakai untuk KPI** — makan memory permanen untuk nilai yang seharusnya dihitung per query (measure).
- **M:M dibiarkan karena "asal jalan"** — filter propagation lemah & plan rumit; dedupe master dulu.
- **Label bulan teks dipakai sebagai kunci sorting** — sorting salah; gunakan `MonthNumber` integer sebagai sort-by column.

## Tips Pro

- Jalankan VertiPaq Analyzer **tiap kuartal**, simpan snapshot Excel, dan bandingkan — pertumbuhan model terlihat sebelum jadi masalah.
- Budget: Pro capacity 1 GB/dataset; mulai diskusi Premium/PPU jika model >700 MB. Model 145 MB masih aman, tapi arah pertumbuhannya harus diawasi.
- Fact unpivot (pola `Attribute`/`Jenis` di report ini) sudah benar — pertahankan; unpivot menurunkan jumlah kolom dan menaikkan kompresi.
- Sebelum menambah calculated column, tanya: "apakah ini harus bisa di-slicer?" Kalau tidak, jadikan measure.
- Kombinasi `ItemID` integer + TRIM kunci adalah perbaikan model dengan ROI tertinggi untuk report bertipe ini — sekaligus memperbaiki akurasi join (trailing space `Nama new item `).

## Terkait

- [Performance Analyzer](./02-performance-analyzer.md) — mengukur query setelah model dirampingkan
- [Optimization Checklist](./03-optimization-checklist.md) — daftar aksi per kategori
- [Filter Types](../06-filters-and-slicers/01-filter-types.md) — biaya filter di model besar
- [Dynamic Filtering](../06-filters-and-slicers/03-dynamic-filtering.md) — TREATAS sebagai pengganti tabel duplikat A/B
- [Model & DAX](../11-user-report-analysis/model-and-dax.md) — daftar relasi M:M dan 161 measures
