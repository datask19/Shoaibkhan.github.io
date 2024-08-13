# ATR Moving Average Trend Following Strategy

This repository contains an implementation of a trend-following trading strategy using the Average True Range (ATR) and Moving Average (MA). The strategy is implemented in Python using the `backtesting` library and calculates indicators manually without relying on the `.rolling` method.

## Overview

- **Strategy**: The strategy buys when the closing price is above the moving average and sells when it is below the moving average. It uses ATR to set dynamic stop-loss and take-profit levels based on market volatility.
- **Indicators**:
  - **Moving Average (MA)**: A simple moving average (SMA) calculated over a specified window.
  - **Average True Range (ATR)**: A measure of market volatility used to set stop-loss and take-profit levels.

## Requirements

- Python 3.7+
- `backtesting` library
- `numpy` library
- `pandas` library
- `yfinance` library


