# 📊 Dokumentasi Belajar: Otomatisasi Report Iklan Pakai SQL & Excel

## 📌 Tentang Proyek Ini
Proyek ini adalah hasil praktik langsung (*hands-on project*) saya waktu belajar mengolah data di **ngulikdata.com**. Di sini, saya mencoba mensimulasikan studi kasus nyata yang sering dihadapi oleh tim marketing: **lambatnya rekap data iklan harian buat bahan meeting hari Senin**.

Di proyek latihan ini, saya belajar bertindak sebagai "Data Analyst" magang yang bertugas membereskan masalah tersebut. Saya belajar cara menarik data mentah dari database pakai **SQL**, menghitung metrik bisnis, lalu memindahkannya ke **Microsoft Excel** biar jadi grafik dashboard yang gampang dibaca orang awam.

* **Tools yang Saya Gunakan:** SQL (PostgreSQL), Microsoft Excel.
* **Materi SQL yang Saya Praktikkan:** Common Table Expressions (CTE), Agregasi (`SUM`), Logika Pengondisian (`CASE WHEN`), dan cara mencegah error pembagian nol (`NULLIF`).
* **Materi Excel yang Saya Praktikkan:** Custom Number Formatting (`0,00"%"`), Bikin berbagai macam grafik (Combo, Bar, Line, Column, Pie Chart), dan merapikan tampilan visual laporan.

---

## 🚀 Alur Pengerjaan (Metode STAR)

### 1. Situation (Masalah yang Dicoba Diselesaikan)
Ceritanya, setiap hari Senin pagi tim marketing harus rapat buat mengecek performa iklan minggu lalu. Masalahnya, data penting seperti biaya iklan, omset, jumlah klik, dan impresi masih berbentuk data mentah harian yang terpisah-pisah di database. Kalau harus rekap manual satu-satu setiap minggu pasti lama banget dan rawan salah hitung.

### 2. Task (Apa Saja yang Harus Saya Buat)
Di sini saya ditantang untuk bikin laporan otomatis yang bisa:
1. Menghitung metrik penting iklan secara pas: **ROAS (Return on Ad Spend)**, **CTR (Click-Through Rate)**, dan **Conversion Rate**.
2. Mengelompokkan status iklan otomatis (Iklan yang sukses banget, menguntungkan, atau balik modal doang).
3. Menampilkan semua data itu dalam bentuk grafik di Excel yang beres dibaca kurang dari 1 menit.

### 3. Action (Langkah-Langkah yang Saya Lakukan)

#### **Langkah 1: Narik dan Olah Data Pakai SQL**
Saya menyusun query SQL menggunakan teknik dua tingkat **CTE (Common Table Expressions)** biar kodenya rapi, gampang dibaca, dan jalannya cepet banget (waktu eksekusi cuma **5ms**):

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

#### **Langkah 2: Bikin Visualisasi & Dashboard di Excel**
Data matang hasil query SQL di atas saya copy ke Microsoft Excel. Di Excel, saya belajar merapikan format angka desimal menjadi format mata uang lokal (Rp) dan persentase (%) pakai trik Custom Format 0,00"%". Setelah datanya rapi, baru deh saya ubah jadi 5 bentuk grafik yang berbeda sesuai kebutuhan analisisnya.  

## 📈 4. Result: Hasil Grafik & Catatan Analisis Saya

Ini dia hasil dashboard Excel yang berhasil saya buat beserta catatan analisis sederhana versi saya::

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

### 🏷️ E. Campaign Status Classification (Kategori Performa)
* **Tujuan:** Mengelompokkan tingkat keberhasilan *campaign* berdasarkan efisiensi biaya (ROAS) untuk melihat kontribusi dan kesehatan portofolio iklan secara keseluruhan.
* **Insight:** Menggunakan grafik lingkaran (*Pie Chart*), kita bisa melihat visualisasi distribusi performa dan kontribusi persentase dari 8 *campaign* aktif perusahaan:

<img width="841" height="498" alt="Campaign Distribution by Performance Status" src="https://github.com/user-attachments/assets/c7bece8a-b8e7-449e-8968-9330aa0858f2" />


* **Analisis & Interpretasi Bisnis:**
  * **Kategori Profitable (38%):** Menjadi porsi terbesar dalam portofolio iklan saat ini (3 dari 8 *campaign*). Ini menandakan mayoritas iklan berjalan stabil dan konsisten menghasilkan keuntungan di atas biaya operasionalnya.
  * **Kategori Star (37%):** Menyumbang kontribusi yang hampir setara besar dengan kategori *Profitable* (3 dari 8 *campaign*). Kelompok ini adalah penggerak pertumbuhan bisnis utama dengan tingkat pengembalian investasi (ROAS) tertinggi.
  * **Kategori Break Even (25%):** Sebanyak 2 *campaign* berada di posisi balik modal (tidak untung, tidak rugi). Meskipun skalanya paling kecil dalam distribusi kue, kelompok inilah yang menjadi fokus utama tim manajemen untuk segera diaudit kinerjanya.
  * **Kategori Needs Review (0%):** Tidak ada satu pun *campaign* yang menyentuh zona merah, membuktikan bahwa strategi eksekusi tim pemasaran sangat aman dan terukur.
 
---

## 💡 Opini & Saran Recommendasi untuk Bisnis
Berdasarkan hasil dashboard di atas, rekomendasi strategis yang saya berikan kepada tim manajemen adalah:

1. **Scale-Up Budget:** Naikkan alokasi dana iklan untuk *campaign* bertema *Flash Sale* pada kuartal berikutnya karena terbukti menghasilkan ROAS di atas 12x.
2. **Audit Product Page (Launch Jamu Kunyit):** Iklan *Jamu Kunyit* diminati (CTR oke), namun tidak menghasilkan profit. Perlu dilakukan audit pada *user experience landing page*, skema harga, atau promo peluncurannya karena ada hambatan yang membuat orang tidak jadi membeli setelah klik.
3. **Maintain Budget:** Kampanye *Ramadan Sale* dan *Launch Serum Vitamin C* harus dipertahankan karena konsisten berada di zona nyaman (**Profitable**).
