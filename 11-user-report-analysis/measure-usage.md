# Index Penggunaan Measure & Kolom — Analisis Dampak (Impact Analysis)

> **Jawab pertanyaan: "Kalau aku hapus/ubah measure X, apa saja yang kena?"**
> Dibangun otomatis dari [pages-detail.md](./pages-detail.md) (lokasi pemakaian visual) + [model-and-dax.md](./model-and-dax.md) (definisi & referensi antar measure). Audit 7 Nov 2026, snapshot pbix 19 Okt 2026.

## Cara Pakai — Sebelum Hapus Measure, Cek 3 Hal

1. **Kolom "Visual" & "Halaman"** → visual mana yang langsung rusak (error "measure not found") di halaman mana.
2. **Kolom "Direferensikan oleh"** → measure lain yang ekspresinya memanggil measure ini → ikut rusak berantai (impact tidak langsung).
3. Kalau dua-duanya kosong → measure tidak dipakai mana-mana → aman dihapus.

> Catatan: measure yang tidak dipakai visual TAPI direferensikan measure lain = **jangan dihapus** sebelum measure pemanggilnya dirapikan. Sebaliknya, measure yang dipakai visual tapi tidak direferensikan measure lain = dampaknya terbatas ke visual itu saja.

## Ringkasan Cepat — Measure Paling "Hidup" (dipakai ≥ 5 visual)

| Measure | Tabel | Visual | Halaman |
|---|---|---|---|
| `Growth` | Deviasi All | 21 | Top 5 Growth (6), Kontribusi Dev (2), Analisa Susut (2), Summary Deviasi (1), Detail Deviasi (1), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), Perbandingan (1), Growth by Resto (1), Growth by Item (1), COM (1) |
| `Deviasi CM` | Deviasi All | 19 | Top 5 Growth (6), Summary Deviasi (2), Deviasi vs BOM (2), Detail Deviasi (1), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), Growth by Item (1), COM (1), Data Deviasi (1) |
| `% BOM CM` | Deviasi All | 17 | Top 10 Item Dev (5), Rekap Brg Masuk (3), Summary Deviasi (2), Deviasi vs BOM (2), Analisa Susut (2), Deviasi by PIC (1), Deviasi by Item (1), Antar Periode (1) |
| `BOM CM` | Deviasi All | 16 | Top 10 Item Dev (5), Rekap Brg Masuk (3), Deviasi vs BOM (2), Analisa Susut (2), Summary Deviasi (1), Deviasi by PIC (1), Deviasi by Item (1), Antar Periode (1) |
| `ABS Deviasi` | Deviasi All | 15 | Summary Deviasi (3), Deviasi by PIC (2), Kontribusi Dev (2), Top 10 Item Dev (2), Deviasi by Area (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), Growth by Item (1), COM (1) |
| `Deviasi LM` | Deviasi All | 13 | Top 5 Growth (6), Detail Deviasi (1), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `Selisih` | Deviasi All | 13 | Top 5 Growth (6), Detail Deviasi (1), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `Perubahan` | Deviasi All | 12 | Top 5 Growth (6), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `Nominal Dev` | Deviasi All | 9 | Top 10 Item Dev (8), Top 5 Waste (1) |
| `Clean BM` | Rekap Mentah BM | 9 | ID Dokumen (4), Tooltip (3), Detail Brg Masuk (1), Rekon Brg Masuk (1) |
| `% GT Deviasi` | Deviasi All | 8 | Top 10 Item Dev (5), Rekap Brg Masuk (3) |
| `Growth YoY` | Deviasi All | 7 | Summary Deviasi (1), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `Deviasi YTD` | Deviasi All | 7 | Detail Deviasi (1), Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `% BOM CM Abs` | Deviasi All | 7 | Analisa Susut (3), Deviasi vs BOM (2), Deviasi by PIC (1), Growth by Item (1) |
| `% to Total` | Waste All | 7 | Top 5 Waste (6), Waste by Item (1) |
| `Nominal Waste` | Waste All | 7 | Top 5 Waste (7) |
| `Selisih BM` | Rekap Mentah BM | 7 | Rekap Brg Masuk (4), Detail Brg Masuk (1), Rekon Brg Masuk (1), ID Dokumen (1) |
| `Analisa Deviasi` | Deviasi All | 6 | Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `Deviasi YoY` | Deviasi All | 6 | Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1), COM (1) |
| `% to Subtotal` | Waste All | 6 | Top 5 Waste (6) |
| `% Sales CM` | Sales All | 5 | Summary Deviasi (2), Deviasi by Area (1), Deviasi by Resto (1), COM (1) |
| `Sales LM` | Sales All | 5 | Top 5 Growth (2), Deviasi by Area (1), Deviasi by Resto (1), COM (1) |
| `Sales GAP` | Sales All | 5 | Top 5 Growth (2), Deviasi by Area (1), Deviasi by Resto (1), COM (1) |
| `% Sales Growth` | Sales All | 5 | Top 5 Growth (2), Deviasi by Area (1), Deviasi by Resto (1), COM (1) |
| `Show Value` | Deviasi All | 5 | Deviasi by Area (1), Deviasi by PIC (1), Deviasi by Resto (1), Deviasi by Item (1), COM (1) |
| `Kategori` | Slicer Trend | 5 | Deviasi by Area (1), Deviasi by Resto (1), Deviasi by Item (1), Growth by Resto (1), Last 13 Months (1) |
| `Multi Value CM` | Deviasi All | 5 | Deviasi by Area (1), Deviasi by Resto (1), Deviasi by Item (1), Growth by Resto (1), Last 13 Months (1) |
| `% Dev Contribution` | Deviasi All | 5 | Kontribusi Dev (2), Deviasi by Resto (1), Deviasi by Item (1), Antar Periode (1) |

## Tabel Lengkap Measure (urut pemakaian visual terbanyak)

| Measure | Tabel | Visual | Halaman (jumlah visual) | Direferensikan oleh | Referensi ke |
|---|---|---|---|---|---|
| `Growth` | Deviasi All | 21 | Top 5 Growth (6)<br>Kontribusi Dev (2)<br>Analisa Susut (2)<br>Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Perbandingan (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>COM (1) | — | Deviasi LM<br>Selisih |
| `Deviasi CM` | Deviasi All | 19 | Top 5 Growth (6)<br>Summary Deviasi (2)<br>Deviasi vs BOM (2)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Growth by Item (1)<br>COM (1)<br>Data Deviasi (1) | % BOM CM<br>% Sales CM<br>A Deviasi<br>ABS Deviasi<br>Analisa Deviasi<br>Avg Dev<br>B Deviasi<br>COM CM<br>Deviasi YTD<br>Deviasi YoY<br>Dynamic Dev<br>EoM<br>Kategori<br>Kategori L/S<br>Loss/Surplus<br>Multi Value CM<br>Perubahan<br>Selisih<br>Selisih YoY<br>Show<br>Show Value<br>Susut<br>Top10Dev<br>Trial<br>Usage<br>Waste<br>Week 1<br>Week 2<br>Week 3 | Deviasi Value |
| `% BOM CM` | Deviasi All | 17 | Top 10 Item Dev (5)<br>Rekap Brg Masuk (3)<br>Summary Deviasi (2)<br>Deviasi vs BOM (2)<br>Analisa Susut (2)<br>Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | % BOM CM Abs<br>% BOM GAP<br>% BOM LM<br>% BOM YTD<br>% BOM YoY<br>A % BOM<br>B % BOM<br>Multi Value CM | BOM CM<br>Deviasi CM |
| `BOM CM` | Deviasi All | 16 | Top 10 Item Dev (5)<br>Rekap Brg Masuk (3)<br>Deviasi vs BOM (2)<br>Analisa Susut (2)<br>Summary Deviasi (1)<br>Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | % BOM CM<br>A BOM<br>B BOM<br>BOM ABS<br>BOM GAP<br>BOM YTD<br>BOM YoY<br>COM CM<br>Multi Value CM<br>Rank BOM<br>Show<br>Show Value | — |
| `ABS Deviasi` | Deviasi All | 15 | Summary Deviasi (3)<br>Deviasi by PIC (2)<br>Kontribusi Dev (2)<br>Top 10 Item Dev (2)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Growth by Item (1)<br>COM (1) | % Dev Contribution<br>ABS LM<br>Rank Dev<br>Top10Dev | Deviasi CM |
| `Deviasi LM` | Deviasi All | 13 | Top 5 Growth (6)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | % Sales LM<br>Analisa Deviasi<br>Growth<br>Perubahan<br>Selisih<br>Show<br>Show Value | Deviasi Value |
| `Selisih` | Deviasi All | 13 | Top 5 Growth (6)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | Dynamic Gap Dev<br>Growth<br>Selisih Abs | Deviasi CM<br>Deviasi LM |
| `Perubahan` | Deviasi All | 12 | Top 5 Growth (6)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | — | Deviasi CM<br>Deviasi LM |
| `Nominal Dev` | Deviasi All | 9 | Top 10 Item Dev (8)<br>Top 5 Waste (1) | % GT Deviasi | Top10Dev |
| `Clean BM` | Rekap Mentah BM | 9 | ID Dokumen (4)<br>Tooltip (3)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1) | — | Pembelian di Luar |
| `% GT Deviasi` | Deviasi All | 8 | Top 10 Item Dev (5)<br>Rekap Brg Masuk (3) | — | Nominal Dev |
| `Growth YoY` | Deviasi All | 7 | Summary Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | — | Deviasi YoY<br>Selisih YoY |
| `Deviasi YTD` | Deviasi All | 7 | Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | % Sales YTD<br>Show<br>Show Value | Deviasi CM |
| `% BOM CM Abs` | Deviasi All | 7 | Analisa Susut (3)<br>Deviasi vs BOM (2)<br>Deviasi by PIC (1)<br>Growth by Item (1) | Analisa Susut | % BOM CM |
| `% to Total` | Waste All | 7 | Top 5 Waste (6)<br>Waste by Item (1) | — | Nominal Waste |
| `Nominal Waste` | Waste All | 7 | Top 5 Waste (7) | % to Subtotal<br>% to Total<br>Kontribusi Waste | — |
| `Selisih BM` | Rekap Mentah BM | 7 | Rekap Brg Masuk (4)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) | ABS Selisih BM<br>Input Benar<br>Input Salah<br>Show BM | Pembelian di Luar |
| `Analisa Deviasi` | Deviasi All | 6 | Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | — | Deviasi CM<br>Deviasi LM<br>Show |
| `Deviasi YoY` | Deviasi All | 6 | Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>COM (1) | % Sales YoY<br>Growth YoY<br>Selisih YoY<br>Show<br>Show Value | Deviasi CM |
| `% to Subtotal` | Waste All | 6 | Top 5 Waste (6) | — | Nominal Waste |
| `% Sales CM` | Sales All | 5 | Summary Deviasi (2)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | % Sales GAP<br>A % Sales<br>B % Sales<br>Multi Value CM | Deviasi CM<br>Sales CM |
| `Sales LM` | Sales All | 5 | Top 5 Growth (2)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | % Sales Growth<br>% Sales LM<br>Sales GAP<br>Sales sd 07 Bulan Lalu<br>Sales sd 14 Bulan Lalu<br>Sales sd 25 Bulan Lalu<br>Sales sd EOM Bulan Lalu | Sales CM |
| `Sales GAP` | Sales All | 5 | Top 5 Growth (2)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | % Sales Growth | Sales CM<br>Sales LM |
| `% Sales Growth` | Sales All | 5 | Top 5 Growth (2)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | — | Sales GAP<br>Sales LM |
| `Show Value` | Deviasi All | 5 | Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>COM (1) | — | BOM CM<br>BOM LM<br>BOM YTD<br>BOM YoY<br>Deviasi CM<br>Deviasi LM<br>Deviasi YTD<br>Deviasi YoY |
| `Kategori` | Slicer Trend | 5 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Growth by Resto (1)<br>Last 13 Months (1) | Multi Value CM | Deviasi CM |
| `Multi Value CM` | Deviasi All | 5 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Growth by Resto (1)<br>Last 13 Months (1) | Multi Value Gap<br>Multi Value LM<br>Perubahan Multi Value<br>Z Score | % BOM CM<br>% Sales CM<br>% Sales GAP<br>BOM CM<br>Deviasi CM<br>Kategori |
| `% Dev Contribution` | Deviasi All | 5 | Kontribusi Dev (2)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | — | ABS Deviasi |
| `Sales CM` | Sales All | 4 | Top 5 Growth (2)<br>Summary Deviasi (1)<br>Data Sales (1) | % Sales CM<br>A Sales<br>B Sales<br>Sales GAP<br>Sales LM<br>Sales YTD<br>Sales YoY<br>Sales sd 07 Bulan Ini<br>Sales sd 14 Bulan Ini<br>Sales sd 25 Bulan Ini<br>Sales sd EOM Bulan Ini | — |
| `Kontribusi CM` | Deviasi All | 4 | Detail Deviasi (4) | Kontribusi CM EOM<br>Kontribusi CM Week 1<br>Kontribusi CM Week 2<br>Kontribusi CM Week 3<br>Kontribusi GAP<br>Kontribusi LM | % Loss/Suprlus<br>% Susut<br>% Trial<br>% Usage<br>% Waste |
| `Susut` | Deviasi All | 4 | Kontribusi Dev (2)<br>Analisa Susut (2) | % Susut<br>Deviasi Total | Deviasi CM |
| `Pembelian di Luar` | 9901 | 4 | Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) | BM Sistem<br>Clean BM<br>Selisih BM | — |
| `Nilai PI` | 3243 | 4 | Rekap PO blm PI (3)<br>Detail PO blm PI (1) | Belum Dibayar<br>Status PO | — |
| `Belum Dibayar` | 3243 | 4 | Rekap PO blm PI (3)<br>Detail PO blm PI (1) | — | Nilai PI |
| `Week 1` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | Growth W2<br>Perubahan W2<br>Week 1 Abs | Deviasi CM |
| `Week 2` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | Growth EOM<br>Growth W2<br>Growth W3<br>Perubahan EOM<br>Perubahan W2<br>Perubahan W3<br>Week 2 Abs | Deviasi CM |
| `Growth W2` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | — | Week 1<br>Week 2 |
| `Week 3` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | Growth W3<br>Perubahan W3<br>Week 3 Abs | Deviasi CM |
| `Growth W3` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | — | Week 2<br>Week 3 |
| `EoM` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | EoM Abs<br>Growth EOM<br>Perubahan EOM | Deviasi CM |
| `Growth EOM` | Deviasi All | 3 | Detail Deviasi (1)<br>Growth by Resto (1)<br>Growth by Item (1) | — | EoM<br>Week 2 |
| `Show Detail` | Deviasi All | 3 | Detail Deviasi (3) | — | — |
| `% Sales LM` | Sales All | 3 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | % Sales GAP | Deviasi LM<br>Sales LM |
| `% Sales GAP` | Sales All | 3 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | Multi Value CM | % Sales CM<br>% Sales LM |
| `Sales YTD` | Sales All | 3 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | % Sales YTD | Sales CM |
| `% Sales YTD` | Sales All | 3 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) | — | Deviasi YTD<br>Sales YTD |
| `Multi Value LM` | Deviasi All | 3 | Deviasi by Area (1)<br>Growth by Resto (1)<br>COM (1) | Multi Value Gap<br>Multi Value Growth<br>Perubahan Multi Value | Multi Value CM |
| `Multi Value Gap` | Deviasi All | 3 | Deviasi by Area (1)<br>Growth by Resto (1)<br>COM (1) | Multi Value Growth | Multi Value CM<br>Multi Value LM |
| `Multi Value Growth` | Deviasi All | 3 | Deviasi by Area (1)<br>Growth by Resto (1)<br>COM (1) | — | Multi Value Gap<br>Multi Value LM |
| `BOM LM` | Deviasi All | 3 | Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | BOM GAP<br>Show<br>Show Value | — |
| `% BOM LM` | Deviasi All | 3 | Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | % BOM GAP<br>Growth % BOM | % BOM CM |
| `BOM GAP` | Deviasi All | 3 | Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | — | BOM CM<br>BOM LM |
| `% BOM GAP` | Deviasi All | 3 | Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | Growth % BOM | % BOM CM<br>% BOM LM |
| `BOM YTD` | Deviasi All | 3 | Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | Show<br>Show Value | BOM CM |
| `% BOM YTD` | Deviasi All | 3 | Deviasi by PIC (1)<br>Deviasi by Item (1)<br>Antar Periode (1) | — | % BOM CM |
| `Waste` | Deviasi All | 3 | Kontribusi Dev (2)<br>Data Waste (1) | % Waste<br>Cek Selisih Waste<br>Deviasi Total | Deviasi CM |
| `% Waste` | Deviasi All | 3 | Kontribusi Dev (3) | % Deviasi<br>Kontribusi CM | Deviasi Total<br>Waste |
| `% Susut` | Deviasi All | 3 | Kontribusi Dev (3) | % Deviasi<br>Kontribusi CM | Deviasi Total<br>Susut |
| `% Trial` | Deviasi All | 3 | Kontribusi Dev (3) | % Deviasi<br>Kontribusi CM | Deviasi Total<br>Trial |
| `% Loss/Suprlus` | Deviasi All | 3 | Kontribusi Dev (3) | % Deviasi<br>Kontribusi CM | Deviasi Total<br>Loss/Surplus |
| `COM CM` | Deviasi All | 3 | COM (3) | COM Gap<br>COM LM | BOM CM<br>Deviasi CM |
| `Selisih Abs` | Deviasi All | 2 | Summary Deviasi (2) | — | Selisih |
| `Resto YoY` | Deviasi All | 2 | Deviasi by Area (1)<br>COM (1) | — | — |
| `Perubahan Multi Value` | Deviasi All | 2 | Deviasi by Area (1)<br>COM (1) | — | Multi Value CM<br>Multi Value LM |
| `BOM ABS` | Deviasi All | 2 | Deviasi by PIC (1)<br>Growth by Item (1) | — | BOM CM |
| `Trial` | Deviasi All | 2 | Kontribusi Dev (2) | % Trial<br>Deviasi Total | Deviasi CM |
| `Loss/Surplus` | Deviasi All | 2 | Kontribusi Dev (2) | % Loss/Suprlus<br>Deviasi Total | Deviasi CM |
| `Perubahan W2` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | Week 1<br>Week 2 |
| `Perubahan W3` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | Week 2<br>Week 3 |
| `Perubahan EOM` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | EoM<br>Week 2 |
| `Week 1 Abs` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | Week 1 |
| `Week 2 Abs` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | Week 2 |
| `Week 3 Abs` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | Week 3 |
| `EoM Abs` | Deviasi All | 2 | Growth by Resto (1)<br>Growth by Item (1) | — | EoM |
| `Analisa Susut` | Deviasi All | 2 | Analisa Susut (2) | — | % BOM CM Abs |
| `Top10Dev` | Deviasi All | 2 | Top 10 Item Dev (2) | Nominal Dev<br>QTY Dev | ABS Deviasi<br>Deviasi CM |
| `Last Update BM` | Update | 1 | Home (1) | — | — |
| `Last Update Dev` | Update | 1 | Home (1) | — | — |
| `Update on Dev` | Update | 1 | Home (1) | — | — |
| `Update on BM` | Update | 1 | Home (1) | — | — |
| `Kontribusi LM` | Deviasi All | 1 | Detail Deviasi (1) | Kontribusi GAP | Kontribusi CM |
| `Rank Dev` | Deviasi All | 1 | Deviasi by Item (1) | — | ABS Deviasi |
| `Rank BOM` | Deviasi All | 1 | Deviasi by Item (1) | — | BOM CM |
| `Show` | Deviasi All | 1 | Antar Periode (1) | Analisa Deviasi | BOM CM<br>BOM LM<br>BOM YTD<br>BOM YoY<br>Deviasi CM<br>Deviasi LM<br>Deviasi YTD<br>Deviasi YoY |
| `A Deviasi` | Inventory A | 1 | Perbandingan (1) | GAP ABS Dev<br>GAP Deviasi | Deviasi CM |
| `A BOM` | Inventory A | 1 | Perbandingan (1) | GAP BOM | BOM CM |
| `A % BOM` | Inventory A | 1 | Perbandingan (1) | GAP % BOM | % BOM CM |
| `A Sales` | Inventory A | 1 | Perbandingan (1) | GAP Sales | Sales CM |
| `A % Sales` | Inventory A | 1 | Perbandingan (1) | GAP % Sales | % Sales CM |
| `B Deviasi` | Inventory A | 1 | Perbandingan (1) | GAP ABS Dev<br>GAP Deviasi | Deviasi CM |
| `B BOM` | Inventory A | 1 | Perbandingan (1) | GAP BOM | BOM CM |
| `B % BOM` | Inventory A | 1 | Perbandingan (1) | GAP % BOM | % BOM CM |
| `B Sales` | Inventory A | 1 | Perbandingan (1) | GAP Sales | Sales CM |
| `B % Sales` | Inventory A | 1 | Perbandingan (1) | GAP % Sales | % Sales CM |
| `GAP Deviasi` | Inventory A | 1 | Perbandingan (1) | — | A Deviasi<br>B Deviasi |
| `GAP ABS Dev` | Inventory A | 1 | Perbandingan (1) | — | A Deviasi<br>B Deviasi |
| `GAP BOM` | Inventory A | 1 | Perbandingan (1) | — | A BOM<br>B BOM |
| `GAP % BOM` | Inventory A | 1 | Perbandingan (1) | — | A % BOM<br>B % BOM |
| `GAP Sales` | Inventory A | 1 | Perbandingan (1) | — | A Sales<br>B Sales |
| `GAP % Sales` | Inventory A | 1 | Perbandingan (1) | — | A % Sales<br>B % Sales |
| `Waste LM` | Waste All | 1 | Waste by Resto (1) | Waste Gap<br>Waste Growth | — |
| `Waste Gap` | Waste All | 1 | Waste by Resto (1) | Waste Growth | Waste LM |
| `Waste Growth` | Waste All | 1 | Waste by Resto (1) | — | Waste Gap<br>Waste LM |
| `Waste YTD` | Waste All | 1 | Waste by Resto (1) | — | — |
| `COM LM` | Deviasi All | 1 | COM (1) | COM Gap<br>COM Growth | COM CM |
| `COM Gap` | Deviasi All | 1 | COM (1) | COM Growth | COM CM<br>COM LM |
| `COM Growth` | Deviasi All | 1 | COM (1) | — | COM Gap<br>COM LM |
| `Count BM Resto` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | % BM<br>Input Benar<br>Input Salah | BM Resto |
| `Count BM Sistem` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | % BM<br>Input Benar<br>Input Salah | BM Sistem |
| `% BM` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | Show BM | Count BM Resto<br>Count BM Sistem |
| `Show BM` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | — | % BM<br>Selisih BM |
| `Input Benar` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | % BM Benar<br>Total BM | Count BM Resto<br>Count BM Sistem<br>Selisih BM |
| `Input Salah` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | % BM Salah<br>Total BM | Count BM Resto<br>Count BM Sistem<br>Selisih BM |
| `Total BM` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | % BM Benar<br>% BM Salah | Input Benar<br>Input Salah |
| `% BM Benar` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | — | Input Benar<br>Total BM |
| `% BM Salah` | Rekap Mentah BM | 1 | Rekon Brg Masuk (1) | — | Input Salah<br>Total BM |
| `Sales sd 07 Bulan Lalu` | Sales All | 1 | Sales Growth (1) | % Sales Growth 07 | Sales LM |
| `Sales sd 07 Bulan Ini` | Sales All | 1 | Sales Growth (1) | % Sales Growth 07 | Sales CM |
| `% Sales Growth 07` | Sales All | 1 | Sales Growth (1) | — | Sales sd 07 Bulan Ini<br>Sales sd 07 Bulan Lalu |
| `Sales sd 14 Bulan Lalu` | Sales All | 1 | Sales Growth (1) | % Sales Growth 14 | Sales LM |
| `Sales sd 14 Bulan Ini` | Sales All | 1 | Sales Growth (1) | % Sales Growth 14 | Sales CM |
| `% Sales Growth 14` | Sales All | 1 | Sales Growth (1) | — | Sales sd 14 Bulan Ini<br>Sales sd 14 Bulan Lalu |
| `Sales sd 25 Bulan Lalu` | Sales All | 1 | Sales Growth (1) | % Sales Growth 25 | Sales LM |
| `Sales sd 25 Bulan Ini` | Sales All | 1 | Sales Growth (1) | % Sales Growth 25 | Sales CM |
| `% Sales Growth 25` | Sales All | 1 | Sales Growth (1) | — | Sales sd 25 Bulan Ini<br>Sales sd 25 Bulan Lalu |
| `Sales sd EOM Bulan Lalu` | Sales All | 1 | Sales Growth (1) | % Sales Growth EOM | Sales LM |
| `Sales sd EOM Bulan Ini` | Sales All | 1 | Sales Growth (1) | % Sales Growth EOM | Sales CM |
| `% Sales Growth EOM` | Sales All | 1 | Sales Growth (1) | — | Sales sd EOM Bulan Ini<br>Sales sd EOM Bulan Lalu |
| `Nama Barang 9901` | 3243 | 1 | Detail PO blm PI (1) | — | — |
| `Status PO` | 3243 | 1 | Detail PO blm PI (1) | — | Nilai PI |
| `Cek Selisih Waste` | Waste All | 1 | Data Waste (1) | — | Waste |

## Measure TIDAK Dipakai Visual Langsung (30 buah)

Perlu dicek kolom "Direferensikan oleh" di tabel atas — kalau juga tidak direferensikan, kandidat pembersihan model.

- `% BOM YoY` — **tidak direferensikan siapa pun** (kandidat hapus)
- `% Deviasi` — **tidak direferensikan siapa pun** (kandidat hapus)
- `% Sales YoY` — **tidak direferensikan siapa pun** (kandidat hapus)
- `% Usage` — dipanggil oleh: Kontribusi CM
- `ABS LM` — **tidak direferensikan siapa pun** (kandidat hapus)
- `ABS Selisih BM` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Avg Dev` — **tidak direferensikan siapa pun** (kandidat hapus)
- `BM Resto` — dipanggil oleh: Count BM Resto
- `BM Sistem` — dipanggil oleh: Count BM Sistem
- `BOM YoY` — dipanggil oleh: Show, Show Value
- `Deviasi Total` — dipanggil oleh: % Loss/Suprlus, % Susut, % Trial, % Usage, % Waste
- `Deviasi Value` — dipanggil oleh: Deviasi CM, Deviasi LM
- `Dynamic Dev` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Dynamic Gap Dev` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Growth % BOM` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kategori Anomali Wasutri` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kategori L/S` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kontribusi CM EOM` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kontribusi CM Week 1` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kontribusi CM Week 2` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kontribusi CM Week 3` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kontribusi GAP` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Kontribusi Waste` — **tidak direferensikan siapa pun** (kandidat hapus)
- `QTY Dev` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Qty Waste` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Sales YoY` — dipanggil oleh: % Sales YoY
- `Selected Cutoff Date` — **tidak direferensikan siapa pun** (kandidat hapus)
- `Selisih YoY` — dipanggil oleh: Growth YoY
- `Usage` — dipanggil oleh: % Usage
- `Z Score` — dipanggil oleh: Kategori Anomali Wasutri

## Kolom yang Dipakai Visual (non-measure)

| Kolom | Visual | Halaman (jumlah visual) |
|---|---|---|
| `Inventory[Nama Barang]` | 61 | Top 5 Growth (6)<br>Top 5 Waste (6)<br>Top 10 Item Dev (5)<br>Rekap Brg Masuk (4)<br>Summary Deviasi (3)<br>Deviasi by Item (3)<br>Deviasi vs BOM (3)<br>Analisa Susut (3)<br>COM (3)<br>Deviasi by Resto (2)<br>Antar Periode (2)<br>Kontribusi Dev (2)<br>Growth by Resto (2)<br>Growth by Item (2)<br>Last 13 Months (2)<br>Waste by Resto (2)<br>Waste by Item (2)<br>Detail Brg Masuk (2)<br>Rekon Brg Masuk (2)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>ID Dokumen (1)<br>Kosongan (1) |
| `Resto[Resto]` | 54 | Summary Deviasi (3)<br>Deviasi by Resto (3)<br>Analisa Susut (3)<br>Top 5 Growth (3)<br>Rekon Brg Masuk (3)<br>Deviasi by Area (2)<br>Deviasi by PIC (2)<br>Antar Periode (2)<br>Kontribusi Dev (2)<br>Deviasi vs BOM (2)<br>Growth by Resto (2)<br>Growth by Item (2)<br>Last 13 Months (2)<br>Waste by Resto (2)<br>COM (2)<br>Waste by Item (2)<br>Detail Brg Masuk (2)<br>Detail Deviasi (1)<br>Deviasi by Item (1)<br>Top 10 Item Dev (1)<br>Top 5 Waste (1)<br>Rekap Brg Masuk (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>ID Dokumen (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Resto[Area]` | 35 | Analisa Susut (3)<br>Deviasi by Area (2)<br>Sales Growth (2)<br>Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Inventory[Tipe]` | 34 | Detail Deviasi (6)<br>Perbandingan (5)<br>Top 5 Growth (2)<br>Summary Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Data Deviasi (1)<br>Kosongan (1) |
| `Deviasi All[Value]` | 34 | Summary Deviasi (9)<br>Top 10 Item Dev (5)<br>Rekap Brg Masuk (5)<br>Detail Deviasi (2)<br>Waste by Resto (2)<br>COM (2)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Kontribusi Dev (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Top 5 Waste (1)<br>Waste by Item (1) |
| `Resto[PIC]` | 33 | Deviasi by PIC (2)<br>Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Deviasi All[Jenis]` | 32 | Top 10 Item Dev (9)<br>Detail Deviasi (6)<br>Summary Deviasi (2)<br>Deviasi vs BOM (2)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>Waste by Item (1)<br>Kosongan (1) |
| `Slicer Resto[PIC]` | 32 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Slicer Resto[Area]` | 31 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Date[Date]` | 31 | ID Dokumen (4)<br>Summary Deviasi (2)<br>Deviasi by PIC (2)<br>Deviasi by Resto (2)<br>Deviasi by Item (2)<br>Growth by Item (2)<br>COM (2)<br>Detail Brg Masuk (2)<br>Rekon Brg Masuk (2)<br>Data Waste (2)<br>Deviasi by Area (1)<br>Kontribusi Dev (1)<br>Growth by Resto (1)<br>Analisa Susut (1)<br>Waste by Resto (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Data Deviasi (1)<br>Data Sales (1) |
| `Slicer Resto[Resto]` | 30 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Resto[AM]` | 29 | Deviasi by Area (2)<br>Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Kosongan (1) |
| `Attribute[Attribute]` | 29 | Top 10 Item Dev (4)<br>Top 5 Growth (3)<br>Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>End (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Slicer Resto[AM]` | 28 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Kosongan (1) |
| `Periode[Periode]` | 26 | Detail Deviasi (2)<br>Data Waste (2)<br>Summary Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Data Deviasi (1)<br>Data Sales (1)<br>Kosongan (1) |
| `Slicer Inventory[Tipe]` | 24 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Data Deviasi (1)<br>Kosongan (1) |
| `Date[Year]` | 22 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Kosongan (1) |
| `Date[Year] (hierarchy)` | 22 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Kosongan (1) |
| `Date[Month] (hierarchy)` | 22 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Analisa Susut (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Sales Growth (1)<br>Rekap PO blm PI (1)<br>Detail PO blm PI (1)<br>Detail PI (1)<br>Kosongan (1) |
| `Slicer Attribute[Attribute]` | 22 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Top 5 Growth (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Data Deviasi (1)<br>Data Waste (1)<br>Data Sales (1)<br>Barang Baru (1)<br>Kosongan (1) |
| `Slicer Inventory[Nama Barang]` | 19 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>Kosongan (1) |
| `Slicer Periode[Periode]` | 19 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Kontribusi Dev (1)<br>Perbandingan (1)<br>Deviasi vs BOM (1)<br>Analisa Susut (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Top 5 Waste (1)<br>Waste by Resto (1)<br>COM (1)<br>Waste by Item (1)<br>Kosongan (1) |
| `Slicer Deviasi All[Jenis]` | 16 | Summary Deviasi (1)<br>Detail Deviasi (1)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Antar Periode (1)<br>Deviasi vs BOM (1)<br>Growth by Resto (1)<br>Growth by Item (1)<br>Last 13 Months (1)<br>Top 10 Item Dev (1)<br>Top 5 Growth (1)<br>Waste by Resto (1)<br>Waste by Item (1)<br>Kosongan (1) |
| `Waste All[Jenis]` | 12 | Top 5 Waste (7)<br>Waste by Item (3)<br>Waste by Resto (2) |
| `Resto[Restoo]` | 8 | Rekon Brg Masuk (2)<br>Deviasi by Area (1)<br>Deviasi by Resto (1)<br>Analisa Susut (1)<br>COM (1)<br>Detail Brg Masuk (1)<br>ID Dokumen (1) |
| `Slicer Date[Date]` | 7 | Deviasi by PIC (1)<br>Deviasi by Resto (1)<br>Deviasi by Item (1)<br>Growth by Item (1)<br>Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1) |
| `Waste All[Value]` | 7 | Waste by Resto (3)<br>Waste by Item (2)<br>Data Waste (2) |
| `Inventory[UoM]` | 6 | Rekap Brg Masuk (3)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) |
| `Deviasi All[Resto]` | 5 | Analisa Susut (2)<br>Deviasi by Area (1)<br>Deviasi by PIC (1)<br>COM (1) |
| `Inventory[(Old) Nama Barang]` | 4 | Deviasi by Resto (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) |
| `Rekap Mentah BM[Value]` | 4 | Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) |
| `4205[#Qty. Terkecil]` | 4 | Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) |
| `3224[Kts (Unit#1)]` | 4 | Rekap Brg Masuk (1)<br>Detail Brg Masuk (1)<br>Rekon Brg Masuk (1)<br>ID Dokumen (1) |
| `3243[Status Pesanan Pembelian]` | 4 | Rekap PO blm PI (2)<br>Detail PO blm PI (2) |
| `3243[Nama Barang]` | 4 | Detail PO blm PI (2)<br>Rekap PO blm PI (1)<br>Barang Baru (1) |
| `3243[Nama Kategori Barang Barang & Jasa]` | 4 | Rekap PO blm PI (2)<br>Detail PO blm PI (2) |
| `3243[Total Harga]` | 4 | Rekap PO blm PI (3)<br>Detail PO blm PI (1) |
| `Sales All[Penjualan Bersih]` | 3 | Deviasi by Area (1)<br>Deviasi by Resto (1)<br>COM (1) |
| `Resto[Area 2]` | 3 | Deviasi by Area (1)<br>Analisa Susut (1)<br>COM (1) |
| `9901[Nama Barang]` | 3 | Detail PI (2)<br>Barang Baru (1) |
| `9901[Nomor #]` | 3 | Detail PI (1)<br>Tooltip (1)<br>ID Dokumen (1) |
| `9901[Nomor # Penerimaan Barang]` | 3 | Detail PI (1)<br>Tooltip (1)<br>ID Dokumen (1) |
| `9901[#Purch.UoM]` | 3 | Detail PI (1)<br>Tooltip (1)<br>ID Dokumen (1) |
| `9901[#Purch.Qty]` | 3 | Detail PI (1)<br>Tooltip (1)<br>ID Dokumen (1) |
| `Resto[Team]` | 2 | Deviasi by Area (1)<br>COM (1) |
| `Date[Period]` | 2 | Perbandingan (1)<br>Last 13 Months (1) |
| `Resto[Type Kitchen]` | 2 | Analisa Susut (2) |
| `Standar Susut[Standar Susut]` | 2 | Analisa Susut (2) |
| `Inventory[Kategori 2]` | 2 | Detail Brg Masuk (1)<br>ID Dokumen (1) |
| `Slicer 3243[Status Pesanan Pembelian]` | 2 | Rekap PO blm PI (1)<br>Detail PO blm PI (1) |
| `Slicer 3243[Nama Barang]` | 2 | Rekap PO blm PI (1)<br>Detail PO blm PI (1) |
| `Slicer 3243[Nama Kategori Barang Barang & Jasa]` | 2 | Rekap PO blm PI (1)<br>Detail PO blm PI (1) |
| `3243[Cabang PO]` | 2 | Rekap PO blm PI (1)<br>Detail PO blm PI (1) |
| `3243[Nomor #]` | 2 | Detail PO blm PI (2) |
| `9901[Jenis Barang Barang & Jasa]` | 2 | Detail PI (1)<br>ID Dokumen (1) |
| `9901[Nomor # Pesanan Pembelian]` | 2 | Detail PI (2) |
| `4205[Nomor #Kirim]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `4205[Nomor #Terima]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `4205[#Qty Kirim]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `4205[#Sat. Kirim]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `3224[Nomor # RI]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `3224[Nomor # PO]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `3224[Kts Terima]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `3224[Satuan]` | 2 | Tooltip (1)<br>ID Dokumen (1) |
| `Slicer Date A[Period]` | 1 | Antar Periode (1) |
| `Date A[Period]` | 1 | Antar Periode (1) |
| `Date A[Date]` | 1 | Antar Periode (1) |
| `Slicer Date B[Period]` | 1 | Antar Periode (1) |
| `Date B[Period]` | 1 | Antar Periode (1) |
| `Date B[Date]` | 1 | Antar Periode (1) |
| `Slicer Inventory A[(Old) Nama Barang]` | 1 | Perbandingan (1) |
| `Inventory A[(Old) Nama Barang]` | 1 | Perbandingan (1) |
| `Slicer Resto A[Resto]` | 1 | Perbandingan (1) |
| `Resto A[Resto]` | 1 | Perbandingan (1) |
| `Slicer Inventory B[(Old) Nama Barang]` | 1 | Perbandingan (1) |
| `Inventory B[(Old) Nama Barang]` | 1 | Perbandingan (1) |
| `Slicer Resto B[Resto]` | 1 | Perbandingan (1) |
| `Resto B[Resto]` | 1 | Perbandingan (1) |
| `Slicer Deviasi All[Group Satuan]` | 1 | Top 5 Growth (1) |
| `Deviasi All[Group Satuan]` | 1 | Top 5 Growth (1) |
| `3243[Tanggal]` | 1 | Detail PO blm PI (1) |
| `3243[Pemasok]` | 1 | Detail PO blm PI (1) |
| `3243[Kode #]` | 1 | Detail PO blm PI (1) |
| `3243[Diskon]` | 1 | Detail PO blm PI (1) |
| `3243[Sisa Uang Muka Faktur Uang Muka Terakhir]` | 1 | Detail PO blm PI (1) |
| `3243[Keterangan PO]` | 1 | Detail PO blm PI (1) |
| `3243[Pembuat Data]` | 1 | Detail PO blm PI (1) |
| `Slicer 9901[Jenis Barang Barang & Jasa]` | 1 | Detail PI (1) |
| `Slicer 9901[Nama Barang]` | 1 | Detail PI (1) |
| `9901[Tanggal]` | 1 | Detail PI (1) |
| `9901[Nama Cabang]` | 1 | Detail PI (1) |
| `9901[Tanggal Penerimaan Barang]` | 1 | Detail PI (1) |
| `9901[Kode #]` | 1 | Detail PI (1) |
| `9901[#Purch.@Price]` | 1 | Detail PI (1) |
| `9901[#Purch.Discount]` | 1 | Detail PI (1) |
| `9901[#Purch.Total]` | 1 | Detail PI (1) |
| `3243[Cek Barang]` | 1 | Barang Baru (1) |
| `9901[Cek Barang]` | 1 | Barang Baru (1) |
| `3224[Nama Barang]` | 1 | Barang Baru (1) |
| `3224[Cek Barang]` | 1 | Barang Baru (1) |
| `4205[Nama Barang]` | 1 | Barang Baru (1) |
| `4205[Cek Barang]` | 1 | Barang Baru (1) |
| `Deviasi All[Nama Barang]` | 1 | Barang Baru (1) |
| `Deviasi All[Cek Barang]` | 1 | Barang Baru (1) |

## Glosarium Fungsi Measure Kunci

> Bagian ini hanya ringkasan measure kunci. **Glosarium lengkap SEMUA 161 measure** (fungsi bisnis + DAX inti + pemakaian + dependensi, 16 keluarga): [measure-glossary.md](./measure-glossary.md).

| Measure | Fungsi bisnis |
|---|---|
| `Deviasi CM / Deviasi LM` | Deviasi (selisih stok teoritis vs aktual) bulan berjalan / bulan lalu. LM otomatis PREVIOUSMONTH, tapi bisa dioverride via Mode DINAMIS (Date B). |
| `Selisih` | Deviasi CM − Deviasi LM (perubahan bulan-ke-bulan). |
| `Growth` | Selisih ÷ ABS(Deviasi LM) — % kenaikan/penurunan deviasi vs bulan lalu. |
| `Kontribusi CM / LM` | Porsi % suatu kategori terhadap total deviasi bulan berjalan / lalu (dipakai donut & pivot "Penyebab"). |
| `ABS Deviasi` | Nilai absolut deviasi — untuk ranking Top 5/Top 10 dan agregasi tanpa saling menghapus plus-minus. |
| `BOM CM / % BOM CM` | Nilai BOM (bill of material / material cost) bulan berjalan, dan rasio deviasi terhadap BOM — indikator kesehatan cost. |
| `Deviasi YTD / YoY / Growth YoY` | Agregasi year-to-date (dengan cutoff periode mingguan) dan perbandingan year-over-year. |
| `Week 1 / Week 2 / Week 3 / EoM + Growth W2/W3/EOM` | Nilai deviasi per periode mingguan (Week 1–3 + End of Month) dan pertumbuhannya antar minggu. |
| `Multi Value CM` | Measure switching: satu visual menampilkan DEVIASI / BOM / %BOM / %SALES / VARIANCE sesuai slicer Trend[Kategori]. |
| `Z Score` | Deteksi anomali deviasi per item (standar deviasi dari rata-rata historis). |
| `BM Sistem` | Barang masuk versi sistem = 3224 (penerimaan) + 4205 (transfer) + Pembelian di Luar. |
| `BM Resto` | Barang masuk versi input restoran = SUM(Rekap Mentah BM[Value]). |
| `Selisih BM` | BM Resto − BM Sistem — selisih input resto vs sistem (inti halaman Rekon Brg Masuk). |
| `Input Benar / Input Salah / Total BM / % BM` | Kualitas input BM per barang: hitungan barang yang inputannya cocok / tidak, dan rasio kecocokan. |
| `Pembelian di Luar` | Pembelian tanpa nomor penerimaan barang resmi (dari 9901, hanya jenis Persediaan). |
| `Update on BM / Last Update BM` | Timestamp update data barang masuk terakhir (dari tabel Update) — ditampilkan di header halaman. |
| `Show Detail / Show / Show Value` | Measure toggle untuk menyembunyikan baris/visual yang tidak relevan (dipakai sebagai filter = SHOW). |
| `A/B Deviasi, GAP Deviasi, dll (Inventory A)` | Keluarga measure pembanding halaman Perbandingan & Antar Periode: periode A vs B dipilih via slicer Inventory A/B + Resto A/B. |

## Terkait

- [model-and-dax.md](./model-and-dax.md) — definisi DAX lengkap 161 measure
- [pages-detail.md](./pages-detail.md) — komponen per halaman (sumber data index ini)
- [REPORT_OVERVIEW.md](./REPORT_OVERVIEW.md) — ringkasan arsitektur report
