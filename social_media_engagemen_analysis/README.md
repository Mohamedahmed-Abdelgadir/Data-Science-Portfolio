# Social Media Engagement Analysis

## Goal

Analyze social media post performance using two complementary approaches:

1. **K-Medoids Clustering** — Group posts by engagement metrics and reach to identify which platform and post-type combinations perform best.
2. **Fuzzy Logic Control System** — Build a Mamdani-type fuzzy inference system that predicts Reach from Likes, Shares, and Comments.

Supporting this are exploratory data analysis (distributions, correlations, categorical breakdowns) and pairwise linear regressions to quantify relationships between engagement variables.

## Dataset

- **Source:** Social Media Engagement Dataset (reduced)
- **Size:** 6,996 rows × 25 columns
- **Key columns:**

| Column | Type | Description |
|---|---|---|
| Platform | Categorical | Facebook, Instagram, LinkedIn, Twitter |
| Post Type | Categorical | Image, Video, Link |
| Likes / Comments / Shares | int | Engagement counts |
| Impressions / Reach | int | How many saw the post |
| Engagement Rate | float | (engagements / impressions) × 100 |
| Audience Age / Age Group | int / cat | Demographics |
| Audience Gender | Categorical | Male, Female, Other |
| Audience Continent | Categorical | Africa, Asia, Europe, NorthAmerica, Oceania, SouthAmerica |
| Sentiment | Categorical | Positive, Negative, Neutral, Mixed |
| Time Periods | Categorical | Morning, Afternoon, Evening, Night |
| Weekday Type | Categorical | Weekday, Weekend |
| Campaign ID / Influencer ID | Categorical | Mostly null (dropped in cleaning) |

## Tech Stack

| Purpose | Library |
|---|---|
| Data manipulation | pandas, numpy |
| Visualization | matplotlib, seaborn |
| Clustering | sklearn-extra (KMedoids), scikit-learn (MinMaxScaler, silhouette_score) |
| Fuzzy logic system | scikit-fuzzy (skfuzzy.control) |
| Regression | scipy.stats (linregress) |

## Pipeline Steps

### 1. Data Loading & Cleaning

The CSV is loaded with `pd.read_csv()`. Four low-value columns are dropped (`Influencer ID`, `Campaign ID`, `Time`, `Date`). Rows with null values and duplicates are removed. `Post Timestamp` is converted to `datetime64[ns]`, and the ` Engagement Rate ` column is renamed by stripping trailing whitespace. Final shape: 6,996 rows × 21 columns.

### 2. Exploratory Data Analysis

Visualizations are generated to understand distributions and relationships:

- **Box plots** — Likes, Comments, Shares; Audience Age; Reach, Impressions
- **Histograms** — 5 subplots showing distributions of Likes, Comments, Shares, Reach, Impressions
- **Correlation heatmap** — Correlation matrix among all numerical engagement variables
- **Pie charts** — Frequency breakdowns for Post Type (Image/Video/Link), Platform (Facebook/Instagram/LinkedIn/Twitter), and Weekday Type (Weekday/Weekend)
- **Bar plots** — Frequency distributions for Age Group, Audience Gender, and Sentiment

### 3. Statistical Analysis (Linear Regression)

Pairwise linear regressions are computed using `scipy.stats.linregress()` to measure the strength of relationships:

| Relationship | R² |
|---|---|
| Likes → Reach | 0.078 |
| Comments → Reach | 0.101 |
| Shares → Reach | 0.111 |
| Likes → Comments | **0.630** |

Individual engagement metrics have very weak linear relationships with Reach (R² < 0.12). The strongest linear relationship is between Likes and Comments.

### 4. K-Medoids Clustering

**Goal:** Partition posts into clusters by their engagement volume and reach, then analyze which platforms and post types dominate the higher-performing cluster.

**Steps:**
1. Feature selection: `['Likes', 'Comments', 'Shares', 'Reach']`
2. Min-Max normalization to [0, 1] using `MinMaxScaler`
3. Feature engineering: `Engagement Total` = Likes + Comments + Shares; individual columns dropped
4. Optimal k selection: K-Medoids fitted for k = 2…6; `silhouette_score` maximized at **k = 2**
5. Final model: `KMedoids(n_clusters=2, random_state=42)` on the normalized 2-feature dataset
6. Cluster assignment merged back; cluster composition analyzed by platform and post type

**Results:**

| Cluster | Dominant Platforms | Dominant Post Types |
|---|---|---|
| 0 (higher performance) | Twitter (1585), Instagram (1151) | Video (1804) |
| 1 (lower performance) | LinkedIn (1577), Facebook (1283) | Link (1892), Image (1176) |

**Conclusion:** Twitter and Instagram generally outperform Facebook and LinkedIn. Video posts outperform Links and Images.

### 5. Fuzzy Logic Control System

**Goal:** Predict Reach from Likes, Shares, and Comments using a fuzzy inference system that maps qualitative input levels to a quantitative reach estimate and linguistic classification.

**Membership Functions:** (all triangular, `fuzz.trimf`)
- `likes` — universe [0, 1000]: terms {low, medium, high}
- `shares` — universe [0, 200]: terms {low, medium, high}
- `comments` — universe [0, 500]: terms {low, medium, high}
- `reach` (output) — universe [500, 5000]: terms {very_low, low, medium, high, very_high}

**Rule Base:** 27 Mamdani-type rules covering all combinations of the three inputs:
- IF likes=low AND shares=low AND comments=low THEN reach=very_low
- IF likes=low AND shares=low AND comments=medium THEN reach=low
- …
- IF likes=high AND shares=high AND comments=high THEN reach=very_high

**Inference:**
- Control system created via `ctrl.ControlSystem` and simulated with `ctrl.ControlSystemSimulation`
- Applied to all 6,996 rows: each row's Likes, Shares, and Comments fed as inputs; crisp reach predicted via centroid defuzzification
- Predicted reach classified into one of 5 linguistic categories by finding the membership function with the highest degree of membership at the predicted value

**Single-example demonstration:**
- Input: likes=500 (medium), shares=190 (high), comments=300 (medium)
- Output: predicted_reach ≈ 4500 (classified as very_high)

## Key Findings

1. **Platform + Post Type matter:** Twitter and Instagram with Video content sees the highest engagement and reach.
2. **Weak linear predictors:** Individual metrics (Likes, Comments, Shares) have low R² (< 0.12) against Reach — the relationship is not linear.
3. **Fuzzy inference works:** The fuzzy system produces reach predictions from engagement inputs, though the rule design and universe ranges could be tuned for more granularity.

## How to Run

1. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn scikit-learn-extra scikit-fuzzy scipy
   ```
2. Open the notebook:
   ```bash
   jupyter notebook Final_DM_Project.ipynb
   ```
3. Run all cells sequentially (adjust CSV path if needed).
