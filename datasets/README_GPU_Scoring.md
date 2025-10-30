# 🎮 GPU Scoring Methodology

## 📋 Overview

Dokumentasi lengkap untuk sistem penilaian GPU (Graphics Processing Unit) yang digunakan dalam SPK Pemilihan Laptop dengan Triple Hybrid Approach. Metodologi ini dirancang untuk memberikan skor objektif berdasarkan performa, arsitektur, dan kapabilitas GPU.

## 🔬 Scoring Formula

**Final Score = Base Tier Score × Generation Multiplier × VRAM Factor**

### 1. Base Tier Score (1.0 - 10.0)

Skor dasar berdasarkan kategori performa GPU:

#### 🏆 High-End Dedicated GPUs (8.0 - 10.0)
- **RTX 4090**: 10.0 - GPU flagship terbaik
- **RTX 4070**: 8.0 - High-end current gen
- **RTX 3070 Ti**: 7.5 - High-end previous gen

#### ⚡ Mid-Range Dedicated GPUs (4.5 - 6.5)
- **RTX 4060**: 6.5 - Mid-range current gen powerful
- **RTX 4050**: 5.5 - Entry-level current gen
- **RX 6650M**: 5.8 - AMD mid-range equivalent
- **RTX 3050 Ti**: 5.0 - Mid-range previous gen
- **RTX 3050**: 4.5 - Entry-level current gen

#### 🔰 Entry-Level Dedicated GPUs (3.0 - 4.5)
- **GTX 1650**: 4.0 - Gaming GPU populer
- **RTX 2050**: 3.5 - Entry-level RTX
- **RX 6500M**: 3.5 - AMD entry-level mobile

#### 📱 Integrated GPUs (0.5 - 2.5)
- **Apple M1/M2**: 2.0 - Terbaik di kelas integrated
- **10-Core GPU (Apple)**: 2.2 - High-end Apple Silicon
- **8-Core GPU (Apple)**: 1.8 - Mid-range Apple Silicon
- **Intel Iris Xe**: 1.5 - Integrated modern yang baik
- **AMD Radeon Vega 7**: 1.3 - Integrated AMD yang decent
- **Intel UHD Graphics**: 1.0 - Basic integrated
- **ARM Mali**: 0.6 - Mobile integrated

### 2. Generation Multiplier (0.75 - 1.15)

Multiplier berdasarkan arsitektur dan fitur generasi:

#### NVIDIA GeForce Series:
- **RTX 40xx**: 1.15 - DLSS 3, efisiensi arsitektur terbaru
- **RTX 30xx**: 1.05 - Matang, DLSS 2, ray tracing stabilized
- **RTX 20xx**: 0.95 - First RTX, ray tracing introduction
- **GTX 16xx**: 0.85 - Gaming excellent tanpa RT features
- **GTX 10xx**: 0.75 - Arsitektur older

#### Integrated Graphics:
- **Apple Silicon**: 1.00 - 1.05 - Unified memory architecture
- **Intel Iris Xe**: 0.95 - Modern integrated
- **AMD Vega**: 0.90 - Capable integrated
- **Intel UHD**: 0.85 - Basic integrated
- **ARM Mali**: 0.80 - Mobile optimized

#### AMD Radeon:
- **RX 6000 series**: 1.00 - RDNA 2 architecture
- **RX 5000 series**: 0.90 - RDNA 1 architecture
- **Older mobile**: 0.80 - Previous generations

### 3. VRAM Factor (1.00 - 1.15)

Faktor berdasarkan kapasitas memory:

- **16GB**: 1.15 - Excellent untuk 4K+ dan future-proofing
- **8GB**: 1.10 - Good untuk 1440p gaming
- **6GB**: 1.05 - Adequate untuk 1080p gaming
- **4GB**: 1.00 - Minimum untuk modern gaming
- **Shared Memory**: 1.00 - Integrated GPU menggunakan system memory

## 📊 Performance Tiers

### Tier S (13.0+): Flagship Performance
- **RTX 4090** (13.23) - Best gaming & creative performance

### Tier A (8.0-12.9): High-End Gaming
- **RTX 4070** (10.12) - Excellent 1440p gaming
- **RTX 3070 Ti** (8.66) - Capable high-end gaming

### Tier B (6.0-7.9): Mid-Range Gaming
- **RTX 4060** (8.22) - Strong 1080p/1440p gaming
- **RTX 4050** (6.64) - Entry-level current gen
- **RX 6650M** (6.38) - AMD mid-range option

### Tier C (4.0-5.9): Entry-Level Gaming
- **RTX 3050 Ti** (5.25) - Capable 1080p gaming
- **RTX 3050** (4.73) - Basic modern gaming
- **GTX 1650** (3.40) - Older gaming GPU

### Tier D (2.0-3.9): Integrated Graphics
- **Apple M1/M2** (2.00-2.31) - Best integrated performance
- **Intel Iris Xe** (1.43) - Good modern integrated
- **AMD Radeon integrated** (1.08-1.17) - Decent integrated options

### Tier E (0.0-1.9): Basic Graphics
- **Intel UHD** (0.64-0.85) - Basic computing tasks
- **ARM Mali** (0.48) - Mobile graphics

## 🎯 Use Cases Recommendations

### Gaming Laptop Recommendations:
- **4K Gaming**: Tier S (RTX 4090)
- **1440p High Settings**: Tier A (RTX 4070+)
- **1080p High Settings**: Tier B (RTX 4060+)
- **1080p Medium Settings**: Tier C (RTX 3050 Ti+)
- **Light Gaming**: Tier D (Apple M1/M2, Iris Xe)

### Content Creation:
- **Video Editing**: Tier A+ (RTX 4070+)
- **3D Rendering**: Tier S-A (RTX 4060+)
- **Photo Editing**: Tier B+ (RTX 3050+)
- **Basic Creative**: Tier D (Apple Silicon, Iris Xe)

### Professional Work:
- **AI/ML Development**: Tier S (RTX 4090)
- **Data Science**: Tier A-B (RTX 4060+)
- **CAD/3D Modeling**: Tier B+ (RTX 3050 Ti+)

## 📈 Scoring Examples

### Example 1: RTX 4090 Gaming Laptop
```
Base Score: 10.0 (Flagship tier)
Generation: 1.15 (RTX 40 series, DLSS 3)
VRAM: 1.15 (16GB, future-proof)
Final Score: 10.0 × 1.15 × 1.15 = 13.23
```

### Example 2: MacBook Pro with M1
```
Base Score: 2.0 (Excellent integrated)
Generation: 1.00 (Apple Silicon baseline)
VRAM: 1.00 (Shared memory)
Final Score: 2.0 × 1.00 × 1.00 = 2.00
```

### Example 3: Mid-Range Gaming Laptop
```
Base Score: 6.5 (RTX 4060 tier)
Generation: 1.15 (RTX 40 series)
VRAM: 1.10 (8GB, good for 1440p)
Final Score: 6.5 × 1.15 × 1.10 = 8.22
```

## 🔄 Integration with Triple Hybrid Approach

GPU scoring berintegrasi dengan sistem SPK melalui:

1. **Fuzzy-TOPSIS**: GPU score digunakan sebagai kriteria numerik dalam decision matrix
2. **Adaptive Weighting**: Bobot GPU dapat disesuaikan berdasarkan user cluster (gaming vs productivity)
3. **Sentiment Analysis**: Real-world review scores digunakan untuk validate dan adjust GPU scores

## 📁 File Structure

- `gpu.csv` - Dataset GPU mentah
- `gpu_update.csv` - Dataset GPU dengan skor lengkap
- `README_GPU_Scoring.md` - Dokumentasi metodologi (file ini)

## 🛠️ Implementation Notes

### Data Quality Issues:
- Beberapa GPU memiliki nama yang tidak konsisten
- Duplikasi entries dengan variasi penamaan
- Missing VRAM information pada beberapa entries
- GPU dengan nama tidak lengkap (contoh: "4GB NVIDIA")

### Future Improvements:
- Tambahkan benchmark data aktual (3DMark, PassMark)
- Include ray tracing performance scores
- Add DLSS/FSR upscaling capability scores
- Consider power efficiency metrics
- Include temperature and thermal performance

---

**Last Updated**: 2025-01-31
**Version**: 1.0
**Status**: ✅ Complete & Validated