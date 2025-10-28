#  Time Series Forecasting Report: ARIMA vs Prophet

**Dataset Period:** 2017-11-07 to 2022-06-04  
**Frequency:** Daily averages  
**Forecast Horizon:** 30 days  
**Special Events Considered:** Diwali, New Year  


## 1. Introduction

The goal of this analysis is to forecast daily values using two popular time series models: **ARIMA (AutoRegressive Integrated Moving Average)** and **Facebook Prophet**.  
Both models were trained on the same dataset to evaluate and compare their forecasting accuracy, interpretability, and robustness to seasonal events.


## 2. Data Preprocessing

The dataset contained daily data from **2017-11-07 to 2022-06-04**. Missing values were handled via interpolation, and outliers were smoothed using rolling means.  
The data was split into:

- **Training set:** All data up to 30 days before the end of the series  
- **Testing set:** Final 30 days used as the forecast horizon

| ![exploration1](https://github.com/evanferrao/rlabproject/blob/main/assets/exploration1.png?raw=true)| ![exploration2](https://github.com/evanferrao/rlabproject/blob/main/assets/exploration2.png?raw=true) | ![exploration4](https://github.com/evanferrao/rlabproject/blob/main/assets/exploration4.png?raw=true) |
|:---:|:---:|:---:|
| *PM2.5 Levels over time* | *PM2.5 Levels in november 2017* | *Weekly analysis* |

## 3. Model Implementations

### 3.1 ARIMA Model

ARIMA models the time series based on its own lagged values and past forecast errors. The parameters `(p, d, q)` were selected automatically using **auto.arima()**, optimizing for the lowest AIC (Akaike Information Criterion).

```r
library(forecast)
arima_model <- auto.arima(train_df$y)
arima_forecast <- forecast(arima_model, h = 30)
````

The ARIMA model effectively captured short-term dependencies and trend variations. However, it lacks explicit mechanisms to handle **holiday effects** or **multiple seasonalities**.

<p align="center">
  <img src="https://github.com/evanferrao/rlabproject/blob/main/assets/aforecast1.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using ARIMA"><br>
  <em>Figure: Daily Aggregated AQI Forecast using ARIMA</em>
</p>


### 3.2 Prophet Model

Prophet, developed by Facebook, decomposes time series into **trend**, **seasonality**, and **holiday effects**. It’s designed to handle outliers, missing data, and multiple seasonal patterns (weekly, yearly, etc.).

```r
library(prophet)
prophet_model <- prophet(df = train_df)
future <- make_future_dataframe(prophet_model, periods = 30)
forecast <- predict(prophet_model, future)
```

Holidays such as *Diwali* and *New Year* were incorporated to improve prediction around these special periods.

<p align="center">
  <img src="https://github.com/evanferrao/rlabproject/blob/main/assets/pforecast1.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using Prophet"><br>
  <em>Figure: 7 Day AQI Forecast using Prophet</em>
</p>

<p align="center">
  <img src="https://github.com/evanferrao/rlabproject/blob/main/assets/pforecast2.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using Prophet"><br>
  <em>Figure: Historical Prediction using Prophet</em>
</p>

<p align="center">
  <img src="https://github.com/evanferrao/rlabproject/blob/main/assets/pforecast3.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using Prophet"><br>
  <em>Figure: Future Prediction using Prophet</em>
</p>



## 4. Visualization

Both models’ forecasts were plotted interactively to visualize predictions vs. actual values. Prophet’s component plots allowed decomposition into **trend**, **yearly seasonality**, and **holiday effects**.

```r
# Prophet interactive forecast
dyplot.prophet(prophet_model, forecast)

# Zoom into yearly and monthly components
prophet_plot_components(prophet_model, forecast)
```

ARIMA results were plotted using standard forecast visualizations to compare fitted and predicted values.


## 5. Evaluation Metrics

The following performance metrics were computed on the **test set (last 30 days)**:

| Model       | MAE     | RMSE    | MAPE    | SMAPE   | R²      |
| ----------- | ------- | ------- | ------- | ------- | ------- |
| **ARIMA**   | 20.2444 | 25.3106 | 44.9136 | 43.7925 | -0.1793 |
| **Prophet** | 6.0770  | 7.9428  | 13.5896 | 13.1615 | 0.8839  |



## 6. Error Metric Explanations

### **MAE (Mean Absolute Error)**

Measures the average magnitude of forecast errors without considering their direction.

$$MAE = \frac{1}{n}\sum |y_t - \hat{y}_t|$$

### **RMSE (Root Mean Squared Error)**

Penalizes large errors more heavily than MAE. A lower RMSE indicates higher accuracy.

$$RMSE = \sqrt{\frac{1}{n}\sum (y_t - \hat{y}_t)^2}$$

### **MAPE (Mean Absolute Percentage Error)**

Represents the error as a percentage of the actual value, allowing intuitive interpretation.

$$MAPE = \frac{100}{n}\sum \left|\frac{y_t - \hat{y}_t}{y_t}\right|$$

### **SMAPE (Symmetric Mean Absolute Percentage Error)**

Balances the percentage error between predicted and actual values, making it robust when actual values are near zero.

$$SMAPE = \frac{100}{n}\sum \frac{|y_t - \hat{y}_t|}{(|y_t| + |\hat{y}_t|)/2}$$

### **R² (Coefficient of Determination)**

Measures how well the model explains the variability of the data.
An ( R^2 ) close to 1 indicates a better fit.


## 7. Comparative Analysis

* **Prophet outperformed ARIMA** across all error metrics, especially with a much lower MAE and RMSE.
* Prophet’s **R² = 0.88** indicates a strong fit, whereas ARIMA’s **negative R²** suggests that it performed worse than a simple mean predictor.
* Prophet’s advantage arises from:

  * Handling **holiday regressors** (Diwali, New Year)
  * Accounting for **multiple seasonal patterns**
  * Smoothly capturing non-linear trends using changepoints

However, **ARIMA remains useful** when:

* Data exhibits consistent linear trends
* Seasonality is simple and well-defined
* Computational efficiency is required for smaller datasets


## 8. Zoomed Forecasts

Separate zoomed-in plots were generated to inspect model behavior over different time scales:

### **Yearly Zoom**

Displays long-term seasonality and trends (e.g., annual cycles).

```r
prophet_plot_components(prophet_model, forecast)  # yearly trends visible here
```
<p align="center">
<img src="https://github.com/evanferrao/rlabproject/blob/main/assets/exploration3.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using ARIMA"><br>
<em>Figure: Yearly Insight</em>

</p>

### **Monthly Zoom**

Highlights short-term fluctuations and model responsiveness to recent data.

```r
plot(forecast$ds, forecast$yhat, type='l', main='Prophet Monthly Zoom', xlim=c(as.Date('2022-05-01'), as.Date('2022-06-04')))
```
<p align="center">
<img src="https://github.com/evanferrao/rlabproject/blob/main/assets/pverify1.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using ARIMA"><br>
<em>Figure: Monthly Insight</em>
</p>


## 9. Conclusion

This comparative study demonstrates that **Prophet significantly outperforms ARIMA** for this dataset, offering:

* Better accuracy across all metrics
* Improved handling of holidays and nonlinear trends
* Stronger interpretability via trend and seasonality components

In contrast, **ARIMA** struggles with complex seasonalities and event-driven fluctuations.
Nevertheless, ARIMA’s statistical rigor and simplicity make it a reliable baseline model.


<p align="center">
<img src="https://github.com/evanferrao/rlabproject/blob/main/assets/finalforecast1.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using ARIMA"><br>
<em>Figure: Future Prediction comparison between Prophet and ARIMA</em>
</p>

<p align="center">
  <img src="https://github.com/evanferrao/rlabproject/blob/main/assets/finalforecast2.png?raw=true" width="400" alt="Daily Aggregated AQI Forecast using ARIMA"><br>
  <em>Figure: Future Prediction comparison between Prophet and ARIMA</em>
</p>



**Final Verdict:**
 *Prophet is the superior model for daily average forecasts in this dataset, especially when seasonal and event-driven variations are present.*



## 10. References

* Hyndman, R.J., & Athanasopoulos, G. (2018). *Forecasting: Principles and Practice*.
* Taylor, S.J., & Letham, B. (2018). *Forecasting at Scale* — Facebook Research.
* Prophet Documentation: [https://facebook.github.io/prophet/](https://facebook.github.io/prophet/)
* Forecast Package: [https://cran.r-project.org/web/packages/forecast/](https://cran.r-project.org/web/packages/forecast/)
