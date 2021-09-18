# Introduction to Stock Performance Analyzer
The phrase "beating the market" means earning an investment return that exceeds the performance of the Standard & Poor's 500 index. Commonly called the S&P 500, it's one of the most popular benchmarks of the overall U.S. stock market performance. Everybody tries to beat the SP500, but few succeed! 
* This code aims to provide a tool to quickly evaluate the performance of Nasdaq and NYSE stocks relative to the SP500 index.
* The code calculates cumulative returns for each ticker and compares it to the SP500 return for the same period.
* NYSE and NASDAQ tickers can be imported from here: https://www.nasdaq.com/market-activity/stocks/screener
* The link above provides a CSV file with all the tickers. 
* For this exercise, we will use Yahoo Finance API to automatically access SP500 hisotrical data. 

# Programming Start Here

Import tools needed for the analysis, we may need all of them
```python
import datetime as dt
import pandas as pd
import pandas_datareader as web
import yfinance as fyf
from yahoo_fin import stock_info as si
import time
```
Let's define start and end for the evaluation period 
```python
start = dt.datetime(2016,9,9)
end = dt.datetime(2021,9,9)
```
Use Yahoo API to access SP500 tickers list
```python
SP500_tickers = si.tickers_sp500()
```
Import SP500 historical data into a dataframe
```python
sp500_df = web.DataReader('^GSPC', 'yahoo', start, end)
```
Calculate percent change for SP500 and then the cumulative return   
It is worth mentioning that the cumulative return is the total change in the investment price over a set time—an aggregate return, not an annualized one.
```python
sp500_df['Pct Change'] = sp500_df['Adj Close'].pct_change()
sp500_return = (sp500_df['Pct Change']+1).cumprod()[-1]
```
Import each stock historical data. We will monitor and report the processing time
```python
t1 = time.perf_counter() #start counting processing time
df_fyf = fyf.download(SP500_tickers, start, end)
t2 = time.perf_counter() #end counting processing time
print(f'Finished in {round(t2-t1,2)} second') 
```    
Reduce the size of the dataframe to keep needed data
```python
df = df_fyf[['Close', 'Low', 'High']]
```
Cumulative stock return for each stock. This calculation method is vectorized, and hence it's faster than a for-loop
```python
stock_pct = df['Close'].pct_change()
stock_return = (stock_pct+1).cumprod()[-1:]
```
Create a varibale to compare the return of each stock to the SP500
```python
returns_compared = round((stock_return/sp500_return),2)
returns_compared= returns_compared.reset_index(drop=True)
returns_compared = returns_compared.T
```

# Returns Ranking Relative to SP500  
Create variable Score to rank all the tickers based on returns relative to SP500
```python
final_df = pd.DataFrame()
final_df['returns_compared'] = returns_compared.iloc[:,0]
final_df['Score'] = final_df['returns_compared'].rank(pct=True) * 100
```
Define criteria for best performers. We will select top 20%     
Basically, the socre has to be greater than 0.8 quantile. This means top 20 percentile. 
```python
best_performers = final_df[final_df['Score'] >= final_df['Score'].quantile(0.8)]
#reset index and rename index to ticker
best_performers= best_performers.reset_index()
best_performers.rename(columns={'index':'Ticker'}, inplace=True)
```
# Results
Based on the selected time ```start``` and ```end```, there are 102 tickers that beat the SP500 return.    
Below is a table that shows some of those tickers along with their score and return relative to SP500. You might be familair with some of the names!   
   
   
![alt text](https://github.com/MouinAlmasoodi/Stock-Performance/blob/main/Results.JPG?raw=true)
