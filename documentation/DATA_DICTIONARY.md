# Data Dictionary & Field Catalog
**Global Crop Yield & Agricultural Input Analytics**  
FAO / World Bank agricultural data · 101 countries · 10 crops · 1990–2013

## 1. Overview

This document describes the data model underlying the Global Crop Yield & Agricultural Input Analytics report: every table, column, relationship, and DAX measure, as implemented in the Power BI model.

## 2. Data Model — Tables

Star schema: one fact table and three dimension tables. The fact table holds 13,130 distinct country-crop-year records.

### `yield_df` (fact) — 13,130 rows

| Column | Type | Description |
| --- | --- | --- |
| `Location_Crop_Key` | Text | Concatenation of `Area` and `Item`, e.g. `Albania-Maize`. A display key, not a model relationship key. |
| `Area` | Text | Country name. Relates to `Dim_Location[Area]`. |
| `Item` | Text | Crop name (10 distinct values). Relates to `Dim_Crop[Item]`. |
| `Year` | Whole number | 1990–2013. 2003 is absent from the source data — see the time-intelligence measures. Relates to `Dim_Date[Year]`. |
| `hg/ha_yield` | Decimal | Yield density in hectograms per hectare. An intensive quantity — see Section 7 before aggregating with SUM. |
| `average_rain_fall_mm_per_year` | Decimal | Country-year rainfall figure, constant across every crop grown in that country-year. |
| `pesticides_tonnes` | Decimal | Country-year pesticide use figure. Same grain as rainfall. |
| `avg_temp` | Decimal | Mean annual temperature for the country-year. |


### `Dim_Crop` — 10 rows

| Column | Type | Description |
| --- | --- | --- |
| `Item` | Text | Crop name. Primary lookup key. |
| `CropID` | Whole number | Surrogate key generated in Power Query. |


### `Dim_Location` — 101 rows

| Column | Type | Description |
| --- | --- | --- |
| `Area` | Text | Country name. Primary lookup key. |
| `LocationID` | Whole number | Surrogate key generated in Power Query. |


### `Dim_Date` — 24 rows

| Column | Type | Description |
| --- | --- | --- |
| `Year` | Whole number | 1990–2013 inclusive, excluding 2003 (24 distinct years present in the source data). |


## 3. Relationships

| From | To | Cardinality | Cross-filter |
| --- | --- | --- | --- |
| `yield_df[Item]` | `Dim_Crop[Item]` | Many-to-one | Single |
| `yield_df[Year]` | `Dim_Date[Year]` | Many-to-one | Single |
| `yield_df[Area]` | `Dim_Location[Area]` | Many-to-one | Single |


## 4. DAX Measures (15)

Grouped by function, with the exact expression as implemented in the model.

### Base aggregation

**`Observations Analyzed`**

```dax
COUNTROWS(yield_df)
```

*Count of distinct country-crop-year records in the fact table. Used on the Executive Overview page as the primary scale indicator for the dataset.*

**`Average Yield (hg/ha)`**

```dax
AVERAGE(yield_df[hg/ha_yield])
```

*Mean yield density across whatever rows are in filter context. This is the model's headline yield figure — a pooled average across all 10 crop types. See the Executive Briefing for how to read it alongside the per-crop breakdown.*

**`Avg Pesticide Use (Tonnes)`**

```dax
AVERAGE(yield_df[pesticides_tonnes])
```

*Mean pesticide use across filter context. Reported as an average rather than a total because pesticide use is recorded once per country-year and repeats across every crop grown that year — an average per country-year is the correct, interpretable aggregation; a sum would overstate the true figure by the number of crops observed.*

**`Average Rainfall (mm)`**

```dax
AVERAGE(yield_df[average_rain_fall_mm_per_year])
```

*Mean annual rainfall across filter context. Same country-year grain as pesticide use.*

**`Average Temperature (°C)`**

```dax
AVERAGE(yield_df[avg_temp])
```

*Mean temperature across filter context, computed on the deduplicated fact table.*

### Ratio / efficiency

**`Yield per Pesticide Tonne`**

```dax
DIVIDE([Average Yield (hg/ha)], [Avg Pesticide Use (Tonnes)], 0)
```

*Yield achieved per tonne of pesticide used — an input-efficiency indicator.*

**`Yield per Rainfall mm`**

```dax
DIVIDE(
    [Average Yield (hg/ha)],
    [Average Rainfall (mm)],
    0
)
```

*Yield achieved per millimetre of rainfall — a water-efficiency indicator.*

### Time intelligence

**`Yield PY`**

```dax
VAR CurrentYear = MAX('Dim_Date'[Year])
VAR HasDataForPreviousYear =
    CALCULATE(
        NOT ISBLANK([Average Yield (hg/ha)]),
        ALL('Dim_Date'[Year]),
        'Dim_Date'[Year] = CurrentYear - 1
    )
RETURN
IF(
    HasDataForPreviousYear,
    CALCULATE(
        [Average Yield (hg/ha)],
        ALL('Dim_Date'[Year]),
        'Dim_Date'[Year] = CurrentYear - 1
    ),
    BLANK()
)
```

*Prior-year yield, calculated with an explicit check for whether a prior year exists in the data before looking it up, rather than assuming a continuous calendar. This matters because 2003 has no records in the source data.*

**`Yield YoY Growth %`**

```dax
AVERAGEX(
    VALUES('Dim_Date'[Year]),
    VAR CurrentYear = 'Dim_Date'[Year]
    VAR CurrentYield = [Average Yield (hg/ha)]
    VAR PreviousYield =
        CALCULATE(
            [Average Yield (hg/ha)],
            ALL('Dim_Date'[Year]),
            'Dim_Date'[Year] = CurrentYear - 1
        )
    RETURN
    IF(
        ISBLANK(PreviousYield) || PreviousYield <= 0 || ISBLANK(CurrentYield),
        BLANK(),
        DIVIDE(CurrentYield - PreviousYield, PreviousYield)
    )
)
```

*Year-over-year growth rate, built on the same gap-aware logic as Yield PY, correctly skipping the missing 2003 boundary instead of producing a false spike or drop.*

### Diagnostic statistic

**`Correlation Yield vs Temp`**

```dax
VAR MeanX = CALCULATE(AVERAGE(yield_df[avg_temp]), ALLSELECTED(yield_df))
VAR MeanY = CALCULATE(AVERAGE(yield_df[hg/ha_yield]), ALLSELECTED(yield_df))
VAR SummarizedTable =
    ADDCOLUMNS(
        SUMMARIZE(yield_df, yield_df[Area], yield_df[Item], yield_df[Year]),
        "@X", CALCULATE(AVERAGE(yield_df[avg_temp])),
        "@Y", CALCULATE(AVERAGE(yield_df[hg/ha_yield]))
    )
VAR Covariance =
    SUMX(SummarizedTable, ([@X] - MeanX) * ([@Y] - MeanY))
VAR VarianceX =
    SUMX(SummarizedTable, ([@X] - MeanX) ^ 2)
VAR VarianceY =
    SUMX(SummarizedTable, ([@Y] - MeanY) ^ 2)
VAR Denominator = SQRT(VarianceX * VarianceY)
RETURN
    DIVIDE(Covariance, Denominator, 0)
```

*Pearson correlation coefficient between temperature and yield, computed via covariance and variance (Power BI has no native CORREL function). A descriptive statistic — it measures association, not causation.*

**`Correlation Yield vs Pesticides`**

```dax
-- Same covariance/variance pattern as Correlation Yield vs Temp,
-- substituting yield_df[pesticides_tonnes] for yield_df[avg_temp]
```

*Pearson correlation between pesticide use and yield.*

**`Correlation Yield vs Rainfall`**

```dax
-- Same covariance/variance pattern as Correlation Yield vs Temp,
-- substituting yield_df[average_rain_fall_mm_per_year] for yield_df[avg_temp]
```

*Pearson correlation between rainfall and yield.*

### Ranking

**`Top Yield Crop`**

```dax
TOPN(1, VALUES(Dim_Crop[Item]), [Average Yield (hg/ha)], DESC)
```

*The single highest-average-yield crop in current filter context.*

**`Top Yield Country`**

```dax
TOPN(1, VALUES(Dim_Location[Area]), [Average Yield (hg/ha)], DESC)
```

*The single highest-average-yield country in current filter context.*

### Spread statistic

**`Yield Variance %`**

```dax
VAR MaxYield = MAXX(VALUES(Dim_Crop[Item]), [Average Yield (hg/ha)])
VAR MinYield = MINX(VALUES(Dim_Crop[Item]), [Average Yield (hg/ha)])
RETURN DIVIDE(MaxYield - MinYield, MinYield, 0)
```

*Ratio of the highest to lowest crop average yield in current context (displayed as 'Crop Yield Spread Ratio'). A high value here is expected — it reflects the crop-type heterogeneity described in the Data Dictionary's interpretation notes.*

## 5. Power Query (M) Transformation — `yield_df`

Load sequence, in applied-step order:

- **Source** — `Csv.Document` reading `yield_df.csv` (see [System Architecture](SYSTEM_ARCHITECTURE.md#5-recommendations-before-wider-distribution) — the source path must be updated for this to refresh on a machine other than the original author's).
- **Promoted headers**
- **Changed column type** — type casting on all 8 columns
- **Removed Columns** — drops an unnamed index column carried over from the CSV
- **Trimmed Text** — whitespace trim on `Area` and `Item`
- **Changed Type** — re-cast `average_rain_fall_mm_per_year` to decimal
- **Filtered Rows** — a placeholder pass-through step, not an active filter
- **Grouped Rows** — `Table.Group` on `{Area, Item, Year}`: `List.Max` for `hg/ha_yield`, `average_rain_fall_mm_per_year`, and `pesticides_tonnes` (constant within each group); `List.Average` for `avg_temp`. This step consolidates the source data to one record per country-crop-year.
- **Added Custom** — `Location_Crop_Key` (`Area & "-" & Item`)
- **Reordered Columns** — for display

## 6. Data Provenance

Source: FAO and World Bank agricultural statistics, 1990–2013, published as the ["Crop Yield Prediction Dataset"](https://www.kaggle.com/) on Kaggle, pre-merged by the dataset's original author. This project ingests that single merged file (`yield_df.csv`). The separate `rainfall.csv`, `temp.csv`, `pesticides.csv`, and `yield.csv` files referenced by the original dataset are not bundled in this repository, since they are not active inputs to the model — see the original Kaggle listing if you need them.

**License: not confirmed.** Check the dataset's Kaggle page for its license terms and cite it explicitly before reusing the underlying data beyond this portfolio context.

## 7. Interpreting the Model — Notes for Any User

- `hg/ha_yield` is an intensive quantity (a density, not a count). Average it; do not sum it across countries, crops, or years — a sum has no real-world unit meaning.
- `pesticides_tonnes` and `average_rain_fall_mm_per_year` are recorded once per country-year and repeat across every crop grown that year. Any new measure built on these two columns should use `AVERAGE` or `MAX`, or should read from a country-year-grain table — never `SUM` directly on `yield_df`.
- `Average Yield (hg/ha)` is a pooled average across 10 botanically dissimilar crop types (cereals vs. roots/tubers), whose natural yield scales differ by roughly 10x. It will not match any single crop's own average — see the [Executive Briefing](EXECUTIVE_BRIEFING.pdf) for the per-crop figures and the reasoning behind the gap.
