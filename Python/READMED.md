# Customer Churn Prediction — Python Notebook

`Customer_churn_prediction.ipynb`

This notebook contains the full Python-based analysis for predicting bank customer churn — covering data cleaning, statistical hypothesis testing, model building, and explainability with SHAP. It is the technical/analytical core of the broader Customer Churn Prediction & Retention Strategy project (paired with a Power BI dashboard built on its output).

---

## What This Notebook Does

1. Cleans and validates a 10,000-row bank customer dataset
2. Statistically tests which customer attributes actually drive churn (not just visual guesswork)
3. Trains and compares three classification models
4. Selects a final model based on business-relevant tradeoffs, not just the highest accuracy metric
5. Explains model predictions using SHAP — at both the population and individual customer level
6. Scores the entire customer base and segments it into actionable risk tiers
7. Closes with concrete, tier-specific business recommendations

---

## Requirements

```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy imbalanced-learn xgboost shap
```

**Libraries used:**
- `pandas`, `numpy` — data manipulation
- `matplotlib`, `seaborn` — visualization
- `scikit-learn` — preprocessing, modeling, evaluation metrics
- `scipy.stats` — chi-square and t-test hypothesis testing
- `imbalanced-learn` — SMOTE / class imbalance handling (referenced; class-weighting was ultimately used)
- `xgboost` — final gradient boosting model
- `shap` — model explainability

---

## Dataset

**Input file**: `Customer-Churn-Records.csv` (10,000 rows)

Key columns: `CreditScore`, `Geography`, `Gender`, `Age`, `Tenure`, `Balance`, `NumOfProducts`, `HasCrCard`, `IsActiveMember`, `EstimatedSalary`, `Satisfaction Score`, `Card Type`, `Point Earned`, and the target `Exited` (1 = churned, 0 = retained).

---

## Notebook Structure

### 1. Data Cleaning
- Standardizes text formatting across categorical columns
- Removes duplicate customer records
- Confirms zero missing values
- **Detects and removes a data leakage column** (`Complain`) after finding a 99.5% correlation with the target — this field was almost certainly recorded as a result of the churn decision, not a predictor of it
- Drops non-predictive identifier columns (`RowNumber`, `CustomerId`, `Surname`)

### 2. Exploratory Data Analysis & Hypothesis Testing
- IQR-based outlier screening across numeric features — all flagged values were confirmed as legitimate (not errors) and retained
- Boxplot visualizations comparing Age, Balance, and CreditScore distributions between churned and retained customers
- **Chi-square tests** (categorical variables: Geography, Gender, Card Type, IsActiveMember, NumOfProducts)
- **Independent t-tests** (continuous variables: Age, Balance, CreditScore)
- Each test follows an explicit H0/H1 hypothesis structure with a stated significance threshold (α = 0.05)

### 3. Model Preparation
- One-hot encodes `Geography`, `Gender`, `Card Type` (`drop_first=True`)
- 80/20 stratified train-test split
- Feature scaling via `StandardScaler`, fit on training data only

### 4. Model Training and Evaluation
Three models trained and evaluated under a consistent framework:
- Logistic Regression (`class_weight='balanced'`)
- Random Forest (`class_weight='balanced'`, threshold-tuned)
- XGBoost (`scale_pos_weight`-balanced, threshold-tuned)

Each model is evaluated via confusion matrix, classification report, ROC-AUC, and a precision-recall threshold sweep — not just a default 0.5 cutoff.

**Final model: XGBoost**, selected for its precision/recall balance and efficiency, with reasoning documented in-notebook.

### 5. Full Dataset Scoring & Risk Segmentation
- Applies the final model to all 10,000 customers (not just the test set) to simulate real-world deployment scoring
- Buckets every customer into a **Low / Medium / High / Very High** risk tier based on predicted churn probability
- Produces a per-tier profile summary (average Age, Balance, Activity Status, etc.)

### 6. SHAP Explainability
- Global SHAP summary plots — overall churn drivers across the full customer base
- Per-tier SHAP summary plots — how driver importance shifts as risk increases
- Individual force plots — explaining specific customer-level predictions (e.g., why one customer was flagged at 82% churn risk)

### 7. Business Recommendations & Conclusion
- Summary table of all statistically validated churn drivers
- Risk tier profile summary
- Tier-specific retention recommendations
- Closing synthesis connecting statistical findings, model output, and business action

---

## How to Run

1. Place `Customer-Churn-Records.csv` in the same directory as the notebook
2. Install the required libraries (see above)
3. Run all cells top to bottom — the notebook is fully sequential; later sections depend on variables/models defined earlier
4. Note: SHAP computation cells may take a few seconds longer on first run

---

## Key Outputs

- `Churn_Probability` and `Churn_Risk` columns appended to the full customer dataset — exported for use in the companion Power BI dashboard
- SHAP summary plots (optionally exported as PNG for embedding in Power BI)
- A complete, documented model comparison and selection rationale

---

## Headline Findings

- **NumOfProducts is the single strongest churn driver**: churn rate rises from 7.6% (2 products) to 100% (4 products)
- **Germany has roughly double the churn rate** of France and Spain
- **Inactive members churn significantly more** than active members
- **Older, higher-balance customers churn more** — a counter-intuitive pattern worth targeted retention focus
- **Card Type has no statistically significant effect** on churn

Full statistical results, exact test values, and complete reasoning are documented inline throughout the notebook.
