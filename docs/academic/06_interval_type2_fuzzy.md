# 🎲 Interval Type-2 Fuzzy TOPSIS: Advanced Uncertainty Modeling

## 📖 Konsep Dasar

**Interval Type-2 Fuzzy (IT2F) TOPSIS** adalah pengembangan lanjutan dari Fuzzy-TOPSIS yang menangani **uncertainty of uncertainty** - yaitu ketidakpastian dalam mendefinisikan fuzzy membership function itu sendiri.

### Evolution: Crisp → Fuzzy → Type-2 Fuzzy

```
Level 0: Crisp Values
RAM 8GB = 8.0 (pasti, tidak ada uncertainty)

Level 1: Type-1 Fuzzy
RAM 8GB = {Medium: 0.6, High: 0.4}
Membership function jelas & pasti

Level 2: Type-2 Fuzzy (IT2F)
RAM 8GB = {Medium: [0.5-0.7], High: [0.3-0.5]}
Membership function sendiri memiliki range uncertainty
```

### Mengapa Type-2 Fuzzy?

**Problem dengan Type-1 Fuzzy:**
Expert A: "RAM 8GB termasuk Medium dengan membership 0.6"
Expert B: "RAM 8GB termasuk Medium dengan membership 0.7"
Expert C: "RAM 8GB termasuk Medium dengan membership 0.5"

**Disagreement antar expert** = Type-1 fuzzy tidak cukup capture uncertainty ini.

**Solusi IT2F:**
"RAM 8GB termasuk Medium dengan membership **antara 0.5 hingga 0.7**"
- Lower bound: 0.5 (expert paling konservatif)
- Upper bound: 0.7 (expert paling optimis)
- Footprint of Uncertainty (FOU): [0.5, 0.7]

## ⚡ Novelty Elements

### 1. Higher-Order Uncertainty Handling
Memodelkan ketidakpastian dalam judgment process itu sendiri, bukan hanya ketidakpastian dalam nilai.

### 2. Robust to Expert Disagreement
Capture variasi pendapat antar expert atau variasi kondisi (e.g., laptop performance varies by usage context).

### 3. Advanced Mathematical Framework
Menggunakan Interval Type-2 Fuzzy Sets dengan upper & lower membership functions.

### 4. Strong Theoretical Foundation
Published dalam high-impact journals, recognized methodology dalam uncertainty modeling.

## 🏗️ Matematika IT2F

### Type-2 Fuzzy Membership Function

**Type-1 Fuzzy Number:**
```
A = {(x, μ(x)) | x ∈ X}
μ(x) = single value in [0, 1]
```

**Interval Type-2 Fuzzy Number:**
```
Ã = {((x, u), μ(x, u)) | x ∈ X, u ∈ Jx ⊆ [0, 1]}
μ(x, u) = 1 for all u ∈ Jx (interval type-2)

Jx = [μ_lower(x), μ_upper(x)]
```

**Visual Representation:**
```
Membership
    1.0 |     /‾‾‾‾‾‾‾‾\
        |    /          \     ← Upper MF
        |   /            \
    0.7 |  /    FOU      \
        | /   (shaded)    \
    0.5 |/________________\  ← Lower MF
        |
    0.0 +-------------------->
           RAM (GB)
        4    8    12   16
```

## 🔧 Implementasi IT2F-TOPSIS

### Step 1: Define IT2F Linguistic Variables

```python
import numpy as np

class IntervalType2FuzzyNumber:
    def __init__(self, name, lower_mf, upper_mf):
        """
        IT2F Triangular Number
        lower_mf: (a1_L, a2_L, a3_L) - Lower membership function
        upper_mf: (a1_U, a2_U, a3_U) - Upper membership function
        """
        self.name = name
        self.lower_mf = lower_mf
        self.upper_mf = upper_mf

# Define IT2F linguistic scales
it2f_ratings = {
    'Very Low': IntervalType2FuzzyNumber(
        'Very Low',
        lower_mf=(0, 0, 2),
        upper_mf=(0, 0, 2.5)
    ),
    'Low': IntervalType2FuzzyNumber(
        'Low',
        lower_mf=(0, 2, 4),
        upper_mf=(0, 2.5, 5)
    ),
    'Medium': IntervalType2FuzzyNumber(
        'Medium',
        lower_mf=(2, 5, 7),
        upper_mf=(2.5, 5, 7.5)
    ),
    'High': IntervalType2FuzzyNumber(
        'High',
        lower_mf=(5, 7.5, 10),
        upper_mf=(5, 8, 10)
    ),
    'Very High': IntervalType2FuzzyNumber(
        'Very High',
        lower_mf=(7.5, 10, 10),
        upper_mf=(8, 10, 10)
    )
}
```

### Step 2: Aggregate Multiple Expert Opinions

```python
def aggregate_expert_ratings(expert_ratings):
    """
    Aggregate multiple expert assessments into IT2F number
    expert_ratings: list of Type-1 fuzzy ratings from different experts
    """
    # Extract all lower & upper bounds
    lower_bounds = [r.lower for r in expert_ratings]
    upper_bounds = [r.upper for r in expert_ratings]

    # IT2F aggregation
    it2f_lower = (
        min([l[0] for l in lower_bounds]),
        np.mean([l[1] for l in lower_bounds]),
        max([l[2] for l in lower_bounds])
    )

    it2f_upper = (
        min([u[0] for u in upper_bounds]),
        np.mean([u[1] for u in upper_bounds]),
        max([u[2] for u in upper_bounds])
    )

    return IntervalType2FuzzyNumber('Aggregated', it2f_lower, it2f_upper)

# Example: 3 experts rate laptop processor
expert1_rating = it2f_ratings['High']      # Optimistic
expert2_rating = it2f_ratings['Medium']    # Conservative
expert3_rating = it2f_ratings['High']      # Optimistic

aggregated = aggregate_expert_ratings([expert1_rating, expert2_rating, expert3_rating])
```

### Step 3: IT2F-TOPSIS Distance Calculation

```python
def it2f_distance(it2f_a, it2f_b):
    """
    Calculate distance between two IT2F numbers
    Using vertex method
    """
    # Lower MF distance
    d_lower = np.sqrt(
        (it2f_a.lower_mf[0] - it2f_b.lower_mf[0])**2 +
        (it2f_a.lower_mf[1] - it2f_b.lower_mf[1])**2 +
        (it2f_a.lower_mf[2] - it2f_b.lower_mf[2])**2
    )

    # Upper MF distance
    d_upper = np.sqrt(
        (it2f_a.upper_mf[0] - it2f_b.upper_mf[0])**2 +
        (it2f_a.upper_mf[1] - it2f_b.upper_mf[1])**2 +
        (it2f_a.upper_mf[2] - it2f_b.upper_mf[2])**2
    )

    # Average distance
    return (d_lower + d_upper) / 2

def it2f_topsis(decision_matrix, weights):
    """
    IT2F-TOPSIS algorithm
    decision_matrix: Matrix of IT2F numbers
    weights: IT2F weights for each criterion
    """
    # Step 1: Normalize IT2F decision matrix
    normalized_matrix = normalize_it2f_matrix(decision_matrix)

    # Step 2: Calculate weighted normalized matrix
    weighted_matrix = multiply_it2f_weights(normalized_matrix, weights)

    # Step 3: Determine IT2F ideal solutions
    A_plus = determine_it2f_positive_ideal(weighted_matrix)
    A_minus = determine_it2f_negative_ideal(weighted_matrix)

    # Step 4: Calculate distances
    distances_plus = []
    distances_minus = []

    for alternative in weighted_matrix:
        d_plus = sum([it2f_distance(alt, ideal) for alt, ideal in zip(alternative, A_plus)])
        d_minus = sum([it2f_distance(alt, ideal) for alt, ideal in zip(alternative, A_minus)])

        distances_plus.append(d_plus)
        distances_minus.append(d_minus)

    # Step 5: Calculate closeness coefficient
    closeness = [
        d_minus / (d_plus + d_minus)
        for d_plus, d_minus in zip(distances_plus, distances_minus)
    ]

    return closeness
```

### Step 4: Type Reduction (Defuzzification)

```python
def karnik_mendel_type_reduction(it2f_number):
    """
    Karnik-Mendel algorithm untuk type reduction
    Convert IT2F → Type-1 Fuzzy
    """
    # Simplified centroid calculation
    # Lower centroid
    c_lower = (it2f_number.lower_mf[0] + it2f_number.lower_mf[1] + it2f_number.lower_mf[2]) / 3

    # Upper centroid
    c_upper = (it2f_number.upper_mf[0] + it2f_number.upper_mf[1] + it2f_number.upper_mf[2]) / 3

    # Type-reduced interval
    return (c_lower, c_upper)

def defuzzify_it2f(it2f_interval):
    """
    Final defuzzification: IT2F → Crisp value
    """
    c_lower, c_upper = it2f_interval
    return (c_lower + c_upper) / 2
```

## 📊 Contoh Aplikasi: Multi-Expert Laptop Evaluation

**Scenario**: 3 expert menilai laptop untuk mahasiswa CS

**Expert 1 (Akademisi):**
- Processor: "High" (fokus computational power)
- RAM: "Very High" (research & ML tasks)
- GPU: "Medium" (tidak terlalu penting)

**Expert 2 (IT Professional):**
- Processor: "Very High" (production workload)
- RAM: "High" (multitasking)
- GPU: "High" (occasional rendering)

**Expert 3 (Mahasiswa Senior):**
- Processor: "Medium" (cukup untuk kuliah)
- RAM: "Medium" (8GB sudah oke)
- GPU: "Low" (jarang gaming)

**IT2F Aggregation Result:**
```
Processor IT2F:
- Lower MF: (2, 6, 8)    # Conservative bound
- Upper MF: (2.5, 7, 10) # Optimistic bound
- Interpretation: "Somewhere between Medium-High and High"

RAM IT2F:
- Lower MF: (2, 5.5, 9)
- Upper MF: (2.5, 6.5, 10)
- Interpretation: "Medium to Very High with high uncertainty"
```

**Advantage**: Capture semua expert opinions & uncertainty, bukan force consensus.

## ✅ Kelebihan IT2F-TOPSIS

1. **Strong Novelty**: IT2F + TOPSIS integration advanced topic (novelty 7/10)
2. **Robust Decision**: Handle disagreement & uncertainty better than Type-1
3. **Academic Rigor**: Strong theoretical foundation untuk publikasi
4. **Real-World Applicable**: Cocok untuk multi-expert decision making
5. **Publication Ready**: High-impact journal potential

## ⚠️ Challenges & Limitations

### 1. Mathematical Complexity (High)
- Advanced fuzzy set theory understanding required
- Complex algorithms (Karnik-Mendel, type reduction)
- Computationally expensive

### 2. Implementation Difficulty
- Limited ready-to-use libraries (mostly custom implementation)
- Debugging & validation challenging
- Longer development time

### 3. Explainability Trade-off
- Harder to explain to non-technical stakeholders
- "Uncertainty of uncertainty" concept abstract

### 4. Diminishing Returns
- For simple decision problems, Type-1 Fuzzy sudah cukup
- Overkill untuk single-expert scenarios

## 🎯 Rekomendasi

**Gunakan IT2F-TOPSIS jika:**
- ✅ Multi-expert evaluation context (3+ experts dengan diverse opinions)
- ✅ High uncertainty environment (volatile criteria, subjective assessment)
- ✅ Timeline 6-8 minggu (complex implementation)
- ✅ Strong mathematical background dalam tim
- ✅ Target publikasi high-impact journal internasional

**Skip IT2F jika:**
- ⚠️ Single expert / uniform opinion
- ⚠️ Timeline <5 minggu
- ⚠️ Prioritas simple & explainable system
- ⚠️ Limited math/fuzzy logic background

## 📚 Research Angle

**Possible Titles:**
- "Interval Type-2 Fuzzy TOPSIS for Laptop Selection Under High Expert Uncertainty"
- "Robust Multi-Criteria Decision Making Using IT2F Sets: A Case Study in Laptop Recommendation"
- "Handling Subjective Uncertainty in Technology Selection: An Interval Type-2 Fuzzy Approach"

**Key Contributions:**
1. Application of IT2F in consumer electronics selection (novel domain)
2. Multi-expert aggregation framework for laptop evaluation
3. Comparative analysis: Crisp vs Type-1 Fuzzy vs Type-2 Fuzzy
4. Sensitivity analysis showing robustness to expert disagreement

## 📚 Tech Stack & References

**Implementation:**
- `numpy`: Numerical computation
- Custom IT2F library (likely need to build from scratch)
- `scipy`: Optimization for type reduction

**Key Papers:**
- Mendel et al. (2006): "Interval Type-2 Fuzzy Logic Systems"
- Chen & Lee (2010): "Fuzzy Multiple Criteria Decision Making"
- Kahraman & Öztayşi (2014): "Fuzzy Analytic Hierarchy Process with Type-2 Fuzzy Sets"

**Complexity**: High (⭐⭐⭐⭐)
**Novelty Score**: 7/10
**Publication Potential**: ⭐⭐⭐⭐
**Recommended for**: Advanced research-focused projects with strong mathematical team
