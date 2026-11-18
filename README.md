# Marketing Mix Modeling in R: Channel ROI and Budget Optimisation

How much did each marketing channel really return, and how should the next dollar be split? This project builds a marketing mix model (MMM) with **adstock** (carry-over) and **saturation** (diminishing returns) from scratch in R, turns it into **average and marginal ROI**, and uses it to **optimise the budget**. The data are simulated with known channel effects, so every estimate is graded against the truth.

## Question

Spend in four channels (TV, search, social, display) over 156 weeks, plus price and promotions. Which channels drive sales, how fast does advertising decay, where are the diminishing returns, and does a model-recommended reallocation actually pay off under the true response curves?

## Key findings

*(Fill in after running the scripts. Suggested: the holdout MAPE of the MMM versus the linear baselines; the ROI table (true vs MMM vs naive regression), naming the channel the naive regression gets most wrong; the channel with the highest marginal ROI at current spend and the one that is saturated; the recommended reallocation and the share of the best possible gain it captures under the true curves; which bootstrap intervals are too wide to act on.)*

## What is in it

- **Realistic simulated data** with the usual traps: flighted TV, search spend that follows seasonal demand (endogenous), social that overlaps with TV, over-spent and saturated display, price and promotion effects, a duplicated week and missing spend values.
- **Cleaning** with a log of every fix (duplicates, calendar gaps, interpolated spend).
- **Naive baselines**: linear regression on raw spend, with and without controls.
- **The MMM**: geometric adstock plus Hill saturation, fitted by variable projection (least squares for the linear part, multi-start L-BFGS-B for decay and half-saturation), with non-negative channel effects.
- **ROI**: average ROI (contribution / spend) and marginal ROI at current spend, compared with the truth.
- **Budget optimisation**: closed-form water-filling (equal marginal returns), checked against a general numeric optimiser, and judged under the true response curves.
- **Uncertainty**: moving-block residual bootstrap with refitting, giving intervals for ROI, marginal ROI and recommended spend.
- **A Quarto report** with the method, tables, charts and limitations.

## Tools

R, dplyr, tidyr, tibble, ggplot2, knitr, Quarto. Base R `optim` and `uniroot` for the model fit and the optimisation.

## Reproduce

```r
install.packages(c("dplyr", "tidyr", "tibble", "ggplot2", "knitr"))
source("R/00_simulate_data.R")
source("R/01_clean_explore.R")
source("R/02_fit_models.R")
source("R/03_budget_optimisation.R")   # includes a 100-replicate bootstrap, a few minutes
quarto::quarto_render("reports/report.qmd")
```

Run from the project root (open `marketing-mix-modeling.Rproj`). The seed is fixed, so results are reproducible.

## Project structure

```
marketing-mix-modeling/
├── R/
│   ├── mmm_functions.R           # simulator, cleaning, adstock/saturation model, ROI, optimisers
│   ├── 00_simulate_data.R
│   ├── 01_clean_explore.R
│   ├── 02_fit_models.R
│   └── 03_budget_optimisation.R
├── data/
│   ├── raw/                      # weekly_marketing.csv (generated)
│   └── clean/                    # .rds results (generated)
├── reports/report.qmd
├── marketing-mix-modeling.Rproj
└── README.md
```

## Limitations

- Simulated data using the same functional form as the fitted model; real data bring misspecification and unmeasured drivers.
- Search spend is endogenous; real work would calibrate it with lift tests or experiments.
- Three years of weekly data give wide intervals, especially for channels with little spend variation.
- The bootstrap keeps spend fixed and ignores model-choice uncertainty.
- The optimiser stays within 50% to 150% of current spend; it does not extrapolate beyond the observed range.
