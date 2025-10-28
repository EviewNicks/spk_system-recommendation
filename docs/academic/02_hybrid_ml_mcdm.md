# 🤖 Hybrid ML-MCDM: Machine Learning untuk Personalisasi MCDM

## 📖 Konsep Dasar

**Hybrid ML-MCDM** menggabungkan kekuatan **Machine Learning** (pembelajaran dari data) dengan **Multi-Criteria Decision Making** (keputusan terstruktur) untuk menciptakan sistem rekomendasi yang **adaptif** dan **personal**.

### Problem dengan MCDM Tradisional

Dalam SAW/TOPSIS klasik, bobot kriteria ditentukan secara **manual**:
- Bobot tetap: Harga=30%, Processor=20%, RAM=15%, dst.
- Sama untuk semua user (tidak personal)
- Tidak belajar dari preferensi historis

**Pertanyaan**: Bagaimana jika sistem bisa **belajar** bobot optimal dari perilaku user sebelumnya?

### Solusi: ML-Enhanced Weighting

Machine Learning model dilatih menggunakan data historis user untuk **memprediksi bobot kriteria yang optimal** berdasarkan profil dan preferensi individual.

## ⚡ Novelty Elements

### 1. Adaptive Weight Learning
Sistem secara otomatis menyesuaikan bobot kriteria berdasarkan:
- Profil user (budget, jurusan, kebutuhan komputasi)
- Historical choices (laptop apa yang dibeli user serupa)
- Feedback loop (rating kepuasan user terhadap rekomendasi)

### 2. Personalization Engine
Setiap user mendapat bobot kriteria yang disesuaikan dengan karakteristik mereka, bukan bobot universal.

### 3. Data-Driven Decision
Bobot tidak lagi berdasarkan intuisi/expert judgment saja, tetapi **evidence-based** dari data real.

## 🏗️ Arsitektur Sistem

```
┌─────────────────────────────────────────┐
│  📊 Historical User Data                │
│  - User profiles (budget, major, usage) │
│  - Past purchases & ratings             │
│  - Feature preferences                  │
└──────────────────┬──────────────────────┘
                   │ Training Data
                   ↓
┌─────────────────────────────────────────┐
│  🤖 ML Model (Random Forest/XGBoost)    │
│  Input: User profile features           │
│  Output: Predicted optimal weights      │
│  - w_harga, w_processor, w_ram, ...     │
└──────────────────┬──────────────────────┘
                   │ Predicted Weights
                   ↓
┌─────────────────────────────────────────┐
│  🧮 TOPSIS/SAW Engine                   │
│  Calculate scores with ML weights       │
└──────────────────┬──────────────────────┘
                   │ Ranking
                   ↓
┌─────────────────────────────────────────┐
│  🎯 Personalized Recommendations        │
│  Top-3 laptops for specific user        │
└─────────────────────────────────────────┘
```

## 🔧 Implementasi Step-by-Step

### Step 1: Data Collection & Preparation

```python
import pandas as pd
from sklearn.ensemble import RandomForestRegressor

# Historical user data
user_data = pd.DataFrame({
    'user_id': [1, 2, 3, ...],
    'budget_range': ['low', 'mid', 'high', ...],
    'major': ['CS', 'Design', 'Business', ...],
    'usage_intensity': [7, 5, 3, ...],  # hours/day
    'portability_need': [0.8, 0.3, 0.6, ...],
    # Target: Preferred weights (from survey/past behavior)
    'weight_harga': [0.4, 0.2, 0.15, ...],
    'weight_processor': [0.2, 0.3, 0.15, ...],
    'weight_ram': [0.15, 0.2, 0.1, ...],
    # ... other criteria weights
})
```

### Step 2: Feature Engineering

```python
# Encode categorical features
from sklearn.preprocessing import LabelEncoder

le_budget = LabelEncoder()
user_data['budget_encoded'] = le_budget.fit_transform(user_data['budget_range'])

le_major = LabelEncoder()
user_data['major_encoded'] = le_major.fit_transform(user_data['major'])

# Features for training
features = ['budget_encoded', 'major_encoded', 'usage_intensity', 'portability_need']
X = user_data[features]

# Target: Weight vector (multi-output regression)
targets = ['weight_harga', 'weight_processor', 'weight_ram', 'weight_storage',
           'weight_battery', 'weight_weight', 'weight_screen', 'weight_gpu']
y = user_data[targets]
```

### Step 3: Train ML Model

```python
from sklearn.model_selection import train_test_split
from sklearn.multioutput import MultiOutputRegressor

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Multi-output model (predict all weights simultaneously)
ml_model = MultiOutputRegressor(RandomForestRegressor(n_estimators=100))
ml_model.fit(X_train, y_train)

# Evaluate
score = ml_model.score(X_test, y_test)
print(f"Model R² Score: {score}")
```

### Step 4: Predict Weights for New User

```python
# New user profile
new_user = {
    'budget_encoded': le_budget.transform(['mid'])[0],
    'major_encoded': le_major.transform(['CS'])[0],
    'usage_intensity': 8,
    'portability_need': 0.7
}

new_user_features = pd.DataFrame([new_user])

# Predict optimal weights
predicted_weights = ml_model.predict(new_user_features)[0]
print(f"Predicted Weights: {predicted_weights}")
# Output: [0.25, 0.28, 0.18, 0.12, 0.05, 0.07, 0.03, 0.02]
```

### Step 5: Run MCDM with ML Weights

```python
# Use predicted weights in TOPSIS
from topsis import TOPSIS

alternatives = load_laptop_data()  # Matrix of laptop specs
topsis = TOPSIS(alternatives, predicted_weights)
ranking = topsis.calculate()

# Top-3 personalized recommendations
print(ranking[:3])
```

## 📊 Use Case: Personalisasi Berdasarkan Profil

**User A - Mahasiswa CS (Programming Focus):**
```
ML Predicted Weights:
- Processor: 0.30 (highest)
- RAM: 0.25
- GPU: 0.15
- Harga: 0.15
- Battery: 0.10
- Others: 0.05
```

**User B - Mahasiswa Desain Grafis:**
```
ML Predicted Weights:
- GPU: 0.35 (highest)
- Screen Size: 0.20
- RAM: 0.20
- Processor: 0.15
- Harga: 0.10
```

**User C - Budget-Conscious Student:**
```
ML Predicted Weights:
- Harga: 0.45 (highest)
- Battery: 0.20
- Weight: 0.15
- Processor: 0.10
- Others: 0.10
```

## ✅ Kelebihan Hybrid ML-MCDM

1. **High Novelty**: Kombinasi ML + MCDM masih jarang diteliti (novelty 9/10)
2. **True Personalization**: Setiap user dapat bobot yang disesuaikan
3. **Scalable**: Semakin banyak data, semakin akurat prediksi
4. **Publikasi Potential**: Sangat tinggi untuk jurnal internasional
5. **Practical Value**: Applicable untuk real-world recommendation systems

## ⚠️ Challenges & Trade-offs

1. **Data Requirement**: Butuh historical data (minimal 50-100 user samples)
   - **Solusi**: Gunakan synthetic data atau survey awal
2. **Complexity**: Dua sistem yang harus diimplementasi (ML + MCDM)
3. **Validation**: Perlu user testing untuk validate ML predictions
4. **Timeline**: Implementasi membutuhkan 6-8 minggu

## 🎯 Rekomendasi

**Gunakan Hybrid ML-MCDM jika:**
- ✅ Timeline >6 minggu
- ✅ Tim memiliki skill ML (Python: scikit-learn)
- ✅ Akses ke user data atau ready melakukan survey
- ✅ Target publikasi internasional

**Research Angle:**
> "Adaptive Multi-Criteria Decision Making for Laptop Selection Using Machine Learning-Enhanced Weight Optimization"

## 📚 Tech Stack

**Required Libraries:**
- `scikit-learn`: ML models (Random Forest, XGBoost)
- `pandas`: Data manipulation
- `numpy`: Numerical computation
- Custom TOPSIS/SAW implementation

**Complexity**: Very High (⭐⭐⭐⭐⭐)
**Novelty Score**: 9/10
**Publication Potential**: ⭐⭐⭐⭐⭐
