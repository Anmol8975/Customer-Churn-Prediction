# Customer Churn — Power BI Dashboard

A 3-page interactive Power BI dashboard built on top of the churn prediction model's output, designed for a retention team to prioritize and act on at-risk customers — not just view analytics.

## Data Source

`churn_dashboard_data.csv` — the scored output of `Customer_churn_prediction.ipynb` (XGBoost model, test ROC-AUC 0.879), containing per-customer churn probability, risk tier, and original demographic/account features.

Original raw dataset: [Bank Customer Churn Dataset](https://www.kaggle.com/datasets/radheshyamkollipara/bank-customer-churn) (Kaggle).

## File

- `Customer_Churn_Dashboard.pbix` — the Power BI report file (open in Power BI Desktop)
- `Churn_Risk_Dashboard_Theme.json` — custom report theme (blue/white, applied via View → Themes → Browse for themes)

## Pages

### 1. Executive Summary
Top-line view for a quick health check on churn across the customer base.
- **KPI cards:** Churn Rate, Total Customers, Churn Customers, AVG Balance, AVG Credit Score, AVG Estimated Salary
- **Customer Metrics by Geography** — table (Churn Rate, Total/Churn Customers, AVG Balance/Points/Credit Score per country)
- **Churn Rate by Risk Category** — bar chart, validates the model's tiers against actual outcomes
- **Churn Rate by Age** — scatter plot, shows the full non-linear age pattern (peak in the 40s–50s)
- **Churn Rate by Gender** — donut
- **Churn Rate by Num of Products** — bar chart (flags the 3–4 product anomaly)
- **Slicers:** Gender, Age Group, Geography, Risk Category

### 2. Risk Segmentation
Deeper cut into how risk tiers relate to other customer attributes.
- **Risk Profile table** — Total/Churn Customers, Churn Rate, AVG Age/Balance/Salary/Tenure, per risk tier
- **Churn Rate by Age Group** — bar chart (0–30 / 31–40 / 41–60 / 60+)
- **Churn Rate by Age Group and Risk Category** — clustered column, cross-cuts the age spike against risk tiers
- **Churn Rate by Active_Status** — donut (Active vs Inactive)
- **Churn Rate by Geography** — bar chart
- **Slicers:** Gender, Geography, Risk Category

### 3. Priority Customer List
Action-oriented page for the retention team — not just analysis, a working prioritization tool.
- **Risk tier toggle buttons:** Low / Medium / High / Very High Risk Customers (bookmark-driven filter buttons)
- **Top 50 Priority Customers table:** Customer ID, Churn Probability, Gender, Priority Score, Geography, Age Group, Is_Activemember, Num_of_products — sorted descending by Priority Score
- **KPI cards:** Total Customers, Churn Rate, AVG Age (all update with the selected risk tier)
- **SHAP Summary visual (Python visual):** live SHAP beeswarm plot for the currently filtered risk tier, explaining which features are driving predictions for that segment
- **Slicers:** Geography, Gender

## Key Measures (DAX)

```dax
Churn Rate = DIVIDE(SUM(churn_data[Exited]), COUNTROWS(churn_data))

High Risk Count = 
CALCULATE(COUNTROWS(churn_data), churn_data[Risk Category] IN {"High Risk", "Very High Risk"})

Priority Score = 
('churn_data'[Balance] * 'churn_data'[Churn Probability]) / 1000

Age_Group = 
SWITCH(
    TRUE(),
    churn_data[Age] <= 30, "0-30",
    churn_data[Age] <= 45, "31-45",
    churn_data[Age] <= 60, "46-60",
    "60+"
)

Active_Status = IF(churn_data[Isactivemember] = 1, "Active", "Inactive")
```

> Note: `Age_Group` is sorted using `Age_Group_Sort` (Column tools → Sort by column) so it displays in logical order (0–30 → 31–45 → 46–60 → 60+) rather than Power BI's default alphabetical/value sort.

## The SHAP Visual (Python Visual)

The SHAP summary chart on the Priority Customer List page is a **Python visual**, which requires:
1. Python enabled in Power BI Desktop (File → Options → Python scripting, pointing to an environment with `shap`, `pandas`, `matplotlib`, `xgboost` installed)
2. The trained model's SHAP values (or the model + preprocessor) available to the script — either precomputed and joined into the dataset, or recomputed live inside the visual's script
3. Because Python visuals can be slow to re-render, filtering (via the risk-tier buttons) may take a few seconds to update — this is expected Power BI/Python visual behavior, not a bug

If Python visuals aren't available in your environment, substitute a static SHAP image (exported from the notebook) per risk tier, swapped via bookmarks tied to the same toggle buttons.

## How to Use This Dashboard

1. Open `Customer_Churn_Dashboard.pbix` in Power BI Desktop
2. If prompted, update the data source path to point to your local copy of `churn_dashboard_data.csv`
3. Click **Refresh** to load current data
4. Apply the theme: **View → Themes → Browse for themes → `Churn_Risk_Dashboard_Theme.json`** (if not already embedded)
5. Use the top-right slicers on each page, or the risk-tier buttons on Page 3, to filter

## Design Notes

- All three pages share consistent slicers (Gender, Geography, Risk Category where applicable) for cross-page filtering consistency
- KPI cards are intentionally duplicated with page-relevant context (e.g., AVG Credit Score on the Executive Summary vs. AVG Points Earned on Risk Segmentation) rather than reused identically, so each page's cards support that page's specific narrative
- Card Type was tested as a driver but found to be weak/flat (19–22% churn range across all 4 types) — kept as a minor supporting chart rather than a headline visual

## Known Limitations

- `Priority Score` (`Balance × Churn Probability`) is a simplified value-at-risk proxy — a CLV-weighted version would be more rigorous but requires assumed margin figures not present in the source data (see the Python notebook's README for the CLV formula used elsewhere in this project)
- The SHAP Python visual's feature names show internal preprocessing prefixes (`cat__`, `remainder__`) as generated by scikit-learn's `ColumnTransformer.get_feature_names_out()`; strip these in the export step for a cleaner label if presenting externally
---

