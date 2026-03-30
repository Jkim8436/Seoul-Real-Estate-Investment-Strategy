# 🏙️ Seoul Real Estate Investment Analysis (2018–2024)

A data-driven investment decision framework that recommends the optimal **district (gu) × building type** combination in Seoul based on an investor's risk profile — Conservative, Balanced, or Aggressive.

> **"Instead of gut feeling, let the data decide where to invest."**

---

## 📌 Project Overview

| Item | Details |
|------|---------|
| **Period** | 2018 – 2024 (7 years) |
| **Scope** | 25 districts × 4 building types in Seoul |
| **Data** | Seoul Open Data Plaza — Real Estate Transaction Records |
| **Approach** | Z-score normalization + weighted composite scoring |
| **Deliverable** | TOP 3 district × building type recommendations per investor profile |

---

## 🎯 Problem Statement

Seoul's real estate market exhibits significant price variation across districts. However, most analyses rely on a single metric — price — which fails to capture the full investment picture.

**Key questions this project answers:**
- Which districts offer the best risk-adjusted returns for each investor type?
- Is a high-price district always a good investment, or are there undervalued areas?
- How stable are these recommendations when the scoring weights are adjusted?

---

## 🔬 Analysis Framework

Three metrics are evaluated for each district, standardized via Z-score, and combined with investor-profile-specific weights:

| Metric | Definition | Preferred Direction |
|--------|-----------|---------------------|
| **Growth** | Average annual price-per-pyeong growth rate (2018–2024) | Higher ↑ |
| **Price** | Current price-per-pyeong relative to Seoul average | Lower (undervalued) ↓ |
| **Volatility** | Standard deviation of annual growth rates | Lower (stable) ↓ |

### Investor Profile Weights

| Profile | Growth | Price | Volatility | Total |
|---------|--------|-------|------------|-------|
| Conservative | 0.5 | 1.0 | **1.5** | 3.0 |
| Balanced | 1.0 | 1.0 | 1.0 | 3.0 |
| Aggressive | **1.5** | 1.0 | 0.5 | 3.0 |

> All profiles sum to **3.0** → scores are on the same scale for fair cross-profile comparison.

---

## 🧠 Key Decisions & Rationale

### Why Z-score normalization?
Growth (%), price (10k KRW/pyeong), and volatility (%) have completely different units and scales. Summing them directly would let the largest-scale variable (price) dominate the result. Z-score standardizes all metrics to mean=0, std=1 — enabling fair aggregation.

### Why two separate notebooks? (Dual Preprocessing)
| Notebook | Preprocessing | Reason |
|----------|--------------|--------|
| Market Overview | **Raw data preserved** | Extreme values (e.g., 2021–2022 surge) are meaningful market signals — removing them distorts trends |
| Investment Analysis | **IQR-based outlier removal** | Outliers skew Z-scores and distort district rankings |

### Are the weights arbitrary?
A **sensitivity analysis** was conducted across 7 weight scenarios (±0.2 range). The TOP 3 results remain stable across all scenarios, confirming that the recommendations are **robust**, not artifacts of arbitrary weight choices.

---

## 📊 Key Findings

### Seoul Market Trends (2018–2024)
- Average transaction price increased significantly, peaking around **2021–2022**
- Transaction volume dropped sharply after 2022 as interest rates rose
- Year-over-year growth rate turned negative in 2023 before recovering

### Investment Recommendations (District × Building Type)

| Profile | TOP 1 | TOP 2 | TOP 3 |
|---------|-------|-------|-------|
| Conservative | Dobong-gu (Row house) | Gwanak-gu (Single-family) | Jungnang-gu (Row house) |
| Balanced | Eunpyeong-gu (Row house) | Geumcheon-gu (Row house) | Jungnang-gu (Row house) |
| Aggressive | Jungnang-gu (Row house) | Jungnang-gu (Officetel) | Geumcheon-gu (Officetel) |

> ※ Exact results depend on the dataset. Run the notebook to reproduce current rankings.

---

## 🗂️ Project Structure

```
seoul-real-estate-analysis/
│
├── data/                                          # Raw CSV files (not tracked in git)
│   ├── 2018.csv
│   ├── 2019.csv
│   ├── ...
│   └── 2024.csv
│
├── Seoul_Real_Estate_Market_Overview.ipynb        # EDA + annual trend analysis (raw data)
├── Seoul_Real_Estate_Investment_Analysis.ipynb    # Investment scoring framework (IQR cleaned)
│
├── 서울시_부동산_시장조사.ipynb                     # Korean version — Market Overview
├── 서울시_부동산_투자분석.ipynb                     # Korean version — Investment Analysis
│
└── README.md
```

---


## 🔍 Notebook Contents

### `Seoul_Real_Estate_Market_Overview.ipynb`
| Section | Content |
|---------|---------|
| Data Loading & Merging | Concatenate 7 years of CSV data |
| Column Renaming | Map Korean column names to English |
| Preprocessing | Remove cancellations, clean year_built |
| EDA | Missing value analysis, price distribution, building type breakdown |
| Trend Analysis | Annual avg price, price/pyeong, transaction volume, YoY growth rate |

### `Seoul_Real_Estate_Investment_Analysis.ipynb`
| Section | Content |
|---------|---------|
| EDA (Pre-removal) | Baseline statistics before outlier removal |
| IQR Outlier Removal | Before vs after comparison with quantified removal rate |
| Feature Engineering | Price per pyeong, YoY growth rates, volatility |
| Z-Score Normalization | Standardize 3 metrics for fair comparison |
| Metric Correlation Check | Verify metric independence before combining |
| Investment Scoring | Weighted composite scores per profile |
| Sensitivity Analysis | Validate result robustness across 7 weight scenarios |
| Building-Type Analysis | Score district × building type combinations |
| Final Recommendations | TOP 3 per investor profile |

---

## 📈 Visualizations

- **Market trend dashboard** — 4-panel chart (price, price/pyeong, volume, growth rate)
- **District comparison bar charts** — avg price, avg price/pyeong (top/bottom 3 highlighted)
- **District × Year heatmap** — price-per-pyeong evolution over time
- **Z-score composite horizontal bars** — investment attractiveness by profile
- **Sensitivity analysis line chart** — ranking stability across weight scenarios
- **Radar chart** — metric profile comparison across 3 investor profiles
- **Building type score charts** — final district × building type rankings

---

## 🚧 Limitations & Future Work

| Limitation | Potential Improvement |
|-----------|----------------------|
| Macroeconomic variables not included | Add interest rate, jeonse-to-price ratio data |
| No future growth signals | Incorporate development plans (transport, redevelopment) |
| Static weights | Build an interactive dashboard for custom weight input |
| Historical analysis only | Add time-series forecasting for price projection |

---

## 👤 Author

**Joshua Kim**  
Data Analyst | Marketing · E-commerce · Business Analytics  
📧 Jkim43844@naver.com  
🔗 [LinkedIn](https://www.linkedin.com/in/joshua-kim-87b478263/)

---

## 📄 License

This project is for portfolio and educational purposes.  
Data source: [Seoul Open Data Plaza](https://data.seoul.go.kr/) (Public Domain)
