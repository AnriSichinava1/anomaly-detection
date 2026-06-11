# ML-Based Anomaly Detection in Digital System Monitoring

**Bachelor's thesis project** — Anri Sitchinava & Tinatin Tavadze

This project replaces traditional static-threshold monitoring with machine learning that learns what "normal" looks like — including recurring expected patterns — and adapts without requiring labeled data per system.

## Headline Result

| | F1 | Recall | Precision |
|---|---|---|---|
| **Cascading Ensemble (unsupervised)** | **0.73** | **81%** | 68% |
| Random Forest (supervised) | 0.83 | 72% | 98% |

The unsupervised ensemble reaches **89% of the supervised performance ceiling without requiring any labels** during training, making it deployable on any new system with zero labeling effort.

## Project Structure

```
anomaly-detection/
├── data/
│   ├── memory_logs_raw.xlsx       # original raw memory measurements
│   ├── memory_logs_raw.csv        # same data as CSV for easy inspection
│   └── labeled.csv                # prepared dataset with features + synthetic anomaly labels
├── notebooks/
│   ├── isolation_forest_model.ipynb  # static threshold baseline + IF
│   ├── lof_model.ipynb               # Local Outlier Factor
│   ├── ocsvm_model.ipynb             # One-Class SVM
│   ├── random_forest_model.ipynb     # supervised performance ceiling
│   ├── ensemble_model.ipynb          # simple AND ensemble
│   ├── best_unsupervised.ipynb       # cascading ensemble (the headline result)
│   └── final_comparison.ipynb        # all models side by side
└── README.md
```

## Setup

```bash
pip install pandas numpy scikit-learn statsmodels matplotlib seaborn openpyxl jupyterlab
```

Then launch Jupyter Lab:

```bash
jupyter lab
```

## Recommended Reading Order

1. **`isolation_forest_model.ipynb`** — establishes the core problem (static thresholds fail) and demonstrates the first ML solution.
2. **`lof_model.ipynb`** — Local Outlier Factor, density-based detection (F1 = 0.35).
3. **`random_forest_model.ipynb`** — supervised baseline defining the performance ceiling (F1 = 0.83).
4. **`best_unsupervised.ipynb`** — cascading ensemble combining four models with smart thresholding (F1 = 0.73, the headline result).
5. **`final_comparison.ipynb`** — all models in one place with comparison plots and summary.

Two supporting notebooks (`ocsvm_model.ipynb`, `ensemble_model.ipynb`) cover alternative approaches and motivate the design of the cascading ensemble.

## Methodology

### Data
- 16,992 memory utilization measurements at 5-minute intervals
- Stable baseline (~41% memory), with recurring expected spikes
- 250 anomalies of 5 types synthetically injected for evaluation:
  - Short spikes, short drops, prolonged increases, prolonged drops, gradual ramps

### Feature Engineering
18 features in three groups:
- **Raw measurement:** memory percentage
- **Rolling statistics:** rolling mean and standard deviation over 1h and 24h windows
- **Rate of change:** time differences over 5min, 30min, 2h, 4h, and 8h windows
- **Contextual deviation:** z-scores and deviations from the typical value at each hour
- **Time features:** hour, minute, day of week, weekend flag

### Models
| Model | Type | Role |
|---|---|---|
| Static Threshold | Rule-based baseline | Demonstrates the problem with traditional monitoring |
| Isolation Forest | Unsupervised | Tree-based anomaly isolation |
| Local Outlier Factor | Unsupervised | Density-based detection |
| One-Class SVM | Unsupervised | Boundary-based detection |
| Random Forest | **Supervised** | Defines the performance ceiling |
| Simple AND Ensemble | Unsupervised | First attempt at model combination |
| Cascading Ensemble | Unsupervised | **The headline result** — score averaging + two-tier thresholds + gap filling |

### Statistical Evaluation
All key notebooks include a statsmodels logistic regression analysis providing Pseudo R², LLR significance test, AIC/BIC, and per-feature p-values.

## Deployment Framework

A tiered approach based on data availability:

- **Tier 1 (critical systems):** train Random Forest where labeled incident data is available — F1 = 0.83
- **Tier 2 (general fleet):** deploy the unsupervised cascading ensemble — F1 = 0.73, no labels needed, generalizes to any new system

## Authors

- **Anri Sitchinava** — Machine learning implementation (model selection, training, validation, statistical analysis)
- **Tinatin Tavadze** — Data engineering (preprocessing, feature engineering, synthetic anomaly generation, visualization design)
