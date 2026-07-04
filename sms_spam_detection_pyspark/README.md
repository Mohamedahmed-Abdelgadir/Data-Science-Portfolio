# SMS Spam Detection Pipeline using PySpark

## Goal

Build an end-to-end SMS Spam Detection pipeline using **PySpark** (Spark's Python API) that ingests raw SMS text data, cleans and preprocesses it, engineers features, trains a classification model, and serves predictions — simulating a real-world data engineering and machine learning workflow on distributed data.

## Dataset

- **Source:** SMS Spam Collection Dataset (Kaggle)
- **Size:** 5,574 rows
- **Columns:**
  - `v1` — Label: `spam` or `ham`
  - `v2` — SMS message text
  - `_c2`, `_c3`, `_c4` — Extra columns from CSV parsing (message fragments split across cells)

## Tech Stack

| Component | Library |
|---|---|
| Distributed data processing | PySpark (SparkSession, DataFrames, SQL functions) |
| ML pipeline & feature engineering | PySpark MLlib (RegexTokenizer, StopWordsRemover, CountVectorizer, IDF) |
| Classification models | PySpark MLlib (LogisticRegression, DecisionTreeClassifier, RandomForestClassifier) |
| Visualization | Matplotlib |
| Data manipulation (visualization helpers) | Pandas |

## Pipeline Steps

### 1. Data Ingestion

A SparkSession is initialized and the CSV is loaded with `inferSchema=True`. The raw schema reveals 5 columns — `v1` (label), `v2` (message), and three unnamed columns (`_c2`, `_c3`, `_c4`) that are mostly null. The data is split 80/20 into training and testing sets via `randomSplit([0.8, 0.2], seed=3)`.

### 2. Data Cleaning & Preprocessing

Investigation of the non-null values in `_c2`, `_c3`, `_c4` shows that some long SMS messages were fragmented across multiple columns during CSV extraction. The fix uses `concat_ws(' ', col('v2'), col('_c2'), col('_c3'), col('_c4'))` to merge all fragments into a single `message` column. The extra columns are then dropped, `v1` is renamed to `label`, and the string labels are mapped to integers (`spam` → `1`, `ham` → `0`).

### 3. Exploratory Data Analysis

- **Class distribution:** 3,850 ham vs. 616 spam — a significant class imbalance (~6:1).
- **Message length:** Computed using PySpark's `length()` function. Most messages are under 200 characters. Notably, **zero spam messages** exceed 200 characters.
- **Correlation analysis:** Log-transformed lengths (`log1p`) to reduce skew, then computed the Pearson (Point-Biserial) correlation with the label. Result: **r = 0.419** — a moderate positive correlation indicating that longer messages tend to be spam.

### 4. Handling Class Imbalance

Oversampling is applied to the minority class: spam messages are resampled with replacement using `sample(withReplacement=True, fraction=ratio)` to approximately match the ham count, yielding a balanced training set of **3,850 ham / 3,717 spam**.

### 5. Feature Engineering (TF-IDF Pipeline)

An NLP feature pipeline is constructed with four stages:

| Stage | Component | Purpose |
|---|---|---|
| 1 | `RegexTokenizer` | Split messages into word tokens on non-word characters |
| 2 | `StopWordsRemover` | Remove common English stop words (the, and, is, ...) |
| 3 | `CountVectorizer` | Build a vocabulary (max 10,000 terms, min 5 docs) and generate term-frequency vectors |
| 4 | `IDF` | Scale term frequencies by inverse document frequency to emphasize rare, discriminative words |

### 6. Model Training

A **Logistic Regression** classifier (`maxIter=20`) is trained on the TF-IDF features. The notebook also defines helper functions (`lr_pipeline`, `test_pipeline`) that wrap the full pipeline — from raw incoming data through concatenation → label encoding → TF-IDF transform → prediction — enabling reuse on arbitrary new data.

Cell placeholders for **Decision Tree** and **Random Forest** classifiers are included for future experimentation.

### 7. Model Evaluation

Metrics are computed on the held-out test set (1,108 rows) using PySpark's `MulticlassClassificationEvaluator`:

| Metric | Score |
|---|---|
| Accuracy | **0.9747** |
| Precision | **0.9743** |
| Recall | **0.9747** |
| F1 Score | **0.9744** |

Misclassified examples are examined manually — false positives (ham flagged as spam) often contain words like "hello", "call", or "free" that overlap with spam vocabulary.

### 8. Model Persistence

The full end-to-end pipeline (tokenizer → remover → CV → IDF → LogisticRegression) is fitted and saved as a `PipelineModel` to disk (`spam_detection_pipeline_model/`) for later reloading and inference.

### 9. Prediction on New Messages

The pipeline is tested on six custom messages:
- Spam correctly flagged: "Congratulations! You've won a $1000 Walmart gift card...", "URGENT! Your mobile number has been selected...", "WINNER WINNER CHICKEN DINNER"
- Ham correctly identified: "Hey, are we still meeting for lunch at 12?", "I'll be home in 10 minutes...", "Hello babe, miss you..."

## Results Summary

The Logistic Regression model achieves **~97.5% F1 score** on unseen test data, demonstrating that a relatively simple TF-IDF + linear classifier approach, combined with proper handling of class imbalance and text preprocessing, is highly effective for SMS spam detection.

## How to Run

1. Install Apache Spark and PySpark (e.g., `pip install pyspark`)
2. Install dependencies: `pip install matplotlib pandas`
3. Open the notebook:
   ```bash
   jupyter notebook sms_spam_detection_pyspark.ipynb
   ```
4. Run all cells sequentially (adjust the CSV path in the data-loading cell if needed).
