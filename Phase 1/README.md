# Phase 1: Data Wrangling & Engineering

**Project:** Developer Market Trends & Salary Drivers

I clean and prepare the raw Stack Overflow Developer Survey data here, for the EDA and
modeling phases that come after. I focus on the columns I need for pay and market
questions, and show a full wrangling workflow: duplicates, missing values, encoding,
feature engineering, and normalization.

## Research Questions

These 5 questions guide the whole project (Phase 1, 2, and 3):

1. **Data quality:** Who took the survey? How reliable is the pay data (about 64% missing)?
2. **Income distribution:** How is pay spread out? How do I find outliers?
3. **Salary drivers:** How do country, experience, education, job type, and remote work affect pay?
4. **Market trends:** Which languages are most used? Most wanted? Highest paid?
5. **Income-satisfaction link:** Does job satisfaction go up with pay and experience?

## Dataset

- **Raw file:** `data/survey_data_duplicates.csv`, 65,447 rows x 114 columns. This is a
  modified copy of the original survey (it has synthetic duplicate rows added).
- **Not in this repo:** the whole `data/` folder is in `.gitignore` (the raw file is about
  150 MB). Place it at `data/survey_data_duplicates.csv` before running the notebook.

## Scope

The raw survey has 100+ columns. I only fully wrangle the ones I need for pay and market
questions:

`ResponseId`, `Country`, `EdLevel`, `Employment`, `RemoteWork`, `CodingActivities`,
`YearsCodePro`, `CompTotal`, `ConvertedCompYearly`

I remove duplicates on **all 114 columns**, since that's a dataset-wide fix, not a
per-column one. I save the result as `survey_data_deduplicated_full.csv`. Any column
outside my core set stays untouched in that file, and gets cleaned later, when a phase
needs it. A later phase should start from `survey_data_deduplicated_full.csv`, not the
raw file, so it never has to redo duplicate removal.

**Scope note:** `JobSat` and the other fields for the market-trend and satisfaction
questions (`WorkExp`, `Age`, `DevType`, `Industry`, `LanguageHaveWorkedWith`,
`LanguageWantToWorkWith`) are **not** processed here. They stay untouched inside
`survey_data_deduplicated_full.csv`. In Phase 2, I explode `LanguageHaveWorkedWith` and
`LanguageWantToWorkWith` (they allow more than one answer), and I check `DevType` and
`Industry` for the same issue. See Phase 2's README for the result.

## Techniques Used

| Step | Technique | Column(s) |
| --- | --- | --- |
| Remove duplicates | `drop_duplicates()` on the full raw record | all columns |
| Standardize | Map inconsistent free-text labels | `Country`, `EdLevel` |
| Encode | One-hot encode a multi-select field | `Employment` |
| Fill missing values | Mode | `RemoteWork`, `EdLevel_clean`, `CodingActivities` |
| Fill missing values | Explicit "Not specified" label (avoids bias) | `Country_clean` |
| Handle missing values | Left as `NaN`, flagged instead (missing rate too high to fill: 48-64%) | `CompTotal`, `ConvertedCompYearly` |
| Feature engineering | Bin into experience tiers | `YearsCodePro` to `ExperienceLevel` |
| Normalize | Log-transform, then Min-Max and Z-score scaling | `ConvertedCompYearly` |

> **Note:** `ConvertedCompYearly_MinMax_experimental` and `ConvertedCompYearly_Zscore_experimental`
> are an early trial, before I handle outliers. I use `_experimental` on purpose, so no
> one mistakes them for a final feature. Phase 2 flags outliers (doesn't delete them), and
> Phase 3 makes the real scaled columns after that, under different names. Don't reuse
> these two columns as-is.

> **`CompTotal`:** I keep it in both output files, unchanged, for reference only (for
> example, to spot-check `ConvertedCompYearly`'s USD conversion). No analysis in this
> project uses `CompTotal` as an input. `ConvertedCompYearly` is the pay field I use
> throughout.

## Key Results

- **Duplicates:** 10 fully duplicated rows found and removed (65,447 to 65,437 rows).
- **Missing rates (core columns):** `Country` 9.94%, `EdLevel` 7.11%, `RemoteWork` 16.25%,
  `CodingActivities` 16.77%, `YearsCodePro` 21.14%, `CompTotal` 48.44%,
  `ConvertedCompYearly` 64.19%.
- **Skew of `ConvertedCompYearly`:** 52.92 before the log-transform, -2.26 after.
- **Output:** `data/survey_data_deduplicated_full.csv` (65,437 rows x 114 columns) and
  `data/survey_data_cleaned_core.csv` (65,437 rows x 26 columns).

## How to Run

```bash
pip install pandas numpy matplotlib
jupyter notebook General_Data_Wrangling.ipynb
```

Place `survey_data_duplicates.csv` in `data/` first, then run all cells top to bottom. I
write two output files: the full deduplicated dataset right after duplicate removal, and
the wrangled core-column dataset at the end.

## Output

- `data/survey_data_deduplicated_full.csv`, all 114 columns, duplicates removed. Start
  here if a later phase needs a column outside my core set.
- `data/survey_data_cleaned_core.csv`, the core columns after standardization, encoding,
  imputation, feature engineering, and normalization: `Country_clean`, `EdLevel_clean`,
  `has_compensation_data`, `ExperienceLevel`, `ConvertedCompYearly_log`,
  `ConvertedCompYearly_MinMax_experimental`, `ConvertedCompYearly_Zscore_experimental`,
  plus the one-hot `Employment` columns and `CompTotal` (reference only, see note above).
  Use this file when a later phase needs exactly these columns, already wrangled.
