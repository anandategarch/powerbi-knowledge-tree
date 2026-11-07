# 🌳 Power BI Knowledge Tree

> **Pohon Pengetahuan Power BI** — knowledge base lengkap berisi DAX, Data Modeling, Power Query (M), Visual, Report Pages, Performance, Security, Deployment, hingga analisis report asli pemilik repo. Ditulis dalam **Bahasa Indonesia** dengan istilah teknis English.
>
> 🤖 **Khusus AI assistant**: baca [`AI_CONTEXT.md`](./AI_CONTEXT.md) dulu — berisi protokol menjawab & routing topik → file.

![Files](https://img.shields.io/badge/File-64-orange) ![Lines](https://img.shields.io/badge/Total%20Lines-12.000%2B-yellow) ![Language](https://img.shields.io/badge/Bahasa-Indonesia-red) ![PowerBI](https://img.shields.io/badge/Power%20BI-Knowledge-brightgreen)

## 📁 Struktur Pohon

```
powerbi-knowledge-tree/
│
├── 🤖 AI_CONTEXT.md .................... Panduan untuk AI assistant (WAJIB DIBACA AI)
├── 🗺️ KNOWLEDGE_MAP.md ................. Peta lengkap + deskripsi tiap file
│
├── 00-quick-reference/ ................. ⚡ Kartu hafalan cepat
│   ├── dax-cheatsheet.md .................. ±80 fungsi DAX + 15 snippet siap copy
│   ├── m-cheatsheet.md .................... ±50 fungsi M + pattern folder/unpivot
│   └── glossary.md ........................ 96 istilah Power BI A–Z
│
├── 01-dax/ ............................. 🧮 17 file: DAX dari nol sampai advanced
│   ├── 01..06 ............................ sintaks, context, measures, agregasi,
│   │                                     iterator, CALCULATE & filter functions
│   ├── 07..12 ............................ time intelligence, relasi (TREATAS!),
│   │                                     logika, information, teks, table functions
│   └── 13..17 ............................ VAR/RETURN, window functions, format,
│                                         resep emas (Z-score, Pareto, A/B), performa
│
├── 02-data-modeling/ ................... 🏛️ 5 file: star schema, relasi, cardinality,
│                                         date dimension, best practices
│
├── 03-power-query/ ..................... 🔧 4 file: bahasa M, transformasi harian,
│                                         parameter & function, resep siap pakai
│
├── 04-visuals/ ......................... 📊 6 file: katalog native & custom visual,
│                                         konfigurasi, conditional formatting,
│                                         small multiples, interaksi & sync slicers
│
├── 05-report-pages/ .................... 📄 6 file: layout, bookmarks, drillthrough,
│                                         tooltips, selection pane, themes (JSON)
│
├── 06-filters-and-slicers/ ............. 🎚️ 3 file: tipe filter, panduan slicer,
│                                         filter dinamis (TREATAS, what-if, dll)
│
├── 07-performance/ ..................... ⚡ 3 file: mesin VertiPaq, Performance
│                                         Analyzer & DAX Studio, checklist optimasi
│
├── 08-security/ ........................ 🔒 2 file: RLS (statis & dinamis),
│                                         object-level security
│
├── 09-publish-and-deploy/ ............. 🚀 5 file: workspace & app, gateway &
│                                         refresh, deployment pipeline, fitur
│                                         service, Git integration (PBIR)
│
├── 10-advanced/ ........................ 🎓 5 file: calculation groups, field
│   │                                     parameters, incremental refresh, visual
│   │                                     dinamis, DAX query view
│
├── 11-user-report-analysis/ ........... 📊 3 file: ANALISIS REPORT ASLI USER
│   ├── REPORT_OVERVIEW.md ................. ringkasan eksekutif (36 halaman,
│   │                                     539 visual, 161 measures)
│   ├── pages-and-visuals.md ............... detail 36 halaman + komposisi visual
│   └── model-and-dax.md ................... relasi, semua measure DAX lengkap,
│                                         Power Query inti
│
└── 12-troubleshooting/ ................ 🩺 2 file: 24 error umum + solusi, FAQ
```

## 📈 Statistik

| Kategori | File | Baris |
|---|---:|---:|
| 00 Quick Reference | 3 | 715 |
| 01 DAX | 17 | 3.187 |
| 02 Data Modeling | 5 | 646 |
| 03 Power Query | 4 | 830 |
| 04 Visuals | 6 | 825 |
| 05 Report Pages | 6 | 742 |
| 06 Filters & Slicers | 3 | 462 |
| 07 Performance | 3 | 411 |
| 08 Security | 2 | 344 |
| 09 Publish & Deploy | 5 | 575 |
| 10 Advanced | 5 | 598 |
| 11 Analisis Report User | 3 | 2.732 |
| 12 Troubleshooting | 2 | 224 |
| **Total** | **64** | **±12.300** |

## 🎯 Cara Pakai

### Untuk Manusia
1. Mulai dari [`KNOWLEDGE_MAP.md`](./KNOWLEDGE_MAP.md) untuk melihat peta lengkap.
2. Ingin cepat? Buka [00-quick-reference](./00-quick-reference/) (cheatsheet).
3. Belajar bertahap: ikuti nomor urut folder 01 → 12.
4. Butuh solusi problem nyata? Langsung ke [12-troubleshooting](./12-troubleshooting/common-errors.md).

### Untuk AI (ChatGPT/Claude/Gemini/dll)
1. Baca [`AI_CONTEXT.md`](./AI_CONTEXT.md) — berisi routing "topik → file".
2. Sebelum menjawab pertanyaan Power BI, AI membaca file terkait dari repo ini.
3. Repo inilah "memori eksternal" — tidak perlu mengingat semuanya, cukup baca ulang.

### Pertanyaan tentang Report Cost Control
Semua konteks report asli user (nama tabel, measure, halaman, pola DAX) ada di
[11-user-report-analysis/REPORT_OVERVIEW.md](./11-user-report-analysis/REPORT_OVERVIEW.md).

## 🌱 Menumbuhkan Pohon

- Tambah file baru: ikuti konvensi di [`AI_CONTEXT.md`](./AI_CONTEXT.md#konvensi-penulisan-file-baru).
- Update `KNOWLEDGE_MAP.md` setiap ada file baru.
- Commit message: `docs(<folder>): <deskripsi>`.

## 🔗 Tautan Terkait

- [DAX Guide (SQLBI)](https://dax.guide/) — referensi fungsi DAX interaktif
- [DAX Patterns (SQLBI)](https://www.daxpatterns.com/) — pola DAX klasik
- [Power BI Docs (Microsoft)](https://learn.microsoft.com/power-bi/)
- [DAX Studio](https://daxstudio.org/) — tool optimasi
- [Tabular Editor](https://tabulareditor.com/) — editing model lanjutan

---

*Dibuat otomatis oleh Z.ai • Knowledge tree ini hidup — terus ditumbuhkan 🌱*
