import datetime
import math
import os
import forecastio
from geopy.geocoders import Nominatim
import pandas as pd
geolocator = Nominatim()
loc=raw_input(" Enter the location:")
location = geolocator.geocode(loc)
lat = location.latitude
lng = -location.longitude
api_key = "beff9da718d4a5c0e6da54235b680e0b"
attributes = ["windSpeed","windBearing"]
times = []
data = {}
for attr in attributes:
    data[attr] = []
year=int(raw_input("year:"))
start = datetime.datetime(year, 1, 1)

d1 = datetime.date(year, 1, 1)
d2 = datetime.date(year + 1, 1, 1)
days= int((d2 - d1).days)#finds the number of days based on leap year
print days

for offset in range(0, days):
    forecast = forecastio.load_forecast(api_key, lat, lng, time=start+datetime.timedelta(offset), units="us")
    h = forecast.daily()
    d = h.data
    for p in d:
        times.append(p.time)
        for attr in attributes:
            result=data[attr].append(p.d[attr])
df = pd.DataFrame(data, index=times)
windavg= df["windSpeed"].mean()
wind= df["windSpeed"]
direction=df["windBearing"]
a= list(wind)
b=list(direction)
myquadrant=[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0] #Test the Cross-over point
dat=list(zip(a,b))

for d in dat:
    ind = int(((d[1] / 360) * 16) )
    myquadrant[ind] += d[0]
sumSin=0.0
sumCos=0.0
sumDir=0.0
pi=math.pi
pi8=math.pi/8.0
for i in range(0,16):
    sumDir=sumDir+myquadrant[i]
    sumSin=sumSin+math.sin(myquadrant[i]*pi8)
    sumCos=sumCos+math.cos(myquadrant[i]*pi8)
avSin=sumSin/16.0
avCos=sumCos/16.0
avDir=sumDir/16.0
DirectionAvg=int(avDir*22.5/10)
print DirectionAvg
print windavg
columns = ['DirectionAvg','windavg','Location','Year']
index=['values:']
df1 = pd.DataFrame({"DirectionAvg":DirectionAvg,"windavg":windavg,"Location":loc,"Year":year},index=index)
df1.to_csv('forecast.csv', mode='a', header=False)
