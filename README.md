# Global Crop Yield & Agricultural Input Analytics

A Power BI investigation into crop yield patterns across 101 countries and 10 crops (1990–2013), examining how yield density relates to pesticide use, rainfall, and temperature.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-217346?style=flat)
![Power Query](https://img.shields.io/badge/Power%20Query-4d4d4d?style=flat)

---

## At a Glance

| | |
| --- | --- |
| **Scope** | 101 countries · 10 crops · 1990–2013 · 13,130 country-crop-year records |
| **Type** | Descriptive & diagnostic analytics (no predictive model) |
| **Tools** | Power Query (M), Power BI data modeling, DAX |
| **Pages** | Executive Overview · Agronomic & Input Impact · Regional & Crop Deep-Dive |

## Screenshots

| Executive Overview | Agronomic & Input Impact |
| --- | --- |
| ![Executive Overview](images/01-executive-overview.png) | ![Agronomic & Input Impact](images/02-agronomic-input-impact.png) |

![Regional & Crop Deep-Dive](images/03-regional-crop-deep-dive.png)

*(Screenshots pending — see [`images/README.md`](images/README.md).)*

## What This Project Answers

- How has global crop yield density changed between 1990 and 2013?
- Which countries and crops achieve the highest yield density, and how much do they differ?
- How strongly are yield, temperature, rainfall, and pesticide use associated with each other?
- Where is a country or crop under- or over-performing relative to its peers?

This is diagnostic and descriptive work — it explains what the historical data shows and how strongly factors are associated. It does not forecast future yields; no regression, classification, or train/test methodology is used anywhere in the model.

## Key Findings

| Metric | Value |
| --- | --- |
| Average yield (pooled, all crops) | 70,969 hg/ha |
| Year-over-year yield growth | 1.23% average, 1990–2013 |
| Top-yielding crop | Potatoes (182,602 hg/ha) |
| Top-producing nation (by yield) | United Kingdom |
| Yield vs. temperature correlation | -0.14 (weak) |
| Yield vs. rainfall correlation | 0.01 (negligible) |
| Average pesticide use | 14,838.70 tonnes per country-year |

Full findings and interpretation: [`documentation/EXECUTIVE_BRIEFING.pdf`](documentation/EXECUTIVE_BRIEFING.pdf).

**A methodology note worth surfacing:** yield density (`hg/ha`) is an intensive quantity, and pesticide/rainfall figures are recorded at country-year grain rather than per-crop — both facts shape how several measures in this model are deliberately built (see [`documentation/DATA_DICTIONARY.md`](documentation/DATA_DICTIONARY.md#7-interpreting-the-model--notes-for-any-user)). Getting this right matters: a naive `SUM` over either of these fields produces numbers that are off by one to two orders of magnitude.

## Data Model

Star schema — one fact table (`yield_df`, 13,130 rows) and three dimensions (`Dim_Crop`, `Dim_Location`, `Dim_Date`), with 15 DAX measures spanning aggregation, ratio, gap-aware time intelligence, and Pearson correlation. Full detail: [`documentation/DATA_DICTIONARY.md`](documentation/DATA_DICTIONARY.md).

## Repository Structure

```
├── dashboard/
│   └── README.md            (add your .pbix here)
├── data/
│   ├── yield_df.csv          ← used by the dashboard
│   ├── yield.csv              raw source file, reference only
│   ├── rainfall.csv           raw source file, reference only
│   ├── temp.csv                raw source file, reference only
│   ├── pesticides.csv         raw source file, reference only
│   └── README.md
├── documentation/
│   ├── DATA_DICTIONARY.md
│   ├── SYSTEM_ARCHITECTURE.md
│   ├── EXECUTIVE_BRIEFING.pdf
│   ├── USER_MANUAL.md
│   └── docx/                 (Word originals of all four documents)
├── images/
│   └── README.md
└── README.md
```

## Documentation

| Document | Purpose |
| --- | --- |
| [Data Dictionary & Field Catalog](documentation/DATA_DICTIONARY.md) | Every table, column, relationship, and DAX measure |
| [System Architecture](documentation/SYSTEM_ARCHITECTURE.md) | Data flow, data preparation, and technical notes |
| [Executive Briefing](documentation/EXECUTIVE_BRIEFING.pdf) | Headline findings, business-facing summary |
| [User Manual](documentation/USER_MANUAL.md) | How to navigate and correctly read the dashboard |

## Running This Locally

1. Open the `.pbix` in [`dashboard/`](dashboard/) in Power BI Desktop.
2. In Power Query (Transform Data), update the `yield_df` source step to point to your local copy of [`data/yield_df.csv`](data/yield_df.csv) — the packaged source path is specific to the original build machine.
3. Refresh, then explore the three report pages.

## Skills Demonstrated

Data modeling (star schema design) · Power Query (M) transformation and data consolidation · DAX (aggregation, ratio, gap-aware time intelligence, manually-derived Pearson correlation) · data quality auditing (grain analysis, intensive-vs-additive quantity handling) · dashboard UX across a three-page narrative · technical and business documentation.

## License

The dashboard, DAX measures, and documentation in this repository are shared for portfolio purposes. The underlying dataset originates from the [Crop Yield Prediction Dataset on Kaggle](https://www.kaggle.com/datasets/patelris/crop-yield-prediction-dataset) — see [`data/README.md`](data/README.md) for its provenance and license status.

## Author

**John O. Ezekiel** — Agricultural & Environmental Engineer | Data Analyst
[LinkedIn](https://www.linkedin.com/in/johnoezekiel) · [GitHub](https://github.com/johnoe0710) · johnoe.office@gmail.com
