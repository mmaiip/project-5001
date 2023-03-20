นักศึกษามีการใช้งานเครื่องมือเขียนโปรแกรมที่เกี่ยวข้องกับครึ่งเทอมแรกของวิชานี้ (Pandas/NumPy, Matplotlib/Seaborn) อย่างมีนัยสําคัญทั้งในกระบวนการจัดการและทําความสะอาดข้อมูล กระบวนการย่อยและวิเคราะห์ข้อมูล รวมถึงกระบวนการสรุปผลและแสดงภาพนิทัศน์ของข้อมูล


# Are Thai healthcare related stock still captivating?

Thai Health industry is found as a outperforming stock growth that the market for at least 5 executives year. They perform even better during the Covid-19 era (2019-2022). Over the past three years, the earnings of businesses in the healthcare sector have increased by 22% annually with profit growth of 14% per year. This indicates that these businesses are producing more sales and subsequently their profits are increasing too.

Graph compare Health index (HELTH) with SET index over 5 year
![alt text](https://user-images.githubusercontent.com/118241553/226198314-6dfabf99-f4b2-4a38-a82c-bdd546c30be9.png)

Unfortunately, after situation of covid-19 recover, the growth of healthcare industry drop. The overall growing pattern are converving to the market (compare with SET index). This lead to the question that, "Are healthcare related stock still captivating. If yes, which kind of stock shall we invest on?

Graph compare Health index (HELTH) with SET index over 1 year
![alt text](https://user-images.githubusercontent.com/118241553/226190434-aa2f2856-5f49-4a87-bf6c-403b7bf7c5ef.png)

Graph compare Health index (HELTH) with SET index over 6 months
![alt text](https://user-images.githubusercontent.com/118241553/226190427-967472ba-9b9a-47ef-9604-e472ed94c030.png)


## Installation
```python
from yahooquery import Ticker
import pandas as pd
import time
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import datetime
import matplotlib.ticker as ticker
```

## Dataset

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

We retrieve data from 4 datasets:

(I) summary_detail - summary information about stock.

(II) asset_profile - information on company profile including address, contact information, industry, sector, business summary, and related company officer, etc.

(III) valuation_measures - valuation measure by quarter comprise of Enterprise Value, Enterprises Value / EBITDA Ratio,	Enterprises Value / Revenue Ratio,	Forward Pe Ratio,	Market Capitalization,	Pb Ratio,	PeRatio,	Peg Ratio,	PsRatio.

(IV) history - transactional data comprise of openning price, highest price, lowest price, close price, volume trade, adjclose, dividends, splits.
											

```python
tickers_sum = tickers.summary_detail
data = tickers.asset_profile
valuation_measures = tickers.valuation_measures
df_history = tickers.history
```
 
 
### Prepare data
 
However, the data is not available for next step computation. We select some data point within dataset 'summary_detail', 'asset_profile' and 'valuation_measures' to use.

#### #valuation_measures
Select only data with data collect quarterly (period type = 3 month)
 
```python
data_vm = valuation_measures.loc[valuation_measures['periodType']=='3M'].reset_index()
data_vm
```
![alt text](https://user-images.githubusercontent.com/118241553/226266023-445f8326-4d5d-4d4b-9299-2abf95b958b6.png)

 
We also prepare "Pe ratio" pivot with quarterly date in order to see their progress over time.
 
```python
piv_pe = pd.pivot_table(data_vm, values="MarketCap",index=["symbol"], columns=["asOfDate"]).reset_index()
piv_pe
```
![alt text](https://user-images.githubusercontent.com/118241553/226198324-03d0e276-17ed-4819-bedd-f75971f6dd5e.png)
 
##### Replace missing data:
With the nature of stock data, just few data is missing out. Some are missing due to the fact that that stock just enter the stock market after 2017. Only BCH stock is found that the data is truly missing due to the late data submission and yahoofinance have not update the new data yet. We solve this missing by inserting more updated data from "www.settrade.com". After replace missing data, we also change date type to datetime to further process.
 
```python
missing = pd.DataFrame({'symbol':['BCH.bk'],
                        'asOfDate':[datetime.date(2022,12,31)],
                        'MarketCap':[51121830000],
                        'PeRatio':[9.74]})
data_vm = pd.concat([data_vm,missing])

data_vm['asOfDate'] = pd.to_datetime(data_vm['asOfDate']).dt.date
```

#### #asset_profile
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

![alt text](https://user-images.githubusercontent.com/118241553/226268477-d3202b5c-700f-48d5-8b2d-0fb534064dac.png)

	
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
![alt text](https://user-images.githubusercontent.com/118241553/226198323-23723607-3dee-4473-8e17-a0b230e0d9a5.png)
	

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

![alt text](https://user-images.githubusercontent.com/118241553/226269209-973ab5fa-5621-42df-8d08-155062a973fd.png)

	
## Which industry are the driving changes
	The healthcare sector composes of firms from different industries and different sizes. We try to find out which characteristics of firms inside the healthcare sector drive growth and which we should beware of.

	We diagnose over 1 year period using data from the last quarter of 2022 through the last quarter of 2023 to see the change over the peak of the Covid-19 era (emergence of COVID-19 virus variant Omicron) until present.

	Firstly, we diagnose firms by their size because we assume that size of firm should affect their potential to do business. We want to see how firms are growing. We focus on their marketcapitalization by preparing market capitalization data pivot with quarter period group by the firm size.
	
```python
data4 = pd.merge(data_vm,data_Cat[['symbol','captype']],how='left',on='symbol') 
data4 = data4.groupby(['captype', 'asOfDate'])['MarketCap'].sum().reset_index()
data4
```

![alt text](https://user-images.githubusercontent.com/118241553/226274141-c6508629-ef31-4bfd-a0b1-958cd2d29a0b.png)
	
```python
tmp_forMerge = pd.DataFrame()
for i in data4['captype'].unique():
    tmp = data4.loc[data4['captype']==i]
    tmp = tmp.sort_values('asOfDate').reset_index(drop=1)
    tmp['pct_change'] = tmp['MarketCap'].pct_change()
    tmp_forMerge = pd.concat([tmp_forMerge,tmp])
    
data4 = pd.merge(data4,tmp_forMerge,on=['captype','asOfDate'], how='left')
data4
```

![alt text](https://user-images.githubusercontent.com/118241553/226274126-aef5dc53-2c26-4467-b34b-c11b37087783.png)	

According to graph below, Blue, orange, and green lines represent large-cap, mid-cap, and small-cap firms respectively. Large-cap firms have slightly negative to positive changes in market capitalization from -0.6% to 16.4%. While small to mid-cap firms grow negatively over time. 
	
```python
sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.lineplot(data=data4, x="asOfDate", y="pct_change", hue='captype',lw=5)
g.axhline(0, c='k', ls='-', lw=1.2)
plt.xticks(rotation=45)

for x, y in zip(data4['asOfDate'], data4['pct_change']):
     plt.text(x = x,
              y = y-0.01,
              s = '{:.3f}'.format(y),
              color = 'black')

plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188963-e8d32df5-b6b6-4cc3-82be-d2d38a01bfbe.png)


Secondly, we look into what firms do. So we divided firms into groups by their industries -- Medical Care Facilities, Specialty Chemicals, Medical Instruments & Supplies, Medical Devices, Packaged Foods, and Medical Distribution. 


```python
data5 = pd.merge(data_vm,data_Cat[['symbol','industry']],how='left',on='symbol') 
data5 = data5.groupby(['industry', 'asOfDate'])['MarketCap'].sum().reset_index()

tmp_forMerge = pd.DataFrame()
for i in data5['industry'].unique():
    tmp = data5.loc[data5['industry']==i]
    tmp = tmp.sort_values('asOfDate').reset_index(drop=1)
    tmp['pct_change'] = tmp['MarketCap'].pct_change()
    tmp_forMerge = pd.concat([tmp_forMerge,tmp])
    
data5 = pd.merge(data5,tmp_forMerge,on=['industry','asOfDate'], how='left')
data5
```

![alt text](https://user-images.githubusercontent.com/118241553/226275360-d18ba8c4-bc2f-412f-b9ad-2f78c704080e.png)
	
All types of industries perform in the same direction. 
- We could see that “Medical care facilities” is less fluctuate than other industries with positive growth.
- Specialty chemical and medical distribution groups are the industries with negative growth throughout 2022 with the lowest growth among all industries.
- Medical devices and medical instruments & supplies are groups that provide essential tools or devices for hospitals. As a supportive industry for the healthcare sector, this group moves along with the Medical care facilities but more fluctuated.
	
```python
sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.lineplot(data=data5, x="asOfDate", y="pct_change", hue='industry',lw=5)
g.axhline(0, c='k', ls='-', lw=1.2)
plt.xticks(rotation=45)

for x, y in zip(data5['asOfDate'], data5['pct_change']):
     plt.text(x = x,
              y = y-0.01,
              s = '{:.3f}'.format(y),
              color = 'black')

plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188964-ba00eb8f-40a7-4422-89aa-e085af44a6f2.png)


Next, we measure company attractiveness by focusing on the “PE ratio”. 
> PE ratio - or 'price-to-earnings ratio' is the ratio for valuing a company that measures its current share price relative to its earnings per share (EPS). The price-to-earnings ratio is also sometimes known as the price multiple or the earnings multiple. P/E ratios are used by investors and analysts to determine the relative value of a company's shares in an apples-to-apples comparison. It can also be used to compare a company against its own historical record or to compare aggregate markets against one another or over time. (Source: https://www.investopedia.com/terms/p/price-earningsratio.asp)

The lower PE ratio compared with other stocks show that the stock we are focusing on is relatively cheaper than other. Seeing from the next figure, we compare PE Ratio with their median PE. Some are about and lower than the median. We cannot see any patterns within the firm-cap type. 
	
```python
data1 = data_vm.loc[data_vm['asOfDate']==datetime.date(2022, 12, 31)]
data1 = data1[['symbol','PeRatio']]
data1 = pd.merge(data1,data_Cat[['symbol','captype']],how='left',on='symbol')
data1 = data1.sort_values(['captype','symbol'])

sns.set(rc={'figure.figsize':(16,12), 'figure.dpi':300}) #rc={'figure.dpi':300}
g = sns.barplot(data = data1, x='PeRatio',  y='symbol', orient='h',hue='captype',dodge=False)
g.axvline(data1.PeRatio.mean(), c='k', ls='-', lw=2.5)
plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188957-d4a38863-7993-4db1-9ac3-dbd0720c9c18.png)

	
So we plot the percentage change of PE by quarter and find their differences.


```python
data2 = data_vm.loc[(data_vm['asOfDate']==datetime.date(2022, 12, 31)) | (data_vm['asOfDate']==datetime.date(2021, 12, 31))]
data2 = data2[['symbol','PeRatio','asOfDate']]

data2 = pd.merge(data2,data_Cat[['symbol','marketCap']],how='left',on='symbol')
data2 = data2.sort_values(['marketCap','symbol','asOfDate'],ascending = [False,True,True])

sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.barplot(data=data2, x="symbol", y="PeRatio", hue="asOfDate")
plt.xticks(rotation=45)
plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188961-b1f6983f-6388-429d-b3f1-98d76fc956c7.png)

We could see that in large-cap firms, PE growing negatively while others are quite mixed. This show that, large firm stocks are relatively cheaper over time.
	
```python
data3 = piv_pe[['symbol','pct_change']]
data3 = pd.merge(data3,data_Cat[['symbol','marketCap','captype','industry']],how='left',on='symbol')
data3 = data3.sort_values(['marketCap','symbol'],ascending = [False,True])

sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.barplot(data=data3, x="symbol", y="pct_change", hue='captype',dodge=False)
plt.xticks(rotation=45)
plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226191708-b1b16d10-be5b-4f2f-8d04-f0a66cdb4af4.png)

Since we aim to select stocks that have the potential to grow positively with low fluctuation, the list of stocks that meet our criteria is large firms in medical care facilities as given: BDMS, BH, RAM, THG, and BCH.

- Bangkok Dusit Medical Services (BDMS)
- Bumrungrad International Hospital (BH)
- Ramkhamhaeng Hospital (RAM)
- Thonburi Healthcare Group PCL (THG)
- Bangkok Chain Hospital PCL (BCH)

Note: Medical care facilities are firms that do business to provide places with health care. They include hospitals, clinics, outpatient care centers, and specialized care centers, such as birthing centers and psychiatric care centers.
	
	
## Which firm should we select to invest?

Using transactional data, to analyze deeper which firms are attractive. We use historical data of daily stock trade from 2017 until the present.

```python
dataSelect = data_price.loc[data_price['symbol'].isin( data_Cat.loc[data_Cat['captype']=='large_cap']['symbol'])]
tmp_forMerge = pd.DataFrame()
for i in dataSelect['symbol'].unique():
    tmp = dataSelect.loc[dataSelect['symbol']==i]
    tmp = tmp.sort_values('date').reset_index(drop=1)
    tmp['pct_change'] = tmp['close'].pct_change()
    tmp_forMerge = pd.concat([tmp_forMerge,tmp])
    
dataSelect = tmp_forMerge.copy().reset_index(drop=1)
dataSelect.head()
```

Firstly, we want to know how each firm grew over time. We plot the “close price” of each firm to see how their price move. 

Here are some insights. 
	
Thai medical care industry has not rapidly grew from the beginning of covid era in 2019 as we make the assumption at first. They begin to grow rapidly because of Omicron in late 2022. Firm revenue grew rapidly as a result of the overwhelming demand for medical care and vaccination. All 5 stocks in consideration grow significantly. 

Here are some important events we need to know before analyzing further:

- THG price spiked in the second quarter of 2022 till its peak and then drop dramatically. The drop in price seems to be due to the fact that from the 4th quarter of 2020 to the present, THG's stock price has increased by 41%, the highest compared to all 5 companies studied by the research department better than the hospital group, which rose about 11%. This may cause some investors to sell for profits.
- RAM price drop drastically in the last quarter of 2022 due to Stock Split.

```python
sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.lineplot(data=dataSelect, x="date", y="close", hue='symbol',lw=5)
plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188966-9f48ad6c-34e4-407d-b3d5-136941dc95b1.png)

	
Next, we consider stock price fluctuation. In the picture may show several price change spike from BH, THG, and RAM. BH, THG, and RAM tend to be morefluctuate than BDMS and BCH. 
	
```python
sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.lineplot(data=dataSelect, x="date", y="pct_change", hue='symbol',lw=5)
plt.show()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188968-45d28251-7163-4ba9-910b-2b8db0034512.png)

	
We want to see how stock price move and there rate of change of each day from 2017 until now to see a better picture of each stock.

We found out that
	
```python
for idx, i in enumerate (dataSelect['symbol'].unique()):
    tmp_ = dataSelect.loc[dataSelect['symbol']==i].reset_index(drop=1)
    fig, axes = plt.subplots(2, 1)
    sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})

    sns.lineplot(data=tmp_, x="date", y="close", hue='symbol',lw=5, ax=axes[0])
    sns.lineplot(data=tmp_, x="date", y="pct_change", hue='symbol',lw=5, ax=axes[1])
    plt.show()
```
BDMS price were fluctuate between 18 - 30 with positive gradually move. BDMS price change is around -10%-10%.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188969-5baa8e35-b0d6-40e6-930b-57701c92eedf.png)

BH price were fluctuate between 50 - 250. BH price change is around -10%-15%.
BH move relatively similar to BDMS. But with a higher stock price, BH is more fluctuate than BDMS

![alt text](https://user-images.githubusercontent.com/38032736/226188971-341eade9-cf46-48cf-a592-fe517dcf6824.png)
	
RAM is a stock with most fluctuate one with frequent stock split. RAM price were fluctuate between 25 - 200. RAM price change is around -80%-20%.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188973-c3ad6bf1-d406-47f7-b51e-a5a14eb935dc.png)

THG price were fluctuate between 18 - 95. THG price change is around -18%-10%.

![alt text](https://user-images.githubusercontent.com/38032736/226188974-f327f83f-33b3-4d1f-8e82-52253b8cbe8b.png)
	
BCH price were fluctuate between 50 - 250. BCH price change is around -15%-10%.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188976-0686953f-84db-461c-a3ab-972b0b067744.png)


Up to this point, eventhough 5 stock we select are in the same industries, there are several differences.

- BDMS would be a stock for investor who are risk averse. The stock have a good positive growth with risk that quite balance between upside and downside risk.

- For those investor who do day to day trade or short term trade. Stock with higher price fluctuation like TGH and RAM may be more suitable.

- BH and BCH came up with moderate risk, but BH have higher maximum negative change.

	
## Focus on the year 2022

Looking at the same parameter as in 5 year duration.
	
Through price change, all 5 are in a pretty good sign. All have positive growth over the year.
	
```python
dataSelect2 = data_price.loc[(data_price['symbol'].isin( data_Cat.loc[data_Cat['captype']=='large_cap']['symbol'])) & 
                            (pd.to_datetime(data_price['date']).dt.date >= datetime.date(2022,1,1)) & (pd.to_datetime(data_price['date']).dt.date <= datetime.date(2022,12,31))]
dataSelect2.head()
```
![alt text](https://user-images.githubusercontent.com/38032736/226188977-c03be305-31c7-4247-b961-4670adaa5c65.png)

In the year 2022, RAM, BH, and TGH still the group with higher fluctuation.

![alt text](https://user-images.githubusercontent.com/38032736/226188978-3d134283-9843-4c1d-8476-e1b577c44c1d.png)

We focus more on each stock, this time we look the rate od return through the histogram to se stock price change distribution.

```python
df_return = pd.DataFrame()
for idx, i in enumerate (dataSelect2['symbol'].unique()):
    tmp_ = dataSelect2.loc[dataSelect2['symbol']==i].reset_index(drop=1)
    fig, axes = plt.subplots(3)
    sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
    sns.lineplot(data=tmp_, x="date", y="close", hue='symbol',lw=5, ax=axes[0])
    sns.lineplot(data=tmp_, x="date", y="pct_change", hue='symbol', ax=axes[1]).set(xticklabels=[])
    
    min_val = tmp_['pct_change'].min()
    max_val = tmp_['pct_change'].max()
    
    abs_max = abs(max_val) if abs(max_val) > abs(min_val) else abs(min_val)
    val_width = max_val - min_val
    n_bins = 17
    bin_width = val_width/n_bins


    sns.histplot(data=tmp_, x="pct_change", ax=axes[2],bins=n_bins, binrange=(-abs_max, abs_max))
    plt.show()
    df_return = pd.concat([df_return,pd.DataFrame({'symbol':[i],'return':[tmp_['pct_change'].sum()]})])
    print ('sum pct_change', i, tmp_['pct_change'].sum())
```

BDMS price grow positively from ฿20 and reach it peak around ฿32 and drop a bit to ฿30 at the end of year. Price fluatuate around -4% - 4% which is relatively low compare to other. Price change distribution are quite symmetry with little skew to the left.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188979-6dc4efa9-7739-4512-a7d9-8b7cba968387.png)
	
BH price grow positively from ฿130 and reach it peak around ฿240 and drop a bit to ฿220 at the end of year. Price fluatuate around -7.5% - 10%. Price change distribution are quite symmetry with little skew to the left.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188982-991782c4-f4d5-46e9-be41-32380e5d8a3e.png)

RAM price grow from ฿40 at the beginning of the year, drop a bit to ฿32, reach it peak around ฿65 and drop to ฿55. Price are quite steady from last 5 month of the year around ฿55. Price fluatuate around -10% - 25% which is relatively high compare to other. Price change distribution are quite symmetry with little skew to the right. RAM have more downside risk compare to the rest.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188984-051d2775-11ea-49ee-b709-711e1b12ad6d.png)
	
TGH price grow from ฿35 at the beginning of year. Its price hike a lot and reach its peak at ฿100 in April then drastically drop in May. After that to the end of year, prices mildly fluctuate around ฿55-฿75. Price fluatuate around -7.5% - 10%. Price change distribution are skew to the left.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188987-79f5c495-43f1-44a1-8b12-58e2ac88d1dc.png)
		
BCH price are fluctuate a lot in 2022. Price start at ฿19 and end at ฿20 which show small differences. But in the between, price are fluctuate up and down around -6% - 6% which is quite moderate. Price change distribution are quite symmetry with little skew to the right.
	
![alt text](https://user-images.githubusercontent.com/38032736/226188988-6ce5de2b-73d3-4106-a9e3-dd31ad94d72d.png)

We also provide a return graph if investor hold stock for a year. The return range from 8% - 70% which are very high comparing to the overall stock market.
	
```python
sns.set(rc={'figure.figsize':(24,12), 'figure.dpi':300})
g = sns.barplot(data=df_return, x="symbol", y="return", hue='symbol',dodge=False)
plt.show()
```

![alt text](https://user-images.githubusercontent.com/38032736/226188989-f361fe34-5956-4a36-9b71-76da93b3fe30.png)

In conclusion
- For risk consideration, stock with lower downside risk (price change distribution that are skew to the left) are BDMS, BH, TGH. 
	
- With the price trend in 2022, BDMS, BH, TGH and RAM have positive price change.
	
- Mild fluctuation stock would be BDMS (around -6 - 6%).
	
- The moderate (around -10% to 10%) are BCH and BH. 

- High fluctuation stock are RAM and THG (more than -10% - 10%)
	
- Stock with highest return if they hold stock for year is THG and lowest is BCH. Even the lowest return stock provide a return around 8% wish is very attractive for holding stock for year.

Up to this point, we conclude that even after the COVID-19 era, healthcare sector stock are still attractive. But be aware that not all stock is interesting. Medical care in a large company  are the one that still perform really well after the COVID era. We suspect that this because the impact of health tourism which can be read further in our last part. Up to this point, investor can customised portfolio by their risk and return preference.

> Medicare care grow with assistance from Government
	
> The healthcare sector has been the beneficiary of government policy that stretches back to 2003 to promote Thailand as a ‘medical hub’. Government support as follows:

> (i) extending the permitted period of stay for visitors traveling to Thailand for medical treatment from China and the CLMV nations to 90 days from 30 days and allowing up to 4 family members to accompany travelers bound for Thai hospitals

> (ii) extending long stay visas from 1 to 10 years for people from 14 specified nations 

> (iii) provide a special dental and health-check package for international travelers

> All of these actions will assist Thai private hospitals in broadening their customer base to include more foreigners, a market segment with greater purchasing power who frequently spend more on healthcare than their local counterparts. As a result, private hospitals are experiencing steady revenue growth and are maintaining excellent profit margins. And this has in fact led to the steady growth of medical tourism in the country.
In addition, Thailand is quickly gaining an international reputation as a high-quality and inexpensive destination for health tourists" to visit and get complicated medical procedures performed which make Thai healthcare industry become more and more attractive.

