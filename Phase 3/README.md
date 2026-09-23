# Phase 3: Modeling (Simple & Multiple Linear Regression)

**Project:** Developer Market Trends & Salary Drivers

I built two models in this notebook to predict developer pay: Simple Linear Regression
(SLR) and Multiple Linear Regression (MLR).

## Scope

- **Input:** `data/survey_data_eda_ready.csv` (Phase 2's output).
- **Data used:** rows with reported pay, no outliers, no missing feature. See "Key
  Results" below for the exact count.
- **Target:** `ConvertedCompYearly_log` (log of yearly pay). I convert predictions back to
  dollars only for reporting.

## Fixed Decisions

- **`Age_num` vs. `WorkExp`:** I dropped `Age_num` and kept `WorkExp` - the two are highly
  correlated (r = 0.85, from Phase 2).
- **Outliers:** I removed them from the data before modeling.

## Techniques Used

| Section | Technique                                                                   | Column(s)                                                         |
| ------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 2       | Filter to a clean modeling subset, print row count at each step             | `has_compensation_data`, `is_comp_outlier`                        |
| 3       | Top-10-plus-Other grouping, ordinal encoding, one-hot encoding              | `Country_clean`, `ExperienceLevel`, `EdLevel_clean`, `RemoteWork` |
| 4       | 80/20 train/test split                                                      | -                                                                 |
| 5       | Simple Linear Regression                                                    | `WorkExp` to `ConvertedCompYearly_log`                            |
| 6       | Multiple Linear Regression, metrics, dollar-scale error, plot, coefficients | full feature set to `ConvertedCompYearly_log`                     |

## Key Results

- **Data used:** I ended up with 15,011 rows (64.1% of the 23,435 respondents who
  reported pay), after removing outliers (21,712 left) and rows missing a feature.
- **Train / test:** 12,008 / 3,003.
- **SLR (`WorkExp` only):** R² = 0.169, RMSE = 0.775 (log scale).
- **MLR (all features):** R² = 0.537, RMSE = 0.579 (log scale).
- **R² gain from adding categorical features:** +0.368.
- **MLR error in dollars:** MAE is about $31,535.
- **Biggest factors (by coefficient size):** country matters most - United States (+1.54),
  Canada (+1.10), United Kingdom (+1.02), and Germany (+0.91) all raise predicted pay the
  most. Being retired lowers it the most (-0.67).

Country explains pay differences much more than experience. But my model still misses almost half of the pay differences (R² = 0.537), so other things I did not use - like programming language or company size - probably matter too. I should not use this model to set exact salaries, only to compare which factors matter most.

## How to Run

```bash
pip install pandas numpy matplotlib scikit-learn
jupyter notebook salary_prediction_modeling.ipynb
```

I need Phase 2's output (`data/survey_data_eda_ready.csv`) in the shared `data/` folder.
Run all cells top to bottom.

## Output

I don't export a new file here. The models, metrics, and plots stay in the notebook.

## Next Steps

Phase 4: Reporting. I'll combine Phase 1, 2, and 3 into one report or dashboard, built
around the project's 5 research questions.
