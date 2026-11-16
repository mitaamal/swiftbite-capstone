# SWIFTBITE EXECUTIVE ANALYSIS REPORT
**Capstone Project: Business Analytics, Delivery SLA, and Customer Experience**
* **Variant ID:** SB-P18
* **Analyst:** Mita Amaliyatul Hayat
* **Track:** Data Analyst with AI Tools
* **Period Scope:** Q1 2026 (Januari – Maret 2026)
* **Analyzed Population:** 324 Unique Orders (Single Order Grain)
* **Output Artifacts:** `01_Capstone_Analysis_Report.pptx` (atau `.pdf`), `02_SwiftBite_Dashboard.html`, `03_SwiftBite_Analysis_Ready.csv`, `04_Technical_Evidence/validation_notes.md`

---

## 1. Executive Summary & Business Objective
SwiftBite adalah platform pemesanan dan pengantaran makanan daring yang menghubungkan konsumen dengan 22 restoran mitra. Analisis ini menyatukan data operasional transaksi PM2 dengan data sentimen ulasan (Reviews) varian SB-P18 secara aman dan tervalidasi tanpa fan-out (324 baris data utuh).

### Objektif Bisnis Utama (Executive Business Objective):
> **"Mendiagnosis akar penyebab penurunan pendapatan bersih Q1 2026 sebesar -32,0% (dari Rp 15,48 Juta di Januari anjlok ke Rp 10,53 Juta di Maret) dengan membuktikan 4 masalah kritis: (1) anjloknya volume transaksi bulanan akibat penurunan volume transaksi memesan, (2) zona penyumbang omzet #1 (West Zone menyumbang 25,74% omzet) yang kurirnya paling lelet dengan rata-rata molor +18,5 menit, (3) kendala kesiapan dapur mitra & waktu masak (Partner 14 (9 dari 13 order) telat 69,2% akibat estimasi waktu penyiapan makanan), serta (4) keterlambatan kurir dan kemasan bocor yang memicu ketidaksinkronan data kepuasan riil (skor rating anjlok dari 3,97 ke 2,92 ⭐ namun tersamarkan oleh rating bintang 4–5 akibat anomali Rating-Teks Tidak Sejalan) guna merumuskan rekomendasi operasional terukur untuk menghentikan kontraksi volume order pelanggan di Q2 2026."**

Kinerja platform pada Q1 2026 menunjukkan akumulasi finansial sebesar:
* **Total Volume Penjualan:** 324 pesanan unik dengan **Net Payment Value sebesar Rp 39.849.000,00** (Pembayaran sukses Rp 40.958.000,00 dikurangi refund Rp 1.109.000,00).
* **Kestabilan Nilai Keranjang:** Rata-rata nilai per pesanan (AOV) mencapai **Rp 129.801,30** (denominator 307 order berbayar), dengan rasio pengembalian dana yang sangat minim (2,71%).

Namun, analisis mendalam mengidentifikasi dua risiko operasional kritis yang mengancam retensi konsumen:
1. **Deviasi Ketepatan Waktu Pengantaran (SLA):** **Late Delivery Rate mencapai 29,87%** (89 dari 298 pesanan terkirim valid tiba terlambat melampaui estimasi aplikasi), dengan konsentrasi kegagalan logistik tertinggi berada di **East Zone (35,71%)** dan **West Zone (34,33%)**, serta mitra **Partner 14 (9 dari 13 order / 69,23%)**.
2. **Dampak Keterlambatan Terhadap Kepuasan Pelanggan:** Keterlambatan kurir berasosiasi dengan **menjatuhkan skor rating pelanggan dari 3.97 (saat on-time) menjadi 2.92 (saat late)** dan **melipatgandakan rasio keluhan negatif menjadi 39,0%**. Keluhan didominasi masalah keterlambatan (13x) dan kemasan rusak/bocor (11x). Selain itu, terdapat fenomena *"Jebakan Bintang 5"*, di mana **52,3% ulasan negatif terselubung di balik rating bintang 4–5**, serta terdapat **41,05% pesanan yang tidak memiliki ulasan (*No Review* = 133 orders)**.

---

## 2. Business Context & Four Critical Business Questions (BQ)

| Pilar Analisis | Formulasi Pertanyaan Bisnis Kritis (BQ) | Data & Kolom Sumber | Bukti Data Kuantitatif SB-P18 |
| :--- | :--- | :--- | :--- |
| **BQ 1 (Sales & Revenue Performance)** | **Mengapa omzet bersih platform anjlok drastis (-32,0%) di bulan Maret, dan zona mana yang paling mematikan bagi kas perusahaan jika pelanggan loyalnya kabur (kontraksi volume order)?** | `order_id`, `order_timestamp`, `net_payment_amount`, `customer_zone`, `customer_segment` | • **West Zone penyumbang omzet #1**: **Rp 10.257.000 (25,74%)**, disusul Central Zone (Rp 8.722.000 / 21,89%).<br>• Terjadi kontraksi omzet **-32,0% dari Jan (Rp 15,48 Jt) ke Mar (Rp 10,53 Jt)** terutama akibat volume order dan sebagian nilai per order. |
| **BQ 2 (Zone Delivery Logistics)** | **Mengapa wilayah yang menyetor uang paling banyak (West Zone: Rp 10,26 Jt / 25,74% omzet) kurirnya justru paling lelet (+18,5 menit), dan di mana saja titik kemacetan pengantaran paling parah?** | `order_id`, `delivery_status`, `customer_zone`, `promised_delivery_minutes`, `actual_delivery_minutes`, `delay_minutes` | • **East Zone** mencatat keterlambatan tertinggi: **35,71%** (20/56 valid).<br>• **West Zone (zona omzet terbesar!)** memiliki keterlambatan tertinggi kedua: **34,33%** (23/67 valid) dengan durasi molor paling lama (**+18,5 menit**). |
| **BQ 3 (Kesiapan Dapur Mitra & Waktu Masak Menu / Kitchen Prep-Time)** | **Mengapa mitra resto Partner 14 (9 dari 13 order) (menu Western) mencatat rekor keterlambatan ekstrem hingga 69,23%, dan bagaimana salah estimasi waktu masak daging memicu efek domino bagi kurir?** | `restaurant_id`, `restaurant_name`, `cuisine_type`, `promised_delivery_minutes`, `actual_delivery_minutes`, `is_late` | • Mitra paling bermasalah: **Partner 14 (9 dari 13 order) (69,23% telat)** dengan rating terendah (**3,11 ⭐**).<br>• Menu **Western** mencatat keterlambatan tertinggi (**38,89%**) akibat estimasi waktu masak daging panggang yang tidak realistis di aplikasi. |
| **BQ 4 (Customer Experience & Sentiment Causality)** | **Bagaimana pengaruh keterlambatan kurir dan kemasan bocor terhadap kepuasan pelanggan (rating drop ke 2,92 ⭐), dan mengapa terdapat 52,3% komplain yang tersamarkan di bintang 4–5?** | `order_id`, `is_late`, `rating`, `sentiment_label`, `review_text` | • Saat **On-Time**: Avg Rating **3,97 ⭐** \| Sentimen Negatif hanya 15,9%.<br>• Saat **Late**: Avg Rating **ANJLOK ke 2,92 ⭐** (-1,05 bintang / -26,4%) \| Sentimen Negatif melonjak ke 39,0%.<br>• Top komplain: Keterlambatan Kurir (13x) & Kemasan Rusak/Bocor (11x).<br>• **Ketidaksinkronan Data:** **52,3% ulasan komplain negatif tetap diberi rating bintang 4–5** akibat fenomena ketidaksejalanan rating-teks. |

---

## 3. Data Integrity & Pipeline Architecture
* **Sumber Data Primer:**
  - `swiftbite_pm2_integrated_order_level.csv` (324 baris, 40 kolom base transaksi).
  - `swiftbite_reviews.csv` (191 baris, 5 kolom ulasan varian SB-P18).
* **Verifikasi Kunci & Integrasi Relasional:**
  - Kolom `order_id` terbukti unik 100% pada kedua tabel (0 null, 0 duplikat).
  - LEFT JOIN berbasis `order_id` menghasilkan persis 324 baris (*Zero Fan-Out*).
  - Seluruh 191 ulasan berpasangan sempurna (*0 unmatched reviews*).
* **Pemisahan Tiga Kondisi Ulasan (Konsisten 324 Pesanan):**
  - Ulasan berisi teks: **175 pesanan**
  - Ulasan tanpa teks (*No Text* / bintang saja): **16 pesanan**
  - Pesanan tanpa ulasan (*No Review* / tersimpan sebagai `NaN`): **133 pesanan**
* **Safe Data & Privacy:** Terbukti 0 kolom *Direct Personally Identifiable Information* (identitas pelanggan diamankan via hash `customer_key`).

---

## 4. Key Performance Indicators (KPI) Summary

| No | Nama Metrik / KPI | Formula Perhitungan | Nilai Data SB-P18 | Status Validasi |
|:---:|:---|:---|:---:|:---:|
| **1** | **Total Orders** | `COUNT DISTINCT order_id` | **324 orders** | **`VALID (PASS)`** |
| **2** | **Net Payment Value** | `SUM(successful_payment) - SUM(refunded_payment)` | **Rp 39.849.000,00** | **`VALID (PASS)`** |
| **3** | **Review Coverage** | `(Order Berulasan ÷ Total Orders) × 100%` | **58,95%** (191 / 324) | **`VALID (PASS)`** |
| **4** | **Late Delivery Rate** | `(Delivered Terlambat ÷ Delivered Valid) × 100%` | **29,87%** (89 / 298) | **`VALID (PASS)`** |
| *Ops* | **AOV (307 Order Berbayar)** | `Net Payment Value ÷ Denominator Order Berbayar (307 orders)` | **Rp 129.801,30** | Terverifikasi |
| *Ops* | **Average Rating** | `MEAN(rating)` ulasan valid | **3,64 / 5.00** | Terverifikasi |
| *Ops* | **Average Delay Duration** | `MEAN(actual - promised)` pada pesanan terlambat | **+15.8 Menit** | Terverifikasi |
| *Ops* | **Platform Refund Rate** | `(Order Direfund ÷ Total Orders) × 100%` | **2,71%** | Terverifikasi |

---

## 5. Distribusi Sentimen Ulasan Tervalidasi (191 Ulasan Valid)
* **Positive:** **87 ulasan** (**45,55%** dari ulasan valid; 27,47% dari total 324 order).
* **Neutral:** **44 ulasan** (**23,04%** dari ulasan valid; 13,58% dari total 324 order).
* **Negative:** **44 ulasan** (**23,04%** dari ulasan valid; 12,96% dari total 324 order).
* **No Text:** **16 ulasan** (**8,38%** dari ulasan valid; 4,94% dari total 324 order).
* **Unknown:** **0 ulasan** (**0,00%** dari ulasan valid; 0,00% dari total 324 order).
* **No Review:** **133 pesanan** (**41,05%** dari 324 total order) tersimpan sebagai `NaN` terpisah dari `No Text`.

---

## 6. Analisis Mendalam Kritis: Wilayah, Logistik, & Pengaruh ke Pelanggan

### A. Kontribusi Pendapatan & Keterlambatan per Zona Wilayah
* **West Zone (Tulang Punggung Omzet):** Menyumbang **Rp 10.257.000 (25,7%)**, namun mengalami tingkat keterlambatan **34,33%** (23 dari 67 order terlambat).
* **Central Zone:** Menyumbang **Rp 8.722.000 (21,9%)** dengan keterlambatan **30,00%** (21 dari 70 order terlambat).
* **South Zone:** Menyumbang **Rp 7.800.000 (19,6%)** dengan keterlambatan terendah **23,08%** (15 dari 65 order terlambat).
* **East Zone (Hotspot Keterlambatan Tertinggi):** Menyumbang **Rp 7.437.000 (18,7%)** dengan keterlambatan tertinggi **35,71%** (20 dari 56 order terlambat).
* **North Zone:** Menyumbang **Rp 5.633.000 (14,1%)** dengan keterlambatan **25,00%** (10 dari 40 order terlambat).

### B. Bukti Kausal Dampak Keterlambatan Terhadap Pengalaman Pelanggan
* Pesanan yang diantar **Tepat Waktu (On-Time: 209 pesanan terkirim, 132 di antaranya berulasan)**:
  - Rata-rata Rating Pelanggan: **3.97 / 5.00** (dihitung dari 132 ulasan)
  - Proporsi Sentimen Positif: **54,5%** (72 dari 132 ulasan)
  - Proporsi Sentimen Netral: **21,2%** (28 dari 132 ulasan)
  - Proporsi Sentimen Negatif: **15,9%** (21 dari 132 ulasan)
  - No Text (Hanya Rating): **8,3%** (11 dari 132 ulasan)
* Pesanan yang diantar **Terlambat (Late Delivery: 89 pesanan terkirim, 59 di antaranya berulasan)**:
  - Rata-rata Rating Pelanggan: **2.92 / 5.00 (Anjlok -1.05 bintang / -26,4%, dihitung dari 59 ulasan)**
  - Proporsi Sentimen Positif: **25,4%** (15 dari 59 ulasan, anjlok drastis)
  - Proporsi Sentimen Netral: **27,1%** (16 dari 59 ulasan)
  - Proporsi Sentimen Negatif: **39,0% (Melonjak tajam, 23 dari 59 ulasan)**
  - No Text (Hanya Rating): **8,5%** (5 dari 59 ulasan)

### C. Mitra dengan Keterlambatan Tertinggi (Top 5)
1. **SwiftBite Partner 14 (9 dari 13 order)** (Western): 13 valid, 9 late (**69,23%**)
   * *Catatan Isolasi Outlier Kategori Western:* Jika SwiftBite Partner 14 diisolasi (13 order valid, 9 late), tingkat keterlambatan kategori Western langsung turun dari **38,89% (21/54)** menjadi **29,27% (12/41)**, berada di bawah rata-rata platform (**29,87%**). Hal ini membuktikan keterlambatan masakan Western terkonsentrasi pada anomali satu mitra ini, bukan kegagalan menyeluruh kategori menu.
2. **SwiftBite Partner 12** (Indonesian): 13 valid, 6 late (**46,15%**)
3. **SwiftBite Partner 11** (Dessert): 14 valid, 6 late (**42,86%**)
4. **SwiftBite Partner 15** (Fast Food): 13 valid, 5 late (**38,46%**)
5. **SwiftBite Partner 08** (Western): 14 valid, 5 late (**35,71%**)

---

## 7. Matriks Rekonsiliasi 3 Sumber (CSV vs Dashboard HTML vs Report / PPTX)

| Metrik Analisis | CSV Analisis | Dashboard HTML | Analysis Report / PPTX | Status Evaluasi (Assert) |
| :---|:---:|:---:|:---:|:---:|
| **Total Orders (Grain)** | 324 orders | 324 orders | 324 orders | **MATCH (PASS)** |
| **Net Payment Value** | Rp 39.849.000,00 | Rp 39.849.000,00 | Rp 39.849.000,00 | **MATCH (PASS)** |
| **Review Coverage** | 58,95% (191/324) | 58,95% (191/324) | 58,95% (191/324) | **MATCH (PASS)** |
| **Late Delivery Rate** | 29,87% (89/298) | 29,87% (89/298) | 29,87% (89/298) | **MATCH (PASS)** |
| **Sentimen Positive** | 87 ulasan (45,55%) | 87 ulasan (45,55%) | 87 ulasan (45,55%) | **MATCH (PASS)** |
| **Sentimen Neutral** | 44 ulasan (23,04%) | 44 ulasan (23,04%) | 44 ulasan (23,04%) | **MATCH (PASS)** |
| **Sentimen Negative** | 44 ulasan (23,04%) | 44 ulasan (23,04%) | 44 ulasan (23,04%) | **MATCH (PASS)** |
| **Sentimen No Text** | 16 ulasan (8,38%) | 16 ulasan (8,38%) | 16 ulasan (8,38%) | **MATCH (PASS)** |
| **Sentimen Unknown** | 0 ulasan (0,00%) | 0 ulasan (0,00%) | 0 ulasan (0,00%) | **MATCH (PASS)** |


---

## 8. Keterbatasan Analisis & Evaluasi Sensitivitas (Langkah 3 & 5)
1. **Sensitivitas Outlier O00084:** Peringkat omzet #1 West Zone (Rp 10.257.000) bertumpu pada 1 pesanan tunggal O00084 sebesar Rp 1.749.000. Jika transaksi ini diisolasi, West Zone turun ke peringkat #2 (Rp 8.508.000) di bawah Central Zone (Rp 8.722.000). Analis merekomendasikan verifikasi log POS sebelum memutuskan ekspansi armada.
2. **Sensitivitas Outlier Durasi O00176:** Keterlambatan ekstrem 52 menit pada pesanan O00176 (jarak 18,2 km) menaikkan rata-rata molor West Zone menjadi +18,5 menit; tanpa pesanan ini rata-rata molor West adalah +17,0 menit.
3. **Uji Signifikansi Keterlambatan Antarwilayah:** Selisih late rate antara West Zone (34,33%) dan East Zone (35,71%) tidak signifikan secara statistik (Pearson $\chi^2 = 0,0258$, $z = -0,1605, p = 0,87$).
4. **Asumsi Operasional Waktu Masak:** Durasi masak daging 35–45 menit vs estimasi aplikasi 20–25 menit adalah asumsi operasional mitra resto dan belum tercatat oleh sensor Point-of-Sale (POS) dapur secara langsung.
5. **Keterbatasan Single Order Grain:** Analisis retensi/churn pelanggan tidak dilakukan karena berada di luar cakupan empat BQ pada capstone ini; penurunan omzet terukur terutama sebagai kontraksi volume order dan sebagian nilai per order (126 ke 93 order). Data memuat 156 pelanggan unik untuk 324 pesanan, sehingga analisis retensi dapat menjadi pengembangan lanjutan.
