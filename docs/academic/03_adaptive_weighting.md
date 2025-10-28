# 🎯 Adaptive Weighting: Personalisasi via User Clustering

## 📖 Konsep Dasar

**Adaptive Weighting System** menggunakan teknik **clustering** untuk mengelompokkan user dengan karakteristik serupa, kemudian memberikan **bobot kriteria yang disesuaikan** untuk setiap cluster. Ini adalah middle-ground antara bobot universal (semua user sama) dan ML-based prediction (butuh training data besar).

### Problem: One Size Doesn't Fit All

MCDM tradisional menggunakan bobot tetap untuk semua user:
```
Bobot Universal:
- Harga: 25%
- Processor: 20%
- RAM: 15%
- Storage: 15%
- Battery: 10%
- Weight: 5%
- Screen: 5%
- GPU: 5%
```

**Masalah**: Mahasiswa CS butuh processor kuat ≠ Mahasiswa bisnis butuh battery tahan lama ≠ Mahasiswa desain butuh GPU bagus.

### Solusi: Cluster-Based Personalization

Kelompokkan user ke dalam **archetypes** berdasarkan profil, lalu assign bobot spesifik untuk setiap archetype.

## ⚡ Novelty Elements

### 1. User Profiling & Clustering
Sistem mengidentifikasi pola similarity antar user menggunakan K-Means clustering berdasarkan:
- Budget range (low/mid/high)
- Usage intensity (casual/moderate/heavy)
- Portability need (mobile/balanced/stationary)
- Performance requirement (basic/standard/high)

### 2. Cluster-Specific Weights
Setiap cluster mendapat bobot kriteria yang dioptimalkan untuk karakteristik kelompok tersebut.

### 3. Hybrid Approach: Expert + Data-Driven
Kombinasi expert judgment (AHP) untuk initial weights + clustering untuk personalization.

## 🏗️ Arsitektur Sistem

```
┌──────────────────────────────────────┐
│  👤 User Input                       │
│  - Budget: Rp 5-8 juta               │
│  - Usage: Programming (8 jam/hari)   │
│  - Portability: Medium (3x/minggu)   │
│  - Performance: High (compile, VM)   │
└───────────────┬──────────────────────┘
                │ Profile Features
                ↓
┌──────────────────────────────────────┐
│  🎯 K-Means Clustering               │
│  Assign user to nearest cluster      │
│  Cluster 0: Budget-Conscious         │
│  Cluster 1: Performance-Focused      │
│  Cluster 2: Portability-Focused      │
│  Cluster 3: Balanced Users           │
└───────────────┬──────────────────────┘
                │ User → Cluster 1
                ↓
┌──────────────────────────────────────┐
│  📊 Retrieve Cluster Weights         │
│  Performance-Focused Weights:        │
│  [0.10, 0.30, 0.25, 0.15, ...]       │
└───────────────┬──────────────────────┘
                │ Personalized Weights
                ↓
┌──────────────────────────────────────┐
│  🧮 TOPSIS/SAW Calculation           │
│  Dengan cluster-specific weights     │
└───────────────┬──────────────────────┘
                │ Ranking
                ↓
┌──────────────────────────────────────┐
│  🏆 Personalized Recommendations     │
└──────────────────────────────────────┘
```

## 🔧 Implementasi Step-by-Step

### Step 1: Define User Archetypes (Survey/Expert)

Lakukan survey terhadap 50-100 mahasiswa untuk identifikasi archetypes:

```python
import pandas as pd

# Survey data
user_survey = pd.DataFrame({
    'user_id': range(1, 101),
    'budget': [5, 7, 12, 6, ...],  # dalam juta
    'usage_hours': [8, 3, 6, 4, ...],
    'portability_score': [0.8, 0.3, 0.5, ...],  # 0-1 scale
    'performance_need': [0.9, 0.4, 0.7, ...],  # 0-1 scale
    # Preferred weights dari survey (optional)
    'pref_weight_harga': [0.4, 0.2, 0.1, ...],
    'pref_weight_processor': [0.2, 0.15, 0.35, ...]
})
```

### Step 2: K-Means Clustering

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

# Features untuk clustering
features = ['budget', 'usage_hours', 'portability_score', 'performance_need']
X = user_survey[features]

# Standardize (agar semua fitur setara)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# K-Means dengan 4 clusters
kmeans = KMeans(n_clusters=4, random_state=42)
user_survey['cluster'] = kmeans.fit_predict(X_scaled)

# Cluster centers
cluster_centers = kmeans.cluster_centers_
```

### Step 3: Define Cluster-Specific Weights

Untuk setiap cluster, hitung average preferred weights dari user dalam cluster tersebut:

```python
# Calculate average weights per cluster
cluster_weights = {}

for cluster_id in range(4):
    cluster_users = user_survey[user_survey['cluster'] == cluster_id]

    # Average weights dari user dalam cluster ini
    avg_weights = {
        'harga': cluster_users['pref_weight_harga'].mean(),
        'processor': cluster_users['pref_weight_processor'].mean(),
        'ram': cluster_users['pref_weight_ram'].mean(),
        # ... kriteria lain
    }

    cluster_weights[cluster_id] = list(avg_weights.values())

# Contoh hasil:
# cluster_weights = {
#     0: [0.40, 0.15, 0.12, 0.10, 0.12, 0.08, 0.02, 0.01],  # Budget-Conscious
#     1: [0.10, 0.30, 0.25, 0.15, 0.05, 0.05, 0.05, 0.05],  # Performance-Focused
#     2: [0.15, 0.10, 0.12, 0.10, 0.25, 0.20, 0.05, 0.03],  # Portability-Focused
#     3: [0.20, 0.18, 0.15, 0.12, 0.12, 0.10, 0.08, 0.05]   # Balanced
# }
```

### Step 4: Assign New User to Cluster

```python
# New user profile
new_user = pd.DataFrame({
    'budget': [7],
    'usage_hours': [8],
    'portability_score': [0.6],
    'performance_need': [0.9]
})

# Standardize dengan scaler yang sama
new_user_scaled = scaler.transform(new_user)

# Predict cluster
user_cluster = kmeans.predict(new_user_scaled)[0]
print(f"User assigned to Cluster: {user_cluster}")  # e.g., Cluster 1

# Get weights untuk cluster ini
personalized_weights = cluster_weights[user_cluster]
```

### Step 5: Run MCDM dengan Cluster Weights

```python
from topsis import TOPSIS

# Load laptop alternatives
laptops = load_laptop_data()

# TOPSIS dengan personalized weights
topsis = TOPSIS(laptops, personalized_weights)
ranking = topsis.calculate()

print(f"Top-3 Recommendations for Cluster {user_cluster}:")
print(ranking[:3])
```

## 📊 Contoh User Archetypes

### Cluster 0: Budget-Conscious Students
```
Profile:
- Budget: Rp 4-6 juta (low)
- Usage: 3-5 jam/hari (moderate)
- Portability: Medium
- Performance: Basic

Weights:
- Harga: 40% (tertinggi)
- Battery: 12%
- Weight: 8%
- Processor: 15%
- RAM: 12%
- Others: 13%
```

### Cluster 1: Performance-Focused (CS/Engineering)
```
Profile:
- Budget: Rp 8-15 juta (high)
- Usage: 8-12 jam/hari (heavy)
- Portability: Low (mostly stationary)
- Performance: High (compile, VM, ML)

Weights:
- Processor: 30% (tertinggi)
- RAM: 25%
- GPU: 5%
- Storage: 15%
- Harga: 10%
- Others: 15%
```

### Cluster 2: Portability-Focused (Mobile Workers)
```
Profile:
- Budget: Rp 6-10 juta (mid)
- Usage: 5-7 jam/hari
- Portability: High (daily mobility)
- Performance: Moderate

Weights:
- Battery: 25% (tertinggi)
- Weight: 20%
- Screen: 5%
- Processor: 10%
- RAM: 12%
- Others: 28%
```

### Cluster 3: Balanced Users
```
Profile:
- Budget: Rp 6-9 juta (mid)
- Usage: 4-6 jam/hari
- Portability: Medium
- Performance: Standard

Weights:
- Semua kriteria relatif seimbang
- Harga: 20%
- Processor: 18%
- RAM: 15%
- Others: ~8-12% each
```

## ✅ Kelebihan Adaptive Weighting

1. **Moderate Novelty**: Clustering + MCDM integration jarang diteliti (novelty 8/10)
2. **Practical**: Tidak butuh training data besar seperti ML approach
3. **Interpretable**: Cluster archetypes mudah dijelaskan dan divalidasi
4. **Scalable**: Mudah ditambahkan cluster baru sesuai data
5. **Feasible**: Implementasi moderate complexity (4-5 minggu)

## ⚠️ Challenges

1. **Survey Requirement**: Butuh 50-100 responden untuk clustering yang robust
   - **Solusi**: Gunakan Google Forms, distribusi di grup mahasiswa
2. **Cluster Count**: Harus tentukan optimal K (jumlah cluster)
   - **Solusi**: Gunakan Elbow Method atau Silhouette Score
3. **Cluster Interpretation**: Perlu validasi expert untuk archetype labeling

## 🎯 Rekomendasi

**Gunakan Adaptive Weighting jika:**
- ✅ Timeline 4-6 minggu
- ✅ Punya akses untuk survey mahasiswa
- ✅ Ingin novelty tinggi dengan complexity moderate
- ✅ Target publikasi nasional/internasional

**Kombinasi Powerful:**
Fuzzy-TOPSIS + Adaptive Weighting = Strong novelty dengan feasible implementation!

## 📚 Tech Stack

**Libraries:**
- `scikit-learn`: KMeans, StandardScaler
- `pandas`: Data processing
- `numpy`: Numerical operations
- `matplotlib`: Cluster visualization

**Complexity**: Medium-High (⭐⭐⭐⭐)
**Novelty Score**: 8/10
**Publication Potential**: ⭐⭐⭐⭐
