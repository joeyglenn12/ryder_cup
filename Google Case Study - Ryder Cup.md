```python
#Web Scraping
import pandas
```


```python
import pandas as pd
import requests
pd.set_option('display.max_columns', None) #so we can see all columns in a wide dataframe
import time
import numpy as np
```


```python

```


```python
url_2023_season = 'https://site.web.api.espn.com/apis/common/v3/sports/golf/pga/statistics/byathlete?region=us&lang=en&contentorigin=espn&isqualified=true&page=1&limit=50&sort=general.amount%3Adesc&season=2023'
url_2022_season = 'https://site.web.api.espn.com/apis/common/v3/sports/golf/pga/statistics/byathlete?region=us&lang=en&contentorigin=espn&isqualified=true&page=1&limit=50&sort=general.amount%3Adesc&season=2022'
url_2021_season = 'https://site.web.api.espn.com/apis/common/v3/sports/golf/pga/statistics/byathlete?region=us&lang=en&contentorigin=espn&isqualified=true&page=1&limit=50&sort=general.amount%3Adesc&season=2021'
```


```python
r_2023 = requests.get(url=url_2023_season).json()
r_2022 = requests.get(url=url_2022_season).json()
r_2021 = requests.get(url=url_2021_season).json()
```


```python
#combine into a collection of dictionaries
all_dicts = [r_2021, r_2022, r_2023]
```


```python
#check the first athlete's values
x = r_2023['athletes'][0]['categories'][0]['totals'][0:15]
print(x)
```

    ['$11,631,495', '1770', '10', '36', '10', '7', '2', '68.1', '308.7', '63.3', '73.8', '1.715', '0', '36.0', '4.694']



```python
#adding the value headers manually
table_headers = ['Earnings','Cup','Evnts','Rnds','Cuts','Top10','Wins','Score','DDIS','DACC','GIR','PUTTS','SAND','BIRDS']
print(table_headers)
```

    ['Earnings', 'Cup', 'Evnts', 'Rnds', 'Cuts', 'Top10', 'Wins', 'Score', 'DDIS', 'DACC', 'GIR', 'PUTTS', 'SAND', 'BIRDS']



```python
#let's pull the stats for 2023
values_2023 =[]
for athlete in r_2023['athletes']:
       for category in athlete['categories']:
           x = category['totals'][0:14]
           values_2023.append(x)

#now we'll put it into a dataframe
df_values_2023 = pd.DataFrame(values_2023, columns=table_headers)

```


```python
#player info is in a different section, we'll have to index new data
info_categories = r_2023['athletes'][0]['athlete'].keys()
info_2023 = []

for athlete in r_2023['athletes']:
    row = []
    for category in info_categories:
        if category in athlete['athlete']:
            row.append(athlete['athlete'][category])
        else:
            row.append(None)
    info_2023.append(row)

#we'll turn this to a data frame too
info_df_2023 = pd.DataFrame(info_2023, columns=info_categories)

```


```python
#combine both dataframes
df_2023 = pd.concat([info_df_2023,df_values_2023], axis=1)
```


```python
#repeat this process for seasons 2021 and 2022
values_2022 =[]
for athlete in r_2022['athletes']:
       for category in athlete['categories']:
           x = category['totals'][0:14]
           values_2022.append(x)

df_values_2022 = pd.DataFrame(values_2022, columns=table_headers)

#2022 athletes
info_categories_2022 = r_2022['athletes'][0]['athlete'].keys()
info_2022 = []

for athlete in r_2022['athletes']:
    row = []
    for category in info_categories_2022:
        if category in athlete['athlete']:
            row.append(athlete['athlete'][category])
        else:
            row.append(None)
    info_2022.append(row)
    
info_df_2022 = pd.DataFrame(info_2022, columns=info_categories)

df_2022 = pd.concat([info_df_2022,df_values_2022], axis=1)

```


```python
#2021
values_2021 =[]
for athlete in r_2021['athletes']:
       for category in athlete['categories']:
           x = category['totals'][0:14]
           values_2021.append(x)

df_values_2021 = pd.DataFrame(values_2021, columns=table_headers)

#2022 athletes
info_categories_2021 = r_2021['athletes'][0]['athlete'].keys()
info_2021 = []

for athlete in r_2021['athletes']:
    row = []
    for category in info_categories_2021:
        if category in athlete['athlete']:
            row.append(athlete['athlete'][category])
        else:
            row.append(None)
    info_2021.append(row)
    
info_df_2021 = pd.DataFrame(info_2021, columns=info_categories)

df_2021 = pd.concat([info_df_2021,df_values_2021], axis=1)
```


```python
#combine all three dataframes using concat
golf_df_concat = pd.concat([df_2021, df_2022, df_2023])



```


```python
#pesky format on earnings, we'll have to remove commas and $ signs
golf_df_concat['Earnings'] = golf_df_concat['Earnings'].str.replace(',', '').str.replace('$', '')
```

    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:2: FutureWarning: The default value of regex will change from True to False in a future version. In addition, single character regular expressions will *not* be treated as literal strings when regex=True.
      



```python

```


```python
#make them numbers not strings
golf_df_concat[['Earnings', 'Cup', 'Evnts', 'Rnds', 'Cuts', 'Top10', 'Wins']] = golf_df_concat[['Earnings', 'Cup', 'Evnts', 'Rnds', 'Cuts', 'Top10', 'Wins']].apply(pd.to_numeric)
golf_df_concat['Score'] = pd.to_numeric(golf_df_concat['Score'], errors='coerce')
golf_df_concat['DDIS'] = pd.to_numeric(golf_df_concat['DDIS'], errors='coerce')
golf_df_concat['DACC'] = pd.to_numeric(golf_df_concat['DACC'], errors='coerce')
golf_df_concat['GIR'] = pd.to_numeric(golf_df_concat['GIR'], errors='coerce')
golf_df_concat['PUTTS'] = pd.to_numeric(golf_df_concat['PUTTS'], errors='coerce')
golf_df_concat['SAND'] = pd.to_numeric(golf_df_concat['SAND'], errors='coerce')
golf_df_concat['BIRDS'] = pd.to_numeric(golf_df_concat['BIRDS'], errors='coerce')


```


```python
golf_df_agg = golf_df_concat.groupby('displayName').agg({'Earnings':'sum','Cup':'sum','Evnts':'sum','Rnds':'sum','Cuts':'sum','Top10':'sum','Wins':'sum','Score':'mean','DDIS':'mean','DACC':'mean','GIR':'mean','PUTTS':'mean','SAND':'mean','BIRDS':'mean'})


```


```python

```


```python

```


```python
golf_df_agg.head()
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Aaron Wise</th>
      <td>3454521</td>
      <td>-12</td>
      <td>24</td>
      <td>86</td>
      <td>19</td>
      <td>4</td>
      <td>0</td>
      <td>69.40</td>
      <td>308.30</td>
      <td>59.2</td>
      <td>69.10</td>
      <td>1.748</td>
      <td>0.0</td>
      <td>43.00</td>
    </tr>
    <tr>
      <th>Abraham Ancer</th>
      <td>5816565</td>
      <td>-10</td>
      <td>27</td>
      <td>98</td>
      <td>24</td>
      <td>9</td>
      <td>1</td>
      <td>69.50</td>
      <td>290.40</td>
      <td>71.1</td>
      <td>69.30</td>
      <td>1.732</td>
      <td>0.0</td>
      <td>50.80</td>
    </tr>
    <tr>
      <th>Adam Hadwin</th>
      <td>4373841</td>
      <td>1182</td>
      <td>39</td>
      <td>134</td>
      <td>31</td>
      <td>8</td>
      <td>0</td>
      <td>69.95</td>
      <td>295.45</td>
      <td>63.8</td>
      <td>67.75</td>
      <td>1.746</td>
      <td>0.0</td>
      <td>55.35</td>
    </tr>
    <tr>
      <th>Adam Schenk</th>
      <td>1803965</td>
      <td>584</td>
      <td>17</td>
      <td>59</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.30</td>
      <td>304.40</td>
      <td>53.7</td>
      <td>66.10</td>
      <td>1.806</td>
      <td>0.0</td>
      <td>49.00</td>
    </tr>
    <tr>
      <th>Adam Scott</th>
      <td>2913198</td>
      <td>-4</td>
      <td>20</td>
      <td>72</td>
      <td>18</td>
      <td>5</td>
      <td>0</td>
      <td>70.10</td>
      <td>312.40</td>
      <td>55.3</td>
      <td>67.70</td>
      <td>1.742</td>
      <td>0.0</td>
      <td>52.70</td>
    </tr>
  </tbody>
</table>
</div>




```python
golf_df_agg.to_excel('golf_player_data.xlsx', index=True)
```


```python
#Great! We've saved it, let's check the data
golf_df_agg.head()
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Aaron Wise</th>
      <td>3454521</td>
      <td>-12</td>
      <td>24</td>
      <td>86</td>
      <td>19</td>
      <td>4</td>
      <td>0</td>
      <td>69.40</td>
      <td>308.30</td>
      <td>59.2</td>
      <td>69.10</td>
      <td>1.748</td>
      <td>0.0</td>
      <td>43.00</td>
    </tr>
    <tr>
      <th>Abraham Ancer</th>
      <td>5816565</td>
      <td>-10</td>
      <td>27</td>
      <td>98</td>
      <td>24</td>
      <td>9</td>
      <td>1</td>
      <td>69.50</td>
      <td>290.40</td>
      <td>71.1</td>
      <td>69.30</td>
      <td>1.732</td>
      <td>0.0</td>
      <td>50.80</td>
    </tr>
    <tr>
      <th>Adam Hadwin</th>
      <td>4373841</td>
      <td>1182</td>
      <td>39</td>
      <td>134</td>
      <td>31</td>
      <td>8</td>
      <td>0</td>
      <td>69.95</td>
      <td>295.45</td>
      <td>63.8</td>
      <td>67.75</td>
      <td>1.746</td>
      <td>0.0</td>
      <td>55.35</td>
    </tr>
    <tr>
      <th>Adam Schenk</th>
      <td>1803965</td>
      <td>584</td>
      <td>17</td>
      <td>59</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.30</td>
      <td>304.40</td>
      <td>53.7</td>
      <td>66.10</td>
      <td>1.806</td>
      <td>0.0</td>
      <td>49.00</td>
    </tr>
    <tr>
      <th>Adam Scott</th>
      <td>2913198</td>
      <td>-4</td>
      <td>20</td>
      <td>72</td>
      <td>18</td>
      <td>5</td>
      <td>0</td>
      <td>70.10</td>
      <td>312.40</td>
      <td>55.3</td>
      <td>67.70</td>
      <td>1.742</td>
      <td>0.0</td>
      <td>52.70</td>
    </tr>
  </tbody>
</table>
</div>




```python
golf_df_agg.tail()
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Viktor Hovland</th>
      <td>13376692</td>
      <td>616</td>
      <td>55</td>
      <td>189</td>
      <td>49</td>
      <td>16</td>
      <td>2</td>
      <td>69.533333</td>
      <td>305.333333</td>
      <td>63.133333</td>
      <td>67.766667</td>
      <td>1.732667</td>
      <td>0.0</td>
      <td>51.466667</td>
    </tr>
    <tr>
      <th>Webb Simpson</th>
      <td>2783012</td>
      <td>1195</td>
      <td>21</td>
      <td>74</td>
      <td>18</td>
      <td>6</td>
      <td>0</td>
      <td>69.200000</td>
      <td>292.500000</td>
      <td>67.100000</td>
      <td>69.400000</td>
      <td>1.734000</td>
      <td>0.0</td>
      <td>61.700000</td>
    </tr>
    <tr>
      <th>Will Zalatoris</th>
      <td>12892434</td>
      <td>3680</td>
      <td>49</td>
      <td>163</td>
      <td>38</td>
      <td>17</td>
      <td>1</td>
      <td>69.750000</td>
      <td>311.050000</td>
      <td>55.900000</td>
      <td>70.200000</td>
      <td>1.755500</td>
      <td>0.0</td>
      <td>48.550000</td>
    </tr>
    <tr>
      <th>Wyndham Clark</th>
      <td>1948049</td>
      <td>535</td>
      <td>15</td>
      <td>56</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.700000</td>
      <td>313.000000</td>
      <td>54.000000</td>
      <td>67.900000</td>
      <td>1.734000</td>
      <td>0.0</td>
      <td>52.800000</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
    </tr>
  </tbody>
</table>
</div>




```python
#what are the lowest/highest average scores
golf_df_agg.describe()
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>9.400000e+01</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.0</td>
      <td>94.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>6.487320e+06</td>
      <td>961.202128</td>
      <td>32.606383</td>
      <td>108.244681</td>
      <td>25.095745</td>
      <td>7.851064</td>
      <td>1.106383</td>
      <td>69.500532</td>
      <td>299.934752</td>
      <td>60.095213</td>
      <td>66.925532</td>
      <td>1.733780</td>
      <td>0.0</td>
      <td>53.157624</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.464426e+06</td>
      <td>809.484378</td>
      <td>16.847972</td>
      <td>57.614717</td>
      <td>14.139147</td>
      <td>6.058776</td>
      <td>1.425332</td>
      <td>3.678197</td>
      <td>18.006235</td>
      <td>5.070876</td>
      <td>3.961661</td>
      <td>0.093496</td>
      <td>0.0</td>
      <td>6.838573</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.596177e+06</td>
      <td>-18.000000</td>
      <td>10.000000</td>
      <td>32.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>34.550000</td>
      <td>147.550000</td>
      <td>34.850000</td>
      <td>35.250000</td>
      <td>0.868500</td>
      <td>0.0</td>
      <td>27.900000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.787208e+06</td>
      <td>486.750000</td>
      <td>20.000000</td>
      <td>59.750000</td>
      <td>13.250000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>69.600000</td>
      <td>295.925000</td>
      <td>56.875000</td>
      <td>66.100000</td>
      <td>1.728000</td>
      <td>0.0</td>
      <td>48.700000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.316210e+06</td>
      <td>779.000000</td>
      <td>27.000000</td>
      <td>88.500000</td>
      <td>20.000000</td>
      <td>6.000000</td>
      <td>1.000000</td>
      <td>69.800000</td>
      <td>302.475000</td>
      <td>60.400000</td>
      <td>67.175000</td>
      <td>1.740750</td>
      <td>0.0</td>
      <td>52.875000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.833034e+06</td>
      <td>1285.750000</td>
      <td>46.750000</td>
      <td>153.500000</td>
      <td>36.500000</td>
      <td>10.000000</td>
      <td>1.000000</td>
      <td>70.100000</td>
      <td>306.700000</td>
      <td>63.183333</td>
      <td>68.750000</td>
      <td>1.755500</td>
      <td>0.0</td>
      <td>57.662500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.018399e+07</td>
      <td>3680.000000</td>
      <td>74.000000</td>
      <td>256.000000</td>
      <td>63.000000</td>
      <td>29.000000</td>
      <td>6.000000</td>
      <td>71.900000</td>
      <td>323.700000</td>
      <td>71.100000</td>
      <td>72.100000</td>
      <td>1.807000</td>
      <td>0.0</td>
      <td>76.900000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#someone avereaged a 34.55 score? That's not right, the lowest score ever shot was 59 by Furyk
#let's find that entry
unreal_golfer = golf_df_agg[golf_df_agg['Score'] == 34.55]
print(unreal_golfer)
```

                 Earnings   Cup  Evnts  Rnds  Cuts  Top10  Wins  Score    DDIS  \
    displayName                                                                  
    Tom Kim       5801346  2018     22    38    20      6     2  34.55  147.55   
    
                  DACC    GIR   PUTTS  SAND  BIRDS  
    displayName                                     
    Tom Kim      34.85  35.25  0.8685   0.0   27.9  



```python
#Tom Kim is good... but he's not that good. Let's go back to ESPN to see what happened
unreal_golfer_2023 = df_2023[df_2023['displayName'] == 'Tom Kim']
print(unreal_golfer_2023)
```

             id               uid                                  guid firstName  \
    21  4602673  s:1100~a:4602673  cce7aed3-de73-3aef-886c-688ba5f9ea88       Tom   
    
       lastName displayName shortName  debutYear  \
    21      Kim     Tom Kim    T. Kim        NaN   
    
                                                    links  \
    21  [{'language': 'en-US', 'rel': ['playercard', '...   
    
                                                 headshot  \
    21  {'href': 'https://a.espncdn.com/i/headshots/go...   
    
                                                   status  age  \
    21  {'id': '1', 'name': 'Active', 'type': 'active'...   20   
    
                                                     flag    Earnings  Cup Evnts  \
    21  {'href': 'https://a.espncdn.com/i/teamlogos/co...  $2,976,766  864    11   
    
       Rnds Cuts Top10 Wins Score   DDIS  DACC   GIR  PUTTS SAND BIRDS  
    21   38   10     3    1  69.1  295.1  69.7  70.5  1.737    0  55.8  



```python
#hm, Tom's averaging 69.1 in 2023. Let's check 2021-2022.
unreal_golfer_2022 = df_2022[df_2022['displayName'] == 'Tom Kim']
print(unreal_golfer_2022)
```

             id               uid                                  guid firstName  \
    44  4602673  s:1100~a:4602673  cce7aed3-de73-3aef-886c-688ba5f9ea88       Tom   
    
       lastName displayName shortName  debutYear  \
    44      Kim     Tom Kim    T. Kim        NaN   
    
                                                    links  \
    44  [{'language': 'en-US', 'rel': ['playercard', '...   
    
                                                 headshot  \
    44  {'href': 'https://a.espncdn.com/i/headshots/go...   
    
                                                   status  age  \
    44  {'id': '1', 'name': 'Active', 'type': 'active'...   20   
    
                                                     flag    Earnings   Cup Evnts  \
    44  {'href': 'https://a.espncdn.com/i/teamlogos/co...  $2,824,580  1154    11   
    
       Rnds Cuts Top10 Wins Score DDIS DACC  GIR  PUTTS SAND BIRDS  
    44    0   10     3    1  00.0  0.0  0.0  0.0  0.000    0   0.0  



```python
#let's fix Tom's data
golf_df_agg.loc['Tom Kim']
```




    Earnings    5.801346e+06
    Cup         2.018000e+03
    Evnts       2.200000e+01
    Rnds        3.800000e+01
    Cuts        2.000000e+01
    Top10       6.000000e+00
    Wins        2.000000e+00
    Score       3.455000e+01
    DDIS        1.475500e+02
    DACC        3.485000e+01
    GIR         3.525000e+01
    PUTTS       8.685000e-01
    SAND        0.000000e+00
    BIRDS       2.790000e+01
    Name: Tom Kim, dtype: float64




```python
golf_df_agg.loc['Tom Kim', "Score"] = 69.1
golf_df_agg.loc['Tom Kim', "DDIS"] = 295.1
golf_df_agg.loc['Tom Kim', "DACC"] = 69.7
golf_df_agg.loc['Tom Kim', "GIR"] = 70.5
golf_df_agg.loc['Tom Kim', "PUTTS"] = 1.737
golf_df_agg.loc['Tom Kim', "SAND"] = 0
golf_df_agg.loc['Tom Kim', "BIRDS"] = 55.8
```


```python
#let's check Tom's updated stats
golf_df_agg.loc['Tom Kim']
```




    Earnings    5801346.000
    Cup            2018.000
    Evnts            22.000
    Rnds             38.000
    Cuts             20.000
    Top10             6.000
    Wins              2.000
    Score            69.100
    DDIS            295.100
    DACC             69.700
    GIR              70.500
    PUTTS             1.737
    SAND              0.000
    BIRDS            55.800
    Name: Tom Kim, dtype: float64




```python
golf_df_agg.describe()
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>9.400000e+01</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.000000</td>
      <td>94.0</td>
      <td>94.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>6.487320e+06</td>
      <td>961.202128</td>
      <td>32.606383</td>
      <td>108.244681</td>
      <td>25.095745</td>
      <td>7.851064</td>
      <td>1.106383</td>
      <td>69.868085</td>
      <td>301.504433</td>
      <td>60.465957</td>
      <td>67.300532</td>
      <td>1.743020</td>
      <td>0.0</td>
      <td>53.454433</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.464426e+06</td>
      <td>809.484378</td>
      <td>16.847972</td>
      <td>57.614717</td>
      <td>14.139147</td>
      <td>6.058776</td>
      <td>1.425332</td>
      <td>0.509384</td>
      <td>8.502709</td>
      <td>4.440033</td>
      <td>2.213913</td>
      <td>0.024592</td>
      <td>0.0</td>
      <td>6.316048</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.596177e+06</td>
      <td>-18.000000</td>
      <td>10.000000</td>
      <td>32.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>68.866667</td>
      <td>275.000000</td>
      <td>50.800000</td>
      <td>57.800000</td>
      <td>1.647000</td>
      <td>0.0</td>
      <td>40.900000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.787208e+06</td>
      <td>486.750000</td>
      <td>20.000000</td>
      <td>59.750000</td>
      <td>13.250000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>69.600000</td>
      <td>295.925000</td>
      <td>57.125000</td>
      <td>66.100000</td>
      <td>1.728167</td>
      <td>0.0</td>
      <td>49.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.316210e+06</td>
      <td>779.000000</td>
      <td>27.000000</td>
      <td>88.500000</td>
      <td>20.000000</td>
      <td>6.000000</td>
      <td>1.000000</td>
      <td>69.800000</td>
      <td>302.475000</td>
      <td>60.400000</td>
      <td>67.200000</td>
      <td>1.740750</td>
      <td>0.0</td>
      <td>52.975000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.833034e+06</td>
      <td>1285.750000</td>
      <td>46.750000</td>
      <td>153.500000</td>
      <td>36.500000</td>
      <td>10.000000</td>
      <td>1.000000</td>
      <td>70.100000</td>
      <td>306.700000</td>
      <td>63.275000</td>
      <td>68.975000</td>
      <td>1.755500</td>
      <td>0.0</td>
      <td>57.662500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.018399e+07</td>
      <td>3680.000000</td>
      <td>74.000000</td>
      <td>256.000000</td>
      <td>63.000000</td>
      <td>29.000000</td>
      <td>6.000000</td>
      <td>71.900000</td>
      <td>323.700000</td>
      <td>71.100000</td>
      <td>72.100000</td>
      <td>1.807000</td>
      <td>0.0</td>
      <td>76.900000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_2023_2 = df_2023.set_index('displayName')
```


```python

```


```python


```


```python
## we still have a bit of cleaning/processing
#we have to set a boolean for "American"
#we could do a "nationality" column, but a boolean will get the job done for our business task
```


```python

```


```python
#create a new column for golfers who are American and currently in the top 50
American_names = ['Scottie Scheffler','Max Homa','Kurt Kitayama','Keegan Bradley','Tom Hoge','Collin Morikawa','Patrick Cantlay','Tony Finau','Sahith Theegala','Rickie Fowler','Taylor Moore','Chris Kirk','Harris English','Jordan Spieth','Brian Harman','Taylor Montgomery','Russel Henley','Justin Thomas','Keith Mitchell','Andrew Putnam','Xander Schauffele','Wyndham Clark','Adam Schenk','Sam Burns','Denny McCarthy','Brendon Todd','Hayden Buckley','Justin Suh','Brandon Wu','Davis Thompson']


golf_df_agg['is_qualified'] = False  
golf_df_agg.loc[golf_df_agg.index.isin(American_names), 'is_qualified'] = True
```


```python
#remove all unqualified candidates, also first cut is a pun
first_cut = golf_df_agg[golf_df_agg['is_qualified'] == True]
first_cut
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Adam Schenk</th>
      <td>1803965</td>
      <td>584</td>
      <td>17</td>
      <td>59</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.300000</td>
      <td>304.400000</td>
      <td>53.700000</td>
      <td>66.100000</td>
      <td>1.806000</td>
      <td>0.0</td>
      <td>49.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2259018</td>
      <td>654</td>
      <td>16</td>
      <td>53</td>
      <td>12</td>
      <td>3</td>
      <td>0</td>
      <td>69.400000</td>
      <td>282.100000</td>
      <td>64.000000</td>
      <td>68.600000</td>
      <td>1.754000</td>
      <td>0.0</td>
      <td>63.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1720861</td>
      <td>516</td>
      <td>13</td>
      <td>43</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.700000</td>
      <td>280.200000</td>
      <td>68.200000</td>
      <td>66.100000</td>
      <td>1.715000</td>
      <td>0.0</td>
      <td>60.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>68</td>
      <td>232</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.700000</td>
      <td>292.433333</td>
      <td>67.133333</td>
      <td>66.966667</td>
      <td>1.753667</td>
      <td>0.0</td>
      <td>60.833333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Chris Kirk</th>
      <td>2783635</td>
      <td>916</td>
      <td>13</td>
      <td>40</td>
      <td>9</td>
      <td>3</td>
      <td>1</td>
      <td>69.600000</td>
      <td>295.900000</td>
      <td>60.900000</td>
      <td>67.900000</td>
      <td>1.689000</td>
      <td>0.0</td>
      <td>60.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>1591</td>
      <td>45</td>
      <td>154</td>
      <td>35</td>
      <td>7</td>
      <td>0</td>
      <td>69.700000</td>
      <td>293.300000</td>
      <td>61.850000</td>
      <td>65.900000</td>
      <td>1.740500</td>
      <td>0.0</td>
      <td>55.350000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Harris English</th>
      <td>8996896</td>
      <td>516</td>
      <td>42</td>
      <td>141</td>
      <td>32</td>
      <td>10</td>
      <td>2</td>
      <td>69.900000</td>
      <td>297.000000</td>
      <td>61.750000</td>
      <td>64.950000</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>48.350000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1629364</td>
      <td>529</td>
      <td>13</td>
      <td>40</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
      <td>69.800000</td>
      <td>303.900000</td>
      <td>65.300000</td>
      <td>70.800000</td>
      <td>1.761000</td>
      <td>0.0</td>
      <td>43.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14114548</td>
      <td>485</td>
      <td>57</td>
      <td>194</td>
      <td>46</td>
      <td>18</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.533333</td>
      <td>54.366667</td>
      <td>66.333333</td>
      <td>1.730667</td>
      <td>0.0</td>
      <td>56.100000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Suh</th>
      <td>1675331</td>
      <td>367</td>
      <td>17</td>
      <td>54</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>70.200000</td>
      <td>293.800000</td>
      <td>63.000000</td>
      <td>67.200000</td>
      <td>1.740000</td>
      <td>0.0</td>
      <td>44.600000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>52</td>
      <td>192</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>307.900000</td>
      <td>55.500000</td>
      <td>67.066667</td>
      <td>1.726000</td>
      <td>0.0</td>
      <td>60.266667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keegan Bradley</th>
      <td>10624750</td>
      <td>3263</td>
      <td>63</td>
      <td>200</td>
      <td>44</td>
      <td>14</td>
      <td>1</td>
      <td>70.066667</td>
      <td>301.966667</td>
      <td>61.433333</td>
      <td>68.000000</td>
      <td>1.753333</td>
      <td>0.0</td>
      <td>44.533333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5050520</td>
      <td>1505</td>
      <td>39</td>
      <td>130</td>
      <td>31</td>
      <td>9</td>
      <td>0</td>
      <td>69.450000</td>
      <td>311.250000</td>
      <td>63.700000</td>
      <td>67.450000</td>
      <td>1.746000</td>
      <td>0.0</td>
      <td>55.500000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Kurt Kitayama</th>
      <td>5693388</td>
      <td>1040</td>
      <td>11</td>
      <td>32</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>70.100000</td>
      <td>306.700000</td>
      <td>54.400000</td>
      <td>67.400000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>61.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Max Homa</th>
      <td>16447832</td>
      <td>3039</td>
      <td>62</td>
      <td>204</td>
      <td>49</td>
      <td>16</td>
      <td>5</td>
      <td>69.766667</td>
      <td>302.933333</td>
      <td>60.400000</td>
      <td>66.200000</td>
      <td>1.721667</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Rickie Fowler</th>
      <td>3003047</td>
      <td>676</td>
      <td>11</td>
      <td>38</td>
      <td>10</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>305.000000</td>
      <td>58.700000</td>
      <td>69.300000</td>
      <td>1.709000</td>
      <td>0.0</td>
      <td>43.800000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sahith Theegala</th>
      <td>6377820</td>
      <td>782</td>
      <td>46</td>
      <td>163</td>
      <td>39</td>
      <td>10</td>
      <td>0</td>
      <td>69.950000</td>
      <td>302.350000</td>
      <td>53.600000</td>
      <td>67.150000</td>
      <td>1.732500</td>
      <td>0.0</td>
      <td>52.600000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Scottie Scheffler</th>
      <td>30183994</td>
      <td>1747</td>
      <td>64</td>
      <td>218</td>
      <td>54</td>
      <td>26</td>
      <td>6</td>
      <td>68.933333</td>
      <td>308.433333</td>
      <td>62.266667</td>
      <td>71.400000</td>
      <td>1.718000</td>
      <td>0.0</td>
      <td>44.500000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2299369</td>
      <td>746</td>
      <td>15</td>
      <td>52</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>306.000000</td>
      <td>56.500000</td>
      <td>63.600000</td>
      <td>1.647000</td>
      <td>0.0</td>
      <td>53.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2745877</td>
      <td>847</td>
      <td>15</td>
      <td>51</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>70.100000</td>
      <td>300.800000</td>
      <td>56.800000</td>
      <td>66.000000</td>
      <td>1.789000</td>
      <td>0.0</td>
      <td>49.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tom Hoge</th>
      <td>8109202</td>
      <td>741</td>
      <td>47</td>
      <td>144</td>
      <td>30</td>
      <td>10</td>
      <td>1</td>
      <td>69.450000</td>
      <td>296.000000</td>
      <td>62.750000</td>
      <td>68.350000</td>
      <td>1.725500</td>
      <td>0.0</td>
      <td>51.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tony Finau</th>
      <td>15309084</td>
      <td>916</td>
      <td>62</td>
      <td>210</td>
      <td>51</td>
      <td>18</td>
      <td>4</td>
      <td>69.333333</td>
      <td>305.033333</td>
      <td>57.833333</td>
      <td>69.666667</td>
      <td>1.723667</td>
      <td>0.0</td>
      <td>54.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Wyndham Clark</th>
      <td>1948049</td>
      <td>535</td>
      <td>15</td>
      <td>56</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.700000</td>
      <td>313.000000</td>
      <td>54.000000</td>
      <td>67.900000</td>
      <td>1.734000</td>
      <td>0.0</td>
      <td>52.800000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
first_cut.describe()
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.700000e+01</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.000000</td>
      <td>27.0</td>
      <td>27.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>8.914718e+06</td>
      <td>1088.000000</td>
      <td>37.777778</td>
      <td>126.518519</td>
      <td>29.740741</td>
      <td>10.370370</td>
      <td>1.555556</td>
      <td>69.662346</td>
      <td>301.108642</td>
      <td>60.140123</td>
      <td>67.656790</td>
      <td>1.734031</td>
      <td>0.0</td>
      <td>53.122222</td>
    </tr>
    <tr>
      <th>std</th>
      <td>7.405417e+06</td>
      <td>828.340185</td>
      <td>21.051829</td>
      <td>70.972568</td>
      <td>17.463352</td>
      <td>7.771487</td>
      <td>1.987138</td>
      <td>0.339639</td>
      <td>7.920723</td>
      <td>4.464373</td>
      <td>1.853982</td>
      <td>0.029456</td>
      <td>0.0</td>
      <td>6.073986</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.629364e+06</td>
      <td>345.000000</td>
      <td>11.000000</td>
      <td>32.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>68.933333</td>
      <td>280.200000</td>
      <td>53.600000</td>
      <td>63.600000</td>
      <td>1.647000</td>
      <td>0.0</td>
      <td>43.300000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.522623e+06</td>
      <td>532.000000</td>
      <td>15.000000</td>
      <td>52.500000</td>
      <td>12.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>69.450000</td>
      <td>296.216667</td>
      <td>56.650000</td>
      <td>66.266667</td>
      <td>1.722167</td>
      <td>0.0</td>
      <td>49.200000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>6.377820e+06</td>
      <td>746.000000</td>
      <td>45.000000</td>
      <td>144.000000</td>
      <td>32.000000</td>
      <td>10.000000</td>
      <td>1.000000</td>
      <td>69.700000</td>
      <td>303.533333</td>
      <td>60.900000</td>
      <td>67.400000</td>
      <td>1.732500</td>
      <td>0.0</td>
      <td>52.800000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.524300e+07</td>
      <td>1272.500000</td>
      <td>54.500000</td>
      <td>193.000000</td>
      <td>45.000000</td>
      <td>18.000000</td>
      <td>2.000000</td>
      <td>69.883333</td>
      <td>306.166667</td>
      <td>62.875000</td>
      <td>68.816667</td>
      <td>1.749667</td>
      <td>0.0</td>
      <td>58.050000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.018399e+07</td>
      <td>3263.000000</td>
      <td>68.000000</td>
      <td>232.000000</td>
      <td>54.000000</td>
      <td>26.000000</td>
      <td>6.000000</td>
      <td>70.300000</td>
      <td>313.000000</td>
      <td>68.333333</td>
      <td>71.400000</td>
      <td>1.806000</td>
      <td>0.0</td>
      <td>63.400000</td>
    </tr>
  </tbody>
</table>
</div>




```python


```


```python
# 6 golfers automatically qualify for points, so if points were taken today Mar21,2023, this is who would auto qual
first_six = first_cut.nlargest(6,'Cup')
first_six
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Keegan Bradley</th>
      <td>10624750</td>
      <td>3263</td>
      <td>63</td>
      <td>200</td>
      <td>44</td>
      <td>14</td>
      <td>1</td>
      <td>70.066667</td>
      <td>301.966667</td>
      <td>61.433333</td>
      <td>68.000000</td>
      <td>1.753333</td>
      <td>0.0</td>
      <td>44.533333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Max Homa</th>
      <td>16447832</td>
      <td>3039</td>
      <td>62</td>
      <td>204</td>
      <td>49</td>
      <td>16</td>
      <td>5</td>
      <td>69.766667</td>
      <td>302.933333</td>
      <td>60.400000</td>
      <td>66.200000</td>
      <td>1.721667</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>68</td>
      <td>232</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.700000</td>
      <td>292.433333</td>
      <td>67.133333</td>
      <td>66.966667</td>
      <td>1.753667</td>
      <td>0.0</td>
      <td>60.833333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Scottie Scheffler</th>
      <td>30183994</td>
      <td>1747</td>
      <td>64</td>
      <td>218</td>
      <td>54</td>
      <td>26</td>
      <td>6</td>
      <td>68.933333</td>
      <td>308.433333</td>
      <td>62.266667</td>
      <td>71.400000</td>
      <td>1.718000</td>
      <td>0.0</td>
      <td>44.500000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>1591</td>
      <td>45</td>
      <td>154</td>
      <td>35</td>
      <td>7</td>
      <td>0</td>
      <td>69.700000</td>
      <td>293.300000</td>
      <td>61.850000</td>
      <td>65.900000</td>
      <td>1.740500</td>
      <td>0.0</td>
      <td>55.350000</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
# remove the top 6 from the choices
USA_team = ['Max Homa','Scottie Scheffler','Keegan Bradley','Kurt Kitayama','Chris Kirk','Tony Finau']
# select rows where displayName is not in USA_team
second_cut = first_cut[~first_cut.index.isin(USA_team)]
second_cut
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Adam Schenk</th>
      <td>1803965</td>
      <td>584</td>
      <td>17</td>
      <td>59</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.300000</td>
      <td>304.400000</td>
      <td>53.700000</td>
      <td>66.100000</td>
      <td>1.806000</td>
      <td>0.0</td>
      <td>49.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2259018</td>
      <td>654</td>
      <td>16</td>
      <td>53</td>
      <td>12</td>
      <td>3</td>
      <td>0</td>
      <td>69.400000</td>
      <td>282.100000</td>
      <td>64.000000</td>
      <td>68.600000</td>
      <td>1.754000</td>
      <td>0.0</td>
      <td>63.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1720861</td>
      <td>516</td>
      <td>13</td>
      <td>43</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.700000</td>
      <td>280.200000</td>
      <td>68.200000</td>
      <td>66.100000</td>
      <td>1.715000</td>
      <td>0.0</td>
      <td>60.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>68</td>
      <td>232</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.700000</td>
      <td>292.433333</td>
      <td>67.133333</td>
      <td>66.966667</td>
      <td>1.753667</td>
      <td>0.0</td>
      <td>60.833333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>1591</td>
      <td>45</td>
      <td>154</td>
      <td>35</td>
      <td>7</td>
      <td>0</td>
      <td>69.700000</td>
      <td>293.300000</td>
      <td>61.850000</td>
      <td>65.900000</td>
      <td>1.740500</td>
      <td>0.0</td>
      <td>55.350000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Harris English</th>
      <td>8996896</td>
      <td>516</td>
      <td>42</td>
      <td>141</td>
      <td>32</td>
      <td>10</td>
      <td>2</td>
      <td>69.900000</td>
      <td>297.000000</td>
      <td>61.750000</td>
      <td>64.950000</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>48.350000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1629364</td>
      <td>529</td>
      <td>13</td>
      <td>40</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
      <td>69.800000</td>
      <td>303.900000</td>
      <td>65.300000</td>
      <td>70.800000</td>
      <td>1.761000</td>
      <td>0.0</td>
      <td>43.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14114548</td>
      <td>485</td>
      <td>57</td>
      <td>194</td>
      <td>46</td>
      <td>18</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.533333</td>
      <td>54.366667</td>
      <td>66.333333</td>
      <td>1.730667</td>
      <td>0.0</td>
      <td>56.100000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Suh</th>
      <td>1675331</td>
      <td>367</td>
      <td>17</td>
      <td>54</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>70.200000</td>
      <td>293.800000</td>
      <td>63.000000</td>
      <td>67.200000</td>
      <td>1.740000</td>
      <td>0.0</td>
      <td>44.600000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>52</td>
      <td>192</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>307.900000</td>
      <td>55.500000</td>
      <td>67.066667</td>
      <td>1.726000</td>
      <td>0.0</td>
      <td>60.266667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5050520</td>
      <td>1505</td>
      <td>39</td>
      <td>130</td>
      <td>31</td>
      <td>9</td>
      <td>0</td>
      <td>69.450000</td>
      <td>311.250000</td>
      <td>63.700000</td>
      <td>67.450000</td>
      <td>1.746000</td>
      <td>0.0</td>
      <td>55.500000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Rickie Fowler</th>
      <td>3003047</td>
      <td>676</td>
      <td>11</td>
      <td>38</td>
      <td>10</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>305.000000</td>
      <td>58.700000</td>
      <td>69.300000</td>
      <td>1.709000</td>
      <td>0.0</td>
      <td>43.800000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sahith Theegala</th>
      <td>6377820</td>
      <td>782</td>
      <td>46</td>
      <td>163</td>
      <td>39</td>
      <td>10</td>
      <td>0</td>
      <td>69.950000</td>
      <td>302.350000</td>
      <td>53.600000</td>
      <td>67.150000</td>
      <td>1.732500</td>
      <td>0.0</td>
      <td>52.600000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2299369</td>
      <td>746</td>
      <td>15</td>
      <td>52</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>306.000000</td>
      <td>56.500000</td>
      <td>63.600000</td>
      <td>1.647000</td>
      <td>0.0</td>
      <td>53.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2745877</td>
      <td>847</td>
      <td>15</td>
      <td>51</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>70.100000</td>
      <td>300.800000</td>
      <td>56.800000</td>
      <td>66.000000</td>
      <td>1.789000</td>
      <td>0.0</td>
      <td>49.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tom Hoge</th>
      <td>8109202</td>
      <td>741</td>
      <td>47</td>
      <td>144</td>
      <td>30</td>
      <td>10</td>
      <td>1</td>
      <td>69.450000</td>
      <td>296.000000</td>
      <td>62.750000</td>
      <td>68.350000</td>
      <td>1.725500</td>
      <td>0.0</td>
      <td>51.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Wyndham Clark</th>
      <td>1948049</td>
      <td>535</td>
      <td>15</td>
      <td>56</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.700000</td>
      <td>313.000000</td>
      <td>54.000000</td>
      <td>67.900000</td>
      <td>1.734000</td>
      <td>0.0</td>
      <td>52.800000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
#highest earners
top_earners = second_cut.nlargest(6, 'Earnings')
top_earners

```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>52</td>
      <td>192</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>307.900000</td>
      <td>55.500000</td>
      <td>67.066667</td>
      <td>1.726000</td>
      <td>0.0</td>
      <td>60.266667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14114548</td>
      <td>485</td>
      <td>57</td>
      <td>194</td>
      <td>46</td>
      <td>18</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.533333</td>
      <td>54.366667</td>
      <td>66.333333</td>
      <td>1.730667</td>
      <td>0.0</td>
      <td>56.100000</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python

```


```python
second_cut['Earnings_per_event'] = second_cut['Earnings'] / second_cut['Evnts']
second_cut['Points_per_event'] = second_cut['Cup'] / second_cut['Evnts']
second_cut['Top10_per_event'] = second_cut['Top10'] / second_cut['Evnts']
second_cut['Wins_per_event'] = second_cut['Wins'] / second_cut['Evnts']

second_cut
```

    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      """Entry point for launching an IPython kernel.
    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      
    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:3: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      This is separate from the ipykernel package so we can avoid doing imports until
    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:4: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      after removing the cwd from sys.path.





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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Adam Schenk</th>
      <td>1803965</td>
      <td>584</td>
      <td>17</td>
      <td>59</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.300000</td>
      <td>304.400000</td>
      <td>53.700000</td>
      <td>66.100000</td>
      <td>1.806000</td>
      <td>0.0</td>
      <td>49.000000</td>
      <td>True</td>
      <td>106115.588235</td>
      <td>34.352941</td>
      <td>0.058824</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2259018</td>
      <td>654</td>
      <td>16</td>
      <td>53</td>
      <td>12</td>
      <td>3</td>
      <td>0</td>
      <td>69.400000</td>
      <td>282.100000</td>
      <td>64.000000</td>
      <td>68.600000</td>
      <td>1.754000</td>
      <td>0.0</td>
      <td>63.400000</td>
      <td>True</td>
      <td>141188.625000</td>
      <td>40.875000</td>
      <td>0.187500</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1720861</td>
      <td>516</td>
      <td>13</td>
      <td>43</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.700000</td>
      <td>280.200000</td>
      <td>68.200000</td>
      <td>66.100000</td>
      <td>1.715000</td>
      <td>0.0</td>
      <td>60.300000</td>
      <td>True</td>
      <td>132373.923077</td>
      <td>39.692308</td>
      <td>0.230769</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>68</td>
      <td>232</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.700000</td>
      <td>292.433333</td>
      <td>67.133333</td>
      <td>66.966667</td>
      <td>1.753667</td>
      <td>0.0</td>
      <td>60.833333</td>
      <td>True</td>
      <td>129652.161765</td>
      <td>27.926471</td>
      <td>0.191176</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>1591</td>
      <td>45</td>
      <td>154</td>
      <td>35</td>
      <td>7</td>
      <td>0</td>
      <td>69.700000</td>
      <td>293.300000</td>
      <td>61.850000</td>
      <td>65.900000</td>
      <td>1.740500</td>
      <td>0.0</td>
      <td>55.350000</td>
      <td>True</td>
      <td>104291.688889</td>
      <td>35.355556</td>
      <td>0.155556</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Harris English</th>
      <td>8996896</td>
      <td>516</td>
      <td>42</td>
      <td>141</td>
      <td>32</td>
      <td>10</td>
      <td>2</td>
      <td>69.900000</td>
      <td>297.000000</td>
      <td>61.750000</td>
      <td>64.950000</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>48.350000</td>
      <td>True</td>
      <td>214211.809524</td>
      <td>12.285714</td>
      <td>0.238095</td>
      <td>0.047619</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1629364</td>
      <td>529</td>
      <td>13</td>
      <td>40</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
      <td>69.800000</td>
      <td>303.900000</td>
      <td>65.300000</td>
      <td>70.800000</td>
      <td>1.761000</td>
      <td>0.0</td>
      <td>43.300000</td>
      <td>True</td>
      <td>125335.692308</td>
      <td>40.692308</td>
      <td>0.153846</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14114548</td>
      <td>485</td>
      <td>57</td>
      <td>194</td>
      <td>46</td>
      <td>18</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.533333</td>
      <td>54.366667</td>
      <td>66.333333</td>
      <td>1.730667</td>
      <td>0.0</td>
      <td>56.100000</td>
      <td>True</td>
      <td>247623.649123</td>
      <td>8.508772</td>
      <td>0.315789</td>
      <td>0.035088</td>
    </tr>
    <tr>
      <th>Justin Suh</th>
      <td>1675331</td>
      <td>367</td>
      <td>17</td>
      <td>54</td>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>70.200000</td>
      <td>293.800000</td>
      <td>63.000000</td>
      <td>67.200000</td>
      <td>1.740000</td>
      <td>0.0</td>
      <td>44.600000</td>
      <td>True</td>
      <td>98548.882353</td>
      <td>21.588235</td>
      <td>0.117647</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>52</td>
      <td>192</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>307.900000</td>
      <td>55.500000</td>
      <td>67.066667</td>
      <td>1.726000</td>
      <td>0.0</td>
      <td>60.266667</td>
      <td>True</td>
      <td>294733.884615</td>
      <td>6.634615</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5050520</td>
      <td>1505</td>
      <td>39</td>
      <td>130</td>
      <td>31</td>
      <td>9</td>
      <td>0</td>
      <td>69.450000</td>
      <td>311.250000</td>
      <td>63.700000</td>
      <td>67.450000</td>
      <td>1.746000</td>
      <td>0.0</td>
      <td>55.500000</td>
      <td>True</td>
      <td>129500.512821</td>
      <td>38.589744</td>
      <td>0.230769</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
      <td>400765.576923</td>
      <td>13.884615</td>
      <td>0.442308</td>
      <td>0.115385</td>
    </tr>
    <tr>
      <th>Rickie Fowler</th>
      <td>3003047</td>
      <td>676</td>
      <td>11</td>
      <td>38</td>
      <td>10</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>305.000000</td>
      <td>58.700000</td>
      <td>69.300000</td>
      <td>1.709000</td>
      <td>0.0</td>
      <td>43.800000</td>
      <td>True</td>
      <td>273004.272727</td>
      <td>61.454545</td>
      <td>0.272727</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Sahith Theegala</th>
      <td>6377820</td>
      <td>782</td>
      <td>46</td>
      <td>163</td>
      <td>39</td>
      <td>10</td>
      <td>0</td>
      <td>69.950000</td>
      <td>302.350000</td>
      <td>53.600000</td>
      <td>67.150000</td>
      <td>1.732500</td>
      <td>0.0</td>
      <td>52.600000</td>
      <td>True</td>
      <td>138648.260870</td>
      <td>17.000000</td>
      <td>0.217391</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
      <td>287451.754098</td>
      <td>15.409836</td>
      <td>0.327869</td>
      <td>0.081967</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2299369</td>
      <td>746</td>
      <td>15</td>
      <td>52</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>306.000000</td>
      <td>56.500000</td>
      <td>63.600000</td>
      <td>1.647000</td>
      <td>0.0</td>
      <td>53.300000</td>
      <td>True</td>
      <td>153291.266667</td>
      <td>49.733333</td>
      <td>0.266667</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2745877</td>
      <td>847</td>
      <td>15</td>
      <td>51</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>70.100000</td>
      <td>300.800000</td>
      <td>56.800000</td>
      <td>66.000000</td>
      <td>1.789000</td>
      <td>0.0</td>
      <td>49.400000</td>
      <td>True</td>
      <td>183058.466667</td>
      <td>56.466667</td>
      <td>0.066667</td>
      <td>0.066667</td>
    </tr>
    <tr>
      <th>Tom Hoge</th>
      <td>8109202</td>
      <td>741</td>
      <td>47</td>
      <td>144</td>
      <td>30</td>
      <td>10</td>
      <td>1</td>
      <td>69.450000</td>
      <td>296.000000</td>
      <td>62.750000</td>
      <td>68.350000</td>
      <td>1.725500</td>
      <td>0.0</td>
      <td>51.000000</td>
      <td>True</td>
      <td>172536.212766</td>
      <td>15.765957</td>
      <td>0.212766</td>
      <td>0.021277</td>
    </tr>
    <tr>
      <th>Wyndham Clark</th>
      <td>1948049</td>
      <td>535</td>
      <td>15</td>
      <td>56</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.700000</td>
      <td>313.000000</td>
      <td>54.000000</td>
      <td>67.900000</td>
      <td>1.734000</td>
      <td>0.0</td>
      <td>52.800000</td>
      <td>True</td>
      <td>129869.933333</td>
      <td>35.666667</td>
      <td>0.266667</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
      <td>291863.923077</td>
      <td>9.807692</td>
      <td>0.365385</td>
      <td>0.057692</td>
    </tr>
  </tbody>
</table>
</div>




```python
#top points
top_points = second_cut.nlargest(6, 'Cup')
top_points
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>68</td>
      <td>232</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.700000</td>
      <td>292.433333</td>
      <td>67.133333</td>
      <td>66.966667</td>
      <td>1.753667</td>
      <td>0.0</td>
      <td>60.833333</td>
      <td>True</td>
      <td>129652.161765</td>
      <td>27.926471</td>
      <td>0.191176</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>1591</td>
      <td>45</td>
      <td>154</td>
      <td>35</td>
      <td>7</td>
      <td>0</td>
      <td>69.700000</td>
      <td>293.300000</td>
      <td>61.850000</td>
      <td>65.900000</td>
      <td>1.740500</td>
      <td>0.0</td>
      <td>55.350000</td>
      <td>True</td>
      <td>104291.688889</td>
      <td>35.355556</td>
      <td>0.155556</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5050520</td>
      <td>1505</td>
      <td>39</td>
      <td>130</td>
      <td>31</td>
      <td>9</td>
      <td>0</td>
      <td>69.450000</td>
      <td>311.250000</td>
      <td>63.700000</td>
      <td>67.450000</td>
      <td>1.746000</td>
      <td>0.0</td>
      <td>55.500000</td>
      <td>True</td>
      <td>129500.512821</td>
      <td>38.589744</td>
      <td>0.230769</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
      <td>287451.754098</td>
      <td>15.409836</td>
      <td>0.327869</td>
      <td>0.081967</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2745877</td>
      <td>847</td>
      <td>15</td>
      <td>51</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>70.100000</td>
      <td>300.800000</td>
      <td>56.800000</td>
      <td>66.000000</td>
      <td>1.789000</td>
      <td>0.0</td>
      <td>49.400000</td>
      <td>True</td>
      <td>183058.466667</td>
      <td>56.466667</td>
      <td>0.066667</td>
      <td>0.066667</td>
    </tr>
  </tbody>
</table>
</div>




```python
#top points per event
top_points_per_event = second_cut.nlargest(6,'Points_per_event')
top_points_per_event
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Rickie Fowler</th>
      <td>3003047</td>
      <td>676</td>
      <td>11</td>
      <td>38</td>
      <td>10</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>305.000000</td>
      <td>58.700000</td>
      <td>69.300000</td>
      <td>1.709000</td>
      <td>0.0</td>
      <td>43.800000</td>
      <td>True</td>
      <td>273004.272727</td>
      <td>61.454545</td>
      <td>0.272727</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2745877</td>
      <td>847</td>
      <td>15</td>
      <td>51</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>70.100000</td>
      <td>300.800000</td>
      <td>56.800000</td>
      <td>66.000000</td>
      <td>1.789000</td>
      <td>0.0</td>
      <td>49.400000</td>
      <td>True</td>
      <td>183058.466667</td>
      <td>56.466667</td>
      <td>0.066667</td>
      <td>0.066667</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2299369</td>
      <td>746</td>
      <td>15</td>
      <td>52</td>
      <td>13</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>306.000000</td>
      <td>56.500000</td>
      <td>63.600000</td>
      <td>1.647000</td>
      <td>0.0</td>
      <td>53.300000</td>
      <td>True</td>
      <td>153291.266667</td>
      <td>49.733333</td>
      <td>0.266667</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2259018</td>
      <td>654</td>
      <td>16</td>
      <td>53</td>
      <td>12</td>
      <td>3</td>
      <td>0</td>
      <td>69.400000</td>
      <td>282.100000</td>
      <td>64.000000</td>
      <td>68.600000</td>
      <td>1.754000</td>
      <td>0.0</td>
      <td>63.400000</td>
      <td>True</td>
      <td>141188.625000</td>
      <td>40.875000</td>
      <td>0.187500</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1629364</td>
      <td>529</td>
      <td>13</td>
      <td>40</td>
      <td>7</td>
      <td>2</td>
      <td>0</td>
      <td>69.800000</td>
      <td>303.900000</td>
      <td>65.300000</td>
      <td>70.800000</td>
      <td>1.761000</td>
      <td>0.0</td>
      <td>43.300000</td>
      <td>True</td>
      <td>125335.692308</td>
      <td>40.692308</td>
      <td>0.153846</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#top top10's
top_top10 = second_cut.nlargest(6, 'Top10')
top_top10
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
      <td>400765.576923</td>
      <td>13.884615</td>
      <td>0.442308</td>
      <td>0.115385</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
      <td>287451.754098</td>
      <td>15.409836</td>
      <td>0.327869</td>
      <td>0.081967</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>52</td>
      <td>192</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>307.900000</td>
      <td>55.500000</td>
      <td>67.066667</td>
      <td>1.726000</td>
      <td>0.0</td>
      <td>60.266667</td>
      <td>True</td>
      <td>294733.884615</td>
      <td>6.634615</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
      <td>291863.923077</td>
      <td>9.807692</td>
      <td>0.365385</td>
      <td>0.057692</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14114548</td>
      <td>485</td>
      <td>57</td>
      <td>194</td>
      <td>46</td>
      <td>18</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.533333</td>
      <td>54.366667</td>
      <td>66.333333</td>
      <td>1.730667</td>
      <td>0.0</td>
      <td>56.100000</td>
      <td>True</td>
      <td>247623.649123</td>
      <td>8.508772</td>
      <td>0.315789</td>
      <td>0.035088</td>
    </tr>
  </tbody>
</table>
</div>




```python
#top10 per event
top_top10_per_event = second_cut.nlargest(6, 'Top10_per_event')
top_top10_per_event
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
      <td>400765.576923</td>
      <td>13.884615</td>
      <td>0.442308</td>
      <td>0.115385</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>52</td>
      <td>192</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>307.900000</td>
      <td>55.500000</td>
      <td>67.066667</td>
      <td>1.726000</td>
      <td>0.0</td>
      <td>60.266667</td>
      <td>True</td>
      <td>294733.884615</td>
      <td>6.634615</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
      <td>291863.923077</td>
      <td>9.807692</td>
      <td>0.365385</td>
      <td>0.057692</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
      <td>287451.754098</td>
      <td>15.409836</td>
      <td>0.327869</td>
      <td>0.081967</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14114548</td>
      <td>485</td>
      <td>57</td>
      <td>194</td>
      <td>46</td>
      <td>18</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.533333</td>
      <td>54.366667</td>
      <td>66.333333</td>
      <td>1.730667</td>
      <td>0.0</td>
      <td>56.100000</td>
      <td>True</td>
      <td>247623.649123</td>
      <td>8.508772</td>
      <td>0.315789</td>
      <td>0.035088</td>
    </tr>
  </tbody>
</table>
</div>




```python
#wins per event
top_wins_per_event = second_cut.nlargest(6, 'Wins_per_event')
top_wins_per_event
```




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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Patrick Cantlay</th>
      <td>20839810</td>
      <td>722</td>
      <td>52</td>
      <td>172</td>
      <td>44</td>
      <td>23</td>
      <td>6</td>
      <td>69.000000</td>
      <td>306.333333</td>
      <td>61.733333</td>
      <td>70.433333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>53.000000</td>
      <td>True</td>
      <td>400765.576923</td>
      <td>13.884615</td>
      <td>0.442308</td>
      <td>0.115385</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17534557</td>
      <td>940</td>
      <td>61</td>
      <td>197</td>
      <td>44</td>
      <td>20</td>
      <td>5</td>
      <td>69.666667</td>
      <td>307.866667</td>
      <td>57.200000</td>
      <td>66.666667</td>
      <td>1.716667</td>
      <td>0.0</td>
      <td>50.866667</td>
      <td>True</td>
      <td>287451.754098</td>
      <td>15.409836</td>
      <td>0.327869</td>
      <td>0.081967</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2745877</td>
      <td>847</td>
      <td>15</td>
      <td>51</td>
      <td>10</td>
      <td>1</td>
      <td>1</td>
      <td>70.100000</td>
      <td>300.800000</td>
      <td>56.800000</td>
      <td>66.000000</td>
      <td>1.789000</td>
      <td>0.0</td>
      <td>49.400000</td>
      <td>True</td>
      <td>183058.466667</td>
      <td>56.466667</td>
      <td>0.066667</td>
      <td>0.066667</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15176924</td>
      <td>510</td>
      <td>52</td>
      <td>177</td>
      <td>46</td>
      <td>19</td>
      <td>3</td>
      <td>69.333333</td>
      <td>305.366667</td>
      <td>58.433333</td>
      <td>69.033333</td>
      <td>1.722667</td>
      <td>0.0</td>
      <td>62.233333</td>
      <td>True</td>
      <td>291863.923077</td>
      <td>9.807692</td>
      <td>0.365385</td>
      <td>0.057692</td>
    </tr>
    <tr>
      <th>Harris English</th>
      <td>8996896</td>
      <td>516</td>
      <td>42</td>
      <td>141</td>
      <td>32</td>
      <td>10</td>
      <td>2</td>
      <td>69.900000</td>
      <td>297.000000</td>
      <td>61.750000</td>
      <td>64.950000</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>48.350000</td>
      <td>True</td>
      <td>214211.809524</td>
      <td>12.285714</td>
      <td>0.238095</td>
      <td>0.047619</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Great! We have our second_cut dataframe which features golfers who qualify to fill the last 6 spots
# We can see there are some consistent names at the top of results categories

```


```python
#One feature I'm noticing isn't here is majors. Let's add that manually
second_cut.index.name = 'displayName'
names = second_cut.index
print(names)
```

    Index(['Adam Schenk', 'Andrew Putnam', 'Brendon Todd', 'Brian Harman',
           'Collin Morikawa', 'Denny McCarthy', 'Harris English', 'Hayden Buckley',
           'Jordan Spieth', 'Justin Suh', 'Justin Thomas', 'Keith Mitchell',
           'Patrick Cantlay', 'Rickie Fowler', 'Sahith Theegala', 'Sam Burns',
           'Taylor Montgomery', 'Taylor Moore', 'Tom Hoge', 'Wyndham Clark',
           'Xander Schauffele'],
          dtype='object', name='displayName')



```python
majors = {'Adam Schenk': 0, 'Andrew Putnam': 0, 'Brendon Todd': 0, 'Brian Harman': 0,
          'Collin Morikawa': 2, 'Denny McCarthy': 0, 'Harris English': 0, 'Hayden Buckley': 0,
          'Jordan Spieth': 3, 'Justin Suh': 0, 'Justin Thomas': 2, 'Keith Mitchell': 0,
          'Patrick Cantlay': 0, 'Rickie Fowler': 0, 'Sahith Theegala': 0, 'Sam Burns': 0,
          'Taylor Montgomery': 0, 'Taylor Moore': 0, 'Tom Hoge': 0, 'Wyndham Clark': 0,
          'Xander Schauffele': 0}
second_cut['majors'] = second_cut.index.map(majors)
#sourced off wikipedia
second_cut.head()
```

    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:7: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      import sys





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
      <th>Earnings</th>
      <th>Cup</th>
      <th>Evnts</th>
      <th>Rnds</th>
      <th>Cuts</th>
      <th>Top10</th>
      <th>Wins</th>
      <th>Score</th>
      <th>DDIS</th>
      <th>DACC</th>
      <th>GIR</th>
      <th>PUTTS</th>
      <th>SAND</th>
      <th>BIRDS</th>
      <th>is_qualified</th>
      <th>Earnings_per_event</th>
      <th>Points_per_event</th>
      <th>Top10_per_event</th>
      <th>Wins_per_event</th>
      <th>majors</th>
    </tr>
    <tr>
      <th>displayName</th>
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
      <th>Adam Schenk</th>
      <td>1803965</td>
      <td>584</td>
      <td>17</td>
      <td>59</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.300000</td>
      <td>304.400000</td>
      <td>53.700000</td>
      <td>66.100000</td>
      <td>1.806000</td>
      <td>0.0</td>
      <td>49.000000</td>
      <td>True</td>
      <td>106115.588235</td>
      <td>34.352941</td>
      <td>0.058824</td>
      <td>0.000000</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2259018</td>
      <td>654</td>
      <td>16</td>
      <td>53</td>
      <td>12</td>
      <td>3</td>
      <td>0</td>
      <td>69.400000</td>
      <td>282.100000</td>
      <td>64.000000</td>
      <td>68.600000</td>
      <td>1.754000</td>
      <td>0.0</td>
      <td>63.400000</td>
      <td>True</td>
      <td>141188.625000</td>
      <td>40.875000</td>
      <td>0.187500</td>
      <td>0.000000</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1720861</td>
      <td>516</td>
      <td>13</td>
      <td>43</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.700000</td>
      <td>280.200000</td>
      <td>68.200000</td>
      <td>66.100000</td>
      <td>1.715000</td>
      <td>0.0</td>
      <td>60.300000</td>
      <td>True</td>
      <td>132373.923077</td>
      <td>39.692308</td>
      <td>0.230769</td>
      <td>0.000000</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>68</td>
      <td>232</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.700000</td>
      <td>292.433333</td>
      <td>67.133333</td>
      <td>66.966667</td>
      <td>1.753667</td>
      <td>0.0</td>
      <td>60.833333</td>
      <td>True</td>
      <td>129652.161765</td>
      <td>27.926471</td>
      <td>0.191176</td>
      <td>0.000000</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15533913</td>
      <td>2965</td>
      <td>52</td>
      <td>170</td>
      <td>42</td>
      <td>19</td>
      <td>2</td>
      <td>69.533333</td>
      <td>296.433333</td>
      <td>68.333333</td>
      <td>70.266667</td>
      <td>1.743333</td>
      <td>0.0</td>
      <td>51.733333</td>
      <td>True</td>
      <td>298729.096154</td>
      <td>57.019231</td>
      <td>0.365385</td>
      <td>0.038462</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python

# Let's tackle the analysis with visuals in two ways
# 1. Which golfers compete/win the most?
# 2. Which golfers have the best statistics?

```


```python
#let's export this in csv and take it to tableau

```


```python
second_cut.to_csv('/Users/joeyglenn4/Desktop/Case Study/second_cut.csv', index=True)
```


```python

```
