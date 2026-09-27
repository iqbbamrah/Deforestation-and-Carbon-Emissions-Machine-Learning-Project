# Deforestation Trajectories & Carbon Emissions

**Team (Group 5, Machine Learning course, University of Waterloo WATSPEED Data Science Certificate):** Gonzalo Vazquez, Iqbal Bamrah, Farnoush Attarzadeh

## Problem

Global forest-loss data is usually summarized as a single "total loss" number per country, which hides very different underlying patterns. A region losing forest steadily at a low rate looks nothing like one experiencing a sudden hotspot event, even if their totals are similar. This project asks two questions:

1. Can countries and subnational regions be grouped into meaningful clusters based on the *shape* of their deforestation trajectory over 22 years (2001–2022), not just the total?
2. Once loss patterns are understood, can forest-loss and carbon-stock variables predict a country's gross carbon emissions?

## Data

- **Source:** Global Forest Data 2001–2022 (Kaggle): 20,000+ rows across countries, states, provinces, and territories, with a year-2000 forest-extent baseline and 22 annual tree-cover-loss variables.
- **Files used** (all in [`Deforestation Data/`](<Deforestation Data>)):
  - `Country tree cover loss.csv` (1,888 rows)
  - `Subnational 1 tree cover loss.csv` (28,000 rows)
  - `Country carbon data.csv` (1,888 rows): gross emissions, gross removals, net flux
  - `Subnational 1 carbon data.csv` (28,000 rows)
- Each region appears once per canopy-density threshold, and the analysis uses threshold 30.
- **Clustering dataset:** 3,029 region-level observations after cleaning.
- **Emissions modeling dataset:** 236-country table (deforestation + carbon merged at threshold 30, no missing values).

## Methodology

**Data preparation**
- Merged country- and subnational-level files into one dataset with a unified `region_name` key.
- Compared canopy-density thresholds (10/30/50) as a methodological decision rather than a simple filter, and selected **threshold 30** as the best balance between forest definition and clustering stability.
- Normalized annual tree-cover loss by each region's **2000 forest extent** (not land area), so trajectories are comparable across regions of very different size.
- Removed regions with zero baseline forest and applied a 1,000-hectare minimum, to avoid unstable percentage-based trajectories from tiny forest bases (3,460 → 3,029 regions).

**Clustering**
- Standardized features with `StandardScaler`.
- Reduced dimensionality with **PCA**, keeping 14 principal components (90.1% of variance).
- Ran **K-Means** (`n_init=10`), selecting **k = 3** using the elbow method and silhouette score.
- Validated by re-running K-Means on the full standardized feature set without PCA: the two solutions agree on 3,026 of 3,029 regions (99.9%), so PCA didn't distort the grouping.

**Predicting carbon emissions**
- Compared absolute (`cumulative_loss_ha`) vs. relative (`cumulative_loss_pct`) forest loss as predictors of gross emissions.
- Compared **Linear Regression** vs. **Random Forest Regressor** on country-level gross emissions.

**Tools:** Python, pandas, scikit-learn (`KMeans`, `PCA`, `StandardScaler`, `RandomForestRegressor`, `LinearRegression`), matplotlib.

## Results

**Three deforestation clusters:**

| Cluster | n | Countries with the most regions in the cluster |
|---|---|---|
| Low & Mostly Stable Deforestation | 2,217 | Russia, Puerto Rico, Philippines, Turkey, Macedonia |
| Moderate & Sustained Deforestation | 702 | Vietnam, Uganda, Ireland, Dominican Republic, Laos |
| High-Intensity Deforestation Hotspots | 110 | Portugal, Cambodia, Malaysia, Benin, Thailand |

A spike in forest loss appeared across all three clusters in 2016–2017, in line with documented global deforestation trends and El Niño-related wildfire activity that year.

**Predicting emissions:**

| Model | R² | MAE | RMSE |
|---|---|---|---|
| Linear Regression | 0.967 | 9,774,348 | 22,268,140 |
| **Random Forest** | **0.978** | **7,085,422** | **18,383,090** |

Absolute forest loss was far more informative than relative loss as a predictor of emissions (r = 0.895 vs. r = 0.073). Top Random Forest predictors: cumulative forest loss (ha), 2000 baseline carbon stock, recent loss, early loss, and 2000 forest extent. Forest gain and average carbon density per hectare contributed comparatively little.

## Key takeaways

- **Deforestation isn't one pattern.** Grouping by trajectory shape surfaces a small set of high-intensity hotspot regions that a "top N by total loss" ranking would blend in with moderate-loss regions.
- **The right representation depends on the question.** Relative loss was right for clustering (fair comparison across region sizes), but absolute loss was right for predicting emissions (emissions are measured in absolute terms).
- **The results are explanatory, not causal.** This identifies strong signals, not a policy-ready causal model.

## How to run

The notebook was built for Google Colab and asks you to upload its input files rather than reading them from a path:

1. Open [`Jupyter Notebook/Deforestation and Carbon Emissions.ipynb`](<Jupyter Notebook/Deforestation and Carbon Emissions.ipynb>) in Google Colab.
2. At the first upload prompt, select `Country tree cover loss.csv` and `Subnational 1 tree cover loss.csv` from `Deforestation Data/`.
3. Run the cells in order. At the second upload prompt (the carbon-emissions section), select `Country carbon data.csv` and `Subnational 1 carbon data.csv`.

## Repo structure

```
├── Deforestation Data/        # the four input CSVs (country + subnational, tree cover loss + carbon)
├── Jupyter Notebook/
│   └── Deforestation and Carbon Emissions.ipynb
├── Report/
│   └── Team 5 Deforestation and Carbon Emissions.pdf
└── README.md
```
