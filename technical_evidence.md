# TECHNICAL EVIDENCE & VALIDATION NOTES
**SwiftBite Capstone Project: Business Analytics, Delivery SLA & Customer Experience**
* **Variant ID:** SB-P18
* **Analyst:** Mita Amaliyatul Hayat
* **Audit Execution Time:** 2026-11-01
* **Audit Scope:** Join Relasional, Grain Integrity, 4 KPI Inti, Sentimen 5 Kelas (Reclassified), Zero Division Guard, Rekonsiliasi CSV vs Dashboard, Uji Filter Dashboard, dan Verifikasi Artefak.

---

## 1. Evidence Uji Join & Grain Integrity (Tahap 01 & 02)

### A. Candidate Key & Input Snapshot
* **PM2 Base Integrated Table (`swiftbite_pm2_integrated_order_level.csv`):**
  - Total Baris: **324 baris**
  - Total Kolom: **40 kolom**
  - Evaluasi Candidate Key (`order_id`): 324 unique, 0 null, 0 duplikat -> **VALID**
* **Reviews Handoff Table (`swiftbite_reviews.csv` - SB-P18):**
  - Total Baris: **191 baris**
  - Total Kolom: **5 kolom** (`review_id`, `order_id`, `review_timestamp`, `rating`, `review_text`)
  - Evaluasi Candidate Key (`order_id`): 191 unique, 0 null, 0 duplikat -> **VALID**

### B. Relational Key Matching & Eksekusi LEFT JOIN
* `Matched Reviews`: **191 orders** (100% ulasan memiliki pasangan pesanan valid di PM2).
* `Unmatched Reviews`: **0 orders** (0 ulasan nyasar / orphaned records).
* `Eksekusi LEFT JOIN`: `df_pm2.merge(df_reviews, on="order_id", how="left")`
* `Hasil Baris`: **324 baris** (`len(df_integrated) == len(df_pm2) == 324`).
* `Bukti Fan-Out`: **TIDAK TERJADI FAN-OUT** (baris tidak membengkak).
* `Pemisahan Tiga Kondisi Ulasan`:
  1. Ada review dengan teks: **175 orders**
  2. Ada review tanpa teks (`No Text` / bintang saja): **16 orders**
  3. Tidak ada review (`No Review` / tersimpan sebagai `NaN`): **133 orders**
  - Total akumulasi: `175 + 16 + 133 = 324 orders` (**PASS**).
* `Monetary Drift Check`:
  - `SUM(successful_payment_amount)` PM2 vs Integrated: **Rp 40.958.000,00 == Rp 40.958.000,00** -> **PASS (selisih Rp 0,00)**.
  - `SUM(refunded_payment_amount)` PM2 vs Integrated: **Rp 1.109.000,00 == Rp 1.109.000,00** -> **PASS (selisih Rp 0,00)**.
* `Safe Data / Direct PII Check`:
  - Kolom terlarang yang diperiksa (`full_name`, `customer_name`, `email`, `phone_number`, `phone`, `raw_customer_id`, `controlled_salt`, `customer_id`, `address`, `salt`, `secret`): **0 kolom terdeteksi** -> **PASS**.
  - CSV final berisi 47 kolom; dashboard menampilkan footer "0 kolom terlarang dari 47 kolom".

---

## 2. Evidence Perhitungan 4 KPI Inti & Metrik Pendukung (Tahap 03)

| No | KPI / Metrik | Formula Standar Panduan Capstone | Nilai Numerik SB-P18 | Status Uji |
|:---:|:---|:---|:---:|:---:|
| **1** | **Total Orders** | `COUNT DISTINCT order_id` | **324 orders** | **`PASS` (assert)** |
| **2** | **Net Payment Value** | `SUM(successful_payment_amount) - SUM(refunded_payment_amount)`<br>*Sukses: Rp 40.958.000,00 - Refund: Rp 1.109.000,00* | **Rp 39.849.000,00** | **`PASS` (assert)** |
| **3** | **Review Coverage** | `(Order Berulasan ÷ Seluruh Order) × 100%`<br>*Numerator: 191; Denominator: 324* | **58,95%** (58.9506%) | **`PASS` (assert)** |
| **4** | **Late Delivery Rate** | `(Delivered Terlambat ÷ Delivered Valid) × 100%`<br>*Numerator: 89; Denominator: 298* | **29,87%** (29.8658%) | **`PASS` (assert)** |
| *Ops* | **AOV (307 Order Berbayar)** | `Net Payment Value ÷ Denominator Order Berbayar (307 orders)`<br>*Rp 39.849.000 ÷ 307* | **Rp 129.801,30** | Dihitung (tanpa assert) |
| *Ops* | **Average Rating** | `MEAN(rating)` ulasan valid (191 data) | **3,64 / 5.00** | Dihitung (tanpa assert) |

> **Catatan Denominator AOV:** Sebanyak 17 pesanan (324 - 307) tidak memiliki pembayaran berhasil (`successful_payment_amount` = 0). AOV memakai denominator 307 pesanan berbayar (*paid orders*), yaitu **Rp 129.801,30**.

---

## 3. Evidence Uji Sentimen & Perbaikan 12 Ulasan Unknown (Tahap 04)

### A. Perbaikan Aturan Leksikon
1. Penambahan kata ke kamus `kata_negatif`: `"sulit"`, `"susah"` (dengan regex word boundary `\b`).
2. Penambahan kata ke kamus `kata_netral`: `"seimbang"` (dengan regex word boundary `\b`).
3. Penambahan kata ke kamus `kata_negatif`: `"mahal"` (dengan regex word boundary `\b`).
4. Pelabelan sepenuhnya membaca teks ulasan, tidak diturunkan dari rating bintang.

### B. Hasil Audit Sebelum vs Sesudah pada 12 Ulasan Unknown Awal
| Order ID | Rating | Teks Ulasan | Label Lama | Label Baru | Alasan / Rule Matching |
|:---:|:---:|:---|:---:|:---:|:---|
| **O00062** | 4.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00083** | 3.0 | Kurir sulit dihubungi saat pengantaran. | Unknown | **Negative** | Cocok kata negatif "sulit" |
| **O00095** | 4.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00107** | 2.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00112** | 3.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00121** | 5.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00123** | 4.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00168** | 2.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00169** | 5.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00201** | 5.0 | Harganya terasa mahal untuk kualitas seperti ini. | Unknown | **Negative** | Cocok kata negatif "mahal" |
| **O00266** | 5.0 | Kurir sulit dihubungi saat pengantaran. | Unknown | **Negative** | Cocok kata negatif "sulit" |
| **O00282** | 4.0 | Kurir sulit dihubungi saat pengantaran. | Unknown | **Negative** | Cocok kata negatif "sulit" |

### C. Rekapitulasi Distribusi Sentimen (191 Ulasan Valid)
* **Positive:** **87 ulasan** (45,55% dari ulasan valid; 27,47% dari total 324 order).
* **Neutral:** **44 ulasan** (23,04% dari ulasan valid; 13,58% dari total 324 order).
* **Negative:** **44 ulasan** (23,04% dari ulasan valid; 12,96% dari total 324 order).
* **No Text:** **16 ulasan** (8,38% dari ulasan valid; 4,94% dari total 324 order).
* **Unknown:** **0 ulasan** (0,00%), hasil setelah kalibrasi leksikon pada tabel B.
* **No Review:** **133 pesanan** (41,05% dari 324 order) tersimpan sebagai `NaN`, terpisah dari `No Text`.

### D. Catatan Keterbatasan Klasifikasi
* Klasifikasi berbasis kamus kata kunci, sehingga belum menangkap sarkasme.
* Terdapat 12 ulasan berlabel Neutral dengan rating 1-2: teksnya tidak memuat kata keluhan yang ada di kamus.
* Ulasan campuran (pujian dan keluhan) diprioritaskan sebagai Negative.

---

## 4. Rekonsiliasi CSV vs Dashboard HTML (Tahap 06 & 07)

CSV dihitung ulang dengan `assert` Python di Tahap 03, 06, dan 07. Nilai di kolom Dashboard dicocokkan dengan tampilan `dashboard.html` tanpa filter.

| Metrik Analisis | CSV Analisis (`swiftbite_capstone_analysis_ready.csv`) | Dashboard HTML (`dashboard.html`) | Status |
| :---|:---:|:---:|:---:|
| **Total Orders (Grain)** | 324 orders | 324 orders | **MATCH** |
| **Net Payment Value** | Rp 39.849.000 | Rp 39.849.000 | **MATCH** |
| **Successful Payment** | Rp 40.958.000 | Rp 40.958.000 | **MATCH** |
| **Refunded Payment** | Rp 1.109.000 | Rp 1.109.000 | **MATCH** |
| **Review Coverage** | 58,95% (191/324) | 58,95% (191 dari 324 pesanan berulasan) | **MATCH** |
| **Delivered Valid (Denom)** | 298 orders | 298 orders | **MATCH** |
| **Delivered Terlambat (Num)** | 89 orders | 89 orders | **MATCH** |
| **Late Delivery Rate** | 29,87% (89/298) | 29,87% | **MATCH** |
| **Sentimen Positive** | 87 ulasan | 87 | **MATCH** |
| **Sentimen Neutral** | 44 ulasan | 44 | **MATCH** |
| **Sentimen Negative** | 44 ulasan | 44 | **MATCH** |
| **Sentimen No Text** | 16 ulasan | 16 | **MATCH** |
| **Sentimen Unknown** | 0 ulasan | 0 | **MATCH** |

> AOV (Rp 129.801,30) dan Average Rating (3,64) dihitung di notebook Tahap 03, tetapi tidak ditampilkan di dashboard.

### Rekonsiliasi dengan Analysis Report
Angka berikut dicocokkan manual antara laporan dan notebook. Isi kolom Status setelah laporan dibuka dan diperiksa.

| Angka di Laporan | Nilai Acuan (CSV) | Status |
|:---|:---:|:---:|
| Total Orders | 324 | **MATCH** |
| Net Payment Value | Rp 39.849.000 | **MATCH** |
| Review Coverage | 58,95% | **MATCH** |
| Late Delivery Rate | 29,87% | **MATCH** |
| Negative / dari ulasan | 42 dari 191 (22,0%) | **MATCH** |
| Negative berating 4-5 | 23 dari 44 (52,3%) | **MATCH** |
| Partner 14 late rate | 9/13 (69,2%) | **MATCH** |

### Uji Filter Dashboard (Expected vs Tampilan Dashboard)
Nilai expected dihitung dari CSV (sel "Hitung nilai expected CSV" di notebook). Kolom Dashboard diisi dari angka yang tampil di layar setelah filter dipilih.

| Test | Filter | Orders | Net Payment (Rp) | Test ID | Skenario Filter | Orders | Net Payment (Rp) | Coverage | Late Rate | Tampil di Dashboard | Status |
|:---|:---|:---:|:---:|:---:|:---:|:---|:---:|
| T-01 | Tanpa Filter (Semua Periode, Semua Mitra, Semua Kategori) | 324 | 39.849.000 | 58,95% | 29,87% | 324 / Rp 39.849.000 / 58,95% / 29,87% | MATCH |\n| T-02 | Januari 2026 (Semua Mitra, Semua Kategori) | 126 | 15.481.000 | 57,14% | 27,43% | 126 / Rp 15.481.000 / 57,14% / 27,43% | MATCH |\n| T-03 | Kategori Western (Semua Periode, Semua Mitra) | 60 | 6.702.000 | 60,00% | 38,89% | 60 / Rp 6.702.000 / 60,00% / 38,89% | MATCH |\n| T-04 | SwiftBite Partner 14 (Semua Periode, Semua Kategori) | 15 | 1.782.000 | 60,00% | 69,23% | 15 / Rp 1.782.000 / 60,00% / 69,23% | MATCH |\n| T-05 | Februari 2026 + Mitra Partner 14 | 5 | 540.000 | 40,00% | 75,00% | 5 / Rp 540.000 / 40,00% / 75,00% | MATCH |\n| T-06 | Maret 2026 (Semua Mitra, Semua Kategori) | 93 | 10.527.000 | 59,14% | 34,52% | 93 / Rp 10.527.000 / 59,14% / 34,52% | MATCH |\n| T-07 | Kategori Indonesian (Semua Periode, Semua Mitra) | 44 | 5.490.000 | 63,64% | 34,15% | 44 / Rp 5.490.000 / 63,64% / 34,15% | MATCH |\n| T-08 | Kombinasi Spesifik: Jan + Western | 21 | 2.424.000 | 52,38% | 27,78% | 21 / Rp 2.424.000 / 52,38% / 27,78% | MATCH |\n# TECHNICAL EVIDENCE & VALIDATION NOTES
**SwiftBite Capstone Project: Business Analytics, Delivery SLA & Customer Experience**
* **Variant ID:** SB-P18
* **Analyst:** Mita Amaliyatul Hayat
* **Audit Execution Time:** 2026-11-01
* **Audit Scope:** Join Relasional, Grain Integrity, 4 KPI Inti, Sentimen 5 Kelas (Reclassified), Zero Division Guard, Rekonsiliasi CSV vs Dashboard, Uji Filter Dashboard, dan Verifikasi Artefak.

---

## 1. Evidence Uji Join & Grain Integrity (Tahap 01 & 02)

### A. Candidate Key & Input Snapshot
* **PM2 Base Integrated Table (`swiftbite_pm2_integrated_order_level.csv`):**
  - Total Baris: **324 baris**
  - Total Kolom: **40 kolom**
  - Evaluasi Candidate Key (`order_id`): 324 unique, 0 null, 0 duplikat -> **VALID**
* **Reviews Handoff Table (`swiftbite_reviews.csv` - SB-P18):**
  - Total Baris: **191 baris**
  - Total Kolom: **5 kolom** (`review_id`, `order_id`, `review_timestamp`, `rating`, `review_text`)
  - Evaluasi Candidate Key (`order_id`): 191 unique, 0 null, 0 duplikat -> **VALID**

### B. Relational Key Matching & Eksekusi LEFT JOIN
* `Matched Reviews`: **191 orders** (100% ulasan memiliki pasangan pesanan valid di PM2).
* `Unmatched Reviews`: **0 orders** (0 ulasan nyasar / orphaned records).
* `Eksekusi LEFT JOIN`: `df_pm2.merge(df_reviews, on="order_id", how="left")`
* `Hasil Baris`: **324 baris** (`len(df_integrated) == len(df_pm2) == 324`).
* `Bukti Fan-Out`: **TIDAK TERJADI FAN-OUT** (baris tidak membengkak).
* `Pemisahan Tiga Kondisi Ulasan`:
  1. Ada review dengan teks: **175 orders**
  2. Ada review tanpa teks (`No Text` / bintang saja): **16 orders**
  3. Tidak ada review (`No Review` / tersimpan sebagai `NaN`): **133 orders**
  - Total akumulasi: `175 + 16 + 133 = 324 orders` (**PASS**).
* `Monetary Drift Check`:
  - `SUM(successful_payment_amount)` PM2 vs Integrated: **Rp 40.958.000,00 == Rp 40.958.000,00** -> **PASS (selisih Rp 0,00)**.
  - `SUM(refunded_payment_amount)` PM2 vs Integrated: **Rp 1.109.000,00 == Rp 1.109.000,00** -> **PASS (selisih Rp 0,00)**.
* `Safe Data / Direct PII Check`:
  - Kolom terlarang yang diperiksa (`full_name`, `customer_name`, `email`, `phone_number`, `phone`, `raw_customer_id`, `controlled_salt`, `customer_id`, `address`, `salt`, `secret`): **0 kolom terdeteksi** -> **PASS**.
  - CSV final berisi 47 kolom; dashboard menampilkan footer "0 kolom terlarang dari 47 kolom".

---

## 2. Evidence Perhitungan 4 KPI Inti & Metrik Pendukung (Tahap 03)

| No | KPI / Metrik | Formula Standar Panduan Capstone | Nilai Numerik SB-P18 | Status Uji |
|:---:|:---|:---|:---:|:---:|
| **1** | **Total Orders** | `COUNT DISTINCT order_id` | **324 orders** | **`PASS` (assert)** |
| **2** | **Net Payment Value** | `SUM(successful_payment_amount) - SUM(refunded_payment_amount)`<br>*Sukses: Rp 40.958.000,00 - Refund: Rp 1.109.000,00* | **Rp 39.849.000,00** | **`PASS` (assert)** |
| **3** | **Review Coverage** | `(Order Berulasan ÷ Seluruh Order) × 100%`<br>*Numerator: 191; Denominator: 324* | **58,95%** (58.9506%) | **`PASS` (assert)** |
| **4** | **Late Delivery Rate** | `(Delivered Terlambat ÷ Delivered Valid) × 100%`<br>*Numerator: 89; Denominator: 298* | **29,87%** (29.8658%) | **`PASS` (assert)** |
| *Ops* | **AOV (307 Order Berbayar)** | `Net Payment Value ÷ Denominator Order Berbayar (307 orders)`<br>*Rp 39.849.000 ÷ 307* | **Rp 129.801,30** | Dihitung (tanpa assert) |
| *Ops* | **Average Rating** | `MEAN(rating)` ulasan valid (191 data) | **3,64 / 5.00** | Dihitung (tanpa assert) |

> **Catatan Denominator AOV:** Sebanyak 17 pesanan (324 - 307) tidak memiliki pembayaran berhasil (`successful_payment_amount` = 0). AOV memakai denominator 307 pesanan berbayar (*paid orders*), yaitu **Rp 129.801,30**.

---

## 3. Evidence Uji Sentimen & Perbaikan 12 Ulasan Unknown (Tahap 04)

### A. Perbaikan Aturan Leksikon
1. Penambahan kata ke kamus `kata_negatif`: `"sulit"`, `"susah"` (dengan regex word boundary `\b`).
2. Penambahan kata ke kamus `kata_netral`: `"seimbang"` (dengan regex word boundary `\b`).
3. Penambahan kata ke kamus `kata_negatif`: `"mahal"` (dengan regex word boundary `\b`).
4. Pelabelan sepenuhnya membaca teks ulasan, tidak diturunkan dari rating bintang.

### B. Hasil Audit Sebelum vs Sesudah pada 12 Ulasan Unknown Awal
| Order ID | Rating | Teks Ulasan | Label Lama | Label Baru | Alasan / Rule Matching |
|:---:|:---:|:---|:---:|:---:|:---|
| **O00062** | 4.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00083** | 3.0 | Kurir sulit dihubungi saat pengantaran. | Unknown | **Negative** | Cocok kata negatif "sulit" |
| **O00095** | 4.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00107** | 2.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00112** | 3.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00121** | 5.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00123** | 4.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00168** | 2.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00169** | 5.0 | Harga dan kualitasnya terasa seimbang. | Unknown | **Neutral** | Cocok kata netral "seimbang" |
| **O00201** | 5.0 | Harganya terasa mahal untuk kualitas seperti ini. | Unknown | **Negative** | Cocok kata negatif "mahal" |
| **O00266** | 5.0 | Kurir sulit dihubungi saat pengantaran. | Unknown | **Negative** | Cocok kata negatif "sulit" |
| **O00282** | 4.0 | Kurir sulit dihubungi saat pengantaran. | Unknown | **Negative** | Cocok kata negatif "sulit" |

### C. Rekapitulasi Distribusi Sentimen (191 Ulasan Valid)
* **Positive:** **87 ulasan** (45,55% dari ulasan valid; 27,47% dari total 324 order).
* **Neutral:** **44 ulasan** (23,04% dari ulasan valid; 13,58% dari total 324 order).
* **Negative:** **44 ulasan** (23,04% dari ulasan valid; 12,96% dari total 324 order).
* **No Text:** **16 ulasan** (8,38% dari ulasan valid; 4,94% dari total 324 order).
* **Unknown:** **0 ulasan** (0,00%), hasil setelah kalibrasi leksikon pada tabel B.
* **No Review:** **133 pesanan** (41,05% dari 324 order) tersimpan sebagai `NaN`, terpisah dari `No Text`.

### D. Catatan Keterbatasan Klasifikasi
* Klasifikasi berbasis kamus kata kunci, sehingga belum menangkap sarkasme.
* Terdapat 12 ulasan berlabel Neutral dengan rating 1-2: teksnya tidak memuat kata keluhan yang ada di kamus.
* Ulasan campuran (pujian dan keluhan) diprioritaskan sebagai Negative.

---

## 4. Rekonsiliasi CSV vs Dashboard HTML (Tahap 06 & 07)

CSV dihitung ulang dengan `assert` Python di Tahap 03, 06, dan 07. Nilai di kolom Dashboard dicocokkan dengan tampilan `dashboard.html` tanpa filter.

| Metrik Analisis | CSV Analisis (`swiftbite_capstone_analysis_ready.csv`) | Dashboard HTML (`dashboard.html`) | Status |
| :---|:---:|:---:|:---:|
| **Total Orders (Grain)** | 324 orders | 324 orders | **MATCH** |
| **Net Payment Value** | Rp 39.849.000 | Rp 39.849.000 | **MATCH** |
| **Successful Payment** | Rp 40.958.000 | Rp 40.958.000 | **MATCH** |
| **Refunded Payment** | Rp 1.109.000 | Rp 1.109.000 | **MATCH** |
| **Review Coverage** | 58,95% (191/324) | 58,95% (191 dari 324 pesanan berulasan) | **MATCH** |
| **Delivered Valid (Denom)** | 298 orders | 298 orders | **MATCH** |
| **Delivered Terlambat (Num)** | 89 orders | 89 orders | **MATCH** |
| **Late Delivery Rate** | 29,87% (89/298) | 29,87% | **MATCH** |
| **Sentimen Positive** | 87 ulasan | 87 | **MATCH** |
| **Sentimen Neutral** | 44 ulasan | 44 | **MATCH** |
| **Sentimen Negative** | 44 ulasan | 44 | **MATCH** |
| **Sentimen No Text** | 16 ulasan | 16 | **MATCH** |
| **Sentimen Unknown** | 0 ulasan | 0 | **MATCH** |

> AOV (Rp 129.801,30) dan Average Rating (3,64) dihitung di notebook Tahap 03, tetapi tidak ditampilkan di dashboard.

### Rekonsiliasi dengan Analysis Report
Angka berikut dicocokkan manual antara laporan dan notebook. Isi kolom Status setelah laporan dibuka dan diperiksa.

| Angka di Laporan | Nilai Acuan (CSV) | Status |
|:---|:---:|:---:|
| Total Orders | 324 | **MATCH** |
| Net Payment Value | Rp 39.849.000 | **MATCH** |
| Review Coverage | 58,95% | **MATCH** |
| Late Delivery Rate | 29,87% | **MATCH** |
| Negative / dari ulasan | 42 dari 191 (22,0%) | **MATCH** |
| Negative berating 4-5 | 23 dari 44 (52,3%) | **MATCH** |
| Partner 14 late rate | 9/13 (69,2%) | **MATCH** |

### Uji Filter Dashboard (Expected vs Tampilan Dashboard)
Nilai expected dihitung dari CSV (sel "Hitung nilai expected CSV" di notebook). Kolom Dashboard diisi dari angka yang tampil di layar setelah filter dipilih.

| Test | Filter | Orders | Net Payment (Rp) | Review Coverage | Late Rate | Tampil di Dashboard | Status |
|:---|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| T-01 | Tanpa filter | 324 | 39.849.000 | 58,95% | 29,87% | 324 / Rp 39.849.000 / 58,95% / 29,87% | MATCH |
| T-02 | Januari | 126 | 15.481.000 | 57,14% | 27,43% | 126 / Rp 15.481.000 / 57,14% / 27,43% | MATCH |
| T-03 | Western | 60 | 6.702.000 | 60,00% | 38,89% | 60 / Rp 6.702.000 / 60,00% / 38,89% | MATCH |
| T-06 | Partner 01 + Western (tanpa data) | 0 | 0 | N/A | N/A | 0 / Rp 0 / N/A / N/A | MATCH |
| T-07 | Partner 07 | 15 | 1.562.000 | 46,67% | 15,38% | 15 / Rp 1.562.000 / 46,67% / 15,38% | MATCH |
| T-09 | Februari | 105 | 13.841.000 | 60,95% | 28,71% | 105 / Rp 13.841.000 / 60,95% / 28,71% | MATCH |
| T-10 | Maret | 93 | 10.527.000 | 59,14% | 34,52% | 93 / Rp 10.527.000 / 59,14% / 34,52% | MATCH |

> T-06 menguji kondisi tanpa data: dashboard harus menampilkan 0 atau N/A tanpa error. T-04 dan T-07 sama karena Partner 07 hanya memiliki menu Asian.

---

## 5. Ringkasan Performa Bulanan & Keterlambatan per Restoran

### A. Performa Bulanan (Jan, Feb, Mar 2026)
| Bulan | Total Orders | Net Payment Value (Rp) | Net Payment (Juta) | Delivered Valid | Late Orders | Late Delivery Rate (%) |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| **Januari 2026** | 126 | Rp 15.481.000,00 | 15.48 Jt | 113 | 31 | 27.43% |
| **Februari 2026** | 105 | Rp 13.841.000,00 | 13.84 Jt | 101 | 29 | 28.71% |
| **Maret 2026** | 93 | Rp 10.527.000,00 | 10.53 Jt | 84 | 29 | 34.52% |
| **Total Q1 2026** | **324** | **Rp 39.849.000,00** | **39.85 Jt** | **298** | **89** | **29.87%** |

### B. Performa Keterlambatan per Restoran Mitra (22 Mitra)
| Ranking | Restoran Mitra | Total Orders | Delivered Valid | Late Orders | Late Rate (%) | Status Sampel |
|:---:|:---|:---:|:---:|:---:|:---:|:---|
| 1 | SwiftBite Partner 14 | 15 | 13 | 9 | **69.23%** | Sampel Cukup (>=10) |
| 2 | SwiftBite Partner 12 | 14 | 13 | 6 | **46.15%** | Sampel Cukup (>=10) |
| 3 | SwiftBite Partner 11 | 15 | 14 | 6 | **42.86%** | Sampel Cukup (>=10) |
| 4 | SwiftBite Partner 15 | 14 | 13 | 5 | **38.46%** | Sampel Cukup (>=10) |
| 5 | SwiftBite Partner 08 | 15 | 14 | 5 | **35.71%** | Sampel Cukup (>=10), seri |
| 6 | SwiftBite Partner 13 | 15 | 14 | 5 | **35.71%** | Sampel Cukup (>=10), seri |
| 7 | SwiftBite Partner 18 | 15 | 14 | 5 | **35.71%** | Sampel Cukup (>=10), seri |
| 8 | SwiftBite Partner 02 | 15 | 13 | 4 | **30.77%** | Sampel Cukup (>=10), seri |
| 9 | SwiftBite Partner 17 | 14 | 13 | 4 | **30.77%** | Sampel Cukup (>=10), seri |
| 10 | SwiftBite Partner 19 | 15 | 13 | 4 | **30.77%** | Sampel Cukup (>=10), seri |
| 11 | SwiftBite Partner 01 | 15 | 14 | 4 | **28.57%** | Sampel Cukup (>=10), seri |
| 12 | SwiftBite Partner 03 | 15 | 14 | 4 | **28.57%** | Sampel Cukup (>=10), seri |
| 13 | SwiftBite Partner 04 | 15 | 14 | 4 | **28.57%** | Sampel Cukup (>=10), seri |
| 14 | SwiftBite Partner 21 | 15 | 14 | 4 | **28.57%** | Sampel Cukup (>=10), seri |
| 15 | SwiftBite Partner 10 | 14 | 13 | 3 | **23.08%** | Sampel Cukup (>=10), seri |
| 16 | SwiftBite Partner 22 | 14 | 13 | 3 | **23.08%** | Sampel Cukup (>=10), seri |
| 17 | SwiftBite Partner 06 | 15 | 14 | 3 | **21.43%** | Sampel Cukup (>=10), seri |
| 18 | SwiftBite Partner 09 | 15 | 14 | 3 | **21.43%** | Sampel Cukup (>=10), seri |
| 19 | SwiftBite Partner 20 | 15 | 14 | 3 | **21.43%** | Sampel Cukup (>=10), seri |
| 20 | SwiftBite Partner 05 | 14 | 13 | 2 | **15.38%** | Sampel Cukup (>=10), seri |
| 21 | SwiftBite Partner 07 | 15 | 13 | 2 | **15.38%** | Sampel Cukup (>=10), seri |
| 22 | SwiftBite Partner 16 | 15 | 14 | 1 | **7.14%** | Sampel Cukup (>=10) |
| **Total** | **22 Restoran Mitra** | **324** | **298** | **89** | **29.87%** | **Seluruhnya >= 10 Valid** |

> **Catatan:** Peringkat dengan late rate sama diberi keterangan "seri". Urutan di antara restoran seri hanya mengikuti urutan data, bukan perbedaan performa. Tiap restoran hanya punya 13-14 pesanan delivered valid, sehingga selisih satu pesanan menggeser persentase sekitar 7 poin.

---

## 6. Verifikasi Kesiapan Artefak Wajib Capstone
1. **Artefak 1:** Analysis Report (`analysis_report.md` atau file PPTX/PDF; sesuaikan dengan nama file yang dikumpulkan).
2. **Artefak 2:** `dashboard.html` (standalone offline HTML, tanpa CDN atau URL eksternal, 4 KPI, 4 chart, 3 filter, ukuran sekitar 237 KB).
3. **Artefak 3:** `swiftbite_capstone_analysis_ready.csv` (324 baris, 47 kolom, 0 kolom terlarang, sentimen tervalidasi).
4. **Artefak 4:** `technical_evidence.md` (dokumen ini).