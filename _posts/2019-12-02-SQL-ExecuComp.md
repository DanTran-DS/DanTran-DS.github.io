---

layout: single
title: Executives' Compensation EDA with SQL
excerpt: ""
tags: [EDA, SQL, Python]
comments: true
category: blog
toc: true
toc_sticky: false

---

# Overview

Nowadays it is easy to find news, articles talking about how large CEO’s compensations are, why they are significantly higher than employees’ or even managements’ salary. 

> According to Economic Policy Institute, “CEO compensation has grown 940% since 1978”. CEO pays was about 17 million dollars in average in 2018. 

This notebook includes perform Exploratory data analysis using SQL on the executives' compensation using data from Execucomp and Compustat.

Execucomp contains executive (CEO, CFO) compensation for firms in the Standard and Poor (S&P) 1000 index from 1992 to present. It has information about several components of CEO compensation package such as base salary, stock/call options, bonus and benefits. The database also has some information regarding executive’s characteristic such as age, gender. 

Compustat is a popular database in financial research due to its spread in both time and number of firm. The database contains more than 500 firms’ fundamentals for more than 84,000 active and inactive companies since 1950. Firm fundamentals are financial information that are reported in quarterly or annual statements such as 10-K, 8-Q, press releases, earnings conference calls etc..

# Connect to WRDS database

Install wrds library if it wasn't


```python
!pip install wrds
```

```python
import wrds
```

Create a connection to wrds database


```python
conn = wrds.Connection(wrds_username='dantran')
```

    Loading library list...
    Done
    

Let's create a pgpass file so we don't need to type the username and password everytime we need to connect to the database.


```python
conn.create_pgpass_file()
```

    Enter your WRDS username [Dan]:dantran
    Enter your password:········
    

# Dataset

Since wrds library returns pandas dataframe, we can change some settings in pandas to show more columns and rows.


```python
import pandas as pd
pd.set_option('display.max_rows', 500)
pd.set_option('display.max_columns', 500)
pd.set_option('display.max_colwidth', 100)
```

List all avaiable libraries in WRDS


```python
conn.list_libraries()
```


Our target is CEO compensation which can be found in Execucomp library. Let's list tables in the library.


```python
conn.list_tables(library='execcomp')
```




    ['anncomp',
     'codirfin',
     'colev',
     'coperol',
     'deferredcomp',
     'directorcomp',
     'ex_black',
     'ex_header',
     'exnames',
     'ltawdtab',
     'outstandingawards',
     'pension',
     'person',
     'planbasedawards',
     'stgrttab']



The table of interest is anncomp whichl ists all named executives, titles, and their compensation data. 
Compensation data includes items such as: salary, stock options, bonuses, and shares owned. Let's look at the first 10 columns of the table.


```python
conn.describe_table(library="execcomp", table="anncomp").head(10)
```

    Approximately 299144 rows in execcomp.anncomp.
    




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
      <th>name</th>
      <th>nullable</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>exec_fullname</td>
      <td>True</td>
      <td>VARCHAR(50)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>cfoann</td>
      <td>True</td>
      <td>VARCHAR(3)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>execdir</td>
      <td>True</td>
      <td>DOUBLE PRECISION</td>
    </tr>
    <tr>
      <th>3</th>
      <td>old_datafmt_flag</td>
      <td>True</td>
      <td>DOUBLE PRECISION</td>
    </tr>
    <tr>
      <th>4</th>
      <td>interlock</td>
      <td>True</td>
      <td>DOUBLE PRECISION</td>
    </tr>
    <tr>
      <th>5</th>
      <td>reprice</td>
      <td>True</td>
      <td>DOUBLE PRECISION</td>
    </tr>
    <tr>
      <th>6</th>
      <td>execrank</td>
      <td>True</td>
      <td>DOUBLE PRECISION</td>
    </tr>
    <tr>
      <th>7</th>
      <td>co_per_rol</td>
      <td>True</td>
      <td>DOUBLE PRECISION</td>
    </tr>
    <tr>
      <th>8</th>
      <td>coname</td>
      <td>True</td>
      <td>VARCHAR(50)</td>
    </tr>
    <tr>
      <th>9</th>
      <td>titleann</td>
      <td>True</td>
      <td>VARCHAR(250)</td>
    </tr>
  </tbody>
</table>
</div>




```python
query = """
            SELECT exec_fullname, total_sec,salary,bonus, year, gender, age, coname,inddesc,city, state, execdir, ceoann
            FROM execcomp.anncomp
            LIMIT 5;
        """

conn.raw_sql(query)
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
      <th>exec_fullname</th>
      <th>total_sec</th>
      <th>salary</th>
      <th>bonus</th>
      <th>year</th>
      <th>gender</th>
      <th>age</th>
      <th>coname</th>
      <th>inddesc</th>
      <th>city</th>
      <th>state</th>
      <th>execdir</th>
      <th>ceoann</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Ira A. Eichner</td>
      <td>None</td>
      <td>460.4</td>
      <td>92.08</td>
      <td>1992.0</td>
      <td>MALE</td>
      <td>NaN</td>
      <td>AAR CORP</td>
      <td>Aerospace &amp; Defense</td>
      <td>Wood Dale</td>
      <td>IL</td>
      <td>1.0</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ira A. Eichner</td>
      <td>None</td>
      <td>474.2</td>
      <td>240.00</td>
      <td>1993.0</td>
      <td>MALE</td>
      <td>NaN</td>
      <td>AAR CORP</td>
      <td>Aerospace &amp; Defense</td>
      <td>Wood Dale</td>
      <td>IL</td>
      <td>1.0</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ira A. Eichner</td>
      <td>None</td>
      <td>480.0</td>
      <td>265.00</td>
      <td>1994.0</td>
      <td>MALE</td>
      <td>64.0</td>
      <td>AAR CORP</td>
      <td>Aerospace &amp; Defense</td>
      <td>Wood Dale</td>
      <td>IL</td>
      <td>1.0</td>
      <td>CEO</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ira A. Eichner</td>
      <td>None</td>
      <td>498.4</td>
      <td>503.10</td>
      <td>1995.0</td>
      <td>MALE</td>
      <td>65.0</td>
      <td>AAR CORP</td>
      <td>Aerospace &amp; Defense</td>
      <td>Wood Dale</td>
      <td>IL</td>
      <td>1.0</td>
      <td>CEO</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ira A. Eichner</td>
      <td>None</td>
      <td>440.5</td>
      <td>400.00</td>
      <td>1996.0</td>
      <td>MALE</td>
      <td>66.0</td>
      <td>AAR CORP</td>
      <td>Aerospace &amp; Defense</td>
      <td>Wood Dale</td>
      <td>IL</td>
      <td>1.0</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



# Descriptive Statistics

Let's see numbers of unique firms and executives, the range of year in the data.
Since the year is in _Float_ format, we use **CAST** to change it into _Interger_.


```python
query = """
            SELECT COUNT(DISTINCT execid) as num_exec, 
                COUNT(DISTINCT gvkey) as num_company,
                CAST(AVG(total_sec) as DECIMAL(10,2)) as Avg_Compensation,
                CAST(MIN(total_sec) as DECIMAL(10,2)) as Min_Compensation,
                CAST(Max(total_sec) as DECIMAL(10,2)) as Max_Compensation,
                CAST(MIN(year) as INT) as from_year,
                CAST(MAX(year) as INT) as to_year
            FROM execcomp.anncomp;
        """

conn.raw_sql(query)
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
      <th>num_exec</th>
      <th>num_company</th>
      <th>avg_compensation</th>
      <th>min_compensation</th>
      <th>max_compensation</th>
      <th>from_year</th>
      <th>to_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>50560</td>
      <td>3772</td>
      <td>2972.71</td>
      <td>-6226.8</td>
      <td>1927510.71</td>
      <td>1992</td>
      <td>2019</td>
    </tr>
  </tbody>
</table>
</div>



The minimum compensation is negative which is odd. We may have data entry error.
Let's look at observations with negative values.


```python
query = """
            SELECT exec_fullname, coname, year, total_sec, salary,
                 bonus, salary, stock_awards,STOCK_UNVEST_VAL, 
                 option_awards, OTHCOMP
            FROM execcomp.anncomp
            WHERE total_sec < 0
            LIMIT 5;
        """

conn.raw_sql(query)
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
      <th>exec_fullname</th>
      <th>coname</th>
      <th>year</th>
      <th>total_sec</th>
      <th>salary</th>
      <th>bonus</th>
      <th>salary</th>
      <th>stock_awards</th>
      <th>stock_unvest_val</th>
      <th>option_awards</th>
      <th>othcomp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>James A. Beer</td>
      <td>AMERICAN AIRLINES GROUP INC</td>
      <td>2006.0</td>
      <td>-2841.179</td>
      <td>122.310</td>
      <td>0.0</td>
      <td>122.310</td>
      <td>-2985.899</td>
      <td>0.0</td>
      <td>0.000</td>
      <td>10.186</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kenneth J. Martin</td>
      <td>WYETH</td>
      <td>2007.0</td>
      <td>-934.875</td>
      <td>416.000</td>
      <td>0.0</td>
      <td>416.000</td>
      <td>-1745.050</td>
      <td>0.0</td>
      <td>-777.370</td>
      <td>34.544</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Richard D. Nanula</td>
      <td>AMGEN INC</td>
      <td>2007.0</td>
      <td>-1964.052</td>
      <td>488.883</td>
      <td>0.0</td>
      <td>488.883</td>
      <td>-2445.757</td>
      <td>0.0</td>
      <td>-321.455</td>
      <td>314.277</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alfred J. Castino</td>
      <td>AUTODESK INC</td>
      <td>2008.0</td>
      <td>-726.558</td>
      <td>240.114</td>
      <td>0.0</td>
      <td>240.114</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>-968.724</td>
      <td>2.052</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Phillip P. DePaul</td>
      <td>OFFICEMAX INC</td>
      <td>2008.0</td>
      <td>-76.793</td>
      <td>87.249</td>
      <td>0.0</td>
      <td>87.249</td>
      <td>-184.167</td>
      <td>0.0</td>
      <td>0.000</td>
      <td>20.125</td>
    </tr>
  </tbody>
</table>
</div>



So the total compensations are negative due to stock awards and option awards which is plausible since if the executive couldn't render the service required for an award, the employer may  reverse the amount of compensation.

# Verify EPI article's number

Let's verify the number from the EPI artical which say Average pay of CEOs at the top 350 firms in 2018 was $17.2 million—or $14.0 million using a more conservative measure.

We need to filter our data.
* Executive: CEO only
* Year: 2018
* Firms: top 350 U.S. firms ranked by sales

Since we don't have Sales data in the Execucomp table (execcomp.anncomp), we need to merge this table with Compustat table (comp.funda) which have firms accounting information. 

First, we filter out CEO from Execucomp table by using **Where** clause on *ceoann* which shows CEO if that executive is the CEO.
Second, we sort Compustat table by *sale* then take the first 350 observations.
Third, we use **Inner Join** since we need data from both tables.
Lastly, we use aggregate function **AVG()** to get the average CEO compensation. 


```python
query = """
        SELECT AVG(total_sec) AS Average_CEO_compensation
        FROM (SELECT * FROM execcomp.anncomp WHERE ceoann ='CEO') AS a 
        INNER JOIN (SELECT * FROM comp.funda 
                    WHERE fyear=2018 AND sale IS NOT NULL
                    ORDER BY sale DESC LIMIT 350) AS b
        ON a.gvkey=b.gvkey and a.year=b.fyear
        """
conn.raw_sql(query)
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
      <th>average_ceo_compensation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17806.933551</td>
    </tr>
  </tbody>
</table>
</div>



The result is 17.81 million dollars which is close to the value from the article. The discrepancy may due to differces in compensation measure. I use *total_sec* which is the total compensation as reported in SEC filing. The article's measure is not mentioned.

# EDA

## Group by Year

Let's check average salary and number of executives, firms by year


```python
query = """
        SELECT  CAST(year AS INT), COUNT(DISTINCT execid) AS num_exec,
                COUNT(DISTINCT gvkey) AS num_firms, 
                CAST(AVG(salary) AS DECIMAL(10,2)) AS avg_salary 
        FROM execcomp.anncomp 
        GROUP BY year
        """
conn.raw_sql(query)
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
      <th>year</th>
      <th>num_exec</th>
      <th>num_firms</th>
      <th>avg_salary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1992</td>
      <td>8024</td>
      <td>1571</td>
      <td>273.09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1993</td>
      <td>9760</td>
      <td>1682</td>
      <td>277.90</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1994</td>
      <td>10613</td>
      <td>1749</td>
      <td>282.94</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1995</td>
      <td>11053</td>
      <td>1849</td>
      <td>291.84</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1996</td>
      <td>11564</td>
      <td>1977</td>
      <td>298.85</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1997</td>
      <td>11950</td>
      <td>2037</td>
      <td>306.60</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1998</td>
      <td>12515</td>
      <td>2071</td>
      <td>316.59</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1999</td>
      <td>12094</td>
      <td>1954</td>
      <td>331.40</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2000</td>
      <td>11466</td>
      <td>1846</td>
      <td>350.15</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2001</td>
      <td>11324</td>
      <td>1848</td>
      <td>365.17</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2002</td>
      <td>11498</td>
      <td>1886</td>
      <td>374.25</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2003</td>
      <td>11751</td>
      <td>1937</td>
      <td>384.85</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2004</td>
      <td>10863</td>
      <td>1879</td>
      <td>411.28</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2005</td>
      <td>9356</td>
      <td>1763</td>
      <td>442.85</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2006</td>
      <td>10892</td>
      <td>1931</td>
      <td>429.45</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2007</td>
      <td>12904</td>
      <td>2286</td>
      <td>421.91</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2008</td>
      <td>12410</td>
      <td>2206</td>
      <td>440.27</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2009</td>
      <td>11847</td>
      <td>2175</td>
      <td>452.37</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2010</td>
      <td>11541</td>
      <td>2151</td>
      <td>469.84</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2011</td>
      <td>11340</td>
      <td>2115</td>
      <td>481.50</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2012</td>
      <td>11171</td>
      <td>2088</td>
      <td>492.76</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2013</td>
      <td>11025</td>
      <td>2058</td>
      <td>505.81</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2014</td>
      <td>10825</td>
      <td>2030</td>
      <td>518.52</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2015</td>
      <td>10503</td>
      <td>1953</td>
      <td>528.51</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2016</td>
      <td>9968</td>
      <td>1870</td>
      <td>540.17</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2017</td>
      <td>9409</td>
      <td>1791</td>
      <td>557.46</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2018</td>
      <td>8802</td>
      <td>1705</td>
      <td>583.14</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2019</td>
      <td>371</td>
      <td>71</td>
      <td>611.85</td>
    </tr>
  </tbody>
</table>
</div>



There are much less observations in 2019 because the year column is the Fiscal Year. 
Many firms have the fiscal year different than calendar year; Although it's December 2019 now but their fiscal year have not ended yet. i.e. Microsoft's 2019 fiscal year ends on June 30, 2020.  

## Male vs Female

Let see the differnet between Male and Female Executive's salary. The salary is in Thousands of dollars.

**COUNT DISTINCT** is used since the data are accross multiple years so an executive can appear several times.


```python
query = """
        SELECT  gender, COUNT(DISTINCT execid), AVG(salary) as avg_salary 
        FROM execcomp.anncomp 
        GROUP BY gender
        """
conn.raw_sql(query)
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
      <th>gender</th>
      <th>count</th>
      <th>avg_salary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FEMALE</td>
      <td>4282</td>
      <td>375.718853</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MALE</td>
      <td>46278</td>
      <td>413.134899</td>
    </tr>
  </tbody>
</table>
</div>



So there are 10 times more male executives than female and in average, male executives has a higher compensation.

Next, let's compute the differences between Male and Female Executive by year. We can see the gap becomes smaller in 2018 compare to 1992.


```python
query = """
        SELECT a.*, b.female_avg_salary, male_avg_salary-female_avg_salary AS Diff
        FROM (SELECT year, CAST(AVG(salary) AS DECIMAL(10,2)) AS male_avg_salary
            FROM execcomp.anncomp
            WHERE gender = 'MALE'
            GROUP BY year) AS a
        JOIN (SELECT year, CAST(AVG(salary) AS DECIMAL(10,2)) AS female_avg_salary
            FROM execcomp.anncomp
            WHERE gender = 'FEMALE'
            GROUP BY year) AS b
        ON a.year = b.year;
        """
conn.raw_sql(query)
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
      <th>year</th>
      <th>male_avg_salary</th>
      <th>female_avg_salary</th>
      <th>diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1992.0</td>
      <td>274.87</td>
      <td>170.50</td>
      <td>104.37</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1993.0</td>
      <td>279.67</td>
      <td>197.06</td>
      <td>82.61</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1994.0</td>
      <td>285.51</td>
      <td>196.15</td>
      <td>89.36</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1995.0</td>
      <td>294.87</td>
      <td>208.80</td>
      <td>86.07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1996.0</td>
      <td>301.95</td>
      <td>221.79</td>
      <td>80.16</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1997.0</td>
      <td>309.68</td>
      <td>238.45</td>
      <td>71.23</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1998.0</td>
      <td>320.41</td>
      <td>239.80</td>
      <td>80.61</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1999.0</td>
      <td>335.67</td>
      <td>255.49</td>
      <td>80.18</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2000.0</td>
      <td>354.01</td>
      <td>287.60</td>
      <td>66.41</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2001.0</td>
      <td>369.80</td>
      <td>296.66</td>
      <td>73.14</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2002.0</td>
      <td>379.01</td>
      <td>307.26</td>
      <td>71.75</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2003.0</td>
      <td>389.71</td>
      <td>318.93</td>
      <td>70.78</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2004.0</td>
      <td>416.92</td>
      <td>337.47</td>
      <td>79.45</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2005.0</td>
      <td>448.36</td>
      <td>370.67</td>
      <td>77.69</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2006.0</td>
      <td>434.47</td>
      <td>366.97</td>
      <td>67.50</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2007.0</td>
      <td>427.28</td>
      <td>357.87</td>
      <td>69.41</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2008.0</td>
      <td>445.82</td>
      <td>377.00</td>
      <td>68.82</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2009.0</td>
      <td>457.85</td>
      <td>391.54</td>
      <td>66.31</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2010.0</td>
      <td>475.71</td>
      <td>407.52</td>
      <td>68.19</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2011.0</td>
      <td>487.90</td>
      <td>412.84</td>
      <td>75.06</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2012.0</td>
      <td>499.53</td>
      <td>423.71</td>
      <td>75.82</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2013.0</td>
      <td>512.46</td>
      <td>439.95</td>
      <td>72.51</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2014.0</td>
      <td>524.62</td>
      <td>458.53</td>
      <td>66.09</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2015.0</td>
      <td>534.62</td>
      <td>472.31</td>
      <td>62.31</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2016.0</td>
      <td>546.63</td>
      <td>483.49</td>
      <td>63.14</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2017.0</td>
      <td>564.99</td>
      <td>494.67</td>
      <td>70.32</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2018.0</td>
      <td>590.35</td>
      <td>522.72</td>
      <td>67.63</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2019.0</td>
      <td>611.94</td>
      <td>610.93</td>
      <td>1.01</td>
    </tr>
  </tbody>
</table>
</div>



## Age Group


```python
query = """SELECT COUNT(*),
                CASE WHEN age < 40 THEN 'less than 40'
                    WHEN age > 60 THEN 'over 60'
                    ELSE 'between 40 and 60'
                END as age_group
            FROM execcomp.anncomp
            WHERE year = 2018
            GROUP BY age_group
            LIMIT 10;
        """
conn.raw_sql(query)
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
      <th>count</th>
      <th>age_group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1647</td>
      <td>over 60</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7037</td>
      <td>between 40 and 60</td>
    </tr>
    <tr>
      <th>2</th>
      <td>167</td>
      <td>less than 40</td>
    </tr>
  </tbody>
</table>
</div>



## By Position 


```python
query = """
        SELECT  COUNT(DISTINCT execid) as num_exec, 
                CAST(AVG(salary) AS DECIMAL(10,2)) as avg_salary,
                CASE 
                    WHEN ceoann = 'CEO' 
                        THEN CASE
                            WHEN execdir = 1 THEN 'CEO and Director'
                            ELSE 'CEO'
                        END
                    WHEN cfoann = 'CFO' 
                        THEN CASE
                            WHEN execdir = 1 THEN 'CFO and Director'
                            ELSE 'CFO'
                        END
                    WHEN execdir = 1 THEN 'Director'
                    ELSE 'Other'
                END as Position              
        FROM execcomp.anncomp 
        WHERE year = 2018
        GROUP BY Position
        ORDER BY avg_salary DESC
        LIMIT 10;
        """
conn.raw_sql(query)
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
      <th>num_exec</th>
      <th>avg_salary</th>
      <th>position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1624</td>
      <td>910.20</td>
      <td>CEO and Director</td>
    </tr>
    <tr>
      <th>1</th>
      <td>64</td>
      <td>783.17</td>
      <td>CEO</td>
    </tr>
    <tr>
      <th>2</th>
      <td>610</td>
      <td>673.95</td>
      <td>Director</td>
    </tr>
    <tr>
      <th>3</th>
      <td>88</td>
      <td>576.09</td>
      <td>CFO and Director</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1571</td>
      <td>515.06</td>
      <td>CFO</td>
    </tr>
    <tr>
      <th>5</th>
      <td>4876</td>
      <td>482.36</td>
      <td>Other</td>
    </tr>
  </tbody>
</table>
</div>



As expected, executives who work as both CEO and Director have highest pay. 
The combination of CEO and Director is the norm but not CFO and Director.
Futhermore, there are only 64 people work as CEO only.

## By Industry


```python
query = """
        SELECT  INDDESC, COUNT(DISTINCT execid) as num_exec, 
                CAST(AVG(total_sec) AS DECIMAL(10,2)) as avg_compensation 
        FROM execcomp.anncomp 
        WHERE year = 2018 AND inddesc IS NOT NULL
        GROUP BY INDDESC
        ORDER BY avg_compensation DESC
        LIMIT 10;
        """
conn.raw_sql(query)
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
      <th>inddesc</th>
      <th>num_exec</th>
      <th>avg_compensation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Multi-Sector Holdings</td>
      <td>10</td>
      <td>13558.45</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Diversified Banks</td>
      <td>28</td>
      <td>13532.91</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Movies &amp; Entertainment</td>
      <td>44</td>
      <td>11749.07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Broadcasting</td>
      <td>51</td>
      <td>10462.11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Copper</td>
      <td>4</td>
      <td>8504.37</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cable &amp; Satellite</td>
      <td>32</td>
      <td>8261.69</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wireless Telecommunication Services</td>
      <td>27</td>
      <td>7950.31</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Integrated Oil &amp; Gas</td>
      <td>21</td>
      <td>7617.38</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Interactive Home Entertainment</td>
      <td>22</td>
      <td>7612.08</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Industrial Conglomerates</td>
      <td>36</td>
      <td>7573.95</td>
    </tr>
  </tbody>
</table>
</div>



The compensation for executive in Multi-Sector Industry is highest. Let's see what executives, firms are in that industry.


```python
query = """
        SELECT  exec_fullname, coname as company_name, total_sec as Total_compensation,
                bonus, salary, stock_awards,STOCK_UNVEST_VAL, option_awards, OTHCOMP
        FROM execcomp.anncomp 
        WHERE year = 2018 AND inddesc = 'Multi-Sector Holdings';
        """
conn.raw_sql(query)
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
      <th>exec_fullname</th>
      <th>company_name</th>
      <th>total_compensation</th>
      <th>bonus</th>
      <th>salary</th>
      <th>stock_awards</th>
      <th>stock_unvest_val</th>
      <th>option_awards</th>
      <th>othcomp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Warren E. Buffett</td>
      <td>BERKSHIRE HATHAWAY</td>
      <td>388.968</td>
      <td>0.000</td>
      <td>100.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>288.968</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Marc David Hamburg</td>
      <td>BERKSHIRE HATHAWAY</td>
      <td>2263.750</td>
      <td>0.000</td>
      <td>2250.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>13.750</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Charles Thomas Munger</td>
      <td>BERKSHIRE HATHAWAY</td>
      <td>100.000</td>
      <td>0.000</td>
      <td>100.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Gregory Edward Abel</td>
      <td>BERKSHIRE HATHAWAY</td>
      <td>18013.750</td>
      <td>2000.000</td>
      <td>16000.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>13.750</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ajit Jain</td>
      <td>BERKSHIRE HATHAWAY</td>
      <td>18013.750</td>
      <td>2000.000</td>
      <td>16000.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>13.750</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Joseph Saul Steinberg</td>
      <td>JEFFERIES FINANCIAL GRP INC</td>
      <td>1603.172</td>
      <td>310.954</td>
      <td>810.693</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>481.525</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Michael J. Sharp, J.D.</td>
      <td>JEFFERIES FINANCIAL GRP INC</td>
      <td>4504.625</td>
      <td>1100.000</td>
      <td>1000.000</td>
      <td>0.000</td>
      <td>1371.983</td>
      <td>0.0</td>
      <td>4.625</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Richard Brian Handler</td>
      <td>JEFFERIES FINANCIAL GRP INC</td>
      <td>44674.213</td>
      <td>0.000</td>
      <td>1000.000</td>
      <td>38722.504</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>451.709</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Brian Paul Friedman</td>
      <td>JEFFERIES FINANCIAL GRP INC</td>
      <td>44367.629</td>
      <td>0.000</td>
      <td>1000.000</td>
      <td>38722.504</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>145.125</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Teresa S. Gendron</td>
      <td>JEFFERIES FINANCIAL GRP INC</td>
      <td>1654.625</td>
      <td>460.000</td>
      <td>500.000</td>
      <td>0.000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>4.625</td>
    </tr>
  </tbody>
</table>
</div>



## By State

Let's find top 10 states with highest compensation.


```python
query = """
        SELECT  state, COUNT(*) as num_exec, AVG(total_sec) AS compensation
        FROM execcomp.anncomp
        WHERE year = 2018
        GROUP BY state
        ORDER BY num_exec DESC
        LIMIT 10;
        """
conn.raw_sql(query)
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
      <th>state</th>
      <th>num_exec</th>
      <th>compensation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CA</td>
      <td>1163</td>
      <td>4642.790913</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TX</td>
      <td>923</td>
      <td>3357.181257</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NY</td>
      <td>712</td>
      <td>5493.089551</td>
    </tr>
    <tr>
      <th>3</th>
      <td>IL</td>
      <td>495</td>
      <td>3754.118117</td>
    </tr>
    <tr>
      <th>4</th>
      <td>MA</td>
      <td>400</td>
      <td>3660.566107</td>
    </tr>
    <tr>
      <th>5</th>
      <td>OH</td>
      <td>367</td>
      <td>3388.423485</td>
    </tr>
    <tr>
      <th>6</th>
      <td>PA</td>
      <td>367</td>
      <td>3474.161616</td>
    </tr>
    <tr>
      <th>7</th>
      <td>FL</td>
      <td>352</td>
      <td>3927.935335</td>
    </tr>
    <tr>
      <th>8</th>
      <td>NJ</td>
      <td>303</td>
      <td>3882.083393</td>
    </tr>
    <tr>
      <th>9</th>
      <td>None</td>
      <td>282</td>
      <td>4309.203521</td>
    </tr>
  </tbody>
</table>
</div>




```

```
