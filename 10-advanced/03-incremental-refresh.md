# Incremental Refresh: Model Besar Tetap Ringan Direfresh

> Incremental refresh (IR) membagu tabel menjadi partisi per periode dan hanya me-refresh partisi terbaru, sehingga model yang terus membesar (145 MB dan bertambah tiap bulan pada report Cost Control) tidak perlu dibaca ulang seluruhnya. Kontraknya sederhana namun ketat: dua parameter bernama persis `RangeStart`/`RangeEnd`, difilter pada kolom tanggal di Power Query.

## Konsep & Penjelasan

### Kenapa dibutuhkan (kasus report ini)
- Sumber = `Folder.Files` di `H:\My Drive` dengan file `YYYY_MM_*.xlsx` per periode: tiap bulan folder bertambah → scan & parse makin lama tiap refresh penuh.
- Tanpa IR: refresh harian membaca ulang semua file (termasuk 2 tahun lalu yang tak pernah berubah).
- Dengan IR: hanya partisi 3 bulan terakhir yang dibaca; data lama "diarsipkan" di partisi terpisah yang tidak disentuh.

### Cara kerja partisi
- Saat pertama di-publish + di-refresh, service membuat partisi per rentang (mis. per bulan) sesuai policy.
- Refresh berikutnya hanya menjalankan query dengan `RangeStart`/`RangeEnd` di-boundary window aktif → partisi lama dilewati.
- Pemisahan ini juga mengurangi tekanan memori: model 145 MB tidak dibaca penuh tiap malam.

### Parameter RangeStart / RangeEnd — aturan wajib
- Nama harus **persis**: `RangeStart` dan `RangeEnd` (case-sensitive, tanpa variasi).
- Type **Date/DateTime** (tidak boleh teks/angka).
- Dibuat sebagai **parameter Power Query** (Manage parameters), lalu dipakai dalam filter baris tabel fact.

### Policy: archive & window
Contoh untuk data cost control 2 tahunan:
- **Archive data starting**: 3 years 0 months 0 days → semua data lebih tua dari 3 tahun masuk partisi historis yang tidak di-refresh.
- **Incrementally refresh data**: 0 years 3 months 0 days → hanya 3 bulan terakhir yang dibaca ulang tiap jadwal.
- Checkbox **Only refresh complete periods**: boundary dibulatkan ke periode penuh (awal/akhir bulan) — mencegah setengah bulan terbaca dua kali dengan definisi beda.

### Detect data changes
- Jika ada kolom *LastUpdated* (datetime) di fact, centang **Detect data changes** dan pilih kolom itu: partisi lama hanya di-refresh bila nilai LastUpdated-nya berubah — berguna untuk koreksi data historis (revisi closing bulan lalu).

### Parallel refresh partitions
- Partisi-partisi window dapat di-refresh **paralel** (jumlah koneksi paralel dapat diatur; default sekitar 8) — refresh 3 bulan = 3 partisi jalan bersamaan lewat gateway.

### Hybrid tables (real-time + in-memory)
- Centang opsi DirectQuery pada policy (butuh kapasitas Premium/Fabric): hari berjalan dilayani DirectQuery (nyaris real-time), historis tetap in-memory.
- Cocok bila manajemen ingin melihat angka hari ini tanpa menunggu refresh malam.

### Setup di Desktop lalu publish
1. Buat parameter `RangeStart`/`RangeEnd` (nilai sementara bebas, mis. 2010-01-01 s/d 2030-12-31).
2. Filter tabel fact: `[Tanggal] >= RangeStart and [Tanggal] < RangeEnd` (perhatikan: `< RangeEnd`, bukan `<=`).
3. Klik kanan tabel > **Incremental refresh** > set policy.
4. Publish ke service → **refresh pertama memuat semua data** sesuai policy (lama! jalankan di jam sepi) → refresh berikutnya baru ringan.

### Partition view di Tabular Editor
- Setelah publish, buka model via Tabular Editor (butuh XMLA endpoint = kapasitas Premium/Fabric): lihat Partitions per tabel — di situ terlihat partisi historis + window, dan bisa diatur Max Parallel Refreshes.

### Catatan kritis: Folder connector & query folding
- **`Folder.Files` tidak mendukung query folding** → filter tanggal pada file Excel tetap "tidak foldable". IR tetap berjalan, tetapi tiap partisi window masih **membaca semua file di folder** lalu memfilter di mesin M — lambat.
- Mitigasi jangka panjang: pindahkan fact ke database (SQL/view ERP) yang foldable, atau pecah folder per tahun agar scan sempit. Untuk detail arsitektur lihat [Gateway & Refresh](../09-publish-and-deploy/02-gateway-and-refresh.md).

### Migrasi data historis
- Refresh pertama = muat ulang seluruh historis ke partisi arsip: siapkan waktu (malam/weekend) dan pastikan gateway stabil.
- Setelah terbentuk, **jangan hapus file lama dari sumber** sebelum yakin partisi arsip tidak akan disentuh ulang (jika ada koreksi historis, gunakan Detect data changes).

## Sintaks & Contoh

Template M untuk tabel fact (mengikuti pola nama file `YYYY_MM_…` di report ini):

```m
let
    RangeStart = #"RangeStart",   // parameter, type date — nama TIDAK boleh diubah
    RangeEnd   = #"RangeEnd",     // parameter, type date — nama TIDAK boleh diubah

    Source = Folder.Files("H:\My Drive\COST CONTROL\Deviasi"),
    NoTemp = Table.SelectRows(Source, each not Text.StartsWith([Name], "~$")),

    // kolom tanggal turunan dari konvensi nama file
    AddTanggal = Table.AddColumn(NoTemp, "Tanggal",
        each #date(
            Number.FromText(Text.Split([Name], "_"){0}),
            Number.FromText(Text.Split([Name], "_"){1}), 1)),

    // FILTER WAJIB — tulis persis seperti ini di step terakhir sebelum expand:
    Filtered = Table.SelectRows(AddTanggal, each
        [Tanggal] >= RangeStart and [Tanggal] < RangeEnd)
in
    Filtered
```

Policy contoh (dialog Incremental refresh):

```text
Table: Deviasi All
Archive data starting      : 3 years 0 months 0 days
Incrementally refresh data : 0 years 3 months 0 days
☑ Only refresh complete periods
☐ Detect data changes      (aktifkan bila ada kolom LastUpdated)
☐ DirectQuery (hybrid)     (butuh Premium/Fabric)
```

## Kesalahan Umum
- **Nama parameter tidak persis** (`RangeStart Date`, `StartRange`) → service tidak mengenali; error "define RangeStart/RangeEnd parameters"; perbaikan: rename persis `RangeStart`/`RangeEnd`.
- **Type bukan Date/DateTime** (teks/angka) → policy tidak bisa dipilih; perbaikan: ubah type parameter.
- **Filter pakai `<= RangeEnd`** → batas atas tumpang tindih satu hari antar window; perbaikan: gunakan `< RangeEnd`.
- **Folder connector lambat walau ada IR** → tidak foldable, semua file tetap dibaca; perbaikan: sumber foldable (SQL) atau folder per tahun.
- **Refresh pertama di jam kerja** → 145 MB dimuat penuh saat orang memakai report; perbaikan: jadwalkan malam/weekend.
- **Mengedit partisi di service tanpa Premium** → XMLA write tidak tersedia di shared capacity; perbaikan: definisikan semua di Desktop sebelum publish.
- **Lupa refresh pertama setelah publish** → model di service kosong; policy baru aktif setelah refresh pertama.

## Tips Pro
- Simpan salinan `.pbip` sebelum mengaktifkan IR — kebijakan menempel pada tabel dan menyulitkan dibongkar.
- Kombinasikan IR dengan menghapus kolom tak terpakai di PQ: ukuran partisi arsip mengecil, model 145 MB bisa turun signifikan.
- Untuk koreksi historis jarang: kolom LastUpdated + Detect data changes lebih murah daripada refresh penuh.
- Audit hasil IR lewat refresh history (durasi refresh harus turun setelah beberapa siklus) dan [DAX Query View](./05-dax-query-view.md) untuk verifikasi angka arsip.
- Jika tabel ERP hanya bisa via export Excel: dorong ke arah view SQL — IR benar-benar terasa bedanya saat folding jalan.

## Terkait
- [Gateway & Refresh](../09-publish-and-deploy/02-gateway-and-refresh.md)
- [Deployment Pipelines](../09-publish-and-deploy/03-deployment-pipelines.md)
- [DAX Query View](./05-dax-query-view.md)
- [Errors Umum](../12-troubleshooting/common-errors.md)
- [Analisis Report User](../11-user-report-analysis/REPORT_OVERVIEW.md)
