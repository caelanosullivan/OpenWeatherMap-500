

```python
"""OpenWeatherMap-randomization

This module employs Python requests, APIs, and JSON traversals to answer a fundamental question: what weather trends can
we observe as we approach the equator? Using the Citipy library and OpenWeatherMap API, this script creates a representative
model of weather across world cities.

The program randomly generates 1500 latitude/longitude pairs and feeds them to the OpenWeatherMapAPI. It performs a
series of successive throttled API calls to extract the weather data in all found cities. It then drops duplicate 
cities, counts the total, and saves the resulting table as a CSV.

To showcase the following relationships, it builds a series of scatter plots using Matplotlib:

Temperature (F) vs. Latitude
Humidity (%) vs. Latitude
Cloudiness (%) vs. Latitude
Wind Speed (mph) vs. Latitude

Finally, in addition to the city data CSV, it returns the scatterplots as png image files.

Helpful documentation:
citipy library - http://desktop.arcgis.com/en/arcmap/10.3/guide-books/map-projections/about-geographic-coordinate-systems.htm
OpenWeatherMap API - https://pypi.python.org/pypi/citipy
geographical coordinate system - https://openweathermap.org/api


"""
```




    'OpenWeatherMap-randomization\n\nThis module employs Python requests, APIs, and JSON traversals to answer a fundamental question: what weather trends can\nwe observe as we approach the equator? Using the Citipy library and OpenWeatherMap API, this script creates a representative\nmodel of weather across world cities.\n\nThe program randomly generates 1500 latitude/longitude pairs and feeds them to the OpenWeatherMapAPI. It performs a\nseries of successive throttled API calls to extract the weather data in all found cities. It then drops duplicate \ncities, counts the total, and saves the resulting table as a CSV.\n\nTo showcase the following relationships, it builds a series of scatter plots using Matplotlib:\n\nTemperature (F) vs. Latitude\nHumidity (%) vs. Latitude\nCloudiness (%) vs. Latitude\nWind Speed (mph) vs. Latitude\n\nFinally, in addition to the city data CSV, it returns the scatterplots as png image files.\n\nHelpful documentation:\ncitipy library - http://desktop.arcgis.com/en/arcmap/10.3/guide-books/map-projections/about-geographic-coordinate-systems.htm\nOpenWeatherMap API - https://pypi.python.org/pypi/citipy\ngeographical coordinate system - https://openweathermap.org/api\n\n\n'



# Analysis

* Max temperatures peak around latitudes clustered around the equator (0 degrees). The temperature plot is not an inverted parabola as expected, i.e. low temperatures are not evenly exhibited in both the highest and lowest latitudes. This makes sense given hemisphere seasonality -- depending on time of year, either the lowest or highest latitudes should exhibit an uptick based on where it's summer.
* Humidity is mostly evenly distributed across latitudes. Some runs of the program showed very low humidities between -25 and 25 degrees.
* Cloudiness distribution appears highly randomized.
* Windspeed appears to increase at the extreme latitudes, possibly exhibiting seasonality like max temperature. Windspeeds appear slightly higher on the extreme side currently with lower max temperatures (winter). It could be useful to further examine the seasonal relationship between temperature and windspeed.


```python
# Import Dependencies
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import json
import requests
import datetime
import time
from citipy import citipy

#Retrieve API keys
from config import api_key
```


```python
# Randomly generate 1500 latitude/longitude pairs
samples = 1500
lats = np.random.uniform(low=-90.000, high=90.000, size=samples) 
lngs = np.random.uniform(low=-180.000, high=180.000, size=samples)

# Convert to list for iteration
lats = list(lats)
lngs = list(lngs)

```


```python
# Create empty lists to hold weather data for found cities
cities = []
countries = []
timestamps = []
dates = []
max_temps = []
humidities = []
windspeeds = []
cloudiness = []


# Iterate through lat/long pairs, feeding to Citipy to generate nearest city name for later use with OpenWeatherMap
j = 0
while j < len(lats):
    print(lats[j], lngs[j])
    
    one_city = citipy.nearest_city(lats[j], lngs[j])
    city_name = one_city.city_name
    city_country_code = one_city.country_code
    
    print(f'Processing Record {j+1} of Set 1 | {city_name}')
    
    # Use city name to build OpenWeatherMap API query
    
    query = city_name + ',' + city_country_code
    base_url = "http://api.openweathermap.org/data/2.5/weather"
    params = {
            "q": query,
            "units": "IMPERIAL",
            "mode": "json",
            "APPID": api_key
            }
    response = requests.get(base_url, params=params)
    city_weather = response.json()

    # Use try/except to skip cities not found by both Citipy and OpenWeatherMap
    # If contained in OpenWeatherMap data, parse JSON for city weather data and append to respective lists 
    try:

        timestamps.append(city_weather['dt'])
        dates.append(datetime.datetime.fromtimestamp\
                     (int(city_weather['dt'])).strftime('%m-%d-%Y'))
        max_temps.append(city_weather['main']['temp'])
        humidities.append(city_weather['main']['humidity'])
        windspeeds.append(city_weather['wind']['speed'])
        cloudiness.append(city_weather['clouds']['all'])
        
        cities.append(one_city.city_name)
        countries.append(one_city.country_code)
        
        j += 1
    
    # If city not found in OpenWeatherMap, report it
    except KeyError:
        del lats[j]
        del lngs[j]
        print("Oops, that city doesn't exist in OpenWeatherMap.")
    
    # Slow the loop down by waiting 2 seconds to conform to OpenWeatherMap's limitation of 60 requests per minute
    time.sleep(2)
```

    -17.4028740197 36.1812674629
    Processing Record 1 of Set 1 | quelimane
    -28.0621492479 -176.011470618
    Processing Record 2 of Set 1 | vaini
    70.9452530373 146.96207524
    Processing Record 3 of Set 1 | chokurdakh
    -38.1698721246 -73.4691202368
    Processing Record 4 of Set 1 | canete
    26.247892803 145.265614403
    Processing Record 5 of Set 1 | katsuura
    -6.56762567778 -82.8933309173
    Processing Record 6 of Set 1 | sechura
    34.2487999469 81.7490437401
    Processing Record 7 of Set 1 | leh
    -72.2613807939 -42.286647366
    Processing Record 8 of Set 1 | ushuaia
    -70.3577581492 -130.564514935
    Processing Record 9 of Set 1 | rikitea
    44.2805188971 -67.3621982541
    Processing Record 10 of Set 1 | bar harbor
    -17.8105661668 -91.4871254003
    Processing Record 11 of Set 1 | hualmay
    73.8928301558 -174.145641599
    Processing Record 12 of Set 1 | mys shmidta
    Oops, that city doesn't exist in OpenWeatherMap.
    36.3025539076 -7.4815753637
    Processing Record 12 of Set 1 | olhao
    -38.795327888 -46.5351541132
    Processing Record 13 of Set 1 | chuy
    -17.4740955908 137.575559162
    Processing Record 14 of Set 1 | mount isa
    27.6931559763 -24.9324481577
    Processing Record 15 of Set 1 | los llanos de aridane
    81.8049375495 -118.22139671
    Processing Record 16 of Set 1 | norman wells
    -63.9458303709 -27.3045479181
    Processing Record 17 of Set 1 | mar del plata
    -56.1997437449 -175.914073609
    Processing Record 18 of Set 1 | vaini
    -50.056634091 96.7357027144
    Processing Record 19 of Set 1 | busselton
    -64.0460117539 -64.1547627211
    Processing Record 20 of Set 1 | ushuaia
    -53.8732170435 -179.17521561
    Processing Record 21 of Set 1 | vaini
    -81.2146170244 151.646741601
    Processing Record 22 of Set 1 | bluff
    -2.7536549395 -61.8783886989
    Processing Record 23 of Set 1 | anori
    83.0384583861 151.802918339
    Processing Record 24 of Set 1 | chokurdakh
    45.4409531425 178.502294906
    Processing Record 25 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    75.6174025122 80.9081182247
    Processing Record 25 of Set 1 | dikson
    16.6874939121 -32.6667016201
    Processing Record 26 of Set 1 | ponta do sol
    -79.2689521812 -114.935034244
    Processing Record 27 of Set 1 | punta arenas
    44.4650296069 -109.221033902
    Processing Record 28 of Set 1 | cody
    27.8545073973 38.9280911282
    Processing Record 29 of Set 1 | tabuk
    58.0185429785 -100.967298819
    Processing Record 30 of Set 1 | flin flon
    47.3729573297 49.6892494293
    Processing Record 31 of Set 1 | marfino
    48.8815416315 -39.8052167363
    Processing Record 32 of Set 1 | nanortalik
    -75.5575700831 -82.0995677269
    Processing Record 33 of Set 1 | ushuaia
    -15.9956238501 -123.308166597
    Processing Record 34 of Set 1 | rikitea
    -0.00284066508399 -73.2733636015
    Processing Record 35 of Set 1 | puerto leguizamo
    46.6685486126 117.151883116
    Processing Record 36 of Set 1 | manzhouli
    -37.5721123267 -34.9171542676
    Processing Record 37 of Set 1 | arraial do cabo
    77.2962416245 -61.7438734633
    Processing Record 38 of Set 1 | narsaq
    5.91821280911 142.353294318
    Processing Record 39 of Set 1 | airai
    Oops, that city doesn't exist in OpenWeatherMap.
    -43.1165200657 155.704308009
    Processing Record 39 of Set 1 | hobart
    72.0854928398 51.3076343279
    Processing Record 40 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    -55.6525661606 43.5842584349
    Processing Record 40 of Set 1 | east london
    80.5584613135 101.923739289
    Processing Record 41 of Set 1 | khatanga
    -3.94368883129 -122.426837677
    Processing Record 42 of Set 1 | atuona
    65.1773152756 -13.4963110672
    Processing Record 43 of Set 1 | hofn
    -55.920045305 -29.3205673862
    Processing Record 44 of Set 1 | chuy
    30.3672883357 -99.4296345908
    Processing Record 45 of Set 1 | kerrville
    3.09312965314 -95.755418904
    Processing Record 46 of Set 1 | puerto ayora
    78.3764915361 42.7912834303
    Processing Record 47 of Set 1 | ostrovnoy
    44.2865678383 173.963336685
    Processing Record 48 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    83.0994869353 -53.5418586125
    Processing Record 48 of Set 1 | upernavik
    -34.0456664191 141.701823974
    Processing Record 49 of Set 1 | mildura
    -85.2475555529 -72.3244549164
    Processing Record 50 of Set 1 | ushuaia
    -10.5000924755 -14.6788057083
    Processing Record 51 of Set 1 | georgetown
    33.7209265323 -176.41046613
    Processing Record 52 of Set 1 | kapaa
    1.4662356195 -147.057641154
    Processing Record 53 of Set 1 | atuona
    -51.3152082335 -73.6811883143
    Processing Record 54 of Set 1 | punta arenas
    21.3617712798 -126.521459468
    Processing Record 55 of Set 1 | san quintin
    53.7411861024 -14.3312906773
    Processing Record 56 of Set 1 | dingle
    84.8762196554 -63.8816419732
    Processing Record 57 of Set 1 | narsaq
    -78.1832924719 -81.573263928
    Processing Record 58 of Set 1 | ushuaia
    -83.1698895451 -106.996213285
    Processing Record 59 of Set 1 | punta arenas
    -42.1303866508 87.3954678322
    Processing Record 60 of Set 1 | busselton
    23.5743623971 -160.544236609
    Processing Record 61 of Set 1 | kapaa
    4.80924283699 107.301290833
    Processing Record 62 of Set 1 | kerteh
    Oops, that city doesn't exist in OpenWeatherMap.
    -52.8264336587 -35.2370483678
    Processing Record 62 of Set 1 | chuy
    -36.8462323622 59.0773785341
    Processing Record 63 of Set 1 | saint-philippe
    53.8118400316 -92.7264311588
    Processing Record 64 of Set 1 | sioux lookout
    -49.5730154529 162.25832349
    Processing Record 65 of Set 1 | tuatapere
    -29.6108736034 170.099370849
    Processing Record 66 of Set 1 | ahipara
    -10.9697945182 -148.162758138
    Processing Record 67 of Set 1 | fare
    8.22107991099 -167.039232078
    Processing Record 68 of Set 1 | kapaa
    14.4357235091 24.707659273
    Processing Record 69 of Set 1 | kutum
    19.3859969176 -107.06867372
    Processing Record 70 of Set 1 | tomatlan
    -34.0985128343 -85.882808177
    Processing Record 71 of Set 1 | lebu
    -57.9081798929 130.506169694
    Processing Record 72 of Set 1 | new norfolk
    -14.0746878549 -150.544997923
    Processing Record 73 of Set 1 | fare
    38.3206372398 72.3087466984
    Processing Record 74 of Set 1 | karakendzha
    Oops, that city doesn't exist in OpenWeatherMap.
    67.1713371652 -5.95200058687
    Processing Record 74 of Set 1 | klaksvik
    54.2215650776 -91.7158808601
    Processing Record 75 of Set 1 | sioux lookout
    -0.0142648338431 9.30029240699
    Processing Record 76 of Set 1 | libreville
    49.714721185 -77.361163079
    Processing Record 77 of Set 1 | matagami
    0.652931531106 140.591034222
    Processing Record 78 of Set 1 | vanimo
    -77.7818542616 -123.089395842
    Processing Record 79 of Set 1 | rikitea
    75.1787159553 117.35568944
    Processing Record 80 of Set 1 | saskylakh
    -56.5721955084 115.43217443
    Processing Record 81 of Set 1 | albany
    -85.0467582453 42.746202665
    Processing Record 82 of Set 1 | port alfred
    11.7270080741 174.797472516
    Processing Record 83 of Set 1 | butaritari
    66.7359488596 147.923366775
    Processing Record 84 of Set 1 | belaya gora
    63.1586746688 70.5742802987
    Processing Record 85 of Set 1 | cheuskiny
    Oops, that city doesn't exist in OpenWeatherMap.
    22.3536187915 146.347327079
    Processing Record 85 of Set 1 | katsuura
    5.29725360817 -142.317700918
    Processing Record 86 of Set 1 | atuona
    -78.7819261359 16.3711502827
    Processing Record 87 of Set 1 | bredasdorp
    -86.0458424084 109.359791221
    Processing Record 88 of Set 1 | albany
    84.0174309017 136.624515301
    Processing Record 89 of Set 1 | nizhneyansk
    Oops, that city doesn't exist in OpenWeatherMap.
    88.0443635722 111.487113301
    Processing Record 89 of Set 1 | saskylakh
    86.6459395797 -48.458447173
    Processing Record 90 of Set 1 | upernavik
    77.4393131619 118.658340445
    Processing Record 91 of Set 1 | saskylakh
    -88.0785288996 -35.8437026659
    Processing Record 92 of Set 1 | ushuaia
    -44.5325429196 -112.085895748
    Processing Record 93 of Set 1 | rikitea
    36.4689484532 152.111918531
    Processing Record 94 of Set 1 | nemuro
    12.4047398408 23.8573140641
    Processing Record 95 of Set 1 | adre
    21.2060991665 134.513768176
    Processing Record 96 of Set 1 | nishihara
    40.850960131 143.161290584
    Processing Record 97 of Set 1 | shizunai
    81.1315687944 118.738063335
    Processing Record 98 of Set 1 | saskylakh
    -2.05718236117 -160.966897792
    Processing Record 99 of Set 1 | samusu
    Oops, that city doesn't exist in OpenWeatherMap.
    5.22916871781 -82.3628016669
    Processing Record 99 of Set 1 | burica
    Oops, that city doesn't exist in OpenWeatherMap.
    -88.7894743742 -84.9845406151
    Processing Record 99 of Set 1 | ushuaia
    -81.4976988282 -12.4518457928
    Processing Record 100 of Set 1 | cape town
    -41.4222514219 -69.3773883065
    Processing Record 101 of Set 1 | san carlos de bariloche
    58.0774838005 -49.4529692288
    Processing Record 102 of Set 1 | paamiut
    43.8358079085 158.347432074
    Processing Record 103 of Set 1 | severo-kurilsk
    -38.6673511288 -159.81802672
    Processing Record 104 of Set 1 | avarua
    55.6535333864 99.6399702201
    Processing Record 105 of Set 1 | lesogorsk
    5.55199422769 -166.632484503
    Processing Record 106 of Set 1 | makakilo city
    -21.7645256143 106.484059229
    Processing Record 107 of Set 1 | carnarvon
    -54.1986653199 16.0485293482
    Processing Record 108 of Set 1 | hermanus
    71.0161161374 -96.341916153
    Processing Record 109 of Set 1 | thompson
    -70.6213953261 -87.0037371989
    Processing Record 110 of Set 1 | punta arenas
    34.8421116698 43.9095603523
    Processing Record 111 of Set 1 | tikrit
    69.0112682837 -105.754117132
    Processing Record 112 of Set 1 | yellowknife
    -77.3657933808 -108.985456164
    Processing Record 113 of Set 1 | punta arenas
    0.917167757746 160.781386017
    Processing Record 114 of Set 1 | kieta
    9.48798876677 -114.668511855
    Processing Record 115 of Set 1 | san patricio
    51.8875988615 10.2689453297
    Processing Record 116 of Set 1 | langelsheim
    -18.9982085859 102.737415022
    Processing Record 117 of Set 1 | carnarvon
    3.10911547629 29.2287369087
    Processing Record 118 of Set 1 | watsa
    -17.1841169414 112.437074283
    Processing Record 119 of Set 1 | karratha
    -37.5411169304 167.529290807
    Processing Record 120 of Set 1 | westport
    49.7358177834 43.0841760491
    Processing Record 121 of Set 1 | mikhaylovka
    2.63851942865 80.2174190216
    Processing Record 122 of Set 1 | matara
    23.2590084509 -42.8767532624
    Processing Record 123 of Set 1 | ponta do sol
    -34.0979719467 -21.5732011965
    Processing Record 124 of Set 1 | sao joao da barra
    -48.4290670658 -69.250291021
    Processing Record 125 of Set 1 | comodoro rivadavia
    -3.7724173341 76.6746893952
    Processing Record 126 of Set 1 | hithadhoo
    48.8714247726 -70.7309893129
    Processing Record 127 of Set 1 | bagotville
    Oops, that city doesn't exist in OpenWeatherMap.
    -11.1753583598 -105.215407064
    Processing Record 127 of Set 1 | puerto ayora
    -34.7463642483 0.484271268994
    Processing Record 128 of Set 1 | luderitz
    -59.0528060521 68.1018997637
    Processing Record 129 of Set 1 | saint-philippe
    5.38677008396 -125.975922352
    Processing Record 130 of Set 1 | atuona
    -62.905887036 126.539426613
    Processing Record 131 of Set 1 | new norfolk
    -5.56255957476 102.512970585
    Processing Record 132 of Set 1 | bengkulu
    0.119815201897 -10.5918444457
    Processing Record 133 of Set 1 | harper
    -63.5765151517 99.5956747476
    Processing Record 134 of Set 1 | busselton
    39.6861346238 78.9842550819
    Processing Record 135 of Set 1 | aksu
    -75.0493397999 -13.9788889191
    Processing Record 136 of Set 1 | cape town
    0.528498676836 31.3596179618
    Processing Record 137 of Set 1 | mubende
    40.9263156149 -52.0704417077
    Processing Record 138 of Set 1 | torbay
    85.3762424133 -98.2097655494
    Processing Record 139 of Set 1 | yellowknife
    -50.8087441244 -129.98191607
    Processing Record 140 of Set 1 | rikitea
    -73.6974333622 89.3792968926
    Processing Record 141 of Set 1 | busselton
    -32.0376495576 93.0551746149
    Processing Record 142 of Set 1 | geraldton
    81.7740438201 -92.3505589139
    Processing Record 143 of Set 1 | qaanaaq
    44.0234379935 23.6339803909
    Processing Record 144 of Set 1 | giurgita
    81.305972061 -107.463455119
    Processing Record 145 of Set 1 | yellowknife
    68.6031876594 132.515617775
    Processing Record 146 of Set 1 | verkhoyansk
    -30.6788415122 16.9403597626
    Processing Record 147 of Set 1 | springbok
    -75.4225199263 64.7937357109
    Processing Record 148 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    63.8555579529 -52.8126039569
    Processing Record 148 of Set 1 | nuuk
    -27.9760958571 6.58600298069
    Processing Record 149 of Set 1 | luderitz
    -38.8585576437 -84.2282295481
    Processing Record 150 of Set 1 | lebu
    -82.7634467647 33.2561641508
    Processing Record 151 of Set 1 | port elizabeth
    81.4913303393 177.94015218
    Processing Record 152 of Set 1 | leningradskiy
    -55.2646363642 81.4828509827
    Processing Record 153 of Set 1 | busselton
    16.2271126794 162.004072448
    Processing Record 154 of Set 1 | butaritari
    40.7532052575 -36.2221172255
    Processing Record 155 of Set 1 | ribeira grande
    3.76926986536 175.616376274
    Processing Record 156 of Set 1 | butaritari
    37.6176368457 -155.528048858
    Processing Record 157 of Set 1 | kapaa
    76.3949928977 -59.0491828502
    Processing Record 158 of Set 1 | upernavik
    21.2881370319 175.170539727
    Processing Record 159 of Set 1 | butaritari
    7.15206778671 39.6731516445
    Processing Record 160 of Set 1 | goba
    66.2082390731 -129.68086818
    Processing Record 161 of Set 1 | norman wells
    63.2534686214 32.7168944968
    Processing Record 162 of Set 1 | porosozero
    4.98258636681 109.171555415
    Processing Record 163 of Set 1 | kuching
    48.0211660859 -27.9787297978
    Processing Record 164 of Set 1 | lagoa
    35.4996278296 158.782697467
    Processing Record 165 of Set 1 | sentyabrskiy
    Oops, that city doesn't exist in OpenWeatherMap.
    27.709027839 68.2982497607
    Processing Record 165 of Set 1 | naudero
    31.4198566306 -107.349194238
    Processing Record 166 of Set 1 | puerto palomas
    -26.1216086039 -107.467494391
    Processing Record 167 of Set 1 | rikitea
    48.7084963008 -29.2553650684
    Processing Record 168 of Set 1 | lagoa
    12.8897707212 100.486145676
    Processing Record 169 of Set 1 | pattaya
    Oops, that city doesn't exist in OpenWeatherMap.
    -5.60058445441 -56.9263411623
    Processing Record 169 of Set 1 | jacareacanga
    65.3643766967 -81.8344175619
    Processing Record 170 of Set 1 | attawapiskat
    Oops, that city doesn't exist in OpenWeatherMap.
    -87.7458731708 -127.16294961
    Processing Record 170 of Set 1 | rikitea
    -60.5010951296 161.403676345
    Processing Record 171 of Set 1 | bluff
    25.3722618271 115.568209227
    Processing Record 172 of Set 1 | ganzhou
    -19.3475793352 -5.33843502356
    Processing Record 173 of Set 1 | jamestown
    -52.698361603 -25.6675366731
    Processing Record 174 of Set 1 | cidreira
    -63.1754639138 109.841414977
    Processing Record 175 of Set 1 | albany
    -64.1828284989 135.935334322
    Processing Record 176 of Set 1 | new norfolk
    63.8695577753 33.3989881879
    Processing Record 177 of Set 1 | segezha
    5.33858162928 -177.521751237
    Processing Record 178 of Set 1 | vaitupu
    Oops, that city doesn't exist in OpenWeatherMap.
    -86.8462396976 158.093439973
    Processing Record 178 of Set 1 | bluff
    42.7074490481 41.2615838842
    Processing Record 179 of Set 1 | ochamchira
    Oops, that city doesn't exist in OpenWeatherMap.
    71.4797571912 -86.8932549705
    Processing Record 179 of Set 1 | clyde river
    -50.7332510021 19.4391274776
    Processing Record 180 of Set 1 | bredasdorp
    60.8390518024 170.501700766
    Processing Record 181 of Set 1 | tilichiki
    -45.5322476167 -90.7966302872
    Processing Record 182 of Set 1 | castro
    -4.11484950387 -12.2734678642
    Processing Record 183 of Set 1 | georgetown
    36.5146801445 -132.844286684
    Processing Record 184 of Set 1 | fortuna
    27.2384502247 96.3351780994
    Processing Record 185 of Set 1 | margherita
    87.5145042838 26.0606407615
    Processing Record 186 of Set 1 | longyearbyen
    79.7120711419 55.5064379256
    Processing Record 187 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    -76.0300131204 -51.9618990747
    Processing Record 187 of Set 1 | ushuaia
    47.0763833711 7.22358884052
    Processing Record 188 of Set 1 | nidau
    -55.4332279655 -61.7890666576
    Processing Record 189 of Set 1 | ushuaia
    -4.01031440574 -87.2444792822
    Processing Record 190 of Set 1 | san cristobal
    -6.53255170078 16.4804923531
    Processing Record 191 of Set 1 | kasongo-lunda
    9.32351120604 152.695861746
    Processing Record 192 of Set 1 | kavieng
    2.98229630302 177.865263886
    Processing Record 193 of Set 1 | rungata
    Oops, that city doesn't exist in OpenWeatherMap.
    5.42313461077 -120.350467251
    Processing Record 193 of Set 1 | cabo san lucas
    64.8149684228 36.5166356303
    Processing Record 194 of Set 1 | solovetskiy
    Oops, that city doesn't exist in OpenWeatherMap.
    -13.3143179664 -108.248297408
    Processing Record 194 of Set 1 | puerto ayora
    -50.764076039 160.49331638
    Processing Record 195 of Set 1 | tuatapere
    -66.384549994 14.1907052317
    Processing Record 196 of Set 1 | hermanus
    42.605372687 131.696631748
    Processing Record 197 of Set 1 | popova
    -39.4799719903 -88.6657611283
    Processing Record 198 of Set 1 | ancud
    -35.6524349766 -81.4971409632
    Processing Record 199 of Set 1 | lebu
    33.0767764142 30.6721900209
    Processing Record 200 of Set 1 | rosetta
    -30.6105665661 90.2887243708
    Processing Record 201 of Set 1 | carnarvon
    -1.40030855494 100.876718531
    Processing Record 202 of Set 1 | solok
    -59.9511232071 143.271494354
    Processing Record 203 of Set 1 | hobart
    -70.787334605 -168.332237396
    Processing Record 204 of Set 1 | vaini
    -84.2937381994 -105.848667148
    Processing Record 205 of Set 1 | punta arenas
    82.2884109453 -153.910951354
    Processing Record 206 of Set 1 | barrow
    -74.8186884557 -99.0047079688
    Processing Record 207 of Set 1 | punta arenas
    -80.4892202297 177.58470108
    Processing Record 208 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    -17.7796642551 -37.1192612322
    Processing Record 208 of Set 1 | caravelas
    53.6647652678 -95.7623510749
    Processing Record 209 of Set 1 | thompson
    -61.2820177412 -72.2912604538
    Processing Record 210 of Set 1 | ushuaia
    -26.9845838566 43.2571097174
    Processing Record 211 of Set 1 | beloha
    27.7359672723 -19.2237218375
    Processing Record 212 of Set 1 | los llanos de aridane
    14.2933863441 128.393522313
    Processing Record 213 of Set 1 | alugan
    -62.3907847629 161.316184506
    Processing Record 214 of Set 1 | bluff
    74.3985659627 -5.42441693286
    Processing Record 215 of Set 1 | klaksvik
    -15.8985833748 139.98805075
    Processing Record 216 of Set 1 | alyangula
    40.0098925471 115.891592909
    Processing Record 217 of Set 1 | mentougou
    20.5623749447 -16.8689226819
    Processing Record 218 of Set 1 | nouadhibou
    77.0219094763 103.968924944
    Processing Record 219 of Set 1 | khatanga
    67.3608977582 63.6144092218
    Processing Record 220 of Set 1 | zapolyarnyy
    -51.2336590139 16.0687716207
    Processing Record 221 of Set 1 | hermanus
    -49.0943499241 107.270485754
    Processing Record 222 of Set 1 | busselton
    89.2215602158 107.49915841
    Processing Record 223 of Set 1 | khatanga
    -44.8185782988 -41.644249621
    Processing Record 224 of Set 1 | chuy
    80.7326361792 51.6568362839
    Processing Record 225 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    -60.6235011092 -8.36478776924
    Processing Record 225 of Set 1 | cape town
    -73.8123004851 -21.3869261872
    Processing Record 226 of Set 1 | ushuaia
    -23.757240484 -25.8882032014
    Processing Record 227 of Set 1 | caravelas
    50.3363946028 -165.867654098
    Processing Record 228 of Set 1 | bethel
    -87.8673895624 110.924868845
    Processing Record 229 of Set 1 | albany
    78.9880628524 173.850602169
    Processing Record 230 of Set 1 | komsomolskiy
    38.654626081 -145.003517546
    Processing Record 231 of Set 1 | kodiak
    -14.2670143742 -179.082956043
    Processing Record 232 of Set 1 | halalo
    Oops, that city doesn't exist in OpenWeatherMap.
    -63.8208535994 148.924294696
    Processing Record 232 of Set 1 | hobart
    19.1103094842 -111.678995419
    Processing Record 233 of Set 1 | cabo san lucas
    32.0471934664 -162.264560487
    Processing Record 234 of Set 1 | kapaa
    46.3334281683 -1.14778337308
    Processing Record 235 of Set 1 | la rochelle
    73.1626439724 -81.6090182511
    Processing Record 236 of Set 1 | qaanaaq
    -49.3648662665 -0.65582229621
    Processing Record 237 of Set 1 | cape town
    -76.1653203509 177.888266991
    Processing Record 238 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    29.1521989241 152.970623669
    Processing Record 238 of Set 1 | hasaki
    -82.5990272497 21.6739486301
    Processing Record 239 of Set 1 | bredasdorp
    42.9513963133 -120.635766025
    Processing Record 240 of Set 1 | bend
    -75.5437054273 45.1766702945
    Processing Record 241 of Set 1 | port alfred
    -16.3656646964 113.37836085
    Processing Record 242 of Set 1 | karratha
    13.3571566315 45.5516589235
    Processing Record 243 of Set 1 | lahij
    37.2961165793 -15.2647062395
    Processing Record 244 of Set 1 | camacha
    -59.7223602399 -140.600827154
    Processing Record 245 of Set 1 | rikitea
    -31.3590478437 -54.0106383687
    Processing Record 246 of Set 1 | bage
    -39.6436817345 20.6892300932
    Processing Record 247 of Set 1 | bredasdorp
    -31.6698306571 7.28249714666
    Processing Record 248 of Set 1 | luderitz
    75.4509640841 -152.222266391
    Processing Record 249 of Set 1 | barrow
    15.3047761047 37.2778244069
    Processing Record 250 of Set 1 | barentu
    68.1683955256 107.25687141
    Processing Record 251 of Set 1 | aykhal
    -66.1417470496 40.6652690871
    Processing Record 252 of Set 1 | port alfred
    8.31836973381 84.6444279743
    Processing Record 253 of Set 1 | kalmunai
    16.2135118911 44.4203655775
    Processing Record 254 of Set 1 | sayyan
    56.5724876932 6.97601409582
    Processing Record 255 of Set 1 | harboore
    25.4332848434 28.8097881903
    Processing Record 256 of Set 1 | asyut
    31.3770536099 32.2812943569
    Processing Record 257 of Set 1 | port said
    57.5212591455 -91.4391864108
    Processing Record 258 of Set 1 | thompson
    -21.0787102424 9.59122194156
    Processing Record 259 of Set 1 | henties bay
    Oops, that city doesn't exist in OpenWeatherMap.
    -36.5563398512 13.7089477003
    Processing Record 259 of Set 1 | cape town
    51.5277000085 -16.8294192949
    Processing Record 260 of Set 1 | dingle
    82.667013462 -95.6968519531
    Processing Record 261 of Set 1 | qaanaaq
    17.2664969056 1.84834935803
    Processing Record 262 of Set 1 | kidal
    4.19528215493 175.695491835
    Processing Record 263 of Set 1 | butaritari
    -16.835630918 -85.5809324747
    Processing Record 264 of Set 1 | lima
    51.9378413513 -87.06902531
    Processing Record 265 of Set 1 | longlac
    Oops, that city doesn't exist in OpenWeatherMap.
    -41.6348957289 -84.4994246827
    Processing Record 265 of Set 1 | ancud
    32.2840206539 178.762860231
    Processing Record 266 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    5.2417155489 -5.58818776507
    Processing Record 266 of Set 1 | grand-lahou
    -55.2219372648 -94.1700393639
    Processing Record 267 of Set 1 | punta arenas
    -6.17837061378 69.5083548584
    Processing Record 268 of Set 1 | hithadhoo
    28.2952759459 -178.249672322
    Processing Record 269 of Set 1 | kapaa
    15.8185184921 22.3785973035
    Processing Record 270 of Set 1 | biltine
    -77.3670169615 4.89627314462
    Processing Record 271 of Set 1 | hermanus
    -4.79930239819 17.5552957169
    Processing Record 272 of Set 1 | bulungu
    8.33353564931 116.652556326
    Processing Record 273 of Set 1 | balabac
    -89.42993005 128.04663931
    Processing Record 274 of Set 1 | new norfolk
    -3.64318029411 171.354268549
    Processing Record 275 of Set 1 | utiroa
    Oops, that city doesn't exist in OpenWeatherMap.
    -78.1443319977 67.8522093129
    Processing Record 275 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -24.3810568217 -6.37132095814
    Processing Record 275 of Set 1 | jamestown
    -75.8449381015 80.545368204
    Processing Record 276 of Set 1 | busselton
    -68.0354043178 168.472071273
    Processing Record 277 of Set 1 | bluff
    86.0608492393 -98.9580614153
    Processing Record 278 of Set 1 | yellowknife
    15.598328999 57.8695175403
    Processing Record 279 of Set 1 | salalah
    59.7536100301 -94.0940509991
    Processing Record 280 of Set 1 | thompson
    19.4489495112 -155.21086068
    Processing Record 281 of Set 1 | hilo
    25.8048145698 111.764098823
    Processing Record 282 of Set 1 | lengshuitan
    18.8046908831 -103.622448437
    Processing Record 283 of Set 1 | coahuayana
    22.0073961682 125.343880951
    Processing Record 284 of Set 1 | ishigaki
    -1.85976989797 19.4009990971
    Processing Record 285 of Set 1 | inongo
    34.6956776397 144.045890838
    Processing Record 286 of Set 1 | hasaki
    -23.4987809125 -134.722955172
    Processing Record 287 of Set 1 | rikitea
    -86.5532017288 -69.7034442558
    Processing Record 288 of Set 1 | ushuaia
    43.4818347281 -69.1590616729
    Processing Record 289 of Set 1 | rockland
    4.24894101408 67.9703823604
    Processing Record 290 of Set 1 | mahibadhoo
    -64.360011722 57.5518933332
    Processing Record 291 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -67.4231023456 83.6375352422
    Processing Record 291 of Set 1 | busselton
    -84.7688657742 -113.741965782
    Processing Record 292 of Set 1 | punta arenas
    -65.3430868348 131.466572283
    Processing Record 293 of Set 1 | new norfolk
    -83.2802136698 -19.9877943548
    Processing Record 294 of Set 1 | ushuaia
    34.9718259112 -14.0339541867
    Processing Record 295 of Set 1 | camacha
    -84.4746552872 2.11390378207
    Processing Record 296 of Set 1 | hermanus
    -69.3488404737 -122.969638746
    Processing Record 297 of Set 1 | rikitea
    84.2276218577 -87.4893981535
    Processing Record 298 of Set 1 | qaanaaq
    -34.9985919818 3.9807719304
    Processing Record 299 of Set 1 | luderitz
    66.7249973722 -175.75001237
    Processing Record 300 of Set 1 | provideniya
    9.98868013673 -104.449140028
    Processing Record 301 of Set 1 | ixtapa
    -54.0032207255 -167.475427711
    Processing Record 302 of Set 1 | avarua
    -89.8372166596 73.737550923
    Processing Record 303 of Set 1 | busselton
    64.6699928173 40.4888624152
    Processing Record 304 of Set 1 | arkhangelsk
    3.4834090513 5.23529323399
    Processing Record 305 of Set 1 | yenagoa
    55.4878499403 145.106710816
    Processing Record 306 of Set 1 | okha
    -8.92549438367 -87.2248609993
    Processing Record 307 of Set 1 | paita
    -16.7148749632 53.7539424631
    Processing Record 308 of Set 1 | ambodifototra
    Oops, that city doesn't exist in OpenWeatherMap.
    -63.2296133102 -120.76244213
    Processing Record 308 of Set 1 | rikitea
    -34.4132433304 5.77583706841
    Processing Record 309 of Set 1 | oranjemund
    -11.9998904768 178.507787844
    Processing Record 310 of Set 1 | asau
    Oops, that city doesn't exist in OpenWeatherMap.
    -58.4910256046 70.4036675527
    Processing Record 310 of Set 1 | saint-philippe
    -22.1661196271 -111.272914787
    Processing Record 311 of Set 1 | rikitea
    89.1068003952 31.2133158461
    Processing Record 312 of Set 1 | berlevag
    28.6952100027 -134.550751508
    Processing Record 313 of Set 1 | pacific grove
    36.1118242042 81.1427743581
    Processing Record 314 of Set 1 | leh
    53.8846698219 -144.106728368
    Processing Record 315 of Set 1 | kodiak
    -53.3362063099 -63.8594112772
    Processing Record 316 of Set 1 | ushuaia
    -27.7362346748 -72.0565476725
    Processing Record 317 of Set 1 | vallenar
    -41.2759427085 -75.8033389272
    Processing Record 318 of Set 1 | ancud
    36.9988432762 -168.934471604
    Processing Record 319 of Set 1 | kapaa
    7.10598389718 -131.633005135
    Processing Record 320 of Set 1 | atuona
    -57.7481498283 -139.436878175
    Processing Record 321 of Set 1 | rikitea
    12.0396268349 -151.12160459
    Processing Record 322 of Set 1 | hilo
    13.004057522 -116.494493409
    Processing Record 323 of Set 1 | cabo san lucas
    20.6727082694 72.797149972
    Processing Record 324 of Set 1 | valsad
    -73.179892533 -158.925115638
    Processing Record 325 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -48.7815616642 -16.1795810738
    Processing Record 325 of Set 1 | jamestown
    68.7796770762 -1.72606346386
    Processing Record 326 of Set 1 | klaksvik
    -15.9646375381 100.670595935
    Processing Record 327 of Set 1 | palabuhanratu
    Oops, that city doesn't exist in OpenWeatherMap.
    -83.4749281591 -36.2554366506
    Processing Record 327 of Set 1 | ushuaia
    28.7019243298 -116.20879463
    Processing Record 328 of Set 1 | san quintin
    -19.2673806283 85.7530839215
    Processing Record 329 of Set 1 | hithadhoo
    36.639877094 37.2847376348
    Processing Record 330 of Set 1 | kilis
    40.7149699155 80.1449090895
    Processing Record 331 of Set 1 | aksu
    77.2220319768 115.031605313
    Processing Record 332 of Set 1 | saskylakh
    36.3960950698 -1.61757662434
    Processing Record 333 of Set 1 | nijar
    71.5889053416 -108.632791207
    Processing Record 334 of Set 1 | yellowknife
    71.9768370985 45.1859168061
    Processing Record 335 of Set 1 | kamenka
    28.8835955576 172.750903407
    Processing Record 336 of Set 1 | butaritari
    -69.8241434125 13.8832066101
    Processing Record 337 of Set 1 | hermanus
    75.0295543456 -166.365191572
    Processing Record 338 of Set 1 | barrow
    -9.71954774825 56.4766012447
    Processing Record 339 of Set 1 | victoria
    21.5255732201 -88.1629418725
    Processing Record 340 of Set 1 | panaba
    53.2256166688 40.0465443699
    Processing Record 341 of Set 1 | chaplygin
    -74.1766219857 81.5957156268
    Processing Record 342 of Set 1 | busselton
    -12.1810489943 -83.6525043974
    Processing Record 343 of Set 1 | huarmey
    50.9245161332 4.57131373528
    Processing Record 344 of Set 1 | kampenhout
    -59.2226877146 173.118198642
    Processing Record 345 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    22.3762466713 -53.7739586758
    Processing Record 345 of Set 1 | codrington
    Oops, that city doesn't exist in OpenWeatherMap.
    -15.0067613623 -12.4673541623
    Processing Record 345 of Set 1 | jamestown
    -71.2291642784 169.596803368
    Processing Record 346 of Set 1 | bluff
    0.606156618508 31.4945215524
    Processing Record 347 of Set 1 | mubende
    53.332229793 -25.5453125805
    Processing Record 348 of Set 1 | grindavik
    -65.3591530313 116.06573842
    Processing Record 349 of Set 1 | albany
    80.0833029731 -54.0741090376
    Processing Record 350 of Set 1 | upernavik
    47.9629527557 -0.781691866594
    Processing Record 351 of Set 1 | laval
    -43.0338187849 64.816386027
    Processing Record 352 of Set 1 | saint-philippe
    79.3313964253 -86.0659632996
    Processing Record 353 of Set 1 | qaanaaq
    -19.2667417448 -161.622051749
    Processing Record 354 of Set 1 | avarua
    5.83532989889 51.2098225199
    Processing Record 355 of Set 1 | eyl
    -0.614045072291 19.9061548534
    Processing Record 356 of Set 1 | boende
    -44.5635474873 -57.174125763
    Processing Record 357 of Set 1 | necochea
    51.719949535 -137.552329131
    Processing Record 358 of Set 1 | sitka
    -83.6145160036 14.1141727627
    Processing Record 359 of Set 1 | bredasdorp
    85.6055164942 118.200850579
    Processing Record 360 of Set 1 | saskylakh
    52.8829740505 -25.8090397538
    Processing Record 361 of Set 1 | grindavik
    27.0502267246 -114.855905031
    Processing Record 362 of Set 1 | guerrero negro
    47.1406835581 -18.2484882254
    Processing Record 363 of Set 1 | dingle
    -26.5732554546 -77.5470391737
    Processing Record 364 of Set 1 | coquimbo
    -47.8601028565 -23.1758967439
    Processing Record 365 of Set 1 | arraial do cabo
    51.2063220444 -31.2426020345
    Processing Record 366 of Set 1 | lagoa
    -55.3190329366 71.3013204044
    Processing Record 367 of Set 1 | saint-philippe
    83.3314286191 157.492075181
    Processing Record 368 of Set 1 | cherskiy
    -0.0623021310797 -96.138715812
    Processing Record 369 of Set 1 | puerto ayora
    -48.3746958343 -150.961516217
    Processing Record 370 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -61.5398275004 -86.9051069392
    Processing Record 370 of Set 1 | punta arenas
    -63.3541331712 -49.8088112881
    Processing Record 371 of Set 1 | ushuaia
    -34.622517943 101.584733106
    Processing Record 372 of Set 1 | busselton
    31.5992627712 -171.386774277
    Processing Record 373 of Set 1 | kapaa
    -6.81792920128 38.4795629569
    Processing Record 374 of Set 1 | chalinze
    -78.1083955738 -74.0690763863
    Processing Record 375 of Set 1 | ushuaia
    -6.0900246069 39.6457571919
    Processing Record 376 of Set 1 | sokoni
    -13.7305570381 124.873817127
    Processing Record 377 of Set 1 | kupang
    -31.302997075 -99.0738347129
    Processing Record 378 of Set 1 | lebu
    83.4866649771 140.89948262
    Processing Record 379 of Set 1 | nizhneyansk
    Oops, that city doesn't exist in OpenWeatherMap.
    -57.1056429976 164.281067709
    Processing Record 379 of Set 1 | bluff
    13.7413079173 84.7918531913
    Processing Record 380 of Set 1 | yanam
    27.2943398517 -23.7781520017
    Processing Record 381 of Set 1 | los llanos de aridane
    -28.9618744893 102.192261594
    Processing Record 382 of Set 1 | carnarvon
    56.3362729905 143.710779167
    Processing Record 383 of Set 1 | okha
    -75.9467639006 -1.20207606961
    Processing Record 384 of Set 1 | hermanus
    67.6575412487 6.58194797066
    Processing Record 385 of Set 1 | sistranda
    -4.2135273501 -40.7710306926
    Processing Record 386 of Set 1 | guaraciaba do norte
    -51.8833533931 85.6664718069
    Processing Record 387 of Set 1 | busselton
    6.30370742786 38.1251051708
    Processing Record 388 of Set 1 | dilla
    Oops, that city doesn't exist in OpenWeatherMap.
    -37.1049681681 -73.3900996138
    Processing Record 388 of Set 1 | arauco
    61.4073863769 -156.284854969
    Processing Record 389 of Set 1 | homer
    21.5846359886 86.2439107648
    Processing Record 390 of Set 1 | udala
    Oops, that city doesn't exist in OpenWeatherMap.
    -83.1945352024 53.4206870646
    Processing Record 390 of Set 1 | port alfred
    35.7378376695 -149.673125291
    Processing Record 391 of Set 1 | kahului
    -27.8060919397 110.109105269
    Processing Record 392 of Set 1 | carnarvon
    75.5441007902 143.441968211
    Processing Record 393 of Set 1 | chokurdakh
    -22.3437228004 -45.9700860158
    Processing Record 394 of Set 1 | pouso alegre
    4.51621326705 78.7916238236
    Processing Record 395 of Set 1 | galle
    -82.8900072306 102.180934732
    Processing Record 396 of Set 1 | albany
    52.7815059593 -134.181844463
    Processing Record 397 of Set 1 | ketchikan
    72.534476131 -133.99434279
    Processing Record 398 of Set 1 | tuktoyaktuk
    47.4392316564 -44.4660683024
    Processing Record 399 of Set 1 | torbay
    -23.1055244956 -104.413154638
    Processing Record 400 of Set 1 | puerto ayora
    -18.3362513767 30.1803009742
    Processing Record 401 of Set 1 | chegutu
    -40.2168736669 -169.006154932
    Processing Record 402 of Set 1 | vaini
    -71.9320842956 -68.266064397
    Processing Record 403 of Set 1 | ushuaia
    -51.840259129 -96.0160627607
    Processing Record 404 of Set 1 | castro
    -15.2327849174 170.715253235
    Processing Record 405 of Set 1 | lakatoro
    67.2329432168 101.595826018
    Processing Record 406 of Set 1 | tura
    5.71232388639 123.299952745
    Processing Record 407 of Set 1 | palimbang
    40.0669725343 3.89825283692
    Processing Record 408 of Set 1 | mahon
    Oops, that city doesn't exist in OpenWeatherMap.
    -58.6332974638 123.848123256
    Processing Record 408 of Set 1 | albany
    55.6459575714 -90.0243772726
    Processing Record 409 of Set 1 | sioux lookout
    29.5608916504 -168.843322211
    Processing Record 410 of Set 1 | kapaa
    77.9277492912 33.4481751049
    Processing Record 411 of Set 1 | vardo
    -49.4048052143 165.313954774
    Processing Record 412 of Set 1 | tuatapere
    -78.9313141671 58.6735822216
    Processing Record 413 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    87.724537018 -170.450170428
    Processing Record 413 of Set 1 | mys shmidta
    Oops, that city doesn't exist in OpenWeatherMap.
    70.762217049 127.508115584
    Processing Record 413 of Set 1 | tiksi
    -0.137254577354 110.993457043
    Processing Record 414 of Set 1 | sri aman
    Oops, that city doesn't exist in OpenWeatherMap.
    -28.881569837 -0.73972694445
    Processing Record 414 of Set 1 | jamestown
    9.05454220493 50.4564977158
    Processing Record 415 of Set 1 | bandarbeyla
    -8.56587451302 -9.65882353309
    Processing Record 416 of Set 1 | georgetown
    67.0226072958 -163.751585076
    Processing Record 417 of Set 1 | nome
    78.152039734 179.730750976
    Processing Record 418 of Set 1 | leningradskiy
    23.7731133277 91.2502935713
    Processing Record 419 of Set 1 | agartala
    68.3641522062 30.9276887827
    Processing Record 420 of Set 1 | verkhnetulomskiy
    80.3880074615 6.28948406211
    Processing Record 421 of Set 1 | barentsburg
    Oops, that city doesn't exist in OpenWeatherMap.
    77.0673018629 -101.021769532
    Processing Record 421 of Set 1 | yellowknife
    -8.06948515242 -156.786894631
    Processing Record 422 of Set 1 | faanui
    75.8534003748 -162.891866058
    Processing Record 423 of Set 1 | barrow
    24.9533872331 -145.9981634
    Processing Record 424 of Set 1 | hilo
    49.0290404687 -108.657863911
    Processing Record 425 of Set 1 | shaunavon
    82.1428213574 14.5485846726
    Processing Record 426 of Set 1 | longyearbyen
    11.8316492763 -13.1820797971
    Processing Record 427 of Set 1 | gaoual
    -57.8995732403 132.583838772
    Processing Record 428 of Set 1 | new norfolk
    51.2375572596 48.4123617288
    Processing Record 429 of Set 1 | yershov
    -20.1250829133 15.017517513
    Processing Record 430 of Set 1 | khorixas
    4.92000767024 -90.29662202
    Processing Record 431 of Set 1 | puerto ayora
    38.6407914154 -29.0214238858
    Processing Record 432 of Set 1 | ribeira grande
    73.8433014755 -68.8827377526
    Processing Record 433 of Set 1 | clyde river
    38.6662079582 -2.19282719935
    Processing Record 434 of Set 1 | albacete
    -22.1899171961 -44.0357606508
    Processing Record 435 of Set 1 | quatis
    -14.4483876274 112.782265215
    Processing Record 436 of Set 1 | ambulu
    Oops, that city doesn't exist in OpenWeatherMap.
    15.8677388822 117.182957273
    Processing Record 436 of Set 1 | aloleng
    -6.7144996024 -119.616400311
    Processing Record 437 of Set 1 | atuona
    9.10239132429 -175.601970388
    Processing Record 438 of Set 1 | kapaa
    -22.0402608499 88.5702478657
    Processing Record 439 of Set 1 | bengkulu
    -64.7006843031 -33.9818078135
    Processing Record 440 of Set 1 | mar del plata
    -84.9094073446 105.005143023
    Processing Record 441 of Set 1 | albany
    16.0472078995 5.36311379966
    Processing Record 442 of Set 1 | abalak
    24.1156497778 146.102239489
    Processing Record 443 of Set 1 | katsuura
    -15.7031043786 -166.958285258
    Processing Record 444 of Set 1 | alofi
    -23.5393374832 160.381567672
    Processing Record 445 of Set 1 | koumac
    -46.2491931607 -52.3637352054
    Processing Record 446 of Set 1 | mar del plata
    41.8604128978 147.170999469
    Processing Record 447 of Set 1 | nemuro
    -3.62312388057 -70.135920196
    Processing Record 448 of Set 1 | puerto narino
    -43.4743956866 59.4190750305
    Processing Record 449 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    53.6602210337 4.58420706687
    Processing Record 449 of Set 1 | den helder
    -19.4933470629 20.1519211232
    Processing Record 450 of Set 1 | rundu
    17.3946820903 17.3508144226
    Processing Record 451 of Set 1 | faya
    Oops, that city doesn't exist in OpenWeatherMap.
    12.8241261294 -176.274698201
    Processing Record 451 of Set 1 | kapaa
    -78.5666728719 -110.04548385
    Processing Record 452 of Set 1 | punta arenas
    -89.5883246597 -94.4328243037
    Processing Record 453 of Set 1 | punta arenas
    27.5092550835 98.9767162754
    Processing Record 454 of Set 1 | dali
    -1.294419617 -111.185467888
    Processing Record 455 of Set 1 | puerto ayora
    -58.6614285661 79.4281766395
    Processing Record 456 of Set 1 | busselton
    -64.7653575106 167.049015151
    Processing Record 457 of Set 1 | bluff
    6.37774314784 -44.6083198705
    Processing Record 458 of Set 1 | salinopolis
    -87.3064337608 99.4382239729
    Processing Record 459 of Set 1 | albany
    16.9077850349 52.248159411
    Processing Record 460 of Set 1 | salalah
    -56.1841824019 -27.0725666745
    Processing Record 461 of Set 1 | chuy
    29.6452485639 -120.407425184
    Processing Record 462 of Set 1 | rosarito
    38.0421555484 -175.686014368
    Processing Record 463 of Set 1 | kapaa
    39.7266560308 120.020018303
    Processing Record 464 of Set 1 | qinhuangdao
    -31.1831624592 142.11351336
    Processing Record 465 of Set 1 | broken hill
    -72.0033362741 24.2569469066
    Processing Record 466 of Set 1 | bredasdorp
    -58.3492533854 -110.189750975
    Processing Record 467 of Set 1 | punta arenas
    49.508582449 -164.260884604
    Processing Record 468 of Set 1 | bethel
    54.6953156942 58.8637972366
    Processing Record 469 of Set 1 | bakal
    -29.8829229902 -163.314580145
    Processing Record 470 of Set 1 | avarua
    15.1839509246 56.1204833353
    Processing Record 471 of Set 1 | salalah
    -38.896714318 50.4492618194
    Processing Record 472 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    87.1149342755 9.02559174424
    Processing Record 472 of Set 1 | barentsburg
    Oops, that city doesn't exist in OpenWeatherMap.
    88.758527077 39.6619158615
    Processing Record 472 of Set 1 | tumannyy
    Oops, that city doesn't exist in OpenWeatherMap.
    70.3792454093 -38.6224574728
    Processing Record 472 of Set 1 | tasiilaq
    7.89589802612 -138.412247617
    Processing Record 473 of Set 1 | atuona
    -65.5630506112 148.810298248
    Processing Record 474 of Set 1 | hobart
    70.2637944846 117.361227694
    Processing Record 475 of Set 1 | saskylakh
    -79.6581555745 128.474474583
    Processing Record 476 of Set 1 | new norfolk
    50.6318587387 -11.1418902434
    Processing Record 477 of Set 1 | dingle
    -70.0904185682 57.5852282922
    Processing Record 478 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    33.8860813685 -105.297342708
    Processing Record 478 of Set 1 | ruidoso
    -45.5485895941 126.936756602
    Processing Record 479 of Set 1 | esperance
    -38.5452087668 -47.7957789957
    Processing Record 480 of Set 1 | chuy
    60.1111352186 54.3075358933
    Processing Record 481 of Set 1 | gayny
    26.7878037889 56.3543854699
    Processing Record 482 of Set 1 | qeshm
    55.938975963 164.302547561
    Processing Record 483 of Set 1 | ust-kamchatsk
    Oops, that city doesn't exist in OpenWeatherMap.
    -76.8556470969 45.3905926852
    Processing Record 483 of Set 1 | port alfred
    82.9987901228 -71.4016245936
    Processing Record 484 of Set 1 | qaanaaq
    15.29749982 54.5912149559
    Processing Record 485 of Set 1 | salalah
    -69.2367604011 170.641015009
    Processing Record 486 of Set 1 | bluff
    5.02990086144 31.996557515
    Processing Record 487 of Set 1 | juba
    Oops, that city doesn't exist in OpenWeatherMap.
    74.8229575507 -31.2335581705
    Processing Record 487 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    53.765401066 -88.9370464591
    Processing Record 487 of Set 1 | geraldton
    14.6701429687 152.897702535
    Processing Record 488 of Set 1 | kavieng
    -70.1369167338 45.909693323
    Processing Record 489 of Set 1 | port alfred
    -1.07620262328 169.422782677
    Processing Record 490 of Set 1 | tabiauea
    Oops, that city doesn't exist in OpenWeatherMap.
    -70.5668733807 15.7323726894
    Processing Record 490 of Set 1 | bredasdorp
    -46.019979593 86.1727573031
    Processing Record 491 of Set 1 | busselton
    -20.3047098569 -136.591360943
    Processing Record 492 of Set 1 | rikitea
    -1.92886610405 -21.0776929373
    Processing Record 493 of Set 1 | georgetown
    -82.3589932459 -54.5425953139
    Processing Record 494 of Set 1 | ushuaia
    25.651110359 30.3662601199
    Processing Record 495 of Set 1 | tahta
    19.952710735 163.999958913
    Processing Record 496 of Set 1 | butaritari
    -65.0468074036 -61.9989305446
    Processing Record 497 of Set 1 | ushuaia
    -60.5448192606 -170.269143619
    Processing Record 498 of Set 1 | vaini
    9.39777185006 26.5789831544
    Processing Record 499 of Set 1 | uwayl
    Oops, that city doesn't exist in OpenWeatherMap.
    -79.7383254882 45.6937897064
    Processing Record 499 of Set 1 | port alfred
    62.9686375907 -104.568117073
    Processing Record 500 of Set 1 | la ronge
    42.8271859742 -47.9529258413
    Processing Record 501 of Set 1 | torbay
    -64.5352033123 124.307348793
    Processing Record 502 of Set 1 | albany
    -22.4465752036 111.066889523
    Processing Record 503 of Set 1 | carnarvon
    -87.6762078009 124.519825545
    Processing Record 504 of Set 1 | new norfolk
    -60.1401761803 -152.394359391
    Processing Record 505 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -72.3549944432 -159.235597123
    Processing Record 505 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    54.8537145591 106.601374564
    Processing Record 505 of Set 1 | kachug
    -60.9247543365 0.462790034509
    Processing Record 506 of Set 1 | cape town
    25.8621988556 169.449135528
    Processing Record 507 of Set 1 | butaritari
    -85.8203538962 159.079561942
    Processing Record 508 of Set 1 | bluff
    -34.5522965759 13.2281498533
    Processing Record 509 of Set 1 | saldanha
    -53.1672840777 54.5882922421
    Processing Record 510 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -78.6328144682 -69.6006012771
    Processing Record 510 of Set 1 | ushuaia
    15.5565961879 32.7398618411
    Processing Record 511 of Set 1 | khartoum
    77.16455485 -174.45319386
    Processing Record 512 of Set 1 | mys shmidta
    Oops, that city doesn't exist in OpenWeatherMap.
    -12.966434286 120.566536608
    Processing Record 512 of Set 1 | waingapu
    71.7456728573 172.200214076
    Processing Record 513 of Set 1 | komsomolskiy
    5.97255939337 140.884428315
    Processing Record 514 of Set 1 | airai
    Oops, that city doesn't exist in OpenWeatherMap.
    73.8860977331 -9.49234505686
    Processing Record 514 of Set 1 | husavik
    -30.0981893188 -179.484113236
    Processing Record 515 of Set 1 | vaini
    -41.441834634 -64.6162586631
    Processing Record 516 of Set 1 | puerto madryn
    -83.8909375974 48.7352657494
    Processing Record 517 of Set 1 | port alfred
    -25.1914864344 170.779231168
    Processing Record 518 of Set 1 | vao
    -23.6517946291 -47.9397054546
    Processing Record 519 of Set 1 | itapetininga
    -57.5750470271 -6.82071659011
    Processing Record 520 of Set 1 | cape town
    -83.8735788031 -163.673343412
    Processing Record 521 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -12.3290998293 -85.8066065628
    Processing Record 521 of Set 1 | chicama
    -55.1647201419 -128.749193044
    Processing Record 522 of Set 1 | rikitea
    11.7264907984 -96.5488380328
    Processing Record 523 of Set 1 | pochutla
    Oops, that city doesn't exist in OpenWeatherMap.
    -5.13366653128 149.893391563
    Processing Record 523 of Set 1 | kimbe
    -42.3334425097 42.088332611
    Processing Record 524 of Set 1 | margate
    -49.6216076458 -133.178460148
    Processing Record 525 of Set 1 | rikitea
    81.3592353165 172.520473583
    Processing Record 526 of Set 1 | pevek
    10.1442411843 -145.221212847
    Processing Record 527 of Set 1 | hilo
    -54.6202249678 -50.497439432
    Processing Record 528 of Set 1 | ushuaia
    -69.5188259194 -159.236028955
    Processing Record 529 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    62.2690341504 153.55920174
    Processing Record 529 of Set 1 | seymchan
    66.6147501779 -24.9450870865
    Processing Record 530 of Set 1 | bolungarvik
    Oops, that city doesn't exist in OpenWeatherMap.
    -20.6605870278 140.020470726
    Processing Record 530 of Set 1 | mount isa
    41.1193167854 -37.6768762996
    Processing Record 531 of Set 1 | ribeira grande
    78.2887004611 121.664613753
    Processing Record 532 of Set 1 | tiksi
    -87.9747764689 144.006364767
    Processing Record 533 of Set 1 | hobart
    -22.762109097 71.3457446016
    Processing Record 534 of Set 1 | grand river south east
    Oops, that city doesn't exist in OpenWeatherMap.
    -79.3760554911 1.61260472213
    Processing Record 534 of Set 1 | hermanus
    64.9884347228 102.772879928
    Processing Record 535 of Set 1 | tura
    13.1037644853 138.577049049
    Processing Record 536 of Set 1 | airai
    Oops, that city doesn't exist in OpenWeatherMap.
    -54.6250621918 -26.1256970079
    Processing Record 536 of Set 1 | cidreira
    80.5769784668 -58.6925369022
    Processing Record 537 of Set 1 | upernavik
    -59.8569431751 47.2492089172
    Processing Record 538 of Set 1 | east london
    -30.7316318675 -41.6898498719
    Processing Record 539 of Set 1 | imbituba
    26.725185049 3.61090463659
    Processing Record 540 of Set 1 | adrar
    -89.9449636755 163.306273618
    Processing Record 541 of Set 1 | bluff
    18.3399827369 83.342887685
    Processing Record 542 of Set 1 | chipurupalle
    -32.1313407222 -56.7547459541
    Processing Record 543 of Set 1 | paso de los toros
    75.0486128377 -48.8725806079
    Processing Record 544 of Set 1 | ilulissat
    -46.1277029744 -129.415080076
    Processing Record 545 of Set 1 | rikitea
    -18.9326098274 169.373561952
    Processing Record 546 of Set 1 | isangel
    -86.7015767394 20.989295787
    Processing Record 547 of Set 1 | bredasdorp
    -85.1937544422 125.466497996
    Processing Record 548 of Set 1 | new norfolk
    -17.2876616089 -163.718772834
    Processing Record 549 of Set 1 | avarua
    -65.9307661299 177.836819681
    Processing Record 550 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    -74.5122656843 22.5160000272
    Processing Record 550 of Set 1 | bredasdorp
    -5.77215923469 -172.340354245
    Processing Record 551 of Set 1 | saleaula
    Oops, that city doesn't exist in OpenWeatherMap.
    -66.8099311305 24.0534868611
    Processing Record 551 of Set 1 | bredasdorp
    -52.7852556272 22.9952364329
    Processing Record 552 of Set 1 | bredasdorp
    -88.0429060627 133.579656797
    Processing Record 553 of Set 1 | hobart
    -79.6444730208 -167.167741076
    Processing Record 554 of Set 1 | avarua
    -47.996178327 -47.5697620039
    Processing Record 555 of Set 1 | mar del plata
    80.7855206794 85.7636742485
    Processing Record 556 of Set 1 | dikson
    14.4666761088 -68.7025140947
    Processing Record 557 of Set 1 | westpunt
    Oops, that city doesn't exist in OpenWeatherMap.
    -60.6677419056 162.888687758
    Processing Record 557 of Set 1 | bluff
    26.3652012115 49.8293637881
    Processing Record 558 of Set 1 | sayhat
    -44.6664581581 72.6231270528
    Processing Record 559 of Set 1 | mahebourg
    -51.4687112812 79.1486444573
    Processing Record 560 of Set 1 | mahebourg
    79.467349038 64.1822958072
    Processing Record 561 of Set 1 | amderma
    Oops, that city doesn't exist in OpenWeatherMap.
    88.0132049412 -29.8494684039
    Processing Record 561 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    76.9305587487 21.1346637639
    Processing Record 561 of Set 1 | longyearbyen
    23.1778966398 110.783179059
    Processing Record 562 of Set 1 | jinji
    58.5267530278 -125.31310985
    Processing Record 563 of Set 1 | fort nelson
    -72.7405489077 -12.2384666286
    Processing Record 564 of Set 1 | cape town
    14.9122849158 12.1588687261
    Processing Record 565 of Set 1 | diffa
    -63.8201976594 -13.2569428208
    Processing Record 566 of Set 1 | cape town
    -42.3347088183 -36.0438433674
    Processing Record 567 of Set 1 | cidreira
    -27.5947929168 -173.173067702
    Processing Record 568 of Set 1 | vaini
    -83.9462609441 52.6771626646
    Processing Record 569 of Set 1 | port alfred
    53.5176736706 -15.1174745806
    Processing Record 570 of Set 1 | dingle
    -27.7785279853 52.5091293969
    Processing Record 571 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -59.7832666703 14.5730605045
    Processing Record 571 of Set 1 | hermanus
    87.351824641 -32.7689921611
    Processing Record 572 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    -25.3844631744 -3.20231045232
    Processing Record 572 of Set 1 | jamestown
    -19.799460973 -141.194466126
    Processing Record 573 of Set 1 | rikitea
    -17.5468446027 147.830210343
    Processing Record 574 of Set 1 | innisfail
    -57.2976049807 -76.7874982591
    Processing Record 575 of Set 1 | punta arenas
    83.0341729836 143.580478165
    Processing Record 576 of Set 1 | chokurdakh
    74.3243015585 158.107772456
    Processing Record 577 of Set 1 | cherskiy
    21.8063438083 -100.218725543
    Processing Record 578 of Set 1 | fernandez
    Oops, that city doesn't exist in OpenWeatherMap.
    13.7140972025 -155.854094777
    Processing Record 578 of Set 1 | hilo
    58.5896900545 -86.08556067
    Processing Record 579 of Set 1 | attawapiskat
    Oops, that city doesn't exist in OpenWeatherMap.
    -15.8503164127 -21.1684013676
    Processing Record 579 of Set 1 | georgetown
    15.6127489407 -46.3643869408
    Processing Record 580 of Set 1 | sinnamary
    61.9545006492 -130.731316624
    Processing Record 581 of Set 1 | whitehorse
    -18.7546047862 -92.7832590935
    Processing Record 582 of Set 1 | hualmay
    73.1968770296 -153.5844133
    Processing Record 583 of Set 1 | barrow
    24.7751944569 -39.6452714032
    Processing Record 584 of Set 1 | ponta do sol
    85.5599002097 -54.6276951111
    Processing Record 585 of Set 1 | upernavik
    -71.4802024916 71.7503618038
    Processing Record 586 of Set 1 | saint-philippe
    65.2974853136 -55.9608208267
    Processing Record 587 of Set 1 | sisimiut
    -57.2092932182 100.973454286
    Processing Record 588 of Set 1 | busselton
    31.0253345164 93.904722651
    Processing Record 589 of Set 1 | along
    79.8182650472 -23.0806074971
    Processing Record 590 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    49.8855399696 -108.747220995
    Processing Record 590 of Set 1 | gull lake
    -38.9706713026 151.494519828
    Processing Record 591 of Set 1 | batemans bay
    68.1734308842 165.375431557
    Processing Record 592 of Set 1 | bilibino
    -6.07259302308 160.404007435
    Processing Record 593 of Set 1 | buala
    -18.9407239458 -124.554766017
    Processing Record 594 of Set 1 | rikitea
    54.1447653768 -74.562036392
    Processing Record 595 of Set 1 | chapais
    32.3802290996 95.0680043726
    Processing Record 596 of Set 1 | along
    55.3740476177 172.399700474
    Processing Record 597 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    80.7211788864 105.306713113
    Processing Record 597 of Set 1 | khatanga
    28.8413256571 -5.98654491583
    Processing Record 598 of Set 1 | tarudant
    Oops, that city doesn't exist in OpenWeatherMap.
    -33.2119088782 160.072814562
    Processing Record 598 of Set 1 | port macquarie
    -42.307627946 114.823906834
    Processing Record 599 of Set 1 | albany
    61.8269910231 -39.0212091292
    Processing Record 600 of Set 1 | tasiilaq
    38.5754594139 -71.5486225523
    Processing Record 601 of Set 1 | mastic beach
    10.2177740639 39.1018074526
    Processing Record 602 of Set 1 | were ilu
    24.8211755747 -38.1287785258
    Processing Record 603 of Set 1 | ponta do sol
    47.2857580795 -56.3356833387
    Processing Record 604 of Set 1 | miquelon
    65.1157144159 -177.824243037
    Processing Record 605 of Set 1 | egvekinot
    38.1063712233 90.6220296368
    Processing Record 606 of Set 1 | hami
    78.4177210143 -178.199476682
    Processing Record 607 of Set 1 | mys shmidta
    Oops, that city doesn't exist in OpenWeatherMap.
    -85.8847008558 13.1900322451
    Processing Record 607 of Set 1 | bredasdorp
    40.8791480164 -27.3974451666
    Processing Record 608 of Set 1 | lagoa
    3.58665802651 32.2599846413
    Processing Record 609 of Set 1 | adjumani
    61.9714336369 -167.903339692
    Processing Record 610 of Set 1 | nome
    -59.9337694255 -56.7144503296
    Processing Record 611 of Set 1 | ushuaia
    -58.062287195 -138.75304479
    Processing Record 612 of Set 1 | rikitea
    -26.4873746554 105.525392303
    Processing Record 613 of Set 1 | carnarvon
    -78.457237862 -7.30965650386
    Processing Record 614 of Set 1 | hermanus
    46.0444737127 -126.63495225
    Processing Record 615 of Set 1 | astoria
    -17.3181870758 -157.832330721
    Processing Record 616 of Set 1 | avarua
    10.2159756484 81.5765987897
    Processing Record 617 of Set 1 | mullaitivu
    Oops, that city doesn't exist in OpenWeatherMap.
    59.4781973961 27.5976987838
    Processing Record 617 of Set 1 | voka
    87.9400787889 111.848213351
    Processing Record 618 of Set 1 | saskylakh
    -33.3345377772 0.966772741034
    Processing Record 619 of Set 1 | luderitz
    42.7322779736 83.831848581
    Processing Record 620 of Set 1 | kuche
    Oops, that city doesn't exist in OpenWeatherMap.
    43.014986214 -39.8550595876
    Processing Record 620 of Set 1 | ribeira grande
    58.8665213967 119.811580674
    Processing Record 621 of Set 1 | khani
    Oops, that city doesn't exist in OpenWeatherMap.
    -63.5137438353 23.8603710168
    Processing Record 621 of Set 1 | bredasdorp
    64.3621293162 55.6143814042
    Processing Record 622 of Set 1 | nizhniy odes
    -78.9002124371 106.420494677
    Processing Record 623 of Set 1 | albany
    -48.965284078 15.9200866716
    Processing Record 624 of Set 1 | hermanus
    -39.3226172498 -73.0380426457
    Processing Record 625 of Set 1 | loncoche
    -30.646537883 112.703214835
    Processing Record 626 of Set 1 | geraldton
    5.77627703274 110.141409554
    Processing Record 627 of Set 1 | sibu
    -71.5627624348 -45.273942265
    Processing Record 628 of Set 1 | ushuaia
    -11.3191928702 -125.985011965
    Processing Record 629 of Set 1 | atuona
    -22.2453361614 131.095284213
    Processing Record 630 of Set 1 | yulara
    -48.7432763533 -57.082950516
    Processing Record 631 of Set 1 | rawson
    -49.4283756479 -88.9048463252
    Processing Record 632 of Set 1 | castro
    83.2196978961 51.4576864891
    Processing Record 633 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    6.78690641907 112.538981334
    Processing Record 633 of Set 1 | miri
    -88.7871003526 163.737464881
    Processing Record 634 of Set 1 | bluff
    -76.8728547974 -52.5136624138
    Processing Record 635 of Set 1 | ushuaia
    -28.4787090115 118.779336334
    Processing Record 636 of Set 1 | northam
    70.7230579676 115.446500944
    Processing Record 637 of Set 1 | saskylakh
    62.6615587597 94.3362596303
    Processing Record 638 of Set 1 | baykit
    -89.6231051854 -3.26852487886
    Processing Record 639 of Set 1 | hermanus
    -73.8325830468 -77.6491439449
    Processing Record 640 of Set 1 | ushuaia
    -0.338720623935 -116.973221478
    Processing Record 641 of Set 1 | san patricio
    -20.1940445925 58.3796109591
    Processing Record 642 of Set 1 | grand river south east
    Oops, that city doesn't exist in OpenWeatherMap.
    -57.5733494259 39.0953937388
    Processing Record 642 of Set 1 | port alfred
    -3.96344320266 102.112762671
    Processing Record 643 of Set 1 | bengkulu
    -21.1757816909 59.2329311059
    Processing Record 644 of Set 1 | bambous virieux
    -78.8744229218 45.7374196641
    Processing Record 645 of Set 1 | port alfred
    14.7985534129 -122.923591006
    Processing Record 646 of Set 1 | constitucion
    Oops, that city doesn't exist in OpenWeatherMap.
    -79.4860724932 4.15921146161
    Processing Record 646 of Set 1 | hermanus
    -11.5016961032 64.0200157091
    Processing Record 647 of Set 1 | grand gaube
    60.2776964027 156.910420027
    Processing Record 648 of Set 1 | omsukchan
    -72.0726338203 148.271456623
    Processing Record 649 of Set 1 | hobart
    -34.7968652955 135.85887122
    Processing Record 650 of Set 1 | port lincoln
    -24.3931275061 74.4958457003
    Processing Record 651 of Set 1 | grand river south east
    Oops, that city doesn't exist in OpenWeatherMap.
    -59.0487991438 146.433960153
    Processing Record 651 of Set 1 | hobart
    80.0666046419 -65.6984651676
    Processing Record 652 of Set 1 | narsaq
    -59.833457902 37.3323341848
    Processing Record 653 of Set 1 | port alfred
    -2.67748898352 58.2367161091
    Processing Record 654 of Set 1 | victoria
    -18.3607388684 112.299666969
    Processing Record 655 of Set 1 | karratha
    -44.2674575371 108.035537243
    Processing Record 656 of Set 1 | busselton
    -0.712995395848 -40.33412275
    Processing Record 657 of Set 1 | acarau
    32.8824727587 -51.2611777221
    Processing Record 658 of Set 1 | saint george
    -79.1577002704 38.5716571202
    Processing Record 659 of Set 1 | port alfred
    -59.212310733 36.9325848538
    Processing Record 660 of Set 1 | port alfred
    13.2991229736 -159.082880668
    Processing Record 661 of Set 1 | hilo
    52.4445943124 -112.308517138
    Processing Record 662 of Set 1 | stettler
    -30.1372892768 -10.7754825311
    Processing Record 663 of Set 1 | jamestown
    16.2249941301 169.901590422
    Processing Record 664 of Set 1 | butaritari
    -89.3531394536 158.232269035
    Processing Record 665 of Set 1 | bluff
    14.6000714225 106.29884491
    Processing Record 666 of Set 1 | champasak
    -29.7977133813 137.217983866
    Processing Record 667 of Set 1 | port augusta
    44.5033592522 -112.673810848
    Processing Record 668 of Set 1 | dillon
    -88.2815566232 15.9433300107
    Processing Record 669 of Set 1 | bredasdorp
    61.3998629802 179.325504555
    Processing Record 670 of Set 1 | beringovskiy
    33.6335452601 34.2688441325
    Processing Record 671 of Set 1 | sur
    Oops, that city doesn't exist in OpenWeatherMap.
    -71.8695842753 20.0679595067
    Processing Record 671 of Set 1 | bredasdorp
    -58.6339082358 -101.036401142
    Processing Record 672 of Set 1 | punta arenas
    9.51784570916 -107.620942787
    Processing Record 673 of Set 1 | coahuayana
    -44.1066669345 41.6655016207
    Processing Record 674 of Set 1 | umzimvubu
    Oops, that city doesn't exist in OpenWeatherMap.
    7.08939389795 -47.7569911353
    Processing Record 674 of Set 1 | cayenne
    -68.8953468261 71.4975771702
    Processing Record 675 of Set 1 | saint-philippe
    84.2341130126 151.117392469
    Processing Record 676 of Set 1 | chokurdakh
    -30.4655070704 162.066929728
    Processing Record 677 of Set 1 | byron bay
    37.5619827429 36.4392853161
    Processing Record 678 of Set 1 | duzici
    Oops, that city doesn't exist in OpenWeatherMap.
    -81.6177921075 30.3379101143
    Processing Record 678 of Set 1 | port elizabeth
    49.0159236542 91.1794403391
    Processing Record 679 of Set 1 | ulaangom
    -24.2786156063 -82.2182924379
    Processing Record 680 of Set 1 | marcona
    Oops, that city doesn't exist in OpenWeatherMap.
    -68.4590212874 23.8132332807
    Processing Record 680 of Set 1 | bredasdorp
    -53.4323635934 -9.92001611808
    Processing Record 681 of Set 1 | cape town
    -26.2822969567 55.757712994
    Processing Record 682 of Set 1 | saint-joseph
    63.4008605697 -146.648076173
    Processing Record 683 of Set 1 | fairbanks
    23.1779385232 94.4333889893
    Processing Record 684 of Set 1 | mawlaik
    41.0706747411 89.7238141616
    Processing Record 685 of Set 1 | urumqi
    Oops, that city doesn't exist in OpenWeatherMap.
    40.4193972344 -3.51471238144
    Processing Record 685 of Set 1 | san fernando de henares
    32.4790185893 -103.288747382
    Processing Record 686 of Set 1 | hobbs
    12.3273293703 42.4565342072
    Processing Record 687 of Set 1 | jibuti
    Oops, that city doesn't exist in OpenWeatherMap.
    56.6679258432 -169.778113659
    Processing Record 687 of Set 1 | provideniya
    -38.7895782218 -113.336368056
    Processing Record 688 of Set 1 | rikitea
    -89.6945571695 -141.306122247
    Processing Record 689 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    22.821037594 53.4454207899
    Processing Record 689 of Set 1 | abu dhabi
    23.2868951756 107.601126337
    Processing Record 690 of Set 1 | nanning
    53.3681109604 -152.830146324
    Processing Record 691 of Set 1 | kodiak
    40.9305392967 -171.285449904
    Processing Record 692 of Set 1 | bethel
    45.1500104955 -125.300489433
    Processing Record 693 of Set 1 | newport
    65.2032704742 138.310599575
    Processing Record 694 of Set 1 | khandyga
    50.506236504 65.7275562293
    Processing Record 695 of Set 1 | derzhavinsk
    67.2554487977 154.610839428
    Processing Record 696 of Set 1 | srednekolymsk
    -19.8845923161 -43.7715758235
    Processing Record 697 of Set 1 | sabara
    Oops, that city doesn't exist in OpenWeatherMap.
    -68.7856934542 175.377556752
    Processing Record 697 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    17.6737787098 -20.1280368591
    Processing Record 697 of Set 1 | santa maria
    -25.8759617315 59.6763910108
    Processing Record 698 of Set 1 | souillac
    66.3306241419 171.14471593
    Processing Record 699 of Set 1 | komsomolskiy
    -45.7140933053 146.823115031
    Processing Record 700 of Set 1 | hobart
    44.0659192948 157.091389442
    Processing Record 701 of Set 1 | severo-kurilsk
    -25.6844802591 -11.9300780327
    Processing Record 702 of Set 1 | jamestown
    -36.1380894768 -87.2987944188
    Processing Record 703 of Set 1 | lebu
    72.859930507 -68.6911130279
    Processing Record 704 of Set 1 | clyde river
    26.359306417 107.22658375
    Processing Record 705 of Set 1 | duyun
    -14.9552883776 151.945566173
    Processing Record 706 of Set 1 | samarai
    55.8421136002 2.51229630772
    Processing Record 707 of Set 1 | bridlington
    -22.774101716 -4.7515531443
    Processing Record 708 of Set 1 | jamestown
    -21.4498787191 -151.693840736
    Processing Record 709 of Set 1 | moerai
    -55.9214600673 -138.770851861
    Processing Record 710 of Set 1 | rikitea
    -88.0401143497 -168.273599789
    Processing Record 711 of Set 1 | vaini
    82.1372186677 -120.621750246
    Processing Record 712 of Set 1 | tuktoyaktuk
    -70.1877949536 120.265310556
    Processing Record 713 of Set 1 | albany
    -30.678644965 -176.281711794
    Processing Record 714 of Set 1 | vaini
    44.216639151 -56.124324604
    Processing Record 715 of Set 1 | saint-pierre
    31.4889983446 -141.882655329
    Processing Record 716 of Set 1 | hilo
    29.5332449156 8.38414328182
    Processing Record 717 of Set 1 | nalut
    -4.74739275266 -6.24530458082
    Processing Record 718 of Set 1 | georgetown
    41.6027022843 -118.154558408
    Processing Record 719 of Set 1 | winnemucca
    57.6218680688 59.7147436704
    Processing Record 720 of Set 1 | uralets
    11.2614825876 -66.2457451312
    Processing Record 721 of Set 1 | guatire
    -85.9287507177 -178.354265896
    Processing Record 722 of Set 1 | vaini
    27.444248174 -176.382291412
    Processing Record 723 of Set 1 | kapaa
    71.9778569431 154.75875245
    Processing Record 724 of Set 1 | srednekolymsk
    24.5129313458 -45.6090922398
    Processing Record 725 of Set 1 | codrington
    Oops, that city doesn't exist in OpenWeatherMap.
    -42.1659624101 -121.475920299
    Processing Record 725 of Set 1 | rikitea
    44.2464108199 83.9923963216
    Processing Record 726 of Set 1 | kuytun
    Oops, that city doesn't exist in OpenWeatherMap.
    -10.81351248 -113.586337707
    Processing Record 726 of Set 1 | rikitea
    35.7618573089 56.1269238328
    Processing Record 727 of Set 1 | shahrud
    57.4261079298 -147.639110526
    Processing Record 728 of Set 1 | sterling
    -51.2015188148 12.9582909568
    Processing Record 729 of Set 1 | hermanus
    47.6213527505 91.5913839946
    Processing Record 730 of Set 1 | ulaangom
    -6.01819295111 172.39760831
    Processing Record 731 of Set 1 | lolua
    Oops, that city doesn't exist in OpenWeatherMap.
    -23.3557417619 -45.0914995757
    Processing Record 731 of Set 1 | ubatuba
    52.7071238181 34.0671524531
    Processing Record 732 of Set 1 | kokorevka
    67.4684679293 -149.388844822
    Processing Record 733 of Set 1 | college
    83.0103148279 -165.273864545
    Processing Record 734 of Set 1 | barrow
    14.8663426482 -98.1431051888
    Processing Record 735 of Set 1 | huazolotitlan
    Oops, that city doesn't exist in OpenWeatherMap.
    11.4984548737 -159.123617053
    Processing Record 735 of Set 1 | hilo
    -29.8131624836 -160.360545708
    Processing Record 736 of Set 1 | avarua
    -77.5343296053 -134.559284711
    Processing Record 737 of Set 1 | rikitea
    23.0151016268 9.931520428
    Processing Record 738 of Set 1 | gat
    Oops, that city doesn't exist in OpenWeatherMap.
    0.298876024786 175.646651107
    Processing Record 738 of Set 1 | temaraia
    Oops, that city doesn't exist in OpenWeatherMap.
    -18.6454729166 34.4499630723
    Processing Record 738 of Set 1 | dondo
    11.6615055013 -104.600945652
    Processing Record 739 of Set 1 | ixtapa
    35.109918929 -124.276145215
    Processing Record 740 of Set 1 | pacific grove
    80.8105195945 -15.7598059218
    Processing Record 741 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    -17.1942224105 176.230209936
    Processing Record 741 of Set 1 | isangel
    85.2690456691 58.5694995758
    Processing Record 742 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    58.7496910129 126.980396183
    Processing Record 742 of Set 1 | tommot
    74.5408405122 169.032481698
    Processing Record 743 of Set 1 | pevek
    -21.6645198959 -113.544718537
    Processing Record 744 of Set 1 | rikitea
    -15.6760758511 102.541134786
    Processing Record 745 of Set 1 | palabuhanratu
    Oops, that city doesn't exist in OpenWeatherMap.
    -82.5425759405 59.344858984
    Processing Record 745 of Set 1 | east london
    -38.4881889719 -96.7398646383
    Processing Record 746 of Set 1 | lebu
    -65.2812376599 -137.073127563
    Processing Record 747 of Set 1 | rikitea
    -38.0842702264 -172.308733206
    Processing Record 748 of Set 1 | vaini
    -75.9969125674 118.754457708
    Processing Record 749 of Set 1 | albany
    -40.7476642574 -135.636869885
    Processing Record 750 of Set 1 | rikitea
    70.7444357551 48.4011329843
    Processing Record 751 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    -21.5692651219 177.99159361
    Processing Record 751 of Set 1 | isangel
    79.6992342054 164.617601784
    Processing Record 752 of Set 1 | cherskiy
    60.6659790842 -120.776374289
    Processing Record 753 of Set 1 | fort nelson
    19.0346097449 98.5663370811
    Processing Record 754 of Set 1 | sansai
    Oops, that city doesn't exist in OpenWeatherMap.
    64.2690870678 161.584055963
    Processing Record 754 of Set 1 | evensk
    85.5787221836 -107.86764743
    Processing Record 755 of Set 1 | yellowknife
    -69.9254283641 -135.143072281
    Processing Record 756 of Set 1 | rikitea
    16.3528919811 51.6476789724
    Processing Record 757 of Set 1 | salalah
    -75.0543222783 83.8814781538
    Processing Record 758 of Set 1 | busselton
    51.8586924101 -176.902645264
    Processing Record 759 of Set 1 | provideniya
    72.5687001326 -94.5895738332
    Processing Record 760 of Set 1 | thompson
    -49.5603307939 -114.503283144
    Processing Record 761 of Set 1 | rikitea
    37.1765695702 68.4572902164
    Processing Record 762 of Set 1 | dusti
    -72.976606188 -143.819275695
    Processing Record 763 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -77.9513769741 147.037872677
    Processing Record 763 of Set 1 | hobart
    8.84383232648 -174.853011953
    Processing Record 764 of Set 1 | kapaa
    59.2258972054 167.781407529
    Processing Record 765 of Set 1 | tilichiki
    -59.0992533022 -144.490952056
    Processing Record 766 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    24.9503015368 49.0594858263
    Processing Record 766 of Set 1 | buqayq
    Oops, that city doesn't exist in OpenWeatherMap.
    -88.5796521556 99.0570894302
    Processing Record 766 of Set 1 | albany
    36.1538253681 -178.391230302
    Processing Record 767 of Set 1 | kapaa
    10.774418193 -109.367112964
    Processing Record 768 of Set 1 | san patricio
    67.0424793128 130.771422898
    Processing Record 769 of Set 1 | batagay-alyta
    15.836006748 86.8804248133
    Processing Record 770 of Set 1 | palasa
    -53.1360356689 -25.3319058414
    Processing Record 771 of Set 1 | cidreira
    85.2678487097 -62.7058232916
    Processing Record 772 of Set 1 | narsaq
    -6.45970356709 169.955923011
    Processing Record 773 of Set 1 | lata
    Oops, that city doesn't exist in OpenWeatherMap.
    80.0321117625 32.8449661328
    Processing Record 773 of Set 1 | vardo
    19.7122114535 153.847226763
    Processing Record 774 of Set 1 | katsuura
    -11.3327820562 126.988856222
    Processing Record 775 of Set 1 | atambua
    69.1523104153 -142.024150471
    Processing Record 776 of Set 1 | aklavik
    68.416558986 100.664271547
    Processing Record 777 of Set 1 | khatanga
    -65.3504606722 170.710170612
    Processing Record 778 of Set 1 | bluff
    23.2560555815 6.78883481967
    Processing Record 779 of Set 1 | gat
    Oops, that city doesn't exist in OpenWeatherMap.
    -59.6323383783 49.1225627204
    Processing Record 779 of Set 1 | east london
    45.8050860099 -163.211282264
    Processing Record 780 of Set 1 | bethel
    21.597451507 -4.96523622533
    Processing Record 781 of Set 1 | taoudenni
    -24.8698570097 -37.4054455808
    Processing Record 782 of Set 1 | sao joao da barra
    68.6128219888 -65.1658068141
    Processing Record 783 of Set 1 | pangnirtung
    -70.4626031514 -47.9246521463
    Processing Record 784 of Set 1 | ushuaia
    -63.0901485837 25.5820713222
    Processing Record 785 of Set 1 | bredasdorp
    86.5776383116 -30.6022178849
    Processing Record 786 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    -80.7630700091 18.1413968355
    Processing Record 786 of Set 1 | bredasdorp
    -66.830098182 -132.388102412
    Processing Record 787 of Set 1 | rikitea
    7.47346601652 -170.139977311
    Processing Record 788 of Set 1 | kapaa
    72.2278570403 68.6475309894
    Processing Record 789 of Set 1 | aksarka
    76.8681666417 -49.634194519
    Processing Record 790 of Set 1 | upernavik
    -29.5888372879 -27.4577278085
    Processing Record 791 of Set 1 | sao joao da barra
    36.5987280384 176.119100088
    Processing Record 792 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    -8.33726650349 73.7276089742
    Processing Record 792 of Set 1 | hithadhoo
    -48.5768550705 -174.105866328
    Processing Record 793 of Set 1 | vaini
    -30.3899061789 80.2929306512
    Processing Record 794 of Set 1 | bambous virieux
    45.8340871952 142.982407834
    Processing Record 795 of Set 1 | novikovo
    Oops, that city doesn't exist in OpenWeatherMap.
    1.34306635646 -121.337776705
    Processing Record 795 of Set 1 | atuona
    15.8042647873 -78.0833495053
    Processing Record 796 of Set 1 | bull savanna
    -31.6590665408 97.3239949748
    Processing Record 797 of Set 1 | geraldton
    54.7698014635 -20.6704142647
    Processing Record 798 of Set 1 | vestmannaeyjar
    -14.4557972018 116.445700012
    Processing Record 799 of Set 1 | praya
    -84.6535790348 154.991766419
    Processing Record 800 of Set 1 | bluff
    42.2502042121 175.923727918
    Processing Record 801 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    -54.9870378745 -131.699742175
    Processing Record 801 of Set 1 | rikitea
    -22.4101649163 151.658718502
    Processing Record 802 of Set 1 | yeppoon
    31.8537785176 149.345629474
    Processing Record 803 of Set 1 | hasaki
    44.1654779702 2.62765243859
    Processing Record 804 of Set 1 | rodez
    -73.8412521397 -168.388041526
    Processing Record 805 of Set 1 | vaini
    73.5152756001 -85.6362135635
    Processing Record 806 of Set 1 | qaanaaq
    9.95466371973 78.6991441654
    Processing Record 807 of Set 1 | karaikkudi
    70.6810367588 -3.57388186968
    Processing Record 808 of Set 1 | klaksvik
    18.4788591049 116.667230378
    Processing Record 809 of Set 1 | puro
    62.661976876 -82.5697790886
    Processing Record 810 of Set 1 | attawapiskat
    Oops, that city doesn't exist in OpenWeatherMap.
    -0.416269911817 77.4541255785
    Processing Record 810 of Set 1 | viligili
    Oops, that city doesn't exist in OpenWeatherMap.
    35.6418823318 -173.728615739
    Processing Record 810 of Set 1 | kapaa
    26.4019662099 -55.0052009462
    Processing Record 811 of Set 1 | codrington
    Oops, that city doesn't exist in OpenWeatherMap.
    -16.2494117669 170.083001254
    Processing Record 811 of Set 1 | vila
    Oops, that city doesn't exist in OpenWeatherMap.
    -14.8928615259 -19.2549799016
    Processing Record 811 of Set 1 | georgetown
    -15.3597049676 -178.38500328
    Processing Record 812 of Set 1 | halalo
    Oops, that city doesn't exist in OpenWeatherMap.
    -33.5519422354 144.784008677
    Processing Record 812 of Set 1 | griffith
    54.0193041875 11.3141518125
    Processing Record 813 of Set 1 | wismar
    -74.0780787047 -8.07017297467
    Processing Record 814 of Set 1 | cape town
    73.7440706195 -28.4863782923
    Processing Record 815 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    -60.9694542506 139.003627684
    Processing Record 815 of Set 1 | new norfolk
    -77.7562523266 -52.9226986181
    Processing Record 816 of Set 1 | ushuaia
    66.4481365527 -21.7815183984
    Processing Record 817 of Set 1 | hvammstangi
    Oops, that city doesn't exist in OpenWeatherMap.
    83.6109495315 -50.143315995
    Processing Record 817 of Set 1 | upernavik
    -83.2223480323 -151.411044894
    Processing Record 818 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -30.7068736115 37.113043684
    Processing Record 818 of Set 1 | richards bay
    -85.5620861976 133.562859858
    Processing Record 819 of Set 1 | hobart
    -67.8307824767 -65.9193168284
    Processing Record 820 of Set 1 | ushuaia
    -52.2707772106 -62.21834034
    Processing Record 821 of Set 1 | ushuaia
    -22.3721786069 -59.4049185601
    Processing Record 822 of Set 1 | filadelfia
    -65.6437230529 111.070518268
    Processing Record 823 of Set 1 | albany
    -27.921945605 40.7818865807
    Processing Record 824 of Set 1 | beloha
    -75.9933368108 108.780582072
    Processing Record 825 of Set 1 | albany
    -36.0147885305 -95.3823058251
    Processing Record 826 of Set 1 | lebu
    -61.3154241783 -141.264545644
    Processing Record 827 of Set 1 | rikitea
    -33.9963418752 -31.1883513889
    Processing Record 828 of Set 1 | arraial do cabo
    35.0142913276 -138.778236767
    Processing Record 829 of Set 1 | fortuna
    -4.47697219534 -16.1802620479
    Processing Record 830 of Set 1 | georgetown
    11.9538580118 -174.034133996
    Processing Record 831 of Set 1 | kapaa
    -78.6618320185 162.972503134
    Processing Record 832 of Set 1 | bluff
    30.2105874811 95.6536866355
    Processing Record 833 of Set 1 | pasighat
    76.7779729051 60.5524375039
    Processing Record 834 of Set 1 | amderma
    Oops, that city doesn't exist in OpenWeatherMap.
    -7.32278105014 7.90732383615
    Processing Record 834 of Set 1 | soyo
    Oops, that city doesn't exist in OpenWeatherMap.
    -46.1596685736 38.8313302179
    Processing Record 834 of Set 1 | east london
    -44.4309231726 -137.443230062
    Processing Record 835 of Set 1 | rikitea
    -84.2326581086 87.0690559727
    Processing Record 836 of Set 1 | busselton
    -82.3954207599 87.3116933699
    Processing Record 837 of Set 1 | busselton
    15.48388243 105.951615662
    Processing Record 838 of Set 1 | pakxe
    -38.1574158739 6.54186159395
    Processing Record 839 of Set 1 | saldanha
    28.8821618379 -13.6520305805
    Processing Record 840 of Set 1 | tias
    24.9673527205 107.178505018
    Processing Record 841 of Set 1 | jinchengjiang
    Oops, that city doesn't exist in OpenWeatherMap.
    -44.3083988094 22.7132109317
    Processing Record 841 of Set 1 | bredasdorp
    -16.1966061104 135.351655834
    Processing Record 842 of Set 1 | ngukurr
    Oops, that city doesn't exist in OpenWeatherMap.
    -39.8402767637 -74.3689017266
    Processing Record 842 of Set 1 | valdivia
    33.7829357782 -9.28744691705
    Processing Record 843 of Set 1 | azimur
    Oops, that city doesn't exist in OpenWeatherMap.
    84.5755328445 94.1682118438
    Processing Record 843 of Set 1 | khatanga
    59.4445904008 -107.595987828
    Processing Record 844 of Set 1 | la ronge
    -3.84274475071 78.6368772242
    Processing Record 845 of Set 1 | hithadhoo
    42.8451099099 84.7137099478
    Processing Record 846 of Set 1 | kuytun
    Oops, that city doesn't exist in OpenWeatherMap.
    -64.3289575365 -86.0457672946
    Processing Record 846 of Set 1 | punta arenas
    85.2196484349 -10.533070187
    Processing Record 847 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    -72.2075166798 165.90528726
    Processing Record 847 of Set 1 | bluff
    -47.31923489 -147.608025554
    Processing Record 848 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    5.65854879807 149.652359753
    Processing Record 848 of Set 1 | lorengau
    -53.0783692996 11.5199460461
    Processing Record 849 of Set 1 | hermanus
    -14.8889866757 -144.80406457
    Processing Record 850 of Set 1 | tautira
    -56.837039121 -101.053984709
    Processing Record 851 of Set 1 | punta arenas
    -5.93567682381 -34.0938884004
    Processing Record 852 of Set 1 | nisia floresta
    -17.3388482519 3.29879199715
    Processing Record 853 of Set 1 | namibe
    -17.4523629849 168.946740598
    Processing Record 854 of Set 1 | vila
    Oops, that city doesn't exist in OpenWeatherMap.
    -12.282982876 -19.4290418879
    Processing Record 854 of Set 1 | georgetown
    27.688192479 -128.006590907
    Processing Record 855 of Set 1 | lompoc
    -64.007606855 -7.9325699376
    Processing Record 856 of Set 1 | cape town
    -36.7927141988 24.2709597269
    Processing Record 857 of Set 1 | kruisfontein
    81.8267099388 -150.801674655
    Processing Record 858 of Set 1 | barrow
    47.8603286169 92.7785169273
    Processing Record 859 of Set 1 | ulaangom
    67.3808242283 163.772729107
    Processing Record 860 of Set 1 | bilibino
    48.65698291 13.7623556858
    Processing Record 861 of Set 1 | hauzenberg
    41.1022657366 -133.591645977
    Processing Record 862 of Set 1 | eureka
    -25.9914082626 -107.455345161
    Processing Record 863 of Set 1 | rikitea
    37.6596366816 36.2037527534
    Processing Record 864 of Set 1 | kadirli
    32.9099351465 111.654492104
    Processing Record 865 of Set 1 | danjiangkou
    68.2758296376 -73.3302868255
    Processing Record 866 of Set 1 | clyde river
    83.7957708777 -29.0407991055
    Processing Record 867 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    88.9154530701 114.162859154
    Processing Record 867 of Set 1 | saskylakh
    86.2302492974 1.59662695949
    Processing Record 868 of Set 1 | barentsburg
    Oops, that city doesn't exist in OpenWeatherMap.
    -42.355687003 -157.134172979
    Processing Record 868 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    13.8045610354 122.708080644
    Processing Record 868 of Set 1 | liboro
    -65.8241035743 -57.4045158764
    Processing Record 869 of Set 1 | ushuaia
    1.25824502075 -5.72951542131
    Processing Record 870 of Set 1 | tabou
    -22.8856091836 127.957291538
    Processing Record 871 of Set 1 | yulara
    48.5824671258 -105.665790604
    Processing Record 872 of Set 1 | assiniboia
    -4.60118198637 -59.3663627815
    Processing Record 873 of Set 1 | borba
    45.4870077708 -132.421904211
    Processing Record 874 of Set 1 | port hardy
    -9.38930399681 -171.283075837
    Processing Record 875 of Set 1 | saleaula
    Oops, that city doesn't exist in OpenWeatherMap.
    -76.1041853951 18.8474017134
    Processing Record 875 of Set 1 | bredasdorp
    -68.3822202533 -45.7143010535
    Processing Record 876 of Set 1 | ushuaia
    -79.9153959274 -12.066964614
    Processing Record 877 of Set 1 | cape town
    53.2504899727 175.85059333
    Processing Record 878 of Set 1 | nikolskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    -24.8654924726 -93.0281759551
    Processing Record 878 of Set 1 | pisco
    -53.1295011978 41.1178844063
    Processing Record 879 of Set 1 | east london
    -23.8336907894 144.281591285
    Processing Record 880 of Set 1 | emerald
    -42.397656944 112.813088333
    Processing Record 881 of Set 1 | albany
    12.3936764282 2.78072135009
    Processing Record 882 of Set 1 | dosso
    -65.3760003977 -174.849934575
    Processing Record 883 of Set 1 | vaini
    4.95973864169 92.5393177005
    Processing Record 884 of Set 1 | banda aceh
    88.8109605482 -100.932476859
    Processing Record 885 of Set 1 | yellowknife
    -12.8157377578 139.905757797
    Processing Record 886 of Set 1 | nhulunbuy
    32.0823344641 -92.4272003335
    Processing Record 887 of Set 1 | ruston
    42.7380897776 96.6354074024
    Processing Record 888 of Set 1 | yumen
    -58.7163764345 -121.183507075
    Processing Record 889 of Set 1 | rikitea
    27.3371560243 35.2255961231
    Processing Record 890 of Set 1 | hurghada
    Oops, that city doesn't exist in OpenWeatherMap.
    -78.3985096121 -168.339660939
    Processing Record 890 of Set 1 | vaini
    -20.0382710634 -68.7894181661
    Processing Record 891 of Set 1 | iquique
    -87.4246962271 -179.223057162
    Processing Record 892 of Set 1 | vaini
    71.7223219544 29.1959091353
    Processing Record 893 of Set 1 | berlevag
    49.067252654 -23.8049842398
    Processing Record 894 of Set 1 | praia da vitoria
    -6.93467035818 156.602455195
    Processing Record 895 of Set 1 | buin
    Oops, that city doesn't exist in OpenWeatherMap.
    -72.0074508083 -133.546046639
    Processing Record 895 of Set 1 | rikitea
    27.8961416671 -62.6691021193
    Processing Record 896 of Set 1 | hamilton
    -66.0168018657 64.6906595775
    Processing Record 897 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -35.7048003852 -81.5803213592
    Processing Record 897 of Set 1 | lebu
    69.6375733367 2.0483067607
    Processing Record 898 of Set 1 | roald
    41.6627749495 111.079523359
    Processing Record 899 of Set 1 | hohhot
    7.47771502251 145.552299539
    Processing Record 900 of Set 1 | lorengau
    -16.0431159613 158.618498645
    Processing Record 901 of Set 1 | kirakira
    74.1994246209 1.3996461857
    Processing Record 902 of Set 1 | roald
    -63.607004213 -56.2400885991
    Processing Record 903 of Set 1 | ushuaia
    -26.9603447158 104.584250697
    Processing Record 904 of Set 1 | carnarvon
    83.2937955152 177.116367712
    Processing Record 905 of Set 1 | leningradskiy
    -73.5628750989 -7.81800115276
    Processing Record 906 of Set 1 | cape town
    -43.6264290115 47.3591681565
    Processing Record 907 of Set 1 | tsihombe
    Oops, that city doesn't exist in OpenWeatherMap.
    87.5840644751 -66.8883663
    Processing Record 907 of Set 1 | qaanaaq
    87.8949391965 9.54188421455
    Processing Record 908 of Set 1 | barentsburg
    Oops, that city doesn't exist in OpenWeatherMap.
    34.2636022935 -174.405524528
    Processing Record 908 of Set 1 | kapaa
    19.0992369767 158.367286409
    Processing Record 909 of Set 1 | butaritari
    20.9216546399 -31.3718143132
    Processing Record 910 of Set 1 | ponta do sol
    -15.2302420382 42.2782237275
    Processing Record 911 of Set 1 | mocambique
    Oops, that city doesn't exist in OpenWeatherMap.
    -87.6018005894 20.8111549839
    Processing Record 911 of Set 1 | bredasdorp
    85.7607664892 29.3794144898
    Processing Record 912 of Set 1 | mehamn
    58.6726435606 -174.486323118
    Processing Record 913 of Set 1 | provideniya
    -84.6178154821 32.0757360067
    Processing Record 914 of Set 1 | port elizabeth
    -59.346102518 109.347321714
    Processing Record 915 of Set 1 | albany
    -75.939697135 -140.903211976
    Processing Record 916 of Set 1 | rikitea
    12.1403654411 -96.490874536
    Processing Record 917 of Set 1 | pochutla
    Oops, that city doesn't exist in OpenWeatherMap.
    19.5379096898 -156.279363237
    Processing Record 917 of Set 1 | hilo
    9.12153256702 -120.77782448
    Processing Record 918 of Set 1 | cabo san lucas
    -87.8917448024 -10.8299956753
    Processing Record 919 of Set 1 | hermanus
    61.6101038139 172.22585127
    Processing Record 920 of Set 1 | kamenskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    7.75814949562 -18.0739951815
    Processing Record 920 of Set 1 | bubaque
    -82.4734782708 24.7775829043
    Processing Record 921 of Set 1 | bredasdorp
    79.8574578021 41.4892049489
    Processing Record 922 of Set 1 | ostrovnoy
    -57.0747947813 -42.9896319758
    Processing Record 923 of Set 1 | mar del plata
    66.9783259331 56.8986441765
    Processing Record 924 of Set 1 | usinsk
    -40.25454752 -30.6590187011
    Processing Record 925 of Set 1 | arraial do cabo
    19.9563092345 -107.528501355
    Processing Record 926 of Set 1 | tomatlan
    74.3710426579 33.3613748763
    Processing Record 927 of Set 1 | vardo
    33.9480422462 51.758875987
    Processing Record 928 of Set 1 | kashan
    65.294657142 -50.8811423645
    Processing Record 929 of Set 1 | nuuk
    78.3711649205 -7.56568303927
    Processing Record 930 of Set 1 | husavik
    -62.5994022052 -74.3933709446
    Processing Record 931 of Set 1 | ushuaia
    38.8873266954 -10.9697403933
    Processing Record 932 of Set 1 | colares
    56.5868878147 10.6639659663
    Processing Record 933 of Set 1 | ryomgard
    41.7466470151 -153.035955914
    Processing Record 934 of Set 1 | kodiak
    -66.870514828 -79.0261444536
    Processing Record 935 of Set 1 | punta arenas
    67.2782572091 -19.7947633628
    Processing Record 936 of Set 1 | skagastrond
    Oops, that city doesn't exist in OpenWeatherMap.
    -68.9848303788 174.894225849
    Processing Record 936 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    63.2750163554 -66.1589645071
    Processing Record 936 of Set 1 | iqaluit
    -84.8917719676 112.524719845
    Processing Record 937 of Set 1 | albany
    79.6739134032 -63.5910864595
    Processing Record 938 of Set 1 | narsaq
    55.4819695273 175.85298869
    Processing Record 939 of Set 1 | beringovskiy
    -83.1882781137 13.1952337573
    Processing Record 940 of Set 1 | bredasdorp
    67.9922966271 -85.2366954149
    Processing Record 941 of Set 1 | attawapiskat
    Oops, that city doesn't exist in OpenWeatherMap.
    50.2798687018 2.63562692272
    Processing Record 941 of Set 1 | arras
    49.6449195238 62.9629407923
    Processing Record 942 of Set 1 | svetlyy
    79.8357440112 -134.74139298
    Processing Record 943 of Set 1 | tuktoyaktuk
    -64.9413861868 64.5167094846
    Processing Record 944 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    54.9585129693 36.8143127926
    Processing Record 944 of Set 1 | zhukovo
    66.8665043244 0.736878960502
    Processing Record 945 of Set 1 | raudeberg
    Oops, that city doesn't exist in OpenWeatherMap.
    39.5936577759 -96.7966207358
    Processing Record 945 of Set 1 | manhattan
    -10.3184534007 124.573429801
    Processing Record 946 of Set 1 | soe
    -14.2663734244 -129.523222235
    Processing Record 947 of Set 1 | rikitea
    -25.8057912948 -72.0261327158
    Processing Record 948 of Set 1 | taltal
    -31.0743097136 140.678485174
    Processing Record 949 of Set 1 | broken hill
    61.5752733956 71.781447153
    Processing Record 950 of Set 1 | cheuskiny
    Oops, that city doesn't exist in OpenWeatherMap.
    -25.1759010689 -27.5330444467
    Processing Record 950 of Set 1 | vila velha
    -27.0590046274 130.118483332
    Processing Record 951 of Set 1 | yulara
    -40.2277532079 -2.29099911095
    Processing Record 952 of Set 1 | saldanha
    60.0749429578 -26.0523836444
    Processing Record 953 of Set 1 | grindavik
    -38.4148350892 -73.654140333
    Processing Record 954 of Set 1 | carahue
    -80.218035647 161.276425362
    Processing Record 955 of Set 1 | bluff
    41.5054528893 -104.293134336
    Processing Record 956 of Set 1 | torrington
    1.1256804009 -70.3898046096
    Processing Record 957 of Set 1 | mitu
    23.8899924109 -51.3753372336
    Processing Record 958 of Set 1 | codrington
    Oops, that city doesn't exist in OpenWeatherMap.
    -5.96929905782 -115.552513817
    Processing Record 958 of Set 1 | atuona
    12.5117022459 -7.69925362817
    Processing Record 959 of Set 1 | bamako
    -81.3519938154 -26.3969417812
    Processing Record 960 of Set 1 | ushuaia
    67.2337674719 -57.5378345178
    Processing Record 961 of Set 1 | sisimiut
    28.5901303789 89.7385668645
    Processing Record 962 of Set 1 | gasa
    -87.1287006681 170.232243551
    Processing Record 963 of Set 1 | bluff
    32.7167703367 -4.78530114809
    Processing Record 964 of Set 1 | mrirt
    Oops, that city doesn't exist in OpenWeatherMap.
    53.6871339368 20.0062381661
    Processing Record 964 of Set 1 | ostroda
    57.3044615007 -116.230398203
    Processing Record 965 of Set 1 | peace river
    15.754333524 -4.82455797672
    Processing Record 966 of Set 1 | tenenkou
    18.0471213958 127.716782978
    Processing Record 967 of Set 1 | pandan
    29.0246625992 -162.873634021
    Processing Record 968 of Set 1 | kapaa
    -10.3121016785 121.023871588
    Processing Record 969 of Set 1 | waingapu
    -63.6377309286 176.554467923
    Processing Record 970 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    -70.2582553388 55.9968286873
    Processing Record 970 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -81.5074537395 145.982418935
    Processing Record 970 of Set 1 | hobart
    -65.0863994542 -169.198264895
    Processing Record 971 of Set 1 | vaini
    -84.8516531002 -160.269168899
    Processing Record 972 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -29.3600905972 -6.27842925584
    Processing Record 972 of Set 1 | jamestown
    -18.2509212148 -25.6139765456
    Processing Record 973 of Set 1 | maceio
    62.3449448381 -103.214584411
    Processing Record 974 of Set 1 | la ronge
    -39.0689946411 -10.3095639275
    Processing Record 975 of Set 1 | jamestown
    36.6274519339 -95.6847776835
    Processing Record 976 of Set 1 | bartlesville
    29.0776618585 179.707202143
    Processing Record 977 of Set 1 | butaritari
    -43.8340805241 -56.0127559975
    Processing Record 978 of Set 1 | necochea
    -69.9942922717 20.657110668
    Processing Record 979 of Set 1 | bredasdorp
    -64.5407055932 -149.253034828
    Processing Record 980 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    69.7627717146 -53.6376822987
    Processing Record 980 of Set 1 | aasiaat
    7.9111407308 -55.6126175054
    Processing Record 981 of Set 1 | nieuw amsterdam
    -35.1314159731 -62.6753828099
    Processing Record 982 of Set 1 | lincoln
    48.1022556005 88.1383825272
    Processing Record 983 of Set 1 | altay
    -82.1548876159 -31.0884065495
    Processing Record 984 of Set 1 | ushuaia
    -64.9065093359 79.9264997294
    Processing Record 985 of Set 1 | busselton
    -68.6113075331 41.8975367928
    Processing Record 986 of Set 1 | port alfred
    -89.5127466373 70.9376562086
    Processing Record 987 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -71.2405490483 -129.657637546
    Processing Record 987 of Set 1 | rikitea
    -30.2568329303 -25.345942036
    Processing Record 988 of Set 1 | sao joao da barra
    -43.9968421548 -108.510516614
    Processing Record 989 of Set 1 | rikitea
    61.875008518 -71.4749578629
    Processing Record 990 of Set 1 | iqaluit
    -80.3916755709 -135.061064726
    Processing Record 991 of Set 1 | rikitea
    50.1865495248 31.6105759372
    Processing Record 992 of Set 1 | yahotyn
    49.8731333346 70.9146478551
    Processing Record 993 of Set 1 | atasu
    -57.2669534379 28.5554900885
    Processing Record 994 of Set 1 | port elizabeth
    -80.238836887 -152.935552393
    Processing Record 995 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    55.1389874264 -29.7291505154
    Processing Record 995 of Set 1 | grindavik
    80.3091019097 81.3642986364
    Processing Record 996 of Set 1 | dikson
    -17.4387927626 166.956014057
    Processing Record 997 of Set 1 | vila
    Oops, that city doesn't exist in OpenWeatherMap.
    -25.4268776567 45.6870021468
    Processing Record 997 of Set 1 | tsihombe
    Oops, that city doesn't exist in OpenWeatherMap.
    40.2165562379 -92.3307772501
    Processing Record 997 of Set 1 | kirksville
    13.0504115749 138.223182114
    Processing Record 998 of Set 1 | airai
    Oops, that city doesn't exist in OpenWeatherMap.
    -71.5517892965 90.5479401968
    Processing Record 998 of Set 1 | busselton
    65.3687183735 168.195868643
    Processing Record 999 of Set 1 | bilibino
    -20.8897638581 142.84743838
    Processing Record 1000 of Set 1 | mount isa
    38.1137693135 -105.510909204
    Processing Record 1001 of Set 1 | canon city
    -67.3687601761 -146.464375613
    Processing Record 1002 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -20.5175544439 69.9035319841
    Processing Record 1002 of Set 1 | grand river south east
    Oops, that city doesn't exist in OpenWeatherMap.
    -5.65069929966 -157.328785163
    Processing Record 1002 of Set 1 | faanui
    -86.4020061881 -106.959796769
    Processing Record 1003 of Set 1 | punta arenas
    54.4707277593 -21.2488495006
    Processing Record 1004 of Set 1 | vestmannaeyjar
    -84.1057583765 -60.4123524167
    Processing Record 1005 of Set 1 | ushuaia
    -21.8463219145 35.4698529768
    Processing Record 1006 of Set 1 | maxixe
    -79.3946566369 8.11992637097
    Processing Record 1007 of Set 1 | hermanus
    70.8327176506 161.416844467
    Processing Record 1008 of Set 1 | cherskiy
    -20.4648915575 112.626959166
    Processing Record 1009 of Set 1 | karratha
    50.9486563429 2.9626196662
    Processing Record 1010 of Set 1 | houthulst
    -50.255120374 8.34227645285
    Processing Record 1011 of Set 1 | cape town
    50.2410239265 -5.26335545502
    Processing Record 1012 of Set 1 | falmouth
    82.6868077065 112.307297194
    Processing Record 1013 of Set 1 | saskylakh
    -83.5743705248 124.712834522
    Processing Record 1014 of Set 1 | new norfolk
    31.9391659 1.83488698787
    Processing Record 1015 of Set 1 | aflu
    Oops, that city doesn't exist in OpenWeatherMap.
    13.8036776796 30.0134259678
    Processing Record 1015 of Set 1 | bara
    Oops, that city doesn't exist in OpenWeatherMap.
    6.61193673365 67.6011160671
    Processing Record 1015 of Set 1 | kulhudhuffushi
    0.280509521192 21.2610300896
    Processing Record 1016 of Set 1 | boende
    17.9694945873 130.74406437
    Processing Record 1017 of Set 1 | gigmoto
    -11.0537544591 78.7960505693
    Processing Record 1018 of Set 1 | hithadhoo
    62.4037336004 93.4409678131
    Processing Record 1019 of Set 1 | severo-yeniseyskiy
    23.0007451653 123.123060101
    Processing Record 1020 of Set 1 | ishigaki
    71.8871251139 -156.576080904
    Processing Record 1021 of Set 1 | barrow
    -72.0944782711 -91.271717807
    Processing Record 1022 of Set 1 | punta arenas
    -70.0645147651 158.269192698
    Processing Record 1023 of Set 1 | bluff
    86.5029374629 -153.844102779
    Processing Record 1024 of Set 1 | barrow
    46.5368953816 55.562064681
    Processing Record 1025 of Set 1 | beyneu
    -46.255188488 138.352568888
    Processing Record 1026 of Set 1 | portland
    -76.9348656914 102.114611719
    Processing Record 1027 of Set 1 | albany
    36.5107661714 -96.2269483314
    Processing Record 1028 of Set 1 | bartlesville
    16.6990545105 -39.2666858051
    Processing Record 1029 of Set 1 | ponta do sol
    -8.52539989199 93.6141211186
    Processing Record 1030 of Set 1 | bengkulu
    27.0435065746 85.8694221525
    Processing Record 1031 of Set 1 | janakpur
    0.141914258596 -11.0234034628
    Processing Record 1032 of Set 1 | harper
    22.5288569597 36.990713576
    Processing Record 1033 of Set 1 | jiddah
    Oops, that city doesn't exist in OpenWeatherMap.
    -9.50811761101 -62.7855363088
    Processing Record 1033 of Set 1 | ariquemes
    8.27064040048 41.5512858779
    Processing Record 1034 of Set 1 | bedesa
    -49.9269990733 115.219383795
    Processing Record 1035 of Set 1 | albany
    -81.6131939509 56.7356635434
    Processing Record 1036 of Set 1 | east london
    48.5019499461 50.3678541811
    Processing Record 1037 of Set 1 | inderborskiy
    Oops, that city doesn't exist in OpenWeatherMap.
    39.9935422789 -106.450308404
    Processing Record 1037 of Set 1 | steamboat springs
    21.600300866 -127.239807699
    Processing Record 1038 of Set 1 | san quintin
    -0.617091018518 22.2350579323
    Processing Record 1039 of Set 1 | boende
    59.9387859647 86.6110277006
    Processing Record 1040 of Set 1 | belyy yar
    7.40343341221 -5.60927597095
    Processing Record 1041 of Set 1 | beoumi
    -46.4593863956 -106.35721613
    Processing Record 1042 of Set 1 | castro
    82.4573190646 117.806644982
    Processing Record 1043 of Set 1 | saskylakh
    73.8120194031 -89.3604449648
    Processing Record 1044 of Set 1 | thompson
    -70.9650019204 141.735600965
    Processing Record 1045 of Set 1 | hobart
    -23.9138972971 -11.2309051303
    Processing Record 1046 of Set 1 | jamestown
    57.1115750883 -58.9318022537
    Processing Record 1047 of Set 1 | saint-augustin
    -78.9124158483 -129.759771208
    Processing Record 1048 of Set 1 | rikitea
    3.3963332427 -115.125703563
    Processing Record 1049 of Set 1 | san patricio
    -83.5308589733 65.1399458801
    Processing Record 1050 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    82.2322204935 23.882616474
    Processing Record 1050 of Set 1 | longyearbyen
    49.721268788 -46.0652852385
    Processing Record 1051 of Set 1 | torbay
    -31.2494151956 -91.0275594276
    Processing Record 1052 of Set 1 | lebu
    -6.54715722298 -112.79982311
    Processing Record 1053 of Set 1 | puerto ayora
    -63.8644024081 100.246990789
    Processing Record 1054 of Set 1 | busselton
    -58.9650439825 145.222846546
    Processing Record 1055 of Set 1 | hobart
    -7.02868600368 170.56523384
    Processing Record 1056 of Set 1 | lolua
    Oops, that city doesn't exist in OpenWeatherMap.
    -87.6642149649 108.034054699
    Processing Record 1056 of Set 1 | albany
    -86.6618593565 143.729629814
    Processing Record 1057 of Set 1 | hobart
    -25.1250530097 75.8541022614
    Processing Record 1058 of Set 1 | grand river south east
    Oops, that city doesn't exist in OpenWeatherMap.
    32.6957216656 -69.3276671927
    Processing Record 1058 of Set 1 | hamilton
    38.8504915618 146.813775894
    Processing Record 1059 of Set 1 | nemuro
    -37.0118611715 34.2866085644
    Processing Record 1060 of Set 1 | umzimvubu
    Oops, that city doesn't exist in OpenWeatherMap.
    6.74165207022 -57.3559303682
    Processing Record 1060 of Set 1 | fort wellington
    -40.2812943011 12.9497170194
    Processing Record 1061 of Set 1 | cape town
    -22.7157552298 51.8678157336
    Processing Record 1062 of Set 1 | saint-leu
    74.5328626552 16.5521581614
    Processing Record 1063 of Set 1 | longyearbyen
    85.0188515201 121.570589773
    Processing Record 1064 of Set 1 | saskylakh
    -51.3855983219 141.301387308
    Processing Record 1065 of Set 1 | new norfolk
    48.2395997111 2.40826004383
    Processing Record 1066 of Set 1 | pithiviers
    -8.73403286751 64.5297184058
    Processing Record 1067 of Set 1 | victoria
    51.2881723813 1.59451472005
    Processing Record 1068 of Set 1 | broadstairs
    -8.00132589951 147.564569852
    Processing Record 1069 of Set 1 | kokoda
    -68.0609062242 -148.096767482
    Processing Record 1070 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    61.185735538 -118.694143086
    Processing Record 1070 of Set 1 | hay river
    -70.9654673214 174.737193307
    Processing Record 1071 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    26.4344861144 45.3293444472
    Processing Record 1071 of Set 1 | buraydah
    17.3219484329 69.4316351771
    Processing Record 1072 of Set 1 | srivardhan
    36.8128860702 8.21014695844
    Processing Record 1073 of Set 1 | wadi maliz
    47.7269058232 -143.467775476
    Processing Record 1074 of Set 1 | sitka
    45.1824674434 -162.047031439
    Processing Record 1075 of Set 1 | bethel
    61.1032363354 80.6785083683
    Processing Record 1076 of Set 1 | kargasok
    -15.5539911839 -155.188670108
    Processing Record 1077 of Set 1 | faanui
    41.0910970508 -107.320618122
    Processing Record 1078 of Set 1 | craig
    77.0977827316 -83.9002715546
    Processing Record 1079 of Set 1 | qaanaaq
    32.2600102679 -28.3206590556
    Processing Record 1080 of Set 1 | ponta delgada
    71.2291436544 -45.6828130025
    Processing Record 1081 of Set 1 | ilulissat
    -77.1844805519 -50.5204220241
    Processing Record 1082 of Set 1 | ushuaia
    -67.8618425476 155.106434308
    Processing Record 1083 of Set 1 | bluff
    77.505512357 -174.92366666
    Processing Record 1084 of Set 1 | mys shmidta
    Oops, that city doesn't exist in OpenWeatherMap.
    55.1795690549 -125.494079752
    Processing Record 1084 of Set 1 | burns lake
    -46.7253664589 -30.7136255059
    Processing Record 1085 of Set 1 | cidreira
    74.4427138759 53.0151532852
    Processing Record 1086 of Set 1 | belushya guba
    Oops, that city doesn't exist in OpenWeatherMap.
    -26.1375577956 162.108172407
    Processing Record 1086 of Set 1 | poya
    20.6006901413 150.593052221
    Processing Record 1087 of Set 1 | katsuura
    -3.18229694448 152.710591176
    Processing Record 1088 of Set 1 | namatanai
    62.3874209048 29.2059476981
    Processing Record 1089 of Set 1 | joensuu
    37.0391168178 93.5478551359
    Processing Record 1090 of Set 1 | yumen
    -13.6360775438 127.091635583
    Processing Record 1091 of Set 1 | port keats
    -84.1902774681 -16.5734822712
    Processing Record 1092 of Set 1 | ushuaia
    -85.0702581435 69.3475647181
    Processing Record 1093 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -43.1015139042 53.3141616137
    Processing Record 1093 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -30.0163332058 2.48563061931
    Processing Record 1093 of Set 1 | luderitz
    72.3749351356 -119.35021274
    Processing Record 1094 of Set 1 | norman wells
    21.1234248439 -141.674812265
    Processing Record 1095 of Set 1 | hilo
    -76.5740178517 38.1929031031
    Processing Record 1096 of Set 1 | port alfred
    58.2103323753 -152.670512466
    Processing Record 1097 of Set 1 | kodiak
    31.0581199045 115.686773521
    Processing Record 1098 of Set 1 | macheng
    15.2338122146 -77.0490456941
    Processing Record 1099 of Set 1 | bull savanna
    69.7468174063 -42.0932856444
    Processing Record 1100 of Set 1 | tasiilaq
    -68.5429167226 131.530323396
    Processing Record 1101 of Set 1 | new norfolk
    -69.7710333197 -37.8263426856
    Processing Record 1102 of Set 1 | ushuaia
    -29.6392595624 -33.0094513569
    Processing Record 1103 of Set 1 | arraial do cabo
    -77.5624304378 148.415391018
    Processing Record 1104 of Set 1 | hobart
    -82.7937064062 39.5057957824
    Processing Record 1105 of Set 1 | port alfred
    38.2682839741 -12.6950881208
    Processing Record 1106 of Set 1 | colares
    -67.1288609633 36.9950157715
    Processing Record 1107 of Set 1 | port alfred
    -79.8938216764 89.1531742011
    Processing Record 1108 of Set 1 | busselton
    -57.2878332622 -38.3579166048
    Processing Record 1109 of Set 1 | mar del plata
    -41.2959690884 -176.303367846
    Processing Record 1110 of Set 1 | vaini
    89.1936936905 -39.2603356921
    Processing Record 1111 of Set 1 | ilulissat
    -4.35316130364 18.629601881
    Processing Record 1112 of Set 1 | bulungu
    78.483813625 -76.9917713231
    Processing Record 1113 of Set 1 | qaanaaq
    -53.1386793213 79.5298345654
    Processing Record 1114 of Set 1 | mahebourg
    41.5224139794 124.940819341
    Processing Record 1115 of Set 1 | huanren
    21.2001904958 77.1673122418
    Processing Record 1116 of Set 1 | anjangaon
    24.9445195005 46.4898064294
    Processing Record 1117 of Set 1 | riyadh
    -50.6205304645 -13.0395112346
    Processing Record 1118 of Set 1 | jamestown
    34.0262126207 22.7195717535
    Processing Record 1119 of Set 1 | darnah
    69.1870053778 8.00620903153
    Processing Record 1120 of Set 1 | sorland
    13.5176379668 -148.492937964
    Processing Record 1121 of Set 1 | hilo
    -62.7383314673 86.7830712301
    Processing Record 1122 of Set 1 | busselton
    -16.1840945863 129.803249444
    Processing Record 1123 of Set 1 | kununurra
    -21.1202670651 85.3673889087
    Processing Record 1124 of Set 1 | hithadhoo
    -52.8386920902 -85.3881548082
    Processing Record 1125 of Set 1 | punta arenas
    -79.8856212394 -98.531451548
    Processing Record 1126 of Set 1 | punta arenas
    23.2016445463 175.300815756
    Processing Record 1127 of Set 1 | butaritari
    0.475349692921 -89.7823263646
    Processing Record 1128 of Set 1 | puerto ayora
    -24.7503488593 113.248608938
    Processing Record 1129 of Set 1 | carnarvon
    -88.2409810889 102.821895388
    Processing Record 1130 of Set 1 | albany
    -15.3521510835 162.14237267
    Processing Record 1131 of Set 1 | kirakira
    64.9850607903 -48.52702766
    Processing Record 1132 of Set 1 | paamiut
    -5.86888251449 40.419511859
    Processing Record 1133 of Set 1 | mtambile
    -17.8676381186 -57.1135441189
    Processing Record 1134 of Set 1 | puerto quijarro
    -2.55062548414 24.5769106338
    Processing Record 1135 of Set 1 | kindu
    74.4864196984 -15.0098437895
    Processing Record 1136 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    15.6113117957 -52.7936474265
    Processing Record 1136 of Set 1 | bathsheba
    -10.0894580805 136.307754276
    Processing Record 1137 of Set 1 | galiwinku
    Oops, that city doesn't exist in OpenWeatherMap.
    -17.1632842589 -28.7274452872
    Processing Record 1137 of Set 1 | coruripe
    6.87925936612 13.309031188
    Processing Record 1138 of Set 1 | ngaoundere
    71.5575897495 110.479130261
    Processing Record 1139 of Set 1 | saskylakh
    -44.0567904063 -127.678158309
    Processing Record 1140 of Set 1 | rikitea
    4.64036281528 -109.45778795
    Processing Record 1141 of Set 1 | lazaro cardenas
    64.6946790697 166.250276781
    Processing Record 1142 of Set 1 | kamenskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    -48.719583144 -174.751039091
    Processing Record 1142 of Set 1 | vaini
    71.4152339033 74.3338347739
    Processing Record 1143 of Set 1 | pangody
    14.292370438 -24.603559479
    Processing Record 1144 of Set 1 | sao filipe
    22.7611486274 -109.13592096
    Processing Record 1145 of Set 1 | cabo san lucas
    40.0311792303 -153.161211614
    Processing Record 1146 of Set 1 | kodiak
    -36.7432915635 -164.860169001
    Processing Record 1147 of Set 1 | avarua
    62.9240838144 -132.223755478
    Processing Record 1148 of Set 1 | whitehorse
    -51.9149429208 135.192994101
    Processing Record 1149 of Set 1 | new norfolk
    -79.0058593501 -161.194205189
    Processing Record 1150 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -36.3303842672 143.807678152
    Processing Record 1150 of Set 1 | bendigo
    2.89028608213 -60.4653499963
    Processing Record 1151 of Set 1 | boa vista
    88.5513884412 173.272029613
    Processing Record 1152 of Set 1 | pevek
    -87.263626599 -35.9696669683
    Processing Record 1153 of Set 1 | ushuaia
    -24.8944136221 36.2031085133
    Processing Record 1154 of Set 1 | inhambane
    9.96786863587 35.4841641775
    Processing Record 1155 of Set 1 | mendi
    -46.0411573796 -88.6993296407
    Processing Record 1156 of Set 1 | castro
    85.7366647297 163.892563883
    Processing Record 1157 of Set 1 | cherskiy
    69.2638089014 84.1945471002
    Processing Record 1158 of Set 1 | karaul
    Oops, that city doesn't exist in OpenWeatherMap.
    87.6655859556 -122.746947463
    Processing Record 1158 of Set 1 | tuktoyaktuk
    85.9770041066 25.9338354665
    Processing Record 1159 of Set 1 | longyearbyen
    -16.9329482716 14.558031188
    Processing Record 1160 of Set 1 | ongandjera
    11.058860166 -28.8671656067
    Processing Record 1161 of Set 1 | sao filipe
    59.394816043 -140.909085111
    Processing Record 1162 of Set 1 | haines junction
    -34.6714429954 120.627613108
    Processing Record 1163 of Set 1 | esperance
    -39.2334086743 -100.712195866
    Processing Record 1164 of Set 1 | ancud
    58.5591699314 18.9365688238
    Processing Record 1165 of Set 1 | vasterhaninge
    31.7938237057 59.7463289071
    Processing Record 1166 of Set 1 | birjand
    -16.1983280331 -140.764384147
    Processing Record 1167 of Set 1 | atuona
    -20.3277847785 119.356369247
    Processing Record 1168 of Set 1 | port hedland
    -59.082041174 -88.4445442225
    Processing Record 1169 of Set 1 | punta arenas
    -73.7952220167 -21.3345499177
    Processing Record 1170 of Set 1 | ushuaia
    -6.9524519222 -57.898067046
    Processing Record 1171 of Set 1 | jacareacanga
    89.5513441445 -79.9179939086
    Processing Record 1172 of Set 1 | qaanaaq
    -62.0380925347 -157.475354547
    Processing Record 1173 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    60.5274295234 -81.4953327607
    Processing Record 1173 of Set 1 | attawapiskat
    Oops, that city doesn't exist in OpenWeatherMap.
    -12.9795928261 -73.0057904673
    Processing Record 1173 of Set 1 | santa ana
    -80.9186160943 19.1793933857
    Processing Record 1174 of Set 1 | bredasdorp
    -54.0767292943 110.374865881
    Processing Record 1175 of Set 1 | albany
    64.7261062657 166.071699337
    Processing Record 1176 of Set 1 | kamenskoye
    Oops, that city doesn't exist in OpenWeatherMap.
    -41.0873776347 50.1650847412
    Processing Record 1176 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -11.2846490462 82.083513579
    Processing Record 1176 of Set 1 | hithadhoo
    70.160824611 -172.554213542
    Processing Record 1177 of Set 1 | lavrentiya
    54.3450585545 103.533454789
    Processing Record 1178 of Set 1 | ust-uda
    -77.3073227816 109.724534324
    Processing Record 1179 of Set 1 | albany
    40.3774054435 57.1553572972
    Processing Record 1180 of Set 1 | baherden
    10.6405804589 -79.5000422003
    Processing Record 1181 of Set 1 | portobelo
    -8.33988216353 -73.7358285318
    Processing Record 1182 of Set 1 | pucallpa
    -12.6426611855 151.9616408
    Processing Record 1183 of Set 1 | samarai
    -51.3653165123 62.5091545075
    Processing Record 1184 of Set 1 | taolanaro
    Oops, that city doesn't exist in OpenWeatherMap.
    -36.9627705732 77.6262790182
    Processing Record 1184 of Set 1 | bambous virieux
    -68.6342616781 174.415882857
    Processing Record 1185 of Set 1 | kaitangata
    Oops, that city doesn't exist in OpenWeatherMap.
    -30.8946415161 97.3867027812
    Processing Record 1185 of Set 1 | carnarvon
    -56.6583012728 -155.777084777
    Processing Record 1186 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    28.9464506186 -139.310625985
    Processing Record 1186 of Set 1 | hilo
    71.3659816163 -140.707454474
    Processing Record 1187 of Set 1 | aklavik
    -64.9233872149 131.094933679
    Processing Record 1188 of Set 1 | new norfolk
    13.1796609251 -5.5609556829
    Processing Record 1189 of Set 1 | san
    -31.0813956022 -131.729044212
    Processing Record 1190 of Set 1 | rikitea
    -64.2857982256 -119.546361197
    Processing Record 1191 of Set 1 | rikitea
    63.1030731728 -50.0967996996
    Processing Record 1192 of Set 1 | paamiut
    69.5958524785 -68.9302941631
    Processing Record 1193 of Set 1 | clyde river
    60.3077427664 -94.1606385495
    Processing Record 1194 of Set 1 | thompson
    -68.6830514423 9.3317458752
    Processing Record 1195 of Set 1 | hermanus
    -66.0771943577 96.1464980088
    Processing Record 1196 of Set 1 | busselton
    84.1829210819 16.2221981337
    Processing Record 1197 of Set 1 | longyearbyen
    -78.7368960627 133.011752733
    Processing Record 1198 of Set 1 | new norfolk
    -30.1576459651 -70.6633915221
    Processing Record 1199 of Set 1 | vicuna
    -37.0670763412 -77.2619074417
    Processing Record 1200 of Set 1 | lebu
    17.2057527113 -75.5181287326
    Processing Record 1201 of Set 1 | morant bay
    -3.47836735184 -63.8387410959
    Processing Record 1202 of Set 1 | tefe
    -43.9490910066 5.26154948801
    Processing Record 1203 of Set 1 | cape town
    0.0272173618827 -66.3792335382
    Processing Record 1204 of Set 1 | sao gabriel da cachoeira
    34.9971131823 -121.068660836
    Processing Record 1205 of Set 1 | morro bay
    74.6487721321 61.4718213433
    Processing Record 1206 of Set 1 | amderma
    Oops, that city doesn't exist in OpenWeatherMap.
    62.5856038431 78.4451616386
    Processing Record 1206 of Set 1 | novoagansk
    -3.2373352715 -148.39269941
    Processing Record 1207 of Set 1 | atuona
    -51.5612779122 -1.68532550615
    Processing Record 1208 of Set 1 | cape town
    -60.0859253887 105.238725045
    Processing Record 1209 of Set 1 | albany
    69.7866672497 -121.288437941
    Processing Record 1210 of Set 1 | norman wells
    87.7465300403 -87.6889980735
    Processing Record 1211 of Set 1 | qaanaaq
    -0.665175696216 -108.577000128
    Processing Record 1212 of Set 1 | puerto ayora
    -76.0295778247 7.91498386494
    Processing Record 1213 of Set 1 | hermanus
    -60.6769759566 79.9463593627
    Processing Record 1214 of Set 1 | busselton
    22.7964055962 -158.626757141
    Processing Record 1215 of Set 1 | kapaa
    -41.2926061442 8.20030514825
    Processing Record 1216 of Set 1 | cape town
    23.8403547438 -38.9337952972
    Processing Record 1217 of Set 1 | ponta do sol
    -3.63246540006 14.2398140694
    Processing Record 1218 of Set 1 | madingou
    16.746126802 170.207091059
    Processing Record 1219 of Set 1 | butaritari
    9.12001722109 -100.725237013
    Processing Record 1220 of Set 1 | tecoanapa
    58.5933837438 130.36200422
    Processing Record 1221 of Set 1 | chagda
    Oops, that city doesn't exist in OpenWeatherMap.
    -67.4719742542 -161.00586433
    Processing Record 1221 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    22.5727644625 112.983148149
    Processing Record 1221 of Set 1 | jiangmen
    28.882268839 -162.222355491
    Processing Record 1222 of Set 1 | kapaa
    -80.2499163488 -96.9526975556
    Processing Record 1223 of Set 1 | punta arenas
    3.50613813097 77.1835001442
    Processing Record 1224 of Set 1 | felidhoo
    Oops, that city doesn't exist in OpenWeatherMap.
    74.0225852326 -142.705688553
    Processing Record 1224 of Set 1 | aklavik
    -17.5092378991 12.8908911551
    Processing Record 1225 of Set 1 | opuwo
    -25.8324729736 -81.2369852992
    Processing Record 1226 of Set 1 | coquimbo
    -34.4931971118 63.9002727083
    Processing Record 1227 of Set 1 | souillac
    65.207145491 158.671649617
    Processing Record 1228 of Set 1 | evensk
    -77.4277813225 -179.122254001
    Processing Record 1229 of Set 1 | vaini
    -38.6212871575 67.1974403828
    Processing Record 1230 of Set 1 | souillac
    49.5309430456 -30.860281849
    Processing Record 1231 of Set 1 | lagoa
    44.8066560245 -1.23435718054
    Processing Record 1232 of Set 1 | arcachon
    68.2910851393 81.7806872447
    Processing Record 1233 of Set 1 | karaul
    Oops, that city doesn't exist in OpenWeatherMap.
    -87.4170386222 56.6474056881
    Processing Record 1233 of Set 1 | port alfred
    77.5929906505 68.5943625771
    Processing Record 1234 of Set 1 | amderma
    Oops, that city doesn't exist in OpenWeatherMap.
    -71.5467936745 125.048795116
    Processing Record 1234 of Set 1 | new norfolk
    -84.6697668718 -173.51444625
    Processing Record 1235 of Set 1 | vaini
    -27.7424389382 -21.5841836404
    Processing Record 1236 of Set 1 | jamestown
    -79.3240601826 57.6519198526
    Processing Record 1237 of Set 1 | east london
    20.4044602927 121.925126731
    Processing Record 1238 of Set 1 | basco
    66.5261639665 -126.371879569
    Processing Record 1239 of Set 1 | norman wells
    -30.918626911 -138.052646813
    Processing Record 1240 of Set 1 | rikitea
    35.4455194233 145.43162248
    Processing Record 1241 of Set 1 | hasaki
    -55.9787756388 -4.85555720546
    Processing Record 1242 of Set 1 | cape town
    13.5679438623 7.86928453857
    Processing Record 1243 of Set 1 | aguie
    53.1376298582 42.1317043489
    Processing Record 1244 of Set 1 | pichayevo
    20.2577223224 -175.30468846
    Processing Record 1245 of Set 1 | kapaa
    13.3613571995 -162.667898637
    Processing Record 1246 of Set 1 | makakilo city
    -12.9496253966 36.760286189
    Processing Record 1247 of Set 1 | lichinga
    66.9201611056 90.9560293075
    Processing Record 1248 of Set 1 | svetlogorsk
    8.89850258095 30.5131787125
    Processing Record 1249 of Set 1 | ler
    Oops, that city doesn't exist in OpenWeatherMap.
    -58.2006468166 -160.902476543
    Processing Record 1249 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    12.4927025328 130.079730086
    Processing Record 1249 of Set 1 | sulangan
    -62.1333942789 154.944866148
    Processing Record 1250 of Set 1 | tuatapere
    33.0095511116 6.46888836385
    Processing Record 1251 of Set 1 | tuggurt
    Oops, that city doesn't exist in OpenWeatherMap.
    76.3708429898 135.555129389
    Processing Record 1251 of Set 1 | nizhneyansk
    Oops, that city doesn't exist in OpenWeatherMap.
    82.9053555152 59.4869437278
    Processing Record 1251 of Set 1 | amderma
    Oops, that city doesn't exist in OpenWeatherMap.
    30.2791271887 77.1224737113
    Processing Record 1251 of Set 1 | mustafabad
    87.8963359043 -175.047114198
    Processing Record 1252 of Set 1 | mys shmidta
    Oops, that city doesn't exist in OpenWeatherMap.
    57.8889324446 59.9510938641
    Processing Record 1252 of Set 1 | nizhniy tagil
    19.9492377353 -3.4713119103
    Processing Record 1253 of Set 1 | araouane
    -13.1516620995 -9.65721259763
    Processing Record 1254 of Set 1 | jamestown
    63.8039600705 -16.223898024
    Processing Record 1255 of Set 1 | hofn
    64.3170185675 129.58764413
    Processing Record 1256 of Set 1 | namtsy
    40.5111428944 94.4719095136
    Processing Record 1257 of Set 1 | hami
    -89.7860367043 -128.607829175
    Processing Record 1258 of Set 1 | rikitea
    -13.7190466008 -87.0306823272
    Processing Record 1259 of Set 1 | huarmey
    -52.255802612 -155.220251763
    Processing Record 1260 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -34.0858411931 -135.909894585
    Processing Record 1260 of Set 1 | rikitea
    -43.7005689016 -4.41353453486
    Processing Record 1261 of Set 1 | saldanha
    36.4164298908 -77.1774424229
    Processing Record 1262 of Set 1 | roanoke rapids
    77.1303543185 -153.930205155
    Processing Record 1263 of Set 1 | barrow
    19.6155495764 -72.2178864509
    Processing Record 1264 of Set 1 | grande riviere du nord
    -17.7070804884 -155.294858323
    Processing Record 1265 of Set 1 | vaitape
    -81.9840326961 -4.19490340921
    Processing Record 1266 of Set 1 | hermanus
    80.7381811862 -12.1312296094
    Processing Record 1267 of Set 1 | illoqqortoormiut
    Oops, that city doesn't exist in OpenWeatherMap.
    -18.9954898833 93.4768021057
    Processing Record 1267 of Set 1 | bengkulu
    -17.0666748366 -121.537459759
    Processing Record 1268 of Set 1 | rikitea
    66.8162406205 166.551248374
    Processing Record 1269 of Set 1 | bilibino
    20.435494278 64.9206546094
    Processing Record 1270 of Set 1 | dwarka
    -73.2863352601 147.752510305
    Processing Record 1271 of Set 1 | hobart
    10.1692134426 129.22191571
    Processing Record 1272 of Set 1 | pilar
    -22.7673221331 -159.272986824
    Processing Record 1273 of Set 1 | avarua
    -65.886014036 -53.7063018627
    Processing Record 1274 of Set 1 | ushuaia
    33.8955365976 168.541367857
    Processing Record 1275 of Set 1 | severo-kurilsk
    -71.0310308712 48.4125570086
    Processing Record 1276 of Set 1 | port alfred
    80.8275235332 178.773879604
    Processing Record 1277 of Set 1 | leningradskiy
    -73.2112659941 -163.64564281
    Processing Record 1278 of Set 1 | mataura
    Oops, that city doesn't exist in OpenWeatherMap.
    -45.0569264264 40.8220044069
    Processing Record 1278 of Set 1 | umzimvubu
    Oops, that city doesn't exist in OpenWeatherMap.
    64.8972905012 30.9542634267
    Processing Record 1278 of Set 1 | kostomuksha
    -19.6469942558 -118.927028028
    Processing Record 1279 of Set 1 | rikitea
    -36.7517712381 162.91007443
    Processing Record 1280 of Set 1 | te anau
    52.0528182773 99.7305913259
    Processing Record 1281 of Set 1 | orlik
    4.92701312618 112.822056472
    Processing Record 1282 of Set 1 | miri
    -32.2798327912 -84.025661026
    Processing Record 1283 of Set 1 | lebu
    -13.8332216779 20.4439827463
    Processing Record 1284 of Set 1 | luena
    17.9697008181 138.977252853
    Processing Record 1285 of Set 1 | airai
    Oops, that city doesn't exist in OpenWeatherMap.
    -73.2679422197 -130.382244541
    Processing Record 1285 of Set 1 | rikitea
    56.1215887883 30.3704567408
    Processing Record 1286 of Set 1 | velikie luki
    Oops, that city doesn't exist in OpenWeatherMap.
    79.6413737945 94.8609380769
    Processing Record 1286 of Set 1 | khatanga
    -80.3268958188 -107.348732596
    Processing Record 1287 of Set 1 | punta arenas
    -29.7489922214 35.9287186457
    Processing Record 1288 of Set 1 | richards bay
    -86.6533223326 89.4825328566
    Processing Record 1289 of Set 1 | albany



```python
# Create and populate summary dataframe

summary_df = pd.DataFrame({'City':cities,
                           'Country':countries,
                           'Latitude':lats,
                           'Longitude':lngs,
                           'Timestamp':timestamps,
                           'Date':dates,
                           'Max Temperature':max_temps,
                           'Humidity':humidities,
                           'Windspeed':windspeeds,
                           'Cloudiness':cloudiness
                          })

# Constrain data to unique cities
summary_df.drop_duplicates()

# Configure dataframe column order and display
summary_df = summary_df[['City',
                         'Country',
                         'Latitude',
                         'Longitude',
                         'Timestamp',
                         'Date',
                         'Max Temperature',
                         'Humidity',
                         'Windspeed',
                         'Cloudiness']]

summary_df
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
      <th>City</th>
      <th>Country</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Timestamp</th>
      <th>Date</th>
      <th>Max Temperature</th>
      <th>Humidity</th>
      <th>Windspeed</th>
      <th>Cloudiness</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>port alfred</td>
      <td>za</td>
      <td>-71.460689</td>
      <td>41.323325</td>
      <td>1511277040</td>
      <td>11-21-2017</td>
      <td>67.75</td>
      <td>85</td>
      <td>17.02</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>san patricio</td>
      <td>mx</td>
      <td>13.143887</td>
      <td>-108.930492</td>
      <td>1511272020</td>
      <td>11-21-2017</td>
      <td>75.20</td>
      <td>94</td>
      <td>3.15</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tulum</td>
      <td>mx</td>
      <td>18.767706</td>
      <td>-86.521111</td>
      <td>1511271600</td>
      <td>11-21-2017</td>
      <td>77.00</td>
      <td>78</td>
      <td>2.37</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>aguas belas</td>
      <td>br</td>
      <td>-9.070107</td>
      <td>-37.227860</td>
      <td>1511276966</td>
      <td>11-21-2017</td>
      <td>91.38</td>
      <td>36</td>
      <td>9.86</td>
      <td>8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bredasdorp</td>
      <td>za</td>
      <td>-40.426650</td>
      <td>21.614013</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>68.00</td>
      <td>45</td>
      <td>19.46</td>
      <td>48</td>
    </tr>
    <tr>
      <th>5</th>
      <td>khatanga</td>
      <td>ru</td>
      <td>85.331334</td>
      <td>94.838027</td>
      <td>1511277042</td>
      <td>11-21-2017</td>
      <td>-3.22</td>
      <td>77</td>
      <td>3.49</td>
      <td>64</td>
    </tr>
    <tr>
      <th>6</th>
      <td>bima</td>
      <td>id</td>
      <td>-7.762703</td>
      <td>118.060211</td>
      <td>1511277024</td>
      <td>11-21-2017</td>
      <td>77.43</td>
      <td>100</td>
      <td>3.27</td>
      <td>88</td>
    </tr>
    <tr>
      <th>7</th>
      <td>rikitea</td>
      <td>pf</td>
      <td>-86.464802</td>
      <td>-136.159612</td>
      <td>1511277043</td>
      <td>11-21-2017</td>
      <td>73.60</td>
      <td>100</td>
      <td>13.22</td>
      <td>68</td>
    </tr>
    <tr>
      <th>8</th>
      <td>punta arenas</td>
      <td>cl</td>
      <td>-52.877178</td>
      <td>-75.162463</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>51.80</td>
      <td>53</td>
      <td>28.86</td>
      <td>75</td>
    </tr>
    <tr>
      <th>9</th>
      <td>rikitea</td>
      <td>pf</td>
      <td>-63.999575</td>
      <td>-128.702328</td>
      <td>1511277043</td>
      <td>11-21-2017</td>
      <td>73.60</td>
      <td>100</td>
      <td>13.22</td>
      <td>68</td>
    </tr>
    <tr>
      <th>10</th>
      <td>san quintin</td>
      <td>mx</td>
      <td>23.437531</td>
      <td>-126.396218</td>
      <td>1511277044</td>
      <td>11-21-2017</td>
      <td>51.10</td>
      <td>65</td>
      <td>2.37</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>puerto ayacucho</td>
      <td>ve</td>
      <td>4.833900</td>
      <td>-64.368439</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>86.00</td>
      <td>70</td>
      <td>8.05</td>
      <td>20</td>
    </tr>
    <tr>
      <th>12</th>
      <td>faanui</td>
      <td>pf</td>
      <td>-6.985938</td>
      <td>-160.156108</td>
      <td>1511277045</td>
      <td>11-21-2017</td>
      <td>78.78</td>
      <td>100</td>
      <td>15.23</td>
      <td>76</td>
    </tr>
    <tr>
      <th>13</th>
      <td>tasiilaq</td>
      <td>gl</td>
      <td>64.507015</td>
      <td>-31.092402</td>
      <td>1511275800</td>
      <td>11-21-2017</td>
      <td>23.00</td>
      <td>57</td>
      <td>3.36</td>
      <td>40</td>
    </tr>
    <tr>
      <th>14</th>
      <td>hilo</td>
      <td>us</td>
      <td>13.676205</td>
      <td>-137.256500</td>
      <td>1511272560</td>
      <td>11-21-2017</td>
      <td>49.01</td>
      <td>72</td>
      <td>7.18</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>adrar</td>
      <td>dz</td>
      <td>29.689560</td>
      <td>-0.782711</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>69.80</td>
      <td>35</td>
      <td>5.17</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>havelock</td>
      <td>us</td>
      <td>32.527613</td>
      <td>-71.685014</td>
      <td>1511272680</td>
      <td>11-21-2017</td>
      <td>49.62</td>
      <td>81</td>
      <td>2.15</td>
      <td>5</td>
    </tr>
    <tr>
      <th>17</th>
      <td>saint george</td>
      <td>bm</td>
      <td>33.067826</td>
      <td>-54.944235</td>
      <td>1511272500</td>
      <td>11-21-2017</td>
      <td>69.80</td>
      <td>60</td>
      <td>13.87</td>
      <td>75</td>
    </tr>
    <tr>
      <th>18</th>
      <td>port alfred</td>
      <td>za</td>
      <td>-83.129959</td>
      <td>54.309035</td>
      <td>1511277040</td>
      <td>11-21-2017</td>
      <td>67.75</td>
      <td>85</td>
      <td>17.02</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>meulaboh</td>
      <td>id</td>
      <td>0.427764</td>
      <td>88.403744</td>
      <td>1511277047</td>
      <td>11-21-2017</td>
      <td>79.77</td>
      <td>100</td>
      <td>1.14</td>
      <td>76</td>
    </tr>
    <tr>
      <th>20</th>
      <td>chuy</td>
      <td>uy</td>
      <td>-53.809398</td>
      <td>-32.219344</td>
      <td>1511277048</td>
      <td>11-21-2017</td>
      <td>61.05</td>
      <td>79</td>
      <td>17.81</td>
      <td>92</td>
    </tr>
    <tr>
      <th>21</th>
      <td>nabire</td>
      <td>id</td>
      <td>-3.846523</td>
      <td>137.387738</td>
      <td>1511277048</td>
      <td>11-21-2017</td>
      <td>69.42</td>
      <td>98</td>
      <td>1.92</td>
      <td>56</td>
    </tr>
    <tr>
      <th>22</th>
      <td>nouadhibou</td>
      <td>mr</td>
      <td>23.033512</td>
      <td>-18.238969</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>75.20</td>
      <td>53</td>
      <td>3.36</td>
      <td>20</td>
    </tr>
    <tr>
      <th>23</th>
      <td>saint george</td>
      <td>bm</td>
      <td>29.969897</td>
      <td>-47.718349</td>
      <td>1511272500</td>
      <td>11-21-2017</td>
      <td>69.80</td>
      <td>60</td>
      <td>13.87</td>
      <td>75</td>
    </tr>
    <tr>
      <th>24</th>
      <td>ushuaia</td>
      <td>ar</td>
      <td>-83.291858</td>
      <td>-19.081350</td>
      <td>1511277050</td>
      <td>11-21-2017</td>
      <td>43.36</td>
      <td>81</td>
      <td>10.76</td>
      <td>92</td>
    </tr>
    <tr>
      <th>25</th>
      <td>khatanga</td>
      <td>ru</td>
      <td>75.622024</td>
      <td>106.025434</td>
      <td>1511277042</td>
      <td>11-21-2017</td>
      <td>-3.22</td>
      <td>77</td>
      <td>3.49</td>
      <td>64</td>
    </tr>
    <tr>
      <th>26</th>
      <td>bluff</td>
      <td>nz</td>
      <td>-56.130960</td>
      <td>169.102249</td>
      <td>1511277050</td>
      <td>11-21-2017</td>
      <td>53.17</td>
      <td>100</td>
      <td>1.59</td>
      <td>44</td>
    </tr>
    <tr>
      <th>27</th>
      <td>san quintin</td>
      <td>mx</td>
      <td>24.160972</td>
      <td>-124.277960</td>
      <td>1511277044</td>
      <td>11-21-2017</td>
      <td>51.10</td>
      <td>65</td>
      <td>2.37</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>hobart</td>
      <td>au</td>
      <td>-87.006278</td>
      <td>139.580399</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>59.00</td>
      <td>82</td>
      <td>10.29</td>
      <td>20</td>
    </tr>
    <tr>
      <th>29</th>
      <td>nouadhibou</td>
      <td>mr</td>
      <td>23.059103</td>
      <td>-21.522026</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>75.20</td>
      <td>53</td>
      <td>3.36</td>
      <td>20</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1244</th>
      <td>la ronge</td>
      <td>ca</td>
      <td>56.808591</td>
      <td>-105.409745</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>-2.21</td>
      <td>84</td>
      <td>8.05</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1245</th>
      <td>torbay</td>
      <td>ca</td>
      <td>43.956578</td>
      <td>-49.681352</td>
      <td>1511276400</td>
      <td>11-21-2017</td>
      <td>34.74</td>
      <td>64</td>
      <td>25.28</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1246</th>
      <td>iqaluit</td>
      <td>ca</td>
      <td>66.841516</td>
      <td>-70.658258</td>
      <td>1511276400</td>
      <td>11-21-2017</td>
      <td>37.40</td>
      <td>87</td>
      <td>29.97</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1247</th>
      <td>san patricio</td>
      <td>mx</td>
      <td>3.213247</td>
      <td>-116.383934</td>
      <td>1511272020</td>
      <td>11-21-2017</td>
      <td>75.20</td>
      <td>94</td>
      <td>3.15</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1248</th>
      <td>yellowknife</td>
      <td>ca</td>
      <td>66.940841</td>
      <td>-115.195594</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>-5.81</td>
      <td>83</td>
      <td>4.70</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1249</th>
      <td>tuktoyaktuk</td>
      <td>ca</td>
      <td>78.963786</td>
      <td>-125.605908</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>-16.61</td>
      <td>75</td>
      <td>10.29</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1250</th>
      <td>vaini</td>
      <td>to</td>
      <td>-89.788379</td>
      <td>-177.574524</td>
      <td>1511276400</td>
      <td>11-21-2017</td>
      <td>75.20</td>
      <td>100</td>
      <td>19.48</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1251</th>
      <td>matagami</td>
      <td>ca</td>
      <td>50.842536</td>
      <td>-78.136873</td>
      <td>1511276580</td>
      <td>11-21-2017</td>
      <td>30.20</td>
      <td>100</td>
      <td>11.41</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1252</th>
      <td>cockburn town</td>
      <td>bs</td>
      <td>25.142077</td>
      <td>-73.865334</td>
      <td>1511277405</td>
      <td>11-21-2017</td>
      <td>80.58</td>
      <td>95</td>
      <td>21.61</td>
      <td>64</td>
    </tr>
    <tr>
      <th>1253</th>
      <td>dikson</td>
      <td>ru</td>
      <td>88.428706</td>
      <td>80.644109</td>
      <td>1511277074</td>
      <td>11-21-2017</td>
      <td>5.25</td>
      <td>100</td>
      <td>11.32</td>
      <td>64</td>
    </tr>
    <tr>
      <th>1254</th>
      <td>bredasdorp</td>
      <td>za</td>
      <td>-77.516365</td>
      <td>17.344162</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>68.00</td>
      <td>45</td>
      <td>19.46</td>
      <td>48</td>
    </tr>
    <tr>
      <th>1255</th>
      <td>bilibino</td>
      <td>ru</td>
      <td>72.872028</td>
      <td>165.230973</td>
      <td>1511277406</td>
      <td>11-21-2017</td>
      <td>4.80</td>
      <td>73</td>
      <td>8.52</td>
      <td>80</td>
    </tr>
    <tr>
      <th>1256</th>
      <td>el alto</td>
      <td>pe</td>
      <td>-4.121394</td>
      <td>-82.977670</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>69.80</td>
      <td>73</td>
      <td>19.46</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1257</th>
      <td>bomi</td>
      <td>sl</td>
      <td>7.283808</td>
      <td>-11.467420</td>
      <td>1511277407</td>
      <td>11-21-2017</td>
      <td>74.77</td>
      <td>100</td>
      <td>3.38</td>
      <td>92</td>
    </tr>
    <tr>
      <th>1258</th>
      <td>bossier city</td>
      <td>us</td>
      <td>32.253661</td>
      <td>-93.649869</td>
      <td>1511276100</td>
      <td>11-21-2017</td>
      <td>46.33</td>
      <td>66</td>
      <td>5.82</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1259</th>
      <td>rikitea</td>
      <td>pf</td>
      <td>-59.830477</td>
      <td>-123.578075</td>
      <td>1511277043</td>
      <td>11-21-2017</td>
      <td>73.60</td>
      <td>100</td>
      <td>13.22</td>
      <td>68</td>
    </tr>
    <tr>
      <th>1260</th>
      <td>hobart</td>
      <td>au</td>
      <td>-44.918038</td>
      <td>149.438144</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>59.00</td>
      <td>82</td>
      <td>10.29</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1261</th>
      <td>butaritari</td>
      <td>ki</td>
      <td>19.152045</td>
      <td>166.379042</td>
      <td>1511277128</td>
      <td>11-21-2017</td>
      <td>83.41</td>
      <td>100</td>
      <td>0.36</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1262</th>
      <td>albany</td>
      <td>au</td>
      <td>-66.306087</td>
      <td>102.954271</td>
      <td>1511277108</td>
      <td>11-21-2017</td>
      <td>51.15</td>
      <td>79</td>
      <td>5.73</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1263</th>
      <td>albany</td>
      <td>au</td>
      <td>-71.014724</td>
      <td>115.868380</td>
      <td>1511277108</td>
      <td>11-21-2017</td>
      <td>51.15</td>
      <td>79</td>
      <td>5.73</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1264</th>
      <td>guerrero negro</td>
      <td>mx</td>
      <td>23.325205</td>
      <td>-118.888749</td>
      <td>1511277409</td>
      <td>11-21-2017</td>
      <td>54.12</td>
      <td>92</td>
      <td>2.48</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1265</th>
      <td>qaanaaq</td>
      <td>gl</td>
      <td>87.968177</td>
      <td>-70.938808</td>
      <td>1511277083</td>
      <td>11-21-2017</td>
      <td>9.84</td>
      <td>100</td>
      <td>18.70</td>
      <td>80</td>
    </tr>
    <tr>
      <th>1266</th>
      <td>ushuaia</td>
      <td>ar</td>
      <td>-79.538653</td>
      <td>-26.688245</td>
      <td>1511277050</td>
      <td>11-21-2017</td>
      <td>43.36</td>
      <td>81</td>
      <td>10.76</td>
      <td>92</td>
    </tr>
    <tr>
      <th>1267</th>
      <td>tiksi</td>
      <td>ru</td>
      <td>83.619581</td>
      <td>123.983572</td>
      <td>1511277135</td>
      <td>11-21-2017</td>
      <td>1.83</td>
      <td>79</td>
      <td>23.06</td>
      <td>56</td>
    </tr>
    <tr>
      <th>1268</th>
      <td>olinda</td>
      <td>br</td>
      <td>-11.516064</td>
      <td>-25.726908</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>78.80</td>
      <td>83</td>
      <td>12.75</td>
      <td>75</td>
    </tr>
    <tr>
      <th>1269</th>
      <td>mamit</td>
      <td>in</td>
      <td>23.866585</td>
      <td>92.360011</td>
      <td>1511277410</td>
      <td>11-21-2017</td>
      <td>65.86</td>
      <td>91</td>
      <td>1.92</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1270</th>
      <td>bluff</td>
      <td>nz</td>
      <td>-74.987515</td>
      <td>167.762908</td>
      <td>1511277050</td>
      <td>11-21-2017</td>
      <td>53.17</td>
      <td>100</td>
      <td>1.59</td>
      <td>44</td>
    </tr>
    <tr>
      <th>1271</th>
      <td>pisco</td>
      <td>pe</td>
      <td>-22.996956</td>
      <td>-95.069028</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>68.00</td>
      <td>77</td>
      <td>6.93</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1272</th>
      <td>seoul</td>
      <td>kr</td>
      <td>35.044780</td>
      <td>127.395002</td>
      <td>1511276160</td>
      <td>11-21-2017</td>
      <td>33.48</td>
      <td>86</td>
      <td>3.04</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1273</th>
      <td>port elizabeth</td>
      <td>za</td>
      <td>-84.320171</td>
      <td>31.723392</td>
      <td>1511272800</td>
      <td>11-21-2017</td>
      <td>64.40</td>
      <td>68</td>
      <td>24.16</td>
      <td>40</td>
    </tr>
  </tbody>
</table>
<p>1274 rows × 10 columns</p>
</div>




```python
# Count number of records found

city_count = len(summary_df)

print(f'Number of cities: {city_count}')
```

    Number of cities: 1274



```python
# Save DataFrame as a csv

summary_df.to_csv("OpenWeatherMap-analysis.csv", encoding="utf-8", index=False)
```

# Latitude vs. Temperature Plot


```python
# Create scatterplot

plt.scatter(summary_df['Latitude'], 
            summary_df['Max Temperature'], color='darkblue')

# Set textual properties

plt.title(f'City Latitude vs. Max Temperature')
plt.xlabel("Latitude")
plt.ylabel("Max Temperature (F)")
plt.grid(True)

plt.show()
```


![png](output_9_0.png)


# Latitude vs. Humidity Plot


```python
# Create scatterplot

plt.scatter(summary_df['Latitude'], 
            summary_df['Humidity'], color='darkblue')

# Set textual properties

plt.title("City Latitude vs. Humidity")
plt.xlabel("Latitude")
plt.ylabel("Humidity (%)")
plt.grid(True)

plt.show()
```


![png](output_11_0.png)


# Latitude vs. Cloudiness Plot


```python
# Create scatterplot

plt.scatter(summary_df['Latitude'], 
            summary_df['Cloudiness'], color='darkblue')

# Set textual properties

plt.title("City Latitude vs. Cloudiness")
plt.xlabel("Latitude")
plt.ylabel("Cloudiness (%)")
plt.grid(True)

plt.show()
```


![png](output_13_0.png)


# Latitude vs. Windspeed Plot


```python
# Create scatterplot

plt.scatter(summary_df['Latitude'], 
            summary_df['Windspeed'], color='darkblue')

# Set textual properties

plt.title("City Latitude vs. Windspeed")
plt.xlabel("Latitude")
plt.ylabel("Windspeed (mph)")
plt.grid(True)

plt.show()
```


![png](output_15_0.png)

