# Conditional Formatting Power BI

> Conditional formatting mewarnai/mengubah tampilan elemen visual berdasarkan nilai data: background color, font color, data bars, icons (termasuk custom SVG), web URL, hingga format string dinamis. Ini adalah teknik paling berdampak untuk report cost-control — angka deviasi yang merah otomatis menarik mata tanpa perlu membaca satu per satu.

## Entry Point

Ada dua jalur yang sama hasilnya:

- Klik visual → panel **Format** → **Visual** → **Cell elements** (matrix/table: per kolom; card: Callout value; data labels: fx).
- Atau klik kanan kolom di visual table/matrix → **Conditional formatting** → pilih jenis.

Ikon **fx** menandakan slot yang bisa diformat kondisional. Setiap jenis menawarkan tiga **metode**:

1. **Color scale** — gradasi antara Minimum / Center / Maximum (center bisa divergent).
2. **Rules** — kondisi if-then bertingkat (>=, <, between, not blank, dll), dievaluasi berurutan dari atas.
3. **Field value** — measure DAX yang **mengembalikan kode warna** (hex `"#FFC7CE"` atau nama `"Red"`) per sel/baris — paling fleksibel.

## Background Color & Font Color

- Background mengisi sel (matrix/table) atau area visual; font color mengubah warna teks — kombinasi keduanya penting agar kontras terjaga.
- **Rules** sangat cocok untuk deviasi: `[Deviasi CM] < 0` → merah muda; `> 0` → hijau muda; sisanya putih.
- **Field value** dipakai ketika logika lebih kompleks dari rules (misal membandingkan dua measure, melihat Z-Score, atau pengecualian area tertentu).

```dax
-- Pattern field value: warna background deviasi
BG Color Deviasi =
SWITCH (
    TRUE (),
    ISBLANK ( [Deviasi CM] ), "#FFFFFF",
    [Deviasi CM] < 0,          "#FFC7CE",   -- loss (merah muda)
    [Deviasi CM] > 0,          "#C6EFCE",   -- surplus (hijau muda)
    "#FFFFFF"
)

-- Versi berbasis Z-Score (deteksi anomali susut ala report user)
BG Color Z Score =
SWITCH (
    TRUE (),
    [Z Score] > 2,  "#FFC7CE",   -- anomali tinggi
    [Z Score] > 1,  "#FFEB9C",   -- waspada
    [Z Score] < -2, "#DDEBF7",   -- anomali rendah
    "#FFFFFF"
)
```

Cara pakai: pilih field value → arahkan ke measure warna. Font color dibuat serupa (teks gelap di atas pastel, atau teks putih di atas merah pekat `#C00000` untuk loss besar).

## Data Bars

- Bar mini di dalam sel tabel/matrix, proporsional terhadap nilai — bagus untuk kolom kontribusi (`% to Total`) atau `COM` per item.
- Opsi penting: **Bar direction**, **positive/negative bar color**, **show bar only** (angka disembunyikan).
- Deviasi yang bisa positif/negatif otomatis menampilkan bar kiri/kanan dari garis nol — sangat efektif di matrix *Detail Deviasi*.

## Icons — Icon Sets & Custom SVG

- **Icon sets** bawaan: panah (naik/turun), bendera, lingkaran, traffic light; diatur via rules atau field value.
- **Custom SVG via field value:** measure mengembalikan string `data:image/svg+xml;utf8,` + markup SVG. Cocok untuk ikon yang tidak tersedia bawaan, misal panah Growth:

```dax
Icon Growth =
VAR svg_up =
    "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16'>" &
    "<path d='M8 2 L14 12 L2 12 Z' fill='#1AAB40'/></svg>"
VAR svg_down =
    "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16'>" &
    "<path d='M8 14 L2 4 L14 4 Z' fill='#D64550'/></svg>"
RETURN
    "data:image/svg+xml;utf8," &
    IF ( [Growth %] >= 0, svg_up, svg_down )
```

Aturan main SVG:

- Gunakan **single quote** di dalam markup (Power BI mem-parse dengan double quote pembungkus).
- Ukur ikon via atribut `width`/`height` di tag svg; posisi (left/right of text) diatur di pengaturan icons.
- Bisa juga URL-encoded (`%3C` dst.) untuk karakter khusus — format utf8 seperti di atas sudah cukup untuk path sederhana.

## Web URL

- Mengubah nilai sel menjadi hyperlink: kolom harus di-set **Data category = Web URL** di model, atau gunakan measure yang mengembalikan URL lengkap.
- Contoh: link dari nama resto ke dashboard per-resto, atau link ke dokumen PO di portal perusahaan. Bisa **diformat kondisional** (field value) agar URL berubah mengikuti konteks baris.

## Alt Text Kondisional

- Slot **Alt text** di General properties juga punya fx: measure yang mengembalikan kalimat deskriptif dinamis — screen reader membaca deskripsi sesuai filter aktif (aksesibilitas yang benar-benar berguna).

## Format String Dinamis

- **Format → Format string (fx)**: measure yang mengembalikan *format string*, bukan angka — memungkinkan satu measure tampil sebagai Rp, %, atau number tergantung konteks (pattern "Multi Value" untuk toggle CM/LM/%).

```dax
-- Format berbeda mengikuti slicer tipe nilai
Format String Multi Value =
SWITCH (
    SELECTEDVALUE ( 'Tipe Nilai'[Nama], "Rp" ),
    "Rp",  "#,0",
    "%",   "0.0%",
    "0.0"
)

-- Format warna merah untuk nilai negatif (didukung visual modern)
Format Negatif Merah =
"#,0.0;[Red]-#,0.0"
```

Catatan: dukungan `[Red]` pada format string bergantung versi Desktop/visual — uji dulu; bila tidak didukung, gunakan font color rules.

## Percent vs Number Thresholds

- **Jangan bandingkan persen dengan angka**: rules `>= 5` pada kolom berformat `%` sebenarnya membandingkan nilai dasar (0.05 = 5%). Selalu cek apakah measure mengembalikan desimal (0.05) atau sudah dikali 100 (5).
- Threshold campuran, misal "merah jika deviasi < -1% **atau** < -Rp 50 juta", hanya bisa dilakukan lewat **field value** dengan `SWITCH(TRUE(), ...)`.

## Sintaks & Contoh

Pola lengkap siap pakai — kumpulkan measure berikut dalam satu folder display folder `Color`:

```dax
-- 1) Background matrix/card
BG Color Deviasi =
SWITCH (
    TRUE (),
    ISBLANK ( [Deviasi CM] ), "#FFFFFF",
    [Deviasi CM] < 0,          "#FFC7CE",
    [Deviasi CM] > 0,          "#C6EFCE",
    "#FFFFFF"
)

-- 2) Font color (kontras teks di atas background pastel)
Font Color Deviasi =
IF ( [Deviasi CM] < 0, "#9C0006", "#006100" )

-- 3) Icon panah Growth (custom SVG)
Icon Growth =
"data:image/svg+xml;utf8," &
IF (
    [Growth %] >= 0,
    "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16'>" &
    "<path d='M8 2 L14 12 L2 12 Z' fill='#1AAB40'/></svg>",
    "<svg xmlns='http://www.w3.org/2000/svg' width='16' height='16'>" &
    "<path d='M8 14 L2 4 L14 4 Z' fill='#D64550'/></svg>"
)
```

Langkah pemasangan: pilih matrix → kolom `Deviasi CM` → Cell elements → **Background → fx → Field value → `BG Color Deviasi`**; ulangi untuk Font color (`Font Color Deviasi`) dan Icons (`Icon Growth`).

## Kesalahan Umum

- **Terlalu banyak warna sekaligus** (background + font + icons + data bars pada kolom yang sama) → visual noise; pembaca tidak tahu fokus ke mana. Satu kanal sinyal per kolom.
- **Warna merah-hijau murni sebagai satu-satunya sinyal** → ~8% pria buta warna merah-hijau; dampingi dengan ikon panah/± dan gunakan palet colorblind-friendly (mis. oranye-biru, atau merah `#D64550`/hijau `#1AAB40` yang lebih aman daripada merah-hijau pekat).
- **Rules dievaluasi berurutan** — urutan kondisi salah membuat kondisi pertama "menelan" sisanya; kondisi paling spesifik taruh paling atas.
- **Lupa ISBLANK()** pada field value → sel kosong ikut terwarna, terlihat seperti ada masalah data.
- **Threshold persen vs desimal tertukar** (5 vs 0.05) → semua sel salah warna.
- **Hardcode warna di banyak tempat** → konsolidasikan palet ke theme + satu folder measure warna.

## Tips Pro

- Buat measure warna terpusat dengan prefix `Color ...` / `Icon ...` (report user menaruh semua di folder khusus) — ganti palet cukup satu tempat.
- Kombinasi klasik untuk matrix deviasi: background pastel (loss/surplus) + font color gelap + icon panah Growth — tiga sinyal redundan yang aman untuk colorblind.
- Gunakan **diverging color scale** (merah–putih–hijau) untuk measure yang berpusat di nol; non-diverging untuk yang selalu positif.
- Icons custom SVG bisa animasi ringan dan multi-bentuk — tapi jangan berlebihan; tujuannya sinyal, bukan dekorasi.
- Uji report dengan filter "worst case" (semua resto loss) — pastikan seluruh matrix tidak menjadi lautan merah tanpa kontras.

## Terkait

- [Visual Configuration (Format Pane)](./03-visual-configuration.md)
- [Katalog Visual Bawaan](./01-native-visuals-catalog.md)
- [Visual Interactions](./06-visual-interactions.md)
- [Themes & Templates](../05-report-pages/06-themes-and-templates.md)
- [Analisis DAX & measure report user](../11-user-report-analysis/model-and-dax.md)
