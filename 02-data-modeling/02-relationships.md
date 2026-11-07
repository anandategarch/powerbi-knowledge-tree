# Relationships — Menghubungkan Fact dan Dimension

> Relasi (relationship) adalah jembatan yang membuat filter mengalir antar tabel. Memilih cardinality dan cross-filter direction yang tepat — default sehat: M:1 single direction — adalah keputusan modeling terpenting setelah star schema.

## Jenis Relasi (Cardinality)

| Jenis | Kapan digunakan | Catatan |
|---|---|---|
| **One-to-many (M:1)** | Default semua fact → dimension | Sisi "one" harus unik; paling cepat |
| **One-to-one (1:1)** | Grain kedua tabel identik | Jarang; pertimbangkan merge di Power Query |
| **Many-to-many (M:M)** | Key ganda di kedua sisi | Weak relationship; biasanya gejala key kotor |

Pada model cost control, semua relasi idealnya M:1 single direction, contohnya:

- `Deviasi All[Nama Barang]` → `Inventory[(Old) Nama Barang]`
- `Waste All[CABANG]` → `Resto[Resto]`
- `Deviasi All[Date]` → `Date[Date]`

Kalau Power BI memaksa M:M saat Anda menginginkan M:1, artinya **key di sisi dimensi tidak unik** — perbaiki datanya, jangan terima M:M begitu saja.

## Active vs Inactive Relationship

Antara dua tabel hanya boleh ada **satu relasi aktif**; relasi tambahan harus dinonaktifkan (inactive) lalu diaktifkan sementara dengan `USERELATIONSHIP`. Kasus klasik: analisis by **tanggal order vs tanggal terima** — PO dibuat di `9901` dengan `Tanggal PO`, barang diterima lewat `3243`/`Rekap Mentah BM` dengan `Tanggal Terima` (contoh DAX di bagian Sintaks & Contoh).

`USERELATIONSHIP` hanya berlaku selama evaluasi measure tersebut — relasi aktif asli "dipinjam sementara", tidak berubah permanen. Fungsi ini hanya bisa dipakai sebagai filter modifier di dalam `CALCULATE`.

## Single vs Both Direction

- **Single (default)**: filter mengalir satu arah — dari sisi "one" (dimensi) ke sisi "many" (fact). Prediktabel dan cepat.
- **Both**: filter mengalir dua arah. Berbahaya karena:
  - **Ambiguitas** — bila ada lebih dari satu jalur antara dua tabel, engine tidak tahu jalur mana yang dipakai → visual error *ambiguity*.
  - **Performa** — engine menyiapkan filter dua arah di setiap relasi untuk setiap query.
  - **Loop potensial** — jalur melingkar memicu error atau hasil tak terduga.

Rule of thumb: **both-direction bukan fitur kelezapan, tapi utang teknis**. Aktifkan hanya pada pola bridge yang disengaja (lihat [Cardinality & Cross Filter](./03-cardinality-and-cross-filter.md)).

## Referential Integrity & Blank Row

Mode Import selalu menjaga integritas relasi dengan mekanisme **blank row**:

- Baris fact yang key-nya tidak ditemukan di dimensi tetap dihitung, tapi dikelompokkan di bawah label **(Blank)** pada sisi dimensi.
- Gejala klasik di report cost control: muncul baris "(Blank)" saat slice by `Area` — artinya ada `CABANG` di fact yang tidak ada di master `Resto` (typo, atau cabang baru belum di-master-kan).
- Di M:M, baris tanpa pasangan **tidak** mendapat blank row — mereka hilang dari hasil filter secara diam-diam, sehingga total bisa "menghilang" tanpa jejak. Alasan lain menghindari M:M.

**Assume referential integrity**: opsi ini hanya relevan untuk DirectQuery — bila dicentang, engine boleh memakai INNER JOIN (bukan LEFT JOIN) saat query ke sumber. Untuk mode Import (seperti model berbasis file Excel), mekanisme blank row selalu dipakai.

## Strong vs Weak Relationship

- **Strong** (M:1 dan 1:1 dengan key unik): engine bisa melakukan *table expansion* — atribut dimensi digabungkan secara logis ke fact saat evaluasi → cepat, dan `RELATED`/`RELATEDTABLE` aman dipakai.
- **Weak** (M:M, atau relasi yang key-nya tidak unik di kedua sisi): tidak ada jaminan integritas → engine memakai jalur evaluasi yang lebih mahal.

## Composite Key

Bila tidak ada key tunggal alami — misalnya `Standar Susut` unik per kombinasi `Type Kitchen` + `Nama Barang` + `Periode` — ada dua opsi:

1. **Merged key column** di Power Query: gabung beberapa kolom menjadi satu key (mis. `"CENTRAL KITCHEN|AYAM|WEEK 1"`), lalu relasikan M:1 ke tabel yang polanya sama.
2. **TREATAS** di DAX tanpa relasi fisik:

```dax
Standar Susut by Transaksi :=
CALCULATE (
    [Standar Susut Nilai],
    TREATAS (
        SUMMARIZE ( 'Deviasi All', Resto[Type Kitchen], Inventory[(Old) Nama Barang], Periode[Periode] ),
        'Standar Susut'[Type Kitchen],
        'Standar Susut'[Nama Barang],
        'Standar Susut'[Periode]
    )
)
```

## Konsistensi Nama Kolom Key

Autodetect relasi mencocokkan nama kolom. Di model nyata, satu entitas bisa punya banyak nama key: `(Old) Nama Barang` (Inventory), `Nama Barang` (Deviasi All), `Nama  new item` (Waste All — perhatikan spasi ganda!), dan `NAMA BARANG` (Rekap Mentah BM). Konsekuensinya:

- Autodetect gagal atau salah pasang → relasi harus di-drag manual di Model view.
- Review model jadi lambat karena harus menghafal sinonim.

Solusi: standarkan nama key di Power Query agar identik dengan nama kolom di tabel dimensi (`Nama Barang` di semua tabel), lalu rename sekali di dimensi bila perlu.

## Sintaks & Contoh

Mengaktifkan relasi inactive sementara dan mengaudit blank row hasil relasi kotor:

```dax
-- 1) Ganti relasi aktif sementara: analisis PO by tanggal terima
--    (relasi aktif : Date[Date] -> 9901[Tanggal PO])
--    (relasi inactive: Date[Date] -> 9901[Tanggal Terima])
Nilai PO by Tanggal Terima :=
CALCULATE (
    SUM ( '9901'[Amount] ),
    USERELATIONSHIP ( 'Date'[Date], '9901'[Tanggal Terima] )
)

-- 2) Audit sumber baris "(Blank)": key fact tanpa pasangan di dimensi
Barang Tanpa Master :=
COUNTROWS (
    FILTER (
        'Deviasi All',
        ISBLANK ( RELATED ( Inventory[(Old) Nama Barang] ) )
    )
)
```

Measure `Barang Tanpa Master` harus menunjukkan 0. Bila bukan 0, telusuri baris fact tersebut di Power Query (Left Anti Join ke `Inventory`) dan perbaiki masternya — bukan sembunyikan blank-nya.

## Tools: Model View

- Drag satu kolom ke kolom lain untuk membuat relasi; klik kanan garis relasi → **Properties** (cardinality, cross-filter direction, active/inactive, "Apply security filter in both directions").
- Rapikan layout: dimensi di atas, fact di bawah — diagram akan terbaca sebagai star schema.
- **Manage relationships**: daftar lengkap semua relasi, tempat tercepat mengecek kardinalitas dan arah filter secara massal.

## Kesalahan Umum

- **Menerima M:M tanpa investigasi** — hampir selalu berarti key dimensi duplikat. Perbaikan: `Table.Distinct`/`Group By` di Power Query, atau pecah tabel bila memang mengandung dua entitas berbeda.
- **Key beda tipe** (teks vs angka) — relasi tidak bisa dibuat atau mencocokkan kosong. Perbaikan: samakan tipe di Power Query (`Text.From` / `Number.From`).
- **Key dengan spasi tersembunyi** ("NAMA  BARANG") — terlihat sama tapi tidak match. Perbaikan: `Text.Trim` + bersihkan spasi ganda.
- **Menyalakan both-direction untuk "memperbaiki" visual kosong** — salah diagnosa; visual kosong biasanya karena blank row, grain, atau measure. Perbaikan: audit data dengan tabel debug dulu.
- **Dua relasi aktif dari Date ke dua kolom tanggal** — tidak diizinkan; jadikan satu inactive + `USERELATIONSHIP`.
- **Relasi fact-to-fact langsung** — dua fact table di-relasikan langsung (M:M diam-diam). Perbaikan: gunakan dimensi bersama (conformed dimension) atau measure `TREATAS`/`CROSSFILTER` eksplisit.

## Tips Pro

- Nama kolom key identik di kedua sisi mempercepat autodetect dan memudahkan review model.
- Key bertipe angka (kode cabang) lebih hemat memori daripada teks panjang — selama keterbacaan master tetap terjaga.
- Setiap kali menambah tabel baru, buka **Manage relationships** dan pastikan sisi "many" selalu fact, bukan sebaliknya.
- Blank row yang muncul di visual adalah alarm data quality gratis — telusuri sebelum di-hidden.

## Terkait

- [Star Schema](./01-star-schema.md)
- [Cardinality & Cross Filter](./03-cardinality-and-cross-filter.md)
- [Date Dimension](./04-date-dimension.md)
- [Modeling Best Practices](./05-modeling-best-practices.md)
- [Cleaning key di Power Query](../03-power-query/04-pq-patterns-recipes.md)
- [CALCULATE](../01-dax/06-calculate-and-filter-functions.md)
- [Analisis relasi report cost control](../11-user-report-analysis/model-and-dax.md)
