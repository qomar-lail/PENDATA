# Normalisasi Data (Preprocessing)

## Pengertian Normalisasi Data
Normalisasi data adalah proses mengubah nilai data ke dalam skala tertentu agar memiliki rentang yang seragam, sehingga tidak ada atribut yang mendominasi dalam proses analisis.

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

# 1. Min-Max Normalization

## Rumus
X' = (X - Xmin) / (Xmax - Xmin)

## Langkah Penyelesaian

### 1. Menentukan nilai minimum dan maksimum (dari seluruh data IPK)
Data IPK: 2, 3, 4, 2, 3, 4, 2  
- Nilai minimum (Xmin) = 2  
- Nilai maksimum (Xmax) = 4  

---

### 2. Melakukan normalisasi

Contoh perhitungan:

- Data ke-1 (IPK = 2):
  (2 - 2) / (4 - 2) = 0 / 2 = 0

- Data ke-2 (IPK = 3):
  (3 - 2) / (4 - 2) = 1 / 2 = 0.5

- Data ke-3 (IPK = 4):
  (4 - 2) / (4 - 2) = 2 / 2 = 1

(Hasil yang sama berlaku untuk data lain dengan nilai IPK yang sama)

---

# 2. Z-Score Normalization

## Rumus
Z = (X - μ) / σ

## Langkah Penyelesaian

### 1. Menghitung rata-rata (μ)
Data IPK: 2, 3, 4, 2, 3, 4, 2  

μ = (2 + 3 + 4 + 2 + 3 + 4 + 2) / 7  
μ = 20 / 7 = 2.86  

---

### 2. Menghitung standar deviasi (σ)

Selisih tiap data dengan rata-rata:
- (2 - 2.86)² = 0.74
- (3 - 2.86)² = 0.02
- (4 - 2.86)² = 1.30
(dan seterusnya untuk semua data)

Jumlah ≈ 4.86  

σ = √(4.86 / 7) ≈ 0.83  

---

### 3. Menghitung Z-Score

Contoh:

- Data ke-1 (IPK = 2):
  (2 - 2.86) / 0.83 = -1.04

- Data ke-2 (IPK = 3):
  (3 - 2.86) / 0.83 = 0.17

- Data ke-3 (IPK = 4):
  (4 - 2.86) / 0.83 = 1.37

---

# 3. Decimal Scaling

## Rumus
X' = X / (10^j)

## Langkah Penyelesaian

### 1. Menentukan nilai maksimum
Nilai maksimum IPK = 4

### 2. Menentukan nilai j
Karena 4 < 10, maka j = 1

### 3. Perhitungan

- Data ke-1 (IPK = 2):
  2 / 10 = 0.2

- Data ke-2 (IPK = 3):
  3 / 10 = 0.3

- Data ke-3 (IPK = 4):
  4 / 10 = 0.4

---

# Penanganan Missing Value Menggunakan WKNN

## Tujuan
Menentukan nilai JML pada data ke-7 yang kosong.

---

## Langkah Penyelesaian

### 1. Menggunakan hasil normalisasi Min-Max

Data ke-7:
- IPK = 2 → hasil normalisasi = 0  
- PO = 3000000 → hasil normalisasi = 0.5  

---

### 2. Menghitung jarak ke data lain

Menggunakan rumus Euclidean:

d = √((IPK1 - IPK2)² + (PO1 - PO2)²)

---

### 3. Contoh perhitungan jarak

#### Jarak ke data ke-2:
Data ke-2: (IPK = 0.5, PO = 0.5)

d = √((0 - 0.5)² + (0.5 - 0.5)²)  
d = √(0.25 + 0) = 0.5  

---

#### Jarak ke data ke-5:
Data ke-5: (IPK = 0.5, PO = 0.5)

d = √((0 - 0.5)² + (0.5 - 0.5)²)  
d = 0.5  

---

### 4. Menentukan tetangga terdekat

Dipilih 2 data dengan jarak terkecil:
- Data ke-2 (JML = 3)
- Data ke-5 (JML = 2)

---

### 5. Menghitung bobot

Rumus:
w = 1 / d

- Bobot data ke-2 = 1 / 0.5 = 2
- Bobot data ke-5 = 1 / 0.5 = 2

---

### 6. Menghitung nilai JML

JML = ((2 × 3) + (2 × 2)) / (2 + 2)  
JML = (6 + 4) / 4 = 2.5  

Karena JML berupa bilangan diskrit, maka dibulatkan menjadi:
**JML = 3**

---

# Hasil Akhir

| No | IPK | PO | JML |
|----|-----|-----|-----|
| 7 | 2 | 3000000 | 3 |

---

# Kesimpulan
Setiap metode normalisasi memiliki cara perhitungan yang berbeda, namun bertujuan untuk menyamakan skala data. Untuk mengatasi missing value digunakan metode WKNN dengan mempertimbangkan jarak dan bobot dari data terdekat sehingga diperoleh hasil yang lebih representatif.