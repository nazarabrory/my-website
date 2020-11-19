+++
authors = ["nazarabrory"]
categories = ["Geostatistics"]
date = 2020-10-17T17:00:00Z
tags = ["Declustering", "GSLIB", "Python"]
title = "Spatial Declustering using GSLIB in Python."
toc = false
[cover]
alternate = ""
caption = ""
image = ""
style = "wide"

+++

Python is a powerfull multipurpose programming language that can help us handle the statistical analysis of spatial data. By using python and jupyter notebook we can make a workflow that can be implemented on any problem set. Bunch of Packages  have been written in Python by the community. One of the geostatistical packages that I think really good is pygeostat, because it can integrate GSLIB (Geostatistical Software Library) into the workflow. In this post, I would like to perform a cell declustering analysis using pygeostat and GSLIB in python.


```python
# import modules packages to python
import pygeostat as gs
```


```python
data = gs.DataFile('datafile.csv')             # import data file  and assign to a variable named `data` 
data.setvarcols(variables=['Primary'])         # Specify variables  
data.setcol('cat', 'Batch')                    # Set Batch column as categorical data
```


```python
data                                           # View assigned data atributes             
```




    DataFile: datafile.csv
    Attributes:
    x: 'Xlocation',  y: 'Ylocation',  cat: 'Batch',  
    Variables:
    Primary




```python
data.data                                       # View the data table
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
      <th>Batch</th>
      <th>Xlocation</th>
      <th>Ylocation</th>
      <th>Primary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>39.5</td>
      <td>18.5</td>
      <td>0.06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>5.5</td>
      <td>1.5</td>
      <td>0.06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>38.5</td>
      <td>5.5</td>
      <td>0.08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>20.5</td>
      <td>1.5</td>
      <td>0.09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.0</td>
      <td>27.5</td>
      <td>14.5</td>
      <td>0.09</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>2.0</td>
      <td>31.5</td>
      <td>41.5</td>
      <td>22.75</td>
    </tr>
    <tr>
      <th>136</th>
      <td>2.0</td>
      <td>34.5</td>
      <td>32.5</td>
      <td>9.42</td>
    </tr>
    <tr>
      <th>137</th>
      <td>2.0</td>
      <td>35.5</td>
      <td>31.5</td>
      <td>8.48</td>
    </tr>
    <tr>
      <th>138</th>
      <td>2.0</td>
      <td>35.5</td>
      <td>33.5</td>
      <td>2.82</td>
    </tr>
    <tr>
      <th>139</th>
      <td>2.0</td>
      <td>36.5</td>
      <td>32.5</td>
      <td>5.26</td>
    </tr>
  </tbody>
</table>
<p>140 rows × 4 columns</p>
</div>




```python
data.describe()                                  # View data summary statistic
```




    count    140.000000
    mean       4.350429
    std        6.726625
    min        0.060000
    25%        0.700000
    50%        2.195000
    75%        5.327500
    max       58.320000
    Name: Primary, dtype: float64




```python
# Make a location plot.
fig, ax = gs.subplots(nrows=1, ncols=2, cbar_mode='each', figsize=(18,18))           # Defining subplots configuration
_= gs.location_plot(data=data, var='Batch', ax= ax[0])                               # Location Plot by batch
_= gs.location_plot(data=data, var='Primary', ax = ax[1])                            # Locatioin Plot by Primary values
```


    
![png](/uploads/blogposts/output_7_0.png)
    


As we can see in the location plot above, there are 2 data batch. The first batch collect the data sparsely while the second batch data were collected to target high values near the prior data. The clustered data like this,  will make spatial data statistics tends to be biased. Let's check the significant changes of the data statistics bias by inspecting histogram.


```python
# Plotting the histogram of Primary data variables foor each batch
_ = gs.histogram_plot(data=data, var='Primary',figsize=(12, 3), title='Primary values histogram (All Batch)')
_ = gs.histogram_plot(data=data[data['Batch'] == 1], var='Primary',figsize=(12, 3),title='Primary values histogram (Batch 1)')
_ = gs.histogram_plot(data=data[data['Batch'] == 2], var='Primary',figsize=(12, 3), title='Primary values histogram (Batch 2)')
```


    
![png](/uploads/blogposts/output_9_0.png)
    



    
![png](/uploads/blogposts/output_9_1.png)
    



    
![png](/uploads/blogposts/output_9_2.png)
    


As we can see from the histogram, data from Batch 2 have a higher mean. When it is combined with data form Batch 1, the mean form the randomly sparse data, which is more reliable, are shifted. So, clustered data like this example, will tend to have a bias statistical summary, over-estimate or underestimate. 

To account for this bias, we need to do a declustering routine to the data. Declustering is simply to assign weight to every data, so the statistical summary will be weighted. Declus.exe from GSLIB (Geostatistical Software Library) is a program to calculate Declustering weight. Here, I would like to perform Data Declustering using GSLIB program with help of pygeostat module to wrap the program, so it can be integrated to the workflow.


```python
# Before running GSLIB program, the data need to be converted into GSLIB format
gs.write_gslib(data, flname='datafile.dat')
```


```python
# Importing the program and get the parameter file 
declus_p = gs.Program(program='declus.exe', getpar=True )
```

    C:\Python_Projects\Declustering\tmp4qjr9yn1\declus.par has been copied to the clipboard
    

By running the code above, the parameter string format automatically copied to the clipboard. Next, paste it into the next block.


```python
parstr = """      Parameters for DECLUS
                  *********************

START OF PARAMETERS:
datafile.dat                -file with data
2   3   0   4               -  columns for X, Y, Z, and variable
-1.0e21     1.0e21          -  trimming limits
declus.sum                  -file for summary output
declus.out                  -file for output with data & weights
1   1                       -Y and Z cell anisotropy (Ysize=size*Yanis)
0                           -0=look for minimum declustered mean (1=max)
24  1.0  25.0               -number of cell sizes, min size, max size
5                           -number of origin offsets
"""
```


```python
# Run the program and use the parameter specified above.
declus_p.run(parstr=parstr, liveoutput=True)
```

    Calling:  ['declus.exe', 'temp']
    
    DECLUS Version: 2.906
    
     data file = datafile.dat                            
     columns =           2           3           0           4
     tmin,tmax =   -1.000000E+21    1.000000E+21
     summary file = declus.sum                              
     output file = declus.out                              
     anisotropy =        1.000000        1.000000
     minmax flag =           0
     ncell min max =          24        1.000000       25.000000
     offsets =           5
    
    
    There are      140 data with:
      mean value            =      4.35043
      minimum and maximum   =       .06000    58.32000
      size of data vol in X =     48.00000
      size of data vol in Y =     48.00000
      size of data vol in Z =       .00000
    
      declustered mean      =      2.52389
      min and max weight    =       .28779     1.79745
      equal weighting       =      1.00000
    
    
    DECLUS Version: 2.906 Finished
    
    Stop - Program terminated.
    
    

As we can see abve, declustering process progress written neatly. Let's open up output file and check the result.


```python
declus_sum = gs.DataFile('declus.sum')                    # Importing declus.sum (summary of declustering process)             
```


```python
# Plot the declus.sum, so we can inspect the lowest mean.
_ = gs.scatter_plot(x=declus_sum['Cell Size'], y=declus_sum['Declustered Mean'], 
                    title='Declustered mean for each cell size', 
                    c=declus_sum['Declustered Mean'], 
                    stat_blk=False, ylim=(2,5), xlim=(0,26))
```


    
![png](/uploads/blogposts/output_19_0.png)
    


As we can see the lowest Declustered mean is around 2.5. Next, we are going to import the resulted data table and calculate histogram and its statistical summary.


```python
declus_out = gs.DataFile('declus.out')               # Importing Declustering Output
declus_out.data                                      # View the data
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
      <th>Batch</th>
      <th>Xlocation</th>
      <th>Ylocation</th>
      <th>Primary</th>
      <th>Declustering Weight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>39.5</td>
      <td>18.5</td>
      <td>0.06</td>
      <td>1.281301</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>5.5</td>
      <td>1.5</td>
      <td>0.06</td>
      <td>1.400103</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>38.5</td>
      <td>5.5</td>
      <td>0.08</td>
      <td>1.613235</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>20.5</td>
      <td>1.5</td>
      <td>0.09</td>
      <td>1.797446</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.0</td>
      <td>27.5</td>
      <td>14.5</td>
      <td>0.09</td>
      <td>1.430162</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>2.0</td>
      <td>31.5</td>
      <td>41.5</td>
      <td>22.75</td>
      <td>0.367498</td>
    </tr>
    <tr>
      <th>136</th>
      <td>2.0</td>
      <td>34.5</td>
      <td>32.5</td>
      <td>9.42</td>
      <td>0.417419</td>
    </tr>
    <tr>
      <th>137</th>
      <td>2.0</td>
      <td>35.5</td>
      <td>31.5</td>
      <td>8.48</td>
      <td>0.432717</td>
    </tr>
    <tr>
      <th>138</th>
      <td>2.0</td>
      <td>35.5</td>
      <td>33.5</td>
      <td>2.82</td>
      <td>0.336913</td>
    </tr>
    <tr>
      <th>139</th>
      <td>2.0</td>
      <td>36.5</td>
      <td>32.5</td>
      <td>5.26</td>
      <td>0.287790</td>
    </tr>
  </tbody>
</table>
<p>140 rows × 5 columns</p>
</div>



Note that Declustering Weight have been assigned to the table. So, we can use this weight to calculate unbiased summary statistics.


```python
# Plotting the histogram of Primary data variables foor each batch
_ = gs.histogram_plot(data=data, var='Primary',figsize=(12, 3), title='Primary values histogram (All Batch)')
_ = gs.histogram_plot(data=data[data['Batch'] == 1], var='Primary',figsize=(12, 3),title='Primary values histogram (Batch 1)')
_ = gs.histogram_plot(data=data[data['Batch'] == 2], var='Primary',figsize=(12, 3), title='Primary values histogram (Batch 2)')

_ = gs.histogram_plot(data=declus_out, var='Primary', weights='Declustering Weight', color='lightblue' ,figsize=(12, 3), title='Primary values histogram (All Batch, Weighted)')
```


    
![png](/uploads/blogposts/output_23_0.png)
    



    
![png](/uploads/blogposts/output_23_1.png)
    



    
![png](/uploads/blogposts/output_23_2.png)
    



    
![png](/uploads/blogposts/output_23_3.png)
    


So, the unbiased declustered mean is 2.52, Note that the mean of raw data is 4.35 which is bias, overestimate. 

#### Closing Remark
Almost every spatial data is biased, declustering process need to be accounted before another analysis or the decision made. By using GSLIB in python, we can build the workflow that suits of our project scale. 

#### Reference 

    Deutsch, C. V., &amp; Journel, A. G. (1998). GSLIB: Geostatistical software library and user's guide. New York: Oxford University Press.
    
    Pygeostat Documentation. (n.d.). Retrieved November 19, 2020, from http://www.ccgalberta.com/pygeostat/welcome.html
