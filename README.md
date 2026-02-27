# Automated Monitoring of Deforestation in Ucayali (Peru) using LandTrendr  
**Patterns, uncertainty, and conservation applications (2001–2021)**

This project explores how **LandTrendr-derived Landsat time-series change metrics** can support **automated monitoring of deforestation** in the Peruvian Amazon, using a case study area in **Ucayali (Padre Abad province)**.

It focuses on:
- Translating LandTrendr segment metrics into decision-ready indicators
- Comparing “deforested” vs “non-deforested” pixels using a reference mask (Hansen Global Forest Change)
- Understanding uncertainty drivers (dispersion, overlap, outliers) that complicate classification

---

## Context (why this matters)

Deforestation is a major driver of biodiversity loss and greenhouse-gas emissions. 

<img src="https://github.com/user-attachments/assets/318088e9-d345-449c-9b17-3b6d20103cee" width="450">

Peru contains a significant share of the Amazon forest, and **Ucayali** is among the regions most affected by forest loss. In this case study, the analysis covers **2001–2021** and supports the broader goal of building monitoring pipelines for conservation, MRV, and early-warning systems.

---

## Study area

- Region: **Ucayali, Peru** (Padre Abad province)
- Period analysed: **2001–2021**
- Spatial framework: LandTrendr outputs aggregated per pixel for the strongest negative segment (“Loss Big-Delta”) within the analysis window.

<img src="assets/img/Figura1_a.png" alt="Study area" width="450">

---

## Data sources

### 1) Spectral change metrics (LandTrendr)
LandTrendr fits a piecewise linear model to Landsat time-series, capturing abrupt disturbances and gradual trends while reducing noise (e.g., cloud contamination, sensor artefacts).

<img src="assets/img/Figura3_ModConceptual_LandTrend.png" alt="Conceptual Model" width="450">

For each pixel, the project uses the **largest negative segment** (“Loss Big-Delta”) between 2001 and 2021 and exports results as GeoTIFF (multi-band).

Key derived attributes (per pixel):
- **yod** — year of disturbance
- **magnitude** — change magnitude between segment vertices
- **duration** — segment duration (years)
- **preval** — pre-change spectral value
- **rate** — magnitude / duration
- **DSNR** — signal-to-noise ratio proxy

The segmentation index used is **TCW (Tasseled Cap Wetness)**, which is sensitive to vegetation cover change.

### 2) Reference mask (deforested vs non-deforested)
A boolean reference mask is derived from **Hansen Global Forest Change (GFC v1.11)** for the 2001–2021 period, used to compare distributions and validate separability.

---

## Methods (high level)

1. **Load LandTrendr raster outputs** (Loss Big-Delta) and the **reference mask**
2. **Build a per-pixel dataframe** from raster bands (predictors + label)
3. **EDA (univariate + bivariate + multivariate)**:
   - distribution analysis (variance, outliers)
   - correlation checks / multicollinearity awareness
   - class comparisons (deforested vs non-deforested)
4. **Statistical testing**:
   - **Mann–Whitney U** tests for distribution differences across classes

---

## Key findings (summary)

### 1) Deforestation is concentrated in time and shows peak “loss years” (2001–2021)
**What we see:** The annual pattern of deforestation is not uniform; loss clusters in specific years, suggesting episodic disturbances (e.g., expansion fronts, infrastructure effects, commodity cycles, enforcement shifts).

— *Deforested area by year of disturbance (YOD)*
<img src="outputs/figures/areas_deforestadas_yod.png" alt="Annual pattern" width="450">

Results of temporal cluster analysis are actionable for monitoring (early warning), attribution work, and prioritising field verification.

---

### 2) Deforested pixels show stronger change signals (magnitude / rate / DSNR), but overlap remains
**What we see:** Compared with non-deforested pixels, deforested pixels tend to exhibit higher **magnitude** and **rate of change**, as well as higher **DSNR** values (signal-to-noise proxy). The boxplots show clear upward shifts in central tendency and dispersion for these variables in the deforested class, indicating stronger and more abrupt spectral responses. 

— *Magnitude, rate, DSNR & preval distribution in deforested and non-deforeted classes*
<img src="outputs/figures/areas_deforestadas_boxplot.png" alt="Boxplot" width="900">

However, the **class distributions overlap substantially**, and the scatterplot (Magnitude vs. DSNR) reveals a wide shared region where both classes coexist. This suggests that simple threshold-based rules are unlikely to separate classes reliably.

— *Magnitude vs DSNR (separability + overlap)*
<img src="outputs/figures/scatterplot_mag_dsnr.png" alt="Scatterplotn" width="450">

In addition, **the dataset is strongly imbalanced** (~93% non-deforested vs. ~7% deforested), which poses an additional challenge for classification. The dominance of the majority class may bias model training and inflate overall accuracy while masking poor detection of deforestation. Therefore, although magnitude, rate and DSNR are informative predictors, robust modelling approaches—ideally incorporating class weighting, resampling strategies, and possibly additional contextual or temporal predictors—are required rather than single-rule classification.

— *Class comparison (deforested vs non-deforested)*
<img src="outputs/figures/areas_deforestadas_vs_no_deforestadas.png" alt="ClassComparison" width="450">


**Why it matters:** These features are informative, but you’ll need robust modelling (or additional predictors) rather than single-rule classification.

---

### 3) Dispersion and outliers are a key challenge for classification and model stability
**What we see:** Feature distributions exhibit strong dispersion and right-skewness in both classes, with heavy tails particularly evident for magnitude, rate of change and DSNR. While extreme values are more frequent in the deforested class, substantial variability is also present in non-deforested pixels. This pattern likely reflects mixed pixels in fragmented landscapes, heterogeneous baseline conditions, residual time-series noise and potential reference label uncertainty.

— *Distribution of key variables*
<img src="outputs/figures/variables_distribucion.png" alt="Distribution" width="600">


**Why it matters:** Such heavy-tailed behaviour poses challenges for classification and model stability. Extreme observations can disproportionately influence decision boundaries and reduce generalisation performance, particularly for linear or parametric models. Therefore, distribution-aware preprocessing (e.g., logarithmic transformations, robust scaling, or tail control strategies) and robust modelling approaches become essential to prevent overfitting to rare but high-magnitude events.

---

### 4) Some predictors are correlated — multicollinearity needs to be managed
**What we see:** The correlation matrix indicates strong linear relationships among several LandTrendr-derived metrics, particularly magnitude, rate of change and DSNR (r ≈ 0.86–0.93). These high correlations suggest that these variables capture closely related aspects of disturbance intensity rather than independent dimensions of change. While this redundancy may not severely affect predictive performance in non-parametric models, it can inflate coefficient variance and reduce interpretability in regression-based approaches. 

 — *Correlation matrix*
 <img src="outputs/figures/matriz_correlacion.png" alt="Correlation" width="450">

**Why it matters:** Feature selection, regularisation or dimensionality reduction techniques should be considered to improve model stability and ensure clearer attribution of predictive effects.

---

### 5) Practical implication: improvements should focus on transformations + feature engineering + additional context
**What we see:** The magnitude distributions demonstrate incomplete class separability: although deforested pixels show a clear shift toward higher values, substantial overlap persists across the central value range. This indicates that separability is statistical rather than threshold-based, with most observations concentrated in the ambiguous overlap region rather than in the extreme tails. 

— *shows incomplete separability*
 <img src="outputs/figures/mag_histogram.png" alt="Separability" width="450">

**Why it matters:** Performance improvements are unlikely to arise solely from increasing model complexity. Instead, emphasis should be placed on distribution-aware transformations, robust outlier handling, feature engineering that captures disturbance dynamics more explicitly, and incorporation of spatial or contextual predictors. Additionally, careful validation of the reference mask and mitigation of boundary effects may reduce artificial overlap and improve signal clarity.

---

## Recommended improvements (next iteration)

Based on the observed overlap/dispersion:
- Consider **log transforms** or other monotonic transforms to improve class separation
- Evaluate **outlier handling** (thresholding / robust estimators) and its impact on performance
- Explore **multi-class severity bins** (e.g., magnitude-based loss intensity) instead of a strict binary split
- Add complementary datasets (field data, additional EO layers) to reduce ambiguity and improve generalisation

---

## Start here (curated notebooks)

- `memoria.ipynb` — descriptive report (context, data, methods, findings)
- `notebooks/main_ROI1.ipynb` — main pipeline for ROI1 (minimal data required)
- `notebooks/main_ROI2.ipynb` — main pipeline for ROI2 (minimal data required)

---

## Data availability & reproducibility

This repo is intentionally kept lightweight. Large datasets should not be committed to Git history.

If you plan to run `main_ROI1.ipynb` / `main_ROI2.ipynb`, place the minimal required GeoTIFFs in:

- `src/data/`

The `.gitignore` is configured to:
- ignore all data by default
- allow only the minimal ROI files required by the two main notebooks (ROI1/ROI2)

---

## Repository structure

```text
deforestation-monitoring-insights/
├── README.md
├── .gitignore
├── memoria.ipynb                 # report notebook
├── notebooks/                    # curated notebooks
│   ├── main_ROI1.ipynb
│   └── main_ROI2.ipynb
├── outputs/
│   └── figures/                  # charts used in the report
├── assets/                       # optional additional assets
├── docs/                         # optional exported report/slides
└── src/
    ├── data/                     # local data (mostly ignored by git)
    └── utils/                    # helper functions (funciones.py, etc.)



