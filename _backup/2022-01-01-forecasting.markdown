---
layout: post
title:  "Forecasting"
date:   2022-01-01 00:00:00 +0100
categories: machine-learning forecasting
---

# Forecasting

The purpose of this section is to learn from scratch how forecast methods work.

**[Forecasting][1]** is the process of making predictions based on past and present data and most commonly by analysis of trends.

**Stock market forecast**: I will use stock market data for this purpose. The goal is to learn about forecasting 
but with a great motivation - make money :-) 

I will try to cover all methods from GLM, Arima to LSTM.

# Main libraries/packages

I will use Yahoo Finance data and some libraries that gathers the data for me.
```
pip install yahoo-fin yfinance
```

- `yahoo-fin`: I will use it for retrieving the Ticker symbols for the different markets. 
A ticker symbol or stock symbol is an abbreviation used to uniquely identify publicly 
traded shares of a particular stock on a particular stock market 
- `yfinance`: Given the ticker symbols, we can use `yfinance` to get stock values. We can retrieve stock values at
different granularity (finest granularity is 1 minute). We will get one data point per day.

```
| Date                |     Open |     High |      Low |    Close |   Adj Close |      Volume | Ticker   |
|:--------------------|---------:|---------:|---------:|---------:|------------:|------------:|:---------|
| 1999-12-31 00:00:00 | 0.901228 | 0.918527 | 0.888393 | 0.917969 |    0.787034 | 1.63811e+08 | AAPL     |
| 2000-01-03 00:00:00 | 0.936384 | 1.00446  | 0.907924 | 0.999442 |    0.856887 | 5.35797e+08 | AAPL     |
| 2000-01-04 00:00:00 | 0.966518 | 0.987723 | 0.90346  | 0.915179 |    0.784643 | 5.12378e+08 | AAPL     |
| 2000-01-05 00:00:00 | 0.926339 | 0.987165 | 0.919643 | 0.928571 |    0.796124 | 7.78322e+08 | AAPL     |
| 2000-01-06 00:00:00 | 0.947545 | 0.955357 | 0.848214 | 0.848214 |    0.727229 | 7.67973e+08 | AAPL     |
```

Columns:
- `Date`: Timestamp 
- `Open`: price of the stock when the market opened on this day
- `High`: highest price of the stock on this day
- `Low`: lowest price of the stock on this day
- `Close`: Close price adjusted for splits.
- `Adj Close`: Adjusted close price adjusted for splits and dividend and/or capital gain distributions.
- `Volume`: Volume on Yahoo Finance's charts are the physical number of shares traded of that stock (not dollar amount) for your given period of time.  
- `Ticker`: identifier

https://machinelearningmastery.com/time-series-forecasting-methods-in-python-cheat-sheet/

# Datasets: train and test

```python
import pandas as pd

data = pd.read_pickle('resources/historical_dow_stock_data.pkl')
data = data[['Date', 'Open', 'Ticker']]
data = data[data.Ticker=='AAPL'].drop_duplicates(ignore_index=True)

events_to_predict = 30
train_data = data[:-events_to_predict].copy().reset_index(drop=True)
test_data = data[-events_to_predict:].copy().reset_index(drop=True)
```

# Autoregressive model

The autoregressive model specifies that the output variable depends linearly on its own previous values and on a stochastic term (an imperfectly predictable term)

$$ X_t = c + \sum_{i=1}^{p} \varphi_i X_{t-i} + \varepsilon_t $$ 

```python
from statsmodels.tsa.ar_model import AutoReg
from augur.eval.metrics import model_metrics

# fit model
model = AutoReg(train_data.Open, lags=10)
model_fit = model.fit()
y_hat = model_fit.predict(len(train_data), len(train_data) + events_to_predict - 1)
model_metrics(test_data.Open, y_hat.tolist())
```

```python
{'mse': 313.46845218555507,
 'rmse': 17.705040304544777,
 'mae': 15.79379801617968,
 'mape': 0.09242717290908554}
```

The model is not updated with the new data available (i.e. it uses the train data (up to t-1) for predicting 
next value (t) but it doesn't add the true value at t to the train data for predicting t+1). 
For such a purpose, we have to do it manually. 

```python
window = 29
train = train_data.Open.tolist()
test = test_data.Open.tolist()
model = AutoReg(train, lags=window)
model_fit = model.fit()
coef = model_fit.params[::-1]
# walk forward over time steps in test
history = train[-window:]
history = [history[i] for i in range(len(history))]
predictions = list()
for t in range(len(test)):
    lag = history[-window:] + [1]
    yhat = (coef * lag).sum()           #  predicted value
    predictions.append(yhat)
    history.append(test[t])             #  true value added to history 
model_metrics(test, predictions)
```
Performance has improved remarkably.
```python
{'mse': 15.984527917239209,
 'rmse': 3.998065521879201,
 'mae': 3.0717770018412067,
 'mape': 0.018307969637371135}
```
**doubt**: can it be used for long term predictions?

# Moving average 

**[Moving-average model (MA model)][2]**  is a common approach for modeling univariate time series. The moving-average model 
specifies that the output variable depends linearly on the current and various past values of a stochastic 
(imperfectly predictable) term.

$$ X_t = \mu + \varepsilon_t \sum_{i=1}^{q} \theta_i \varepsilon_{t-i}$$ 

```python
# fit model
model = ARIMA(train_data.Open, order=(0, 0, 1))
# order = (p, d, q) order of the model for the autoregressive (p), differences (d), and moving average components (q).
model_fit = model.fit()
# make prediction
y_hat = model_fit.predict(len(train_data), len(train_data) + events_to_predict - 1)
model_metrics(test_data.Open, y_hat.tolist())
```

```python
{'mse': 20203.231487138346,
 'rmse': 142.13807191297605,
 'mae': 141.24901695061772,
 'mape': 0.8462173357299907}
```

# ARMA: autoregressive moving average

**[Autoregressive moving average model][3]**: It combines Autoregressive model (AR) and moving average model models (MA).

The AR part involves on its own lagged values. The MA part involves modeling the error as a linear combination of error 
terms.

$$ X_t = c + \varepsilon_t + \sum_{i=1}^{p} \varphi_i X_{t-i} + \sum_{i=1}^{q} \theta_i \varepsilon_{t-i}$$ 


```python
# fit model
model = ARIMA(train_data.Open, order=(10, 0, 1))
# order = (p, d, q) order of the model for the autoregressive (p), differences (d), and moving average components (q).
model_fit = model.fit()
# make prediction
y_hat = model_fit.predict(len(train_data), len(train_data) + events_to_predict - 1)
model_metrics(test_data.Open, y_hat.tolist())
```

```python
{'mse': 415.99645254453776,
 'rmse': 20.395991090028886,
 'mae': 18.223052286736102,
 'mape': 0.10665761934668148}
```

## How to tune p and q

We are going to run several times the ARMA model with a combination of p and q. I'm going to use RMSE as model metric to
be minimised.

I'm using a grid for `p` and `q` with values from 1 to 30 in steps of 5. After locating the minima, I will repeat the 
grid approach but zooming in this area.

I will use it later.

### According to wikipedia

Finding appropriate values of p and q in the ARMA(p,q) model can be facilitated by plotting the partial autocorrelation 
functions for an estimate of p, and likewise using the autocorrelation functions for an estimate of q. 
Extended autocorrelation functions (EACF) can be used to simultaneously determine p and q. 
Further information can be gleaned by considering the same functions for the residuals of a model fitted with an initial 
selection of p and q.

Brockwell & Davis recommend using Akaike information criterion (AIC) for finding p and q. Another possible choice for 
order determining is the BIC criterion. 

# Reference
[1]: https://en.wikipedia.org/wiki/Forecasting
[2]: https://en.wikipedia.org/wiki/Moving-average_model
[3]: https://en.wikipedia.org/wiki/Autoregressive%E2%80%93moving-average_model