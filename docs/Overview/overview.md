# SPK Pemilihan Rekomendasi Laptop dengan menggabungkan Metode Fuzzy-TOPSIS + Adaptive Weighting + Sentiment Analysis

## Apa Itu Triple Hybrid?

**Triple Hybrid Approach** adalah strategi novelty untuk proyek SPK Pemilihan Laptop yang menggabungkan **3 metode** dalam 1 sistem terintegrasi:

```
Fuzzy-TOPSIS (Uncertainty Handling)
    +
Adaptive Weighting (Personalization)
    +
Sentiment Analysis (Real-World Feedback)
    =
SMART RECOMMENDATION SYSTEM
```

---

## 3 Masalah yang Diselesaikan

### 1 Ketidakpastian Penilaian → Fuzzy Logic

```
❌ Problem: "RAM 8GB = berapa? 7/10? 8/10?"
✅ Solution: "RAM 8GB = Sedang (60%) + Tinggi (40%)"
```

### 2 One-Size-Fits-All → Adaptive Weighting

```
❌ Problem: Mahasiswa CS ≠ Mahasiswa Bisnis
✅ Solution: Setiap user profile dapat bobot berbeda
```

### 3 Spec vs Reality Gap → Sentiment Analysis

```
❌ Problem: Spec bagus, tapi review buruk
✅ Solution: Tambah kriteria "User Satisfaction" dari reviews
```

---

## 📊 Kriteria MCDM: Spesifikasi & Scoring

### Overview: 9 Kriteria Terintegrasi

**Total Kriteria:** 9 (8 teknis + 1 sentimen)

| No | Kriteria | Tipe | Definisi | Scoring | Sumber Data |
|----|----------|------|----------|----------|-------------|
| 1 | **Harga** | Cost | Biaya laptop (Rp) | Min-Max | Marketplace |
| 2 | **Processor** | Benefit | Performa CPU (0-10) | Benchmark | PassMark |
| 3 | **RAM** | Benefit | Memori (GB) | Linear scaling | Spec |
| 4 | **Penyimpanan** | Benefit | Storage (GB) | Linear scaling | Spec |
| 5 | **Daya Tahan Baterai** | Benefit | Jam operasi | Linear scaling | Review |
| 6 | **Berat Laptop** | Cost | Berat (kg) | Max-Min | Spec |
| 7 | **Ukuran Layar** | Benefit | Diagonal (inch) | Linear scaling | Spec |
| 8 | **Kartu Grafis** | Benefit | GPU benchmark (0-10) | Benchmark | PassMark |
| 9 | **User Satisfaction** | Benefit | Sentiment reviews | NLP | E-commerce |

---

---

### 🎯 Bobot Kriteria (Adaptive per Jurusan)

#### Default Template Weights
```
[Harga, Processor, RAM, Storage, Battery, Weight, Screen, GPU, User_Satisfaction]
```

**Computer Science:** `[0.10, 0.30, 0.25, 0.15, 0.05, 0.05, 0.05, 0.05, 0.15]`
- **Processor & RAM**: Prioritas tertinggi (55% total)
- **User Satisfaction**: Weighted untuk validasi real-world

**Desain Grafis:** `[0.10, 0.15, 0.15, 0.10, 0.10, 0.05, 0.20, 0.15, 0.10]`
- **GPU & Screen**: Prioritas untuk visual work (55% total)

**Bisnis:** `[0.35, 0.10, 0.10, 0.10, 0.25, 0.20, 0.05, 0.05, 0.15]`
- **Harga, Battery, Weight**: Prioritas portabilitas (80% total)

**Balanced:** `[0.25, 0.20, 0.15, 0.12, 0.10, 0.08, 0.05, 0.05, 0.10]`
- **Distribusi seimbang untuk general use**

---

### 📋 Data Collection & Validation

#### Laptop Specification Matrix (15-20 Alternatives)
```
LaptopName   Harga  Processor  RAM   Storage  Battery  Weight  Screen  GPU
Lenovo X1    12jt  Intel i7    16GB   512GB    8jam    1.8kg   14"     RTX3050
Asus V15     7jt   Intel i5    8GB    512GB    6jam    1.5kg   15.6"   MX350
HP Pavilion  7.5jt AMD Ryzen5 8GB    1TB     7jam    1.7kg   15.6"   RX550
... (continue for all alternatives)
```

#### Sentiment Data Integration
```
LaptopName   AvgSentiment  TotalReviews  AvgRating
Lenovo X1    0.91          89            4.6/5
Asus V15     0.825         127           4.2/5
HP Pavilion  0.75          156           3.9/5
```

## Arsitektur Sistem

```
Input User Profile
    ↓
[1] User Selection → Personalized Weights
    ↓
[2] Laptop Data → Criteria Matrix (9xN)
    ↓
[3] Normalization → Scaled Matrix
    ↓
[4] Weighted Matrix → Apply User Weights
    ↓
TOPSIS Calculation → Distance to Ideal Solution
    ↓
Personalized Rankings
```

---

## Component Breakdown

---

## Component Breakdown

### Layer 1: Fuzzy-TOPSIS

**Fungsi**: Tangani ketidakpastian dalam penilaian

```
Traditional: Processor = 7.0 (pasti)
Fuzzy: Processor = "Bagus" (range 5-9)

 Lebih natural
 Robust terhadap variasi
```

### Layer 2: Adaptive Weighting

**Fungsi**: Personalisasi bobot per user

**Approach: Hybrid Template System**

```
Step 1: User pilih jurusan
├─ Dropdown: "Computer Science", "Design", "Business", dll
├─ Auto-assign jurusan template weights
└─ Display bobot default

Step 2: User customize (optional)
├─ Sliders untuk adjust weight per kriteria
├─ Auto-normalization ensure total = 100%
└─ Real-time preview perubahan

Jurusan Templates:
├─ Computer Science: [10%, 30%, 25%, 15%, 5%, 5%, 5%, 5%]
├─ Design Grafis: [10%, 15%, 15%, 10%, 10%, 5%, 20%, 15%]
├─ Bisnis: [35%, 10%, 10%, 10%, 15%, 10%, 5%, 5%]
└─ Lainnya: [balanced distribution]

 Feasible: Very High (1 minggu implementation)
 Novelty: Good (7/10)
 UX: Excellent (starts with template, user can customize)

```

### Layer 3: Sentiment Analysis

**Fungsi**: Integrasi user reviews

```
Laptop A:
- Spec: Good (8/10)
- Reviews: Excellent (127 reviews, 4.5/5)
- Sentiment Score: 0.91

Laptop B:
- Spec: Excellent (9/10)
- Reviews: Bad (203 reviews, 2.3/5) ⚠️
- Sentiment Score: 0.38

 Detect hidden issues
 Real-world validation
```

---

## Contoh Praktis

### Scenario: Budi - Mahasiswa CS

**Input:**

```
Budget: Rp 8 juta
Jurusan: Computer Science
Usage: Programming 8 jam/hari
```

**System Process:**

**Step 1 - Adaptive Weighting:**

```
Jurusan Selection → Budi = Computer Science
Template Weights: [Harga:10%, Processor:30%, RAM:25%, ...]
User can customize via sliders if needed
```

**Step 2 - Sentiment Analysis:**

```
Web scraping reviews:
- Laptop A: 0.825 (positive)
- Laptop B: 0.91 (very positive)
- Laptop C: 0.38 (negative) !
```

**Step 3 - Fuzzy-TOPSIS:**

```
Calculate with fuzzy ranges & personalized weights
Ranking:
1. Lenovo ThinkPad: 0.87
2. Asus VivoBook: 0.72
3. HP Pavilion: 0.68
```

**Output:**

```
 Recommendation: Lenovo ThinkPad
- Score: 0.87/1.0
- Processor i7 (9/10) → Perfect for CS
- RAM 16GB (10/10) → Smooth multitasking
- Reviews: 4.5/5 (89 reviews)
```

---

## Keunggulan

| Feature               | Benefit                                     |
| --------------------- | ------------------------------------------- |
| **3 Layer Novelty**   | Novelty score: 8.5/10                       |
| **Personalization**   | Setiap user dapat rekomendasi sesuai profil |
| **Real-World Data**   | Sentiment dari actual user reviews          |
| **Robust**            | Fuzzy handle uncertainty dengan baik        |
| **Publication Ready** | Strong research contribution                |

---

## Implementation Timeline

### 6 Minggu Roadmap

```
Week 1-2: Foundation
├─ Baseline SAW + TOPSIS
├─ Data collection (15-20 laptops)
└─ Basic UI (Streamlit)

Week 3: Fuzzy Layer
├─ Fuzzy membership functions
├─ Fuzzy-TOPSIS implementation
└─ Comparison: Classical vs Fuzzy

Week 4: Adaptive Layer
├─ Define jurusan-based weight templates
├─ Build template selection UI (dropdown)
├─ Implement weight customization sliders
└─ Auto-normalization logic

<!-- ├─ User survey (50-100 responden) -->
<!-- ├─ K-Means clustering -->
<!-- └─ Cluster weights assignment -->


Week 5: Sentiment Layer
├─ Web scraping (Tokopedia/Shopee)
├─ NLP sentiment analysis
└─ Integration as 9th criterion

Week 6: Integration & Validation
├─ Combine all layers
├─ Comparative analysis
├─ User testing
└─ Documentation
```

---

## Academic Value

### Research Contributions

1. **Methodological Innovation**

   - Novel integration Fuzzy + Clustering + NLP

2. **Practical Application**

   - Real-world recommendation system

3. **Empirical Validation**
   - Comparative study: Crisp vs Fuzzy vs Hybrid

### Possible Title

> "Adaptive Fuzzy-TOPSIS Integrated with Sentiment Analysis for Personalized Laptop Selection"

---

## Tech Stack

```
Core:
├─ Python 3.8+
├─ NumPy, Pandas (data processing)
└─ scikit-learn (clustering)

Fuzzy Logic:
└─ scikit-fuzzy

Web Scraping:
├─ BeautifulSoup4
├─ Selenium
└─ requests

NLP & Sentiment:
├─ TextBlob (basic)
├─ transformers (IndoBERT)
└─ nltk

UI:
├─ Streamlit (recommended)
└─ Flask/FastAPI (alternative)
```

---

## Quick Decision: Should You Use This?

### YES, if:

- Timeline 6-8 minggu
- Tim 2-3 orang
- Skill: Python + basic ML
- Goal: Publikasi / Nilai A+

### Consider Simpler, if:

- Timeline <4 minggu → Fuzzy only
- Solo project → Pick 2 layers
- Limited skills → Focus quality over quantity

---

## Detailed Documentation

**Untuk penjelasan lengkap setiap component:**

- `/docs/academic/01_fuzzy_topsis.md` - Fuzzy Logic details
- `/docs/academic/03_adaptive_weighting.md` - Clustering approach
- `/docs/academic/04_sentiment_enhanced_mcdm.md` - NLP integration
- `/docs/academic/00_overview_novelty_strategies.md` - All strategies comparison

---

## Next Steps

1. **Review dokumentasi lengkap** di `/docs/academic/`
2. **Diskusi tim**: Pilih approach yang sesuai timeline
3. **Prototype baseline**: Start dengan classical TOPSIS
4. **Incremental development**: Add layers progressively
5. **Validation**: Test & compare setiap layer

---

**Complexity**: Medium-High (⭐⭐⭐⭐)
**Novelty Score**: 8.5/10
**Timeline**: 6-8 weeks
**Publication Potential**: ⭐⭐⭐⭐⭐

**Status**: Ready for Implementation
