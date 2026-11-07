# Fitur Power BI Service yang Wajib Dikuasai

> Power BI Service bukan sekadar "tempat parkir report" — ia punya fitur monitoring, governance, distribusi, dan analisis yang membuat report Cost Control bisa dioperasikan seperti produk. Materi ini memetakan fitur service yang paling relevan untuk report 36 halaman yang dikonsumsi manajemen AM/PIC per area.

## Konsep & Penjelasan

### Semantic model vs Report vs Dashboard — bedanya!
| Objek | Isi | Diedit oleh | Catatan |
|---|---|---|---|
| **Semantic model** (dulu *dataset*) | Tabel, relasi, measures (161 buah di report ini), RLS, koneksi | Pengembang di Desktop | Satu model bisa dipakai banyak report |
| **Report** | Halaman, visual, filter (36 halaman, 539 visual) | Pengembang di Desktop/service | Ter-binding ke satu semantic model |
| **Dashboard** | Kumpulan *tile* pin dari report (1 halaman saja) | Siapa pun dengan akses | Khusus monitoring ringkas; **alert hanya ada di sini** |

Kesalahpahaman klasik: "dashboard" dipakai orang sebagai istilah umum untuk report — di Power BI keduanya objek berbeda dengan kemampuan berbeda.

### Usage metrics
- Buka report/App > `…` > **Usage metrics**: views, viewers, platform (web/mobile), periode ±30 hari terakhir.
- Kegunaan untuk report ini: tahu halaman mana yang jarang dibuka → kandidat disederhanakan/dihapus dari navigasi App; ukur adopsi per area (AM vs PIC).
- Versi *improved usage metrics* memberi tambahan distribution method (link/app/share).

### Monitor semantic model & lineage view
- **Refresh history**: riwayat sukses/gagal + pesan error (lihat [Gateway & Refresh](./02-gateway-and-refresh.md)).
- **Lineage view**: peta visual dependensi — data source → semantic model → report/dashboard. Satu pandangan menjawab "jika model ini kuhapus, apa yang mati?"
- **Impact analysis**: dari lineage/kartu item, lihat berapa downstream item yang terdampak sebelum rename/hapus measure atau kolom. Wajib dilakukan sebelum refactor besar (161 measures punya banyak dependen).

### Endorsement: Promote & Certified
- **Promoted** (promosi oleh pemilik): "ini versi resmi menurut tim pembuat".
- **Certified** (badge biru): diverifikasi oleh *certifier* yang ditunjuk tenant admin — standar tertinggi kepercayaan.
- Untuk report cost control yang jadi acuan rapat direksi: minimal Promoted; idealnya Certified supaya tidak ada versi liar beredar.

### Sensitivity labels (Microsoft Purview)
- Label klasifikasi (Public / Internal / Confidential …) yang menempel pada item; bisa membatasi export, sharing ke luar, dan menambah watermark saat di-export.
- Aktifkan `Confidential – Cost Control` pada model & report; cek tenant admin untuk kebijakan wajib.

### Q&A natural language
- Kotak "Ask a question about your data" di service/visual Q&A: pengguna mengetik pertanyaan → visual dibuat otomatis.
- Parser Q&A terutama English; untuk istilah bahasa Indonesia gunakan **synonyms** di model view dan sembunyikan kolom teknis.
- Manfaat nyata: AM bisa bertanya "top 10 deviasi by area" tanpa belajar navigasi 36 halaman.

### Analyze in Excel & analyze feature
- **Analyze in Excel**: pivot Excel live ke semantic model (koneksi MDX); butuh *build permission*; RLS tetap berlaku. Catatan: fitur client MDX klasik tidak menerapkan calculation groups — angka bisa tampil mentah (lihat [Calculation Groups](../10-advanced/01-calculation-groups.md)).
- Dari visual juga tersedia opsi analyze/spotlight ringan di beberapa tipe visual; untuk "kenapa naik?" gunakan drillthrough halaman detail (report ini punya halaman Detail Deviasi untuk itu).

### Export report
- **File > Export > PDF/PPTX** untuk report; Excel/PNG/CSV untuk visual tertentu.
- RLS dan sensitivity label tetap berlaku; visual berat dapat membuat export sebagian gagal — sederhanakan dulu.

### Subscriptions (email dengan snapshot PDF)
- Buat **Subscribe to report/app**: email berisi snapshot image/PDF + link, terjadwal (harian/mingguan, atau *after data refresh*).
- Pasca-refresh adalah favorit: direksi menerima PDF ringkasan pukul 05.00 tanpa membuka app.
- Snapshot = gambar statis; interaktif tetap harus lewat link.

### Alerts pada dashboard tiles
- Alert hanya bisa dibuat di **Dashboard** (tile card/gauge/KPI dengan nilai numerik) → set ambang (mis. `% BOM` melewati batas) → email + push notification di mobile app.
- Untuk report (bukan dashboard), alternatif: pin tile penting ke dashboard khusus monitoring, atau gunakan Power Automate + data alerts.

### Performance monitoring service
- Sumber diagnosis: usage metrics, refresh history, dan di sisi Fabric — *query insights*.
- Di sisi author: Performance Analyzer di Desktop + copy query ke [DAX Query View](../10-advanced/05-dax-query-view.md) untuk mengukur measure.
- Gejala umum report besar (539 visual, model 145 MB): halaman Summary Deviasi (48 visual) lambat — pecah ke halaman lebih kecil / kurangi visual per halaman.

### Mobile app & mobile layout
- Setiap halaman bisa punya **mobile layout** sendiri (View > Mobile layout di Desktop); tanpa layout, Power BI tetap merender otomatis — hasilnya sering sempit.
- Power BI mobile app mendukung alert push, subscription, dan RLS — AM cek deviasi area-nya dari ponsel.

### Publish to web — RISIKO publik!
- **Publish to web** membuat URL publik anonim: siapa pun dengan link bisa melihat, tanpa login.
- **RLS dan sensitivity label tidak berlaku** pada embed ini; jangan pernah dipakai untuk data biaya/resto internal.
- Tenant admin sebaiknya mematikan publish to web; untuk embed aman gunakan **Embed in SharePoint Online / Teams** (secure, RLS jalan) atau REST API embed bagi developer.

### Workspace OneDrive sync
- File Excel/CSV yang disimpan di OneDrive dan dihubungkan ke workspace akan otomatis muncul sebagai item & dataset yang **tersinkron ±tiap jam** tanpa gateway — jalur paling mulus untuk file cost control kecil (lihat rekomendasi migrasi di [Gateway & Refresh](./02-gateway-and-refresh.md)).

## Sintaks & Contoh

Rencana pemakaian fitur untuk report Cost Control:

```text
Saat rilis  : Endorse "Promoted" + label Confidential
Distribusi  : App + audience per area (AM/PIC/Direksi)
Monitoring  : Dashboard "CC Monitor" (tile %BOM, Deviasi CM) + alert ambang
Notifikasi  : Subscription after-refresh 05.00 (PDF) ke direksi
Mobile      : Mobile layout untuk halaman Summary & by Area
Governance  : Lineage + impact analysis sebelum refactor measures
Ukuran      : Usage metrics bulanan → rapikan 36 halaman
```

## Kesalahan Umum
- **Publish to web dipakai karena "praktis"** → data internal tersebar publik; perbaikan: embed ke Teams/SharePoint, matikan fitur di tenant.
- **Mengira alert bisa di report** → tidak; alert hanya di dashboard tile; perbaikan: buat dashboard monitoring khusus.
- **Semua halaman dipaksa masuk navigasi App** → manajemen kehilangan arah di 36 halaman; perbaikan: kurasi 6–8 halaman utama.
- **Q&A dibiarkan default** → pertanyaan bahasa Indonesia menghasilkan visual aneh; perbaikan: atur synonyms & hidden fields.
- **Export PDF report super berat gagal** → perbaikan: kurangi visual per halaman dulu, atau bagikan subscription PDF.
- **Certified dianggap sekadar label ganti nama** → tanpa proses verifikasi, sertifikasi kehilangan makna; perbaikan: susun checklist QA (angka vs sumber, refresh, RLS) sebelum minta endorse.

## Tips Pro
- Pasang "Last Update Dev" (report ini sudah punya) di halaman Home — cara paling murah menjawab "data ini sampai kapan?".
- Gunakan usage metrics sebagai alat negosiasi: halaman yang 0 views selama 3 bulan adalah kandidat pensiun.
- Subscription PDF ke direksi mengurangi permintaan ad-hoc "kirimkan laporannya" — otomatis tiap pagi.
- Lineage view dicetak/screenshot masuk dokumentasi repo — peta dependensi hidup.
- Untuk AM di lapangan: uji mobile layout di layar kecil betulan (bukan hanya di Desktop preview).

## Terkait
- [Workspaces](./01-workspaces.md)
- [Gateway & Refresh](./02-gateway-and-refresh.md)
- [Deployment Pipelines](./03-deployment-pipelines.md)
- [Git Integration](./05-git-integration.md)
- [DAX Query View](../10-advanced/05-dax-query-view.md)
- [Errors Umum](../12-troubleshooting/common-errors.md)
