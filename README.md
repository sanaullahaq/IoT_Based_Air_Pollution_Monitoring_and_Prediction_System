# IoT-Based Air Pollution Monitoring and Prediction System

An end-to-end IoT system for real-time air quality monitoring and future pollutant level forecasting using ARIMA time-series models.

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Sensor Specifications](#sensor-specifications)
- [Cloud Integration (ThingSpeak)](#cloud-integration-thingspeak)
- [Prediction Model (ARIMA)](#prediction-model-arima)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
  - [Hardware Setup](#hardware-setup)
  - [Software Setup](#software-setup)
- [Results](#results)
- [Publication](#publication)
- [Contributors](#contributors)
- [License](#license)

## Overview

This project was developed for **CSE299.9 – Junior Design Course**. It consists of two major components:

1. **Hardware Sensor Node** – An Arduino Mega 2560 with multiple sensors collects temperature, humidity, ammonia (NH₃), carbon monoxide (CO), and PM2.5 data, then transmits it to the ThingSpeak cloud via an ESP-01 Wi-Fi module.
2. **Prediction Engine** – Python-based ARIMA (AutoRegressive Integrated Moving Average) models forecast future pollutant levels using historical data collected from the sensor node.

The system provides a foundation for low-cost, real-time air quality monitoring with predictive capabilities, enabling proactive environmental decision-making.

## System Architecture

```
┌─────────────────────────┐     ┌──────────────┐     ┌─────────────────────┐
│  Sensor Node            │     │  ThingSpeak   │     │  Prediction Model   │
│                         │     │  Cloud API    │     │                     │
│  DHT11 ──────────────┐  │     │              │     │  output.csv ──────┐  │
│  MQ-135 ─────────────┤  │     │  Channel ID  │     │  │                │  │
│  MQ-7 ───────────────┤  ├─────►  & Write Key ├─────►  ▼                │  │
│  Dust Sensor ────────┘  │     │              │     │  Data Cleaning    │  │
│                         │     │  HTTP GET    │     │  → Resampling     │  │
│  Arduino Mega 2560      │     │  Requests    │     │  → Stationarity   │  │
│  + ESP-01 (Wi-Fi)       │     │              │     │  → ARIMA Fit      │  │
└─────────────────────────┘     └──────────────┘     │  → Forecast       │
                                                      └─────────────────────┘
```

## Hardware Components

| Component | Purpose |
|-----------|---------|
| Arduino Mega 2560 | Main microcontroller |
| ESP-01 (ESP8266) | Wi-Fi communication module |
| DHT11 | Temperature and humidity sensor |
| MQ-135 | Ammonia (NH₃) gas sensor |
| MQ-7 | Carbon monoxide (CO) sensor |
| GP2Y1010AU0F (compatible) | Optical dust sensor (PM2.5) |

## Sensor Specifications

| Sensor | Parameter | Unit | Arduino Pin | Conversion Method |
|--------|-----------|------|-------------|-------------------|
| DHT11 | Humidity | % | A0 | Standard DHT library |
| DHT11 | Temperature | °C | A0 | Standard DHT library |
| MQ-135 | NH₃ | µg/m³ | A1 | Logarithmic (R0, slope, intercept) |
| MQ-7 | CO | mg/m³ | A2 | Logarithmic (R0, slope, intercept) |
| Dust Sensor | PM2.5 | µg/m³ | A3 + D12 | PWM voltage-to-density |

## Cloud Integration (ThingSpeak)

Data is uploaded to ThingSpeak via HTTP GET requests through the ESP-01 module.

**ThingSpeak Field Mapping:**

| Field | Parameter |
|-------|-----------|
| field1 | Humidity |
| field2 | Temperature |
| field3 | NH₃ |
| field4 | CO |
| field5 | PM2.5 |

The system uploads data approximately every 10 seconds per parameter with automatic Wi-Fi reconnection and retry logic.

## Prediction Model (ARIMA)

### Workflow

1. **Data Collection** – Sensor readings stored as `output.csv` (~2600 records at 3–4 minute intervals over 7 days).
2. **Resampling** – Data aggregated to 1-hour intervals (mean) for meaningful predictions (~168 hourly records).
3. **Train/Test Split** – First 6 days (144 hours) for training, last 24 hours for validation.
4. **Stationarity Testing** – Augmented Dickey-Fuller (ADF) test applied; non-stationary series (NH₃) transformed via log + differencing.
5. **Model Selection** – ACF/PACF plots used to determine ARIMA orders (p, d, q).
6. **Forecasting** – 24-hour ahead forecast with 95% confidence intervals.
7. **Evaluation** – Mean Absolute Percentage Error (MAPE).

### ARIMA Model Orders

| Parameter | Order (p, d, q) | Model Type |
|-----------|-----------------|------------|
| Humidity | (1, 1, 1) | ARIMA |
| NH₃ | (2, 1, 3) | ARIMA (log-transformed) |
| CO | (1, 0, 2) | ARMA |
| PM2.5 | (3, 0, 1) | ARMA |
| Temperature | (5, 1, 1) | ARIMA |

### Dependencies (Python)

- `pandas` – data manipulation and resampling
- `numpy` – numerical computation
- `matplotlib` / `seaborn` – visualization (correlation heatmaps, time-series plots)
- `statsmodels` – ARIMA modeling, ADF test, ACF/PACF plots

## Repository Structure

```
IoT_Based_Air_Pollution_Monitoring_and_Prediction_System/
├── README.md
├── Codes/
│   ├── Arduino (Hardware Code)/
│   │   └── Final_CODE/
│   │       └── Final_CODE.ino          # Arduino Mega firmware
│   └── Prediction (ARIMA) Model/
│       ├── Finals (Jupyter Notebook).ipynb
│       ├── Finals_(Jupyter_Notebook)_Updated.ipynb  # Updated with embedded plots
│       ├── Finals (Python FIle).py                  # Standalone Python script
│       ├── output.csv                              # Raw sensor dataset
│       └── final_data_with_AQI.xlsx                # Data with computed AQI
└── Report/
    ├── CSE299.9 Group 3 Project Report.docx
    └── CSE299.9 Group 3 Project Report.pdf
```

## Getting Started

### Hardware Setup

1. Connect sensors to the Arduino Mega 2560 as per the pin mapping table above.
2. Configure the ESP-01 module with your Wi-Fi credentials (SSID and password) in `Final_CODE.ino`.
3. Set up a ThingSpeak channel and replace the `apiKey` in the code with your Write API Key.
4. Upload `Final_CODE.ino` to the Arduino Mega using the Arduino IDE.
5. Ensure the DHT sensor library is installed in the Arduino IDE.

### Software Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-org>/IoT_Based_Air_Pollution_Monitoring_and_Prediction_System.git
   cd IoT_Based_Air_Pollution_Monitoring_and_Prediction_System
   ```
2. Install Python dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn statsmodels
   ```
3. Run the prediction model:
   ```bash
   jupyter notebook "Codes/Prediction (ARIMA) Model/Finals (Jupyter Notebook).ipynb"
   ```
   Or execute the standalone script:
   ```bash
   python "Codes/Prediction (ARIMA) Model/Finals (Python FIle).py"
   ```

## Results

The ARIMA models produce 24-hour forecasts for each pollutant/weather parameter with 95% confidence bands. Forecast accuracy is evaluated using MAPE against the held-out test set. Visualization outputs include:

- Time-series plots of actual vs. forecasted values
- ACF and PACF plots for model order selection
- Correlation heatmaps and pairplots of all parameters

## Publication

This project was published at an IEEE conference:

> **An IoT Based Air Pollution Monitoring and Prediction System** – IEEE Xplore, Document 9775871  
> [View Publication](https://ieeexplore.ieee.org/document/9775871)

## Contributors

**CSE299.9 – Group 3**  
Department of Electrical and Computer Engineering  
North South University, Bangladesh

## License

This project is provided for academic and reference purposes. See the report for detailed project documentation.
