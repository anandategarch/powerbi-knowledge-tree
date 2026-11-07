# Star Schema — Fondasi Data Model Power BI

> Star schema adalah pola desain model di mana tabel **fact** (fakta) transaksional dikelilingi tabel **dimension** (dimensi) melalui relasi many-to-one. Mesin Power BI (VertiPaq) dioptimalkan untuk pola ini: DAX lebih sederhana, filter propagation terprediksi, dan ukuran file lebih ramping.

## Fact Table vs Dimension Table

**Fact table** menyimpan *event* — angka yang bisa dijumlahkan (additive):

- `Deviasi All`, `Waste All`, `Sales All`, `9901` (PO), `3243` (status pesanan), dan `Rekap Mentah BM` adalah fact table: satu baris mewakili satu pengukuran.
- Isinya: kolom nilai (`Nilai`, `Amount`, `QTY`) dan foreign key ke dimensi (`Resto`, `Nama Barang`, `Date`, `Periode`, `Attribute`).
- Tidak menyimpan atribut deskriptif — `Area`, `PIC`, `Kategori` hidup di dimensi, bukan di fact.

**Dimension table** menyimpan *konteks* — untuk slice, dice, filter, dan label visual:

- `Inventory` (master barang, PK `(Old) Nama Barang`), `Resto` (Resto, Area, PIC, AM, Type Kitchen), `Date`, `Periode` (WEEK 1 sampai EOM), `Attribute` (NOMINAL/QTY).
- Cirinya: baris relatif sedikit (puluhan sampai ribuan), punya primary key unik, dan kolomnya menjadi kandidat slicer/axis/legend.

Aturan praktis: **fact menjawab "berapa", dimension menjawab "siapa / apa / kapan / di mana"**.

## Diagram Contoh: Model Cost Control

```
 Inventory                  Resto                    Date
 (Old) Nama Barang (PK)     Resto (PK)               Date (PK)
 Kategori, Satuan, HNA      Area, PIC, AM,           Tahun, BulanNo,
                           Type Kitchen             Kuartal, Minggu
     |                         |                         |
     | M:1                     | M:1                     | M:1
     +-----------+------------+-------------------------+
              +--+------------+--+
              |   Deviasi All   |
              | Resto, Nama     |
              | Barang, Date,   |
              | Periode,        |
              | Attribute,      |
              | Jenis, Nilai    |
              +--+----------+--+
                 | M:1      | M:1
             +---+---+  +---+------+
             |Periode|  |Attribute |
             |WEEK 1-|  |NOMINAL / |
             |EOM    |  |QTY       |
             +-------+  +----------+
```

Pola yang sama berlaku untuk `Waste All` dan `Sales All` — beberapa fact table boleh berbagi dimensi yang sama (disebut **conformed dimensions**), sehingga slicer `Area` atau `Periode` bekerja konsisten di semua halaman report.

## Snowflake vs Star

- **Star**: dimensi langsung menempel ke fact, dan dimensi sudah **denormalized** — semua atribut cabang (Area, PIC, AM, Type Kitchen) berada dalam satu tabel `Resto`.
- **Snowflake**: dimensi dipecah lagi menjadi sub-dimensi (mis. `Resto` → tabel `Area` terpisah → tabel `PIC` terpisah). Secara teori lebih "normal", tetapi di Power BI justru menambah jumlah relasi dan hop filter tanpa manfaat kompresi berarti.

Gunakan snowflake hanya jika sub-dimensi benar-benar dipakai bersama oleh banyak dimensi lain atau datanya sangat besar; selebihnya, flatten menjadi star.

## Anti-Pattern: One Big Table

Menggabungkan semua kolom (fakta + seluruh atribut dimensi) ke satu tabel "supaya gampang" adalah kesalahan klasik:

- **Ukuran membengkak** — nilai seperti `Area` dan `PIC` diulang di setiap baris transaksi. Kompresi VertiPaq sangat efektif untuk nilai berulang pada dimensi kecil, tetapi buruk saat atribut tekstual menempel di tabel besar.
- **Performa** — setiap visual harus memindai tabel besar; filter tidak bisa "dipendekkan" lewat dimensi kecil.
- **Tidak bisa slice konsisten** — kategori yang diperbaiki di bulan ini tidak otomatis retroaktif ke bulan lalu; label ganda ("AYAM" vs "Ayam") menjadi sumber deviasi semu.
- **DAX meledak** — tanpa relasi, semua perbandingan harus `FILTER` + `LOOKUPVALUE` manual.

## Grain (Granularity) Fakta

**Grain** = definisi satu baris fact, dinyatakan sebagai kombinasi dimensi terkecil. Contoh pada model cost control:

> Satu baris `Deviasi All` = **resto x barang x tanggal/periode x attribute x jenis**

Konsekuensi grain:

- Grain antar fact boleh berbeda — `Sales All` bisa per resto per hari per attribute, sementara `Rekap Mentah BM` per barang per tanggal penerimaan.
- **Jangan campur grain dalam satu tabel** (misal baris detail transaksi dicampur baris subtotal bulanan) — agregasi jadi double counting.
- Semua measure dievaluasi dalam grain fact-nya; untuk naik/turun grain gunakan measure dengan agregasi eksplisit, bukan menambah baris summary.

## Sintaks & Contoh

Langkah praktis membangun star schema dari data mentah:

1. **Identifikasi event** → kandidat fact (deviasi, waste, sales, PO).
2. **Identifikasi atribut deskriptif** → kandidat dimensi (barang, cabang, tanggal, periode).
3. **Keluarkan atribut dari fact** — simpan hanya key di fact (merge di Power Query, bukan duplikasi).
4. **Bersihkan key** — dedup dimensi, TRIM/UPPER key agar relasi M:1 bersih.
5. **Buat relasi M:1 single direction** dari setiap fact ke setiap dimensi.
6. **Verifikasi grain** — `DISTINCTCOUNT` kombinasi key harus sama dengan jumlah baris fact.

Setelah berbentuk star, measure sederhana dan bisa di-slice dari dimensi mana pun:

```dax
-- Measure: total nilai deviasi, hanya attribute NOMINAL
Deviasi Rp :=
CALCULATE (
    SUM ( 'Deviasi All'[Nilai] ),
    'Attribute'[Attribute] = "NOMINAL"
)

-- Taruh di matrix dengan Resto[Area] di rows:
-- filter Area -> Resto -> Deviasi All terjadi otomatis via relasi M:1,
-- tanpa perlu LOOKUPVALUE atau FILTER manual.
```

## Kesalahan Umum

- **Menyimpan `Area`/`PIC`/`Kategori` di tabel fact** — data membengkak dan label bisa inkonsisten antar bulan. Perbaikan: simpan hanya key `Resto`; atribut diambil via relasi ke dimensi `Resto`.
- **Dimensi tidak unik** — key `(Old) Nama Barang` ganda membuat relasi terdeteksi M:M. Perbaikan: dedup (`Table.Distinct`) atau `Group By` di Power Query sebelum load.
- **Fact tanpa date key** (hanya teks "WEEK 1") — time intelligence mati. Perbaikan: parse tanggal dari nama file (YYYY_MM) lalu relasikan ke `Date`.
- **Fact berformat wide** (kolom `QTY_STOK`, `QTY_WASTE`, `NOMINAL_SALES`, ...). Perbaikan: unpivot menjadi kolom `Attribute` + `Jenis` + `Nilai`.
- **Menjadikan `Standar Susut` dimensi label** padahal grain-nya `Type Kitchen x barang x periode` — tabel itu adalah fact rencana/standar; relasikan sebagai fact lalu bandingkan dengan measure, bukan sebagai atribut.

## Tips Pro

- Simpan definisi grain di **description** tabel (Model view → Properties) — dokumentasi hidup yang muncul sebagai tooltip di panel Fields.
- Fact idealnya **long format** (hasil unpivot) — buktinya: belasan kolom `QTY_*`/`NOMINAL_*` menjadi dua kolom kecil plus dimensi `Attribute`.
- Kalau report lambat, periksa dulu apakah ada atribut tekstual nyasar di fact — biasanya penyebab utama pembengkakan model.
- Conformed dimensions (`Resto`, `Date`, `Periode`) membuat halaman Deviasi, Waste, dan Sales bisa share slicer yang sama — manfaat terbesar star schema.

## Terkait

- [Relationships](./02-relationships.md)
- [Cardinality & Cross Filter](./03-cardinality-and-cross-filter.md)
- [Date Dimension](./04-date-dimension.md)
- [Modeling Best Practices](./05-modeling-best-practices.md)
- [Unpivot & transformasi lain](../03-power-query/02-common-transformations.md)
- [Resep Power Query](../03-power-query/04-pq-patterns-recipes.md)
- [CALCULATE](../01-dax/06-calculate-and-filter-functions.md)
- [Analisis model report cost control](../11-user-report-analysis/model-and-dax.md)
