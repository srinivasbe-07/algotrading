# 🚀 Srini Swing Indicator

![Version](https://img.shields.io/badge/version-v1.0-blue.svg) ![Pine Script](https://img.shields.io/badge/TradingView-Pine%20Script-blue) ![License](https://img.shields.io/badge/license-MIT-green)

**Srini Swing Indicator** is a customizable Pine Script strategy designed for TradingView. It helps traders detect swing highs and lows, automatically generate entry and exit signals, and manage risk with dynamic profit targets.

---

## 📋 Features
- 🔥 **Swing High/Low Calculation**: Detects key price levels using user-defined candle lengths.
- 💹 **Auto Trading Signals**: Automatically triggers long and short positions.
- 💡 **Positional & Intraday Trading**: Configurable for both styles of trading.
- 📅 **Options Trading Support**: Supports trading based on selected weekdays for options strategies.
- 🎯 **Target Profit**: 0.5% default profit target with customizable settings.
- 🔔 **Alerts**: Notifies you of entry and exit signals with TradingView alerts.
- 🕒 **3:16 PM Exit for Intraday**: Automatically exits all open positions at 3:16 PM for intraday trading.


---

## 📦 Installation

1. Copy the **Srini Swing Indicator** Pine Script code into TradingView's Pine Editor.
2. Save and apply the script to your chart.
3. Adjust the parameters according to your strategy.

---

## ⚙️ Parameters

| Parameter        | Description                                            | Default       |
|------------------|--------------------------------------------------------|---------------|
| **Length**       | Number of candles used to calculate swing highs/lows   | 10            |
| **Position Type**| Select between **Positional** and **Intraday** modes   | Positional    |
| **Options Day**  | Choose the day of the week for options trading         | Thursday      |
| **Target Profit**| Profit target for exits (percentage of entry price)    | 0.5%          |

---

## 📊 Example Signals

### Long Entry
- **Condition**: Price crosses above the swing high.
  
### Short Entry
- **Condition**: Price crosses below the swing low.

---

## 🔔 Alerts

| Alert Type       | Trigger Condition                                                  |
|------------------|--------------------------------------------------------------------|
| **Long Entry**   | When price crosses above the swing high                            |
| **Short Entry**  | When price crosses below the swing low                             |
| **Long Exit**    | When long position exit conditions are met or at 3:16 PM           |
| **Short Exit**   | When short position exit conditions are met or at 3:16 PM          |

---

## 🛠️ Customization

You can modify the following parameters in the script:
- **Swing Length**: Adjust the number of candles for calculating swing highs/lows.
- **Profit Targets**: Set your preferred target profit percentage.
- **Day Selection**: Choose specific days for options trading.

---

## 📚 Documentation

For further customization, refer to the official [TradingView Pine Script Documentation](https://www.tradingview.com/pine-script-docs/en/v5/).

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.
