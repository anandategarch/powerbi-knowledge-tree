# Object-Level Security (OLS) & Keamanan Menyeluruh

> OLS menyembunyikan **tabel dan kolom** dari role tertentu — bukan hanya barisnya (RLS), melainkan objeknya sama sekali tidak terlihat dan tidak bisa dipakai. File ini juga merangkum lapisan keamanan lain: sensitivity label, workspace roles, App audiences, sharing link, export control, hingga checklist menyeluruh.

## Konsep & Penjelasan

### OLS vs RLS vs "Hidden" biasa

| Mekanisme | Menyembunyikan | Catatan |
|---|---|---|
| **Hidden** (standard) | Kolom/tabel dari *field list* saat authoring | Bukan security: data tetap mengalir ke visual, semua user tetap "melihat" nilainya lewat visual |
| **RLS** | **Baris** data per user | Data kolom tetap ada; baris yang tidak berhak disaring |
| **OLS** | **Objek** (tabel/kolom) per role | Field tidak muncul sama sekali bagi role itu; tidak bisa dipakai visual, measure, maupun export |

Perilaku OLS bagi user yang tidak berhak: field tidak terlihat di field list, dan **visual/report yang memakai field itu akan error** untuk role tersebut. Oleh karena itu OLS harus direncanakan bersama desain halaman, bukan ditambal belakangan.

### Setup via Tabular Editor (security metadata)

OLS tidak punya antarmuka di Power BI Desktop — ia disimpan sebagai **metadata security pada role** di dalam model. Alur kerjanya:

1. Publish dataset, lalu hubungkan **Tabular Editor** melalui **XMLA endpoint** (menulis ke XMLA membutuhkan kapasitas Premium/PPU; istilah UI bisa sedikit berbeda antar versi tool).
2. Buka role yang dimaksud, tandai tabel/kolom yang ingin diamankan sebagai **secured** (mis. `Inventory[Harga Modal]`, `Standar Susut[Target]`).
3. Simpan — metadata OLS tersimpan di dataset.
4. Jangan menimpa model dengan publish ulang dari Desktop setelah OLS dipasang: deploy dari Desktop bisa menghapus metadata yang tidak dipahaminya. Kelola perubahan berikutnya tetap dari tool eksturnal (Tabular Editor / ALM toolkit).

Pola yang lebih mudah dirawat daripada mengamankan kolom satu-satu: kumpulkan kolom sensitif ke **tabel khusus** (mis. `Inventory Cost`) sehingga OLS cukup diamankan di tingkat tabel.

### OLS + RLS — kombinasi

Keduanya bisa dipasang pada role yang sama dan saling melengkapi (defense-in-depth):

> User AM Area Jakarta: **RLS** membatasi baris ke cabang Jakarta; **OLS** menyembunyikan kolom `Harga Modal` (cost price) sehingga meskipun melihat barisan item area-nya, harga modal tidak pernah muncul.

Aturan main saat mendesain: kolom yang di-secure **tidak boleh direferensikan** oleh measure, hierarchy, atau visual yang dipakai role tersebut — kalau iya, halaman itu error untuk role ini. Solusinya: pisahkan halaman sensitive ke audience terpisah (lihat Apps audiences) atau duplikasi versi visual tanpa field tersebut.

### Sensitive data pattern (salary, cost price)

Pola umum untuk data sensitif di report operasional seperti ini:

1. **Pisahkan** kolom sensitif ke tabel sendiri (`Inventory Cost`, `Standar Susut Target`) — OLS satu tabel, bukan sepuluh kolom.
2. **Pisahkan halaman** yang memakai kolom itu; jangan campur di halaman yang dilihat semua role.
3. Untuk angka turunan yang tetap perlu tampil (mis. margin tanpa harga modal), buat measure yang sudah diagregasi — jangan ekspos kolom mentahnya.

### Sensitivity label (Microsoft Purview) — singkat

Jika admin sudah mengaktifkan Purview, file/report bisa diberi label (Public / Internal / Confidential / Highly Confidential). Efeknya: label menempel saat data di-export ke Excel/PowerPoint, memicu kebijakan DLP (mis. larangan share ke luar organisasi), dan muncul sebagai penanda di Service. Label adalah lapisan *file*, bukan pengganti RLS/OLS — gunakan bersamaan.

### Workspace access roles

| Role workspace | Kelola akses | Publish/update App | Edit content | Kena RLS? |
|---|---|---|---|---|
| **Admin** | Ya | Ya | Ya | Tidak (lihat semua) |
| **Member** | Tidak | Ya | Ya | Tidak |
| **Contributor** | Tidak | Tidak | Ya | Tidak |
| **Viewer** | Tidak | Tidak | Tidak | **Ya** |

Aturan tata kelola: role workspace hanya untuk tim pembuat; konsumen diberi akses lewat **App** (bukan diundang ke workspace sebagai Viewer berjamaah).

### Apps audiences — master & sensitive

Satu App bisa punya beberapa **audience**; tiap audience melihat subset halaman berbeda:

- **Master audience**: semua halaman operasional (deviasi, waste, growth) — untuk AM/PIC.
- **Sensitive audience** (mis. "HO – Finance"): menambahkan halaman yang memakai kolom cost/standar susut.

User yang masuk dua audience otomatis melihat gabungan halaman. RLS tetap berlaku di atasnya — audience mengatur *halaman*, RLS mengatur *baris*, OLS mengatur *objek*.

### Sharing vs App, dan risiko link

| Jalur | Karakter | Risiko |
|---|---|---|
| **App** | Kurasi, halaman per audience, link satu untuk semua | Paling terkontrol — jalur default |
| **Share link – Specific people** | Hanya orang yang disebut | Rendah |
| **Share link – People in your organization** | Semua karyawan bisa membuka | Sedang — pakai hanya untuk data internal yang boleh dilihat semua |
| **Anyone with the link (public)** | Internet bisa membuka | **Tinggi — jangan pernah untuk data bisnis**; nonaktifkan di admin portal bila tidak dibutuhkan |

Opsi share tambahan yang perlu diperhatikan: "Allow recipients to reshare" (boleh meneruskan) dan "Allow recipients to build content with the data" (Build — bisa bikin report baru di atas dataset; tetap kena RLS, tapi beri hanya yang memang butuh).

### Export data control

Pengaturan (admin portal + dataset) menentukan user bisa mengekspor:

- **Tidak bisa export** sama sekali,
- hanya **summarized data** (apa yang terlihat di visual), atau
- termasuk **underlying data** (baris mentah di belakang visual).

Ekspor selalu mengikuti RLS/OLS user. Untuk data cost-control, posisi yang disarankan: summarized saja; underlying hanya untuk role finance. Kombinasikan dengan sensitivity label agar file hasil export ikut berlabel.

### Certified datasets permission

Endorsement dataset ada dua tingkat: **Promoted** (by self/team) dan **Certified** (naik kelas setelah direview). Pengesahan (certification) hanya bisa dilakukan oleh **authorized reviewer** yang didaftarkan admin di portal — bukan sembarang pembuat report. Manfaatnya: user lain percaya memakai dataset resmi, dan tidak tumbuh dataset duplikat dengan angka berbeda-beda (sumber kebenaran tunggal).

### Checklist keamanan menyeluruh

| # | Lapisan | Aksi |
|---|---|---|
| 1 | Workspace | Role minimal: pembuat = Contributor ke atas; konsumen tidak diundang ke workspace |
| 2 | Data baris | Dynamic RLS + `UserSecurity`; fail-closed |
| 3 | Objek | OLS untuk tabel/kolom sensitif (cost price, standar susut) |
| 4 | Halaman | App audiences: master vs sensitive |
| 5 | File | Sensitivity label (Purview) untuk report & export |
| 6 | Export | Summarized only secara default; underlying per exception |
| 7 | Link | Kebijakan share: Specific people; "anyone with the link" diblokir |
| 8 | Kredensial | Gateway pakai akun service/organizational dengan scope minimum — jangan akun pribadi |
| 9 | Audit | Review anggota workspace + isi `UserSecurity` tiap kuartal; pantau audit log |
| 10 | Endorsement | Dataset utama di-Certified lewat authorized reviewer |

## Sintaks & Contoh

Skenario implementasi untuk report cost-control (rangkaian penuh):

```text
1. RLS  : UserSecurity(Area, Email) → role "Dynamic Users"
          filter: [Email] = USERPRINCIPALNAME()
2. OLS  : via Tabular Editor (XMLA) — secure tabel 'Inventory Cost'
          (berisi Harga Modal) untuk role yang sama
3. App  : Audience "Master" (semua halaman operasional)
          Audience "HO – Finance" (+ halaman margin & standar susut)
4. Export: dataset setting → summarized only
5. Label: Purview label "Confidential – Internal" pada report & dataset
6. Uji  : matriks role × halaman → pastikan kombinasi (3) tidak error
          karena visual memakai kolom yang di-OLS
```

Matriks uji (contoh):

| Role/Audience | Halaman operasional | Halaman margin (cost) |
|---|---|---|
| AM Area (Master) | ✔ tampil, RLS area-nya | ✘ tidak ada di audience |
| Finance (Sensitive) | ✔ | ✔ (OLS tabel cost dibuka untuk role finance) |

## Kesalahan Umum

- **Menganggap "Hidden" = keamanan** — hidden hanya menyembunyikan dari field list; data tetap mengalir ke visual. Security harus RLS/OLS.
- **OLS dipasang di kolom yang dipakai visual umum** — halaman error untuk role itu; pindahkan field sensitif ke halaman/audience terpisah.
- **Share "Anyone with the link" untuk kemudahan** — risiko tertinggi; ganti ke App atau Specific people.
- **Semua user diundang sebagai Viewer ke workspace** — menggembungkan daftar anggota & sulit diaudit; pakai App.
- **Contributor diberikan ke konsumen** — otomatis melihat semua data (RLS tidak berlaku); beri akses lewat App.
- **Publish ulang dari Desktop setelah OLS** — metadata security bisa tertimpa; kelola lewat tool XMLA.
- **Export underlying menyala default** — baris mentah (harga modal) bocor ke Excel; batasi summarized.

## Tips Pro

- Pisahkan **workspace Dev dan Prod**: OLS/RLS diatur di Prod; perubahan model lewat deployment pipeline/ALM Toolkit, bukan edit manual.
- Buat **dokumen matriks akses** (siapa → role → audience → area) di repo; review tiap kuartal bersama HR (turnover karyawan).
- Gunakan **security group**, bukan individu, di semua lapisan (role RLS, audience App, share) — onboarding/deboarding cukup satu tempat di M365.
- "Fail closed" di semua lapisan: mapping RLS kosong = tidak melihat; kolom OLS di visual = error yang disengaja — lebih baik gagal terlihat daripada bocor diam-diam.
- Uji penetrasi sederhana secara berkala: login sebagai user biasa, coba Build report baru & export underlying — pastikan tetap terbatas.

## Terkait

- [Row-Level Security](./01-rls.md) — filter baris (pasangan OLS)
- [Dynamic Filtering](../06-filters-and-slicers/03-dynamic-filtering.md) — TREATAS untuk model terkunci
- [Filter Types](../06-filters-and-slicers/01-filter-types.md) — filter biasa vs security filter
- [Optimization Checklist](../07-performance/03-optimization-checklist.md) — service & monitoring
- [Report Overview](../11-user-report-analysis/REPORT_OVERVIEW.md) — tabel sensitif di report ini
