# ⚡ Dynamic Real-Time System: Live Price Tracking & Auto-Update MCDM

## 📖 Konsep Dasar

**Dynamic Real-Time System** adalah SPK yang secara otomatis **meng-update data alternatif** (harga, availability, specs) dari platform e-commerce secara real-time atau berkala, kemudian **recalculate MCDM ranking** untuk memberikan rekomendasi yang selalu up-to-date.

### Problem: Static Data Goes Stale

MCDM tradisional menggunakan dataset statis:
```
Data Collection: 1 Januari 2025
├─ Laptop A: Rp 7.000.000
├─ Laptop B: Rp 12.000.000
└─ Laptop C: Rp 9.500.000

Problem (1 Februari 2025):
├─ Laptop A: Rp 6.500.000 (discount!) ← Data lama miss opportunity
├─ Laptop B: Out of stock ← User kecewa beli laptop unavailable
└─ Laptop C: Rp 10.200.000 (harga naik) ← Recommendation tidak akurat
```

**Impact**: Rekomendasi tidak relevan, user experience buruk.

### Solusi: Live Data Integration

Sistem secara otomatis scraping data terbaru dari e-commerce setiap:
- Real-time: Saat user request recommendation
- Scheduled: Setiap 6/12/24 jam via background job
- Event-triggered: Saat detect price drop alert

## ⚡ Novelty Elements

### 1. Real-Time Data Pipeline
Automated web scraping & data refresh untuk ensure data accuracy.

### 2. Temporal Analysis
Track price volatility, identify best time to buy, price trend forecasting.

### 3. Dynamic Availability Handling
Auto-filter out-of-stock items, prioritize in-stock alternatives.

### 4. Alert System
Notify user saat laptop favorit mereka discount atau back in stock.

## 🏗️ Arsitektur Sistem

```
┌──────────────────────────────────────────┐
│  🌐 E-Commerce APIs / Web Scraping       │
│  Tokopedia, Shopee, Lazada, Blibli      │
└────────────────┬─────────────────────────┘
                 │ Scheduled Jobs (Cron)
                 │ Every 12 hours
                 ↓
┌──────────────────────────────────────────┐
│  📊 Data Collection & Storage            │
│  PostgreSQL / MongoDB                    │
│  - Current prices                        │
│  - Historical price trends               │
│  - Stock availability                    │
│  - Timestamp metadata                    │
└────────────────┬─────────────────────────┘
                 │ Latest Data
                 ↓
┌──────────────────────────────────────────┐
│  🔄 Change Detection & Alerts            │
│  Detect: Price drops, stock changes      │
│  Trigger: User notifications             │
└────────────────┬─────────────────────────┘
                 │ Updated Matrix
                 ↓
┌──────────────────────────────────────────┐
│  🧮 MCDM Engine (TOPSIS/SAW)             │
│  Recalculate ranking with fresh data    │
└────────────────┬─────────────────────────┘
                 │ Live Ranking
                 ↓
┌──────────────────────────────────────────┐
│  🎯 Real-Time Recommendations            │
│  + Price trend charts                    │
│  + "Best time to buy" suggestions        │
└──────────────────────────────────────────┘
```

## 🔧 Implementasi Step-by-Step

### Step 1: Setup Database untuk Time-Series Data

```python
# Database schema (PostgreSQL)
CREATE TABLE laptop_prices (
    id SERIAL PRIMARY KEY,
    laptop_name VARCHAR(255),
    platform VARCHAR(50),  -- tokopedia, shopee, lazada
    price DECIMAL(10, 2),
    in_stock BOOLEAN,
    scraped_at TIMESTAMP DEFAULT NOW(),
    INDEX (laptop_name, scraped_at)
);

CREATE TABLE laptop_specs (
    laptop_name VARCHAR(255) PRIMARY KEY,
    processor_score INT,
    ram_gb INT,
    storage_gb INT,
    battery_hours INT,
    weight_kg DECIMAL(3, 2),
    screen_inch DECIMAL(3, 1),
    gpu_score INT,
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Step 2: Web Scraping dengan Scheduling

```python
import schedule
import time
from datetime import datetime
from scraper import scrape_tokopedia, scrape_shopee

def scrape_all_platforms():
    """Scrape prices dari semua platform"""
    laptops = ['Asus VivoBook', 'Lenovo ThinkPad', 'Acer Aspire']

    for laptop in laptops:
        # Scrape Tokopedia
        tokped_data = scrape_tokopedia(laptop)
        save_to_db(laptop, 'tokopedia', tokped_data)

        # Scrape Shopee
        shopee_data = scrape_shopee(laptop)
        save_to_db(laptop, 'shopee', shopee_data)

        time.sleep(5)  # Rate limiting

    print(f"Data updated at {datetime.now()}")

def save_to_db(laptop_name, platform, data):
    """Save scraped data ke database"""
    conn = get_db_connection()
    cursor = conn.cursor()

    cursor.execute("""
        INSERT INTO laptop_prices (laptop_name, platform, price, in_stock)
        VALUES (%s, %s, %s, %s)
    """, (laptop_name, platform, data['price'], data['in_stock']))

    conn.commit()
    conn.close()

# Schedule scraping setiap 12 jam
schedule.every(12).hours.do(scrape_all_platforms)

# Run scheduler
while True:
    schedule.run_pending()
    time.sleep(60)  # Check every minute
```

### Step 3: Get Latest Data untuk MCDM

```python
def get_latest_laptop_data():
    """Retrieve data terbaru untuk MCDM calculation"""
    conn = get_db_connection()

    # Query: Latest price per laptop (average dari semua platform)
    query = """
    SELECT
        lp.laptop_name,
        AVG(lp.price) as avg_price,
        MAX(lp.in_stock::int) as available,  -- 1 if any platform has stock
        ls.processor_score,
        ls.ram_gb,
        ls.storage_gb,
        ls.battery_hours,
        ls.weight_kg,
        ls.screen_inch,
        ls.gpu_score
    FROM (
        SELECT DISTINCT ON (laptop_name, platform)
            laptop_name, platform, price, in_stock, scraped_at
        FROM laptop_prices
        ORDER BY laptop_name, platform, scraped_at DESC
    ) lp
    JOIN laptop_specs ls ON lp.laptop_name = ls.laptop_name
    WHERE lp.scraped_at >= NOW() - INTERVAL '24 hours'
    GROUP BY lp.laptop_name, ls.processor_score, ls.ram_gb, ls.storage_gb,
             ls.battery_hours, ls.weight_kg, ls.screen_inch, ls.gpu_score
    """

    df = pd.read_sql(query, conn)
    conn.close()

    # Filter out out-of-stock items
    df = df[df['available'] == 1]

    return df
```

### Step 4: Real-Time MCDM Calculation

```python
from flask import Flask, jsonify
from topsis import TOPSIS

app = Flask(__name__)

@app.route('/api/recommendations')
def get_recommendations():
    """API endpoint untuk real-time recommendations"""

    # Get latest data
    laptops = get_latest_laptop_data()

    if laptops.empty:
        return jsonify({'error': 'No laptops available'}), 404

    # Prepare criteria matrix
    criteria_cols = ['avg_price', 'processor_score', 'ram_gb', 'storage_gb',
                     'battery_hours', 'weight_kg', 'screen_inch', 'gpu_score']
    criteria_matrix = laptops[criteria_cols].values

    # Weights (dapat disesuaikan per user)
    weights = [0.25, 0.20, 0.15, 0.10, 0.10, 0.05, 0.05, 0.10]

    # TOPSIS calculation
    topsis = TOPSIS(criteria_matrix, weights)
    scores = topsis.calculate()

    laptops['topsis_score'] = scores
    laptops = laptops.sort_values('topsis_score', ascending=False)

    # Return top-3 recommendations
    recommendations = laptops.head(3)[['laptop_name', 'avg_price', 'topsis_score']].to_dict('records')

    return jsonify({
        'recommendations': recommendations,
        'updated_at': datetime.now().isoformat()
    })

if __name__ == '__main__':
    app.run(debug=True)
```

### Step 5: Price Trend Analysis & Alerts

```python
def analyze_price_trend(laptop_name, days=30):
    """Analisis trend harga untuk prediksi best time to buy"""
    conn = get_db_connection()

    query = """
    SELECT scraped_at::date as date, AVG(price) as avg_price
    FROM laptop_prices
    WHERE laptop_name = %s
      AND scraped_at >= NOW() - INTERVAL '%s days'
    GROUP BY scraped_at::date
    ORDER BY date
    """

    df = pd.read_sql(query, conn, params=(laptop_name, days))

    # Simple trend analysis
    price_std = df['avg_price'].std()
    current_price = df['avg_price'].iloc[-1]
    min_price = df['avg_price'].min()
    avg_price = df['avg_price'].mean()

    # Recommendation logic
    if current_price <= min_price * 1.05:
        recommendation = "🔥 BEST TIME TO BUY - Price at lowest point!"
    elif current_price <= avg_price * 0.95:
        recommendation = "✅ GOOD TIME TO BUY - Price below average"
    elif current_price >= avg_price * 1.1:
        recommendation = "⏳ WAIT - Price above average, may drop soon"
    else:
        recommendation = "⚖️ AVERAGE PRICE - Consider waiting for discount"

    return {
        'current_price': current_price,
        'min_price': min_price,
        'avg_price': avg_price,
        'volatility': price_std,
        'recommendation': recommendation,
        'trend_data': df.to_dict('records')
    }
```

## 📊 Contoh Use Case: Price Drop Alert

**User Wishlist:**
```python
user_wishlist = {
    'user_id': 123,
    'laptop_name': 'Lenovo ThinkPad X1',
    'target_price': 11000000  # User mau beli kalau < Rp 11 juta
}
```

**Background Job (Check Hourly):**
```python
def check_price_alerts():
    """Check if any wishlist items hit target price"""
    wishlists = get_all_wishlists()

    for item in wishlists:
        current_price = get_current_price(item['laptop_name'])

        if current_price <= item['target_price']:
            send_notification(
                user_id=item['user_id'],
                message=f"🔔 ALERT: {item['laptop_name']} now Rp {current_price:,} "
                        f"(target: Rp {item['target_price']:,})"
            )
```

## ✅ Kelebihan Dynamic Real-Time System

1. **Maximum Novelty**: Real-time MCDM masih sangat jarang (novelty 9/10)
2. **High Practical Value**: User dapat actionable, up-to-date recommendations
3. **Competitive Edge**: Price tracking & alerts = killer feature
4. **Research Contribution**: Temporal analysis in MCDM = strong academic angle
5. **Scalable**: Architecture dapat handle multiple products & platforms

## ⚠️ Challenges & Complexity

### 1. Technical Complexity (Very High)
- Web scraping infrastructure
- Database management (time-series data)
- Background job scheduling
- API development
- Real-time data processing

### 2. Maintenance Overhead
- E-commerce site structure changes → scraper breaks
- Rate limiting & anti-scraping measures
- Server uptime & monitoring requirements

### 3. Legal & Ethical Concerns
- E-commerce ToS may prohibit scraping
- Alternative: Use official APIs (limited availability)

### 4. Data Quality
- Missing data handling
- Price inconsistencies across platforms
- Stock status accuracy

## 🎯 Rekomendasi

**Gunakan Dynamic Real-Time System jika:**
- ✅ Timeline >8 minggu (complex implementation)
- ✅ Tim experienced dengan web development & DevOps
- ✅ Akses server untuk hosting & scheduling
- ✅ Target publikasi internasional high-impact

**Simplifikasi untuk Feasibility:**
- Reduce to semi-real-time (update 1x per day, bukan hourly)
- Focus 1-2 platform saja (Tokopedia + Shopee)
- Manual data update via admin panel (skip auto-scraping)

## 📚 Tech Stack

**Backend:**
- `Flask/FastAPI`: REST API
- `Celery`: Background task scheduling
- `PostgreSQL`: Time-series data storage
- `Redis`: Caching & task queue

**Scraping:**
- `BeautifulSoup4` / `Selenium`: Web scraping
- `requests`: HTTP requests

**Deployment:**
- `Heroku` / `DigitalOcean`: Hosting
- `Cron jobs`: Scheduled scraping

**Complexity**: Very High (⭐⭐⭐⭐⭐)
**Novelty Score**: 9/10
**Publication Potential**: ⭐⭐⭐⭐
