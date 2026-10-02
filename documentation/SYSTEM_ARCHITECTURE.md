# System Architecture & Technical Documentation
**Global Crop Yield & Agricultural Input Analytics**

## 1. What This System Does

This is a descriptive and diagnostic Power BI dashboard over a static, historical dataset covering 101 countries and 10 crops from 1990 to 2013. It computes aggregates, gap-aware year-over-year growth, and Pearson correlation coefficients between yield and three agricultural inputs. It does not perform predictive modeling — there is no regression, no classification, no train/test split, and no evaluation metric anywhere in the model. Its scope is explaining and summarizing historical patterns, not forecasting future outcomes.

## 2. Data Flow

1. A single pre-merged CSV (`yield_df.csv`) is read via Power Query. This file was prepared outside this project by the original dataset author, drawing on four underlying FAO/World Bank sources.
2. Power Query applies type casting, trimming, and a consolidation step that reduces the source data to one record per country-crop-year (see Section 4).
3. Three dimension tables (`Dim_Crop`, `Dim_Location`, `Dim_Date`) are derived from the fact table by distinct-value extraction, forming a star schema.
4. The model loads into Power BI's in-memory engine with 15 DAX measures layered on top.
5. Three report pages (Executive Overview, Agronomic & Input Impact, Regional & Crop Deep-Dive) consume the model across 37 visuals.

## 3. Data Model

Star schema — one fact table (`yield_df`, 13,130 rows) and three dimensions (`Dim_Crop`, `Dim_Location`, `Dim_Date`), joined by single-direction many-to-one relationships. Full column-level detail and DAX text is in the [Data Dictionary](DATA_DICTIONARY.md).

## 4. Data Preparation

The source file pairs each yield observation against multiple temperature readings for the same country, crop, and year, while the yield, rainfall, and pesticide figures themselves stay constant within each such group. The Power Query layer consolidates this to one record per country-crop-year: yield, rainfall, and pesticide values are carried through via a max aggregation (safe, since they are constant within each group), and temperature is averaged across all original readings for that group. This is implemented as a single `Table.Group` step, detailed in the [Data Dictionary](DATA_DICTIONARY.md#5-power-query-m-transformation--yield_df).

Two measure-design decisions follow directly from the grain of the source data:

- Pesticide use and rainfall are reported as **averages**, not totals, because both are recorded at country-year grain and repeat across every crop grown in a given country-year. Summing them would overstate the true figure by the number of crops observed for that country-year.
- Yield is never summed across countries, crops, or years, because `hg/ha` is a density (an intensive quantity) rather than a count. The dashboard's scale indicator is **Observations Analyzed** (a genuine row count) rather than a summed yield figure.

## 5. Recommendations Before Wider Distribution

- The Power Query source step currently points to an absolute file path on the author's own machine. Before sharing this file with anyone else, or publishing it, parameterize the source folder path (Power Query parameter) or place the CSV in a fixed relative location alongside the `.pbix` — otherwise a refresh on any other machine will fail with a file-not-found error.
- Add a short on-page note near the Executive Overview treemap and the Crop Efficiency Benchmarking chart clarifying that Average Yield is pooled across ten botanically dissimilar crop types. This keeps a viewer from misreading the gap between the pooled KPI and any individual crop's average as an inconsistency.

## 6. Suggested Future Enhancement

A more granular version of this model could add a crop-category calculated column to `Dim_Crop` (e.g., "Roots & Tubers" vs. "Cereals & Legumes") and report average yield per category rather than one pooled figure. This is a genuine enhancement opportunity, not a required fix — the current pooled figure is accurate as computed; a category breakdown would simply make it easier to interpret at a glance.

## 7. Maintenance

- The data source is static and historical — there is no live or scheduled refresh dependency beyond keeping the source CSV path valid.
- If the source data is ever replaced or extended, re-check that the non-temperature columns remain constant within each `Area`–`Item`–`Year` group before trusting the `Table.Group` step's max aggregation — that assumption holds for the current dataset and should be re-verified against any new one.
