# FAQ Power BI: Pertanyaan Nyata, Jawaban Langsung

> Kumpulan pertanyaan yang paling sering diajukan pemilik dan pengguna report Cost Control — dijawab langsung tanpa teori berlebih. Diperuntukkan bagi penulis report maupun konsumen (AM/PIC) yang penasaran "kenapa angkanya begitu".

## Konsep & Penjelasan — Tanya Jawab

### Q1. Kenapa total persen tidak 100%?
Baris yang kosong (BLANK) dihilangkan engine dari denominator; pembulatan tampilan (dua baris 33,3% + 33,3% + 33,3% = 99,9%) menambah kecurigaan. Solusi: hitung persentase sebagai **measure** (totals akan dihitung ulang oleh engine, bukan di-sum), dan tampilkan satu desimal lebih bila perlu. Jangan pernah menjumlah baris persen di Excel — pasti meleset.

### Q2. Kapan pakai measure vs calculated column?
- **Measure**: hasil dihitung saat visual meminta (filter context) — untuk semua angka yang tampil di report (deviasi, % BOM, growth). Ukuran model tetap ramping.
- **Calculated column**: dihitung sekali saat refresh, disimpan di model — hanya untuk key/join/flag statis (mis. kolom bulan dari tanggal, kategori item).
- Aturan cepat: kalau nilainya tergantung filter/slicer → measure. Kalau nilainya tetap per baris → kolom (dan cek apakah bisa dikerjakan di Power Query — lebih baik lagi).

### Q3. Kenapa measure saya lambat?
Urutan tersangka: (1) iterator `FILTER(..., <kolom>)` non-foldable → ganti filter boolean; (2) `ALL` terlalu besar / `FILTER(ALL(table))` pada tabel fact besar; (3) relasi many-to-many ganda arah; (4) `VALUES`/`SUMMARIZE` di dalam iterator; (5) visual meminta measure berat-baris (matrix 500 baris × measure berat). Diagnosis: Performance Analyzer → copy query → uji di [DAX Query View](../10-advanced/05-dax-query-view.md); profil mendalam di DAX Studio.

### Q4. Bisa ambil data dari Google Drive?
**Bisa — report ini membuktikannya**: folder Excel di Drive lokal (`H:\My Drive\COST CONTROL\…` lewat `Folder.Files`) dan master `Inventory` dari Google Sheets yang di-publish (`Web.Contents`). Catatannya di sisi refresh: sumber lokal butuh gateway + mesin selalu online, dan published URL bisa berubah. Untuk jangka panjang: pindah Excel ke OneDrive/SharePoint dan pakai connector Google Sheets resmi — detail di [Gateway & Refresh](../09-publish-and-deploy/02-gateway-and-refresh.md).

### Q5. Kenapa PREVIOUSMONTH kosong di level tahun?
`PREVIOUSMONTH` butuh konteks **bulan**; di level tahun ia mengembalikan BLANK (tidak ada "bulan sebelumnya" dari satu tahun). Di level tahun gunakan `SAMEPERIODLASTYEAR` / `PARALLELPERIOD(...,-1,YEAR)`. Syarat umum: tabel `Date` harus ditandai **Mark as date table** dan berisi tanggal kontinu.

### Q6. Kenapa slicer tidak tersinkron antar halaman?
Sync slicers hanya menyambungkan slicer pada **field yang sama**. Dengan 292 slicer di report ini, mudah ada satu yang mengacu ke field beda (mis. `Date[Bulan]` vs `Date[Month Number]`). Perbaikan: satu halaman master slicer → View > Sync slicers → pastikan grup field identik; pertimbangkan mengkonsolidasi dimensi dengan [Field Parameters](../10-advanced/02-field-parameters.md) agar jumlah slicer turun drastis.

### Q7. Gimana bikin Top 10 dinamis?
Pola measure ranking (bisa diganti 5/10/20 tanpa mengubah model):

```dax
Rank Dev = RANKX(ALL(Inventory[(Old) Nama Barang]), [Deviasi CM], , DESC)
-- filter visual: [Rank Dev] <= [N Value]  (N = what-if parameter)
```
Alternatif tanpa DAX: filter Top-N bawaan visual (Type: Top N by value) — tapi tidak bisa dipakai lintas visual.

### Q8. Kenapa angka beda dengan Excel?
Periksa berurutan: (1) **filter context** — slicer/periode aktif di Power BI vs filter manual Excel; (2) **join fan-out** setelah merge — baris dobel di Excel tidak; (3) **BLANK handling** — Power BI membuang baris kosong dari rata-rata; (4) **locale tanggal** — teks `dd/mm` terbaca beda; (5) **pembulatan tampilan** vs nilai penuh. Verifikasi dengan query audit di DAX Query View, bukan berdebat screenshot.

### Q9. Bisakah Power BI handle file 1 GB?
Ya, tapi beda-beda konteksnya: file `.pbix` **hasil** di shared Pro dibatasi ±1 GB per dataset (Premium/Fabric jauh lebih besar) — dan kompresi VertiPaq membuat 1 GB model bisa berasal dari source ber-GB. Model 145 MB seperti report ini masih nyaman. Kunci: buang kolom tak terpakai di PQ, matikan auto date/time, dan pakai [Incremental Refresh](../10-advanced/03-incremental-refresh.md) saat tumbuh.

### Q10. Kenapa growth negatif jadi positif?
Klasik: pembanding negatif. `-5%` dibanding `-10%` secara matematis **naik** (+100%) padahal secara bisnis tetap buruk. Perbaikan ala report ini: pakai **ABS pada denominator**:

```dax
Growth = DIVIDE([Selisih], ABS([Deviasi LM]))
```
Dengan ABS: pembanding -10% dihitung magnitudonya 10%, sehingga perbaikan dari -10% ke -5% tampil +50% (wajar), bukan +100% menyesatkan.

### Q11. Gimana backup pbix otomatis?
Tiga lapis: (1) **Version history di service** untuk report (±30 hari) — otomatis; (2) **PBIP + Git** untuk model+report — backup penuh dan bisa di-rollback ke commit mana pun (cara terbaik, lihat [Git Integration](../09-publish-and-deploy/05-git-integration.md)); (3) salin file secara terjadwal:

```powershell
$stamp = Get-Date -Format "yyyyMMdd_HHmm"
Copy-Item "H:\My Drive\COST CONTROL\report.pbip" `
    "H:\Backup\report_$stamp.pbip" -Force
```
Jangan andalkan satu file `report_final_FIX2.pbix` di flashdisk.

### Q12. Kenapa visual hilang setelah refresh?
Biasanya bukan "hilang", melainkan: (1) baris baru tidak match relasi (key kotor) → data tak terlihat; (2) pattern `[Show]`/filter "not blank" menghapus baris yang kini kosong; (3) RLS member batasi (user lain melihat setnya sendiri); (4) file sumber belum diupload PIC saat refresh jalan. Cek Refresh history + halaman QA, jangan langsung rebuild visual.

### Q13. Kapan pakai Import vs DirectQuery?
- **Import** (default, seperti report ini): tercepat, semua fitur DAX, tapi butuh refresh terjadwal.
- **DirectQuery**: data selalu mutakhir tanpa refresh dataset; cocok ERP transaksional yang butuh near-real-time — tapi DAX terbatas (beberapa fungsi tidak didukung), lambat, beban di sumber.
- Aturan: mulai Import; pindah ke DQ/composite hanya jika kebutuhan real-time atau volume benar-benar menghendaki.

### Q14. Cara kerja Z-score untuk anomali?
Z-score = seberapa jauh satu resto menyimpang dari rata-rata grup, dalam satuan simpangan baku:

```dax
Mean Waste = AVERAGEX(ALL(Resto[Resto]), [Waste CM])
StdDev  = STDEVX.P(ALL(Resto[Resto]), [Waste CM])
Z Score = DIVIDE([Waste CM] - [Mean Waste], [StdDev])
Kategori Anomali = SWITCH(TRUE(), ABS([Z Score]) > 2, "Anomali", "Normal")
```
Interpretasi: `|Z| > 2` = ±2 simpangan baku dari rata-rata grup — kandidat investigasi (pola yang dipakai halaman Analisa Susut untuk menandai resto/jenis waste anomali).

### Q15. Cara publish ke mobile?
Publish ke workspace → **Update App** (audiens AM/PIC) → pengguna buka lewat Power BI mobile app. Sisi penulis: aktifkan **Mobile layout** per halaman (View > Mobile layout), susun ulang visual untuk layar sempit, dan uji di ponsel betulan. Subscription/alert juga muncul di mobile.

### Q16. Apa itu PBIR?
**PBIR** = format definisi report berbasis JSON (folder `.Report/definition/` — `report.json`, `pages/page.json`, `visual.json` per visual). Versi *enhanced* (yang dipakai report ini) menyimpan **satu file per visual** sehingga cocok untuk Git diff/merge — fondasi dari [Git Integration](../09-publish-and-deploy/05-git-integration.md). Lawannya adalah `.pbix` biner tunggal yang tidak bisa di-diff.

## Sintaks & Contoh

Query QA andalan untuk menjawab "benar tidak angkanya?" (lihat [DAX Query View](../10-advanced/05-dax-query-view.md)):

```dax
EVALUATE
SUMMARIZECOLUMNS(
    'Resto'[Area], 'Attribute'[Attribute],
    "Dev CM", [Deviasi CM],
    "Growth", DIVIDE([Deviasi CM] - [Deviasi LM], ABS([Deviasi LM]))
)
ORDER BY 'Resto'[Area]
```

## Kesalahan Umum — mitos vs fakta
- **"Total harus hasil jumlah baris"** → faktanya total measure dihitung ulang engine pada konteks total; jumlah baris manual sering salah (lihat Q1).
- **"Angka Power BI salah, Excel benar"** → hampir selalu beda filter/join/locale; selesaikan dengan query audit (Q8), bukan dendam antar tool.
- **"Growth negatif berarti turun"** → dengan denominator negatif tanda menipu; pakai ABS (Q10).
- **"Backup = copy .pbix manual"** → lupa terus; otomatislah (Q11).
- **"Semua halaman harus masuk navigasi"** → 36 halaman tanpa kurasi justru membingungkan manajemen.

## Tips Pro
- Jadikan FAQ ini halaman hidden `Help` di dalam App — pengguna menjawab pertanyaannya sendiri sebelum menghubungi Anda.
- Tambahkan pertanyaan baru setiap kali ada pertanyaan berulang dari AM/PIC — FAQ hidup, bukan artefak sekali tulis.
- Pasangkan tiap jawaban dengan link artikel terkait (knowledge tree) — struktur ini sudah melakukannya.
- Saat demo ke manajemen, tunjukkan Q10 (growth ABS) — ini yang paling sering memicu debat angka.

## Terkait
- [Errors Umum](./common-errors.md)
- [Gateway & Refresh](../09-publish-and-deploy/02-gateway-and-refresh.md)
- [Calculation Groups](../10-advanced/01-calculation-groups.md)
- [Field Parameters](../10-advanced/02-field-parameters.md)
- [Incremental Refresh](../10-advanced/03-incremental-refresh.md)
- [Git Integration](../09-publish-and-deploy/05-git-integration.md)
- [Analisis Report User](../11-user-report-analysis/REPORT_OVERVIEW.md)
