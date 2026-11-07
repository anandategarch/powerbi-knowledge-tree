# Errors Umum Power BI: Diagnosis & Perbaikan

> Kumpulan error Power BI yang paling sering muncul saat mengembangkan dan mengoperasikan report Cost Control — dari DAX, model, Power Query, sampai refresh di service. Format: Error | Penyebab | Solusi, disusun agar bisa di-scan cepat saat produksi sedang bermasalah.

## Konsep & Penjelasan

### Cara membaca error Power BI
- **Dari mana errornya?** Tiga lapis: (1) *Power Query/M* — terjadi saat load (buka Desktop / refresh); (2) *Engine DAX* — terjadi saat visual dihitung; (3) *Service/refresh* — terjadi saat scheduled refresh (cek Refresh history).
- Baca pesan error dari **dalam ke luar**: baris pertama biasanya teknis (`Expression.Error`), baris terakhir biasanya konteks (query mana/step mana).
- Selalu cek dulu: apakah error muncul di **semua visual** (model/refresh) atau **satu visual** (DAX/interaksi)?

### Pola diagnosis 5 menit
1. Apakah barunya saja terjadi? → kembalikan perubahan terakhir (Git: `git checkout -- .` atau restore version history).
2. Apakah error ada di Desktop & service, atau salah satu? → kalau hanya service: kredensial/gateway/privacy level.
3. Apakah error muncul dengan filter tertentu? → konteks/filter measure.
4. Apakah muncul setelah file sumber berubah (nama file, sheet, kolom)? → query PQ rusak karena struktur sumber berubah.
5. Apakah muncul hanya pada satu user? → RLS/kredensial pribadi, bukan bug model.

### Kode error refresh di service yang sering muncul
| Potongan pesan di Refresh history | Arti & rujukan |
|---|---|
| `invalid credentials` | Login/kredensial kedaluwarsa atau salah → error #8 |
| `gateway not reachable / offline` | Mesin gateway mati → error #9 |
| `file is locked / being used by another process` | Excel dibuka saat refresh → error #7 |
| `key not found / couldn't find folder` | `H:\My Drive` belum sinkron / path berubah → error #18 |
| `data sources cannot be combined (firewall)` | Privacy level bentrok → error #10 |

## Sintaks & Contoh

Empat pasang (salah → benar) yang menutup mayoritas kasus:

```dax
-- A. "A table of multiple values was supplied…"
-- SALAH: kolom dipakai langsung di tempat skalar
Selisih = [Deviasi CM] - 'Deviasi All'[NOMINAL_DEVIASI]
-- BENAR: agregasikan dulu, atau ambil nilai tunggal
Selisih = [Deviasi CM] - SUM('Deviasi All'[NOMINAL_DEVIASI])
-- atau
Periode Dipilih = SELECTEDVALUE('Date'[Year], YEAR(TODAY()))
```

```m
// B. Duplicate rows setelah merge (fan-out)
// SALAH: merge dengan tabel yang key-nya dobel
Gabung = Table.NestedJoin(Fact, {"Nama Barang"}, Master, {"Nama Barang"}, "M", JoinKind.LeftOuter)
// BENAR: dedupe master dulu
Master = Table.Distinct(Master, {"Nama Barang"})
```

```m
// C. Relasi unmatched karena kunci kotor (trailing space — kasus nyata di report ini:
//    kolom "Nama  new item " dan "(Old) Nama Barang")
TrimKey = Table.TransformColumns(Source, {{"Nama Barang", Text.Trim, type text}})
UpperKey = Table.TransformColumns(TrimKey, {{"Nama Barang", Text.Upper, type text}})
```

```dax
-- D. Total persen "salah" (sum of percentages)
-- SALAH: persen dihitung per baris (mis. di Excel) lalu dijumlahkan visual → total 115% dsb.
-- BENAR: hitung sebagai measure — total dihitung ulang oleh engine
% Kontribusi := DIVIDE([Deviasi CM], CALCULATE([Deviasi CM], ALL(Resto)))
```

## Kesalahan Umum — Daftar Error | Penyebab | Solusi

| # | Error | Penyebab | Solusi |
|---|---|---|---|
| 1 | "A table of multiple values was supplied where a single value was expected" | Kolom/tabel dipakai di posisi skalar (mis. `SUM` hilang, `SELECTEDVALUE` tidak dipakai) | Agregasi (`SUM/MAX`), `SELECTEDVALUE(col, default)`, atau `VALUES` + `HASONEVALUE` guard |
| 2 | "The expression refers to multiple columns. Multiple columns cannot be converted to a scalar value." | Filter table multi-kolom dijahit langsung ke skalar; `FILTER` hasilnya dipakai sebagai nilai | Bungkus dalam `CALCULATE`; pastikan `LOOKUPVALUE`/`SELECTEDVALUE` menerima skalar |
| 3 | "Circular dependency detected" (calculated column / measure) | Dua kolom/measure saling refer; relasi dua-arah ganda; calc column yang agregat tabel sendiri | Putus satu arah: jadikan measure; ubah relasi single-direction; pindahkan logika ke measure |
| 4 | Baris visual blank / hilang | Relasi unmatched (trailing space/case beda — `Nama  new item ` vs `(Old) Nama Barang`); filter context memang kosong | Bersihkan key di PQ (TRIM/UPPER); cek dengan visual tabel key kiri vs kanan; tangani BLANK secara eksplisit |
| 5 | Total baris tidak sesuai (total persen ≠ 100%, total salah) | Persentase di-sum di total; agregasi implicit pada calculated column; total dihitung dari kolom, bukan measure | Hitung ratio sebagai **measure** (totals dihitung ulang oleh engine); jangan agregasi kolom persen |
| 6 | "We couldn't get data from the data model / not enough memory" | Model terlalu besar saat kompresi/visual (145 MB + 539 visual); measure iterator berat | Kurangi kolom, matikan auto date/time, pecah visual per halaman, optimasi measure (lihat [FAQ](./faq.md) "measure lambat") |
| 7 | Refresh gagal: file locked / `~$` | Excel sumber sedang dibuka orang saat refresh jalan | Filter `~$` di PQ; jadwalkan refresh di jam tanpa orang mengedit; disiplin closing file |
| 8 | Refresh gagal: kredensial | Password/OAuth token kedaluwarsa; kredensial belum diisi di service | Dataset settings > data source credentials > edit; re-login Google/Sheets; test dengan Refresh now |
| 9 | Refresh gagal: gateway offline / tidak bisa konek | Mesin gateway mati/log off/tidur; `H:\My Drive` tidak sinkron | Gateway always-on (mini PC/VM), auto-login, power never sleep; pantau heartbeat; lihat [Gateway & Refresh](../09-publish-and-deploy/02-gateway-and-refresh.md) |
| 10 | `Formula.Firewall` / "data sources cannot be combined" | Privacy level berbeda digabung dalam satu query | Samakan privacy level (Organizational) atau pisahkan query per sumber lalu merge di model |
| 11 | "Failed to save modifications to the server… cannot be loaded onto the table" | Nama kolom duplikat setelah expand/merge; tipe data bentrok; nama reserved | Rename unik sebelum expand; cek `Table.ColumnNames`; hindari nama yang sama dua kolom |
| 12 | Duplicate rows setelah merge | Join fan-out (master key dobel) | `Table.Distinct` pada master sebelum merge; validasi key unik di PQ |
| 13 | Measure tidak muncul di field list | Measure hidden (atau salah tabel/di display folder) | Unhide di model view; cek display folder; pastikan bukan calculated column |
| 14 | Slicer tidak memfilter visual | Edit interactions off; slicer dari tabel disconnected (sengaja, ala `Periode`/`Trend`) | Nyalakan edit interactions; untuk tabel disconnected pakai `TREATAS`/`SELECTEDVALUE` di measure |
| 15 | "Cannot display the visual / request too large / too much data" | Query result melebihi batas visual (baris terlalu banyak) | Tambah TOPN/filter default; kurangi granularitas; pecah halaman |
| 16 | Visual tidak terfilter lintas tabel | Relationship inactive (atau tidak ada); relasi M:M tanpa bridge | Aktifkan dengan `USERELATIONSHIP`, atau perbaiki model; hindari relasi ganda searah |
| 17 | Datetime/format beda locale | `dd/mm` vs `mm/dd` dibaca sebagai teks/serial number (45283) | Set locale di PQ per kolom; gunakan tipe Date eksplisit; jangan join tanggal sebagai teks |
| 18 | "Expression.Error: We couldn't find the specified folder/file" (`H:\My Drive`) | Google Drive belum sinkron / mesin log off / path berubah | Pastikan sync jalan & mesin online; pindah sumber ke OneDrive/SharePoint (rekomendasi) |
| 19 | `Web.Contents` Google Sheets 404 / data macet | Published URL berubah saat sheet di-publish ulang | Pakai connector Google Sheets resmi (OAuth) — URL tidak bergantung publish |
| 20 | "Token Eof expected" / DAX merah di editor | Syntax error (kurang kurung, koma, kutip) | Format ulang measure; pecah baris; cek koma desimal vs titik (locale) |
| 21 | "Couldn't load the model / file is being used by another process" | `.pbix` dibuka di instance Desktop lain / ter-lock sinkronisasi OneDrive | Tutup instance lain; copy ke folder kerja lokal; jangan edit langsung dari OneDrive |
| 22 | "The column 'X' of the table wasn't found" | Kolom di-rename/dihapus setelah measure/query PQ mereferensikannya | Kembalikan nama lama; cari semua dependen (Advanced Editor / Ctrl+F) dan perbarui sekaligus |
| 23 | Slicer/axis memunculkan nilai **(Blank)** | Baris fact tanpa match ke dimensi (key kotor) membentuk nilai blank | Bersihkan key (TRIM/UPPER); matikan "show items with no data"; buang baris tanpa match di PQ bila memang sampah |
| 24 | Analyze in Excel gagal / angka calc group mentah | Build permission belum diberikan; klien MDX klasik tidak menerapkan calculation group | Beri Build permission di dataset; untuk calc group, verifikasi di report/web — lihat [Calculation Groups](../10-advanced/01-calculation-groups.md) |

## Tips Pro
- Simpan halaman `QA Checks` hidden berisi tabel angka kunci vs sumber Excel — saat error #4/#5 muncul, halaman ini jadi titik diagnosis.
- Saat error service: buka **Refresh history** dulu, lalu **lineage view** — dua layar itu menyelesaikan 80% kasus produksi.
- Log setiap error produksi (tanggal, error, solusi) di repo — knowledge base seperti file ini tumbuh dari log tersebut.
- Setelah perbaikan apa pun: jalankan [DAX Query View](../10-advanced/05-dax-query-view.md) untuk memastikan angka kembali sama dengan sumber.
- Kenali pola: error yang muncul *setelah* perubahan terakhir hampir selalu disebabkan perubahan itu — jangan berburu hantu di tempat lain.

## Terkait
- [FAQ](./faq.md)
- [Gateway & Refresh](../09-publish-and-deploy/02-gateway-and-refresh.md)
- [Calculation Groups](../10-advanced/01-calculation-groups.md)
- [DAX Query View](../10-advanced/05-dax-query-view.md)
- [Analisis Model & DAX](../11-user-report-analysis/model-and-dax.md)
