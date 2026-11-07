# Workspaces: Fondasi Kolaborasi & Deployment di Power BI Service

> Workspace adalah wadah bersama di Power BI Service tempat report, semantic model, dan item lain disimpan dan dikelola; App adalah kemasan distribusi read-only untuk konsumen. Materi ini membahas roles, sharing, publish dari Desktop, version history, dan pola DEV/TEST/PROD — semuanya sangat relevan ketika report Cost Control (36 halaman) dibagikan ke manajemen AM/PIC per area.

## Konsep & Penjelasan

### My Workspace vs Workspace bersama
- **My Workspace** adalah area pribadi: hanya pemilik yang bisa mengakses, tidak punya roles/anggota, **tidak bisa membuat App**, dan pengaturan aksesnya terbatas. Cocok untuk eksperimen, bukan untuk produksi.
- **Workspace bersama** (new workspace experience) memungkinkan banyak orang berkolaborasi dengan role berbeda, membuat App, mengatur scheduled refresh, dan mengelola akses secara teraudit.
- Aturan praktis: apa pun yang akan dilihat orang lain (seperti report Cost Control ke manajemen) wajib hidup di workspace bersama, bukan My Workspace.

### Roles di workspace
| Kemampuan | Admin | Member | Contributor | Viewer |
|---|---|---|---|---|
| Mengubah setelan workspace, menghapus workspace | ✔ | – | – | – |
| Menambah/menghapus orang (kecuali Admin) | ✔ | ✔ | – | – |
| Publish & update App | ✔ | ✔ | – | – |
| Upload/mengubah/menghapus konten (report, semantic model) | ✔ | ✔ | ✔ | – |
| Menjalankan refresh, mengelola koneksi data source | ✔ | ✔ | ✔ | – |
| Melihat & berinteraksi dengan konten (RLS tetap berlaku) | ✔ | ✔ | ✔ | ✔ |

Untuk report Cost Control: pengembang diberi Admin/Contributor di workspace kerja; manajemen cukup **Viewer** (atau lebih baik lagi, akses via App — lihat bawah).

### Workspace vs App — perbedaan & kapan dipakai
- **Workspace** = dapur: tempat semua versi konten (termasuk yang belum siap) dilihat oleh tim pengembang.
- **App** = piring saji: snapshot read-only pada saat app di-update, dengan navigasi khusus dan branding. Konsumen tidak bisa mengedit; mereka hanya melihat.
- Kapan pakai App: audiens luas (manajemen AM/PIC/direksi), butuh navigasi terkurasi (dari 36 halaman, tunjukkan hanya 5–6 halaman utama di nav), butuh tampilan profesional dengan logo.
- Kapan cukup workspace Viewer langsung: tim kecil internal yang memang butuh interaksi lintas item; tapi App hampir selalu lebih rapi untuk konsumen.

### Membuat App (audience, navigation, permissions)
1. Buka workspace → semua item siap → klik **Create app**.
2. **Setup**: nama, logo, deskripsi, support contact, link.
3. **Navigation**: susun urutan; bisa menambahkan report tertentu, halaman tertentu (deep link), atau link eksternal. Halaman teknis (mis. Tooltip, ID Dokumen) disembunyikan dari navigasi.
4. **Audience**: kelompok penerima, misal `AM`, `PIC`, `Direksi`. Setiap audience dapat melihat konten/navigasi yang berbeda-beda. Gunakan security group Entra ID per area (mis. `SG-AM-Jabodetabek`, `SG-PIC-Bali`), bukan email per orang.
5. **Permissions**: ada opsi "entire organization" — hindari untuk data cost control; gunakan audience terbatas.
6. Publish app → bagikan URL app. Setiap ada perubahan di workspace, klik **Update app** agar konsumen melihat versi baru (App tidak sinkron otomatis).

### Sharing individual (reshare & build permission)
- Tombol **Share** pada item (report/semantic model/App) membagikan ke email/security group.
- Dua opsi penting di dialog Share:
  - **Allow recipients to reshare**: penerima boleh mem-forward akses — rantai sharing sulit diaudit; default off, biarkan off.
  - **Allow recipients to build content** (Build permission): penerima boleh membuat report baru / Analyze in Excel / koneksi XMLA read dari semantic model. Aktifkan hanya untuk sesama analis, bukan manajemen.
- Sharing ≠ membership: penerima share tetap "orang luar workspace", hanya melihat item tersebut, dan RLS tetap berlaku.

### Link sharing & risikonya
- Tiga tipe link: **Specific people**, **People in your organization with the link**, **Anyone with the link**.
- Link "anyone" dapat diteruskan ke luar organisasi tanpa login dan **RLS efektif tidak berlaku** untuk akses anonim — sangat berbahaya untuk data biaya/resto. Tenant admin sebaiknya mematikan opsi ini.
- Aturan emas: data cost control internal = specific people atau security group. Jangan pernah "anyone".

### Publish dari Desktop (pilih workspace, replace dataset)
- Di Power BI Desktop: **File > Publish > Publish to Power BI** → pilih workspace tujuan (bisa diubah dari dialog, jangan asal klik Enter ke workspace terakhir).
- Jika di workspace sudah ada semantic model dengan nama sama, muncul dialog **Replace**:
  - Replace semantic model: model lama ditimpa; kredensial/jadwal refresh umumnya dipertahankan — tapi selalu verifikasi di tab dataset setelah publish.
  - Report di service ditimpa versi baru. Jika ada App, **jangan lupa Update app**.
- **Save a copy** di service atau **Save As** di Desktop membuat duplikat ke workspace lain — ini cara manual mempromosikan DEV→TEST tanpa Deployment Pipeline (Pro).

### Version history
- Di service: buka report > menu `…` > **Version history** → lihat dan restore versi sebelumnya (riwayat versi terakhir, sekitar 30 hari).
- Version history hanya untuk **report**, bukan semantic model — jadi tetap wajib backup file master `.pbip`/`.pbix` (idealnya via Git: lihat [Git Integration](./05-git-integration.md)).

### Strategi workspace DEV / TEST / PROD
| Stage | Nama contoh | Isi & role | Akses |
|---|---|---|---|
| DEV | `CC [DEV]` | Semua eksperimen; pengembang = Admin/Contributor | Pengembang saja |
| TEST | `CC [TEST]` | Kandidat rilis + data uji realistis; tester = Contributor/Viewer | Pengembang + PIC terpilih |
| PROD | `CC [PROD]` | Hanya hasil deploy; pengembang tidak mengedit langsung | Manajemen via App (Viewer) |

- Tahapan ini dipetakan 1:1 ke [Deployment Pipelines](./03-deployment-pipelines.md).
- Jangan pernah mengedit langsung di PROD (download–edit–upload) — perubahan akan hilang saat deploy berikutnya dan file master berbeda dari yang produksi.

## Sintaks & Contoh

Alur distribusi report Cost Control (teks diagram):

```text
Desktop (report.pbip — master di Git)
   │  File > Publish > pilih workspace
   ▼
CC [DEV] ──pipeline──▶ CC [TEST] ──pipeline──▶ CC [PROD]
 (pengembang,          (tester + PIC,           │
  bebas utak-atik)      data realistis)         └─▶ App "Cost Control"
                                                    Audience: [AM] [PIC] [Direksi]
                                                    (security group per area)
```

Checklist share yang aman:

```text
☐ Share lewat App audience / security group (bukan email perorangan)
☐ Reshare = OFF
☐ Build permission = OFF untuk manajemen
☐ RLS role di semantic model aktif (AM hanya lihat area-nya)
☐ Update App setelah publish terakhir
☐ Link "anyone" tidak dipakai
```

## Kesalahan Umum
- **Share satu per satu ke puluhan email** → audit susah; perbaikan: buat security group per area dan pakai App audience.
- **Memberikan role Member/Contributor padahal orang itu hanya perlu lihat** → dia bisa menghapus konten; perbaikan: Viewer + App.
- **Centang "allow reshare" asal-asalan** → akses menyebar tanpa kontrol; perbaikan: matikan reshare, minta orang minta akses lewat admin.
- **Publish ke workspace terakhir tanpa cek dialog** → DEV malah terpublish ke PROD; perbaikan: selalu baca nama workspace sebelum OK.
- **Lupa Update App** → manajemen melihat data lama padahal workspace sudah baru; perbaikan: jadikan Update app bagian wajib SOP publish.
- **Menggunakan My Workspace untuk produksi** → tidak bisa App, tidak bisa kolaborasi; perbaikan: pindahkan ke workspace bersama.
- **Edit langsung di PROD dari service** → drift dengan file master; perbaikan: perubahan hanya lewat DEV→deploy.

## Tips Pro
- Naming convention yang seragam: `CC [DEV]`, `CC [TEST]`, `CC [PROD]` — akan otomatis rapi di Deployment Pipeline.
- Deskripsikan setiap workspace (owner, tujuan, kontak) supaya orang baru tidak salah target.
- Lakukan access review tiap kuartal: keluarkan orang yang pindah tugas dari audience App.
- Untuk 36 halaman: di App navigation, kelompokkan (Ringkasan / Deviasi / Waste / Purchasing) dan taruh link "Data lengkap" di bawah.
- Simpan file master sebagai PBIP di repo Git — workspace service adalah *output*, bukan *source of truth*.

## Terkait
- [Gateway & Refresh](./02-gateway-and-refresh.md)
- [Deployment Pipelines](./03-deployment-pipelines.md)
- [Fitur Power BI Service](./04-power-bi-service-features.md)
- [Git Integration](./05-git-integration.md)
- [Analisis Report User](../11-user-report-analysis/REPORT_OVERVIEW.md)
