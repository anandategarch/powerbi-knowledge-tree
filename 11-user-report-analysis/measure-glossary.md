# Glosarium Lengkap — Semua 161 Measure

> Kamus fungsi bisnis SELURUH 161 measure report Cost Control, dikelompokkan 16 keluarga.
> Tiap baris: **fungsi bisnis**, **DAX inti** (satu baris — DAX lengkap di [model-and-dax.md](./model-and-dax.md)), **dipakai di** (jumlah visual + halaman utama), dan **dipanggil oleh** (measure lain yang bergantung — indeks dampak penghapusan).
> Sumber: DataModel pbix asli (snapshot 19 Okt 2026) + [pages-detail.md](./pages-detail.md). Generate 7 Nov 2026.

## Cara Pakai

1. Cari measure per nama (Ctrl+F) — semua 161 ada, urut per keluarga.
2. Ingin tahu **bolehkah dihapus**? Lihat kolom *Dipanggil oleh* + *Dipakai* — dua-duanya “—” = aman (tapi cek juga [measure-usage.md](./measure-usage.md)).
3. Ingin tahu **artinya apa di halaman tertentu**? Kolom *Dipakai* menyebut halaman yang memakainya; komposisi visual detailnya di [pages-detail.md](./pages-detail.md).

## Ringkasan Keluarga

| # | Keluarga | Jumlah | Inti fungsi |
|---|---|---|---|
| 1 | [Deviasi inti](#1-deviasi-inti) | 11 | Metrik utama Cost Control: selisih stok teoritis vs aktual. |
| 2 | [BOM dan % BOM](#2-bom-dan--bom) | 14 | BOM = biaya standar pemakaian (jenis “BOM” di Deviasi All). |
| 3 | [Jenis deviasi dan porsi](#3-jenis-deviasi-dan-porsi) | 12 | Model unpivot: Deviasi All[Jenis] membedakan WASTE / SUSUT / TRIAL / LOSS-SURPLUS / USAGE. |
| 4 | [Kontribusi & ranking](#4-kontribusi--ranking) | 15 | Porsi jenis/item thd total (donut “Penyebab”), ranking item, dan mesin nilai Top 10 ([Top10Dev]). |
| 5 | [Multi Value dan format dinamis](#5-multi-value-dan-format-dinamis) | 7 | Measure switching via tabel Trend (DEVIASI/BOM/%BOM/%SALES/VARIANCE) + format tampilan QTY/NOMINAL via slicer Attribute. |
| 6 | [YTD & YoY](#6-ytd--yoy) | 6 | Agregasi tahun-berjalan dengan cutoff mingguan (bukan sisa bulan) dan perbandingan tahun sebelumnya. |
| 7 | [Weekly (cutoff tanggal)](#7-weekly-cutoff-tanggal) | 15 | Deviasi per potongan bulan: Week 1 (±tgl 7), Week 2 (14), Week 3 (25), EoM — dari kolom Deviasi All[Periode]. |
| 8 | [Sales](#8-sales) | 23 | Penjualan bersih (Sales All). |
| 9 | [Waste](#9-waste) | 10 | Keluarga waste (Waste All) — versi QTY & NOMINAL, perbandingan bulan, kontribusi item. |
| 10 | [Barang Masuk (BM)](#10-barang-masuk-bm) | 14 | Rekonsiliasi barang masuk: input resto (Rekap Mentah BM) vs sistem (3224+4205+Pembelian di Luar). |
| 11 | [Update (timestamp)](#11-update-timestamp) | 4 | Teks kapan data terakhir di-refresh — dari tabel Update (Google Sheet), ditampilkan di header halaman. |
| 12 | [PO/PI dan 9901](#12-popi-dan-9901) | 5 | Purchasing: PO (3243) vs Pesanan Invoice (9901) — utang PO & pencarian barang baru. |
| 13 | [Perbandingan A/B](#13-perbandingan-ab) | 16 | Membandingkan dua set barang/resto (slicer Inventory A/B + Resto A/B, tabel A/B fisik terpisah dari model utama) — halaman Perbandingan. |
| 14 | [COM](#14-com) | 4 | Cost of Menu/Cost realisasi = BOM + deviasi. |
| 15 | [Z Score & anomali](#15-z-score--anomali) | 2 | Deteksi anomali per item (Wasutri = Waste + Susut + Trial) berbasis distribusi historis. |
| 16 | [Show / toggle](#16-show--toggle) | 3 | Measure utilitas untuk menyembunyikan baris/visual yang tidak relevan (dipakai sebagai filter visual = “Show”). |
| | **Total** | **161** | |

## 1. Deviasi inti

> Metrik utama Cost Control: selisih stok teoritis vs aktual. [Deviasi Value] adalah mesin; CM/LM pembungkusnya. Mode DINAMIS (slicer Mode) mengganti “bulan berjalan/lalu” jadi periode A/B.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Deviasi CM** | Deviasi All | Deviasi (selisih stok teoritis vs aktual) bulan berjalan. Inti seluruh report. Mode DINAMIS: bila slicer Mode aktif, konteks tanggal diganti Date A (TREATAS) — jadi “CM” = periode A. | `IF( Filtered, CALCULATE( [Deviasi Value], TREATAS( VALUES('Date A'[Date]), 'Date'[Date] ) ), [Deviasi Value] )` | **19** — Top 5 Growth, Summary Deviasi, Deviasi vs BOM (+9 hal.) | % BOM CM, % Sales CM, A Deviasi, ABS Deviasi, Analisa Deviasi, Avg Dev, B Deviasi, COM CM, Deviasi YTD, Deviasi YoY, Dynamic Dev, EoM, Kategori, Kategori L/S, Loss/Surplus, Multi Value CM, Perubahan, Selisih, Selisih YoY, Show, Show Value, Susut, Top10Dev, Trial, Usage, Waste, Week 1, Week 2, Week 3 |
| **Deviasi LM** | Deviasi All | Deviasi bulan lalu (PREVIOUSMONTH). Mode DINAMIS: mengikuti Date B — jadi “LM” = periode B. Pasangan tetap [Deviasi CM]. | `IF( Filtered, CALCULATE( [Deviasi Value], TREATAS( VALUES('Date B'[Date]), 'Date'[Date] ) ), CALCULATE( [Deviasi Value], PREVIO…` | **13** — Top 5 Growth, Detail Deviasi, Deviasi by Area (+5 hal.) | % Sales LM, Analisa Deviasi, Growth, Perubahan, Selisih, Show, Show Value |
| **Deviasi Value** | Deviasi All | Mesin penghitung deviasi: SUM(Deviasi All[Value]) dengan KEEPFILTERS jenis WASTE/SUSUT/TRIAL/LOSS-SURPLUS/USAGE (meniadakan baris BOM dsb.). Semua measure deviasi lain berpangkal ke sini. | `CALCULATE( SUM('Deviasi All'[Value]), KEEPFILTERS( 'Deviasi All'[Jenis] IN {"WASTE", "SUSUT", "TRIAL", "LOSS/SURPLUS", "USAGE"}…` | — | Deviasi CM, Deviasi LM |
| **Selisih** | Deviasi All | Perubahan deviasi bulan-ke-bulan = [Deviasi CM] − [Deviasi LM]. Inti halaman Growth by Resto/Item. | `[Deviasi CM]-[Deviasi LM]` | **13** — Top 5 Growth, Detail Deviasi, Deviasi by Area (+5 hal.) | Dynamic Gap Dev, Growth, Selisih Abs |
| **Selisih Abs** | Deviasi All | Nilai absolut [Selisih] — untuk ranking/magnitude tanpa saling menghapus plus-minus. | `ABS([Selisih])` | **2** — Summary Deviasi | — |
| **Growth** | Deviasi All | % kenaikan/penurunan deviasi vs bulan lalu = [Selisih] ÷ \|[Deviasi LM]\|. | `DIVIDE([Selisih],ABS([Deviasi LM]))` | **21** — Top 5 Growth, Kontribusi Dev, Analisa Susut (+11 hal.) | — |
| **ABS Deviasi** | Deviasi All | \|[Deviasi CM]\| — dipakai Top 5/Top 10 dan agregasi tanpa plus-minus saling menghapus. | `ABS([Deviasi CM])` | **15** — Summary Deviasi, Deviasi by PIC, Kontribusi Dev (+7 hal.) | % Dev Contribution, ABS LM, Rank Dev, Top10Dev |
| **ABS LM** | Deviasi All | \|[Deviasi LM]\| (ABS + PREVIOUSMONTH). | `CALCULATE([ABS Deviasi],PREVIOUSMONTH('Date'[Date]))` | — | — |
| **Kategori** | Deviasi All | Label “Surplus”/“Loss” dari tanda [Deviasi CM] (≥0 = Surplus). | `IF([Deviasi CM] >= 0, "Surplus", "Loss")` | **5** — Deviasi by Area, Deviasi by Resto, Deviasi by Item (+2 hal.) | — |
| **Kategori L/S** | Deviasi All | Label “Surplus”/“Loss” khusus jenis LOSS/SURPLUS (nilai deviasi jenis itu). | `VAR LS = CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="LOSS/SURPLUS") RETURN IF(LS>=0,"Surplus","Loss")` | — | — |
| **Avg Dev** | Deviasi All | Rata-rata deviasi per resto = [Deviasi CM] ÷ jumlah resto aktif (DISTINCTCOUNT RESTO). | `DIVIDE([Deviasi CM],DISTINCTCOUNT('Deviasi All'[RESTO]))` | — | — |

## 2. BOM dan % BOM

> BOM = biaya standar pemakaian (jenis “BOM” di Deviasi All). % BOM = rasio deviasi thd BOM; tolok ukur standar susut.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **BOM CM** | Deviasi All | Nilai BOM (bill of material / biaya standar pemakaian) bulan berjalan — ABS(SUM(Value)) jenis “BOM”. Mode DINAMIS: konteks diganti Date A. | `IF( Filtered, CALCULATE( ABS(SUM('Deviasi All'[Value])), 'Deviasi All'[Jenis] = "BOM", TREATAS( VALUES('Date A'[Date]), 'Date'[…` | **16** — Top 10 Item Dev, Rekap Brg Masuk, Deviasi vs BOM (+5 hal.) | % BOM CM, A BOM, B BOM, BOM ABS, BOM GAP, BOM YTD, BOM YoY, COM CM, Multi Value CM, Rank BOM, Show, Show Value |
| **BOM LM** | Deviasi All | BOM bulan lalu (PREVIOUSMONTH). Mode DINAMIS: mengikuti Date B. | `IF( Filtered, CALCULATE( ABS(SUM('Deviasi All'[Value])), KEEPFILTERS( 'Deviasi All'[Jenis] = "BOM" ), TREATAS( VALUES('Date B'[…` | **3** — Deviasi by PIC, Deviasi by Item, Antar Periode | BOM GAP, Show, Show Value |
| **BOM GAP** | Deviasi All | Perubahan BOM bulan-ke-bulan = [BOM CM] − [BOM LM]. | `[BOM CM]-[BOM LM]` | **3** — Deviasi by PIC, Deviasi by Item, Antar Periode | — |
| **BOM ABS** | Deviasi All | \|[BOM CM]\|. | `ABS([BOM CM])` | **2** — Deviasi by PIC, Growth by Item | — |
| **BOM YTD** | Deviasi All | BOM tahun-berjalan dengan cutoff minggu terpilih (akumulasi EoM Jan→bulan lalu + minggu berjalan). | `VAR SelectedYear = SELECTEDVALUE('Date'[Year]) VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber]) VAR SelectedWeek = SELECT…` | **3** — Deviasi by PIC, Deviasi by Item, Antar Periode | Show, Show Value |
| **BOM YoY** | Deviasi All | BOM periode sama tahun lalu (SAMEPERIODLASTYEAR). | `CALCULATE([BOM CM], SAMEPERIODLASTYEAR('Date'[Date]))` | — | Show, Show Value |
| **% BOM CM** | Deviasi All | Rasio deviasi thd BOM = [Deviasi CM] ÷ [BOM CM] — indikator kesehatan cost. Khusus 3 item susut (Ayam Cincang, Cabe Rawit, Cabai Frozen) pembilangnya dikalikan (1 − rata2 Standar Susut). | `IF( IsKhusus, DIVIDE([Deviasi CM] * (1 - AvgSusut), [BOM CM]), DIVIDE([Deviasi CM], [BOM CM]) )` | **17** — Top 10 Item Dev, Rekap Brg Masuk, Summary Deviasi (+5 hal.) | % BOM CM Abs, % BOM GAP, % BOM LM, % BOM YTD, % BOM YoY, A % BOM, B % BOM, Multi Value CM |
| **% BOM LM** | Deviasi All | % BOM bulan lalu. | `CALCULATE([% BOM CM],PREVIOUSMONTH('Date'[Date]))` | **3** — Deviasi by PIC, Deviasi by Item, Antar Periode | % BOM GAP, Growth % BOM |
| **% BOM GAP** | Deviasi All | Perubahan % BOM = [% BOM CM] − [% BOM LM]. | `[% BOM CM]-[% BOM LM]` | **3** — Deviasi by PIC, Deviasi by Item, Antar Periode | Growth % BOM |
| **% BOM YTD** | Deviasi All | % BOM versi YTD (DATESYTD). | `CALCULATE([% BOM CM],DATESYTD('Date'[Date]))` | **3** — Deviasi by PIC, Deviasi by Item, Antar Periode | — |
| **% BOM YoY** | Deviasi All | % BOM periode sama tahun lalu. | `CALCULATE([% BOM CM], SAMEPERIODLASTYEAR('Date'[Date]))` | — | — |
| **Growth % BOM** | Deviasi All | % pertumbuhan % BOM = [% BOM GAP] ÷ \|[% BOM LM]\|. | `DIVIDE([% BOM GAP],ABS([% BOM LM]))` | — | — |
| **% BOM CM Abs** | Deviasi All | \|[% BOM CM]\| — dipakai pembanding standar susut. | `ABS([% BOM CM])` | **7** — Analisa Susut, Deviasi vs BOM, Deviasi by PIC (+1 hal.) | Analisa Susut |
| **Analisa Susut** | Deviasi All | Klasifikasi item: “Under/Good/Over Standard” — bandingkan \|% BOM\| dengan rata-rata Standar Susut. Inti halaman Analisa Susut. | `SWITCH( TRUE(), ISBLANK([% BOM CM Abs]), BLANK(), [% BOM CM Abs] < AVERAGE('Standar Susut'[Standar Susut]), "Under Standard", […` | **2** — Analisa Susut | — |

## 3. Jenis deviasi dan porsi

> Model unpivot: Deviasi All[Jenis] membedakan WASTE / SUSUT / TRIAL / LOSS-SURPLUS / USAGE. Porsi = kontribusi tiap jenis thd total.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Waste** | Deviasi All | Deviasi jenis WASTE (nilai absolut). | `ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="WASTE"))` | **3** — Kontribusi Dev, Data Waste | % Waste, Cek Selisih Waste, Deviasi Total |
| **Susut** | Deviasi All | Deviasi jenis SUSUT (nilai absolut). | `ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="SUSUT"))` | **4** — Kontribusi Dev, Analisa Susut | % Susut, Deviasi Total |
| **Trial** | Deviasi All | Deviasi jenis TRIAL (nilai absolut). | `ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="TRIAL"))` | **2** — Kontribusi Dev | % Trial, Deviasi Total |
| **Usage** | Deviasi All | Deviasi jenis USAGE (nilai absolut). | `ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="USAGE"))` | — | % Usage |
| **Loss/Surplus** | Deviasi All | Deviasi jenis LOSS/SURPLUS (nilai absolut). | `ABS(CALCULATE([Deviasi CM], 'Deviasi All'[Jenis]="LOSS/SURPLUS"))` | **2** — Kontribusi Dev | % Loss/Suprlus, Deviasi Total |
| **Deviasi Total** | Deviasi All | Total keempat jenis = Waste+Susut+Trial+Loss/Surplus (USAGE di luar). Dasar donut komposisi & porsi. | `[Waste]+[Susut]+[Trial]+[Loss/Surplus]` | — | % Loss/Suprlus, % Susut, % Trial, % Usage, % Waste |
| **% Waste** | Deviasi All | Porsi WASTE thd [Deviasi Total]. | `DIVIDE([Waste],[Deviasi Total])` | **3** — Kontribusi Dev | % Deviasi, Kontribusi CM |
| **% Susut** | Deviasi All | Porsi SUSUT thd [Deviasi Total]. | `DIVIDE([Susut],[Deviasi Total])` | **3** — Kontribusi Dev | % Deviasi, Kontribusi CM |
| **% Trial** | Deviasi All | Porsi TRIAL thd [Deviasi Total]. | `DIVIDE([Trial],[Deviasi Total])` | **3** — Kontribusi Dev | % Deviasi, Kontribusi CM |
| **% Usage** | Deviasi All | Porsi USAGE thd [Deviasi Total]. | `DIVIDE([Usage],[Deviasi Total])` | — | Kontribusi CM |
| **% Loss/Suprlus** | Deviasi All | Porsi LOSS/SURPLUS thd [Deviasi Total]. | `DIVIDE([Loss/Surplus],[Deviasi Total])` | **3** — Kontribusi Dev | % Deviasi, Kontribusi CM |
| **% Deviasi** | Deviasi All | Jumlah keempat porsi (pengecekan konsistensi; normalnya 100%). | `[% Waste]+[% Susut]+[% Trial]+[% Loss/Suprlus]` | — | — |

## 4. Kontribusi & ranking

> Porsi jenis/item thd total (donut “Penyebab”), ranking item, dan mesin nilai Top 10 ([Top10Dev]).

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Kontribusi CM** | Deviasi All | Porsi % suatu jenis deviasi thd total bulan berjalan — IF HASONEVALUE: pilih % jenis terpilih; kalau total: SUMX semua jenis. Dipakai donut & pivot “Penyebab”. | `IF( HASONEVALUE('Deviasi All'[Jenis]), SWITCH( SELECTEDVALUE('Deviasi All'[Jenis]), "WASTE", [% Waste], "SUSUT", [% Susut], "TR…` | **4** — Detail Deviasi | Kontribusi CM EOM, Kontribusi CM Week 1, Kontribusi CM Week 2, Kontribusi CM Week 3, Kontribusi GAP, Kontribusi LM |
| **Kontribusi LM** | Deviasi All | Kontribusi bulan lalu (PREVIOUSMONTH). | `CALCULATE([Kontribusi CM],PREVIOUSMONTH('Date'[Date]))` | **1** — Detail Deviasi | Kontribusi GAP |
| **Kontribusi GAP** | Deviasi All | Perubahan kontribusi = CM − LM. | `[Kontribusi CM]-[Kontribusi LM]` | — | — |
| **Kontribusi CM Week 1** | Deviasi All | Kontribusi pada cutoff Week 1 (Periode=“WEEK 1”). | `CALCULATE([Kontribusi CM],Periode[Periode]="WEEK 1")` | — | — |
| **Kontribusi CM Week 2** | Deviasi All | Kontribusi pada cutoff Week 2. | `CALCULATE([Kontribusi CM],Periode[Periode]="WEEK 2")` | — | — |
| **Kontribusi CM Week 3** | Deviasi All | Kontribusi pada cutoff Week 3. | `CALCULATE([Kontribusi CM],Periode[Periode]="WEEK 3")` | — | — |
| **Kontribusi CM EOM** | Deviasi All | Kontribusi pada akhir bulan (EoM). | `CALCULATE([Kontribusi CM],Periode[Periode]="EOM")` | — | — |
| **% GT Deviasi** | Deviasi All | Porsi item thd grand total ([Nominal Dev] dengan ALL Nama Barang) — kontribusi item keseluruhan. | `VAR Total = CALCULATE([Nominal Dev],ALL(Inventory[Nama Barang])) RETURN DIVIDE([Nominal Dev],Total)` | **8** — Top 10 Item Dev, Rekap Brg Masuk | — |
| **% Dev Contribution** | Deviasi All | Kontribusi \|deviasi\| per kombinasi Resto×Barang (CROSSJOIN) thd ALLSELECTED — analisa penyebab per matriks. | `Pembilang: menjumlahkan ABS Deviasi sesuai konteks baris matriks yang aktif VAR _Numerator = SUMX( CROSSJOIN( VALUES('Resto'[Re…` | **5** — Kontribusi Dev, Deviasi by Resto, Deviasi by Item (+1 hal.) | — |
| **Rank Dev** | Deviasi All | Ranking item dalam resto: RANKX Dense DESC atas [ABS Deviasi], hanya di level item (ISINSCOPE), blank/0 dilewati. | `IF( NOT ISBLANK(CurrentValue) && CurrentValue <> 0, RANKX( ALL('Resto'[Resto]), [ABS Deviasi], , DESC, Dense ) ) )` | **1** — Deviasi by Item | — |
| **Rank BOM** | Deviasi All | Ranking item seperti Rank Dev tetapi atas [BOM CM]. | `IF( NOT ISBLANK(CurrentValue) && CurrentValue <> 0, RANKX( ALL('Resto'[Resto]), [BOM CM], , DESC, Dense ) ) )` | **1** — Deviasi by Item | — |
| **Top10Dev** | Deviasi All | Mesin nilai Top 10: jenis Waste/Susut/Trial → [ABS Deviasi]; jenis lain (Loss/Surplus, Usage) → [Deviasi CM]. | `IF( JenisDev IN {"Waste","Susut","Trial"}, [ABS Deviasi], [Deviasi CM] )` | **2** — Top 10 Item Dev | Nominal Dev, QTY Dev |
| **Nominal Dev** | Deviasi All | [Top10Dev] versi NOMINAL (Rp) — dipakai visual Top 10 Item Dev. | `CALCULATE([Top10Dev],Attribute[Attribute]="NOMINAL")` | **9** — Top 10 Item Dev, Top 5 Waste | % GT Deviasi |
| **QTY Dev** | Deviasi All | [Top10Dev] versi QTY (jumlah satuan). | `FORMAT( CALCULATE( [Top10Dev], Attribute[Attribute] = "QTY" ), "#,##0" ) & " " & MAX('Deviasi All'[Satuan])` | — | — |
| **Analisa Deviasi** | Deviasi All | Narasi otomatis arah perubahan CM vs LM dengan toleransi 1%: “tetap/membesar/mengecil” + arah sign + teks khusus bila salah satu bulan kosong. Baris Hide disembunyikan. | `IF( _Show = "Hide", BLANK(), SWITCH( TRUE(), Tidak ada data bulan ini ISBLANK(_Curr), "Tidak ada deviasi", Bulan lalu tidak ada…` | **6** — Deviasi by Area, Deviasi by PIC, Deviasi by Resto (+3 hal.) | — |

## 5. Multi Value dan format dinamis

> Measure switching via tabel Trend (DEVIASI/BOM/%BOM/%SALES/VARIANCE) + format tampilan QTY/NOMINAL via slicer Attribute.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Multi Value CM** | Deviasi All | Measure switching untuk grafik multi-metrik: SELECTEDVALUE(Trend[Kategori]) → DEVIASI/BOM/%BOM; %SALES & VARIANCE hanya untuk attribute NOMINAL (butuh data sales). | `SWITCH( SELECTEDVALUE('Trend'[Kategori]), "DEVIASI", [Deviasi CM], "BOM", [BOM CM], "% BOM", [% BOM CM], "% SALES", IF( SELECTE…` | **5** — Deviasi by Area, Deviasi by Resto, Deviasi by Item (+2 hal.) | Multi Value Gap, Multi Value LM, Perubahan Multi Value, Z Score |
| **Multi Value LM** | Deviasi All | Multi Value bulan lalu. | `CALCULATE([Multi Value CM],PREVIOUSMONTH('Date'[Date]))` | **3** — Deviasi by Area, Growth by Resto, COM | Multi Value Gap, Multi Value Growth, Perubahan Multi Value |
| **Multi Value Gap** | Deviasi All | Multi Value CM − LM. | `[Multi Value CM]-[Multi Value LM]` | **3** — Deviasi by Area, Growth by Resto, COM | Multi Value Growth |
| **Multi Value Growth** | Deviasi All | % pertumbuhan Multi Value = Gap ÷ \|LM\|. | `DIVIDE([Multi Value Gap],ABS([Multi Value LM]))` | **3** — Deviasi by Area, Growth by Resto, COM | — |
| **Perubahan Multi Value** | Deviasi All | % perubahan \|Multi Value\| CM vs LM. | `DIVIDE(ABS([Multi Value CM])-ABS([Multi Value LM]),ABS([Multi Value LM]))` | **2** — Deviasi by Area, COM | — |
| **Dynamic Dev** | Deviasi All | Format tampilan deviasi dinamis: QTY → “#,##0 + Satuan”; NOMINAL → “±Rp #,##0”. Menyesuaikan slicer Attribute. | `SWITCH( SELECTEDVALUE(Attribute[Attribute]), "QTY", FORMAT([Deviasi CM], "#,##0") & " " & MAX('Deviasi All'[Satuan]), "NOMINAL"…` | — | — |
| **Dynamic Gap Dev** | Deviasi All | Format dinamis (QTY/NOMINAL) untuk [Selisih] — pasangan Dynamic Dev pada visual perubahan. | `SWITCH( SELECTEDVALUE(Attribute[Attribute]), "QTY", FORMAT([Selisih], "#,##0") & " " & MAX('Deviasi All'[Satuan]), "NOMINAL", I…` | — | — |

## 6. YTD & YoY

> Agregasi tahun-berjalan dengan cutoff mingguan (bukan sisa bulan) dan perbandingan tahun sebelumnya.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Deviasi YTD** | Deviasi All | Deviasi tahun-berjalan sampai cutoff minggu terpilih: akumulasi EoM Januari→bulan lalu + minggu berjalan (bukan sisa bulan). | `VAR SelectedYear = SELECTEDVALUE('Date'[Year]) VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber]) VAR SelectedWeek = SELECT…` | **7** — Detail Deviasi, Deviasi by Area, Deviasi by PIC (+4 hal.) | % Sales YTD, Show, Show Value |
| **Selected Cutoff Date** | Deviasi All | Tanggal cutoff YTD: Week 1→tgl 7, Week 2→14, Week 3→25, EoM→akhir bulan. Dipakai keluarga YTD. | `VAR SelectedYear = SELECTEDVALUE('Date'[Year]) VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber]) VAR SelectedWeek = SELECT…` | — | — |
| **Deviasi YoY** | Deviasi All | Deviasi periode sama tahun lalu (SAMEPERIODLASTYEAR). | `CALCULATE([Deviasi CM], SAMEPERIODLASTYEAR('Date'[Date]))` | **6** — Deviasi by Area, Deviasi by PIC, Deviasi by Resto (+3 hal.) | % Sales YoY, Growth YoY, Selisih YoY, Show, Show Value |
| **Selisih YoY** | Deviasi All | [Deviasi CM] − [Deviasi YoY]. | `[Deviasi CM]-[Deviasi YoY]` | — | Growth YoY |
| **Growth YoY** | Deviasi All | % pertumbuhan YoY = [Selisih YoY] ÷ \|[Deviasi YoY]\|. | `DIVIDE([Selisih YoY],ABS([Deviasi YoY]))` | **7** — Summary Deviasi, Deviasi by Area, Deviasi by PIC (+4 hal.) | — |
| **Resto YoY** | Deviasi All | Jumlah resto aktif (DISTINCTCOUNT RESTO) periode sama tahun lalu. | `CALCULATE(DISTINCTCOUNT('Deviasi All'[RESTO]),SAMEPERIODLASTYEAR('Date'[Date]))` | **2** — Deviasi by Area, COM | — |

## 7. Weekly (cutoff tanggal)

> Deviasi per potongan bulan: Week 1 (±tgl 7), Week 2 (14), Week 3 (25), EoM — dari kolom Deviasi All[Periode].

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Week 1** | Deviasi All | Deviasi CM pada periode Week 1 (cutoff ±tgl 7). \|nilai\| < 0.1 dianggap BLANK. | `VAR Nilai = CALCULATE( [Deviasi CM], 'Deviasi All'[Periode] = "Week 1" ) RETURN IF( ABS(Nilai) < 0.1, BLANK(), Nilai )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | Growth W2, Perubahan W2, Week 1 Abs |
| **Week 2** | Deviasi All | Deviasi CM pada periode Week 2 (cutoff tgl 14). | `VAR Nilai = CALCULATE( [Deviasi CM], 'Deviasi All'[Periode] = "Week 2" ) RETURN IF( ABS(Nilai) < 0.1, BLANK(), Nilai )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | Growth EOM, Growth W2, Growth W3, Perubahan EOM, Perubahan W2, Perubahan W3, Week 2 Abs |
| **Week 3** | Deviasi All | Deviasi CM pada periode Week 3 (cutoff tgl 25). | `VAR Nilai = CALCULATE( [Deviasi CM], 'Deviasi All'[Periode] = "Week 3" ) RETURN IF( ABS(Nilai) < 0.1, BLANK(), Nilai )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | Growth W3, Perubahan W3, Week 3 Abs |
| **EoM** | Deviasi All | Deviasi CM pada akhir bulan (EoM). | `VAR Nilai = CALCULATE( [Deviasi CM], 'Deviasi All'[Periode] = "EoM" ) RETURN IF( ABS(Nilai) < 0.1, BLANK(), Nilai )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | EoM Abs, Growth EOM, Perubahan EOM |
| **Week 1 Abs** | Deviasi All | \|[Week 1]\|. | `ABS([Week 1])` | **2** — Growth by Resto, Growth by Item | — |
| **Week 2 Abs** | Deviasi All | \|[Week 2]\|. | `ABS([Week 2])` | **2** — Growth by Resto, Growth by Item | — |
| **Week 3 Abs** | Deviasi All | \|[Week 3]\|. | `ABS([Week 3])` | **2** — Growth by Resto, Growth by Item | — |
| **EoM Abs** | Deviasi All | \|[EoM]\|. | `ABS([EoM])` | **2** — Growth by Resto, Growth by Item | — |
| **Growth W2** | Deviasi All | % pertumbuhan Week 2 vs Week 1 (blank bila salah satu kosong). | `IF( OR( ISBLANK([Week 2]), ISBLANK([Week 1]) ), BLANK(), DIVIDE( [Week 2] - [Week 1], ABS([Week 1]) ) )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | — |
| **Growth W3** | Deviasi All | % pertumbuhan Week 3 vs Week 2. | `IF( OR( ISBLANK([Week 3]), ISBLANK([Week 2]) ), BLANK(), DIVIDE( [Week 3] - [Week 2], ABS([Week 2]) ) )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | — |
| **Growth EOM** | Deviasi All | % pertumbuhan EoM vs Week 2. | `IF( OR( ISBLANK([EoM]), ISBLANK([Week 2]) ), BLANK(), DIVIDE( [EoM] - [Week 2], ABS([Week 2]) ) )` | **3** — Detail Deviasi, Growth by Resto, Growth by Item | — |
| **Perubahan** | Deviasi All | % perubahan \|deviasi\| CM vs LM = (\|CM\|−\|LM\|) ÷ \|LM\|. | `DIVIDE(ABS([Deviasi CM])-ABS([Deviasi LM]),ABS([Deviasi LM]))` | **12** — Top 5 Growth, Deviasi by Area, Deviasi by PIC (+4 hal.) | — |
| **Perubahan W2** | Deviasi All | % perubahan \|Week 2\| vs \|Week 1\|. | `IF( ISBLANK([Week 2]), BLANK(), DIVIDE( ABS([Week 2]) - ABS([Week 1]), ABS([Week 1]) ) )` | **2** — Growth by Resto, Growth by Item | — |
| **Perubahan W3** | Deviasi All | % perubahan \|Week 3\| vs \|Week 2\|. | `IF( ISBLANK([Week 3]), BLANK(), DIVIDE( ABS([Week 3]) - ABS([Week 2]), ABS([Week 2]) ) )` | **2** — Growth by Resto, Growth by Item | — |
| **Perubahan EOM** | Deviasi All | % perubahan \|EoM\| vs \|Week 2\|. | `IF( ISBLANK([EoM]), BLANK(), DIVIDE( ABS([EoM]) - ABS([Week 2]), ABS([Week 2]) ) )` | **2** — Growth by Resto, Growth by Item | — |

## 8. Sales

> Penjualan bersih (Sales All). Hanya attribute NOMINAL; cutoff mingguan untuk perbandingan antar bulan.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Sales CM** | Sales All | Penjualan bersih bulan berjalan — hanya attribute NOMINAL (QTY → BLANK). Mesin semua measure sales. | `IF( SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL", SUM('Sales All'[Penjualan Bersih]), BLANK() )` | **4** — Top 5 Growth, Summary Deviasi, Data Sales | % Sales CM, A Sales, B Sales, Sales GAP, Sales LM, Sales YTD, Sales YoY, Sales sd 07 Bulan Ini, Sales sd 14 Bulan Ini, Sales sd 25 Bulan Ini, Sales sd EOM Bulan Ini |
| **Sales LM** | Sales All | Sales bulan lalu (PREVIOUSMONTH). | `CALCULATE([Sales CM],PREVIOUSMONTH('Date'[Date]))` | **5** — Top 5 Growth, Deviasi by Area, Deviasi by Resto (+1 hal.) | % Sales Growth, % Sales LM, Sales GAP, Sales sd 07 Bulan Lalu, Sales sd 14 Bulan Lalu, Sales sd 25 Bulan Lalu, Sales sd EOM Bulan Lalu |
| **Sales GAP** | Sales All | Perubahan sales = CM − LM. | `[Sales CM]-[Sales LM]` | **5** — Top 5 Growth, Deviasi by Area, Deviasi by Resto (+1 hal.) | % Sales Growth |
| **% Sales Growth** | Sales All | % pertumbuhan sales = GAP ÷ \|LM\|. | `DIVIDE([Sales GAP],ABS([Sales LM]))` | **5** — Top 5 Growth, Deviasi by Area, Deviasi by Resto (+1 hal.) | — |
| **Sales YoY** | Sales All | Sales periode sama tahun lalu. | `CALCULATE([Sales CM], SAMEPERIODLASTYEAR('Date'[Date]))` | — | % Sales YoY |
| **% Sales YoY** | Sales All | Rasio deviasi YoY thd sales YoY. | `DIVIDE([Deviasi YoY],[Sales YoY])` | — | — |
| **Sales YTD** | Sales All | Sales tahun-berjalan dengan cutoff minggu terpilih (pola sama Deviasi YTD). | `VAR SelectedYear = SELECTEDVALUE('Date'[Year]) VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber]) VAR SelectedWeek = SELECT…` | **3** — Deviasi by Area, Deviasi by Resto, COM | % Sales YTD |
| **% Sales YTD** | Sales All | Rasio [Deviasi YTD] ÷ [Sales YTD]. | `DIVIDE([Deviasi YTD],[Sales YTD])` | **3** — Deviasi by Area, Deviasi by Resto, COM | — |
| **% Sales CM** | Sales All | Rasio deviasi thd sales bulan berjalan — indikator “% SALES”. | `DIVIDE([Deviasi CM],[Sales CM])` | **5** — Summary Deviasi, Deviasi by Area, Deviasi by Resto (+1 hal.) | % Sales GAP, A % Sales, B % Sales, Multi Value CM |
| **% Sales LM** | Sales All | Rasio deviasi bulan lalu thd sales bulan lalu. | `DIVIDE([Deviasi LM],[Sales LM])` | **3** — Deviasi by Area, Deviasi by Resto, COM | % Sales GAP |
| **% Sales GAP** | Sales All | Perubahan rasio % sales = CM − LM (dipakai Multi Value VARIANCE). | `[% Sales CM]-[% Sales LM]` | **3** — Deviasi by Area, Deviasi by Resto, COM | Multi Value CM |
| **Sales sd 07 Bulan Ini** | Sales All | Sales sampai cutoff Week 1 bulan berjalan (Periode=“WEEK 1”). | `CALCULATE([Sales CM],Periode[Periode]="WEEK 1")` | **1** — Sales Growth | % Sales Growth 07 |
| **Sales sd 07 Bulan Lalu** | Sales All | Sales sampai cutoff Week 1 bulan lalu. | `CALCULATE([Sales LM],Periode[Periode]="WEEK 1")` | **1** — Sales Growth | % Sales Growth 07 |
| **Sales sd 14 Bulan Ini** | Sales All | Sales sampai cutoff Week 2 bulan berjalan. | `CALCULATE([Sales CM],Periode[Periode]="WEEK 2")` | **1** — Sales Growth | % Sales Growth 14 |
| **Sales sd 14 Bulan Lalu** | Sales All | Sales sampai cutoff Week 2 bulan lalu. | `CALCULATE([Sales LM],Periode[Periode]="WEEK 2")` | **1** — Sales Growth | % Sales Growth 14 |
| **Sales sd 25 Bulan Ini** | Sales All | Sales sampai cutoff Week 3 bulan berjalan. | `CALCULATE([Sales CM],Periode[Periode]="WEEK 3")` | **1** — Sales Growth | % Sales Growth 25 |
| **Sales sd 25 Bulan Lalu** | Sales All | Sales sampai cutoff Week 3 bulan lalu. | `CALCULATE([Sales LM],Periode[Periode]="WEEK 3")` | **1** — Sales Growth | % Sales Growth 25 |
| **Sales sd EOM Bulan Ini** | Sales All | Sales sampai akhir bulan berjalan. | `CALCULATE([Sales CM],Periode[Periode]="EOM")` | **1** — Sales Growth | % Sales Growth EOM |
| **Sales sd EOM Bulan Lalu** | Sales All | Sales sampai akhir bulan lalu. | `CALCULATE([Sales LM],Periode[Periode]="EOM")` | **1** — Sales Growth | % Sales Growth EOM |
| **% Sales Growth 07** | Sales All | % pertumbuhan sales pada cutoff Week 1 (bulan ini vs lalu). | `IF( ISBLANK([Sales sd 07 Bulan Ini]), BLANK(), DIVIDE( [Sales sd 07 Bulan Ini] - [Sales sd 07 Bulan Lalu], [Sales sd 07 Bulan L…` | **1** — Sales Growth | — |
| **% Sales Growth 14** | Sales All | % pertumbuhan sales pada cutoff Week 2. | `IF( ISBLANK([Sales sd 14 Bulan Ini]), BLANK(), DIVIDE( [Sales sd 14 Bulan Ini] - [Sales sd 14 Bulan Lalu], [Sales sd 14 Bulan L…` | **1** — Sales Growth | — |
| **% Sales Growth 25** | Sales All | % pertumbuhan sales pada cutoff Week 3. | `IF( ISBLANK([Sales sd 25 Bulan Ini]), BLANK(), DIVIDE( [Sales sd 25 Bulan Ini] - [Sales sd 25 Bulan Lalu], [Sales sd 25 Bulan L…` | **1** — Sales Growth | — |
| **% Sales Growth EOM** | Sales All | % pertumbuhan sales pada akhir bulan. | `IF( ISBLANK([Sales sd EOM Bulan Ini]), BLANK(), DIVIDE( [Sales sd EOM Bulan Ini] - [Sales sd EOM Bulan Lalu], [Sales sd EOM Bul…` | **1** — Sales Growth | — |

## 9. Waste

> Keluarga waste (Waste All) — versi QTY & NOMINAL, perbandingan bulan, kontribusi item.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Qty Waste** | Waste All | Waste versi QTY (jumlah satuan) — teks terformat “#,##0 + SATUAN”. | `FORMAT( CALCULATE( SUM('Waste All'[Value]), Attribute[Attribute] = "QTY" ), "#,##0" ) & " " & MAX('Waste All'[SATUAN])` | — | — |
| **Nominal Waste** | Waste All | Waste versi NOMINAL (Rp) — mesin measure waste lain. | `CALCULATE(SUM('Waste All'[Value]),Attribute[Attribute]="NOMINAL")` | **7** — Top 5 Waste | % to Subtotal, % to Total, Kontribusi Waste |
| **Waste LM** | Waste All | Waste bulan lalu (PREVIOUSMONTH). | `CALCULATE(SUM('Waste All'[Value]),PREVIOUSMONTH('Date'[Date]))` | **1** — Waste by Resto | Waste Gap, Waste Growth |
| **Waste Gap** | Waste All | Perubahan waste = CM − LM. | `SUM('Waste All'[Value])-[Waste LM]` | **1** — Waste by Resto | Waste Growth |
| **Waste Growth** | Waste All | % pertumbuhan waste. | `DIVIDE([Waste Gap],ABS([Waste LM]))` | **1** — Waste by Resto | — |
| **Waste YTD** | Waste All | Waste tahun-berjalan dengan cutoff minggu terpilih (pola Deviasi YTD). | `VAR SelectedYear = SELECTEDVALUE('Date'[Year]) VAR SelectedMonth = SELECTEDVALUE('Date'[MonthNumber]) VAR SelectedWeek = SELECT…` | **1** — Waste by Resto | — |
| **Kontribusi Waste** | Waste All | Porsi item thd total waste pada konteks filter berjalan (REMOVEFILTERS). | `DIVIDE( [Nominal Waste], CALCULATE( [Nominal Waste], REMOVEFILTERS('Waste All'[Jenis]) ) )` | — | — |
| **% to Subtotal** | Waste All | Porsi item thd subtotal kelasnya (ALL Nama Barang, filter jenis tetap). | `VAR Total = CALCULATE([Nominal Waste],ALL('Inventory'[Nama Barang])) RETURN DIVIDE([Nominal Waste],Total)` | **6** — Top 5 Waste | — |
| **% to Total** | Waste All | Porsi item thd grand total waste (ALL Nama Barang + ALL Jenis). | `VAR Total = CALCULATE([Nominal Waste],ALL('Inventory'[Nama Barang]),ALL('Waste All'[Jenis])) RETURN DIVIDE([Nominal Waste],Total)` | **7** — Top 5 Waste, Waste by Item | — |
| **Cek Selisih Waste** | Waste All | Kontrol konsistensi: [Waste] (via jenis) vs SUM langsung Waste All[Value] — idealnya 0. | `[Waste]-SUM('Waste All'[Value])` | **1** — Data Waste | — |

## 10. Barang Masuk (BM)

> Rekonsiliasi barang masuk: input resto (Rekap Mentah BM) vs sistem (3224+4205+Pembelian di Luar). Inti halaman 22–24.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **BM Sistem** | Rekap Mentah BM | Barang masuk versi sistem = 3224 (penerimaan gudang) + 4205 (transfer) + [Pembelian di Luar]. | `SUM('3224'[Kts (Unit#1)])+SUM('4205'[#Qty. Terkecil])+[Pembelian di Luar]` | — | Count BM Sistem |
| **BM Resto** | Rekap Mentah BM | Barang masuk versi input resto = SUM(Rekap Mentah BM[Value]). | `SUM('Rekap Mentah BM'[Value])` | — | Count BM Resto |
| **Selisih BM** | Rekap Mentah BM | Inti rekonsiliasi BM = [BM Resto] − [BM Sistem]. 0 = input resto cocok sistem. | `VAR Sistem = SUM('3224'[Kts (Unit#1)])+SUM('4205'[#Qty. Terkecil])+[Pembelian di Luar] VAR Inputan = SUM('Rekap Mentah BM'[Valu…` | **7** — Rekap Brg Masuk, Detail Brg Masuk, Rekon Brg Masuk (+1 hal.) | ABS Selisih BM, Input Benar, Input Salah, Show BM |
| **Clean BM** | Rekap Mentah BM | Gabungan semua sumber (3224+4205+Pembelian di Luar+Rekap Mentah) — dipakai header rekap BM. | `SUM('3224'[Kts (Unit#1)])+SUM('4205'[#Qty. Terkecil])+[Pembelian di Luar]+SUM('Rekap Mentah BM'[Value])` | **9** — ID Dokumen, Tooltip, Detail Brg Masuk (+1 hal.) | — |
| **Count BM Resto** | Rekap Mentah BM | Banyaknya tanggal dengan input resto ≠ 0 (frekuensi input). | `COUNTROWS( FILTER( VALUES('Date'[Date]), [BM Resto] <> 0 ) )` | **1** — Rekon Brg Masuk | % BM, Input Benar, Input Salah |
| **Count BM Sistem** | Rekap Mentah BM | Banyaknya tanggal dengan nilai sistem ≠ 0. | `COUNTROWS( FILTER( VALUES('Date'[Date]), [BM Sistem] <> 0 ) )` | **1** — Rekon Brg Masuk | % BM, Input Benar, Input Salah |
| **% BM** | Rekap Mentah BM | Kecocokan frekuensi = MIN(count) ÷ MAX(count). 1 = frekuensi resto & sistem sama. | `DIVIDE( MIN([Count BM Resto], [Count BM Sistem]), MAX([Count BM Resto], [Count BM Sistem]) )` | **1** — Rekon Brg Masuk | Show BM |
| **ABS Selisih BM** | Rekap Mentah BM | \|[Selisih BM]\| — untuk ranking/kolom magnitudo. | `ABS([Selisih BM])` | — | — |
| **Show BM** | Rekap Mentah BM | Hide bila [% BM]=1 dan [Selisih BM]=0 (barang sudah recon sempurna → baris disembunyikan). | `IF(AND([% BM]=1,[Selisih BM]=0),"Hide","Show")` | **1** — Rekon Brg Masuk | — |
| **Input Benar** | Rekap Mentah BM | Jumlah barang yang inputnya cocok (kedua count > 0 dan selisih = 0). | `CALCULATE( DISTINCTCOUNT(Inventory[Nama Barang]), FILTER( VALUES(Inventory[Nama Barang]), [Count BM Resto] > 0 && [Count BM Sis…` | **1** — Rekon Brg Masuk | % BM Benar, Total BM |
| **Input Salah** | Rekap Mentah BM | Jumlah barang yang inputnya TIDAK cocok (kedua count > 0 tapi selisih ≠ 0). | `CALCULATE( DISTINCTCOUNT(Inventory[Nama Barang]), FILTER( VALUES(Inventory[Nama Barang]), [Count BM Resto] > 0 && [Count BM Sis…` | **1** — Rekon Brg Masuk | % BM Salah, Total BM |
| **Total BM** | Rekap Mentah BM | [Input Benar] + [Input Salah] — total barang dinilai. | `[Input Benar]+[Input Salah]` | **1** — Rekon Brg Masuk | % BM Benar, % BM Salah |
| **% BM Benar** | Rekap Mentah BM | Porsi input benar = [Input Benar] ÷ [Total BM]. | `DIVIDE([Input Benar],[Total BM])` | **1** — Rekon Brg Masuk | — |
| **% BM Salah** | Rekap Mentah BM | Porsi input salah = [Input Salah] ÷ [Total BM]. | `DIVIDE([Input Salah],[Total BM])` | **1** — Rekon Brg Masuk | — |

## 11. Update (timestamp)

> Teks kapan data terakhir di-refresh — dari tabel Update (Google Sheet), ditampilkan di header halaman.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Update on Dev** | Update | Teks “Updated on : dd mmmm yyyy” — tanggal update data deviasi terakhir (tabel Update, keterangan “Update on Deviasi”). | `"Updated on : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Update on Deviasi"),"dd mmmm yyyy")` | **1** — Home | — |
| **Last Update Dev** | Update | Teks “Last Update : mmmm yyyy - Week n” — periode data deviasi terbaru (ditentukan dari tanggal + periode terakhir di data). | `"Last Update : " & FORMAT(Last, "mmmm yyyy") & " - " & LastPeriod` | **1** — Home | — |
| **Update on BM** | Update | Teks “Updated on : dd mmmm yyyy” — tanggal update data barang masuk. | `"Updated on : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Update on Barang Masuk"),"dd mmmm yyyy")` | **1** — Home | — |
| **Last Update BM** | Update | Teks “Last Update : dd mmmm yyyy” — update terakhir barang masuk. | `"Last Update : "&FORMAT(CALCULATE(MAX('Update'[Tanggal]),'Update'[Keterangan]="Last Update Barang Masuk"),"dd mmmm yyyy")` | **1** — Home | — |

## 12. PO/PI dan 9901

> Purchasing: PO (3243) vs Pesanan Invoice (9901) — utang PO & pencarian barang baru. [Pembelian di Luar] bagian BM Sistem.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Nilai PI** | 3243 | Nilai pesanan invoice (PI) yang sudah diterima per PO×barang — join ke 9901 (#Purch.Total) via Nomor # + Nama Barang (SUMX SUMMARIZE). | `CALCULATE( SUM('9901'[#Purch.Total]), REMOVEFILTERS('Date'), '9901'[Nomor # Pesanan Pembelian] = _PO, '9901'[Nama Barang] = _Ba…` | **4** — Rekap PO blm PI, Detail PO blm PI | Belum Dibayar, Status PO |
| **Belum Dibayar** | 3243 | Utang PO = SUM(3243[Total Harga]) − [Nilai PI]. Inti halaman Rekap PO blm PI. | `SUM('3243'[Total Harga])-[Nilai PI]` | **4** — Rekap PO blm PI, Detail PO blm PI | — |
| **Status PO** | 3243 | Status pembayaran PO per Nomor #: “Sama” bila Total Harga = Nilai PI, selain itu “Beda”. | `IF( _TotalHargaPO = _NilaiPIPO, "Sama", "Beda" )` | **1** — Detail PO blm PI | — |
| **Nama Barang 9901** | 3243 | Daftar nama barang 9901 yang cocok dengan PO 3243 (CONCATENATEX per Nomor # + Kode #) — jembatan pencarian barang baru. | `CONCATENATEX( DISTINCT( SELECTCOLUMNS( FILTER( '9901', '9901'[Nomor # Pesanan Pembelian] = _PO && '9901'[Kode #] = _Kode ), "Na…` | **1** — Detail PO blm PI | — |
| **Pembelian di Luar** | 9901 | Pembelian tanpa nomor penerimaan barang (ISBLANK Nomor # Penerimaan) jenis Persediaan — dari 9901. Komponen [BM Sistem]. | `CALCULATE( SUM('9901'[#Prime.Qty]), ISBLANK('9901'[Nomor # Penerimaan Barang]), '9901'[Jenis Barang Barang & Jasa] = "Persediaa…` | **4** — Rekap Brg Masuk, Detail Brg Masuk, Rekon Brg Masuk (+1 hal.) | BM Sistem, Clean BM, Selisih BM |

## 13. Perbandingan A/B

> Membandingkan dua set barang/resto (slicer Inventory A/B + Resto A/B, tabel A/B fisik terpisah dari model utama) — halaman Perbandingan.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **A Deviasi** | Inventory A | [Deviasi CM] untuk SET A — barang terpilih di slicer Inventory A + resto di Resto A (TREATAS). Dipakai halaman Perbandingan. | `CALCULATE( [Deviasi CM], TREATAS( VALUES('Inventory A'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang] ), TREATAS( VALUES('Rest…` | **1** — Perbandingan | GAP ABS Dev, GAP Deviasi |
| **B Deviasi** | Inventory A | [Deviasi CM] untuk SET B (Inventory B + Resto B). | `CALCULATE( [Deviasi CM], TREATAS( VALUES('Inventory B'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang] ), TREATAS( VALUES('Rest…` | **1** — Perbandingan | GAP ABS Dev, GAP Deviasi |
| **GAP Deviasi** | Inventory A | Perbedaan antar set = [A Deviasi] − [B Deviasi]. | `[A Deviasi]-[B Deviasi]` | **1** — Perbandingan | — |
| **GAP ABS Dev** | Inventory A | Perubahan magnitudo = \|A\| − \|B\| (positif = membesar). | `ABS([A Deviasi])-ABS([B Deviasi])` | **1** — Perbandingan | — |
| **A BOM** | Inventory A | [BOM CM] untuk SET A. | `CALCULATE( [BOM CM], TREATAS( VALUES('Inventory A'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang] ), TREATAS( VALUES('Resto A'…` | **1** — Perbandingan | GAP BOM |
| **B BOM** | Inventory A | [BOM CM] untuk SET B. | `CALCULATE( [BOM CM], TREATAS( VALUES('Inventory B'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang] ), TREATAS( VALUES('Resto B'…` | **1** — Perbandingan | GAP BOM |
| **GAP BOM** | Inventory A | [A BOM] − [B BOM]. | `[A BOM]-[B BOM]` | **1** — Perbandingan | — |
| **A % BOM** | Inventory A | [% BOM CM] untuk SET A. | `CALCULATE( [% BOM CM], TREATAS( VALUES('Inventory A'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang] ), TREATAS( VALUES('Resto…` | **1** — Perbandingan | GAP % BOM |
| **B % BOM** | Inventory A | [% BOM CM] untuk SET B. | `CALCULATE( [% BOM CM], TREATAS( VALUES('Inventory B'[(Old) Nama Barang]), 'Deviasi All'[Nama Barang] ), TREATAS( VALUES('Resto…` | **1** — Perbandingan | GAP % BOM |
| **GAP % BOM** | Inventory A | [A % BOM] − [B % BOM]. | `[A % BOM]-[B % BOM]` | **1** — Perbandingan | — |
| **A Sales** | Inventory A | [Sales CM] untuk SET A (Resto A). | `CALCULATE( [Sales CM], TREATAS( VALUES('Resto A'[Resto]), 'Sales All'[Nama Cabang] ) )` | **1** — Perbandingan | GAP Sales |
| **B Sales** | Inventory A | [Sales CM] untuk SET B (Resto B). | `CALCULATE( [Sales CM], TREATAS( VALUES('Resto B'[Resto]), 'Sales All'[Nama Cabang] ) )` | **1** — Perbandingan | GAP Sales |
| **GAP Sales** | Inventory A | [A Sales] − [B Sales]. | `[A Sales]-[B Sales]` | **1** — Perbandingan | — |
| **A % Sales** | Inventory A | [% Sales CM] untuk SET A. | `CALCULATE( [% Sales CM], TREATAS( VALUES('Resto A'[Resto]), 'Deviasi All'[Resto] ) )` | **1** — Perbandingan | GAP % Sales |
| **B % Sales** | Inventory A | [% Sales CM] untuk SET B. | `CALCULATE( [% Sales CM], TREATAS( VALUES('Resto B'[Resto]), 'Deviasi All'[Resto] ) )` | **1** — Perbandingan | GAP % Sales |
| **GAP % Sales** | Inventory A | [A % Sales] − [B % Sales]. | `[A % Sales]-[B % Sales]` | **1** — Perbandingan | — |

## 14. COM

> Cost of Menu/Cost realisasi = BOM + deviasi. Halaman COM (20).

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **COM CM** | Deviasi All | Realisasi cost = [BOM CM] + [Deviasi CM] (standar + penyimpangan). Inti halaman COM. | `[BOM CM]+[Deviasi CM]` | **3** — COM | COM Gap, COM LM |
| **COM LM** | Deviasi All | COM bulan lalu (PREVIOUSMONTH). | `CALCULATE([COM CM], PREVIOUSMONTH('Date'[Date]))` | **1** — COM | COM Gap, COM Growth |
| **COM Gap** | Deviasi All | Perubahan COM = CM − LM. | `[COM CM]-[COM LM]` | **1** — COM | COM Growth |
| **COM Growth** | Deviasi All | % pertumbuhan COM = Gap ÷ LM. | `DIVIDE([COM Gap],[COM LM])` | **1** — COM | — |

## 15. Z Score & anomali

> Deteksi anomali per item (Wasutri = Waste + Susut + Trial) berbasis distribusi historis.

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Z Score** | Deviasi All | Skor standar deviasi item vs rata-rata historis (ALLSELECTED per tanggal): (nilai − mean) ÷ stdev. Dasar deteksi anomali Wasutri. | `DIVIDE( CurrentValue - MeanValue, StdDevValue )` | — | Kategori Anomali Wasutri |
| **Kategori Anomali Wasutri** | Deviasi All | Label “Anomali” bila \|Z Score\| > 1, selain itu “Normal”. | `SWITCH( TRUE(), ABS([Z Score]) > 1, "Anomali", "Normal" )` | — | — |

## 16. Show / toggle

> Measure utilitas untuk menyembunyikan baris/visual yang tidak relevan (dipakai sebagai filter visual = “Show”).

| Measure | Tabel | Fungsi bisnis | DAX inti | Dipakai (visual — halaman) | Dipanggil oleh |
|---|---|---|---|---|---|
| **Show** | Deviasi All | Toggle tampil/hide baris: “Hide” bila jumlah seluruh metrik utama (CM/LM/YTD/YoY deviasi+BOM) < 0 — dipakai sebagai filter visual. (Kondisi ABS<0 hampir tak pernah terpenuhi, praktisnya selalu “Show”.) | `VAR Nilai = [Deviasi CM]+[BOM CM]+ [Deviasi LM]+[BOM LM]+ [Deviasi YTD]+[BOM YTD]+ [Deviasi YoY]+[BOM YoY] RETURN IF(ABS(Nilai)…` | **1** — Antar Periode | Analisa Deviasi |
| **Show Value** | Deviasi All | Jumlah metrik utama (sama seperti Show) sebagai angka — untuk conditional formatting / deteksi baris kosong. | `VAR Nilai = [Deviasi CM]+[BOM CM]+ [Deviasi LM]+[BOM LM]+ [Deviasi YTD]+[BOM YTD]+ [Deviasi YoY]+[BOM YoY] RETURN Nilai` | **5** — Deviasi by Area, Deviasi by PIC, Deviasi by Resto (+2 hal.) | — |
| **Show Detail** | Deviasi All | Toggle baris detail per Tipe×Jenis (mis. COM DEVIASI hanya WASTE/TRIAL/SUSUT/LS; CONSUME hanya USAGE/LS) — menyembunyikan kombinasi yang tidak relevan di Detail Deviasi. | `SWITCH( TRUE(), _Tipe = "COM DEVIASI" && _Jenis IN {"WASTE", "TRIAL", "SUSUT", "LOSS/SURPLUS"}, "SHOW", _Tipe = "COM DEVIASI" &…` | **3** — Detail Deviasi | — |

## Catatan Lintas-Keluarga

- **Rantai inti**: `[Deviasi Value]` → `[Deviasi CM]/[Deviasi LM]` → `[Selisih]`/`[Growth]` → turunannya (YTD, YoY, weekly). Mengubah satu = mengganti angka seluruh report.
- **Mode DINAMIS** (slicer `Mode`): “CM” membaca `Date A`, “LM” membaca `Date B` — jadi CM/LM bisa dua periode bebas, bukan wajib bulan ini/lalu.
- **Attribute** (NOMINAL/QTY) mengubah mesin `[Top10Dev]`, `[Sales CM]`, waste, dan format `[Dynamic Dev]`; **Trend** mengubah `[Multi Value CM]`; **Periode** (Week 1/2/3/EoM) mengatur cutoff YTD & weekly.
- 30 measure tidak dipakai visual langsung (daftar + analisis kandidat hapus: [measure-usage.md](./measure-usage.md)).

## Terkait

- [model-and-dax.md](./model-and-dax.md) — DAX lengkap 161 measure (verbatin) + relasi + Power Query
- [measure-usage.md](./measure-usage.md) — index dampak penghapusan (pemakaian + dependensi + 30 measure tak terpakai)
- [pages-detail.md](./pages-detail.md) — komponen 539 visual per halaman (sumber kolom “Dipakai”)
- [REPORT_OVERVIEW.md](./REPORT_OVERVIEW.md) — ringkasan arsitektur report
