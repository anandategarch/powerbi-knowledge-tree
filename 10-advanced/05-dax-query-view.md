# DAX Query View: Query DAX Langsung Tanpa Visual

> DAX Query View (DQV) memungkinkan menjalankan query `EVALUATE` langsung di Desktop/service — audit angka, eksplorasi data, dan benchmark measure tanpa membuat visual. Jejak penggunaannya (`DaxQueryView_Desktop`) bahkan tercatat di metadata tooling report Cost Control ini — pemiliknya sudah memakai fitur ini, dan materi ini menjadikannya sistematis.

## Konsep & Penjelasan

### Apa itu DQV
- Tab/pane di Power BI Desktop (**View > DAX query view**) dan di Fabric portal pada semantic model (mode **Explore**: *Visual query* dan *DAX query*).
- Menjalankan satu atau beberapa query DAX terhadap model aktif; hasil muncul di grid — seperti "SSMS untuk DAX".
- GA sejak 2024; bekerja pada semantic model apa pun yang bisa dibuka (termasuk live ke service).

### Kegunaan utama (kenapa ini penting untuk report 161 measures)
- **Audit angka**: sebelum bilang "angka deviasi benar/salah" ke rapat, verifikasi dengan query langsung — bukan dengan menatap visual.
- **Explore data tanpa visual**: cek konten tabel (baris apa saja di `Inventory`? kolom `Attribute` isinya?), termasuk menemukan trailing space pada key join.
- **Testing performa measure**: jalankan query yang sama sebelum/sesudah refactor; bandingkan durasi.
- **Debug bertahap**: definisikan versi measure di dalam query (`DEFINE MEASURE`) tanpa menyentuh model — uji rumus kandidat lalu bandingkan berdampingan dengan measure lama.
- **Validasi pasca-deploy**: setelah pipeline deploy ke PROD, jalankan query angka kunci dan bandingkan dengan sumber/TEST (lihat [Deployment Pipelines](../09-publish-and-deploy/03-deployment-pipelines.md)).

### Anatomi query
- `DEFINE` (opsional): `MEASURE`, `VAR`.
- `EVALUATE` + tabel ekspresi (`SUMMARIZECOLUMNS`, `TOPN`, `ROW`, filter).
- `ORDER BY` pada kolom yang di-project.
- Beberapa blok `EVALUATE` dalam satu editor = beberapa hasil grid (batch regression test).

### Meng-copy query dari visual (Performance Analyzer)
1. **View > Performance Analyzer > Start recording** → interaksikan visual.
2. Pada baris visual > **Copy query**.
3. Paste ke DQV → jalankan. Query hasil copy mengandung detail query engine (termasuk `ADDMISSINGITEMS`, variabel internal) — bisa langsung dieksekusi; baca sebagai "apa yang benar-benar diminta visual ke engine".
4. Di Fabric portal, mode *Visual query* (drag-drop) bisa dikonversi/lihat sebagai **DAX query** — convert visual ke DAX sebaliknya juga memudahkan belajar.

## Sintaks & Contoh

Query audit deviasi per area (persis model report ini):

```dax
EVALUATE
SUMMARIZECOLUMNS(
    'Resto'[Area],
    "Dev CM",  [Deviasi CM],
    "Dev LM",  [Deviasi LM],
    "Growth",  DIVIDE([Deviasi CM] - [Deviasi LM], ABS([Deviasi LM]))
)
ORDER BY 'Resto'[Area]
```

Debug bertahap dengan DEFINE MEASURE (versi kandidat vs versi produksi):

```dax
DEFINE
    MEASURE 'Deviasi All'[Deviasi CM v2] =
        CALCULATE(
            [Value Dev All],
            'Date'[Date] <= [Selected Cutoff Date]
        )
EVALUATE
ROW(
    "Versi lama", [Deviasi CM],
    "Versi v2",   [Deviasi CM v2],
    "Selisih",    [Deviasi CM] - [Deviasi CM v2]
)
```

Explore + temukan masalah data (trailing space di key):

```dax
EVALUATE
TOPN(20, ALL(Inventory[(Old) Nama Barang]), Inventory[(Old) Nama Barang], ASC)

-- atau panjang string key (mendeteksi spasi tersembunyi):
EVALUATE
SELECTCOLUMNS(
    TOPN(10, ALL(Inventory[(Old) Nama Barang])),
    "Nama", Inventory[(Old) Nama Barang],
    "Len",  LEN(Inventory[(Old) Nama Barang])
)
```

Top-N ranking untuk QA halaman Top 10:

```dax
EVALUATE
TOPN(10, ALL(Inventory[(Old) Nama Barang]), [Deviasi CM], DESC)
```

## Kesalahan Umum
- **Menulis ekspresi tanpa `EVALUATE`** → error sintaks; perbaikan: setiap blok hasil harus diawali `EVALUATE` (setelah `DEFINE`).
- **ORDER BY kolom yang tidak di-project** `SUMMARIZECOLUMNS` → error; perbaikan: order by kolom yang muncul di hasil.
- **Salah nama measure/kolom** (apalagi model ini punya kolom dengan trailing space seperti `Nama  new item `) → "column not found"; perbaikan: copy nama persis dari field list, atau query `TOPN` dulu untuk inspeksi.
- **Query tanpa TOPN pada tabel besar** → grid lambat/terpotong (hasil preview dibatasi); perbaikan: selalu bungkus `TOPN` saat explore.
- **Menganggap hasil DQV = query visual mentah** → query hasil Performance Analyzer berisi artefak engine; perbaikan: gunakan untuk memahami pola, tulis versi bersih untuk keperluan audit.
- **Lupa bahwa DEFINE hanya berlaku dalam query itu** → measure versi test tidak tersimpan ke model; perbaikan: memang begitu desainnya — setelah yakin, pindahkan ke model (Desktop) dan commit lewat Git.

## Tips Pro
- Simpan query audit sebagai file `queries/*.dax` di repo knowledge tree — regression test hidup yang bisa dijalankan ulang tiap rilis (dan dijalankan di PROD pasca-deploy).
- Gunakan DQV untuk *pair programming angka*: saat PIC/AM meragukan satu angka, jalankan query bersama di layar — lebih cepat daripada berdebat screenshot.
- Untuk profiling mendalam (server timings, materialization), gunakan **DAX Studio** — DQV untuk sehari-hari, DAX Studio untuk otopsi performa.
- Pattern favorit QA: satu file berisi banyak `EVALUATE` (total, per area, top item, growth ABS) — satu kali Run = satu laporan QA.
- Kombinasikan dengan [Calculation Groups](./01-calculation-groups.md): audit bahwa item calc group memberi angka identik dengan measure lama sebelum refactor besar.

## Terkait
- [Calculation Groups](./01-calculation-groups.md)
- [Deployment Pipelines](../09-publish-and-deploy/03-deployment-pipelines.md)
- [Git Integration](../09-publish-and-deploy/05-git-integration.md)
- [Analisis Model & DAX](../11-user-report-analysis/model-and-dax.md)
- [Errors Umum](../12-troubleshooting/common-errors.md)
- [FAQ](../12-troubleshooting/faq.md)
