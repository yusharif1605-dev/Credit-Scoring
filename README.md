
# Credit-Scoring

A self-directed, end-to-end data science project built to develop skills relevant to a junior data scientist role in banking risk management. This marks the beginning of my journey into credit risk modeling.

## Objective

Build a credit default risk model using the Lending Club loan dataset, covering the full pipeline a real-world risk model would go through: exploratory analysis, leakage-safe feature screening, correlation/multicollinearity checks, missing value handling, and time-aware modeling and validation. The goal is both a working model and a portfolio piece that reflects real industry practice, not just a Kaggle-style accuracy exercise.

## Background

- **Dataset**: Lending Club approved/funded loan data (~145 features per loan). Since the dataset only contains loans that were already approved and funded, this project models **default risk**, not loan approval.
- **Target**: Binary — default vs. non-default, built from `loan_status`. `Charged Off` and `Default` are treated as the same "bad" outcome; `Fully Paid` is "good." Rows with a `Does not meet the credit policy` prefix have that prefix stripped before merging into the same outcome buckets.
- **Why time-based validation matters**: Loan performance is affected by macroeconomic conditions, so the data violates the i.i.d. assumption underlying a random train/test split. `issue_d` is used as the chronological anchor for an out-of-time (OOT) split: 2007–2016 train, 2017 validation, 2018 OOT test.
- **Tools**: Python (Anaconda), `credit_score` conda environment, Jupyter notebooks in VS Code, version-controlled on GitHub.

## Workflow

- [X] **Domain feature study** — reviewed the core credit risk concepts driving default risk (income, DTI, credit utilization, delinquency/derogatory marks, loan grade/subgrade, open credit accounts, interest rate, credit history length) before touching the data, to be able to interpret results rather than just generate them.
- [ ] **Quick EDA** — light outlier/sanity check for impossible values (e.g. negative income, DTI outside a plausible range), using domain knowledge as a sanity-check layer on the data.
- [X] **Leakage screening** — automated IV (Information Value) and single-feature AUC testing on the training split only, after removing post-origination/outcome columns (payment history, recoveries, hardship/settlement fields, identifiers) so they can't leak into the target.
- [ ] **Correlation / multicollinearity check** — pairwise correlation matrix + VIF, run after leakage screening (so leaked features don't distort it) and before imputation (so redundancy findings can inform imputation and feature selection).
- [ ] **Missing value mechanism analysis** — understand *why* values are missing (e.g. missing-not-at-random cases like `mths_since_last_delinq`, which is often missing because the borrower was never delinquent) before deciding how to handle them.
- [ ] **Imputation**
- [ ] **Modeling** — with a re-check of VIF/correlation if using linear models (lower priority for tree-based models).

## Key design decisions

- **Loan grade, subgrade, and interest rate** are Lending Club's own internal risk outputs. They're expected to correlate strongly with default and are used as a **pipeline sanity check** — if they don't show strong predictive power, something is likely wrong upstream. Whether to include them in the final model (vs. treat them purely as a validation signal) is still an open decision, since training on them risks reproducing Lending Club's own model rather than building an independent one.
- **IV/WOE screening** is run across the full eligible column set (not just the domain-selected features), so that features outside prior domain knowledge still get a fair statistical chance to surface as predictive.
- **Missing values** are treated as their own bin/category during IV/WOE screening rather than dropped or imputed beforehand, since missingness itself can carry predictive signal.

## Next steps

- Complete quick EDA
- Run correlation/VIF analysis on the surviving feature set
- Investigate missingness mechanisms for key features before choosing an imputation strategy
- Longer-term: build out institutional/regulatory vocabulary (WOE scorecards, PD/LGD/EAD, Basel II/III, IFRS9/CECL, SR 11-7) and consider producing mock model documentation as a portfolio differentiator
