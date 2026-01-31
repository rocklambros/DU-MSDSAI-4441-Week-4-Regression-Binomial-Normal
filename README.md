# Problem Set 4: Regression, Binomial & Normal Distributions

**Course:** DU MSDS/AI 4441 — Graduate Statistics
**Author:** Rock Lambros
**Format:** R Markdown (`.Rmd`) knitted to Word/PDF/HTML

## Overview

This assignment covers four core statistical topics through applied R programming:

1. **Least-squares regression** on IMF GDP per capita data (2014 vs 2024)
2. **Absolute error minimization** using `optim()` as an alternative to least-squares
3. **Region identification** in mixed country/region datasets
4. **Normal approximation to the binomial distribution** with continuity correction

## Repository Structure

```
.
├── C4441_RockLambros_ps4.Rmd      # Main R Markdown source (13 code blocks)
├── C4441_RockLambros_ps4.docx     # Knitted Word output
├── imf_gdp_prelim.csv             # IMF GDP per capita dataset (607 rows, 71 cols)
├── ps4.docx                       # Original assignment prompt
├── ps4_explanations.md            # Conceptual companion explanations
└── README.md
```

## Data Pipeline

```
imf_gdp_prelim.csv
    │
    ├── Q1: Filter to GDP PPP per capita rows
    │       ├── Part 1: Scatter plot (X2014 vs X2024, alpha=0.2)
    │       ├── Part 2: Least-squares fit via lm() → slope + intercept
    │       ├── Part 3: total.absolute.error() function
    │       ├── Part 4: optim() minimizes absolute error
    │       └── Part 5: Compare LS (blue) vs MAE (red) lines + zoom panels
    │
    ├── Q2: Identify regions (blank currency + 1 row per entity)
    │
    └── Q3: Normal approximation to binomial
            ├── normal.approximation(n, p) — provided function
            ├── Part 1: 3×3 grid {n=5,25,125} × {p=0.1,0.2,0.5}
            └── Part 2: Summary of approximation quality vs n and p
```

## Key Functions

| Function | Location | Purpose |
|---|---|---|
| `total.absolute.error(slope, intercept, data)` | Rmd block 4 | Sum of \|actual - predicted\| for a linear model |
| `tae.for.optim(params)` | Rmd block 5 | Wrapper for `optim()` (takes vector `c(slope, intercept)`) |
| `normal.approximation(n, p)` | Rmd block 9 | Normal CDF bins approximating Binomial(n, p) probabilities |
| `plot.binom.approx(n, p)` | Rmd block 11 | Plots exact vs approximate and returns sum of abs differences |

## Dependencies

- **R** (>= 4.0)
- `knitr` — R Markdown processing
- `tidyverse` — data manipulation (`dplyr`) and visualization (`ggplot2`)

## Build

```r
# From R console
rmarkdown::render("C4441_RockLambros_ps4.Rmd", output_format = "all")

# Single format
rmarkdown::render("C4441_RockLambros_ps4.Rmd", output_format = "word_document")
```

Or open in RStudio and press **Cmd+Shift+K**.

## Data Conventions

- GDP rows filtered by: `SERIES_NAME == "Gross domestic product (GDP), Constant prices, Per capita, purchasing power parity (PPP) international dollar, ICP benchmark 2021"`
- Year columns prefixed with `X` (e.g., `X2014`, `X2024`) due to R's column naming rules
- Regions have blank `PRIMARY_DOMESTIC_CURRENCY`; countries typically have 3 rows, regions have 1
- Data objects use `dat.` prefix; ggplot objects stored as `g` for incremental layering

## Key Findings

- **Least-squares vs absolute error:** The LS line is pulled more toward Macao (a high-leverage outlier), while the absolute-error line better represents the bulk of countries
- **Region detection:** 11 regional aggregates identified using currency + row-count heuristics
- **Normal approximation:** Quality improves with larger n and p closer to 0.5, consistent with the rule of thumb np >= 5 and n(1-p) >= 5

## Source

IMF World Economic Outlook (WEO) database — [data.imf.org](https://data.imf.org/en/datasets/IMF.RES:WEO)
