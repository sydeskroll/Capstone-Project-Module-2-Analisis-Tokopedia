# 🛒 Analisis Strategi Pertumbuhan & Profitabilitas Tokopedia: Unit Economics, Retensi, dan SLA Logistik
Analisis ini berfokus pada evaluasi performa bisnis Tokopedia dengan menganalisis data pengguna, transaksi, dan pengiriman. Mengingat pertumbuhan *e-commerce* yang sangat cepat, analisis ini ditujukan untuk membedah tantangan perusahaan dalam menjaga profitabilitas dan retensi pengguna di tengah gempuran promo besar-besaran, serta mengevaluasi efektivitas sistem logistik.

---

## ❓ Business Question

Untuk menerjemahkan kebutuhan bisnis menjadi langkah yang terukur, analisis ini akan menjawab beberapa pertanyaan berikut:

1. Seberapa besar dampak promo dan retensi terhadap profitabilitas jangka panjang?
2. Segmen user mana yang cenderung menjadi *promo-hunter* dan berisiko *churn*?
3. Bagaimana tren order, GMV, *net revenue*, dan promo dari waktu ke waktu?
4. Metode pembayaran apa yang paling dominan dan apakah ada anomali transaksi?
5. Kota atau kelompok umur mana yang berkontribusi paling besar terhadap transaksi?
6. Seberapa besar keterlambatan pengiriman memengaruhi *repeat order* dan pengalaman pengguna?
7. Apakah terdapat hubungan antara metode pembayaran dan lokasi pengguna, serta antara lokasi dan durasi pengiriman?
8. Apakah ada anomali data sistem seperti tanggal transaksi mendahului tanggal registrasi atau transaksi yang tidak valid secara bisnis?

---

## 🎯 Objective

**Goal utama:** Menghasilkan *insight* berbasis data yang valid dan bersih untuk membantu Tokopedia dalam:

* Mengoptimalkan *budget* promo agar lebih tepat sasaran.
* Meningkatkan SLA (*Service Level Agreement*) logistik dan pengiriman.
* Memperbaiki tingkat retensi pengguna secara berkelanjutan.

---

## 🏢 Case Study (Stakeholders & Positioning)

* **Positioning / Role:** Data Analyst yang bertugas membongkar anomali data *legacy system* dan memberikan rekomendasi strategis.
* **Stakeholders Utama:**
* **Tim Marketing:** Untuk mengevaluasi ROI dari kampanye promo (WIB, subsidi ongkir, *cashback*) dan menekan angka pengguna *promo-hunter* yang mudah *churn*.
* **Tim Operasional / Logistik:** Untuk mengidentifikasi performa mitra kurir dan dampak keterlambatan pengiriman terhadap retensi pelanggan.
* **Tim Management (C-Level):** Untuk mengambil keputusan terkait profitabilitas jangka panjang berdasarkan tren GMV dan *Net Revenue*.

---

## 📖 Data Dictionary

Analisis ini menggunakan tiga dataset utama. Berikut adalah penjelasan kolom-kolomnya:

### 1. Dataset `users` (`tokopedia_users.csv`)

| Kolom | Penjelasan | Peran dalam analisis |
| --- | --- | --- |
| `user_id` | ID unik pengguna | *Key* untuk menghubungkan dengan *transactions* |
| `join_date` | Tanggal bergabung | Dasar *cohort retention* & fitur `join_month` |
| `gender` | Gender pengguna | Profil pengguna |
| `location` | Kota/lokasi pengguna | Segmentasi lokasi & uji chi-square |
| `age` | Umur dalam tahun | Segmentasi `age_group` & profil pengguna |

### 2. Dataset `transactions` (`tokopedia_transactions.csv`)

| Kolom | Penjelasan | Peran dalam analisis |
| --- | --- | --- |
| `order_id` | ID unik order | *Key* untuk menghubungkan dengan *deliveries* |
| `user_id` | ID pembeli | *Key* untuk menghubungkan dengan *users* |
| `order_date` | Tanggal order dibuat | Tren transaksi & fitur periode bulanan |
| `payment_method` | Metode pembayaran | Standardisasi kategori, agregasi GMV & uji chi-square |
| `promo_amount` | Nilai potongan harga | Menghitung total promo, *promo rate*, & *net amount* |
| `total_amount` | Nilai bruto transaksi | Menghitung GMV & AOV (*Average Order Value*) |

### 3. Dataset `deliveries` (`tokopedia_deliveries.csv`)

| Kolom | Penjelasan | Peran dalam analisis |
| --- | --- | --- |
| `order_id` | ID order dikirim | *Key* untuk menghubungkan dengan *transactions* |
| `courier_name` | Nama mitra kurir | Perbandingan performa kurir |
| `dispatch_date` | Tanggal order dikirim | Titik awal perhitungan durasi pengiriman |
| `delivery_date` | Tanggal order diterima | Titik akhir perhitungan `delivery_days` |

---

## 🧹 Data Collection & Cleaning

Data mentah melalui proses pembersihan ketat untuk memastikan kualitas analisis. Berikut adalah tahapan yang dilakukan:
1. **Data Duplikat:** Melakukan deduplikasi *key* (`user_id` dan `order_id`) untuk mencegah data ganda. Tidak ditemukan adanya duplikat pada awal dataset
2. **Data Formatting (Tanggal):** Mengonversi kolom `join_date`, `order_date`, `dispatch_date`, dan `delivery_date` ke format *datetime* standar.
3. **Validasi Nilai Transaksi:** Memfilter baris anomali dimana `total_amount` < 0 atau `promo_amount` < 0. Sebanyak **3.032 baris data invalid berhasil dikeluarkan** (Sisa data transaksi: 296.968).
4. **Standardisasi Metode Pembayaran:** Membersihkan format teks, *typo*, dan menggabungkan metode yang sama (contoh: *BCA VA* dan *Mandiri VA* diubah menjadi `bank_transfer`, variasi penulisan *gopay* disatukan).
5. **Penanganan Missing Values:**
* Kolom `gender` (4.894 data kosong) diisi dengan kategori 'Unknown'.
* Kolom `courier_name` (15.085 data kosong) tetap dipertahankan, namun dieksklusi saat melakukan agregasi performa kurir.
6. **Penanganan Outlier:** Menghapus nilai transaksi ekstrem (*outlier* pada `total_amount`) menggunakan metode aturan *Interquartile Range* (IQR) pada `Q3 + 1.5 * IQR` agar perhitungan GMV lebih representatif.

---

## 🛠️ Tools & Libraries Used

Proses ekstraksi, transformasi, analisis statistik, hingga visualisasi dilakukan menggunakan bahasa pemrograman **Python** dengan bantuan *libraries* berikut:

* **Manipulasi Data:** `pandas`, `numpy`
* **Visualisasi Data:** `matplotlib.pyplot`, `seaborn`
* **Uji Statistik:** `scipy.stats` (`mannwhitneyu`, `chi2_contingency` untuk uji hubungan antar variabel)

---

## 💡 Key Insights

*(Catatan: Bagian ini siap diisi dengan rangkuman kesimpulan setelah proses analisis / Exploratory Data Analysis di notebook Anda selesai dijalankan).*

1. **Dampak Promo:** [Ketik insight mengenai efektivitas budget promo vs retensi di sini...]
2. **Perilaku Pengguna:** [Ketik insight mengenai demografi, metode pembayaran terpopuler, dan segmen promo-hunter di sini...]
3. **Logistik & Retensi:** [Ketik insight mengenai rata-rata waktu pengiriman, performa kurir, dan bagaimana keterlambatan menurunkan repeat order di sini...]
4. **Anomali Bisnis:** [Ketik insight mengenai temuan sistem legacy/transaksi tidak lazim yang perlu diperbaiki tim engineer di sini...]

---

## 🔗 Links

* 📊 **[Link Dashboard Tableau / Looker Studio]** *(Masukkan URL di sini)*
* 🖥️ **[Link Slide Presentasi (PDF / Google Slides)]** *(Masukkan URL di sini)*
