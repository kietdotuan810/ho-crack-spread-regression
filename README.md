# Heating Oil Crack Spread: Does East Coast Inventory Drive Refining Margins?

**Author:** Tuan Kiet Do · **Tools:** Python, pandas, statsmodels, EIA API, Yahoo Finance · **Date:** July 2026

This project tests whether US distillate inventory data from the Energy Information Administration (EIA) predicts the NYMEX heating oil (HO) crack spread, the margin heating oil earns over crude oil. The crack spread is the core margin metric for fuel distributors, so a reliable fundamental driver is directly useful for timing hedges.

![Regression dashboard](results/ho_regression_analysis.png)

## Key findings

**1. Geography is the decisive modelling choice.** National distillate inventory showed almost no relationship with the crack spread (r = −0.03). NYMEX HO is priced at New York Harbor, so the relevant supply is East Coast (PADD1) inventory. Switching to PADD1 raised the correlation to **r = −0.67**.

**2. Inventory leads price, not the other way round.** Granger causality tests show PADD1 inventory deviation predicts the crack spread, while the crack spread does not predict inventory:

| Direction | Levels p-value | First differences p-value |
|---|---|---|
| PADD1 deviation → crack spread | 0.0000 | 0.0022 |
| Crack spread → PADD1 deviation | 0.587 | 0.629 |

**3. The relationship survives every robustness check.**

| Model (2023–2025, 156 weeks) | PADD1 coefficient | Diagnostics |
|---|---|---|
| First differences (primary, causal) | −0.0054 (p < 0.001) | Durbin-Watson 2.23, VIF 1.02 |
| Levels (fair value) | −0.0092 (p < 0.001) | R² 61.7%, VIF 1.65 |
| Levels + lagged crack (momentum control) | −0.0021 (p < 0.001) | Durbin-Watson 1.99 |

The levels coefficient is stable across regimes: −0.0092 on 2023–2025 versus −0.0078 when the 2022 energy crisis year is included.

**In practical terms:** a 100 Mbbl week-on-week increase in the PADD1 inventory deficit is associated with a **$0.54/bbl wider crack spread** the following week.

## Methodology

**Data**
- NYMEX HO and WTI front-month futures (Yahoo Finance), resampled to Wednesday closes to match the EIA release schedule
- EIA weekly PADD1 distillate stocks, series `PET.WDIST1A1.W`
- 5-year seasonal baseline (2017–2021 average by ISO week) to measure how tight inventory is versus normal for the time of year

**Crack spread:** `HO ($/gal) × 42 − WTI ($/bbl)`, in $/bbl

**Samples:** 2023–2025 as the primary sample, excluding the COVID period and the 2026 Iran conflict spike. 2022–2025 is used as a stability check, with an energy-crisis dummy and a natural gas interaction term.

**Model development: four iterations**

| Stage | Change | Why |
|---|---|---|
| 1 | Baseline: 10 variables, national inventory | R² of 90% was almost entirely momentum (lagged crack); inventory not significant (p = 0.92) |
| 2 | National → PADD1 East Coast inventory | Correct geography for New York Harbor pricing; correlation −0.03 → −0.67 |
| 3 | Dropped redundant inventory measure, DXY and natural gas | VIF fell from 203 to 1.65 |
| 4 | First differences as primary model | Augmented Dickey-Fuller tests showed unit roots; Durbin-Watson improved from 0.36 to 2.23 |

**Statistical tests:** Augmented Dickey-Fuller, Engle-Granger cointegration, Granger causality, Breusch-Pagan, variance inflation factors, Durbin-Watson, Newey-West HAC standard errors (4 lags), 52-week rolling R².

## Limitations

- Rolling R² averages about 0.60 but drops to about 0.35 in early 2025, and the model overestimated the crack through most of 2024. Factors outside inventory and seasonality (likely refinery utilisation) dominated in those periods.
- Seasonal demand is proxied by a calendar index rather than actual heating degree days.
- 156 weekly observations is enough for this specification, but too few to test non-linear threshold effects.

**Next steps:** walk-forward out-of-sample validation, heating degree days for the Northeast, East Coast refinery utilisation, and an error correction model on a longer sample.

## How to run

```bash
pip install -r requirements.txt
export EIA_API_KEY=your_key_here      # free key: https://www.eia.gov/opendata/register.php
python ho_regression.py
```

The script prints the full test output and saves the dashboard to `results/ho_regression_analysis.png`.

## Repository contents

```
├── README.md
├── ho_regression.py                  # data pipeline, models, tests, charts
├── requirements.txt
├── results/ho_regression_analysis.png
└── report/HO_Crack_Spread_Regression.pdf   # full write-up
```

## Notes

Uses public data only. Code was developed with AI-assisted tools; model design, iteration decisions and interpretation are the author's. This is research, not investment advice.
