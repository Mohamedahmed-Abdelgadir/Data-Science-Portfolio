# Latent Semantic Analysis using Singular Value Decomposition

## Goal

Extract latent topics and semantic meaning from unstructured text using **Latent Semantic Analysis (LSA)** via **Singular Value Decomposition (SVD)** — enabling automatic categorization and similarity search across thousands of documents.

## Scenario

Framed as a **social listening agency** that tracks online reputation for brands. The agency scrapes hundreds of thousands of reviews, comments, and complaints daily. Manual review is impossible, so an automated pipeline is needed to group similar documents and surface consumer trends before PR crises escalate.

## Dataset

The **20 Newsgroups** dataset from `sklearn.datasets`:

| Property | Value |
|---|---|
| Source | sklearn (`fetch_20newsgroups`) |
| Subset | Training (removed headers, footers, quotes) |
| Documents | **11,314** |
| Categories | 20 (computers, sports, politics, religion, science, etc.) |

## Tech Stack

| Library | Purpose |
|---|---|
| sklearn (fetch_20newsgroups, TfidfVectorizer, TruncatedSVD) | Data loading, feature extraction, dimensionality reduction |
| sklearn.metrics.pairwise (cosine_similarity) | Document similarity search |
| NLTK (WordNetLemmatizer) | Lemmatization |
| matplotlib, seaborn | Visualizations (histograms, bar plots, elbow curve) |
| numpy | Numerical operations |
| re | Regex text cleaning |

## Pipeline Steps

### 1. Data Loading

The 20 Newsgroups training subset is loaded via `fetch_20newsgroups(subset='train', remove=('headers', 'footers', 'quotes'))`. The dataset contains 11,314 forum posts and emails spanning 20 diverse topics.

### 2. Exploratory Data Analysis

- **Document Length Distribution:** Histogram reveals that the vast majority of documents are under 200 words — indicating the agency deals primarily with short consumer reactions rather than long-form articles.
- **Top-20 Frequent Words:** The initial word frequency bar plot reveals heavy pollution from dataset artifacts (`ax`), broken contractions (`don`, `ve`, `wa`), and conversational fluff (`just`, `like`, `know`). This motivates aggressive text preprocessing.

### 3. Text Preprocessing

A custom `clean_text()` function applies:

1. **Regex removal** — emails, `.edu` domains, punctuation, numbers
2. **Lowercasing** and whitespace stripping
3. **Lemmatization** via `WordNetLemmatizer` (reduces inflected words to base form)
4. **Custom stop word removal** — built on sklearn's `ENGLISH_STOP_WORDS` with additional fluff terms (e.g., `like`, `just`, `know`, `ax`, `don`, `wa`)

This process is iterative: after the first pass, remaining noise words are identified and added to the custom stop list, then preprocessing runs again for a cleaner result.

### 4. Feature Engineering (TF-IDF)

Documents are converted to numerical feature vectors using `TfidfVectorizer`:
- `max_df=0.90` — ignore words appearing in >90% of documents (too broad)
- `min_df=5` — ignore words appearing in <5 documents (typos / rare terms)

**Result:** 11,314 rows × **13,865 columns** sparse matrix, where each value represents the TF-IDF weight of a word in a document.

### 5. Singular Value Decomposition (SVD)

`TruncatedSVD` factorizes the TF-IDF matrix into three components:
- **U** (Document-Topic matrix) — how each document relates to latent topics
- **Σ** (Singular values) — strength of each topic
- **Vᵀ** (Topic-Word matrix) — how words relate to latent topics

**Optimal k selection (Elbow Method):**

| k | Explained Variance |
|---|---|
| 2 | 0.74% |
| 5 | 1.76% |
| 10 | **3.03%** |
| 20 | 4.78% |
| 50 | 8.47% |
| 100 | 13.06% |

The curve levels off between 10 and 20 topics. **k = 10** is chosen as the optimal balance between dimensionality reduction and information retention.

### 6. Topic Discovery

The Topic-Word matrix (Vᵀ) is inspected: for each of the 10 latent topics, the top 7 words by weight are extracted and interpreted into business-relevant categories:

| Topic | Top Words | Business Interpretation |
|---|---|---|
| **1 & 2** | window, problem, file, drive, thanks | **General Tech Support** |
| **3 & 4** | god, jesus, bible, program, christian | **Religion** (mixed with tech terms = topic bleed) |
| **5** | key, chip, encryption, clipper, government | **Cybersecurity** |
| **6** | game, team, player, chip, window | **Sports & Gaming** |
| **7** | thanks, mail, address, email, advance | **Forum Noise** (polite phrases) |
| **8** | card, driver, video, monitor, color | **Computer Hardware** |
| **9** | jxp, chastity, shameful, intellect, gordon | **Spam / Email Signature** |
| **10** | car, bike, engine, mile, key | **Vehicles** |

### 7. Document Similarity Search

Using `cosine_similarity` on the reduced 10-dimensional document-topic vectors, the pipeline finds the closest related document to any given query.

**Demonstration:** Document #1000 (discussing mouse cursor / driver issues) is matched to Document #11204 (discussing mouse driver configuration) with a **similarity score of 0.9986** — successfully grouping related tech support threads out of 11,314 noisy posts.

## Key Findings

1. **SVD effectively extracts latent topics** from noisy text, even with low explained variance (~3% at k=10), which is expected for high-dimensional sparse TF-IDF matrices.
2. **Topic bleed occurs** — frequent or polarizing words (e.g., `god`, `window`) can appear across unrelated topics, requiring iterative stop word refinement.
3. **The pipeline successfully groups related documents** — the cosine similarity search correctly paired two technical posts about mouse drivers, demonstrating practical value for ticket routing and duplicate detection.
4. **k = 10 topics** provides a good balance: enough granularity to distinguish Cybersecurity from Hardware from Vehicles, but compressed enough to avoid noise.

## How to Run

1. Install dependencies:
   ```bash
   pip install scikit-learn nltk matplotlib seaborn numpy
   ```
2. Open the notebook:
   ```bash
   jupyter notebook LSA_using_SVD.ipynb
   ```
3. Run all cells sequentially. The 20 Newsgroups dataset will be downloaded automatically by sklearn.
