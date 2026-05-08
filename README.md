# 🏠 Smart Home Bio-Analytics

**Occupancy Detection from Thermodynamic Sensor Fingerprints for Routine Mining and Energy Consumption Forecasting**

> *STAT 5306 Applied Time Series Analysis in Data Analytics — University of Texas at Arlington, May 2026*
> *Authors: Prithwiraj Chatterjee, Fahad Ullah Syed | Advisor: Dr. Mahmoud Ali Jawad*

---

## 📌 Overview

This project presents a novel **bio-analytics pipeline** that infers room-level human presence from raw thermodynamic sensor data — temperature and relative humidity — **without motion sensors, cameras, or any labeled ground-truth data**.

By computing outdoor-corrected rate-of-change signals that isolate the metabolic heat and respiratory moisture occupants inject into indoor air, we generate continuous **occupancy scores (0–1)** for each of 8 monitored rooms in a residential dwelling. These scores power two downstream tasks:

1. **Unsupervised household routine mining** — recovering weekly schedules, wake/sleep times, meal patterns, and weekday vs. weekend behavioral differences
2. **Hourly appliance energy forecasting** — via SARIMAX and Random Forest models augmented with occupancy features

---

## 📊 Results at a Glance

| Model | RMSE (Wh) | MAE (Wh) | MAPE (%) |
|---|---|---|---|
| **Random Forest** ⭐ | **51.183** | **32.179** | **33.346** |
| SARIMAX | 63.704 | 43.220 | 49.132 |
| Seasonal Naïve | 78.570 | 43.127 | 40.714 |

- **4 of the top 6 most important RF features** are derived occupancy scores
- **7 anomalous days** detected out of 137 via z-score energy monitoring
- Occupancy scores collectively **outweigh** outdoor weather variables in predictive power

---

## 🗂️ Repository Structure

```
.
├── final_code_.R                  # Complete R analysis pipeline
├── final_project_report.pdf       # Full academic paper
├── final_project_code.pdf         # Annotated code with outputs
├── SmartHome_BioAnalytics.pptx    # Presentation slides
└── README.md
```

---

## 🧠 Methodology

### The Bio-Analytics Framework (5 Modules)

```
Raw Sensor Streams (T + RH)
        │
        ▼
┌─────────────────────────┐
│  Module 1: EDA          │  Heatmaps, correlations, STL decomposition
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Module 2: Occupancy    │  Outdoor-corrected rate-of-change scoring
│  Score Engineering      │  occ_r,t = 0.4·ΔRH_signal + 0.3·ΔT_signal + 0.3·L_signal
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Module 3: Routine      │  Hour × day-of-week aggregation
│  Mining                 │  (no clustering algorithm required)
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Module 4: Forecasting  │  SARIMAX(2,0,0)(2,0,0)₂₄ + Random Forest (200 trees)
└────────────┬────────────┘
             ▼
┌─────────────────────────┐
│  Module 5: Anomaly      │  Z-score flagging |z| > 2
│  Detection              │
└─────────────────────────┘
```

### Occupancy Score Formula

For each indoor room `r` at each 10-minute timestep `t`:

```
occ(r,t) = clip[0,1] [
    0.4 × min(1, ΔRH_signal / 3)   +   # Humidity: respiratory/bathing moisture
    0.3 × min(1, ΔT_signal  / 0.5) +   # Temperature: metabolic body heat
    0.3 × L_signal                      # Lights: active occupancy proxy
]
```

Where:
- `ΔRH_signal = max(0, ΔRH_room − ΔRH_outdoor)` — outdoor-corrected humidity change
- `ΔT_signal = max(0, ΔT_room − ΔT_outdoor)` — outdoor-corrected temperature change
- `L_signal = lights / max(lights)` — normalized lighting (Kitchen, Living Room, Home Office only)

---

## 📦 Dataset

**UCI Appliances Energy Prediction Dataset**
- **Source:** Candanedo, Feldheim & Derome (2017) — [UCI ML Repository](https://archive.ics.uci.edu/ml/datasets/Appliances+energy+prediction)
- **Period:** January 11 – May 27, 2016
- **Resolution:** 10-minute intervals → aggregated to hourly
- **Observations:** 19,735 raw / 3,266 hourly (after lag engineering)
- **Location:** Low-energy house in Stambruges, Belgium

### Sensor Summary

| Room | Sensor | Mean T (°C) | Mean RH (%) | Included |
|---|---|---|---|---|
| Kitchen | T1 / RH_1 | 21.7 | 40.3 | ✅ |
| Living Room | T2 / RH_2 | 20.3 | 40.4 | ✅ |
| Laundry | T3 / RH_3 | 22.3 | 39.2 | ✅ |
| Home Office | T4 / RH_4 | 20.9 | 39.0 | ✅ |
| Bathroom | T5 / RH_5 | 19.6 | 50.9 | ✅ |
| Garage/North | T6 / RH_6 | 7.9 | 54.6 | ❌ Semi-outdoor |
| Bedroom 1 | T7 / RH_7 | 20.3 | 35.4 | ✅ |
| Bedroom 2 | T8 / RH_8 | 22.0 | 42.9 | ✅ |
| Parents Room | T9 / RH_9 | 19.5 | 41.6 | ✅ |
| **Outside** | T_out / RH_out | **7.4** | **79.8** | Reference |

---

## ⚙️ Setup & Usage

### Requirements

```r
install.packages(c(
  "tidyverse",
  "lubridate",
  "forecast",
  "randomForest",
  "Metrics",
  "corrplot",
  "gridExtra",
  "reshape2"
))
```

### Running the Pipeline

1. Download the dataset from the [UCI ML Repository](https://archive.ics.uci.edu/ml/datasets/Appliances+energy+prediction) and save as `energydata_complete.csv`

2. Update the file path in `final_code_.R`:
```r
df <- read_csv("path/to/energydata_complete.csv", show_col_types = FALSE)
```

3. Run the script end-to-end:
```bash
Rscript final_code_.R
```

The script executes all 5 modules sequentially and generates 20 plots.

---

## 📈 Key Visualizations

| Plot | Description |
|---|---|
| Indoor temperature (2 weeks) | Overnight drops reveal unoccupied rooms |
| Kitchen & Bedroom heatmaps | Hour-by-date temperature grids show occupancy signatures |
| Bathroom humidity heatmap | Shower/bath spikes as direct human-presence signals |
| Correlation matrix | All indoor sensors + weather (21×21) |
| Occupancy scores (7 days) | Per-room 0–1 score facets |
| Occupancy matrix (3 weeks) | All rooms × time as filled heatmap |
| Routine heatmap | Hour × day-of-week reveals weekly schedule |
| Weekday vs weekend profiles | Activity onset, peaks, sleep patterns |
| Energy vs occupancy scatter | LOESS-smoothed relationship |
| Anomaly detection | Daily z-score flagging |
| STL decomposition | Seasonal, trend, remainder components |
| ACF / PACF | Confirms lag-24 daily seasonality |
| All models vs actual | Full test period overlay |
| 7-day zoom | Intra-day forecast comparison |
| Residuals over time | SARIMAX vs RF bias analysis |
| Actual vs predicted scatter | Per-model clustering around diagonal |
| Model comparison bars | RMSE, MAE, MAPE side by side |
| RF feature importance | Occupancy vs lag vs context features |

---

## 🔑 Key Findings

**Occupancy Detection**
- Bathroom humidity spikes (reaching 80–90% RH) at 07:00–09:00 and 18:00–21:00 provide the clearest human-presence signal in the dataset
- Nocturnal warming in bedrooms — invisible to motion sensors — is detectable via temperature rate-of-change
- Outdoor correction is critical: without subtracting `ΔRH_outdoor`, rainy days produce false-positive occupancy readings

**Routine Mining**
- Activity is lowest 00:00–06:00 across all days; peaks around 10:00–12:00 and 18:00–20:00
- Weekday mornings rise sharply at 08:00; weekend mornings are delayed ~1 hour (sleep-in effect)
- Saturday shows the highest overall household occupancy

**Forecasting**
- RF achieves **35% RMSE reduction** over the seasonal naïve baseline and **20% over SARIMAX**
- The SARIMAX model selected by `auto.arima()`: **ARIMA(2,0,0)(2,0,0)[24]**
- Using `d=1` (differencing) caused ~150% MAPE; reverting to `d=0` with `lag24` as exogenous regressor fixed this
- Individual room occupancy scores (Kitchen `occ_1`, Bedrooms `occ_7`, `occ_8`) outperform the house-level aggregate alone

---

## 📚 References

1. Candanedo et al. (2017) — *Data driven prediction models of energy use of appliances in a low-energy house*, Energy and Buildings
2. Hyndman & Athanasopoulos (2021) — *Forecasting: Principles and Practice*, OTexts
3. Breiman (2001) — *Random Forests*, Machine Learning
4. Cleveland et al. (1990) — *STL: A Seasonal-Trend Decomposition Procedure Based on Loess*, Journal of Official Statistics
5. Box et al. (2015) — *Time Series Analysis: Forecasting and Control*, Wiley

---

## 📄 License

This project was developed for academic purposes at the University of Texas at Arlington. Dataset credits to Candanedo, Feldheim & Derome (2017) via UCI ML Repository.
