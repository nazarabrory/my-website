+++
authors = ["nazarabrory"]
categories = []
date = 2021-01-10T17:00:00Z
draft = true
tags = ["Data Validation"]
title = "DrillHole Data Validation using Pandas: Collar Data"
toc = false
[cover]
alternate = ""
caption = ""
image = ""
style = ""

+++
{
 "cells": [
  {
   "source": [
    "## DrillHole Data Validation using Pandas: Collar Data"
   ],
   "cell_type": "markdown",
   "metadata": {}
  },
  {
   "source": [
    "Drillhole Data consist of at least three Data (Collar, Survey, and Interval Data). All of this data arranged in table form with specified columns connected with HOLE ID as the index key. Collar Data is a point representation on 3D space for each Drillhole, so, Collar data need to have at least these columns they are HOLE_ID, X location, Y location, and Z location.\n",
    "\n",
    "This post will focus on performing data validation for Collar Data. There are two main things to consider:\n",
    "\n",
    "1. Data Duplicate (HOLE_ID mentioned more than one)\n",
    "2. Null Data (Unfilled Data Entry)\n",
    "\n",
    "Let's perform this validation using pandas and define it as a function. Here is the code:"
   ],
   "cell_type": "markdown",
   "metadata": {}
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "id": "alpha-ozone",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Importing Pandas Package\n",
    "import pandas as pd"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "id": "proud-latter",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Here is the list of function for collar data validation:\n",
    "# Code for counting unfilled data entry\n",
    "def blank_count(collar_data):\n",
    "    return collar_data[collar_data.isna().any(axis=1)].isna().sum()\n",
    "# Code for displaying unfilled data \n",
    "def blank_data(collar_data):\n",
    "    return collar_data[collar_data.isna().any(axis=1)]\n",
    "# Code for counting duplicated Hole ID\n",
    "def duplicate_count(collar_data):\n",
    "    return collar_data[collar_data.duplicated(subset='hole_id', keep=False)].groupby(by='hole_id').size()\n",
    "# COde for displaying duplicated data\n",
    "def duplicate_data(collar_data):\n",
    "    return collar_data[collar_data.duplicated(subset='hole_id', keep=False)]"
   ]
  },
  {
   "source": [
    "Let's perform these code with Collar Data."
   ],
   "cell_type": "markdown",
   "metadata": {}
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "id": "white-pursuit",
   "metadata": {},
   "outputs": [
    {
     "output_type": "stream",
     "name": "stdout",
     "text": [
      "      hole_id  max_depth       X_Dum      Y_Dum    Z_Dum\n0     ARC-046     100.00  173431.704  47070.088  583.189\n1     ARC-O46     100.00  173431.704  47070.088  583.189\n2     BHD-003     100.00  173113.656  46894.455  597.301\n3     BHP-008      76.00  172974.924  47005.830  580.689\n4     BHP-008      76.00  172974.924  47005.830  580.689\n..        ...        ...         ...        ...      ...\n492  ZKY_6A02     100.00  172939.181  46718.258      NaN\n493  ZKY_6A03     126.60  172927.059  46736.341  530.130\n494   ZKY_804     163.15  172972.829  46779.813  535.919\n495  ZKY_8A01     155.00  172946.266  46789.551  540.705\n496  ZKY_8A02      54.75  172929.384  46825.806  552.474\n\n[497 rows x 5 columns]\n"
     ]
    }
   ],
   "source": [
    "# Importing Collar data\n",
    "collar = pd.read_csv('Collar.csv')\n",
    "print(collar)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "id": "applied-drink",
   "metadata": {},
   "outputs": [
    {
     "output_type": "stream",
     "name": "stdout",
     "text": [
      "<class 'pandas.core.frame.DataFrame'>\nRangeIndex: 497 entries, 0 to 496\nData columns (total 5 columns):\n #   Column     Non-Null Count  Dtype  \n---  ------     --------------  -----  \n 0   hole_id    491 non-null    object \n 1   max_depth  495 non-null    float64\n 2   X_Dum      493 non-null    float64\n 3   Y_Dum      491 non-null    float64\n 4   Z_Dum      492 non-null    float64\ndtypes: float64(4), object(1)\nmemory usage: 19.5+ KB\nNone\n"
     ]
    }
   ],
   "source": [
    "# Displaying Collar data configuration\n",
    "print(collar.info())"
   ]
  },
  {
   "source": [
    "As we can see, the collar data consist of 497rows and 5columns. But, each column not showing the same number of rows filled (as shown from 'collar.info()'). Let's find out what exactly the problem is by running python functions that we have defined before."
   ],
   "cell_type": "markdown",
   "metadata": {}
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "id": "planned-orientation",
   "metadata": {},
   "outputs": [
    {
     "output_type": "stream",
     "name": "stdout",
     "text": [
      " Here it is the number of NaN values for each column: \n"
     ]
    },
    {
     "output_type": "execute_result",
     "data": {
      "text/plain": [
       "hole_id      6\n",
       "max_depth    2\n",
       "X_Dum        4\n",
       "Y_Dum        6\n",
       "Z_Dum        5\n",
       "dtype: int64"
      ]
     },
     "metadata": {},
     "execution_count": 5
    }
   ],
   "source": [
    "print(' Here it is the number of NaN values for each column: ')\n",
    "blank_count(collar)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "id": "ambient-contrast",
   "metadata": {},
   "outputs": [
    {
     "output_type": "stream",
     "name": "stdout",
     "text": [
      "Here it is the list of all rows containing Null/NaN values: \n"
     ]
    },
    {
     "output_type": "execute_result",
     "data": {
      "text/plain": [
       "             hole_id  max_depth       X_Dum      Y_Dum    Z_Dum\n",
       "79           LHD-022       39.6  172237.858        NaN  596.466\n",
       "163  TCR_10A01_C-D-2        2.0         NaN  46461.490  613.481\n",
       "289          YRC-171       54.0  172080.494        NaN      NaN\n",
       "295              NaN       24.0         NaN  46424.247  586.379\n",
       "302          YRC-183        NaN  171980.458        NaN  630.992\n",
       "343          YRC-224       51.0         NaN  46556.562  601.378\n",
       "358          YRC-239       51.0  171960.228        NaN  633.460\n",
       "362              NaN       48.0  171845.916  46422.987  601.624\n",
       "382          YRC-265       40.0  171989.313  46477.457      NaN\n",
       "390              NaN      150.0  172119.147  46059.678  544.726\n",
       "403          YRC-285       40.0         NaN  46523.548  635.082\n",
       "421              NaN       29.1  171729.979  46466.611  647.114\n",
       "422          ZKC 201       60.5  171811.544  46507.197      NaN\n",
       "423          ZKC 202      119.0  171802.935        NaN  643.579\n",
       "429         ZKC 2A03        NaN  171784.969  46537.062  647.629\n",
       "451              NaN       14.5  171804.980  46586.899  662.381\n",
       "454          ZKY 001      125.1  172819.457  46730.173      NaN\n",
       "477         ZKY_1001       35.4  172999.758        NaN  539.752\n",
       "490              NaN      141.6  172890.838  46719.308  532.918\n",
       "492         ZKY_6A02      100.0  172939.181  46718.258      NaN"
      ],
      "text/html": "<div>\n<style scoped>\n    .dataframe tbody tr th:only-of-type {\n        vertical-align: middle;\n    }\n\n    .dataframe tbody tr th {\n        vertical-align: top;\n    }\n\n    .dataframe thead th {\n        text-align: right;\n    }\n</style>\n<table border=\"1\" class=\"dataframe\">\n  <thead>\n    <tr style=\"text-align: right;\">\n      <th></th>\n      <th>hole_id</th>\n      <th>max_depth</th>\n      <th>X_Dum</th>\n      <th>Y_Dum</th>\n      <th>Z_Dum</th>\n    </tr>\n  </thead>\n  <tbody>\n    <tr>\n      <th>79</th>\n      <td>LHD-022</td>\n      <td>39.6</td>\n      <td>172237.858</td>\n      <td>NaN</td>\n      <td>596.466</td>\n    </tr>\n    <tr>\n      <th>163</th>\n      <td>TCR_10A01_C-D-2</td>\n      <td>2.0</td>\n      <td>NaN</td>\n      <td>46461.490</td>\n      <td>613.481</td>\n    </tr>\n    <tr>\n      <th>289</th>\n      <td>YRC-171</td>\n      <td>54.0</td>\n      <td>172080.494</td>\n      <td>NaN</td>\n      <td>NaN</td>\n    </tr>\n    <tr>\n      <th>295</th>\n      <td>NaN</td>\n      <td>24.0</td>\n      <td>NaN</td>\n      <td>46424.247</td>\n      <td>586.379</td>\n    </tr>\n    <tr>\n      <th>302</th>\n      <td>YRC-183</td>\n      <td>NaN</td>\n      <td>171980.458</td>\n      <td>NaN</td>\n      <td>630.992</td>\n    </tr>\n    <tr>\n      <th>343</th>\n      <td>YRC-224</td>\n      <td>51.0</td>\n      <td>NaN</td>\n      <td>46556.562</td>\n      <td>601.378</td>\n    </tr>\n    <tr>\n      <th>358</th>\n      <td>YRC-239</td>\n      <td>51.0</td>\n      <td>171960.228</td>\n      <td>NaN</td>\n      <td>633.460</td>\n    </tr>\n    <tr>\n      <th>362</th>\n      <td>NaN</td>\n      <td>48.0</td>\n      <td>171845.916</td>\n      <td>46422.987</td>\n      <td>601.624</td>\n    </tr>\n    <tr>\n      <th>382</th>\n      <td>YRC-265</td>\n      <td>40.0</td>\n      <td>171989.313</td>\n      <td>46477.457</td>\n      <td>NaN</td>\n    </tr>\n    <tr>\n      <th>390</th>\n      <td>NaN</td>\n      <td>150.0</td>\n      <td>172119.147</td>\n      <td>46059.678</td>\n      <td>544.726</td>\n    </tr>\n    <tr>\n      <th>403</th>\n      <td>YRC-285</td>\n      <td>40.0</td>\n      <td>NaN</td>\n      <td>46523.548</td>\n      <td>635.082</td>\n    </tr>\n    <tr>\n      <th>421</th>\n      <td>NaN</td>\n      <td>29.1</td>\n      <td>171729.979</td>\n      <td>46466.611</td>\n      <td>647.114</td>\n    </tr>\n    <tr>\n      <th>422</th>\n      <td>ZKC 201</td>\n      <td>60.5</td>\n      <td>171811.544</td>\n      <td>46507.197</td>\n      <td>NaN</td>\n    </tr>\n    <tr>\n      <th>423</th>\n      <td>ZKC 202</td>\n      <td>119.0</td>\n      <td>171802.935</td>\n      <td>NaN</td>\n      <td>643.579</td>\n    </tr>\n    <tr>\n      <th>429</th>\n      <td>ZKC 2A03</td>\n      <td>NaN</td>\n      <td>171784.969</td>\n      <td>46537.062</td>\n      <td>647.629</td>\n    </tr>\n    <tr>\n      <th>451</th>\n      <td>NaN</td>\n      <td>14.5</td>\n      <td>171804.980</td>\n      <td>46586.899</td>\n      <td>662.381</td>\n    </tr>\n    <tr>\n      <th>454</th>\n      <td>ZKY 001</td>\n      <td>125.1</td>\n      <td>172819.457</td>\n      <td>46730.173</td>\n      <td>NaN</td>\n    </tr>\n    <tr>\n      <th>477</th>\n      <td>ZKY_1001</td>\n      <td>35.4</td>\n      <td>172999.758</td>\n      <td>NaN</td>\n      <td>539.752</td>\n    </tr>\n    <tr>\n      <th>490</th>\n      <td>NaN</td>\n      <td>141.6</td>\n      <td>172890.838</td>\n      <td>46719.308</td>\n      <td>532.918</td>\n    </tr>\n    <tr>\n      <th>492</th>\n      <td>ZKY_6A02</td>\n      <td>100.0</td>\n      <td>172939.181</td>\n      <td>46718.258</td>\n      <td>NaN</td>\n    </tr>\n  </tbody>\n</table>\n</div>"
     },
     "metadata": {},
     "execution_count": 6
    }
   ],
   "source": [
    "print('Here it is the list of all rows containing Null/NaN values: ')\n",
    "blank_data(collar)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "id": "pediatric-assist",
   "metadata": {},
   "outputs": [
    {
     "output_type": "stream",
     "name": "stdout",
     "text": [
      "Here it is the number duplicated hole_id:\n"
     ]
    },
    {
     "output_type": "execute_result",
     "data": {
      "text/plain": [
       "hole_id\n",
       "BHP-008          2\n",
       "BTC_301_A-B-5    2\n",
       "HFD-023          2\n",
       "LHD-001          2\n",
       "LHD-037          2\n",
       "ZKC 202          2\n",
       "ZKC 301          2\n",
       "ZKC_6A01         2\n",
       "dtype: int64"
      ]
     },
     "metadata": {},
     "execution_count": 7
    }
   ],
   "source": [
    "print('Here it is the number duplicated hole_id:')\n",
    "duplicate_count(collar)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "id": "brave-voltage",
   "metadata": {},
   "outputs": [
    {
     "output_type": "stream",
     "name": "stdout",
     "text": [
      "Here it is the data which have duplicates:\n"
     ]
    },
    {
     "output_type": "execute_result",
     "data": {
      "text/plain": [
       "           hole_id  max_depth       X_Dum      Y_Dum    Z_Dum\n",
       "3          BHP-008      76.00  172974.924  47005.830  580.689\n",
       "4          BHP-008      76.00  172974.924  47005.830  580.689\n",
       "15   BTC_301_A-B-5       2.00  171707.390  46461.860  648.864\n",
       "16   BTC_301_A-B-5       2.00  171707.390  46461.860  648.864\n",
       "25         HFD-023      46.30  172350.246  46645.453  624.827\n",
       "26         HFD-023      46.30  172350.246  46645.453  624.827\n",
       "57         LHD-001      98.00  172152.255  46558.831  593.214\n",
       "58         LHD-001      98.00  172152.255  46558.831  593.214\n",
       "95         LHD-037      56.20  172420.803  46553.502  586.616\n",
       "107        LHD-037      56.20  172420.803  46553.502  586.616\n",
       "295            NaN      24.00         NaN  46424.247  586.379\n",
       "362            NaN      48.00  171845.916  46422.987  601.624\n",
       "390            NaN     150.00  172119.147  46059.678  544.726\n",
       "421            NaN      29.10  171729.979  46466.611  647.114\n",
       "423        ZKC 202     119.00  171802.935        NaN  643.579\n",
       "424        ZKC 202      95.10  171794.712  46561.361  651.903\n",
       "433        ZKC 301      90.90  171704.288  46476.070  652.028\n",
       "434        ZKC 301      70.30  171684.791  46513.767  666.293\n",
       "451            NaN      14.50  171804.980  46586.899  662.381\n",
       "452       ZKC_6A01      82.95  171866.022  46550.562  637.903\n",
       "471       ZKC_6A01     155.50  172918.504  46754.440  533.267\n",
       "490            NaN     141.60  172890.838  46719.308  532.918"
      ],
      "text/html": "<div>\n<style scoped>\n    .dataframe tbody tr th:only-of-type {\n        vertical-align: middle;\n    }\n\n    .dataframe tbody tr th {\n        vertical-align: top;\n    }\n\n    .dataframe thead th {\n        text-align: right;\n    }\n</style>\n<table border=\"1\" class=\"dataframe\">\n  <thead>\n    <tr style=\"text-align: right;\">\n      <th></th>\n      <th>hole_id</th>\n      <th>max_depth</th>\n      <th>X_Dum</th>\n      <th>Y_Dum</th>\n      <th>Z_Dum</th>\n    </tr>\n  </thead>\n  <tbody>\n    <tr>\n      <th>3</th>\n      <td>BHP-008</td>\n      <td>76.00</td>\n      <td>172974.924</td>\n      <td>47005.830</td>\n      <td>580.689</td>\n    </tr>\n    <tr>\n      <th>4</th>\n      <td>BHP-008</td>\n      <td>76.00</td>\n      <td>172974.924</td>\n      <td>47005.830</td>\n      <td>580.689</td>\n    </tr>\n    <tr>\n      <th>15</th>\n      <td>BTC_301_A-B-5</td>\n      <td>2.00</td>\n      <td>171707.390</td>\n      <td>46461.860</td>\n      <td>648.864</td>\n    </tr>\n    <tr>\n      <th>16</th>\n      <td>BTC_301_A-B-5</td>\n      <td>2.00</td>\n      <td>171707.390</td>\n      <td>46461.860</td>\n      <td>648.864</td>\n    </tr>\n    <tr>\n      <th>25</th>\n      <td>HFD-023</td>\n      <td>46.30</td>\n      <td>172350.246</td>\n      <td>46645.453</td>\n      <td>624.827</td>\n    </tr>\n    <tr>\n      <th>26</th>\n      <td>HFD-023</td>\n      <td>46.30</td>\n      <td>172350.246</td>\n      <td>46645.453</td>\n      <td>624.827</td>\n    </tr>\n    <tr>\n      <th>57</th>\n      <td>LHD-001</td>\n      <td>98.00</td>\n      <td>172152.255</td>\n      <td>46558.831</td>\n      <td>593.214</td>\n    </tr>\n    <tr>\n      <th>58</th>\n      <td>LHD-001</td>\n      <td>98.00</td>\n      <td>172152.255</td>\n      <td>46558.831</td>\n      <td>593.214</td>\n    </tr>\n    <tr>\n      <th>95</th>\n      <td>LHD-037</td>\n      <td>56.20</td>\n      <td>172420.803</td>\n      <td>46553.502</td>\n      <td>586.616</td>\n    </tr>\n    <tr>\n      <th>107</th>\n      <td>LHD-037</td>\n      <td>56.20</td>\n      <td>172420.803</td>\n      <td>46553.502</td>\n      <td>586.616</td>\n    </tr>\n    <tr>\n      <th>295</th>\n      <td>NaN</td>\n      <td>24.00</td>\n      <td>NaN</td>\n      <td>46424.247</td>\n      <td>586.379</td>\n    </tr>\n    <tr>\n      <th>362</th>\n      <td>NaN</td>\n      <td>48.00</td>\n      <td>171845.916</td>\n      <td>46422.987</td>\n      <td>601.624</td>\n    </tr>\n    <tr>\n      <th>390</th>\n      <td>NaN</td>\n      <td>150.00</td>\n      <td>172119.147</td>\n      <td>46059.678</td>\n      <td>544.726</td>\n    </tr>\n    <tr>\n      <th>421</th>\n      <td>NaN</td>\n      <td>29.10</td>\n      <td>171729.979</td>\n      <td>46466.611</td>\n      <td>647.114</td>\n    </tr>\n    <tr>\n      <th>423</th>\n      <td>ZKC 202</td>\n      <td>119.00</td>\n      <td>171802.935</td>\n      <td>NaN</td>\n      <td>643.579</td>\n    </tr>\n    <tr>\n      <th>424</th>\n      <td>ZKC 202</td>\n      <td>95.10</td>\n      <td>171794.712</td>\n      <td>46561.361</td>\n      <td>651.903</td>\n    </tr>\n    <tr>\n      <th>433</th>\n      <td>ZKC 301</td>\n      <td>90.90</td>\n      <td>171704.288</td>\n      <td>46476.070</td>\n      <td>652.028</td>\n    </tr>\n    <tr>\n      <th>434</th>\n      <td>ZKC 301</td>\n      <td>70.30</td>\n      <td>171684.791</td>\n      <td>46513.767</td>\n      <td>666.293</td>\n    </tr>\n    <tr>\n      <th>451</th>\n      <td>NaN</td>\n      <td>14.50</td>\n      <td>171804.980</td>\n      <td>46586.899</td>\n      <td>662.381</td>\n    </tr>\n    <tr>\n      <th>452</th>\n      <td>ZKC_6A01</td>\n      <td>82.95</td>\n      <td>171866.022</td>\n      <td>46550.562</td>\n      <td>637.903</td>\n    </tr>\n    <tr>\n      <th>471</th>\n      <td>ZKC_6A01</td>\n      <td>155.50</td>\n      <td>172918.504</td>\n      <td>46754.440</td>\n      <td>533.267</td>\n    </tr>\n    <tr>\n      <th>490</th>\n      <td>NaN</td>\n      <td>141.60</td>\n      <td>172890.838</td>\n      <td>46719.308</td>\n      <td>532.918</td>\n    </tr>\n  </tbody>\n</table>\n</div>"
     },
     "metadata": {},
     "execution_count": 8
    }
   ],
   "source": [
    "print('Here it is the data which have duplicates:')\n",
    "duplicate_data(collar)"
   ]
  },
  {
   "source": [
    "Next, we can fix the data by assigning the correct data values, dropping unnecessary data, or editing mistyped data."
   ],
   "cell_type": "markdown",
   "metadata": {}
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.8.8-final"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}