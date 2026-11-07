# Information Functions: ISFILTERED, SELECTEDVALUE, ISINSCOPE

> Fungsi informasi "bertanya" tentang konteks: apakah kolom ini difilter, berapa nilai yang terlihat, di level matrix mana kita berada. Mereka adalah otak dari pattern Mode DINAMIS (`ISFILTERED('Mode'[Mode])`), header dinamis, dan subtotal dengan logika berbeda per level.

## Konsep: Membaca Filter Context dari Dalam

Measure dievaluasi dalam sebuah filter context (slicer, axis, visual, relasi). Fungsi informasi memungkinkan measure **bereaksi** terhadap konteks itu alih-alih hanya dijumlahkan: menyembunyikan [% Sales] saat attribute QTY, berubah judul saat user memilih tren tertentu, atau menghitung rata-rata di subtotal alih-alih penjumlahan. Semuanya sangat murah secara performa — aman dipanggil berkali-kali.

## Function Reference

| Function | Deskripsi | Contoh |
|---|---|---|
| ISBLANK | Apakah **nilai** ini blank | `ISBLANK([Deviasi CM])` |
| ISEMPTY | Apakah **tabel** ini kosong | `ISEMPTY(VALUES('Deviasi All'[RESTO]))` |
| HASONEVALUE | Kolom punya tepat satu nilai terlihat (dari sumber filter apa pun) | `HASONEVALUE('Resto'[Resto])` |
| HASONEFILTER | Kolom punya tepat satu nilai yang difilter **langsung** | `HASONEFILTER('Resto'[Resto])` |
| ISFILTERED | Kolom/tabel difilter langsung (slicer, axis) | `ISFILTERED('Mode'[Mode])` |
| ISCROSSFILTERED | Kolom terfilter tak langsung lewat relasi | `ISCROSSFILTERED('Deviasi All'[RESTO])` |
| ISINSCOPE | Kolom sedang menjadi level baris/kolom visual saat ini | `ISINSCOPE('Inventory'[Nama Barang])` |
| SELECTEDVALUE | Nilai tunggal kolom, atau default | `SELECTEDVALUE('Trend'[Kategori], "Semua")` |
| ISSELECTEDMEASURE | Measure aktif (khusus calculation group) | `ISSELECTEDMEASURE([Deviasi CM])` |
| USERNAME / USERPRINCIPALNAME | Identitas user untuk RLS | `USERPRINCIPALNAME()` |
| LOOKUPVALUE | Ambil nilai berdasar kunci (ringkas — detail di file relasional) | `LOOKUPVALUE(Resto[Area], Resto[Resto], [Cabang])` |

## ISBLANK vs ISEMPTY

`ISBLANK` menerima **nilai skalar**; `ISEMPTY` menerima **tabel**. Keduanya sering tertukar padahal pertanyaannya berbeda: "apakah nilainya kosong" vs "apakah tidak ada baris sama sekali".

```dax
-- Bulan ini belum ada baris deviasi untuk resto itu?
Ada Data Deviasi? =
IF(
    ISEMPTY(VALUES('Deviasi All'[RESTO])),
    "Belum ada deviasi bulan ini",
    FORMAT([Deviasi CM], "#,##0")
)
```

## HASONEVALUE vs HASONEFILTER vs ISFILTERED vs ISCROSSFILTERED

- **HASONEVALUE(col)** → TRUE jika `VALUES(col)` mengembalikan tepat satu baris — *dari mana pun* filternya berasal (slicer, axis, cross-filter relasi).
- **HASONEFILTER(col)** → TRUE hanya jika kolom difilter **langsung** dengan tepat satu nilai (mis. satu pilihan slicer).
- **ISFILTERED(col)** → TRUE jika ada filter langsung pada kolom/tabel (satu atau banyak nilai). Inti deteksi mode di report ini.
- **ISCROSSFILTERED(col)** → TRUE jika kolom terfilter **lewat relasi** dari tabel lain (bukan filter langsung).

## Sintaks & Contoh

### Pattern asli: Mode DINAMIS dengan ISFILTERED

```dax
-- Slicer 'Mode' (disconnected) mengubah perilaku measure,
-- TIDAK memfilter visual. Inilah cara deteksinya:
Deviasi CM =
VAR Filtered = ISFILTERED('Mode'[Mode])
RETURN
IF(
    Filtered,
    CALCULATE(
        [Deviasi Value],
        TREATAS(VALUES('Date A'[Date]), 'Date'[Date])
    ),
    [Deviasi Value]
)
```

`ISFILTERED('Mode'[Mode])` → TRUE hanya saat user memilih sesuatu di slicer Mode; measure lalu beralih ke relasi virtual Date A (lihat [Relational Functions](./08-relational-functions.md)).

### SELECTEDVALUE dengan default — header dan cabang aman

```dax
-- Header kartu/matrix dinamis: ikut pilihan slicer 'Trend'
Header Trend =
SELECTEDVALUE('Trend'[Kategori], "Semua Kategori")

-- Judul yang aman di total row:
Judul Kartu =
"Deviasi — " & SELECTEDVALUE('Resto'[Resto], "Semua Cabang")

-- Sales hanya bermakna dalam NOMINAL — sembunyikan di mode QTY / total:
Sales CM =
IF(
    SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL",
    SUM('Sales All'[Penjualan Bersih]),
    BLANK()
)
```

`SELECTEDVALUE(col, default)` setara `IF(HASONEVALUE(col), VALUES(col), default)` — tulis yang ringkas. Tanpa default, baris total (banyak nilai) mengembalikan BLANK.

### ISINSCOPE — subtotal beda logika per level matrix

```dax
-- Level item: tampilkan nilai.
-- Level resto: subtotal pakai rata-rata item, bukan penjumlahan.
-- Grand total: kembali ke nilai global.
Deviasi (Matrix) =
SWITCH(
    TRUE(),
    ISINSCOPE('Inventory'[Nama Barang]),
        [Deviasi CM],
    ISINSCOPE('Resto'[Resto]),
        AVERAGEX(VALUES('Inventory'[Nama Barang]), [Deviasi CM]),
    [Deviasi CM]
)
```

**ISINSCOPE vs SELECTEDVALUE untuk level matrix**: `ISINSCOPE` bertanya "apakah saya sedang berada di level ini di visual" — bekerja juga saat level punya banyak nilai. `SELECTEDVALUE` hanya berguna ketika konteksnya tepat satu nilai (mis. slicer). Untuk subtotal hierarchy, ISINSCOPE adalah alat yang tepat; urutkan pengecekan dari level terdalam ke terluar.

### ISSELECTEDMEASURE — khusus calculation group

```dax
-- Di dalam calculation item (calculation group), berlaku hanya ke measure tertentu:
IF(
    ISSELECTEDMEASURE([Deviasi CM]),
    CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH('Date'[Date])),
    SELECTEDMEASURE()
)
```

Hanya bermakna dalam calculation group — biasa dipakai agar item "LM" tidak merusak measure lain yang tidak relevan.

### USERNAME / USERPRINCIPALNAME — RLS

```dax
-- Filter RLS pada tabel Resto: cabang hanya melihat datanya sendiri
Resto[Email PIC] = USERPRINCIPALNAME()

-- Pengecualian untuk user pusat:
IF(
    USERPRINCIPALNAME() = "bi@perusahaan.com",
    TRUE(),
    Resto[Email PIC] = USERPRINCIPALNAME()
)
```

## Kesalahan Umum

- **SELECTEDVALUE tanpa default** → di total/slicer multi-pilih hasil BLANK; kalau hasilnya dipakai sebagai kondisi (`SELECTEDVALUE(Attribute[Attribute]) = "NOMINAL"`), total row ikut BLANK — kadang disengaja ([Sales CM] jadi kosong di total), kadang bug. *Perbaikan*: tulis default eksplisit sesuai niat.
- **`[X] = BLANK()` untuk cek kosong** → TRUE juga saat `[X] = 0` karena koersi perbandingan. *Perbaikan*: pakai `ISBLANK([X])`.
- **HASONEVALUE dipakai untuk deteksi slicer langsung** → cross-filter relasi bisa membuatnya TRUE tanpa ada slicer. *Perbaikan*: gunakan `HASONEFILTER` kalau yang dimaksud filter langsung.
- **ISFILTERED untuk filter tak langsung** → FALSE karena hanya mendeteksi filter langsung. *Perbaikan*: gunakan `ISCROSSFILTERED`.
- **ISEMPTY pada skalar / ISBLANK pada tabel** → type error. *Perbaikan*: ISBLANK untuk nilai, ISEMPTY untuk tabel.
- **ISINSCOPE dipakai di kartu tunggal** → selalu FALSE (tidak ada level visual); cabang grand total yang terpilih. *Perbaikan*: pahami konteks visual sebelum memilih fungsi.

## Tips Pro

- Kombinasi emas report ini: `ISFILTERED` (deteksi mode) + `SELECTEDVALUE` (baca pilihan) + `TREATAS` (terapkan ke tabel sungguhan) = measure adaptif yang bentuknya berubah mengikuti slicer.
- `SELECTEDVALUE(col, default)` ≡ `IF(HASONEVALUE(col), VALUES(col), default)` — singkat, tapi tulis default-nya.
- Untuk subtotal matrix: urutkan `SWITCH(TRUE(), ...)` dari level terdalam (item) ke terluar (total) — analog urutan prioritas SWITCH di [Logical Functions](./09-logical-functions.md).
- RLS: selalu `USERPRINCIPALNAME()` untuk filter produksi; `USERNAME()` untuk pengujian di Desktop (hasilnya beda antar lingkungan).
- Fungsi informasi murah, tapi `VALUES(kolom_besar)` berulang kali dalam satu measure tetap mahal — simpan ke `VAR`.

## Terkait

- [Relational Functions](./08-relational-functions.md) — TREATAS pasangan ISFILTERED pada Mode DINAMIS
- [Logical Functions](./09-logical-functions.md) — SWITCH(TRUE()) untuk cabang ISINSCOPE/ISBLANK
- [Time Intelligence](./07-time-intelligence.md) — SELECTEDVALUE Year/Month/Week pada YTD cutoff mingguan
- [Text Functions](./11-text-functions.md) — label & header dinamis hasil SELECTEDVALUE
- [Data Model & DAX Report Cost Control](../11-user-report-analysis/model-and-dax.md) — measure ISINSCOPE dan SELECTEDVALUE di report asli
