# Performance Evaluation of Energy Forecasts Service — Technical Manual & Service Specification v1.0

**Author:** INESC TEC
**Version:** 1.0
**Last updated:** 21 Dec 2025

---

## 1. Business Context & Definitions

The Performance Evaluation of Energy Forecasts Service is an AI-enabled service that evaluates the quality of energy forecasts by comparing forecasted versus measured energy production of offshore assets (and other energy assets where applicable). It supports energy analysts and system operators in identifying systematic forecasting errors, tracking performance over time and improving future forecasting models.

Energy forecasting is essential for operational planning and market operations. This service addresses the need to evaluate and improve forecast accuracy, helping reduce penalties from imbalance markets and improving grid integration of offshore renewable sources.

### Key Terms

- **Forecast time series:** Predicted energy production values over time (hourly/sub-hourly).
- **Measured/Actual time series:** Observed production values used as ground truth for evaluation.
- **Backtesting:** Historical evaluation where forecasts are compared with actual outcomes over selected intervals.
- **MAE / RMSE:** Standard error metrics quantifying average deviation and squared-error deviation.
- **Forecast bias (%):** Systematic over/under prediction tendency.

### 1.1 PPC (Park Owner Context)

This service is intended for PPC, the owner and operator of a PV portfolio, to strengthen operational planning and portfolio oversight by providing reliable short-term and day-ahead production forecasts with quantified uncertainty. PPC's primary objective is to reduce imbalance exposure and operational risk while maximising portfolio value, which depends on forecasts that are accurate, stable and traceable. Robust forecasting also improves resilience during grid events and severe weather and supports internal reporting and audit needs.

In day-to-day use, PPC relies on forecast outputs to set production expectations, manage risk buffers and decide when operational plans should be updated. The uncertainty information helps PPC anticipate periods of elevated risk and take preventive actions, especially when conditions may drive rapid production changes or operational constraints. Persistent differences between forecasted and measured production are treated as diagnostic signals that can indicate performance degradation, availability limitations or emerging equipment issues.

PPC supplies the necessary site context and operational telemetry and remains responsible for operational decision-making. Data handling expectations include strict time alignment, consistent site identifiers and clear documentation of inputs and assumptions used to generate each forecast.

Operationally, PPC benefits from rolling updates on a fixed cadence, complemented by on-demand refreshes around operational decision points. Each response includes an explicit issue time and sufficient metadata to support reproducibility and audit trails. Performance is tracked using a compact set of indicators covering forecast accuracy, uncertainty calibration, responsiveness and service reliability, alongside business impact measures such as improved imbalance outcomes and better handling of constrained production periods.

---

## 2. Problem Statement

The objective is to deliver a reliable, production-grade service that predicts photovoltaic power production for a portfolio of PV sites over the next 24–48 hours. The service must operate continuously and provide forecasts that are consistent, traceable and suitable for operational use across changing weather and site conditions.

The service shall expose an authenticated interface for requesting forecasts per site and time range, returning time-aligned outputs at a fixed temporal resolution. Forecasts must be generated with a clear issue time, include enough metadata to support auditability and reproducibility and remain within declared physical or operational limits where relevant.

The forecasting approach may combine historical site measurements, operational signals and exogenous drivers such as weather and solar position, provided inputs are available at the time the forecast is issued. The service will be assessed as an operational capability, with emphasis on forecast quality, stability over time and service reliability (availability and response performance).

---

## 3. Data Description

### 3.1 Data Dictionaries

#### 3.1.1 Data Dictionary – Weather Station

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp` | Timestamp | DD/MM/YYYY HH:MM | timestamp field | 21-03-2025 00:15 |
| Battery Voltage Meteohelix | `battery_voltage_meteohelix` | Numeric | V | Battery supply voltage measured by the MeteoHelix at the given timestamp | 3.6 |
| Battery Voltage Meteowind | `battery_voltage_meteowind` | Numeric | V | Battery supply voltage measured by the MeteoWind at the given timestamp | 4.0 |
| Wind Speed Average | `wind_speed_avg` | Numeric | m/s | Wind speed | 2.22 |
| Wind Direction Average | `wind_direction_avg` | Numeric | ° | Average wind direction (0-360) | 90.65 |
| Alarm | `alarm_flag` | Numeric | - | 1 if at least one alarm condition is active, else 0 | 1 |
| Humidity | `humidity` | Numeric | % | Relative humidity | 40 |
| Air Pressure | `air_pressure_avg` | Numeric | Pa | Average air pressure | 101413.64 |
| Irradiance | `irradiance` | Numeric | W/m² | Solar irradiance (global horizontal) | 1.22 |
| Temperature | `temperature` | Numeric | °C | Ambient air temperature | 18.98 |

#### 3.1.2 Data Dictionary – Instrumental Buoy

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp` | Timestamp | DD/MM/YYYY HH:MM | Measurement timestamp (UTC) | 21-03-2025 00:15 |
| Buoy ID | `buoy_id` | String | - | Unique buoy identifier | BUOY_01 |
| Current direction | `current_direction` | Numeric | ° | Current direction. Range: 0–360°. Resolution: 0.01°. Accuracy: ±5° to ±7.5° | 245.32 |
| Sensor tilt | `sensor_tilt` | Numeric | ° | Tilt of the current sensor. Range: 0–35°. Resolution: 0.01°. Accuracy: ±1.5° | 2.15 |
| Current speed | `current_speed` | Numeric | cm/s | Current speed. Range: 0–100 cm/s. Resolution: 0.1 mm/s (=0.01 cm/s). Accuracy: ±0.15 mm/s (=±0.015 cm/s) | 12.34 |
| Water temperature | `water_temperature` | Numeric | °C | Temperature. Range: -5–40 °C. Resolution: 0.01 °C. Accuracy: ±0.1 °C | 16.27 |
| SPL (Broadband) | `spl_broadband` | Numeric | dB re 1 µPa | Sound Pressure Level (broadband) | 85.3 |
| SPL (Deci-decade bands) | `spl_deci_decade_bands` | Numeric | dB re 1 µPa | SPL per deci-decade frequency bands | 70.1 |
| Spectrogram | `spectrogram` | Array / Matrix | dB re 1 µPa (or relative) | Spectrogram representation computed from acoustic signal | [T×F matrix] |
| Cetacean detections | `detection_cetaceans` | Boolean | - | Automatic detections of some cetacean species | True / False |
| Vessel detections | `detection_vessels` | Boolean | - | Automatic detections of vessels | True / False |
| Waves (TBD) | `ocean_waves_tbd` | Text | - | Wave-related parameters | TBD |
| Wind (TBD) | `ocean_wind_tbd` | Text | - | Wind-related parameters | TBD |
| Barometric pressure (TBD) | `ocean_barometric_pressure_tbd` | Text | - | Barometric pressure | TBD |

#### 3.1.3 Data Dictionary – Sentinel

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Geopotential height at 500 mb | `HGT500` | number | m (geopotential meters) | Geopotential height at 500 hPa. | ~5000–6000 m |
| Geopotential height at 850 mb | `HGT850` | number | m (geopotential meters) | Geopotential height at 850 hPa. | ~1300–1700 m |
| Geopotential height (model level 1) | `HGTlev1` | number | m | Geopotential height at model level 1. | model dependent |
| Geopotential height (model level 2) | `HGTlev2` | number | m | Geopotential height at model level 2. | model dependent |
| Geopotential height (model level 3) | `HGTlev3` | number | m | Geopotential height at model level 3. | model dependent |
| Temperature at 500 mb | `T500` | number | K | Air temperature at 500 hPa. | ~230–270 K |
| Temperature at 850 mb | `T850` | number | K | Air temperature at 850 hPa. | ~250–310 K |
| CAPE | `cape` | number | J/kg | Convective available potential energy. | 0–5000 J/kg (e.g. 1200) |
| CIN | `cin` | number | J/kg | Convective inhibition (usually negative). | ≤0 J/kg (e.g. -50) |
| Cloud cover (high) | `cfh` | number | fraction (0–1) | Cloud cover at high levels. | 0.0 clear … 1.0 overcast |
| Cloud cover (low+mid) | `cfl` | number | fraction (0–1) | Cloud cover at low and mid levels. | 0.0–1.0 |
| Cloud cover (mid) | `cft` | number | fraction (0–1) | Cloud cover at mid levels. | 0.0–1.0 |
| Accumulated precipitation | `prec` | number | mm | Total accumulated rainfall between model outputs. | 0–100+ mm per step |
| Convective precipitation | `conv_prec` | number | mm | Total accumulated convective rainfall between model outputs. | 0–100 mm per step |
| Accumulated snowfall (large-scale) | `snow_prec` | number | mm (liquid equivalent) | Total accumulated large-scale snowfall between model outputs. | 0–100 mm per step |
| Water equivalent of accumulated snow depth | `weasd` | number | mm (kg/m²) | Water equivalent of accumulated snow on the ground. | 0–500 mm |
| Snow level | `snowlevel` | number | m a.s.l. | Approximate altitude of the snow level. | 0–3000 m+ depending on situation |
| Downwelling shortwave flux (surface) | `swflx` | number | W/m² | Surface downwelling shortwave radiation. | 0–1000 W/m² (e.g. 742) |
| Downwelling longwave flux (surface) | `lwflx` | number | W/m² | Surface downwelling longwave radiation. | 100–450 W/m² |
| Latent heat flux (downward) | `lhflx` | number | W/m² | Surface downward latent heat flux. | ~-50–400 W/m² |
| Sensible heat flux (downward) | `shflx` | number | W/m² | Surface downward sensible heat flux. | ~-50–400 W/m² |
| Air temperature at 2 m | `temp` | number | K | Air temperature at 2 meters above ground. | ~250–320 K (e.g. 290.7) |
| Relative humidity at 2 m | `rh` | number | fraction (0–1) | Relative humidity at 2 meters. | 0.0–1.0 (e.g. 0.82) |
| Zonal wind at 10 m | `u` | number | m/s | East–west wind component at 10 m (positive eastward). | -50–50 m/s |
| Meridional wind at 10 m | `v` | number | m/s | North–south wind component at 10 m (positive northward). | -50–50 m/s |
| Wind speed at 10 m | `mod` | number | m/s | Wind speed magnitude at 10 m. | 0–50 m/s |
| Wind direction at 10 m | `dir` | number | degrees (0–360) | Meteorological wind direction at 10 m. | 0/360=N, 90=E, 180=S, 270=W |
| Wind gust | `wind_gust` | number | m/s | Modelled near-surface wind gust. | 0–50 m/s |
| Sea surface temperature | `sst` | number | K | Sea surface (skin) temperature. | 270–310 K |
| Visibility | `visibility` | number | m | Horizontal visibility at the surface. | 0–50,000 m |
| PBL height | `pbl_height` | number | m AGL | Planetary boundary layer height above ground. | 0–4000 m |
| Topography | `topo` | number | m a.s.l. | Terrain elevation (orography). | 0–4000+ m |
| Land/water mask | `lwm` | integer (categorical) | 0/1 | Indicates land (1) or water (0). | 0=water, 1=land |
| Land use / vegetation type | `land_use` | integer (categorical) | class id | Discrete land cover class used by the source model. | integer codes (e.g. 1,2,3…) |
| Zonal wind (model level 1) | `ulev1` | number | m/s | Zonal wind component at model level 1. | -100–100 m/s |
| Zonal wind (model level 2) | `ulev2` | number | m/s | Zonal wind component at model level 2. | -100–100 m/s |
| Zonal wind (model level 3) | `ulev3` | number | m/s | Zonal wind component at model level 3. | -100–100 m/s |
| Meridional wind (model level 1) | `vlev1` | number | m/s | Meridional wind component at model level 1. | -100–100 m/s |
| Meridional wind (model level 2) | `vlev2` | number | m/s | Meridional wind component at model level 2. | -100–100 m/s |
| Meridional wind (model level 3) | `vlev3` | number | m/s | Meridional wind component at model level 3. | -100–100 m/s |
| Mean sea level pressure | `mslp` | number | hPa | Sea-level reduced pressure. | 950–1050 hPa |

#### 3.1.4 Data Dictionary – SCADA

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp_utc` | Timestamp | YYYY-MM-DD HH:MM | Logging time of the measurement | 18/3/2025 14:00 |
| average active power delivered | `turbine_activepower_avg` | Numeric | kW | Average active power delivered to the grid in the logging | 0-6000 (ex: 2450) |
| Average rotor rotational speed | `rotor_rpm_avg` | Numeric | RPM | Average rotor rotational speed in the logging interval | 0-20 (ex: 14.9) |

> **Status:** SCADA data dictionary is under consolidation with the data provider. The table above defines the minimum expected fields required to support training, inference and evaluation.

#### 3.1.5 Data Dictionary – Wave Energy Converter

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp_utc` | Timestamp | YYYY-MM-DD HH:MM | Measurement timestamp (UTC) | 2026-02-01 12:00 |
| Plant ID | `plant_id` | Text | - | Unique plant identifier | WAVE_PLANT_01 |
| WEC ID | `wec_id` | Text | - | Device identifier | WEC_03 |
| Operational mode | `op_mode` | Text | - | Device operating mode | Normal/Derated/Safe |
| PTO status (optional) | `pto_status` | Text | - | Power take-off status | On/Off/Fault |
| Control setpoint (optional) | `control_setpoint` | Numeric | - | Control setpoint (if available) | TBD |

> **Status:** WEC device-level telemetry is expected to be available but the variable list is not final. The table above specifies the minimum expected fields to support operational regime identification and feature engineering.

---

## 4. Analytics, Scope & Update Frequency

- **Temporal scope:** Forecasts cover the next 24–48 hours ahead of the forecast issue time, with outputs aligned to a fixed UTC time grid (15 min).
- **Update frequency:** Forecasts are generated on demand via API requests.
- **Output format:** For each site, the service returns a structured forecast package including:
  - **Issue time & validity window:** Timestamp (UTC) when the forecast is generated, plus the start/end times the forecast covers (e.g. next 24–48h).
  - **Time-indexed site forecast:** A site-level time series of PV production values at the agreed step over the validity window.
  - **Traceability metadata:** Minimal identifiers (e.g. forecast/run ID and model version) so the forecast can be tracked in logs and reproduced if needed.

---

## 5. Evaluation Protocols & Metrics

The purpose of the evaluation is to verify that the service operates reliably and delivers consistent, accurate and operationally usable PV production forecasts for a site. Evaluation focuses on compliance with the agreed metrics, service availability and delivery of the specified forecast outputs for each request.

### 5.1 Data Usage & Forecasting Protocol

- The service shall generate forecasts on demand, for horizons up to 24–48 hours ahead of the stated issue time (UTC), aligned to the agreed time step.
- Only information available at or before the issue time may be used (e.g. telemetry up to that time and weather inputs available at that time).
- Forecast outputs shall be time-aligned and include a unique identifier and minimal traceability fields (e.g. run/forecast ID, model version).
- The service shall behave consistently under repeated requests for the same issue time and configuration.

### 5.2 Data Gaps and Exceptions

- Time periods with missing, invalid or flagged measurements in actual production shall be excluded from forecast-quality scoring.
- If a site has insufficient recent data to produce a forecast, the service shall return a clear, structured error (or documented fallback behaviour) and such events shall be counted under service reliability KPIs.

### 5.3 Service Evaluation Metrics & KPIs

The service shall be evaluated using the following quantitative metrics:

- **MAE:** Mean Absolute Error of forecasted PV production versus measured net AC production, aggregated across sites and forecast steps.
- **RMSE:** Root Mean Squared Error of forecasted PV production versus measured net AC production, aggregated across sites and forecast steps.
- **Availability:** Percentage of forecast requests that return a valid response within the agreed service levels.

---

## 6. Deliverables & Submissions

The selected provider shall deliver three reports, aligned with the lifecycle of the service, together with the required technical specifications and deployment artefacts.

### 6.1 Deliverable Reports

1. **Pre-Service Deliverable – Service Design & Setup Report**
   Submitted prior to service start, this report shall describe the proposed analytical approach, forecasting methodology, baseline definition, data requirements, system architecture, security measures and integration plan. It shall also define the operational schedule and procedures for service execution.

2. **Intermediate Deliverable – Interim Performance & Operations Report**
   Submitted at an agreed midpoint of the service period, this report shall summarise service operation to date, data coverage, preliminary analytical results and performance against the defined metrics and KPIs. Any issues, adaptations or refinements to the methodology shall be documented.

3. **Final Deliverable – Final Evaluation & Recommendations Report**
   Submitted at the end of the service period, this report shall present final performance results and site-level insights. The report shall also include guidance for future service continuation or scaling.

### 6.2 Technical Specifications & Submissions

- **Service Interface Documentation:** Full documentation of APIs, data formats, authentication and access controls.
- **Deployment Artefacts:** The provider shall specify the deployment approach. Where containerisation is used, a Dockerfile or equivalent container specification shall be delivered to support reproducible deployment. Alternative deployment models shall be clearly documented.
- **Configuration, Versioning & Handover:** Documentation of configuration parameters, model and data versioning and operational handover procedures.
- **Security & Data Protection Documentation:** Description of data handling, access control and compliance with applicable data protection requirements.
