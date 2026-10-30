# Pairs-Trading-Research

import pandas as pd
import numpy as np
import statsmodels.api as sm

from statsmodels.tsa.ar_model import AutoReg, ar_select_order
from statsmodels.tsa.regime_switching.markov_autoregression import MarkovAutoregression

import pandas_datareader as pdr
import pandas as pd

df = pd.read_csv("/content/data_xy.csv")

df['mid_VOO'] = (df['bid_px_00_VOO'] + df['ask_px_00_VOO']) / 2
df['mid_SPY'] = (df['bid_px_00_SPY'] + df['ask_px_00_SPY']) / 2
#Mid price of each etf

df['mispricing'] = np.log(df['mid_VOO']) - np.log(df['mid_SPY'])
df['mispricing_lag'] = df['mispricing'].shift(1)
#Mispricing Variable

df['depth_VOO'] = df['ask_sz_00_VOO'] - df['bid_sz_00_VOO'] / df['mid_VOO']
df['depth_SPY'] = df['ask_sz_00_SPY'] - df['bid_sz_00_SPY'] / df['mid_SPY']

df['liq_mid'] = df['depth_VOO'] - df['depth_SPY']
df['liq_lag'] = df['liq_mid'].shift(1).dropna()
#Liquidity Variable

df
new_df = df[['mid_VOO', 'mid_SPY', 'liq_mid', 'depth_VOO', 'depth_SPY']]

import numpy as np
import matplotlib.pyplot as plt

coef = 0.9954
periods = 400
shock = np.array([coef**t for t in range(periods)])
half_life = int(np.log(0.5) / np.log(coef))

plt.plot(shock)
plt.axvline(half_life, color='red', linestyle='--', label=f'Half-life: {half_life} periods')
plt.axhline(0.5, color='gray', linestyle=':')
plt.title('Mispricing Impulse Response: How Long Does a Dislocation Last?')
plt.xlabel('Periods After Shock')
plt.ylabel('Remaining Mispricing (normalized)')
plt.legend()
plt.tight_layout()
plt.show()

X = df[['mispricing_lag', 'liq_lag']]   # independent variables
y = df["mispricing"]             # dependent variable

# Drop rows with NaN values from X and y
data_for_model = pd.concat([y, X], axis=1).dropna()
y = data_for_model["mispricing"]
X = data_for_model[['mispricing_lag', 'liq_lag']]

X = sm.add_constant(X)           # adds intercept (VERY important)

model = sm.OLS(y, X).fit()

print(model.summary())

plt.figure(figsize=(10,10))
plt.plot(df['ts_event'],df['mid_VOO'])
plt.plot(df['ts_event'],df['mid_SPY'])
plt.show()

plt.figure(figsize=(10,10))
plt.plot(df['ts_event'],df['liq_lag'])
plt.plot(df['ts_event'],df['mispricing_lag'])
plt.plot(df['ts_event'],df['mispricing'])
plt.show()

import statsmodels.api as sm
import matplotlib.pyplot as plt

# ACF (autocorrelation function)
sm.graphics.tsa.plot_acf(paired['mispricing'], lags=50)
plt.show()

# PACF (partial autocorrelation)
sm.graphics.tsa.plot_pacf(paired['mispricing'], lags=50)
plt.show()

plt.figure(figsize=(10,10))
plt.plot(voo['ts_event'],voo['bid_px_00'])
plt.plot(voo['ts_event'],voo['ask_px_00'])
plt.show()

plt.figure(figsize=(50,10))
plt.plot(spy['ts_event'],spy['bid_px_00'])
plt.plot(spy['ts_event'],spy['ask_px_00'])
plt.show()

df['d_mispricing'] = paired['mispricing'].diff().dropna()
mispricing_mod = MarkovAutoregression(df['d_mispricing'], k_regimes=2, order=1, switching_variance=True)

mispricing_res = mispricing_mod.fit(disp=True)
print(mispricing_res.summary())

mispricing_res.filtered_marginal_probabilities[0].plot(figsize=(12,4))
