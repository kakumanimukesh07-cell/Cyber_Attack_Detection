# 🛡️ Network Intrusion Detection Using Machine Learning

> **Course:** ESI6613 Applied Data Intelligence

A multi-class classification system that detects cyberattacks from network flow data using six machine learning models. The best model — K-Nearest Neighbors — achieved a **weighted F1 score of 0.9972** on unseen test data.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Data Preprocessing](#data-preprocessing)
- [Models](#models)
- [Results](#results)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Usage](#usage)

---

## Project Overview

This project builds and evaluates multiple classification models to detect different types of network attacks from flow-level data. The goal is to identify the best-performing model for predicting attack types on unseen traffic.

**Attack Classes:**

| Label | Attack Type |
|-------|-------------|
| 0 | BENIGN |
| 1 | DoS GoldenEye |
| 2 | DoS Hulk |
| 3 | DoS Slowhttptest |
| 4 | DoS Slowloris |
| 5 | FTP Patator |
| 6 | Heartbleed |
| 7 | SSH Patator |

---

## Dataset

| Property | Value |
|----------|-------|
| Total samples | 429,735 |
| Features | 78 numerical predictors |
| Target classes | 8 |
| Train/Test split | 70% / 30% |
| Training subsample | 50,000 (stratified) |

The dataset is **severely imbalanced** — BENIGN traffic dominates at 35,000+ samples while some attack types are barely represented (e.g., Heartbleed). This motivated the use of **weighted F1 score** as the primary evaluation metric.

---

## Data Preprocessing

1. **Column cleaning** — Stripped whitespace from all column names.
2. **Feature removal** — Dropped 19 redundant features including:
   - Duplicate columns (e.g., `Fwd Header Length.1`)
   - Zero-variance columns (e.g., `Bwd PSH Flags`, `Fwd URG Flags`, all bulk rate features)
   - Highly correlated pairs (e.g., `Flow IAT Min`, `Init_Win_bytes_forward`)
3. **Missing values** — Non-numeric entries coerced to `NaN`; affected rows dropped.
4. **Label encoding** — Target variable encoded 0–7 using `LabelEncoder`.
5. **Scaling** — `StandardScaler` applied for distance-based and linear models (KNN, SVM, Logistic Regression, Naive Bayes). Tree-based models used unscaled data.
6. **Stratified subsampling** — 50,000 rows drawn from the training set to reduce runtime while preserving class proportions.

---

## Models

Six models spanning different algorithmic families were evaluated:

| Model | Key Hyperparameters |
|-------|---------------------|
| **K-Nearest Neighbors (KNN)** | `n_neighbors=3`, distance weighted, `ball_tree` |
| **Random Forest** | 100 trees, `max_depth=10` |
| **Histogram Gradient Boosting** | 200 iterations, `max_depth=6`, `learning_rate=0.1` |
| **Linear SVM** | Squared hinge loss |
| **Logistic Regression** | `saga` solver, multinomial, `max_iter=3000` |
| **Gaussian Naive Bayes** | Default |

---

## Results

| Model | Weighted F1 Score |
|-------|:-----------------:|
| **KNN** ⭐ | **0.9972** |
| Random Forest | 0.9969 |
| Gradient Boosting | 0.9892 |
| Linear SVM | 0.9869 |
| Logistic Regression | ~0.93 |
| Naive Bayes | ~0.93 |

**KNN** was selected as the final model. Its strong performance is attributed to the clustered nature of network attack traffic — similar attacks produce similar patterns in flow duration, packet lengths, and inter-arrival times. Distance weighting further reinforces this by giving closer neighbors more influence.

All models struggled with minority classes (DoS Slowhttptest, DoS Slowloris, Heartbleed) due to the severe class imbalance. KNN was retrained on the full training set and predictions were exported to CSV.

---

## Repository Structure

```
├── data/
│   └── network_flows.csv          # Raw dataset (not included — see below)
├── notebooks/
│   └── intrusion_detection.ipynb  # Main analysis notebook
├── outputs/
│   └── predictions.csv            # Final KNN predictions on test set
├── README.md
└── requirements.txt
```

---

## Installation

```bash
git clone https://github.com/your-username/network-intrusion-detection.git
cd network-intrusion-detection
pip install -r requirements.txt
```

**Requirements:**
```
scikit-learn
pandas
numpy
matplotlib
seaborn
```

---

## Usage

Open and run the Jupyter notebook end-to-end:

```bash
jupyter notebook notebooks/intrusion_detection.ipynb
```

The notebook will:
1. Load and preprocess the dataset
2. Train all six models on a stratified subsample
3. Evaluate each model on the full test set
4. Generate visualizations (class distribution, histograms, correlation heatmap, boxplots, confusion matrices, ROC curves, precision-recall curves)
5. Retrain the best model (KNN) on the full training set and export predictions

---

## 📊 Evaluation Metric

**Weighted F1 Score** was chosen as the primary metric. It balances precision (cost of false alarms) and recall (cost of missed attacks), weighted by class size — making it appropriate for this severely imbalanced dataset.
