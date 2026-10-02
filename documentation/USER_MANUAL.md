# User Operating & Maintenance Manual
**Global Crop Yield & Agricultural Input Analytics**

## 1. Report Pages

### Executive Overview

Four headline KPI cards (Observations Analyzed, Average Yield, Yield YoY Growth %, Count of Area), a yield trend line (1990–2013), a top-10-nations bar chart, a crop treemap sized by Average Yield, and a YoY growth-rate variance chart by crop.  
**Filters:** crop dropdown, year-range slider.

### Agronomic & Input Impact

Four KPI cards (Avg Pesticide Use, Yield per Pesticide Tonne, Yield vs. Temp Correlation, Yield vs. Rainfall Correlation), a pesticide-vs-yield scatter chart with a year-by-year play axis, a temperature-vs-yield combo chart, a rainfall-vs-productivity trend line, and an Input Intensity Impact Matrix pivot table.  
**Filters:** location dropdown, crop dropdown, year-range slider.

### Regional & Crop Deep-Dive

Four KPI cards (Top Yielding Crop, Top Producing Nation, Crop Yield Spread Ratio, Active Crop Varieties), a geographic map sized by Average Yield, a Crop Efficiency Benchmarking bar chart, a historical crop trajectory line chart, and a Regional Micro-Analysis Matrix pivot table with drill-down by country.  
**Filters:** year-range slider, crop dropdown, location dropdown.

## 2. How to Use the Filters

- **Crop / location dropdowns:** single- or multi-select. Selecting a subset recalculates every measure on the page, including the correlation and ratio measures — these are context-sensitive, not fixed constants.
- **Year-range slider:** drag either handle to narrow the 1990–2013 window. 2003 has no data in the source — the year-over-year growth measures are built to skip this gap correctly rather than showing a false spike or drop at that boundary.
- **Play axis:** the pesticide-vs-yield scatter chart (Agronomic page) includes a play axis — press the play button to animate the chart year by year.

## 3. Reading the KPIs Correctly

- **Average Yield** is pooled across 10 crop types with very different natural yield scales (roots/tubers vs. cereals). It will not match any single crop's average shown elsewhere on the report — see the [Executive Briefing](EXECUTIVE_BRIEFING.pdf) for the full per-crop breakdown.
- **Avg Pesticide Use** and **Average Rainfall** are country-year figures, not per-crop figures — filtering to a single crop will not change these two measures' values for a given country-year, since the same reading applies across every crop grown there.
- **Correlation values** measure association only. A correlation near zero (e.g., Yield vs. Rainfall: 0.01) means no meaningful linear relationship was found in this dataset, at this level of aggregation — not that rainfall is unimportant to agriculture in general.

## 4. Maintenance

- Data is static and historical (1990–2013) — no scheduled refresh is required for normal use.

**Before sharing this file with anyone else:**  
The Power Query source step points to an absolute path on the original author's machine. Opening this file on a different machine, or refreshing after receiving it from someone else, will fail with a file-not-found error until this is corrected. Fix: convert the file path to a Power Query parameter, or place the source CSV in a fixed relative location alongside the `.pbix` and update the Source step accordingly.

- If the source data is ever updated or replaced, re-check that the non-temperature columns are constant within each `Area`–`Item`–`Year` group before trusting any KPI on a refresh — see [System Architecture](SYSTEM_ARCHITECTURE.md#7-maintenance).
