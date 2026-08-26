# 💊 Pharmaceutical Batch Quality & Process Performance Analytics

**What actually drives dissolution performance in tablet manufacturing — formulation, or the process settings on the compression line?**

An end-to-end Excel analytics project built on 1,005 real pharmaceutical production batches — from raw CSV cleaning through statistical hypothesis testing to a fully interactive, slicer-driven dashboard.

<img width="1343" height="681" alt="image" src="https://github.com/user-attachments/assets/ab7fb5ea-67cc-4f06-ba1d-48bfc9d43edd" />
---

## 📌 TL;DR

> **Formulation strength — not raw material impurity, not compression force — is the dominant, statistically confirmed driver of dissolution performance.** A relationship that looked real in a simple regression (compression force → dissolution) disappeared almost entirely once strength was properly controlled for in a multiple regression. That's the core analytical story this project tells, end to end.

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Tools & Techniques](#tools--techniques)
- [Data Cleaning](#data-cleaning-power-query)
- [Data Modeling](#data-modeling--star-schema)
- [DAX Measures](#dax-measures)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Statistical Analysis](#statistical-analysis)
- [Interactive Dashboard](#interactive-dashboard)
- [Key Findings](#key-findings)
- [Limitations](#limitations)
- [Repository Structure](#repository-structure)
- [How to Explore This Project](#how-to-explore-this-project)
- [Data Source & Citation](#data-source--citation)

---

## Overview

Dissolution is one of the most important quality attributes of an oral tablet — it determines how reliably a patient actually absorbs the intended dose. This project asks a genuine, operationally relevant question of a real manufacturing dataset: **which upstream variables actually explain batch-to-batch differences in dissolution** — and does it end there, or does one variable's apparent effect turn out to be riding on another's?

The entire workflow — cleaning, relational data modeling, DAX, hypothesis testing, regression, and an interactive dashboard — was built natively in **Excel 2016**, deliberately without reaching for external statistical software, to demonstrate that a full, rigorous analytics pipeline can live inside a tool nearly every workplace already has.

---

## Dataset

This project uses a real, peer-reviewed, publicly available manufacturing dataset — not a synthetic or toy dataset.

| | |
|---|---|
| **Batches** | 1,005 real production batches |
| **Time span** | November 2018 – April 2021 |
| **Product** | Film-coated, immediate-release tablets (cholesterol-lowering medicine) across 4 strengths and 9 batch sizes |
| **Files used** | `Laboratory.csv` (raw material + final product quality results), `Process.csv` (extracted, per-batch summary features of the tablet compression process) |
| **Source** | Žagar & Mihelič (2022), *Scientific Data* (Nature) — see [full citation](#data-source--citation) |

All batch, material, and personnel identifiers in the original dataset were anonymized by the original researchers prior to publication.

---

## Tools & Techniques

`Excel 2016` · `Power Query (M)` · `Power Pivot` · `DAX` · `PivotTables & PivotCharts` · `Slicers & Timelines` · `Data Analysis ToolPak` · `ANOVA` · `Bonferroni-corrected pairwise t-tests` · `Simple & Multiple Linear Regression` · `Correlation Analysis`

---

## Data Cleaning (Power Query)

Both source files were cleaned entirely in Power Query, with every transformation step kept auditable in the Applied Steps pane.

**Laboratory.csv**
- Verified no duplicate batches via Group By.
- **Corrected a mid-project data assumption:** the `start` field was initially believed to be a mix of full dates and month-only text. Direct verification against the raw source showed **all 1,005 batches record month-and-year only** — there is no day-of-month anywhere in the source. A `start_date_precision` flag documents this transparently.
- `api_l_impurity` was 37% null — traced entirely to a single raw material supplier not reporting that test (a structural gap, not random missingness), so an `api_l_impurity_tested` flag was added instead of imputing.
- Seven smaller-gap columns (<7% missing) were median-imputed.
<img width="291" height="212" alt="image" src="https://github.com/user-attachments/assets/15cbb891-3a8f-4af2-94e0-95e70d7cee8d" />

**Process.csv**
- Verified no duplicates.
- Fixed literal `"#N/A"` text in 6 columns before type conversion.
- **Discovered a cross-file duplication issue:** those same 6 columns turned out to be exact duplicates of columns already present in Laboratory.csv — except Laboratory's versions were complete while Process's were missing the final 18 batches. The redundant columns were removed in favor of the complete versions.

**Merge:** joined on batch/code (Left Outer) → **`Fact_Batch`**, confirmed at exactly 1,005 rows with no duplication.

<img width="455" height="320" alt="image" src="https://github.com/user-attachments/assets/748f7eea-69c1-491e-a789-3f5ed939ae1a" />

---

## Data Modeling — Star Schema

A proper relational **star schema** was built rather than one flat table, to demonstrate data modeling technique and simplify every DAX measure and dashboard slicer that follows.

```
Dim_Product (25 unique product combinations: code, strength, size)
        │
        │  code  (one-to-many)
        ▼
Fact_Batch (1,005 batch-level records: quality + process measurements)
```

Relationship built and activated in Power Pivot's Diagram View.

<img width="381" height="131" alt="image" src="https://github.com/user-attachments/assets/f87fabc6-9566-49fd-baf6-ed6cb35fbaf0" />

---

## DAX Measures

| Measure | DAX |
|---|---|
| Total Batches | `COUNTROWS(Fact_Batch)` |
| Avg Dissolution | `AVERAGE(Fact_Batch[dissolution_av])` |
| Avg Batch Yield | `AVERAGE(Fact_Batch[batch_yield])` |
| Avg Total Impurities | `AVERAGE(Fact_Batch[impurities_total])` |
| Avg API Impurity | `AVERAGE(Fact_Batch[api_l_impurity])` |

Validated with a test PivotTable (strength × all 5 measures) — the first early signal that 5MG consistently outperformed the other strengths.

<img width="543" height="118" alt="image" src="https://github.com/user-attachments/assets/08473266-aa6d-4e6a-ad55-a21314312d40" />

---

## Exploratory Data Analysis

**Correlation screening** (Pearson, across all 1,005 batches):

| Variable Pair | r | Read |
|---|---:|---|
| Compression force vs. Dissolution | −0.36 | Moderate |
| Compression force vs. Tablet tensile strength | −0.49 | Moderate |
| Total waste vs. Batch yield | −0.55 | Moderate–strong (sanity check) |
| Tablet press speed vs. Dissolution | −0.105 | Weak |
| API impurity vs. Dissolution | −0.036 | Negligible |
| SREL (production max) vs. Dissolution | 0.056 | Negligible |

**Dissolution histogram:** a histogram of `dissolution_av` was built to check the shape and spread of the distribution before running ANOVA — mirroring the same distribution-check approach the original dataset's own authors used (they validated dissolution against a normal curve to compute a process-capability index). The distribution looked reasonably bell-shaped with no extreme outliers, supporting the use of ANOVA and t-tests downstream.

<img width="640" height="346" alt="image" src="https://github.com/user-attachments/assets/3ba9d688-5e0e-4c15-bc9b-8fc3409313f4" />

**Outlier scan:** the 11 worst-dissolution batches belonged **exclusively** to the 20M/40M strength groups — an early, strong signal that formulation strength, not raw material impurity, was the main driver of poor outcomes.

---

## Statistical Analysis

### One-Way ANOVA
Does strength affect dissolution?

| F | p-value | F crit | Conclusion |
|---:|---:|---:|---|
| 181.663 | 4.97E-94 | 2.614 | **Highly significant** |

<img width="596" height="267" alt="image" src="https://github.com/user-attachments/assets/8f3efdb0-7f34-4e26-9c51-46672f9cb8c1" />


### Post-hoc Pairwise t-tests (Bonferroni α = 0.0083)

| Pair | p-value | Significant? |
|---|---:|---|
| 5MG vs 10M | 4.33E-21 | ✅ |
| 5MG vs 20M | 5.26E-75 | ✅ |
| 5MG vs 40M | 9.28E-49 | ✅ |
| 10M vs 20M | 3.61E-19 | ✅ |
| 10M vs 40M | 4.79E-20 | ✅ |
| 20M vs 40M | 0.00083 | ✅ (weakest) |

**Confirmed order:** 5MG (93.72%) > 10M (91.27%) > 20M (89.18%) > 40M (88.15%)

<img width="443" height="245" alt="image" src="https://github.com/user-attachments/assets/a98c9c2c-c922-484d-91aa-b8f47d6d62ef" />

### Simple Regression
```
dissolution_av = 94.55 − 0.6455 × main_CompForce_mean      R² = 0.132, p = 9.55E-33
```
Looked like a real, independent effect — on its own.
### Multiple Regression — the confounding discovery
Adding strength (as dummy variables, 40M baseline) alongside compression force and tablet speed:

| Predictor | Coefficient | p-value | Significant? |
|---|---:|---:|---|
| Is_5MG (vs. 40M) | +5.31 | 1.09E-24 | ✅ |
| Is_10M (vs. 40M) | +2.48 | 9.28E-06 | ✅ |
| Is_20M (vs. 40M) | +0.42 | 0.381 | ❌ |
| Tablet speed | +0.0143 | 0.024 | ✅ (marginal) |
| **Compression force** | **−0.0145** | **0.837** | **❌** |

**R² rose from 0.132 → 0.356.** Compression force's coefficient collapsed to statistically insignificant once strength was controlled for — its apparent effect in the simple model was **confounded with strength**, not an independent cause. This is the project's central analytical finding.

<img width="624" height="306" alt="image" src="https://github.com/user-attachments/assets/bedd74cd-c17b-49d4-a1f1-68bc754c9f65" />

### Weekend vs. Weekday — Abandoned as Invalid
A planned comparison was **not run**, because the `start_date` field records month+year only — any weekday/weekend classification built on a defaulted day-of-month would be an artifact of reconstruction, not a real observation. Documented here as a deliberate methodological decision, not a gap.

---

## Interactive Dashboard

A single-page dashboard translates every finding above into something explorable, not just readable.

- **5 live KPI cards** (Total Batches, Avg Dissolution, Avg Batch Yield, Avg Total Impurities, Avg API Impurity), all DAX-driven
- **7 charts:** Dissolution / Yield / Total Impurities by Strength, Dissolution Trend Over Time, Batch Count Mix (doughnut), a Compression Force vs. Dissolution scatter plot with trendline, and a Regression Coefficient chart visualizing the confounding finding
- **Two "Insights" panels** — Statistical Insights and EDA Insights — surfacing the analysis in plain language directly on the dashboard
- **Interactivity:** Strength slicer, Size slicer, and an Analysis Period timeline, all connected via Report Connections to every pivot and chart — including the KPI cards
- **Consistent 4-color palette** mapped to strength across every chart (navy/teal/light-blue/amber), with amber deliberately reserved for 40M — the weakest performer — as a visual cue that reinforces the data's own finding

---

## Key Findings

1. **Formulation strength is the dominant, statistically confirmed driver of dissolution** — confirmed independently by ANOVA, all 6 pairwise post-hoc tests, and multiple regression.
2. **Compression force's apparent effect was confounded with strength** — a textbook example of why single-variable regression can mislead.
3. **A planned analysis (weekend vs. weekday) was correctly identified as invalid** once verified against the raw source data, rather than run and reported anyway.
4. Raw material (API) impurity showed a negligible relationship with dissolution in this dataset.

---

## Limitations

- `start_date` has month-year precision only — any daily or weekday-level analysis is not valid on this data.
- The multiple regression model uses a deliberately focused set of predictors; a larger model could incorporate additional raw-material and process variables.
- Correlation and regression identify statistical association, not proof of causation.
- This dashboard is an analytical/portfolio artifact, not a validated release-decision tool.

---

## Repository Structure

```
pharma-batch-quality-analytics/
├── README.md
├── data/
│   ├── raw/
│   │   ├── Laboratory.csv
│   │   └── Process.csv
│   └── README.md                # data dictionary / column notes
├── workbook/
│   └── Pharmaceutical_Batch_Analytics.xlsx
├── docs/
│   ├── dashboard_preview.png
│   ├── dax_measures.md
│   ├── statistical_summary.md
│   └── screenshots/
│       ├── data_cleaning_steps.png
│       ├── star_schema_diagram.png
│       ├── anova_output.png
│       ├── regression_output.png
│       └── dashboard_full.png
└── report/
    └── Pharmaceutical_Batch_Analytics_Report.docx
```

---

## How to Explore This Project

1. Clone or download this repository.
2. Open `workbook/Pharmaceutical_Batch_Analytics.xlsx` in Excel (2016 or later).
3. Go to the **Dashboard** sheet.
4. Use the **Strength** and **Size** slicers, and the **Analysis Period** timeline, to filter the view.
5. Read the **Statistical Insights** and **EDA Insights** panels for the plain-language findings behind the charts.
6. For the full statistical workflow and detailed write-up, see `report/Pharmaceutical_Batch_Analytics_Report.docx`.

---

## Data Source & Citation

This project uses the dataset published in:

> Žagar, J., Mihelič, J. **Big data collection in pharmaceutical manufacturing and its use for product quality predictions.** *Scientific Data* **9**, 99 (2022).
> DOI: [10.1038/s41597-022-01203-x](https://doi.org/10.1038/s41597-022-01203-x)
> Data records: [https://doi.org/10.6084/m9.figshare.c.5645578.v1](https://doi.org/10.6084/m9.figshare.c.5645578.v1)
> Licensed under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/)

All analysis, cleaning decisions, statistical testing, and dashboard design in this repository are original work built on top of that published dataset.

---

*Built by Rabiya as part of a data analytics portfolio — feedback and questions welcome.*
