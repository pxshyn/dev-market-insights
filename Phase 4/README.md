# Phase 4: Reporting (Findings & Power BI Dashboard)

**Project:** Developer Market Trends & Salary Drivers

This is where I pull Phase 1-3 together. I go back to the 5 questions I asked in Phase 1,
answer each one with the numbers I found, and add a few things I noticed along the way.

## Power BI Dashboard

[**View the live dashboard**](https://app.powerbi.com/links/Rt-xLG2iOq?ctid=a74aa7fa-f7f9-4020-a84a-057abbba6e9b&pbi_source=linkShare&bookmarkGuid=7d471da5-1d4b-4ae2-818d-3b8e6bd90d98)

![Page 1: Who took this survey, and how reliable is the pay data?](images/dashboard_page1_survey_profile.png)
![Page 2: Compensation Deep Dive](images/dashboard_page2_compensation.png)
![Page 3: Market Trends & Satisfaction](images/dashboard_page3_market_satisfaction.png)

The dashboard has 3 pages: who took the survey, pay by group, and language trends with job
satisfaction. Two things to know when you read it:

- Page 1 only counts the 23,435 people who reported pay, so its numbers are smaller than
  the Phase 2 profile (all 65,437 people).
- Page 2 filters out outliers (`IS COMPENSATION OUTLIERS = False`). That is why both median
  cards show 70.00K. The $65,000 median with outliers is from Phase 2.

## Answers to the Research Questions

### 1. Data quality: Who took the survey? How reliable is the pay data (about 64% missing)?

Mostly senior and mid-level developers from a handful of countries. The pay data is usable,
but thin: only 23,435 of 65,437 people (35.81%) reported it.

- Top 5 countries: United States (11,095), Germany (4,947), India (4,231), United Kingdom
  (3,224), Ukraine (2,672).
- Experience: Senior 18,460, Mid-level 12,653, Junior 10,834, Entry-level 9,663.
- Among people who reported pay (dashboard page 1), the split is similar: Senior 35.66%,
  Mid-level 26.9%, Junior 22.36%, Entry-level 14.69%.

### 2. Income distribution: How is pay spread out? How do I find outliers?

Pay is very skewed to the right, so the usual outlier rules fail on the raw value. I tried
3 methods and went with IQR on the log scale, which flagged 1,723 people (7.35%).

| Method        | Lower bound | Upper bound | Outliers flagged  |
| ------------- | ----------- | ----------- | ----------------- |
| 3-sigma (raw) | -474,116    | 646,426     | 89                |
| IQR (raw)     | -80,177     | 220,861     | 978               |
| **IQR (log)** | 5,454       | 647,443     | **1,723 (7.35%)** |

Skewness drops from 52.92 to -2.26 after the log-transform (Phase 1). The median is $65,000
with outliers and $70,000 without.

### 3. Salary drivers: How do country, experience, education, job type, and remote work affect pay?

Country matters most. Experience and remote work come next. I did not test job type
(`DevType`) against pay, so I can't say anything about it.

- **Country:** in the Phase 3 model, the United States (+1.54), Canada (+1.10), United
  Kingdom (+1.02), and Germany (+0.91) stand out, compared with Brazil (log scale).
- **Model fit:** `WorkExp` alone gives R² = 0.169. Adding the categorical features gives
  0.537.
- **Experience and education:** both have positive coefficients, but smaller than country
  (experience +0.244, Master's +0.21, Professional/Doctoral +0.19).
- **Employment:** I saw pay differences in Phase 2, but I don't trust the Phase 3
  `Employment` coefficients (see "Limits").

For remote work, the dashboard medians are a bit higher than Phase 2 because the dashboard
leaves out outliers. I read them from the chart, so they are rounded.

| Remote work | Phase 2 median (with outliers) | Dashboard median (no outliers) |
| ----------- | ------------------------------ | ------------------------------ |
| Remote      | $75,000 (n=9,591)              | about 80K                      |
| Hybrid      | $66,592 (n=9,907)              | about 70K                      |
| In-person   | $44,586 (n=3,937)              | about 51K                      |

By experience level, the dashboard shows about 96K for Senior, 73K for Mid-level, 50K for
Junior, and 35K for Entry-level.

### 4. Market trends: Which languages are most used? Most wanted? Highest paid?

JavaScript is the most used. Rust and Go gain the most interest. The best-paid languages are
the less common ones, like Erlang and Elixir.

- **Most used:** JavaScript 62.75%, HTML/CSS 53.25%, Python 51.42%, SQL 51.35%, TypeScript
  38.75%.
- **Most wanted:** among those top-10 languages, Python leads with 44.93%.
- **Biggest Want-vs-Have gap:** Rust (+18.26 pp), Go (+11.26 pp), Zig (+5.50 pp), Kotlin
  (+3.75 pp), Elixir (+3.11 pp).
- **Highest median pay (n >= 30):** Erlang $100,636 (n=229), Elixir $96,000 (n=598),
  Clojure $95,541 (n=350), Nim $94,924 (n=34), Ruby $90,221 (n=1,372).

The dashboard's "most used" chart shows counts, so its order is a bit different (SQL above
HTML/CSS, and PHP appears). I use the Phase 2 percentages here.

### 5. Income-satisfaction link: Does job satisfaction go up with pay and experience?

Only a little. The link is positive, but weak, for both.

- Satisfaction vs. pay (log): Pearson r = 0.080, Spearman r = 0.103 (n=16,075).
- Satisfaction vs. years of professional coding: Pearson r = 0.104, Spearman r = 0.119.
- Most people rate their job well. The most common score is 8 out of 10.
- On dashboard page 3, the scatter plot has points at every pay level for every
  satisfaction score. I see no clear upward trend.

## Other Things I Noticed

- **Missing pay is the main data problem.** `ConvertedCompYearly` is missing 64.19% and
  `CompTotal` 48.44%, too much to fill, so I left them empty (Phase 1).
- **Outliers change the story a little.** Removing them moves the median up by $5,000
  (Phase 2).
- **`Age_num` and `WorkExp` say almost the same thing** (r = 0.85), so I kept only
  `WorkExp` for the model.
- **Satisfaction data is thin.** 55.49% of people did not answer `JobSat`.
- **The model is for comparing, not predicting.** The average error is about $31,535
  (MAE), and I only used 15,011 rows (64.1% of the pay subset).
- **Check country medians with care.** Antigua and Barbuda and Andorra show up in the top
  10 on my dashboard, next to the United States (about 143K), but the chart has no sample
  size per country.

## What I Take From This

If I compare developer pay, I start with country, because it explains more than anything
else I tested. I use the median and flag outliers, since a few huge salaries pull the
average a long way.

I would not choose a language just because it pays well. The top-paid ones are rare, and my
data can't show that learning one raises your pay. I also would not assume a bigger salary
means a happier developer. The link is weak (r = 0.080).

## Limits

- My model has no p-values or confidence intervals, and I used a single train/test split.
- Pay is in USD and not adjusted for cost of living, so the country effect likely mixes
  real pay gaps with price differences.
- For `Employment`, I did not drop a reference group, and the field is multi-select, so
  those coefficients are hard to read.
- Language, industry, and company size are not in the model. That is a big part of why R²
  stops at 0.537.
- I did not check if the rows I dropped before modeling are different from the rest.

## Project Structure

| Phase                      | Folder                      |
| -------------------------- | --------------------------- |
| 1. Data Wrangling          | [`Phase 1/`](../Phase%201/) |
| 2. EDA                     | [`Phase 2/`](../Phase%202/) |
| 3. Modeling                | [`Phase 3/`](../Phase%203/) |
| 4. Reporting (this folder) | [`Phase 4/`](./)            |

- `images/` - dashboard screenshots.
