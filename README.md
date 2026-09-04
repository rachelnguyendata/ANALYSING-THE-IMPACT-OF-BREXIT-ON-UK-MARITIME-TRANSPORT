# Analysing the Impact of Brexit on UK Seaport Operations

The study of Ro-Ro and Lo-Lo freight at United Kingdom seaports, using Python for analysis and Power BI for visualisation.

## Project Overview

This project examines how the United Kingdom's departure from the European Union Single Market and Customs Union on 1 January 2021 affected freight moving through United Kingdom seaports. It compares roll-on roll-off (Ro-Ro) and lift-on lift-off (Lo-Lo) cargo, which together account for 38 per cent of national tonnage, and forecasts total freight volumes to 2026. The analysis combines two strands: a difference-in-differences design that explains what has already happened, and time series forecasting that projects what is likely to follow.

### Key Findings

- **Ro-Ro fell 21.7% relative to Lo-Lo** at national level for international traffic (p < 0.001, R² = 0.983), closely matching the 21.1% reported independently from European Union data by Mac Domhnaill (2025)
- **Port level estimates are not significant** across all four robustness checks, ranging from −0.6% to −4.3%, because losses at ports facing continental Europe offset gains at Irish Sea ports
- **Freight was redistributed, not lost** — Plymouth fell 74.6% and Ramsgate 71.8%, while Cairnryan and Larne each rose 35.0% and Belfast rose 18.1%
- **Prophet outperformed ARIMA** on all three accuracy measures (MAPE 3.33% vs 7.19%, MAE 3,553.8 vs 7,581.3, RMSE 4,560.2 vs 8,161.6)
- **A permanently lower plateau** — the 2026 forecast holds at 103,300 to 104,900 thousand tonnes per quarter, with no return to the volumes of the 2010s

## Research Questions

1. How have total freight volumes at United Kingdom seaports changed since Brexit?
2. To what extent has Brexit affected Ro-Ro freight differently from Lo-Lo freight across United Kingdom ports?
3. What predictive analytics techniques are most suitable for forecasting freight volumes at United Kingdom ports?
4. How can predictive analytics forecast freight volumes at United Kingdom ports?

## Data Sources

All data are secondary and openly published by the United Kingdom Department for Transport.

| Table | Coverage | Used for |
|---|---|---|
| **PORT0301** | Annual, all major ports, by cargo group and direction | Descriptive statistics, port level regression, route diversion, cargo composition |
| **PORT0201** | Annual, national, split into international and domestic | National difference-in-differences on international traffic |
| **PORT0502** | Quarterly, 53 major ports, both directions | Time series forecasting |

The three published Department for Transport tables are overlapping views of the same underlying data, so combining them would produce double counting. Three tables were selected, each serving a distinct purpose. All tonnage figures are in thousand tonnes.

## Methodology

The study follows the CRISP-DM framework within a business intelligence approach.

### Data preparation
- Extract, transform and load from OpenDocument source files
- Cleaning, validation and cross checking between sources
- Classification into Ro-Ro and Lo-Lo by cargo code
- Split into pre-Brexit (2015 to 2019) and post-Brexit (2021 to 2024) periods

### Explanatory analysis
- Descriptive statistics by cargo handling mode and by port
- Difference-in-differences regression on log-transformed tonnage, with Lo-Lo as the comparison group for Ro-Ro
- Specification A at port level with unit fixed effects, Specification B at national level on international traffic
- Four robustness checks: two size thresholds, weighted least squares, and a restricted time window
- Route diversion analysis across ports

### Predictive analysis
- Quarterly total tonnage series constructed from PORT0502
- Trained to 2023, tested against the known outturn for 2024 and 2025
- Prophet compared with ARIMA, with ARIMA parameters selected by grid search on the Akaike Information Criterion
- Evaluated with MAE, RMSE and MAPE
- 2026 forecast produced with the better performing model

## Repository Structure

```
├── README.md                    # Project documentation
├── analysis.ipynb               # Jupyter notebook — full analysis pipeline
├── data/                        # DfT source files (PORT0301, PORT0201, PORT0502)
├── outputs/                     # Exported CSV files for Power BI and figures
└── dashboard.pbix               # Power BI dashboard
```

## Analysis Workflow

The notebook (`analysis.ipynb`) covers the full pipeline:

1. **Data understanding** — load the three Department for Transport tables, inspect structure and coverage
2. **Data preparation** — clean, validate, classify by mode, split into periods
3. **Descriptive analysis** — tonnage by mode and by port, raw pre and post comparison
4. **Regression** — difference-in-differences at port and national level, plus robustness checks
5. **Route diversion** — percentage change by port for each mode, with coordinates for mapping
6. **Forecasting** — Prophet and ARIMA, evaluation on 2024 and 2025, forecast for 2026
7. **Export** — CSV files for the Power BI dashboard

## Key Results

### Regression (Table 4.4)

| | Specification A (port level) | Specification B (national, international) |
|---|---|---|
| Brexit × Ro-Ro | −1.3% | **−21.7%** |
| p-value | 0.745 | < 0.001 |
| Post-Brexit | −8.8% | −6.7% |
| COVID-19 | −6.7% | −10.6% |
| Observations | 270 | 20 |
| R² | 0.974 | 0.983 |

### Robustness checks (Table 4.5)

| Specification | n | Units | Differential | p |
|---|---|---|---|---|
| Threshold 0.5 million tonnes | 573 | 36 | −4.3% | 0.731 |
| Threshold 1 million tonnes | 461 | 29 | −0.6% | 0.967 |
| Weighted least squares | 573 | 36 | −2.3% | 0.884 |
| Window 2015 to 2024 | 334 | 34 | −2.3% | 0.768 |

The consistently insignificant port level estimate is itself informative: it points to offsetting movements between ports rather than a uniform decline.

### Route diversion, Ro-Ro

| Largest declines | | Largest increases | |
|---|---|---|---|
| Plymouth | −74.6% | Cairnryan | +35.0% |
| Ramsgate | −71.8% | Larne | +35.0% |
| Tees & Hartlepool | −40.5% | London | +20.7% |
| Hull | −40.4% | Belfast | +18.1% |
| Newhaven | −40.3% | Warrenpoint | +13.2% |
| Tyne | −38.1% | Heysham | +12.9% |

### Forecast accuracy, test period 2024 to 2025

| Model | MAE | RMSE | MAPE |
|---|---|---|---|
| **Prophet** | **3,553.8** | **4,560.2** | **3.33%** |
| ARIMA | 7,581.3 | 8,161.6 | 7.19% |

ARIMA order: (2,1,2)(1,1,1)[4]. Prophet was selected for the 2026 forecast.

### Cargo composition at major ports, 2024

| Cargo group | Share |
|---|---|
| Liquid bulk | 38.7% |
| Ro-Ro | 23.6% |
| Dry bulk | 19.2% |
| Lo-Lo | 14.4% |
| Other general cargo | 4.0% |

Ro-Ro and Lo-Lo together account for 38.0 per cent of the approximately 421 million tonnes handled at major ports.

## Business Intelligence Dashboard

The Power BI dashboard visualises the outputs exported from Python. It performs no calculation of its own, so that what the dashboard displays is exactly what the analysis produced. The automated forecasting available in Power BI, which uses exponential smoothing, was deliberately not used, since it would not correspond to the Prophet and ARIMA models specified in the analysis.

Five views:
1. Total freight volume over time
2. Ro-Ro and Lo-Lo comparison
3. Variation across ports, mapped
4. Forecast evaluation for 2024 and 2025
5. Forecast for 2026

## Technologies Used

- **Python 3** in Jupyter Notebook (Anaconda distribution) — all data preparation, analysis and modelling
  - pandas, numpy — data handling and numerical operations
  - odfpy, openpyxl — reading OpenDocument source files
  - statsmodels — ordinary and weighted least squares, SARIMAX
  - prophet — time series forecasting
  - scikit-learn — forecast accuracy metrics
- **Microsoft Power BI** — all visualisation, built from the CSV files exported by the notebook
- **Statistical methods**: difference-in-differences regression, weighted least squares, ARIMA, Prophet

The division of labour is deliberate. Python produces every number and writes the results to CSV; Power BI reads those files and presents them. No figure in this project is calculated inside the dashboard.

## How to Run

1. Install dependencies:
   ```bash
   pip install pandas numpy statsmodels prophet scikit-learn odfpy openpyxl
   ```
2. Place the Department for Transport source files in the `data/` folder
3. Open the notebook:
   ```bash
   jupyter notebook analysis.ipynb
   ```
4. Run all cells from a clean kernel using Restart and Run All
5. Exported CSV files are written to `outputs/` and can be loaded into the Power BI dashboard


## Author

Thi Thanh Ngan Nguyen

## Date

August 2026

---

**Note**: This project uses only secondary data published by the United Kingdom Department for Transport under the Open Government Licence. No personal data were collected or processed.
