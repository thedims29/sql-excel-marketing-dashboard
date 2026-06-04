# 📊 Digital Marketing Campaign Automation & Performance Dashboard

## 📌 Project Overview
Proyek ini menyelesaikan masalah klasik di tim marketing: **lambatnya proses penarikan data mentah harian menjadi laporan siap pakai untuk rapat mingguan**. 

Di sini, saya berperan sebagai Data Analyst yang mengotomatisasi seluruh proses tersebut. Mulai dari mengekstrak data mentah dari database menggunakan **SQL (Advanced CTE)**, mengolah metrik bisnis krusial, hingga menyajikannya ke dalam **Executive Dashboard di Microsoft Excel** untuk menghasilkan rekomendasi taktis berbasis data bagi manajemen.

*   **Tools Used:** SQL (PostgreSQL), Microsoft Excel.
*   **SQL Core Concepts:** Common Table Expressions (CTE), Data Aggregation (`SUM`), Logical Conditional (`CASE WHEN`), Error Handling (`NULLIF`).
*   **Excel Dashboard Techniques:** Custom Number Formatting (`0,00"%"`), Multi-Type Visualizations (Combo, Bar, Line, Column charts), Executive Report Styling.

---

## 🚀 The STAR Breakdown

### 1. Situation (Situasi)
Setiap hari Senin pagi, tim marketing mengadakan *Weekly Meeting* untuk meninjau performa iklan. Namun, data performa taktis (biaya iklan, pendapatan, jumlah klik, impresi, dan konversi) masih berupa data mentah harian (*daily metrics*) yang terpisah-pisah di database. Proses rekapitulasi manual memakan waktu lama dan rentan salah hitung (*human error*), sehingga menghambat pengambilan keputusan cepat.

### 2. Task (Tugas)
Saya ditugaskan untuk membuat sebuah sistem pelaporan otomatis yang bisa:
1. Menghitung metrik performa utama secara presisi: **ROAS (Return on Ad Spend)**, **CTR (Click-Through Rate)**, dan **Conversion Rate**.
2. Mengklasifikasikan status keberhasilan tiap *campaign* (Star, Profitable, Break Even, Needs Review).
3. Menyajikan visualisasi data yang interaktif dan mudah dipahami oleh *C-Level Executives* dalam waktu singkat.

### 3. Action (Aksi)

#### **Tahap 1: Data Extraction & Metrik Engineering (SQL)**
Saya merancang query terstruktur menggunakan dua tingkat **Common Table Expressions (CTE)** untuk memastikan kode tetap bersih dan efisien (waktu eksekusi hanya **5ms**):

```sql
WITH campaign_metrics AS (
  SELECT
    c.name AS campaign_name,
    SUM(dm.spend) AS total_spend,
    SUM(dm.revenue) AS total_revenue,
    SUM(dm.impressions) AS total_impressions,
    SUM(dm.clicks) AS total_clicks,
    SUM(dm.conversions) AS total_conversions
  FROM campaigns c
  LEFT JOIN daily_metrics dm ON c.id = dm.campaign_id
  GROUP BY c.id, c.name
),

campaign_report AS (
  SELECT
    campaign_name,
    total_spend,
    total_revenue,
    ROUND(total_revenue / NULLIF(total_spend, 0), 2) AS roas,
    ROUND(total_clicks * 100.0 / NULLIF(total_impressions, 0), 2) AS ctr,
    ROUND(total_conversions * 100.0 / NULLIF(total_clicks, 0), 2) AS conversion_rate,
    CASE 
      WHEN total_revenue / NULLIF(total_spend, 0) > 5.0 THEN 'Star'
      WHEN total_revenue / NULLIF(total_spend, 0) > 2.0 THEN 'Profitable'
      WHEN total_revenue / NULLIF(total_spend, 0) > 1.0 THEN 'Break Even'
      ELSE 'Needs Review'
    END AS status
  FROM campaign_metrics
)

SELECT * FROM campaign_report ORDER BY roas DESC;

```

## 📈 4. Result: Data Visualizations & Deep-Dive Insights

Berikut adalah hasil analisis mendalam dari dashboard yang telah berhasil dibangun:

### A. Campaign Performance: Spend vs Revenue
* **Tujuan:** Melihat perbandingan langsung antara modal iklan yang keluar dengan omset yang didapat.
* **Insight:** *Flash Sale 11.11* dan *Flash Sale 9.9* menjadi mesin pencetak uang utama perusahaan. Meskipun biaya iklannya relatif kecil dibandingkan *Brand Awareness Q3*, pendapatan yang dihasilkan justru melesat sangat tinggi.

<img width="922" height="558" alt="Campaign Performance Spend vs Revenue" src="https://github.com/user-attachments/assets/ee86093c-3a02-4d38-91d1-a46cf2f0f8e5" />

---

### B. Campaign Efficiency: Return on Ad Spend (ROAS)
* **Tujuan:** Mengukur efisiensi pengembalian dana dari setiap rupiah yang diinvestasikan pada iklan.
* **Insight:** Dua *campaign* berstatus **Star** memimpin efisiensi biaya, yaitu *Flash Sale 11.11* (ROAS **13.66x**) dan *Flash Sale 9.9* (ROAS **12.16x**). Di sisi lain, *Launch Jamu Kunyit* berada di batas kritis **Break Even (ROAS 1.00x)**—artinya, modal dan omset seimbang (tidak untung, tidak rugi).

<img width="964" height="556" alt="Campaign Efficiency Return on Ad Spend (ROAS)" src="https://github.com/user-attachments/assets/52f6460b-2fe9-487a-bffa-6a5afd31bb4e" />

---

### C. Audience Interest: Click-Through Rate (CTR)
* **Tujuan:** Mengukur seberapa menarik visual, teks, dan *copywriting* iklan di mata audiens.
* **Insight:** Grafik garis menunjukkan bahwa minat tertinggi audiens berada pada momen *Flash Sale 9.9* (**6.30%**) dan *11.11* (**6.25%**). Namun, ada temuan menarik pada *Launch Jamu Kunyit*: nilai CTR-nya cukup kompetitif (**3.36%**), yang berarti orang-orang sebenarnya tertarik untuk mengklik iklan tersebut, tetapi performa akhirnya (ROAS) buruk.

<img width="917" height="498" alt="Audience Interest Click Through Rate" src="https://github.com/user-attachments/assets/f37ba75b-212d-4b91-a507-de79f72f1843" />

---

### D. Purchase Optimization: Conversion Rate
* **Tujuan:** Mengevaluasi efektivitas halaman penjualan (*landing page*) dalam mengubah klik menjadi transaksi riil.
* **Insight:** Menjawab keanehan pada produk *Jamu Kunyit* tadi, grafik konversinya berada di angka **3.63%** (cukup oke), namun karena nilai penjualannya kecil, dia tetap berada di zona *Break Even*. Sementara itu, *Brand Awareness Q3* mencatat konversi terendah (**1.38%**), hal ini wajar karena tujuan utamanya adalah mengenalkan *brand*, bukan mengejar penjualan langsung (*hard-selling*).

<img width="964" height="527" alt="Purchase Optimization Conversion Rate" src="https://github.com/user-attachments/assets/94ca0b63-f7aa-49fe-be85-efd703363d96" />

---

## 💡 Data-Driven Business Recommendations
Berdasarkan hasil dashboard di atas, rekomendasi strategis yang saya berikan kepada tim manajemen adalah:

1. **Scale-Up Budget:** Naikkan alokasi dana iklan untuk *campaign* bertema *Flash Sale* pada kuartal berikutnya karena terbukti menghasilkan ROAS di atas 12x.
2. **Audit Product Page (Launch Jamu Kunyit):** Iklan *Jamu Kunyit* diminati (CTR oke), namun tidak menghasilkan profit. Perlu dilakukan audit pada *user experience landing page*, skema harga, atau promo peluncurannya karena ada hambatan yang membuat orang tidak jadi membeli setelah klik.
3. **Maintain Budget:** Kampanye *Ramadan Sale* dan *Launch Serum Vitamin C* harus dipertahankan karena konsisten berada di zona nyaman (**Profitable**).
