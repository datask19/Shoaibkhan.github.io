# ATR Moving Average Trend Following Strategy

This repository contains an implementation of a trend-following trading strategy using the Average True Range (ATR) and Moving Average (MA). The strategy is implemented in Python using the `backtesting` library and calculates indicators manually without relying on the `.rolling` method.

## Overview

- **Strategy**: The strategy buys when the closing price is above the moving average and sells when it is below the moving average. It uses ATR to set dynamic stop-loss and take-profit levels based on market volatility.
- **Indicators**:
  - **Moving Average (MA)**: A simple moving average (SMA) calculated over a specified window.
  - **Average True Range (ATR)**: A measure of market volatility used to set stop-loss and take-profit levels.
 
## Strategy code
  - import pandas as pd
import numpy as np
from backtesting import Backtest, Strategy
import yfinance as yf

# Fetch historical data
def fetch_data(ticker, start_date, end_date):
    data = yf.download(ticker, start=start_date, end=end_date)
    data.index.name = 'Date'
    return data

# Calculate moving average manually
def calculate_moving_average(series, window):
    weights = np.ones(window) / window
    sma = np.convolve(series, weights, mode='valid')
    # To match the length of the input series
    sma = np.concatenate([np.full(window-1, np.nan), sma])
    return pd.Series(sma, index=series.index)

# Calculate ATR manually
def calculate_atr(data, period=14):
    high_low = data['High'] - data['Low']
    high_prev_close = (data['High'] - data['Close'].shift()).abs()
    low_prev_close = (data['Close'].shift() - data['Low']).abs()
    tr = pd.concat([high_low, high_prev_close, low_prev_close], axis=1).max(axis=1)
    
    atr = np.zeros_like(tr)
    for i in range(period, len(tr)):
        atr[i] = tr[i-period:i].mean()
    
    # Fill initial values with NaN to match the length of the input series
    atr[:period-1] = np.nan
    return pd.Series(atr, index=tr.index)

# Define the strategy
class ATRMovingAverageStrategy(Strategy):
    short_window = 50
    atr_period = 14
    atr_multiplier = 1.5

    def init(self):
        # Initialize moving averages and ATR
        self.mavg = self.data.Short_Mavg
        self.atr = self.data.ATR

    def next(self):
        # Entry signal: Price above moving average
        if self.data.Close[-1] > self.mavg[-1] and not self.position:
            # Dynamic stop-loss and take-profit based on ATR
            stop_loss = self.data.Close[-1] - self.atr[-1] * self.atr_multiplier
            take_profit = self.data.Close[-1] + self.atr[-1] * self.atr_multiplier
            self.buy(sl=stop_loss, tp=take_profit)

        # Exit signal: Price below moving average
        elif self.data.Close[-1] < self.mavg[-1] and self.position:
            self.sell()

# Fetch data
data = fetch_data('AAPL', '2022-01-01', '2024-01-01')
data['Short_Mavg'] = calculate_moving_average(data['Close'], window=50)
data['ATR'] = calculate_atr(data, period=14)
data = data.sort_index()
bt = Backtest(data, ATRMovingAverageStrategy, cash=10_000, commission=.002)
stats = bt.run()
bt.plot()
print(stats)


## Requirements

- Python 3.7+
- `backtesting` library
- `numpy` library
- `pandas` library
- `yfinance` library


