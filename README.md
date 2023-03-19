นักศึกษามีการใช้งานเครื่องมือเขียนโปรแกรมที่เกี่ยวข้องกับครึ่งเทอมแรกของวิชานี้ (Pandas/NumPy, Matplotlib/Seaborn) อย่างมีนัยสําคัญทั้งในกระบวนการจัดการและทําความสะอาดข้อมูล กระบวนการย่อยและวิเคราะห์ข้อมูล รวมถึงกระบวนการสรุปผลและแสดงภาพนิทัศน์ของข้อมูล

# Are Thai healthcare related stock still captivating?

Thai Health industry is found as a outperforming stock growth that the market for at least 5 executives year. They perform even better during to Covid 19. Over the past three years, the earnings of businesses in the healthcare sector have increased by 22% annually with profit growth of 14% per year. This indicates that these businesses are producing more sales and subsequently their profits are increasing too.

![alt text](https://user-images.githubusercontent.com/38032736/226188957-d4a38863-7993-4db1-9ac3-dbd0720c9c18.png)

<insert graph 5 year>

Unfortunately, after situation of covid-19 recover, the growth of healthcare industry drop. The overall growing pattern are converving to the market (compare with SET index). This lead to the question that, "Are healthcare related stock still captivating. If yes, which kind of stock shall we invest on?

<insert graph 1 year and 6 months>

## Installation
```python
from yahooquery import Ticker
import pandas as pd
import os
import time
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import datetime
import matplotlib.ticker as ticker

os.chdir(os.getcwd())
```


## dataset

We use data from import - yahooquery which retrieved nearly all data from Yahoo Finance. We select top largest 35 stocks with in healthcare sector.

```python
focus_symbol = ['BDMS.bk','BH.bk','RAM.bk','THG.bk','BCH.bk','CHG.bk','VIBHA.bk','STGT.bk','PRINC.bk','SKR.bk',
                'MASTER.bk','PR9.bk','CMR.bk','RJH.bk','NTV.bk','M-CHAI.bk','EKH.bk','VIH.bk','TOG.bk','LPH.bk',
                'RPH.bk','AHC.bk','IMH.bk','BIZ.bk','BIS.bk','WPH.bk','KDH.bk','D.bk','WINMED.bk','SMD.bk','KTMS.bk',
                'EFORL.bk','NEW.bk','TM.bk','LDC.bk'] 
tickers  = Ticker(focus_symbol, asynchronous=True)
```

Note: we remove 2 stocks from the consideration:
1. SVH: BDMS, SVH's largest shareholder, delisted Samitivej Public Company Limited (SVH) from being listed on the SET. SVH shares were be traded on the stock exchange on its the last day on Dec 7, 2022.
2. KLINIQ: is more likely to be a beauty clinic rather than medical clinic.


We aim to see progress since before Covid situation until current time. Therefore, data retrieved from the beginning of 2017 to the end of February 2023 (Current year). 

```python
start = '2017-01-01'
end = '2023-02-28'
```
## Data preparation

stock profile comprise of 

(i) 'summary_detail' - information about stock at some point of time.

(ii) 'asset_profile' - information on company profile including address, contact information, industry, sector, business summary, and related company officer, etc.

(iii) 'valuation_measures' - valuation measure by quarter comprise of Enterprise Value, Enterprises Value / EBITDA Ratio,	Enterprises Value / Revenue Ratio,	Forward Pe Ratio,	Market Capitalization,	Pb Ratio,	PeRatio,	Peg Ratio,	PsRatio.
											

```python
tickers_sum = tickers.summary_detail
data = tickers.asset_profile
valuation_measures = tickers.valuation_measures
```

```python
tickers_sum
```
<insert picture>

```python
data
```
<insert picture>

```python
valuation_measures
```
<insert picture>
 
 
### Prepare data
 
However, the data is not available for next step computation. We select some data point within dataset 'summary_detail', 'asset_profile' and 'valuation_measures' to use.

### #valuation_measures
Select only data with data collect quarterly (period type = 3 month)
 
```python
data_vm = valuation_measures.loc[valuation_measures['periodType']=='3M'].reset_index()
data_vm
```
<insert df result from python>

 
We also prepare "Pe ratio" for later use. Pe ratio will be pivot with date in order to see their progress over time.
 
> Pe ratio - or 'price-to-earnings ratio' is the ratio for valuing a company that measures its current share price relative to its earnings per share (EPS). The price-to-earnings ratio is also sometimes known as the price multiple or the earnings multiple. P/E ratios are used by investors and analysts to determine the relative value of a company's shares in an apples-to-apples comparison. It can also be used to compare a company against its own historical record or to compare aggregate markets against one another or over time. (Source: https://www.investopedia.com/terms/p/price-earningsratio.asp)
 
```python
piv_pe = pd.pivot_table(data_vm, values="MarketCap",index=["symbol"], columns=["asOfDate"]).reset_index()
piv_pe
```
<insert df result from python>
 
### Replace missing data:
With the nature of stock data, just few data is missing out. Some are missing due to the fact that that stock just enter the stock market after 2017. Only BCH stock is found that the data is truly missing due to the late data submission and yahoofinance have not update the new data yet. We solve this missing by inserting more updated data from search engine.
 
```python
missing = pd.DataFrame({'symbol':['BCH.bk'],
                        'asOfDate':[datetime.date(2022,12,31)],
                        'MarketCap':[51121830000],
                        'PeRatio':[9.74]})
data_vm = pd.concat([data_vm,missing])
```

### #asset_profile
Select several column that will be used further.
 
```python
data_Cat = pd.DataFrame()
for i in data:
    try:
        if type(data[i]) == dict:
            tmp = {'symbol':[i], 'industry': [data[i]['industry']] ,'sector': [data[i]['sector']], 'marketCap':[tickers_sum[i]['marketCap']]}
            data_Cat = pd.concat([data_Cat, pd.DataFrame(tmp)], ignore_index=True)
    except :
        pass
data_Cat.reset_index(drop=1,inplace=True)    
data_Cat.head()
```
<insert df result from python>

Add new column provide a firm size from market capitalisation data.
The critiria are:
 
  (i) Small Cap: Market Cap < 10,000 MB
                                   
  (ii) Mid Cap: 10,000 MB < Market Cap. < 50,000 MB
                                               
  (iii) Large Cap: Market Cap > 50,000 MB
                                     
source: https://knowledge.bualuang.co.th/knowledge-base/market-cap/
 
```python
data_Cat.loc[(data_Cat['marketCap'] < 10000000000),'captype'] = 'small_cap'
data_Cat.loc[(data_Cat['marketCap'] < 50000000000) & (data_Cat['marketCap'] >= 10000000000),'captype'] = 'mid_cap'
data_Cat.loc[(data_Cat['marketCap'] >= 50000000000),'captype'] = 'large_cap'
data_Cat.head() 
```
<insert df result from python>
 

### #prepare transaction history
```python
data_price = pd.DataFrame()
start = '2017-01-01'
end = '2023-02-28'
try:
    df_history = tickers.history(interval = '1d', start= start, end = end )
    df_history = df_history.reset_index()
    data_price = pd.concat([data_price, df_history], ignore_index=True)
except:
    print('can not pull price')
    pass

data_price['date'] = pd.to_datetime(data_price['date'], utc=True).dt.date
data_price.head()
``` 
<insert df result from python>

	
## Which industry are the driving changes



## Medicare care grow with assistance from Government
The healthcare sector has been the beneficiary of government policy that stretches back to 2003 to promote Thailand as a ‘medical hub’. Government support as follows:

(i) extending the permitted period of stay for visitors traveling to Thailand for medical treatment from China and the CLMV nations to 90 days from 30 days and allowing up to 4 family members to accompany travelers bound for Thai hospitals

(ii) extending long stay visas from 1 to 10 years for people from 14 specified nations 

(iii) provide a special dental and health-check package for international travelers

All of these actions will assist Thai private hospitals in broadening their customer base to include more foreigners, a market segment with greater purchasing power who frequently spend more on healthcare than their local counterparts. As a result, private hospitals are experiencing steady revenue growth and are maintaining excellent profit margins. And this has in fact led to the steady growth of medical tourism in the country.
In addition, Thailand is quickly gaining an international reputation as a high-quality and inexpensive destination for health tourists" to visit and get complicated medical procedures performed which make Thai healthcare industry become more and more attractive.


