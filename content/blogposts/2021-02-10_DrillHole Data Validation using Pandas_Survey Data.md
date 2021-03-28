+++
authors = ["nazarabrory"]
categories = ["Data validation"]
date = 2021-01-10T17:00:00Z
draft = false
tags = ["Survey","Validation", "Duplicate", "Null","Azimuth", "Dip", "Deviation" "Pandas"]
title = "DrillHole Data Validation using Pandas: Survey Data"
toc = false
[cover]
alternate = ""
caption = ""
image = ""
style = ""

+++


Drillhole Data consist of at least three Data (Collar, Survey, and Interval Data). All of this data arranged in table form with specified columns connected with HOLE ID as the index key. Survey Data is a representation of distance and direction on 3D space for each Drillhole. So, Survey data need to have at least these columns: HOLE_ID, Meterage of Measurement along the drillhole path, Measured Azimuth, and Measured Dip.

Basically, these are things to consider when dealing with data validation for Survey Data:

1. Null Data (Unfilled Data Entry)
2. Data availability based on collar's 'hole_id'
3. Surveys beyond total depth
4. Azimuth & Dip Deviation



```python
# Import pandas library for data handling
import pandas as pd
import numpy as np
```


```python
# Import collar and survey file. Set 'hole_id' to be the index for each dataset. 
survey = pd.read_csv('Survey.csv', index_col='hole_id')
collar = pd.read_csv('CollarDataFix.csv', index_col='hole_id') # The collar data have been validated before.
```


```python
print('The collar data contain ' + str(len(collar)) + ' data entries.Here it is the table format for collar data:')
print(collar.head(5))
```

    The collar data contain 469 data entries.Here it is the table format for collar data:
               max_depth       X_Dum      Y_Dum    Z_Dum
    hole_id                                             
    ARC-046        100.0  173431.704  47070.088  583.189
    BHD-003        100.0  173113.656  46894.455  597.301
    BHP-008         76.0  172974.924  47005.830  580.689
    BHP-008-A      190.0  172978.697  47000.926  580.581
    BHP-009         60.0  173019.415  46907.174  550.761
    


```python
print('The survey data contain ' + str(len(survey)) + ' data entries. Here it is the table format for survey data:')
print(survey.head(5))
```

    The survey data contain 517 data entries. Here it is the table format for survey data:
               depth  azimuth   dip
    hole_id                        
    ARC-046      0.0      0.0 -90.0
    BHD-003      0.0    180.0 -55.0
    BHD-003    100.0    179.0 -56.0
    BHP-008      0.0     90.0 -65.0
    BHP-008-A    0.0     90.0 -65.0
    

As we can see, we survey data have 517 data entries while collar data have 469 data entries. There's 48 difference, this indicate there's some survey data which have more than one survey data measurement for each drill hole or maybe there's false data such as mistyped data, survey data without collar data availability, duplicate data, null values, etc. Let's find out and validate this survey data.

## Null values check


```python
# Checking whether there's Null data in the survey table.
print(survey.isnull().any())
```

    depth      False
    azimuth    False
    dip        False
    dtype: bool
    

There's no Null values in survey data (All data are well filled).

## Data Availability based on Collar data 

Survey data availability need to be checked  based on Collar data availability which correspond to its hole_id. First, we need to check unique hole_id number for each dataset. 


```python
print("The collar data's hole_id entries : " + str(len(collar.index.unique())))
print("The survey data hole_id entries : " + str(len(survey.index.unique())))

```

    The collar data's hole_id entries : 469
    The survey data hole_id entries : 490
    

There is different number of entry between collar and survey data. Collar have 469 data entry meanwhile survey have 490 data entry. So there is 21 hole_id in survey datasetis not available on collar dataset. Let's see those data, and remove it from survey data ( since it cannot be used wihtout collar data. )


```python
# Displaying 'hole_id' in Survey data as well as its number of measurement on each hole_id
survey_sum = survey.groupby(by='hole_id').count()
# Merging data between collar and survey_sum data which available on both dataset. 
collar_survey = collar.merge(survey_sum, how='inner', left_index=True, right_index=True)

print('Here it is the merged dataset:')
print(collar_survey)
```

    Here it is the merged dataset:
               max_depth       X_Dum      Y_Dum    Z_Dum  depth  azimuth  dip
    hole_id                                                                  
    ARC-046       100.00  173431.704  47070.088  583.189      1        1    1
    BHD-003       100.00  173113.656  46894.455  597.301      2        2    2
    BHP-008        76.00  172974.924  47005.830  580.689      1        1    1
    BHP-008-A     190.00  172978.697  47000.926  580.581      1        1    1
    BHP-009        60.00  173019.415  46907.174  550.761      1        1    1
    ...              ...         ...        ...      ...    ...      ...  ...
    ZKY_6A01      136.70  172910.100  46772.546  539.579      1        1    1
    ZKY_6A03      126.60  172927.059  46736.341  530.130      1        1    1
    ZKY_804       163.15  172972.829  46779.813  535.919      1        1    1
    ZKY_8A01      155.00  172946.266  46789.551  540.705      1        1    1
    ZKY_8A02       54.75  172929.384  46825.806  552.474      1        1    1
    
    [468 rows x 7 columns]
    

We've already have unique 'hole_id' entries which available on both dataset. Let's update our dataset based on these hole_id.


```python
# Updating dataset
c_s_i = collar_survey.index
collar = collar.loc[c_s_i]
survey = survey.loc[c_s_i]
```

Here it is the updated datasets for both collar data and survey data.


```python
collar.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 468 entries, ARC-046 to ZKY_8A02
    Data columns (total 4 columns):
     #   Column     Non-Null Count  Dtype  
    ---  ------     --------------  -----  
     0   max_depth  468 non-null    float64
     1   X_Dum      468 non-null    float64
     2   Y_Dum      468 non-null    float64
     3   Z_Dum      468 non-null    float64
    dtypes: float64(4)
    memory usage: 18.3+ KB
    


```python
survey.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 495 entries, ARC-046 to ZKY_8A02
    Data columns (total 3 columns):
     #   Column   Non-Null Count  Dtype  
    ---  ------   --------------  -----  
     0   depth    495 non-null    float64
     1   azimuth  495 non-null    float64
     2   dip      495 non-null    float64
    dtypes: float64(3)
    memory usage: 15.5+ KB
    

## Survey data at surface availability and Survey data beyond total depth

Survey data are measured at surface and on a certain interval in the path of drillhole. If there's no survey data measurement at surface, we cannot use that hole_id. Also, we need to account for data measured beyond the total depth, because it is impossible to happen (it might be an error on measurement or data input).


```python
survey_atsurface = survey.groupby(by='hole_id')['depth'].min() == 0
survey_atsurface.describe()
```




    count      468
    unique       2
    top       True
    freq       387
    Name: depth, dtype: object



As we can see on summary data above, there's 387 out of 468 hole_id have survey data at surface. Again, survey data at surface need to be available. So, we need to recheck the data to make correction or drop the data. Let's assume we decide to drop the data instead.


```python
# Dropping data with no surface survey data
survey = survey.loc[survey_atsurface]
collar = collar.loc[survey_atsurface]
```


```python
# Checking max depth from collar table vs survey table
survey_maxdepth = survey.groupby(by='hole_id')['depth'].max()
collar_maxdepth = collar['max_depth']
```


```python
scdepth = pd.concat([survey_maxdepth, collar_maxdepth], axis=1)
scdepth['S<=C'] = (scdepth['depth'] <= scdepth['max_depth'])

print('Here it is the summary:')
scdepth['S<=C'].describe()
```

    Here it is the summary:
    




    count      387
    unique       1
    top       True
    freq       387
    Name: S<=C, dtype: object



As we can see above, there's no unique survey depth which exceed  maxdepth data. If we'd found that we should make correction to the data or drop it.

### Azimuth and Dip Deviation

Azimuth and Dip data are measured along the drill path. These measured  values deviation/changes can't be too large. otherwise, the collar will be broken. 


```python
# Calculating depth, azimuth and dip difference for each hole_id

depth_diff = survey.groupby('hole_id')['depth'].diff().fillna(0)
depth_diff.name = 'depth_diff'

survey_diff = survey.groupby(by='hole_id')[['azimuth', 'dip']].rolling(2).apply(
    lambda x: 180 - abs(abs(x[0] - x[1]) - 180)).fillna(0).rename(
    columns={'azimuth':'azimuth difference', 
             'dip': 'dip difference'}).reset_index().set_index('hole_id')

# Combining colculated data into survey data
survey['depth_diff'] = depth_diff.to_list()
survey['azimuth_diff'] = survey_diff['azimuth difference'].to_list()
survey['dip_diff'] = survey_diff['dip difference'].to_list()
```


```python
# Displaying data summary
print(survey.describe().T)
```

                  count       mean         std   min   25%   50%   75%    max
    depth         414.0   7.645894   33.720484   0.0   0.0   0.0   0.0  300.0
    azimuth       414.0  69.826087  124.399828   0.0   0.0   0.0  90.0  360.0
    dip           414.0 -81.612319   13.530123 -90.0 -90.0 -90.0 -70.0  -47.0
    depth_diff    414.0   5.135386   22.204506   0.0   0.0   0.0   0.0  201.0
    azimuth_diff  414.0   0.413043    3.958584   0.0   0.0   0.0   0.0   65.0
    dip_diff      414.0   0.061594    0.323343   0.0   0.0   0.0   0.0    3.0
    

As we can see, the max value of dip_diff is quite tolerable but isn't true for azimuth_diff.Drillhole's deviation can't be too large otherwise the collar would broke. If there's these kind of data, it might because of mistyped data, or inaccurate survey measurement. Let's see the data which have large azimuth deviations with an assumetion that the drill deviation threshold is 5 degree.


```python
# Displaying data with suspicious azimuth and dip deviation
survey_invalid = survey.loc[survey[survey['azimuth_diff'] > 5].index.unique()]
print(survey_invalid)
```

             depth  azimuth   dip  depth_diff  azimuth_diff  dip_diff
    hole_id                                                          
    HFD-068    0.0    360.0 -90.0         0.0           0.0       0.0
    HFD-068  167.0    330.0 -89.0       167.0          30.0       1.0
    LHD-034    0.0    360.0 -70.0         0.0           0.0       0.0
    LHD-034   80.0    350.0 -69.0        80.0          10.0       1.0
    LHD-034  185.0    353.0 -71.0       105.0           3.0       2.0
    YRC-276    0.0      0.0 -90.0         0.0           0.0       0.0
    YRC-276   50.0     65.0 -88.0        50.0          65.0       2.0
    YRC-276  209.0    100.0 -87.5       159.0          35.0       0.5
    

We can recheck the data and correct it. Let's assume these data has invalid survey measurement, so we need to drop these data form our project.


```python
survey = survey.drop(survey_invalid.index)
collar = collar.drop(survey_invalid.index)
```

survey.info()

Here it is the collar and survey data that have been validated. Finally, let's save the data into '.csv' file.


```python
survey.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 406 entries, ARC-046 to YRC-287
    Data columns (total 6 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   depth         406 non-null    float64
     1   azimuth       406 non-null    float64
     2   dip           406 non-null    float64
     3   depth_diff    406 non-null    float64
     4   azimuth_diff  406 non-null    float64
     5   dip_diff      406 non-null    float64
    dtypes: float64(6)
    memory usage: 22.2+ KB
    


```python
collar.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 384 entries, ARC-046 to YRC-287
    Data columns (total 4 columns):
     #   Column     Non-Null Count  Dtype  
    ---  ------     --------------  -----  
     0   max_depth  384 non-null    float64
     1   X_Dum      384 non-null    float64
     2   Y_Dum      384 non-null    float64
     3   Z_Dum      384 non-null    float64
    dtypes: float64(4)
    memory usage: 15.0+ KB
    


```python
# export data to csv
survey.to_csv('SurveyDataFix.csv')
collar.to_csv('CollarDataFix.csv')
```
