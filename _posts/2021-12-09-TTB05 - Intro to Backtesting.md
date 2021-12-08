---
layout: nbpost
title:  "Python Trading Toolbox: A gentle introduction to backtesting"
date:   2021-12-09 01:00:00 +0000
categories: trading toolbox
author: Stefano Basurto
---
We started this series by introducing some indicators based on price. Our goal is to use indicators, along with price and volume, to make investment decisions: to choose when to buy or sell a financial asset.
There are different ways we can incorporate price, volume, and indicators in our investment decision process. The first, the most traditional, is to interpret their patterns in a discretional way, as followers of [Technical Analysis](https://en.wikipedia.org/wiki/Technical_analysis) do.
Indicators can also be employed in a more quantitative approach as building blocks of a trading system that removes human discretion from the investment process. [Algorithmic Trading](https://en.wikipedia.org/wiki/Algorithmic_trading), in particular, is an approach based on trading strategies that take positions in financial instruments on their own without human intervention. We can also use price, volume, and indicators as part of a more complex machine learning model for our investment decisions.

> *(This post was originally published on **[Towards Data Science](https://towardsdatascience.com/python-trading-toolbox-05-backtesting-84266edb1d59)** on October 7, 2020)*

Whatever way we choose to use our indicators, we need to answer an important question: how good are our indicators, or combinations of indicators, to inform our investment decisions? In other words, will the use of any indicator lead to better results than not using them at all?

A process that can help us to answer this question is known as [**backtesting**](https://www.investopedia.com/terms/b/backtesting.asp). With backtesting, we apply a trading or investment strategy to historical data to generate hypothetical results. We can then analyze those results to evaluate the profitability and the risk of our strategy.

This process has its own pitfalls: there is no guarantee that a strategy that performed well on historical data will perform well in real trading. Real trading involves many factors that cannot be simulated or tested on historical data. Also, since financial markets keep evolving fast, the future may exhibit patterns not present in historical data. However, if a strategy cannot prove itself valid in a backtest most probably will never work in real trading. Backtesting can at least help us to weed out the strategies that do not prove themselves worthy.

Several frameworks make it easy to backtest trading strategies using Python. Two popular examples are [**Zipline**](https://www.zipline.io/) and [**Backtrader**](https://www.backtrader.com/). Frameworks like *Zipline* and *Backtrader* include all the tools needed to design, test, and implement an algorithmic trading strategy. They can even automate the submission of real orders to an execution broker.

With this article, we are taking a different approach: we want to investigate how to build and test a trading system from scratch using Python, *pandas*, and NumPy as our only tools. Why should we want to do that?
To start with, building a backtest from scratch is an excellent exercise and will help to understand in detail how a strategy works. Also, we may find ourselves in situations where we need to implement solutions not available in existing frameworks. Or, you may want to embark on the journey of creating your own backtesting framework!

## Backtesting our first system

We can create a barebone backtest using Python with NumPy and *pandas*. To build an example, we are going to use prices for the *Campbell Soup Company* stock (**CPB**) traded on the NYSE. I downloaded five years of trading history from [Yahoo! Finance](https://uk.finance.yahoo.com/quote/CPB): the file is available [here](https://raw.githubusercontent.com/stebas101/TradingToolbox/master/data/CPB.csv).

We start by setting up our environment and load the price series into a data frame:

<div class="prompt input_prompt">
    In [1]:
</div>

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
pd.plotting.register_matplotlib_converters()

# This is needed if you're using Jupyter to visualize charts:
%matplotlib inline

import sys
import matplotlib as mpl
import mplfinance as mpf
print(f"Python version: {sys.version}")
print(f"pandas version: {pd.__version__}")
print(f"numpy version: {np.__version__}")
print(f"mplfinanceversion: {mpf.__version__}")

datafile = 'data/CPB.csv'
data = pd.read_csv(datafile, index_col = 'Date')
data.index = pd.to_datetime(data.index) # Converting the dates from string to datetime format

data
```

<div class="prompt output_prompt">
    Out [1]:
</div>

    Python version: 3.7.7 (default, May  6 2020, 04:59:01) 
    [Clang 4.0.1 (tags/RELEASE_401/final)]
    pandas version: 1.1.0
    numpy version: 1.19.1
    mplfinanceversion: 0.12.7a0





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-09-02</th>
      <td>47.389999</td>
      <td>47.770000</td>
      <td>47.009998</td>
      <td>47.770000</td>
      <td>41.054787</td>
      <td>2547200</td>
    </tr>
    <tr>
      <th>2015-09-03</th>
      <td>46.910000</td>
      <td>48.680000</td>
      <td>46.070000</td>
      <td>48.529999</td>
      <td>41.707958</td>
      <td>2364100</td>
    </tr>
    <tr>
      <th>2015-09-04</th>
      <td>48.230000</td>
      <td>48.500000</td>
      <td>47.439999</td>
      <td>47.939999</td>
      <td>41.200890</td>
      <td>2019300</td>
    </tr>
    <tr>
      <th>2015-09-08</th>
      <td>48.500000</td>
      <td>49.410000</td>
      <td>48.500000</td>
      <td>49.380001</td>
      <td>42.438469</td>
      <td>2458700</td>
    </tr>
    <tr>
      <th>2015-09-09</th>
      <td>49.779999</td>
      <td>49.779999</td>
      <td>48.590000</td>
      <td>48.720001</td>
      <td>41.871243</td>
      <td>2198900</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.900002</td>
      <td>53.500000</td>
      <td>52.389999</td>
      <td>53.480000</td>
      <td>53.480000</td>
      <td>1375400</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.509998</td>
      <td>54.080002</td>
      <td>53.259998</td>
      <td>53.290001</td>
      <td>53.290001</td>
      <td>1432100</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.290001</td>
      <td>53.290001</td>
      <td>51.930000</td>
      <td>52.139999</td>
      <td>52.139999</td>
      <td>1827300</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.349998</td>
      <td>52.709999</td>
      <td>52.020000</td>
      <td>52.610001</td>
      <td>52.610001</td>
      <td>2018800</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.810001</td>
      <td>52.880001</td>
      <td>51.189999</td>
      <td>51.400002</td>
      <td>51.400002</td>
      <td>3060900</td>
    </tr>
  </tbody>
</table>
<p>1259 rows × 6 columns</p>
</div>



### A basic strategy

For our example, we are going to test a basic **moving average crossover system** based on a 20-day Exponential Moving Average (EMA) and a 200-day Simple Moving Average (SMA) of the daily closing price (using *Adjusted Close* in this example). We are going to buy the stock (take a *long position*) whenever the 20-day EMA crosses the 200-day SMA from below.

We add columns with our moving averages to the data frame:

<div class="prompt input_prompt">
    In [2]:
</div>

```python
df = data.copy()

sma_span = 200
ema_span = 20

df['sma200'] = df['Adj Close'].rolling(sma_span).mean()
df['ema20'] = df['Adj Close'].ewm(span=ema_span).mean()

df.round(3)
```

<div class="prompt output_prompt">
    Out [2]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>sma200</th>
      <th>ema20</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-09-02</th>
      <td>47.39</td>
      <td>47.77</td>
      <td>47.01</td>
      <td>47.77</td>
      <td>41.055</td>
      <td>2547200</td>
      <td>NaN</td>
      <td>41.055</td>
    </tr>
    <tr>
      <th>2015-09-03</th>
      <td>46.91</td>
      <td>48.68</td>
      <td>46.07</td>
      <td>48.53</td>
      <td>41.708</td>
      <td>2364100</td>
      <td>NaN</td>
      <td>41.398</td>
    </tr>
    <tr>
      <th>2015-09-04</th>
      <td>48.23</td>
      <td>48.50</td>
      <td>47.44</td>
      <td>47.94</td>
      <td>41.201</td>
      <td>2019300</td>
      <td>NaN</td>
      <td>41.325</td>
    </tr>
    <tr>
      <th>2015-09-08</th>
      <td>48.50</td>
      <td>49.41</td>
      <td>48.50</td>
      <td>49.38</td>
      <td>42.438</td>
      <td>2458700</td>
      <td>NaN</td>
      <td>41.647</td>
    </tr>
    <tr>
      <th>2015-09-09</th>
      <td>49.78</td>
      <td>49.78</td>
      <td>48.59</td>
      <td>48.72</td>
      <td>41.871</td>
      <td>2198900</td>
      <td>NaN</td>
      <td>41.701</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.90</td>
      <td>53.50</td>
      <td>52.39</td>
      <td>53.48</td>
      <td>53.480</td>
      <td>1375400</td>
      <td>48.478</td>
      <td>51.571</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.51</td>
      <td>54.08</td>
      <td>53.26</td>
      <td>53.29</td>
      <td>53.290</td>
      <td>1432100</td>
      <td>48.519</td>
      <td>51.735</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.29</td>
      <td>53.29</td>
      <td>51.93</td>
      <td>52.14</td>
      <td>52.140</td>
      <td>1827300</td>
      <td>48.553</td>
      <td>51.773</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.35</td>
      <td>52.71</td>
      <td>52.02</td>
      <td>52.61</td>
      <td>52.610</td>
      <td>2018800</td>
      <td>48.584</td>
      <td>51.853</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.81</td>
      <td>52.88</td>
      <td>51.19</td>
      <td>51.40</td>
      <td>51.400</td>
      <td>3060900</td>
      <td>48.611</td>
      <td>51.810</td>
    </tr>
  </tbody>
</table>
<p>1259 rows × 8 columns</p>
</div>



As we can see, by using a 200-day SMA we get **NaN** values in the first 199 rows for the respective column. This is just an exercise, and we can just get rid of those rows to perform our backtest. In real practice, we may consider using a different indicator to avoid losing so much data. To remove the *NaN* values:

<div class="prompt input_prompt">
    In [3]:
</div>

```python
df.dropna(inplace=True)

df.round(3)
```

<div class="prompt output_prompt">
    Out [3]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>sma200</th>
      <th>ema20</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-17</th>
      <td>62.75</td>
      <td>62.75</td>
      <td>61.87</td>
      <td>62.44</td>
      <td>54.582</td>
      <td>2064300</td>
      <td>49.113</td>
      <td>54.320</td>
    </tr>
    <tr>
      <th>2016-06-20</th>
      <td>62.63</td>
      <td>63.15</td>
      <td>62.39</td>
      <td>62.41</td>
      <td>54.556</td>
      <td>1459400</td>
      <td>49.181</td>
      <td>54.343</td>
    </tr>
    <tr>
      <th>2016-06-21</th>
      <td>62.65</td>
      <td>63.21</td>
      <td>62.49</td>
      <td>62.85</td>
      <td>54.940</td>
      <td>1161900</td>
      <td>49.247</td>
      <td>54.400</td>
    </tr>
    <tr>
      <th>2016-06-22</th>
      <td>63.08</td>
      <td>63.08</td>
      <td>62.23</td>
      <td>62.59</td>
      <td>54.713</td>
      <td>1395800</td>
      <td>49.314</td>
      <td>54.429</td>
    </tr>
    <tr>
      <th>2016-06-23</th>
      <td>62.68</td>
      <td>62.92</td>
      <td>62.27</td>
      <td>62.64</td>
      <td>54.757</td>
      <td>1177000</td>
      <td>49.376</td>
      <td>54.461</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.90</td>
      <td>53.50</td>
      <td>52.39</td>
      <td>53.48</td>
      <td>53.480</td>
      <td>1375400</td>
      <td>48.478</td>
      <td>51.571</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.51</td>
      <td>54.08</td>
      <td>53.26</td>
      <td>53.29</td>
      <td>53.290</td>
      <td>1432100</td>
      <td>48.519</td>
      <td>51.735</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.29</td>
      <td>53.29</td>
      <td>51.93</td>
      <td>52.14</td>
      <td>52.140</td>
      <td>1827300</td>
      <td>48.553</td>
      <td>51.773</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.35</td>
      <td>52.71</td>
      <td>52.02</td>
      <td>52.61</td>
      <td>52.610</td>
      <td>2018800</td>
      <td>48.584</td>
      <td>51.853</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.81</td>
      <td>52.88</td>
      <td>51.19</td>
      <td>51.40</td>
      <td>51.400</td>
      <td>3060900</td>
      <td>48.611</td>
      <td>51.810</td>
    </tr>
  </tbody>
</table>
<p>1060 rows × 8 columns</p>
</div>



Let's have a look at our data in a chart:

<div class="prompt input_prompt">
    In [4]:
</div>

```python
def plot_system1(data):
    df = data.copy()
    dates = df.index
    price = df['Adj Close']
    sma200 = df['sma200']
    ema20 = df['ema20']
    
    with plt.style.context('fivethirtyeight'):
        fig = plt.figure(figsize=(14,7))
        plt.plot(dates, price, linewidth=1.5, label='CPB price - Daily Adj Close')
        plt.plot(dates, sma200, linewidth=2, label='200 SMA')
        plt.plot(dates, ema20, linewidth=2, label='20 EMA')
        plt.title("A Simple Crossover System")
        plt.ylabel('Price($)')
        plt.legend()
    
    plt.show() # This is needed only if not in Jupyter

plot_system1(df)
```

<div class="prompt output_prompt">
    Out [4]:
</div>


![png](/assets/images/TTB05%20-%20Intro%20to%20Backtesting_9_0.png)


To keep track of our positions in the data frame we add a column that contains, for each row, the number **1** for the days when we have a long position and the number **0** for the days when we have no position:

<div class="prompt input_prompt">
    In [5]:
</div>

```python
# Our trading condition:
long_positions = np.where(df['ema20'] > df['sma200'], 1, 0)
df['Position'] = long_positions

df.round(3)
```

<div class="prompt output_prompt">
    Out [5]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>sma200</th>
      <th>ema20</th>
      <th>Position</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-17</th>
      <td>62.75</td>
      <td>62.75</td>
      <td>61.87</td>
      <td>62.44</td>
      <td>54.582</td>
      <td>2064300</td>
      <td>49.113</td>
      <td>54.320</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2016-06-20</th>
      <td>62.63</td>
      <td>63.15</td>
      <td>62.39</td>
      <td>62.41</td>
      <td>54.556</td>
      <td>1459400</td>
      <td>49.181</td>
      <td>54.343</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2016-06-21</th>
      <td>62.65</td>
      <td>63.21</td>
      <td>62.49</td>
      <td>62.85</td>
      <td>54.940</td>
      <td>1161900</td>
      <td>49.247</td>
      <td>54.400</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2016-06-22</th>
      <td>63.08</td>
      <td>63.08</td>
      <td>62.23</td>
      <td>62.59</td>
      <td>54.713</td>
      <td>1395800</td>
      <td>49.314</td>
      <td>54.429</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2016-06-23</th>
      <td>62.68</td>
      <td>62.92</td>
      <td>62.27</td>
      <td>62.64</td>
      <td>54.757</td>
      <td>1177000</td>
      <td>49.376</td>
      <td>54.461</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.90</td>
      <td>53.50</td>
      <td>52.39</td>
      <td>53.48</td>
      <td>53.480</td>
      <td>1375400</td>
      <td>48.478</td>
      <td>51.571</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.51</td>
      <td>54.08</td>
      <td>53.26</td>
      <td>53.29</td>
      <td>53.290</td>
      <td>1432100</td>
      <td>48.519</td>
      <td>51.735</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.29</td>
      <td>53.29</td>
      <td>51.93</td>
      <td>52.14</td>
      <td>52.140</td>
      <td>1827300</td>
      <td>48.553</td>
      <td>51.773</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.35</td>
      <td>52.71</td>
      <td>52.02</td>
      <td>52.61</td>
      <td>52.610</td>
      <td>2018800</td>
      <td>48.584</td>
      <td>51.853</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.81</td>
      <td>52.88</td>
      <td>51.19</td>
      <td>51.40</td>
      <td>51.400</td>
      <td>3060900</td>
      <td>48.611</td>
      <td>51.810</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>1060 rows × 9 columns</p>
</div>



Whatever rule we are trying to implement, it's a good idea to **inspect our signals** to make sure that everything works as intended. Pesky mistakes like to hide inside this kind of computations: it's way too easy to start testing a system, only to discover later that we did not implement our rules correctly. In particular, we need to be wary of introducing any form of **look-ahead bias**: this happens when we include in our trading rule data that is not actually available at the time the rule is evaluated. If a system backtest produces results that are too good to be true, look-ahead bias is the most likely culprit.

We can inspect our signals by checking the numeric variables in our data frame and by plotting the signals on a chart.

To select the days when the trading signal is triggered:

<div class="prompt input_prompt">
    In [6]:
</div>

```python
buy_signals = (df['Position'] == 1) & (df['Position'].shift(1) == 0)

df.loc[buy_signals].round(3)
```

<div class="prompt output_prompt">
    Out [6]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>sma200</th>
      <th>ema20</th>
      <th>Position</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-12-30</th>
      <td>60.93</td>
      <td>61.09</td>
      <td>60.33</td>
      <td>60.47</td>
      <td>53.802</td>
      <td>1204500</td>
      <td>52.734</td>
      <td>52.833</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2017-05-25</th>
      <td>58.45</td>
      <td>58.86</td>
      <td>57.65</td>
      <td>58.83</td>
      <td>52.975</td>
      <td>1858600</td>
      <td>51.651</td>
      <td>51.778</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2019-04-10</th>
      <td>39.19</td>
      <td>39.57</td>
      <td>38.91</td>
      <td>39.13</td>
      <td>37.678</td>
      <td>2936200</td>
      <td>35.912</td>
      <td>36.021</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



We have only three signals in this case. To make sure that we are applying the crossover trading rule correctly, we can include the days before the signals in the selection:

<div class="prompt input_prompt">
    In [7]:
</div>

```python
buy_signals_prev = (df['Position'].shift(-1) == 1) & (df['Position'] == 0)

df.loc[buy_signals | buy_signals_prev].round(3)
```

<div class="prompt output_prompt">
    Out [7]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>sma200</th>
      <th>ema20</th>
      <th>Position</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-12-29</th>
      <td>60.56</td>
      <td>61.02</td>
      <td>60.48</td>
      <td>60.95</td>
      <td>54.229</td>
      <td>940200</td>
      <td>52.745</td>
      <td>52.731</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2016-12-30</th>
      <td>60.93</td>
      <td>61.09</td>
      <td>60.33</td>
      <td>60.47</td>
      <td>53.802</td>
      <td>1204500</td>
      <td>52.734</td>
      <td>52.833</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2017-05-24</th>
      <td>58.16</td>
      <td>58.43</td>
      <td>57.93</td>
      <td>58.38</td>
      <td>52.570</td>
      <td>2204000</td>
      <td>51.655</td>
      <td>51.652</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2017-05-25</th>
      <td>58.45</td>
      <td>58.86</td>
      <td>57.65</td>
      <td>58.83</td>
      <td>52.975</td>
      <td>1858600</td>
      <td>51.651</td>
      <td>51.778</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2019-04-09</th>
      <td>38.46</td>
      <td>39.81</td>
      <td>38.46</td>
      <td>39.49</td>
      <td>37.688</td>
      <td>5173300</td>
      <td>35.903</td>
      <td>35.846</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2019-04-10</th>
      <td>39.19</td>
      <td>39.57</td>
      <td>38.91</td>
      <td>39.13</td>
      <td>37.678</td>
      <td>2936200</td>
      <td>35.912</td>
      <td>36.021</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Everything looks good so far: on the days before the signal, `ema20` is below `sma200`, and it crosses above on the signal days. We can do a similar inspection for the signals to exit our positions: I leave this exercise to you.

We can plot markers for our signals on a chart:

<div class="prompt input_prompt">
    In [8]:
</div>

```python
def plot_system1_sig(data):
    df = data.copy()
    dates = df.index
    price = df['Adj Close']
    sma200 = df['sma200']
    ema20 = df['ema20']
    
    buy_signals = (df['Position'] == 1) & (df['Position'].shift(1) == 0)
    buy_marker = sma200 * buy_signals - (sma200.max()*.05)
    buy_marker = buy_marker[buy_signals]
    buy_dates = df.index[buy_signals]
    sell_signals = (df['Position'] == 0) & (df['Position'].shift(1) == 1)
    sell_marker = sma200 * sell_signals + (sma200.max()*.05)
    sell_marker = sell_marker[sell_signals]
    sell_dates = df.index[sell_signals]
    
    with plt.style.context('fivethirtyeight'):
        fig = plt.figure(figsize=(14,7))
        plt.plot(dates, price, linewidth=1.5, label='CPB price - Daily Adj Close')
        plt.plot(dates, sma200, linewidth=2, label='200 SMA')
        plt.plot(dates, ema20, linewidth=2, label='20 EMA')
        plt.scatter(buy_dates, buy_marker, marker='^', color='green', s=160, label='Buy')
        plt.scatter(sell_dates, sell_marker, marker='v', color='red', s=160, label='Sell')
        plt.title("A Simple Crossover System with Signals")
        plt.ylabel('Price($)')
        plt.legend()
    
    plt.show() # This is needed only if not in Jupyter

plot_system1_sig(df)
```

<div class="prompt output_prompt">
    Out [8]:
</div>


![png](/assets/images/TTB05%20-%20Intro%20to%20Backtesting_17_0.png)


From the chart, we can see that there is a mismatch between Buy and Sell signals. The first signal is a Sell (without a Buy) since we start from a long position at the beginning of the series. The last signal is a Buy (without a Sell) because we keep a long position at the end of the series.

### Strategy returns

We can now compute the return for our strategy as a percentage of the initial investment and compare it to the returns of a *Buy and Hold* strategy that simply buys our stock at the beginning of the period and holds it until the end.

The price series that we are going to use to calculate the returns is **Adjusted Close**: by using an adjusted price we make sure that the effects of dividends, stock splits, and other [corporate actions](https://www.investopedia.com/articles/03/081303.asp) on returns are taken into account in our calculation.

<div class="prompt input_prompt">
    In [9]:
</div>

```python
# The returns of the Buy and Hold strategy:
df['Hold'] = np.log(df['Adj Close'] / df['Adj Close'].shift(1))

# The returns of the Moving Average strategy:
df['Strategy'] = df['Position'].shift(1) * df['Hold']

# We need to get rid of the NaN generated in the first row:
df.dropna(inplace=True)

df
```

<div class="prompt output_prompt">
    Out [9]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>sma200</th>
      <th>ema20</th>
      <th>Position</th>
      <th>Hold</th>
      <th>Strategy</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-20</th>
      <td>62.630001</td>
      <td>63.150002</td>
      <td>62.389999</td>
      <td>62.410000</td>
      <td>54.555550</td>
      <td>1459400</td>
      <td>49.180615</td>
      <td>54.342710</td>
      <td>1</td>
      <td>-0.000481</td>
      <td>-0.000481</td>
    </tr>
    <tr>
      <th>2016-06-21</th>
      <td>62.650002</td>
      <td>63.209999</td>
      <td>62.490002</td>
      <td>62.849998</td>
      <td>54.940170</td>
      <td>1161900</td>
      <td>49.246776</td>
      <td>54.399611</td>
      <td>1</td>
      <td>0.007025</td>
      <td>0.007025</td>
    </tr>
    <tr>
      <th>2016-06-22</th>
      <td>63.080002</td>
      <td>63.080002</td>
      <td>62.230000</td>
      <td>62.590000</td>
      <td>54.712894</td>
      <td>1395800</td>
      <td>49.314336</td>
      <td>54.429448</td>
      <td>1</td>
      <td>-0.004145</td>
      <td>-0.004145</td>
    </tr>
    <tr>
      <th>2016-06-23</th>
      <td>62.680000</td>
      <td>62.919998</td>
      <td>62.270000</td>
      <td>62.639999</td>
      <td>54.756611</td>
      <td>1177000</td>
      <td>49.375926</td>
      <td>54.460606</td>
      <td>1</td>
      <td>0.000799</td>
      <td>0.000799</td>
    </tr>
    <tr>
      <th>2016-06-24</th>
      <td>61.450001</td>
      <td>63.490002</td>
      <td>61.049999</td>
      <td>62.400002</td>
      <td>54.546814</td>
      <td>3845400</td>
      <td>49.439304</td>
      <td>54.468816</td>
      <td>1</td>
      <td>-0.003839</td>
      <td>-0.003839</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.900002</td>
      <td>53.500000</td>
      <td>52.389999</td>
      <td>53.480000</td>
      <td>53.480000</td>
      <td>1375400</td>
      <td>48.478148</td>
      <td>51.571117</td>
      <td>1</td>
      <td>0.008450</td>
      <td>0.008450</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.509998</td>
      <td>54.080002</td>
      <td>53.259998</td>
      <td>53.290001</td>
      <td>53.290001</td>
      <td>1432100</td>
      <td>48.519216</td>
      <td>51.734820</td>
      <td>1</td>
      <td>-0.003559</td>
      <td>-0.003559</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.290001</td>
      <td>53.290001</td>
      <td>51.930000</td>
      <td>52.139999</td>
      <td>52.139999</td>
      <td>1827300</td>
      <td>48.553018</td>
      <td>51.773409</td>
      <td>1</td>
      <td>-0.021816</td>
      <td>-0.021816</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.349998</td>
      <td>52.709999</td>
      <td>52.020000</td>
      <td>52.610001</td>
      <td>52.610001</td>
      <td>2018800</td>
      <td>48.584327</td>
      <td>51.853084</td>
      <td>1</td>
      <td>0.008974</td>
      <td>0.008974</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.810001</td>
      <td>52.880001</td>
      <td>51.189999</td>
      <td>51.400002</td>
      <td>51.400002</td>
      <td>3060900</td>
      <td>48.611054</td>
      <td>51.809934</td>
      <td>1</td>
      <td>-0.023268</td>
      <td>-0.023268</td>
    </tr>
  </tbody>
</table>
<p>1059 rows × 11 columns</p>
</div>



The returns for the whole period are simply a sum of the daily log returns:

<div class="prompt input_prompt">
    In [10]:
</div>

```python
returns = np.exp(df[['Hold', 'Strategy']].sum()) - 1

print(f"Buy and hold return: {round(returns['Hold']*100,2)}%")
print(f"Strategy return: {round(returns['Strategy']*100,2)}%")
```

<div class="prompt output_prompt">
    Out [10]:
</div>

    Buy and hold return: -5.83%
    Strategy return: 10.3%


Those returns are related to a period of 1060 days. If we want to compare them to returns from other periods we need to **annualize** them:

<div class="prompt input_prompt">
    In [11]:
</div>

```python
n_days = len(df)

# Assuming 252 trading days in a year:
ann_returns = 252 / n_days * returns

print(f"Buy and hold annualized return: {round(ann_returns['Hold']*100,2)}%")
print(f"Strategy annualized return: {round(ann_returns['Strategy']*100,2)}%")
```

<div class="prompt output_prompt">
    Out [11]:
</div>

    Buy and hold annualized return: -1.39%
    Strategy annualized return: 2.45%


So far, our simple strategy seems to do better than to *buy and hold*: this particular stock had a negative trend over the period, while the simple MA crossover strategy gave us a return in excess of 10%. I have actually chosen that stock series on purpose: it has a bearish trend at the beginning of the period before turning bullish. Do not expect to be always so lucky with the results!

Unless you are familiar with log returns, you may wonder why and how we used **logarithms** in the return calculations. Here is a bit of maths to explain that in case that sounds all new to you. Otherwise, feel free to skip to the next section.

In quantitative finance it's very common to use logarithms to calculate returns: they make some computations easier to handle. If the daily return **$R_t$** is defined as:

$$R_t = \frac{P_t - P_{t-1}}{P_{t-1}}$$

where $P_t$ is the price at date $t$, the logarithmic return $r_t$ is defined as:

$$r_t = \ln (1 + R_t)$$

By applying some basic algebra, it's possible to compute the daily log return as:

$$r_t = \ln \frac{P_t}{P_{t-1}} =  \ln P_t - \ln P_{t-1}$$

Why are log-returns handy? If we have a series of daily returns and we need to calculate the return for the whole period, with log returns we can make a sum of them By contrast, with regular returns, we need to a multiplication:

$$\text{Return over the period} =  (1+R_1)\times(1+R_2)\times\dots\times(1+R_T) = \exp \left( \sum_{t=0}^{T} r_t \right)$$

where *T* is the number of days in the period of time that we are considering. Calculating the annualized returns comes easier as well.

## A more complex strategy

The strategy we have just tested had only two possible positions: we were either *long* (holding the stock) or *flat* (holding just cash). It would be interesting to try and test a strategy that adds the possibility of a *short* position as well (selling borrowed shares and buying them back when exiting the position). To build an example of this strategy we include two simple moving averages, one for the daily highs, and one for the daily lows. We also add a 15-day exponential moving average.
We take positions based on the following rules:
- when the EMA is above the higher SMA (plus a 2% offset), we take a long position (buy)
- when the EMA is below the lower SMA (minus a 2% offset), we take a short position (sell short)
- in all other situations (EMA between the to SMAs), we keep out of the market

I added the offset to the SMAs to reduce the numbers of false signals. Let's prepare a new data frame:

<div class="prompt input_prompt">
    In [12]:
</div>

```python
df2 = data.copy()

sma_span = 40
ema_span = 15

df2['H_sma'] = df2['High'].rolling(sma_span).mean()
df2['L_sma'] = df2['Low'].rolling(sma_span).mean()
df2['C_ema'] = df2['Close'].ewm(span=ema_span).mean()

df2.dropna(inplace=True)

df2.round(3)
```

<div class="prompt output_prompt">
    Out [12]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>H_sma</th>
      <th>L_sma</th>
      <th>C_ema</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-10-28</th>
      <td>51.27</td>
      <td>51.38</td>
      <td>50.33</td>
      <td>50.79</td>
      <td>43.917</td>
      <td>1368900</td>
      <td>50.745</td>
      <td>49.844</td>
      <td>50.660</td>
    </tr>
    <tr>
      <th>2015-10-29</th>
      <td>50.69</td>
      <td>51.37</td>
      <td>50.38</td>
      <td>51.20</td>
      <td>44.272</td>
      <td>1035700</td>
      <td>50.835</td>
      <td>49.928</td>
      <td>50.728</td>
    </tr>
    <tr>
      <th>2015-10-30</th>
      <td>51.11</td>
      <td>51.45</td>
      <td>50.78</td>
      <td>50.79</td>
      <td>43.917</td>
      <td>1184300</td>
      <td>50.904</td>
      <td>50.046</td>
      <td>50.735</td>
    </tr>
    <tr>
      <th>2015-11-02</th>
      <td>50.88</td>
      <td>50.99</td>
      <td>50.36</td>
      <td>50.74</td>
      <td>43.874</td>
      <td>1169700</td>
      <td>50.966</td>
      <td>50.119</td>
      <td>50.736</td>
    </tr>
    <tr>
      <th>2015-11-03</th>
      <td>50.50</td>
      <td>50.75</td>
      <td>49.70</td>
      <td>50.23</td>
      <td>43.433</td>
      <td>1278200</td>
      <td>51.000</td>
      <td>50.149</td>
      <td>50.673</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.90</td>
      <td>53.50</td>
      <td>52.39</td>
      <td>53.48</td>
      <td>53.480</td>
      <td>1375400</td>
      <td>50.791</td>
      <td>49.927</td>
      <td>51.923</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.51</td>
      <td>54.08</td>
      <td>53.26</td>
      <td>53.29</td>
      <td>53.290</td>
      <td>1432100</td>
      <td>50.897</td>
      <td>50.029</td>
      <td>52.094</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.29</td>
      <td>53.29</td>
      <td>51.93</td>
      <td>52.14</td>
      <td>52.140</td>
      <td>1827300</td>
      <td>50.983</td>
      <td>50.100</td>
      <td>52.100</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.35</td>
      <td>52.71</td>
      <td>52.02</td>
      <td>52.61</td>
      <td>52.610</td>
      <td>2018800</td>
      <td>51.054</td>
      <td>50.175</td>
      <td>52.163</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.81</td>
      <td>52.88</td>
      <td>51.19</td>
      <td>51.40</td>
      <td>51.400</td>
      <td>3060900</td>
      <td>51.130</td>
      <td>50.229</td>
      <td>52.068</td>
    </tr>
  </tbody>
</table>
<p>1220 rows × 9 columns</p>
</div>



Here we are making use of High and Low prices in addition to Close prices. To plot those values on a chart, it's a good idea to use [**OHLC bars** or **candlesticks**](https://towardsdatascience.com/trading-toolbox-03-ohlc-charts-95b48bb9d748). We are going to use the **mplfinance** library for this. If you have not done that yet, you can easily install *mplfinance* using:

```pip install --upgrade mplfinance```

To integrate the candlestick chart with our existing style, I am going to apply the [*External Axes Method*](https://github.com/matplotlib/mplfinance/blob/master/markdown/subplots.md#external-axes-method) of *mplfinance*:

<div class="prompt input_prompt">
    In [13]:
</div>

```python
import mplfinance as mpf
import numpy as np

def plot_system2(data):
    df2 = data.copy()
    dates = np.arange(len(df2)) # We need this for mpl.plot()
    price = df2['Adj Close']
    h_sma = df2['H_sma']*1.02
    l_sma = df2['L_sma']*.98
    c_ema = df2['C_ema']
    
    with plt.style.context('fivethirtyeight'):
        fig = plt.figure(figsize=(14,7))
        ax = plt.subplot(1,1,1)
        mpf.plot(df2, ax=ax, show_nontrading=False, type='candle')
        ax.plot(dates, h_sma, linewidth=2, color='blue', label='High 40 SMA + 2%')
        ax.plot(dates, l_sma, linewidth=2, color='blue', label='Low 40 SMA - 2%')
        ax.plot(dates, c_ema, linewidth=1.5, color='red', linestyle='--', label='Close 15 EMA')
        plt.title("A System with Long-Short Positions")
        ax.set_ylabel('Price($)')
        plt.legend()
    
    plt.show() # This is needed outside of Jupyter

plot_system2(df2)
```

<div class="prompt output_prompt">
    Out [13]:
</div>


![png](/assets/images/TTB05%20-%20Intro%20to%20Backtesting_30_0.png)


We can examine more in detail any specific range of dates:

<div class="prompt input_prompt">
    In [14]:
</div>

```python
plot_system2(df2['2019-07-01':'2019-12-31'])
```

<div class="prompt output_prompt">
    Out [14]:
</div>


![png](/assets/images/TTB05%20-%20Intro%20to%20Backtesting_32_0.png)


We then apply our trading rule and add the position columns:

<div class="prompt input_prompt">
    In [15]:
</div>

```python
offset = 0.02
long_positions = np.where(df2['C_ema'] > df2['H_sma']*(1+offset), 1, 0)
short_positions = np.where(df2['C_ema'] < df2['L_sma']*(1-offset), -1, 0)
df2['Position'] = long_positions + short_positions

df2.round(3)
```

<div class="prompt output_prompt">
    Out [15]:
</div>




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
      <th>H_sma</th>
      <th>L_sma</th>
      <th>C_ema</th>
      <th>Position</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-10-28</th>
      <td>51.27</td>
      <td>51.38</td>
      <td>50.33</td>
      <td>50.79</td>
      <td>43.917</td>
      <td>1368900</td>
      <td>50.745</td>
      <td>49.844</td>
      <td>50.660</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2015-10-29</th>
      <td>50.69</td>
      <td>51.37</td>
      <td>50.38</td>
      <td>51.20</td>
      <td>44.272</td>
      <td>1035700</td>
      <td>50.835</td>
      <td>49.928</td>
      <td>50.728</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2015-10-30</th>
      <td>51.11</td>
      <td>51.45</td>
      <td>50.78</td>
      <td>50.79</td>
      <td>43.917</td>
      <td>1184300</td>
      <td>50.904</td>
      <td>50.046</td>
      <td>50.735</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2015-11-02</th>
      <td>50.88</td>
      <td>50.99</td>
      <td>50.36</td>
      <td>50.74</td>
      <td>43.874</td>
      <td>1169700</td>
      <td>50.966</td>
      <td>50.119</td>
      <td>50.736</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2015-11-03</th>
      <td>50.50</td>
      <td>50.75</td>
      <td>49.70</td>
      <td>50.23</td>
      <td>43.433</td>
      <td>1278200</td>
      <td>51.000</td>
      <td>50.149</td>
      <td>50.673</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2020-08-26</th>
      <td>52.90</td>
      <td>53.50</td>
      <td>52.39</td>
      <td>53.48</td>
      <td>53.480</td>
      <td>1375400</td>
      <td>50.791</td>
      <td>49.927</td>
      <td>51.923</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-08-27</th>
      <td>53.51</td>
      <td>54.08</td>
      <td>53.26</td>
      <td>53.29</td>
      <td>53.290</td>
      <td>1432100</td>
      <td>50.897</td>
      <td>50.029</td>
      <td>52.094</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-08-28</th>
      <td>53.29</td>
      <td>53.29</td>
      <td>51.93</td>
      <td>52.14</td>
      <td>52.140</td>
      <td>1827300</td>
      <td>50.983</td>
      <td>50.100</td>
      <td>52.100</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-08-31</th>
      <td>52.35</td>
      <td>52.71</td>
      <td>52.02</td>
      <td>52.61</td>
      <td>52.610</td>
      <td>2018800</td>
      <td>51.054</td>
      <td>50.175</td>
      <td>52.163</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2020-09-01</th>
      <td>52.81</td>
      <td>52.88</td>
      <td>51.19</td>
      <td>51.40</td>
      <td>51.400</td>
      <td>3060900</td>
      <td>51.130</td>
      <td>50.229</td>
      <td>52.068</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>1220 rows × 10 columns</p>
</div>



We can plot our signals on the chart:

<div class="prompt input_prompt">
    In [16]:
</div>

```python
def plot_system2_sig(data):
    df2 = data.copy()
    dates = np.arange(len(df2)) # We need this for mpl.plot()
    price = df2['Adj Close']
    h_sma = df2['H_sma']*1.02
    l_sma = df2['L_sma']*.98
    c_ema = df2['C_ema']
    
    def reindex_signals(signals, markers):
        '''
        - takes two pd.Series (boolean, float)
        - returns signals and markers reindexed to an integer range, and their index
        '''
        signals.reset_index(drop=True, inplace=True)
        signals = signals[signals==True]
        dates = signals.index
        markers = markers[dates]
        markers.index = dates
        return signals, markers, dates
    
    buy_signals = (df2['Position'] == 1) & (df2['Position'].shift(1) != 1)
    buy_marker = h_sma * buy_signals[buy_signals==True] - (h_sma.max()*.04)
    buy_signals, buy_marker, buy_dates = reindex_signals(buy_signals, buy_marker)
    
    exit_buy_signals = (df2['Position'] != 1) & (df2['Position'].shift(1) == 1)
    exit_buy_marker = h_sma * exit_buy_signals + (h_sma.max()*.04)
    exit_buy_signals, exit_buy_marker, exit_buy_dates = reindex_signals(exit_buy_signals, exit_buy_marker)
    
    sell_signals = (df2['Position'] == -1) & (df2['Position'].shift(1) != -1)
    sell_marker = l_sma * sell_signals + (l_sma.max()*.04)
    sell_signals, sell_marker, sell_dates = reindex_signals(sell_signals, sell_marker)
    
    exit_sell_signals = (df2['Position'] != -1) & (df2['Position'].shift(1) == -1)
    exit_sell_marker = l_sma * exit_sell_signals - (l_sma.max()*.04)
    exit_sell_signals, exit_sell_marker, exit_sell_dates = reindex_signals(exit_sell_signals, exit_sell_marker)
    
    with plt.style.context('fivethirtyeight'):
        fig = plt.figure(figsize=(14,7))
        ax = plt.subplot(1,1,1)
        mpf.plot(df2, ax=ax, show_nontrading=False, type='candle')
        ax.plot(dates, h_sma, linewidth=2, color='blue', label='High 40 SMA + 2%')
        ax.plot(dates, l_sma, linewidth=2, color='blue', label='Low 40 SMA - 2%')
        ax.plot(dates, c_ema, linewidth=1.5, color='red', linestyle='--', label='Close 15 EMA')
        ax.scatter(buy_dates, buy_marker, marker='^', color='green', s=160, label='Buy')
        ax.scatter(exit_buy_dates, exit_buy_marker, marker='v', s=160, label='Exit Buy')
        ax.scatter(sell_dates, sell_marker, marker='v', color='red', s=160, label='Sell')
        ax.scatter(exit_sell_dates, exit_sell_marker, marker='^', color='orange', s=160, label='Exit Sell')
        plt.title("A System with Long-Short Signals")
        ax.set_ylabel('Price($)')
        plt.legend()
    
    plt.show() # This is needed outside of Jupyter

plot_system2_sig(df2)
```

<div class="prompt output_prompt">
    Out [16]:
</div>


![png](/assets/images/TTB05%20-%20Intro%20to%20Backtesting_36_0.png)


This system has more signals than the previous one and the chart looks quite crowded. We can take a look at any date range in detail:

<div class="prompt input_prompt">
    In [17]:
</div>

```python
plot_system2_sig(df2['2018-12-01':'2019-05-30'])
```

<div class="prompt output_prompt">
    Out [17]:
</div>


![png](/assets/images/TTB05%20-%20Intro%20to%20Backtesting_38_0.png)


We apply the same calculation as before to obtain the strategy's return:

<div class="prompt input_prompt">
    In [18]:
</div>

```python
# The returns of the Buy and Hold strategy:
df2['Hold'] = np.log(df2['Adj Close'] / df2['Adj Close'].shift(1))

# The returns of the Moving Average strategy:
df2['Strategy'] = df2['Position'].shift(1) * df2['Hold']

# We need to get rid again of the NaN generated in the first row:
df2.dropna(inplace=True)

returns2 = np.exp(df2[['Hold', 'Strategy']].sum()) -1

print(f"Buy and hold return: {round(returns2['Hold']*100,2)}%")
print(f"Strategy return: {round(returns2['Strategy']*100,2)}%")
```

<div class="prompt output_prompt">
    Out [18]:
</div>

    Buy and hold return: 17.04%
    Strategy return: -5.25%


As before, we can annualize the returns:

<div class="prompt input_prompt">
    In [19]:
</div>

```python
n_days2 = len(df2)

# Assuming 252 trading days in a year:
ann_returns2 = 252 / n_days2 * returns2

print(f"Buy and hold annualized return: {round(ann_returns2['Hold']*100,2)}%")
print(f"Strategy annualized return: {round(ann_returns2['Strategy']*100,2)}%")
```

<div class="prompt output_prompt">
    Out [19]:
</div>

    Buy and hold annualized return: 3.52%
    Strategy annualized return: -1.09%


In this case, our strategy is actually worse better than the *buy and hold* strategy.

You might have noted that I used the **unadjusted price series** to evaluate the signals, while I kept using adjusted prices to calculate returns. Evaluating signals using non-adjusted prices has the risk to introduce false triggers whenever dividends, splits, or other corporate actions create a gap in the prices. Here, I just used a price series that is commonly available and that everyone can download for free. If all we have is unadjusted prices, we should correct our backtest using all the information available about corporate actions.  

## Conclusion
Is that all we need to perform backtests and to select strategies we can rely on? Definitely not: in our backtests, we made (although implicitly) some assumptions and simplifications that can substantially affect our results.
To start with, we assumed that a stock can be bought at exactly the closing price of the day the signal is triggered. This is, in reality,  not guaranteed: the actual price will be somewhere in the range of the day after the signal occurred. Then, **transaction costs** have to be included. For example:
- Brokerage **fees** are paid to execute and clear our orders.
- The **spread** between Bid and Ask price is a cost component.
- If we buy on leverage we need to pay **interest**. Similarly, if we borrow shares to sell short we need to pay interest on that loan.

Some of those factors are more evident than others to understand and include in the model.

When we want to evaluate the performance of a system and compare it to that of other systems, the return over a given period is only one of the many **performance and risk metrics** that we want to consider. Some examples are:

- Percentage of *Winning Trades vs Losing Trades*.
- Maximum [Drawdown](https://www.investopedia.com/terms/d/drawdown.asp): the difference between the high point in our cumulated profits and the low point.
- *Standard Deviation* and *Sharpe Ratio* of the returns.
- [Risk/Reward ratio](https://www.investopedia.com/terms/r/riskrewardratio.asp), this is the prospective return we can earn from a trade for each dollar invested.

The barebone backtests we have just introduced provides the starting point to calculate all those metrics and to build a more realistic system.

For all those reasons, unless we want to build a complete system from scratch, if we need to backtest a strategy realistically the best option is most probably to use a complete solution like *Zipline* or *Backtrader*. However, I still keep learning a lot about indicators and strategies when writing a backtest from scratch, and this is an exercise I definitely recommend.

