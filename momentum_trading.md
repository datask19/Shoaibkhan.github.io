# Momentum Trading Strategy

## Overview

The Momentum Trading Strategy is designed to capitalize on strong trends by using two key technical indicators: Momentum and Relative Strength Index (RSI). This strategy aims to identify and exploit periods of upward or downward momentum in an asset's price.

## Strategy Components

1. **Momentum Indicator**:
   - **Purpose**: Measures the rate of change in the asset’s price to identify the strength of the current trend.
   - **Period**: 14 days.

2. **Relative Strength Index (RSI)**:
   - **Purpose**: Evaluates whether an asset is overbought or oversold, indicating potential reversal points.
   - **Period**: 14 days.

## Entry and Exit Criteria

### Buy Signal
- **Condition 1**: The Momentum Indicator is above zero, indicating positive momentum.
- **Condition 2**: The RSI is above 50, suggesting a bullish trend.

### Sell Signal
- **Condition 1**: The Momentum Indicator is below zero, indicating a loss of momentum.
- **Condition 2**: The RSI drops below 50, suggesting a bearish trend.

### Exit Criteria
- **Stop-Loss**: Set at a percentage below the entry price (e.g., 2%) to limit potential losses.
- **Take-Profit**: Based on a predefined target or when momentum starts to decrease.

![Strategy Plot](file:///C:/Users/Shoaib%20khan/Downloads/pics%20for%20algo%20port/Figure_3.png)


## startegy code 

Below is an example implementation of the Momentum Trading Strategy using Backtrader in Python:

```python
import backtrader as bt
import yfinance as yf

class MomentumTradingStrategy(bt.SignalStrategy):
    def __init__(self):
        # Define the momentum and RSI indicators
        self.momentum = bt.indicators.Momentum(self.data.close, period=14)
        self.rsi = bt.indicators.RelativeStrengthIndex(period=14)

    def next(self):
        if not self.position:
            # Buy Signal
            if self.momentum[0] > 0 and self.rsi[0] > 50:
                self.buy()
        else:
            # Sell Signal
            if self.momentum[0] < 0 or self.rsi[0] < 50:
                self.sell()

class MomentumBacktest(bt.Cerebro):
    def __init__(self):
        super().__init__()
        self.addstrategy(MomentumTradingStrategy)

    def load_data(self, symbol, start_date, end_date):
        data = yf.download(symbol, start=start_date, end=end_date)
        data = data[['Open', 'High', 'Low', 'Close', 'Volume']]
        data = bt.feeds.PandasData(dataname=data)
        self.adddata(data)

    def run_backtest(self):
        # Add analyzers
        self.addanalyzer(bt.analyzers.SharpeRatio, _name='sharpe_ratio')
        results = self.run()
        return results[0]  # Return the first strategy result

def main():
    symbols = ['TSLA', 'GOOGL', 'MSFT', 'NVDA']
    start_date = '2019-01-01'
    end_date = '2023-01-01'
    
    for symbol in symbols:
        cerebro = MomentumBacktest()
        cerebro.load_data(symbol, start_date, end_date)
        strategy = cerebro.run_backtest()
        
        # Access analyzers
        sharpe_ratio = strategy.analyzers.sharpe_ratio.get_analysis()
        annual_return = sharpe_ratio.get('sharperatio', 'N/A')
        annual_volatility = sharpe_ratio.get('annualvol', 'N/A')

        try:
            annual_return = float(annual_return)
            annual_volatility = float(annual_volatility)
            print(f"{symbol} Backtest Results:")
            print(f"Annual Return: {annual_return:.2%}")
            print(f"Annual Volatility: {annual_volatility:.2%}")
        except ValueError:
            print(f"{symbol} Backtest Results:")
            print(f"Annual Return: {annual_return}")
            print(f"Annual Volatility: {annual_volatility}")

        cerebro.plot(style='candlestick')

if __name__ == "__main__":
    main()
