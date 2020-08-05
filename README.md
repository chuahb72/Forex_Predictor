# Problem Statement
According to the 2019 Triennial Central Bank Survey of FX and OTC derivatives markets, the **foreign exchange or forex market** is the largest financial market in the world with a daily volume of **$6.6 trillion**. This is larger than the entire world's stock markets put together (approximately $84 billion for equities worldwide).

Within this huge amount of **intraday transactions**, approximately 70% are made by **institutional** investors while the remaning 30% made by private **retail** traders.

In this project, we will attempt to create a **predictive model** for **Intraday Retail Traders**. The model will use **historical Price and technical indicators data** to predict **probability of price increase or decrease within the next defined time period (24hrs, 7 days etc)**. 

The model must first **generalize well** for both seen and unseen future time-series data. Once that is achieved, model should conceptually allow user to **break-even or be profitable**.

# Disclaimer
- Foreign Exchange (FOREX) is a form of leveraged trading with significant levels of risk.
- One may suffer the complete loss of all initial investments.
- One should only trade what one is ready to lose.
- Past results will not guarantee future gains/losses
- The developer will not be responsible for any losses incurred by any users
- Any trading advice given during the course of development is just for education and illustrations only.

# Forex Basics: Reward-Risk Ratio
It is impossible to profit at every trade. Traders objective is to accumulate more wins than losses (net positive) over any time period.<br>
<br>
One of the most popular method is to set TP and SL with ratio > 1. Of course the **caveat is that when SL is closer to the entry price, the probabilty of price reaching the the SL level is higher than reaching the TP level**.
<br>
<br>
![](./images/equity_curve_rrr3.png)

# Target Users
### Why Retail investors?
- Forex is no stranger to automated algorithmic trading softwares. Institutional investor with massive financial and compuational resources has been using predictive quant algorithms for many years.
- Online brokers/trading platforms has become very accessible to the general public. Present day restail investors are increasingly savvy and demand for **automated trading machines** is growing.

### Why Intraday?
- The success of a **Short term trade** (typical holding time from few minutes up to a week) is less influenced by macro economic factors. It is largely speculative.
- Intraday traders relied heavily on technical indicators which are mathematical transformationa of historical price data. (quantitative in nature)
- There are 3 broad indicators categories (few examples below)
    - Trend: Simple Moving Average (SMA), Exponetial Moving Average (EMA)
    - Momentum: Relative Strength index (RSI), Moving Average Convergence/Divergence (MACD), Stochastics Oscillator
    - Volatility: Average True Range (ATR), Bollinger Bands (BB)

# Methodology
![](./images/methodology.png)

# Directory Structure
```
capstone
|__ codes
|   |__ 1.0_Forex_Basics.ipynb: Basic information, Risk-Reward ratio & win rates
|   |__ 2.0_Data_Cleaning_EDA.ipynb: Data import, cleaning, EDA, re-sampling
|   |__ 3.0_ARIMA_Modelling.ipynb: ARIMA time-series model for D1 & H1 timeframe
|   |__ 4.0_Target_define_D1.ipynb: Multi-Class Definition for D1 timeframe
|   |__ 4.1_Target_define_H1.ipynb: Multi-Class Definition for H1 timeframe
|   |__ 5.0_Feature_Engineering_D1.ipynb: Multi-Class Definition for D1 timeframe
|   |__ 5.1_Feature_Engineering_H1.ipynb: Multi-Class Definition for H1 timeframe
|__ datasets
|   |__ DAT_ASCII_EURUSD_M1_2003~2019.csv
|   |__ df_d1.csv
|   |__ df_d1_feature.csv
|   |__ df_d1_merge.csv
|   |__ df_h1.csv
|   |__ df_h1_feature.csv
|   |__ df_h1_merge.csv
|   |__ df_h4.csv
|   |__ df_m15.csv
|   |__ df_m30.csv
|__ images
|   |__ equity_curve_rrr3.png
|   |__ eurusd_pip.jpg
|   |__ metatrader4_stop_loss_take_profit__1.jpg
|   |__ methodology.png
|   |__ OHLC.png
|   |__ Positive-RRR.jpg
|   |__ RRR_winrate.png
|__ Presentation.pdf
|__ README.md
```
# Data
- DAT_ASCII_EURUSD_M1_2003~2019.csv: raw 1 minute OHLC data from 2003 to 2019
- df_d1.csv: resampled 24hr OHLC data
- df_d1_feature.csv: 24hr OHLC data + feature engineering
- df_d1_merge.csv: 24hr OHLC data + multi-class labels
- df_h1.csv: resampled 1hr OHLC data
- df_h1_feature.csv: 1hr OHLC data + feature engineering
- df_h1_merge.csv: 1hr OHLC data + multi-class labels
- df_h4.csv: resampled 4hr OHLC data
- df_m15.csv: resampled 15min OHLC data
- df_m30.csv: resampled 30min OHLC data

## Data Dictionary
|Feature|Type|Dataset|Description|
|---|---|---|---|

# Multi-Class Definition
- Set Take Profit (TP) at 1 sigma of look ahead prices (140 pips for D1, 55 pips for H1)
- Set Stop Loss at (SL) 0.33 * (1 sigma), **Reward-Risk Ratio of 3**
- Using D1 time frame as example
    - For every look ahead day, if price did not cross -0.33 * 140 and crosses +140 TP >> Label 1, Profitable Long/Buy Trade
    - For every look ahead day, if price did not cross +0.33 * 140 and crosses -140 TP >> Label 2, Profitable Short/Sell Trade

# Feature Engineering
- Close Price Shifts
    -Price movements can be very correlated to their previous period close. We create new columns for shift 1,2,3,5,8,13,21,34,55,89. The relative price distance between current close and shifted closed will also be added.
    - Relative distances between each shifted prices will also be introduced into the modelling
    
- Simple Moving Averages
    - A simple moving average (SMA) calculates the average of a selected range of prices, usually closing prices, by the number of periods in that range.
    - The SMA is a technical indicator that can aid in determining if an asset price will continue or reverse a bull or bear trend.
    - The relative distances of the current close price to the SMA can also help predict price movement
    - We create new columns for SMA with periods 2,3,5,8,13,21,34,55,89. 
    - The relative distances between current close price and SMAs will also be added
    - Relative distances between each SMAs will also be introduced into the modelling
    
- Relative Strength Index (RSI)
    - The relative strength index (RSI) is a popular momentum oscillator developed in 1978.
    - The RSI provides technical traders signals about bullish and bearish price momentum, and it is often plotted beneath the graph of an asset's price.
    - An asset is usually considered overbought when the RSI is above 70% and oversold when it is below 30%.
    - We create new columns for RSI(14)
    - formula for RSI: https://en.wikipedia.org/wiki/Relative_strength_index
    - RSI(14) is a rather noisy signal line.
    - We will introduce shifts and simple moving averages to this indicator.
    - Similarly, relative distances between the shifts/MA and current RSI will be added
    - Relative distances between the shifts/MAs will also be added


# Models Metric Summary
- D1 Timeframe

|Models|ARIMA|Log Regression|Random Forest|XGBoost|FNN|RNN|
|---|---|---|---|---|---|---|
|Modelling Time|43s|---|---|---|---|---|
|Long Trade Precision|NA|---|---|---|---|---|
|Short Trade Precision|NA|---|---|---|---|---|
|Overall Precision|NA|---|---|---|---|---|


### Type 1, 2 error implications
- TBD

# Conclusion & Recommendation
- TBD