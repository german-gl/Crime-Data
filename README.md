# Crime-Data

Utilized Python, Pandas and Folium to extract Boston, MA city data to create an animated heatmap displaying over 30,000 crime hotspots since June 1, 2015.  


crime.ipynb
crime.ipynb_Notebook unstarred
Crime Zone Heatmaps with Python and Folium
Initialize a Folium map

Additional tile providers: http://leaflet-extras.github.io/leaflet-providers/preview/

[ ]
import folium
boston_lat_lon = [ 42.3202, -71.1500]
m = folium.Map(
    location = boston_lat_lon,
    zoom_start = 11,
    titles = 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
    attr = 'Map data: &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)',
)
m

Import data from file

[ ]
import pandas as pd
df = pd.read_csv("boston_crime.csv")
print(df)
       INCIDENT_NUMBER  OFFENSE_CODE    OFFENSE_CODE_GROUP  \
0           I182070945           619               Larceny   
1           I182070943          1402             Vandalism   
2           I182070941          3410                 Towed   
3           I182070940          3114  Investigate Property   
4           I182070938          3114  Investigate Property   
...                ...           ...                   ...   
319068   I050310906-00          3125       Warrant Arrests   
319069   I030217815-08           111              Homicide   
319070   I030217815-08          3125       Warrant Arrests   
319071   I010370257-00          3125       Warrant Arrests   
319072       142052550          3125       Warrant Arrests   

                        OFFENSE_DESCRIPTION DISTRICT REPORTING_AREA SHOOTING  \
0                        LARCENY ALL OTHERS      D14            808      NaN   
1                                 VANDALISM      C11            347      NaN   
2                       TOWED MOTOR VEHICLE       D4            151      NaN   
3                      INVESTIGATE PROPERTY       D4            272      NaN   
4                      INVESTIGATE PROPERTY       B3            421      NaN   
...                                     ...      ...            ...      ...   
319068                       WARRANT ARREST       D4            285      NaN   
319069  MURDER, NON-NEGLIGIENT MANSLAUGHTER      E18            520      NaN   
319070                       WARRANT ARREST      E18            520      NaN   
319071                       WARRANT ARREST      E13            569      NaN   
319072                       WARRANT ARREST       D4            903      NaN   

           OCCURRED_ON_DATE  YEAR  MONTH DAY_OF_WEEK  HOUR    UCR_PART  \
0       2018-09-02 13:00:00  2018      9      Sunday    13    Part One   
1       2018-08-21 00:00:00  2018      8     Tuesday     0    Part Two   
2       2018-09-03 19:27:00  2018      9      Monday    19  Part Three   
3       2018-09-03 21:16:00  2018      9      Monday    21  Part Three   
4       2018-09-03 21:05:00  2018      9      Monday    21  Part Three   
...                     ...   ...    ...         ...   ...         ...   
319068  2016-06-05 17:25:00  2016      6      Sunday    17  Part Three   
319069  2015-07-09 13:38:00  2015      7    Thursday    13    Part One   
319070  2015-07-09 13:38:00  2015      7    Thursday    13  Part Three   
319071  2016-05-31 19:35:00  2016      5     Tuesday    19  Part Three   
319072  2015-06-22 00:12:00  2015      6      Monday     0  Part Three   

                   STREET        Lat       Long                     Location  
0              LINCOLN ST  42.357791 -71.139371  (42.35779134, -71.13937053)  
1                HECLA ST  42.306821 -71.060300  (42.30682138, -71.06030035)  
2             CAZENOVE ST  42.346589 -71.072429  (42.34658879, -71.07242943)  
3              NEWCOMB ST  42.334182 -71.078664  (42.33418175, -71.07866441)  
4                DELHI ST  42.275365 -71.090361  (42.27536542, -71.09036101)  
...                   ...        ...        ...                          ...  
319068        COVENTRY ST  42.336951 -71.085748  (42.33695098, -71.08574813)  
319069           RIVER ST  42.255926 -71.123172  (42.25592648, -71.12317207)  
319070           RIVER ST  42.255926 -71.123172  (42.25592648, -71.12317207)  
319071  NEW WASHINGTON ST  42.302333 -71.111565  (42.30233307, -71.11156487)  
319072      WASHINGTON ST  42.333839 -71.080290  (42.33383935, -71.08029038)  

[319073 rows x 17 columns]
Access dataframe values

[ ]
#df.head()
#df['DISTRICT']
#df.DISTRICT
#type(df.DISTRICT)
#df.DISTRICT[3]
#df.iloc[0,4]
#df.DISTRICT.value_counts()
#df.lat.describe()
#df.lat / 5.0
#df.DAY_OF_WEEK == "Tuesday"
0         False
1          True
2         False
3         False
4         False
          ...  
319068    False
319069    False
319070    False
319071     True
319072    False
Name: DAY_OF_WEEK, Length: 319073, dtype: bool
Rename or drop columns.

[ ]
columns = {
      'OCCURRED_ON_DATE': 'date',
      'OFFENSE_CODE_GROUP': 'offense',
      'SHOOTING': 'shooting',
      'Lat': 'lat',
      'Long': 'lon',
}
df = df.rename(columns=columns)
df = df[ list(columns.values()) ]
Deal with data types

[ ]
type(df.date[0])
df.date = pd.to_datetime(df.date)
df = df.sort_values(by = "date")
print(df.date[0:10])
129056   2015-06-15 00:00:00
314676   2015-06-15 00:00:00
310350   2015-06-15 00:00:00
253464   2015-06-15 00:00:00
8793     2015-06-15 00:00:00
318414   2015-06-15 00:00:00
317446   2015-06-15 00:00:00
303001   2015-06-15 00:00:00
317447   2015-06-15 00:00:00
318621   2015-06-15 00:01:00
Name: date, dtype: datetime64[ns]
Deal with null values.

[ ]
df.shooting = (df.shooting == "Y")
df = df.dropna()
df.head()

Group data by month

[ ]
import datetime
from dateutil.relativedelta import relativedelta

months = []
start = datetime.datetime(2015, 6, 1)
while start < df.date.max():
  end = start + relativedelta(months=+1)
  mask = (start <= df.date) & (df.date < end)
  df_month = df[mask]
  df_month = df_month[ ['lat', 'lon']]
  months.append(df_month)
  start = end
print(months[0])    

#Importing the datetime module from the Python Standard Library and the relativedelta function from the dateutil library.

#Initializing an empty list months to store the data for each month.

#Creating a starting date start of June 1st, 2015.

#Using a while loop to iterate through each month until the maximum date in the 'date' column of the DataFrame df is reached.

#For each iteration of the loop, the end of the month is calculated as start + relativedelta(months=+1) using the relativedelta function. This allows the code to find the end of the month for any given start date.

#The next step is to create a boolean mask mask that selects all rows where the 'date' column is greater than or equal to start and less than end.

#The data for each month is extracted from the DataFrame df using the mask, and stored in a new DataFrame df_month containing only the 'lat' and 'lon' columns.

#The data for each month is then appended to the list months.

#The starting date is updated to be the end of the previous month, and the loop continues until the maximum date in the 'date' column of the DataFrame df is reached.

#Finally, the first month of data is printed using print(months[0]).

#This code segments the data in the DataFrame df into separate dataframes for each month and stores them in a list. The data for each month is stored in a new DataFrame df_month that contains only the 'lat' and 'lon' columns 
                        
              lat        lon
129056  42.291093 -71.065945
314676  42.300217 -71.080979
310350  42.293606 -71.071887
253464  42.283634 -71.082813
8793    -1.000000  -1.000000
...           ...        ...
314737  42.380275 -71.060377
314736  42.380275 -71.060377
314735  42.380275 -71.060377
314708  42.288705 -71.078108
314724  42.280587 -71.074322

[4066 rows x 2 columns]
Create heatmap

[ ]
from folium.plugins import HeatMapWithTime

m = folium.Map(boston_lat_lon, zoom_start=11)
hm = HeatMapWithTime(
    data = [ m.values.tolist() for m in months],
    radius = 5,
    max_opacity = 0.5,
    auto_play = False,
)
hm.add_to(m)
m

Spatially clustered data

[ ]
LAT_LON_GRID = 0.005

def custom_round(val, resolution):
  return round(val / resolution) * resolution

def cluster(df_interval):
  data = df_interval.copy()
  data = custom_round(data, LAT_LON_GRID)
  data = data.groupby(["lat", "lon"]).size().reset_index(name = "weight")
  data.weight = data.weight / data.weight.max()
  return data

start = datetime.datetime(2015, 6, 1)
end = start + relativedelta(months=+1)
mask = (start <= df.date) & (df.date < end)
df_month = df[mask]
df_month = df_month[ ['lat', 'lon']]

print(cluster(df_month))
        lat     lon    weight
0    -1.000  -1.000  0.066667
1    42.235 -71.140  0.013333
2    42.235 -71.125  0.013333
3    42.240 -71.140  0.053333
4    42.240 -71.125  0.066667
..      ...     ...       ...
436  42.390 -71.015  0.013333
437  42.390 -71.010  0.093333
438  42.390 -71.005  0.080000
439  42.390 -71.000  0.040000
440  42.395 -71.010  0.080000

[441 rows x 3 columns]
Redraw heat map

[ ]
from folium.plugins import HeatMapWithTime

m = folium.Map(boston_lat_lon, zoom_start=11)
hm = HeatMapWithTime(
    data = [ cluster(m).values.tolist() for m in months],
    radius = 15,
    max_opacity = 0.5,
    auto_play = False,
)
hm.add_to(m)
m
