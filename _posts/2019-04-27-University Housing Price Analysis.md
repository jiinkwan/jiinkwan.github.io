
---

_You are currently looking at **version 1.1** of this notebook. To download notebooks and datafiles, as well as get help on Jupyter notebooks in the Coursera platform, visit the [Jupyter Notebook FAQ](https://www.coursera.org/learn/python-data-analysis/resources/0dhYG) course resource._

---


```python
import pandas as pd
import numpy as np
from scipy.stats import ttest_ind
```

# Assignment 4 - Hypothesis Testing
This assignment requires more individual learning than previous assignments - you are encouraged to check out the [pandas documentation](http://pandas.pydata.org/pandas-docs/stable/) to find functions or methods you might not have used yet, or ask questions on [Stack Overflow](http://stackoverflow.com/) and tag them as pandas and python related. And of course, the discussion forums are open for interaction with your peers and the course staff.

Definitions:
* A _quarter_ is a specific three month period, Q1 is January through March, Q2 is April through June, Q3 is July through September, Q4 is October through December.
* A _recession_ is defined as starting with two consecutive quarters of GDP decline, and ending with two consecutive quarters of GDP growth.
* A _recession bottom_ is the quarter within a recession which had the lowest GDP.
* A _university town_ is a city which has a high percentage of university students compared to the total population of the city.

**Hypothesis**: University towns have their mean housing prices less effected by recessions. Run a t-test to compare the ratio of the mean price of houses in university towns the quarter before the recession starts compared to the recession bottom. (`price_ratio=quarter_before_recession/recession_bottom`)

The following data files are available for this assignment:
* From the [Zillow research data site](http://www.zillow.com/research/data/) there is housing data for the United States. In particular the datafile for [all homes at a city level](http://files.zillowstatic.com/research/public/City/City_Zhvi_AllHomes.csv), ```City_Zhvi_AllHomes.csv```, has median home sale prices at a fine grained level.
* From the Wikipedia page on college towns is a list of [university towns in the United States](https://en.wikipedia.org/wiki/List_of_college_towns#College_towns_in_the_United_States) which has been copy and pasted into the file ```university_towns.txt```.
* From Bureau of Economic Analysis, US Department of Commerce, the [GDP over time](http://www.bea.gov/national/index.htm#gdp) of the United States in current dollars (use the chained value in 2009 dollars), in quarterly intervals, in the file ```gdplev.xls```. For this assignment, only look at GDP data from the first quarter of 2000 onward.

Each function in this assignment below is worth 10%, with the exception of ```run_ttest()```, which is worth 50%.


```python
# Use this dictionary to map state names to two letter acronyms
states = {'OH': 'Ohio', 'KY': 'Kentucky', 'AS': 'American Samoa', 'NV': 'Nevada', 'WY': 'Wyoming', 'NA': 'National', 'AL': 'Alabama', 'MD': 'Maryland', 'AK': 'Alaska', 'UT': 'Utah', 'OR': 'Oregon', 'MT': 'Montana', 'IL': 'Illinois', 'TN': 'Tennessee', 'DC': 'District of Columbia', 'VT': 'Vermont', 'ID': 'Idaho', 'AR': 'Arkansas', 'ME': 'Maine', 'WA': 'Washington', 'HI': 'Hawaii', 'WI': 'Wisconsin', 'MI': 'Michigan', 'IN': 'Indiana', 'NJ': 'New Jersey', 'AZ': 'Arizona', 'GU': 'Guam', 'MS': 'Mississippi', 'PR': 'Puerto Rico', 'NC': 'North Carolina', 'TX': 'Texas', 'SD': 'South Dakota', 'MP': 'Northern Mariana Islands', 'IA': 'Iowa', 'MO': 'Missouri', 'CT': 'Connecticut', 'WV': 'West Virginia', 'SC': 'South Carolina', 'LA': 'Louisiana', 'KS': 'Kansas', 'NY': 'New York', 'NE': 'Nebraska', 'OK': 'Oklahoma', 'FL': 'Florida', 'CA': 'California', 'CO': 'Colorado', 'PA': 'Pennsylvania', 'DE': 'Delaware', 'NM': 'New Mexico', 'RI': 'Rhode Island', 'MN': 'Minnesota', 'VI': 'Virgin Islands', 'NH': 'New Hampshire', 'MA': 'Massachusetts', 'GA': 'Georgia', 'ND': 'North Dakota', 'VA': 'Virginia'}
```


```python
pd.read_table('university_towns.txt')
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
      <th>Alabama[edit]</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Auburn (Auburn University)[1]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Florence (University of North Alabama)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jacksonville (Jacksonville State University)[2]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Livingston (University of West Alabama)[2]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Montevallo (University of Montevallo)[2]</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Troy (Troy University)[2]</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Tuscaloosa (University of Alabama, Stillman Co...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Tuskegee (Tuskegee University)[5]</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Alaska[edit]</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Fairbanks (University of Alaska Fairbanks)[2]</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Arizona[edit]</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Flagstaff (Northern Arizona University)[6]</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Tempe (Arizona State University)</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Tucson (University of Arizona)</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Arkansas[edit]</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Arkadelphia (Henderson State University, Ouach...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Conway (Central Baptist College, Hendrix Colle...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Fayetteville (University of Arkansas)[7]</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Jonesboro (Arkansas State University)[8]</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Magnolia (Southern Arkansas University)[2]</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Monticello (University of Arkansas at Monticel...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Russellville (Arkansas Tech University)[2]</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Searcy (Harding University)[5]</td>
    </tr>
    <tr>
      <th>23</th>
      <td>California[edit]</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Angwin (Pacific Union College)[2]</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Arcata (Humboldt State University)[5]</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Berkeley (University of California, Berkeley)[5]</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Chico (California State University, Chico)[2]</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Claremont (Claremont McKenna College, Pomona C...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Cotati (California State University, Sonoma)[2]</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>536</th>
      <td>Cheney (Eastern Washington University)[2]</td>
    </tr>
    <tr>
      <th>537</th>
      <td>Ellensburg (Central Washington University)[5]</td>
    </tr>
    <tr>
      <th>538</th>
      <td>Pullman (Washington State University)[5]</td>
    </tr>
    <tr>
      <th>539</th>
      <td>University District, Seattle (University of Wa...</td>
    </tr>
    <tr>
      <th>540</th>
      <td>West Virginia[edit]</td>
    </tr>
    <tr>
      <th>541</th>
      <td>Athens (Concord University)[2]</td>
    </tr>
    <tr>
      <th>542</th>
      <td>Buckhannon (West Virginia Wesleyan College)[2]</td>
    </tr>
    <tr>
      <th>543</th>
      <td>Fairmont (Fairmont State University)[2]</td>
    </tr>
    <tr>
      <th>544</th>
      <td>Glenville (Glenville State College)[2]</td>
    </tr>
    <tr>
      <th>545</th>
      <td>Huntington (Marshall University)[2]</td>
    </tr>
    <tr>
      <th>546</th>
      <td>Montgomery (West Virginia University Institute...</td>
    </tr>
    <tr>
      <th>547</th>
      <td>Morgantown (West Virginia University)[2]</td>
    </tr>
    <tr>
      <th>548</th>
      <td>Shepherdstown (Shepherd University)[2]</td>
    </tr>
    <tr>
      <th>549</th>
      <td>West Liberty (West Liberty University)[2]</td>
    </tr>
    <tr>
      <th>550</th>
      <td>Wisconsin[edit]</td>
    </tr>
    <tr>
      <th>551</th>
      <td>Appleton (Lawrence University)</td>
    </tr>
    <tr>
      <th>552</th>
      <td>Eau Claire (University of Wisconsin–Eau Claire)</td>
    </tr>
    <tr>
      <th>553</th>
      <td>Green Bay (University of Wisconsin-Green Bay)</td>
    </tr>
    <tr>
      <th>554</th>
      <td>La Crosse (University of Wisconsin–La Crosse, ...</td>
    </tr>
    <tr>
      <th>555</th>
      <td>Madison (University of Wisconsin–Madison)[2]</td>
    </tr>
    <tr>
      <th>556</th>
      <td>Menomonie (University of Wisconsin–Stout)[2]</td>
    </tr>
    <tr>
      <th>557</th>
      <td>Milwaukee (Marquette University, University of...</td>
    </tr>
    <tr>
      <th>558</th>
      <td>Oshkosh (University of Wisconsin–Oshkosh)</td>
    </tr>
    <tr>
      <th>559</th>
      <td>Platteville (University of Wisconsin–Plattevil...</td>
    </tr>
    <tr>
      <th>560</th>
      <td>River Falls (University of Wisconsin–River Fal...</td>
    </tr>
    <tr>
      <th>561</th>
      <td>Stevens Point (University of Wisconsin–Stevens...</td>
    </tr>
    <tr>
      <th>562</th>
      <td>Waukesha (Carroll University)</td>
    </tr>
    <tr>
      <th>563</th>
      <td>Whitewater (University of Wisconsin–Whitewater...</td>
    </tr>
    <tr>
      <th>564</th>
      <td>Wyoming[edit]</td>
    </tr>
    <tr>
      <th>565</th>
      <td>Laramie (University of Wyoming)[5]</td>
    </tr>
  </tbody>
</table>
<p>566 rows × 1 columns</p>
</div>




```python
def get_list_of_university_towns():
    '''Returns a DataFrame of towns and the states they are in from the 
    university_towns.txt list. The format of the DataFrame should be:
    DataFrame( [ ["Michigan", "Ann Arbor"], ["Michigan", "Yipsilanti"] ], 
    columns=["State", "RegionName"]  )
    
    The following cleaning needs to be done:

    1. For "State", removing characters from "[" to the end.
    2. For "RegionName", when applicable, removing every character from " (" to the end.
    3. Depending on how you read the data, you may need to remove newline character '\n'. '''
    
    states_df = pd.DataFrame.from_dict(states, orient = 'index')
    states_df.columns = ['State']
    states_df
    
    university_town = pd.read_table('./university_towns.txt', header = None)
    
    university_town.columns = ['RegionName']
    university_town['RegionName'] =university_town['RegionName'].apply(lambda x: x[:x.find('[')])
    university_town = pd.merge(university_town,states_df, how = 'left', left_on = 'RegionName', right_on = 'State')
    university_town = university_town.fillna(method = 'ffill')
    university_town = university_town[~(university_town['RegionName'] == university_town['State'])]
    university_town['RegionName'] = university_town['RegionName'].apply(lambda x: x[:x.find('(')].strip())
    university_town = university_town[['State','RegionName']]
    
    
    return university_town.reset_index(drop = True)
```


```python
def get_recession_start():
    '''Returns the year and quarter of the recession start time as a 
    string value in a format such as 2005q3'''
    gdp = pd.read_excel('gdplev.xls',skiprows= 4, header = 1, index_col = 'Unnamed: 4')[
       'GDP in billions of chained 2009 dollars.1'].dropna()
    gdp = pd.DataFrame(gdp)
    #gdp.reset_index(drop=True)
    gdp.columns = ['GDP']
    #gdp.index.rename('Quarter')
    #gdp[gdp['Quarter'] >= '2000q1']
    #gdp
    #gdp = gdp.iloc[214:].reset_index(drop=True)
    gdp = gdp.loc['2000q1':]
    gdp['GDP Diff'] = gdp['GDP'].diff(1)
    gdp = gdp.reset_index()
    gdp.columns = ['Quarter', 'GDP', 'GDP Diff']    
    for i in range(1,len(gdp)):
        if (gdp.loc[i, 'GDP Diff'] < 0) & (gdp.loc[i+1, 'GDP Diff'] < 0):
            return gdp.loc[i,'Quarter']
    return None
```


```python
get_recession_start()
```




    '2008q3'




```python
def get_quarter_before_recession_start():
    '''Returns the year and quarter of the recession start time as a 
    string value in a format such as 2005q3'''
    gdp = pd.read_excel('gdplev.xls',skiprows= 4, header = 1, index_col = 'Unnamed: 4')[
       'GDP in billions of chained 2009 dollars.1'].dropna()
    gdp = pd.DataFrame(gdp)
    #gdp.reset_index(drop=True)
    gdp.columns = ['GDP']
    #gdp.index.rename('Quarter')
    #gdp[gdp['Quarter'] >= '2000q1']
    #gdp
    #gdp = gdp.iloc[214:].reset_index(drop=True)
    gdp = gdp.loc['2000q1':]
    gdp['GDP Diff'] = gdp['GDP'].diff(1)
    gdp = gdp.reset_index()
    gdp.columns = ['Quarter', 'GDP', 'GDP Diff']    
    for i in range(1,len(gdp)):
        if (gdp.loc[i, 'GDP Diff'] < 0) & (gdp.loc[i+1, 'GDP Diff'] < 0):
            return gdp.loc[i-1,'Quarter']
    return None
```


```python
get_quarter_before_recession_start()
```




    '2008q2'




```python
def get_recession_end():
    '''Returns the year and quarter of the recession end time as a 
    string value in a format such as 2005q3'''
    gdp = pd.read_excel('gdplev.xls',skiprows= 4, header = 1, index_col = 'Unnamed: 4')[
       'GDP in billions of chained 2009 dollars.1'].dropna()
    gdp = pd.DataFrame(gdp)
    #gdp.reset_index(drop=True)
    gdp.columns = ['GDP']
    #gdp.index.rename('Quarter')
    #gdp[gdp['Quarter'] >= '2000q1']
    #gdp
    #gdp = gdp.iloc[214:].reset_index(drop=True)
    gdp = gdp.loc['2000q1':]
    gdp['GDP Diff'] = gdp['GDP'].diff(1)
    gdp = gdp.reset_index()
    gdp.columns = ['Quarter', 'GDP', 'GDP Diff'] 
    gdp_after = gdp[gdp['Quarter' ] > get_recession_start()]
    gdp_after = gdp_after.reset_index(drop = True)
    for i in range(1,len(gdp_after)):
        if (gdp_after.loc[i, 'GDP Diff'] > 0) & (gdp_after.loc[i+1, 'GDP Diff'] > 0):
            return gdp_after.loc[i+1,'Quarter']    
    return None
```


```python
get_recession_end()
```




    '2009q4'




```python
def get_recession_bottom():
    '''Returns the year and quarter of the recession bottom time as a 
    string value in a format such as 2005q3'''
    
    gdp = pd.read_excel('gdplev.xls',skiprows= 4, header = 1, index_col = 'Unnamed: 4')[
       'GDP in billions of chained 2009 dollars.1'].dropna()
    gdp = pd.DataFrame(gdp)
    #gdp.reset_index(drop=True)
    gdp.columns = ['GDP']
    #gdp.index.rename('Quarter')
    #gdp[gdp['Quarter'] >= '2000q1']
    #gdp
    #gdp = gdp.iloc[214:].reset_index(drop=True)
    gdp = gdp.loc['2000q1':]
    gdp['GDP Diff'] = gdp['GDP'].diff(1)
    gdp = gdp.reset_index()
    gdp.columns = ['Quarter', 'GDP', 'GDP Diff'] 
    gdpRecession = gdp[(gdp['Quarter'] >= get_recession_start()) & (gdp['Quarter'] <= get_recession_end())]
    
    return list(gdpRecession.loc[gdpRecession['GDP'] == gdpRecession['GDP'].min(),'Quarter'])[0]
```


```python
get_recession_bottom()
```




    '2009q2'




```python
def convert_housing_data_to_quarters():
    '''Converts the housing data to quarters and returns it as mean 
    values in a dataframe. This dataframe should be a dataframe with
    columns for 2000q1 through 2016q3, and should have a multi-index
    in the shape of ["State","RegionName"].
    
    Note: Quarters are defined in the assignment description, they are
    not arbitrary three month periods.
    
    The resulting dataframe should have 67 columns, and 10,730 rows.
    '''
    
    housingPriceQuarter = pd.read_csv('City_Zhvi_AllHomes.csv')
    #2000~2015
    years = [year for year in range(2000,2016)]
    quarters = ['q1', 'q2','q3','q4']
    months = ['01','02','03','04','05','06','07','08','09','10','11','12']
    for year in years:
        YMs = []
        for month in months:
            #print(str(year) + '-' + month)
            YMs.append(str(year) + '-' + month)

        housingPriceQuarter[str(year) + quarters[0]] = housingPriceQuarter[YMs[0:3]].mean(axis = 1)
        housingPriceQuarter[str(year) + quarters[1]] = housingPriceQuarter[YMs[3:6]].mean(axis = 1)
        housingPriceQuarter[str(year) + quarters[2]] = housingPriceQuarter[YMs[6:9]].mean(axis = 1)
        housingPriceQuarter[str(year) + quarters[3]] = housingPriceQuarter[YMs[9:12]].mean(axis = 1)
    
    #2016~
    month2016 = ['2016-01','2016-02','2016-03','2016-04','2016-05','2016-06','2016-07','2016-08','2016-01']
    housingPriceQuarter[str(2016) + quarters[0]] = housingPriceQuarter[month2016[0:3]].mean(axis = 1)
    housingPriceQuarter[str(2016) + quarters[1]] = housingPriceQuarter[month2016[3:6]].mean(axis = 1)
    housingPriceQuarter[str(2016) + quarters[2]] = housingPriceQuarter[month2016[6:8]].mean(axis = 1)
    columns = ['State','RegionName','2000q1', '2000q2', '2000q3', '2000q4', '2001q1', '2001q2', '2001q3',
               '2001q4', '2002q1', '2002q2', '2002q3', '2002q4', '2003q1', '2003q2',
               '2003q3', '2003q4', '2004q1', '2004q2', '2004q3', '2004q4', '2005q1',
               '2005q2', '2005q3', '2005q4', '2006q1', '2006q2', '2006q3', '2006q4',
               '2007q1', '2007q2', '2007q3', '2007q4', '2008q1', '2008q2', '2008q3',
               '2008q4', '2009q1', '2009q2', '2009q3', '2009q4', '2010q1', '2010q2',
               '2010q3', '2010q4', '2011q1', '2011q2', '2011q3', '2011q4', '2012q1',
               '2012q2', '2012q3', '2012q4', '2013q1', '2013q2', '2013q3', '2013q4',
               '2014q1', '2014q2', '2014q3', '2014q4', '2015q1', '2015q2', '2015q3',
               '2015q4', '2016q1', '2016q2', '2016q3']

    housingPriceQuarterNew = housingPriceQuarter[columns].copy()
    housingPriceQuarterNew['State'] = housingPriceQuarterNew['State'].apply(lambda x: states[x])
    housingPriceQuarterNew = housingPriceQuarterNew.set_index(['State', 'RegionName'])
    return housingPriceQuarterNew
```


```python
convert_housing_data_to_quarters().shape
```




    (10730, 67)




```python
def run_ttest():
    '''First creates new data showing the decline or growth of housing prices
    between the recession start and the recession bottom. Then runs a ttest
    comparing the university town values to the non-university towns values, 
    return whether the alternative hypothesis (that the two groups are the same)
    is true or not as well as the p-value of the confidence. 
    
    Return the tuple (different, p, better) where different=True if the t-test is
    True at a p<0.01 (we reject the null hypothesis), or different=False if 
    otherwise (we cannot reject the null hypothesis). The variable p should
    be equal to the exact p value returned from scipy.stats.ttest_ind(). The
    value for better should be either "university town" or "non-university town"
    depending on which has a lower mean price ratio (which is equivilent to a
    reduced market loss).'''
    university_town = get_list_of_university_towns()
    university_town = university_town.set_index(['State', 'RegionName'])
    housingPriceQuarterNew = convert_housing_data_to_quarters()
    uniTownPrice = pd.merge(university_town, housingPriceQuarterNew, how = 'inner', left_index=True, right_index=True)
    nonUniTownPrice = housingPriceQuarterNew[~(housingPriceQuarterNew.index.isin(uniTownPrice.index))]
    uniTownPriceRecession = uniTownPrice.loc[:,get_quarter_before_recession_start():get_recession_bottom()].dropna()
    nonUniTownPriceRecession = nonUniTownPrice.loc[:,get_quarter_before_recession_start():get_recession_bottom()].dropna()
    
    uniTownPriceRecession['Market Loss'] = uniTownPriceRecession[get_quarter_before_recession_start()]/uniTownPriceRecession[get_recession_bottom()]
    nonUniTownPriceRecession['Market Loss'] = nonUniTownPriceRecession[get_quarter_before_recession_start()]/nonUniTownPriceRecession[get_recession_bottom()]
    t_stat, p_value = ttest_ind(uniTownPriceRecession["Market Loss"],
                                nonUniTownPriceRecession["Market Loss"])
    
    if p_value < 0.01:
        different = True
    else:
        different = False
    if t_stat < 0:
        better = "university town"
    else:
        better = "non-university town"
    return (different, p_value, better)
```


```python
run_ttest()
```




    (True, 0.002724063704761164, 'university town')




```python

```
