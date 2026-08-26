# Data Dictionary

This folder contains the two raw source files used in this project, exactly as published by the original researchers (Žagar & Mihelič, 2022 — see main [README](../README.md) for full citation). No columns were renamed in the raw files themselves; all cleaning was performed in Power Query and is documented below and in the main README.

---

## `Laboratory.csv`

1,005 rows — one row per final product batch. Combines batch identifiers, incoming raw material analysis, intermediate (tablet core) testing, and final product quality results.

### Identifiers & Categorical Attributes

| Column | Description | Used in this analysis? |
|---|---|---|
| `batch` | Unique final product batch number | Yes — primary key |
| `code` | Product sub-family code (maps to strength + size) | Yes — joins to `Dim_Product` |
| `strength` | Product strength: 5MG / 10M / 20M / 40M | **Yes — primary analysis variable** |
| `size` | Batch size category (one of 9 distinct sizes) | Yes — dashboard slicer |
| `start` | Batch start date, **month-and-year precision only** | Yes — see cleaning note below |

### Raw Material — Active Ingredient (API)

| Column | Description | Used? |
|---|---|---|
| `api_code`, `api_batch` | API supplier/batch identifiers | Used to trace missingness pattern |
| `api_water` | API water content | Median-imputed (<7% missing) |
| `api_total_impurities` | Total API impurities | Median-imputed (<7% missing) |
| `api_l_impurity` | Named API impurity ("Impurity L") | 37% null — see cleaning note below |
| `api_content` | API assay/content | Median-imputed (<7% missing) |
| `api_ps01`, `api_ps05`, `api_ps09` | API particle size distribution percentiles | Median-imputed (<7% missing) |

### Raw Material — Excipients

| Column | Description | Used? |
|---|---|---|
| `lactose_batch`, `smcc_batch`, `starch_batch` | Excipient batch identifiers | Not used in this analysis |
| `lactose_water`, `lactose_sieve0045/015/025` | Lactose moisture & sieve fractions | Not used |
| `smcc_water`, `smcc_td`, `smcc_bd`, `smcc_ps01/05/09` | Silicified microcrystalline cellulose properties | Not used |
| `starch_ph`, `starch_water` | Starch pH & moisture | Not used |

### Intermediate & Final Product Quality

| Column | Description | Used? |
|---|---|---|
| `tbl_min/max/av_hardness` | Tablet core hardness | Not used directly (informed `tbl_tensile`) |
| `tbl_min/max_thickness`, `fct_min/max_thickness` | Tablet thickness (core / film-coated) | Not used directly |
| `tbl_min/max_weight`, `tbl_rsd_weight`, `fct_rsd_weight` | Tablet weight & weight variability | Median-imputed (<7% missing) |
| `tbl_tensile`, `fct_tensile` | Tensile strength (normalized hardness measure) | Used in EDA correlation |
| `dissolution_av`, `dissolution_min` | **Drug release, average & minimum** | **Yes — primary outcome variable** |
| `residual_solvent` | Residual solvent content | Not used directly |
| `impurities_total` | Total final-product impurities | **Yes — secondary outcome (DAX measure)** |
| `impurity_o`, `impurity_l` | Named final-product impurities | Not used directly |

### Cleaning Notes

- **`start` date issue:** originally assumed to be a mix of full dates and month-only text. Direct verification against the raw source confirmed all 1,005 rows are month-and-year only. A `start_date_precision` flag column documents this.
- **`api_l_impurity` (37% null):** traced to a single API supplier code not reporting this test. An `api_l_impurity_tested` flag was added rather than imputing.
- Seven columns with <7% missingness were median-imputed (see table above).

---

## `Process.csv`

Per-batch, extracted summary features of the tablet compression process (originally derived from high-frequency time series data by the paper's authors — the raw second-by-second time series itself was not used in this project).

| Column | Description | Used? |
|---|---|---|
| `main_CompForce_mean` | Mean main compression force | **Yes — key regression predictor** |
| `tbl_speed_mean` | Mean tablet press speed | **Yes — regression predictor** |
| `tbl_fill_mean` | Mean tablet fill depth | Not used directly |
| `SREL_production_max` | Max standard relative deviation of compression force | Used in EDA correlation |
| `ejection_mean` | Mean ejection force | Not used |
| `total_waste` | Total wasted tablets | Used in EDA correlation (sanity check) |
| `batch_yield` | **Batch yield (%)** | **Yes — secondary outcome (DAX measure)** |
| `dissolution_av`, `dissolution_min`, `residual_solvent`, `impurities_total`, `impurity_o`, `impurity_l` | Duplicates of the same columns in Laboratory.csv | **Removed** — see cleaning note |

### Cleaning Notes

- Six columns (listed above) contained literal `"#N/A"` text and were null for the same final 18 batches.
- These were later found to be **exact duplicates** of columns already present in Laboratory.csv, where the complete, non-null versions live. The Process.csv duplicates were dropped in the final `Fact_Batch` table.

---

*For the star schema these two files were merged into (`Fact_Batch` + `Dim_Product`), see the main [README](../README.md#data-modeling--star-schema).*
