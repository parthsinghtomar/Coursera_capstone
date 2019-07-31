
<h1>Exploring the Neighborhoods in Alaska: Restaurants according to Classes of region</h1>
This is the final project of IBM Specialization courses (9) by Coursera.
Gathering information about it can take a long time than you think. 
That is why I decided to find better place for my research and decided in Alaska (USA). So this is my Capstone project.


<h1>Table of Contents</h1>
1. Discussion and Background of the Business Problem

2. Data Preparation

3. Explore regions in Singapore

4. Analyze Each Neighborhood

5. Cluster Neighborhoods

6. Examine Clusters

7. Results and Discussion

8. Conclusion


```python
import numpy as np # library to handle data in a vectorized manner

import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values
!pip install beautifulsoup4

!python3 -m pip install lxml

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

!conda install -c conda-forge folium=0.5.0 --yes # uncomment this line if you haven't completed the Foursquare API lab
import folium # map rendering library

print('Libraries imported.')
```

    Solving environment: done
    
    ## Package Plan ##
    
      environment location: /opt/conda/envs/Python36
    
      added / updated specs: 
        - geopy
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        ca-certificates-2019.6.16  |       hecc5488_0         145 KB  conda-forge
        geopy-1.20.0               |             py_0          57 KB  conda-forge
        geographiclib-1.49         |             py_0          32 KB  conda-forge
        openssl-1.1.1c             |       h516909a_0         2.1 MB  conda-forge
        certifi-2019.6.16          |           py36_1         149 KB  conda-forge
        ------------------------------------------------------------
                                               Total:         2.5 MB
    
    The following NEW packages will be INSTALLED:
    
        geographiclib:   1.49-py_0         conda-forge
        geopy:           1.20.0-py_0       conda-forge
    
    The following packages will be UPDATED:
    
        ca-certificates: 2019.5.15-0                   --> 2019.6.16-hecc5488_0 conda-forge
        certifi:         2019.6.16-py36_0              --> 2019.6.16-py36_1     conda-forge
    
    The following packages will be DOWNGRADED:
    
        openssl:         1.1.1c-h7b6447c_1             --> 1.1.1c-h516909a_0    conda-forge
    
    
    Downloading and Extracting Packages
    ca-certificates-2019 | 145 KB    | ##################################### | 100% 
    geopy-1.20.0         | 57 KB     | ##################################### | 100% 
    geographiclib-1.49   | 32 KB     | ##################################### | 100% 
    openssl-1.1.1c       | 2.1 MB    | ##################################### | 100% 
    certifi-2019.6.16    | 149 KB    | ##################################### | 100% 
    Preparing transaction: done
    Verifying transaction: done
    Executing transaction: done
    Requirement already satisfied: beautifulsoup4 in /opt/conda/envs/Python36/lib/python3.6/site-packages (4.7.1)
    Requirement already satisfied: soupsieve>=1.2 in /opt/conda/envs/Python36/lib/python3.6/site-packages (from beautifulsoup4) (1.7.1)
    Requirement already satisfied: lxml in /opt/conda/envs/Python36/lib/python3.6/site-packages (4.3.1)
    Solving environment: done
    
    ## Package Plan ##
    
      environment location: /opt/conda/envs/Python36
    
      added / updated specs: 
        - folium=0.5.0
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        branca-0.3.1               |             py_0          25 KB  conda-forge
        altair-3.1.0               |           py36_0         724 KB  conda-forge
        folium-0.5.0               |             py_0          45 KB  conda-forge
        vincent-0.4.4              |             py_1          28 KB  conda-forge
        ------------------------------------------------------------
                                               Total:         822 KB
    
    The following NEW packages will be INSTALLED:
    
        altair:  3.1.0-py36_0 conda-forge
        branca:  0.3.1-py_0   conda-forge
        folium:  0.5.0-py_0   conda-forge
        vincent: 0.4.4-py_1   conda-forge
    
    
    Downloading and Extracting Packages
    branca-0.3.1         | 25 KB     | ##################################### | 100% 
    altair-3.1.0         | 724 KB    | ##################################### | 100% 
    folium-0.5.0         | 45 KB     | ##################################### | 100% 
    vincent-0.4.4        | 28 KB     | ##################################### | 100% 
    Preparing transaction: done
    Verifying transaction: done
    Executing transaction: done
    Libraries imported.


<h1>1 Discussion and Background of the Business Problem</h1>
You won’t be surprised to hear that Alaska’s specialty is fresh fish. 
Salmon, halibut, and crab—plucked right from some of the world’s most pristine waters—may well be one of the main reasons you visit Alaska.

But here’s the rub: it’s hard to find a restaurant that serves memorable meals. And it’s expensive: even a basic dinner entrée can run you up to $30.

Imagine that we want to open new restaurant in Alaska. But we should find better borough for it. 
We should find it in Unified Home-Rule classed place due to the some reasons. (A “unified municipality” is an organized borough (unified, home-rule borough). 
A unified municipality is defined as such by the Local Boundary Commission in 3 AAC 110.990(1). 
The Alaska Constitution recognizes only two types of municipalities, cities and boroughs. 
The legislature consistently treats unified municipalities as boroughs. 
For example, State statutes utilize the same standards for incorporation of a borough as they do for incorporation of a unified municipality . 
By contrast, the legislature has established separate standards for incorporation of a city.)

<h1> 2. Data Preparation: </h1>


```python
from bs4 import BeautifulSoup
response_obj = requests.get('https://en.wikipedia.org/wiki/List_of_boroughs_and_census_areas_in_Alaska').text
print(type(response_obj))
```

    <class 'str'>



```python
soup = BeautifulSoup(response_obj,'html.parser')
#print (soup.prettify())
```


```python
Alaska_Table = soup.find('table', class_ = 'wikitable sortable')
```


```python
Name = []
Region = []
Area = []

for row in Alaska_Table.findAll("tr"):
    #print (row)    
    Ward = row.findAll('td')
    #print (len(Ward))
   
    if len(Ward)==10: #Only extract table body not heading
        Name.append(Ward[0].find(text=True).rstrip())
        Region.append(Ward[2].find(text=True).rstrip())
        Area.append(Ward[8].find(text=True).rstrip())
```


```python
Alaska_data=pd.DataFrame(Name,columns=['Name'])
Alaska_data['Region']=Region
Alaska_data['Area_SqKm']=Area
Alaska_data
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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>013</td>
      <td>Second</td>
      <td>6,988</td>
    </tr>
    <tr>
      <th>1</th>
      <td>020</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
    </tr>
    <tr>
      <th>2</th>
      <td>060</td>
      <td>Second</td>
      <td>505</td>
    </tr>
    <tr>
      <th>3</th>
      <td>068</td>
      <td>Home Rule</td>
      <td>12,750</td>
    </tr>
    <tr>
      <th>4</th>
      <td>090</td>
      <td>Second</td>
      <td>7,366</td>
    </tr>
    <tr>
      <th>5</th>
      <td>100</td>
      <td>Home Rule</td>
      <td>2,344</td>
    </tr>
    <tr>
      <th>6</th>
      <td>110</td>
      <td>Unified Home Rule</td>
      <td>2,716</td>
    </tr>
    <tr>
      <th>7</th>
      <td>122</td>
      <td>Second</td>
      <td>16,013</td>
    </tr>
    <tr>
      <th>8</th>
      <td>130</td>
      <td>Second</td>
      <td>4,840</td>
    </tr>
    <tr>
      <th>9</th>
      <td>150</td>
      <td>Second</td>
      <td>6,560</td>
    </tr>
    <tr>
      <th>10</th>
      <td>164</td>
      <td>Home Rule</td>
      <td>23,782</td>
    </tr>
    <tr>
      <th>11</th>
      <td>170</td>
      <td>Second</td>
      <td>24,682</td>
    </tr>
    <tr>
      <th>12</th>
      <td>185</td>
      <td>Home Rule</td>
      <td>88,817</td>
    </tr>
    <tr>
      <th>13</th>
      <td>188</td>
      <td>Home Rule</td>
      <td>35,898</td>
    </tr>
    <tr>
      <th>14</th>
      <td>195</td>
      <td>Home Rule</td>
      <td>3,829</td>
    </tr>
    <tr>
      <th>15</th>
      <td>220</td>
      <td>Unified Home Rule</td>
      <td>2,874</td>
    </tr>
    <tr>
      <th>16</th>
      <td>230</td>
      <td>First</td>
      <td>452</td>
    </tr>
    <tr>
      <th>17</th>
      <td>-</td>
      <td>-</td>
      <td>323,440</td>
    </tr>
    <tr>
      <th>18</th>
      <td>275</td>
      <td>Unified Home Rule</td>
      <td>2,570</td>
    </tr>
    <tr>
      <th>19</th>
      <td>282</td>
      <td>Home Rule</td>
      <td>7,650</td>
    </tr>
  </tbody>
</table>
</div>




```python
Alaska_data['Name']='Aleutians East Borough','Anchorage','Bristol Bay Borough','Denali Borough','Fairbanks North Star Borough','Haines Borough','Juneau','Kenai Peninsula Borough','Ketchikan Gateway Borough','Kodiak Island Borough','Lake and Peninsula Borough','Matanuska-Susitna Borough','North Slope Borough','Northwest Arctic Borough','Petersburg Borough','Sitka','Skagway','Unorganized Borough','Wrangell','Yakutat'
Alaska_data
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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aleutians East Borough</td>
      <td>Second</td>
      <td>6,988</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bristol Bay Borough</td>
      <td>Second</td>
      <td>505</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Denali Borough</td>
      <td>Home Rule</td>
      <td>12,750</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fairbanks North Star Borough</td>
      <td>Second</td>
      <td>7,366</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Haines Borough</td>
      <td>Home Rule</td>
      <td>2,344</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Juneau</td>
      <td>Unified Home Rule</td>
      <td>2,716</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Kenai Peninsula Borough</td>
      <td>Second</td>
      <td>16,013</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Ketchikan Gateway Borough</td>
      <td>Second</td>
      <td>4,840</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Kodiak Island Borough</td>
      <td>Second</td>
      <td>6,560</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Lake and Peninsula Borough</td>
      <td>Home Rule</td>
      <td>23,782</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Matanuska-Susitna Borough</td>
      <td>Second</td>
      <td>24,682</td>
    </tr>
    <tr>
      <th>12</th>
      <td>North Slope Borough</td>
      <td>Home Rule</td>
      <td>88,817</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Northwest Arctic Borough</td>
      <td>Home Rule</td>
      <td>35,898</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Petersburg Borough</td>
      <td>Home Rule</td>
      <td>3,829</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Sitka</td>
      <td>Unified Home Rule</td>
      <td>2,874</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Skagway</td>
      <td>First</td>
      <td>452</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Unorganized Borough</td>
      <td>-</td>
      <td>323,440</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Wrangell</td>
      <td>Unified Home Rule</td>
      <td>2,570</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Yakutat</td>
      <td>Home Rule</td>
      <td>7,650</td>
    </tr>
  </tbody>
</table>
</div>




```python
from geopy.geocoders import Nominatim
geolocator = Nominatim()
Alaska_data['Area_Name_Coord']= Alaska_data['Name'].apply(geolocator.geocode).apply(lambda x: (x.latitude, x.longitude))
```

    /opt/conda/envs/Python36/lib/python3.6/site-packages/ipykernel/__main__.py:2: DeprecationWarning: Using Nominatim with the default "geopy/1.20.0" `user_agent` is strongly discouraged, as it violates Nominatim's ToS https://operations.osmfoundation.org/policies/nominatim/ and may possibly cause 403 and 429 HTTP errors. Please specify a custom `user_agent` with `Nominatim(user_agent="my-application")` or by overriding the default `user_agent`: `geopy.geocoders.options.default_user_agent = "my-application"`. In geopy 2.0 this will become an exception.
      from ipykernel import kernelapp as app



```python
Alaska_data[['Latitude', 'Longitude']] = Alaska_data['Area_Name_Coord'].apply(pd.Series)
```


```python
Alaska_data.drop(['Area_Name_Coord'], axis=1, inplace=True)
Alaska_data
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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aleutians East Borough</td>
      <td>Second</td>
      <td>6,988</td>
      <td>55.051244</td>
      <td>-162.891689</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bristol Bay Borough</td>
      <td>Second</td>
      <td>505</td>
      <td>58.737034</td>
      <td>-156.875387</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Denali Borough</td>
      <td>Home Rule</td>
      <td>12,750</td>
      <td>63.878678</td>
      <td>-149.650166</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fairbanks North Star Borough</td>
      <td>Second</td>
      <td>7,366</td>
      <td>64.864904</td>
      <td>-146.775162</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Haines Borough</td>
      <td>Home Rule</td>
      <td>2,344</td>
      <td>59.083123</td>
      <td>-135.343057</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Juneau</td>
      <td>Unified Home Rule</td>
      <td>2,716</td>
      <td>58.301950</td>
      <td>-134.419734</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Kenai Peninsula Borough</td>
      <td>Second</td>
      <td>16,013</td>
      <td>60.096827</td>
      <td>-151.788033</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Ketchikan Gateway Borough</td>
      <td>Second</td>
      <td>4,840</td>
      <td>55.489748</td>
      <td>-131.011963</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Kodiak Island Borough</td>
      <td>Second</td>
      <td>6,560</td>
      <td>57.543377</td>
      <td>-153.357412</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Lake and Peninsula Borough</td>
      <td>Home Rule</td>
      <td>23,782</td>
      <td>58.327711</td>
      <td>-156.154765</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Matanuska-Susitna Borough</td>
      <td>Second</td>
      <td>24,682</td>
      <td>62.340248</td>
      <td>-149.479329</td>
    </tr>
    <tr>
      <th>12</th>
      <td>North Slope Borough</td>
      <td>Home Rule</td>
      <td>88,817</td>
      <td>69.533513</td>
      <td>-153.822068</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Northwest Arctic Borough</td>
      <td>Home Rule</td>
      <td>35,898</td>
      <td>67.238513</td>
      <td>-159.981635</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Petersburg Borough</td>
      <td>Home Rule</td>
      <td>3,829</td>
      <td>56.751221</td>
      <td>-133.458841</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Sitka</td>
      <td>Unified Home Rule</td>
      <td>2,874</td>
      <td>57.052497</td>
      <td>-135.337612</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Skagway</td>
      <td>First</td>
      <td>452</td>
      <td>59.622636</td>
      <td>-135.409437</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Unorganized Borough</td>
      <td>-</td>
      <td>323,440</td>
      <td>63.417431</td>
      <td>-157.671865</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Wrangell</td>
      <td>Unified Home Rule</td>
      <td>2,570</td>
      <td>56.204567</td>
      <td>-132.043255</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Yakutat</td>
      <td>Home Rule</td>
      <td>7,650</td>
      <td>59.572735</td>
      <td>-139.578312</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Alaska'

geolocator = Nominatim(user_agent="foursquare_agent")
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinate of Alaska areas are {}, {}.'.format(latitude, longitude))
```

    The geograpical coordinate of Alaska areas are 64.4459613, -149.680909.



```python
# create map of Toronto using latitude and longitude values
map_alaska= folium.Map(location=[latitude, longitude], zoom_start=3)

# add markers to map
for lat, lng , Name , Region in zip(Alaska_data['Latitude'], Alaska_data['Longitude'], Alaska_data['Name'], Alaska_data['Region']):
    label = '{}, {}'.format(Region, Name)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=10,
        popup=label,
        color='red',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_alaska)  
    
map_alaska
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfOTY1YTNhYWZjODE0NDI1M2FmODNlMmFmOWZiYzhhODIgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4MiA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4MicsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNjQuNDQ1OTYxMywtMTQ5LjY4MDkwOV0sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiAzLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbWF4Qm91bmRzOiBib3VuZHMsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBsYXllcnM6IFtdLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd29ybGRDb3B5SnVtcDogZmFsc2UsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3CiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIH0pOwogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl8xMDVhYmYyMWVkMzc0ZWY5YTcyNzM1OGNhZTJhMTczMiA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgJ2h0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nJywKICAgICAgICAgICAgICAgIHsKICAiYXR0cmlidXRpb24iOiBudWxsLAogICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwKICAibWF4Wm9vbSI6IDE4LAogICJtaW5ab29tIjogMSwKICAibm9XcmFwIjogZmFsc2UsCiAgInN1YmRvbWFpbnMiOiAiYWJjIgp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4Mik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOGVlMjEwMDI5YzRmNDBkOTkxZDYwZGY4M2JlYmJlMTYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs1NS4wNTEyNDQzLC0xNjIuODkxNjg5Ml0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJyZWQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4Mik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xZmQzM2FjZDdjMzM0ZjQzOTAwMzM1NDZhM2ZkMDY5YiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82ZmI2MjIwZDJjNzU0NzU1ODg5ZmU1OWU0M2Y0NzAwZSA9ICQoJzxkaXYgaWQ9Imh0bWxfNmZiNjIyMGQyYzc1NDc1NTg4OWZlNTllNDNmNDcwMGUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNlY29uZCwgQWxldXRpYW5zIEVhc3QgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMWZkMzNhY2Q3YzMzNGY0MzkwMDMzNTQ2YTNmZDA2OWIuc2V0Q29udGVudChodG1sXzZmYjYyMjBkMmM3NTQ3NTU4ODlmZTU5ZTQzZjQ3MDBlKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzhlZTIxMDAyOWM0ZjQwZDk5MWQ2MGRmODNiZWJiZTE2LmJpbmRQb3B1cChwb3B1cF8xZmQzM2FjZDdjMzM0ZjQzOTAwMzM1NDZhM2ZkMDY5Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81NDY5NjEzYzVlMGI0Njk5ODMxODMyMTVmNGYwMTM2YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzYxLjIxNjMxMjksLTE0OS44OTQ4NTIzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY2NjFhOWI3NTQxNDQwNzJhZTg3ZjM0MWQwOWE1NDRiID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQzNWE3M2M3MDc0MzRkNzQ4NmViN2Y4ODU0YjcxYzk0ID0gJCgnPGRpdiBpZD0iaHRtbF80MzVhNzNjNzA3NDM0ZDc0ODZlYjdmODg1NGI3MWM5NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VW5pZmllZCBIb21lIFJ1bGUsIEFuY2hvcmFnZTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNjY2MWE5Yjc1NDE0NDA3MmFlODdmMzQxZDA5YTU0NGIuc2V0Q29udGVudChodG1sXzQzNWE3M2M3MDc0MzRkNzQ4NmViN2Y4ODU0YjcxYzk0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzU0Njk2MTNjNWUwYjQ2OTk4MzE4MzIxNWY0ZjAxMzZjLmJpbmRQb3B1cChwb3B1cF82NjYxYTliNzU0MTQ0MDcyYWU4N2YzNDFkMDlhNTQ0Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNTFhNDJmM2ExZWI0OTM4YmQ4N2Y4OTgyYzQ5M2UwZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU4LjczNzAzNDEsLTE1Ni44NzUzODY3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg2NWQ3NDFlMjFhMTQ4MzQ4N2YxNzRkNmZlOGVlZDRjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzI4NTM1MWE3OGI4NzQ0ZTliMjNlZGZkOWMwZjVlNThiID0gJCgnPGRpdiBpZD0iaHRtbF8yODUzNTFhNzhiODc0NGU5YjIzZWRmZDljMGY1ZTU4YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Vjb25kLCBCcmlzdG9sIEJheSBCb3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84NjVkNzQxZTIxYTE0ODM0ODdmMTc0ZDZmZThlZWQ0Yy5zZXRDb250ZW50KGh0bWxfMjg1MzUxYTc4Yjg3NDRlOWIyM2VkZmQ5YzBmNWU1OGIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZjUxYTQyZjNhMWViNDkzOGJkODdmODk4MmM0OTNlMGQuYmluZFBvcHVwKHBvcHVwXzg2NWQ3NDFlMjFhMTQ4MzQ4N2YxNzRkNmZlOGVlZDRjKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzY1ZWViMjg0ZDU3OTQxNzE5YWEzYTlhYTgwZjNhM2UyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNjMuODc4Njc4LC0xNDkuNjUwMTY2XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzgzOWI2OGQ3ZjA3MjQ5OWI4ZjZkYmRjM2E5NDIxZjM5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzUxZTk3OWVlNDFhMjRhZWE5MjRiMmUwOWQ1MDg5ODhiID0gJCgnPGRpdiBpZD0iaHRtbF81MWU5NzllZTQxYTI0YWVhOTI0YjJlMDlkNTA4OTg4YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SG9tZSBSdWxlLCBEZW5hbGkgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODM5YjY4ZDdmMDcyNDk5YjhmNmRiZGMzYTk0MjFmMzkuc2V0Q29udGVudChodG1sXzUxZTk3OWVlNDFhMjRhZWE5MjRiMmUwOWQ1MDg5ODhiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzY1ZWViMjg0ZDU3OTQxNzE5YWEzYTlhYTgwZjNhM2UyLmJpbmRQb3B1cChwb3B1cF84MzliNjhkN2YwNzI0OTliOGY2ZGJkYzNhOTQyMWYzOSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xYWZlNDU5YzQxNjM0MDhjOTA1MDk0MmY3ZWMwNzhjOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzY0Ljg2NDkwMzksLTE0Ni43NzUxNjE5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY2MmFiNjZiZGM3MTQ2ZDFiYmRlMjU2OTViMWUxMjNjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2RlZDEyYzg1Njk5YTQ3YzY4MmVmNTRkZjUyNGE5NGQ1ID0gJCgnPGRpdiBpZD0iaHRtbF9kZWQxMmM4NTY5OWE0N2M2ODJlZjU0ZGY1MjRhOTRkNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Vjb25kLCBGYWlyYmFua3MgTm9ydGggU3RhciBCb3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82NjJhYjY2YmRjNzE0NmQxYmJkZTI1Njk1YjFlMTIzYy5zZXRDb250ZW50KGh0bWxfZGVkMTJjODU2OTlhNDdjNjgyZWY1NGRmNTI0YTk0ZDUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMWFmZTQ1OWM0MTYzNDA4YzkwNTA5NDJmN2VjMDc4YzkuYmluZFBvcHVwKHBvcHVwXzY2MmFiNjZiZGM3MTQ2ZDFiYmRlMjU2OTViMWUxMjNjKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I4N2I0N2E0MTExNTRiYjQ5ZjIwMjQ5ODA0MGQxNjBkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTkuMDgzMTIzMiwtMTM1LjM0MzA1NzNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAicmVkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOTY1YTNhYWZjODE0NDI1M2FmODNlMmFmOWZiYzhhODIpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOWM1YmJkMGU1MGM0NDVlNTk0OGJhNTc1N2EwM2QxZmYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzE0YjBlNTJlYTdhNGViYWI1ZjM1OWJlOTZkMmQwZGIgPSAkKCc8ZGl2IGlkPSJodG1sXzMxNGIwZTUyZWE3YTRlYmFiNWYzNTliZTk2ZDJkMGRiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ib21lIFJ1bGUsIEhhaW5lcyBCb3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85YzViYmQwZTUwYzQ0NWU1OTQ4YmE1NzU3YTAzZDFmZi5zZXRDb250ZW50KGh0bWxfMzE0YjBlNTJlYTdhNGViYWI1ZjM1OWJlOTZkMmQwZGIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjg3YjQ3YTQxMTE1NGJiNDlmMjAyNDk4MDQwZDE2MGQuYmluZFBvcHVwKHBvcHVwXzljNWJiZDBlNTBjNDQ1ZTU5NDhiYTU3NTdhMDNkMWZmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzg2ZjBkYzliZmVhNDQ2ZTk5N2QzYTdmMjgxMmJhNmJkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTguMzAxOTQ5NiwtMTM0LjQxOTczNF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJyZWQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4Mik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hZDYxYWYyYWI3ZDM0ZGU3YWJjNmI3NGIyYzgzYTcyYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8xMDA5YjhiODc3OTg0MzljOWUyNTY0OGJhZjE0N2ZmNCA9ICQoJzxkaXYgaWQ9Imh0bWxfMTAwOWI4Yjg3Nzk4NDM5YzllMjU2NDhiYWYxNDdmZjQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlVuaWZpZWQgSG9tZSBSdWxlLCBKdW5lYXU8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2FkNjFhZjJhYjdkMzRkZTdhYmM2Yjc0YjJjODNhNzJjLnNldENvbnRlbnQoaHRtbF8xMDA5YjhiODc3OTg0MzljOWUyNTY0OGJhZjE0N2ZmNCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl84NmYwZGM5YmZlYTQ0NmU5OTdkM2E3ZjI4MTJiYTZiZC5iaW5kUG9wdXAocG9wdXBfYWQ2MWFmMmFiN2QzNGRlN2FiYzZiNzRiMmM4M2E3MmMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMjI4NzA4ODFmZmQ0NDJjNGExMTFjZjJjZmRmNWQ4OTAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs2MC4wOTY4MjcyLC0xNTEuNzg4MDMzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzgwODY2YTU4ZWQwNDQyNDM4ZTkwMzE3MDM1ZDg5Njk0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzY3MTU4ZWZjZjU5YzRlNmFhODAwOTRkZDg3NTY4NTA5ID0gJCgnPGRpdiBpZD0iaHRtbF82NzE1OGVmY2Y1OWM0ZTZhYTgwMDk0ZGQ4NzU2ODUwOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Vjb25kLCBLZW5haSBQZW5pbnN1bGEgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfODA4NjZhNThlZDA0NDI0MzhlOTAzMTcwMzVkODk2OTQuc2V0Q29udGVudChodG1sXzY3MTU4ZWZjZjU5YzRlNmFhODAwOTRkZDg3NTY4NTA5KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzIyODcwODgxZmZkNDQyYzRhMTExY2YyY2ZkZjVkODkwLmJpbmRQb3B1cChwb3B1cF84MDg2NmE1OGVkMDQ0MjQzOGU5MDMxNzAzNWQ4OTY5NCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wZDMwODM5Nzk0Y2Y0YjNkOWYzMDgyMjBlN2U5NzcwYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU1LjQ4OTc0OCwtMTMxLjAxMTk2M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJyZWQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4Mik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84YmZiNDg4NThhMTM0Y2E5YTYwOWU2OWUzMjA3YmZiMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81YjA3OThlNTA1YzU0Zjk0OGQyMDFjYzFmMmNmZWU2NCA9ICQoJzxkaXYgaWQ9Imh0bWxfNWIwNzk4ZTUwNWM1NGY5NDhkMjAxY2MxZjJjZmVlNjQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNlY29uZCwgS2V0Y2hpa2FuIEdhdGV3YXkgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOGJmYjQ4ODU4YTEzNGNhOWE2MDllNjllMzIwN2JmYjIuc2V0Q29udGVudChodG1sXzViMDc5OGU1MDVjNTRmOTQ4ZDIwMWNjMWYyY2ZlZTY0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzBkMzA4Mzk3OTRjZjRiM2Q5ZjMwODIyMGU3ZTk3NzBjLmJpbmRQb3B1cChwb3B1cF84YmZiNDg4NThhMTM0Y2E5YTYwOWU2OWUzMjA3YmZiMik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9iM2NhYzI5ZGRiZWI0YjYxOGYzNTc4ODQyZjg2YTUwMSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU3LjU0MzM3NjUsLTE1My4zNTc0MTI0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U2M2E3MTkyNTFhOTQ4MTJhNjgzYWFiNDk3YmMyYTM5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2I1MjIwMjFlMGRkNjQ1MDdiODk5MTY5MWUyOGEzMGE3ID0gJCgnPGRpdiBpZD0iaHRtbF9iNTIyMDIxZTBkZDY0NTA3Yjg5OTE2OTFlMjhhMzBhNyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Vjb25kLCBLb2RpYWsgSXNsYW5kIEJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2U2M2E3MTkyNTFhOTQ4MTJhNjgzYWFiNDk3YmMyYTM5LnNldENvbnRlbnQoaHRtbF9iNTIyMDIxZTBkZDY0NTA3Yjg5OTE2OTFlMjhhMzBhNyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iM2NhYzI5ZGRiZWI0YjYxOGYzNTc4ODQyZjg2YTUwMS5iaW5kUG9wdXAocG9wdXBfZTYzYTcxOTI1MWE5NDgxMmE2ODNhYWI0OTdiYzJhMzkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfM2ZjODU3MmNiM2FjNGUzZGI0OTc1ZDlhZjFjN2I5ZWEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs1OC4zMjc3MTA5LC0xNTYuMTU0NzY1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2NiY2NkZmRlZDBhMDRkNjM5NzU4MGFjMGVlMmY2ZWJmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzgyNDZkZjM3NjU4ODQzMzc4OGFiZDgyYjhmZWQ3NjA4ID0gJCgnPGRpdiBpZD0iaHRtbF84MjQ2ZGYzNzY1ODg0MzM3ODhhYmQ4MmI4ZmVkNzYwOCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SG9tZSBSdWxlLCBMYWtlIGFuZCBQZW5pbnN1bGEgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfY2JjY2RmZGVkMGEwNGQ2Mzk3NTgwYWMwZWUyZjZlYmYuc2V0Q29udGVudChodG1sXzgyNDZkZjM3NjU4ODQzMzc4OGFiZDgyYjhmZWQ3NjA4KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzNmYzg1NzJjYjNhYzRlM2RiNDk3NWQ5YWYxYzdiOWVhLmJpbmRQb3B1cChwb3B1cF9jYmNjZGZkZWQwYTA0ZDYzOTc1ODBhYzBlZTJmNmViZik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81YTA0NWQ2OWRhMGU0NGEzODQ3YzA4Y2RmNDM2NzE3NyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzYyLjM0MDI0ODEsLTE0OS40NzkzMjg4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzQyYWIyMjYzYTNlYTRlMWJhMWJkZWE3MjYyZDlmOTM4ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2FiNjBjNjEzMDZiNDQxYmU5Mjk4ZmU2YjA3NDNlOTk3ID0gJCgnPGRpdiBpZD0iaHRtbF9hYjYwYzYxMzA2YjQ0MWJlOTI5OGZlNmIwNzQzZTk5NyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2Vjb25kLCBNYXRhbnVza2EtU3VzaXRuYSBCb3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF80MmFiMjI2M2EzZWE0ZTFiYTFiZGVhNzI2MmQ5ZjkzOC5zZXRDb250ZW50KGh0bWxfYWI2MGM2MTMwNmI0NDFiZTkyOThmZTZiMDc0M2U5OTcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNWEwNDVkNjlkYTBlNDRhMzg0N2MwOGNkZjQzNjcxNzcuYmluZFBvcHVwKHBvcHVwXzQyYWIyMjYzYTNlYTRlMWJhMWJkZWE3MjYyZDlmOTM4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2QyMzAwZmMzMWY1ODQ1YjA4NjNmMzE0MjM1MTExODhiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNjkuNTMzNTEyOSwtMTUzLjgyMjA2ODFdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAicmVkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOTY1YTNhYWZjODE0NDI1M2FmODNlMmFmOWZiYzhhODIpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMzcwZWYyMzYxMGQ0NDdhMjlkNTgyN2JiMmVhMTdkYzQgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzViZmUxMDFmNWZhNDY0Njg3MTkzOTMzNTY2MzI3OGQgPSAkKCc8ZGl2IGlkPSJodG1sXzM1YmZlMTAxZjVmYTQ2NDY4NzE5MzkzMzU2NjMyNzhkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ib21lIFJ1bGUsIE5vcnRoIFNsb3BlIEJvcm91Z2g8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzM3MGVmMjM2MTBkNDQ3YTI5ZDU4MjdiYjJlYTE3ZGM0LnNldENvbnRlbnQoaHRtbF8zNWJmZTEwMWY1ZmE0NjQ2ODcxOTM5MzM1NjYzMjc4ZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9kMjMwMGZjMzFmNTg0NWIwODYzZjMxNDIzNTExMTg4Yi5iaW5kUG9wdXAocG9wdXBfMzcwZWYyMzYxMGQ0NDdhMjlkNTgyN2JiMmVhMTdkYzQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNzZlOWQxOTliYjAxNGM2YmI5ZjYyMGI2NTdmYjlkNTEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs2Ny4yMzg1MTI3LC0xNTkuOTgxNjM0N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJyZWQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4Mik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9kZjJiMDM3OGM4Mzc0YTliOTdmY2ZmYTY4YzZiMDdjMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kOTIyNGNhNmFjZDU0Y2ZmYWE4ODQyNjk1OWYzZjlkMiA9ICQoJzxkaXYgaWQ9Imh0bWxfZDkyMjRjYTZhY2Q1NGNmZmFhODg0MjY5NTlmM2Y5ZDIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhvbWUgUnVsZSwgTm9ydGh3ZXN0IEFyY3RpYyBCb3JvdWdoPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9kZjJiMDM3OGM4Mzc0YTliOTdmY2ZmYTY4YzZiMDdjMy5zZXRDb250ZW50KGh0bWxfZDkyMjRjYTZhY2Q1NGNmZmFhODg0MjY5NTlmM2Y5ZDIpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNzZlOWQxOTliYjAxNGM2YmI5ZjYyMGI2NTdmYjlkNTEuYmluZFBvcHVwKHBvcHVwX2RmMmIwMzc4YzgzNzRhOWI5N2ZjZmZhNjhjNmIwN2MzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzc0Zjk3ZTA5NjRlZTQ0Zjc4YmNhY2Y2YjRjZTFmOWUxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTYuNzUxMjIwNywtMTMzLjQ1ODg0MDZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAicmVkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOTY1YTNhYWZjODE0NDI1M2FmODNlMmFmOWZiYzhhODIpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNWUzZGRhM2U5YTMwNDE3MThiZTk5OTQ1MWUyNWFjZjMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZWZhZjE0MTRjZDcwNGNlMDkzMTkwZDkyMGUxZGJhZGMgPSAkKCc8ZGl2IGlkPSJodG1sX2VmYWYxNDE0Y2Q3MDRjZTA5MzE5MGQ5MjBlMWRiYWRjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ib21lIFJ1bGUsIFBldGVyc2J1cmcgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNWUzZGRhM2U5YTMwNDE3MThiZTk5OTQ1MWUyNWFjZjMuc2V0Q29udGVudChodG1sX2VmYWYxNDE0Y2Q3MDRjZTA5MzE5MGQ5MjBlMWRiYWRjKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzc0Zjk3ZTA5NjRlZTQ0Zjc4YmNhY2Y2YjRjZTFmOWUxLmJpbmRQb3B1cChwb3B1cF81ZTNkZGEzZTlhMzA0MTcxOGJlOTk5NDUxZTI1YWNmMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xMTUzNGVmNTVkNTY0M2MzYjFjNmYzMzJhZDY2OWIyNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU3LjA1MjQ5NzMsLTEzNS4zMzc2MTI0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA1YTE5ZGJjY2U4YjQ4Njg4MGMzOGFkMzZiMjU2MjJhID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzk1YjRiZGQzYWJiMTQ5YWI4MTlhMDVlNDZmN2IxNDBlID0gJCgnPGRpdiBpZD0iaHRtbF85NWI0YmRkM2FiYjE0OWFiODE5YTA1ZTQ2ZjdiMTQwZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VW5pZmllZCBIb21lIFJ1bGUsIFNpdGthPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wNWExOWRiY2NlOGI0ODY4ODBjMzhhZDM2YjI1NjIyYS5zZXRDb250ZW50KGh0bWxfOTViNGJkZDNhYmIxNDlhYjgxOWEwNWU0NmY3YjE0MGUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMTE1MzRlZjU1ZDU2NDNjM2IxYzZmMzMyYWQ2NjliMjQuYmluZFBvcHVwKHBvcHVwXzA1YTE5ZGJjY2U4YjQ4Njg4MGMzOGFkMzZiMjU2MjJhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2I3NDk3ZWU1MTQxZDQwZjY4MGU5NjM3ZDVkMzUxNDY4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTkuNjIyNjM2MiwtMTM1LjQwOTQzNjhdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAicmVkIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogMTAsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfOTY1YTNhYWZjODE0NDI1M2FmODNlMmFmOWZiYzhhODIpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfM2E0YjA1OWZkNDJkNDI2MGEwMGIxOTUxMWQ0N2VkMjcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfN2I4NjU0YmFkZjgzNGVmNTgzM2Q4NTUyYzNiYzYzZGEgPSAkKCc8ZGl2IGlkPSJodG1sXzdiODY1NGJhZGY4MzRlZjU4MzNkODU1MmMzYmM2M2RhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5GaXJzdCwgU2thZ3dheTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfM2E0YjA1OWZkNDJkNDI2MGEwMGIxOTUxMWQ0N2VkMjcuc2V0Q29udGVudChodG1sXzdiODY1NGJhZGY4MzRlZjU4MzNkODU1MmMzYmM2M2RhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2I3NDk3ZWU1MTQxZDQwZjY4MGU5NjM3ZDVkMzUxNDY4LmJpbmRQb3B1cChwb3B1cF8zYTRiMDU5ZmQ0MmQ0MjYwYTAwYjE5NTExZDQ3ZWQyNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82ZjRkMzVkYmNhOGE0MjYxODQwZTA0MDQzMDhlYjU5NiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzYzLjQxNzQzMTA1LC0xNTcuNjcxODY1MDQ4NDU3XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2M5ODdmM2M3Nzk4ZDQ1NjY5ZDI5YWFhOTBhMGE1NWUwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzgyNDY3ODU2ZDM1YzRiYmZiNDA2MjJhYTRmZmYxMDMyID0gJCgnPGRpdiBpZD0iaHRtbF84MjQ2Nzg1NmQzNWM0YmJmYjQwNjIyYWE0ZmZmMTAzMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+LSwgVW5vcmdhbml6ZWQgQm9yb3VnaDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYzk4N2YzYzc3OThkNDU2NjlkMjlhYWE5MGEwYTU1ZTAuc2V0Q29udGVudChodG1sXzgyNDY3ODU2ZDM1YzRiYmZiNDA2MjJhYTRmZmYxMDMyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzZmNGQzNWRiY2E4YTQyNjE4NDBlMDQwNDMwOGViNTk2LmJpbmRQb3B1cChwb3B1cF9jOTg3ZjNjNzc5OGQ0NTY2OWQyOWFhYTkwYTBhNTVlMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80ZGU5ZDJiOWNiYmU0NzA0OTE5M2U4YWJkNTVhNDBjYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU2LjIwNDU2NzMsLTEzMi4wNDMyNTUyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogInJlZCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDEwLAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzk2NWEzYWFmYzgxNDQyNTNhZjgzZTJhZjlmYmM4YTgyKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzY0MDQwYjUzNzk0ODRlZjliMDEyMzMxODM2OTk0NjlkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzk4ZmUxZjcwMDRkMDQ5ZDhiMWM3ZGE3OTZhMzA2MWYxID0gJCgnPGRpdiBpZD0iaHRtbF85OGZlMWY3MDA0ZDA0OWQ4YjFjN2RhNzk2YTMwNjFmMSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VW5pZmllZCBIb21lIFJ1bGUsIFdyYW5nZWxsPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82NDA0MGI1Mzc5NDg0ZWY5YjAxMjMzMTgzNjk5NDY5ZC5zZXRDb250ZW50KGh0bWxfOThmZTFmNzAwNGQwNDlkOGIxYzdkYTc5NmEzMDYxZjEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNGRlOWQyYjljYmJlNDcwNDkxOTNlOGFiZDU1YTQwY2MuYmluZFBvcHVwKHBvcHVwXzY0MDQwYjUzNzk0ODRlZjliMDEyMzMxODM2OTk0NjlkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzIxZWMwYjFjM2MzMzRmOTU4OTUxYmQwZGFkZTc2ZTdjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTkuNTcyNzM0NSwtMTM5LjU3ODMxMjQzODc4MV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJyZWQiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiAxMCwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF85NjVhM2FhZmM4MTQ0MjUzYWY4M2UyYWY5ZmJjOGE4Mik7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85MThiOTE5ZGUxZGI0MTcyYTI3NWEzYTYyNGY1YTZjMiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81NDViZjNkMzNiYmU0M2JjOGJmNTViNjQ3YmE5MmYwMSA9ICQoJzxkaXYgaWQ9Imh0bWxfNTQ1YmYzZDMzYmJlNDNiYzhiZjU1YjY0N2JhOTJmMDEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhvbWUgUnVsZSwgWWFrdXRhdDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOTE4YjkxOWRlMWRiNDE3MmEyNzVhM2E2MjRmNWE2YzIuc2V0Q29udGVudChodG1sXzU0NWJmM2QzM2JiZTQzYmM4YmY1NWI2NDdiYTkyZjAxKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzIxZWMwYjFjM2MzMzRmOTU4OTUxYmQwZGFkZTc2ZTdjLmJpbmRQb3B1cChwb3B1cF85MThiOTE5ZGUxZGI0MTcyYTI3NWEzYTYyNGY1YTZjMik7CgogICAgICAgICAgICAKICAgICAgICAKPC9zY3JpcHQ+" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



<h1> 3. Explore regions in Alaska  </h1>


```python
CLIENT_ID = 'BURPSDHBL0XRO133LCY4VBS5QUUGYFT13GKQXWYNEGIBQVLA' # your Foursquare ID
CLIENT_SECRET = 'TLE5JH52YACWQORB2GDVM22OIYZTTUHI0GX003C2WJYXGGQL' # your Foursquare Secret
VERSION = '20190729' # Foursquare API version

print('Your credentails:')
print('CLIENT_ID: ' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
```

    Your credentails:
    CLIENT_ID: BURPSDHBL0XRO133LCY4VBS5QUUGYFT13GKQXWYNEGIBQVLA
    CLIENT_SECRET:TLE5JH52YACWQORB2GDVM22OIYZTTUHI0GX003C2WJYXGGQL


<h3>Let's create a function to get the venues to all the regions in Alaska</h3>
Exploring the regions

1.Create the get request url (Foursquare ID and Secret are necessary) 1.a. Number of Venues we will look for is 100 2.a. Radius of Search Would be 100 m.

2.Create a json from the request object (Need requests Module)

3.Create the lists Containing all the information

4.From the lists create the dataframe.


```python
def getNearbyVenues(names, latitudes, longitudes, radius=500):
    LIMIT = 100 # limit of number of venues returned by Foursquare API

    venues_list=[]
    for name, lat, lng in zip(names, latitudes, longitudes):
        print(name)
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            LIMIT)
            
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        
        # return only relevant information for each nearby venue
        venues_list.append([(
            name, 
            lat, 
            lng, 
            v['venue']['name'], 
            v['venue']['location']['lat'], 
            v['venue']['location']['lng'],  
            v['venue']['categories'][0]['name']) for v in results])

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['Neighborhood', 
                  'Neighborhood_Latitude', 
                  'Neighborhood_Longitude', 
                  'Venue', 
                  'Venue_Latitude', 
                  'Venue_Longitude', 
                  'Venue_Category']
    
    return(nearby_venues)
```


```python
alaska_Venues = getNearbyVenues(names=Alaska_data['Name'],
                                   latitudes=Alaska_data['Latitude'],
                                   longitudes=Alaska_data['Longitude']
                                  )
```

    Aleutians East Borough
    Anchorage
    Bristol Bay Borough
    Denali Borough
    Fairbanks North Star Borough
    Haines Borough
    Juneau
    Kenai Peninsula Borough
    Ketchikan Gateway Borough
    Kodiak Island Borough
    Lake and Peninsula Borough
    Matanuska-Susitna Borough
    North Slope Borough
    Northwest Arctic Borough
    Petersburg Borough
    Sitka
    Skagway
    Unorganized Borough
    Wrangell
    Yakutat



```python
print(alaska_Venues.shape)
alaska_Venues.head()
```

    (152, 7)





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
      <th>Neighborhood</th>
      <th>Neighborhood_Latitude</th>
      <th>Neighborhood_Longitude</th>
      <th>Venue</th>
      <th>Venue_Latitude</th>
      <th>Venue_Longitude</th>
      <th>Venue_Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Anchorage</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Glacier BrewHouse</td>
      <td>61.217719</td>
      <td>-149.896839</td>
      <td>Brewery</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Humpy's Great Alaskan Alehouse</td>
      <td>61.216427</td>
      <td>-149.894146</td>
      <td>Bar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anchorage</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Alaska Center for the Performing Arts</td>
      <td>61.216989</td>
      <td>-149.893718</td>
      <td>Performing Arts Venue</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Anchorage</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Wild Scoops</td>
      <td>61.217839</td>
      <td>-149.891494</td>
      <td>Ice Cream Shop</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anchorage</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Orso</td>
      <td>61.217657</td>
      <td>-149.895940</td>
      <td>Seafood Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python
Alaska_Venues_only_restaurant = alaska_Venues[alaska_Venues['Venue_Category']\
                                                          .str.contains('Restaurant')].reset_index(drop=True)
Alaska_Venues_only_restaurant.index = np.arange(1, len(Alaska_Venues_only_restaurant)+1)
print ("Shape of the Data-Frame with Venue Category only Restaurant: ", Alaska_Venues_only_restaurant.shape)
Alaska_Venues_only_restaurant.head()

alaska_restuarant = Alaska_data

# merge alaska_grouped with singapore to add latitude/longitude for each neighborhood
alaska_restuarant = alaska_restuarant.join(Alaska_Venues_only_restaurant.set_index('Neighborhood'), on='Name')

alaska_restuarant.head()
```

    Shape of the Data-Frame with Venue Category only Restaurant:  (29, 7)





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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Neighborhood_Latitude</th>
      <th>Neighborhood_Longitude</th>
      <th>Venue</th>
      <th>Venue_Latitude</th>
      <th>Venue_Longitude</th>
      <th>Venue_Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aleutians East Borough</td>
      <td>Second</td>
      <td>6,988</td>
      <td>55.051244</td>
      <td>-162.891689</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Orso</td>
      <td>61.217657</td>
      <td>-149.895940</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Ginger</td>
      <td>61.217682</td>
      <td>-149.890564</td>
      <td>Asian Restaurant</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Crow's Nest</td>
      <td>61.217838</td>
      <td>-149.899718</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Bubbly Mermaid Oyster Bar</td>
      <td>61.218149</td>
      <td>-149.889501</td>
      <td>Seafood Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python
Central_Alaska_restuarent = alaska_restuarant[alaska_restuarant['Region'] == 'Unified Home Rule'].reset_index(drop=True)
Central_Alaska_restuarent.head()
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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Neighborhood_Latitude</th>
      <th>Neighborhood_Longitude</th>
      <th>Venue</th>
      <th>Venue_Latitude</th>
      <th>Venue_Longitude</th>
      <th>Venue_Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Orso</td>
      <td>61.217657</td>
      <td>-149.895940</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Ginger</td>
      <td>61.217682</td>
      <td>-149.890564</td>
      <td>Asian Restaurant</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Crow's Nest</td>
      <td>61.217838</td>
      <td>-149.899718</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Bubbly Mermaid Oyster Bar</td>
      <td>61.218149</td>
      <td>-149.889501</td>
      <td>Seafood Restaurant</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>Pangaea Restaurant and Lounge</td>
      <td>61.216479</td>
      <td>-149.891934</td>
      <td>Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python
### Number of Unique Categories in the Dataframe 
print('There are {} unique categories.'.format(len(Central_Alaska_restuarent['Venue_Category'].unique())))
```

    There are 12 unique categories.



```python
print (Central_Alaska_restuarent['Venue_Category'].value_counts())
```

    Seafood Restaurant           7
    Restaurant                   4
    American Restaurant          4
    Sushi Restaurant             4
    Mexican Restaurant           2
    Asian Restaurant             2
    Thai Restaurant              2
    Mediterranean Restaurant     1
    Theme Restaurant             1
    Japanese Restaurant          1
    Cajun / Creole Restaurant    1
    Name: Venue_Category, dtype: int64



```python
# create a dataframe of top 10 categories
alaska_Central_Venues_Top10 = Central_Alaska_restuarent['Venue_Category'].value_counts()[0:10].to_frame(name='frequency')
alaska_Central_Venues_Top10=alaska_Central_Venues_Top10.reset_index()
#Singapore_Venues_Top10

alaska_Central_Venues_Top10.rename(index=str, columns={"index": "Venue_Category", "frequency": "Frequency"}, inplace=True)
alaska_Central_Venues_Top10
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
      <th>Venue_Category</th>
      <th>Frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Seafood Restaurant</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Restaurant</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>American Restaurant</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sushi Restaurant</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Mexican Restaurant</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Asian Restaurant</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Thai Restaurant</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Mediterranean Restaurant</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Theme Restaurant</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Japanese Restaurant</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
import seaborn as sns
from matplotlib import pyplot as plt
fig = plt.figure(figsize=(18,7))
s=sns.barplot(x="Venue_Category", y="Frequency", data=alaska_Central_Venues_Top10)
s.set_xticklabels(s.get_xticklabels(), rotation=30)
plt.title('10 Most Frequently Occuring Venues in Central Alaska', fontsize=15)
plt.xlabel("Venue Category", fontsize=15)
plt.ylabel ("Frequency", fontsize=15)
plt.savefig("Most_Freq_Venues.png", dpi=300)
plt.show()
```


![png](output_27_0.png)


<h1> 4. Analyze Each Neighborhood </h1>


```python
# create a dataframe of top 10 categories
alaska_Central_Venues_Top10 = Central_Alaska_restuarent['Venue_Category'].value_counts()[0:10].to_frame(name='frequency')
alaska_Central_Venues_Top10=alaska_Central_Venues_Top10.reset_index()
#Alaska_Venues_Top10

alaska_Central_Venues_Top10.rename(index=str, columns={"index": "Venue_Category", "frequency": "Frequency"}, inplace=True)
#alaska_Central_Venues_Top10

import seaborn as sns
from matplotlib import pyplot as plt
fig = plt.figure(figsize=(18,7))
s=sns.barplot(x="Venue_Category", y="Frequency", data=alaska_Central_Venues_Top10)
s.set_xticklabels(s.get_xticklabels(), rotation=30)
plt.title('10 Most Frequently Occuring Venues in Central Alaska', fontsize=15)
plt.xlabel("Venue Category", fontsize=15)
plt.ylabel ("Frequency", fontsize=15)
plt.savefig("Most_Freq_Venues.png", dpi=300)
plt.show()
```


![png](output_29_0.png)



```python
# one hot encoding
alaska_onehot = pd.get_dummies(Alaska_Venues_only_restaurant[['Venue_Category']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
alaska_onehot['Neighborhood'] = Alaska_Venues_only_restaurant['Neighborhood'] 

# move neighborhood column to the first column
fixed_columns = [alaska_onehot.columns[-1]] + list(alaska_onehot.columns[:-1])
alaska_onehot = alaska_onehot[fixed_columns]

alaska_onehot.head()
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
      <th>Neighborhood</th>
      <th>American Restaurant</th>
      <th>Asian Restaurant</th>
      <th>Cajun / Creole Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Mediterranean Restaurant</th>
      <th>Mexican Restaurant</th>
      <th>Restaurant</th>
      <th>Seafood Restaurant</th>
      <th>Sushi Restaurant</th>
      <th>Thai Restaurant</th>
      <th>Theme Restaurant</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anchorage</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Anchorage</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Anchorage</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Anchorage</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
alaska_onehot.shape
```




    (29, 12)




```python
alaska_grouped = alaska_onehot.groupby('Neighborhood').mean().reset_index()
alaska_grouped
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
      <th>Neighborhood</th>
      <th>American Restaurant</th>
      <th>Asian Restaurant</th>
      <th>Cajun / Creole Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Mediterranean Restaurant</th>
      <th>Mexican Restaurant</th>
      <th>Restaurant</th>
      <th>Seafood Restaurant</th>
      <th>Sushi Restaurant</th>
      <th>Thai Restaurant</th>
      <th>Theme Restaurant</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Anchorage</td>
      <td>0.117647</td>
      <td>0.058824</td>
      <td>0.058824</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.058824</td>
      <td>0.117647</td>
      <td>0.235294</td>
      <td>0.176471</td>
      <td>0.117647</td>
      <td>0.058824</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Juneau</td>
      <td>0.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sitka</td>
      <td>0.100000</td>
      <td>0.100000</td>
      <td>0.000000</td>
      <td>0.1</td>
      <td>0.1</td>
      <td>0.100000</td>
      <td>0.200000</td>
      <td>0.300000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



<h1>5. Cluster Neighborhoods</h1>


```python
alaska_grouped.shape
```




    (3, 12)




```python
num_top_venues = 3
for hood in alaska_grouped['Neighborhood']:
    print("----"+hood+"----")
    temp = alaska_grouped[alaska_grouped['Neighborhood'] == hood].T.reset_index()
    temp.columns = ['venue','freq']
    temp = temp.iloc[1:]
    temp['freq'] = temp['freq'].astype(float)
    temp = temp.round({'freq': 2})
    print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
    print('\n')
```

    ----Anchorage----
                     venue  freq
    0   Seafood Restaurant  0.24
    1     Sushi Restaurant  0.18
    2  American Restaurant  0.12
    
    
    ----Juneau----
                     venue  freq
    0  American Restaurant   0.5
    1     Sushi Restaurant   0.5
    2     Asian Restaurant   0.0
    
    
    ----Sitka----
                     venue  freq
    0   Seafood Restaurant   0.3
    1           Restaurant   0.2
    2  American Restaurant   0.1
    
    



```python
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    
    return row_categories_sorted.index.values[0:num_top_venues]
```


```python
num_top_venues = 10

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['Neighborhood']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
neighborhoods_venues_sorted = pd.DataFrame(columns=columns)
neighborhoods_venues_sorted['Neighborhood'] = alaska_grouped['Neighborhood']

for ind in np.arange(alaska_grouped.shape[0]):
    neighborhoods_venues_sorted.iloc[ind, 1:] = return_most_common_venues(alaska_grouped.iloc[ind, :], num_top_venues)

neighborhoods_venues_sorted.head()
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
      <th>Neighborhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Anchorage</td>
      <td>Seafood Restaurant</td>
      <td>Sushi Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Restaurant</td>
      <td>American Restaurant</td>
      <td>Theme Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Cajun / Creole Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Mediterranean Restaurant</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Juneau</td>
      <td>Sushi Restaurant</td>
      <td>American Restaurant</td>
      <td>Theme Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Seafood Restaurant</td>
      <td>Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Mediterranean Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Cajun / Creole Restaurant</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sitka</td>
      <td>Seafood Restaurant</td>
      <td>Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Mediterranean Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>American Restaurant</td>
      <td>Theme Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Sushi Restaurant</td>
    </tr>
  </tbody>
</table>
</div>




```python
# import k-means from clustering stage
from sklearn.cluster import KMeans

# set number of clusters
kclusters = 3

alaska_grouped_clustering = alaska_grouped.drop('Neighborhood', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(alaska_grouped_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10]
```




    array([0, 1, 2], dtype=int32)




```python
# add clustering labels
neighborhoods_venues_sorted.insert(0, 'ClusterLabels', kmeans.labels_)

alaska_merged = Alaska_data

# merge alaska_grouped with singapore to add latitude/longitude for each neighborhood
alaska_merged = alaska_merged.join(neighborhoods_venues_sorted.set_index('Neighborhood'), on='Name')

alaska_merged.head() # check the last columns!
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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>ClusterLabels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aleutians East Borough</td>
      <td>Second</td>
      <td>6,988</td>
      <td>55.051244</td>
      <td>-162.891689</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>0.0</td>
      <td>Seafood Restaurant</td>
      <td>Sushi Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Restaurant</td>
      <td>American Restaurant</td>
      <td>Theme Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Cajun / Creole Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Mediterranean Restaurant</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bristol Bay Borough</td>
      <td>Second</td>
      <td>505</td>
      <td>58.737034</td>
      <td>-156.875387</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Denali Borough</td>
      <td>Home Rule</td>
      <td>12,750</td>
      <td>63.878678</td>
      <td>-149.650166</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fairbanks North Star Borough</td>
      <td>Second</td>
      <td>7,366</td>
      <td>64.864904</td>
      <td>-146.775162</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
alaska_merged['ClusterLabels'].fillna(0, inplace=True)
alaska_merged['ClusterLabels'] = alaska_merged['ClusterLabels'].apply(np.int64)
alaska_merged['ClusterLabels']
```




    0     0
    1     0
    2     0
    3     0
    4     0
    5     0
    6     1
    7     0
    8     0
    9     0
    10    0
    11    0
    12    0
    13    0
    14    0
    15    2
    16    0
    17    0
    18    0
    19    0
    Name: ClusterLabels, dtype: int64




```python
# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors


# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=4)

# set color scheme for the clusters
x = np.arange(kclusters)
ys = [i + x + (i*x)**2 for i in range(kclusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(alaska_merged['Latitude'], alaska_merged['Longitude'], alaska_merged['Name'], alaska_merged['ClusterLabels']):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzgzNjJlZWZmNTJkNjRlMjI5NWIwZDIzZDM1MDRlZjUzIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1MyA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1MycsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbNjQuNDQ1OTYxMywtMTQ5LjY4MDkwOV0sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB6b29tOiA0LAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgbWF4Qm91bmRzOiBib3VuZHMsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBsYXllcnM6IFtdLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgd29ybGRDb3B5SnVtcDogZmFsc2UsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3CiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIH0pOwogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl9mODZkMDU1NTljYTg0ZDRhYWE4MWZiODFiMDRjNjliYyA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgJ2h0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nJywKICAgICAgICAgICAgICAgIHsKICAiYXR0cmlidXRpb24iOiBudWxsLAogICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwKICAibWF4Wm9vbSI6IDE4LAogICJtaW5ab29tIjogMSwKICAibm9XcmFwIjogZmFsc2UsCiAgInN1YmRvbWFpbnMiOiAiYWJjIgp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNWI3ODYyN2Q1NmY2NDNhOThmMzZkZDZkMTFhY2RlYmEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs1NS4wNTEyNDQzLC0xNjIuODkxNjg5Ml0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8zYTljOWYyOTQyMTk0NmI1OTk0MzhmNjI0M2M2ZmU5MiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF85ZGFhZjNhZmYyZTA0ODQyYjNiMWM5MzMyMzlkOTU0NCA9ICQoJzxkaXYgaWQ9Imh0bWxfOWRhYWYzYWZmMmUwNDg0MmIzYjFjOTMzMjM5ZDk1NDQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkFsZXV0aWFucyBFYXN0IEJvcm91Z2ggQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8zYTljOWYyOTQyMTk0NmI1OTk0MzhmNjI0M2M2ZmU5Mi5zZXRDb250ZW50KGh0bWxfOWRhYWYzYWZmMmUwNDg0MmIzYjFjOTMzMjM5ZDk1NDQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNWI3ODYyN2Q1NmY2NDNhOThmMzZkZDZkMTFhY2RlYmEuYmluZFBvcHVwKHBvcHVwXzNhOWM5ZjI5NDIxOTQ2YjU5OTQzOGY2MjQzYzZmZTkyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzJiNzRkNDJmZWEyMTQ2NmI4M2U1MzExOTM0N2Q0ZjUyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNjEuMjE2MzEyOSwtMTQ5Ljg5NDg1MjNdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNmMzZjc4NGRlNWM3NDE5ZmFlYjE5MjkxMWMzZDJjZDggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNDE4MTk3OGZmODVmNGI0YmJjZjc1MGM4ZTU2ZjZiNmEgPSAkKCc8ZGl2IGlkPSJodG1sXzQxODE5NzhmZjg1ZjRiNGJiY2Y3NTBjOGU1NmY2YjZhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5BbmNob3JhZ2UgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82YzNmNzg0ZGU1Yzc0MTlmYWViMTkyOTExYzNkMmNkOC5zZXRDb250ZW50KGh0bWxfNDE4MTk3OGZmODVmNGI0YmJjZjc1MGM4ZTU2ZjZiNmEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmI3NGQ0MmZlYTIxNDY2YjgzZTUzMTE5MzQ3ZDRmNTIuYmluZFBvcHVwKHBvcHVwXzZjM2Y3ODRkZTVjNzQxOWZhZWIxOTI5MTFjM2QyY2Q4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RlMTgxYzNjNWY3MzQ4NGM4ODkwNzM5ZTNhNjE3OWUxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTguNzM3MDM0MSwtMTU2Ljg3NTM4NjddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMzAxYWUwNjU4MDMxNDYwMmIyODhjYzBiZDc1NzdiNTYgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZDU3ZjEwODA4ZGI4NGRlZDllNzIyNmMzN2UxOTYwMzIgPSAkKCc8ZGl2IGlkPSJodG1sX2Q1N2YxMDgwOGRiODRkZWQ5ZTcyMjZjMzdlMTk2MDMyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CcmlzdG9sIEJheSBCb3JvdWdoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMzAxYWUwNjU4MDMxNDYwMmIyODhjYzBiZDc1NzdiNTYuc2V0Q29udGVudChodG1sX2Q1N2YxMDgwOGRiODRkZWQ5ZTcyMjZjMzdlMTk2MDMyKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2RlMTgxYzNjNWY3MzQ4NGM4ODkwNzM5ZTNhNjE3OWUxLmJpbmRQb3B1cChwb3B1cF8zMDFhZTA2NTgwMzE0NjAyYjI4OGNjMGJkNzU3N2I1Nik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xN2Y2ZTRiMjE1NmU0ZWM4YTg3YzNhZThmYWE3OGEwOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzYzLjg3ODY3OCwtMTQ5LjY1MDE2Nl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9lYTA0ZjRkYzYxMjE0MTk2YTgyZGIxNDczYzA4YjI3YiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9mMDk0YWYwM2I0NDA0OTQyOTk1ZjdjY2ZkZTczZTcwZiA9ICQoJzxkaXYgaWQ9Imh0bWxfZjA5NGFmMDNiNDQwNDk0Mjk5NWY3Y2NmZGU3M2U3MGYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRlbmFsaSBCb3JvdWdoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZWEwNGY0ZGM2MTIxNDE5NmE4MmRiMTQ3M2MwOGIyN2Iuc2V0Q29udGVudChodG1sX2YwOTRhZjAzYjQ0MDQ5NDI5OTVmN2NjZmRlNzNlNzBmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzE3ZjZlNGIyMTU2ZTRlYzhhODdjM2FlOGZhYTc4YTA5LmJpbmRQb3B1cChwb3B1cF9lYTA0ZjRkYzYxMjE0MTk2YTgyZGIxNDczYzA4YjI3Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84ZGM3ZTljYmM5NzQ0ZWFkOTJlMGU2MDIyZjZjYjk5NyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzY0Ljg2NDkwMzksLTE0Ni43NzUxNjE5XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzgzNjJlZWZmNTJkNjRlMjI5NWIwZDIzZDM1MDRlZjUzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzJhYjAxNTU1YmI3ZTRmOWZiMjQ1MmMzOGYyNTBhNzYxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2VlMTdiMjBjZjdhOTQxMWE4MjcwNGRjOWM3ZmY4MjEwID0gJCgnPGRpdiBpZD0iaHRtbF9lZTE3YjIwY2Y3YTk0MTFhODI3MDRkYzljN2ZmODIxMCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RmFpcmJhbmtzIE5vcnRoIFN0YXIgQm9yb3VnaCBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzJhYjAxNTU1YmI3ZTRmOWZiMjQ1MmMzOGYyNTBhNzYxLnNldENvbnRlbnQoaHRtbF9lZTE3YjIwY2Y3YTk0MTFhODI3MDRkYzljN2ZmODIxMCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl84ZGM3ZTljYmM5NzQ0ZWFkOTJlMGU2MDIyZjZjYjk5Ny5iaW5kUG9wdXAocG9wdXBfMmFiMDE1NTViYjdlNGY5ZmIyNDUyYzM4ZjI1MGE3NjEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYzAwNDk0MjhhOGU2NDU3YzhlMGViNDVlZGFiZDJhZDYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs1OS4wODMxMjMyLC0xMzUuMzQzMDU3M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9hYzIwNjk0OGUzYWE0YTVkYWVmMmU3OWIxMDA0ZWQ0MyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF83YTIxNzIxNmZkMTU0ZDFjYmJhNTVhZjFjMTY4MjNiYSA9ICQoJzxkaXYgaWQ9Imh0bWxfN2EyMTcyMTZmZDE1NGQxY2JiYTU1YWYxYzE2ODIzYmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhhaW5lcyBCb3JvdWdoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYWMyMDY5NDhlM2FhNGE1ZGFlZjJlNzliMTAwNGVkNDMuc2V0Q29udGVudChodG1sXzdhMjE3MjE2ZmQxNTRkMWNiYmE1NWFmMWMxNjgyM2JhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2MwMDQ5NDI4YThlNjQ1N2M4ZTBlYjQ1ZWRhYmQyYWQ2LmJpbmRQb3B1cChwb3B1cF9hYzIwNjk0OGUzYWE0YTVkYWVmMmU3OWIxMDA0ZWQ0Myk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yYjY0MzAxNmNmMTI0ZDNiOTFiMDBiMjFkZmEwZTljNyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU4LjMwMTk0OTYsLTEzNC40MTk3MzRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwMDBmZiIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MDAwZmYiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYTM5OWIxOGJlM2NiNGNlOGE5OWRjZTliNjJkNTJhMDcgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNjU3NDY0ODRjMGM3NGE1OTg2ODRlMjllYmZmODAzMDQgPSAkKCc8ZGl2IGlkPSJodG1sXzY1NzQ2NDg0YzBjNzRhNTk4Njg0ZTI5ZWJmZjgwMzA0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5KdW5lYXUgQ2x1c3RlciAxPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9hMzk5YjE4YmUzY2I0Y2U4YTk5ZGNlOWI2MmQ1MmEwNy5zZXRDb250ZW50KGh0bWxfNjU3NDY0ODRjMGM3NGE1OTg2ODRlMjllYmZmODAzMDQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmI2NDMwMTZjZjEyNGQzYjkxYjAwYjIxZGZhMGU5YzcuYmluZFBvcHVwKHBvcHVwX2EzOTliMThiZTNjYjRjZThhOTlkY2U5YjYyZDUyYTA3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNhZjA3OWJmNDIwZTRmZWY5ODVkMjdlYjhhZTNmMWZlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNjAuMDk2ODI3MiwtMTUxLjc4ODAzM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8yMzVkNTIxZmU0OGU0NzFmODhlYTQ5Mjk3OTk2NDdjOCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yODA2MTYxYjc0MzI0Njk3OTBmMDJhM2Y2ZjZmMzJhZCA9ICQoJzxkaXYgaWQ9Imh0bWxfMjgwNjE2MWI3NDMyNDY5NzkwZjAyYTNmNmY2ZjMyYWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktlbmFpIFBlbmluc3VsYSBCb3JvdWdoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMjM1ZDUyMWZlNDhlNDcxZjg4ZWE0OTI5Nzk5NjQ3Yzguc2V0Q29udGVudChodG1sXzI4MDYxNjFiNzQzMjQ2OTc5MGYwMmEzZjZmNmYzMmFkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzNhZjA3OWJmNDIwZTRmZWY5ODVkMjdlYjhhZTNmMWZlLmJpbmRQb3B1cChwb3B1cF8yMzVkNTIxZmU0OGU0NzFmODhlYTQ5Mjk3OTk2NDdjOCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80OGVjZjg0ZWRlOWI0ZjA0ODQ1M2Y0MDdkZTZiOTA0NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU1LjQ4OTc0OCwtMTMxLjAxMTk2M10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8xNGFmNzllZmE2MDA0Yjg4ODE2MjQ3MGJiZTg1ZGI4MiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hNjFhNjcxNmZiMWE0MTYwODFkNjRlNGZjNmEzOGU2OSA9ICQoJzxkaXYgaWQ9Imh0bWxfYTYxYTY3MTZmYjFhNDE2MDgxZDY0ZTRmYzZhMzhlNjkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktldGNoaWthbiBHYXRld2F5IEJvcm91Z2ggQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xNGFmNzllZmE2MDA0Yjg4ODE2MjQ3MGJiZTg1ZGI4Mi5zZXRDb250ZW50KGh0bWxfYTYxYTY3MTZmYjFhNDE2MDgxZDY0ZTRmYzZhMzhlNjkpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNDhlY2Y4NGVkZTliNGYwNDg0NTNmNDA3ZGU2YjkwNDQuYmluZFBvcHVwKHBvcHVwXzE0YWY3OWVmYTYwMDRiODg4MTYyNDcwYmJlODVkYjgyKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk1YjJmNjVmYzE2ZjQ4NWVhN2Q2NmIyYjUyZTg1ZGM3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTcuNTQzMzc2NSwtMTUzLjM1NzQxMjRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfMTc4YWEzNzMxZWYwNDc1NWIyMjcxYmRhMzFiYTU5OTkgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfYzA5YTMxZWRmNGZmNGUyMDkyYjZmYTZhMzA2ODk0MTcgPSAkKCc8ZGl2IGlkPSJodG1sX2MwOWEzMWVkZjRmZjRlMjA5MmI2ZmE2YTMwNjg5NDE3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Lb2RpYWsgSXNsYW5kIEJvcm91Z2ggQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8xNzhhYTM3MzFlZjA0NzU1YjIyNzFiZGEzMWJhNTk5OS5zZXRDb250ZW50KGh0bWxfYzA5YTMxZWRmNGZmNGUyMDkyYjZmYTZhMzA2ODk0MTcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTViMmY2NWZjMTZmNDg1ZWE3ZDY2YjJiNTJlODVkYzcuYmluZFBvcHVwKHBvcHVwXzE3OGFhMzczMWVmMDQ3NTViMjI3MWJkYTMxYmE1OTk5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzZhNzBmZGU1OTA5MTQ0YjJhNjQ5OTg5ZDM4ZTNjZDNkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTguMzI3NzEwOSwtMTU2LjE1NDc2NV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF8wMzk2ZTNkY2UwMTY0ZTlhYmRhZjYyM2Q2YTI1YTdkNyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF82M2ZlNzM3ODQ4ZDI0NGQzYTc3NjAyZjNmNWM2NGQ3ZiA9ICQoJzxkaXYgaWQ9Imh0bWxfNjNmZTczNzg0OGQyNDRkM2E3NzYwMmYzZjVjNjRkN2YiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxha2UgYW5kIFBlbmluc3VsYSBCb3JvdWdoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMDM5NmUzZGNlMDE2NGU5YWJkYWY2MjNkNmEyNWE3ZDcuc2V0Q29udGVudChodG1sXzYzZmU3Mzc4NDhkMjQ0ZDNhNzc2MDJmM2Y1YzY0ZDdmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzZhNzBmZGU1OTA5MTQ0YjJhNjQ5OTg5ZDM4ZTNjZDNkLmJpbmRQb3B1cChwb3B1cF8wMzk2ZTNkY2UwMTY0ZTlhYmRhZjYyM2Q2YTI1YTdkNyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81ODQ4OWU5ZGFhZmE0OGNkYTRhYjBjMjVkZmJiNzQ4NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzYyLjM0MDI0ODEsLTE0OS40NzkzMjg4XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzgzNjJlZWZmNTJkNjRlMjI5NWIwZDIzZDM1MDRlZjUzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzEzMjdmNjNhMjNjMjQzNDJhMzg3YTc2MWI1ZDVmM2ExID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzdjNDQyMjk2M2FiZTQ2NzZhNDdlNzdiYzM1MjBhNzk0ID0gJCgnPGRpdiBpZD0iaHRtbF83YzQ0MjI5NjNhYmU0Njc2YTQ3ZTc3YmMzNTIwYTc5NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TWF0YW51c2thLVN1c2l0bmEgQm9yb3VnaCBDbHVzdGVyIDA8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzEzMjdmNjNhMjNjMjQzNDJhMzg3YTc2MWI1ZDVmM2ExLnNldENvbnRlbnQoaHRtbF83YzQ0MjI5NjNhYmU0Njc2YTQ3ZTc3YmMzNTIwYTc5NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81ODQ4OWU5ZGFhZmE0OGNkYTRhYjBjMjVkZmJiNzQ4NC5iaW5kUG9wdXAocG9wdXBfMTMyN2Y2M2EyM2MyNDM0MmEzODdhNzYxYjVkNWYzYTEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDQyNGE2Y2Q2NTdhNGEyOGJmNWQzNDc0MjIyNjhiOGEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs2OS41MzM1MTI5LC0xNTMuODIyMDY4MV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF84MDExYWUzMzJiZmM0MWUyYjkxMzk4NDVmYWNhNGIxOSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF84NGM1NDFhMzM4ZGM0NGMzYWMwYzQzZDc4ZjY3NDRlZiA9ICQoJzxkaXYgaWQ9Imh0bWxfODRjNTQxYTMzOGRjNDRjM2FjMGM0M2Q3OGY2NzQ0ZWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIFNsb3BlIEJvcm91Z2ggQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84MDExYWUzMzJiZmM0MWUyYjkxMzk4NDVmYWNhNGIxOS5zZXRDb250ZW50KGh0bWxfODRjNTQxYTMzOGRjNDRjM2FjMGM0M2Q3OGY2NzQ0ZWYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNDQyNGE2Y2Q2NTdhNGEyOGJmNWQzNDc0MjIyNjhiOGEuYmluZFBvcHVwKHBvcHVwXzgwMTFhZTMzMmJmYzQxZTJiOTEzOTg0NWZhY2E0YjE5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQ5MjRlMGFiYTY5MTQ4NDU5MjIxZGM3Y2Q1NDZkZWVjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNjcuMjM4NTEyNywtMTU5Ljk4MTYzNDddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfOTYyYmJmY2M1YzIzNGUzMzkwMTM4N2Q0MTEyNmFkNzggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMzhiYmI5YTgwNmNlNDQ4MjhmNWE0OTZhMzU1MDM3MTcgPSAkKCc8ZGl2IGlkPSJodG1sXzM4YmJiOWE4MDZjZTQ0ODI4ZjVhNDk2YTM1NTAzNzE3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ob3J0aHdlc3QgQXJjdGljIEJvcm91Z2ggQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF85NjJiYmZjYzVjMjM0ZTMzOTAxMzg3ZDQxMTI2YWQ3OC5zZXRDb250ZW50KGh0bWxfMzhiYmI5YTgwNmNlNDQ4MjhmNWE0OTZhMzU1MDM3MTcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNDkyNGUwYWJhNjkxNDg0NTkyMjFkYzdjZDU0NmRlZWMuYmluZFBvcHVwKHBvcHVwXzk2MmJiZmNjNWMyMzRlMzM5MDEzODdkNDExMjZhZDc4KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2E2ZDMxMmRkMzBjMTQzMDg5NTc0MzFlYTIwY2JlMjQ0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTYuNzUxMjIwNywtMTMzLjQ1ODg0MDZdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjVhOTk0MDE1Y2YxNDc5NjgxNGEzODliMWFkNGEyYzMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMjJiOGNhMjQyYjNhNDc0YmE5YjNkNzA0NTcyYjhhNWEgPSAkKCc8ZGl2IGlkPSJodG1sXzIyYjhjYTI0MmIzYTQ3NGJhOWIzZDcwNDU3MmI4YTVhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QZXRlcnNidXJnIEJvcm91Z2ggQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mNWE5OTQwMTVjZjE0Nzk2ODE0YTM4OWIxYWQ0YTJjMy5zZXRDb250ZW50KGh0bWxfMjJiOGNhMjQyYjNhNDc0YmE5YjNkNzA0NTcyYjhhNWEpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYTZkMzEyZGQzMGMxNDMwODk1NzQzMWVhMjBjYmUyNDQuYmluZFBvcHVwKHBvcHVwX2Y1YTk5NDAxNWNmMTQ3OTY4MTRhMzg5YjFhZDRhMmMzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2EyZjE2YzUwM2ZkODQwZmE5MDJlNDdmNmU2NDgwNmE4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTcuMDUyNDk3MywtMTM1LjMzNzYxMjRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiIzgwZmZiNCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiM4MGZmYjQiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjExNDVhNmQ1NmYzNDlhMmI1NmRhODZhYmY1YzU3MTggPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfODllN2IxMDA2MjkxNDcwMzgyY2M1ZTRhYzE4OGU4ZDYgPSAkKCc8ZGl2IGlkPSJodG1sXzg5ZTdiMTAwNjI5MTQ3MDM4MmNjNWU0YWMxODhlOGQ2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TaXRrYSBDbHVzdGVyIDI8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2YxMTQ1YTZkNTZmMzQ5YTJiNTZkYTg2YWJmNWM1NzE4LnNldENvbnRlbnQoaHRtbF84OWU3YjEwMDYyOTE0NzAzODJjYzVlNGFjMTg4ZThkNik7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hMmYxNmM1MDNmZDg0MGZhOTAyZTQ3ZjZlNjQ4MDZhOC5iaW5kUG9wdXAocG9wdXBfZjExNDVhNmQ1NmYzNDlhMmI1NmRhODZhYmY1YzU3MTgpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDQ1ODI2ZWZiMjc5NGNiYWI2MThlMGM0N2Q0ZTZiODUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs1OS42MjI2MzYyLC0xMzUuNDA5NDM2OF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9iYTkxNjhiYWU4Y2E0ZjI1YmMzMjE3MDY2YTBhOTY1NyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF8yMDBiZDYzNGM1ZGI0MjJjODBjMTU5MzUzZTI4MTA3YSA9ICQoJzxkaXYgaWQ9Imh0bWxfMjAwYmQ2MzRjNWRiNDIyYzgwYzE1OTM1M2UyODEwN2EiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNrYWd3YXkgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9iYTkxNjhiYWU4Y2E0ZjI1YmMzMjE3MDY2YTBhOTY1Ny5zZXRDb250ZW50KGh0bWxfMjAwYmQ2MzRjNWRiNDIyYzgwYzE1OTM1M2UyODEwN2EpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfZDQ1ODI2ZWZiMjc5NGNiYWI2MThlMGM0N2Q0ZTZiODUuYmluZFBvcHVwKHBvcHVwX2JhOTE2OGJhZThjYTRmMjViYzMyMTcwNjZhMGE5NjU3KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzgwYzkwOWEzNTM3MzRiYTc5N2U2MGJjMzZlNDljM2Y0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNjMuNDE3NDMxMDUsLTE1Ny42NzE4NjUwNDg0NTddLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiI2ZmMDAwMCIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiNmZjAwMDAiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfODM2MmVlZmY1MmQ2NGUyMjk1YjBkMjNkMzUwNGVmNTMpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjY5MTczY2RlN2YzNDA3Y2FkYzAxMTRlYmMxNDVjMTUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNmEyNWRiMjBiNGUxNDk2NGE2OTk5YWI0YzgxMDY5MjcgPSAkKCc8ZGl2IGlkPSJodG1sXzZhMjVkYjIwYjRlMTQ5NjRhNjk5OWFiNGM4MTA2OTI3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Vbm9yZ2FuaXplZCBCb3JvdWdoIENsdXN0ZXIgMDwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZjY5MTczY2RlN2YzNDA3Y2FkYzAxMTRlYmMxNDVjMTUuc2V0Q29udGVudChodG1sXzZhMjVkYjIwYjRlMTQ5NjRhNjk5OWFiNGM4MTA2OTI3KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzgwYzkwOWEzNTM3MzRiYTc5N2U2MGJjMzZlNDljM2Y0LmJpbmRQb3B1cChwb3B1cF9mNjkxNzNjZGU3ZjM0MDdjYWRjMDExNGViYzE0NWMxNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83MTA2YjY4Y2E3MjQ0NTI3OTE3MDMyNjhkODI3MmYzYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzU2LjIwNDU2NzMsLTEzMi4wNDMyNTUyXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogIiNmZjAwMDAiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwXzgzNjJlZWZmNTJkNjRlMjI5NWIwZDIzZDM1MDRlZjUzKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2MxZjQxN2U1OTgyOTRmMTI5Njk4NDE2NDJiNzExYzRmID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ4NTRhZTY2NTg0ZjQ4ZDU4OGEwNTNkMmRjNzVkMzlkID0gJCgnPGRpdiBpZD0iaHRtbF80ODU0YWU2NjU4NGY0OGQ1ODhhMDUzZDJkYzc1ZDM5ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+V3JhbmdlbGwgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9jMWY0MTdlNTk4Mjk0ZjEyOTY5ODQxNjQyYjcxMWM0Zi5zZXRDb250ZW50KGh0bWxfNDg1NGFlNjY1ODRmNDhkNTg4YTA1M2QyZGM3NWQzOWQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfNzEwNmI2OGNhNzI0NDUyNzkxNzAzMjY4ZDgyNzJmM2EuYmluZFBvcHVwKHBvcHVwX2MxZjQxN2U1OTgyOTRmMTI5Njk4NDE2NDJiNzExYzRmKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzdiOGI4YzY5YjY3NDQ4MTFiYmFhZGFiNWZiNTRhNGQ5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNTkuNTcyNzM0NSwtMTM5LjU3ODMxMjQzODc4MV0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICIjZmYwMDAwIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF84MzYyZWVmZjUyZDY0ZTIyOTViMGQyM2QzNTA0ZWY1Myk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF82NDE5MmE2NTc4NTA0OGQ2YTFlNTdjZTE5YzFjZjZlYyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9hOTEwZTJmZWYwMmY0MmJjYjA5NDBiNjZlMDIxYzUxNiA9ICQoJzxkaXYgaWQ9Imh0bWxfYTkxMGUyZmVmMDJmNDJiY2IwOTQwYjY2ZTAyMWM1MTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPllha3V0YXQgQ2x1c3RlciAwPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF82NDE5MmE2NTc4NTA0OGQ2YTFlNTdjZTE5YzFjZjZlYy5zZXRDb250ZW50KGh0bWxfYTkxMGUyZmVmMDJmNDJiY2IwOTQwYjY2ZTAyMWM1MTYpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfN2I4YjhjNjliNjc0NDgxMWJiYWFkYWI1ZmI1NGE0ZDkuYmluZFBvcHVwKHBvcHVwXzY0MTkyYTY1Nzg1MDQ4ZDZhMWU1N2NlMTljMWNmNmVjKTsKCiAgICAgICAgICAgIAogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



<h1> 6. Examine Clusters </h1>


```python
alaska_cluster1=alaska_merged.loc[alaska_merged['ClusterLabels'] == 0, alaska_merged.columns[[0] + list(range(1, alaska_merged.shape[1]))]]
print ("No of Neighbourhood in Cluster Label 0: %d" %(alaska_cluster1.shape[0]))
alaska_cluster1
```

    No of Neighbourhood in Cluster Label 0: 18





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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>ClusterLabels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aleutians East Borough</td>
      <td>Second</td>
      <td>6,988</td>
      <td>55.051244</td>
      <td>-162.891689</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anchorage</td>
      <td>Unified Home Rule</td>
      <td>1,697</td>
      <td>61.216313</td>
      <td>-149.894852</td>
      <td>0</td>
      <td>Seafood Restaurant</td>
      <td>Sushi Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Restaurant</td>
      <td>American Restaurant</td>
      <td>Theme Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Cajun / Creole Restaurant</td>
      <td>Asian Restaurant</td>
      <td>Mediterranean Restaurant</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bristol Bay Borough</td>
      <td>Second</td>
      <td>505</td>
      <td>58.737034</td>
      <td>-156.875387</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Denali Borough</td>
      <td>Home Rule</td>
      <td>12,750</td>
      <td>63.878678</td>
      <td>-149.650166</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fairbanks North Star Borough</td>
      <td>Second</td>
      <td>7,366</td>
      <td>64.864904</td>
      <td>-146.775162</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Haines Borough</td>
      <td>Home Rule</td>
      <td>2,344</td>
      <td>59.083123</td>
      <td>-135.343057</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Kenai Peninsula Borough</td>
      <td>Second</td>
      <td>16,013</td>
      <td>60.096827</td>
      <td>-151.788033</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Ketchikan Gateway Borough</td>
      <td>Second</td>
      <td>4,840</td>
      <td>55.489748</td>
      <td>-131.011963</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Kodiak Island Borough</td>
      <td>Second</td>
      <td>6,560</td>
      <td>57.543377</td>
      <td>-153.357412</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Lake and Peninsula Borough</td>
      <td>Home Rule</td>
      <td>23,782</td>
      <td>58.327711</td>
      <td>-156.154765</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Matanuska-Susitna Borough</td>
      <td>Second</td>
      <td>24,682</td>
      <td>62.340248</td>
      <td>-149.479329</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>North Slope Borough</td>
      <td>Home Rule</td>
      <td>88,817</td>
      <td>69.533513</td>
      <td>-153.822068</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Northwest Arctic Borough</td>
      <td>Home Rule</td>
      <td>35,898</td>
      <td>67.238513</td>
      <td>-159.981635</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Petersburg Borough</td>
      <td>Home Rule</td>
      <td>3,829</td>
      <td>56.751221</td>
      <td>-133.458841</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Skagway</td>
      <td>First</td>
      <td>452</td>
      <td>59.622636</td>
      <td>-135.409437</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Unorganized Borough</td>
      <td>-</td>
      <td>323,440</td>
      <td>63.417431</td>
      <td>-157.671865</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Wrangell</td>
      <td>Unified Home Rule</td>
      <td>2,570</td>
      <td>56.204567</td>
      <td>-132.043255</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Yakutat</td>
      <td>Home Rule</td>
      <td>7,650</td>
      <td>59.572735</td>
      <td>-139.578312</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
alaska_cluster3=alaska_merged.loc[alaska_merged['ClusterLabels'] == 2, alaska_merged.columns[[0] + list(range(1, alaska_merged.shape[1]))]]
print ("No of Neighbourhood in Cluster Label 2: %d" %(alaska_cluster3.shape[0]))
alaska_cluster3
```

    No of Neighbourhood in Cluster Label 2: 1





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
      <th>Name</th>
      <th>Region</th>
      <th>Area_SqKm</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>ClusterLabels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>15</th>
      <td>Sitka</td>
      <td>Unified Home Rule</td>
      <td>2,874</td>
      <td>57.052497</td>
      <td>-135.337612</td>
      <td>2</td>
      <td>Seafood Restaurant</td>
      <td>Restaurant</td>
      <td>Mexican Restaurant</td>
      <td>Mediterranean Restaurant</td>
      <td>Japanese Restaurant</td>
      <td>Asian Restaurant</td>
      <td>American Restaurant</td>
      <td>Theme Restaurant</td>
      <td>Thai Restaurant</td>
      <td>Sushi Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



<h1>7. Discussion </h1>
The first important observation noticed while creating this report is that the Foursquare data returned is not the same each time an API call is made and hence, 
the results are not reproducible unless one does save this data to a file for future use. Because of this, a high limit for each call needs to be used in order to maximize
the number fo results. Morover, the radius used during the search had an important impact on the number of results because some central of neighborhoods are too dense 
compared to others. When it comes to performing k-means on the grouped data, special attention mus be placed when choosing the number of clusters as well as the top number 
of venues to be used. For instance, all previous values tried (<9) resulted in contradictory results. Also, the number of top venues (10) had an impact on the resultst,
but not as important the number of clusters so we sticked to 10.

<h1>8. Conclusion </h1>
We have used the city's data set to obtain each neighborhood's location in the form of a GEOJSON file from which we extracted each neighborhoods polygon's centroid. 
Using this location information and Foursquare data we obtained 3 clusters using the top 10 venues for each individual neighborhood.
Finally, this information allowed us to recommend three areas of the city to customers interested in cheap restaurant, these areas were chosen because of their 
proximity to the city and their respective clusters, however one must notice that cluster 3 includes both near and far neighborhoods.
