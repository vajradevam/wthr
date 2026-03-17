## Cleaning

### Data Cleaning and Preparation Overview

The original dataset consisted of multivariate weather station observations recorded at regular time intervals. However, exploratory analysis revealed significant missingness across several variables, especially those related to air quality measurements. Since the objective of the study is focused on **AQI and temperature prediction**, the preprocessing pipeline was designed to retain meaningful signals while removing structural noise and sensor artifacts.

The first step involved **visualizing missing values over time** using a full time-series heatmap. This allowed us to distinguish between two types of missing data: **structural missingness** caused by sensors not yet being operational, and **periodic missingness** caused by sensors operating at lower sampling frequencies than the main station logger. The heatmap and month-wise missingness analysis clearly showed that **AQI and particulate matter sensors were largely inactive before August 2024**. Prior to this period, the AQI column contained extremely sparse data and therefore did not provide reliable information for predictive modelling. Including this section of the dataset would introduce artificial noise and bias models toward missing patterns rather than meaningful atmospheric relationships. For this reason, the dataset was **trimmed so that the new starting point is August 2024**, the point at which AQI readings became consistently available.

After trimming, another issue became evident: several variables had large numbers of missing values because different sensors operate at **different sampling periodicities**. For example, some sensors record measurements at every timestamp, while others record readings every **third timestamp**. This pattern does not indicate actual absence of information but rather a **lower-frequency sampling schedule**. To address this, the dataset was interpolated using **time-based interpolation**, which estimates missing values based on surrounding timestamps. Time interpolation is particularly suitable for environmental time-series because meteorological variables such as temperature, humidity, pressure, and AQI typically change smoothly over short time intervals. This method reconstructs intermediate values in a physically reasonable manner without introducing abrupt discontinuities. After interpolation, forward and backward filling were applied to handle any remaining edge gaps.

During preprocessing it was also observed that some columns were either **categorical or redundant sensor duplicates** that contributed large amounts of missing values without improving predictive capability. In particular, wind direction variables are categorical and cannot be meaningfully interpolated, while the secondary sensor block (`*_1` variables) represents auxiliary measurements that are sparsely populated. These columns were removed from the dataset to improve data quality and reduce noise in downstream models.

Finally, the cleaned dataset was visualized again using a heatmap where **interpolated values were highlighted**, allowing verification that interpolation primarily filled periodic sampling gaps rather than long outages. The resulting dataset contains continuous time-series observations with minimal missing values and consistent temporal resolution, making it suitable for machine learning models aimed at **air quality and temperature forecasting**.

---

### Remaining Features and Their Relevance

After cleaning, the remaining variables consist primarily of meteorological and air-quality related measurements. Their importance for predicting **AQI** and **temperature** can be ranked based on known atmospheric relationships and empirical evidence from environmental modelling.

---

# Feature Importance for AQI Prediction

**Tier 1 – Direct AQI Drivers (Highest Importance)**

1. `pm_2_5_ug_m` – primary pollutant used in AQI calculation
2. `pm_10_ug_m` – coarse particulate concentration
3. `pm_1_ug_m` – fine particulate concentration
4. `temp_degc` – temperature strongly influences pollutant chemistry
5. `hum_pct` – humidity affects particulate formation and dispersion

**Tier 2 – Atmospheric Conditions Influencing Dispersion**
6. `avg_wind_speed_km_h` – pollutant dispersion
7. `high_wind_speed_km_h`
8. `barometer_mb` – atmospheric pressure systems influence air stagnation
9. `absolute_pressure_mb`
10. `dew_point_degc` – moisture–pollution interaction

**Tier 3 – Secondary Environmental Factors**
11. `solar_rad_w_m_2` – photochemical reactions affecting pollution
12. `uv_index` – photochemical smog formation
13. `rain_mm` – precipitation removes particulates
14. `wet_bulb_degc`

**Tier 4 – Derived Thermal Indices (Lower Importance)**
15. `heat_index_degc`
16. `thw_index_degc`
17. `thsw_index_degc`
18. `wind_chill_degc`

---

# Feature Importance for Temperature Prediction

**Tier 1 – Core Temperature Indicators**

1. `temp_degc`
2. `high_temp_degc`
3. `low_temp_degc`
4. `inside_temp_degc`

**Tier 2 – Moisture and Thermal Balance Variables**
5. `dew_point_degc`
6. `hum_pct`
7. `wet_bulb_degc`

**Tier 3 – Radiation and Energy Balance**
8. `solar_rad_w_m_2`
9. `uv_index`

**Tier 4 – Atmospheric and Wind Influences**
10. `avg_wind_speed_km_h`
11. `barometer_mb`
12. `absolute_pressure_mb`

**Tier 5 – Derived Thermal Indices**
13. `heat_index_degc`
14. `thw_index_degc`
15. `thsw_index_degc`

---

The preprocessing pipeline focused on **removing structural missingness, preserving meaningful temporal relationships, and reconstructing periodic sensor gaps** through time interpolation. Trimming the dataset to August 2024 ensures that AQI measurements are consistently available, while dropping redundant or categorical variables improves dataset quality. The resulting cleaned dataset contains continuous meteorological and air-quality measurements suitable for predictive modelling. Among the remaining variables, particulate concentrations and meteorological dispersion factors are the strongest predictors for AQI, while temperature prediction relies primarily on direct thermal measurements, humidity variables, and solar radiation indicators.


During the preprocessing stage, the temporal integrity of the dataset was verified by analyzing the differences between consecutive timestamps. This was done by computing the time difference between each pair of adjacent observations and examining the frequency distribution of these intervals. The analysis revealed that the majority of timestamps were separated by **15-minute intervals**, confirming that the primary sampling resolution of the dataset is 15 minutes. However, a significant number of **5-minute differences** were also observed. These shorter intervals do not necessarily indicate that the dataset was originally sampled every 5 minutes; rather, they are likely the result of irregular logging patterns, slight timestamp offsets, or missing intermediate records during data collection or merging from multiple sensors. A few rare larger gaps (such as 30 minutes or several hours) were also detected, which likely correspond to brief sensor outages or station downtime.

To ensure a consistent temporal structure for time-series modeling, the dataset was subsequently **resampled to a strict 15-minute time grid**. Resampling aligns all observations to a uniform temporal framework, eliminating irregular timestamp offsets and ensuring that each observation corresponds to a fixed interval in time. Interestingly, the size of the dataset remained largely unchanged after resampling, which indicates that the original data was already predominantly aligned with the 15-minute grid despite the occasional irregular intervals observed in the timestamp difference analysis. Establishing a consistent temporal resolution is essential for downstream forecasting models because it guarantees that lag features, rolling statistics, and other temporal transformations operate on a stable and predictable time axis.
