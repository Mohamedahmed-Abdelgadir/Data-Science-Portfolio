# Coffee Quality Analysis — Multivariate Statistical Analysis

## Goal

Translate multivariate statistical theory into actionable supply chain decisions for a high-end specialty coffee roaster. Using **Hotelling's T² tests**, **Mahalanobis distance**, and **simultaneous confidence intervals** — all implemented from scratch using matrix algebra — the analysis addresses six critical procurement challenges:

1. **Baseline Profiling** — Characterize the mean, covariance, and correlation structure of sensory metrics
2. **Quality Control** — Detect abnormal batches via multivariate outlier detection
3. **Target Benchmarking** — Test if new shipments meet multidimensional quality targets (one-sample T²)
4. **Supplier Comparison** — Compare flavor profiles across origins/processing methods (two-sample T²)
5. **Risk Management** — Establish B2B quality guarantees via simultaneous confidence intervals
6. **Intervention ROI** — Measure multivariate impact of a roasting protocol change (paired T²)

## Dataset

- **Source:** Coffee Quality Database from CQI (Specialty Coffee Quality Institute) via Kaggle
- **Loading:** `kagglehub` API from `volpatto/coffee-quality-database-from-cqi`
- **Rows:** 1,339 (after cleaning)
- **Species:** Arabica and Robusta

### 10 Core Sensory Variables (0–10 scale)

| Variable | Description |
|---|---|
| Aroma | Smell before, during, and after brewing |
| Flavor | Primary taste and aroma while in the mouth |
| Aftertaste | How long pleasant flavors linger after swallowing |
| Acidity | Brightness / crispness (lively or fruity notes) |
| Body | Physical thickness / weight in the mouth |
| Balance | Harmony across flavor, aftertaste, acidity, and body |
| Uniformity | Consistency across multiple cups of the same batch |
| Clean Cup | Absence of negative / non-coffee flavors |
| Sweetness | Pleasing fullness without harshness |
| Cupper.Points | Taster's overall rating (Overall) |

### Defect Variables

| Variable | Description |
|---|---|
| Category.One.Defects | Severe fatal flaws (specialty: zero allowed) |
| Category.Two.Defects | Minor flaws (partial insect damage, chipped beans) |

## Tech Stack

| Library | Purpose |
|---|---|
| pandas, numpy | Data manipulation, matrix algebra |
| matplotlib, seaborn | Visualizations (histograms, heatmaps, Q-Q plots) |
| kagglehub | Dataset loading via Kaggle API |
| scipy.stats | Shapiro-Wilk normality test, F-distribution, chi-square |

*All Hotelling's T² statistics and Mahalanobis distances are implemented from scratch using matrix operations — no high-level multivariate libraries.*

## Analysis Phases

### A. Data Loading & Exploration

The dataset is loaded via Kaggle's API (`kagglehub.load_dataset`). Initial exploration reveals the structure of the sensory variables, species distribution (Arabica vs. Robusta), and defect counts.

### B. Data Cleaning

String-based sensory columns (originally stored as text with comma separators) are parsed and cast to numeric types. Rows with missing values are handled, and the final cleaned dataset contains **1,339 observations** with 10 continuous sensory variables.

### C. Normality Check

- **Histograms** with overlaid density curves for each of the 10 sensory variables
- **Q-Q plots** against the theoretical normal distribution
- **Shapiro-Wilk test** applied to each variable to formally assess univariate normality

The normality assessment informs the validity of the multivariate normal assumption required for Hotelling's T² inference.

### D. Baseline Profiling

- **Mean vector** — average scores across all 10 sensory dimensions
- **Covariance matrix** — pairwise variability and co-variation
- **Correlation matrix** — standardized pairwise relationships, visualized as a heatmap

This establishes the benchmark profile against which all subsequent comparisons are measured.

### E. Quality Control (Mahalanobis Distance Outlier Detection)

Mahalanobis distance is computed for each observation:

$D_i^2 = (x_i - \bar{x})^T S^{-1} (x_i - \bar{x})$

where $\bar{x}$ is the sample mean vector and $S$ is the sample covariance matrix. Observations exceeding the chi-square critical value ($\chi^2_{10, 0.05}$) are flagged as multivariate outliers.

**Result:** A small number of batches are identified as statistically abnormal across the joint sensory profile, flagging them for QA review before entering the supply chain.

### F. Target Benchmarking (One-Sample Hotelling's T²)

Tests whether the mean vector of a new shipment equals a predefined target profile $\mu_0$:

$T^2 = n(\bar{x} - \mu_0)^T S^{-1} (\bar{x} - \mu_0)$

The test statistic is compared against the F-distribution: $\frac{n-p}{p(n-1)} T^2 \sim F_{p, n-p}$.

**Result:** Determines whether incoming lots meet the roaster's exacting multidimensional quality standards.

### G. Supplier Comparison (Two-Sample Hotelling's T²)

Compares the mean vectors of two independent groups (e.g., Mexico vs. Guatemala):

$T^2 = (\bar{x}_1 - \bar{x}_2)^T S_p^{-1} (\bar{x}_1 - \bar{x}_2) \cdot \frac{n_1 n_2}{n_1 + n_2}$

where $S_p$ is the pooled covariance matrix.

**Result:** Statistically significant divergence detected between regional terroirs (e.g., Mexico vs. Guatemala), informing blending and roasting optimization.

### H. Risk Management (Simultaneous Confidence Intervals)

Two types of simultaneous confidence intervals are constructed for the mean vector:

1. **T² Intervals** — Based on the overall Hotelling's T² ellipse:
   $\bar{x}_i \pm \sqrt{\frac{p(n-1)}{n-p} F_{p, n-p}(\alpha)} \sqrt{\frac{s_{ii}}{n}}$

2. **Bonferroni Intervals** — More conservative, with adjusted per-variable alpha:
   $\bar{x}_i \pm t_{n-1}(\alpha/2p) \sqrt{\frac{s_{ii}}{n}}$

**Result:** Bonferroni bounds provide tighter, more practical guarantees for B2B wholesale contracting without over-promising quality.

### I. Intervention Impact (Paired Hotelling's T²)

Measures the multivariate effect of a simulated change in roasting protocol by analyzing paired differences (pre/post on the same batches):

$T^2 = n \bar{d}^T S_d^{-1} \bar{d}$

where $\bar{d}$ is the mean difference vector and $S_d$ is the covariance of differences.

**Result:** Quantifies whether the intervention produces a statistically significant multivariate shift in the sensory profile, enabling ROI-calibrated decisions about process changes.

## Key Findings

| Analysis | Result |
|---|---|
| **Quality Control** | Small number of multivariate outliers identified via Mahalanobis distance — flagged for QA review |
| **Target Benchmarking** | One-sample T² assesses whether shipments meet multidimensional target profiles |
| **Supplier Comparison** | Significant terroir divergence (Mexico vs. Guatemala) detected — blending/roasting optimization recommended |
| **Risk Management** | Bonferroni intervals provide tighter bounds than T² intervals — recommended for B2B contracting |
| **Intervention Impact** | Paired T² quantifies multivariate shift from roasting protocol changes |

## Strategic Outcomes

1. **Targeted Procurement** — Regional terroir differences (Mexico vs. Guatemala) inform blending and roasting curves
2. **Risk Mitigation** — Bonferroni simultaneous confidence bounds enable safe B2B wholesale contracts
3. **Controlled Innovation** — Volatile processing methods (e.g., Extended Anaerobic Fermentation) isolated into specialty product lines

## How to Run

1. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn scipy kagglehub
   ```
2. Open the notebook:
   ```bash
   jupyter notebook CQA.ipynb
   ```
3. Run all cells sequentially. The dataset will be downloaded automatically via Kaggle's API on first run.
