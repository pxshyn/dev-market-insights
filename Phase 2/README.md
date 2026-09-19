# Phase 2: Exploratory Data Analysis (EDA)

**Project:** Comprehensive Analysis of Developer Market Trends & Salary Drivers

This step explores the cleaned Stack Overflow Developer Survey data produced by Phase 1, to understand
respondent representativeness, the shape and drivers of developer compensation, technology-adoption trends,
and the link between compensation and job satisfaction.

## Scope

Phase 2 builds directly on Phase 1's two outputs and does **not** repeat any of Phase 1's work:

- **Base table:** `data/survey_data_cleaned_core.csv` — the core columns Phase 1 already cleaned, standardized,
  encoded, and engineered (`Country_clean`, `EdLevel_clean`, one-hot `Employment`, `ExperienceLevel`,
  `ConvertedCompYearly_log`/`_MinMax_experimental`/`_Zscore_experimental`).
- **Supplementary table:** `data/survey_data_deduplicated_full.csv` (65,437 rows × 114 columns) — used only to
  pull in the additional columns Phase 1 intentionally left untouched: `JobSat`, `WorkExp`, `Age`, `DevType`,
  `Industry`, `LanguageHaveWorkedWith`, `LanguageWantToWorkWith`, plus the **pre-imputation** originals of
  `RemoteWork` and `CodingActivities` (to audit Phase 1's mode imputation).
- **Join key:** `ResponseId`, validated as unique in both files with a `LEFT JOIN` before any analysis runs.
- **Missing-value policy:** no additional imputation. Every analysis uses `dropna()` on the specific fields it
  needs (complete-case analysis) and reports the resulting sample size `n`, per the project's Phase 1 ↔ Phase 2
  alignment.
- **`DevType` / `Industry`:** Section 2 explicitly checks whether either field contains multiple `;`-separated
  values per respondent (the same risk already handled for the language fields via `.explode()`). The check
  found **zero** multi-value responses in either column, so both are treated as single-select and used directly
  — no additional standardization beyond the `dropna()` policy above is needed. See Section 2 of the notebook
  for the exact check; re-run it if the underlying data changes.
- **Out of scope for this phase:** deleting outlier rows (they are flagged, never dropped) and any Min-Max /
  Z-score normalization (deferred to Phase 3, after outlier handling is finalized — Phase 1's
  `_MinMax_experimental` / `_Zscore_experimental` columns are a preliminary trial, not final features).

## Central Research Questions

1. **Data Quality & Representativeness** — Who took the survey, and how reliable is the salary data (~64% missing)?
2. **Income Distribution** — How is developer compensation distributed, and how are outliers identified?
3. **Salary Drivers** — How do country, years of experience, education level, employment type, and remote-work
   arrangement affect compensation?
4. **Market Trends** — Which programming languages are most used, which are most wanted (Want vs. Have), and
   which command the highest median salaries?
5. **Income–Satisfaction Link** — Does job satisfaction (`JobSat`) increase with compensation and experience?

## Techniques Implemented

| Section | Technique                                                                                     | Column(s) / Output                                              |
| ------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 1       | Key-uniqueness & row-count readiness checks, `LEFT JOIN` on `ResponseId`                        | `JobSat`, `WorkExp`, `Age`, `DevType`, `Industry`, language fields |
| 1       | Imputation audit — compares pre- vs. post-Phase-1-imputation values                             | `RemoteWork_imputed`, `CodingActivities_imputed`                  |
| 2       | Respondent-profile distributions; compensation missing-rate by country/employment; multi-select check on `DevType`/`Industry` | `Country_clean`, `Employment`, `ExperienceLevel`, `Industry`, `DevType` |
| 3       | Distribution visualization (histogram/KDE/boxplot), 3-method outlier comparison (3σ raw, IQR raw, IQR log) | `ConvertedCompYearly`, `ConvertedCompYearly_log`, `is_comp_outlier` |
| 4       | Group-wise median/IQR compensation comparison, with and without flagged outliers                | Country, `ExperienceLevel`, `EdLevel_clean`, `Employment` dummies, `RemoteWork` |
| 5       | Pearson & Spearman correlation matrix, multicollinearity check, scatter plots                   | `ConvertedCompYearly_log`, `WorkExp`, `YearsCodePro_num`, `Age_num`, `JobSat` |
| 6       | Multi-select field explosion (`.str.split(';').explode()`) with question-specific denominators; median salary by language filtered to `n >= 30` (`MIN_N_LANG`) | `LanguageHaveWorkedWith`, `LanguageWantToWorkWith`, Want-vs-Have gap, median salary by language |
| 7       | Distribution and group comparison of job satisfaction; correlation with compensation             | `JobSat`                                                          |
| 8       | Consolidated export for the next phase                                                          | `is_comp_outlier`, `RemoteWork_imputed`, `CodingActivities_imputed` |

## Key Analytical Results

**Respondent profile & representativeness**
- **Compensation missing rate:** 64.19% of all respondents did not report `ConvertedCompYearly`; the compensation-analysis
  subset (`has_compensation_data == True`) has **23,435 respondents (35.81%)** of the full 65,437.
- **Full-time employees:** 45,162 respondents; median full-time compensation is **$67,666**.
- **`JobSat` missing rate:** 55.49% (29,126 respondents answered, 44.51%); of those, the distribution skews positive (mode at 8/10).
- **Respondent concentration:** United States (11,095), Germany (4,947), India (4,231), United Kingdom (3,224), and
  Ukraine (2,672) are the top 5 named countries by respondent count; ~9.94% of respondents fall under the
  `"Not specified"` country label and are excluded from country-level salary comparisons.
- **Experience mix:** Senior is the largest group with a determinable `ExperienceLevel` (18,460), followed by
  Mid-level (12,653), Junior (10,834), and Entry-level (9,663).
- **`DevType` / `Industry`:** confirmed single-select (0 multi-value responses in either column, out of ~59,500 and
  ~28,900 non-null responses respectively). Top `Industry` category is Software Development (11,918); top `DevType`
  is Developer, full-stack (18,260).

**Income distribution & outliers**
- Outlier-method comparison on `ConvertedCompYearly`:

  | Method | Lower bound | Upper bound | Outliers flagged |
  | --- | --- | --- | --- |
  | 3-sigma (raw) | -474,116 | 646,426 | 89 |
  | IQR (raw) | -80,177 | 220,861 | 978 |
  | **IQR (log)** — used for `is_comp_outlier` | 5,454 | 647,443 | **1,723 (7.35% of the compensation subset)** |

  The raw-scale methods produce a negative lower bound (impossible for compensation) because the distribution is
  heavily right-skewed (skewness ≈ 53) — this is why IQR-on-log was selected for `is_comp_outlier`.
- **Median compensation:** $65,000 with outliers included, **$70,000** with the 1,723 flagged outliers excluded.

**Salary drivers**
- **Remote work:** Remote median $75,000 (n=9,591) > Hybrid median $66,592 (n=9,907) > In-person median $44,586 (n=3,937).
- Country, `ExperienceLevel`, `EdLevel_clean`, and `Employment` all show visible median-compensation spread in the
  notebook's group-wise comparisons (Section 4); see the notebook for the full breakdown by group.

**Correlation & multicollinearity**
- `Age_num` correlations: `WorkExp` 0.848, `YearsCodePro_num` 0.832, `ConvertedCompYearly_log` 0.312, `JobSat` 0.070.
- **`Age_num` vs. `WorkExp`: r = 0.85 — strong multicollinearity, not resolved in this notebook.** Phase 3 must
  choose one of the two, or combine them, before using both as raw numeric model features.
- `YearsCodePro_num` vs. `JobSat`: Pearson r = 0.104, Spearman r = 0.119 (weak positive relationship).

**Market trends**
- **Want-vs-Have gap (top 5 by gap):** Rust (+18.26 pp: 12.65% have worked with it vs. 30.91% want to), Go
  (+11.26 pp), Zig (+5.50 pp), Kotlin (+3.75 pp), Elixir (+3.11 pp).
- **Highest median salary by language** (filtered to `n >= 30`, top 5): Erlang $100,636 (n=229), Elixir $96,000
  (n=598), Clojure $95,541 (n=350), Nim $94,924 (n=34), Ruby $90,221 (n=1,372).

**Income–satisfaction link**
- `JobSat` vs. `ConvertedCompYearly_log`: Pearson r = 0.080, Spearman r = 0.103, on n = 16,075 respondents who
  reported both — a weak positive relationship, not a strong driver of satisfaction on its own.

## How to Run

```bash
pip install pandas numpy matplotlib seaborn
jupyter notebook market_trends_and_salary_drivers_eda.ipynb
```

Prerequisites: Phase 1 must have already been run, so that `data/survey_data_cleaned_core.csv` and
`data/survey_data_deduplicated_full.csv` both exist in the shared `data/` folder at the repo root. Run all
cells top to bottom; the notebook performs its own readiness checks (unique keys, expected row counts) before
proceeding, and will raise an `AssertionError` immediately if either input file is missing or misaligned.

## Output Artifacts

- `data/survey_data_eda_ready.csv` — the joined, EDA-ready dataset: all core columns from Phase 1, plus the
  columns joined in this phase (`JobSat`, `WorkExp`, `Age`, `DevType`, `Industry`, `LanguageHaveWorkedWith`,
  `LanguageWantToWorkWith`, `RemoteWork_raw`, `CodingActivities_raw`) and the flags created here
  (`is_comp_outlier`, `RemoteWork_imputed`, `CodingActivities_imputed`, `Age_num`). 65,437 rows × 40 columns.
- In-notebook visualizations and summary tables for each research question (not persisted to disk; re-run the
  notebook to reproduce them).

## Next Steps (Phase 3 Alignment)

Phase 3 (Modeling / Advanced Analysis) should start from `data/survey_data_eda_ready.csv` rather than
re-joining the Phase 1 files, and should:

1. **Resolve the `Age_num` vs. `WorkExp` multicollinearity (r = 0.85) flagged in Section 5** before using both as
   model features — this is an open item, not yet decided anywhere in Phase 1 or 2.
2. Recompute official Min-Max / Z-score scaling for compensation **after** deciding how `is_comp_outlier` rows
   are treated in modeling, under new column names (Phase 1's `_MinMax_experimental` / `_Zscore_experimental`
   are explicitly a preliminary trial and were computed before outlier isolation).
3. Test country and `ExperienceLevel` as priority salary-driver features, per Section 4's findings.
4. Evaluate whether language choice (from `LanguageHaveWorkedWith`) adds predictive signal beyond country and
   experience, given the median-salary-by-language results in Section 6 (Erlang, Elixir, and Clojure command the
   highest medians among languages with at least 30 respondents).
