# Customer Churn Prediction (Python)

Python/scikit-learn pipeline for predicting bank customer churn, with fair model comparison, class-imbalance handling, and SHAP explainability.

## Dataset

[Bank Customer Churn Dataset](https://www.kaggle.com/datasets/radheshyamkollipara/bank-customer-churn) (Kaggle) — 10,000 rows, 18 features, binary `Exited` target (~20.4% churn rate).

## Requirements

```bash
pip install pandas numpy scikit-learn xgboost shap matplotlib seaborn scipy openpyxl
```

Developed and tested with:
- Python 3.13
- scikit-learn ≥ 1.4
- xgboost ≥ 2.0
- shap ≥ 0.44

## How to Run

1. Download `Churn_Modelling.csv` from the Kaggle dataset link above and place it in the project directory (or update the file path in the first code cell).
2. Open `Customer_churn_prediction.ipynb` in Jupyter, VS Code, or Kaggle Notebooks.
3. Run cells top to bottom — the notebook is sequential and each section depends on variables created earlier (no need to skip around).
4. The final cells export `churn_dashboard_data.csv`, used as the Power BI data source (see the separate dashboard README/repo if applicable).

## Notebook Structure

| Section | What It Does |
|---|---|
| 1. Data Loading & Cleaning | Load CSV, check nulls/dtypes, drop non-predictive ID columns |
| 2. Exploratory Data Analysis | Churn rate by Geography, Gender, Card Type, Age |
| 3. Hypothesis Testing | Chi-square tests (categorical drivers) and t-tests (continuous drivers) to statistically validate churn drivers before modeling |
| 4. Feature Engineering | `Age_Category` binning; ratio features (`Balance_to_Salary_Ratio`, `Balance_per_Product`, `Tenure_Age_Ratio`) — tested but not used in the final model (see Notes) |
| 5. Preprocessing | `ColumnTransformer` — `OneHotEncoder` for categoricals, `StandardScaler` for Logistic Regression only (tree models skip scaling) |
| 6. Train/Test Split | 80/20 stratified split, held out before any tuning |
| 7. Model Comparison | Logistic Regression, Random Forest, XGBoost — each wrapped in a `Pipeline`, compared via 5-fold `StratifiedKFold` cross-validation |
| 8. Hyperparameter Tuning | `GridSearchCV` (XGBoost) and `RandomizedSearchCV` (Random Forest) — **both** tuned, not just the winning model, to keep the comparison fair |
| 9. Class Imbalance | `scale_pos_weight` applied to XGBoost (≈3.9, derived from the training set's class ratio) |
| 10. Threshold Selection | Precision/recall tradeoff evaluated across thresholds 0.2–0.7; 0.6 selected to balance false negatives against false positives |
| 11. Final Evaluation | Confusion matrix, classification report, ROC-AUC on the held-out test set |
| 12. Risk Segmentation | Full-dataset scoring with the trained model, binned into 4 risk tiers (Low / Medium / High / Very High) |
| 13. SHAP Explainability | Global summary, per-risk-tier summaries, and individual-customer waterfall plots |
| 14. Business Recommendations | Markdown summary tying statistical findings and model output to specific retention actions |
| 15. Export | Final scored dataset written to CSV for Power BI |

## Key Design Decisions (and Why)

- **Pipelines everywhere:** scaling and encoding are fit only on training folds, preventing data leakage during cross-validation — this was a deliberate fix after an earlier version manually scaled the full dataset before splitting.
- **All 3 models tuned, not just XGBoost:** an earlier iteration compared a tuned XGBoost against *untuned* Random Forest/Logistic Regression, which is not a fair comparison. Random Forest was re-tuned with `RandomizedSearchCV` before the final model choice was confirmed.
- **Threshold ≠ 0.5:** the default classification threshold was deliberately overridden after evaluating the business cost of false negatives (lost customers) vs. false positives (unnecessary retention offers).
- **`X` passed to pipelines is always raw/unencoded:** the `ColumnTransformer` inside each `Pipeline` handles encoding internally, so raw categorical text columns (e.g., `Geography = "Germany"`) should be passed directly — do not pre-encode before calling `.fit()`.

## Results

| Model | Tuned 5-fold CV ROC-AUC |
|---|---|
| Logistic Regression | 0.781 |
| Random Forest | 0.858 |
| **XGBoost (final)** | **0.864** |

**Final model on held-out test set:** ROC-AUC 0.879, Recall 69%, Precision 60% (threshold = 0.6)

## Notes / Known Limitations

- The three engineered ratio features (`Balance_to_Salary_Ratio`, `Balance_per_Product`, `Tenure_Age_Ratio`) produced a negligible change in ROC-AUC (0.8623 → 0.8638) and are not required for the final model — likely because XGBoost's tree splits already capture these interactions implicitly from the raw features. They remain in the exported dataset for dashboard readability.
- `Age_Category` binning did produce a real improvement (test ROC-AUC 0.876 → 0.879) and revealed a non-linear churn spike (51.1%) in the 41–60 band that the raw `Age` average did not show — this is the more valuable engineering result of the two.
- CLV/margin figures referenced in the business recommendations section are assumption-based (no true revenue data in the source dataset) and should be treated as illustrative, not exact.

## License / Data Source

Dataset used under Kaggle's dataset terms: [radheshyamkollipara/bank-customer-churn](https://www.kaggle.com/datasets/radheshyamkollipara/bank-customer-churn). Code in this repository is available for personal/educational use.
- **Inactive members churn significantly more** than active members
- **Older, higher-balance customers churn more** — a counter-intuitive pattern worth targeted retention focus
- **Card Type has no statistically significant effect** on churn

Full statistical results, exact test values, and complete reasoning are documented inline throughout the notebook.
