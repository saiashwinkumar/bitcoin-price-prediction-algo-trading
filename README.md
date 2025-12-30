# Bitcoin Daily Price Prediction and Algortihmic Trading Model
- Bitcoin prices are highly volatile and driven by market behavior, technical factors, and global sentiment.
- Predicting the exact price is unreliable- too much randomness.
- Instead, we aim to predict a probable price range and direction for the next day, turning those forecasts into trading signals.

## Objective
- Predict the next day’s Bitcoin price range (95% confidence) using quantile regression to account for uncertainty and volatility.
- Turn these forecasts into trading signals based on predicted returns and market risk.
- Measure both accuracy and profitability using MAPE, Sharpe ratio, and drawdown.

## Data and Inputs
- Market Data: Yahoo Finance, Coingecko, Binance (OHLCV data —    Open, High, Low, Close, Volume).
- Market Sentiment: Fear & Greed Index to measure investor mood.
- On-Chain Metrics: Transaction count, active addresses, hash rate (network health indicators).
- External Factors (for extended models): S&P 500, VIX, and DXY indices (risk-on/risk-off signals).
- Target variable: next-day average-price log return log(A(t+1)) − log(C(t)), where A = (open+high+low+close)/4 and C = today’s close

Engineered features: 
1. Momentum and trend: moving averages
2. Volatility: realized vol (7/30 days)
3. Market regime: EWMA volatility 
4. Technical Indicators: RSI, MACD, Stochastics, ADX, Bollinger Bands, SMA/EMA, Pivot Points (Supports & Resistances), VWMA, Ichimoku Base.
Network and sentiment features for behavioral dimension.
5. Standardized using train-set statistics ⟶ inverse transform for evaluation.

## Model Architecture
### Baseline Models:
- Naïve: P(t+1) = P(t)
- ARIMA: classic time-series model ΔlogP(t) with (p,q) ∈ {0,1,2}, AIC-based selection.
### Machine Learning Models:
1. LSTM: 60-day input window, dropout, early stopping to prevent overfitting.
2. Switch Model: uses LSTM only on high-volatility days; Naïve otherwise.
3. Quantile Gradient Boosting Regression (QGBR): 
  - Predicts lower (2.5%) and upper (97.5%) price quantiles.
  - Uses a rolling 720-day training window, refitting every 5 days.
  - Adds Conformal Calibration to ensure 95% coverage reliability.

## Model Forecast Results
- Before (example): Naive ≈ 1.27, ARIMA ≈ 2–6, LSTM ≈ 1.26–1.40 (variable).
- After (single split): Naive_mape ≈ 1.269, lstm_mape ≈ 1.25, switch_mape ≈ 1.261
- QGBR Model:
  - MAPE: 0.985% (compared to Naive MAPE: 1.242%) 
  - Coverage (PICP): 0.947
  - MIW: 3300.16
  - Winkler Score: 4480.05 
  - Direction Hit Rate: 93%
<img width="640" height="648" alt="image" src="https://github.com/user-attachments/assets/fa4f31e8-58cb-47be-9ac3-edd1b0abf310" />

## Prediction to Action
Idea: Turn model outputs into trading signal.
- Signal = “How much higher/lower do we think tomorrow’s average price will be vs today’s close?”
- We compute a predicted return:
  <img width="236" height="70" alt="image" src="https://github.com/user-attachments/assets/9209077b-efc3-4fa7-b781-fc2846c0d382" />
- Dynamic filter (trade only when it matters): We set a moving threshold (τt) = 70th percentile of (|z|) over the last ~120 days.
- Trade Logic: trade only if |zₜ| > τₜ (70th percentile)
- Position = tanh(k,zₜ)* tilt_cap
- Positive (z) ⟶ long, Negative ⟶ short

## Risk Management
- Tiny Trend Baseline: Maintain 20–40% exposure in uptrends.
- Volatility Targeting: Adjust leverage to maintain ~20% annual volatility.
- Leverage Cap: Max 1.5× exposure.
- Execution Logic:
  - One trade per day at close.
  - Include 0.1% one-way fee and slippage filter.
  - Skip negligible trades to control turnover.
 
## Simulating Real Trading
- Start portfolio: $100,000.
- Every day, decide position size based on signal strength.
- Apply fees and update equity curve.
- Backtest period split: 80% training / 20% testing.
- Training data was used to calibrate and choose the best hyperparameters (such as k, tilt cap, leverage cap etc.)
- Record daily portfolio value, returns and trades.

## Simulation Results
<img width="1415" height="775" alt="image" src="https://github.com/user-attachments/assets/230bd87f-e59f-48e0-a7d7-d1481656944d" />

- Results appear strong but verified no look-ahead or leakage.
- Correlation between predicted & true average ≈ 0.9997 (MAPE 0.91%).
- High correlation due to same-day mean price similarity, not data overlap.
- Trading signals (|z| > τ) align precisely with major trend days → clean equity curve.
- Exits occur once |z| ≤ τ, ensuring disciplined trade closure.

<img width="1072" height="431" alt="image" src="https://github.com/user-attachments/assets/c07a656d-4972-44b8-9eb8-468411cef7ab" />

- Model generalizes well to unseen data → high profitability even out-of-sample.
- Forecasts are tradable, not just statistically accurate.
- Scatter plots show 45° calibration line → unbiased predictions.
- Hit rate increases with |z|, meaning stronger confidence = better accuracy.
- Trades occur selectively on strong trends → fewer trades, higher quality.
- Volatility targeting keeps drawdowns <1%, allowing compounding.

<img width="514" height="213" alt="image" src="https://github.com/user-attachments/assets/b1eb071a-4d40-4ef4-9ad0-a668a8956629" />

## Key Takeaways
- Predicting ranges is more stable than single-point forecasts.
- Quantile regression + calibration ensures trustworthy confidence intervals.
- ML-driven signals + risk filters lead to profitable and explainable trading behavior.
- Demonstrates how ML forecasts + trading logic can yield explainable AI investing.




















