# Row-Level Security (RLS) — Filter Baris per User

> RLS menyaring baris data berdasarkan siapa yang membuka report: **static RLS** (satu role per nilai, mis. `Area = "JAKARTA"`) atau **dynamic RLS** (tabel mapping `UserSecurity(Email, Area)` + `USERPRINCIPALNAME()`), dievaluasi sebelum DAX visual berjalan sehingga tidak bisa dimatikan dari interaksi report.

## Konsep & Penjelasan

### Cara kerja RLS

RLS didefinisikan sebagai **role** di model (Modeling → Manage roles). Role berisi filter DAX pada tabel; saat user membuka report, Power BI Service mengevaluasi role yang diberikan ke user tersebut, lalu filter itu diterapkan ke *semua query* — measure, visual, export, semuanya. User tidak bisa melihat atau mengubahnya.

Penting: filter role harus dipasang di **sisi dimensi yang benar**. Dengan relasi single-direction dim → fact, filter di `Resto` otomatis menjalar ke `Deviasi All`/`Waste All`/`Sales All`. Filter yang dipasang di tabel fact hanya memengaruhi tabel itu, tidak menjalar ke dimensi lain.

### Static RLS

Satu role per nilai — untuk report cost-control berarti satu role per Area (JAKARTA, BANDUNG, SURABAYA, …), masing-masing dengan filter:

```dax
'Resto'[Area] = "JAKARTA"
```

Sesuai untuk: nilai sedikit dan stabil. Kelemahan: setiap area baru = edit model + publish ulang + atur ulang mapping di Service — rawan telat dan salah.

### Dynamic RLS — pola `UserSecurity(Area, Email)`

Satu role untuk semua user. Yang menentukan data hanya isi tabel mapping. Untuk report user, tabel ini alami ditaruh di Google Sheets (pola master `Inventory` yang sudah dipakai):

| Area | Email |
|---|---|
| JAKARTA | budi.santoso@perusahaan.com |
| JAKARTA | sari@perusahaan.com |
| BANDUNG | deni@perusahaan.com |
| JAWA TIMUR | amo@perusahaan.com |

Role-nya cuma satu, filternya di `UserSecurity`:

```dax
[Email] = USERPRINCIPALNAME ()
```

`USERPRINCIPALNAME()` mengembalikan UPN (email login) user yang sedang membuka report di Service. Jadi hanya baris mapping miliknya yang lolos → filter menjalar `UserSecurity → Area → Resto → seluruh fact`.

Varian tanpa relasi (kalau menambah dim `Area` tidak diinginkan) — filter dipasang langsung di `Resto`:

```dax
[Area] =
LOOKUPVALUE (
    UserSecurity[Area],
    UserSecurity[Email], USERPRINCIPALNAME ()
)
```

Desain ini **fail-closed**: user yang tidak terdaftar → `LOOKUPVALUE` mengembalikan BLANK → tidak ada baris `Resto` yang match → user melihat kosong (bukan melihat semua). Ini perilaku yang diinginkan.

Granularitas bebas dipilih: mapping per **Area**, per **PIC** (AM melihat anak buahnya), atau per **Resto** (GM melihat cabangnya) — cukup ganti kolom mapping; measure dan visual tidak berubah.

### Struktur filter dynamic RLS (disarankan)

```text
UserSecurity (many) ──M:1──> dim Area (distinct) ──1:M──> Resto ──1:M──> Fact
role filter: UserSecurity[Email] = USERPRINCIPALNAME()
```

Tambahkan dim `Area` kecil (`DISTINCT(Resto[Area])`) agar relasi bersih M:1 — sekaligus menghindari M:M yang membingungkan engine.

### Testing: View as

Sebelum publish:

- **Modeling → View as → Roles**: pilih role → seluruh visual menampilkan data seolah Anda user itu.
- **View as → Other people**: masukkan UPN orang lain (mis. `deni@perusahaan.com`) — menguji dynamic RLS tanpa punya akunnya.
- Uji tiga skenario: user terdaftar satu area, user multi-area (baris mapping dua kali), dan user **tidak terdaftar** (harus kosong).

### RLS dengan bidirectional filter — kapan perlu

Single direction (dim → fact) cukup untuk 90% kasus. Bidirectional dibutuhkan saat filter harus **balik arah dari fact ke dim lain**, contoh nyata di model ini:

> User Area Jakarta seharusnya hanya melihat **item yang punya deviasi di areanya** — artinya filter `Resto` harus menjalar `Deviasi All` (fact) lalu naik ke `Inventory` (dim item).

Opsi, diurut dari yang paling disarankan:

1. **Satu relasi spesifik dibidirectional** (`Deviasi All → Inventory`), sisanya single — jangan bidirectional semua.
2. `CROSSFILTER` di measure tertentu — mengubah arah hanya dalam konteks measure, model tetap bersih.
3. **TREATAS** di measure (lihat [Dynamic Filtering](../06-filters-and-slicers/03-dynamic-filtering.md)) — jika model tidak boleh diubah.

### RLS di grup & memberikan akses di Service

1. Publish dataset → **Power BI Service → dataset → Security** (jika lewat App, tetap ke dataset yang sama).
2. Tambahkan user atau **security group** (AD/M365) ke role — gunakan group, bukan individu, agar onboarding cukup lewat IT.
3. User mendapat akses lewat: share report langsung, App, atau workspace Viewer. Yang penting: **RLS tetap dievaluasi** pada semua jalur konsumsi.

### RLS vs workspace roles — jangan tertukar

| Workspace role | Bisa edit model | Kena RLS? |
|---|---|---|
| Admin / Member / Contributor | Ya | **TIDAK — melihat semua data** |
| Viewer | Tidak | Ya, sesuai role RLS yang dipetakan |
| Share/App viewer tanpa edit | Tidak | Ya |

Implikasi tata kelola: jangan berikan Contributor hanya agar orang bisa buka report — mereka otomatis melihat seluruh data. Berikan akses konsumsi (Viewer/App), dan sisakan edit untuk tim pembuat.

### Measure-level workaround (bukan security sungguhan)

Kadang model terkunci dan hanya boleh menambah measure. Pola "sembunyikan via BLANK":

```dax
-- Hanya tampil nilai jika user berhak (dinilai dari mapping):
Value Secured =
VAR UserArea =
    CALCULATETABLE (
        VALUES ( UserSecurity[Area] ),
        UserSecurity[Email] = USERPRINCIPALNAME ()
    )
RETURN
    IF ( ISINSCOPE ( 'Resto'[Resto] ) && NOT ISEMPTY ( UserArea ), [Multi Value CM] )
```

`ISINSCOPE` berguna mengontrol tampilan per level hierarchy (mis. sembunyikan angka di level negara, tampil di level cabang). **Peringatan**: ini penyamaran visual, bukan keamanan — user dengan izin Build/export masih bisa menyusun measure sendiri dan mencapai barisnya. Gunakan hanya untuk UX, keamanan tetap harus RLS (baris) + OLS (objek, lihat [OLS](./02-object-level-security.md)).

### Dynamic RLS dengan TREATAS saat relasi tak bisa

Jika menambah relasi/tabel dim tidak mungkin (model read-only, atau relasi dipakai pihak lain), terapkan pembatasan di dalam measure:

```dax
Deviasi Secured =
CALCULATE (
    [Deviasi Value],
    TREATAS (
        CALCULATETABLE (
            VALUES ( UserSecurity[Area] ),
            UserSecurity[Email] = USERPRINCIPALNAME ()
        ),
        'Resto'[Area]
    )
)
```

TREATAS "menyuntikkan" area milik user sebagai filter atas `Resto[Area]` tanpa mengubah model. Sekali lagi: ini aman hanya untuk report yang dipakai konsumsi murni (tanpa Build/export underlying); untuk keamanan sejati, relasi + RLS tetap yang benar.

## Sintaks & Contoh

Langkah setup dynamic RLS untuk report cost-control:

```text
1. PQ: load UserSecurity dari Google Sheets (kolom: Area, Email)
       → TRIM + LOWER(Email) agar match UPN
2. Model: UserSecurity[Area] M:1 dimArea[Area]; dimArea 1:M Resto[Area]
          (atau varian LOOKUPVALUE tanpa relasi)
3. Modeling → Manage roles → New "Dynamic Users"
       → tabel UserSecurity → DAX: [Email] = USERPRINCIPALNAME()
4. View as → Other people → uji 3 skenario (terdaftar/multi/tidak terdaftar)
5. Publish → dataset → Security → assign security group ke role
```

Checklist perilaku yang benar:

```text
□ User terdaftar 1 area  → hanya cabang area itu
□ Multi-area mapping     → gabungan area miliknya (OR antar baris mapping)
□ Tidak terdaftar         → kosong (fail-closed), bukan semua data
□ Export & subscribe      → mengikuti RLS
□ Admin workspace         → melihat semua (sadari & dokumentasikan)
```

## Kesalahan Umum

- **Email mapping tidak match UPN** (gmail vs domain korporat, huruf besar) → user melihat kosong. Perbaikan: LOWER+TRIM di PQ, dan standarkan sumber ke M365.
- **Memberikan Contributor/Member supaya "bisa lihat semua"** — itu memang benar efeknya, tapi sekaligus memberi hak edit; gunakan Viewer + RLS untuk konsumen.
- **Filter role dipasang di tabel fact** → tidak menjalar ke dimensi lain; pindah ke sisi dim (`Resto`/`dimArea`).
- **Banyak static roles (satu per area)** — maintenance nightmare; ganti ke dynamic RLS.
- **Semua relasi dibidirectional demi RLS** — ambigu & performa turun; bidirectional hanya pada satu relasi spesifik, atau pakai CROSSFILTER/TREATAS.
- **Lupa View as sebelum publish** — bug RLS baru ketahuan saat GM komplain melihat data area lain.
- **UserSecurity tidak di-refresh** — offboarding lambat; orang yang pindah masih melihat area lama. Jadwalkan refresh bersama data lain.

## Tips Pro

- Simpan `UserSecurity` di Google Sheets yang sama dengan master lain (`Inventory` sudah dari Sheets) — satu sumber kebenaran untuk HR mapping; refresh terjadwal.
- Beri nama role yang jelas ("Dynamic – All Users") dan satu saja; kombinasi role = union data (semakin banyak role, semakin luas aksesnya) — sadari ini saat memberi multi-role.
- RLS mengecilkan data yang diproses — filter restriktif justru bisa mempercepat query untuk user tersebut; tapi jangan jadikan RLS alat optimasi.
- Untuk kebutuhan "semua area kecuali satu", buat kolom flag di `UserSecurity` (mis. `Level: HO/AM/GM`) alih-alih membuat role kebalikan.
- Audit akses secara berkala: Usage metrics + audit log Power BI; bandingkan isi `UserSecurity` dengan daftar karyawan aktif.

## Terkait

- [Object-Level Security](./02-object-level-security.md) — sembunyikan tabel/kolom, bukan hanya baris
- [Dynamic Filtering](../06-filters-and-slicers/03-dynamic-filtering.md) — TREATAS dan tabel disconnected
- [Filter Types](../06-filters-and-slicers/01-filter-types.md) — filter pane vs RLS
- [Model & DAX](../11-user-report-analysis/model-and-dax.md) — struktur dim `Resto` untuk target filter
- [Report Overview](../11-user-report-analysis/REPORT_OVERVIEW.md) — konteks report
