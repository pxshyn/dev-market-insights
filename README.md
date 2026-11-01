# Developer Market Trends & Salary Drivers

I built this project to find what drives developer pay and job satisfaction, using the
Stack Overflow Developer Survey. I clean the data, explore it, build a simple model, and
report the findings, across 4 phases.

[![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)](https://www.python.org/) [![NumPy](https://img.shields.io/badge/NumPy-013243?logo=numpy&logoColor=white)](https://numpy.org/) [![pandas](https://img.shields.io/badge/pandas-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/) [![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C)](https://matplotlib.org/) [![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0)](https://seaborn.pydata.org/) [![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/) [![Jupyter](https://img.shields.io/badge/Jupyter-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/) [![Power BI](https://img.shields.io/badge/Power_BI-F2C811?logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)

**Live dashboard:** [View on Power BI](https://app.powerbi.com/groups/me/reports/e378ce91-9ee0-4d3c-9c20-b2764bd2778f?ctid=a74aa7fa-f7f9-4020-a84a-057abbba6e9b&pbi_source=linkShare)

## Research Questions

1. **Data quality:** Who took the survey? How reliable is the pay data (about 64% missing)?
2. **Income distribution:** How is pay spread out? How do I find outliers?
3. **Salary drivers:** How do country, experience, education, job type, and remote work affect pay?
4. **Market trends:** Which languages are most used? Most wanted? Highest paid?
5. **Income-satisfaction link:** Does job satisfaction go up with pay and experience?

## Project Phases

| Phase                 | What I do                                                                        | Folder                   |
| --------------------- | -------------------------------------------------------------------------------- | ------------------------ |
| 1. Data Wrangling     | Clean duplicates, missing values, and text labels. Encode and engineer features. | [`Phase 1/`](Phase%201/) |
| 2. EDA                | Explore the cleaned data. Answer all 5 research questions with charts and stats. | [`Phase 2/`](Phase%202/) |
| 3. Modeling           | Build a Simple and a Multiple Linear Regression to predict pay.                  | [`Phase 3/`](Phase%203/) |
| 4. Power BI Dashboard | Bring the findings together in an interactive 4-page dashboard.                  | [`Phase 4/`](Phase%204/) |

Each phase folder has its own README with the full details and results. Phases 1-3 also
have a notebook. Phase 4 has a `.pbix` file instead, because I built it in Power BI.

## Data

The raw survey file is not in this repo (about 150 MB, listed in `.gitignore`). Download
it here: [survey_data_duplicates.csv](https://github.com/pxshyn/dev-market-insights/releases/download/data-v1/survey_data_duplicates.csv)

Place it at `data/survey_data_duplicates.csv` before running Phase 1.

## How to Run

Each phase's README has its own setup steps. In general: run Phase 1 first, then Phase 2,
then Phase 3, in that order. Each phase reads the file(s) the one before it saved. Phase 4
is a Power BI file, so there is nothing to run: open it in Power BI Desktop, or use the
live dashboard link above.
