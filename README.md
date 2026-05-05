# 🏠 House Prices and Market Speed: Regression Analysis of U.S. Housing Data

**W203 Statistics for Data Science — Final Project | UC Berkeley MIDS Program**  
*Group 3: Naik, Sarvepalli, Habib | Summer 2024*

---

## 📋 Table of Contents
- [Problem Statement](#problem-statement)
- [Dataset](#dataset)
- [Approach & Methodology](#approach--methodology)
- [Results](#results)
- [Impact](#impact)
- [Limitations & Challenges](#limitations--challenges)
- [Key Learnings & Findings](#key-learnings--findings)

---

## Problem Statement

**Research Question:**  
*What is the relationship between average house values and the average number of days it takes for a home to go from listed to pending sale across regions throughout the United States?*

The U.S. housing market is a key indicator of consumer financial health and broader economic strength. Even as mortgage rates rose sharply in recent years, demand remained resilient — raising the question of whether higher-priced homes attract buyers faster or slower. Understanding the relationship between home prices and market speed (days to pending) has real value for buyers, sellers, and real estate professionals trying to make informed pricing and timing decisions.

This is a **purely descriptive** regression analysis — we do not make causal claims. We describe the statistical relationship as it exists in the data.

---

## Dataset

| Property | Detail |
|---|---|
| **Source** | [Zillow Research Data](https://www.zillow.com/research/data/) |
| **Type** | Observational, cross-sectional |
| **Time Period** | June 2024 (single cross-section) |
| **Unit of Observation** | U.S. metropolitan region |
| **Initial Rows** | 674 regions |
| **Final Rows After Cleaning** | 672 regions |

**Three raw datasets were merged:**
1. `Metro_zhvi_uc_sfrcondo_tier_0.33_0.67_sm_sa_month.csv` — Zillow Home Value Index (ZHVI): seasonally adjusted, smoothed typical home value for single-family homes/condos per region
2. `Metro_mean_doz_pending_uc_sfrcondo_sm_month.csv` — Mean Days to Pending: average days for a listing to go from open to pending status
3. `Metro_mean_listings_price_cut_amt_uc_sfrcondo_sm_month.csv` — Mean Price Cut Amount (collected but not used in final models)

**Key Variables:**
- **X (Feature):** `HomeValue` — Zillow Home Value Index in USD
- **Y (Outcome):** `DaysPending` — Average days for a listing to go from open to pending sale

**Train/Test Split:**
| Set | N | % |
|---|---|---|
| Exploration Set | 201 | 30% |
| Confirmation Set | 471 | 70% |

> All modeling decisions were made on the exploration set. All reported results are from the confirmation set.

---

## Approach & Methodology

### Data Wrangling
- Merged three Zillow datasets on `RegionID` (unique region identifier)
- Renamed columns systematically with suffixes (`DaysPending`, `HomeValue`, `PriceCut`) to avoid collision
- Removed null values and empty state/region entries
- Subsetted to June 2024 data for a clean cross-section
- Exported cleaned dataset to CSV for reproducibility

### Exploratory Data Analysis
- Plotted distributions of `HomeValue` and `DaysPending` — both showed heavy right skew
- Scatter plots revealed non-linearity in the raw (level-level) relationship
- Tested four functional forms on the exploration set:
  - **Level-Level:** High standard error (1.047e-05), extreme outliers visible in residuals
  - **Log-Level:** Non-constant variance remained in the feature variable
  - **Level-Log:** Gives absolute (not proportional) effect; rejected for interpretability
  - **Log-Log:** Best fit — normalized distributions, proportional interpretation (elasticity), residuals most randomly distributed
  - **Polynomial (squared):** Attempted but rejected for overcomplication and poor interpretability

### Final Model
```
log(DaysPending) ~ log(HomeValue)
```
- Applied **Heteroskedasticity-Consistent (HC1) robust standard errors** via `vcovHC` + `coeftest` from the `sandwich` and `lmtest` packages
- Used `stargazer` to format the regression table with both OLS and robust coefficient test columns
- Supplemented with a **Wald test** for model-level significance checking
- Confirmed results on the held-out **confirmation set (n = 471)**

### Large Sample Model Assumptions Evaluated
| Assumption | Assessment |
|---|---|
| IID | Cautiously assumed — regional averages reduce (but don't eliminate) geographic clustering concerns |
| Finite variance / No heavy tails | Satisfied after log transformation of both variables |
| No perfect multicollinearity | Not applicable (single feature model); confirmed by successful `lm()` execution |
| No perfect collinearity | Verified — single predictor model |

---

## Results

**Final Model:** `log(DaysPending) ~ log(HomeValue)` on confirmation set (n = 471)

| | OLS | Robust (HC1) |
|---|---|---|
| **log(HomeValue)** | -0.066 (SE: 0.042) | -0.066* (SE: 0.040) |
| **Constant** | 4.495*** (SE: 0.525) | 4.495*** (SE: 0.501) |
| **R²** | 0.005 | — |
| **Adjusted R²** | 0.003 | — |
| **F-statistic** | 2.509 (df = 1; 469) | — |

*Note: * p<0.10; ** p<0.05; *** p<0.01*

**Interpretation:**
- A **1% increase in Home Value** is associated with a **~0.07% decrease in Days to Pending**
- Direction: Higher-priced homes go pending *slightly faster*
- Practical example: A $50,000 price increase on a $100,000 home predicted to take 50 days to go pending would reduce that time by approximately **~4 days**
- The coefficient is **marginally significant at the 10% level** (robust SE), but the overall F-statistic (p > 0.05) indicates the model is **not statistically significant at conventional thresholds**
- **R² = 0.005** — Home Value alone explains less than 1% of variation in Days to Pending

**Conclusion:** We **fail to reject the null hypothesis**. There is a very weak negative relationship between home values and market speed, but it is not statistically significant. Other factors (local market conditions, seasonality, inventory levels, interest rate environment) are likely far more important drivers.

---

## Impact

- Provides a **data-driven baseline description** of the price-speed relationship in U.S. housing markets as of mid-2024
- Demonstrates that **home price alone is a poor predictor of market speed**, which is a useful and non-obvious finding for real estate professionals making pricing strategy decisions
- Highlights the need for **multivariate models** that incorporate inventory, regional economic conditions, or mortgage rate sensitivity to better explain market speed
- Establishes a **reproducible, well-documented analysis pipeline** using publicly available Zillow data that can be extended or updated with new time periods

---

## Limitations & Challenges

### Statistical Limitations
- **Low R² (0.005):** Home value explains virtually none of the variance in days to pending — the model is descriptive but has very low explanatory power
- **Marginal significance only:** The coefficient is significant only at p < 0.10, not the conventional p < 0.05 threshold; the F-statistic further confirms the model's overall non-significance
- **IID assumption:** Regional average data reduces but does not eliminate geographic clustering (e.g., Northeast metros may be correlated with each other). A proper treatment would require spatial regression or region fixed effects
- **Single time point:** Using only June 2024 limits generalizability; seasonal effects cannot be controlled for

### Data Limitations
- **Zillow coverage bias:** Zillow estimates 70% of listed homes are on their platform — the dataset is not the full population, and listing behavior may differ across platform and non-platform homes
- **Regional aggregation:** Using regional *averages* hides within-region heterogeneity. High-value and low-value homes in the same region are collapsed into a single point
- **No causal inference possible:** Cross-sectional, observational data — we cannot say higher prices *cause* faster sales
- **Omitted variable bias:** Key drivers like mortgage rates, local inventory levels, school quality, and economic conditions are absent from the model

### Technical Challenges
- Merging three datasets with inconsistent column naming conventions required systematic renaming functions
- Selecting the right functional form required iterating through four model types and evaluating residual plots for each
- Balancing the exploration/confirmation set split while maintaining sufficient sample size in both sets

---

## Key Learnings & Findings

1. **Log-log transformation is both statistically and economically justified** when variables span wide ranges (home values from ~$100K to $1.5M+) — it compresses outliers and enables elasticity interpretation
2. **Robust standard errors are essential** for real-world data that rarely satisfies homoskedasticity; the HC1 correction matters even when residual plots look passable
3. **Statistical significance ≠ practical significance** (and vice versa): A marginally significant coefficient with R² < 0.01 is a finding in itself — it tells us the relationship is weak and the model is incomplete
4. **Exploration/confirmation splitting disciplines the modeling process** — committing to functional form decisions before seeing the confirmation set prevents overfitting your narrative to the data
5. **Descriptive analysis requires honest framing:** Correctly framing the null result as meaningful (other factors matter more) is as important as finding a strong effect
6. **Data wrangling is often the hardest part:** ~50% of the R code in this project is preprocessing — merging, renaming, filtering, and ordering — before any modeling begins

---

**R Packages Used:**
`ggplot2`, `broom`, `sandwich`, `lmtest`, `magrittr`, `tidyr`, `dplyr`, `stargazer`, `caTools`

---

*Data sourced from [Zillow Research](https://www.zillow.com/research/data/) — ZHVI All Homes (SFR, Condo/Co-op, Smoothed, Seasonally Adjusted) & Mean Days to Pending (Smooth, All Homes, Monthly)*
