# Developer Market Trends & Salary Drivers

**Author:** Quan Le  
**University:** UEH (University of Economics Ho Chi Minh City)  
**Student ID:** 31241021233

I built this project to find what drives developer pay and job satisfaction, using the
Stack Overflow Developer Survey. I clean the data, explore it, build a simple model, and
report the findings, across 4 phases.

## Research Questions

1. **Data quality:** Who took the survey? How reliable is the pay data (about 64% missing)?
2. **Income distribution:** How is pay spread out? How do I find outliers?
3. **Salary drivers:** How do country, experience, education, job type, and remote work affect pay?
4. **Market trends:** Which languages are most used? Most wanted? Highest paid?
5. **Income-satisfaction link:** Does job satisfaction go up with pay and experience?

## Project Phases

| Phase             | What I do                                                                          | Folder                   |
| ----------------- | ---------------------------------------------------------------------------------- | ------------------------ |
| 1. Data Wrangling | Clean duplicates, missing values, and text labels. Encode and engineer features.   | [`Phase 1/`](Phase%201/) |
| 2. EDA            | Explore the cleaned data. Answer all 5 research questions with charts and stats.   | [`Phase 2/`](Phase%202/) |
| 3. Modeling       | Build a Simple and a Multiple Linear Regression to predict pay.                    | [`Phase 3/`](Phase%203/) |
| 4. Reporting      | Planned. Combine all findings into one report, built around the 5 questions above. | [`Phase 4/`](Phase%204/) |

Each phase folder has its own notebook and README (except for Phase 4), with the full details and results.

## Data

The raw survey file is not in this repo (about 150 MB, listed in `.gitignore`). I plan to
host it on GitHub Releases and link it here once that's set up.

## How to Run

Each phase's README has its own setup steps. In general: run Phase 1 first, then Phase 2,
then Phase 3, in that order. Each phase reads the file(s) the one before it saved.
