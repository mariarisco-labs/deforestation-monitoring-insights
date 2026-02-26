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

![image](https://github.com/user-attachments/assets/318088e9-d345-449c-9b17-3b6d20103cee)

Peru contains a significant share of the Amazon forest, and **Ucayali** is among the regions most affected by forest loss. In this case study, the analysis covers **2001–2021** and supports the broader goal of building monitoring pipelines for conservation, MRV, and early-warning systems.

---

## Study area

- Region: **Ucayali, Peru** (Padre Abad province)
- Period analysed: **2001–2021**
- Spatial framework: LandTrendr outputs aggregated per pixel for the strongest negative segment (“Loss Big-Delta”) within the analysis window.

![Study area](assets/img/Figura1_a.png)

---

## Data sources

### 1) Spectral change metrics (LandTrendr)
LandTrendr fits a piecewise linear model to Landsat time-series, capturing abrupt disturbances and gradual trends while reducing noise (e.g., cloud contamination, sensor artefacts).

![Study area](assets/img/Figura3_ModConceptual_LandTrend.png)

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
3. **EDA (univariate + bivariate)**:
   - distribution analysis (variance, outliers)
   - correlation checks / multicollinearity awareness
   - class comparisons (deforested vs non-deforested)
4. **Statistical testing**:
   - **Mann–Whitney U** tests for distribution differences across classes

---

## Key findings (summary)

- A large fraction of the study area is classified as deforested over 2001–2021, with a clear temporal pattern and peak years of loss.
- **magnitude**, **rate**, and **DSNR** tend to be higher in deforested pixels, while **preval** shows strong overlap between classes.
- Distributions show **high dispersion** and **outliers**, suggesting:
  - mixed land-cover mosaics and forest fragmentation
  - potential reference-mask errors (omission/commission) or noise-driven artefacts
- Statistical tests confirm **significant differences** for key variables, but overlap implies classification remains non-trivial without additional features or robust handling of outliers and transformations.

> Add 2–4 key figures here from `outputs/figures/` (examples below):
> - `areas_deforestadas_yod.png` (annual loss pattern)
> - `variables_distribucion.png` (feature distributions)
> - `matriz_correlacion.png` (correlation)
> - `areas_deforestadas_vs_no_deforestadas.png` (class comparison)
>
> Example usage:
> `![Annual pattern](outputs/figures/<figure_name>.png)`

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



