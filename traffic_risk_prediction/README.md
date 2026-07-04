# Traffic Risk Prediction — Big Data Pipeline

## Goal

Analyze how weather conditions affect urban traffic behavior in London using a modern **Data Lake architecture**. The project processes large-scale synthetic weather and traffic datasets through a multi-layer pipeline to predict **traffic congestion** and **accident risks** under various weather scenarios, ultimately supporting data-driven traffic planning and management decisions.

## Data Lake Architecture

```
Bronze (CSV) ──→ Silver (Parquet) ──→ Gold (Parquet) ──→ HDFS ──→ Analytics
   MinIO              MinIO               MinIO           Hadoop     Monte Carlo
   Raw data           Cleaned             Feature-         Dist.     Factor Analysis
                      data                engineered       storage   Dashboard
```

## Datasets

Two synthetic datasets (5,000 rows each):

| Dataset | Key Columns |
|---|---|
| **Weather** | weather_id, date_time, city, season, temperature_c, humidity, rain_mm, wind_speed_kmh, visibility_m, weather_condition |
| **Traffic** | traffic_id, date_time, city, area, vehicle_count, avg_speed_kmh, accident_count, congestion_level, road_condition, visibility_m |

## Tech Stack

| Technology | Purpose |
|---|---|
| Docker / Docker Compose | Container orchestration (MinIO, Hadoop) |
| MinIO (S3-compatible) | Data Lake storage (Bronze / Silver / Gold) |
| Hadoop HDFS | Distributed file system for Gold layer |
| Pandas, NumPy | Data cleaning, transformation, analysis |
| Scikit-learn (LogisticRegression, StandardScaler) | Probabilistic modeling |
| FactorAnalyzer | Latent factor extraction (Varimax rotation) |
| Matplotlib, Seaborn | Static visualizations |
| Streamlit, Plotly Express | Interactive web dashboard |
| Parquet | Columnar storage format |

## Pipeline Phases

### Phase 1: Infrastructure & Data Ingestion (Bronze Layer)

Sets up a **MinIO** server via Docker Compose (ports 9000/9001, credentials `admin`/`admin123`), creates three buckets (`bronze`, `silver`, `gold`), and uploads the raw CSV datasets into the Bronze bucket.

**Outputs:** `weather_raw.csv`, `traffic_raw.csv` in MinIO Bronze bucket

### Phase 2: Data Cleaning (Bronze → Silver)

Reads raw data from MinIO Bronze and applies extensive cleaning:

| Dataset | Actions | Rows Before | Rows After |
|---|---|---|---|
| **Weather** | Remove duplicate IDs, parse mixed date formats, impute missing city/season, filter temperature (-6 to 40°C), humidity (0–100), wind (<200 km/h), visibility (50–10,000m), standardize weather_condition | 5,000 | ~3,024 |
| **Traffic** | Same ID/date fixes, fill missing city/area, cap avg_speed (>150 removed) & vehicle_count (>5,000 capped), impute accident_count (fill 0), standardize congestion_level & road_condition | 5,000 | ~4,151 |

**Outputs:** `weather_cleaned.parquet`, `traffic_cleaned.parquet` in MinIO Silver bucket

### Phase 3: Transformation & Gold Layer (Silver → Gold → HDFS)

Reads cleaned Parquet from Silver, performs feature engineering (extract `day`, `hour` from `date_time`), creates daily aggregations, and saves four Gold datasets to MinIO. All Gold datasets are then uploaded to **Hadoop HDFS** (1 NameNode + 1 DataNode via Docker).

**Outputs:**
- `weather_gold.parquet`, `traffic_gold.parquet` — feature-engineered
- `weather_daily.parquet`, `traffic_daily.parquet` — daily aggregates
- All four replicated to HDFS (`/weather/`, `/traffic/`)

### Phase 4: Dataset Merging

Merges cleaned weather and traffic datasets via **INNER JOIN** on `date_time` and `city`. Standardizes merge keys (datetime conversion, lowercase city). Drops records with missing merge keys.

**Result:** 153 rows × 18 columns (limited matches due to exact time-city pairing)

**Output:** `merged_data.parquet`

### Phase 5: Monte Carlo Simulation

Loads merged data (153 rows), performs EDA (numerical + categorical distribution plots), creates binary targets (`is_congested`, `high_accident_risk`), and trains **two Logistic Regression models** (with `StandardScaler`) to predict:

1. **Congestion probability** from weather features
2. **Accident risk probability** from weather features

Runs **10,000 Monte Carlo simulations** per scenario across 7 weather conditions:

| Scenario | Congestion Probability | Accident Risk Probability |
|---|---|---|
| Baseline (Normal) | ~64.3% | ~47.0% |
| **Low Visibility** | **~68.8%** ⬆ | ~51.2% |
| **Heavy Rain** | ~66.4% | **~59.6%** ⬆ |
| Extreme Cold | ~64.5% | ~50.9% |
| Extreme Heat | ~64.0% | ~42.8% ⬇ |
| Strong Winds | ~57.9% | ~47.2% |
| High Humidity | ~56.5% ⬇ | ~50.3% |

**Key findings:**
- **Low Visibility** causes the highest congestion (~69%)
- **Heavy Rain** is the most dangerous for accident risk (~60%)
- **High Humidity** results in smoothest traffic (~57%)
- **Extreme Heat** shows the lowest accident risk (~43%)

### Phase 6: Factor Analysis

Applies **Factor Analysis** (Varimax rotation) on the merged dataset to identify latent dimensions:

| Factor | Interpretation | Dominant Variable | Loading |
|---|---|---|---|
| Factor 1 | **Weather Severity** | wind_speed_kmh | 0.997 |
| Factor 2 | **Traffic Flow / Stress** | avg_speed_kmh | 0.569 |
| Factor 3 | **Accident Risk** | rain_mm | 0.426 |

Cumulative variance explained: ~25.6% (modest, reflecting data limitations). Suitability tests: Bartlett's (p=0.837), KMO (0.491).

### Phase 7: Interactive Dashboard

Built with **Streamlit** + **Plotly Express**, deployed via ngrok. Three tabs:
1. **Dataset Overview** — Key metrics, interactive scatter plot (speed vs. selectable weather variable), raw data viewer
2. **Monte Carlo Simulation** — Scenario selector, congestion probability distribution histogram
3. **Factor Analysis** — Factor interpretations, loadings table, interactive heatmap

## How to Run

1. **Start infrastructure (Phase 1 & 3):**
   ```bash
   cd Phase_1_2_3_5_6_7/Phase1
   docker compose up -d        # Starts MinIO
   cd ../Phase3/hadoop_cluster
   docker compose up -d        # Starts Hadoop HDFS
   ```

2. **Run notebooks sequentially** (each notebook handles MinIO/HDFS connectivity):
   - `Phase2/Phase2(cleaning_code)/cleanning.ipynb`
   - `Phase3/Phase3_Coding/code_phase3.ipynb`
   - `phase 4/Copy_of_Untitled6.ipynb`
   - `Phase5/MontoCarlo.ipynb`
   - `phase 6/Untitled20.ipynb`
   - `phase 7/Phase_7.ipynb`

3. **Install Python dependencies:**
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn factor-analyzer streamlit plotly minio
   ```
