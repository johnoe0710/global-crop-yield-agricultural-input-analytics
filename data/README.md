# Data

Five files from the source dataset, as published. Only the first is actually read by the dashboard's Power Query layer — the other four are the underlying files it was pre-merged from, kept here for full transparency and reproducibility.

## `yield_df.csv` — used by the dashboard

28,242 rows. The pre-merged file the Power Query layer reads (`Csv.Document` source step), later consolidated by the dashboard's ETL to 13,130 distinct country-crop-year records — see the [Data Dictionary](../documentation/DATA_DICTIONARY.md) for the exact transformation.

| Column | Description |
| --- | --- |
| `Area` | Country name |
| `Item` | Crop name (10 crops) |
| `Year` | 1990–2013 (2003 absent) |
| `hg/ha_yield` | Yield density, hectograms per hectare |
| `average_rain_fall_mm_per_year` | Country-year annual rainfall |
| `pesticides_tonnes` | Country-year pesticide use |
| `avg_temp` | Country-year average temperature |

## Underlying source files — reference only, not read by the dashboard

| File | Rows | Columns | Description |
| --- | --- | --- | --- |
| `yield.csv` | 56,717 | `Domain Code`, `Domain`, `Area Code`, `Area`, `Element Code`, `Element`, `Item Code`, `Item`, `Year Code`, `Year`, `Unit`, `Value` | Raw FAO yield records across all crops and countries (broader than the 10 crops / 101 countries used in `yield_df.csv`) |
| `rainfall.csv` | 6,727 | `Area`, `Year`, `average_rain_fall_mm_per_year` | Raw World Bank rainfall-by-country-year records |
| `temp.csv` | 71,311 | `year`, `country`, `avg_temp` | Raw temperature-by-country-year records (this is the file whose multiple readings per country-year caused the duplication in `yield_df.csv` — see the Data Dictionary) |
| `pesticides.csv` | 4,349 | `Domain`, `Area`, `Element`, `Item`, `Year`, `Unit`, `Value` | Raw FAO pesticide-use-by-country-year records |

These four files are not joined or read anywhere in this project's own Power Query — `yield_df.csv` arrives already merged from the original dataset author. They're included here so the full lineage of the data is inspectable, not just the final merged output.

## Source

FAO and World Bank agricultural statistics, 1990–2013, published as the [**Crop Yield Prediction Dataset**](https://www.kaggle.com/datasets/patelris/crop-yield-prediction-dataset) on Kaggle.

## License

Not confirmed. Check the Kaggle listing above for current license terms before reusing this data outside a portfolio context.
