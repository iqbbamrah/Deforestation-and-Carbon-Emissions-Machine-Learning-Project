# Deforestation-and-Carbon-Emissions-Machine-Learning-Project
Group Project for the University of Waterloo WATSPEED Machine Learning course as part of the Data Science Certificate.

# Deforestation Trajectories & Carbon Emissions

Unsupervised clustering of global deforestation patterns (2001–2022) combined with a supervised model predicting country-level carbon emissions from forest-loss and carbon-stock data.

**Team:** Gonzalo Vazquez, Iqbal Bamrah, Farnoush Attarzadeh

## Problem

Global forest-loss data is usually summarized as a single "total loss" number per country, which hides very different underlying patterns. A country losing forest steadily at a low rate looks nothing like one experiencing a sudden hotspot event, even if their totals are similar. This project asks two questions:

1. Can we group ~3,000 countries/subnational regions into meaningful clusters based on the *shape* of their deforestation trajectory over 22 years, not just the total?
2. Once we understand loss patterns, can forest-loss and carbon-stock variables predict a country's gross carbon emissions?

## Data

- **Source:** [Global Forest Data 2001–2022 (Kaggle)](https://www.kaggle.com) — 20,000+ rows across countries, states, provinces, and territories; year-2000 forest extent baseline; 22 annual tree-cover-loss variables.
- **Files used:** country-level and subnational tree-cover-loss files, plus matching country- and subnational-level carbon files (gross emissions, gross removals, net flux).
- **Final clustering dataset:** 3,029 region-level observations after cleaning.
- **Final emissions modeling dataset:** 236-country table (deforestation + carbon merged, threshold-30, no missing values).

## Methodology

**Data preparation**
- Merged country- and subnational-level files into one working dataset with a unified `region_name` key.
- Compared canopy-density thresholds (10/30/50) — a genuine methodological decision, not a simple filter — and selected **threshold 30** as the best balance between forest definition and clustering stability.
- Normalized annual tree-cover loss by each region's **2000 forest extent** (not land area), so trajectories are comparable across regions of very different size.
- Filtered out regions with zero baseline forest extent and applied a 1,000-hectare minimum to avoid unstable percentage-based trajectories from tiny forest bases.

**Clustering**
- Standardized features with `StandardScaler`.
- Reduced 21 variables to **11 principal components** via PCA, retaining ~90.8% of variance.
- Ran **K-Means** (`n_init=10`), selecting **k=3** using the elbow method and silhouette score.
- Validated by re-running K-Means on the full standardized feature set without PCA. Near-perfect agreement with the PCA-based solution, confirming PCA didn't distort the underlying grouping.

**Predictive extension (carbon emissions)**
- Compared absolute (`cumulative_loss_ha`) vs. relative (`cumulative_loss_pct`) forest loss as predictors of gross emissions. Absolute loss was far more informative (r = 0.895 vs. r = 0.073).
- Compared **Linear Regression** vs. **Random Forest Regressor** on country-level gross emissions.

**Tools:** Python, pandas, scikit-learn (`KMeans`, `PCA`, `StandardScaler`, `RandomForestRegressor`, `LinearRegression`), matplotlib.

## Results

**Three deforestation clusters identified:**

| Cluster | Label | n | Pattern |
|---|---|---|---|
| 1 | Low & Mostly Stable Deforestation | 2,952 | Global baseline; low, stable annual loss |
| 0 | Moderate & Sustained Deforestation | 622 | Elevated, sustained loss (e.g. DRC) |
| 2 | High-Intensity Deforestation Hotspots | 162 | 20–30x baseline loss; concentrated in Indonesia, Laos, Vietnam — linked to palm oil and agricultural expansion |

A consistent spike in forest loss appeared across **all three clusters in 2016–2017**, aligning with documented global deforestation trends and El Niño-related wildfire activity that year.

**Emissions prediction:**

| Model | R² | MAE | RMSE |
|---|---|---|---|
| Linear Regression | 0.967 | 9,774,348 | 22,268,140 |
| **Random Forest** | **0.978** | **7,085,422** | **18,383,090** |

Top predictors (Random Forest feature importance): cumulative forest loss (ha), 2000 baseline carbon stock, recent loss, early loss, 2000 forest extent. Forest gain and average carbon density per hectare contributed comparatively little.

## Key takeaways

- Deforestation isn't one pattern — grouping by trajectory shape (not just total loss) surfaces a small set of high-intensity hotspot regions that a simple "top N by total loss" ranking would blend in with moderate-loss regions.
- Representation matters for modeling: **relative** loss was right for clustering (fair comparison across region sizes), but **absolute** loss was right for predicting emissions (emissions are measured in absolute terms).
- Results are explanatory, not causal — this is a strong signal-finding exercise, not a policy-ready causal model.

## Repo structure
```
├── data/                      # raw + cleaned CSVs (or links if too large for GitHub)
├── notebooks/
│   └── deforestation_carbon_analysis.ipynb
├── report/
│   └── deforestation_carbon_emissions_report.pdf
└── README.md
```

## Possible extensions
- Time-series clustering (e.g. DTW-based) instead of static-feature clustering, to more directly capture trajectory shape.
- Add climate/policy covariates (deforestation-driver datasets, protected-area status) to move from explanatory to more causally-informed modeling.
- Deploy the emissions model behind a small API/dashboard for interactive "what-if" exploration by region.
