# Seoul Real Estate Investment Analysis (2018–2024)

A data-driven framework for recommending district (gu) × building-type
investment targets in Seoul, tailored to three investor risk profiles —
Conservative, Balanced, and Aggressive — and stress-tested with
statistical significance testing, regression-based decomposition, and
out-of-sample validation.

> Built as a solo analytical project within a 4-person team submission
> (see [Author & Contribution](#author--contribution)).

---

## Project Overview

| Item | Details |
|------|---------|
| **Period** | 2018 – 2024 (7 years) |
| **Scope** | 25 districts (gu) × 4 building types in Seoul |
| **Data** | Seoul Open Data Plaza — Real Estate Transaction Records (~800K transactions) |
| **Approach** | Z-score composite scoring, validated with ANOVA, Welch's t-test, OLS regression, and a temporal holdout test |
| **Deliverable** | TOP 3 district × building-type recommendations per investor profile, plus a district-level drill-down to specific dong (neighborhoods) |

---

## Problem Statement

Seoul's real estate market shows large price variation across districts,
but most public analyses stop at a single metric — price. This project
asks three questions and, critically, checks whether the answers hold up
under statistical scrutiny and out-of-sample testing rather than stopping
at a descriptive ranking:

- Which districts offer the best risk-adjusted profile for each investor type?
- Is a district's low average price a genuine location discount, or an
  artifact of what's built there (older stock, smaller units)?
- Would this scoring framework, built on 2018–2021 data, have actually
  outperformed the market in 2022–2024?

---

## Analysis Framework

Three metrics are computed per district, standardized via Z-score, and
combined with investor-profile-specific weights:

| Metric | Definition | Preferred Direction |
|--------|-----------|---------------------|
| **Growth** | Average annual price-per-pyeong growth rate, 2018–2024 | Higher ↑ |
| **Price** | Current price-per-pyeong relative to the Seoul average | Lower (undervalued) ↓ |
| **Volatility** | Standard deviation of annual growth rates | Lower (stable) ↓ |

### Investor Profile Weights

| Profile | Growth | Price | Volatility | Total |
|---------|--------|-------|------------|-------|
| Conservative | 0.5 | 1.0 | **1.5** | 3.0 |
| Balanced | 1.0 | 1.0 | 1.0 | 3.0 |
| Aggressive | **1.5** | 1.0 | 0.5 | 3.0 |

All profiles sum to 3.0, so scores stay on a comparable scale across profiles.

---

## Statistical Validation

Descriptive rankings alone can't distinguish a real pattern from noise.
Three checks were added on top of the scoring framework:

### 1. ANOVA — Is the district effect real, or just detectable?

| Test | Result |
|------|--------|
| F-statistic | 7,123.67 (p ≈ 0) |
| Eta-squared (effect size) | **0.186** — district explains 18.6% of price variance |

With ~750K transactions, a p-value near zero is close to guaranteed even
for small differences — so effect size, not just significance, is the
number that matters here. 18.6% says district is a real but partial
driver of price; the remaining variance comes from building type, floor,
age, and factors outside this dataset (school zones, transit access,
building brand).

### 2. Welch's t-test — TOP3 vs. the rest

Each profile's TOP3 districts were tested against the remaining 22 using
Welch's t-test (robust to unequal sample sizes across districts). All
three profiles showed a statistically significant price gap (p ≈ 0),
confirming the TOP3 picks aren't indistinguishable from the rest of the
market.

### 3. OLS Regression — Decomposing what actually drives price

```
log(price_per_pyeong) ~ district + building_type + building_age + year + floor
```
- **R² = 0.543** (703,065 transactions)
- Building age: **–1.33% per year** (p ≈ 0) — older stock trades at a
  discount, net of location/type/floor, not a premium
- District effects range from **–2.8% (Yongsan-gu)** to **–57.3%
  (Dobong-gu)** relative to the priciest baseline district

**Why this sometimes disagrees with the Z-score ranking, and why that's
useful, not a bug:** the Z-score framework asks *"is this district's
current average price low?"* The regression asks a sharper question:
*"controlling for building type, floor, and age, is the location itself
cheap?"* Dobong-gu ranks well on the first (low raw average → Conservative
TOP2) but is the single lowest on the second — its low average price
is driven mainly by an older, lower-rise housing mix, not necessarily an
undervalued location. Both numbers are correct; they answer different
investment questions, and the gap between them is itself informative.

*(An earlier pass also included land area as a regressor; it showed a
statistically insignificant, near-zero coefficient (p = 0.977) while its
~20% missingness silently cost a quarter of the usable sample. It was
dropped, which recovered ~150K rows without any loss of explanatory power.)*

![Regression-adjusted district price premium/discount](assets/regression_district_effects.png)

### 4. Temporal Holdout Validation — Does this framework predict forward?

Every result above describes the past. The framework's real claim is
forward-looking, so it was tested directly: TOP3 rankings were computed
using **only 2018–2021 data**, then checked against actual district-level
growth in **2022–2024** — years the scoring never saw.

| Profile | TOP3 (from 2018–2021 data) | Actual growth, 2022–2024 | Seoul average |
|---------|---------------------------|--------------------------|---------------|
| Conservative | Seongbuk-gu, Jungnang-gu, Guro-gu | 7.29% | 9.53% |
| Balanced | Seongbuk-gu, Jungnang-gu, Geumcheon-gu | 5.61% | 9.53% |
| Aggressive | Seongbuk-gu, Nowon-gu, Jungnang-gu | 5.32% | 9.53% |

**All three profiles underperformed the Seoul average in the holdout window.**
This is reported as-is rather than adjusted away: 2022–2024 gains
concentrated in already-premium districts (Yongsan-gu, Seongdong-gu,
Gangnam-gu all posted the largest absolute price increases in the
district × year heatmap below) — a re-rating pattern where existing
premium locations pulled further ahead, rather than the undervalued,
low-volatility districts this framework is built to favor. It's a
legitimate limitation of any static, backward-looking scoring system
during a regime shift, not a data error, and it's the honest answer to
the question this project set out to ask.

![Holdout validation — training-window TOP3 actual growth vs. Seoul average](assets/holdout_validation.png)

Weight robustness was checked separately from forward-looking validity —
sensitivity analysis confirms the TOP3 ranking is stable across nearby
weight choices (±0.2), so the holdout result above reflects a genuine
market regime shift rather than the framework being fragile to its own
weight settings:

![TOP5 ranking stability across weight scenarios](assets/sensitivity_analysis.png)

---

## Key Findings

### Seoul Market Trends (2018–2024)
- Average price per pyeong rose from ~2,371 (10k KRW) in 2018 to
  ~4,126 in 2024, with a clear inflection in 2022
- Transaction volume fell **65%** from 2020 (174,126) to 2022 (61,399),
  coinciding with the interest-rate hiking cycle
- 2022 posted the only negative YoY growth (–1.9%), followed by a sharp
  rebound (+19.6% in 2023) — volume stayed depressed even as price
  rebounded, consistent with thinner, higher-end-skewed demand rather
  than a broad recovery

![Seoul Real Estate Market Overview — price, price per pyeong, transaction volume, and YoY growth by year](assets/market_overview.png)

![District × year price-per-pyeong heatmap](assets/district_year_heatmap.png)

### Investment Recommendations (District × Building Type)

| Profile | TOP 1 | TOP 2 | TOP 3 |
|---------|-------|-------|-------|
| Conservative | Jungnang-gu (Row house) | Gwanak-gu (Single-family) | Dobong-gu (Row house) |
| Balanced | Jungnang-gu (Row house) | Eunpyeong-gu (Row house) | Geumcheon-gu (Row house) |
| Aggressive | Jungnang-gu (Officetel) | Geumcheon-gu (Officetel) | Jungnang-gu (Row house) |

![TOP5 districts by investor profile](assets/top5_districts_by_profile.png)

### Dong-Level Drill-Down

Within each profile's TOP3 districts, a further pass identifies the
specific dong (neighborhood) driving the recommendation — e.g. Jungnang-gu's
signal is concentrated in **Myeonmok-dong** and **Junghwa-dong** across
all three profiles. Dong-level estimates rest on far fewer transactions
than district-level ones, so any dong with fewer than 100 transactions
or data missing in more than 2 of the 7 years is explicitly flagged as
low-confidence rather than silently included.

### So What?

- **Jungnang-gu is the only district to rank in the TOP3 across all
  three risk profiles** — it combines above-average growth (9.6%),
  below-average volatility (5.0), and a 22% discount to the Seoul
  average, a combination none of the other 24 districts match on all
  three metrics simultaneously.
- Conservative portfolios consistently favor low-volatility, undervalued
  outer districts — a capital-preservation profile.
- The regression shows this "undervaluation" is partly a housing-stock
  composition effect, not purely a location discount — worth flagging
  to anyone using this framework for an actual allocation decision.
- The holdout validation is the most important finding in this project:
  a scoring system tuned to historical patterns does not automatically
  generalize through a market regime change, and that limitation is
  worth stating plainly rather than optimizing the backtest away.

### Business Impact

For a conservative investor allocating toward Seoul residential
property, this analysis narrows a 25-district, 4-building-type search
space (100 combinations) down to a defensible shortlist — row houses in
Jungnang-gu's Myeonmok-dong and Junghwa-dong — in a fraction of the time
a manual district-by-district comparison would take, while surfacing
two caveats a purely descriptive ranking would miss:

1. **Composition risk** — the regression indicates part of Jungnang-gu's
   apparent discount reflects its older, lower-rise housing stock rather
   than pure location undervaluation, so due diligence should separate
   "cheap because underrated" from "cheap because of what's built there."
2. **Regime risk** — the same framework, built on pre-2022 data, would
   have underperformed the broader Seoul market during the most recent
   rate-hiking and re-rating cycle. A user should treat this as one input
   alongside current macro conditions (rate trajectory, jeonse-to-price
   ratio), not a standalone buy signal.

This framing — a fast, defensible shortlist plus explicit, quantified
caveats — is the practical output a static scoring model can responsibly
deliver, and the honest boundary of what it can't.

---

## Project Structure

```
seoul-real-estate-analysis/
├── data/                                    # Yearly CSVs (not tracked in git)
│   ├── 2018.csv
│   ├── ...
│   └── 2024.csv
├── notebooks/
│   └── seoul_real_estate_analysis.ipynb     # Full pipeline, single notebook
├── assets/                                  # Exported chart images used in this README
└── README.md
```

### Notebook Contents

| Section | Content |
|---------|---------|
| Data Loading & Cleaning | Merge 7 yearly CSVs, drop cancellations, validate `year_built`, engineer `price_per_pyeong` / `building_age` |
| Market-Wide EDA | Raw price distribution, building-type breakdown, annual trend dashboard |
| Outlier Treatment | IQR-based removal for cross-district comparison (raw data preserved for trend analysis) |
| District Metrics | Growth, volatility, relative price ratio per district |
| **ANOVA** | District effect significance + eta-squared |
| Z-Score Scoring | Investor-profile composite scores, metric independence check |
| **Welch's t-test** | TOP3 vs. rest significance test, per profile |
| **OLS Regression** | Price determinant decomposition (district / building type / age / floor / year) |
| Sensitivity Analysis | TOP3 stability across 7 weight scenarios (±0.2 range) |
| **Temporal Holdout Validation** | Train on 2018–2021, test against actual 2022–2024 outcomes |
| Building-Type Drill-Down | District × building-type re-scoring within each profile's TOP3 |
| Dong-Level Drill-Down | Neighborhood-level recommendation with confidence flagging |
| Final Recommendations | TOP3 summary table per profile |

---

## Visualizations

All charts below are referenced inline above; the full set, with the
filename each should be saved as in `assets/`:

| Chart | Asset filename |
|---|---|
| Market trend dashboard (price, price/pyeong, volume, YoY growth) | `assets/market_overview.png` |
| District × year price heatmap | `assets/district_year_heatmap.png` |
| TOP5 districts by investor profile | `assets/top5_districts_by_profile.png` |
| Sensitivity analysis line chart (weight robustness) | `assets/sensitivity_analysis.png` |
| Regression-adjusted district premium/discount | `assets/regression_district_effects.png` |
| Holdout validation (TOP3 actual growth vs. Seoul average) | `assets/holdout_validation.png` |
| District comparison bars (growth / volatility / relative price) | `assets/district_metric_bars.png` |
| Radar chart — metric emphasis by profile | `assets/profile_radar.png` |
| Building-type and dong-level score charts | `assets/bldg_type_scores.png`, `assets/dong_level_scores.png` |

---

## What Makes This Project Different

- Goes beyond descriptive ranking — every headline result is checked
  against a formal significance test, an alternative model specification,
  or an out-of-sample validation before being reported
- Reports a validation failure (the holdout result) instead of adjusting
  the framework until it passes — the limitation is treated as a finding
- Decomposes *why* a district looks cheap (location vs. housing-stock
  composition) rather than stopping at "cheap"
- Drills down two levels of geographic resolution (gu → dong) with
  explicit confidence flagging rather than presenting neighborhood-level
  numbers at the same certainty as district-level ones

---

## Limitations & Future Work

| Limitation | Potential Improvement |
|-----------|----------------------|
| No macroeconomic variables | Add interest rate and jeonse-to-price ratio data — directly relevant given the 2022 rate-hike inflection observed in this dataset |
| Static scoring framework, shown to underperform in the 2022–2024 holdout | Explore a regime-aware model (e.g. separate scoring under rising vs. falling rate environments) |
| No forward-looking indicators | Incorporate leading indicators (construction permits, population inflow, transit/redevelopment plans) |
| Dong-level estimates thin for smaller neighborhoods | Extend the transaction window or apply shrinkage estimation for low-volume dong |
| Single train/holdout split | Not enough years in this dataset for a multi-fold backtest; a longer historical window would allow one |

---

## Author & Contribution

**Joshua Kim**
Data Analyst | Marketing · E-commerce · Business Analytics

This was completed as a 4-person team project; all data analysis, the
scoring methodology, statistical validation, and code in this repository
are my individual work. Team contribution beyond the analysis was limited
to the presentation deck.

📧 Jkim43844@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/joshua-kim-87b478263/)

---

## License

This project is for portfolio and educational purposes.
Data source: [Seoul Open Data Plaza](https://data.seoul.go.kr/) (Public Domain)
