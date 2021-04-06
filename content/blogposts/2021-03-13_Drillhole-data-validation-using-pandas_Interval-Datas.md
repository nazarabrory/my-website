+++
authors = ["nazarabrory"]
categories = ["Data validation"]
date = 2021-03-13
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

Drillhole Data consist of at least three Data (Collar, Survey, and Interval Data). All of this data arranged in table form with specified columns connected with HOLE ID as the index key. Survey Data is a representation of distance and direction on 3D space for each Drillhole. Meanwhile, Interval data is a representation of data which measured on a specified interval/ scale. So, Interval data need to have at least these columns: HOLE_ID, FROM, TO, and Interval Data (such as Lithology, Assay, etc.)

Basically, these are things to consider when dealing with data validation for Interval data:

1. Checking and correcting Interval Range
2. Detect and Fixing Missing BHID entries in interval File
3. Detect and Fixing Missing FROM - TO entries in interval File
4. Filter Interval data based on Collar Data to be used
5. Detect and Fixing FROM less than previous TO
6. Detect and Fixing FROM greater than or equal to TO
7. Detect and Fixing missing Interval Entries
7. Checking inappropriate entries of Interval Data


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
print(interval_len(lith))
```

               interval_min  interval_max  interval_length
    BHID                                                  
    ARC-046             0.0        100.00           100.00
    BHD-003             3.0        100.00            97.00
    BHP-008             0.0         75.00            75.00
    BHP-008-A          76.0        190.00           114.00
    BHP-009             0.0         56.00            56.00
    ...                 ...           ...              ...
    ZKY_6A03            0.0        126.60           126.60
    ZKY_804             0.0        163.15           163.15
    ZKY_8A01            0.0        155.00           155.00
    ZKY_8A02            0.0         54.75            54.75
    NaN                 0.0        137.50           137.50
    
    [491 rows x 3 columns]
    


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
print(collar[collar['BHID'].isin(['BHP-008', 'BHP-008-A'])])
```

            BHID  MAXDEPTH           X          Y        Z
    2    BHP-008      76.0  172974.924  47005.830  580.689
    3  BHP-008-A     190.0  172978.697  47000.926  580.581
    


```python
print(survey[survey['BHID'].isin(['BHP-008', 'BHP-008-A'])])
```

            BHID   AT  AZIMUTH   DIP  AT_diff  AZIMUTH_diff  DIP_diff
    3    BHP-008  0.0     90.0 -65.0      0.0           0.0       0.0
    4  BHP-008-A  0.0     90.0 -65.0      0.0           0.0       0.0
    

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
print(i)
```

            BHID    FROM      TO LITHOLOGY
    0    ARC-046    0.00    1.00       SCQ
    1        NaN    1.00    2.00       SCQ
    2        NaN    1.00    3.00       SCQ
    3    ARC-046    3.00    4.00       SCQ
    5    ARC-046    5.00    6.00       SCQ
    6        NaN     NaN    7.00       SCQ
    7    ARC-046    7.00    8.00       SCQ
    8        NaN    8.00     NaN       SCQ
    9    ARC-046    9.00   10.00       SCQ
    73   ARC-046   77.00   78.00       SLM
    74       NaN   78.00   79.00       SLM
    75   ARC-046   79.00   80.00       SLM
    79   ARC-046   83.00   84.00       SLM
    80       NaN   84.00   85.00       SLM
    81       NaN   85.00   86.00       SLM
    82       NaN   86.00   87.00       SLM
    83   ARC-046   87.00   88.00       SLM
    95   ARC-046   99.00  100.00       SLM
    96       NaN    0.00    2.00      SOIL
    97       NaN    2.00    3.00       BXA
    98   BHD-003    3.00    3.80       BXA
    223  BHP-008   74.00   75.00       SLM
    224      NaN   75.00   76.00       SLM
    225  BHP-008   76.00   77.00       SLM
    322  BHP-009   54.00   56.00       SLM
    323      NaN   56.00   58.00       SLM
    324      NaN   58.00   60.00       SLM
    325      NaN    2.00    1.00       SLM
    326      NaN    1.00    2.00       SLM
    327  BTC_101    2.00    2.80       SLM
    370  HFD-020   19.00   20.00       AND
    371      NaN   20.00   21.00       AND
    372      NaN   21.00   22.00       AND
    373      NaN   22.00   23.00       AND
    374  HFD-020   23.00   24.00       AND
    378  HFD-020   27.00   28.00       SCB
    379      NaN   28.00   29.00       SCB
    380  HFD-020   29.00   30.00       SCB
    387  HFD-020   35.50   36.50       SLM
    388      NaN   36.50   37.80       SLM
    389  HFD-020     NaN     NaN       SLM
    413  HFD-020   69.00   74.00       SLM
    414      NaN   74.00   79.00       SLM
    415  HFD-020   79.00   82.00       SLM
    557  HFD-022  125.00  127.00       SCG
    558      NaN  127.00  128.90       SCG
    559      NaN  128.90  129.55       SCG
    560      NaN  129.55  131.00       SCG
    561      NaN  131.00  137.50       SLM
    562      NaN    0.00    2.20       SSL
    563      NaN    2.20    3.00       AND
    564  HFD-023    3.00    4.25       AND
    584  HFD-023   23.00   23.90       SLM
    585      NaN   23.90   25.00       SCB
    586      NaN   25.00   28.20       SLM
    587      NaN   28.20   30.00       SLM
    588  HFD-023   30.00   31.00       SLM
    636  HFD-024  103.00  104.00       SCB
    637      NaN  104.00  106.00       SCB
    638  HFD-024  106.00  108.50       SCB
    639      NaN    0.00    1.50       SMD
    640  HFD-032    1.50    2.50       SMD
    688  HFD-032   69.50   74.50       AND
    689      NaN   74.50   79.50       AND
    690  HFD-032   79.50   84.50       AND
    

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
print(lith.loc[i.index])
```

            BHID    FROM      TO LITHOLOGY
    0    ARC-046    0.00    1.00       SCQ
    1    ARC-046    1.00    2.00       SCQ
    2    ARC-046    1.00    3.00       SCQ
    3    ARC-046    3.00    4.00       SCQ
    5    ARC-046    5.00    6.00       SCQ
    6    ARC-046     NaN    7.00       SCQ
    7    ARC-046    7.00    8.00       SCQ
    8    ARC-046    8.00     NaN       SCQ
    9    ARC-046    9.00   10.00       SCQ
    73   ARC-046   77.00   78.00       SLM
    74   ARC-046   78.00   79.00       SLM
    75   ARC-046   79.00   80.00       SLM
    79   ARC-046   83.00   84.00       SLM
    80   ARC-046   84.00   85.00       SLM
    81   ARC-046   85.00   86.00       SLM
    82   ARC-046   86.00   87.00       SLM
    83   ARC-046   87.00   88.00       SLM
    95   ARC-046   99.00  100.00       SLM
    96   BHD-003    0.00    2.00      SOIL
    97   BHD-003    2.00    3.00       BXA
    98   BHD-003    3.00    3.80       BXA
    223  BHP-008   74.00   75.00       SLM
    224  BHP-008   75.00   76.00       SLM
    225  BHP-008   76.00   77.00       SLM
    322  BHP-009   54.00   56.00       SLM
    323  BHP-009   56.00   58.00       SLM
    324  BHP-009   58.00   60.00       SLM
    325  BTC_101    2.00    1.00       SLM
    326  BTC_101    1.00    2.00       SLM
    327  BTC_101    2.00    2.80       SLM
    370  HFD-020   19.00   20.00       AND
    371  HFD-020   20.00   21.00       AND
    372  HFD-020   21.00   22.00       AND
    373  HFD-020   22.00   23.00       AND
    374  HFD-020   23.00   24.00       AND
    378  HFD-020   27.00   28.00       SCB
    379  HFD-020   28.00   29.00       SCB
    380  HFD-020   29.00   30.00       SCB
    387  HFD-020   35.50   36.50       SLM
    388  HFD-020   36.50   37.80       SLM
    389  HFD-020     NaN     NaN       SLM
    413  HFD-020   69.00   74.00       SLM
    414  HFD-020   74.00   79.00       SLM
    415  HFD-020   79.00   82.00       SLM
    557  HFD-022  125.00  127.00       SCG
    558  HFD-022  127.00  128.90       SCG
    559  HFD-022  128.90  129.55       SCG
    560  HFD-022  129.55  131.00       SCG
    561  HFD-022  131.00  137.50       SLM
    562  HFD-023    0.00    2.20       SSL
    563  HFD-023    2.20    3.00       AND
    564  HFD-023    3.00    4.25       AND
    584  HFD-023   23.00   23.90       SLM
    585  HFD-023   23.90   25.00       SCB
    586  HFD-023   25.00   28.20       SLM
    587  HFD-023   28.20   30.00       SLM
    588  HFD-023   30.00   31.00       SLM
    636  HFD-024  103.00  104.00       SCB
    637  HFD-024  104.00  106.00       SCB
    638  HFD-024  106.00  108.50       SCB
    639  HFD-032    0.00    1.50       SMD
    640  HFD-032    1.50    2.50       SMD
    688  HFD-032   69.50   74.50       AND
    689  HFD-032   74.50   79.50       AND
    690  HFD-032   79.50   84.50       AND
    

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
print(f)
```

                  BHID   FROM     TO LITHOLOGY
    5          ARC-046    5.0    6.0       SCQ
    6          ARC-046    NaN    7.0       SCQ
    7          ARC-046    7.0    8.0       SCQ
    8          ARC-046    8.0    NaN       SCQ
    9          ARC-046    9.0   10.0       SCQ
    12         ARC-046   14.0   13.0       SLM
    13         ARC-046    NaN    NaN       SLM
    14         ARC-046    NaN    NaN       SLM
    15         ARC-046    NaN    NaN       SLM
    16         ARC-046    NaN    NaN       SLM
    17         ARC-046    NaN    NaN       SLM
    18         ARC-046   22.0   23.0       SLM
    152        BHD-003   57.6   58.8       BXA
    153        BHD-003    NaN    NaN       BXA
    154        BHD-003    NaN    NaN       SLM
    155        BHD-003    NaN    NaN       SLM
    156        BHD-003    NaN    NaN       SLM
    157        BHD-003   65.0   67.0       SLM
    259        BHP-008  122.0  124.0       SLM
    260        BHP-008    NaN    NaN       SLM
    261        BHP-008    NaN    NaN       BFL
    262        BHP-008    NaN    NaN       BFL
    263        BHP-008    NaN    NaN       BFL
    264        BHP-008    NaN    NaN       SLM
    265        BHP-008  132.0  133.0       SLM
    266        BHP-008  133.0  134.0       BFL
    267        BHP-008    NaN  136.0       SLM
    268        BHP-008  136.0  137.0       SLM
    307        BHP-009   25.0   27.0       SLM
    308        BHP-009   27.0    NaN       SLM
    309        BHP-009   32.0   34.0       SLM
    330        BTC_102    1.0    2.0       SLM
    331        BTC_102    2.0    NaN       SLM
    332        BTC_102    3.5    4.5       SLM
    345  BTC_301_A-B-9    0.0    2.0       AND
    346        BTC3A01    NaN    1.0       SLM
    347        BTC3A01    1.0    2.0       SLM
    356        HFD-020    4.5    5.5       SLM
    357        HFD-020    NaN    6.5       SLM
    358        HFD-020    6.5    7.5       SLM
    364        HFD-020   12.5   13.5       SLM
    365        HFD-020   13.5    NaN       SLM
    366        HFD-020   14.5   15.5       SLM
    368        HFD-020   17.2   18.0       AND
    369        HFD-020   18.0    NaN       AND
    370        HFD-020   19.0   20.0       AND
    375        HFD-020   24.0   25.0       AND
    376        HFD-020    NaN   26.0       AND
    377        HFD-020   26.0   27.0       SCB
    384        HFD-020   32.5   33.5       SLM
    385        HFD-020   33.5    NaN       SLM
    386        HFD-020   34.5   35.5       SLM
    388        HFD-020   36.5   37.8       SLM
    389        HFD-020    NaN    NaN       SLM
    390        HFD-020   40.8   42.5       SLM
    

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
print(display_fromto_na(f1))
```

           BHID   FROM     TO LITHOLOGY
    5   ARC-046   14.0   13.0       SLM
    6   ARC-046   13.0    NaN       SLM
    7   ARC-046    NaN    NaN       SLM
    8   ARC-046    NaN    NaN       SLM
    9   ARC-046    NaN    NaN       SLM
    10  ARC-046    NaN   22.0       SLM
    11  ARC-046   22.0   23.0       SLM
    12  BHD-003   57.6   58.8       BXA
    13  BHD-003   58.8    NaN       BXA
    14  BHD-003    NaN    NaN       SLM
    15  BHD-003    NaN    NaN       SLM
    16  BHD-003    NaN   65.0       SLM
    17  BHD-003   65.0   67.0       SLM
    18  BHP-008  122.0  124.0       SLM
    19  BHP-008  124.0    NaN       SLM
    20  BHP-008    NaN    NaN       BFL
    21  BHP-008    NaN    NaN       BFL
    22  BHP-008    NaN    NaN       BFL
    23  BHP-008    NaN  132.0       SLM
    24  BHP-008  132.0  133.0       SLM
    


```python
f2 = patch_fromto_na(f, shrink=True, infer=False)
print(display_fromto_na(f2))
```

           BHID   FROM     TO LITHOLOGY
    5   ARC-046   14.0   13.0       SLM
    6   ARC-046   13.0    NaN       SLM
    7   ARC-046    NaN   22.0       SLM
    8   ARC-046   22.0   23.0       SLM
    9   BHD-003   57.6   58.8       BXA
    10  BHD-003   58.8    NaN       BXA
    11  BHD-003    NaN   65.0       SLM
    12  BHD-003   65.0   67.0       SLM
    13  BHP-008  122.0  124.0       SLM
    14  BHP-008  124.0    NaN       SLM
    15  BHP-008    NaN    NaN       BFL
    16  BHP-008    NaN  132.0       SLM
    17  BHP-008  132.0  133.0       SLM
    


```python
f3 = patch_fromto_na(f, shrink=True, infer=True)
print(display_fromto_na(f3))
```

           BHID   FROM     TO LITHOLOGY
    13  BHP-008  122.0  124.0       SLM
    14  BHP-008  124.0    NaN       SLM
    15  BHP-008    NaN    NaN       BFL
    16  BHP-008    NaN  132.0       SLM
    17  BHP-008  132.0  133.0       SLM
    

Let's apply this function to our 'lith' data.


```python
lith = patch_fromto_na(lith, shrink=True, infer=True)
```


```python
print(display_fromto_na(lith))
```

            BHID   FROM     TO LITHOLOGY
    254  BHP-008  122.0  124.0       SLM
    255  BHP-008  124.0    NaN       SLM
    256  BHP-008    NaN    NaN       BFL
    257  BHP-008    NaN  132.0       SLM
    258  BHP-008  132.0  133.0       SLM
    

The remaining unfilled data need to be fill manually since there's no clue about the extent. It need to be check on the core box. Let's assume we've checked the interval and let's fill out the data based on our measurement.


```python
lith.loc[255, 'TO'] = 126
lith.loc[256, 'TO'] = 130
lith = patch_fromto_na(lith)
```


```python
print(display_fromto_na(lith))
```

    There's no missing FROM-TO found. (All data entries already have FROM-TO value.)
    

## Filter Interval data based on Collar Data to be used

All interval data need to have collar and survey data, so the data can be used. here it is the code to filter out the data availability based on collar data.


```python
def interval_incollar(interval_data,collar_data):
    """Function to filter interval data based on available collar data"""
    return interval_data[interval_data['BHID'].isin(collar_data['BHID'])].reset_index(drop=True)
```


```python
print(interval_incollar(lith, collar))
```

              BHID  FROM    TO LITHOLOGY
    0      ARC-046   0.0   1.0       SCQ
    1      ARC-046   1.0   2.0       SCQ
    2      ARC-046   1.0   3.0       SCQ
    3      ARC-046   3.0   4.0       SCQ
    4      ARC-046   4.0   5.0       SCQ
    ...        ...   ...   ...       ...
    12755  YRC-287  25.0  26.0       AND
    12756  YRC-287  26.0  27.0       AND
    12757  YRC-287  27.0  28.0       AND
    12758  YRC-287  28.0  29.0       AND
    12759  YRC-287  29.0  30.0       AND
    
    [12760 rows x 4 columns]
    

Here it is the code to make sure data integrity. First code is to check whether there's interval data that is not in collar. Second code is to check whether there's collar data that doesn't have interval data.


```python
def interval_notincollar(interval_data,collar_data):
    """Function to filter interval data based on available collar data"""
    return interval_data[~interval_data['BHID'].isin(collar_data['BHID'])].reset_index(drop=True)
```


```python
print(interval_notincollar(lith, collar))
```

              BHID   FROM     TO LITHOLOGY
    0      HFD-068   0.00  39.20       SVE
    1      HFD-068  39.20  50.60       SVE
    2      HFD-068  50.60  57.25       SLM
    3      HFD-068  57.25  63.00       SCB
    4      HFD-068  63.00  68.00       SCB
    ...        ...    ...    ...       ...
    3498  ZKY_8A02  38.40  51.70       CLY
    3499  ZKY_8A02  51.70  52.00      CLOS
    3500  ZKY_8A02  52.00  53.90     CLYST
    3501  ZKY_8A02  53.90  54.40      CLOS
    3502  ZKY_8A02  54.40  54.75      DALT
    
    [3503 rows x 4 columns]
    


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
print(collar_withnointerval(collar,lith))
```

    All entries in collar data have interval data
    

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
print(ih)
```

          BHID  FROM   TO LITHOLOGY
    1  ARC-046   1.0  2.0       SCQ
    2  ARC-046   1.0  3.0       SCQ
    


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
print(fix_fltpt(ih))
```

          BHID  FROM   TO LITHOLOGY
    0  ARC-046   1.0  2.0       SCQ
    1  ARC-046   2.0  3.0       SCQ
    


```python
lith = fix_fltpt(lith)
```


```python
print(display_fltpt(lith))
```

    There's no FROM which less than prevoius than TO
    

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
print(fg)
```

            BHID  FROM    TO LITHOLOGY
    12   ARC-046  14.0  13.0       SLM
    121  BHD-003  27.0  27.0       SLM
    124  BHD-003  31.0  30.0       SLM
    318  BTC_101   2.0   1.0       SLM
    


```python
fq = display_fgtet(lith, buffer=True)
print(fq)
```

            BHID  FROM    TO LITHOLOGY
    11   ARC-046  11.0  12.0       SLM
    12   ARC-046  14.0  13.0       SLM
    13   ARC-046  13.0  17.5       SLM
    120  BHD-003  25.0  26.0       SLM
    121  BHD-003  27.0  27.0       SLM
    122  BHD-003  27.0  28.0       SCG
    123  BHD-003  28.0  29.0       SLM
    124  BHD-003  31.0  30.0       SLM
    125  BHD-003  30.0  31.0       SLM
    317  BHP-009  58.0  60.0       SLM
    318  BTC_101   2.0   1.0       SLM
    319  BTC_101   1.0   2.0       SLM
    


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
print(fix_fgtet(fq))
```

           BHID  FROM    TO LITHOLOGY
    0   ARC-046  11.0  12.0       SLM
    1   ARC-046  12.0  13.0       SLM
    2   ARC-046  13.0  17.5       SLM
    3   BHD-003  25.0  26.0       SLM
    4   BHD-003  26.0  27.0       SLM
    5   BHD-003  27.0  28.0       SCG
    6   BHD-003  28.0  29.0       SLM
    7   BHD-003  29.0  30.0       SLM
    8   BHD-003  30.0  31.0       SLM
    9   BHP-009  58.0  60.0       SLM
    10  BTC_101   0.0   1.0       SLM
    11  BTC_101   1.0   2.0       SLM
    


```python
lith = fix_fgtet(lith)
```


```python
print(display_fgtet(lith))
```

    There's no FROM which greater than or equal to TO.
    

## Checking missing Lithology entries


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
print(display_lith_na(lith, buffer=True))
```

            BHID  FROM    TO LITHOLOGY
    386  HFD-020  44.5  45.4       SMD
    387  HFD-020  45.4  48.0       NaN
    388  HFD-020  48.0  49.0       SMD
    

Let's assume the NaN lithology is same as previous and following LITHOLOGY. So, let's change NaN at row 387 into SMD.


```python
lith.loc[387, 'LITHOLOGY'] = 'SMD'
```

## Checking inappropriate entries of Interval Data


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
lith_list = ['SCQ', 'SLM', 'SOIL', 'BXA', 'CLOS', 'SCG', 'AND', 'BFL', 
             'SMD', 'DALT', 'CLY', 'SCB', 'SSL', 'SST', 'SVE', 'STR',
             'SCF', 'BSH', 'NCR', 'CAV', 'CALV', 'LMS', 'SCL']
```


```python
print(display_unmatched(lith, checklist=lith_list))
```

            BHID  FROM    TO LITHOLOGY
    189  BHP-008  34.0  36.0   Andesit
    191  BHP-008  37.0  38.0   Andesit
    282  BHP-009   0.0   2.0     TANAH
    


```python
lith = lith.replace({'Andesit':'AND', 'TANAH':'SOIL'})
```


```python
print(display_unmatched(lith,checklist=lith_list))
```

    There's no unmatched entries
    

### Exporting data 


```python
lith.to_csv('LithologyDataFix.csv')
collar.to_csv('CollarDataFix.csv')
survey.to_csv('SurveyDataFix.csv')
```
