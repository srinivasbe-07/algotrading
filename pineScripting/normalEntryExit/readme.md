# Entry/Exit Alert Script with Horizontal Lines

This TradingView Pine Script strategy generates dynamic entry, exit, target, and stop loss signals for long and short positions based on user-defined or automatically calculated price levels. The script also plots horizontal lines for these levels on the chart and provides alert conditions when the entry or exit conditions are met.

## Features

- **Dynamic Entry/Exit Signals**: Automatically generates signals for both long and short positions based on price crossovers with user-defined or calculated entry, target, and stop loss prices.
- **Price Alerts**: Configurable alerts for long/short entries and exits, allowing traders to be notified when specific price conditions are met.
- **Customizable Inputs**: Users can manually set the entry, target, and stop loss prices, or let the script calculate them dynamically.
- **Visual Chart Representation**: Plots horizontal lines for the entry, target, and stop loss levels on the chart for easy visualization.

## Inputs

The script allows for the manual input of the following parameters:

- **Entry Price**: The price level where you want to enter the trade. The default value is `0.0`, but it can be set to any specific price.
- **Target Price**: The price level where you want to take profit (exit). If left as `0.0`, the script will calculate the target as 0.5% above the entry price.
- **Stoploss Price**: The price level where you want to set the stop loss. If left as `0.0`, the script will calculate the stop loss as 0.5% below the entry price.

## Exit Conditions

- **Long Exit**: If the price crosses below the stop loss or above the target price.
- **Short Exit**: If the price crosses above the stop loss or below the target price.

Upon exit, the script resets the position counters and prepares for a new entry.

## How to Use

1. **Configure Inputs**:
   - You can manually input the entry, target, and stop loss prices or leave them at `0.0` to allow the script to automatically calculate them (target is 0.5% above entry, stop loss is 0.5% below entry).
   
2. **Alerts**:
   - The script provides alerts for the following conditions:
     - **Long Entry**: Triggered when the price crosses above the entry price.
     - **Long Exit**: Triggered when the price crosses above the target or below the stop loss.
     - **Short Entry**: Triggered when the price crosses below the entry price.
     - **Short Exit**: Triggered when the price crosses below the target or above the stop loss.

   Set up alerts in TradingView based on these conditions for real-time notifications.

3. **Chart Visualization**:
   - The script plots three horizontal lines representing the entry price (blue), target price (green), and stop loss price (red). These lines help to visualize the levels for entry and exit directly on the chart.

## Script Logic

### Entry Logic:
- **Long Entry**: A long position is initiated when the price crosses above the `Entry Price`.
- **Short Entry**: A short position is initiated when the price crosses below the `Entry Price`.

### Exit Logic:
- **Long Exit**: A long position is exited when the price crosses above the `Target Price` or below the `Stoploss Price`.
- **Short Exit**: A short position is exited when the price crosses below the `Target Price` or above the `Stoploss Price`.

## Example

For an entry price of `100`, if the target and stop loss are left as `0.0`, the script will calculate them as follows:
- **Target Price**: `100 + (100 * 0.5%) = 100.5`
- **Stoploss Price**: `100 - (100 * 0.5%) = 99.5`

The script will then trigger alerts and plot horizontal lines at these levels on the chart.

## Alerts

The script supports the following alert conditions:
- **Long Entry**: `"Long Entry triggered"`
- **Long Exit**: `"Long Exit triggered"`
- **Short Entry**: `"Short Entry triggered"`
- **Short Exit**: `"Short Exit triggered"`

You can set up these alerts in TradingView by navigating to the Alerts panel and selecting the appropriate condition.

## License

This script is open for personal use. Modification and redistribution are allowed under the [MIT License](https://opensource.org/licenses/MIT).
