# Normalisasi Data (Preprocessing)

## Pengertian Normalisasi Data
Normalisasi data merupakan proses transformasi nilai data ke dalam skala tertentu agar memiliki rentang yang seragam. Proses ini bertujuan untuk menghindari dominasi atribut tertentu dalam proses analisis serta meningkatkan kinerja algoritma data mining.

---

## Data Contoh

| No | IPK | PO | JML |
|----|-----|--------|-----|
| 1 | 2 | 2000000 | 2 |
| 2 | 3 | 3000000 | 3 |
| 3 | 4 | 2000000 | 2 |
| 4 | 2 | 2000000 | 3 |
| 5 | 3 | 3000000 | 2 |
| 6 | 4 | 4000000 | 3 |
| 7 | 2 | 3000000 | - |

---

## 1. Min-Max Normalization

### Rumus
X' = (X - Xmin) / (Xmax - Xmin)

### Langkah Penyelesaian
1. Menentukan nilai minimum dan maksimum:
   - IPK minimum = 2
   - IPK maksimum = 4

2. Melakukan perhitungan normalisasi:
   - IPK = 2 → (2 - 2) / (4 - 2) = 0
   - IPK = 3 → (3 - 2) / (4 - 2) = 0.5
   - IPK = 4 → (4 - 2) / (4 - 2) = 1

### Hasil Normalisasi IPK

| IPK | Hasil |
|-----|-------|
| 2   | 0     |
| 3   | 0.5   |
| 4   | 1     |

---

## 2. Z-Score Normalization

### Rumus
Z = (X - μ) / σ

### Langkah Penyelesaian
1. Menghitung rata-rata (μ):
   μ = (2 + 3 + 4 + 2 + 3 + 4 + 2) / 7 = 2.86

2. Menghitung standar deviasi (σ):
   σ ≈ 0.83

3. Menghitung nilai Z-Score:
   - IPK = 2 → (2 - 2.86) / 0.83 = -1.04
   - IPK = 3 → (3 - 2.86) / 0.83 = 0.17
   - IPK = 4 → (4 - 2.86) / 0.83 = 1.37

---

## 3. Decimal Scaling

### Rumus
X' = X / (10^j)

### Langkah Penyelesaian
1. Menentukan nilai maksimum:
   - IPK maksimum = 4

2. Menentukan nilai j:
   - Karena nilai maksimum < 10, maka j = 1

3. Melakukan perhitungan:
   - IPK = 2 → 2 / 10 = 0.2
   - IPK = 3 → 3 / 10 = 0.3
   - IPK = 4 → 4 / 10 = 0.4

---

# Penanganan Missing Value Menggunakan WKNN

## Pengertian
WKNN (Weighted K-Nearest Neighbor) merupakan metode yang digunakan untuk mengisi data yang hilang berdasarkan kedekatan jarak dengan data lain, dengan mempertimbangkan bobot dari masing-masing tetangga terdekat.

---

## Langkah Penyelesaian

### 1. Menggunakan hasil normalisasi Min-Max
Data ke-7:
- IPK = 0
- PO = 0.5

---

### 2. Menghitung jarak menggunakan Euclidean Distance

Rumus:
d = √((x1 - x2)² + (y1 - y2)²)

Contoh perhitungan (dengan data ke-2):
d = √((0 - 0.5)² + (0.5 - 0.5)²) = 0.5

---

### 3. Menentukan K tetangga terdekat
Dipilih 2 data terdekat:
- Data ke-2 (JML = 3)
- Data ke-5 (JML = 2)

---

### 4. Menghitung bobot

Rumus:
w = 1 / d

- w1 = 1 / 0.5 = 2
- w2 = 1 / 0.5 = 2

---

### 5. Menghitung nilai akhir

JML = ((2 × 3) + (2 × 2)) / (2 + 2) = 2.5

Karena data bersifat diskrit, maka hasil dibulatkan menjadi:
**JML = 3**

---

## Hasil Akhir

| No | IPK | PO | JML |
|----|-----|-----|-----|
| 7 | 2 | 3000000 | 3 |

---

## Kesimpulan
Normalisasi data dilakukan untuk menyamakan skala data sebelum proses analisis. Metode yang digunakan meliputi Min-Max, Z-Score, dan Decimal Scaling. Sedangkan untuk menangani missing value digunakan metode WKNN dengan mempertimbangkan jarak dan bobot dari data terdekat.