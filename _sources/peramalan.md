# Laporan Analisis: Prediksi Kadar NO2 di Wilayah Talango Menggunakan KNN Regression

**Sumber Data:** Sentinel-5P L2 (Copernicus Data Space)  
**Periode Data:** 1 Oktober 2024 – 1 Mei 2026  
**Lokasi:** Talango, Sumenep, Jawa Timur, Indonesia  
**Metode:** K-Nearest Neighbors (KNN) Regression  

---

## 1. Pendahuluan

Notebook ini merupakan implementasi pipeline lengkap untuk mengambil, memproses, dan memprediksi data konsentrasi Nitrogen Dioksida (NO2) atmosfer di wilayah Talango, Kabupaten Sumenep, Jawa Timur. Data bersumber dari satelit **Sentinel-5P** melalui platform **Copernicus Data Space Ecosystem (CDSE)** yang diakses menggunakan library `openEO`. Model prediksi yang digunakan adalah **K-Nearest Neighbors (KNN) Regression** dengan variasi jumlah lag waktu.

---

## 2. Alur Kerja (Pipeline)

```
Instalasi Library
     ↓
Koneksi & Autentikasi ke Copernicus (openEO)
     ↓
Pengambilan Data Sentinel-5P (NO2)
     ↓
Eksekusi Batch Job & Download File .nc
     ↓
Eksplorasi & Pembacaan File NetCDF
     ↓
Interpolasi Missing Value (Spasial)
     ↓
Agregasi Temporal → Time Series Harian
     ↓
Penanganan Tanggal Hilang
     ↓
Deteksi & Penanganan Outlier (IQR)
     ↓
Pembentukan Dataset Supervised (Lag Features)
     ↓
Training & Evaluasi Model KNN
     ↓
Visualisasi Hasil Prediksi
```

---

## 3. Detail Tahapan

### 3.1 Instalasi Library

Library utama yang digunakan:

| Library | Versi | Kegunaan |
|---|---|---|
| `openeo` | 0.50.0 | Akses data satelit Copernicus |
| `netCDF4` | 1.7.4 | Membaca file format NetCDF (.nc) |
| `xarray` | 2025.1.1 | Manipulasi data multidimensi |
| `numpy` | — | Komputasi numerik |
| `pandas` | — | Manipulasi data tabular |
| `matplotlib` | — | Visualisasi data |
| `scikit-learn` | — | Model machine learning |

---

### 3.2 Koneksi ke Copernicus Data Space

```python
import openeo
connection = openeo.connect("openeo.dataspace.copernicus.eu").authenticate_oidc()
```

Autentikasi dilakukan menggunakan **OIDC device code flow**, yang merupakan metode login berbasis browser yang aman untuk akses API Copernicus.

---

### 3.3 Pengambilan Data Sentinel-5P NO2

Data diambil dari koleksi `SENTINEL_5P_L2` dengan konfigurasi berikut:

- **Band:** NO2
- **Rentang waktu:** 1 Oktober 2024 – 1 Mei 2026
- **Bounding Box (Spatial Extent):**
  - West: 112.68 | East: 113.09
  - South: -7.20 | North: -6.89

**Area of Interest (AOI)** didefinisikan sebagai polygon GeoJSON dengan 13 titik koordinat yang melingkupi wilayah Talango secara lebih presisi.

Setelah data diambil, dilakukan dua tahap agregasi:

1. **Agregasi temporal harian** — nilai rata-rata per hari untuk menghindari duplikasi data dalam satu hari.
2. **Agregasi spasial** — nilai rata-rata di dalam area poligon AOI.

**Eksekusi batch job:**
```python
job = s5post.execute_batch(title="NO2 in Talango", outputfile="NO2Talango.nc")
```
Job selesai dalam ±6 menit (status: `finished`, progress 100%).

---

### 3.4 Eksplorasi File NetCDF

File hasil unduhan (`NO2Talango.nc`) dibaca menggunakan library `netCDF4`. Struktur data yang ditemukan:

| Variabel | Keterangan |
|---|---|
| `t` | Dimensi waktu |
| `x` | Dimensi longitude (8 kolom) |
| `y` | Dimensi latitude (9 baris) |
| `crs` | Informasi sistem koordinat |
| `NO2` | Nilai konsentrasi NO2 |

**Dimensi data NO2:** `(574 timestep × 9 baris × 8 kolom)`

Data NO2 bertipe `numpy.ma.MaskedArray`, yang berarti terdapat banyak nilai yang tidak tersedia (`--`) akibat tutupan awan atau keterbatasan sensor satelit.

---

### 3.5 Interpolasi Nilai Hilang (Spasial)

Karena banyak sel grid yang bernilai `--` (masked), dilakukan interpolasi linear per piksel (per grid point) menggunakan `pandas.Series.interpolate`:

```python
for i in range(no2.shape[1]):     # loop 9 baris
    for j in range(no2.shape[2]): # loop 8 kolom
        series = pd.Series(no2[:, i, j])
        no2_filled[:, i, j] = series.interpolate(method='linear', limit_direction='both')
```

Setiap titik grid diinterpolasi secara independen sepanjang dimensi waktu.

---

### 3.6 Pembentukan Time Series Harian

Setelah interpolasi spasial, nilai NO2 diagregasi menjadi satu nilai per hari dengan menghitung rata-rata seluruh grid:

```python
new_no2.append(np.mean(no2_filled[i]))
```

Hasilnya disimpan sebagai DataFrame dengan kolom `date` dan `NO2`, lalu diekspor ke file CSV (`NO2_Talango_timeseries.csv`).

---

### 3.7 Penanganan Tanggal yang Hilang

Pengecekan kelengkapan data terhadap rentang penuh 1 Oktober 2024 – 1 Mei 2026 menemukan **4 tanggal yang hilang**:

| Tanggal Hilang |
|---|
| 2025-01-30 |
| 2025-01-31 |
| 2026-02-24 |
| 2026-05-01 |

Tanggal yang hilang diisi dengan **interpolasi berbasis waktu** (`method='time'`), lalu diverifikasi ulang — hasilnya **0 tanggal yang hilang**.

---

### 3.8 Deteksi dan Penanganan Outlier

Metode yang digunakan adalah **Interquartile Range (IQR)**:

```
Lower Bound = Q1 - 1.5 × IQR
Upper Bound = Q3 + 1.5 × IQR
```

**Hasil deteksi:** Ditemukan **1 outlier** pada tanggal **17 Oktober 2024** dengan nilai NO2 = `0.000047`.

Outlier tersebut ditandai sebagai `NaN` kemudian diisi ulang menggunakan **interpolasi linear**, sehingga data akhir bersih dari outlier (0 missing value setelah interpolasi).

---

### 3.9 Analisis Korelasi Lag

Sebelum membangun model, dilakukan analisis korelasi antara nilai NO2 hari ini `NO2(t)` dengan nilai pada hari-hari sebelumnya (`lag features`) hingga 30 hari ke belakang. Hasil menunjukkan pola yang konsisten:

| Lag | Korelasi |
|---|---|
| t-1 | **0.9063** (tertinggi) |
| t-2 | 0.8062 |
| t-3 | 0.7317 |
| t-4 | 0.6736 |
| t-7 | 0.5837 |
| t-14 | 0.4288 |
| t-30 | 0.3279 (terendah) |

Semakin jauh lag waktu, semakin rendah korelasinya — yang berarti nilai NO2 hari sebelumnya sangat berpengaruh terhadap nilai hari ini.

---

### 3.10 Pembentukan Dataset Supervised

Fungsi `create_supervised(data, n_lag)` digunakan untuk mengubah time series menjadi format supervised learning dengan fitur lag:

| Versi | Fitur Input | Jumlah Baris |
|---|---|---|
| `supervised_df` (lag=4) | NO2(t-4) s.d. NO2(t-1) | 574 baris |
| `supervised_df10` (lag=10) | NO2(t-10) s.d. NO2(t-1) | 568 baris |
| `supervised_df30` (lag=30) | NO2(t-30) s.d. NO2(t-1) | 548 baris |

---

### 3.11 Training & Evaluasi Model KNN

Model yang digunakan adalah `KNeighborsRegressor` dengan `n_neighbors=5`. Data dibagi dengan rasio **80:20** (tanpa shuffle, menjaga urutan temporal).

**Metrik Evaluasi:**

| Model | Train Size | Test Size | RMSE | R² Score | MAPE |
|---|---|---|---|---|---|
| KNN – 4 Lag | 459 | 115 | 0.083874 | **0.7971** | 13.38% |
| KNN – 10 Lag | 454 | 114 | 0.105595 | 0.6667 | 19.88% |
| KNN – 30 Lag | 438 | 110 | 0.000005 | 0.2221 | 16.06% |

> **Catatan:** RMSE yang sangat kecil pada KNN-30 (0.000005) namun R² yang rendah (0.22) mengindikasikan kemungkinan adanya kebocoran data atau efek dari data yang terlalu homogen di bagian akhir dataset (nilai NO2 yang konstan akibat interpolasi berulang).

---

## 4. Visualisasi

Notebook menghasilkan empat grafik utama:

1. **Grafik Deteksi Outlier (IQR)** — menampilkan garis batas atas/bawah dan titik outlier berwarna merah.
2. **Grafik NO2 Setelah Outlier Removal** — data time series yang sudah bersih.
3. **Grafik Prediksi KNN-4** — perbandingan nilai aktual vs prediksi dengan lag 4 hari.
4. **Grafik Prediksi KNN-10** — perbandingan nilai aktual vs prediksi dengan lag 10 hari.
5. **Grafik Prediksi KNN-30** — perbandingan nilai aktual vs prediksi dengan lag 30 hari.

---

## 5. Kesimpulan

| Aspek | Hasil |
|---|---|
| Sumber data | Sentinel-5P L2 via Copernicus openEO |
| Wilayah | Talango, Sumenep, Jawa Timur |
| Periode | Oktober 2024 – Mei 2026 (578 hari) |
| Outlier ditemukan | 1 titik (17 Oktober 2024) |
| Model terbaik | KNN dengan lag 4 hari |
| R² terbaik | 0.7971 |
| MAPE terbaik | 13.38% |

Model KNN dengan **4 lag hari** memberikan performa terbaik berdasarkan R² Score (0.80) dan MAPE (13.38%). Penambahan lag hari yang lebih panjang (10 dan 30 hari) justru menurunkan akurasi model, yang konsisten dengan hasil analisis korelasi yang menunjukkan lag pendek (t-1, t-2, t-3) memiliki korelasi jauh lebih tinggi.

---

## 6. Saran Pengembangan

- Mencoba model yang lebih kompleks seperti **LSTM** atau **Random Forest** untuk menangkap pola nonlinear.
- Menambahkan **normalisasi/scaling** yang konsisten antara data train dan test sebelum KNN.
- Melakukan **hyperparameter tuning** nilai `k` pada KNN (saat ini menggunakan k=5 secara default).
- Mempertimbangkan **cross-validation time series** (misal: TimeSeriesSplit) untuk evaluasi yang lebih robust.
- Menambahkan fitur eksogen seperti data cuaca atau musim untuk meningkatkan akurasi.

---

*Laporan ini dibuat berdasarkan analisis notebook Google Colab `Untitled0.ipynb`.*
