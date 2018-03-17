

```python
import requests as req
import json
from citipy import citipy
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import time
import csv
```


```python
weather_df = pd.DataFrame()
weather_df['Latitude'] = [np.random.uniform(-90,90) for x in range(1500)]
weather_df['Longitude'] = [np.random.uniform(-180, 180) for x in range(1500)]

weather_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>23.319659</td>
      <td>66.924238</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-86.440005</td>
      <td>103.192761</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-26.524224</td>
      <td>48.583067</td>
    </tr>
    <tr>
      <th>3</th>
      <td>33.476057</td>
      <td>169.717345</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-1.338421</td>
      <td>-87.840204</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Create columns for city, country,temperature,humidity,cloudiness,wind speed

weather_df['City'] = ""
weather_df['Country'] = ""
weather_df['Temperature (F)'] = ""
weather_df['Humidity (%)'] = ""
weather_df['Cloudiness (%)'] = ""
weather_df['Wind Speed (mph)'] = ""
 
#Use citipy library to get the closest city and country name to the random coordinate

for index, row in weather_df.iterrows():
    near_city = citipy.nearest_city(row['Latitude'], row['Longitude']).city_name
    near_country = citipy.nearest_city(row['Latitude'], row['Longitude']).country_code
    
    weather_df.set_value(index,"City", near_city)
    weather_df.set_value(index,"Country", near_country)
    
    
weather_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>City</th>
      <th>Country</th>
      <th>Temperature (F)</th>
      <th>Humidity (%)</th>
      <th>Cloudiness (%)</th>
      <th>Wind Speed (mph)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>23.319659</td>
      <td>66.924238</td>
      <td>keti bandar</td>
      <td>pk</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>-86.440005</td>
      <td>103.192761</td>
      <td>albany</td>
      <td>au</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>-26.524224</td>
      <td>48.583067</td>
      <td>taolanaro</td>
      <td>mg</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>33.476057</td>
      <td>169.717345</td>
      <td>severo-kurilsk</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>-1.338421</td>
      <td>-87.840204</td>
      <td>san cristobal</td>
      <td>ec</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
#Remove duplicate cities and countries and NA values

weather_df = weather_df.drop_duplicates(['City','Country'],keep = "first")
weather_df = weather_df.dropna()

#Get the sample size
len(weather_df['City'].value_counts())
```




    623




```python
api_key = "9e046c1f8098cee75dad264a4c38cc30"
url = "http://api.openweathermap.org/data/2.5/weather?" 
units = "imperial"
counter = 1

for index, row in weather_df.iterrows():
    try:
        latitude_check = row["Latitude"]
        longitude_check = row["Longitude"]
        city = citipy.nearest_city(latitude_check, longitude_check)
        cityname = city.city_name
        country_name = city.country_code
        
        query_url = url + "appid=" + api_key + "&units=" + units + "&q=" + cityname + "," + country_name
    
        print("Retrieving data for city #" + str(counter) + " for " + cityname + "," + country_name)
        print("URL: " + query_url)
        print("-----------------------------------------------------------------------------")
    
        response = req.get(query_url).json()
        
        temp = response["main"]["temp_max"]
        humid = response["main"]["humidity"]
        cloudy = response["clouds"]["all"]
        wind = response["wind"]["speed"]
         
        newlatitude = response["coord"]["lat"]
        newlongtitude = response["coord"]["lon"]
    
        weather_df.set_value(index, "City", cityname)
        weather_df.set_value(index, "Country", country_name)
        weather_df.set_value(index, "Temperature (F)", temp)
        weather_df.set_value(index, "Humidity (%)", humid)
        weather_df.set_value(index, "Cloudiness (%)", cloudy)
        weather_df.set_value(index, "Wind Speed (mph)", wind)
         
        weather_df.set_value(index, "Latitude", newlatitude)
        weather_df.set_value(index, "Longitude", newlongitude)
    except:
        print("Getting an error, skipping this one!")
    
    counter = counter + 1
    
    time.sleep(2)

print("---------------------------------")
print("Data Retrieval Complete")
print("---------------------------------")
```

    Retrieving data for city #1 for rikitea,pf
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=rikitea,pf
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #2 for hermanus,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hermanus,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #3 for mataura,pf
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=mataura,pf
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #4 for fairbanks,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=fairbanks,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #5 for balkhash,kz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=balkhash,kz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #6 for ushuaia,ar
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ushuaia,ar
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #7 for busselton,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=busselton,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #8 for abnub,eg
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=abnub,eg
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #9 for colares,pt
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=colares,pt
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #10 for samusu,ws
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=samusu,ws
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #11 for verkhoyansk,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=verkhoyansk,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #12 for vardo,no
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=vardo,no
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #13 for praia da vitoria,pt
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=praia da vitoria,pt
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #14 for atuona,pf
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=atuona,pf
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #15 for kodiak,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kodiak,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #16 for saint-pierre,pm
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saint-pierre,pm
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #17 for saldanha,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saldanha,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #18 for broome,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=broome,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #19 for provideniya,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=provideniya,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #20 for dikson,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=dikson,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #21 for nalut,ly
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nalut,ly
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #22 for khatanga,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=khatanga,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #23 for bethel,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bethel,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #24 for newport,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=newport,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #25 for ksenyevka,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ksenyevka,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #26 for challans,fr
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=challans,fr
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #27 for trinidad,cu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=trinidad,cu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #28 for coroico,bo
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=coroico,bo
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #29 for brae,gb
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=brae,gb
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #30 for qaanaaq,gl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=qaanaaq,gl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #31 for saint george,bm
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saint george,bm
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #32 for victoria,sc
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=victoria,sc
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #33 for castro,cl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=castro,cl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #34 for hobart,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hobart,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #35 for fortuna,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=fortuna,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #36 for talhar,pk
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=talhar,pk
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #37 for laguna,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=laguna,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #38 for namibe,ao
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=namibe,ao
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #39 for hovd,mn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hovd,mn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #40 for sitka,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sitka,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #41 for vaini,to
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=vaini,to
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #42 for platanos,gr
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=platanos,gr
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #43 for villazon,bo
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=villazon,bo
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #44 for okhansk,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=okhansk,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #45 for sheridan,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sheridan,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #46 for bredasdorp,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bredasdorp,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #47 for sangar,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sangar,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #48 for arraial do cabo,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=arraial do cabo,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #49 for acapulco,mx
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=acapulco,mx
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #50 for sarakhs,ir
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sarakhs,ir
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #51 for lasa,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=lasa,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #52 for atar,mr
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=atar,mr
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #53 for birao,cf
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=birao,cf
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #54 for chicama,pe
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=chicama,pe
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #55 for lobito,ao
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=lobito,ao
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #56 for beyneu,kz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=beyneu,kz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #57 for meulaboh,id
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=meulaboh,id
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #58 for hithadhoo,mv
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hithadhoo,mv
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #59 for ahipara,nz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ahipara,nz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #60 for mahebourg,mu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=mahebourg,mu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #61 for dagana,sn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=dagana,sn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #62 for kaitangata,nz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kaitangata,nz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #63 for cuite,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=cuite,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #64 for port alfred,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=port alfred,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #65 for thisted,dk
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=thisted,dk
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #66 for amderma,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=amderma,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #67 for cherskiy,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=cherskiy,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #68 for mount gambier,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=mount gambier,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #69 for north bend,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=north bend,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #70 for grand river south east,mu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=grand river south east,mu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #71 for sadiqabad,pk
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sadiqabad,pk
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #72 for bluff,nz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bluff,nz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #73 for fukue,jp
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=fukue,jp
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #74 for barrow,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=barrow,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #75 for nikolskoye,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nikolskoye,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #76 for butaritari,ki
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=butaritari,ki
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #77 for ilulissat,gl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ilulissat,gl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #78 for tumannyy,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tumannyy,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #79 for isangel,vu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=isangel,vu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #80 for puerto carreno,co
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=puerto carreno,co
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #81 for dhidhdhoo,mv
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=dhidhdhoo,mv
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #82 for hilo,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hilo,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #83 for kaohsiung,tw
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kaohsiung,tw
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #84 for mar del plata,ar
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=mar del plata,ar
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #85 for nkhotakota,mw
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nkhotakota,mw
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #86 for belushya guba,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=belushya guba,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #87 for pevek,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=pevek,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #88 for kapaa,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kapaa,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #89 for novouzensk,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=novouzensk,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #90 for albany,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=albany,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #91 for gejiu,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=gejiu,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #92 for yellowknife,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=yellowknife,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #93 for cayenne,gf
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=cayenne,gf
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #94 for sainte-agathe-des-monts,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sainte-agathe-des-monts,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #95 for zuwarah,ly
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=zuwarah,ly
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #96 for vitina,gr
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=vitina,gr
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #97 for bengkulu,id
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bengkulu,id
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #98 for dandong,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=dandong,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #99 for thompson,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=thompson,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #100 for sobolevo,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sobolevo,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #101 for makakilo city,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=makakilo city,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #102 for vabalninkas,lt
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=vabalninkas,lt
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #103 for bangkal,ph
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bangkal,ph
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #104 for jamestown,sh
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=jamestown,sh
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #105 for barranca,pe
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=barranca,pe
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #106 for sao joao da barra,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sao joao da barra,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #107 for bachaquero,ve
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bachaquero,ve
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #108 for manokwari,id
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=manokwari,id
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #109 for champerico,gt
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=champerico,gt
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #110 for ayagoz,kz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ayagoz,kz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #111 for port hedland,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=port hedland,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #112 for geraldton,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=geraldton,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #113 for hobyo,so
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hobyo,so
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #114 for torbay,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=torbay,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #115 for oudtshoorn,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=oudtshoorn,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #116 for lira,ug
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=lira,ug
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #117 for cape town,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=cape town,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #118 for vanimo,pg
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=vanimo,pg
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #119 for husavik,is
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=husavik,is
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #120 for sokoni,tz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sokoni,tz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #121 for dakoro,ne
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=dakoro,ne
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #122 for penaflor,cl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=penaflor,cl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #123 for qaqortoq,gl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=qaqortoq,gl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #124 for egvekinot,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=egvekinot,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #125 for zeya,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=zeya,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #126 for jaciara,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=jaciara,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #127 for punta arenas,cl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=punta arenas,cl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #128 for eureka,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=eureka,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #129 for acajutla,sv
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=acajutla,sv
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #130 for saint anthony,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saint anthony,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #131 for gazli,uz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=gazli,uz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #132 for verkhnyaya inta,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=verkhnyaya inta,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #133 for bambous virieux,mu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bambous virieux,mu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #134 for sao filipe,cv
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sao filipe,cv
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #135 for moerai,pf
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=moerai,pf
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #136 for chilca,pe
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=chilca,pe
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #137 for urucara,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=urucara,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #138 for norman wells,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=norman wells,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #139 for chuy,uy
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=chuy,uy
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #140 for los llanos de aridane,es
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=los llanos de aridane,es
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #141 for esperance,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=esperance,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #142 for sibolga,id
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sibolga,id
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #143 for saint-philippe,re
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saint-philippe,re
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #144 for marcona,pe
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=marcona,pe
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #145 for klaksvik,fo
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=klaksvik,fo
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #146 for lavrentiya,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=lavrentiya,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #147 for raudeberg,no
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=raudeberg,no
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #148 for narsaq,gl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=narsaq,gl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #149 for le port,re
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=le port,re
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #150 for guerrero negro,mx
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=guerrero negro,mx
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #151 for jungapeo,mx
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=jungapeo,mx
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #152 for ostrovnoy,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ostrovnoy,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #153 for saleaula,ws
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saleaula,ws
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #154 for gigmoto,ph
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=gigmoto,ph
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #155 for avarua,ck
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=avarua,ck
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #156 for keelung,tw
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=keelung,tw
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #157 for amboasary,mg
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=amboasary,mg
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #158 for kahului,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kahului,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #159 for christchurch,nz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=christchurch,nz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #160 for carballo,es
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=carballo,es
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #161 for aksu,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=aksu,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #162 for karakendzha,tj
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=karakendzha,tj
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #163 for la ronge,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=la ronge,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #164 for lebu,cl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=lebu,cl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #165 for pisco,pe
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=pisco,pe
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #166 for dzaoudzi,yt
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=dzaoudzi,yt
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #167 for adrar,dz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=adrar,dz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #168 for comodoro rivadavia,ar
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=comodoro rivadavia,ar
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #169 for faya,td
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=faya,td
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #170 for umm durman,sd
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=umm durman,sd
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #171 for kalmunai,lk
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kalmunai,lk
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #172 for sokolo,ml
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sokolo,ml
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #173 for seoul,kr
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=seoul,kr
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #174 for yanam,in
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=yanam,in
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #175 for pontal do parana,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=pontal do parana,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #176 for tsihombe,mg
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tsihombe,mg
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #177 for saskylakh,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saskylakh,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #178 for myra,no
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=myra,no
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #179 for hami,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hami,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #180 for tasiilaq,gl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tasiilaq,gl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #181 for new norfolk,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=new norfolk,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #182 for adwa,et
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=adwa,et
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #183 for college,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=college,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #184 for tiksi,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tiksi,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #185 for shimoda,jp
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=shimoda,jp
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #186 for nizwa,om
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nizwa,om
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #187 for rovaniemi,fi
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=rovaniemi,fi
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #188 for lompoc,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=lompoc,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #189 for poronaysk,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=poronaysk,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #190 for souillac,mu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=souillac,mu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #191 for sentyabrskiy,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sentyabrskiy,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #192 for korla,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=korla,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #193 for coihaique,cl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=coihaique,cl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #194 for galiwinku,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=galiwinku,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #195 for katsuura,jp
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=katsuura,jp
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #196 for the pas,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=the pas,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #197 for tuktoyaktuk,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tuktoyaktuk,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #198 for cabo san lucas,mx
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=cabo san lucas,mx
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #199 for yar-sale,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=yar-sale,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #200 for ust-nera,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ust-nera,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #201 for puerto ayora,ec
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=puerto ayora,ec
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #202 for chippewa falls,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=chippewa falls,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #203 for springbok,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=springbok,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #204 for bac lieu,vn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=bac lieu,vn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #205 for anito,ph
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=anito,ph
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #206 for kotagiri,in
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kotagiri,in
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #207 for talnakh,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=talnakh,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #208 for samarai,pg
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=samarai,pg
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #209 for iskateley,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=iskateley,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #210 for storforshei,no
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=storforshei,no
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #211 for severo-kurilsk,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=severo-kurilsk,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #212 for ucluelet,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=ucluelet,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #213 for hamilton,bm
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=hamilton,bm
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #214 for carnarvon,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=carnarvon,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #215 for nizhneyansk,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nizhneyansk,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #216 for kyshtovka,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=kyshtovka,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #217 for gimli,ca
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=gimli,ca
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #218 for vestmannaeyjar,is
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=vestmannaeyjar,is
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #219 for beira,mz
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=beira,mz
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #220 for maun,bw
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=maun,bw
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #221 for corsicana,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=corsicana,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #222 for nykoping,se
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nykoping,se
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #223 for alenquer,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=alenquer,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #224 for yara,cu
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=yara,cu
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #225 for port-gentil,ga
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=port-gentil,ga
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #226 for sawakin,sd
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=sawakin,sd
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #227 for cidreira,br
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=cidreira,br
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #228 for nouadhibou,mr
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=nouadhibou,mr
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #229 for longyearbyen,sj
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=longyearbyen,sj
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #230 for flinders,au
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=flinders,au
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #231 for santa marinella,it
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=santa marinella,it
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #232 for meyungs,pw
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=meyungs,pw
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #233 for krechevitsy,ru
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=krechevitsy,ru
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #234 for port elizabeth,za
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=port elizabeth,za
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #235 for auki,sb
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=auki,sb
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #236 for saint george,us
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=saint george,us
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #237 for upernavik,gl
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=upernavik,gl
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #238 for tabuk,sa
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tabuk,sa
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #239 for samana,do
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=samana,do
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #240 for tomohon,id
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=tomohon,id
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!
    Retrieving data for city #241 for jining,cn
    URL: http://api.openweathermap.org/data/2.5/weather?appid=9e046c1f8098cee75dad264a4c38cc30&units=imperial&q=jining,cn
    -----------------------------------------------------------------------------
    Getting an error, skipping this one!



```python
# Save the DataFrame as a csv
weather_df.to_csv("weatherpy_data.csv", encoding="utf-8", index=False)
```


```python
weather_df = pd.read_csv("weatherpy_data.csv")
weather_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>City</th>
      <th>Country</th>
      <th>Temperature (F)</th>
      <th>Humidity (%)</th>
      <th>Cloudiness (%)</th>
      <th>Wind Speed (mph)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-53.16</td>
      <td>-110.561490</td>
      <td>punta arenas</td>
      <td>cl</td>
      <td>42.80</td>
      <td>70.0</td>
      <td>36.0</td>
      <td>5.82</td>
    </tr>
    <tr>
      <th>1</th>
      <td>34.16</td>
      <td>78.218930</td>
      <td>leh</td>
      <td>in</td>
      <td>10.31</td>
      <td>87.0</td>
      <td>80.0</td>
      <td>1.63</td>
    </tr>
    <tr>
      <th>2</th>
      <td>33.30</td>
      <td>44.345838</td>
      <td>baghdad</td>
      <td>iq</td>
      <td>57.20</td>
      <td>58.0</td>
      <td>0.0</td>
      <td>2.24</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-54.81</td>
      <td>-37.357866</td>
      <td>ushuaia</td>
      <td>ar</td>
      <td>42.80</td>
      <td>75.0</td>
      <td>40.0</td>
      <td>12.75</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-2.19</td>
      <td>-57.363607</td>
      <td>nhamunda</td>
      <td>br</td>
      <td>80.69</td>
      <td>76.0</td>
      <td>44.0</td>
      <td>4.21</td>
    </tr>
  </tbody>
</table>
</div>



# Scatter Plot- Temperature vs. Latitude


```python
x= weather_df["Latitude"]
y= weather_df["Temperature (F)"]
plt.scatter (x,y, edgecolor = 'black')
plt.title('Temperature (F) vs. Latitude')
plt.ylabel("Temperature (F)")
plt.xlabel('Latitude')

plt.savefig("temp_lat.png")
plt.show()
```


![png](output_8_0.png)


# Scatter Plot- Humidity(%) vs. Latitude


```python
x= weather_df["Latitude"]
y= weather_df["Humidity (%)"]
plt.scatter (x,y, edgecolor = 'black')
plt.title('Humidity vs. Latitude')
plt.ylabel("Humidity(%)")
plt.xlabel('Latitude')
plt.savefig("hum_lat.png")
plt.show()

```


![png](output_10_0.png)


# Scatter Plot- Cloudiness vs. Latitude


```python
x= weather_df["Latitude"]
y= weather_df["Cloudiness (%)"]
plt.scatter (x,y,edgecolor = 'black')
plt.title('Cloudiness (%) vs. Latitude')
plt.ylabel("Cloudiness (%)")
plt.xlabel('Latitude')
plt.savefig("lat_cloud.png")
plt.show()

```


![png](output_12_0.png)


# Wind Speed (mph) vs. Latitude


```python
x= weather_df["Latitude"]
y= weather_df["Wind Speed (mph)"]
plt.scatter (x,y,edgecolor = 'black')
plt.title('Wind Speed (mph) (%) vs. Latitude')
plt.ylabel("Wind Speed (mph)")
plt.xlabel('Latitude')
plt.savefig("lat_wind.png")
plt.show()

```


![png](output_14_0.png)

