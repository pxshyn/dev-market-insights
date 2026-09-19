# Phase 1: Data Wrangling & Engineering

**Project:** Comprehensive Analysis of Developer Market Trends & Salary Drivers

This step cleans and prepares the raw Stack Overflow Developer Survey dataset for downstream exploratory data analysis (EDA) and modeling. It focuses on a core set of columns relevant to compensation and market-trend questions, and demonstrates a full data wrangling workflow: duplicate handling, missing-value treatment, categorical standardization/encoding, feature engineering, and normalization.

## Central Research Questions

These five questions drive the project end to end (Phase 1 → Phase 2 → Phase 3):

1. **Data Quality & Representativeness** — Who took the survey, and how reliable is the salary data (~64% missing)?
2. **Income Distribution** — How is developer compensation distributed, and how are outliers identified?
3. **Salary Drivers** — How do country, years of experience, education level, employment type, and remote-work arrangement affect compensation?
4. **Market Trends** — Which programming languages are most used, which are most wanted (Want vs. Have), and which command the highest median salaries?
5. **Income–Satisfaction Link** — Does job satisfaction (`JobSat`) increase with compensation and experience?

## Dataset

- **Raw file:** `data/survey_data_duplicates.csv` — 65,447 rows × 114 columns. This is a modified copy of the original survey data (e.g. synthetic duplicate rows were introduced), so figures like the duplicate count above reflect this modified copy.
- **Not committed to this repo:** the entire `data/` folder is listed in `.gitignore` (raw file ~150 MB; derived outputs are regenerable). The raw file is **not included** — place it manually at `data/survey_data_duplicates.csv` before running the notebook.

## Scope

The raw survey has 100+ columns. To keep this step focused and clearly demonstrate each technique, only the columns most relevant to the compensation/market-trend analysis are fully wrangled here:

`ResponseId`, `Country`, `EdLevel`, `Employment`, `RemoteWork`, `CodingActivities`, `YearsCodePro`, `CompTotal`, `ConvertedCompYearly`

Duplicate removal is the one step applied to **all 114 columns**, since it's a dataset-wide correction rather than a per-column technique — the result is saved as `survey_data_deduplicated_full.csv`. Any column outside the core set above is left untouched in this file and gets wrangled individually, using the same principles demonstrated here, when a later phase actually needs it — that phase should start from `survey_data_deduplicated_full.csv` rather than the raw file, so it never has to re-handle duplicates.

**Scope note:** `JobSat` and the other fields needed for the market-trend and satisfaction questions (`WorkExp`, `Age`, `DevType`, `Industry`, `LanguageHaveWorkedWith`, `LanguageWantToWorkWith`) are **not** processed in this phase. They are carried forward untouched inside `survey_data_deduplicated_full.csv`. In Phase 2, `LanguageHaveWorkedWith`/`LanguageWantToWorkWith` are exploded (they are multi-select), and `DevType`/`Industry` are verified against the same multi-select risk — see Phase 2's README for the result of that check and exactly what treatment each field gets.

## Techniques Applied

| Step                     | Technique                                                                                                     | Column(s)                                         |
| ------------------------ | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| Duplicate removal        | `drop_duplicates()` on the full raw record                                                                    | all columns                                       |
| Standardization          | Manual mapping of inconsistent free-text labels                                                               | `Country`, `EdLevel`                              |
| Encoding                 | One-hot encoding of a multi-select field                                                                      | `Employment`                                      |
| Missing value imputation | Mode imputation                                                                                               | `RemoteWork`, `EdLevel_clean`, `CodingActivities` |
| Missing value imputation | Explicit "Not specified" label (avoids geographic bias from mode imputation)                                  | `Country_clean`                                   |
| Missing value handling   | Left as `NaN` — missing rate too high (48–64%) to impute without distorting the distribution; flagged instead | `CompTotal`, `ConvertedCompYearly`                |
| Feature engineering      | Binning into experience tiers                                                                                 | `YearsCodePro` → `ExperienceLevel`                |
| Normalization            | Log-transform, then Min-Max and Z-score scaling on the transformed values                                     | `ConvertedCompYearly`                             |

> **Disclaimer:** `ConvertedCompYearly_MinMax_experimental` and `ConvertedCompYearly_Zscore_experimental` are an early technical trial only, computed *before* outlier isolation — the `_experimental` suffix is intentional so they can't be mistaken for a final feature. Phase 2 (EDA) flags compensation outliers explicitly without deleting them, and Phase 3 (Modeling) recomputes the official scaled features afterward, under different column names — downstream phases should not reuse these two columns as-is.

> **`CompTotal`:** kept in both output files unmodified, for reference/audit only (e.g. spot-checking `ConvertedCompYearly`'s USD conversion against the respondent's self-reported local-currency total). No analysis anywhere in this project — Phase 1, 2, or the Phase 3 plan — uses `CompTotal` as an input; `ConvertedCompYearly` is the compensation field used throughout.

## Key Results

- **Duplicates:** 10 fully duplicated rows detected and removed (65,447 → 65,437 rows).
- **Missing rates (core columns):** `Country` 9.94%, `EdLevel` 7.11%, `RemoteWork` 16.25%, `CodingActivities` 16.77%, `YearsCodePro` 21.14%, `CompTotal` 48.44%, `ConvertedCompYearly` 64.19%.
- **Skewness of `ConvertedCompYearly`:** 52.92 before transformation → -2.26 after log-transform.
- **Output:** `data/survey_data_deduplicated_full.csv` (65,437 rows × 114 columns) and `data/survey_data_cleaned_core.csv` (65,437 rows × 26 columns).

## How to Run

```bash
pip install pandas numpy matplotlib
jupyter notebook General_Data_Wrangling.ipynb
```

Place `survey_data_duplicates.csv` in `data/` first, then run all cells top to bottom. The notebook then writes two outputs: the full deduplicated dataset right after duplicate removal, and the fully wrangled core-column dataset at the end.

## Output

- `data/survey_data_deduplicated_full.csv` — all 114 original columns, duplicates removed. Use this as the starting point for any later phase that needs a column outside the core set.
- `data/survey_data_cleaned_core.csv` — the core columns after standardization, encoding, imputation, feature engineering, and normalization: `Country_clean`, `EdLevel_clean`, `has_compensation_data`, `ExperienceLevel`, `ConvertedCompYearly_log`, `ConvertedCompYearly_MinMax_experimental`, `ConvertedCompYearly_Zscore_experimental`, plus the one-hot encoded `Employment` categories, `CompTotal` (kept for reference only — see disclaimer above). Use this when a later phase needs exactly these columns already wrangled.

## Next Steps

Phase 2: Exploratory Data Analysis, starting from `data/survey_data_deduplicated_full.csv` (for any additional column) and `data/survey_data_cleaned_core.csv` (for the core columns already wrangled here).
