+++
authors = ["nazarabrory"]
categories = ["Data validation"]
date = 2021-02-10
draft = false
tags = ["Survey","Validation","Interval Data", "Lithology", "Pandas"]
title = "DrillHole Data Validation using Pandas: Interval Data"
toc = false
[cover]
alternate = ""
caption = ""
image = ""
style = ""

+++


```python
import numpy as np
import pandas as pd

collar = pd.read_csv('CollarDataFix.csv')
survey = pd.read_csv('SurveyDataFix.csv')
lith = pd.read_csv('LithologyData.csv')
```

## Checking Interval Range


```python
def interval_len(interval_data):
    """Function to calculate minimum from value (interval min), maximum of to value (interval max), and interval length"""
    interval_data.reset_index(drop=True)
    ig = interval_data.groupby('BHID',dropna=False)
    igmin = ig['FROM'].min()
    igmin.name = 'interval_min'
    
    igmax = ig['TO'].max()
    igmax.name = 'interval_max'
    
    interval_len = igmax - igmin
    interval_len.name = 'interval_length'
    
    return pd.concat([igmin, igmax, interval_len], axis=1)
```


```python
interval_len(lith)
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
      <th>interval_min</th>
      <th>interval_max</th>
      <th>interval_length</th>
    </tr>
    <tr>
      <th>BHID</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ARC-046</th>
      <td>0.0</td>
      <td>100.00</td>
      <td>100.00</td>
    </tr>
    <tr>
      <th>BHD-003</th>
      <td>3.0</td>
      <td>100.00</td>
      <td>97.00</td>
    </tr>
    <tr>
      <th>BHP-008</th>
      <td>0.0</td>
      <td>75.00</td>
      <td>75.00</td>
    </tr>
    <tr>
      <th>BHP-008-A</th>
      <td>76.0</td>
      <td>190.00</td>
      <td>114.00</td>
    </tr>
    <tr>
      <th>BHP-009</th>
      <td>0.0</td>
      <td>56.00</td>
      <td>56.00</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>ZKY_6A03</th>
      <td>0.0</td>
      <td>126.60</td>
      <td>126.60</td>
    </tr>
    <tr>
      <th>ZKY_804</th>
      <td>0.0</td>
      <td>163.15</td>
      <td>163.15</td>
    </tr>
    <tr>
      <th>ZKY_8A01</th>
      <td>0.0</td>
      <td>155.00</td>
      <td>155.00</td>
    </tr>
    <tr>
      <th>ZKY_8A02</th>
      <td>0.0</td>
      <td>54.75</td>
      <td>54.75</td>
    </tr>
    <tr>
      <th>NaN</th>
      <td>0.0</td>
      <td>137.50</td>
      <td>137.50</td>
    </tr>
  </tbody>
</table>
<p>491 rows × 3 columns</p>
</div>




```python
# Displaying the interval data that start more than 5 m
interval_min = interval_len(lith)['interval_min']
interval_min[interval_min > 5]
```




    BHID
    BHP-008-A    76.0
    Name: interval_min, dtype: float64



As we can see, BHP-008-a have a interval_min = 76 m. It might be the adjacent of BHP-008. Let's check the collar and survey data of both BHID.


```python
collar[collar['BHID'].isin(['BHP-008', 'BHP-008-A'])]
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
      <th>BHID</th>
      <th>MAXDEPTH</th>
      <th>X</th>
      <th>Y</th>
      <th>Z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>BHP-008</td>
      <td>76.0</td>
      <td>172974.924</td>
      <td>47005.830</td>
      <td>580.689</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BHP-008-A</td>
      <td>190.0</td>
      <td>172978.697</td>
      <td>47000.926</td>
      <td>580.581</td>
    </tr>
  </tbody>
</table>
</div>




```python
survey[survey['BHID'].isin(['BHP-008', 'BHP-008-A'])]
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
      <th>BHID</th>
      <th>AT</th>
      <th>AZIMUTH</th>
      <th>DIP</th>
      <th>AT_diff</th>
      <th>AZIMUTH_diff</th>
      <th>DIP_diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>BHP-008</td>
      <td>0.0</td>
      <td>90.0</td>
      <td>-65.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BHP-008-A</td>
      <td>0.0</td>
      <td>90.0</td>
      <td>-65.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



From collar and survey data we know that BHP-008-A is an adjacent of BHP-008. So, these data could be merged into one BHID.


```python
# Change BHP-008-A into BHP-008
lith = lith.replace('BHP-008-A','BHP-008')
collar = collar[collar['BHID'] != 'BHP-008-A'].reset_index(drop=True)
survey = survey[survey['BHID'] != 'BHP-008-A'].reset_index(drop=True)
```

## Detect Missing BHID entries in interval File


```python
def display_holeid_na(interval_data):
    """Function to display a DataFrame that contain NaN BHID, along with previous and following row"""
    na_index = interval_data.loc[interval_data['BHID'].isna()].index.to_list()
    buffer_index= []
    for i in na_index:
        buffer_index.append(i-1)
        buffer_index.append(i+1)
    na_index = list(set(na_index + buffer_index))
    na_index.sort()
    
    if len(na_index) > 0: 
        return interval_data.loc[na_index]
    else:
        return "There's no missing HoleID found. (All data entries already have BHID value.)"
```


```python
pd.set_option('display.max_rows', 100)
i = display_holeid_na(lith)
i
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARC-046</td>
      <td>0.00</td>
      <td>1.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>1.00</td>
      <td>2.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>1.00</td>
      <td>3.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ARC-046</td>
      <td>3.00</td>
      <td>4.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ARC-046</td>
      <td>5.00</td>
      <td>6.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>7.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ARC-046</td>
      <td>7.00</td>
      <td>8.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>8</th>
      <td>NaN</td>
      <td>8.00</td>
      <td>NaN</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ARC-046</td>
      <td>9.00</td>
      <td>10.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>73</th>
      <td>ARC-046</td>
      <td>77.00</td>
      <td>78.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>74</th>
      <td>NaN</td>
      <td>78.00</td>
      <td>79.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>75</th>
      <td>ARC-046</td>
      <td>79.00</td>
      <td>80.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>79</th>
      <td>ARC-046</td>
      <td>83.00</td>
      <td>84.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>80</th>
      <td>NaN</td>
      <td>84.00</td>
      <td>85.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>81</th>
      <td>NaN</td>
      <td>85.00</td>
      <td>86.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>82</th>
      <td>NaN</td>
      <td>86.00</td>
      <td>87.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>83</th>
      <td>ARC-046</td>
      <td>87.00</td>
      <td>88.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>95</th>
      <td>ARC-046</td>
      <td>99.00</td>
      <td>100.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>96</th>
      <td>NaN</td>
      <td>0.00</td>
      <td>2.00</td>
      <td>SOIL</td>
    </tr>
    <tr>
      <th>97</th>
      <td>NaN</td>
      <td>2.00</td>
      <td>3.00</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>98</th>
      <td>BHD-003</td>
      <td>3.00</td>
      <td>3.80</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>223</th>
      <td>BHP-008</td>
      <td>74.00</td>
      <td>75.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>224</th>
      <td>NaN</td>
      <td>75.00</td>
      <td>76.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>225</th>
      <td>BHP-008</td>
      <td>76.00</td>
      <td>77.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>322</th>
      <td>BHP-009</td>
      <td>54.00</td>
      <td>56.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>323</th>
      <td>NaN</td>
      <td>56.00</td>
      <td>58.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>324</th>
      <td>NaN</td>
      <td>58.00</td>
      <td>60.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>325</th>
      <td>NaN</td>
      <td>2.00</td>
      <td>1.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>326</th>
      <td>NaN</td>
      <td>1.00</td>
      <td>2.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>327</th>
      <td>BTC_101</td>
      <td>2.00</td>
      <td>2.80</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>370</th>
      <td>HFD-020</td>
      <td>19.00</td>
      <td>20.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>371</th>
      <td>NaN</td>
      <td>20.00</td>
      <td>21.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>372</th>
      <td>NaN</td>
      <td>21.00</td>
      <td>22.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>373</th>
      <td>NaN</td>
      <td>22.00</td>
      <td>23.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>374</th>
      <td>HFD-020</td>
      <td>23.00</td>
      <td>24.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>378</th>
      <td>HFD-020</td>
      <td>27.00</td>
      <td>28.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>379</th>
      <td>NaN</td>
      <td>28.00</td>
      <td>29.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>380</th>
      <td>HFD-020</td>
      <td>29.00</td>
      <td>30.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>387</th>
      <td>HFD-020</td>
      <td>35.50</td>
      <td>36.50</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>388</th>
      <td>NaN</td>
      <td>36.50</td>
      <td>37.80</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>389</th>
      <td>HFD-020</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>413</th>
      <td>HFD-020</td>
      <td>69.00</td>
      <td>74.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>414</th>
      <td>NaN</td>
      <td>74.00</td>
      <td>79.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>415</th>
      <td>HFD-020</td>
      <td>79.00</td>
      <td>82.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>557</th>
      <td>HFD-022</td>
      <td>125.00</td>
      <td>127.00</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>558</th>
      <td>NaN</td>
      <td>127.00</td>
      <td>128.90</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>559</th>
      <td>NaN</td>
      <td>128.90</td>
      <td>129.55</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>560</th>
      <td>NaN</td>
      <td>129.55</td>
      <td>131.00</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>561</th>
      <td>NaN</td>
      <td>131.00</td>
      <td>137.50</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>562</th>
      <td>NaN</td>
      <td>0.00</td>
      <td>2.20</td>
      <td>SSL</td>
    </tr>
    <tr>
      <th>563</th>
      <td>NaN</td>
      <td>2.20</td>
      <td>3.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>564</th>
      <td>HFD-023</td>
      <td>3.00</td>
      <td>4.25</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>584</th>
      <td>HFD-023</td>
      <td>23.00</td>
      <td>23.90</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>585</th>
      <td>NaN</td>
      <td>23.90</td>
      <td>25.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>586</th>
      <td>NaN</td>
      <td>25.00</td>
      <td>28.20</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>587</th>
      <td>NaN</td>
      <td>28.20</td>
      <td>30.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>588</th>
      <td>HFD-023</td>
      <td>30.00</td>
      <td>31.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>636</th>
      <td>HFD-024</td>
      <td>103.00</td>
      <td>104.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>637</th>
      <td>NaN</td>
      <td>104.00</td>
      <td>106.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>638</th>
      <td>HFD-024</td>
      <td>106.00</td>
      <td>108.50</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>639</th>
      <td>NaN</td>
      <td>0.00</td>
      <td>1.50</td>
      <td>SMD</td>
    </tr>
    <tr>
      <th>640</th>
      <td>HFD-032</td>
      <td>1.50</td>
      <td>2.50</td>
      <td>SMD</td>
    </tr>
    <tr>
      <th>688</th>
      <td>HFD-032</td>
      <td>69.50</td>
      <td>74.50</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>689</th>
      <td>NaN</td>
      <td>74.50</td>
      <td>79.50</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>690</th>
      <td>HFD-032</td>
      <td>79.50</td>
      <td>84.50</td>
      <td>AND</td>
    </tr>
  </tbody>
</table>
</div>



## Patching missing BHID


```python
def patch_holeid_na(interval_data, loop=1):
    """Function to patch missing BHID. This algorithm try to patch NaN data with the previous or the following data. 
       Sometimes to patch all data, the function need to be run in some loops."""
    idc = interval_data.copy()
    idc.reset_index(drop=True, inplace=True)
    
    na_index = idc.loc[idc['BHID'].isna()].index.to_list()
    nacount = len(na_index)
    
    for r in range(loop):
        for i in idc.loc[idc['BHID'].isna()].index.to_list():
            na_index = idc.loc[idc['BHID'].isna()].index.to_list()
            if (idc.loc[i,'FROM'] == 0) and (pd.notna(idc.loc[i + 1, 'BHID'])):
                idc.loc[i, 'BHID'] = idc.loc[i + 1, 'BHID']

            elif (idc.loc[i, 'TO'] == idc.loc[i + 1,'FROM']) and (pd.notna(idc.loc[i + 1, 'BHID'])):
                idc.loc[i,'BHID'] = idc.loc[i + 1, 'BHID']

            elif (idc.loc[i,'FROM'] == idc.loc[i - 1,'TO']) and (pd.notna(idc.loc[i - 1, 'BHID'])):
                idc.loc[i,'BHID'] = idc.loc[i - 1, 'BHID']
    return idc
```


```python
lith = patch_holeid_na(lith, loop=2)
lith.loc[i.index]
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARC-046</td>
      <td>0.00</td>
      <td>1.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ARC-046</td>
      <td>1.00</td>
      <td>2.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARC-046</td>
      <td>1.00</td>
      <td>3.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ARC-046</td>
      <td>3.00</td>
      <td>4.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ARC-046</td>
      <td>5.00</td>
      <td>6.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>7.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ARC-046</td>
      <td>7.00</td>
      <td>8.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ARC-046</td>
      <td>8.00</td>
      <td>NaN</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ARC-046</td>
      <td>9.00</td>
      <td>10.00</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>73</th>
      <td>ARC-046</td>
      <td>77.00</td>
      <td>78.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>74</th>
      <td>ARC-046</td>
      <td>78.00</td>
      <td>79.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>75</th>
      <td>ARC-046</td>
      <td>79.00</td>
      <td>80.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>79</th>
      <td>ARC-046</td>
      <td>83.00</td>
      <td>84.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>80</th>
      <td>ARC-046</td>
      <td>84.00</td>
      <td>85.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>81</th>
      <td>ARC-046</td>
      <td>85.00</td>
      <td>86.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>82</th>
      <td>ARC-046</td>
      <td>86.00</td>
      <td>87.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>83</th>
      <td>ARC-046</td>
      <td>87.00</td>
      <td>88.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>95</th>
      <td>ARC-046</td>
      <td>99.00</td>
      <td>100.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>96</th>
      <td>BHD-003</td>
      <td>0.00</td>
      <td>2.00</td>
      <td>SOIL</td>
    </tr>
    <tr>
      <th>97</th>
      <td>BHD-003</td>
      <td>2.00</td>
      <td>3.00</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>98</th>
      <td>BHD-003</td>
      <td>3.00</td>
      <td>3.80</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>223</th>
      <td>BHP-008</td>
      <td>74.00</td>
      <td>75.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>224</th>
      <td>BHP-008</td>
      <td>75.00</td>
      <td>76.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>225</th>
      <td>BHP-008</td>
      <td>76.00</td>
      <td>77.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>322</th>
      <td>BHP-009</td>
      <td>54.00</td>
      <td>56.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>323</th>
      <td>BHP-009</td>
      <td>56.00</td>
      <td>58.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>324</th>
      <td>BHP-009</td>
      <td>58.00</td>
      <td>60.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>325</th>
      <td>BTC_101</td>
      <td>2.00</td>
      <td>1.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>326</th>
      <td>BTC_101</td>
      <td>1.00</td>
      <td>2.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>327</th>
      <td>BTC_101</td>
      <td>2.00</td>
      <td>2.80</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>370</th>
      <td>HFD-020</td>
      <td>19.00</td>
      <td>20.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>371</th>
      <td>HFD-020</td>
      <td>20.00</td>
      <td>21.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>372</th>
      <td>HFD-020</td>
      <td>21.00</td>
      <td>22.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>373</th>
      <td>HFD-020</td>
      <td>22.00</td>
      <td>23.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>374</th>
      <td>HFD-020</td>
      <td>23.00</td>
      <td>24.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>378</th>
      <td>HFD-020</td>
      <td>27.00</td>
      <td>28.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>379</th>
      <td>HFD-020</td>
      <td>28.00</td>
      <td>29.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>380</th>
      <td>HFD-020</td>
      <td>29.00</td>
      <td>30.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>387</th>
      <td>HFD-020</td>
      <td>35.50</td>
      <td>36.50</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>388</th>
      <td>HFD-020</td>
      <td>36.50</td>
      <td>37.80</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>389</th>
      <td>HFD-020</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>413</th>
      <td>HFD-020</td>
      <td>69.00</td>
      <td>74.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>414</th>
      <td>HFD-020</td>
      <td>74.00</td>
      <td>79.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>415</th>
      <td>HFD-020</td>
      <td>79.00</td>
      <td>82.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>557</th>
      <td>HFD-022</td>
      <td>125.00</td>
      <td>127.00</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>558</th>
      <td>HFD-022</td>
      <td>127.00</td>
      <td>128.90</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>559</th>
      <td>HFD-022</td>
      <td>128.90</td>
      <td>129.55</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>560</th>
      <td>HFD-022</td>
      <td>129.55</td>
      <td>131.00</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>561</th>
      <td>HFD-022</td>
      <td>131.00</td>
      <td>137.50</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>562</th>
      <td>HFD-023</td>
      <td>0.00</td>
      <td>2.20</td>
      <td>SSL</td>
    </tr>
    <tr>
      <th>563</th>
      <td>HFD-023</td>
      <td>2.20</td>
      <td>3.00</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>564</th>
      <td>HFD-023</td>
      <td>3.00</td>
      <td>4.25</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>584</th>
      <td>HFD-023</td>
      <td>23.00</td>
      <td>23.90</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>585</th>
      <td>HFD-023</td>
      <td>23.90</td>
      <td>25.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>586</th>
      <td>HFD-023</td>
      <td>25.00</td>
      <td>28.20</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>587</th>
      <td>HFD-023</td>
      <td>28.20</td>
      <td>30.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>588</th>
      <td>HFD-023</td>
      <td>30.00</td>
      <td>31.00</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>636</th>
      <td>HFD-024</td>
      <td>103.00</td>
      <td>104.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>637</th>
      <td>HFD-024</td>
      <td>104.00</td>
      <td>106.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>638</th>
      <td>HFD-024</td>
      <td>106.00</td>
      <td>108.50</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>639</th>
      <td>HFD-032</td>
      <td>0.00</td>
      <td>1.50</td>
      <td>SMD</td>
    </tr>
    <tr>
      <th>640</th>
      <td>HFD-032</td>
      <td>1.50</td>
      <td>2.50</td>
      <td>SMD</td>
    </tr>
    <tr>
      <th>688</th>
      <td>HFD-032</td>
      <td>69.50</td>
      <td>74.50</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>689</th>
      <td>HFD-032</td>
      <td>74.50</td>
      <td>79.50</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>690</th>
      <td>HFD-032</td>
      <td>79.50</td>
      <td>84.50</td>
      <td>AND</td>
    </tr>
  </tbody>
</table>
</div>



## Detect Missing FROM - TO entries in interval File


```python
def display_fromto_na(interval_data):
    """Function to display a DataFrarme that contain NaN for FROM and TO columns, along with previous and following row"""
    na_index = interval_data.loc[interval_data[['FROM', 'TO']].isna().any(axis=1)].index.to_list()
    buffer_index= []
    for i in na_index:
        buffer_index.append(i-1)
        buffer_index.append(i+1)
    na_index_c = list(set( na_index + buffer_index))
    na_index_c.sort()
    if len(na_index_c) > 0: 
        return interval_data.loc[na_index_c]
    else:
        return "There's no missing FROM-TO found. (All data entries already have FROM-TO value.)"
```


```python
f = display_fromto_na(lith)
f
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>ARC-046</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>7.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ARC-046</td>
      <td>7.0</td>
      <td>8.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ARC-046</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ARC-046</td>
      <td>9.0</td>
      <td>10.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ARC-046</td>
      <td>14.0</td>
      <td>13.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>14</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>15</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>16</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>17</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>18</th>
      <td>ARC-046</td>
      <td>22.0</td>
      <td>23.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>152</th>
      <td>BHD-003</td>
      <td>57.6</td>
      <td>58.8</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>153</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>154</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>155</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>156</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>157</th>
      <td>BHD-003</td>
      <td>65.0</td>
      <td>67.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>259</th>
      <td>BHP-008</td>
      <td>122.0</td>
      <td>124.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>260</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>261</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>262</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>263</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>264</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>265</th>
      <td>BHP-008</td>
      <td>132.0</td>
      <td>133.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>266</th>
      <td>BHP-008</td>
      <td>133.0</td>
      <td>134.0</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>267</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>136.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>268</th>
      <td>BHP-008</td>
      <td>136.0</td>
      <td>137.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>307</th>
      <td>BHP-009</td>
      <td>25.0</td>
      <td>27.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>308</th>
      <td>BHP-009</td>
      <td>27.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>309</th>
      <td>BHP-009</td>
      <td>32.0</td>
      <td>34.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>330</th>
      <td>BTC_102</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>331</th>
      <td>BTC_102</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>332</th>
      <td>BTC_102</td>
      <td>3.5</td>
      <td>4.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>345</th>
      <td>BTC_301_A-B-9</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>346</th>
      <td>BTC3A01</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>347</th>
      <td>BTC3A01</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>356</th>
      <td>HFD-020</td>
      <td>4.5</td>
      <td>5.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>357</th>
      <td>HFD-020</td>
      <td>NaN</td>
      <td>6.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>358</th>
      <td>HFD-020</td>
      <td>6.5</td>
      <td>7.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>364</th>
      <td>HFD-020</td>
      <td>12.5</td>
      <td>13.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>365</th>
      <td>HFD-020</td>
      <td>13.5</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>366</th>
      <td>HFD-020</td>
      <td>14.5</td>
      <td>15.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>368</th>
      <td>HFD-020</td>
      <td>17.2</td>
      <td>18.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>369</th>
      <td>HFD-020</td>
      <td>18.0</td>
      <td>NaN</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>370</th>
      <td>HFD-020</td>
      <td>19.0</td>
      <td>20.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>375</th>
      <td>HFD-020</td>
      <td>24.0</td>
      <td>25.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>376</th>
      <td>HFD-020</td>
      <td>NaN</td>
      <td>26.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>377</th>
      <td>HFD-020</td>
      <td>26.0</td>
      <td>27.0</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>384</th>
      <td>HFD-020</td>
      <td>32.5</td>
      <td>33.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>385</th>
      <td>HFD-020</td>
      <td>33.5</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>386</th>
      <td>HFD-020</td>
      <td>34.5</td>
      <td>35.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>388</th>
      <td>HFD-020</td>
      <td>36.5</td>
      <td>37.8</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>389</th>
      <td>HFD-020</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>390</th>
      <td>HFD-020</td>
      <td>40.8</td>
      <td>42.5</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>



### Patching missing FROM TO


```python
def patch_fromto_na(interval_data, shrink=False, infer=False):
    """Function to patch missing FROM -TO entries. This infer data from the previous or the following data.
        There are two parameter: 
        1. shrink = (True/False) if set to True, interval that have the same entries will be collapsed/combined
        2. infer = (True/False) if set to True, the function try to infer the NaN by calculating mid-point"""
    
    interval_data.reset_index(drop=True, inplace=True)
    
    # Patching FROM
    na_index_from = interval_data.loc[interval_data['FROM'].isna()].index.to_list()
    for i in na_index_from:
        if pd.notna(interval_data.loc[i-1, 'TO']):
            interval_data.loc[i, 'FROM']  = interval_data.loc[i-1, 'TO']
            
    # Patching TO
    na_index_to = interval_data.loc[interval_data['TO'].isna()].index.to_list()
    for i in na_index_to:
        if pd.notna(interval_data.loc[i+1, 'FROM']):
            interval_data.loc[i, 'TO']  = interval_data.loc[i+1, 'FROM']
    
    # Shrinking NaN if the entries still the same
    if shrink==True:
        # Updating na_index for both from and to
        na_index_from = interval_data.loc[interval_data['FROM'].isna()].index.to_list()
        na_index_to = interval_data.loc[interval_data['TO'].isna()].index.to_list()
        na_index_all = list(set(na_index_from) & set(na_index_to))
  
        # Preparing the list of column to be checked whether the row is still the same BHID.
        col_check = [x for x in interval_data.columns.to_list() if x not in ['FROM', 'TO']]
        
        # Checking whether the row is shrink-able
        for i in na_index_all:
            x = i + 1
            a = list(interval_data.loc[i, col_check])
            b = list(interval_data.loc[x, col_check])
            if a == b:     
                interval_data = interval_data.drop(index=i)
                
    interval_data.reset_index(drop=True, inplace=True)    
    if infer == True:
        na_index_from = interval_data.loc[interval_data['FROM'].isna()].index.to_list()
        na_index_to = interval_data.loc[interval_data['TO'].isna()].index.to_list()

        for i in na_index_to:
            if pd.notna(interval_data.loc[i, 'FROM']) or pd.notna(interval_data.loc[i +1, 'TO']):
                mid = (interval_data.loc[i, 'FROM'] + ((interval_data.loc[i +1, 'TO'] - interval_data.loc[i, 'FROM'])/2))
                interval_data.loc[i, 'TO'] = mid

        for i in na_index_from:
            if i != 0:
                if pd.notna(interval_data.loc[i-1, 'FROM']) or pd.notna(interval_data.loc[i, 'TO']):
                    mid = (interval_data.loc[i-1, 'FROM'] + ((interval_data.loc[i, 'TO'] - interval_data.loc[i-1, 'FROM'])/2))
                    interval_data.loc[i, 'FROM'] = mid
            else:
                interval_data.loc[i, 'FROM'] = 0        
    
    return interval_data
```

I've code patching algorithm onto three level of 'aggressiveness' based on confidence level. Below is the illustration on the difference of each parameters.


```python
f1 = patch_fromto_na(f, shrink=False, infer=False)
display_fromto_na(f1)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>ARC-046</td>
      <td>14.0</td>
      <td>13.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ARC-046</td>
      <td>13.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>9</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>22.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>11</th>
      <td>ARC-046</td>
      <td>22.0</td>
      <td>23.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>12</th>
      <td>BHD-003</td>
      <td>57.6</td>
      <td>58.8</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>13</th>
      <td>BHD-003</td>
      <td>58.8</td>
      <td>NaN</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>15</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>16</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>65.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BHD-003</td>
      <td>65.0</td>
      <td>67.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>18</th>
      <td>BHP-008</td>
      <td>122.0</td>
      <td>124.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>19</th>
      <td>BHP-008</td>
      <td>124.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>20</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>21</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>22</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>23</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>132.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>24</th>
      <td>BHP-008</td>
      <td>132.0</td>
      <td>133.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>




```python
f2 = patch_fromto_na(f, shrink=True, infer=False)
display_fromto_na(f2)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>ARC-046</td>
      <td>14.0</td>
      <td>13.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ARC-046</td>
      <td>13.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ARC-046</td>
      <td>NaN</td>
      <td>22.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>8</th>
      <td>ARC-046</td>
      <td>22.0</td>
      <td>23.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BHD-003</td>
      <td>57.6</td>
      <td>58.8</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BHD-003</td>
      <td>58.8</td>
      <td>NaN</td>
      <td>BXA</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BHD-003</td>
      <td>NaN</td>
      <td>65.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>12</th>
      <td>BHD-003</td>
      <td>65.0</td>
      <td>67.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>13</th>
      <td>BHP-008</td>
      <td>122.0</td>
      <td>124.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BHP-008</td>
      <td>124.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>15</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>16</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>132.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BHP-008</td>
      <td>132.0</td>
      <td>133.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>




```python
f3 = patch_fromto_na(f, shrink=True, infer=True)
display_fromto_na(f3)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13</th>
      <td>BHP-008</td>
      <td>122.0</td>
      <td>124.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BHP-008</td>
      <td>124.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>15</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>16</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>132.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BHP-008</td>
      <td>132.0</td>
      <td>133.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>



Let's apply this function to our 'lith' data.


```python
lith = patch_fromto_na(lith, shrink=True, infer=True)
```


```python
display_fromto_na(lith)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>254</th>
      <td>BHP-008</td>
      <td>122.0</td>
      <td>124.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>255</th>
      <td>BHP-008</td>
      <td>124.0</td>
      <td>NaN</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>256</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>BFL</td>
    </tr>
    <tr>
      <th>257</th>
      <td>BHP-008</td>
      <td>NaN</td>
      <td>132.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>258</th>
      <td>BHP-008</td>
      <td>132.0</td>
      <td>133.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>



The remaining unfilled data need to be fill manually since there's no clue about the extent. It need to be check on the core box. Let's assume we've checked the interval and let's fill out the data based on our measurement.


```python
lith.loc[255, 'TO'] = 126
lith.loc[256, 'TO'] = 130
lith = patch_fromto_na(lith)
```


```python
display_fromto_na(lith)
```




    "There's no missing FROM-TO found. (All data entries already have FROM-TO value.)"



## Filter Interval data based on Collar Data to be used

All interval data need to have collar and survey data, so the data can be used. here it is the code to filter out the data availability based on collar data.


```python
def interval_incollar(interval_data,collar_data):
    """Function to filter interval data based on available collar data"""
    return interval_data[interval_data['BHID'].isin(collar_data['BHID'])].reset_index(drop=True)
```


```python
interval_incollar(lith, collar)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARC-046</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ARC-046</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARC-046</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ARC-046</td>
      <td>3.0</td>
      <td>4.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ARC-046</td>
      <td>4.0</td>
      <td>5.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>12755</th>
      <td>YRC-287</td>
      <td>25.0</td>
      <td>26.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>12756</th>
      <td>YRC-287</td>
      <td>26.0</td>
      <td>27.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>12757</th>
      <td>YRC-287</td>
      <td>27.0</td>
      <td>28.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>12758</th>
      <td>YRC-287</td>
      <td>28.0</td>
      <td>29.0</td>
      <td>AND</td>
    </tr>
    <tr>
      <th>12759</th>
      <td>YRC-287</td>
      <td>29.0</td>
      <td>30.0</td>
      <td>AND</td>
    </tr>
  </tbody>
</table>
<p>12760 rows × 4 columns</p>
</div>



Here it is the code to make sure data integrity. First code is to check whether there's interval data that is not in collar. Second code is to check whether there's collar data that doesn't have interval data.


```python
def interval_notincollar(interval_data,collar_data):
    """Function to filter interval data based on available collar data"""
    return interval_data[~interval_data['BHID'].isin(collar_data['BHID'])].reset_index(drop=True)
```


```python
interval_notincollar(lith, collar)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HFD-068</td>
      <td>0.00</td>
      <td>39.20</td>
      <td>SVE</td>
    </tr>
    <tr>
      <th>1</th>
      <td>HFD-068</td>
      <td>39.20</td>
      <td>50.60</td>
      <td>SVE</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HFD-068</td>
      <td>50.60</td>
      <td>57.25</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HFD-068</td>
      <td>57.25</td>
      <td>63.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>4</th>
      <td>HFD-068</td>
      <td>63.00</td>
      <td>68.00</td>
      <td>SCB</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3498</th>
      <td>ZKY_8A02</td>
      <td>38.40</td>
      <td>51.70</td>
      <td>CLY</td>
    </tr>
    <tr>
      <th>3499</th>
      <td>ZKY_8A02</td>
      <td>51.70</td>
      <td>52.00</td>
      <td>CLOS</td>
    </tr>
    <tr>
      <th>3500</th>
      <td>ZKY_8A02</td>
      <td>52.00</td>
      <td>53.90</td>
      <td>CLYST</td>
    </tr>
    <tr>
      <th>3501</th>
      <td>ZKY_8A02</td>
      <td>53.90</td>
      <td>54.40</td>
      <td>CLOS</td>
    </tr>
    <tr>
      <th>3502</th>
      <td>ZKY_8A02</td>
      <td>54.40</td>
      <td>54.75</td>
      <td>DALT</td>
    </tr>
  </tbody>
</table>
<p>3503 rows × 4 columns</p>
</div>




```python
def collar_withnointerval(collar_data, interval_data):
    """Function to filter collar data that doesn't have interval data"""
    a = collar_data[~collar_data['BHID'].isin(interval_data['BHID'])].reset_index(drop=True)
    if len(a) > 0:
        return a
    else:
        return "All entries in collar data have interval data"
```


```python
collar_withnointerval(collar,lith)
```




    'All entries in collar data have interval data'



Let's filter the interval data based on collar data availability.


```python
lith = interval_incollar(lith, collar)
```

### 'FROM' less than previous 'TO'


```python
def display_fltpt(interval_data):
    """Function to display FROM whcih less than previous TO."""
    interval_data.reset_index(drop=True, inplace=True)
    idg = interval_data.groupby('BHID', dropna=False)
    fltpt = idg.apply(lambda x : x['FROM'] < (x['TO'].shift(1)))
    fltpt =  fltpt.reset_index(drop=True)
    fltpt_index = interval_data[fltpt].index.to_list()
    if len(fltpt_index) > 0:
        buffer_index= []
        for i in fltpt_index:
            buffer_index.append(i-1)
        index_c = list(set( fltpt_index + buffer_index))
        index_c.sort()
        return interval_data.loc[index_c]
    else:
        return "There's no FROM which less than prevoius than TO"
```


```python
ih = display_fltpt(lith)
ih
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>ARC-046</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARC-046</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>SCQ</td>
    </tr>
  </tbody>
</table>
</div>




```python
def fix_fltpt(interval_data):
    """Function to fix value of FROM which less than previous TO. The FROM value will be override with previous TO """
    interval_data.reset_index(drop=True, inplace=True)
    idg = interval_data.groupby('BHID', dropna=False)
    fltpt = idg.apply(lambda x : x['FROM'] < (x['TO'].shift(1)))
    fltpt =  fltpt.reset_index(drop=True)
    fltpt_index = interval_data[fltpt].index.to_list()
    if len(fltpt_index) > 0:
        for i in fltpt_index:
            if i !=0 :
                bhid = interval_data.loc[i, 'BHID']
                prevbhid = interval_data.loc[i-1, 'BHID']
                if bhid == prevbhid:
                    interval_data.loc[i, 'FROM'] = interval_data.loc[i-1, 'TO'] 
        return interval_data
    else:
        return "There's nothing to fix"
```


```python
fix_fltpt(ih)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARC-046</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SCQ</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ARC-046</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>SCQ</td>
    </tr>
  </tbody>
</table>
</div>




```python
lith = fix_fltpt(lith)
```


```python
display_fltpt(lith)
```




    "There's no FROM which less than prevoius than TO"



### 'FROM' greater than or equal to 'TO'


```python
def display_fgtet(interval_data, buffer=False):
    """Function to Display FROM which greater or equal than TO"""
    idc = interval_data.copy()
    idc.reset_index(drop=True, inplace=True)
    idg = idc.groupby('BHID')
    fgtet = idg.apply(lambda x : x['FROM'] >= x['TO']).reset_index(drop=True)
    fgtet_index = idc[fgtet].index.to_list()
    fgtet_index = fgtet_index[:-1]
    if buffer == True:
        if len(fgtet_index) > 0:
            buffer_index= []
            for i in fgtet_index:
                buffer_index.append(i-1)
                buffer_index.append(i+1)

            index_c = list(set(fgtet_index + buffer_index))
            index_c.sort()
            return idc.loc[index_c]
        else:
            return "There's no FROM which greater than or equal to TO."
    else:
        if len(fgtet_index) > 0:
            fgtet_index.sort()
            return idc.loc[fgtet_index]
        else:
            return "There's no FROM which greater than or equal to TO."
```


```python
fg = display_fgtet(lith, buffer=False)
fg
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>ARC-046</td>
      <td>14.0</td>
      <td>13.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>121</th>
      <td>BHD-003</td>
      <td>27.0</td>
      <td>27.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>124</th>
      <td>BHD-003</td>
      <td>31.0</td>
      <td>30.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>318</th>
      <td>BTC_101</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>




```python
fq = display_fgtet(lith, buffer=True)
fq
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11</th>
      <td>ARC-046</td>
      <td>11.0</td>
      <td>12.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ARC-046</td>
      <td>14.0</td>
      <td>13.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>13</th>
      <td>ARC-046</td>
      <td>13.0</td>
      <td>17.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>120</th>
      <td>BHD-003</td>
      <td>25.0</td>
      <td>26.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>121</th>
      <td>BHD-003</td>
      <td>27.0</td>
      <td>27.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>122</th>
      <td>BHD-003</td>
      <td>27.0</td>
      <td>28.0</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>123</th>
      <td>BHD-003</td>
      <td>28.0</td>
      <td>29.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>124</th>
      <td>BHD-003</td>
      <td>31.0</td>
      <td>30.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>125</th>
      <td>BHD-003</td>
      <td>30.0</td>
      <td>31.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>317</th>
      <td>BHP-009</td>
      <td>58.0</td>
      <td>60.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>318</th>
      <td>BTC_101</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>319</th>
      <td>BTC_101</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>




```python
def fix_fgtet(interval_data):
    """Function to fix FROM which greater or equal than TO"""
    interval_data.reset_index(drop=True, inplace=True)
    idg = interval_data.groupby('BHID', dropna=False)
    fgtet = idg.apply(lambda x : x['FROM'] >= x['TO'])
    fgtet =  fgtet.reset_index(drop=True)
    fgtet_index = interval_data[fgtet].index.to_list()
    if len(fgtet_index) > 0:
        for i in fgtet_index:
            if i !=0 :
                bhid = interval_data.loc[i, 'BHID']
                prevbhid = interval_data.loc[i-1, 'BHID']
                if bhid == prevbhid:
                    interval_data.loc[i, 'FROM'] = interval_data.loc[i-1, 'TO'] 
                else:
                    interval_data.loc[i, 'FROM'] = 0
        return interval_data
    else:
        return "There's nothing to fix"
```


```python
fix_fgtet(fq)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARC-046</td>
      <td>11.0</td>
      <td>12.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ARC-046</td>
      <td>12.0</td>
      <td>13.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARC-046</td>
      <td>13.0</td>
      <td>17.5</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BHD-003</td>
      <td>25.0</td>
      <td>26.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BHD-003</td>
      <td>26.0</td>
      <td>27.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>5</th>
      <td>BHD-003</td>
      <td>27.0</td>
      <td>28.0</td>
      <td>SCG</td>
    </tr>
    <tr>
      <th>6</th>
      <td>BHD-003</td>
      <td>28.0</td>
      <td>29.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>7</th>
      <td>BHD-003</td>
      <td>29.0</td>
      <td>30.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>8</th>
      <td>BHD-003</td>
      <td>30.0</td>
      <td>31.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BHP-009</td>
      <td>58.0</td>
      <td>60.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BTC_101</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>SLM</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BTC_101</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>SLM</td>
    </tr>
  </tbody>
</table>
</div>




```python
lith = fix_fgtet(lith)
```


```python
display_fgtet(lith)
```




    "There's no FROM which greater than or equal to TO."



### Checking Lithology column


```python
def display_lith_na(interval_data, buffer=False):
    idc = interval_data.copy()
    na_index = idc[idc['LITHOLOGY'].isna()].index.to_list()
    if buffer == True:
        buffer_index = []
        for i in na_index:
            buffer_index.append(i-1)
            buffer_index.append(i+1)
            index_c = list(set(na_index + buffer_index))
            index_c.sort()
        return idc.loc[index_c]
    else:
        return idc.loc[na_index]
```


```python
display_lith_na(lith, buffer=True)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>386</th>
      <td>HFD-020</td>
      <td>44.5</td>
      <td>45.4</td>
      <td>SMD</td>
    </tr>
    <tr>
      <th>387</th>
      <td>HFD-020</td>
      <td>45.4</td>
      <td>48.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>388</th>
      <td>HFD-020</td>
      <td>48.0</td>
      <td>49.0</td>
      <td>SMD</td>
    </tr>
  </tbody>
</table>
</div>



Let's assume the NaN lithology is same as previous and following LITHOLOGY. So, let's change NaN at row 387 into SMD.


```python
lith.loc[387, 'LITHOLOGY'] = 'SMD'
```


```python
lith['LITHOLOGY'].unique()
```




    array(['SCQ', 'SLM', 'SOIL', 'BXA', 'CLOS', 'SCG', 'AND', 'Andesit',
           'BFL', 'SMD', 'TANAH', 'DALT', 'CLY', 'SCB', 'SSL', 'SST', 'SVE',
           'STR', 'SCF', 'BSH', 'NCR', 'CAV', 'CALV', 'LMS', 'SCL'],
          dtype=object)




```python
lith_list = ['SCQ', 'SLM', 'SOIL', 'BXA', 'CLOS', 'SCG', 'AND', 'BFL', 
             'SMD', 'DALT', 'CLY', 'SCB', 'SSL', 'SST', 'SVE', 'STR',
             'SCF', 'BSH', 'NCR', 'CAV', 'CALV', 'LMS', 'SCL']
```


```python
def display_unmatched(interval_data, checklist):
    idc = interval_data.copy()
    display = idc[~idc['LITHOLOGY'].isin(lith_list)]
    if len(display) > 0:
        return display
    else:
        return "There's no unmatched entries"
```


```python
display_unmatched(lith, checklist=lith_list)
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
      <th>BHID</th>
      <th>FROM</th>
      <th>TO</th>
      <th>LITHOLOGY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>189</th>
      <td>BHP-008</td>
      <td>34.0</td>
      <td>36.0</td>
      <td>Andesit</td>
    </tr>
    <tr>
      <th>191</th>
      <td>BHP-008</td>
      <td>37.0</td>
      <td>38.0</td>
      <td>Andesit</td>
    </tr>
    <tr>
      <th>282</th>
      <td>BHP-009</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>TANAH</td>
    </tr>
  </tbody>
</table>
</div>




```python
lith = lith.replace({'Andesit':'AND', 'TANAH':'SOIL'})
```


```python
display_unmatched(lith,checklist=lith_list)
```




    "There's no unmatched entries"



### Exporting data 


```python
lith.to_csv('LithologyDataFix.csv')
collar.to_csv('CollarDataFix.csv')
survey.to_csv('SurveyDataFix.csv')
```
