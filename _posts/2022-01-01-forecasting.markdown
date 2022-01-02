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

# Reference
[1]: https://en.wikipedia.org/wiki/Forecasting