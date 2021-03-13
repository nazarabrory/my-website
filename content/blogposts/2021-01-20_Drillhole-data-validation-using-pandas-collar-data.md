+++
authors = ["nazarabrory"]
categories = ["Data validation"]
date = 2021-01-10T17:00:00Z
draft = false
tags = ["Collar","Validation", "Duplicate", "Null", "Pandas"]
title = "DrillHole Data Validation using Pandas: Collar Data"
toc = false
[cover]
alternate = ""
caption = ""
image = ""
style = ""

+++


Drillhole Data consist of at least three Data (Collar, Survey, and Interval Data). All of this data arranged in table form with specified columns connected with HOLE ID as the index key. Collar Data is a point representation on 3D space for each Drillhole, so, Collar data need to have at least these columns they are HOLE_ID, X location, Y location, and Z location.

This post will focus on performing data validation for Collar Data. There are two main things to consider:

1. Data Duplicate (HOLE_ID mentioned more than one)
2. Null Data (Unfilled Data Entry)

Let's perform this validation using pandas and define it as a function. Here is the code:


```python
# Importing Pandas Package
import pandas as pd
```


```python
# Here is the list of function for collar data validation:
# Code for counting unfilled data entry
def blank_count(collar_data):
    return collar_data[collar_data.isna().any(axis=1)].isna().sum()
# Code for displaying unfilled data 
def blank_data(collar_data):
    return collar_data[collar_data.isna().any(axis=1)]
# Code for counting duplicated Hole ID
def duplicate_count(collar_data):
    return collar_data[collar_data.duplicated(subset='hole_id', keep=False)].groupby(by='hole_id').size()
# COde for displaying duplicated data
def duplicate_data(collar_data):
    return collar_data[collar_data.duplicated(subset='hole_id', keep=False)]
```

Let's perform these code with Collar Data.


```python
# Importing Collar data
collar = pd.read_csv('Collar.csv')
print(collar)
```

          hole_id  max_depth       X_Dum      Y_Dum    Z_Dum
    0     ARC-046     100.00  173431.704  47070.088  583.189
    1     ARC-O46     100.00  173431.704  47070.088  583.189
    2     BHD-003     100.00  173113.656  46894.455  597.301
    3     BHP-008      76.00  172974.924  47005.830  580.689
    4     BHP-008      76.00  172974.924  47005.830  580.689
    ..        ...        ...         ...        ...      ...
    492  ZKY_6A02     100.00  172939.181  46718.258      NaN
    493  ZKY_6A03     126.60  172927.059  46736.341  530.130
    494   ZKY_804     163.15  172972.829  46779.813  535.919
    495  ZKY_8A01     155.00  172946.266  46789.551  540.705
    496  ZKY_8A02      54.75  172929.384  46825.806  552.474
    
    [497 rows x 5 columns]
    


```python
# Displaying Collar data configuration
print(collar.info())
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 497 entries, 0 to 496
    Data columns (total 5 columns):
     #   Column     Non-Null Count  Dtype  
    ---  ------     --------------  -----  
     0   hole_id    491 non-null    object 
     1   max_depth  495 non-null    float64
     2   X_Dum      493 non-null    float64
     3   Y_Dum      491 non-null    float64
     4   Z_Dum      492 non-null    float64
    dtypes: float64(4), object(1)
    memory usage: 19.5+ KB
    None
    

As we can see, the collar data consist of 497rows and 5columns. But, each column not showing the same number of rows filled (as shown from 'collar.info()'). Let's find out what exactly the problem is by running python functions that we have defined before.


```python
print(' Here it is the number of NaN values for each column: ')
blank_count(collar)
```

     Here it is the number of NaN values for each column: 
    
    hole_id      6
    max_depth    2
    X_Dum        4
    Y_Dum        6
    Z_Dum        5
    dtype: int64




```python
print('Here it is the list of all rows containing Null/NaN values: ')
print(blank_data(collar))
```

    Here it is the list of all rows containing Null/NaN values: 
                 hole_id  max_depth       X_Dum      Y_Dum    Z_Dum
    79           LHD-022       39.6  172237.858        NaN  596.466
    163  TCR_10A01_C-D-2        2.0         NaN  46461.490  613.481
    289          YRC-171       54.0  172080.494        NaN      NaN
    295              NaN       24.0         NaN  46424.247  586.379
    302          YRC-183        NaN  171980.458        NaN  630.992
    343          YRC-224       51.0         NaN  46556.562  601.378
    358          YRC-239       51.0  171960.228        NaN  633.460
    362              NaN       48.0  171845.916  46422.987  601.624
    382          YRC-265       40.0  171989.313  46477.457      NaN
    390              NaN      150.0  172119.147  46059.678  544.726
    403          YRC-285       40.0         NaN  46523.548  635.082
    421              NaN       29.1  171729.979  46466.611  647.114
    422          ZKC 201       60.5  171811.544  46507.197      NaN
    423          ZKC 202      119.0  171802.935        NaN  643.579
    429         ZKC 2A03        NaN  171784.969  46537.062  647.629
    451              NaN       14.5  171804.980  46586.899  662.381
    454          ZKY 001      125.1  172819.457  46730.173      NaN
    477         ZKY_1001       35.4  172999.758        NaN  539.752
    490              NaN      141.6  172890.838  46719.308  532.918
    492         ZKY_6A02      100.0  172939.181  46718.258      NaN
    


```python
print('Here it is the number duplicated hole_id:')
print(duplicate_count(collar))
```

    Here it is the number duplicated hole_id:
    hole_id
    BHP-008          2
    BTC_301_A-B-5    2
    HFD-023          2
    LHD-001          2
    LHD-037          2
    ZKC 202          2
    ZKC 301          2
    ZKC_6A01         2
    dtype: int64
    


```python
print('Here it is the data which have duplicates:')
print(duplicate_data(collar))
```

    Here it is the data which have duplicates:
               hole_id  max_depth       X_Dum      Y_Dum    Z_Dum
    3          BHP-008      76.00  172974.924  47005.830  580.689
    4          BHP-008      76.00  172974.924  47005.830  580.689
    15   BTC_301_A-B-5       2.00  171707.390  46461.860  648.864
    16   BTC_301_A-B-5       2.00  171707.390  46461.860  648.864
    25         HFD-023      46.30  172350.246  46645.453  624.827
    26         HFD-023      46.30  172350.246  46645.453  624.827
    57         LHD-001      98.00  172152.255  46558.831  593.214
    58         LHD-001      98.00  172152.255  46558.831  593.214
    95         LHD-037      56.20  172420.803  46553.502  586.616
    107        LHD-037      56.20  172420.803  46553.502  586.616
    295            NaN      24.00         NaN  46424.247  586.379
    362            NaN      48.00  171845.916  46422.987  601.624
    390            NaN     150.00  172119.147  46059.678  544.726
    421            NaN      29.10  171729.979  46466.611  647.114
    423        ZKC 202     119.00  171802.935        NaN  643.579
    424        ZKC 202      95.10  171794.712  46561.361  651.903
    433        ZKC 301      90.90  171704.288  46476.070  652.028
    434        ZKC 301      70.30  171684.791  46513.767  666.293
    451            NaN      14.50  171804.980  46586.899  662.381
    452       ZKC_6A01      82.95  171866.022  46550.562  637.903
    471       ZKC_6A01     155.50  172918.504  46754.440  533.267
    490            NaN     141.60  172890.838  46719.308  532.918
    

Next, we can fix the data by assigning the correct data values, dropping unnecessary data, or editing mistyped data.
