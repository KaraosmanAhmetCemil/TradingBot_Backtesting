import json
import datetime
import os
import numpy as np
import pandas as pd
import pandas_ta as ta
from pandas_ta import atr
import websocket
from binance.client import Client
from backtesting import Backtest, Strategy
from backtesting.lib import crossover
from config import API_KEY, API_SECRET
import time

# =============================================================================
# CONFIGURATION & INITIALIZATION
# =============================================================================

# Binance API Initialization (Testnet)


# Binance WebSocket (For Live Trading, not backtesting)
# SOCKET = "wss://testnet.binance.vision/ws/btcusdt@kline_1m"

# Initialize Binance Client (Test Mode)
# testnet doesnt matter for backtesting only for live trading
testnet = False  # True → live trading on testnet (fake money), False → live trading on real account
client = Client(API_KEY, API_SECRET, testnet=testnet)


# =============================================================================
# DATA FETCHING FUNCTIONS
# =============================================================================

def get_binance_4h_data(
    symbol="BTCUSDT",       # Trading pair
    interval="4h",          # Kline interval (4-hour)
    start_str="2015-01-01", # Start date for historical data
    end_str="2025-02-16"    # End date for historical data
    ):
    """
    Fetch historical Kline data from Binance API and prepare it for analysis.
    
    Args:
        symbol: Trading pair (e.g., BTCUSDT)
        interval: Candle time interval (1m, 1h, 4h, etc.)
        start_str: Start date in YYYY-MM-DD format
        end_str: End date in YYYY-MM-DD format
    
    Returns:
        DataFrame with Open, High, Low, Close, Volume data
    """
    
    # Fetch historical Kline data from Binance API
    historical_klines = client.get_historical_klines(symbol, interval, start_str, end_str)
    
    # Create DataFrame from raw data
    df = pd.DataFrame(historical_klines, columns=[
        "timestamp", "Open", "High", "Low", "Close", "Volume", "CloseTime", "QuoteAssetVolume",
        "NumberOfTrades", "TakerBuyBaseVolume", "TakerBuyQuoteVolume", "Ignore"
    ])
    
    # Convert timestamp and set as index
    df["timestamp"] = pd.to_datetime(df["timestamp"], unit="ms")
    df.set_index("timestamp", inplace=True)
    df = df.astype(float)

    # Validate sufficient data for calculations
    if len(df) < 200:
        raise ValueError(f"Error: Not enough data for SMA calculation. Received {len(df)} rows, but 200 are required.")

    return df[["Open", "High", "Low", "Close", "Volume"]]


# =============================================================================
# TECHNICAL INDICATOR FUNCTIONS
# =============================================================================

def custom_rsi(data, length=21):
    """
    Calculate custom RSI using HLC3 and volume normalization.
    
    Args:
        data: DataFrame with High, Low, Close, Volume columns
        length: RSI period length
    
    Returns:
        RSI values as pandas Series
    """
    # Calculate HLC3 (average of High, Low, Close)
    hlc3 = (data["High"] + data["Low"] + data["Close"]) / 3
    
    # Normalize volume relative to its moving average
    normalized_volume = data["Volume"] / data["Volume"].rolling(length).mean()
    
    # Calculate RSI on volume-adjusted price
    rsi = ta.rsi(hlc3 * normalized_volume, length=length)
    
    return rsi


def safe_sma(close_prices, period):
    """
    Calculate SMA with safety check for sufficient data length.
    
    Args:
        close_prices: Series of closing prices
        period: SMA period
    
    Returns:
        SMA values as numpy array (NaN if insufficient data)
    """
    if len(close_prices) < period:
        return np.full(len(close_prices), np.nan)  # Fill with NaN to prevent errors
    return ta.sma(close_prices, period).values


# =============================================================================
# TRADING STRATEGY CLASS
# =============================================================================

class CustomRSIStrategy(Strategy):
    """
    Custom trading strategy using RSI, SMA, and ATR-based risk management.
    """
    
    # Strategy parameters
    rsi_period = 21
    sma_period = 50
    atr_period = 14
    atr_multiplier_sl = 1.5  # Stop Loss multiplier
    atr_multiplier_tp = 5.0  # Take Profit multiplier
    
    # RSI threshold parameters
    buy_threshold = 55     # RSI level to trigger a buy
    sell_threshold = 45    # RSI level to trigger a sell

    def init(self):
        """
        Initialize technical indicators when strategy starts.
        """
        # RSI indicator with custom calculation
        self.rsi = self.I(custom_rsi, self.data.df, self.rsi_period)
        
        # Simple Moving Average for trend direction
        self.sma = self.I(safe_sma, pd.Series(self.data.Close), self.sma_period)
        
        # Average True Range for volatility-based position sizing
        self.atr = self.I(ta.atr,
                          pd.Series(self.data.High),
                          pd.Series(self.data.Low),
                          pd.Series(self.data.Close),
                          self.atr_period)

    def next(self):
        """
        Execute trading logic on each new candle.
        """
        # Skip if indicators are not ready (NaN values)
        if np.isnan(self.sma[-1]) or np.isnan(self.atr[-1]):
            return

        current_price = self.data.Close[-1]

        # Calculate dynamic stop loss and take profit based on ATR
        stop_loss = current_price - self.atr_multiplier_sl * self.atr[-1]
        take_profit = current_price + self.atr_multiplier_tp * self.atr[-1]

        # BUY CONDITION: No position + RSI crosses above buy threshold + Price above SMA
        if not self.position and crossover(self.rsi, self.buy_threshold) and current_price > self.sma[-1]:
            self.buy(sl=stop_loss, tp=take_profit)

        # SELL CONDITION: Existing position + RSI crosses below sell threshold
        elif self.position and crossover(self.sell_threshold, self.rsi):
            self.position.close()


# =============================================================================
# BACKTEST EXECUTION
# =============================================================================

# Fetch historical data for backtesting
print("Fetching historical data...")
binance_data = get_binance_4h_data()

# Initialize backtest with strategy
bt = Backtest(binance_data, CustomRSIStrategy, 
            cash=1_000_000,       # Starting cash, increased to avoid errors
            commission=0.0002     # Commission per trade
              )

# OPTIONAL: Parameter optimization (commented out for initial run)
# stats = bt.optimize(
#     buy_threshold=range(50, 70, 5),
#     sell_threshold=range(30, 50, 5),
#     sma_period=range(20, 100, 10),
#     maximize='Return [%]'
# )

# Run backtest and display results
print("Running backtest...")
stats = bt.run()
print("\n" + "="*50)
print("BACKTEST RESULTS")
print("="*50)
print(stats)

# Create backtest folder if it doesn't exist
backtest_folder = "Backtests"
if not os.path.exists(backtest_folder):
    os.makedirs(backtest_folder)
    
# Create filename with timestamp
current_time = datetime.datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
filename = f"{backtest_folder}/backtest_result_{current_time}.html"

# Plot results
print("\nGenerating performance chart...")
bt.plot(
    resample='1D',      # Convert 4-hour candles to daily for cleaner visualization
    filename=filename,  # Save to backtest folder with timestamp
    open_browser=True  # Set to True if you want it to open automatically
)

print(f"Backtest chart saved as: {filename}")
