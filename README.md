# Tradingbot_Backtesting

# Trading Strategy Backtest - Binance Setup Guide

## 📋 Project Overview
This project implements a custom RSI-based trading strategy with backtesting capabilities using Binance historical data. The strategy uses RSI, SMA, and ATR indicators for automated trading decisions.

## 🚀 Quick Start

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```

### 2. Set Up Binance API Keys

#### Step 1: Create Binance Account
1. Go to [Binance.com](https://www.binance.com) and create an account
2. Complete identity verification (KYC) if required

#### Step 2: Enable Testnet for Safe Testing
1. Go to [Binance Testnet](https://testnet.binance.vision/)
2. Click "Generate API Key" 
3. Log in with your Binance credentials
4. Create a new API key with these permissions:
   - **Enable Reading** ✅
   - **Enable Spot & Margin Trading** ✅
   - **Enable Withdrawals** ❌ (not needed for testing)

#### Step 3: Configure API Keys
Edit the existing `config.py` file and add your API keys:

```python
# config.py
API_KEY = "your_api_key_here_from_testnet"
API_SECRET = "your_secret_key_here_from_testnet"
```

**Example:**
```python
# config.py
API_KEY = "aBcDeFgHiJkLmNoPqRsTuVwXyZ1234567890abcdef"
API_SECRET = "zYxWvUtSrQpOnMlKjIhGfEdCbA9876543210zyxwv"
```

### 3. Run the Backtest
```bash
python Tradingbot_Backtesting.py
```

## 🔧 File Structure
```
project/
├── Tradingbot_Backtesting.py  # Main trading strategy
├── config.py                  # API keys (already exists)
├── requirements.txt           # Dependencies
├── backtest/                  # Auto-created folder for results
│   └── backtest_result_2024-01-15_14-30-25.html
└── README.md                  # This file
```

## ⚙️ Configuration

### Testnet vs Live Trading
- **Testnet**: Uses fake money - perfect for testing
- **Live**: Uses real money - enable only when ready

### Switching to Live Trading

When you’re ready to trade with real funds:

1. Go to [Binance.com](https://www.binance.com/) → **API Management**.  
2. Create new API keys for your live account (**do not use testnet keys**).  
3. Update `config.py` with your **live API key and secret**.  
4. Ensure `testnet` is set to `False` when initializing the client:  
   ```python
   client = Client(API_KEY, API_SECRET, testnet=False)
   ````
   THIS USES REAL MONEY NOW IF U WANNA USE LIVE WITH FAKE MONEY SET TO FALSE



## 📊 Strategy Parameters
The strategy uses these default parameters (customizable):
- **RSI Period**: 21
- **SMA Period**: 50  
- **ATR Period**: 14
- **Buy Threshold**: RSI > 55
- **Sell Threshold**: RSI < 45

## 🛡️ Safety Notes

### 🔒 Security Best Practices
- Never commit `config.py` to version control (it's in .gitignore)
- Use environment variables in production
- Regularly rotate API keys

### 📈 Risk Warning
- This is for educational purposes only
- Test thoroughly on Testnet before using real funds
- Past performance doesn't guarantee future results
- Cryptocurrency trading involves significant risk

## ❓ Troubleshooting

### Common Issues

**"Invalid API key" Error**
- Check if keys are copied correctly (no extra spaces)
- Verify you're using Testnet keys for testing
- Ensure API key permissions are correct

**"Not enough data" Error**
- The script requires at least 200 data points
- Try a earlier start date in `get_binance_4h_data()`

**Module Not Found Errors**
```bash
pip install --upgrade -r requirements.txt
```

## 📈 Understanding Results

After running the backtest, you'll see:
- **Equity Curve**: Portfolio value over time
- **Trade History**: All buy/sell transactions
- **Performance Metrics**: Sharpe ratio, max drawdown, etc.
- **HTML Report**: Interactive chart automatically opens in your browser

## 🔄 Customization

### Modify Strategy Parameters
Edit the `CustomRSIStrategy` class in `Tradingbot_Backtesting.py`:
```python
class CustomRSIStrategy(Strategy):
    rsi_period = 21        # Change RSI period
    buy_threshold = 55     # Adjust buy signal threshold
    # ... other parameters
```

### Change Trading Pair
Modify the `get_binance_4h_data()` call in `Tradingbot_Backtesting.py`:
```python
binance_data = get_binance_4h_data(symbol="ETHUSDT")  # Change to Ethereum
```
