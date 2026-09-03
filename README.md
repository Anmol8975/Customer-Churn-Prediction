# Customer Churn Prediction & Retention Strategy

An end-to-end churn analytics project for a retail bank — from raw data to a business-ready decision tool. This project predicts which customers are likely to leave, statistically validates *why*, and translates model output into a prioritized, actionable retention strategy via an interactive Power BI dashboard.

---

## Business Context

Customer churn is costly for banks — acquiring a new customer typically costs far more than retaining an existing one. This project analyzes a bank's customer data to:
- Predict which customers are likely to churn
- Understand the statistically validated drivers behind churn
- Segment the customer base into actionable risk tiers
- Recommend specific, targeted retention actions for each tier

---

## Dataset

10,000 customer records with demographic, financial, and behavioral features.

| Column | Description |
|---|---|
| CreditScore | Customer's credit score |
| Geography | Country (France, Germany, Spain) |
| Gender | Male / Female |
| Age | Customer age |
| Tenure | Years as a bank customer |
| Balance | Account balance |
| NumOfProducts | Number of bank products held |
| HasCrCard | Whether customer holds a credit card |
| IsActiveMember | Whether customer is an active member |
| EstimatedSalary | Estimated annual salary |
| Satisfaction Score | Self-reported satisfaction (1–5) |
| Card Type | Diamond / Gold / Platinum / Silver |
| Point Earned | Loyalty points earned |
| **Exited** | **Target variable** (1 = churned, 0 = retained) |

---

## Tools & Libraries

- **Python**: pandas, NumPy, scikit-learn, XGBoost, SHAP, SciPy, Matplotlib, Seaborn
- **Power BI**: Power Query, DAX, interactive visuals

---

## Workflow

### 1. Data Cleaning

- Standardized text casing/whitespace across categorical columns (`Geography`, `Gender`, `Card Type`)
- Removed duplicate records (deduplicated on `CustomerId`)
- Confirmed zero missing values across all columns
- **Data leakage detection**: found that the `Complain` column was 99.5% correlated with the target (`Exited`) — customers who complained churned almost every time, and vice versa. This strongly suggested the field was recorded *after* the churn decision (e.g., logged during account closure), not a genuine predictive signal. **Excluded `Complain`, along with `RowNumber`, `CustomerId`, and `Surname`, from modeling.**

### 2. Outlier Analysis

Applied the IQR (Interquartile Range) method across all numeric features. Findings:
- Flagged "outliers" in Age, CreditScore, Balance, and NumOfProducts were all legitimate real-world values (e.g., older customers, high credit scores, customers holding multiple products) rather than data entry errors.
- **Decision: retained all flagged values** — removing them would have introduced bias against real customer segments, and tree-based models handle such values natively.

### 3. Exploratory Data Analysis & Statistical Hypothesis Testing

Used chi-square tests for categorical variables and independent t-tests for continuous variables (α = 0.05) to validate churn drivers rather than relying on visual inspection alone.

| Variable | Test | Statistic | p-value | Result | Direction |
|---|---|---|---|---|---|
| NumOfProducts | Chi-square | 1501.50 | ~0.0 | **Significant** | Non-linear — very strong effect |
| Geography | Chi-square | 300.63 | 5.25e-66 | **Significant** | Germany ≈ 2× churn rate of France/Spain |
| IsActiveMember | Chi-square | 243.69 | 6.15e-55 | **Significant** | Inactive members churn far more |
| Gender | Chi-square | 112.40 | 2.93e-26 | **Significant** | Female > Male churn rate |
| Age | T-test | 29.76 | ~0.0 | **Significant** | Churned customers ~7.4 years older on average |
| Balance | T-test | 11.94 | 1.2e-32 | **Significant** | Churned customers hold higher balances |
| CreditScore | T-test | -2.68 | 0.0074 | Significant, but small effect | Minimal practical impact |
| Card Type | Chi-square | 5.05 | 0.168 | **Not significant** | No meaningful relationship to churn |

**Headline finding**: Churn rate by number of products held is dramatic and non-linear:

| Products | Customers | Churn Rate |
|---|---|---|
| 1 | 5,084 | 27.7% |
| 2 | 4,590 | 7.6% |
| 3 | 266 | 82.7% |
| 4 | 60 | 100.0% |

Customers with 3–4 products — despite being only 3.3% of the customer base — churn at 83–100%, representing a small but near-certain loss segment.

### 4. Model Preparation

- One-hot encoded categorical variables (`Geography`, `Gender`, `Card Type`) with `drop_first=True` to avoid multicollinearity
- 80/20 stratified train-test split (preserving the ~20% churn rate in both sets)
- Standardized numeric features using `StandardScaler`, fit on training data only to prevent data leakage into the test set

### 5. Model Training & Evaluation

Three classification models were trained and evaluated under a consistent framework, with threshold tuning (beyond the naive 0.5 default) to properly balance precision and recall for a moderately imbalanced target (~20% churn).

| Model | Precision | Recall | ROC-AUC |
|---|---|---|---|
| Logistic Regression (class-weighted) | 0.39 | 0.72 | 0.780 |
| Random Forest (threshold = 0.2) | 0.46 | 0.79 | **0.868** |
| XGBoost (threshold = 0.4) | 0.49 | 0.75 | 0.855 |

**Final model: XGBoost**, selected for its strong precision/recall balance at a practical operating threshold and fast training time, despite Random Forest holding a marginally higher AUC — the decision was based on operating-point performance and deployment considerations rather than a single metric.

### 6. Full-Dataset Risk Scoring & Segmentation

The final model was applied to the complete 10,000-customer base (not just the held-out test set) to simulate real deployment, where every current customer needs a churn score — not just previously unseen ones.

Customers were segmented into four risk tiers based on predicted churn probability:

| Risk Tier | Customers | Avg. Age | Churn Rate | Key Traits |
|---|---|---|---|---|
| Low | 6,548 | 36.2 | 1.41% | Younger, active, healthy segment |
| Medium | 1,339 | 41.2 | 14.94% | Emerging age/inactivity signal |
| High | 988 | 42.3 | 68.22% | Concentrated in Germany, less active |
| Very High | 1,125 | 48.9 | 95.29% | Oldest, least active, highest product count & balance |

### 7. SHAP Explainability

Applied SHAP (`TreeExplainer`) at both the population and individual-customer level to move beyond black-box predictions:

- **Global analysis**: confirmed Age, NumOfProducts, IsActiveMember, Balance, and Geography_Germany as the top drivers of churn — independently validating the statistical EDA findings using the trained model itself.
- **Per-tier analysis**: SHAP summary plots were generated separately for each risk tier, revealing that driver intensity increases progressively — e.g., Age's influence becomes far more extreme in the Very High tier than in Low.
- **Individual-level explanation**: force plots were used to explain specific customer predictions (e.g., a customer flagged at 82% churn risk was driven primarily by high balance, German geography, older age, and low product count — despite being an active member with good credit).

### 8. Financial & Business Framing

- **Balance at Risk**: total account balance held by High + Very High risk customers
- **Expected Loss**: probability-weighted balance exposure (`Churn_Probability × Balance`), a more precise estimate than simple tier-based totals
- **Priority Score**: `Churn_Probability × Balance`, used to rank customers by combined risk and business value — ensuring retention effort focuses on customers who are both likely to leave *and* costly to lose, not risk alone

---

## Business Recommendations

| Risk Tier | Recommended Action |
|---|---|
| **Very High** | Immediate, high-touch retention: dedicated relationship manager outreach, senior-focused banking perks, and review of product bundling for customers with unusually high product counts |
| **High** | Targeted, personalized outreach with country-specific offers, given the strong concentration of German customers in this tier |
| **Medium** | Lower-cost, automated interventions — re-engagement email campaigns and activity-based nudges |
| **Low** | Standard engagement and periodic monitoring; no urgent action required |

**Priority investigation flag**: The extreme churn rate among 3–4 product customers (83–100%) warrants direct investigation — likely causes include fee overload, product complexity, or dissatisfaction with cross-sold products. Recommend a mandatory relationship manager review triggered whenever a customer reaches their 3rd product.

---

## Power BI Dashboard

An interactive three-page dashboard built on the model's scored output:

1. **Executive Summary** — high-level KPIs (Total Customers, Churn Rate, Balance at Risk, Expected Loss), customer distribution by risk tier, churn rate by age curve, and geographic view
2. **Risk Segmentation** — detailed tier-by-tier profile table, churn rate breakdowns by Geography, Gender, and Number of Products, with full cross-filtering
3. **Priority Customer List** — a sortable, filterable action list of the top-priority at-risk customers (ranked by Priority Score), paired with SHAP summary visuals for High and Very High risk segments (toggle via bookmark buttons)

All dashboard metrics were cross-validated against the Python source calculations to confirm consistency.

---

## Repository Contents

```
├── Customer_churn_prediction.ipynb     # Full Python analysis: cleaning, EDA, modeling, SHAP
├── Customer_Churn_Scored_PowerBI.csv   # Model output used as the Power BI data source
├── Customer_Churn_Prediction.pbix      # Interactive Power BI dashboard
└── README.md
```

---

## Key Takeaways

- A model's value isn't in its accuracy score alone — it's in whether it leads to a decision someone can actually act on.
- Statistical validation (chi-square, t-tests) should precede and confirm machine learning findings, not be skipped in favor of "the model will figure it out."
- Model selection should weigh business-relevant tradeoffs (precision/recall balance, operating threshold) rather than defaulting to the single highest AUC score.
- Explainability (SHAP) turns a probability into an actionable reason — critical for any model informing real business decisions in a regulated industry like banking.

---

## Future Improvements

- Hyperparameter tuning via `GridSearchCV` / `RandomizedSearchCV`
- K-Fold (Stratified) cross-validation for more robust performance estimates
- `sklearn.Pipeline` to formalize the preprocessing + modeling workflow and reduce leakage risk
- Probability calibration check to validate that predicted probabilities are practically reliable
- Model persistence via `joblib` for deployment readiness

---

## Author

**Anmol Verma**
Data Analyst | Risk Analyst

