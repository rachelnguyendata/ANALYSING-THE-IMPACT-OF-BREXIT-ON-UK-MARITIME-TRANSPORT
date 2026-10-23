# Analysing the Impact of Brexit on UK Seaport Operations

Author: Thi Thanh Ngan (Rachel) Nguyen

Date: September 2026

Project type: MSc Business Intelligence and Analytics dissertation, University of Huddersfield

Tools Used: Python (pandas, statsmodels, Prophet, scikit-learn) · Microsoft Power BI · Jupyter Notebook

## Project Overview

This project examines how the United Kingdom's departure from the European Union Single Market and Customs Union on 1 January 2021 affected freight moving through United Kingdom seaports. It compares roll-on roll-off (Ro-Ro) and lift-on lift-off (Lo-Lo) cargo, which together account for 38 per cent of national tonnage, and forecasts total freight volumes to 2026. The analysis combines two strands: a difference-in-differences design that explains what has already happened, and time series forecasting that projects what is likely to follow.

## ⚡ Key Findings at a Glance

| # | Finding | Why it matters |
|---|---------|----------------|
| 1 | **UK port freight fell 11%**, from 472 million tonnes (2019) to 420 million tonnes (2025), and has not recovered since COVID. | The fall looks permanent, not temporary, so ports need to plan for the lower level. |
| 2 | **Brexit cut Ro-Ro freight (lorries and trailers) by 21.7% more than Lo-Lo freight (containers)**, after removing the effect of COVID. The result is statistically significant (p < 0.001). | Ro-Ro traffic depends on fast border crossings with the EU, so it was hit hardest by the new customs checks. |
| 3 | **The freight did not disappear. It moved to other ports.** Dover lost 26% of its Ro-Ro traffic (about 6.7 million tonnes a year), while Irish Sea ports grew: Cairnryan and Larne +35%, Belfast +18%. | Some ports now have spare capacity, while others risk congestion. |
| 4 | **The Prophet forecasting model was twice as accurate as ARIMA** (3.3% vs 7.2% average error) when tested on real 2024–2025 data. | It gives port operators and hauliers a reliable planning tool. |
| 5 | **2026 forecast: stable at about 103–105 million tonnes per quarter** (≈416 million tonnes for the year). | Plan for stable demand, not a return to pre-Brexit volumes. |

## 📑 Table of Contents

1. [📌 Background & Overview](#background)
2. [📂 Dataset Description & Data Structure](#dataset)
3. [🧭 Approach & Methodology](#approach)
4. [🧹 Data Cleaning & Preparation](#cleaning)
5. [🔍 Analysis & Results](#analysis)
6. [📊 Visualisation & Insights (Power BI)](#visualisation)
7. [💡 Recommendations](#recommendations)
8. [🚧 Limitations & Next Steps](#limitations)
9. [🗂️ Repository Structure](#structure)
10. [👋 About Me](#about)

---

<a id="background"></a>
## 1. 📌 Background & Overview

### The business problem

More than 80% of world trade by volume travels by sea, and the UK depends on its seaports for almost all goods traded with Europe. When the UK left the EU Single Market and Customs Union on **31 December 2020**, lorries crossing to Europe suddenly faced customs declarations, border checks and extra paperwork.

Everyone expected UK ports to be affected. But **no one had measured what actually happened inside the UK port system**: which ports lost freight, which gained it, and what volumes to expect next. Earlier studies only predicted or simulated the impact, or looked at it from the EU side.

### Objective

Use public government data to:

- **Measure** how freight volumes at UK seaports changed after Brexit.
- **Separate** the effect of Brexit from the effect of COVID-19.
- **Compare** the two main ways cargo is shipped (Ro-Ro and Lo-Lo, explained below) and find out which ports lost or gained traffic.
- **Forecast** freight volumes for 2026.
- **Build a Power BI dashboard** that decision makers can reuse to track the situation.

### 👤 Who is this project for?

✔️ **Port operators**: where to cut or add capacity

✔️ **Hauliers and shipping companies**: how routes have shifted and what volume to plan for

✔️ **Policymakers and port authorities**: where to invest in border infrastructure

### ❓ Research Questions

1. How have total freight volumes at UK seaports changed since Brexit?
2. Did Brexit affect Ro-Ro freight differently from Lo-Lo freight, and at which ports?
3. Which forecasting technique is most accurate for UK port freight?
4. What freight volume should UK ports expect in 2026?

### 📖 Key terms 

| Term | Meaning |
|------|---------|
| **Ro-Ro** (Roll-on/Roll-off) | Cargo **driven** on and off the ship on wheels, such as lorries and trailers. It is fast and flexible, and it is the main way goods cross the Channel (e.g. Dover–Calais). |
| **Lo-Lo** (Lift-on/Lift-off) | Cargo **lifted** on and off by cranes, usually shipping containers. It is slower and mostly connects to long-distance routes. |
| **Difference-in-Differences (DiD)** | A method that compares a group affected by an event (Ro-Ro) with a similar, less-affected group (Lo-Lo), before and after the event. Anything that hit both groups equally, like the economy, cancels out. The extra change left over is the effect of the event. |
| **Forecasting model** | A model that learns patterns from past data, such as trends and seasons, to predict future values. |

---

<a id="dataset"></a>
## 2. 📂 Dataset Description & Data Structure

### 📌 Data Source

- **Source:** UK Department for Transport (DfT), *Port Freight Statistics*. These are accredited official statistics.
- **Licence:** Open Government Licence (publicly available, free to reuse with attribution)
- **Period covered:** 2009–2025 (quarterly data runs to a provisional Q1 2026)
- **Coverage:** All major UK ports in the DfT statistics (53 ports in the quarterly series)
- **Size:** 53,594 raw rows across 3 tables, about **34,842 rows after cleaning**
- **Format:** `.ods` spreadsheets

### 1️⃣ Tables Used

| Table | Content | Years | Rows | Used for |
|-------|---------|-------|------|----------|
| **PORT0201** | National freight by cargo type, international vs domestic | 2000–2025 | 2,725 | National Brexit effect on cross-border trade |
| **PORT0301** | Freight by port, cargo group and direction (inwards/outwards) | 2000–2025 | 28,513 | Port-level analysis (main source) |
| **PORT0502** | Quarterly tonnage by port | 2009–2026 | 22,356 | Forecasting |

<details>
<summary>📋 <b>Data quality issues found in the raw files</b> (click to expand)</summary>

| Issue | Example | Risk if not fixed |
|-------|---------|-------------------|
| Notes and title rows at the top of each sheet | Description text above the real header | Wrong column names, failed imports |
| Non-numeric symbols in number columns | `Z` (not applicable), `LOW` (below display unit) | Columns read as text, so calculations fail |
| Totals already included in the rows | "Both directions" = Inwards + Outwards; "All cargo" already contains Ro-Ro + Lo-Lo | **Double counting** |
| Summary row mixed with port rows | "Total at all major ports" in PORT0502 | National total almost **doubled** |

</details>

---

<a id="approach"></a>
## 3. 🧭 Approach & Methodology

The project follows **CRISP-DM**, a standard six-step framework for analytics projects: Business understanding → Data understanding → Data preparation → Modelling → Evaluation → Deployment.

<img width="742" height="1412" alt="research_process" src="https://github.com/user-attachments/assets/aa791a86-d4dd-4f34-8a41-b9f43de30f1a" />

### Why these methods?

| Step | Method chosen | Why this method |
|------|---------------|-----------------|
| Describe the data | Descriptive statistics (mean, median, % change) | Shows the size and direction of change before any modelling |
| Measure the Brexit effect | **Difference-in-Differences regression** with Lo-Lo as the comparison group | A simple before/after comparison mixes Brexit with COVID. DiD isolates the Brexit effect and is the standard method in recent Brexit trade research. |
| Check the result is reliable | **4 robustness checks** (dropping small ports, weighting by size, changing the time window) | Confirms the result doesn't depend on one particular modelling choice |
| Forecast volumes | **Prophet vs Seasonal ARIMA** | Prophet handles trend breaks and seasonality well. ARIMA is the established benchmark. Testing both on the same data shows which is actually better instead of assuming it. |
| Measure accuracy | MAE, RMSE, MAPE on unseen 2024–2025 data | The models are tested on real quarters they never saw during training |
| Communicate results | **Power BI dashboard** with 3 KPIs | Decision makers need a visual, reusable tool, not just a regression table |

### 📈 KPIs tracked in the dashboard

1. **Total freight volume**: is freight going up or down overall?
2. **Freight by mode (Ro-Ro vs Lo-Lo)**: did the two cargo types behave differently?
3. **Port-level change**: where was freight lost, and where did it reappear?

---

<a id="cleaning"></a>
## 4. 🧹 Data Cleaning & Preparation

All cleaning was done in **Python (pandas)** through an Extract, Transform, Load (ETL) process.

| Step | What I did | Why |
|------|------------|-----|
| 1. Extract | Read the 3 `.ods` files with `odfpy` / `openpyxl` and skipped the header and note rows | Get clean tables with the correct column names |
| 2. Fix data types | Converted number columns to numeric. `Z`, `LOW` and "not available" became missing values and those rows were dropped. | Allow calculations and remove unusable values |
| 3. Avoid double counting | Used a **single direction** and a **single cargo group** when calculating totals | "Both directions" and "All cargo" already include the other rows |
| 4. Remove summary rows | Removed "Total at all major ports" from PORT0502, leaving 53 individual ports (≈421m tonnes a year) | This row nearly doubled the national total |
| 5. Filter the period | Kept 2009–2025. Regression uses 2015–2024 and excludes 2020 as the COVID/transition year. | Avoid distortion from the 2008–09 financial crisis and use only final (non-provisional) figures |
| 6. Split for forecasting | Trained on 2009–2023 (60 quarters) and tested on 2024–2025 (8 quarters) | Gives a fair test of the models on data they had never seen |

---

<a id="analysis"></a>
## 5. 🔍 Analysis & Results

### 5.1 Before vs after Brexit: average national tonnage (thousand tonnes)

| Mode | Pre-Brexit (2015–2019) | Post-Brexit (2021–2024) | Change |
|------|-----------------------:|------------------------:|-------:|
| Ro-Ro | 105,824.7 | 99,418.0 | **-6.1%** |
| Lo-Lo | 65,558.9 | 59,745.6 | **-8.9%** |

💬 **What this means:** Both modes fell, but this simple comparison can't separate Brexit from COVID and other shocks. The regression below does that.

### 5.2 Measuring the Brexit effect (Difference-in-Differences)

| | Port level (each port separately) | National level (international traffic) |
|---|---:|---:|
| **Extra fall in Ro-Ro vs Lo-Lo (Brexit effect)** | -1.3% | **-21.7%** |
| p-value | 0.745 (not significant) | **< 0.001 (significant)** |
| COVID-19 effect (controlled for) | -6.7% | -10.6% |
| Observations | 270 | 20 |
| R² | 0.974 | 0.983 |

💬 **What this means:**

- **Nationally, Brexit reduced Ro-Ro freight by about 22% compared with Lo-Lo.** This almost exactly matches an independent study using EU port data (Mac Domhnaill, 2026: -21.1%), so two different datasets give the same answer.
- **At port level the effect is not significant, and that is a finding in itself.** Some ports lost a lot of Ro-Ro and others gained a lot, so on average they cancel out. This shows that Brexit **redistributed** freight between ports (see the maps in Section 6).

<details>
<summary>🔧 <b>Technical detail: model equations and robustness checks</b> (click to expand)</summary>

**Port-level model**

$$\ln T_{it} = \delta\,Brexit_{it} + \beta\,Post_t + \varphi\,Covid_t + \theta\,Trend_t + \alpha_i + \varepsilon_{it}$$

**National model**

$$\ln T_{mt} = \delta\,Brexit_{mt} + \beta\,Post_t + \varphi\,Covid_t + \rho\,RoRo_m + \varepsilon_{mt}$$

- `Brexit` = 1 for Ro-Ro from 2021 onwards. **δ is the Brexit effect.**
- `Post` = 1 from 2021, which captures changes that hit both modes.
- `Covid` = 1 in 2020, which removes the pandemic effect.
- `α` = port-mode fixed effects, so a large port is never compared directly with a small one.
- Standard errors are clustered by port-mode cell.

**Robustness checks (port level)**

| Model | Observations | Brexit effect | p-value |
|-------|------------:|--------------:|--------:|
| P1: only units ≥ 0.5m tonnes/year | 573 | -4.3% | 0.731 |
| P2: only units ≥ 1m tonnes/year | 461 | -0.6% | 0.967 |
| P3: weighted by tonnage (WLS) | 573 | -2.3% | 0.884 |
| P4: 2015–2024 window | 334 | -2.3% | 0.768 |

All four checks give the same conclusion, so the port-level result does not depend on how small ports are treated.

</details>

### 5.3 Forecasting: which model is more accurate?

Both models were trained on 2009–2023 and tested on the 8 real quarters of 2024–2025.

| Model | MAE (thousand tonnes) | RMSE (thousand tonnes) | MAPE (average % error) |
|-------|----:|-----:|-----:|
| **Prophet** ✅ | **3,553.8** | **4,560.2** | **3.33%** |
| ARIMA | 7,581.3 | 8,161.6 | 7.19% |

💬 **What this means:** Prophet's error is **less than half** of ARIMA's on every measure. ARIMA kept under-predicting. In Q2 2025 it forecast 92,727 thousand tonnes against an actual 106,163. Prophet followed the real data closely, and in Q4 2025 it forecast 102,679 against an actual 102,894. **Prophet was selected for the 2026 forecast.**

### 5.4 Forecast for 2026 (Prophet, thousand tonnes)

| Quarter | Q1 | Q2 | Q3 | Q4 |
|---------|---:|---:|---:|---:|
| **Forecast** | 103,489.0 | 104,861.6 | 103,274.3 | 104,533.4 |

💬 **What this means:** Freight is expected to stay **stable at the lower level reached after 2020**, about 416 million tonnes for the year compared with about 420 million in 2025. There is no sign of a return to pre-Brexit volumes.

---

<a id="visualisation"></a>
## 6. 📊 Visualisation & Insights (Power BI)

### 6.1 Total freight volume, 2009–2025 (million tonnes)

<img width="1137" height="632" alt="total_annual_volume" src="https://github.com/user-attachments/assets/da9e3ad9-9289-4542-a29a-c7dced4668d4" />

📌 **Insight:** Volumes held at 470–507m tonnes through the 2010s, then dropped sharply to 429m in 2020. After a short rebound in 2022 (450m), they fell again to **420m in 2025**. Freight did not bounce back after COVID, which points to a **structural change rather than a temporary shock**.

### 6.2 Ro-Ro vs Lo-Lo by year (million tonnes)

<img width="1137" height="638" alt="roro_lolo_by_year" src="https://github.com/user-attachments/assets/51f62940-af67-4f87-94cc-e0da11fc82b2" />

📌 **Insight:** Ro-Ro is always the larger mode. It peaked at **108m tonnes in 2018** and has stayed around **97–102m since Brexit**. Lo-Lo moved less and recovered to 67m in 2025, which suggests that some cargo **switched from lorries to containers**.

### 6.3 Where did freight move? Port-level change, pre-Brexit (2015–19) vs post-Brexit (2021–24)

🔴 Red = Ro-Ro  🔵 Blue = Lo-Lo  (bubble size = size of change)

<img width="1417" height="802" alt="port_map_north" src="https://github.com/user-attachments/assets/b8e6bd9e-bae7-4d4d-a7c3-342bcb8b3cb3" />

<img width="1385" height="802" alt="port_map_south" src="https://github.com/user-attachments/assets/f659291a-b60e-4f9d-93f4-5fa5dc0431da" />

| 📉 Biggest Ro-Ro losers (EU-facing) | Change | 📈 Biggest Ro-Ro winners (Irish Sea and London) | Change |
|---|---:|---|---:|
| Tees & Hartlepool | -40.5% | Cairnryan | +35.0% |
| Hull | -40.4% | Larne | +35.0% |
| Newhaven | -40.3% | London | +20.7% |
| **Dover** (largest Ro-Ro port) | **-26.3%** (≈ -6.7m tonnes/yr) | Belfast | +18.1% |
| Portsmouth | -26.0% | Heysham | +12.9% |

📌 **Insight:** Ro-Ro traffic **moved away from ports facing continental Europe and towards Irish Sea routes**. Shippers are avoiding the new EU border checks on the short Channel crossings. Even ports on the same coast went in opposite directions: Harwich Lo-Lo fell 49.8% while Hull Lo-Lo grew 15.9%.

<details>
<summary>📋 <b>Full port-by-port table</b> (click to expand)</summary>

**Ro-Ro, average annual tonnage (thousand tonnes)**

| Port | Pre (2015–19) | Post (2021–24) | % change |
|------|---:|---:|---:|
| Aberdeen | 364.7 | 370.0 | 1.4 |
| Belfast | 6,883.7 | 8,126.3 | 18.1 |
| Bristol | 1,058.8 | 1,072.9 | 1.3 |
| Cairnryan | 2,739.0 | 3,697.1 | 35.0 |
| Dover | 25,509.1 | 18,797.9 | -26.3 |
| Felixstowe | 3,509.7 | 3,653.8 | 4.1 |
| Fishguard | 391.6 | 270.1 | -31.0 |
| Grimsby & Immingham | 16,425.3 | 16,002.4 | -2.6 |
| Harwich | 3,899.1 | 3,848.1 | -1.3 |
| Heysham | 4,554.1 | 5,141.7 | 12.9 |
| Holyhead | 5,022.6 | 4,224.0 | -15.9 |
| Hull | 2,381.4 | 1,419.1 | -40.4 |
| Larne | 2,739.3 | 3,697.3 | 35.0 |
| Liverpool | 7,704.7 | 7,510.1 | -2.5 |
| Loch Ryan | 2,420.6 | 2,713.3 | 12.1 |
| London | 7,804.2 | 9,420.0 | 20.7 |
| Medway | 519.2 | 424.7 | -18.2 |
| Milford Haven | 897.6 | 713.2 | -20.5 |
| Newhaven | 601.3 | 359.0 | -40.3 |
| Orkney | 212.4 | 175.0 | -17.6 |
| Plymouth | 66.8 | 17.0 | -74.6 |
| Poole | 328.8 | 256.5 | -22.0 |
| Portsmouth | 2,613.8 | 1,935.4 | -26.0 |
| Ramsgate | 8.2 | 2.3 | -71.8 |
| Southampton | 1,794.5 | 1,414.7 | -21.2 |
| Tees & Hartlepool | 2,025.5 | 1,206.0 | -40.5 |
| Tyne | 867.2 | 537.1 | -38.1 |
| Warrenpoint | 2,126.9 | 2,407.7 | 13.2 |

**Lo-Lo, average annual tonnage (thousand tonnes)**

| Port | Pre (2015–19) | Post (2021–24) | % change |
|------|---:|---:|---:|
| Felixstowe | 24,205.1 | 18,615.3 | -23.1 |
| London | 11,979.7 | 14,281.8 | 19.2 |
| Southampton | 9,925.1 | 8,458.2 | -14.8 |
| Liverpool | 5,589.8 | 5,318.9 | -4.9 |
| Tees & Hartlepool | 2,445.5 | 2,361.6 | -3.4 |
| Forth | 2,273.7 | 2,118.3 | -6.8 |
| Grimsby & Immingham | 2,045.4 | 2,146.6 | 5.0 |
| Hull | 1,775.9 | 2,058.5 | 15.9 |
| Belfast | 1,688.6 | 1,750.1 | 3.6 |
| Medway | 930.0 | 484.9 | -47.9 |
| Bristol | 885.7 | 980.0 | 10.7 |
| Clyde | 624.3 | 506.7 | -18.8 |
| Tyne | 404.0 | 295.4 | -26.9 |
| Portsmouth | 285.1 | 194.9 | -31.7 |
| Warrenpoint | 269.3 | 155.7 | -42.2 |
| Dover | 104.1 | 51.2 | -50.8 |
| Aberdeen | 80.1 | 34.8 | -56.5 |
| Harwich | 20.9 | 10.5 | -49.8 |
| Manchester | 1.9 | 0.1 | -92.6 |

</details>

### 6.4 Forecast test: actual vs predicted, 2024–2025 (thousand tonnes)

<img width="1421" height="787" alt="forecast_test_2024_2025_part2" src="https://github.com/user-attachments/assets/837533c3-5a76-402b-b5a8-36286131d1a7" />

📌 **Insight:** The orange line (Prophet) stays close to the actual line (light blue). The dark blue line (ARIMA) drops well below it in 2025.

### 6.5 Forecast for 2026 (thousand tonnes)

<img width="1337" height="745" alt="forecast_2026" src="https://github.com/user-attachments/assets/f0707182-a387-4ef3-9b54-d7f881b4df73" />

📌 **Insight:** The forecast (dark blue) stays flat at **103–105K thousand tonnes per quarter**. The long decline is expected to level off, but at the new lower level.

---

<a id="recommendations"></a>
## 7. 💡 Recommendations

| Who | Insight it is based on | Recommendation |
|-----|------------------------|----------------|
| **Operators of EU-facing ports** (Dover, Portsmouth, Newhaven, Hull) | Ro-Ro fell 26–40% and has not recovered (Findings 2–3) | Don't keep capacity for a cargo stream that is shrinking. **Diversify into other cargo types or new services** to use spare berths and labour. |
| **Operators of Irish Sea ports** (Cairnryan, Larne, Belfast, Heysham) | Ro-Ro grew 13–35% (Finding 3) | **Review terminal space, storage yards and road/rail links now** and expand where needed, before the extra traffic causes congestion. Congestion is the main driver of delay and cost. |
| **Hauliers and shipping companies** | Volume is forecast to be stable in 2026 and Prophet is accurate to about 3% (Findings 4–5) | **Plan fleet, staff and safety stock for stable demand** of about 103–105m tonnes per quarter. This avoids paying for spare capacity and running short at peak times. |
| **Policymakers and port authorities** | Freight moved away from short-sea EU routes because of customs friction (Findings 2–3) | **Prioritise customs facilitation and border infrastructure at the worst-affected EU-facing ports.** Use the Power BI dashboard, built only on free public data, as a low-cost monitoring tool. |

---

<a id="limitations"></a>
## 8. 🚧 Limitations & Next Steps

**Limitations**

- The forecast covers the **53 major ports** in the quarterly series, not every UK port.
- The **2025 figures are provisional** and may be revised by the DfT.
- Both models slightly **under-predicted** actual volumes in the test period, so the 2026 forecast may be a little low.

**Next steps**

- Re-run the models when the final 2025 and 2026 data are published.
- Add the smaller ports and test the models over a longer period.
- Forecast **Ro-Ro and Lo-Lo separately, and for each port**, not just the national total.
- Add route-level data (e.g. UK–Ireland vs UK–France) to show exactly where traffic moved.
- Publish the dashboard to Power BI Service with scheduled refresh, so stakeholders always see the latest figures.

---

<a id="structure"></a>
## 9. 🗂️ Repository Structure

```
├── README.md
├── data/                 # DfT source tables (PORT0201, PORT0301, PORT0502) and cleaned datasets
├── jupyter notebooks/    # Python notebooks: data preparation, regression, forecasting
├── outputs/              # Tables and files exported from Python for Power BI
├── Power BI/             # Power BI dashboard (.pbix)
└── images/               # Charts and maps shown in this README
```

---

<a id="about"></a>
## 10. 👋 About Me

I'm **Thi Thanh Ngan (Rachel) Nguyen**, an MSc Business Intelligence and Analytics graduate from the University of Huddersfield. I enjoy turning messy public and business data into clear answers and dashboards that people can act on, especially in **supply chain, logistics and operations**.

📫 LinkedIn: https://www.linkedin.com/in/ngandata/?isSelfProfile=true

Email: ngan203133@gmail.com 

Portfolio: [github.com/rachelnguyendata](https://github.com/rachelnguyendata)

---

**Note**: This project uses only secondary data published by the United Kingdom Department for Transport under the Open Government Licence. No personal data were collected or processed.
