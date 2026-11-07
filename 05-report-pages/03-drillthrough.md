# Drillthrough — Dari Summary ke Halaman Detail

> Drillthrough memungkinkan pengguna klik kanan (atau tombol) pada sebuah data point dan "melompat" ke halaman detail yang sudah terfilter konteksnya — pola Summary Deviasi → Detail Deviasi pada report user. Dokumen ini membahas setup field well, tombol back otomatis, keep all filters, cross-report drillthrough, hingga beda drillthrough vs drill down vs expand.

## Konsep Dasar

- Drillthrough = navigasi antar halaman **membawa filter konteks** (field tertentu) dari visual sumber ke halaman target.
- Halaman target mendefinisikan field penerima di field well **Drill through** (panel Visualizations, saat tidak ada visual terpilih → tampilan halaman).
- Saat pengguna klik kanan titik data yang mengandung field tersebut → menu **Drill through** menawarkan halaman target.
- Measure **tidak bisa** dimasukkan ke Drill through well (hanya field/kolom) — nilai measure di halaman target dihitung ulang dalam konteks filter yang diteruskan.

## Setup Standar

1. Buat halaman detail (misal *Detail Deviasi*) — kosongkan visual sementara.
2. Klik area kosong halaman → panel Visualizations → tarik field penerima ke **Drill through** well, contoh: `Resto[Nama]`, `Item[Nama Item]`, `Area[Nama]`.
3. Power BI otomatis menambahkan **tombol Back** di pojok kanvas — bisa direstyle/dipindah (jangan dihapus kalau tidak diganti tombol custom Action → Back).
4. Bangun visual detail di halaman itu — semua measure otomatis terfilter oleh field drillthrough.
5. Kembali ke halaman summary → klik kanan bar/kategori → **Drill through → Detail Deviasi**.

## Keep All Filters

- Toggle **"Keep all filters"** (di sekitar Drill through well): bila ON, **semua filter yang sedang aktif** pada halaman sumber (periode, area, slicer apa pun) ikut diteruskan ke halaman target — perilaku yang diinginkan hampir selalu.
- Bila OFF, hanya field di Drill through well yang diteruskan — halaman detail "bersih" dari filter lain.
- Filter yang diteruskan muncul di **Filters pane** halaman target sebagai filter read-only (badge drillthrough) — pengguna bisa melihat konteksnya.

## Tombol Back & Tombol Drillthrough

- **Back button otomatis**: merah panah kiri; berfungsi seperti tombol back browser di dalam sesi drillthrough. Restyle sesuai tema.
- Alternatif: Insert → Button → pilih Navigation → set Action type **Back** untuk tombol custom bergaya nav bar.
- **Drillthrough button**: Insert → Button → Navigator → **Drill through**; tombol ini otomatis tahu target & **otomatis disabled** (diredupkan) saat konteks tidak tersedia — sinyal UX yang sangat baik.
- Letakkan tombol back di posisi konsisten (kiri atas) di semua halaman detail.

## Drillthrough + Tooltip Page

- Halaman tipe tooltip juga bisa menerima field Drill through — hover pada data point memunculkan tooltip yang kaya (mini detail page) tanpa berpindah halaman.
- Kombinasi yang powerful untuk report user: hover bar resto di Summary → tooltip card deviasi; klik kanan → drillthrough penuh ke halaman Detail Deviasi.
- Aturan praktis: tooltip = konteks cepat (3–5 visual ringan), drillthrough = analisa dalam (halaman penuh).

## Cross-Report Drillthrough

- Drillthrough bisa **antar report** (satu pbix → pbix lain): aktifkan di report target melalui report settings di Power BI Service.
- Pengguna klik kanan data point di report A → Drill through → pilih halaman report B (harus berada di workspace My Workspace atau workspace yang sama).
- Field dihubungkan berdasarkan **nama kolom & nama tabel yang sama persis** — disiplin naming antar report menjadi syarat.
- Cocok untuk arsitektur: report ringkasan korporat → report cost-control detail per divisi.

## Dynamic M Security (Ringkas)

- Untuk halaman detail yang berisi data sensitif per resto/PIC, amankan dengan **Row-Level Security (RLS)** — termasuk **dynamic security** di level Power Query (M): parameter/kueri M difilter berdasarkan `USERNAME()`/`USERPRINCIPALNAME()` sebelum data masuk model.
- Relevansi dengan drillthrough: drillthrough hanya memfilter tampilan — **bukan pengaman**. Pengguna tetap hanya boleh drillthrough ke resto yang boleh ia lihat; itu ditegakkan oleh RLS, bukan oleh desain halaman.
- Uji RLS via **View as Roles** di Desktop sebelum publish.

## Drillthrough vs Drill Down vs Expand

| Aspek | Drill Down | Drill Through | Expand (matrix) |
|---|---|---|---|
| Lokasi | visual yang sama | halaman lain | matrix yang sama |
| Mekanisme | hierarki di satu field well | field di Drill through well | tombol +/- per baris |
| Konteks | level hierarki | field tertentu + filter aktif | level hierarki baris |
| Cocok untuk | eksplorasi cepat | analisa mendalam kontekstual | tabel rinci bertingkat |

## Best Practices

- **Batasi jumlah field di Drill through well** — tiap field menambah opsi menu & kompleksitas; 2–4 field inti (Resto, Item, Area, PIC) cukup.
- **Judul dinamis** agar pengguna tahu konteksnya:

```dax
Title Detail Deviasi =
    "Detail Deviasi — "
    & SELECTEDVALUE ( Resto[Nama], "Semua Resto" )
    & " | " & SELECTEDVALUE ( 'Periode'[Nama], "Semua Periode" )
```

- Sembunyikan halaman drillthrough dari tab (klik kanan → Hide page) bila hanya diakses lewat drillthrough — mengurangi kebingungan navigasi.
- Sertakan **ringkasan konteks** di atas halaman detail (card kecil: nama resto, area, PIC) — tidak semua pengguna memperhatikan filter pane.
- Pertahankan pola konsisten: satu halaman detail drillthrough per entitas (per resto / per item), jangan satu halaman mencoba melayani semua konteks sekaligus dengan logika IF rumit.
- Untuk report user: *Summary Deviasi* → *Detail Deviasi* adalah pasangan drillthrough alami; halaman *Deviasi by Resto/Item* bisa juga men-supply drillthrough dengan field berbeda (Area→ halaman by Resto, dll).

## Sintaks & Contoh

```text
[Blueprint: Summary Deviasi → Detail Deviasi]
Halaman target : Detail Deviasi (hidden)
Drill through  : Resto[Nama], Item[Nama Item]
Keep all filters: ON  (periode & area dari summary ikut)
Back button    : otomatis, restyle ikon ◀ warna tema
Header halaman : textbox dengan Title Detail Deviasi (measure dinamis)
Konten         : matrix item × bulan (conditional formatting),
                 donut komposisi, combo COM vs Deviasi%
Tombol akses   : button "Detail" di tiap bar →
                 Action = Drill through (auto-disabled bila tak tersedia)
```

## Kesalahan Umum

- **Menaruh measure ke Drill through well** → tidak didukung; gunakan field kolom (measure cukup dihitung ulang otomatis).
- **Keep all filters OFF tanpa sadar** → halaman detail menampilkan data periode yang berbeda dari halaman sumber.
- **Menghapus tombol back otomatis tanpa pengganti** → pengguna terjebak di halaman detail (harus klik tab manual).
- **Halaman drillthrough dibiarkan terlihat di tab** dan dibuka langsung tanpa konteks → tampil "Semua Resto" dan membingungkan; gunakan judul dinamis dengan default jelas.
- **Field drillthrough terlalu banyak** → menu klik kanan penuh dan tiap field menambah kompleksitas pemeliharaan.
- **Menganggap drillthrough sebagai security** → tanpa RLS, pengguna bisa tetap membuka halaman detail dan melihat semua resto.

## Tips Pro

- Kombinasi tombol Navigator **Drill through** + tooltip "Lihat detail item ini" memberi affordance yang jelas tanpa perlu klik kanan (yang tidak diketahui semua pengguna).
- Tambahkan tombol **Reset filter** (bookmark) di halaman detail untuk membersihkan filter tambahan yang pengguna ubah sendiri di sana.
- Untuk mobile, drillthrough tetap berfungsi lewat tap-and-hold — pastikan tombol back ukurannya layak sentuh (≥ 40px).
- Auditrail konteks: visual kecil "sumber: [Selected Resto]" meminimalkan salah baca saat screenshot dibagikan via WhatsApp/rapat.
- Performance: halaman drillthrough yang berat (banyak visual) tetap harus lean — pengguna bolak-balik summary-detail ratusan kali.

## Terkait

- [Tooltips](./04-tooltips.md)
- [Bookmarks](./02-bookmarks.md)
- [Page Layout](./01-page-layout.md)
- [Visual Interactions](../04-visuals/06-visual-interactions.md)
- [Katalog Visual Bawaan](../04-visuals/01-native-visuals-catalog.md)
- [Struktur halaman report user](../11-user-report-analysis/pages-and-visuals.md)
