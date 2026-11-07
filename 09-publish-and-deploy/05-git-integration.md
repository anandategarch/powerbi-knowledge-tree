# Git Integration: Version Control untuk Report Power BI

> Workspace Power BI/Fabric kini bisa dihubungkan langsung ke Azure DevOps atau GitHub, dan file report bisa disimpan sebagai PBIP/PBIR — kumpulan file JSON per halaman dan per visual. Ini mengubah pembuatan report dari "file biner besar" menjadi praktik software: diff, branch, pull request, dan rollback — dan format PBIR-enhanced (yang dipakai report Cost Control ini) memang dirancang untuk itu.

## Konsep & Penjelasan

### Format PBIP & PBIR — anatomi file
Saat menyimpan dari Desktop dengan **Save As > Power BI project (*.pbip)**, yang dihasilkan adalah folder teks, bukan biner tunggal:

```text
report.pbip                     ← file proyek kecil (penunjuk artefak)
.Report/
  definition.pbir               ← penanda + $schema + versi format
  definition/
    report.json                 ← properti report-level (theme, settings)
    pages/
      pages.json                ← urutan daftar halaman
      <pageId>/page.json        ← 1 file per halaman (nama, visible, ukuran)
      <pageId>/visuals/
        <visualId>/visual.json  ← 1 file per visual (posisi, tipe, config)
    bookmarks/ …                ← bookmark, mobile layout, dll.
.SemanticModel/                 ← model dalam format TMDL (teks)
  model.tmdl, tables/*.tmdl     ← skema, measures 161 buah, relasi
```

Poin penting:
- `definition.pbir` menandai "folder ini adalah report"; `report.json` mengatur properti lintas halaman.
- `page.json` per halaman → menambah halaman tidak menyentuh file halaman lain.
- `visual.json` per visual (ID folder visual berupa hash yang stabil) → rename halaman/visual tidak me-regenerasi seluruh file.
- Folder `<pageId>` adalah ID (hash), bukan nama — persis struktur `Report/definition/pages/...` yang terekstrak dari pbix report ini: PBIR-enhanced.

### Git integration di workspace (Fabric/Power BI)
- Buka workspace > **Source control** (Git integration) > connect ke **Azure DevOps** atau **GitHub** (organisasi, repository, branch, root folder).
- Panel Source control menampilkan item yang berubah (baru/diubah/dihapus), lalu dua aksi utama:
  - **Commit** — kirim perubahan item dari workspace ke branch (dengan pesan commit).
  - **Update** (pull) — tarik perubahan dari branch ke workspace.
- Item yang didukung: report (PBIR), semantic model (TMDL), dan sebagian besar item Fabric lainnya.
- Alur kerja paling umum: ubah di Desktop → Save (pbip) di folder repo lokal → `git push` → di service klik Update → workspace terbaru.

### Merge conflict — kenapa PBIR mengurangi drama
- Konflik Git terjadi bila **file yang sama** diubah paralel. Pada format biner `.pbix`, setiap konflik = pilih salah satu file utuh (kalah total satu sisi).
- Pada PBIR: dua orang mengedit visual berbeda = file `visual.json` berbeda → **tidak ada konflik**. Konflik hanya muncul bila visual yang sama atau `page.json`/`report.json` (mis. dua orang menambah halaman sekaligus) — dan penyelesaiannya masih bisa manual per baris JSON.
- Praktik aman: pisahkan pekerjaan per halaman ("kamu halaman Waste, aku halaman Purchasing").

### .gitignore untuk pbip
```gitignore
# Binari besar — jangan masuk repo
*.pbix
*.pbit

# Layout diagram yang berubah tiap dibuka (noise diff)
**/diagramLayout.json
**/semanticModelDiagramLayout.json

# Cache & artefak lokal Desktop
**/cache/
**/.vs/
```
- Beberapa tim juga meng-ignore `annotations.json` karena urutannya sering berubah tanpa makna — coba dulu di tim kecil.
- Yang **harus** di-commit: `.pbip`, seluruh `.Report/`, `.SemanticModel/` (TMDL), dan dokumentasi md (repo knowledge tree ini).

### Branch strategy untuk tim kecil
- `main` = sumber kebenaran yang selalu dicerminkan ke workspace PROD.
- Kerja di **feature branch per unit kerja**: `feat/summary-kpi`, `fix/slicer-sync`, `feat/calc-group-periode` — satu PR tidak lebih dari beberapa halaman/measure.
- **PR review** di GitHub/Azure DevOps: reviewer membaca diff `visual.json` (posisi/config) dan `*.tmdl` (measure). Untuk reviewer non-teknis, screenshot before/after di deskripsi PR.
- Naming commit semantik: `feat(page): tambah toggle chart/table Summary Deviasi`, `fix(dax): growth pakai ABS denominator`.
- Tag rilis `v1.4` setiap Update App — pasangan dengan changelog.

### Alternatif tanpa Git integration
- **PBIP + Git manual** (tanpa menghubungkan workspace): simpan pbip di repo, commit manual, distribusi ke service tetap lewat publish/pipeline. Ini pendekatan repo dokumentasi saat ini.
- **Deployment pipeline synergy**: workspace DEV di-connect ke Git (single source of truth); TEST/PROD hanya menerima hasil deploy — jangan pernah commit dari PROD (lihat [Deployment Pipelines](./03-deployment-pipelines.md)).

### Kenapa file PBIR-enhanced = siap untuk Git
- Teks JSON yang bisa di-diff per visual → review bermakna.
- ID stabil → rename tidak memicu rewrite massal.
- Semantic model sebagai TMDL → measure 161 buah bisa dibedah satu-satu di PR.
- Bisa dikembalikan ke versi mana pun tanpa menyimpan `report_v2_final_FIX.pbix` — penyakit klasik file sharing.

## Sintaks & Contoh

Siklus kerja harian (1 pengembang, repo knowledge tree + report):

```bash
# mulai unit kerja baru
git checkout main && git pull
git checkout -b feat/top10-deviasi-dinamis

# setelah mengedit report di Desktop (Save as pbip ke folder repo)
git status                       # lihat visual.json mana yang berubah
git add .Report .SemanticModel
git commit -m "feat(top10): what-if N untuk Top 10 Item Dev"
git push -u origin feat/top10-deviasi-dinamis

# buka PR di GitHub, review, merge ke main
git checkout main && git pull
```

Lalu di service: workspace DEV > Source control > **Update** (pull `main`) → deploy ke TEST/PROD via pipeline.

Diff yang sehat vs yang harus diwaspadai:

```text
✔ +1 file  .Report/definition/pages/<id>/visuals/<vid>/visual.json  (visual baru)
✔ ~        .SemanticModel/tables/Deviasi All.tmdl                    (measure baru)
✖ ~ ribuan file berubah sekaligus → kemungkinan besar file di-format ulang /
   urutan objek berubah: kembali dulu (git checkout -- .), lalu ubah bertahap.
```

## Kesalahan Umum
- **Commit file `.pbix` 148 MB** → repo membengkak dan Git melambat; perbaikan: `.gitignore` sejak commit pertama.
- **Dua orang mengedit `page.json`/halaman yang sama** → konflik sering; perbaikan: bagi kerja per halaman, satu PR per unit kecil.
- **Lupa Update (pull) di service** → workspace service versi lama padahal Git sudah maju; perbaikan: jadikan "pull setelah merge" bagian dari SOP.
- **Menyimpan PBIX dan PBIP paralel** → dua sumber kebenaran; perbaikan: PBIP adalah master, PBIX hanya export sesekali.
- **Merge sendiri ke `main` tanpa PR** → hilang momen review; perbaikan: branch protection (require PR) di GitHub.
- **Men-commit dari PROD** → sejarah kotor; perbaikan: PROD read-only, perubahan hanya lahir di DEV.

## Tips Pro
- Simpan repo dokumentasi (knowledge tree ini) dan kode report dalam satu repo — PR otomatis menyatukan "perubahan" dan "penjelasannya".
- Screenshot sebelum/sesudah di body PR — reviewer visual (atasan/PIC) tidak perlu paham JSON.
- Gunakan `git log -- .Report/definition/pages/<pageId>` untuk melihat sejarah satu halaman.
- Tag `v1.x` setiap Update App + catat di changelog → jawaban instan "kapan grafik ini berubah?".
- Backup bonus: repo = backup penuh report + model, lebih aman daripada copy manual (lihat [FAQ](../12-troubleshooting/faq.md)).

## Terkait
- [Workspaces](./01-workspaces.md)
- [Deployment Pipelines](./03-deployment-pipelines.md)
- [Calculation Groups](../10-advanced/01-calculation-groups.md)
- [Dynamic Visuals & Formatting](../10-advanced/04-dynamic-visuals-and-formatting.md)
- [FAQ](../12-troubleshooting/faq.md)
- [Analisis Report User](../11-user-report-analysis/REPORT_OVERVIEW.md)
