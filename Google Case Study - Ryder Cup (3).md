```python
#Web Scraping ESPN Public Data
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
#adding the value headers manually
table_headers = ['Earnings','Cup','Evnts','Rnds','Cuts','Top10','Wins','Score','DDIS','DACC','GIR','PUTTS','SAND','BIRDS']
print(table_headers)
```

    ['Earnings', 'Cup', 'Evnts', 'Rnds', 'Cuts', 'Top10', 'Wins', 'Score', 'DDIS', 'DACC', 'GIR', 'PUTTS', 'SAND', 'BIRDS']



```python
#pull stats for 2023
values_2023 =[]
for athlete in r_2023['athletes']:
       for category in athlete['categories']:
           x = category['totals'][0:14]
           values_2023.append(x)

#turn into dataframe
df_values_2023 = pd.DataFrame(values_2023, columns=table_headers)

```


```python
#player info is in a different section
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

#create a player info df
info_df_2023 = pd.DataFrame(info_2023, columns=info_categories)

```


```python
#combine both dataframes
df_2023 = pd.concat([info_df_2023,df_values_2023], axis=1)
```


```python
#repeat for seasons 2021 and 2022
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

#2021 athletes
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
#reformat on earnings, we'll have to remove commas and $ signs
golf_df_concat['Earnings'] = golf_df_concat['Earnings'].str.replace(',', '').str.replace('$', '')
```

    /Users/joeyglenn4/opt/anaconda3/lib/python3.7/site-packages/ipykernel_launcher.py:2: FutureWarning: The default value of regex will change from True to False in a future version. In addition, single character regular expressions will *not* be treated as literal strings when regex=True.
      



```python

```


```python
#reformat categories from strings to numbers
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
#Quick Save
golf_df_agg.to_excel('golf_player_data.xlsx', index=True)
```


```python
#Data check
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
      <td>18</td>
      <td>61</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.40</td>
      <td>304.50</td>
      <td>53.5</td>
      <td>66.00</td>
      <td>1.808</td>
      <td>0.0</td>
      <td>50.90</td>
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
      <td>13957192</td>
      <td>713</td>
      <td>56</td>
      <td>193</td>
      <td>50</td>
      <td>17</td>
      <td>2</td>
      <td>69.566667</td>
      <td>305.133333</td>
      <td>63.933333</td>
      <td>67.566667</td>
      <td>1.735000</td>
      <td>0.0</td>
      <td>51.10</td>
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
      <td>61.70</td>
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
      <td>48.55</td>
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
      <td>52.80</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15608924</td>
      <td>585</td>
      <td>53</td>
      <td>181</td>
      <td>47</td>
      <td>20</td>
      <td>3</td>
      <td>69.400000</td>
      <td>305.300000</td>
      <td>59.433333</td>
      <td>69.200000</td>
      <td>1.726667</td>
      <td>0.0</td>
      <td>61.10</td>
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
      <td>9.200000e+01</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.0</td>
      <td>92.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>6.776147e+06</td>
      <td>1015.673913</td>
      <td>33.836957</td>
      <td>112.554348</td>
      <td>26.097826</td>
      <td>8.152174</td>
      <td>1.152174</td>
      <td>69.535326</td>
      <td>300.308514</td>
      <td>60.072283</td>
      <td>66.812138</td>
      <td>1.734138</td>
      <td>0.0</td>
      <td>52.900362</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.671968e+06</td>
      <td>825.992618</td>
      <td>17.431358</td>
      <td>59.594853</td>
      <td>14.576106</td>
      <td>6.182099</td>
      <td>1.459601</td>
      <td>3.709235</td>
      <td>17.933058</td>
      <td>4.986574</td>
      <td>3.976978</td>
      <td>0.094267</td>
      <td>0.0</td>
      <td>6.790910</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.730193e+06</td>
      <td>-18.000000</td>
      <td>11.000000</td>
      <td>34.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>34.650000</td>
      <td>147.750000</td>
      <td>35.550000</td>
      <td>35.250000</td>
      <td>0.872500</td>
      <td>0.0</td>
      <td>27.100000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.903400e+06</td>
      <td>546.250000</td>
      <td>20.000000</td>
      <td>65.000000</td>
      <td>14.000000</td>
      <td>4.000000</td>
      <td>0.000000</td>
      <td>69.600000</td>
      <td>296.000000</td>
      <td>56.825000</td>
      <td>65.875000</td>
      <td>1.729917</td>
      <td>0.0</td>
      <td>48.900000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.533484e+06</td>
      <td>817.500000</td>
      <td>27.500000</td>
      <td>91.000000</td>
      <td>20.500000</td>
      <td>6.000000</td>
      <td>1.000000</td>
      <td>69.833333</td>
      <td>302.616667</td>
      <td>60.575000</td>
      <td>67.025000</td>
      <td>1.742000</td>
      <td>0.0</td>
      <td>52.400000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.521836e+06</td>
      <td>1311.000000</td>
      <td>47.250000</td>
      <td>154.500000</td>
      <td>37.250000</td>
      <td>10.000000</td>
      <td>1.000000</td>
      <td>70.137500</td>
      <td>307.075000</td>
      <td>63.037500</td>
      <td>68.850000</td>
      <td>1.757250</td>
      <td>0.0</td>
      <td>56.950000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.061599e+07</td>
      <td>3680.000000</td>
      <td>75.000000</td>
      <td>260.000000</td>
      <td>64.000000</td>
      <td>30.000000</td>
      <td>6.000000</td>
      <td>71.900000</td>
      <td>323.700000</td>
      <td>71.100000</td>
      <td>72.066667</td>
      <td>1.808000</td>
      <td>0.0</td>
      <td>78.300000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Minimum average score is 34.55, lowest score in PGA was shot by Furyk (58)
#find that entry
unreal_golfer = golf_df_agg[golf_df_agg['Score'] == 34.55]
print(unreal_golfer)
```

    Empty DataFrame
    Columns: [Earnings, Cup, Evnts, Rnds, Cuts, Top10, Wins, Score, DDIS, DACC, GIR, PUTTS, SAND, BIRDS]
    Index: []



```python
#Tom Kim is good... but he's not that good
unreal_golfer_2023 = df_2023[df_2023['displayName'] == 'Tom Kim']
print(unreal_golfer_2023)
```

             id               uid                                  guid firstName  \
    19  4602673  s:1100~a:4602673  cce7aed3-de73-3aef-886c-688ba5f9ea88       Tom   
    
       lastName displayName shortName  debutYear  \
    19      Kim     Tom Kim    T. Kim        NaN   
    
                                                    links  \
    19  [{'language': 'en-US', 'rel': ['playercard', '...   
    
                                                 headshot  \
    19  {'href': 'https://a.espncdn.com/i/headshots/go...   
    
                                                   status  age  \
    19  {'id': '1', 'name': 'Active', 'type': 'active'...   20   
    
                                                     flag    Earnings  Cup Evnts  \
    19  {'href': 'https://a.espncdn.com/i/teamlogos/co...  $3,237,766  917    12   
    
       Rnds Cuts Top10 Wins Score   DDIS  DACC   GIR  PUTTS SAND BIRDS  
    19   42   11     3    1  69.3  295.5  71.1  70.5  1.745    0  54.2  



```python
#ESPN incorrectly entered his stats as 0 in 2021-22
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
#Clean record, locking in 2023 stats. You could argue that this is inconsistent, but Tom Kim won't qualify for the final dataset anyways.
#The dataset will be filtered for American candidates for the Ryder Cup.
golf_df_agg.loc['Tom Kim']
```




    Earnings    6.062346e+06
    Cup         2.071000e+03
    Evnts       2.300000e+01
    Rnds        4.200000e+01
    Cuts        2.100000e+01
    Top10       6.000000e+00
    Wins        2.000000e+00
    Score       3.465000e+01
    DDIS        1.477500e+02
    DACC        3.555000e+01
    GIR         3.525000e+01
    PUTTS       8.725000e-01
    SAND        0.000000e+00
    BIRDS       2.710000e+01
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
#Check Tom's updated stats
golf_df_agg.loc['Tom Kim']
```




    Earnings    6062346.000
    Cup            2071.000
    Evnts            23.000
    Rnds             42.000
    Cuts             21.000
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
#check for any outliers
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
      <td>9.200000e+01</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.000000</td>
      <td>92.0</td>
      <td>92.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>6.776147e+06</td>
      <td>1015.673913</td>
      <td>33.836957</td>
      <td>112.554348</td>
      <td>26.097826</td>
      <td>8.152174</td>
      <td>1.152174</td>
      <td>69.909783</td>
      <td>301.910145</td>
      <td>60.443478</td>
      <td>67.195290</td>
      <td>1.743534</td>
      <td>0.0</td>
      <td>53.212319</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.671968e+06</td>
      <td>825.992618</td>
      <td>17.431358</td>
      <td>59.594853</td>
      <td>14.576106</td>
      <td>6.182099</td>
      <td>1.459601</td>
      <td>0.495253</td>
      <td>7.971194</td>
      <td>4.374593</td>
      <td>2.206917</td>
      <td>0.025273</td>
      <td>0.0</td>
      <td>6.228607</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.730193e+06</td>
      <td>-18.000000</td>
      <td>11.000000</td>
      <td>34.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>68.933333</td>
      <td>280.700000</td>
      <td>50.800000</td>
      <td>57.800000</td>
      <td>1.646000</td>
      <td>0.0</td>
      <td>42.600000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.903400e+06</td>
      <td>546.250000</td>
      <td>20.000000</td>
      <td>65.000000</td>
      <td>14.000000</td>
      <td>4.000000</td>
      <td>0.000000</td>
      <td>69.600000</td>
      <td>296.000000</td>
      <td>57.050000</td>
      <td>65.975000</td>
      <td>1.730250</td>
      <td>0.0</td>
      <td>49.075000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.533484e+06</td>
      <td>817.500000</td>
      <td>27.500000</td>
      <td>91.000000</td>
      <td>20.500000</td>
      <td>6.000000</td>
      <td>1.000000</td>
      <td>69.833333</td>
      <td>302.616667</td>
      <td>60.600000</td>
      <td>67.075000</td>
      <td>1.742000</td>
      <td>0.0</td>
      <td>52.600000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.521836e+06</td>
      <td>1311.000000</td>
      <td>47.250000</td>
      <td>154.500000</td>
      <td>37.250000</td>
      <td>10.000000</td>
      <td>1.000000</td>
      <td>70.137500</td>
      <td>307.075000</td>
      <td>63.162500</td>
      <td>69.016667</td>
      <td>1.757250</td>
      <td>0.0</td>
      <td>56.950000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.061599e+07</td>
      <td>3680.000000</td>
      <td>75.000000</td>
      <td>260.000000</td>
      <td>64.000000</td>
      <td>30.000000</td>
      <td>6.000000</td>
      <td>71.900000</td>
      <td>323.700000</td>
      <td>71.100000</td>
      <td>72.066667</td>
      <td>1.808000</td>
      <td>0.0</td>
      <td>78.300000</td>
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
      <td>18</td>
      <td>61</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.400000</td>
      <td>304.500000</td>
      <td>53.500000</td>
      <td>66.000000</td>
      <td>1.808000</td>
      <td>0.0</td>
      <td>50.900000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2314939</td>
      <td>679</td>
      <td>17</td>
      <td>57</td>
      <td>13</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>283.100000</td>
      <td>63.000000</td>
      <td>68.700000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>61.700000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1742013</td>
      <td>522</td>
      <td>14</td>
      <td>47</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>69.900000</td>
      <td>280.700000</td>
      <td>66.700000</td>
      <td>65.500000</td>
      <td>1.717000</td>
      <td>0.0</td>
      <td>59.700000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>1899</td>
      <td>69</td>
      <td>234</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.800000</td>
      <td>292.466667</td>
      <td>67.166667</td>
      <td>66.633333</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>59.966667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Chris Kirk</th>
      <td>3177760</td>
      <td>1024</td>
      <td>15</td>
      <td>48</td>
      <td>11</td>
      <td>4</td>
      <td>1</td>
      <td>69.800000</td>
      <td>295.800000</td>
      <td>61.600000</td>
      <td>66.600000</td>
      <td>1.697000</td>
      <td>0.0</td>
      <td>63.900000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15965913</td>
      <td>3040</td>
      <td>53</td>
      <td>174</td>
      <td>43</td>
      <td>20</td>
      <td>2</td>
      <td>69.600000</td>
      <td>296.533333</td>
      <td>68.700000</td>
      <td>69.966667</td>
      <td>1.743667</td>
      <td>0.0</td>
      <td>52.300000</td>
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
      <td>9063496</td>
      <td>530</td>
      <td>43</td>
      <td>145</td>
      <td>33</td>
      <td>10</td>
      <td>2</td>
      <td>70.000000</td>
      <td>296.850000</td>
      <td>62.500000</td>
      <td>64.750000</td>
      <td>1.742000</td>
      <td>0.0</td>
      <td>49.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1836289</td>
      <td>594</td>
      <td>14</td>
      <td>44</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.800000</td>
      <td>304.500000</td>
      <td>64.400000</td>
      <td>69.400000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>45.600000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14858548</td>
      <td>612</td>
      <td>58</td>
      <td>198</td>
      <td>47</td>
      <td>19</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.300000</td>
      <td>55.133333</td>
      <td>66.133333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>56.366667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>345</td>
      <td>53</td>
      <td>194</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.600000</td>
      <td>307.633333</td>
      <td>55.966667</td>
      <td>66.933333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>60.766667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keegan Bradley</th>
      <td>10811950</td>
      <td>3305</td>
      <td>64</td>
      <td>204</td>
      <td>45</td>
      <td>14</td>
      <td>1</td>
      <td>70.133333</td>
      <td>301.766667</td>
      <td>62.100000</td>
      <td>67.533333</td>
      <td>1.756667</td>
      <td>0.0</td>
      <td>46.166667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5093720</td>
      <td>1513</td>
      <td>40</td>
      <td>134</td>
      <td>32</td>
      <td>9</td>
      <td>0</td>
      <td>69.700000</td>
      <td>310.850000</td>
      <td>63.750000</td>
      <td>67.250000</td>
      <td>1.750000</td>
      <td>0.0</td>
      <td>54.700000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Kurt Kitayama</th>
      <td>5693388</td>
      <td>1040</td>
      <td>12</td>
      <td>34</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>70.400000</td>
      <td>306.400000</td>
      <td>55.000000</td>
      <td>66.500000</td>
      <td>1.764000</td>
      <td>0.0</td>
      <td>57.100000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Max Homa</th>
      <td>16514432</td>
      <td>3053</td>
      <td>63</td>
      <td>208</td>
      <td>50</td>
      <td>16</td>
      <td>5</td>
      <td>69.933333</td>
      <td>302.733333</td>
      <td>60.600000</td>
      <td>65.800000</td>
      <td>1.724333</td>
      <td>0.0</td>
      <td>51.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Patrick Cantlay</th>
      <td>21172810</td>
      <td>784</td>
      <td>53</td>
      <td>176</td>
      <td>45</td>
      <td>23</td>
      <td>6</td>
      <td>69.100000</td>
      <td>306.366667</td>
      <td>62.433333</td>
      <td>70.566667</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>53.533333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Rickie Fowler</th>
      <td>3209972</td>
      <td>741</td>
      <td>12</td>
      <td>42</td>
      <td>11</td>
      <td>4</td>
      <td>0</td>
      <td>69.600000</td>
      <td>305.500000</td>
      <td>57.700000</td>
      <td>69.400000</td>
      <td>1.707000</td>
      <td>0.0</td>
      <td>44.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sahith Theegala</th>
      <td>6899820</td>
      <td>870</td>
      <td>47</td>
      <td>167</td>
      <td>40</td>
      <td>11</td>
      <td>0</td>
      <td>69.950000</td>
      <td>302.400000</td>
      <td>54.550000</td>
      <td>67.050000</td>
      <td>1.733000</td>
      <td>0.0</td>
      <td>51.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17659657</td>
      <td>971</td>
      <td>62</td>
      <td>201</td>
      <td>45</td>
      <td>20</td>
      <td>5</td>
      <td>69.766667</td>
      <td>308.033333</td>
      <td>58.100000</td>
      <td>66.866667</td>
      <td>1.720667</td>
      <td>0.0</td>
      <td>50.100000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Scottie Scheffler</th>
      <td>30615994</td>
      <td>1821</td>
      <td>65</td>
      <td>222</td>
      <td>55</td>
      <td>27</td>
      <td>6</td>
      <td>69.033333</td>
      <td>308.433333</td>
      <td>62.600000</td>
      <td>71.466667</td>
      <td>1.724333</td>
      <td>0.0</td>
      <td>44.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2382435</td>
      <td>783</td>
      <td>16</td>
      <td>56</td>
      <td>14</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>304.900000</td>
      <td>55.000000</td>
      <td>62.800000</td>
      <td>1.646000</td>
      <td>0.0</td>
      <td>52.500000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2825077</td>
      <td>865</td>
      <td>16</td>
      <td>55</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>70.400000</td>
      <td>301.200000</td>
      <td>57.400000</td>
      <td>65.600000</td>
      <td>1.790000</td>
      <td>0.0</td>
      <td>48.200000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tom Hoge</th>
      <td>8109202</td>
      <td>741</td>
      <td>48</td>
      <td>146</td>
      <td>30</td>
      <td>10</td>
      <td>1</td>
      <td>69.550000</td>
      <td>296.150000</td>
      <td>63.150000</td>
      <td>68.250000</td>
      <td>1.729000</td>
      <td>0.0</td>
      <td>52.050000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tony Finau</th>
      <td>15456084</td>
      <td>953</td>
      <td>63</td>
      <td>214</td>
      <td>52</td>
      <td>18</td>
      <td>4</td>
      <td>69.433333</td>
      <td>304.966667</td>
      <td>58.400000</td>
      <td>69.533333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>55.066667</td>
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
      <td>15608924</td>
      <td>585</td>
      <td>53</td>
      <td>181</td>
      <td>47</td>
      <td>20</td>
      <td>3</td>
      <td>69.400000</td>
      <td>305.300000</td>
      <td>59.433333</td>
      <td>69.200000</td>
      <td>1.726667</td>
      <td>0.0</td>
      <td>61.100000</td>
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
      <td>2.600000e+01</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.000000</td>
      <td>26.0</td>
      <td>26.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9.369234e+06</td>
      <td>1153.076923</td>
      <td>39.538462</td>
      <td>132.769231</td>
      <td>31.153846</td>
      <td>11.000000</td>
      <td>1.615385</td>
      <td>69.744872</td>
      <td>301.410897</td>
      <td>60.182051</td>
      <td>67.393590</td>
      <td>1.736301</td>
      <td>0.0</td>
      <td>53.456410</td>
    </tr>
    <tr>
      <th>std</th>
      <td>7.495006e+06</td>
      <td>831.900808</td>
      <td>21.033746</td>
      <td>70.764006</td>
      <td>17.469270</td>
      <td>7.828154</td>
      <td>2.001538</td>
      <td>0.353986</td>
      <td>7.747845</td>
      <td>4.333838</td>
      <td>1.977024</td>
      <td>0.029878</td>
      <td>0.0</td>
      <td>5.528432</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.742013e+06</td>
      <td>345.000000</td>
      <td>12.000000</td>
      <td>34.000000</td>
      <td>7.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>69.033333</td>
      <td>280.700000</td>
      <td>53.500000</td>
      <td>62.800000</td>
      <td>1.646000</td>
      <td>0.0</td>
      <td>44.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.913248e+06</td>
      <td>598.500000</td>
      <td>16.000000</td>
      <td>56.000000</td>
      <td>12.250000</td>
      <td>4.000000</td>
      <td>0.000000</td>
      <td>69.562500</td>
      <td>296.612500</td>
      <td>56.325000</td>
      <td>66.033333</td>
      <td>1.724917</td>
      <td>0.0</td>
      <td>50.300000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.504511e+06</td>
      <td>824.500000</td>
      <td>46.000000</td>
      <td>150.000000</td>
      <td>34.000000</td>
      <td>10.000000</td>
      <td>1.000000</td>
      <td>69.733333</td>
      <td>303.900000</td>
      <td>61.100000</td>
      <td>66.991667</td>
      <td>1.733500</td>
      <td>0.0</td>
      <td>52.650000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.542360e+07</td>
      <td>1394.750000</td>
      <td>56.750000</td>
      <td>197.000000</td>
      <td>46.500000</td>
      <td>18.750000</td>
      <td>2.000000</td>
      <td>69.925000</td>
      <td>306.150000</td>
      <td>62.900000</td>
      <td>69.075000</td>
      <td>1.753750</td>
      <td>0.0</td>
      <td>56.916667</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.061599e+07</td>
      <td>3305.000000</td>
      <td>69.000000</td>
      <td>234.000000</td>
      <td>55.000000</td>
      <td>27.000000</td>
      <td>6.000000</td>
      <td>70.400000</td>
      <td>313.000000</td>
      <td>68.700000</td>
      <td>71.466667</td>
      <td>1.808000</td>
      <td>0.0</td>
      <td>63.900000</td>
    </tr>
  </tbody>
</table>
</div>




```python
new_df = df_2023.set_index('displayName')
first_cut.update(new_df[['Cup']])
first_cut['Cup'] = first_cut['Cup'].astype(int)

```


```python
# 6 golfers automatically qualify for the American team on points, 
#if points were taken today, this is who would auto qualify
first_six = first_cut.nlargest(6,'Cup')
index_list = first_six.index.tolist()
first_six
#print(index_list)

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
      <th>Scottie Scheffler</th>
      <td>30615994</td>
      <td>1844</td>
      <td>65</td>
      <td>222</td>
      <td>55</td>
      <td>27</td>
      <td>6</td>
      <td>69.033333</td>
      <td>308.433333</td>
      <td>62.6</td>
      <td>71.466667</td>
      <td>1.724333</td>
      <td>0.0</td>
      <td>44.400000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Max Homa</th>
      <td>16514432</td>
      <td>1801</td>
      <td>63</td>
      <td>208</td>
      <td>50</td>
      <td>16</td>
      <td>5</td>
      <td>69.933333</td>
      <td>302.733333</td>
      <td>60.6</td>
      <td>65.800000</td>
      <td>1.724333</td>
      <td>0.0</td>
      <td>51.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keegan Bradley</th>
      <td>10811950</td>
      <td>1153</td>
      <td>64</td>
      <td>204</td>
      <td>45</td>
      <td>14</td>
      <td>1</td>
      <td>70.133333</td>
      <td>301.766667</td>
      <td>62.1</td>
      <td>67.533333</td>
      <td>1.756667</td>
      <td>0.0</td>
      <td>46.166667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Kurt Kitayama</th>
      <td>5693388</td>
      <td>1040</td>
      <td>12</td>
      <td>34</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>70.400000</td>
      <td>306.400000</td>
      <td>55.0</td>
      <td>66.500000</td>
      <td>1.764000</td>
      <td>0.0</td>
      <td>57.100000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Chris Kirk</th>
      <td>3177760</td>
      <td>1024</td>
      <td>15</td>
      <td>48</td>
      <td>11</td>
      <td>4</td>
      <td>1</td>
      <td>69.800000</td>
      <td>295.800000</td>
      <td>61.6</td>
      <td>66.600000</td>
      <td>1.697000</td>
      <td>0.0</td>
      <td>63.900000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sam Burns</th>
      <td>17659657</td>
      <td>984</td>
      <td>62</td>
      <td>201</td>
      <td>45</td>
      <td>20</td>
      <td>5</td>
      <td>69.766667</td>
      <td>308.033333</td>
      <td>58.1</td>
      <td>66.866667</td>
      <td>1.720667</td>
      <td>0.0</td>
      <td>50.100000</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
# remove the top 6 from the choices, they're on the team
USA_team = []
USA_team.extend(index_list)
# select rows where displayName is not in USA_team
second_cut = first_cut.drop(index=USA_team)
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
      <td>18</td>
      <td>61</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.400000</td>
      <td>304.500000</td>
      <td>53.500000</td>
      <td>66.000000</td>
      <td>1.808000</td>
      <td>0.0</td>
      <td>50.900000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2314939</td>
      <td>679</td>
      <td>17</td>
      <td>57</td>
      <td>13</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>283.100000</td>
      <td>63.000000</td>
      <td>68.700000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>61.700000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1742013</td>
      <td>522</td>
      <td>14</td>
      <td>47</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>69.900000</td>
      <td>280.700000</td>
      <td>66.700000</td>
      <td>65.500000</td>
      <td>1.717000</td>
      <td>0.0</td>
      <td>59.700000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>739</td>
      <td>69</td>
      <td>234</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.800000</td>
      <td>292.466667</td>
      <td>67.166667</td>
      <td>66.633333</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>59.966667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15965913</td>
      <td>860</td>
      <td>53</td>
      <td>174</td>
      <td>43</td>
      <td>20</td>
      <td>2</td>
      <td>69.600000</td>
      <td>296.533333</td>
      <td>68.700000</td>
      <td>69.966667</td>
      <td>1.743667</td>
      <td>0.0</td>
      <td>52.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>480</td>
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
      <td>9063496</td>
      <td>536</td>
      <td>43</td>
      <td>145</td>
      <td>33</td>
      <td>10</td>
      <td>2</td>
      <td>70.000000</td>
      <td>296.850000</td>
      <td>62.500000</td>
      <td>64.750000</td>
      <td>1.742000</td>
      <td>0.0</td>
      <td>49.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1836289</td>
      <td>594</td>
      <td>14</td>
      <td>44</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.800000</td>
      <td>304.500000</td>
      <td>64.400000</td>
      <td>69.400000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>45.600000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14858548</td>
      <td>628</td>
      <td>58</td>
      <td>198</td>
      <td>47</td>
      <td>19</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.300000</td>
      <td>55.133333</td>
      <td>66.133333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>56.366667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>377</td>
      <td>53</td>
      <td>194</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.600000</td>
      <td>307.633333</td>
      <td>55.966667</td>
      <td>66.933333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>60.766667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5093720</td>
      <td>450</td>
      <td>40</td>
      <td>134</td>
      <td>32</td>
      <td>9</td>
      <td>0</td>
      <td>69.700000</td>
      <td>310.850000</td>
      <td>63.750000</td>
      <td>67.250000</td>
      <td>1.750000</td>
      <td>0.0</td>
      <td>54.700000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Patrick Cantlay</th>
      <td>21172810</td>
      <td>821</td>
      <td>53</td>
      <td>176</td>
      <td>45</td>
      <td>23</td>
      <td>6</td>
      <td>69.100000</td>
      <td>306.366667</td>
      <td>62.433333</td>
      <td>70.566667</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>53.533333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Rickie Fowler</th>
      <td>3209972</td>
      <td>741</td>
      <td>12</td>
      <td>42</td>
      <td>11</td>
      <td>4</td>
      <td>0</td>
      <td>69.600000</td>
      <td>305.500000</td>
      <td>57.700000</td>
      <td>69.400000</td>
      <td>1.707000</td>
      <td>0.0</td>
      <td>44.000000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Sahith Theegala</th>
      <td>6899820</td>
      <td>869</td>
      <td>47</td>
      <td>167</td>
      <td>40</td>
      <td>11</td>
      <td>0</td>
      <td>69.950000</td>
      <td>302.400000</td>
      <td>54.550000</td>
      <td>67.050000</td>
      <td>1.733000</td>
      <td>0.0</td>
      <td>51.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2382435</td>
      <td>783</td>
      <td>16</td>
      <td>56</td>
      <td>14</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>304.900000</td>
      <td>55.000000</td>
      <td>62.800000</td>
      <td>1.646000</td>
      <td>0.0</td>
      <td>52.500000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2825077</td>
      <td>865</td>
      <td>16</td>
      <td>55</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>70.400000</td>
      <td>301.200000</td>
      <td>57.400000</td>
      <td>65.600000</td>
      <td>1.790000</td>
      <td>0.0</td>
      <td>48.200000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tom Hoge</th>
      <td>8109202</td>
      <td>755</td>
      <td>48</td>
      <td>146</td>
      <td>30</td>
      <td>10</td>
      <td>1</td>
      <td>69.550000</td>
      <td>296.150000</td>
      <td>63.150000</td>
      <td>68.250000</td>
      <td>1.729000</td>
      <td>0.0</td>
      <td>52.050000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tony Finau</th>
      <td>15456084</td>
      <td>976</td>
      <td>63</td>
      <td>214</td>
      <td>52</td>
      <td>18</td>
      <td>4</td>
      <td>69.433333</td>
      <td>304.966667</td>
      <td>58.400000</td>
      <td>69.533333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>55.066667</td>
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
      <td>15608924</td>
      <td>617</td>
      <td>53</td>
      <td>181</td>
      <td>47</td>
      <td>20</td>
      <td>3</td>
      <td>69.400000</td>
      <td>305.300000</td>
      <td>59.433333</td>
      <td>69.200000</td>
      <td>1.726667</td>
      <td>0.0</td>
      <td>61.100000</td>
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
      <td>21172810</td>
      <td>821</td>
      <td>53</td>
      <td>176</td>
      <td>45</td>
      <td>23</td>
      <td>6</td>
      <td>69.100000</td>
      <td>306.366667</td>
      <td>62.433333</td>
      <td>70.566667</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>53.533333</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15965913</td>
      <td>860</td>
      <td>53</td>
      <td>174</td>
      <td>43</td>
      <td>20</td>
      <td>2</td>
      <td>69.600000</td>
      <td>296.533333</td>
      <td>68.700000</td>
      <td>69.966667</td>
      <td>1.743667</td>
      <td>0.0</td>
      <td>52.300000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Xander Schauffele</th>
      <td>15608924</td>
      <td>617</td>
      <td>53</td>
      <td>181</td>
      <td>47</td>
      <td>20</td>
      <td>3</td>
      <td>69.400000</td>
      <td>305.300000</td>
      <td>59.433333</td>
      <td>69.200000</td>
      <td>1.726667</td>
      <td>0.0</td>
      <td>61.100000</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Tony Finau</th>
      <td>15456084</td>
      <td>976</td>
      <td>63</td>
      <td>214</td>
      <td>52</td>
      <td>18</td>
      <td>4</td>
      <td>69.433333</td>
      <td>304.966667</td>
      <td>58.400000</td>
      <td>69.533333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>55.066667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>377</td>
      <td>53</td>
      <td>194</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.600000</td>
      <td>307.633333</td>
      <td>55.966667</td>
      <td>66.933333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>60.766667</td>
      <td>True</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14858548</td>
      <td>628</td>
      <td>58</td>
      <td>198</td>
      <td>47</td>
      <td>19</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.300000</td>
      <td>55.133333</td>
      <td>66.133333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>56.366667</td>
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
      <td>18</td>
      <td>61</td>
      <td>12</td>
      <td>1</td>
      <td>0</td>
      <td>70.400000</td>
      <td>304.500000</td>
      <td>53.500000</td>
      <td>66.000000</td>
      <td>1.808000</td>
      <td>0.0</td>
      <td>50.900000</td>
      <td>True</td>
      <td>100220.277778</td>
      <td>32.444444</td>
      <td>0.055556</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Andrew Putnam</th>
      <td>2314939</td>
      <td>679</td>
      <td>17</td>
      <td>57</td>
      <td>13</td>
      <td>3</td>
      <td>0</td>
      <td>69.500000</td>
      <td>283.100000</td>
      <td>63.000000</td>
      <td>68.700000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>61.700000</td>
      <td>True</td>
      <td>136172.882353</td>
      <td>39.941176</td>
      <td>0.176471</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Brendon Todd</th>
      <td>1742013</td>
      <td>522</td>
      <td>14</td>
      <td>47</td>
      <td>9</td>
      <td>3</td>
      <td>0</td>
      <td>69.900000</td>
      <td>280.700000</td>
      <td>66.700000</td>
      <td>65.500000</td>
      <td>1.717000</td>
      <td>0.0</td>
      <td>59.700000</td>
      <td>True</td>
      <td>124429.500000</td>
      <td>37.285714</td>
      <td>0.214286</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Brian Harman</th>
      <td>8816347</td>
      <td>739</td>
      <td>69</td>
      <td>234</td>
      <td>54</td>
      <td>13</td>
      <td>0</td>
      <td>69.800000</td>
      <td>292.466667</td>
      <td>67.166667</td>
      <td>66.633333</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>59.966667</td>
      <td>True</td>
      <td>127773.144928</td>
      <td>10.710145</td>
      <td>0.188406</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Collin Morikawa</th>
      <td>15965913</td>
      <td>860</td>
      <td>53</td>
      <td>174</td>
      <td>43</td>
      <td>20</td>
      <td>2</td>
      <td>69.600000</td>
      <td>296.533333</td>
      <td>68.700000</td>
      <td>69.966667</td>
      <td>1.743667</td>
      <td>0.0</td>
      <td>52.300000</td>
      <td>True</td>
      <td>301243.641509</td>
      <td>16.226415</td>
      <td>0.377358</td>
      <td>0.037736</td>
    </tr>
    <tr>
      <th>Denny McCarthy</th>
      <td>4693126</td>
      <td>480</td>
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
      <td>10.666667</td>
      <td>0.155556</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Harris English</th>
      <td>9063496</td>
      <td>536</td>
      <td>43</td>
      <td>145</td>
      <td>33</td>
      <td>10</td>
      <td>2</td>
      <td>70.000000</td>
      <td>296.850000</td>
      <td>62.500000</td>
      <td>64.750000</td>
      <td>1.742000</td>
      <td>0.0</td>
      <td>49.300000</td>
      <td>True</td>
      <td>210778.976744</td>
      <td>12.465116</td>
      <td>0.232558</td>
      <td>0.046512</td>
    </tr>
    <tr>
      <th>Hayden Buckley</th>
      <td>1836289</td>
      <td>594</td>
      <td>14</td>
      <td>44</td>
      <td>8</td>
      <td>3</td>
      <td>0</td>
      <td>69.800000</td>
      <td>304.500000</td>
      <td>64.400000</td>
      <td>69.400000</td>
      <td>1.755000</td>
      <td>0.0</td>
      <td>45.600000</td>
      <td>True</td>
      <td>131163.500000</td>
      <td>42.428571</td>
      <td>0.214286</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Jordan Spieth</th>
      <td>14858548</td>
      <td>628</td>
      <td>58</td>
      <td>198</td>
      <td>47</td>
      <td>19</td>
      <td>2</td>
      <td>69.866667</td>
      <td>303.300000</td>
      <td>55.133333</td>
      <td>66.133333</td>
      <td>1.728667</td>
      <td>0.0</td>
      <td>56.366667</td>
      <td>True</td>
      <td>256181.862069</td>
      <td>10.827586</td>
      <td>0.327586</td>
      <td>0.034483</td>
    </tr>
    <tr>
      <th>Justin Thomas</th>
      <td>15326162</td>
      <td>377</td>
      <td>53</td>
      <td>194</td>
      <td>48</td>
      <td>19</td>
      <td>2</td>
      <td>69.600000</td>
      <td>307.633333</td>
      <td>55.966667</td>
      <td>66.933333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>60.766667</td>
      <td>True</td>
      <td>289172.867925</td>
      <td>7.113208</td>
      <td>0.358491</td>
      <td>0.037736</td>
    </tr>
    <tr>
      <th>Keith Mitchell</th>
      <td>5093720</td>
      <td>450</td>
      <td>40</td>
      <td>134</td>
      <td>32</td>
      <td>9</td>
      <td>0</td>
      <td>69.700000</td>
      <td>310.850000</td>
      <td>63.750000</td>
      <td>67.250000</td>
      <td>1.750000</td>
      <td>0.0</td>
      <td>54.700000</td>
      <td>True</td>
      <td>127343.000000</td>
      <td>11.250000</td>
      <td>0.225000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Patrick Cantlay</th>
      <td>21172810</td>
      <td>821</td>
      <td>53</td>
      <td>176</td>
      <td>45</td>
      <td>23</td>
      <td>6</td>
      <td>69.100000</td>
      <td>306.366667</td>
      <td>62.433333</td>
      <td>70.566667</td>
      <td>1.737000</td>
      <td>0.0</td>
      <td>53.533333</td>
      <td>True</td>
      <td>399486.981132</td>
      <td>15.490566</td>
      <td>0.433962</td>
      <td>0.113208</td>
    </tr>
    <tr>
      <th>Rickie Fowler</th>
      <td>3209972</td>
      <td>741</td>
      <td>12</td>
      <td>42</td>
      <td>11</td>
      <td>4</td>
      <td>0</td>
      <td>69.600000</td>
      <td>305.500000</td>
      <td>57.700000</td>
      <td>69.400000</td>
      <td>1.707000</td>
      <td>0.0</td>
      <td>44.000000</td>
      <td>True</td>
      <td>267497.666667</td>
      <td>61.750000</td>
      <td>0.333333</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Sahith Theegala</th>
      <td>6899820</td>
      <td>869</td>
      <td>47</td>
      <td>167</td>
      <td>40</td>
      <td>11</td>
      <td>0</td>
      <td>69.950000</td>
      <td>302.400000</td>
      <td>54.550000</td>
      <td>67.050000</td>
      <td>1.733000</td>
      <td>0.0</td>
      <td>51.300000</td>
      <td>True</td>
      <td>146804.680851</td>
      <td>18.489362</td>
      <td>0.234043</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Taylor Montgomery</th>
      <td>2382435</td>
      <td>783</td>
      <td>16</td>
      <td>56</td>
      <td>14</td>
      <td>4</td>
      <td>0</td>
      <td>69.300000</td>
      <td>304.900000</td>
      <td>55.000000</td>
      <td>62.800000</td>
      <td>1.646000</td>
      <td>0.0</td>
      <td>52.500000</td>
      <td>True</td>
      <td>148902.187500</td>
      <td>48.937500</td>
      <td>0.250000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Taylor Moore</th>
      <td>2825077</td>
      <td>865</td>
      <td>16</td>
      <td>55</td>
      <td>11</td>
      <td>1</td>
      <td>1</td>
      <td>70.400000</td>
      <td>301.200000</td>
      <td>57.400000</td>
      <td>65.600000</td>
      <td>1.790000</td>
      <td>0.0</td>
      <td>48.200000</td>
      <td>True</td>
      <td>176567.312500</td>
      <td>54.062500</td>
      <td>0.062500</td>
      <td>0.062500</td>
    </tr>
    <tr>
      <th>Tom Hoge</th>
      <td>8109202</td>
      <td>755</td>
      <td>48</td>
      <td>146</td>
      <td>30</td>
      <td>10</td>
      <td>1</td>
      <td>69.550000</td>
      <td>296.150000</td>
      <td>63.150000</td>
      <td>68.250000</td>
      <td>1.729000</td>
      <td>0.0</td>
      <td>52.050000</td>
      <td>True</td>
      <td>168941.708333</td>
      <td>15.729167</td>
      <td>0.208333</td>
      <td>0.020833</td>
    </tr>
    <tr>
      <th>Tony Finau</th>
      <td>15456084</td>
      <td>976</td>
      <td>63</td>
      <td>214</td>
      <td>52</td>
      <td>18</td>
      <td>4</td>
      <td>69.433333</td>
      <td>304.966667</td>
      <td>58.400000</td>
      <td>69.533333</td>
      <td>1.729667</td>
      <td>0.0</td>
      <td>55.066667</td>
      <td>True</td>
      <td>245334.666667</td>
      <td>15.492063</td>
      <td>0.285714</td>
      <td>0.063492</td>
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
      <td>15608924</td>
      <td>617</td>
      <td>53</td>
      <td>181</td>
      <td>47</td>
      <td>20</td>
      <td>3</td>
      <td>69.400000</td>
      <td>305.300000</td>
      <td>59.433333</td>
      <td>69.200000</td>
      <td>1.726667</td>
      <td>0.0</td>
      <td>61.100000</td>
      <td>True</td>
      <td>294508.000000</td>
      <td>11.641509</td>
      <td>0.377358</td>
      <td>0.056604</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Great! We have our second_cut dataframe which features golfers who qualify to fill the last 6 spots
# We can see there are some consistent names at the top of results categories

```


```python
second_cut.index.name = 'displayName'
names = second_cut.index
print(names)
```

    Index(['Adam Schenk', 'Andrew Putnam', 'Brendon Todd', 'Brian Harman',
           'Collin Morikawa', 'Denny McCarthy', 'Harris English', 'Hayden Buckley',
           'Jordan Spieth', 'Justin Thomas', 'Keith Mitchell', 'Patrick Cantlay',
           'Rickie Fowler', 'Sahith Theegala', 'Taylor Montgomery', 'Taylor Moore',
           'Tom Hoge', 'Tony Finau', 'Wyndham Clark', 'Xander Schauffele'],
          dtype='object', name='displayName')



```python
#let's export this in csv and take it to tableau

```


```python
second_cut.to_csv('/Users/joeyglenn4/Desktop/Case Study/second_cut.csv', index=True)
```


```python

```


```python

```
