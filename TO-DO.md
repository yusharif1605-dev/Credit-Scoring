# TO-DO

Working task list for the Credit-Scoring project. README.md stays high-level (objective, background, workflow); this file tracks the actual next actions.

## Immediate — before rerunning IV/WOE

- [X] Run quick EDA pass (missing values + outlier/impossible-value detection) — **diagnostic only, no imputing or dropping yet**
  - [X] Missing value counts/percentage per column (core 16 + remaining 44 columns from the 47-column IV shortlist)
  - [X] Flag columns where missingness looks structural vs. possibly random
  - [X] Check for impossible/implausible values (negative `dti`, `annual_inc` ≤ $1, extreme `revol_util`/`bc_util`/`il_util`, `earliest_cr_line` before 1950)
  - [X] Check for disguised missing values (`dti` = 999, `tot_hi_cred_lim`/`total_rev_hi_lim` = 9999999)
  - [X] Review numeric distributions (extended percentiles 1%–99.9%) against domain expectations
  - [X] Categorical sanity check (`sub_grade`, `grade`, `term`, `verification_status`, `home_ownership`) — no issues found
  - [X] Date logic check (`earliest_cr_line` vs. `issue_d`) — 0 logic violations, 9 implausible pre-1950 entries found and nulled
- [X] Fix flagged issues (placeholder → `NaN` conversion, 99.9th-percentile capping for extreme outliers) — not full imputation, just correcting clear data errors
- [X] **Rerun IV/WOE test with `optbinning` on the cleaned `final_candidates` list (34 columns)** and compare against the original 47-column result to see if rankings shifted — **next action**

## Key finding this pass — structural missingness cluster

- Discovered a 13-column cluster (`il_util`, `mths_since_rcnt_il`, `all_util`, `open_acc_6m`, `total_cu_tl`, `inq_last_12m`, `open_il_12m`, `max_bal_bc`, `open_il_24m`, `total_bal_il`, `open_act_il`, `inq_fi`, `open_rv_12m`, `open_rv_24m`) that is **structurally missing**, not randomly missing — 100% missing 2007–2014, still 95.6% missing in 2015, only becoming available from 2016 onward (Lending Club started collecting these fields partway through 2015/2016).
- Within the actual training window (2007–2016), this cluster is **~73–77% missing** — far too sparse to impute responsibly.
- **Decision: dropped this cluster from the candidate feature list entirely**, rather than imputing or using a missingness flag, since doing so risked the model learning "is this a post-2016 loan" rather than genuine credit risk signal (a subtle temporal-leakage-adjacent issue distinct from the post-origination leakage already screened out).
- A separate 9-column cluster (`avg_cur_bal`, `mo_sin_old_rev_tl_op`, `mo_sin_rcnt_rev_tl_op`, `num_tl_op_past_12m`, `tot_cur_bal`, `tot_hi_cred_lim`, `num_rev_tl_bal_gt_0`, `total_rev_hi_lim`, `num_actv_rev_tl`) was checked the same way — missingness stayed consistent (~5.4% overall, ~6.4% in-window), confirming it's genuine/evenly-spread missingness rather than structural. **Kept** for the later missing value mechanism analysis step.

## Next in workflow (after IV/WOE is re-confirmed clean)

- [ ] Correlation / multicollinearity check — pairwise correlation matrix + VIF
- [ ] Missing value mechanism analysis (MCAR / MAR / MNAR reasoning per column) — the kept 9-column ~6.4% cluster starts here
- [ ] Imputation strategy per feature
- [ ] Modeling — re-check VIF/correlation if using linear models

## Documentation

- [ ] Update README workflow checklist to tick "Quick EDA" now that it's genuinely complete
- [ ] Note in README the structural missingness cluster finding and the decision to drop those 13 columns
- [ ] Note in README whether `grade` / `sub_grade` / `int_rate` were ultimately kept in or excluded from the final model, and why

## Longer-term / portfolio

- [ ] Study WOE scorecard construction (points-based scaling, not just binning)
- [ ] Learn PD / LGD / EAD and expected loss concepts
- [ ] Get working-level understanding of Basel II/III, IFRS9/CECL, SR 11-7
- [ ] Produce mock model documentation as a portfolio differentiator once modeling is done
