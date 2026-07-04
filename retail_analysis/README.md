# Walmart Retail Sales Analysis

## Goal

Perform an **Exploratory Data Analysis (EDA)** on Walmart's weekly sales data across 45 stores to answer five specific business questions about store performance, holiday impact, seasonal trends, and relationships with economic indicators.

## Dataset

- **Source:** Walmart Sales Dataset (45 stores)
- **Rows:** 6,435
- **Time Period:** Feb 5, 2010 — Oct 26, 2012 (~2 years 9 months)
- **Missing Values:** None
- **Duplicates:** None

| Column | Type | Description |
|---|---|---|
| Store | int | Store ID (1–45) |
| Date | datetime | Week start date |
| Weekly_Sales | float | Sales for that week (USD) |
| Holiday_Flag | int | 1 = holiday week, 0 = non-holiday |
| Temperature | float | Temperature (Fahrenheit) |
| Fuel_Price | float | Fuel price (USD/gallon) |
| CPI | float | Consumer Price Index |
| Unemployment | float | Unemployment rate (%) |

## Tech Stack

| Library | Purpose |
|---|---|
| pandas | Data loading, cleaning, aggregation |
| matplotlib | Line plots, bar charts, scatter plots, box plots |
| seaborn | Box plot visualizations |

## Analysis Walkthrough

### 1. Data Loading & Exploration

The CSV is loaded and inspected. The dataset has no null values and no duplicates — a clean, ready-to-analyze structure. The `Date` column is parsed to `datetime64[ns]` format.

### 2. Outlier Detection

IQR-based outlier detection on `Weekly_Sales` identifies **34 outlier rows** (Q1 = 553,350; Q3 = 1,420,159; IQR = 866,808). Nearly all outliers are extremely high-sale weeks around **Thanksgiving and Christmas** (e.g., December 24 weeks).

### 3. Aggregate Sales Over Time

A time-series line plot of total weekly sales across all stores shows clear recurring peaks — consistent with holiday-driven shopping behavior.

### 4. Q1 — Store with Maximum Total Sales

**Finding: Store 20** has the highest cumulative sales (~$301.4M), followed closely by Store 4 (~$299.5M).

### 5. Q2 — Store with Most Variable Sales

**Finding: Store 14** has the highest standard deviation in weekly sales (~$317,570), indicating the most volatile sales pattern. Store 10 is second (~$302,262).

### 6. Q3 — Holiday Impact Analysis

Holiday weeks are compared against the mean non-holiday weekly sales (~$1,046,964):

| Holiday | Mean Holiday Sales | Negative Impact (below non-holiday mean)? |
|---|---|---|
| Super Bowl | $1,128,778 | No |
| Labour Day | $1,066,708 | No |
| Thanksgiving | $1,066,307 | No |
| **Christmas** | **$959,825** | **Yes** |

**Finding:** Only Christmas correlates with lower sales than a typical non-holiday week. Super Bowl, Labour Day, and Thanksgiving all show elevated sales.

### 7. Q4 — Monthly & Semester View

- **Highest average month:** December (~$1.21M)
- **Lowest average month:** January (~$0.95M)
- **Total by semester (4-month blocks):**
  | Semester (Months) | Total Sales |
  |---|---|
  | 1 (Jan–Apr) | ~$2.147B |
  | 2 (May–Aug) | **~$2.338B** |
  | 3 (Sep–Dec) | ~$2.252B |

**Finding:** The second semester (May–August) has the highest total sales — possibly driven by summer seasonal demand and back-to-school shopping.

### 8. Q5 — Relationships Between Sales and Numeric Features

Scatter and line plots are generated for each feature vs. `Weekly_Sales`:

| Feature vs. Sales | Observed Relationship |
|---|---|
| Temperature | No clear correlation |
| Fuel_Price | No clear pattern |
| CPI | **Negative correlation** — as CPI rises, sales tend to decline |
| Unemployment | Weak / inconclusive |

**Finding:** CPI is the only feature with a visibly clear relationship: higher Consumer Price Index → lower weekly sales.

## Key Findings Summary

| Question | Answer |
|---|---|
| Store with highest total sales | **Store 20** (~$301M) |
| Store with most variable sales | **Store 14** (std ~$317K) |
| Holiday with negative sales impact | **Christmas only** |
| Best month for sales | **December** |
| Best semester for sales | **Semester 2 (May–Aug)** |
| Economic factor most correlated with sales | **CPI** (negative relationship) |

## Data Quality Note

The `Date` column uses **DD-MM-YYYY** format but was parsed with `format='mixed'`, which may cause month/day swapping in month-level aggregations (e.g., December records might correspond to February dates). Month-based findings should be interpreted with this caveat.

## How to Run

```bash
pip install pandas matplotlib seaborn
jupyter notebook DSM_Project.ipynb
```

Run all cells sequentially. The dataset CSV is included in the same directory.
