# Seoul Real Estate Investment Analysis (2018–2024)

![Python](https://img.shields.io/badge/Python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54) ![Pandas](https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white) ![statsmodels](https://img.shields.io/badge/statsmodels-8B0000?style=for-the-badge) ![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)

A framework for recommending district (gu) × building-type investment targets in Seoul, tailored to three investor risk profiles — Conservative, Balanced, Aggressive — checked against statistical tests, a regression model, and an out-of-sample validation, with an interactive dashboard for screening the results.

> Completed as part of a 4-person team project. All analysis, modeling, and code in this repository are individual work — see [Individual Contribution](#individual-contribution).

---

## At a glance

| | |
|---|---|
| **Goal** | Recommend district × building-type investment targets for three investor profiles, and test whether the scoring framework would actually have predicted forward |
| **Method** | Z-score composite scoring (growth, price, volatility, profile-weighted) with a two-level drill-down (district → dong) |
| **Validation** | ANOVA (district effect significance), Welch's t-test (TOP3 vs. rest), OLS regression (isolating location effect from housing-stock composition), 7-scenario weight-sensitivity check, and a temporal holdout (train 2018–2021, test against actual 2022–2024) |
| **Results** | Jungnang-gu tops every profile's shortlist; OLS (R²=0.543) shows Dobong-gu's "cheap" price is mostly housing-stock age, not location discount; the holdout **failed** — all three profiles underperformed the Seoul average in 2022–2024, reported as a genuine finding rather than adjusted away |
| **Tech** | Python, Pandas, statsmodels (OLS, ANOVA), SciPy (Welch's t-test); React/SVG for the screening dashboard |

---

## Live Dashboard

An interactive screening tool — switch investor profile, see the district/building-type/dong shortlist and every validation chart (holdout, regression effects, sensitivity) update live.

**[Open the dashboard →](#)** *(update this link once GitHub Pages is enabled — see Setup below)*

Until then: download `dashboard.html` from this repo and open it directly in a browser — it's a self-contained file, no server or build step needed.

---

## The Question

Seoul's real estate market varies a lot by district, but most public write-ups stop at comparing raw prices. This project asks three sharper questions instead:

1. Which districts fit each investor profile once growth, price, and volatility are all weighed together?
2. When a district looks cheap, is that a real location discount — or just a reflection of older, smaller housing stock?
3. Would this scoring approach, built on 2018–2021 data, have actually picked winners in 2022–2024?

---

## Scoring Framework

Each district gets three metrics, standardized to Z-scores so they're comparable, then combined with profile-specific weights.

| Metric | What it measures | Better direction |
|---|---|---|
| Growth | Avg. annual price/pyeong growth, 2018–2024 | Higher |
| Price | Current price/pyeong vs. Seoul average | Lower |
| Volatility | Std. dev. of annual growth | Lower |

| Profile | Growth | Price | Volatility |
|---|---|---|---|
| Conservative | 0.5 | 1.0 | 1.5 |
| Balanced | 1.0 | 1.0 | 1.0 |
| Aggressive | 1.5 | 1.0 | 0.5 |

Weights always sum to 3.0, so scores stay comparable across profiles.

---

## Statistical Validation

A descriptive ranking alone can't tell a real pattern from noise. Four checks were layered on top of the scoring framework.

**ANOVA — is the district effect real?**
F = 7,123.67, p ≈ 0, eta-squared = 0.186. With ~750K rows, near-zero p-values are almost automatic, so eta-squared is the number that matters: district explains about 19% of price variance. Real, but partial — building type, floor, age, and factors outside this data (school zones, transit, brand) make up the rest.

**Welch's t-test — TOP3 vs. everyone else**
Each profile's TOP3 districts were tested against the remaining 22. All three profiles came back significant (p ≈ 0) — the picks aren't indistinguishable from the rest of the market.

**OLS regression — what actually drives price?**
```
log(price_per_pyeong) ~ district + building_type + building_age + year + floor
```
R² = 0.543 on 703,065 transactions. Building age: −1.33% per year (p ≈ 0) — older stock is a discount, not a redevelopment premium. District effects range from −2.8% (Yongsan-gu) to −57.3% (Dobong-gu) relative to the priciest baseline district.

This sometimes disagrees with the Z-score ranking, and that's useful rather than a bug. The Z-score asks "is this district's average price low?" The regression asks a sharper version: "controlling for building type, floor, and age, is the *location* cheap?" Dobong-gu scores well on the first question (Conservative TOP2) but worst on the second — its low average price looks driven mainly by an older, lower-rise housing mix, not necessarily an undervalued location. Both numbers are correct; they just answer different questions.

*(Land area was tested as a regressor and dropped — its coefficient was statistically meaningless (p = 0.977), and its ~20% missingness was quietly costing a quarter of the usable sample. Removing it recovered ~150K rows with no loss of explanatory power.)*

<img width="765" height="685" alt="Regression-adjusted district price premium/discount" src="https://github.com/user-attachments/assets/bdf74767-8177-4638-9513-60e0ac07e4f8" />

**Temporal holdout — does this predict forward?**
Everything above describes the past. So the framework was tested directly: TOP3 rankings were built using only 2018–2021 data, then checked against what actually happened in 2022–2024.

| Profile | TOP3 (from 2018–2021 data) | Actual growth, 2022–2024 | Seoul average |
|---|---|---|---|
| Conservative | Seongbuk-gu, Jungnang-gu, Guro-gu | 7.29% | 9.53% |
| Balanced | Seongbuk-gu, Jungnang-gu, Geumcheon-gu | 5.61% | 9.53% |
| Aggressive | Seongbuk-gu, Nowon-gu, Jungnang-gu | 5.32% | 9.53% |

All three profiles underperformed the Seoul average. This is reported as-is, not adjusted away. 2022–2024 gains concentrated in already-premium districts (Yongsan-gu, Seongdong-gu, Gangnam-gu posted the largest absolute increases) — a re-rating pattern that favored existing premium locations over the undervalued, stable ones this framework targets. A static, backward-looking scoring system has a real blind spot during a regime shift, and that's the honest answer to question 3 above.

<img width="810" height="446" alt="Holdout validation: training-window TOP3 actual growth vs. Seoul average" src="https://github.com/user-attachments/assets/65aed280-bc9f-4a00-80d1-d8b13ba4e024" />

Weight choice wasn't the problem, separately: a sensitivity check across 7 weight scenarios (±0.2) shows the TOP3 ranking stays stable. The holdout miss reflects a genuine market shift, not a fragile model.

<img width="1010" height="390" alt="TOP5 ranking stability across weight scenarios" src="https://github.com/user-attachments/assets/78d4777d-50ca-4a28-84e3-8197eeb1b68b" />

---

## Market Trends (2018–2024)

- Price/pyeong rose from ~2,371 (10k KRW) in 2018 to ~4,126 in 2024, with a clear break in 2022
- Transaction volume fell 65% from 2020 (174,126) to 2022 (61,399), tracking the rate-hiking cycle
- 2022 was the only year with negative YoY growth (−1.9%), followed by a sharp rebound (+19.6% in 2023) — volume stayed low even as price rebounded, pointing to thinner, higher-end-skewed demand rather than a broad recovery

<img width="1010" height="633" alt="Market overview: price, price/pyeong, volume, and YoY growth by year" src="https://github.com/user-attachments/assets/72b02f87-b962-48c6-b4f9-48ab32c5bf9c" />

<img width="798" height="645" alt="District x year price-per-pyeong heatmap" src="https://github.com/user-attachments/assets/dc0ef8e6-9f28-4d3c-be5f-155d215f4eff" />

---

## Recommendations

The table below is the final building-type drill-down: within each profile's TOP3 districts, building types are re-scored to find the best district × building-type combination.

| Profile | TOP 1 | TOP 2 | TOP 3 |
|---|---|---|---|
| Conservative | Jungnang-gu (Row house) | Gwanak-gu (Single-family) | Dobong-gu (Row house) |
| Balanced | Jungnang-gu (Row house) | Eunpyeong-gu (Row house) | Geumcheon-gu (Row house) |
| Aggressive | Jungnang-gu (Officetel) | Geumcheon-gu (Officetel) | Jungnang-gu (Row house) |

This builds on the plain district-level ranking (TOP5 districts by score, before building type is factored in):

<img width="999" height="529" alt="TOP5 districts by investor profile, district-level score only" src="https://github.com/user-attachments/assets/cb4702e5-cdc4-44f6-94ed-d0317f0909f8" />

**Dong-level drill-down.** A separate pass narrows each profile's TOP3 *districts* down to the specific dong (neighborhood) driving the signal — not a further split of the building-type table above, but an independent re-scoring at the neighborhood level. Jungnang-gu's signal comes mainly from Myeonmok-dong and Junghwa-dong across all three profiles. Dong-level estimates rest on far fewer transactions than district-level ones, so any dong under 100 transactions, or missing data in more than 2 of the 7 years, is flagged low-confidence (shown as gray bars) rather than folded in silently — for example, Seodaemun-gu's Daesin-dong shows an outlier score under the Aggressive profile driven by fewer than 100 transactions, and should be read as noise, not signal.

<img width="1008" height="365" alt="TOP5 dongs within each profile's TOP3 districts, with low-confidence dongs shown in gray" src="https://github.com/user-attachments/assets/d4f0ba3d-615f-4c49-a1b9-b53b071b8887" />

---

## So What

- Jungnang-gu is the only district in every profile's TOP3 — it's simultaneously above-average growth (9.6%), below-average volatility (5.0), and 22% cheaper than the Seoul average. No other district hits all three at once.
- Conservative portfolios consistently land on low-volatility, undervalued outer districts — a capital-preservation pattern.
- Part of that "undervaluation" is a housing-stock composition effect, not a pure location discount (see the regression above) — worth knowing before treating it as a buy signal.
- The holdout miss is the most important result in this project: a scoring system tuned to historical patterns doesn't automatically carry through a regime change, and that's worth saying plainly.

## Business Impact

For a conservative investor, this analysis narrows a 100-combination search space (25 districts × 4 building types) down to a specific, defensible shortlist — row houses in Jungnang-gu's Myeonmok-dong and Junghwa-dong — far faster than a manual district-by-district comparison, while flagging two things a plain ranking would miss:

- **Composition risk.** Jungnang-gu's discount is partly explained by older, lower-rise housing stock, not pure location undervaluation. Due diligence should separate "underrated" from "cheap because of what's built there."
- **Regime risk.** The same framework, trained on pre-2022 data, would have underperformed the broader market in the most recent cycle. Best used as one input alongside current rate and jeonse-to-price conditions — not a standalone buy signal.

A fast shortlist plus explicit, quantified caveats is what a static scoring model can responsibly deliver — and the honest edge of what it can't.

---

## Repo Structure

```
Seoul-Real-Estate-Investment-Strategy/
├── data/                                  # yearly CSVs, not tracked in git
│   ├── 2018.csv
│   ├── ...
│   └── 2024.csv
├── notebooks/
│   └── Seoul_Real_Estate_Investment_Project.ipynb   # full pipeline, single notebook
├── dashboard.html                         # standalone interactive screening dashboard
├── assets/                                # chart images referenced in this README
└── README.md
```

**Notebook sections**

| Section | Content |
|---|---|
| Data loading & cleaning | Merge 7 yearly CSVs, drop cancellations, validate `year_built`, engineer `price_per_pyeong` / `building_age` |
| Market-wide EDA | Raw price distribution, building-type breakdown, annual trend dashboard |
| Outlier treatment | IQR removal for cross-district comparison (raw data kept for trend analysis) |
| District metrics | Growth, volatility, relative price ratio per district |
| ANOVA | District effect significance + eta-squared |
| Z-score scoring | Composite scores per profile, metric independence check |
| Welch's t-test | TOP3 vs. rest significance, per profile |
| OLS regression | Price determinant decomposition |
| Sensitivity analysis | TOP3 stability across 7 weight scenarios |
| Temporal holdout | Train on 2018–2021, test against actual 2022–2024 |
| Building-type drill-down | Re-scoring within each profile's TOP3 |
| Dong-level drill-down | Neighborhood recommendation, confidence-flagged |
| Final summary | TOP3 table per profile |

---

## Setup

**Notebook:**
```bash
jupyter notebook Seoul_Real_Estate_Investment_Project.ipynb
```

**Dashboard:** download `dashboard.html` and open it in any browser — it loads React/Babel from CDN, so an internet connection is needed but no local server or build step. To make it link-accessible instead of download-only, enable **GitHub Pages** for this repo (Settings → Pages → Deploy from branch → root) — it'll be live at `https://jkim8436.github.io/Seoul-Real-Estate-Investment-Strategy/dashboard.html`.

---

## What's Different About This One

- Every headline result is checked against a significance test, an alternative model, or an out-of-sample test — not left as a descriptive ranking
- The holdout failure is reported, not tuned away — a limitation treated as a finding
- Decomposes *why* a district looks cheap (location vs. housing-stock mix) instead of stopping at "cheap"
- Drills down two levels of geography (gu → dong) with confidence flagging, instead of presenting neighborhood numbers at the same certainty as district-level ones
- The static analysis is paired with an interactive dashboard, so the validation results (holdout, regression, sensitivity) aren't just described — they can be explored per profile

---

## Limitations & Next Steps

| Limitation | Next step |
|---|---|
| No macro variables | Add interest rate, jeonse-to-price ratio — directly relevant given the 2022 inflection in this data |
| Static scoring, underperformed in the 2022–2024 holdout | Try a regime-aware model (separate scoring for rate-rising vs. rate-falling periods) |
| No forward-looking indicators | Add construction permits, population inflow, transit/redevelopment plans |
| Dong-level estimates thin for small neighborhoods | Extend the window, or use shrinkage estimation |
| Single train/holdout split | Not enough years for multiple folds; a longer history would allow it |

---

## Individual Contribution

Completed as a 4-person team project. All data analysis, scoring methodology, statistical validation, and code in this repository — including the dashboard — are the author's individual work; team contribution beyond the analysis was the presentation deck.

---

## License

Portfolio and educational use. Data: [Seoul Open Data Plaza](https://data.seoul.go.kr/) (Public Domain)
