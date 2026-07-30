# Internal Rating System (SNI) — Credit Scoring Model

**Status:** Initial working model pushed as-is. Known methodological issues are documented below and being addressed iteratively — see Limitations & Next Steps.

## Context

Developed as part of a Risk Management course, this project builds an Internal Rating System (IRS/SNI) to estimate the probability of default (PD) of retail banking clients, following the Basel III definition of default (DPD > 90 days over a 12-month window).

## Data

5,753 client records with personal, financial, and credit-history variables (19 numeric, 10 categorical). After cleaning: 5,720 rows × 26 columns.
## Data Availability

This project uses `base_SNI.xlsx`, a dataset of 5,753 retail banking client records (personal, financial, and credit-history variables) used for an academic Risk Management course project. **The raw data file is not included in this repository** due to confidentiality — it is not publicly available and is not redistributed here.

To reproduce this analysis, you would need a dataset with a comparable structure:
- One row per client
- Personal/demographic fields (age, marital status, profession)
- Financial fields (income, balances, commitments)
- Credit history fields (number of loans, days past due)
- A column allowing construction of a Basel III default flag (DPD > 90 days over 12 months)

The notebook expects the file at the project root as `base_SNI.xlsx` — update the file path in the first cell if your data is structured differently.
## Methodology

- **Data quality**: missing value treatment (deletion above 80% missing, median/mode imputation with missingness flags below that), duplicate detection
- **Target definition**: Basel III default flag (DPD_12M > 90 days), 2.03% default rate (48:1 class imbalance)
- **Data leakage control**: excluded DPD_12M, client/account IDs, and snapshot dates before variable selection
- **Variable selection**: Information Value (IV) filtering, Cramer's V for categorical dependence, correlation filtering for numeric variables — 25 → 13 final variables
- **Resampling**: SMOTE-NC applied to train set only (test set left imbalanced to reflect real-world distribution)
- **Model**: Logistic regression scorecard, PD-to-score transformation (Offset=600, Factor≈72.13), 7-class risk grid (A–G) with monotonic default rates

## Key Results

| Metric | Train | Test |
|---|---|---|
| AUC | 0.799 | 0.764 |
| GINI | 0.598 | 0.529 |
| Recall | — | 71.4% |


## Repo Structure
├── notebooks/
│ └── sni_credit_scoring.ipynb
├── results/
│ ├── figures/
│ └── tables/
├── requirements.txt
└── README.md
## Limitations & Next Steps

- **Categorical encoding**: currently uses LabelEncoder, which imposes an artificial ordinal structure on nominal variables (profession, nationality, etc.). Since IV/WoE was already computed for variable selection, switching to WoE-encoding for the model itself is the priority fix..
- **Single model, no challenger**: a gradient boosting challenger model (with SHAP explainability) is planned for comparison.
