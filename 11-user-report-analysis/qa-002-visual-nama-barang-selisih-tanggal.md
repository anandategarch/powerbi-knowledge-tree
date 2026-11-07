# Q&A 002 — Visual: Nama Barang × Selisih × Tanggal Selisih

> **Pertanyaan user:** "visual yang aku mau nama barang, selisih, tanggal selisih nya"
> **Halaman terkait:** `Rekap Brg Masuk` (lanjutan Q&A 001).
> **Ditanyakan:** 7 Nov 2026.

User ingin visual (Table atau Matrix) yang menampilkan 3 hal:
1. **Nama Barang**
2. **Selisih** (`[Selisih BM]`)
3. **Tanggal** selisih terjadi

---

## Cara 1 — Pakai `Rekap Mentah BM[TANGGAL]` (Paling Cepat)

**Field yang di-drag ke Table visual:**

| Slot | Field | Sumber |
|---|---|---|
| Rows / Values (kolom 1) | `Nama Barang` | `Inventory` |
| Columns (kolom 2) | `TANGGAL` | `Rekap Mentah BM` (calculated column yang sudah ada) |
| Values | `[Selisih BM]` | measure existing |

**Langkah:**

1. Halaman `Rekap Brg Masuk` → klik area kosong → **Visualizations → Table** (atau Matrix kalau mau pivot).
2. Drag `Inventory[Nama Barang]` ke Columns / Rows.
3. Drag `Rekap Mentah BM[TANGGAL]` ke Columns (urutkan ascending).
4. Drag measure `[Selisih BM]` ke Values.
5. Format: kolom Selisih → **Conditional formatting → Background color** → rules (merah `< 0`, hijau `> 0`, abu-abu `= 0`) biar surplus/minus kelihatan.

### ⚠️ Catatan akurasi

`[Selisih BM]` = `Inputan − Sistem`, di mana:

- **Inputan** = `SUM('Rekap Mentah BM'[Value])` ← terfilter `Rekap Mentah BM[TANGGAL]` ✅
- **Sistem** = `SUM('3224'[Kts (Unit#1)]) + SUM('4205'[#Qty. Terkecil]) + [Pembelian di Luar]` ← **TIDAK** terfilter `TANGGAL` itu (lihat [model-and-dax.md](./model-and-dax.md)).

Konsekuensi: kalau di-breakdown per tanggal, **sisi sistem akan tetap menampilkan total seluruh periode**, bukan per tanggal itu. `[Selisih BM]` jadi timpang per tanggal.

Cara 1 cocok kalau tujuan visual: "lihat tanggal input restoran + total sistem periode berjalan" — bukan per-hari-per-hari.

---

## Cara 2 — Pakai `Date[Date]` + Perbaiki Measure (Akurat Per Tanggal)

### Langkah 1: Pastikan tiap fact table punya kolom TANGGAL yang konsisten

- `Rekap Mentah BM[TANGGAL]` → **sudah ada** ✅
- `3224[TANGGAL]` & `4205[TANGGAL]` → cek native datetime di Model view, rename jadi `TANGGAL`. Kalau tidak ada, buat calc column.
- `9901[TANGGAL]` → cari kolom tanggal native (biasanya `Tanggal # Pesanan` / `Tanggal Penerimaan`).

### Langkah 2: Tambahkan TREATAS di measure (pola ala `[Deviasi CM]`)

Buat measure baru supaya yang asli tetap aman:

```dax
Selisih BM (by Date) =
VAR Sistem =
    CALCULATE(
        SUM('3224'[Kts (Unit#1)]) + SUM('4205'[#Qty. Terkecil]) + [Pembelian di Luar],
        TREATAS( VALUES('Date'[Date]), '3224'[TANGGAL] ),
        TREATAS( VALUES('Date'[Date]), '4205'[TANGGAL] ),
        TREATAS( VALUES('Date'[Date]), '9901'[TANGGAL] )
    )
VAR Inputan =
    CALCULATE(
        SUM('Rekap Mentah BM'[Value]),
        TREATAS( VALUES('Date'[Date]), 'Rekap Mentah BM'[TANGGAL] )
    )
RETURN
    Inputan - Sistem
```

> TREATAS ganda dalam satu CALCULATE itu filter intersection — semua tabel harus cocok di tanggal yang dipilih. Sesuaikan nama field kalau native date column namanya berbeda.

**Alternatif lebih bersih — bikin relasi fisik** di Model view:

- `3224[TANGGAL]` → `Date[Date]` (Many:1, Single)
- `4205[TANGGAL]` → `Date[Date]` (Many:1, Single)
- `9901[TANGGAL]` → `Date[Date]` (Many:1, Single)
- `Rekap Mentah BM[TANGGAL]` → `Date[Date]` (Many:1, Single)

Setelah ini, `[Selisih BM]` asli otomatis benar per tanggal tanpa measure baru. Disarankan kalau tabel-tabel ini sering dipakai dengan filter tanggal di banyak halaman.

### Langkah 3: Bangun visualnya

**Table visual** (detail per row):

| Slot | Field |
|---|---|
| Values | `Inventory[Nama Barang]` |
| Values | `Date[Date]` |
| Values | `[Selisih BM (by Date)]` (atau `[Selisih BM]` kalau sudah pakai relasi fisik) |

**Matrix visual** (pivot — lebih ringkas):

| Slot | Field |
|---|---|
| Rows | `Inventory[Nama Barang]` |
| Columns | `Date[Date]` (atau `Date[Period]` kalau mau bulanan `yyyy-mm`) |
| Values | `[Selisih BM]` |
| Subtotal | On untuk Rows + Columns |

### Langkah 4: Tunjukkan hanya barang yang ada selisih

Filter pane → tarik measure `[Selisih BM]` ke "Filters on this visual" → **Advanced filtering** → "is not blank" dan "is not equal to 0" → Apply filter. Visual jadi ringkas, cuma barang yang benar-benar ada selisih di tanggal itu.

---

## Kesalahan Umum

| Kesalahan | Akibat | Solusi |
|---|---|---|
| Pakai `Date[Date]` sebelum relasi/TREATAS dibuat | Kolom tanggal tampil tapi selisih tidak terbagi per tanggal (semua tanggal tampil angka sama) | Pilih Cara 1 dulu, atau kerjakan Langkah 2 di Cara 2 |
| Pakai `Rekap Mentah BM[NAMA BARANG]` sebagai row | Nama barang tidak konsisten dengan halaman lain (halaman lain pakai `Inventory[Nama Barang]` yang sudah di-SWITCH rename) | Selalu pakai `Inventory[Nama Barang]` |
| Lupa conditional formatting | Selisih positif/negatif tidak kontras | Background color rules: `<0` merah, `>0` hijau, `=0` abu-abu |
| Pakai `Nama Barang` native dari `3224`/`4205` | Tidak match dengan relasi M:1 ke `Inventory` → beberapa barang hilang | Pakai `3224[Cek Barang]` / `4205[Cek Barang]` (calc column lookup), atau langsung `Inventory[Nama Barang]` |
| Lupa sync slicer `Date` | Filter tanggal hilang saat pindah halaman Detail/Rekon | View → Sync slicers, centang Sync+Visible 3 halaman BM |

## Tips Pro

- **Sort by Selisih BM desc** di tabel → barang dengan selisih terbesar muncul paling atas.
- **Drill through**: chart bar di halaman Rekap Brg Masuk sudah ada drill-through ke `ID Dokumen`. Pastikan field `Date[Date]` ikut sebagai drillthrough field supaya dari chart bisa drill ke tabel detail per tanggal.
- **Tampilan ringkas**: kalau tanggal terlalu banyak, ganti kolom matrix dari `Date[Date]` ke `Date[Period]` (`yyyy-mm`) — satu selisih per bulan per barang.
- **Validasi cepat**: bikin card `Total Selisih = SUMX(VALUES(Inventory[Nama Barang]), [Selisih BM])`, bandingkan dengan grand total visual sebelum/sesudah langkah 2 → harus sama. Kalau beda, ada barang yang tanggalnya nyasar.

## Terkait

- [qa-001-tambah-tanggal-rekap-brg-masuk.md](./qa-001-tambah-tanggal-rekap-brg-masuk.md) — dasar menambahkan tanggal di halaman Rekap Brg Masuk
- [model-and-dax.md](./model-and-dax.md) — definisi `[Selisih BM]`, `[BM Sistem]`, `[BM Resto]`
- [../04-visuals/03-visual-configuration.md](../04-visuals/03-visual-configuration.md) — konfigurasi visual & conditional formatting
- [../01-dax/08-relational-functions.md](../01-dax/08-relational-functions.md) — pola TREATAS
