# MSCS 634 – Project Deliverable 1: Data Collection, Cleaning, and Exploration

**Project:** Advanced Data Mining for Data-Driven Insights and Predictive Modeling

**Authors:**
- Pranavi Balakulla
- Sandeep Reddy Banda
- Sowmya Cheruku
- Vineela Reddy Karnatakam
**Dataset:** IBM Telco Customer Churn (7,043 customers × 21 attributes)

This repository contains the first deliverable of a four-part data mining project. The goal here is to select a
dataset that can support every later technique (regression, classification, clustering and association rules),
clean it, and explore it to produce insights that guide the modeling work in Deliverables 2–4.

---

## Repository Contents

```
MSCS_634_ProjectDeliverable_1/
├── Deliverable1_Data_Cleaning_EDA.ipynb   # Main notebook: loading, cleaning, EDA, insights (with outputs)
├── README.md                              # This file
├── requirements.txt                       # Python packages needed to run the notebook
├── data/
│   ├── Telco-Customer-Churn.csv           # Raw dataset (unchanged)
│   └── telco_churn_cleaned.csv            # Cleaned dataset produced by the notebook (input for Deliverables 2-4)
└── figures/                               # All charts exported by the notebook
```

**To run:** `pip install -r requirements.txt`, then open the notebook in Jupyter and choose *Run All*. The notebook
reads `data/Telco-Customer-Churn.csv`; if the file is missing it downloads IBM's public copy automatically.

---

## Dataset Summary

| Item | Detail |
|---|---|
| Source | IBM sample dataset, also published on Kaggle as [Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) |
| Domain | Customer behavior – a telecom company selling home phone and internet services |
| Size | **7,043 records**, **21 attributes** (1 ID, 3 numeric, 16 categorical, 1 target) |
| Target | `Churn` – whether the customer left in the last month (Yes/No) |
| Attribute groups | **Demographics** (gender, SeniorCitizen, Partner, Dependents) · **Services** (phone, multiple lines, internet type, 6 add-ons) · **Account** (tenure, contract, paperless billing, payment method, monthly and total charges) |

**Why this dataset:** it exceeds the size requirements, has real data-quality problems to fix, and supports all four
project deliverables:

| Deliverable | What the dataset offers |
|---|---|
| 2 – Regression | Continuous targets (`MonthlyCharges`, `TotalCharges`) |
| 3 – Classification | A clear binary target (`Churn`) with business value |
| 3 – Clustering | Customer segmentation by tenure, spend and services |
| 4 – Association rules | Mostly Yes/No service attributes, which map directly to transaction items |

---

## Data Cleaning Steps

| # | Issue found | Action taken | Why |
|---|---|---|---|
| 1 | `TotalCharges` stored as text; 11 cells contained only a space, so `isna()` reported **0 missing values** | Stripped whitespace from all text columns and converted `TotalCharges` to numeric, exposing **11 true missing values** | Hidden blanks would break every numeric calculation and model |
| 2 | 11 missing `TotalCharges` | Imputed with **0** | All 11 customers have `tenure = 0` (brand-new, never billed). Median imputation would have assigned them \$1,397 — impossible for a 0-month customer. Dropping them would remove a valid customer group |
| 3 | 22 rows identical except for `customerID` | **Kept** | All are first-month, month-to-month customers on 1–2 basic services with no add-ons; different IDs and mixed churn outcomes show they are different people on the same plan, not data-entry duplicates. No duplicate IDs or fully duplicated rows exist |
| 4 | `SeniorCitizen` coded `0/1` while all other binary fields use `Yes/No` | Recoded to `Yes/No` | Consistent encoding across all binary attributes |
| 5 | Redundant levels `No internet service` (6 columns) and `No phone service` (`MultipleLines`) | Merged into `No` | Verified first that these levels match `InternetService == "No"` / `PhoneService == "No"` exactly, so no information is lost; columns become clean Yes/No flags |
| 6 | Possible noisy data | IQR fences and \|z\| > 3 rule: **0 outliers**. 7 domain-rule checks: **0 violations**. Billing check (`TotalCharges` vs `tenure × MonthlyCharges`): 27 mismatches > 25%, all explainable by price changes for short-tenure customers → **kept** | Extreme values are real customers, not errors |
| 7 | Small month-to-month variation in tenure | Smoothed by **binning** into `tenure_group` (0–12, 13–24, 25–48, 49–60, 61–72 months) | Standard noise-smoothing method; bins are also needed as items for association rules |
| 8 | Helper columns for analysis | Added `Churn_flag` (0/1), `tenure_group`, `NumServices` | Lets churn *rates* be averaged and service bundling be analyzed |

**Result:** 7,043 rows × 24 columns, **0 missing values**, every categorical column validated against its allowed
values. Saved as `data/telco_churn_cleaned.csv`.

---

## Exploratory Data Analysis

The notebook includes nine labeled figures (all saved in `figures/`):

1. Target class balance
2. Numeric distributions by churn status (histograms)
3. Outlier check (box plots)
4. Churn rate for every level of all 16 categorical features
5. Feature ranking by Cramér's V (chi-square) and point-biserial correlation
6. Correlation heatmap (multicollinearity check)
7. Tenure vs total charges; monthly charges by internet type
8. Interaction heatmaps (contract × tenure, contract × internet service)
9. Churn rate by number of services subscribed

![Churn rate by category](figures/04_churn_rate_by_category.png)

![Interaction heatmaps](figures/08_interaction_heatmaps.png)

---

## Key Insights

1. **Churn is substantial and imbalanced:** 26.5% of customers churned (1,869 of 7,043), about a 3 : 1 class ratio.
2. **Contract type is the strongest driver** (Cramér's V = 0.41). Month-to-month customers churn at **42.7%** vs
   **2.8%** on two-year contracts, and they make up **88.6% of all churners**.
3. **The first year is the riskiest period.** Churn is **47.4%** in months 0–12 and **6.6%** after five years; churners'
   median tenure is 10 months vs 38 for customers who stay.
4. **Fiber optic customers churn at 41.9%**, roughly double the DSL rate, even though fiber is the premium product.
5. **Electronic-check payers churn at 45.3%**, versus 15–19% for every other payment method.
6. **Protective add-ons are linked to retention.** Among internet users, churn is 41.6% without TechSupport vs 15.2%
   with it, and 41.8% without OnlineSecurity vs 14.6% with it.
7. **Effects combine:** month-to-month + fiber optic churns at **≈ 55%**; two-year + DSL at **≈ 2%**.
8. **Bundling reduces churn:** churn peaks at 3 services (45%) and falls to 5% for customers with all 9 services.
9. **Weak features:** `gender` and `PhoneService` have no statistically significant association with churn.
10. **Redundancy:** `TotalCharges` ≈ `tenure × MonthlyCharges` (r = 0.83 with tenure); `NumServices` and `MonthlyCharges`
    correlate at r = 0.85.

### How these insights guide the next deliverables

* **Deliverable 2 (Regression):** predict `MonthlyCharges` from the service mix; use Ridge/Lasso alongside linear
  regression because of the multicollinearity found; exclude `TotalCharges` as a predictor (it is derived from the
  target).
* **Deliverable 3 (Classification):** stratified splits and class weighting for the 73/27 imbalance; evaluate with
  recall, F1 and ROC-AUC rather than accuracy; test dropping `gender` and `PhoneService`; tree-based models should
  capture the contract × internet × tenure interactions.
* **Deliverable 3 (Clustering):** segment on scaled tenure, monthly charges and number of services, then profile each
  cluster's churn rate.
* **Deliverable 4 (Association rules):** use the Yes/No service flags, contract, internet type, payment method and
  `tenure_group` as items, and look for high-lift rules that lead to `Churn = Yes`.

---

## Challenges and How They Were Addressed

| Challenge | Solution |
|---|---|
| **Hidden missing values** – `isna()` showed none because blanks were stored as a single space, which also made `TotalCharges` a text column | Scanned every text column for whitespace-only values, stripped whitespace, then converted types before counting missing values |
| **Choosing the right imputation** | Investigated the affected rows first; all had `tenure = 0`, so 0 was the logically correct value rather than a statistical estimate |
| **Duplicate-looking rows** | Inspected them before deleting; they were distinct customers on identical basic plans, so they were kept |
| **Redundant category levels** | Proved the levels were fully implied by another column before merging, so the change is lossless |
| **Telling noise from real behaviour** | Combined statistical (IQR, z-score), domain-rule and cross-field checks; nothing qualified as an error, so no rows were removed |
| **Class imbalance hides patterns in raw counts** | Compared churn *rates* per group and used density-normalised histograms so both classes are visible on the same scale |
| **Correlation missed a real pattern** – `NumServices` has r ≈ −0.02 with churn | Plotted the relationship and found it is non-linear (rises then falls), a reminder to check visuals as well as coefficients |
