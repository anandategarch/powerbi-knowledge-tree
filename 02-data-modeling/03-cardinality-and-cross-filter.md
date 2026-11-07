# Cardinality & Cross Filter Direction

> Cardinality menentukan "bentuk" relasi (M:1, 1:1, M:M) dan cross-filter direction menentukan arah aliran filter. Kombinasi keduanya menentukan performa, keamanan (RLS), dan kebenaran angka di report.

## Cardinality: Detail Implikasi

### Many-to-One (M:1) — default sehat

- Filter dimensi → fact (single direction) sudah cukup untuk 95% kebutuhan slicing.
- Engine VertiPaq melakukan *table expansion*: atribut dimensi digabungkan secara logis ke fact saat evaluasi → cepat.
- Syarat: key sisi "one" unik dan tidak blank.

### One-to-One (1:1)

- Kedua kolom harus unik; cocok untuk tabel dengan grain identik.
- Lebih sering menandakan dua tabel seharusnya di-merge di Power Query — kecuali keduanya dari sumber berbeda yang tidak boleh digabung (mis. master dari Google Sheets + lampiran lokal).

### Many-to-Many (M:M)

- Relasi *weak*: engine tidak bisa menjamin referential integrity, evaluasi lebih mahal, dan baris tanpa pasangan hilang diam-diam (tanpa blank row).
- Wajar untuk relasi dimensi-ke-dimensi via bridge (mis. `Resto` — `Standar Susut` — `Inventory` via Type Kitchen), tetapi untuk fact → dimension, M:M hampir selalu berarti key dimensi kotor.

## Cross-Filter Direction: Single vs Both

| Aspek | Single | Both |
|---|---|---|
| Prediktabilitas | Tinggi — satu jalur filter | Rendah — jalur bisa bercabang |
| Performa | Optimal | Overhead di setiap query |
| Ambiguity error | Tidak mungkin | Mungkin saat jalur ganda |
| RLS | Arah terkontrol | Bisa menyebar terlalu jauh |

**Contoh visual error ambiguity**: bila `Inventory ←(both) Waste All (both)→ Resto` sekaligus `Inventory ←(both) Deviasi All (both)→ Resto`, lalu satu visual mengambil kolom dari `Inventory` dan `Resto` bersamaan, engine menemukan lebih dari satu jalur filter antara kedua tabel → visual gagal dengan pesan *ambiguity*. Solusinya: kembalikan relasi ke single direction.

## Kapan Both-Direction Dibolehkan

Pola sah satu-satunya: **dimensi ke dimensi melalui bridge table** yang grain-nya "hubungan" (bukan transaksi):

```
 Resto (dim) --M:1--> Standar Susut (bridge: Type Kitchen x Barang) <--M:1-- Inventory (dim)
```

Dengan both-direction di kedua relasi tersebut, slicer `Resto[Area]` bisa memfilter visual daftar `Inventory` (barang yang punya standar susut di area itu). Catatan:

- Aktifkan both **hanya** pada relasi menuju/dari bridge, bukan ke fact.
- Ukur dampaknya di Performance Analyzer.
- Alternatif tanpa both-direction permanen: measure dengan `CROSSFILTER` (lebih terkontrol).

## Dampak RLS dengan Both-Direction

Row-Level Security (RLS) mengalir mengikuti arah cross-filter. Perilaku penting:

- Default: security filter mengalir satu arah (dimensi terproteksi → fact). Dimensi lain yang berelasi ke fact **tidak otomatis terproteksi**.
- Opsi **"Apply security filter in both directions"** pada relasi membuat filter RLS menyeberang ke dimensi lain — berguna (misal PIC hanya melihat resto-nya, dan Area-nya ikut terfilter), tetapi berhati-hatilah: dimensi yang tidak terproteksi bisa "bocor" daftar nilainya bila hubungan both-direction salah pasang.
- M:M + RLS adalah kombinasi rawan — uji setiap perubahan dengan **View As Roles**.

## Sintaks & Contoh

Dimensi `Resto` berisi hierarki natural: `Area` → `Resto`, plus atribut `PIC` dan `AM`.

**Kasus 1 — slicer Area memfilter measure deviasi** (kebutuhan harian):

```dax
Deviasi Rp by Area :=
CALCULATE (
    SUM ( 'Deviasi All'[Nilai] ),
    'Attribute'[Attribute] = "NOMINAL"
)
-- Slicer Resto[Area] -> filter Resto -> (relasi M:1 single) -> Deviasi All
-- TIDAK butuh both-direction sama sekali.
```

**Kasus 2 — slicer kategori barang memfilter visual daftar Resto** (dimensi → dimensi):

```dax
Resto Terdampak :=
CALCULATE (
    DISTINCTCOUNT ( 'Deviasi All'[Resto] ),
    CROSSFILTER ( 'Deviasi All'[Resto], 'Resto'[Resto], BOTH )
)
-- Slicer Inventory[Kategori] mengalir ke fact via single direction (aman),
-- lalu CROSSFILTER membalik arah relasi fact->Resto HANYA untuk measure ini,
-- sehingga DISTINCTCOUNT hanya menghitung resto yang punya transaksi.
```

Pola `CROSSFILTER`-per-measure lebih aman daripada both-direction permanen: visual lain tidak terdampak, dan maksud analitis menjadi eksplisit di dalam measure.

## Kesalahan Umum

- **Mengaktifkan both-direction untuk mengatasi visual kosong** — akar masalah biasanya blank row (key fact tidak ada di dimensi) atau measure yang salah grain. Perbaikan: audit "(Blank)" dulu, lalu kembalikan direction ke single.
- **M:M dibiarkan pada fact → dimension** — perbaiki key: dedup dimensi (TRIM, UPPER, hapus duplikat) sampai relasi bisa M:1.
- **1:1 dipakai karena "tabelnya kecil"** — lebih baik merge di Power Query; 1:1 menambah relasi tanpa manfaat analitis.
- **Lupa cek "Apply security filter in both directions"** saat setup RLS — filter keamanan tidak mengalir sesuai ekspektasi, atau sebaliknya mengalir terlalu jauh.
- **Both-direction di banyak relasi sekaligus** — menuju model ambiguity; setiap penambahan tabel baru bisa memicu error visual yang sulit dilacak.

## Tips Pro

- Prinsip: **ubah model, bukan arah filter, agar slicing bekerja**. Bila butuh dimensi "berbicara" dengan dimensi, pertimbangkan bridge + measure `CROSSFILTER`.
- Model user membuktikan pola ini bisa jalan: seluruh relasi M:1 single-direction, hierarki Area → Resto tetap ter-filter sempurna.
- Cek kesehatan arah filter lewat **Manage relationships** — kolom "Cross filter" harus seragam "Single" kecuali bridge yang disengaja.
- Saat mengaudit model warisan, daftar semua relasi M:M dan tanyakan: "bisakah ini dinaikkan kelas menjadi M:1 setelah cleaning?" Jawabannya hampir selalu bisa.

## Terkait

- [Star Schema](./01-star-schema.md)
- [Relationships](./02-relationships.md)
- [Date Dimension](./04-date-dimension.md)
- [Modeling Best Practices](./05-modeling-best-practices.md)
- [CALCULATE](../01-dax/06-calculate-and-filter-functions.md)
- [Analisis model report cost control](../11-user-report-analysis/model-and-dax.md)
