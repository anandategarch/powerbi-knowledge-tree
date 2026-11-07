# Bookmarks — Menangkap State Report

> Bookmark menyimpan "snapshot" kondisi report — filter/slicer yang aktif (data), tampilan show/hide visual (display), atau keduanya — dan dipanggil kembali lewat tombol, link, atau pane Bookmarks. Ini fondasi untuk tombol Reset, navigasi, toggle chart, dan popup layer di report multi-halaman seperti milik user (33 action button).

## Konsep & Anatomy

- Bookmark dibuat/dikelola di pane **Bookmarks** (View → Bookmarks).
- **Options saat menangkap bookmark** (menu `…` pada bookmark → Options):
  - **Data** — menyimpan state filter, slicer, sort, dan posisi drill saat bookmark dibuat.
  - **Display** — menyimpan state tampilan: visual show/hide (Selection pane), spotlight/focus, dan pengaturan tampilan visual.
  - Default: keduanya aktif — matikan sesuai kebutuhan pola.
- **View** (tombol play kecil) — menjalankan bookmark saat desain untuk pengujian.
- Bookmark yang dibuat di Desktop tersedia untuk semua pemirsa di Service; **Personal bookmarks** (dibuat pemirsa di Service) bersifat privat per user dan bisa dijadikan default tampilan pribadi.

## Personal Bookmarks di Service

- Pemirsa report dapat menyimpan kombinasi filter/slicer sendiri (baris perintah **Bookmarks → Personal bookmarks**) tanpa izin edit.
- Bisa di-set sebagai tampilan default pribadi; tidak mengganggu bookmark designer.
- Relevan untuk report cost-control: tiah PIC/manajer area menyimpan "halaman miliknya" tanpa minta developer bikin halaman khusus.

## Use Case Utama

### 1. Tombol Reset / Clear Filters
1. Set semua slicer ke kondisi default (semua terpilih/bersih).
2. Tambah bookmark **Data ON, Display OFF** → beri nama `Reset`.
3. Buat button → Action → **Bookmark** → pilih `Reset`.
- Pola yang dipakai tombol **Reset** di report user; jarak klik jauh lebih cepat daripada clear manual tiap slicer.

### 2. Navigasi (Button → Bookmark/Page)
- Button Action bisa: **Page navigation** (langsung ke halaman) atau **Bookmark** (bisa menggabungkan pindah halaman + set filter).
- Struktur nav user (Home → Summary → Detail → by-dimensi) memakai 33 action button; letakkan strip nav konsisten di tiap halaman.

### 3. Toggle Chart / Halaman Virtual
- Dua visual menempati area sama: satu di-hide via Selection pane, satu tampil.
- Bookmark `Tampil A` (display) → tombol A; bookmark `Tampil B` → tombol B.
- Hasil: satu "halaman" berisi dua tampilan tanpa duplikasi layout.

### 4. Toggle Topik Analisa (pattern Multi Value CM)
- Kombinasi **bookmark + field parameter / slicer pilihan tipe nilai** (CM, LM, Waste, Growth): bookmark menyimpan state slicer (Data) sehingga tombol "Deviasi" / "Waste" / "Sales" memutar seluruh halaman ke topik itu — chart, matrix, dan judul dinamisnya ikut berganti.
- Judul dinamis: measure `SELECTEDVALUE('Tipe Nilai'[Nama])` di textbox/card.

### 5. Popup Layer (Info/Detail Overlay)
- Buat group berisi panel popup (rectangle semi transparan + textbox/chart) di layer atas → hide via Selection pane.
- Bookmark `Popup ON` (Display) & `Popup OFF`; tombol ℹ membuka, tombol ✕ menutup.
- Lebih ringan daripada membuat halaman terpisah untuk info kecil.

## Bookmark + Selection Pane (Show/Hide)

- Pola show/hide mengandalkan **Selection pane** (lihat [Selection & Z-Order](./05-selection-and-z-order.md)): klik mata untuk hide/show objek, lalu tangkap bookmark dengan **Display ON, Data OFF**.
- Golden rule: bookmark display **tidak boleh** ikut menyimpan data kalau Anda tidak ingin slicer ikut berubah — centang opsinya dengan sadar.
- Saat bookmark dibuat, seluruh state di halaman aktif tersimpan — pilihannya yang menentukan apa yang di-*playback*.

## Bookmark Groups

- Di pane Bookmarks, drag beberapa bookmark ke dalam **group** (klik kanan → Group) — group mempermudah organisasi (`Nav`, `Toggle`, `Reset`).
- Menjalankan **View** pada group memutar bookmark berurutan — cocok untuk mode presentasi langkah-demi-langkah di rapat review deviasi.
- Group juga jadi dokumentasi: reviewer langsung paham fungsi tiap bookmark dari strukturnya.

## Hubungan dengan Action Button

- Button (Insert → Button) punya **Action**: Bookmark, Page navigation, Back, Web URL, Q&A.
- Button drillthrough bisa **auto-disabled** (diredupkan otomatis) ketika konteks tidak tersedia — bawaan Power BI untuk tombol navigasi drillthrough.
- Beri **tooltip** di button dan state hover/press berbeda agar terasa seperti aplikasi.
- Semua 33 action button user idealnya konsisten: posisi strip nav, ukuran 32–40px, ikon jelas, tooltip deskriptif.

## Sintaks & Contoh

```text
[Setup toggle "Deviasi ↔ Waste ↔ Sales" (pattern Multi Value CM)]
1. Buat field parameter 'Tipe Nilai' (CM Deviasi, Waste, Sales Growth)
   → jadikan slicer horizontal di header.
2. Semua chart utama memakai measure yang membaca
   SELECTEDVALUE('Tipe Nilai'[Nama]) lewat SWITCH.
3. Buat judul dinamis:
   Title Topik = "Analisa " & SELECTEDVALUE('Tipe Nilai'[Nama], "Deviasi")
4. Slicer dipilih "Deviasi" → Add bookmark "btn_Deviasi" (Data ON, Display OFF).
   Ulangi untuk "Waste", "Sales".
5. Tiga tombol di header → Action → Bookmark masing-masing.
→ Satu halaman, tiga sudut pandang, nol duplikasi layout.
```

## Kesalahan Umum

- **Menangkap state "kotor"** — bookmark dibuat saat ada slicer terpilih/sort berubah tanpa disadari → tombol Reset justru menyimpan filter lama. Selalu bersihkan dulu, lalu Add.
- **Lupa memilih option Data/Display** → toggle tampilan ikut mengubah filter (atau sebaliknya), perilaku jadi sulit diprediksi.
- **Nama bookmark asal-asalan** (`Bookmark 1`, `Bookmark 2`) → 20 bookmark kemudian tidak ada yang berani dihapus; pakai prefix `btn_`, `reset_`, `popup_`.
- **Mengedit halaman setelah bookmark dibuat tanpa Update** → bookmark lama merekam struktur sebelumnya; setelah desain berubah, buka tiap bookmark → **Update**.
- **Bookmark menumpuk perubahan kecil** (banyak versi hampir sama) → jaga jumlah bookmark seminimal fungsinya.
- **Menyembunyikan visual via bookmark tapi visual tetap ditumpuk visible** di kondisi lain → beban render tetap ada.

## Tips Pro

- Urutan pembuatan: selesaikan **desain halaman dulu**, baru buat semua bookmark — mengurangi kebutuhan Update berulang.
- Simpan satu bookmark "master default" per halaman (semua default + semua popup tertutup) sebagai titik pulih saat desain berantakan.
- Untuk navigasi yang hanya pindah halaman, pilih **Page navigation** di button, bukan bookmark — lebih sederhana dan tidak menyimpan state.
- Kombinasi bookmark + alt text & tab order menjaga aksesibilitas: tombol harus bisa diakses keyboard dan screen reader membaca fungsinya.
- Uji di Service dengan user uji: personal bookmarks mereka tidak merepotkan desain — promosikan fitur ini untuk mengurangi permintaan "halaman khusus".

## Terkait

- [Page Layout](./01-page-layout.md)
- [Selection & Z-Order](./05-selection-and-z-order.md)
- [Drillthrough](./03-drillthrough.md)
- [Visual Interactions](../04-visuals/06-visual-interactions.md)
- [Themes & Templates](./06-themes-and-templates.md)
- [Overview report user (action buttons & navigasi)](../11-user-report-analysis/REPORT_OVERVIEW.md)
