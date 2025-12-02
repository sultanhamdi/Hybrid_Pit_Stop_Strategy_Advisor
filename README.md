# Hybrid Pit Stop Strategy Advisor (HPSSA)

Sistem kecerdasan buatan untuk memberikan rekomendasi strategi pit stop pada balapan Formula 1. Sistem ini menggunakan pendekatan **hybrid** dengan menggabungkan tiga metode yang berbeda untuk menghasilkan keputusan.

> Final Project - Computational Intelligence Course (Semester 3)

## Arsitektur Sistem

Sistem HPSSA terdiri dari 5 modul yang saling terintegrasi:

1. **Modul 1 (Fuzzy Logic)** - Menganalisis kondisi ban dan cuaca untuk menghasilkan skor urgensi pit stop (0-10)
2. **Modul 2 (Bayesian Network)** - Memprediksi probabilitas terjadinya Safety Car berdasarkan tipe trek, cuaca, dan histori
3. **Modul 3 (First-Order Logic)** - Memvalidasi strategi terhadap regulasi FIA (mandatory compound rule)
4. **Modul 4 (Decision Engine)** - Mengintegrasikan ketiga modul di atas dengan data telemetri FastF1 untuk simulasi balapan
5. **Modul 5 (Real-time Interface)** - Menyediakan interface untuk analisis real-time dengan integrasi Weather API

## Modul-Modul

### Modul 1: Fuzzy Logic (Penganalisis Kondisi)

Menggunakan **Fuzzy Inference System** untuk menghitung skor urgensi pit stop (0-10) berdasarkan kondisi yang bersifat gradual.

**Variabel Input:**

- Umur Ban (0-60 laps): baru, awal, tengah, akhir, kritis
- Compound (0-10): hard, medium, soft, wet
- Cuaca (0-100%): cerah, mendung, hujan

**Variabel Output:**

- Urgensi (0-10): sangat_rendah, rendah, sedang, tinggi, sangat_tinggi

**Aturan Fuzzy:**

1. Ban kritis → urgensi sangat tinggi
2. Ban akhir → urgensi tinggi
3. Hujan + Ban slick → urgensi sangat tinggi (wrong tire)
4. Cerah + Ban wet → urgensi sangat tinggi (wrong tire)

### Modul 2: Bayesian Network (Risk Analyzer)

Menghitung probabilitas terjadinya **Safety Car** berdasarkan faktor-faktor yang saling mempengaruhi.

**Struktur DAG:**

```
    Trek ─────────┐
                  │
    Cuaca ────────┼──► Probabilitas SC
                  │
    Histori SC ───┘
```

**Conditional Probability Table (CPT):**
| Trek | Cuaca | Histori | P(SC) |
|------|-------|---------|-------|
| Permanent | Dry | Low | 10% |
| Street | Wet | High | 90% |

### Modul 3: First-Order Logic (Validator Regulasi)

Memvalidasi strategi pit stop berdasarkan **regulasi FIA Formula 1 (Article 30)**.

**Aturan FIA:**

- Wajib menggunakan minimal **2 compound berbeda** selama balapan (dry maupun wet)

**Knowledge Base:**

```prolog
IsDry(Soft), IsDry(Medium), IsDry(Hard)
IsWet(Intermediate), IsWet(Wet)
Different(X, Y) untuk setiap pasangan compound berbeda
Used(x) ∧ Used(y) ∧ Different(x, y) → MandatorySatisfied()
```

### Modul 4: Decision Engine (Race Simulator)

Mengintegrasikan ketiga modul untuk simulasi strategi menggunakan **data telemetri nyata dari FastF1**.

**Logika Keputusan:**

- Urgensi > 7.5 → **BOX BOX (Critical)**
- Urgensi >= 4.5 + SC Risk > 30% → **BOX BOX (Strategic+SC Risk)**
- Safety Car aktif → **BOX BOX (SC Opportunity)**
- Lainnya → **STAY OUT**

### Modul 5: Real-time Interface

Interface untuk input manual kondisi balapan dengan integrasi **Open-Meteo Weather API**.

**Parameter Input:**

- Circuit: Monaco, Silverstone, Spa-Francorchamps, dll.
- Tire Age: 0-60 laps
- Compound: SOFT, MEDIUM, HARD, INTER, WET
- Track Type: Street / Permanent
- SC History: Low / Medium / High

## Instalasi

### Prerequisites

- Python 3.10+
- pip

### Setup

```bash
# Clone repository
git clone https://github.com/sultanhamdi/Hybrid_Pit_Stop_Strategy_Advisor.git
cd Hybrid_Pit_Stop_Strategy_Advisor

# Buat virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# atau
.venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt
```

### Dependencies

```
numpy
pandas
matplotlib
seaborn
scikit-fuzzy
networkx
fastf1
requests
```

## Penggunaan

### 1. Jalankan Notebook

```bash
jupyter notebook HPSSA.ipynb
```

### 2. Test Case yang Tersedia

| GP        | Tahun | Driver | Karakteristik                       |
| --------- | ----- | ------ | ----------------------------------- |
| Monaco    | 2023  | VER    | Street circuit, high SC probability |
| Abu Dhabi | 2021  | VER    | Dramatis SC di lap terakhir         |
| Canada    | 2024  | RUS    | Multiple pit strategies             |

### 3. Real-time Analysis

```python
advisor = RealTimeAdvisor()
advisor.set_history(['SOFT'])
result = advisor.get_recommendation(
    tire_age=28,
    compound='MEDIUM',
    weather=10,  # atau gunakan get_weather(lat, lon)
    track_type='Street',
    sc_history='High',
    is_sc=False
)
```

## Output

### Visualisasi Strategy Modul 4

![Monaco 2023 Strategy](monaco_2023_strategy.png)

### Contoh Output realtime modul 5

```
==================================================
REAL-TIME STRATEGY ANALYSIS
==================================================
Tire: MEDIUM (28 laps) | Weather: 10% | Track: Street
--------------------------------------------------
[1] Urgency Score (Fuzzy): 6.2/10
[2] SC Probability (Bayesian): 70%
[3] Regulation Check (FOL): Valid - 2 different dry compounds used
--------------------------------------------------
RECOMMENDATION: BOX BOX (Strategic)
Reason: Strategic pit window is open
==================================================
```

## Struktur File

```
Hybrid_Pit_Stop_Strategy_Advisor/
├── HPSSA.ipynb              # Main notebook
├── README.md                # Dokumentasi
├── circuit coordinate.csv   # Database koordinat 24 sirkuit F1
├── requirements.txt         # Dependencies
├── .gitignore
├── Logical Agents/          # AIMA library untuk FOL
│   ├── logic.py
│   ├── utils.py
│   └── ...
└── aima-data/               # Data pendukung AIMA
```

## Referensi

- [FastF1 Documentation](https://docs.fastf1.dev/)
- [scikit-fuzzy](https://pythonhosted.org/scikit-fuzzy/)
- [AIMA Python](https://github.com/aimacode/aima-python)
- [Open-Meteo API](https://open-meteo.com/)
- [FIA Formula 1 Sporting Regulations](https://www.fia.com/regulation/category/110)

## Author

- **Sultan Hamdi** - Lead
- **Fatih** - Member
- **Belva** - Member

## License

MIT License
