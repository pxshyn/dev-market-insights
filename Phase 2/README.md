# Phase 2: Exploratory Data Analysis (EDA)

**Project:** Developer Market Trends & Salary Drivers

I explore Phase 1's cleaned data here: who took the survey, how pay is spread out and
what drives it, tech trends, and the link between pay and job satisfaction.

## Scope

I build on Phase 1's two files. I don't repeat any of Phase 1's work.

- **Base table:** `data/survey_data_cleaned_core.csv` - the core columns Phase 1 already
  cleaned (`Country_clean`, `EdLevel_clean`, one-hot `Employment`, `ExperienceLevel`,
  `ConvertedCompYearly_log`/`_MinMax_experimental`/`_Zscore_experimental`).
- **Extra table:** `data/survey_data_deduplicated_full.csv` (65,437 rows x 114 columns) -
  I only use this for the columns Phase 1 left untouched: `JobSat`, `WorkExp`, `Age`,
  `DevType`, `Industry`, the two language fields, and the pre-imputation originals of
  `RemoteWork` and `CodingActivities` (to check Phase 1's imputation).
- **Join key:** `ResponseId`. I check it's unique in both files before joining.
- **Missing values:** no new imputation. Each analysis uses `dropna()` and reports its
  sample size `n`.
- **`DevType` / `Industry`:** I checked if either field has multiple `;`-separated values
  per person, like the language fields do. I found zero multi-value rows, so I treat both
  as single-select and use them as-is. See Section 2 of the notebook.
- **Not in this phase:** dropping outlier rows (I only flag them) and Min-Max / Z-score
  scaling (that's for Phase 3, after outlier handling is set).

## Research Questions

1. **Data quality:** Who took the survey? How reliable is the pay data (about 64% missing)?
2. **Income distribution:** How is pay spread out? How do I find outliers?
3. **Salary drivers:** How do country, experience, education, job type, and remote work affect pay?
4. **Market trends:** Which languages are most used? Most wanted? Highest paid?
5. **Income-satisfaction link:** Does job satisfaction go up with pay and experience?

## Techniques Used

| Section | Technique | Column(s) |
| --- | --- | --- |
| 1 | Key checks, join on `ResponseId` | `JobSat`, `WorkExp`, `Age`, `DevType`, `Industry`, language fields |
| 1 | Imputation audit (before vs. after Phase 1) | `RemoteWork_imputed`, `CodingActivities_imputed` |
| 2 | Respondent profile, missing-pay rate by country/job type, multi-select check | `Country_clean`, `Employment`, `ExperienceLevel`, `Industry`, `DevType` |
| 3 | Distribution plots, 3 outlier methods (3-sigma raw, IQR raw, IQR log) | `ConvertedCompYearly`, `ConvertedCompYearly_log`, `is_comp_outlier` |
| 4 | Median/IQR pay by group, with and without outliers | Country, `ExperienceLevel`, `EdLevel_clean`, `Employment`, `RemoteWork` |
| 5 | Pearson & Spearman correlation, multicollinearity check | `ConvertedCompYearly_log`, `WorkExp`, `YearsCodePro_num`, `Age_num`, `JobSat` |
| 6 | Explode multi-select fields, median pay by language (n >= 30) | `LanguageHaveWorkedWith`, `LanguageWantToWorkWith` |
| 7 | Job satisfaction distribution and group comparison | `JobSat` |
| 8 | Export for Phase 3 | `is_comp_outlier`, `RemoteWork_imputed`, `CodingActivities_imputed` |

## Key Results

**Who took the survey**
- 64.19% didn't report pay. The pay-analysis subset (`has_compensation_data == True`) is
  **23,435 respondents (35.81%)** of all 65,437.
- Full-time employees: 45,162 people, median pay **$67,666**.
- `JobSat` missing rate: 55.49% (29,126 people answered). Of those, most rate their job
  positively (mode at 8/10).
- Top 5 countries by count: United States (11,095), Germany (4,947), India (4,231), United
  Kingdom (3,224), Ukraine (2,672). About 9.94% are `"Not specified"`.
- Experience mix: Senior is the largest group (18,460), then Mid-level (12,653), Junior
  (10,834), Entry-level (9,663).
- `DevType` / `Industry`: both single-select, confirmed. Top industry is Software
  Development (11,918). Top dev type is Developer, full-stack (18,260).

**Pay distribution and outliers**

| Method | Lower bound | Upper bound | Outliers flagged |
| --- | --- | --- | --- |
| 3-sigma (raw) | -474,116 | 646,426 | 89 |
| IQR (raw) | -80,177 | 220,861 | 978 |
| **IQR (log)**, used for `is_comp_outlier` | 5,454 | 647,443 | **1,723 (7.35%)** |

The raw methods give a negative lower bound, which makes no sense for pay - the data is
very skewed (skewness about 53). This is why I used IQR on the log scale.
Median pay: **$65,000** with outliers, **$70,000** without them.

**Salary drivers**
- Remote work: Remote median $75,000 (n=9,591), Hybrid $66,592 (n=9,907), In-person
  $44,586 (n=3,937).
- Country, `ExperienceLevel`, `EdLevel_clean`, and `Employment` all show clear pay
  differences by group (see notebook Section 4 for the full breakdown).

**Correlation and multicollinearity**
- `Age_num` correlates with: `WorkExp` 0.848, `YearsCodePro_num` 0.832,
  `ConvertedCompYearly_log` 0.312, `JobSat` 0.070.
- **`Age_num` vs. `WorkExp`: r = 0.85, strongly correlated. Not solved here.** Phase 3
  must pick one, or combine them.
- `YearsCodePro_num` vs. `JobSat`: Pearson r = 0.104, Spearman r = 0.119 (weak).

**Market trends**
- Top 5 Want-vs-Have gap: Rust (+18.26 pp), Go (+11.26 pp), Zig (+5.50 pp), Kotlin
  (+3.75 pp), Elixir (+3.11 pp).
- Top 5 median pay by language (n >= 30): Erlang $100,636 (n=229), Elixir $96,000 (n=598),
  Clojure $95,541 (n=350), Nim $94,924 (n=34), Ruby $90,221 (n=1,372).

**Income and satisfaction**
- `JobSat` vs. `ConvertedCompYearly_log`: Pearson r = 0.080, Spearman r = 0.103 (n=16,075).
  A weak link - pay alone doesn't explain much of job satisfaction.

## How to Run

```bash
pip install pandas numpy matplotlib seaborn
jupyter notebook market_trends_and_salary_drivers_eda.ipynb
```

I need Phase 1's two files (`survey_data_cleaned_core.csv` and
`survey_data_deduplicated_full.csv`) in the shared `data/` folder. Run all cells top to
bottom.

## Output

- `data/survey_data_eda_ready.csv` - the joined dataset: Phase 1's core columns, plus
  `JobSat`, `WorkExp`, `Age`, `DevType`, `Industry`, the two language fields,
  `RemoteWork_raw`, `CodingActivities_raw`, and the flags I made here (`is_comp_outlier`,
  `RemoteWork_imputed`, `CodingActivities_imputed`, `Age_num`). 65,437 rows x 40 columns.
- Charts and tables in the notebook (not saved to disk, re-run to see them again).

## Next Steps (Phase 3)

Phase 3 should start from `data/survey_data_eda_ready.csv`, and should:

1. **Solve the `Age_num` vs. `WorkExp` correlation (r = 0.85)** before using both as
   model features. Not decided yet.
2. Recompute Min-Max / Z-score scaling for pay, after deciding how to handle outliers, with
   new column names (Phase 1's `_MinMax_experimental` / `_Zscore_experimental` are just a
   first try).
3. Test country and `ExperienceLevel` as top salary-driver features (Section 4).
4. Check if programming language adds a pay signal beyond country and experience (Section
   6: Erlang, Elixir, and Clojure pay the most among common languages).
