---
layout: post
title: Notebook style MD test
date: 2019-11-13 21:50:28 +0000
categories: jekyll update
---

<div class=mytestclass>{{ "Test phrase" }}</div>

| Date | Price | SMA 5 | Prices Included |
|:----:|:-----:|:-----:|:---------------:|
| 1    | 1     | NaN   | -               |
| 2    | 7     | NaN   | -               |
| 3    | 2     | NaN   | -               |
| 4    | 4     | NaN   | -               |
| 5    | 3     | 3.4   | 1, 7, 2, 4, 3     |
| 6    | 2     | 3.6   | 7, 2, 4, 3, 2     |
| 7    | 4     | 3.0   | 2, 4, 3, 2, 4     |
| 8    | 8     | 4.2   | 4, 3, 2, 4, 8     |
| 9    | 1     | 3.6   | 3, 2, 4, 8, 1     |
| 10   | 8     | 4.6   | 2, 4, 8, 1, 8     |




<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[1]:
  </div>
  <div class="input_area" markdown="1">
    
```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
import sys

print('Python version: ' + sys.version)
print('pandas version: ' + pd.__version__)
print('matplotlib version: ' + mpl.__version__)
```

  </div>
</div>
  
  {:.output_stream}</p>

<pre><code>  Python version: 3.7.4 (default, Aug 13 2019, 15:17:50) 
[Clang 4.0.1 (tags/RELEASE_401/final)]
pandas version: 0.25.1
matplotlib version: 3.1.1
</code></pre>
<p>
In **Jupyter**, it's always a good idea to display the chart within the notebook:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[2]:
  </div>
  <div class="input_area" markdown="1">
    
```python
%matplotlib inline
```

  </div>
</div>
  
If instead you are executing your in the prompt on through a script editor, you will need to add `plt.show()` to the code each time we are plotting a chart in order to make it visible.

Next, we load data into a DataFrame. I have obtained a CSV file of daily data for SPY from [Yahoo! Finance]. You can download [my csv file here]. 

[Yahoo! Finance]: https://uk.finance.yahoo.com/quote/SPY/history?p=SPY
[my csv file here]: https://raw.githubusercontent.com/stebas101/TradingToolbox/master/data/SPY.csv

<div markdown=0>
  <div class="input">
    <div class="prompt input_prompt">
      In&nbsp;[3]:
    </div>
  </div>
</div>
    
```python
datafile = 'data/SPY.csv'
data = pd.read_csv(datafile, index_col = 'Date')
data
```

  </div>
</div>
  



  <div markdown="0">
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
      <td>2014-08-20</td>
      <td>198.119995</td>
      <td>199.160004</td>
      <td>198.080002</td>
      <td>198.919998</td>
      <td>180.141846</td>
      <td>72763000</td>
    </tr>
    <tr>
      <td>2014-08-21</td>
      <td>199.089996</td>
      <td>199.759995</td>
      <td>198.929993</td>
      <td>199.500000</td>
      <td>180.667160</td>
      <td>67791000</td>
    </tr>
    <tr>
      <td>2014-08-22</td>
      <td>199.339996</td>
      <td>199.690002</td>
      <td>198.740005</td>
      <td>199.190002</td>
      <td>180.386368</td>
      <td>76107000</td>
    </tr>
    <tr>
      <td>2014-08-25</td>
      <td>200.139999</td>
      <td>200.589996</td>
      <td>199.149994</td>
      <td>200.199997</td>
      <td>181.301010</td>
      <td>63855000</td>
    </tr>
    <tr>
      <td>2014-08-26</td>
      <td>200.330002</td>
      <td>200.820007</td>
      <td>200.279999</td>
      <td>200.330002</td>
      <td>181.418716</td>
      <td>47298000</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>2019-08-13</td>
      <td>287.739990</td>
      <td>294.149994</td>
      <td>287.359985</td>
      <td>292.549988</td>
      <td>292.549988</td>
      <td>94299800</td>
    </tr>
    <tr>
      <td>2019-08-14</td>
      <td>288.070007</td>
      <td>288.739990</td>
      <td>283.760010</td>
      <td>283.899994</td>
      <td>283.899994</td>
      <td>135622100</td>
    </tr>
    <tr>
      <td>2019-08-15</td>
      <td>284.880005</td>
      <td>285.640015</td>
      <td>282.390015</td>
      <td>284.649994</td>
      <td>284.649994</td>
      <td>99556600</td>
    </tr>
    <tr>
      <td>2019-08-16</td>
      <td>286.480011</td>
      <td>289.329987</td>
      <td>284.709991</td>
      <td>288.850006</td>
      <td>288.850006</td>
      <td>83018300</td>
    </tr>
    <tr>
      <td>2019-08-19</td>
      <td>292.190002</td>
      <td>293.079987</td>
      <td>291.440002</td>
      <td>292.329987</td>
      <td>292.329987</td>
      <td>53571800</td>
    </tr>
  </tbody>
</table>
<p>1258 rows × 6 columns</p>
</div>
  </div>
  


As we can see, our data frame has six columns in total, one for Open, High, Low, Close, Adjusted Close prices and Volume respectively. To keep things simple, in our examples we will use only the Adjusted Close prices. That is the price series that reflects dividends, stock splits and other corporate events that affect stock returns.


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[4]:
  </div>
  <div class="input_area" markdown="1">
    
```python
close = data['Adj Close']
close.index = pd.to_datetime(close.index) # This converts the index to pandas datetime
close
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  Date
2014-08-20    180.141846
2014-08-21    180.667160
2014-08-22    180.386368
2014-08-25    181.301010
2014-08-26    181.418716
                 ...    
2019-08-13    292.549988
2019-08-14    283.899994
2019-08-15    284.649994
2019-08-16    288.850006
2019-08-19    292.329987
Name: Adj Close, Length: 1258, dtype: float64</code></pre>
<p>


We can easily plot the price series for visual inspection:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[5]:
  </div>
  <div class="input_area" markdown="1">
    
```python
close.plot()
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  <matplotlib.axes._subplots.AxesSubplot at 0x114933790></code></pre>
<p>



![png](TTB01%20-%20Simple%20Moving%20Average_files/TTB01%20-%20Simple%20Moving%20Average_12_1.png)


Pandas make calculating a 50-day moving average easy. Using the __rolling()__ method we set a 50-day window, on which we calculate the arithmetic average (mean) using the __mean()__ method: 


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[6]:
  </div>
  <div class="input_area" markdown="1">
    
```python
sma50 = close.rolling(window=50).mean()
sma50
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  Date
2014-08-20           NaN
2014-08-21           NaN
2014-08-22           NaN
2014-08-25           NaN
2014-08-26           NaN
                 ...    
2019-08-13    293.540820
2019-08-14    293.635375
2019-08-15    293.696567
2019-08-16    293.805137
2019-08-19    293.926582
Name: Adj Close, Length: 1258, dtype: float64</code></pre>
<p>


As we expect, the first 49 values of the series are empty:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[7]:
  </div>
  <div class="input_area" markdown="1">
    
```python
sma50.iloc[45:52]
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  Date
2014-10-23           NaN
2014-10-24           NaN
2014-10-27           NaN
2014-10-28           NaN
2014-10-29    178.725250
2014-10-30    178.750461
2014-10-31    178.806655
Name: Adj Close, dtype: float64</code></pre>
<p>


We can now plot our first moving average on a chart. To improve the appearance, we can use a predefined style:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[8]:
  </div>
  <div class="input_area" markdown="1">
    
```python
plt.style.use('fivethirtyeight')
```

  </div>
</div>
  
You can experiment with different styles, check out what is available [here].

We can now plot our chart:

[here]: https://tonysyu.github.io/raw_content/matplotlib-style-gallery/gallery.html


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[9]:
  </div>
  <div class="input_area" markdown="1">
    
```python
plt.figure(figsize = (12,6))
plt.plot(close, label='SPY Adj Close', linewidth = 2)
plt.plot(sma50, label='50 day rolling SMA', linewidth = 1.5)
plt.xlabel('Date')
plt.ylabel('Adjusted closing price ($)')
plt.title('Price with a single Simple Moving Average')
plt.legend()
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  <matplotlib.legend.Legend at 0x115dfbbd0></code></pre>
<p>



![png](TTB01%20-%20Simple%20Moving%20Average_files/TTB01%20-%20Simple%20Moving%20Average_20_1.png)


### Use of one moving average

Used on its own, our simple moving average does a good job by smoothing the price movements and helping us to visually identify the trend: when the average line is climbing up, we have an uptrend. When the average points down, we are in a downtrend.

We can do more than that: it is possible to generate trading signals by using one single moving average. When the closing price moves above the moving average from below, we have a buy signal:

![buy signal](images/fig01.png)

Similarly, when the price crosses the moving average from above, a sell signal is generated:

![sell signal](images/fig02.png)

### Selecting date ranges

Let's compare  two moving averages with different length, 20 and 50-day respectively:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[10]:
  </div>
  <div class="input_area" markdown="1">
    
```python
sma20 = close.rolling(window=20).mean()

plt.figure(figsize = (12,6))
plt.plot(close, label='SPY Adj Close', linewidth = 2)
plt.plot(sma20, label='20 day rolling SMA', linewidth = 1.5)
plt.plot(sma50, label='50 day rolling SMA', linewidth = 1.5)
plt.xlabel('Date')
plt.ylabel('Adjusted closing price ($)')
plt.title('Price with Two Simple Moving Averages')
plt.legend()
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  <matplotlib.legend.Legend at 0x104522250></code></pre>
<p>



![png](TTB01%20-%20Simple%20Moving%20Average_files/TTB01%20-%20Simple%20Moving%20Average_22_1.png)


Our chart is now getting a bit crowded. It would be nice to be able to zoom on a date range of our choice. We could use a __plt.xlim()__ instruction (e.g. `plt.xlim('2017-01-01','2018-12-31')`: just try to add it to the code above). However, I want to explore a different route: building a new dataframe that includes price and moving averages:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[11]:
  </div>
  <div class="input_area" markdown="1">
    
```python
priceSma_df = pd.DataFrame({
      'Adj Close' : close,
      'SMA 20' : sma20,
      'SMA 50' : sma50
     })

priceSma_df
```

  </div>
</div>
  



  <div markdown="0">
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
      <th>Adj Close</th>
      <th>SMA 20</th>
      <th>SMA 50</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2014-08-20</td>
      <td>180.141846</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-21</td>
      <td>180.667160</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-22</td>
      <td>180.386368</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-25</td>
      <td>181.301010</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-26</td>
      <td>181.418716</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>2019-08-13</td>
      <td>292.549988</td>
      <td>295.381998</td>
      <td>293.540820</td>
    </tr>
    <tr>
      <td>2019-08-14</td>
      <td>283.899994</td>
      <td>294.689998</td>
      <td>293.635375</td>
    </tr>
    <tr>
      <td>2019-08-15</td>
      <td>284.649994</td>
      <td>293.980998</td>
      <td>293.696567</td>
    </tr>
    <tr>
      <td>2019-08-16</td>
      <td>288.850006</td>
      <td>293.564998</td>
      <td>293.805137</td>
    </tr>
    <tr>
      <td>2019-08-19</td>
      <td>292.329987</td>
      <td>293.286497</td>
      <td>293.926582</td>
    </tr>
  </tbody>
</table>
<p>1258 rows × 3 columns</p>
</div>
  </div>
  


Having all of our series in a single dataframe makes it easy to create a snap plot:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[12]:
  </div>
  <div class="input_area" markdown="1">
    
```python
priceSma_df.plot()
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  <matplotlib.axes._subplots.AxesSubplot at 0x1160c0c50></code></pre>
<p>



![png](TTB01%20-%20Simple%20Moving%20Average_files/TTB01%20-%20Simple%20Moving%20Average_26_1.png)


Although creating a snap plot with the dataframe own .plot() method is handy, I prefer to customize my chart using the individual function calls. Another benefit of having our all of our series in one dataframe is that we can easily select a **date range** (e.g. dates in 2017 and 2018, included) and plot only data within that range:


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[13]:
  </div>
  <div class="input_area" markdown="1">
    
```python
plt.figure(figsize = (12,6))
plt.plot(priceSma_df['2017':'2018']['Adj Close'], label='SPY Adj Close', linewidth = 2)
plt.plot(priceSma_df['2017':'2018']['SMA 20'], label='20 days rolling SMA', linewidth = 1.5)
plt.plot(priceSma_df['2017':'2018']['SMA 50'], label='50 days rolling SMA', linewidth = 1.5)
plt.xlabel('Date')
plt.ylabel('Adjusted closing price ($)')
plt.title('Price with Two Simple Moving Averages - Selected Date Range')
plt.legend()
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  <matplotlib.legend.Legend at 0x103742c90></code></pre>
<p>



![png](TTB01%20-%20Simple%20Moving%20Average_files/TTB01%20-%20Simple%20Moving%20Average_28_1.png)


Slicing data by date range is one of the great features of pandas. All of the following work:

- `priceSma_df['2017-04-01':'2017-06-15']`: range defined by two specific dates

- `priceSma_df['2017-01]` : prices in a given month

- `priceSma_df['2017]` : prices in a given year



### Use of two moving averages

By combining two moving averages, we can use a technique called *double crossover method*. With this setting, a buy signal is generated whenever the shorter moving average crosses the longer from below. Similarly, a sell signal is generated whenever the shorter moving average crosses the longer from above:

![Two MAs](images/fig03.png)

Compared with the technique that uses just one single moving average and the price, the double crossover method produces fewer whipsaws. On the other hand, it can generate signals with some delay.


### Use of three moving averages


If we can use two moving averages together, why not three then?


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[14]:
  </div>
  <div class="input_area" markdown="1">
    
```python
sma200 = close.rolling(window=200).mean()
priceSma_df['SMA 200'] = sma200
priceSma_df
```

  </div>
</div>
  



  <div markdown="0">
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
      <th>Adj Close</th>
      <th>SMA 20</th>
      <th>SMA 50</th>
      <th>SMA 200</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2014-08-20</td>
      <td>180.141846</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-21</td>
      <td>180.667160</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-22</td>
      <td>180.386368</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-25</td>
      <td>181.301010</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>2014-08-26</td>
      <td>181.418716</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>2019-08-13</td>
      <td>292.549988</td>
      <td>295.381998</td>
      <td>293.540820</td>
      <td>277.238597</td>
    </tr>
    <tr>
      <td>2019-08-14</td>
      <td>283.899994</td>
      <td>294.689998</td>
      <td>293.635375</td>
      <td>277.327894</td>
    </tr>
    <tr>
      <td>2019-08-15</td>
      <td>284.649994</td>
      <td>293.980998</td>
      <td>293.696567</td>
      <td>277.444337</td>
    </tr>
    <tr>
      <td>2019-08-16</td>
      <td>288.850006</td>
      <td>293.564998</td>
      <td>293.805137</td>
      <td>277.589019</td>
    </tr>
    <tr>
      <td>2019-08-19</td>
      <td>292.329987</td>
      <td>293.286497</td>
      <td>293.926582</td>
      <td>277.731843</td>
    </tr>
  </tbody>
</table>
<p>1258 rows × 4 columns</p>
</div>
  </div>
  



<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[15]:
  </div>
  <div class="input_area" markdown="1">
    
```python
start = '2016'
end = '2019'

plt.figure(figsize = (12,6))
plt.plot(priceSma_df[start:end]['Adj Close'], label='SPY Adj Close', linewidth = 2)
plt.plot(priceSma_df[start:end]['SMA 20'], label='20 day rolling SMA', linewidth = 1.5)
plt.plot(priceSma_df[start:end]['SMA 50'], label='50 day rolling SMA', linewidth = 1.5)
plt.plot(priceSma_df[start:end]['SMA 200'], label='200 day rolling SMA', linewidth = 1.5)
plt.xlabel('Date')
plt.ylabel('Adjusted closing price ($)')
plt.title('Price with Three Simple Moving Averages')
plt.legend()
```

  </div>
</div>
  



  {:.output_data_text}</p>

<pre><code>  <matplotlib.legend.Legend at 0x1164b9d50></code></pre>
<p>



![png](TTB01%20-%20Simple%20Moving%20Average_files/TTB01%20-%20Simple%20Moving%20Average_31_1.png)


See how the long term 200 SMA helps us to identify the trend in a very smooth manner.

This brings us to the *triple crossover method*. The set of moving averages we have used, with length 20, 50 and 200 days respectively, is actually widely used among analysts. We can choose to consider a buy signal when the 20 SMA crosses the 50 SMA from below, but only when both averages are above the 200 SMA. All the buy crosses that occur below the 200 SMA will be disregarded.

![Three MAs](images/fig04.png)
![Three MAs](images/fig05.png)

The examples in this article are just some of the many possibilities that can be generated by combining price and moving averages of different lengths. Also, price and moving averages can be combined with other available [technical indicators] or indicators that we can create ourselves.
Python and pandas provide all the power and flexibility needed to research and build a profitable trading system.

[technical indicators]: https://school.stockcharts.com/doku.php?id=technical_indicators


<div class="input">
  <div class="prompt input_prompt">
    In&nbsp;[None]:
  </div>
  <div class="input_area" markdown="1">
    
```python

```

  </div>
</div>
  