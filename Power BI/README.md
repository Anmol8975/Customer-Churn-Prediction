# Customer Churn — Power BI Dashboard

`Customer_Churn_Prediction.pbix`

An interactive, three-page Power BI dashboard built on the output of the churn prediction model (`Customer_Churn_Scored_PowerBI.csv`). It translates model predictions into a business-facing tool for exploring churn risk and prioritizing retention action — no Python knowledge required to use it.

---

## Data Source

This dashboard reads from `Customer_Churn_Scored_PowerBI.csv`, which contains the original customer dataset plus two model-generated columns:
- `Churn_Probability` — predicted likelihood of churn (0–1), from the final XGBoost model
- `Churn_Risk` — risk tier derived from that probability (Low / Medium / High / Very High)

See the dataset's own README for full column definitions and important usage notes (in particular, the `Complain` column should not be used in any analysis — it is a data leakage artifact retained only for traceability).

---

## Pages

### 1. Executive Summary
A high-level, at-a-glance view of the overall customer base.

- **KPI cards**: Total Customers, Total Churn Customers, Churn Rate, Balance at Risk, Expected Loss, Average Churn Probability
- **Total Customers by Churn Risk** — population distribution across tiers
- **Gender split** — customer composition by gender
- **Card Type table** — customer count, average points, average balance, and churn rate by card type
- **Churn Rate by Age** (scatter plot) — visualizes how churn risk rises and peaks across age groups
- **Churn Rate by Geography** (map) — geographic view highlighting Germany's disproportionately high churn rate

*Purpose: answer "what's the overall situation?" in under 30 seconds.*

### 2. Risk Segmentation
A detailed breakdown of churn risk by tier and key drivers.

- **KPI cards** (same core metrics as page 1, filterable independently)
- **Risk Profile table** — Total Customers, Average Age, Churn Rate, Expected Loss, and average behavioral metrics, broken down by risk tier
- **Churn Rate by Gender** (donut)
- **Churn Rate by Geography** (bar chart)
- **Churn Rate by Churn Risk** (donut) — validates the tier definitions against actual churn outcomes
- **Churn Rate by Number of Products** (bar chart) — the dashboard's most striking finding: churn rate rises from 7.6% (2 products) to 100% (4 products), with customer counts shown via tooltip for transparency

*Purpose: answer "how does risk break down, and what's driving it?"*

### 3. Priority Customer List
An actionable, filterable list for the retention team.

- **Bookmark toggle buttons**: switch between "High Risk" and "Very High Risk" customer views (also drives which SHAP image is displayed)
- **KPI cards** (dynamic, reflect only the currently filtered/selected group): Total Customers, Average Churn Probability, Sum of Priority Score
- **Top 50 Priority Customers table** — sorted by `Priority Score` (Churn Probability × Balance), showing Customer ID, Churn Probability, Balance, Age, Geography, Activity Status, and Number of Products
- **SHAP Summary image** — embedded static SHAP plot (generated in Python) explaining the top churn drivers specifically within the currently selected risk tier
- **Slicers**: Churn Risk, Gender, Geography — for further drill-down

*Purpose: answer "who should we act on first, and why?"*

---

## Interactivity

- All visuals are cross-filterable — clicking any chart element filters the rest of the page
- Slicers for Geography, Gender, Churn Risk, and Card Type are available across pages
- Bookmarks on the Priority Customer List page toggle between High and Very High risk views, including swapping the embedded SHAP image

---

## Key DAX Measures

| Measure | Logic |
|---|---|
| Churn Rate | Churned Customers ÷ Total Customers |
| Balance at Risk | Sum of Balance where Churn Risk is High or Very High |
| Expected Loss | Sum of (Churn Probability × Balance) across all customers |
| Priority Score | Churn Probability × Balance (calculated column) — used to rank the Priority Customer List |

---

## Validation

All churn rate and count figures in this dashboard were manually cross-checked against the source Python notebook's calculations to confirm consistency — e.g., churn rate by risk tier and by number of products matched exactly between the two tools.

---

## Theme

A custom theme (`Churn_Analytics_Theme.json`) was applied for consistent typography (Segoe UI) and a coordinated blue/teal/coral color palette across all visuals, KPI cards, and tables.

---

## How to Use

1. Open `Customer_Churn_Prediction.pbix` in Power BI Desktop
2. If prompted, update the data source path to point to your local copy of `Customer_Churn_Scored_PowerBI.csv`
3. Use the slicers and bookmark buttons on each page to explore the data — no editing required for standard use
4. To refresh with new scored data, replace the CSV and click **Refresh** in the Home ribbon

---

## Companion Files

- `Customer_churn_prediction.ipynb` — Python notebook containing the full modeling, statistical testing, and SHAP analysis behind this dashboard's data
- `Customer_Churn_Scored_PowerBI.csv` — the underlying scored dataset
