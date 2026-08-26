# DAX Measures

All measures were built in Power Pivot's Measure dialog (not typed directly into worksheet cells), so every chart, KPI card, and PivotTable in the dashboard references the same single, reusable calculation — no duplicated logic, no risk of two visuals quietly disagreeing with each other.

All five measures are defined on the `Fact_Batch` table.

---

### Total Batches

```DAX
Total Batches := COUNTROWS(Fact_Batch)
```

Counts the number of batches in the current filter context. Powers the "Total Batches" KPI card and responds to the Strength, Size, and Analysis Period slicers.

---

### Avg Dissolution

```DAX
Avg Dissolution := AVERAGE(Fact_Batch[dissolution_av])
```

The project's primary outcome metric. Feeds the "Avg Dissolution" KPI card, the "Average Dissolution by Strength" chart, and the "Average Dissolution Trend Over Time" chart.

---

### Avg Batch Yield

```DAX
Avg Batch Yield := AVERAGE(Fact_Batch[batch_yield])
```

Secondary outcome metric. Feeds the "Avg Batch Yield" KPI card and the "Average Batch Yield by Strength" chart. Notably shows close to the inverse ordering of the dissolution measure across strengths — 5MG (best on dissolution) is weakest on yield, and vice versa.

---

### Avg Total Impurities

```DAX
Avg Total Impurities := AVERAGE(Fact_Batch[impurities_total])
```

Feeds the "Avg Total Impurities" KPI card and the "Average Total Impurities by Strength" chart.

---

### Avg API Impurity

```DAX
Avg API Impurity := AVERAGE(Fact_Batch[api_l_impurity])
```

Feeds the "Avg API Impurity" KPI card. Note: this measure averages over only the batches where `api_l_impurity` was actually tested (see the `api_l_impurity_tested` flag in the [data dictionary](../data/README.md)) — `AVERAGE` in DAX automatically ignores blanks, so this does not need an explicit filter.

---

## Validation

All five measures were validated together using a single test PivotTable:

```
Rows:   strength (from Dim_Product)
Values: Total Batches, Avg Dissolution, Avg Batch Yield,
        Avg Total Impurities, Avg API Impurity
```

This confirmed the `Fact_Batch` ↔ `Dim_Product` relationship was working correctly, and produced the project's first early signal: **5MG already stood out with the highest dissolution and lowest impurity levels**, while 20M/40M trailed — a pattern that the formal statistical analysis (ANOVA, post-hoc tests, regression) later confirmed rigorously.

---

## Design Note: Why Measures, Not Calculated Columns

Every one of these is a **measure** (calculated on the fly, in whatever filter context a visual is currently in) rather than a **calculated column** (a fixed value computed once per row). This is what allows the same "Avg Dissolution" measure to show the overall average on the KPI card, the per-strength average on the bar chart, and a filtered average the moment someone clicks the Strength slicer — all from one formula, with no duplication.
