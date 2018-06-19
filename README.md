# Observations

* City temperature is relatively normally distributed across latitudes. There is a peak temperature in the chart at roughly +25 latitude, rather than the equator. This is likely because the data was pulled in June 18, 2018, a few days before the Northern Hemisphere's Summer Equinox. A year-long average would likely yield a peak around 0 latitude, but that historical data is not available from the free tier of the Open Weather Map API.

* City temperatures in the southern hemisphere have less variance against latitude than in northern hemisphere cities. This is likely due to narrower non-Antarctic landmasses in the Southern Hemisphere. On the Longtitude v. Latitude chart, negative latitude cities are narrowly clustered longtitudinally in bands around -60 W (most of South America), +25 E (Subequatorial Africa), and +150 E (Oceania and much of Indonesia). Positive latitude cities have a much more broad and even longitudinal distribution, reflecting more available landmass East-West.

* Humidity, Wind Speed, and Cloud Cover do not have any discernable relationship with Latitude.

```python
#import dependencies

import matplotlib.pyplot as plt
from pprint import pprint
import json
import random
import numpy as np
import pandas as pd
from owm_api import key
from citipy import citipy
import requests
```


```python
# Step 1: Generate list of random cities

#Use random to generate a list of lat + long.
#Use citipy to find nearest city to coordinates

city = ''
cities = []

#this acts as a check to ensure that the generated city in loop is not a repeat
unique_cities_check = []

#Up to 10,000 attempts to create a list of 1000 unique cities
#In any given sample there's roughly 13% attrition in cities generated by CitiPy 
#which throws an error in OWM city search. Sample of 1000 is sufficient to confidently clear
#500+ cities.
for i in range(5000):

    if len(cities) < 1000:
        lat = random.uniform(-90.0,90.0)
        long = random.uniform(-180.0,180.0)
        
        city = citipy.nearest_city(lat,long)
        
        city_country = city.city_name + ' ' + city.country_code
        
        if city_country not in unique_cities_check:
            unique_cities_check.append(city_country)
            cities.append([city.city_name, city.country_code])
            
    else:
        break
```


```python
# Step 2
# Generate weather data based on the city and country data from OpenWeatherMap API

url = 'https://api.openweathermap.org/data/2.5/weather'

#Weather outputs
weather_data_list = []

for i in range(len(cities)):
    city = cities[i][0]
    country = cities[i][1]
        
    query_params={'q': city + ',' + country,
                  'units': 'imperial',
                  'appid': key}
    
    response = requests.get(url, params = query_params).json()

    #try-except because not every city in CityPy is found in OWM, about 13% attrition
    try:
        weather_data_list.append({'City': city,
                                  'Country': country,
                                  'Longitude': response['coord']['lon'], 
                                  'Latitude': response['coord']['lat'],
                                  'Max Temperature': response['main']['temp_max'],
                                  'Humidity': response['main']['humidity'],
                                  'Cloud Cover': response['clouds']['all'],
                                  'Wind Speed': response['wind']['speed']
                                 })
        
    except:
        pass
```


```python
cities_df = pd.DataFrame(weather_data_list)

cities_df = cities_df[['City',
                       'Country',
                       'Longitude',
                       'Latitude',
                       'Max Temperature',
                       'Humidity',
                       'Cloud Cover',
                       'Wind Speed']]



cities_df.head()
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
      <th>City</th>
      <th>Country</th>
      <th>Longitude</th>
      <th>Latitude</th>
      <th>Max Temperature</th>
      <th>Humidity</th>
      <th>Cloud Cover</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>vaini</td>
      <td>to</td>
      <td>-175.20</td>
      <td>-21.20</td>
      <td>77.00</td>
      <td>78</td>
      <td>75</td>
      <td>10.29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>rikitea</td>
      <td>pf</td>
      <td>-134.97</td>
      <td>-23.12</td>
      <td>73.55</td>
      <td>100</td>
      <td>92</td>
      <td>24.99</td>
    </tr>
    <tr>
      <th>2</th>
      <td>torbay</td>
      <td>ca</td>
      <td>-52.73</td>
      <td>47.66</td>
      <td>50.00</td>
      <td>87</td>
      <td>90</td>
      <td>14.99</td>
    </tr>
    <tr>
      <th>3</th>
      <td>busselton</td>
      <td>au</td>
      <td>115.35</td>
      <td>-33.64</td>
      <td>60.68</td>
      <td>100</td>
      <td>92</td>
      <td>19.28</td>
    </tr>
    <tr>
      <th>4</th>
      <td>turukhansk</td>
      <td>ru</td>
      <td>87.96</td>
      <td>65.80</td>
      <td>66.26</td>
      <td>77</td>
      <td>12</td>
      <td>4.52</td>
    </tr>
  </tbody>
</table>
</div>




```python
cities_df.plot.scatter('Latitude', 
                       'Longitude',
                       title='City Latitude v. City Longitude',
                       s=40,
                       color='LightBlue',
                       edgecolor='Black',
                       grid=True) 

```




    <matplotlib.axes._subplots.AxesSubplot at 0x1118c80f0>




![png](output_5_1.png)



```python
cities_df.plot.scatter('Latitude', 
                       'Max Temperature', 
                       title='City Latitude v. Max Temperature (6/18/2018)', 
                       marker='o', 
                       s=40, 
                       color='LightBlue', 
                       edgecolor='black', 
                       grid=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11b8e57f0>




![png](output_6_1.png)



```python
cities_df.plot.scatter('Latitude', 
                       'Humidity', 
                       title='City Latitude v. Humidity (6/18/2018)', 
                       marker='o', 
                       s=40, 
                       color='LightBlue', 
                       edgecolor='black', 
                       grid=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11bf36a90>




![png](output_7_1.png)



```python
cities_df.plot.scatter('Latitude', 
                       'Cloud Cover', 
                       title='City Latitude v. Cloud Cover (6/18/2018)', 
                       marker='o', 
                       s=40, 
                       color='LightBlue', 
                       edgecolor='black', 
                       grid=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11c019828>




![png](output_8_1.png)



```python
cities_df.plot.scatter('Latitude', 
                       'Wind Speed', 
                       title='City Latitude v. Wind Speed (6/18/2018)', 
                       marker='o', 
                       s=40, 
                       color='LightBlue', 
                       edgecolor='black', 
                       grid=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11c06f2b0>




![png](output_9_1.png)
