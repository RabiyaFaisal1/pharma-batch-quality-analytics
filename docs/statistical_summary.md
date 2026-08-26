# Statistical Summary

Full statistical workflow and results for the Pharmaceutical Batch Quality & Process Performance Analytics project. This is the detailed backing document for the [Statistical Analysis](../README.md#statistical-analysis) section of the main README.

---

## 1. One-Way ANOVA — Does Strength Affect Dissolution?

**Question:** Does mean dissolution genuinely differ across the four formulation strengths (5MG, 10M, 20M, 40M), or could the differences we see just be random sampling noise?

**Method:** One-way ANOVA (Excel Data Analysis ToolPak), `dissolution_av` as the dependent variable, `strength` as the grouping factor.

| Source of Variation | F | p-value | F crit |
|---|---:|---:|---:|
| Between Groups | 181.663 | 4.97E-94 | 2.614 |

**Result:** F (181.66) far exceeds F crit (2.61); p is effectively zero. **The null hypothesis of equal means is decisively rejected** — dissolution genuinely differs by strength.

Group means:

| Strength | n | Mean Dissolution |
|---|---:|---:|
| 5MG | 249 | 93.72 |
| 10M | 213 | 91.27 |
| 20M | 445 | 89.18 |
| 40M | 98 | 88.15 |

Group variances ranged 6.63–7.74 (ratio ~1.17×), supporting the equal-variances assumption used in the ANOVA and the pairwise tests below.

---

## 2. Post-hoc Pairwise Comparisons

ANOVA only tells us *some* group differs — not *which* ones. Six pairwise two-sample t-tests (equal variances assumed) were run, with a **Bonferroni-corrected significance threshold of 0.05 ÷ 6 ≈ 0.0083** to control the false-positive rate across multiple comparisons.

| Pair | Mean Difference | t Stat | p-value | Significant (< 0.0083)? |
|---|---:|---:|---:|---|
| 5MG vs 10M | 2.45 | 9.90 | 4.33E-21 | ✅ |
| 5MG vs 20M | 4.54 | 20.80 | 5.26E-75 | ✅ |
| 5MG vs 40M | 5.58 | 17.31 | 9.28E-49 | ✅ |
| 10M vs 20M | 2.09 | 9.23 | 3.61E-19 | ✅ |
| 10M vs 40M | 3.13 | 9.84 | 4.79E-20 | ✅ |
| 20M vs 40M | 1.04 | 3.36 | 0.00083 | ✅ (weakest of the six) |

**Result:** every pairwise comparison is statistically significant. All four strengths are genuinely, independently distinct from one another, in a strict order:

```
5MG (93.72%) > 10M (91.27%) > 20M (89.18%) > 40M (88.15%)
```

The 20M–40M comparison is the closest of the six — these two strengths are the most similar to each other, both sitting at the lower-performing end.

---

## 3. Simple Linear Regression

**Question:** Does compression force predict dissolution?

```
dissolution_av = 94.55 − 0.6455 × main_CompForce_mean
```

| Metric | Value |
|---|---:|
| R² | 0.132 |
| Coefficient (compression force) | −0.6455 |
| p-value | 9.55E-33 |

**Initial read:** statistically significant negative relationship — for every one-unit increase in compression force, dissolution drops ~0.645 points on average. Explains ~13.2% of dissolution's variance. Taken alone, this looks like a real, independent driver.

---

## 4. Multiple Regression — Testing for Confounding

**Question:** Does compression force's effect survive once formulation strength is also accounted for?

**Model:** `dissolution_av ~ main_CompForce_mean + tbl_speed_mean + Is_5MG + Is_10M + Is_20M` (40M as baseline; 3 dummy variables to avoid the dummy-variable trap)

| Predictor | Coefficient | p-value | Significant? |
|---|---:|---:|---|
| Is_5MG (vs. 40M) | +5.31 | 1.09E-24 | ✅ |
| Is_10M (vs. 40M) | +2.48 | 9.28E-06 | ✅ |
| Is_20M (vs. 40M) | +0.42 | 0.381 | ❌ |
| Tablet speed | +0.0143 | 0.024 | ✅ (marginal) |
| **Compression force** | **−0.0145** | **0.837** | **❌** |

**R² = 0.356** (up from 0.132 in the simple model).

**Result — the key finding:** compression force's coefficient collapsed from a strongly significant −0.6455 to a statistically insignificant −0.0145 once strength was controlled for. **Interpretation:** compression force was never an independent cause of dissolution differences — it was acting as a proxy for strength, because different strength formulations are likely manufactured using systematically different compression settings. Once the true underlying driver (strength) is directly modeled, compression force's apparent contribution nearly disappears.

This is a textbook example of **confounding**, and it's the single most analytically important result in this project: a relationship that looked genuine and statistically clean in a simple model did not survive being tested more rigorously.

---

## 5. Weekend vs. Weekday — Abandoned as Statistically Invalid

An initial pass classified batches as weekday/weekend production (713 weekday / 292 weekend), giving dissolution averages of 90.68 (weekday) vs. 90.16 (weekend).

**Before running a formal t-test on this split, the classification itself was checked against the raw source data.** This revealed that `start_date` records **month-and-year only** for every single batch — the day-of-month had been defaulted to 1 during earlier cleaning, since no real day value exists in the source. A weekday/weekend label built on a defaulted day is therefore an artifact of date reconstruction, not a real observation of when a batch was actually produced.

**Decision:** the planned t-test was not run or reported. This is documented as a deliberate methodological limitation, not a completed (and misleading) result.

---

## 6. Written Summary

> Across 1,005 batches spanning 4 product strengths, formulation strength emerged as the dominant, statistically confirmed driver of dissolution performance. A one-way ANOVA found highly significant differences across the four strength groups (F=181.66, p<0.001), and all six pairwise post-hoc comparisons confirmed every strength is genuinely distinct from every other, in a consistent order: 5MG > 10M > 20M > 40M. A simple regression initially suggested compression force had a significant negative relationship with dissolution, but a multiple regression combining compression force, tablet speed, and strength showed compression force's effect became statistically insignificant once strength was controlled for, while strength's effect remained strong — indicating compression force's apparent influence was largely confounded with strength rather than an independent causal driver. A planned weekend-versus-weekday comparison could not be validly tested, since the source data's start_date field records only month and year, not day of production — this is noted as a data-granularity limitation rather than a completed finding.
