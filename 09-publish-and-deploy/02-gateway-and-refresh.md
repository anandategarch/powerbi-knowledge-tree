# Gateway & Refresh: Menjaga Data Tetap Mutakhir

> Scheduled refresh membuat semantic model di Power BI Service selalu terbaru tanpa buka Desktop; on-premises data gateway menjembatani sumber lokal ke cloud. Karena sumber report Cost Control berupa folder Excel di Google Drive lokal (`H:\My Drive`) + Google Sheets + tabel ERP, topik ini adalah titik paling rawan dari seluruh arsitektur — dan motivasi utama migrasi ke cloud.

## Konsep & Penjelasan

### Mode penyimpanan menentukan kebutuhan refresh
- **Import** (seperti model 145 MB milik report ini): data disalin ke dalam model → **wajib refresh** berkala; kecepatan visual tercepat.
- **DirectQuery**: data tidak disalin; visual mem-query sumber real-time → tidak perlu refresh dataset, tapi lambat dan beban di sumber.
- **Live connection** (ke Analysis Services/semantic model lain): tidak ada model sendiri, tidak ada refresh.
- **Composite/hybrid**: kombinasi (lihat [Incremental Refresh](../10-advanced/03-incremental-refresh.md)).

### Personal vs Standard (On-premises data gateway)
- **Personal mode**: terpasang per-user, hanya 1 orang, tidak bisa cluster, tidak mendukung DirectQuery/live, dan fiturnya terbatas — gunakan hanya sementara.
- **Standard (On-premises data gateway)**: satu gateway dipakai banyak orang, bisa **cluster** (HA + load balancing), mendukung DirectQuery, dan dikelola lewat service account. Ini yang wajib dipakai tim.
- Gateway mengambil data dengan *pull* dari cloud: komputer gateway harus online saat jadwal refresh berjalan.

### Kenapa arsitektur saat ini rawan (studi kasus report ini)
Sumber data report:
1. **Folder Excel Google Drive** via `Folder.Files("H:\My Drive\COST CONTROL\…")` — drive lokal hasil sinkronisasi Google Drive = data source *on-premises* → butuh gateway **dan** mesin dengan Google Drive sync yang selalu online.
2. **Google Sheets** master `Inventory` via `Web.Contents("https://docs.google.com/spreadsheets/d/e/…")` (published URL).
3. **Tabel purchasing** (`9901`, `3243`, dll.) hasil export ERP.

Risiko nyata: laptop mati / tutup / log off → refresh gagal; file dibuka orang (lock `~$`); published URL Google Sheets berubah saat di-publish ulang. Karena itu:

> **Rekomendasi migrasi**: pindahkan Excel ke OneDrive/SharePoint (connector native di service, refresh tanpa gateway), ganti `Web.Contents(published URL)` dengan connector **Google Sheets** resmi (cloud, OAuth), dan tarik tabel ERP langsung dari database (bukan export Excel).

### Google Drive/Sheets connector di service
- **Google Sheets connector** resmi: terdaftar sebagai sumber *cloud* → refresh di service **tanpa gateway**; kredensial OAuth2 disimpan di dataset settings; re-login berkala saat token kedaluwarsa.
- **Google Drive folder**: tidak ada connector "folder" cloud — `Folder.Files` di drive lokal selalu dianggap on-premises → butuh gateway; atau migrasi ke SharePoint/OneDrive.

### Refresh types
- **Full refresh**: seluruh tabel dibaca ulang tiap jadwal (default).
- **Incremental refresh**: hanya partisi periode terbaru yang dibaca (butuh policy — lihat [Incremental Refresh](../10-advanced/03-incremental-refresh.md)).
- **Hybrid**: kombinasi DirectQuery (hari berjalan) + import (historis) untuk near-real-time.
- **On-demand**: tombol *Refresh now* di service; berguna setelah ganti kredensial atau uji coba.
- **Via API/XMLA**: refresh dipicu program (mis. setelah ETL/extract ERP selesai) — cara paling tepat untuk report ERP.

### Scheduled refresh: setup & limit
- Dataset settings → **Refresh** → on, pilih zona waktu, frekuensi (harian/mingguan), jam.
- Limit: shared capacity **Pro ± 8×/hari**; Premium/Premium Per User/Fabric ± 48×/hari; durasi per refresh di shared sekitar 2 jam.
- **Best practice window**: refresh sebelum jam kerja (mis. 04.30–05.30) dan/atau setelah SLA upload file oleh PIC — jangan siang ketika file sedang diedit.
- Untuk data bulanan (cost control), tambahkan jadwal khusus di tanggal closing (mis. tgl 1 jam 06.00).

### Data source credentials & gateway binding
- Dataset settings → **Data source credentials**: edit kredensial, pilih gateway (cluster), atur privacy level.
- **Privacy levels**:
  - *Private*: data sensitif; tidak boleh digabungkan bebas dengan sumber lain.
  - *Organizational*: semua di organisasi melihat data itu.
  - *Public*: seperti halaman web publik.
- Kombinasi Private + Public dalam satu query dapat memicu error `Formula.Firewall` (data sources can't be combined) → samakan level (mis. semua *Organizational*) atau pisahkan query lalu merge di model.

### Refresh notification & failures
- Aktifkan **email notification on failure** di dataset settings (juga di account settings > notifications) — kirim ke tim, bukan personal.
- Cek **Refresh history** (riwayat terbaru ±30 hari): kolom pesan error menunjukkan sebab (credentials, gateway offline, file locked, dll).
- Pola umum kegagalan & artinya:
  - `invalid credentials` → login ulang di data source credentials.
  - `gateway offline / unable to connect` → mesin gateway mati/bermasalah.
  - `file is locked / ~$` → Excel sedang dibuka orang.
  - `key not found / couldn't find folder` → `H:\My Drive` tidak tersinkron/mesin log off.

## Sintaks & Contoh

Pattern Power Query yang tahan refresh (dari report Cost Control):

```m
let
    Source = Folder.Files("H:\My Drive\COST CONTROL\Deviasi"),
    // Buang file temp Excel yang terkunci (~$) — penyebab gagal refresh klasik
    NoTemp = Table.SelectRows(Source, each
        not Text.StartsWith([Name], "~$")
        and not Text.StartsWith([Name], ".")),
    // Ambil tanggal dari konvensi nama file YYYY_MM_...
    AddPeriode = Table.AddColumn(NoTemp, "Periode",
        each #date(
            Number.FromText(Text.Split([Name], "_"){0}),
            Number.FromText(Text.Split([Name], "_"){1}), 1)),
    ...
in ...
```

Konfigurasi jadwal contoh:

```text
Dataset  : CC-CostControl
Gateway  : GW-Office-01 (cluster 2 node)
Jadwal   : 04.30 WIB harian  |  06.00 WIB tanggal 1 (closing bulanan)
Notif    : email ke bi-team@company.com saat gagal
Privacy  : semua sumber = Organizational
```

## Kesalahan Umum
- **Mesin/laptop gateway ikut tidur (sleep) atau log off** → refresh gagal berurutan; perbaikan: gateway di mini PC/VM always-on, Windows auto-login, power plan never sleep.
- **File Excel dibuka orang saat refresh** → `file locked`; perbaikan: filter `~$` di PQ + disiplin kerja (jangan edit file sumber jam refresh).
- **Kredensial Google/Sheets kedaluwarsa** → perbaikan: re-auth di data source credentials; pantau email notifikasi.
- **Published URL Google Sheets berubah setelah re-publish** → `Web.Contents` 404; perbaikan: pakai connector Google Sheets resmi dengan OAuth (URL tidak berubah).
- **Privacy level campur aduk** → `Formula.Firewall`; perbaikan: samakan *Organizational* atau pisahkan query.
- **Refresh penuh tiap bulan makin lama** (model makin besar, file makin banyak) → perbaikan: aktifkan incremental refresh; rapikan kolom yang tidak dipakai.
- **Jadwal pas jam orang upload file** → data setengah bulan; perbaikan: window setelah SLA upload (mis. PIC wajib upload sebelum jam 4 subuh).
- **Menyalakan refresh 8×/hari habis untuk data yang hanya berubah malam** → sia-sia kuota; perbaikan: 1–2× cukup, selebihnya on-demand.

## Tips Pro
- Migrasi bertahap tanpa big-bang: (1) Excel → OneDrive/SharePoint; (2) Sheets → connector resmi; (3) ERP → view SQL dengan query folding. Setelah itu gateway hanya untuk data yang benar-benar lokal.
- Simpan kredensial dengan **service account** bersama (bukan email pribadi), supaya tidak hilang saat orang resign.
- Pasang dua gateway dalam satu cluster + load balancing aktif — refresh besar (145 MB) tidak menyumbat satu node.
- Picu refresh via API/XMLA **setelah** ETL ERP selesai (bukan jam fixed) — data selalu segar tanpa menunggu.
- Tambahkan halaman "Last Update Dev" (report ini sudah punya di Home) yang menampilkan `MAX(Date)` supaya manajemen tahu data sampai kapan.

## Terkait
- [Workspaces](./01-workspaces.md)
- [Deployment Pipelines](./03-deployment-pipelines.md)
- [Incremental Refresh](../10-advanced/03-incremental-refresh.md)
- [Errors Umum](../12-troubleshooting/common-errors.md)
- [FAQ](../12-troubleshooting/faq.md)
- [Analisis Report User](../11-user-report-analysis/REPORT_OVERVIEW.md)
