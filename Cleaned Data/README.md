# Customer_Churn_Predictions.csv — Data Dictionary

Final scored output of the churn prediction model (`Customer_churn_prediction.ipynb`). This is the file used as the data source for the Power BI dashboard.

- **Rows:** 10,000 (full customer base — not just the test set)
- **Columns:** 25
- **Source dataset:** [Bank Customer Churn Dataset](https://www.kaggle.com/datasets/radheshyamkollipara/bank-customer-churn) (Kaggle), enriched with engineered features and model predictions

> **Important:** predictions in this file come from a model trained on 80% of the data (`X_train`) and then applied to the full 10,000 rows for business scoring purposes. This is standard practice for generating production-style risk scores, but it means the accuracy/precision/recall figures reported in the project README are based on the held-out test set only, not on this full file — see [Notes](#notes-on-usage) below.

## Column Reference

### Identifiers
| Column | Type | Description |
|---|---|---|
| `Rownumber` | int | Row index from the original source file |
| `Customerid` | int | Unique customer identifier |
| `Surname` | string | Customer surname (retained from source data; not used as a model feature) |

### Raw Customer Attributes (from source dataset)
| Column | Type | Description |
|---|---|---|
| `Creditscore` | int | Customer's credit score |
| `Geography` | string | Country: France, Germany, or Spain |
| `Gender` | string | Male or Female |
| `Age` | int | Customer age in years |
| `Tenure` | int | Years as a customer with the bank |
| `Balance` | float | Account balance |
| `Numofproducts` | int | Number of bank products held (1–4) |
| `Hascrcard` | int (0/1) | Whether the customer has a credit card |
| `Isactivemember` | int (0/1) | Whether the customer is an active member |
| `Estimatedsalary` | float | Estimated annual salary |
| `Exited` | int (0/1) | **Target variable** — actual churn outcome (1 = churned) |
| `Complain` | int (0/1) | Whether the customer has filed a complaint |
| `Satisfaction Score` | int | Customer satisfaction score (from complaint resolution, 1–5) |
| `Card Type` | string | Diamond, Gold, Platinum, or Silver |
| `Point Earned` | int | Loyalty points earned |

### Engineered Features
| Column | Type | Description | Used in Final Model? |
|---|---|---|---|
| `Age_Category` | string | Age binned into `Young` (<30), `Adult` (30–44), `Middle_aged` (45–59), `Senior` (60+) | **Yes** — improved test ROC-AUC 0.876 → 0.879 and revealed the 45–59 churn spike |
| `Balance_to_Salary_Ratio` | float | `Balance / Estimatedsalary` | Tested, negligible model impact — retained for dashboard context only |
| `Balance_per_Product` | float | `Balance / Numofproducts` | Tested, negligible model impact — retained for dashboard context only |
| `Tenure_Age_Ratio` | float | `Tenure / Age` | Tested, negligible model impact — retained for dashboard context only |

### Model Output
| Column | Type | Description |
|---|---|---|
| `Predicted_churn_Probability` | float (0–1) | XGBoost's predicted probability of churn |
| `Predicted_churn` | int (0/1) | Binary prediction at the chosen decision threshold (**0.6**) |
| `Risk Category` | string | Risk tier derived from `Predicted_churn_Probability`: Low Risk (<0.25), Medium Risk (0.25–0.5), High Risk (0.5–0.75), Very High Risk (≥0.75) |

## Notes on Usage

- **`Predicted_churn` and `Risk Category` reflect a threshold of 0.6**, chosen deliberately to balance false negatives (missed churners) against false positives (unnecessary retention spend) — see the main project README for the reasoning.
- **This file includes predictions for customers the model was trained on** (the original 8,000-row training split is part of these 10,000 rows). This is intentional for generating full-base risk scores for dashboard/business use, but means **this file should not be used to recompute or report model accuracy metrics** — those are documented separately from the held-out test set only (ROC-AUC 0.879, Recall 69%, Precision 60%).
- `Exited` is the ground-truth outcome; `Predicted_churn` is the model's guess. Comparing the two directly in this file will show inflated agreement for the ~80% of rows that were in the training set.

## Risk Tier Distribution (Full 10,000 Rows)

| Risk Category | Count | % of Base |
|---|---|---|
| Low Risk | 3,996 | 39.96% |
| Medium Risk | 3,020 | 30.20% |
| High Risk | 1,615 | 16.15% |
| Very High Risk | 1,369 | 13.69% |

## Related Files

- `Customer_churn_prediction.ipynb` — generates this file (see its README for full modeling methodology)
- `Customer_Churn_Dashboard.pbix` — Power BI dashboard consuming this file as its data source

---

## Used By

- `Customer_Churn_Prediction.pbix` — Power BI dashboard (Executive Summary, Risk Segmentation, and Priority Customer List pages all read from this file)
