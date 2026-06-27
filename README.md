# Credit Default Risk Analysis & Prediction

A complete end-to-end data science project analysing credit card default risk using the UCI Credit Card Default dataset. The project covers exploratory data analysis in R, feature engineering and machine learning in Python, cloud-based model training in Databricks, and an interactive dashboard built in Tableau.

---

## Live Dashboard

[View the Tableau Dashboard](https://public.tableau.com/app/profile/dren.buqa/viz/CreditDefaultRiskAnalysis_17825611171240/Dashboard?publish=yes)

---

## Project Structure

```
credit-default-risk/
├── data/
│   └── credit_default.xls              # Raw UCI dataset (not tracked in Git)
├── r_analysis/
│   └── 01_eda.R                        # Exploratory data analysis in R
├── python/
│   ├── 02_feature_engineering.ipynb    # Feature engineering in Python
│   └── 03_model_training.ipynb         # Model training and evaluation
├── outputs/
│   ├── 01_default_distribution.png
│   ├── 02_default_by_education.png
│   ├── 03_default_by_age.png
│   ├── 04_credit_limit_distribution.png
│   ├── 05_correlation_heatmap.png
│   ├── 06_default_by_payment_status.png
│   ├── 07_roc_curves.png
│   ├── 08_feature_importance.png
│   ├── 09_confusion_matrix_xgb.png
│   ├── credit_default_clean.csv
│   ├── credit_default_engineered.csv
│   ├── model_results.csv
│   └── feature_importance.csv
└── README.md
```

---

## Dataset

- **Source:** [UCI Machine Learning Repository — Default of Credit Card Clients](https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients)
- **Size:** 30,000 clients, 25 features
- **Target variable:** `default` — whether a client defaulted on their next payment (1 = default, 0 = no default)
- **Key features:** Credit limit, age, education, marriage status, 6-month payment history, bill amounts, payment amounts

---

## Tools & Technologies

| Phase | Tools |
|---|---|
| Exploratory Data Analysis | R, tidyverse, ggplot2, corrplot, skimr |
| Feature Engineering | Python, pandas, numpy |
| Model Training | Python, scikit-learn, XGBoost |
| Cloud Execution | Databricks |
| Visualisation | Tableau Public |
| Version Control | Git, GitHub |

---

## Methodology

### Phase 1 — Exploratory Data Analysis (R)

Loaded and explored the raw dataset to understand distributions, correlations, and patterns before modelling.

**Key findings:**
- **22.1% of clients defaulted** — a meaningful class imbalance that shaped all modelling decisions
- Payment delay (PAY_0) is the strongest predictor: clients even 1 month delayed default at 33%, rising to 75%+ beyond 2 months
- Clients aged 51+ have the highest default rate (25.4%), contrary to the assumption that younger clients are riskier
- Lower credit limits strongly correlate with higher default risk

### Phase 2 — Feature Engineering (Python)

Engineered 7 new features from the raw payment and billing data to improve model performance:

| Feature | Description |
|---|---|
| `avg_payment_delay` | Average payment delay across 6 months |
| `months_delayed` | Number of months with a positive delay |
| `avg_bill_amt` | Average bill amount over 6 months |
| `avg_pay_amt` | Average payment amount over 6 months |
| `bill_to_limit_ratio` | How much of the credit limit is being used |
| `pay_to_bill_ratio` | How much of the bill is actually being paid (capped at 5x) |
| `consecutive_delay` | Whether PAY_0 and PAY_2 are both delayed |

### Phase 3 — Model Training & Evaluation (Python + Databricks)

Trained three classification models on an 80/20 train-test split with stratification to preserve class balance. Model training was reproduced in Databricks to demonstrate cloud-based ML execution.

**Why ROC-AUC over accuracy:**
A naive model predicting "No Default" for every client would achieve 77.9% accuracy — matching or beating a basic classifier. In credit risk, accuracy is misleading due to class imbalance. ROC-AUC measures the model's ability to distinguish between defaulters and non-defaulters regardless of threshold, making it the appropriate metric here.

**Why recall matters more than precision:**
Missing a real defaulter (false negative) costs a bank significantly more than flagging a good client as risky (false positive). Models were therefore evaluated with a focus on default recall alongside ROC-AUC.

---

## Results

### Model Comparison

| Model | ROC-AUC | Accuracy | Default Recall | Default Precision |
|---|---|---|---|---|
| Logistic Regression | 0.7398 | 81% | 28% | 65% |
| Random Forest | 0.7578 | 81% | 36% | 63% |
| XGBoost | 0.7529 | 75% | 57% | 45% |
| **XGBoost (threshold=0.3)** | **0.7529** | **63%** | **74%** | **34%** |

### Selected Model: XGBoost with threshold 0.3

XGBoost with a lowered decision threshold of 0.3 was selected as the final model. By reducing the threshold from the default 0.5 to 0.3, default recall improved from 57% to 74% — meaning the model correctly identifies 74% of clients who will default. The tradeoff is lower precision (more false alarms), which is acceptable in a banking context where the cost of missing a defaulter far outweighs the cost of additional scrutiny on a good client.

### Top Predictive Features (Random Forest)

1. PAY_0 — most recent payment status
2. avg_payment_delay — average delay across 6 months
3. months_delayed — number of months with delay
4. LIMIT_BAL — credit limit
5. PAY_AMT1, PAY_AMT2 — recent payment amounts

---

## Visualisations

All plots are saved in the `outputs/` folder and include:

- Default rate distribution (pie chart)
- Default rate by education level
- Default rate by age group
- Credit limit distribution by default status
- Correlation heatmap of key variables
- Default rate by payment status (PAY_0)
- ROC curves for all three models
- Feature importance from Random Forest
- Confusion matrix for XGBoost

---

## How to Run

### R Analysis
```r
# Install dependencies
install.packages(c("tidyverse", "ggplot2", "corrplot", "skimr", "readxl"))

# Run the EDA script
source("r_analysis/01_eda.R")
```

### Python Notebooks
```bash
# Install dependencies
pip install pandas numpy scikit-learn xgboost matplotlib seaborn

# Open notebooks in VS Code or Jupyter
jupyter notebook python/02_feature_engineering.ipynb
jupyter notebook python/03_model_training.ipynb
```

### Dataset
Download the dataset from the [UCI repository](https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients) and place it as `data/credit_default.xls`.

---

## Key Takeaways

- Payment delay history is by far the most powerful predictor of credit default — a client delayed by 2+ months has a 70%+ chance of defaulting
- Accuracy is a misleading metric for imbalanced classification problems; ROC-AUC and recall are more appropriate in credit risk contexts
- Threshold tuning is a practical and effective technique for aligning model behaviour with business priorities
- All three models achieve ROC-AUC between 0.74–0.76, well above the 0.50 random baseline, confirming genuine predictive power

---

## Author

**Dren Buqa**
B.Sc. Applied Data Science — Modul University Vienna
[GitHub](https://github.com/drenbuqa) · [Tableau Public](https://public.tableau.com/app/profile/dren.buqa)
