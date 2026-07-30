# Seoul Real Estate Investment Analysis (2018–2024)

A framework for recommending district (gu) × building-type investment
targets in Seoul, tailored to three investor risk profiles — Conservative,
Balanced, Aggressive — and checked against statistical tests, a regression
model, and an out-of-sample validation.

> Completed as part of a 4-person team project. All analysis, modeling,
> and code in this repository are individual work — see
> [Author & Contribution](#author--contribution).

---

## Overview

| | |
|---|---|
| Period | 2018–2024 (7 years) |
| Scope | 25 districts × 4 building types |
| Data | Seoul Open Data Plaza, real estate transactions (~800K rows) |
| Method | Z-score composite scoring, tested with ANOVA, Welch's t-test, OLS regression, temporal holdout |
| Output | TOP3 district × building-type picks per profile, drilled down to specific dong (neighborhoods) |

---

## The Question

Seoul's real estate market varies a lot by district, but most public
write-ups stop at comparing raw prices. This project asks three sharper
questions instead:

1. Which districts fit each investor profile once growth, price, and
   volatility are all weighed together?
2. When a district looks cheap, is that a real location discount — or
   just a reflection of older, smaller housing stock?
3. Would this scoring approach, built on 2018–2021 data, have actually
   picked winners in 2022–2024?

---

## Scoring Framework

Each district gets three metrics, standardized to Z-scores so they're
comparable, then combined with profile-specific weights.

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

A descriptive ranking alone can't tell a real pattern from noise. Four
checks were layered on top of the scoring framework.

**ANOVA — is the district effect real?**
F = 7,123.67, p ≈ 0, eta-squared = 0.186. With ~750K rows, near-zero
p-values are almost automatic, so eta-squared is the number that matters:
district explains about 19% of price variance. Real, but partial —
building type, floor, age, and factors outside this data (school zones,
transit, brand) make up the rest.

**Welch's t-test — TOP3 vs. everyone else**
Each profile's TOP3 districts were tested against the remaining 22. All
three profiles came back significant (p ≈ 0) — the picks aren't
indistinguishable from the rest of the market.

**OLS regression — what actually drives price?**

```
log(price_per_pyeong) ~ district + building_type + building_age + year + floor
```

R² = 0.543 on 703,065 transactions. Building age: −1.33% per year
(p ≈ 0) — older stock is a discount, not a redevelopment premium.
District effects range from −2.8% (Yongsan-gu) to −57.3% (Dobong-gu)
relative to the priciest baseline district.

This sometimes disagrees with the Z-score ranking, and that's useful
rather than a bug. The Z-score asks "is this district's average price
low?" The regression asks a sharper version: "controlling for building
type, floor, and age, is the *location* cheap?" Dobong-gu scores well on
the first question (Conservative TOP2) but worst on the second — its low
average price looks driven mainly by an older, lower-rise housing mix,
not necessarily an undervalued location. Both numbers are correct; they
just answer different questions.

*(Land area was tested as a regressor and dropped — its coefficient was
statistically meaningless (p = 0.977), and its ~20% missingness was
quietly costing a quarter of the usable sample. Removing it recovered
~150K rows with no loss of explanatory power.)*

<!-- IMAGE: assets/regression_district_effects.png — Regression-adjusted district price premium/discount -->

**Temporal holdout — does this predict forward?**
Everything above describes the past. So the framework was tested
directly: TOP3 rankings were built using only 2018–2021 data, then
checked against what actually happened in 2022–2024.

| Profile | TOP3 (from 2018–2021 data) | Actual growth, 2022–2024 | Seoul average |
|---|---|---|---|
| Conservative | Seongbuk-gu, Jungnang-gu, Guro-gu | 7.29% | 9.53% |
| Balanced | Seongbuk-gu, Jungnang-gu, Geumcheon-gu | 5.61% | 9.53% |
| Aggressive | Seongbuk-gu, Nowon-gu, Jungnang-gu | 5.32% | 9.53% |

All three profiles underperformed the Seoul average. This is reported
as-is, not adjusted away. 2022–2024 gains concentrated in already-premium
districts (Yongsan-gu, Seongdong-gu, Gangnam-gu posted the largest
absolute increases) — a re-rating pattern that favored existing premium
locations over the undervalued, stable ones this framework targets. A
static, backward-looking scoring system has a real blind spot during a
regime shift, and that's the honest answer to question 3 above.

<!-- IMAGE: assets/holdout_validation.png — Holdout validation: training-window TOP3 actual growth vs. Seoul average -->

Weight choice wasn't the problem, separately: a sensitivity check across
7 weight scenarios (±0.2) shows the TOP3 ranking stays stable. The
holdout miss reflects a genuine market shift, not a fragile model.

<!-- IMAGE: assets/sensitivity_analysis.png — TOP5 ranking stability across weight scenarios -->

---

## Market Trends (2018–2024)

- Price/pyeong rose from ~2,371 (10k KRW) in 2018 to ~4,126 in 2024, with
  a clear break in 2022
- Transaction volume fell 65% from 2020 (174,126) to 2022 (61,399),
  tracking the rate-hiking cycle
- 2022 was the only year with negative YoY growth (−1.9%), followed by a
  sharp rebound (+19.6% in 2023) — volume stayed low even as price
  rebounded, pointing to thinner, higher-end-skewed demand rather than a
  broad recovery

<!-- IMAGE: assets/market_overview.png — Market overview: price, price/pyeong, volume, and YoY growth by year -->
<!-- IMAGE: assets/district_year_heatmap.png — District x year price-per-pyeong heatmap -->

---

## Recommendations

| Profile | TOP 1 | TOP 2 | TOP 3 |
|---|---|---|---|
| Conservative | Jungnang-gu (Row house) | Gwanak-gu (Single-family) | Dobong-gu (Row house) |
| Balanced | Jungnang-gu (Row house) | Eunpyeong-gu (Row house) | Geumcheon-gu (Row house) |
| Aggressive | Jungnang-gu (Officetel) | Geumcheon-gu (Officetel) | Jungnang-gu (Row house) |

<!-- IMAGE: assets/top5_districts_by_profile.png — TOP5 districts by investor profile -->

**Dong-level drill-down.** Within each profile's TOP3, a further pass
narrows to the specific dong (neighborhood) driving the signal —
Jungnang-gu's comes mainly from Myeonmok-dong and Junghwa-dong across all
three profiles. Dong-level estimates rest on far fewer transactions than
district-level ones, so any dong under 100 transactions, or missing data
in more than 2 of the 7 years, is flagged low-confidence rather than
folded in silently.

---

## So What

- Jungnang-gu is the only district in every profile's TOP3 — it's
  simultaneously above-average growth (9.6%), below-average volatility
  (5.0), and 22% cheaper than the Seoul average. No other district hits
  all three at once.
- Conservative portfolios consistently land on low-volatility,
  undervalued outer districts — a capital-preservation pattern.
- Part of that "undervaluation" is a housing-stock composition effect,
  not a pure location discount (see the regression above) — worth
  knowing before treating it as a buy signal.
- The holdout miss is the most important result in this project: a
  scoring system tuned to historical patterns doesn't automatically
  carry through a regime change, and that's worth saying plainly.

## Business Impact

For a conservative investor, this analysis narrows a 100-combination
search space (25 districts × 4 building types) down to a specific,
defensible shortlist — row houses in Jungnang-gu's Myeonmok-dong and
Junghwa-dong — far faster than a manual district-by-district comparison,
while flagging two things a plain ranking would miss:

- **Composition risk.** Jungnang-gu's discount is partly explained by
  older, lower-rise housing stock, not pure location undervaluation.
  Due diligence should separate "underrated" from "cheap because of what's built there."
- **Regime risk.** The same framework, trained on pre-2022 data, would
  have underperformed the broader market in the most recent cycle. Best
  used as one input alongside current rate and jeonse-to-price
  conditions — not a standalone buy signal.

A fast shortlist plus explicit, quantified caveats is what a static
scoring model can responsibly deliver — and the honest edge of what it can't.

---

## Repo Structure

```
seoul-real-estate-analysis/
├── data/                                  # yearly CSVs, not tracked in git
│   ├── 2018.csv
│   ├── ...
│   └── 2024.csv
├── notebooks/
│   └── seoul_real_estate_analysis.ipynb   # full pipeline, single notebook
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

## Image Placement Guide

Where each exported chart goes, and the filename it should be saved as
in `assets/` — matches the `<!-- IMAGE: -->` markers above.

| Chart | Goes under section | Filename |
|---|---|---|
| Market overview (4-panel) | Market Trends | `market_overview.png` |
| District × year heatmap | Market Trends | `district_year_heatmap.png` |
| TOP5 districts by profile | Recommendations | `top5_districts_by_profile.png` |
| Sensitivity line chart | Statistical Validation (holdout subsection) | `sensitivity_analysis.png` |
| Regression district effects | Statistical Validation (regression subsection) | `regression_district_effects.png` |
| Holdout validation bars | Statistical Validation (holdout subsection) | `holdout_validation.png` |
| District metric bars (growth/vol/price) | Market Trends or a new subsection, your call | `district_metric_bars.png` |
| Radar chart (profile comparison) | Recommendations | `profile_radar.png` |
| Building-type / dong score charts | Recommendations | `bldg_type_scores.png`, `dong_level_scores.png` |

---

## What's Different About This One

- Every headline result is checked against a significance test, an
  alternative model, or an out-of-sample test — not left as a
  descriptive ranking
- The holdout failure is reported, not tuned away — a limitation
  treated as a finding
- Decomposes *why* a district looks cheap (location vs. housing-stock
  mix) instead of stopping at "cheap"
- Drills down two levels of geography (gu → dong) with confidence
  flagging, instead of presenting neighborhood numbers at the same
  certainty as district-level ones

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

## Author & Contribution

**Joshua Kim** — Data Analyst · Marketing · E-commerce · Business Analytics

Completed as a 4-person team project. All data analysis, scoring
methodology, statistical validation, and code here are individual work;
team contribution beyond the analysis was the presentation deck.

📧 Jkim43844@gmail.com · 🔗 [LinkedIn](https://www.linkedin.com/in/joshua-kim-87b478263/)

---

## License

Portfolio and educational use. Data: [Seoul Open Data Plaza](https://data.seoul.go.kr/) (Public Domain)
