# 💬 Sentiment-Enhanced MCDM: Integrasi User Reviews dalam Keputusan

## 📖 Konsep Dasar

**Sentiment-Enhanced MCDM** menggabungkan kriteria objektif (spesifikasi teknis) dengan kriteria subjektif berbasis **user reviews** dari platform e-commerce. Sistem melakukan **sentiment analysis** terhadap review pengguna untuk menghasilkan skor kepuasan sebagai kriteria tambahan dalam MCDM.

### Gap dalam MCDM Tradisional

MCDM klasik hanya menggunakan spesifikasi teknis:
```
Kriteria Objektif:
✓ RAM: 8GB
✓ Processor: Intel i5
✓ Harga: Rp 7 juta
✓ Battery: 8 jam

Missing:
✗ Apakah user puas dengan performa real?
✗ Bagaimana pengalaman user dengan after-sales?
✗ Apakah ada masalah recurring (keyboard rusak, overheat)?
```

### Solusi: Sentiment Analysis Integration

Tambahkan kriteria **"User Satisfaction Score"** yang diekstrak dari:
- Review Tokopedia, Shopee, Lazada
- Rating & komentar user actual
- Real-world experience vs spec sheet

## ⚡ Novelty Elements

### 1. Hybrid Objective-Subjective Criteria
Kombinasi data kuantitatif (specs) dengan data kualitatif (user feedback).

### 2. NLP Integration in MCDM
Penggunaan Natural Language Processing untuk mengolah teks review menjadi skor numerik.

### 3. Real-World Validation
Hasil MCDM lebih mencerminkan kepuasan user actual, bukan hanya spesifikasi di kertas.

### 4. Dynamic Feedback Loop
Skor sentiment dapat di-update berkala sesuai review terbaru (living system).

## 🏗️ Arsitektur Sistem

```
┌─────────────────────────────────────────┐
│  🛒 E-Commerce Platforms                │
│  Tokopedia, Shopee, Lazada              │
└──────────────┬──────────────────────────┘
               │ Web Scraping
               ↓
┌─────────────────────────────────────────┐
│  📝 User Reviews Collection             │
│  Laptop A: 127 reviews                  │
│  Laptop B: 89 reviews                   │
│  Laptop C: 203 reviews                  │
└──────────────┬──────────────────────────┘
               │ Text Data
               ↓
┌─────────────────────────────────────────┐
│  🧠 Sentiment Analysis (NLP)            │
│  Model: TextBlob / IndoBERT             │
│  Output: Sentiment polarity [-1, 1]     │
└──────────────┬──────────────────────────┘
               │ Sentiment Scores
               ↓
┌─────────────────────────────────────────┐
│  📊 Enhanced Criteria Matrix            │
│  8 specs + 1 sentiment criterion        │
│  [Harga, CPU, RAM, ..., Satisfaction]   │
└──────────────┬──────────────────────────┘
               │ 9-Criteria Matrix
               ↓
┌─────────────────────────────────────────┐
│  🧮 TOPSIS/SAW Calculation              │
│  Sentiment-aware ranking                │
└──────────────┬──────────────────────────┘
               │ Ranking
               ↓
┌─────────────────────────────────────────┐
│  🏆 Recommendations + Review Insights   │
└─────────────────────────────────────────┘
```

## 🔧 Implementasi Step-by-Step

### Step 1: Web Scraping Reviews

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

def scrape_tokopedia_reviews(product_url):
    """Scrape reviews dari halaman produk Tokopedia"""
    headers = {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(product_url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')

    reviews = []
    # Selector sesuai struktur Tokopedia (dapat berubah)
    review_elements = soup.find_all('div', class_='review-content')

    for element in review_elements:
        review_text = element.find('span', class_='review-text').text
        rating = element.find('div', class_='rating').get('data-rating')
        reviews.append({
            'text': review_text,
            'rating': int(rating)
        })

    return reviews

# Scrape reviews untuk setiap laptop
laptop_urls = {
    'Asus VivoBook': 'https://tokopedia.com/...',
    'Lenovo ThinkPad': 'https://tokopedia.com/...',
    # ... laptop lainnya
}

all_reviews = {}
for laptop_name, url in laptop_urls.items():
    all_reviews[laptop_name] = scrape_tokopedia_reviews(url)
```

### Step 2: Sentiment Analysis

```python
from textblob import TextBlob
import numpy as np

def analyze_sentiment(review_text):
    """Analisis sentiment menggunakan TextBlob"""
    blob = TextBlob(review_text)
    polarity = blob.sentiment.polarity  # -1 (negative) to 1 (positive)
    return polarity

# Untuk Bahasa Indonesia, gunakan IndoBERT (lebih akurat)
from transformers import pipeline

sentiment_pipeline = pipeline('sentiment-analysis',
                              model='indolem/indobert-base-uncased')

def analyze_sentiment_indo(review_text):
    """Sentiment analysis untuk teks Bahasa Indonesia"""
    result = sentiment_pipeline(review_text)[0]
    # Convert to polarity score [-1, 1]
    if result['label'] == 'POSITIVE':
        return result['score']
    else:
        return -result['score']

# Calculate average sentiment per laptop
sentiment_scores = {}

for laptop_name, reviews in all_reviews.items():
    sentiments = [analyze_sentiment(r['text']) for r in reviews]
    avg_sentiment = np.mean(sentiments)
    sentiment_scores[laptop_name] = avg_sentiment

# Contoh output:
# sentiment_scores = {
#     'Asus VivoBook': 0.65,    # Mostly positive
#     'Lenovo ThinkPad': 0.82,  # Very positive
#     'Acer Aspire': -0.23,     # Slightly negative
# }
```

### Step 3: Normalize Sentiment ke [0, 1]

```python
# TOPSIS/SAW butuh nilai positif [0, 1]
# Konversi dari [-1, 1] ke [0, 1]

def normalize_sentiment(sentiment_polarity):
    return (sentiment_polarity + 1) / 2

normalized_sentiment = {
    laptop: normalize_sentiment(score)
    for laptop, score in sentiment_scores.items()
}

# Contoh:
# normalized_sentiment = {
#     'Asus VivoBook': 0.825,   # (0.65 + 1) / 2
#     'Lenovo ThinkPad': 0.91,  # (0.82 + 1) / 2
#     'Acer Aspire': 0.385      # (-0.23 + 1) / 2
# }
```

### Step 4: Augment Criteria Matrix

```python
# Original criteria matrix (8 kriteria)
criteria_matrix = pd.DataFrame({
    'Laptop': ['Asus VivoBook', 'Lenovo ThinkPad', 'Acer Aspire'],
    'Harga': [7000000, 12000000, 5500000],
    'Processor': [7, 9, 6],  # Normalized scores
    'RAM': [8, 16, 8],
    'Storage': [512, 512, 256],
    'Battery': [7, 8, 6],
    'Weight': [1.5, 1.8, 1.7],
    'Screen': [14, 14, 15.6],
    'GPU': [5, 8, 4]
})

# Tambahkan sentiment criterion (kriteria ke-9)
criteria_matrix['User_Satisfaction'] = criteria_matrix['Laptop'].map(normalized_sentiment)

print(criteria_matrix)
```

### Step 5: Run MCDM dengan 9 Kriteria

```python
from topsis import TOPSIS

# Update weights untuk 9 kriteria (tambah weight untuk sentiment)
weights = [0.20, 0.15, 0.12, 0.10, 0.08, 0.05, 0.05, 0.05, 0.20]
#          [Harga, CPU, RAM, Storage, Battery, Weight, Screen, GPU, Satisfaction]

# Sentiment mendapat bobot 20% (significant impact)

alternatives = criteria_matrix.drop('Laptop', axis=1).values
topsis = TOPSIS(alternatives, weights)
scores = topsis.calculate()

# Ranking dengan sentiment consideration
criteria_matrix['TOPSIS_Score'] = scores
ranking = criteria_matrix.sort_values('TOPSIS_Score', ascending=False)

print("Sentiment-Enhanced Ranking:")
print(ranking[['Laptop', 'TOPSIS_Score', 'User_Satisfaction']])
```

## 📊 Contoh Impact Analysis

### Scenario: Laptop dengan Spec Bagus tapi Review Buruk

**Laptop X - High Specs:**
- Processor: 9/10 (Intel i7 Gen 12)
- RAM: 10/10 (16GB)
- GPU: 8/10 (RTX 3050)
- Harga: 6/10 (Rp 11 juta - competitive)

**Tapi User Reviews Negatif:**
- "Overheat saat gaming" (-0.4)
- "Build quality buruk, casing gampang retak" (-0.6)
- "After-sales service lambat" (-0.5)
- Average Sentiment: -0.3 → Normalized: 0.35

**Tanpa Sentiment:**
- TOPSIS Score: 0.78 (Rank #2)

**Dengan Sentiment (bobot 20%):**
- TOPSIS Score: 0.62 (Rank #5 ↓)

**Insight**: Sistem dapat detect laptop dengan "hidden issues" yang tidak terlihat dari spec sheet.

## ✅ Kelebihan Sentiment-Enhanced MCDM

1. **High Novelty**: Sentiment analysis + MCDM integration jarang diteliti (novelty 8/10)
2. **Real-World Grounded**: Hasil lebih realistis karena consider user experience
3. **Fraud Detection**: Identify laptops dengan spec bagus tapi kualitas buruk
4. **Practical Value**: User lebih trust recommendation yang consider reviews
5. **Publikasi Angle**: Strong research contribution untuk journal submission

## ⚠️ Challenges & Solutions

### 1. Web Scraping Complexity
**Challenge**: E-commerce site structure berubah, rate limiting
**Solution:**
- Gunakan Selenium untuk dynamic content
- Implement retry logic & delays
- Cache hasil scraping untuk reduce requests

### 2. Bahasa Indonesia NLP Accuracy
**Challenge**: TextBlob kurang akurat untuk Bahasa Indonesia
**Solution:**
- Gunakan IndoBERT atau multilingual model
- Pre-trained model: `indolem/indobert-base-uncased`
- Fine-tune dengan dataset review lokal jika perlu

### 3. Review Spam & Fake Reviews
**Challenge**: Review palsu dapat bias sentiment score
**Solution:**
- Filter by verified purchase
- Weight by reviewer credibility
- Outlier detection untuk remove spam

## 🎯 Rekomendasi

**Gunakan Sentiment-Enhanced MCDM jika:**
- ✅ Timeline 5-6 minggu
- ✅ Skill web scraping (BeautifulSoup/Selenium)
- ✅ Interest dalam NLP & text analysis
- ✅ Ingin novelty tinggi dengan practical value

**Best Combination:**
Fuzzy-TOPSIS + Sentiment Analysis = Strong academic contribution!

## 📚 Tech Stack

**Web Scraping:**
- `BeautifulSoup4`: HTML parsing
- `Selenium`: Dynamic content handling
- `requests`: HTTP requests

**NLP & Sentiment Analysis:**
- `TextBlob`: English sentiment (basic)
- `transformers`: IndoBERT (Indonesian)
- `nltk`: Text preprocessing

**Data Processing:**
- `pandas`: Data manipulation
- `numpy`: Numerical operations

**Complexity**: Medium-High (⭐⭐⭐⭐)
**Novelty Score**: 8/10
**Publication Potential**: ⭐⭐⭐⭐
