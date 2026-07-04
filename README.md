# Data Science Portfolio

A curated collection of academic and personal projects spanning big data engineering, multivariate statistics, machine learning, and natural language processing.

## Tech Stack

**Languages & Frameworks:** Python, PySpark, SQL  
**Data Processing:** pandas, NumPy, PySpark DataFrames, MinIO, Hadoop HDFS  
**Machine Learning:** scikit-learn, PySpark MLlib, scikit-fuzzy, sklearn-extra  
**Statistics:** Hotelling's T², Factor Analysis, Monte Carlo Simulation, ANOVA  
**NLP:** NLTK, TF-IDF, TruncatedSVD, Latent Semantic Analysis  
**Visualization:** matplotlib, seaborn, Plotly, Streamlit  
**Infrastructure:** Docker, Docker Compose, MinIO (S3), Hadoop HDFS

## Projects

| # | Project | Description | Key Techniques |
|---|---------|-------------|----------------|
| 1 | [Traffic Risk Prediction](traffic_risk_prediction) | 7-phase Big Data pipeline analyzing how weather affects urban traffic. Full Data Lake architecture (Bronze → Silver → Gold → HDFS) with Monte Carlo simulation, factor analysis, and interactive Streamlit dashboard. | Docker, MinIO, Hadoop HDFS, Logistic Regression, Monte Carlo, Factor Analysis, Streamlit |
| 2 | [Coffee Quality Analysis](coffee_quality_analysis) | Multivariate statistical quality control for specialty coffee procurement. Hotelling's T² (one-sample, two-sample, paired), Mahalanobis distance outlier detection, and Bonferroni simultaneous confidence intervals — all implemented from scratch. | Hotelling's T², Mahalanobis Distance, Simultaneous CIs, Shapiro-Wilk |
| 3 | [SMS Spam Detection](sms_spam_detection_pyspark) | End-to-end PySpark ML pipeline on 5.5K SMS messages. TF-IDF feature engineering, oversampling for class imbalance, Logistic Regression classification at 97.5% F1 score. | PySpark, TF-IDF, Logistic Regression, Oversampling |
| 4 | [Social Media Engagement Analysis](social_media_engagemen_analysis) | Clustering 7K social media posts by engagement and reach using K-Medoids, plus a 27-rule fuzzy logic inference system predicting Reach from Likes, Shares, and Comments. | K-Medoids, Silhouette Score, Fuzzy Logic (Mamdani), Linear Regression |
| 5 | [LSA using SVD](latent_semantic_analysis_SVD) | Latent Semantic Analysis on 11K documents from 20 Newsgroups. TF-IDF vectorization, TruncatedSVD with elbow-based k-selection, topic discovery, and cosine similarity document search. | TruncatedSVD, TF-IDF, Cosine Similarity, NLTK, Elbow Method |
| 6 | [Walmart Sales EDA](retail_analysis) | Exploratory analysis of 6.4K weekly sales records across 45 Walmart stores. Answers 5 business questions on store performance, holiday impact, seasonal patterns, and economic correlations. | Descriptive Analytics, IQR Outlier Detection, Time Series |

## Navigation

Each project folder contains its own `README.md` with a detailed walkthrough of the pipeline, methodology, and results. Click the project name above to navigate.
