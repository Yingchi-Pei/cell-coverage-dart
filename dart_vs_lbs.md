
# Cell Coverage by DART

## Introduction
This analysis effort is to study how the cell coverage (or footfall distribution?, we don't have an accurate name for this) varies for each cell tower in different geo-location and time period.

To do this, we match our MySingtel DART data with LBS data, so that we can have the GPS lat lon for each LBS record.

The matching criteria is by `IMSI` and `DateTime`. As we study on daily level, to ensure we have enough records for analysis, we keep in all the records if the time difference between DART and LBS is no more than 10 seconds.

## Code

Import required packages and data

```python
import pandas as pd
from glob import glob
from bokeh.io import output_file, output_notebook, show
from bokeh.models import(
    GMapPlot, GMapOptions, ColumnDataSource, Circle, Triangle, LogColorMapper, BasicTicker, ColorBar,
    DataRange1d, PanTool, WheelZoomTool, BoxZoomTool, ZoomInTool
)
from bokeh.models.mappers import ColorMapper, LinearColorMapper
from bokeh.palettes import Viridis5

# doi: date of interest
doi = '20170823'
files = glob('data/weekday/merged_top20cid_'+doi+'.csv')
df_from_each_file = (pd.read_csv(f, header=0) for f in files)
df = pd.concat(df_from_each_file, ignore_index=True)
```

Drop useless column

```python
df.drop(df.columns[0],axis=1,inplace=True)
print(df.imsi.count())
df.columns = ['imsi', 'time_lbs', 'cellid', 'event', 'lat_lbs', 'lon_lbs',
              'time_dart', 'lon_dart', 'lat_dart', 'diff_sec']
df.head(2)
```

    943895


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>imsi</th>
      <th>time_lbs</th>
      <th>cellid</th>
      <th>event</th>
      <th>lat_lbs</th>
      <th>lon_lbs</th>
      <th>time_dart</th>
      <th>lon_dart</th>
      <th>lat_dart</th>
      <th>diff_sec</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>52501602470C20002AF77E3337200CAC8146F9</td>
      <td>2017-08-23 18:45:34</td>
      <td>525-1-746-7369675</td>
      <td>200</td>
      <td>1.440519762</td>
      <td>103.8032032</td>
      <td>2017-08-23 18:47:20</td>
      <td>103.800926</td>
      <td>1.440365</td>
      <td>106</td>
    </tr>
    <tr>
      <th>1</th>
      <td>52501602470C20002AF77E3337200CAC8146F9</td>
      <td>2017-08-23 18:45:53</td>
      <td>525-1-746-7369675</td>
      <td>200</td>
      <td>1.440519762</td>
      <td>103.8032032</td>
      <td>2017-08-23 18:47:20</td>
      <td>103.800926</td>
      <td>1.440365</td>
      <td>87</td>
    </tr>
  </tbody>
</table>
</div>


Only keep the records with time difference <= 10 seconds:

```python
df = df[(df.diff_sec<=10) & (df.diff_sec >=-10)]
```


```python
print(df.imsi.count())
df_sortcell = pd.DataFrame({'count' : df.groupby(["lat_lbs", "lon_lbs", "cellid"]).size()}).reset_index()
```

    102785



```python
df_sortcell = df_sortcell.sort_values(['count'], ascending=False).reset_index(drop=True)
```


```python
df_sortcell.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lat_lbs</th>
      <th>lon_lbs</th>
      <th>cellid</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.343174</td>
      <td>103.86069</td>
      <td>525-1-747-7316971</td>
      <td>2832</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.45013459</td>
      <td>103.8100161</td>
      <td>525-1-710-7362835</td>
      <td>2502</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.428977</td>
      <td>103.834083</td>
      <td>525-1-710-7371472</td>
      <td>2482</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.330331</td>
      <td>103.741301</td>
      <td>525-1-748-7334573</td>
      <td>1983</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.317568052</td>
      <td>103.8509492</td>
      <td>525-1-713-7390973</td>
      <td>1953</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.315338</td>
      <td>103.765611</td>
      <td>525-1-748-7487372</td>
      <td>1940</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.350719201</td>
      <td>103.852005</td>
      <td>525-1-714-7383831</td>
      <td>1902</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1.384033633</td>
      <td>103.743502</td>
      <td>525-1-746-7364732</td>
      <td>1883</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.325046</td>
      <td>103.890315</td>
      <td>525-1-747-7311373</td>
      <td>1760</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1.349346586</td>
      <td>103.7404333</td>
      <td>525-1-746-7483872</td>
      <td>1652</td>
    </tr>
  </tbody>
</table>
</div>

Keep the Top 10 cell towers for visualisation.

If you want to investigate on specific cell tower, you can modify the index number here:

```python
cells = list(df['cellid'].value_counts()[:10].index)
print(cells)
```

    ['525-1-747-7316971', '525-1-710-7362835', '525-1-710-7371472', '525-1-748-7334573', '525-1-713-7390973', '525-1-748-7487372', '525-1-714-7383831', '525-1-746-7364732', '525-1-747-7311373', '525-1-746-7483872']


```python
df_use = df[df['cellid'].isin(cells)]
```

Plot the map using `bokeh`

```python
import random
def get_spaced_colors(n):
    def rgb_to_hex(rgb):
        return '#%02x%02x%02x' % rgb
    max_value = 16500000 #255**3
    interval = int(max_value / n)
    # (0,0,0) is black, so we want to avoid black by starting from 10
    colors = [hex(I)[2:].zfill(6) for I in range(1000, max_value, interval)]
    return [rgb_to_hex((int(i[:2], 16), int(i[2:4], 16), int(i[4:], 16))) for i in colors]

colors = get_spaced_colors(len(cells))
cells_colors = dict(zip(cells, colors))
df_use['colors'] = df_use['cellid'].map(lambda x: cells_colors[x])
```

```python
df_lbs = df_use[['cellid', 'lat_lbs', 'lon_lbs', 'colors']]
```

Footfall distribution for the whole day 

```python
map_options = GMapOptions(lat=1.353, lng=103.83, map_type="roadmap", zoom=12)

def plot_df(df_dart, df_lbs, title='Dart plot'):
    p_day = GMapPlot(
        x_range=DataRange1d(), y_range=DataRange1d(), map_options=map_options,
        plot_width=950, plot_height=700
    )
    p_day.title.text = title

    p_day.api_key = "AIzaSyDpUETNSv04jmgUFKfv-fGqX2U1e1fwUsw"
    source_dart = ColumnDataSource(
        data=dict(
            lat=df_dart.lat_dart.tolist(),
            lon=df_dart.lon_dart.tolist(),
            color=df_dart.colors
        )
    )

    circle = Circle(x="lon", y="lat", size=5, fill_color='color', fill_alpha=0.3, line_color=None)
    source_lbs = ColumnDataSource(
        data=dict(
            lat=df_lbs.lat_lbs.tolist(),
            lon=df_lbs.lon_lbs.tolist(),
            color=df_lbs.colors
        )
    )

    triangle = Triangle(x="lon", y="lat", size=14, fill_color='color', fill_alpha=0.8, line_color='black')
    p_day.add_glyph(source_dart, circle)
    p_day.add_glyph(source_lbs, triangle)
    p_day.add_tools(PanTool(), WheelZoomTool(), ZoomInTool())
    #output_file("gmap_plot.html")
    output_notebook()
    show(p_day)

plot_df(df_use, df_lbs, 'MySingtel Dart Data vs. LBS on '+doi)

```
![20170823-all](pics/20170823_all.png?raw=true)

```python
import datetime
def within_time(start, end, time_str):
    curr_time = time_str.split(' ')[1].strip()
    curr_time = datetime.datetime.strptime(curr_time,'%H:%M:%S').time()
    if start <= end:
        return start <= curr_time <= end
    else:
        return start <= curr_time or curr_time <= end

start_time = datetime.time(10, 0, 0)
end_time = datetime.time(16, 0, 0)
```
Distribution from 10am to 4pm (Day)

```python
df_daytime = df_use[df_use.apply(lambda x: within_time(start_time, end_time, x['time_dart']), axis=1)]
map_options = GMapOptions(lat=1.34, lng=103.75, map_type="roadmap", zoom=14)
plot_df(df_daytime, df_lbs, title='Dart in Daytime 10am - 4pm')
```
![20170823-day](pics/20170823_day.png?raw=true)

Distribution from 10pm to 4am (Night)

```python
start_time = datetime.time(22, 0, 0)
end_time = datetime.time(4, 0, 0)
df_nighttime = df_use[df_use.apply(lambda x: within_time(start_time, end_time, x['time_dart']), axis=1)]
plot_df(df_nighttime, df_lbs, title='Dart in Nighttime 10pm - 4am')
```
![20170823-night](pics/20170823_night.png?raw=true)

## Insights
1. During daytime, both the footfall count and the distribution area tends to be larger than night
2. The geo-location of the cell tower also matters a lot. For example, if it's near a large mrt station, the weekday vs. weekend difference is not very large. But for a cell tower in residential area (many HDBs), the weekday vs. weekend distribution is quite obvious.

For some cell tower level demo, please refer to the confluence page: [20170924 Cell Coverage by Mysingtel Dart Data](https://dataspark.atlassian.net/wiki/spaces/DSS/pages/104300584/20170924+Cell+Coverage+by+Mysingtel+Dart+Data) 


## Limitations

1. The time match is not exact match. If the IMSI device is moving rapidly (e.g. on express way), the error from the dart location will be large.
2. The dart data I use to match with LBS is shixin's transformed dart data (on staging `hdfs:/user/luoshixin/dart_transformed/`). MySingtel DART data is quite messy, i.e. today's dart data may contain the data from one month ago. Shixin's transformed data take into 3 days' raw dart data to get output for 1 day. 
