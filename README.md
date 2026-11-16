# SwiftBite Capstone (SB-P18)

Capstone Bootcamp **Data Analyst with AI Tools** (Future Innovators – Plan International), 2026.
Analis: Mita Amaliyatul Hayat

**Dashboard interaktif (live):** https://mitaamal.github.io/swiftbite-dashboard/
**Portofolio:** https://mitaamal.github.io/Portofolio_Mita_Amaliyatul_Hayat/

## Latar Belakang
SwiftBite adalah platform pesan-antar makanan dengan 22 restoran mitra. Net Payment turun dari Rp 15,48 juta (Januari) ke Rp 10,53 juta (Maret 2026). Proyek ini mencari penyebabnya dari data transaksi dan ulasan pelanggan.

## Pertanyaan Bisnis
1. **BQ-01:** Mengapa omzet Maret turun tajam dan zona mana yang paling terdampak?
2. **BQ-02:** Di mana keterlambatan pengantaran paling parah (per zona, mitra, dan jenis menu)?
3. **BQ-03:** Bagaimana keterlambatan dan kemasan bocor memengaruhi kepuasan pelanggan?

## Temuan Utama
- Net Payment turun 32% (Rp 15,48 jt ke Rp 10,53 jt), sementara jumlah order turun dari 126 ke 93.
- Late delivery rate 29,87% (89 dari 298 pesanan terkirim) dan naik tiap bulan: 27,43% (Jan), 28,71% (Feb), 34,52% (Mar).
- Menu Western paling sering terlambat (38,89%). Partner 14 mencapai 69,23% (9 dari 13 order), tapi sampelnya kecil, jadi perlu dibaca hati-hati.
- 23 dari 44 ulasan negatif (52,3%) justru diberi rating bintang 4-5, jadi rating saja kurang andal untuk mengukur kepuasan.
- 133 order (41,05%) tidak punya ulasan.

## Rekomendasi
1. Kalibrasi estimasi waktu masak menu Western dan tambah alokasi kurir di zona rawan terlambat.
2. Standarisasi kemasan anti-bocor dan SOP kurir saat jam sibuk.
3. Program reaktivasi pelanggan dan voucher kompensasi untuk pesanan terlambat.

## Metode
- Dua sumber data (324 pesanan dan 191 ulasan) digabung dengan LEFT JOIN pada `order_id`, hasilnya tetap 324 baris tanpa duplikasi.
- Validasi: 0 order_id duplikat, 0 ulasan tidak cocok, rekonsiliasi pembayaran selisih Rp 0, 0 kolom data pribadi (PII).
- Sentimen diklasifikasi dengan kamus kata kunci pada teks ulasan (bukan dari rating).
- Tools: Python (pandas), HTML, CSS, JavaScript.

## Isi Repository
| File | Keterangan |
|---|---|
| `Capstone_SwiftBite_SB-P18.ipynb` | Notebook: integrasi data, validasi, analisis |
| `SwiftBite_Capstone_Analysis_Report_SB-P18_pptx.pdf` | Laporan analisis (slide) |
| `01_Capstone_Analysis_Report.md` | Laporan analisis versi teks |
| `02_SwiftBite_Dashboard.html` | Dashboard interaktif (7 KPI, 6 grafik, 3 filter) |
| `technical_evidence.md` | Bukti teknis validasi data |

## Catatan
- Dataset tidak disertakan karena berasal dari materi bootcamp. Notebook membutuhkan file CSV sumber untuk dijalankan ulang.
- Data hanya mencakup 3 bulan (Q1 2026), jadi belum bisa menunjukkan pola musiman.
- Ulasan bersifat sukarela, sehingga ada kemungkinan bias pemilihan sampel.
