# Themes & Templates (.json Theme vs .pbit Template)

> Theme mengunci identitas visual report (palet warna, font, format default semua visual) dalam satu file JSON yang bisa diimpor ke report mana pun — contoh nyata: base theme `CY26SU02` pada pbix user. Dokumen ini membahas struktur JSON theme lengkap, import/export, beda theme dengan template `.pbit`, palet cost-control (merah loss / hijau surplus), hingga aksesibilitas & tools editor.

## Konsep

- **Report theme** = file JSON yang menimpa default format visual: warna data, font, judul, background, border, hingga pengaturan spesifik per tipe visual.
- **Built-in themes** tersedia dari View → Themes dropdown; tema custom diimpor dari file `.json`.
- File `.pbix` menyimpan referensi base theme (identifikasi internal seperti `CY26SU02` pada report user) — custom JSON yang Anda impor di-layer di atasnya.
- Satu theme dipakai satu report; mengganti theme mengubah semua visual yang belum diformat manual (format manual menang atas theme — konsistensi penuh hanya jika tim disiplin tidak hardcode format).

## Struktur JSON Theme

Bagian utama file theme:

- **`name`** — nama tema.
- **`dataColors`** — array warna data series (warna pertama = series pertama, dst).
- **`good` / `neutral` / `bad`** — warna semantik untuk KPI (naik/turun/netral) dan conditional formatting default.
- **`maximum` / `center` / `minimum`** — warna default color scale (diverging).
- **`textClasses`** — kelas tipografi global: `callout` (angka card), `title`, `header`, `label`, dsb.
- **`visualStyles`** — default format per visual: `"*"` (semua visual) atau nama visual spesifik (`"page"`, `"card"`, `"slicer"`, …), lalu nama section (`"title"`, `"background"`, `"border"` …).

## Sintaks & Contoh — JSON Theme Lengkap (Power BI Yellow)

Tema siap pakai bertema kuning Power BI `#F2C811` dengan semantik cost-control (hijau surplus, merah loss):

```json
{
  "name": "Cost Control — Power BI Yellow",
  "dataColors": [
    "#F2C811", "#12239E", "#E66C37", "#118DFF",
    "#6B007B", "#E044A7", "#0C7C7C", "#D9B300"
  ],
  "good": "#1AAB40",
  "neutral": "#F2C811",
  "bad": "#D64550",
  "maximum": "#1AAB40",
  "center": "#FFFFFF",
  "minimum": "#D64550",
  "textClasses": {
    "callout": { "fontFace": "Segoe UI Semibold", "fontSize": 30, "color": "#252423" },
    "title":   { "fontFace": "Segoe UI Semibold", "fontSize": 12, "color": "#252423" },
    "header":  { "fontFace": "Segoe UI Semibold", "fontSize": 10, "color": "#252423" },
    "label":   { "fontFace": "Segoe UI", "fontSize": 10, "color": "#252423" }
  },
  "visualStyles": {
    "*": {
      "*": {
        "background": [
          { "show": true, "transparency": 0,
            "color": { "solid": { "color": "#FFFFFF" } } }
        ],
        "border": [
          { "show": true, "radius": 6,
            "color": { "solid": { "color": "#E6E6E6" } } }
        ],
        "visualHeader": [ { "show": false } ],
        "title": [
          { "show": true, "fontFace": "Segoe UI Semibold", "fontSize": 12,
            "fontColor": { "solid": { "color": "#252423" } },
            "alignment": "left",
            "background": { "solid": { "color": "#FFFFFF" } } }
        ],
        "dropShadow": [ { "show": false } ]
      }
    },
    "page": {
      "*": {
        "background": [
          { "color": { "solid": { "color": "#F7F7F5" } }, "transparency": 0 }
        ],
        "outspace": [
          { "color": { "solid": { "color": "#F7F7F5" } } }
        ]
      }
    }
  }
}
```

Simpan sebagai `theme-cost-control.json` → import (lihat bawah) → seluruh report berubah konsisten.

## Import, Export & Built-in

- **Import:** View → Themes → panah dropdown → **Browse for themes** → pilih `.json`.
- **Export current:** dropdown yang sama → **Save current theme** — cara terbaik mempelajari struktur: ubah format visual di Desktop, export, baca JSON yang dihasilkan, salin section ke theme master.
- **Built-in themes:** mulai dari sini untuk quick win, lalu export → modifikasi → jadikan corporate theme.
- Mengganti theme **menimpa** tema sebelumnya (bukan stack); format manual per visual tetap menang — sebelum apply theme besar, audit visual yang sudah di-hardcode.

## Theme .json vs Template .pbit vs File .pbix

| Aspek | Theme (`.json`) | Template (`.pbit`) | Report (`.pbix`) |
|---|---|---|---|
| Isi | warna, font, format default | struktur report: halaman, visual, model, query, theme — **tanpa data** | lengkap + data |
| Dipakai untuk | identitas visual lintas report | standarisasi report baru tim | file kerja harian |
| Saat dibuka | — (di-import ke pbix) | minta parameter/koneksi lalu build data | langsung jadi |
| Ukuran | beberapa KB | kecil–sedang | bisa ratusan MB |

- Pola korporat yang sehat: **satu theme JSON** (brand) + **satu/dua template pbit** (struktur report standar cost-control: header, slicer panel, nav, halaman drillthrough siap isi) — report baru dimulai dari template, bukan dari nol atau copy-paste pbix lama.
- Template menyimpan parameter (server, path) sehingga tiap cabang/divisi tinggal menunjuk sumber datanya.

## Custom Palettes & Font

- **Jangan lebih dari ~7 warna data** — selebihnya berulang; kategori banyak > pakai small multiples/treemap, bukan warna baru.
- Definisikan **warna semantik** di satu tempat: `good` hijau `#1AAB40` (surplus), `bad` merah `#D64550` (loss), `neutral` kuning `#F2C811` (perhatian) — dipakai measure conditional formatting & ikon agar konsisten dengan theme.
- **Font:** tetap pada font yang tersedia di semua perangkat (Segoe UI, DIN, Tahoma, Arial). Custom font hanya dirender jika terpasang di mesin pemirsa — di Service pemirsa lain akan melihat fallback, merusak layout.
- `dataColors` pertama otomatis jadi warna series/bar pertama — urutan array = urutan series default.

## High Contrast & Aksesibilitas

- Kontras teks terhadap background minimal rasio **4.5:1** (WCAG AA) — cek dengan alat contrast checker; kuning `#F2C811` dengan teks hitam `#252423` lolos, dengan teks putih tidak.
- Uji report lewat **View → High contrast** dan pertimbangkan palet colorblind-safe (oranye–biru) bila audiens luas.
- Jangan andalkan warna sebagai satu-satunya pembawa makna — dampingi ikon panah/± (lihat [Conditional Formatting](../04-visuals/04-conditional-formatting.md)).

## Mengelola Theme di Organisasi

- Simpan theme JSON di repo bersama (git) — versioning, review, dan rollback jelas; ini praktik yang tepat untuk repo knowledge-tree ini.
- Terbitkan tema brand resmi (satu file, satu owner); perubahan lewat pull request, bukan edit-an-di-desktop-lalu-kirim-via-chat.
- Dokumentasikan pemetaan warna: "merah = loss, hijau = surplus, kuning = perhatian/Z-Score 1–2" agar semua developer dan measure warna konsisten.
- Untuk membagi tanpa repo: theme bisa juga dibagikan sebagai file atau lewat organizational custom visuals store tidak berlaku — theme tetap file JSON biasa.

## Tools Editor

- **Power BI theme generator/ builder berbasis web** (mis. theme generator dari komunitas powerbi.tips) — klik-klik warna, export JSON.
- Cara paling andal tetap: **export current theme** dari Desktop lalu edit — struktur dijamin sesuai versi engine.
- Validasi JSON (formatter/linter) sebelum import; koma hilang = gagal import dengan pesan tidak selalu jelas.

## Kesalahan Umum

- **Mengira theme dan template sama** — theme hanya styling; template membawa halaman/model/query.
- **Hardcode format per visual setelah import theme** → theme berikutnya tidak berefek penuh; format manual menang atas theme.
- **> 10 warna di `dataColors`** "supaya semua kategori beda" → tidak terbaca; kelola kategori di level visual.
- **Custom font yang tidak dimiliki pemirsa** → fallback font merusak tata letak di Service.
- **Semantik warna tidak konsisten** (merah untuk loss di satu halaman, merah untuk highlight di halaman lain) → pengguna salah membaca sinyal.
- **Mengedit JSON tanpa export-restore** → typo format section membuat sebagian visual tidak berubah tanpa error yang jelas.

## Tips Pro

- Mulai dari tema user: export base theme pbix (CY26SU02 lineage), rapikan jadi corporate JSON — perubahan revolusioner dari kondisi nyata lebih mudah di-review.
- Simpan measure warna conditional formatting (`BG Color Deviasi`, `Icon Growth`) di satu folder dan samakan literal hex dengan `good`/`bad` theme.
- Taro komentar di repo (bukan di JSON, JSON tidak mendukung komentar): dokumentasikan arti tiap warna di file terpisah.
- Tiap ganti theme besar: screenshot 3–4 halaman kunci sebelum/sesudah untuk approval visual manajemen.
- Kombinasikan theme + halaman template (pbit) + sync slicers = report baru yang konsisten dalam hitungan jam, bukan hari.

## Terkait

- [Page Layout](./01-page-layout.md)
- [Selection & Z-Order](./05-selection-and-z-order.md)
- [Conditional Formatting](../04-visuals/04-conditional-formatting.md)
- [Visual Configuration (Format Pane)](../04-visuals/03-visual-configuration.md)
- [Katalog Visual Bawaan](../04-visuals/01-native-visuals-catalog.md)
- [Overview report user (base theme CY26SU02)](../11-user-report-analysis/REPORT_OVERVIEW.md)
