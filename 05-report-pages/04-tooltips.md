# Tooltips — Default, Field Well & Tooltip Page

> Tooltip adalah kartu informasi yang muncul saat hover pada data point — mulai dari default sederhana (isi Tooltip field well) sampai **report page tooltip**: halaman mini khusus berisi visual interaktif. Report user punya halaman "Tooltip" tersendiri (3 visual); dokumen ini membahas cara setup, desain hover card restoran, performa, dan perilaku tooltip di mobile.

## Tiga Bentuk Tooltip

1. **Default tooltip** — kotak kecil otomatis berisi kategori + nilai + field di well **Tooltips** pada tiap visual.
2. **Tooltip page (report page tooltip)** — halaman yang didesain sebagai tooltip; bisa berisi visual apa pun (card, mini chart, tabel) dan dikaitkan sebagai tooltip visual lain. Tooltip page **interaktif**: saat terbuka, kursor bisa masuk ke dalamnya untuk membaca/meng-klik visual yang ada di dalamnya.
3. **Tooltip field well pada visual lain** — beberapa visual menerima Tooltip well sebagai slot informasi pelengkap (bukan halaman terpisah).

Ketiganya tidak saling menggantikan — kombinasikan: default untuk nilai cepat, tooltip page untuk konteks kaya.

## Default Tooltip & Tooltip Field Well

- Tarik measure/field apa pun ke well **Tooltips** di visual — nilainya muncul di tooltip mengikuti titik data yang di-hover.
- Gunakan untuk informasi pelengkap yang tidak perlu tampil permanen: `[Z Score]`, `[Growth %]`, `[Deviasi LM]` sebagai pembanding, atau `[Kategori Anomali]`.
- Batasi jumlah field (3–6) — tooltip yang memanjang sampai ke bawah layar adalah anti-pattern.
- Pengaturan tampilan default tooltip ada di **Format visual → Tooltip card**: background, transparency, teks label (bisa diubah/dimatikan), dan Type (Default vs Report page).
- Tips konten: taruh **measure pembanding** di tooltip, bukan duplikat data label — pembaca mendapat "second layer" informasi tanpa menambah clutter chart.

## Report Page Tooltip (Tooltip Page)

### Setup halaman tooltip
1. Buat halaman baru → klik kanan tab → rename "Tooltip".
2. **Format halaman → Page information → Allow use as tooltip = ON**.
3. **Page size → Type = Tooltip** — ukuran default **320×240** (boleh custom, misal 400×280; jangan terlalu besar, tooltip harus melengkapi bukan menutup chart).
4. Bangun isinya: card/mini visual ringan; field yang ingin diteruskan (misal `Resto[Nama]`) ditaruh di well **Drill through** halaman tooltip agar konteks hover dikenali.
5. Halaman tooltip otomatis tidak tampil di daftar tab sebagai halaman biasa; bisa juga di-hide manual untuk kebersihan.

### Mengaitkan ke visual
- Pilih visual sumber → panel **Format → General → Properties → Tooltip card** → **Type: Report page** → pilih halaman tooltip.
- Satu tooltip page bisa dipakai banyak visual lintas halaman — report user memakai satu halaman Tooltip untuk konteks hover konsisten di seluruh report.
- Bila Type dibiarkan Default, visual memakai tooltip bawaan meskipun tooltip page tersedia — kesalahan kaitan paling umum.

### Perilaku
- Tooltip mengikuti kursor dan tetap terbuka selama kursor berada di atas visual — pengguna bisa mengarahkan kursor ke **dalam** tooltip dan berinteraksi (hover/memilih) dengan visual di dalamnya (interactive tooltip).
- Field di Drill through well halaman tooltip menangkap nilai dari data point yang di-hover — filter konteks diterapkan ke semua measure di dalam tooltip page.

## Sintaks & Contoh — Hover Card Restoran

Pola halaman "Tooltip" milik user (3 visual ringan) — hover pada bar/point resto menampilkan:

```text
Halaman "Tooltip" (320×240, allow as tooltip = ON)
  Drill through well : Resto[Nama]
  Visual 1 : Card  → [Deviasi YTD]   (callout besar + warna kondisional merah/hijau)
  Visual 2 : Card  → [Growth %]      (label "Growth vs LM", ikon panah via SVG)
  Visual 3 : Card  → [Z Score]       (badge anomali: High / Normal / Low)
Latar     : background senada theme, border tipis
Kaitkan di: bar chart Deviasi by Resto, scatter Sales vs Waste, matrix per area
```

Pengguna membandingkan tiga angka kunci dalam sekali hover — tanpa berpindah halaman — lalu klik kanan/drillthrough bila butuh analisa penuh. Judul mini di dalam tooltip bisa dibuat dinamis:

```dax
Tooltip Title =
    SELECTEDVALUE ( Resto[Nama], "Resto" ) & " — periode " &
    SELECTEDVALUE ( 'Periode'[Nama], "berjalan" )
```

## Kapan Tooltip, Kapan Drillthrough?

| Kebutuhan | Solusi |
|---|---|
| Nilai pelengkap instan (Z-Score, pembanding LM) | Tooltip field well |
| 2–4 angka kunci + konteks per entitas | Tooltip page |
| Analisa penuh per resto/item (matrix, tren, komposisi) | Drillthrough page |
| Definisi measure / dokumentasi singkat | Tooltip pada card header |

Aturan praktis: **tooltip = sekali hover, drillthrough = satu keputusan analisa**. Jangan memaksakan halaman drillthrough penuh ke dalam tooltip 320×240.

## Performa Tooltip

- Tooltip dievaluasi **saat hover** — measure berat (time intelligence bertingkat, iterator besar, `CALCULATE` berulang) membuat tiap hover terasa lambat dan pengguna menganggap report-nya yang lambat.
- Batasi visual di tooltip page (2–4) dan hindari visual berjuta data point (scatter penuh, tabel detail).
- Tooltip page yang dipakai lintas banyak visual harus paling ramping di antara semua halaman — dia dirender paling sering.
- Ukur dengan Performance Analyzer: mulai recording → hover ke tiap chart → pastikan durasi DAX query tooltip wajar (< ~500 ms di model sedang).

## Tooltip di Mobile

- Di aplikasi mobile Power BI, **tap** pada data point menampilkan tooltip (default maupun report page tooltip).
- Desain touch-friendly: elemen minimal, angka besar; hindari mini chart yang butuh presisi hover.
- Uji lewat View → Mobile layout — tooltip page yang bagus di desktop bisa terlalu kecil/terpotong di layar ponsel; sesuaikan ukuran callout.
- Di mobile, interaksi "masuk ke dalam tooltip" lebih terbatas — prioritaskan angka, bukan interaktivitas.

## Kesalahan Umum

- **Menaruh measure berat di tooltip** → tiap hover memicu query lambat; pindahkan yang berat ke halaman drillthrough saja.
- **Tooltip page penuh visual** (5+) → render lambat dan kecil tidak terbaca; 2–4 visual ringan sudah ideal.
- **Lupa Allow use as tooltip = ON** → halaman tidak muncul di pilihan Type: Report page.
- **Lupa menaruh field konteks di Drill through well halaman tooltip** → tooltip menampilkan agregat seluruh data, bukan resto yang di-hover.
- **Type tooltip dibiarkan Default** padahal tooltip page sudah dibuat → visual tidak memakai halaman yang sudah susah payah didesain.
- **Ukuran tooltip page terlalu besar** → menutup chart yang sedang dianalisa; patuhi kisaran ~320×240 s/d ~500px.
- **Isi tooltip menduplikasi data labels** yang sudah tampil di chart → sia-sia; tooltip untuk informasi tambahan (pembanding, skor, konteks).

## Tips Pro

- Standarkan satu tooltip page per entitas (resto, item, PIC) dan pakai lintas halaman — konsistensi membuat pengguna hafal pola "hover = tiga angka kunci".
- Beri warna kondisional pada card di tooltip (merah loss / hijau surplus) supaya sinyal terbaca sekilas tanpa membaca angka.
- Tooltip page dan drillthrough page berbagi field well yang sama — desain keduanya berdampingan agar konteks konsisten.
- Untuk KPI header, tooltip pada card bisa menjelaskan definisi measure ("Deviasi CM = selisih stok teoritis vs aktual bulan berjalan") — edukasi tanpa dokumentasi terpisah.
- Uji tooltip di layar kecil (laptop 13") dan di titik data tepi kanan chart — posisi munculnya bisa terpotong layar.
- Simpan halaman tooltip sebagai bagian template report (pbit) supaya report baru langsung punya pola hover standar.

## Terkait

- [Drillthrough](./03-drillthrough.md)
- [Page Layout](./01-page-layout.md)
- [Visual Configuration (Format Pane)](../04-visuals/03-visual-configuration.md)
- [Conditional Formatting](../04-visuals/04-conditional-formatting.md)
- [Katalog Visual Bawaan](../04-visuals/01-native-visuals-catalog.md)
- [Struktur halaman report user](../11-user-report-analysis/pages-and-visuals.md)
