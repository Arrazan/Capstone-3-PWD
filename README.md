# Capstone 3 — Analisis Program GrabUnlimited

# Data yang digunakan merupakan Data sintetis, bukan merupakan data asli milik Grab

Analisis data end-to-end terhadap program berlangganan **GrabUnlimited** milik Grab, berdasarkan 300.000 pesanan GrabFood periode **November 2023 – Maret 2024**.

## Background Story

GrabUnlimited adalah program berlangganan bulanan Grab: member membayar biaya untuk mendapatkan promo extra. Program ini berhasil menaikkan frekuensi transaksi, tetapi ada asumsi pemanfaatan celah oleh oknum sindikat yang menyebabkan subsidi bocor (anggaran subsidi lebih besar dari yang seharusnya). Karena Grab juga tetap memberikan promo untuk non-member, analisis ini juga mengukur perbedaan privilege yang sesungguhnya antara member dan non-member, untuk melihat apakah keunggulan eksklusif GrabUnlimited masih relevan.

## Data

Tiga file CSV mentah dibaca di awal notebook (tidak disertakan di folder ini — letakkan berdampingan dengan notebook agar bisa dijalankan):

| File | Baris | Deskripsi |
|---|---|---|
| `grab_food_orders.csv` | 300.000 | status pesanan, promo discount, alasan pembatalan, timestamp order/driver tiba/selesai |
| `grab_users.csv` | 40.000 | status keanggotaan GrabUnlimited, umur akun |
| `grab_merchants.csv` | 8.000 | jenis kuliner, kota |

Dua gambar logo (`Grab Logo - Colored - zonalogo.com.svg`, `GrabUnlimited_Final_Logo_RGB_horizontal.png`) hanya dirujuk di cell markdown pembuka untuk tampilan, dan tidak dibutuhkan untuk menjalankan analisisnya.

## Alur Kerja

1. **Import data** — membaca tiga file CSV dan membuat salinan kerja agar data asli tidak rusak.
2. **Cleaning Data**
   - Data profiling (`.info()`, `.head()`)
   - Standardization
   - Duplicate checking
   - Fixing inconsistency errors
   - Missing values
   - Outlier checking
   - Merging data yang sudah dibersihkan
3. **EDA (Exploratory Data Analysis)**
   - Verifikasi hasil cleaning & feature engineering
   - Distribusi tiap kolom: order status, promo discount, cuisine type & kota, umur akun, durasi pengantaran
   - Analisis waktu
   - Analisis bivariate
   - Perbandingan member vs. non-member
   - Deteksi ghost account
4. **Insight & Recommendation** — temuan soal privilege program, kebocoran subsidi, celah integritas data, dan pembatalan, masing-masing dipasangkan dengan rekomendasi dan estimasi dampak finansial.
5. **Simulasi penghematan** — menghitung dampak bila rekomendasi #1 dan #2 dijalankan.
6. **Exporting file** — menulis dataset bersih yang sudah diberi flag ke `grab_tableau.csv` untuk divisualisasikan di Tableau.

## Temuan Utama

- Member dan non-member tidak berbeda secara berarti pada kelima metrik yang diuji (frekuensi promo, nilai promo, jumlah pesanan, kecepatan layanan, tingkat pembatalan) — GrabUnlimited saat ini tidak memberi privilege apa pun yang terukur.
- **Rp 539.510.000 (16,69%)** dari anggaran promo 5 bulan menempel pada transaksi yang mustahil secara logis (timestamp rusak) atau berasal dari akun "hantu" (riwayat pesanan mendahului tanggal pembuatan akun) — proyeksi **Rp 1.294.824.000/tahun**.
- Database tidak punya kontrol integritas: 7 label berbeda untuk yang seharusnya cuma 2 status pesanan (membuat 25,2% pesanan sukses hilang dari laporan), kolom waktu disimpan sebagai teks, nilai promo di luar nalar, dan 9.603 pesanan tercatat selesai sebelum driver tiba di resto.
- 40,26% pesanan yang dibatalkan tidak punya alasan tercatat, sehingga keputusan kompensasi driver tidak punya dasar; dari yang beralasan, 66,4% berakar pada masalah di sisi merchant (resto tutup atau tidak ditemukan).

## Rekomendasi

1. Terapkan validasi integritas data di level database (tolak urutan waktu yang mustahil, standarkan nilai status, batasi rentang promo, wajibkan alasan pembatalan).
2. Bekukan hak promo akun ghost sampai dilakukan verifikasi ulang identitas.
3. Rancang ulang promo member agar benar-benar eksklusif, lalu ukur ulang frekuensi transaksi setelah satu bulan.
4. Perbaiki akurasi data merchant (jam operasional, titik koordinat) untuk menekan pembatalan akibat merchant.

Rekomendasi 1–2 saja sudah menutup **Rp 539.510.000 (16,69% anggaran promo)** dalam 5 bulan, atau **Rp 1.294.824.000/tahun**. Rekomendasi 3–4 menentukan arah anggaran **Rp 1,27 miliar** berikutnya dan menyasar 40% pembatalan yang selama ini tidak bisa ditindaklanjuti.

## Tech Stack

Python 3.13 · pandas · numpy · seaborn · matplotlib

## Output

`grab_tableau.csv` — 297.109 baris × 22 kolom, dataset bersih dan sudah diberi flag yang digunakan untuk membangun dashboard Tableau.


-------------------------------------------------------------------------------------------
 # Capstone 3 — GrabUnlimited Program Analysis

 # The data used is synthetic data; it is not actual data belonging to Grab.

End-to-end data analysis of Grab's **GrabUnlimited** subscription program, based on 300,000 GrabFood orders from **November 2023 to March 2024**.

## Background

GrabUnlimited is Grab's monthly subscription program: members pay a fee in exchange for extra promos. While it successfully increased transaction frequency, there's a hypothesis that a syndicate exploited loopholes in the promo mechanism, leaking subsidy budget beyond what was actually needed. Since Grab also gives promos to non-members, this analysis also measures the real privilege gap between members and non-members to see whether GrabUnlimited's exclusive benefits still hold up.

## Data

Three raw CSV files are read at the start of the notebook (not included in this folder — place them alongside the notebook to run it):

| File | Rows | Description |
|---|---|---|
| `grab_food_orders.csv` | 300,000 | order status, promo discount, cancel reason, order/driver-arrival/completion timestamps |
| `grab_users.csv` | 40,000 | GrabUnlimited membership flag, account age |
| `grab_merchants.csv` | 8,000 | cuisine type, city |

Two logo images (`Grab Logo - Colored - zonalogo.com.svg`, `GrabUnlimited_Final_Logo_RGB_horizontal.png`) are referenced in the intro markdown cell for display only and aren't required to run the analysis.

## Workflow

1. **Import data** — load the three CSVs and create working copies to protect the raw data.
2. **Cleaning**
   - Data profiling (`.info()`, `.head()`)
   - Standardization
   - Duplicate checking
   - Fixing inconsistency errors
   - Missing values
   - Outlier checking
   - Merging the cleaned tables
3. **Exploratory Data Analysis (EDA)**
   - Cleaning verification & feature engineering
   - Distribution per column: order status, promo discount, cuisine types & cities, account age, delivery duration
   - Time-based analysis
   - Bivariate analysis
   - Member vs. non-member comparison
   - Ghost-account detection
4. **Insight & Recommendation** — findings on program privilege, subsidy leakage, data-integrity gaps, and cancellations, each paired with a recommendation and estimated financial impact.
5. **Savings simulation** — quantifies the impact if recommendations #1 and #2 are implemented.
6. **Export** — writes the cleaned, flagged dataset to `grab_tableau.csv` for visualization in Tableau.

## Key findings

- Members and non-members show no meaningful difference across the five metrics tested (promo frequency, promo value, order count, delivery speed, cancellation rate) — GrabUnlimited currently offers no measurable privilege.
- **Rp 539,510,000 (16.69%)** of the 5-month promo budget is tied to transactions that are logically impossible (broken timestamps) or linked to "ghost" accounts (order history predating account creation) — projected at **Rp 1,294,824,000/year**.
- The database has no integrity controls: 7 different labels for what should be 2 order statuses (hiding 25.2% of successful orders from reports), time columns stored as text, promo values outside any sane range, and 9,603 orders marked complete before the driver even arrived.
- 40.26% of cancelled orders have no recorded reason, blocking any driver-compensation decision; of those with a reason, 66.4% trace back to merchant-side issues (restaurant closed or not found).

## Recommendations

1. Enforce data-integrity validation at the database level (reject impossible timestamps, standardize status values, bound the promo range, require a cancel reason).
2. Freeze promo eligibility for ghost accounts pending identity re-verification.
3. Redesign member promos to be genuinely exclusive, then re-measure transaction frequency after one month.
4. Improve merchant data accuracy (operating hours, coordinates) to cut merchant-caused cancellations.

Recommendations 1–2 alone close **Rp 539,510,000 (16.69% of the promo budget)** over 5 months, or **Rp 1,294,824,000/year**. Recommendations 3–4 determine the direction of the next **Rp 1.27 billion** in promo budget and address the 40% of cancellations that currently can't be acted on.

## Tech stack

Python 3.13 · pandas · numpy · seaborn · matplotlib

## Output

`grab_tableau.csv` — 297,109 rows × 22 columns, the cleaned and flagged dataset used to build the Tableau dashboard.
