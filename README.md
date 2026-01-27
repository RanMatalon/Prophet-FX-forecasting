# Prophet-FX-forecasting

**Disclaimer:**
This model is intended **for educational and analytical purposes only**.
Financial forecasts produced by Prophet do **not** constitute investment advice and should not be used for trading decisions.

This repository contains a Jupyter Notebook demonstrating time series forecasting using **Meta Prophet**.
The project focuses on financial time series FX data retrieved via **yfinance** (Yahoo finance) API library.

This project uses Meta Prophet, an additive time series forecasting model designed to handle trend, seasonality, and holiday effects in a robust and interpretable way.
The model requires datetime and its numeric measure.

We begin the pipeline by loading the EURO-SHEKEL rates from the last 2 years (Can be any FX rates or any other period).

The model examines the core components:

-- *Trend: piecewise linear growth*
-- *Seasonality: Yearly, Monthly, Weekly, Daily*
-- *Holidays effects*
-- *Automatic changepoint detection*

The model is evaluated using hyper-parameter tuning:

1. **changepoint_prior_scale** (Usual range: 0.001 to 0.5): What it controls - The flexibility of the trend - how much the model can adjust to changes in the growth rate.
   Low values (0.001 - 0.01): Makes the trend very smooth and stable, resists sudden changes, better for currencies with steady, predictable trends.
   High values (0.1 - 0.5): Makes the trend very flexible, can adapt (too) quickly to sudden changes, may overfit to noise.
   Example: If a central bank suddenly changes interest rates, a higher value helps the model adapt faster.

2. **seasonality_prior_scale** (Usual range: 0.01 to 10.0): How strongly seasonal patterns are fit to the data. High values are good when there are clear day-of-week or monthly effects.

3. **seasonality_mode** ('additive' or 'multiplicative'): How seasonality combines with the trend - y = trend + seasonality / y = trend*seasonality.

4. **changepoint_range** (Usual range: 0.8 to 0.9): What it controls - What portion of the historical data can have trend changepoints ? (The rest, must follow the established trend).
   Example: If there was a major policy change in the last 10% of your data, 0.9 helps capture it.

**How it works together ?**
*Final Prediction = Trend (changepoint_prior_scale, changepoint_range)+ Seasonalities (seasonality_prior_scale, seasonality_mode) + Error*

So we have got the final prediction form, but how does Prophet **validates** the model ? using time series cross-validation.
What controls the ratios and data for the inner validation used for the best model ? **Another two hyper-parameters: cv_initial, cv_horizon**

What are they ? and how do they work together ?
*cv_initial: The minimum amount of historical data used for the initial training period.*
Important: For yearly seasonality at least 365 is required.
*cv_horizon: How far into the future you want to test predictions (your forecast window). The first forecast starts AFTER this initial period*
Important: Longer horizons = harder to predict = higher error (which is expected!)

Example: cv_initial=365, cv_horizon=30
Fold 1: Train on [Jan 2024 ─ Jan 2025] → Examine next 30 days
Fold 2: Train on [feb 2024 ─ Feb 2025] → Examine next 30 days
Fold 3: Train on [Mar 2024 ─ Mar 2025] → Examine next 30 days
...
**NOTE: # of Folds will be: (Total_data_length - cv_initial) / cv_horizon**

What are the trade-offs ?

* Longer initial / Shorter horizon - Better predicting, lower MAE (Easier to predict), Might be overfitted.
* Shorter initial / Longer horizon - Less predicting knowledge, Increases MAE (Harder to predict), Enhaces low-data model to learn predictive patters.

<img width="932" height="373" alt="image" src="https://github.com/user-attachments/assets/8b50e03f-3398-4f7e-825e-8305b473331d" />

Especially for this need i created the function calculate_optimal_cv_initial, which takes inputs of (data_length,forecast_horizon_days,target_folds,min_ratio,max_initial_days) and
creates the optimal cv_initial while keeping the constraints of the min_ratio.
Can recommand if possible higher folds can be reached, too.
If target folds are not possible given the data and horizon, there will be fitted a default cv_initial.

After the evaluating process, we split a df_train dataframe, that has time_back_window, cuts the last time_back_window days for future backtesting.

The plots describe the trends and different seasonalities found,
forecasting future records including confidence interval uncertainity (default is 80%), and residuals distribution analysis.
A table of forecasting with confidence intervals is added, too.


**Disclaimer:**
This model is intended **for educational and analytical purposes only**.
Financial forecasts produced by Prophet do **not** constitute investment advice and should not be used for trading decisions.
