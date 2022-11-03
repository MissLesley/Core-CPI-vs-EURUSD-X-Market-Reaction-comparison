# Core-CPI-vs-EURUSD-X-Market-Reaction-comparison
#I am trying to create a market comparison of how the EURUSD=X ticker has been reacting to the Core CPI announcements since 2020 to date.
#This study is restricted to a 2hr timeframe; 1hr before the announcement and 1hr after the announcement; data to be presented at a 15minute interval. 
#There are two data sets expected as the inal results
#a) Continuos Data -a graph of the Core CPI market expectation and actual values reported in every monthly event. 
#b) Discontinous Data -graphs of the EURUSD=X behaviour within the 2hr timeframe on the specific date of announcement. 
#These two should however be combined into one. 
#The parameter are as follows; Historical Core Cpi values (market expectations & actual), Specific historic dates of announcement, Historic values of EURUSD=X within the 1hr before and after announcement timeframe set at a 15minute interval,EURUSD=X support lines on the dates of announcements. 
#Here are my attemps with problems encountered described.

import yfinance as yf
import pandas_datareader as pdr
import pandas as pd
from datetime import datetime, timedelta
from time import time
from tqdm import tqdm
from pathlib import Path
import plotly.graph_objects as go
import plotly.express as px
import logging
import sys
import matplotlib.pyplot as plt
from datetime import datetime, timedelta
import os

beginning_date = '2020-01-01'
date_today = datetime.now()
end_date = date_today

fred_assets = {'Core_CPI': 'STICKCPIM157SFRBATL'}
yfinance_assets = {'EURUSD':'EURUSD=X'}

def get_fred_prices(assets):
  df_temp = pd.DataFrame(index=pd.date_range(start=beginning_date,end=end_date,freq='D'))
  # df_yearly = pd.DataFrame(index=pd.date_range(start=beginning_date,end=end_date,freq='A'))
  for key, value in assets.items():
    print('Getting {} with symbol {}'.format(key, value))
    try:
      df_temp[f'{key}'] = pdr.DataReader(f'{value}','fred',beginning_date,date_today)
    except:
      print('Error getting {}'.format(value))
    df_temp.ffill(inplace=True)
  return(df_temp)

def get_yfinance_prices(assets):
  df_temp = pd.DataFrame(index=pd.date_range(start=beginning_date,end=end_date,freq='D'))
  for key, value in assets.items():
    print('Getting {} with symbol {}'.format(key, value))
    try:
      df_temp[f'{key}'] = yf.download(f'{value}',beginning_date,date_today,progress=True).drop(columns=['Open','High','Low','Close','Volume'])
    except:
      print('Error getting {}'.format(value))
    df_temp.ffill(inplace=True)
  return(df_temp)

def get_asset_prices(filename):
  df_fred = get_fred_prices(fred_assets)
  df_yfinance = get_yfinance_prices(yfinance_assets)
  df = pd.concat([df_fred, df_yfinance], axis=1)
  return(df)

df = get_asset_prices('asset_prices')
df.dropna(how='all', inplace=True)

print (df)

#output;
            Core_CPI  EURUSD
2020-01-01  0.296860     NaN
2020-01-02  0.296860     NaN
2020-01-03  0.296860     NaN
2020-01-04  0.296860     NaN
2020-01-05  0.296860     NaN
...              ...     ...
2022-10-30  0.681338     NaN
2022-10-31  0.681338     NaN
2022-11-01  0.681338     NaN
2022-11-02  0.681338     NaN
2022-11-03  0.681338     NaN

[1038 rows x 2 columns]
#Problem -(a) the Core CPI fetched from FRED corresponds with is MOM on investing.com yet I am searching for YOY; Question- how to match the data?
          (b) EURUSD values returned are NAN, why? Yet when I use the method below, i can fetch the data
          
forex_data = yf.download('EURUSD=X', start='2019-01-02')
forex_data.index = pd.to_datetime(forex_data.index)
forex_data.tail()

#out:

Open	High	Low	Close	Adj Close	Volume
Date						
2022-10-13 00:00:00+01:00	0.970798	0.979509	0.963595	0.970798	0.970798	0
2022-10-14 00:00:00+01:00	0.975134	0.980383	0.970949	0.975134	0.975134	0
2022-10-17 00:00:00+01:00	0.973937	0.984707	0.972450	0.973937	0.973937	0
2022-10-18 00:00:00+01:00	0.984640	0.987362	0.981460	0.984640	0.984640	0
2022-10-19 00:00:00+01:00	0.986096	0.987362	0.980969	0.981547	0.981547	0

          (c) I attempted to resolve this by using investipy and investiny since yahoo finance is missing some data on economic events, but i keep getting error 403. 
  #Questions
  1)How do I fetch the historical economic calendar on investing.com
  2) a second option to part (1) would be fetching the historical calendar on FRED, but which CPI of the 43 announced say on 13/10/2022 is similar to the Core CPI
     as recorded on investing.com. 
  3) How do I isolate the specific dates of annoucements in the fetched economic calendar. 
  4) How do I isolate the EURUSD=X values in the two hour-15 minute interval timeframe on the announcement date.
  5) How do I fetch the support lines for the specific isolated dates of Core CPI announcement? 
  6) How do i combine all these into one graph per year or two graphs per year (semi-annual)
