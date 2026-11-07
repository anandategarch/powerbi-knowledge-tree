# Performance Analyzer & DAX Studio — Diagnosa Query dan Render

> Performance Analyzer (Power BI Desktop) mencatat waktu tiap visual menjadi tiga komponen (DAX Query, Visual Display, Other); DAX Studio menelanjang query lebih dalam (server timings, xmSQL, cache warm vs cold). Bersama keduanya, halaman lambat seperti *Summary Deviasi* (48 visual) bisa didiagnosis per visual, bukan dikira-kira.

## Konsep & Penjelasan

### Performance Analyzer di Power BI Desktop

Cara pakai:

1. Buka report di Desktop → panel **View → Performance Analyzer**.
2. Klik **Start recording**.
3. Klik **Refresh visuals** (mensimulasikan render pertama semua visual halaman aktif) — atau interaksikan slicer seperti user biasa.
4. Setiap visual tercatat dengan durasi per komponen (ms), bisa di-expand.

Tiga komponen yang dicatat:

| Komponen | Arti | Sumber masalah tipikal |
|---|---|---|
| **DAX Query** | Waktu engine menghitung (VertiPaq + Formula Engine) | Measure berat: iterator, TREATAS berulang, model besar |
| **Visual Display** | Waktu browser/client menggambar visual | Terlalu banyak visual, custom visual, elemen desain (shape) banyak |
| **Other** | Waktu tunggu lain (mis. menunggu query visual lain selesai) | Query antrean di halaman padat |

Pada visual pertama di daftar sering muncul **(waiting time)** — waktu menunggu giliran eksekusi, bukan waktu kerjanya sendiri.

### Membaca hasil: sort by duration

- Klik header kolom **Duration** → urutkan descending. Visual paling atas = prioritas perbaikan nomor satu (aturan 80/20: biasanya 2–3 visual menyumbang mayoritas waktu halaman).
- Bandingkan rasio DAX Query vs Visual Display:
  - **DAX Query dominan** → masalah di model/measure → bawa query ke DAX Studio.
  - **Visual Display dominan** → masalah di client: kurangi jumlah visual, custom visual, efek render.
- Tombol **Copy query** menyalin DAX query visual tersebut (lengkap dengan `DEFINE`/filter context) dan membukanya langsung di DAX Studio jika terpasang.

### DAX Studio — server timings, xmSQL, cache

Setup: buka DAX Studio → connect → pilih "Power BI Desktop – <nama model>" (instance Desktop yang sedang jalan terdeteksi otomatis).

Yang dipakai untuk diagnosis:

- **All Queries**: daftar semua query berjalan beserta durasinya; lihat bar chart distribusi durasi untuk menemukan outlier (query terpanjang).
- **Server Timings** (per query): total duration dipecah **FE vs SE** dengan bar horizontal — langsung terlihat engine mana yang sibuk. Angka **SE** besar = scan/filter di VertiPaq (masalah model); **FE** besar = logika DAX (iterator, konteks rumit).
- **xmSQL**: teks query SE — memperlihatkan scan mana yang mahal (join, filter kolom mana).
- **Query Plan**: logical + physical plan untuk kasus rumit (aktifkan di Options).
- **Cache — warm vs cold**: query yang dijalankan dua kali — eksekusi pertama *cold* (menembus engine penuh), kedua *warm* (kena cache). Selalu **Clear cache** (ada di toolbar) sebelum mengukur yang setara dengan pengalaman user pertama.
- **Benchmark**: jalankan query berulang kali dengan opsi warm/cold — ambil median dari beberapa run.

### Query Diagnostics (Power Query) — singkat

Untuk masalah **refresh** (bukan render): di Power Query editor, tab **Tools → Diagnose step** merekam durasi tiap langkah. Sangan relevan untuk report ini yang sumbernya `Folder.Files` ratusan Excel di Google Drive — langkah Combine/Expand biasanya biang keroknya. Untuk analisis refresh di level hasil akhir, gunakan refresh history di Power BI Service.

### Best practice benchmark

- Uji **cold cache** (clear cache / buka halaman dari halaman lain) untuk first-render, dan warm untuk interaksi rutin — laporkan keduanya.
- Ulangi 3–5 kali, ambil **median**; jangan menilai dari satu run.
- Matikan beban lain (scheduled refresh, aplikasi lain) saat mengukur.
- Ukur **sebelum** dan **sesudah** setiap perubahan, satu perubahan satu pengukuran — kalau banyak diubah bersamaan, tidak tahu mana yang berpengaruh.
- Angka Desktop ≠ angka Service; validasi halaman kritis di kapasitas target.

## Sintaks & Contoh

### Studi kasus nyata: halaman Summary Deviasi (48 visual)

Komposisi halaman (lihat [Pages & Visuals](../11-user-report-analysis/pages-and-visuals.md)): 17 shape, **11 slicer**, 9 KPI card, 5 bar/column chart, 1 waterfall, matrix, dan tombol navigasi.

Langkah diagnosis:

```text
1. View → Performance Analyzer → Start recording → Refresh visuals.
2. Sort by Duration descending.
3. Temuan tipikal (pola halaman seperti ini):
   - 9 KPI card = 9 query terpisah; masing-masing mengevaluasi
     [Deviasi CM]/[BOM CM] yang berisi TREATAS + ISFILTERED
     → DAX Query total jadi besar, padahal visualnya "kecil".
   - Waterfall dengan banyak kategori → Visual Display besar
     (client menggambar banyak segmen).
   - 11 slicer = 11 query tambahan hanya untuk mengisi daftar nilai.
   - 17 shape hampir gratis (Visual Display saja) — bukan prioritas.
4. Copy query KPI card terlama → DAX Studio → Server Timings:
   - SE dominan? → cek VertiPaq Analyzer (kolom boros) → rampingkan model.
   - FE dominan? → tulis ulang measure (VAR, kurangi iterator).
```

Contoh temuan klasik dan perbaikannya — iterator `FILTER` yang memindai tabel penuh:

```dax
-- KURVA BELAJAR: FILTER atas seluruh tabel = scan besar
CALCULATE (
    [Deviasi Value],
    FILTER ( 'Deviasi All', 'Deviasi All'[Jenis] = "BOM" )
)

-- LEBIH BAIK: filter kolom (predikat boolean) — engine push-down ke SE
CALCULATE (
    [Deviasi Value],
    'Deviasi All'[Jenis] = "BOM"
)
```

Contoh lain — measure keluarga `[Z Score]` (`SUMMARIZE` + `AVERAGEX` atas `ALLSELECTED`) menambah beban FE; jika hanya dipakai di 2 visual, pertimbangkan mengevaluasi sekali dengan VAR, atau memindahkan sebagian logika ke refresh (calculated table) bila frekuensinya tinggi.

### Format laporan hasil (template)

| Visual | Komponen | Sebelum (ms) | Aksi | Sesudah (ms) |
|---|---|---|---|---|
| Card `Deviasi CM` | DAX Query | 940 | Gabung KPI jadi 1 visual multi-row; rewrite measure | 310 |
| Waterfall susut | Visual Display | 1.850 | Kurangi kategori; ganti bar chart | 600 |
| Slicer `Nama Barang` | DAX Query | 420 | Ganti dropdown + Text Filter | 90 |

## Kesalahan Umum

- **Mengukur hanya dengan warm cache** — hasil terlalu optimis; user pertama membuka halaman selalu cold.
- **Hanya melihat total duration** — tanpa memecah DAX vs Visual Display, salah sasaran optimasi (DAX di-rewrite padahal masalahnya render).
- **Mengubah banyak hal sekaligus** — tidak bisa mengatribusikan perbaikan; satu perubahan satu pengukuran.
- **Copy query ke DAX Studio lupa clear cache** — query terlihat "cepat" karena cache, kesimpulan menyesatkan.
- **Optimasi DAX padahal masalahnya 48 visual per halaman** — jumlah visual adalah variabel terbesar; kurangi dulu, baru optimasi measure.
- **Menilai dari satu run** — selalu median dari beberapa pengukuran.

## Tips Pro

- Tetapkan **budget**: DAX Query per visual < 500 ms; first render halaman < 3 detik (jaringan kantor). Halaman yang lolos budget tidak perlu disentuh — fokus pada pelanggar.
- Prioritas halaman: yang paling padat visualnya dan dipakai harian (*Summary Deviasi*, *Detail Deviasi*).
- Simpan hasil Performance Analyzer (copy ke Excel) per versi report — bukti before/after sangat berguna saat rapat.
- Kombinasikan dengan [VertiPaq Analyzer](./01-vertipaq-engine.md): SE lambat hampir selalu berakar ke cardinality kolom, bukan ke DAX-nya.
- Setelah setiap optimasi, verifikasi angka bisnis tetap sama (guard: total `[Deviasi Value]` sebelum = sesudah) — performa boleh naik, angka tidak boleh berubah.

## Terkait

- [VertiPaq Engine](./01-vertipaq-engine.md) — akar masalah sisi Storage Engine
- [Optimization Checklist](./03-optimization-checklist.md) — aksi lanjutan hasil diagnosis
- [Slicer Guide](../06-filters-and-slicers/02-slicer-guide.md) — 11 slicer per halaman dan biayanya
- [Filter Types](../06-filters-and-slicers/01-filter-types.md) — mode Apply untuk DirectQuery/model besar
- [Report Overview](../11-user-report-analysis/REPORT_OVERVIEW.md) — konteks 36 halaman & 539 visual
