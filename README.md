# Pairs-Trading-Research

Group project for F434 at Indiana University's Kelley School of Business, studying
what happens to the SPY and VOO order books when their prices temporarily diverge.

**Team:** Brooklyn Clark, Griffin Dusseau, Nithin Saravanan, Sameer Siddiqui

## Research Question
SPY and VOO both track the S&P 500, so their prices should move almost in lockstep.
How often do they break apart, how long do those breaks last, and what does
liquidity look like while they're happening?

## Data
- 1-second top-of-book quote data (best bid, best ask, and sizes) for SPY and VOO
- Regular trading hours only
- 1,422,008 total observations

## Defining a Dislocation
1. Compute the mid price for each ETF from its best bid and ask
2. Form the ratio Mid(SPY) / Mid(VOO)
3. Compute a rolling intraday z-score of that ratio
4. Flag a dislocation when the absolute z-score exceeds 2.58 (the 99% two-sided threshold)

## Methods
- **Stationarity:** Augmented Dickey-Fuller test on the price ratio
- **Persistence and liquidity:** OLS regression of mispricing (log price spread) on its own lag and lagged liquidity
- **Time-series structure:** AR(3) autoregressive model, with ACF and PACF to guide lag selection
- **Regime detection:** two-regime Markov Switching autoregression with switching variance, separating low- and high-volatility states

## Key Findings
- **Dislocations occur about 3% of the time.** 41,815 of 1.42 million observations (2.94%) crossed the ±2.58 threshold, roughly three times what a normal distribution predicts. The z-score distribution has fat tails, so normal-based risk models would understate how often these events happen.
- **The relationship is stable and mean-reverting.** The ADF statistic of −23.4 (p < 0.01) shows the price ratio is stationary, so deviations are temporary rather than permanent drift.
- **Dislocations are persistent in the short run.** The lagged mispricing coefficient of 0.9954 implies a half-life of about 150 seconds, so a typical gap closes gradually over a few minutes rather than instantly.
- **Liquidity dries up during dislocations.** Bid-ask spreads widen for both ETFs and quoted size falls, pointing to fragile liquidity and higher trading costs exactly when the opportunity appears. Lagged liquidity had a small negative coefficient, consistent with liquidity helping correct mispricing.
- **Two distinct regimes.** The Markov model separates a calm, highly persistent regime (AR coefficient 0.91) from a volatile regime that is much less persistent (0.25) and tends to end quickly.

## Limitations
- The regression's R² of 0.99 comes mostly from the lagged mispricing term, which is expected for a highly persistent series, so it shouldn't be read as strong predictive power from liquidity.
- Wider spreads during dislocations mean transaction costs could absorb much of the potential arbitrage profit. A backtest with realistic costs would be the natural next step.

## Tools
Python, pandas, NumPy, statsmodels, Matplotlib, Google Colab

## Files
- `Pairs_Trading_Project.ipynb`: full analysis notebook
