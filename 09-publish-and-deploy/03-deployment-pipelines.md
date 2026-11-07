# Deployment Pipelines: Promosi Terkontrol DEV → TEST → PROD

> Deployment pipeline adalah fitur Pro/Premium yang memindahkan (deploy) report + semantic model antar stage secara terkontrol, lengkap dengan aturan penggantian koneksi data per stage. Bagi pemilik report Cost Control yang dilihat manajemen, pipeline memberi disiplin: PROD hanya boleh berisi hasil deploy yang sudah diuji — bukan hasil edit langsung.

## Konsep & Penjelasan

### Struktur stage
- Pipeline berisi 3 stage default — **DEV → TEST → PROD** — dan dapat ditambah hingga 10 stage bila perlu.
- Setiap stage di-**assign** ke satu workspace (buat dulu workspace-nya: [Workspaces](./01-workspaces.md)); idealnya ketiganya sekapasitas (Premium/PPU/Fabric) agar bisa deploy lintas stage.
- Nama workspace sebaiknya seragam (`CC [DEV]`, `CC [TEST]`, `CC [PROD]`) — pipeline otomatis menandai mana yang sudah ter-assign.

### Apa yang terjadi saat deploy
- **Semantic model**: pipeline menyalin *definisi/metadata* (skema, measures, relasi, RLS) ke stage target. Untuk mode import, **data tidak ikut tersalin** — jalankan refresh di stage target untuk mengisi data.
- **Report**: salinan baru dibuat di target dan **otomatis di-binding (auto-binding)** ke semantic model stage target — bukan ke model di DEV. Inilah jaminan angka TEST tidak bocor dari model DEV.
- Item lain (datalflow, paginated report) ikut jika didukung.

### Data source rules & parameter rules — kunci koneksi dev→prod
Tanpa rules, model di PROD akan membaca **sumber yang sama dengan DEV** — angka uji bisa tampil di depan manajemen. Aturan yang tersedia:
- **Data source rule**: ubah koneksi per stage, mis. `Folder.Files("H:\My Drive\COST CONTROL\DEV")` di DEV, `"…\PROD"` di PROD, atau `Server/Database` SQL yang berbeda.
- **Parameter rule**: jika model memakai PQ parameter (mis. `Env`, `PathSource`), nilainya bisa diganti per stage — cara paling bersih untuk report ini.

### Selective deployment (filter)
- Tombol Deploy punya opsi **Select items to deploy**: pilih hanya semantic model, hanya report, atau item tertentu.
- Ada petunjuk *related items* (report menyangkut model tertentu) — jangan deploy report tanpa model tujuannya.
- Gunakan selective saat hotfix satu halaman: jangan repot deploy semuanya.

### Staging data (data uji yang realistis)
- Stage TEST sebaiknya memakai **data produksi-like**: kalau DEV pakai file dummy, TEST harus sudah pakai salinan data real (folder PROD read-only atau database test berisi data asli).
- Setelah deploy ke TEST/PROD: refresh dataset, cek **refresh history**, lalu bandingkan angka TEST vs PROD (pakai [DAX Query View](../10-advanced/05-dax-query-view.md)) sebelum Update app.

### Solusi tanpa Pro/Premium
Jika kapasitas Premium/PPU/Fabric belum tersedia, pola manual yang tetap terkontrol:
1. **Parameter file**: satu file master `.pbip` dengan PQ parameter `Env` (`DEV`/`TEST`/`PROD`) yang mengubah path/kanal sumber.
2. **Save As / Save a copy** ke workspace target, ganti nilai parameter, publish, verifikasi koneksi di dataset settings.
3. **ALM Toolkit** (gratis) untuk diff skema model antar environment sebelum publish.
4. **Version control alternatif**: PBIP + Git branch per perubahan (lihat [Git Integration](./05-git-integration.md)) — branch `main` selalu dicerminkan ke PROD.
Kekurangan pola manual: bergantung disiplin manusia (lupa ganti parameter = kejadian klasik DEV terpublish ke PROD).

### Testing strategy
- **Test page hidden**: buat halaman `QA Checks` (hidden) berisi tabel angka kunci vs sumber Excel; tidak masuk navigasi App.
- **Test koneksi**: setelah deploy, cek *refresh history* & lineage (lihat [Fitur Service](./04-power-bi-service-features.md)) — pastikan model PROD benar-benar menunjuk sumber PROD.
- **Compare stage**: gunakan tampilan compare pipeline (item-level) sebelum menekan Deploy.
- Uji di perangkat manajemen: buka App di HP (mobile layout), karena AM sering cek dari ponsel.

### Audit & monitoring deployment
- Pipeline mencatat **who deployed what, when** — jejak ini pengganti "siapa tadi mengubah grafik?".
- Kombinasikan dengan usage metrics App untuk melihat adopsi pasca rilis.
- Simpan catatan rilis di repo (changelog.md) — semangatnya sama dengan dokumentasi knowledge tree ini.

## Sintaks & Contoh

Contoh rules untuk report Cost Control:

```text
Pipeline: CC Pipeline
├─ DEV   workspace: CC [DEV]
│    sumber: Folder "H:\My Drive\COST CONTROL\DEV"
│    parameter: Env="DEV", ClosingDay=1
├─ TEST  workspace: CC [TEST]
│    sumber: Folder "H:\My Drive\COST CONTROL\TEST"  (salinan realistis)
│    parameter: Env="TEST"
└─ PROD  workspace: CC [PROD]
     sumber: Folder "H:\My Drive\COST CONTROL\PROD"
     parameter: Env="PROD"

Rules yang diset:
• Data source rule (Folder.Files) : DEV/TEST/PROD berbeda path
• Parameter rule (Env, PathSource) : nilai per stage
• Setelah deploy PROD → refresh → Update app → changelog
```

Checklist sebelum menekan Deploy ke PROD:

```text
☐ Compare stage: tidak ada perubahan tak terduga
☐ QA Checks page: angka = sumber Excel
☐ Data source rules PROD sudah benar (bukan DEV!)
☐ Refresh PROD sukses, refresh history bersih
☐ Update App + kabari audiens (AM/PIC)
```

## Kesalahan Umum
- **Tidak mengatur data source rules** → PROD membaca file DEV; angka dummy tampil ke direksi; perbaikan: set rules + verifikasi lineage setelah deploy.
- **Deploy lalu lupa refresh** → visual blank/error; perbaikan: refresh adalah bagian dari definisi "deploy selesai".
- **Mengedit langsung di TEST/PROD** → perubahan local tertimpa saat deploy berikutnya; perbaikan: semua edit di DEV.
- **Rename item di satu stage** → pipeline menganggap item lama hilang + item baru muncul (duplikat); perbaikan: rename sekaligus di DEV lalu deploy ulang.
- **Memakai sumber data sama persis untuk semua stage** → testing jadi tidak bermakna; perbaikan: data TEST = salinan realistis.
- **Deploy Jumat sore buru-buru** → tidak ada waktu rollback; perbaikan: jadwal deploy tetap (mis. Selasa pagi) + tag versi.

## Tips Pro
- **Tim kecil (bahkan 1 orang) tetap untung**: pipeline memaksa kebiasaan "PROD harus dari deploy" — proteksi dari diri sendiri.
- Ikon + warna stage di pipeline membantu; jangan pakai nama samar.
- Kombinasikan dengan Git: workspace DEV di-connect ke Git (single source of truth), pipeline hanya mempromosikan yang sudah di-commit.
- Simpan tag rilis (v1.2, v1.3) di repo setiap deploy PROD — rollback jadi mudah.
- Jadikan "Update App" langkah resmi terakhir; App yang stale = komplain pertama dari manajemen.

## Terkait
- [Workspaces](./01-workspaces.md)
- [Gateway & Refresh](./02-gateway-and-refresh.md)
- [Git Integration](./05-git-integration.md)
- [DAX Query View](../10-advanced/05-dax-query-view.md)
- [Incremental Refresh](../10-advanced/03-incremental-refresh.md)
- [Analisis Report User](../11-user-report-analysis/REPORT_OVERVIEW.md)
