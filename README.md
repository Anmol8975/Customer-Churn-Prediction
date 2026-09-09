# Customer Churn Prediction & Retention Analytics

An end-to-end churn prediction system for a bank's retail customers — covering model development, fairness-checked model selection, explainability, and a 3-page interactive Power BI dashboard for retention teams.

## Dataset

[Bank Customer Churn Dataset](https://www.kaggle.com/datasets/radheshyamkollipara/bank-customer-churn) — 10,000 customer records with demographic, account, and behavioral attributes, and a binary churn (`Exited`) label.

## Project Overview

This project goes beyond building a single classifier — it's structured as a full workflow a risk/retention analytics team would actually use:

1. **Exploratory & statistical analysis** — hypothesis testing (chi-square, t-tests) to validate churn drivers before modeling
2. **Feature engineering** — domain-driven ratios (Balance-to-Salary, Balance-per-Product, Tenure-to-Age) and Age binning
3. **Fair model comparison** — Logistic Regression, Random Forest, and XGBoost, each properly tuned (not just the final pick) before declaring a winner
4. **Class imbalance handling** — `scale_pos_weight` to correct for the ~20% churn base rate
5. **Business-driven threshold selection** — explicit precision/recall tradeoff reasoning, not a default 0.5 cutoff
6. **Explainability** — SHAP analysis at the global, risk-tier, and individual customer level
7. **Deployment-style output** — a Power BI dashboard built for retention teams to act on, not just a notebook

## Key Findings

| Finding | Detail |
|---|---|
| **Age is non-linear** | Churn spikes to **51.1%** in the 41–60 age band — 3–4x every adjacent group. The raw average-age comparison (44.8 vs 37.4) completely hid this. |
| **More products ≠ more loyalty** | Churn is lowest at 2 products (7.6%) but rises sharply to 82.7% at 3 products and 100% at 4 — the opposite of the intuitive assumption. |
| **Geography** | Germany churns at 32.4%, roughly double France (16.2%) and Spain (16.7%), p < 0.001. |
| **Inactivity** | Inactive members churn at 26.9% vs. 14.3% for active members. |
| **Model validation holds up** | Customers flagged "Very High Risk" by the model actually churned at 75.97%, vs. 2.58% for "Low Risk" — confirming the risk tiers reflect real outcomes, not just a good AUC number. |

## Modeling Approach

- **Pipeline:** `ColumnTransformer` (OneHotEncoder for categoricals, StandardScaler for Logistic Regression) + classifier, wrapped in a single scikit-learn `Pipeline` to prevent data leakage across cross-validation folds.
- **Model comparison:** All three candidate models — not just XGBoost — were tuned via `GridSearchCV` / `RandomizedSearchCV` with 5-fold `StratifiedKFold` before comparing scores, to avoid an unfair "tuned vs. untuned" comparison.

| Model | Tuned CV ROC-AUC |
|---|---|
| Logistic Regression | 0.781 |
| Random Forest | 0.858 |
| **XGBoost (final)** | **0.864** |

- **Class imbalance:** `scale_pos_weight` (≈3.9, calculated from the train split's class ratio) applied to XGBoost.
- **Threshold:** 0.6 was chosen deliberately over the default 0.5 — balancing false negatives (missed churners) against false positives (wasted retention spend), rather than optimizing for accuracy alone.
- **Final test performance:** ROC-AUC **0.879**, Recall 69%, Precision 60% at threshold 0.6.

## Explainability

SHAP (`TreeExplainer`) was used at three levels:
- **Global** — overall feature importance across all customers
- **Per risk tier** — separate SHAP summaries for Low / Medium / High / Very High Risk segments, to see whether churn drivers differ by risk level
- **Individual customer** — waterfall plots explaining specific predictions, supporting case-by-case review

## Power BI Dashboard

A 3-page interactive dashboard built from the model's output:

1. **Executive Summary** — top-line KPIs, churn rate by geography/age/gender/product count
2. **Risk Segmentation** — risk-tier profiles, age × risk category breakdown, activity status and geography cross-cuts
3. **Priority Customer List** — a filterable, ranked action list (`Priority Score = Balance × Churn Probability`) with a live embedded SHAP summary visual, so retention teams see both *who* to prioritize and *why*

All pages are cross-filtered by Gender, Geography, and Risk Category, with a custom theme.

## Tech Stack

**Python:** pandas, scikit-learn, XGBoost, SHAP, matplotlib, scipy (`chi2_contingency`, `ttest_ind`)
**BI:** Power BI (DAX measures, calculated columns, cross-page filtering)
**Validation:** `StratifiedKFold`, `GridSearchCV`, `RandomizedSearchCV`

## Repository Structure

```
├── Customer_churn_prediction.ipynb   # Full analysis: EDA, hypothesis testing, modeling, SHAP
├── churn_dashboard_data.csv          # Model output exported for Power BI
├── Churn_Risk_Dashboard.pbix         # Power BI dashboard (3 pages)
├── Churn_Risk_Dashboard_Theme.json   # Custom Power BI theme
└── README.md
```

## Business Recommendations (Summary)

- Treat the **41–60 age band** as the single highest-priority segment for retention investment
- Audit the customer journey for **3–4 product holders** — near-universal churn suggests over-extension or mis-selling rather than loyalty
- Prioritize **Germany** and **inactive members** for targeted outreach
- Validate any retention offer with an **A/B test** before scaling to the full flagged population (~30% of customers)

## Limitations

- CLV and margin figures used in prioritization are **assumption-based estimates** (no real revenue data in the source dataset) and should be treated as illustrative of the framework, not precise financial figures.
- Engineered ratio features (Balance-to-Salary, Balance-per-Product, Tenure-to-Age) did not meaningfully improve XGBoost performance (likely because tree-based splits already captured the interaction) — kept for dashboard readability, not model lift.

## Author

**Anmol Verma**
Data Analyst | Risk Analyst

