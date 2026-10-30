# Indian Stocks Value-at-Risk (VaR) — Live Calculation

Live Jupyter notebook that estimates **10-day Value-at-Risk (VaR)** and **Expected Shortfall (ES / CVaR)** for an equal-weighted portfolio of 10 NSE large-cap stocks using real market data from Yahoo Finance.

**Portfolio value:** ₹1 Crore  
**Horizon:** 10 trading days  
**Confidence levels:** 95% and 99%

---

## Portfolio

| Ticker        | Sector                    |
|---------------|---------------------------|
| RELIANCE.NS   | Energy / Conglomerate     |
| TCS.NS        | IT                        |
| HDFCBANK.NS   | Banking                   |
| INFY.NS       | IT                        |
| ICICIBANK.NS  | Banking                   |
| HINDUNILVR.NS | FMCG                      |
| SBIN.NS       | Banking (PSU)             |
| BHARTIARTL.NS | Telecom                   |
| ITC.NS        | FMCG                      |
| LT.NS         | Infrastructure / Engineering |

Equal weights (10% each). Data period: **2019-01-01 → present** (live download).

---

## Methods Implemented

| Method | Description |
|--------|-------------|
| **Historical** | Empirical percentile of overlapping rolling N-day portfolio returns |
| **Parametric (sample)** | Normal approximation using sample covariance |
| **Parametric (EWMA)** | Normal approximation with RiskMetrics EWMA volatility (λ = 0.94) |
| **Student-t Monte Carlo** | Correlated multivariate Student-t scenarios (df = 5, 50,000 simulations) |
| **Filtered Historical Simulation (FHS)** | Historical standardised residuals re-scaled by current EWMA volatility |

For each method and confidence level the notebook also reports **Expected Shortfall** (mean loss beyond VaR).

---

## Additional Analysis

1. **Return diagnostics** — mean, volatility, skewness, excess kurtosis of 10-day portfolio returns  
2. **Individual stock snapshot** — last close and annualised volatility  
3. **Correlation structure** of daily log returns  
4. **Rolling Historical VaR backtest** (1-day 95% VaR, 250-day lookback) with exceedance count and **Kupiec unconditional coverage test**  
5. **Horizon sensitivity** — VaR at 1, 5, 10, 20, 40 and 60 trading days (overlapping vs non-overlapping historical, parametric, Student-t)  
6. **Dynamic conclusions** computed directly from the latest run results (no hard-coded text)

---

## Requirements

```bash
pip install yfinance pandas numpy matplotlib scipy seaborn
```

- Python 3.9+ recommended  
- Internet access required for live data download via `yfinance`

---

## How to Run

1. Open the notebook in Jupyter / VS Code / Google Colab.  
2. Run all cells in order (or use “Run All”).  
3. The first data cell downloads adjusted-close prices; subsequent cells compute VaR, ES, diagnostics, backtest and sensitivity.

Key parameters (easy to change at the top of the notebook):

```python
tickers = [...]                 # NSE tickers (suffix .NS)
start_date = "2019-01-01"
rolling_window = 10             # horizon in trading days
confidence_levels = [0.95, 0.99]
portfolio_value = 10_000_000    # ₹1 Crore
simulations = 50_000
ewma_lambda = 0.94
student_df = 5
```

---

## Typical Output (illustrative)

Results are recomputed live; numbers below are from a recent run (data through late Oct 2026).

| Confidence | Method                  | VaR (₹)  | VaR (%) | ES (₹)   | ES (%) |
|------------|-------------------------|----------|---------|----------|--------|
| 95%        | Historical              | ~417k    | ~4.2%   | ~763k    | ~7.6%  |
| 95%        | Parametric (sample)     | ~513k    | ~5.1%   | ~655k    | ~6.5%  |
| 95%        | Parametric (EWMA)       | ~456k    | ~4.6%   | ~582k    | ~5.8%  |
| 95%        | Student-t Monte Carlo   | ~641k    | ~6.4%   | ~934k    | ~9.3%  |
| 95%        | Filtered HS             | ~416k    | ~4.2%   | ~758k    | ~7.6%  |
| 99%        | Historical              | ~765k    | ~7.7%   | ~1.64M   | ~16.4% |
| 99%        | Student-t Monte Carlo   | ~1.08M   | ~10.8%  | ~1.46M   | ~14.6% |
| 99%        | Filtered HS             | ~697k    | ~7.0%   | ~1.72M   | ~17.2% |

**Distributional features (10-day portfolio returns):**  
Strong negative skew (≈ −1.9) and high excess kurtosis (≈ 15) → fat left tail. Historical / FHS / Student-t capture extremes better than pure normal methods.

**Backtest (1-day 95% Historical VaR):**  
Exception rate typically close to the nominal 5%; Kupiec test often fails to reject correct coverage at the 5% significance level.

---

## Interpretation Notes

- Indian equities exhibit **negative skew and fat tails**; normal parametric VaR can understate tail risk. Prefer **Historical**, **Filtered HS**, or **Student-t MC** for limit setting.  
- EWMA vol reflects the **current** risk regime; sample vol reflects the full-history average.  
- Kupiec tests only **unconditional coverage** (not independence of exceptions).  
- This is pure **market-risk** VaR (no liquidity, credit or operational risk).  
- Student-t degrees of freedom and EWMA λ are fixed; calibrate on a validation set if deploying in production.  
- Equal weights are used for transparency; the code can be extended to arbitrary weights.

---

## File Structure

```
.
├── Indian_Stocks_VaR_Live.ipynb   # Main notebook (live calculation)
└── README.md                      # This file
```

---

## Licence & Disclaimer

Educational / research use. Market data is obtained from Yahoo Finance and is subject to its terms. This notebook is **not** investment advice and should not be used as the sole basis for risk limits or trading decisions without independent validation.